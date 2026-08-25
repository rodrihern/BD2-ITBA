# TP6 Restricciones declarativas

## Ejercicio 1

![](attachments/Pasted%20image%2020260825190729.png)![](attachments/Pasted%20image%2020260825190742.png)![](attachments/Pasted%20image%2020260825190752.png)

a.
```sql
ALTER TABLE trabaja_en
ADD CONSTRAINT fk_trabaja_empleado
FOREIGN KEY (TipoE, NroE)
REFERENCES empleado(TipoE, NroE)
ON DELETE CASCADE
ON UPDATE RESTRICT;

ALTER TABLE trabaja_en
ADD CONSTRAINT fk_trabaja_proyecto
FOREIGN KEY (IdProy)
REFERENCES pryecto(IdProy)
ON DELETE RESTRICT
ON UPDATE CASCADE;

ALTER TABLE auspicio
ADD CONSTRAINT fk_auspicio_proyecto
FOREIGN KEY (IdProy)
REFERENCES pryecto(IdProy)
ON DELETE RESTRICT
ON UPDATE RESTRICT;

ALTER TABLE auspicio
ADD CONSTRAINT fk_auspicio_empleado
FOREIGN KEY (TipoE, NroE)
REFERENCES empleado(TipoE, NroE)
ON DELETE SET NULL
ON UPDATE RESTRICT;
```

b.
i. se borra, nadie lo referencia
ii. lo mismo nadie referencia al proyecto 3
iii. no se realiza por el restrict del on delete de la tabla trabaja_en
iv. se borra la row de trabaja_en por el cascade y se setea en null la row de auspicio
v. se acepta, ahora las que referenciaban al proyecto 1 referencian al 3
vi. Se acepta y se actualiza la tupla en trabaja_en porque tiene cascade

c.

Acepta?

|     | Simple | Parcial | full |
| --- | ------ | ------- | ---- |
| i   | si     | si      | no   |
| ii  | si     | si      | si   |
| iii | no     | no      | no   |
| iv  | si     | no      | no   |

## Ejercicio 2

![](attachments/Pasted%20image%2020260825192337.png)![](attachments/Pasted%20image%2020260825192357.png)![](attachments/Pasted%20image%2020260825192407.png)![](attachments/Pasted%20image%2020260825192426.png)

a.
i. borra las de cliente y las de instalacion por el cascade
ii. no se puede porque no hay una con idServ S5 para referenciar
iii. se hace tranqui porque nadie lo referencia con la D
iv. noup pues lo referencia la tabla REFERENCIA y tiene restrict
v. no por el restrict de INSTALACION

b.
TLT
Esto puso nash

```sql
INSERT INTO REFERENCIA VALUES ('S1','Caso1', NULL, NULL); 
-- ⇒ Simple, Parcial y Full
-- Ambos NULL. Simple no chequea; Parcial/Full permiten “todo NULL”.

INSERT INTO REFERENCIA VALUES ('S1','Caso2', 'D', NULL); 
-- ⇒ Simple, Parcial y no Full
-- Hay mixta ('D', NULL). Parcial exige que exista algún cliente con Zona='D' (sí: (D,3)); Full prohíbe mixtas.

INSERT INTO REFERENCIA VALUES ('S1','Caso4', 'Z', NULL); 
-- ⇒ Simple, no Parcial y no Full
-- Zona='Z' no existe. Simple pasa por tener NULL; Parcial exige coincidencia en Zona (falla); Full prohíbe mixtas.

INSERT INTO REFERENCIA VALUES ('S1','Caso7', 'A', 3); 
-- ⇒ No Simple, no Parcial y no Full
-- Ambos no nulos pero no existe (A,3).
```

## Ejercicio 3

![](attachments/Pasted%20image%2020260825193738.png)
![](attachments/Pasted%20image%2020260825193754.png)![](attachments/Pasted%20image%2020260825193817.png)![](attachments/Pasted%20image%2020260825193828.png)

a.

| Restriccion | Tabla/s            | Atributo/s                                     | Tipo        | Recurso   |
| ----------- | ------------------ | ---------------------------------------------- | ----------- | --------- |
| A1          | articulo           | nacionallidad                                  | de atributo | CHECK     |
| A2          | articulo           | fecha_pub                                      | de atributo | CHECK     |
| A3          | articulo           | fecha_pub, nacionalidad                        | de tupla    | CHECK     |
| A4          | contiene           | id_articulo, idioma, cod_palabra               | de tabla    | ASSERTION |
| A5          | articulo, contiene | id_articulo, nacionalidad, idioma, cod_palabra | de db       | ASSERTION |
| ...         |                    |                                                |             |           |
TLT

por nash

```sql
-- A1
ALTER TABLE ARTICULO
        ADD CONSTRAINT ck_art_nacionalidad
        CHECK (nacionalidad IN ('Argentino','Español','Inglés','Alemán','Chileno'));
        
-- A2
ALTER TABLE ARTICULO
        ADD CONSTRAINT ck_art_fecha_pub_min
        CHECK (fecha_pub >= DATE '2010-01-01');
        
-- A3
ALTER TABLE ARTICULO
        ADD CONSTRAINT ck_art_2017_solo_arg
        CHECK (EXTRACT(YEAR FROM fecha_pub) <> 2017
                OR nacionalidad = 'Argentino');
                
-- A4
CREATE ASSERTION as_art_max_10_palabras
        CHECK (
        NOT EXISTS (
            SELECT 1
            FROM CONTIENE c
            GROUP BY c.id_articulo
            HAVING COUNT(*) > 10
        )
    );
    
-- A5
CREATE ASSERTION as_art_mas10_solo_arg
CHECK (
  NOT EXISTS (
    SELECT 1
    FROM ARTICULO a
    JOIN CONTIENE c ON c.id_articulo = a.id_articulo
    WHERE a.nacionalidad <> 'Argentino'
    GROUP BY a.id_articulo, a.nacionalidad
    HAVING COUNT(*) > 10
  )
);

CREATE ASSERTION as_art_tope_15
CHECK (
  NOT EXISTS (
    SELECT 1
    FROM CONTIENE c
    GROUP BY c.id_articulo
    HAVING COUNT(*) > 15
  )
);
```

de baja el resto
TLT