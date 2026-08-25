# TP4 Vistas

## Ej 1

![](attachments/Pasted%20image%2020260814165808.png)

Primero creemos las tablas

```sql
CREATE TABLE proveedor (
	id_proveedor VARCHAR(10) PRIMARY KEY,
	nombre VARCHAR(30) NOT NULL,
	rubro VARCHAR(15) NOT NULL,
	ciudad VARCHAR(30) NOT NULL
);

CREATE TABLE articulo (
	id_articulo VARCHAR(10) PRIMARY KEY,
	descrip VARCHAR(30) NOT NULL,
	peso NUMERIC(5,2) NOT NULL,
	ciudad VARCHAR(30) NOT NULL
);

CREATE TABLE envio (
	id_proveedor VARCHAR(10) NOT NULL,
	id_articulo VARCHAR(10) NOT NULL,
	cantidad NUMERIC(5, 0) NOT NULL,
	
	PRIMARY KEY (id_proveedor, id_articulo),
	
	FOREIGN KEY (id_proveedor) REFERENCES proveedor(id_proveedor),
	FOREIGN KEY (id_articulo) REFERENCES articulo(id_articulo)
	
)
```

ahora vamos con las vistas


```sql
-- a)
CREATE VIEW envios500 AS
SELECT *
FROM envio
WHERE cantidad >= 500;

-- b)
CREATE VIEW envios500_999 AS
SELECT *
FROM envios500
WHERE cantidad <= 999;

-- c)
CREATE VIEW detalle_envios AS
SELECT a.descrip, a.peso, p.nombre, e.cantidad
FROM proveedor p JOIN envio e ON p.id_proveedor = e.id_proveedor
JOIN articulo a ON e.id_articulo = a.id_articulo;
```

![](attachments/Pasted%20image%2020260814172452.png)

i. no hay ningun problema ni con ni sin check option, se va a hacer
ii. Este con CHECK OPTION no se va a hacer pues viola la condicion de pertenencia (no pertenece a esa vista si tiene cantidad = 300)
iii. todo pelota siga siga
iv. mismo problema de la condicion de pertenencia

## Ej 2

![](attachments/Pasted%20image%2020260814172859.png)

a.
```
Query 1 ERROR at Line 1: : Field of view 'mydb.buenos_proveedores' underlying table doesn't have a default value
```

Eso pasa porque la tabla proveedor tiene un campo nombre `NOT NULL` sin un valor default

b. 
Para ello hacemos

```sql
ALTER TABLE proveedor MODIFY nombre VARCHAR(30) NULL;
```

y volvemos a correr el insert y funciona

c y d.
```
Query 1 ERROR at Line 1: : CHECK OPTION failed 'mydb.buenos_proveedores'
```

por el tema de que no pertenece a la vista

## Ej 3

![](attachments/Pasted%20image%2020260823161805.png)

la primera si y la segunda no por el tema de la *propiedad de preservacion de la clave*. en la tabla *distribuidor* la clave es *id_distribuidor* mientras que en la tabla departamento la clave es *(id_departamento, id_distribuidor)*

![](attachments/Pasted%20image%2020260823162149.png)![](attachments/Pasted%20image%2020260823162156.png)

```
Query 1 ERROR at Line 1: : Duplicate entry '1050' for key 'distribuidor.PRIMARY'
```

Falla porque viola una restriccion de integridad de unicidad


## Ej 4

![](attachments/Pasted%20image%2020260823163052.png)

a.
```sql
CREATE VIEW empleado_dist_20 AS
SELECT id_empleado, nombre, apellido, sueldo, fecha_nacimiento
FROM empleado e
WHERE e.id_distribuidor = 20;
```

es de seleccion proyeccion y es actualizable

b.
```sql
CREATE VIEW empleado_dist_20_80 AS
SELECT id_empleado, nombre, apellido, sueldo, fecha_nacimiento
FROM empleado_dist_20
WHERE fecha_nacimiento BETWEEN '1980-01-01' AND '1989-12-31';
```

c.
Los controles del check option serian si pertenecen a la vista o no.
el de cascade en b tambien chequearia que el id_distribuidor sea 20.
igual como no tiene id_distribiudor, si este tiene no permite null o no tiene default, tambien fallaria

d.
```sql
CREATE VIEW peliculas_entregadas AS
SELECT r.id_pelicula, SUM(r.cantidad) AS cantidad_entregada
FROM renglon_entrega r
GROUP BY r.id_pelicula;
```

Tiene funciones de agregacion bro, no es actualizable

e.
```sql
CREATE VIEW distribuidoras_nacionales AS
SELECT d.id_distribuidor, d.nombre, d.direccion, d.telefono, d.tipo,
       n.nro_inscripcion, n.encargado
FROM distribuidor d JOIN nacional n
ON d.id_distribuidor = n.id_distribuidor;
```

actualizable ambas tablas porque ambas preservan la clave
## Ej 5

![](attachments/Pasted%20image%2020260823164058.png)

>[!quote]
>**La idea de "clave preservada" es sobre el resultado del JOIN, no sobre las tablas por separado.** Una clave se preserva si, en el conjunto de filas que devuelve el join, ese valor de clave sigue apareciendo **una sola vez por fila** — es decir, si el join no lo duplica.
>
> — Claude


Los casos que pide son los de [Reglas de relaciones (RIR)](../notas/2_Vistas.md#Reglas%20de%20relaciones%20(RIR))

En la primera:
- Caso 2: FK ≡ Z (secundarios), con FK ∩ K = ∅ — relación N:1 o 1:1
- preserva la clave de la ciudad

Em la segunda:
- Caso 3: FK ⊂ K — relación N:N o entidad débil
- se preserva la clave de renglon_entrega