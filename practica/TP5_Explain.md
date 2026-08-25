# TP4 EXPLAIN ANALYZE

## Ej 1

![](attachments/Pasted%20image%2020260823184714.png)

Creamos la tabla e insertamos los valores

```sql
CREATE TABLE materia (
	codigo INTEGER PRIMARY KEY,
	nombre VARCHAR(40) NOT NULL
);

INSERT INTO materia (codigo, nombre) VALUES
(10, 'Introduccion a la Computacion'),
(20, 'Programacion I'),
(30, 'Estructura de Datos y Algoritmos'),
(40, 'Base de Datos I'),
(50, 'Programación IV'),
(60, 'Base de Datos II');
```

ahora corremos 

```sql
SET @@explain_format=TREE;
```

para que lo muestre como un arbol xd

ahora corremos 

```sql
EXPLAIN SELECT * FROM materia;
```

y nos sale 

![](attachments/Pasted%20image%2020260823185525.png)

y corriendo

```sql
EXPLAIN ANALYZE SELECT * FROM materia;
```

nos sale 

![](attachments/Pasted%20image%2020260823185606.png)

La diferencia es que `EXPLAIN` solamente explica que hace y da el costo en una unidad arbitraria, mientras que `EXPLAIN ANALYZE` ademas corre la query y te tira el actual time. Como hace la query de verdad hay que tener cuidado con los posibles side effects en la tabla

## Ej 2
![](attachments/Pasted%20image%2020260823191148.png)
![](attachments/Pasted%20image%2020260823191119.png)![](attachments/Pasted%20image%2020260823191126.png)

Si ponemos codigo como `PRIMARY KEY` va a crear un indice asi que va a estar re piola

```
Rows fetched before execution  (cost=0..0 rows=1)
```

si hacemos que la primary key sean ambos tira 

```
Covering index lookup on materia using PRIMARY (codigo = 10)  (cost=0.35 rows=1)
```

osea tiene un indice pero no tan bueno porque no los puede tener indexado por codigo ahora (podrian haber mas de una fila con el mismo codigo). Asi que tiene un indice btree que es tipo, primero por codigo despues por materia.

declarandolos como `UNIQUE` pasa lo mismo que con las primary keys

con los indices pasa esto:

```sql
ALTER TABLE materia DROP PRIMARY KEY;
CREATE UNIQUE INDEX uq_codigo ON materia(codigo);
```

nos da lo mismo que con la `PRIMARY KEY` y con este indice

```sql
ALTER TABLE materia DROP INDEX uq_codigo;
CREATE UNIQUE INDEX uq_codigo_nombre ON materia(codigo, nombre);
```

nos tira

```
"-> Covering index lookup on materia using uq_codigo_nombre (codigo = 10)  (cost=0.35 rows=1)
"
```

que es basicamente lo mismo que lo que nos daba antes

![](attachments/Pasted%20image%2020260823193112.png)
![](attachments/Pasted%20image%2020260823194037.png)![](attachments/Pasted%20image%2020260823193135.png)
![](attachments/Pasted%20image%2020260823193152.png)![](attachments/Pasted%20image%2020260823193124.png)

TLT
key takeaways:
- no es lo mismo una PK que una PK compuesta para los indices
- UNIQUE se comporta igual que una PK
- si voy a hacer una query tipo `WHERE codigo = 60 and nombre = '...'`  ahi si tener un indice (codigo, nombre) es una locura (cost = 0)
- para el sort ayuda el indice

sin indice
```
"-> Sort: materia.codigo (cost=0.85 rows=6) (actual time=0.286..0.287 rows=6 loops=1)

-> Table scan on materia (cost=0.85 rows=6) (actual time=0.0786..0.0879 rows=6 loops=1)"
```

con indice

```
"-> Index scan on materia using PRIMARY (cost=0.85 rows=6) (actual time=0.111..0.122 rows=6 loops=1)"
```

- El `JOIN` anda mejor si estoy haciendo un `JOIN ON` algo que tiene indice
- indice compuesto NO ayuda en un `OR`
## Ej 3

![](attachments/Pasted%20image%2020260823192510.png)

TLT