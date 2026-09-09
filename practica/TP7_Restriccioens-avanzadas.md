
# TP 7 Restricciones avanzadas

## Ejercicio 1

![](attachments/Pasted%20image%2020260909163828.png)

### a.

```sql
CREATE TABLE HIS_ENTREGA (
    id_log INT PRIMARY KEY AUTO_INCREMENT,
    fecha_op TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    operacion ENUM('INSERT','UPDATE','DELETE') NOT NULL,
    usuario  VARCHAR(100) NOT NULL
    -- y ahora agregamos data para identificar sobre que fila fue hecho y que se hizo
    tabla_afectada ENUM('ENTREGA', 'RENGLON_ENTREGA')
    nro_entrega numeric(10, 2) NOT NULL, -- no pongo fk porque si lo borre no tengo a que referenciar bro
);
```

### b.

3 operaciones $\times$ 2 tablas = 6 triggers

Creamos un procedure para reutilizar

```sql
DELIMITER $$

CREATE PROCEDURE sp_log_entrega(
    IN p_operacion  ENUM('INSERT','UPDATE','DELETE'),
    IN p_tabla      ENUM('ENTREGA','RENGLON_ENTREGA'),
    IN p_nro        NUMERIC(10,0),
)
BEGIN
    INSERT INTO his_entrega (operacion, usuario, tabla_afectada,
                             nro_entrega)
    VALUES (p_operacion, CURRENT_USER(), p_tabla, p_nro);
END$$
```

Creamos los triggers

```sql
CREATE TRIGGER tr_entrega_ai AFTER INSERT ON entrega
FOR EACH ROW CALL sp_log_entrega('INSERT', 'ENTREGA', NEW.nro_entrega)$$

CREATE TRIGGER tr_entrega_au AFTER UPDATE ON entrega
FOR EACH ROW CALL sp_log_entrega('UPDATE', 'ENTREGA', NEW.nro_entrega)$$

CREATE TRIGGER tr_entrega_ad AFTER DELETE ON entrega
FOR EACH ROW CALL sp_log_entrega('DELETE', 'ENTREGA', OLD.nro_entrega)$$

CREATE TRIGGER tr_renglon_ai AFTER INSERT ON renglon_entrega
FOR EACH ROW CALL sp_log_entrega('INSERT', 'RENGLON_ENTREGA', NEW.nro_entrega)$$

CREATE TRIGGER tr_renglon_au AFTER UPDATE ON renglon_entrega
FOR EACH ROW CALL sp_log_entrega('UPDATE', 'RENGLON_ENTREGA', NEW.nro_entrega)$$

CREATE TRIGGER tr_renglon_ad AFTER DELETE ON renglon_entrega
FOR EACH ROW CALL sp_log_entrega('DELETE', 'RENGLON_ENTREGA', OLD.nro_entrega)$$

DELIMITER ;
```

### c.

`FOR EACH ROW` corre por cada fila afectada. `FOR EACH STATEMENT` corre una sola vez por sentencia.

si una sentencia afecta 4 filas:

`FOR EACH ROW` $\rightarrow$ corre 4 veces
`FOR EACH STATEMENT` $\rightarrow$ corre 1 vez


## Ejercicio 2

![](attachments/Pasted%20image%2020260909171334.png)

El minimo sueldo en la tabla empleado_1 seria $300$

### a)

Correria 3 veces. Veamos el minimo sueldo por cada insercion


| tupla insertada | min(sueldo) |
| --------------- | ----------- |
| (1, 700)        | 700         |
| (2, 300)        | 300         |
| (3, 300)        | 300         |

luego a cada sueldo se le debera restar $(700 + 2 \cdot 300) \cdot 0.05 = 65$

| id_empleado | sueldo |
| ----------- | ------ |
| 100         | 435    |
| 200         | 635    |

### b) 

correria una vez, siendo el minimo sueldo 300. luego abria que restar $300 \cdot 0.05 = 15$

| id_empleado | sueldo |
| ----------- | ------ |
| 100         | 485    |
| 200         | 585    |

## Ejercicio 3

![](attachments/Pasted%20image%2020260909172135.png)

### a.

