
# Seguridad y Transacciones

Queremos proteger frente a accesos malintencionados.

## Tipos de amenazas

- Perdida de integridad
- Perdida de disponibilidad
- Perdida de confidencialidad

## Niveles de seguridad

- Sistema de bases de datos
- Sistema operativo
- Red
- Fisico
- Humano


## Cifrado de datos

Los datos sensibles deberian estar hasheados o cifrados. En general hasheados, como contraseñas o tarjetas de creditos.

## Usuarios y permisos

**authentication**: se quien sos

```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password'
```

**authorization**: se que permisos tenes

```sql
GRANT <permiso> ON <nombre_db>.<nombre_tabla> TO 'username'@'host' [WITH GRANT OPTION];
```

**WITH GRANT OPTION**: permite que un usuario maneje privilegios de otros usuarios (solo los permisos que se le otrogaron a el)


eso es para dar, para sacar un permiso en lugar de GRANT ponemos `REVOKE` y `FROM` en lugar de TO.

y para que se aplique tengo que tirar un

```sql
FLUSH PRIVILEGES;
```

### Tipos de permisos

- ALL
- CREATE
- DROP
- DELETE
- INSERT
- SELECT
- UPDATE

### Ejemplo

![](attachments/Pasted%20image%2020260908152523.png)
## Roles

Podemos otorgar privilegios a un rol, y despues el rol a un usuario

```sql
CREATE ROLE 'developer'

GRANT SELECT ON tablita TO 'developer'

GRANT 'developer' TO 'dev1'@'localhost'
```

## Resolucion de ejercicios

Quiere que las soluciones lo hagamos con grafos dirigidos tipo asi:

![](attachments/Pasted%20image%2020260908152653.png)

Sirve para ver que usuario le otorga privilegios a que usuario

![Ej-permisos](drawings/Ej-permisos.md)

## Transacciones

Mecanismo para definir unidades logicas del procesamiento de una base de datos. 

Una transacción es una unidad de trabajo que agrupa una o más sentencias SQL y se ejecuta como un todo indivisible: o se aplican todos sus efectos, o ninguno.



Si alguna de todas las sentencias falla por cualquier motivo, se hace rollback de toda la transaccion.

### ACID

- **Atomicidad**: cada transaccion es todo o nada
- **Consistencia**: lleva a la base de datos a un estado consistente
- **Isolation**: Ejecucion concurrente = secuencial
- **Durabilidad**: queda persistido una vez terminada la transaccion

### Diagrama de transicion de estados

![](attachments/Pasted%20image%2020260908161926.png)

### Ventajas

- Productividad y utilizacino de recursos mejorados
- Tiempo de espera reducido

La idea es optimizar uso de procesador mientras estamos bloqueados en I/O.
### Problemas

- **Race conditions**
- **Dirty read**
- **non-repeatable read**: un dato se modifica
- **phantom read**: un dato se inserta/borra

### Mecanismos de control de concurrencia

- **Shared lock**: muchas pueden leer
- **Exclusive Lock**: mutex, una sola puede leer/escribir
- **Control de versiones**: pone una version a cada uno
- **timestamp ordering**: la que tiene timestamp mas chico es el que llego primero y es el que ganó

### Niveles de aislamiento

1. **Read uncommitted**: permite dirty read
2. **Read commited**: minimiza dirty reads pero no evita los non-repeatable reads
3. **Repeatable read**: previene las non-repeatable pero no las phantom
4. **Serializable**: evita todos pero es el que mas performance penaliza

Entre mas "problemas permito" mas rendimiento tengo. O visto de otra manera, si quero ahorrarme todos los problemas, son todos mutex, pierdo la performance que gano con la concurrencia.

Antes de cada transaccion uno fija el nivel.

## Indices

- **Indices ordenados**: B-Tree
- **Indices asociativos**: hash table


| B-tree                              | Hash                            |
| ----------------------------------- | ------------------------------- |
| comparaciones, order by, query LIKE | por = o $\ne$, claves completas |

```sql
CREATE INDEX myindex ON users (DNI) USING HASH
```



## Recovery

![BD2_CLASE 11b - Recovery.pptx](../teoria/BD2_CLASE%2011b%20-%20Recovery.pptx.pdf)

