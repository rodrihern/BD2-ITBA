
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

El minimo sueldo en la tabla empleado seria $300$

$300 \cdot 0.05 = 15$ 

### a)

Correria 3 veces por lo que las tuplas quedarian

| id_empleado | sueldo                   |
| ----------- | ------------------------ |
| 100         | $500 - 3 \cdot 15 = 445$ |
| 200         | $600 - 3 \cdot 15 = 545$ |

### b) 

correria una vez por lo que quedarian

| id_empleado | sueldo           |
| ----------- | ---------------- |
| 100         | $500 - 15 = 485$ |
| 200         | $600 - 15= 585$  |

## Ejercicio 3

![](attachments/Pasted%20image%2020260909172135.png)