Tendriamos que tener un record de cuando se inserto el empleado, y cada vez que cambio de departamento.

Para lo primero basta con hacer

```sql
ALTER TABLE empleado
    ADD COLUMN fecha_alta DATE NOT NULL DEFAULT (CURRENT_DATE);
```

(si no existiese un default podemos agregar un trigger en el insert para que setee eso)

Y luego otra tabla para registrar los movimientos de cuando se cambio de departamento

```sql
CREATE TABLE mov_empleado (
	id_movimiento INT AUTO_INCREMENT PRIMARY KEY,
	id_empleado NUMERIC(6,0) NOT NULL,
	id_distribuidor NUMERIC(5,0) NULL,
	id_departamento NUMERIC(4,0) NULL,
	fecha_desde DATE NOT NULL,
	fecha_hasta DATE NULL, -- NULL = tramo en curso
	CONSTRAINT chk_mov_fechas CHECK (fecha_hasta IS NULL OR fecha_hasta >= fecha_desde)

);
```

### b.

```sql
CREATE TABLE his_empleado (
	id_empleado NUMERIC(6,0) PRIMARY KEY,
	fecha_alta DATE NOT NULL,
	tiempo_promedio DECIMAL(10,2) NULL, -- días promedio por departamento
);
```

### c.

Van 3 triggers sobre `EMPLEADO`: alta, cambio de departamento y baja. Como el recálculo del promedio se repite en dos de ellos, lo separamos en un procedure.

Criterio adoptado: el promedio se calcula **solo sobre los tramos cerrados** (`fecha_hasta IS NOT NULL`). El departamento actual no cuenta hasta que el empleado se va de él. Tampoco cuentan los tramos sin departamento asignado: se exige que las dos columnas del par `(id_distribuidor, id_departamento)` sean NOT NULL, porque el departamento se identifica con el par y la FK compuesta admite que una sola sea NULL (MATCH SIMPLE). Esos tramos son huecos, no permanencias.

```sql
DELIMITER $$

-- Recalcula el promedio de un empleado a partir de sus tramos ya cerrados
CREATE PROCEDURE sp_recalcular_his_empleado(IN p_id_empleado NUMERIC(6,0))
BEGIN
    UPDATE his_empleado h
       SET h.tiempo_promedio = (
             SELECT AVG(DATEDIFF(m.fecha_hasta, m.fecha_desde))
               FROM mov_empleado m
              WHERE m.id_empleado      = p_id_empleado
                AND m.fecha_hasta      IS NOT NULL
                AND m.id_departamento  IS NOT NULL
                AND m.id_distribuidor  IS NOT NULL
           )
     WHERE h.id_empleado = p_id_empleado;
END$$
```

Los triggers:

```sql
-- 1) Alta: nace la fila del historico y se abre el primer tramo
CREATE TRIGGER tr_empleado_ai
AFTER INSERT ON empleado
FOR EACH ROW
BEGIN
    INSERT INTO his_empleado (id_empleado, fecha_alta, tiempo_promedio)
    VALUES (NEW.id_empleado, NEW.fecha_alta, NULL);

    INSERT INTO mov_empleado (id_empleado, id_distribuidor, id_departamento, fecha_desde)
    VALUES (NEW.id_empleado, NEW.id_distribuidor, NEW.id_departamento, NEW.fecha_alta);
END$$

-- 2) Cambio de departamento: cierra el tramo viejo, abre el nuevo y recalcula
CREATE TRIGGER tr_empleado_au
AFTER UPDATE ON empleado
FOR EACH ROW
BEGIN
    IF NOT (NEW.id_departamento <=> OLD.id_departamento AND NEW.id_distribuidor <=> OLD.id_distribuidor)
    THEN
        UPDATE mov_empleado
           SET fecha_hasta = CURRENT_DATE
         WHERE id_empleado = NEW.id_empleado
           AND fecha_hasta IS NULL;

        INSERT INTO mov_empleado (id_empleado, id_distribuidor, id_departamento, fecha_desde)
        VALUES (NEW.id_empleado, NEW.id_distribuidor, NEW.id_departamento, CURRENT_DATE);

        CALL sp_recalcular_his_empleado(NEW.id_empleado);
    END IF;

    -- fecha_alta esta duplicada en his_empleado: hay que mantenerla sincronizada
    IF NOT (NEW.fecha_alta <=> OLD.fecha_alta) THEN
        UPDATE his_empleado SET fecha_alta = NEW.fecha_alta
         WHERE id_empleado = NEW.id_empleado;
    END IF;
END$$

-- 3) Baja: se cierra el ultimo tramo (ahora si cuenta) y se recalcula
CREATE TRIGGER tr_empleado_ad
AFTER DELETE ON empleado
FOR EACH ROW
BEGIN
    UPDATE mov_empleado
       SET fecha_hasta = CURRENT_DATE
     WHERE id_empleado = OLD.id_empleado
       AND fecha_hasta IS NULL;

    CALL sp_recalcular_his_empleado(OLD.id_empleado);
END$$

DELIMITER ;
```

Detalles a justificar:

- **`<=>` y no `=`.** `id_departamento` e `id_distribuidor` admiten NULL. Si un empleado pasa de `NULL` a `2`, la comparacion `NEW.id_departamento <> OLD.id_departamento` da UNKNOWN y el `IF` no entra: el cambio se perderia en silencio. `<=>` es la igualdad null-safe de MySQL (equivale a `IS DISTINCT FROM` de Postgres).
- **No hay FK desde `mov_empleado` ni `his_empleado` hacia `empleado`**, por lo mismo del ejercicio 1: el historico tiene que sobrevivir a la baja.
- **`tiempo_promedio` queda NULL mientras no haya ningun tramo cerrado**, porque `AVG` sobre un conjunto vacio devuelve NULL. Por eso la columna es nullable.

### d.

Un stored procedure hace lo mismo pero **invocado explicitamente por el usuario** (`CALL`), sin depender de ningun evento. Recalcula todo desde cero a partir del dato crudo:

```sql
DELIMITER $$

CREATE PROCEDURE sp_recalcular_todo()
BEGIN
    -- 1) dar de alta en el historico a los empleados que todavia no estan
    INSERT INTO his_empleado (id_empleado, fecha_alta, tiempo_promedio)
    SELECT e.id_empleado, e.fecha_alta, NULL
      FROM empleado e
     WHERE NOT EXISTS (SELECT 1 FROM his_empleado h
                        WHERE h.id_empleado = e.id_empleado);

    -- 2) recalcular todos los promedios desde los tramos cerrados
    UPDATE his_empleado h
       SET h.fecha_alta = COALESCE(
                            (SELECT e.fecha_alta FROM empleado e
                              WHERE e.id_empleado = h.id_empleado),
                            h.fecha_alta),   -- si el empleado fue dado de baja, se conserva
           h.tiempo_promedio = (
             SELECT AVG(DATEDIFF(m.fecha_hasta, m.fecha_desde))
               FROM mov_empleado m
              WHERE m.id_empleado      = h.id_empleado
                AND m.fecha_hasta      IS NOT NULL
                AND m.id_departamento  IS NOT NULL
                AND m.id_distribuidor  IS NOT NULL
           );
END$$

DELIMITER ;
```

Se ejecuta con:

```sql
CALL sp_recalcular_todo();
```

El tiempo total no se guarda: se deriva en el momento de consultarlo, asi nunca queda desactualizado.

```sql
SELECT h.id_empleado,
       DATEDIFF(CURRENT_DATE, h.fecha_alta) AS tiempo_total,
       h.tiempo_promedio
FROM his_empleado h;
```

| | Trigger (c) | Stored procedure (d) |
|---|---|---|
| Quien lo invoca | el DBMS, ante el evento | el usuario, con `CALL` |
| Cuando corre | en el instante del cambio | cuando alguien lo ejecuta |
| Alcance | el empleado afectado | todos, de cero |
| Costo | se paga en cada `UPDATE` | se paga cuando se corre |
### e.

Depende como se haga la consulta. Osea como guardamos la fecha de alta, cuando uno hace la query lo deriva. Si tomamos el tiempo promedio, sin contar el ultimo, entonces es el dato mas actualizado tambien en el caso de la tabla

## Ejercicio 4

![](attachments/Pasted%20image%2020260909185656.png)

tlt