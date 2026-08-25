
# Vistas

Presentan una parte de la bd que es de interes para grupos particulares de usuarios. Tiene que ver con abstraccion y por seguridad.

## creacion y destruccion

```sql
CREATE VIEW <nombre> [(nombre_col1, nombre_col2)] AS
<query>
[WITH [CASCADED | LOCAL] CHECK OPTION];
```

**NO** usar LOCAL, y el default es CASCADED asi que directamente ponemos `WITH CHECK OPTION`

```SQL
DROP VIEW <nombre> [RESTRICT | CASCADE]
```

## Actualizaciones

Cada actualizacion de una tupla en una vista debe poder *propagarse sin ambiguedades* a la db

(obvio que si hay un cambio en las tablas de la db, la vista se actualiza)

Hay casos en los que no se propagan como en una funcion de agregacion (imaginate que tengo el total que es la suma de muchas cosas, si cambio el total, como se que es lo que tiene que cambiar)

Para que se actualice una vista el camino tiene que ser:
	cambio en la vista -> propaga a un cambio en la db -> ese cambio impacta en la vista

### Condiciones para que una vista sea actualizable

Actualizable si:
- conserva todas las columnas de la primary key
- **no** contiene funciones de agregacion o innformacion derivada
- **no** contiene la clausula DISTINCT
- **no** incluye subconsultas en el SELECT

pero ademas no tiene que que violar ninguna restriccion definida sobre la tabla base

## Check Option

Lo que hace es fijarse que que no se viole la condicion de pertenencia, osea chequea que el cambio pertenezca a la tabla

### Ejemplo 1

![](attachments/Pasted%20image%2020260811160034.png)

con el check option el insert con cantidad 300 no pasa, pues no pertenece a la tabla. Sin check option lo que pasa es que se termina insertando en la tabla original y no se ve ningun cambio en la vista.

Con check option tampoco te deja el de abajo, la del set cantidad = 100, pues no pertenece a la vista si tienen cantidad 100.

### Ejemplo 2

Aca vemos por que no usar el `WITH LOCAL CHECK OPTION`

![](attachments/Pasted%20image%2020260811160817.png)

CASCADED mira todas las vistas hacia arriba, LOCAL solo la condicion de la vista actual. Por lo que si le puse local, el set=300 no viola la condicion del <1000 entonces termina pasando que impacta en la db y no lo vemos en la vista. (lo que supuestamente queremos evitar con el check option)

## Vistas actualizables a partir de 2 o mas tablas/vistas

Solo se puede modificar una de las tablas base, de la cual se cumple la propiedad de preservacion de la clave.

El ensamble tiene que venir de un JOIN o un producto cartesiano, **no** puede venir de interseccion/union/diferencia.

## Reglas de relaciones (RIR)

### Caso 1: FK ≡ K — herencia tipo-subtipo

Persona es supertipo, Empleado es subtipo (todo Empleado es una Persona).

```
Persona    (dni, nombre, fecha_nac)
Empleado   (dni, sueldo, area)
```

### Caso 2: FK ≡ Z (secundarios), con FK ∩ K = ∅ — relación N:1 o 1:1

Empleado (N) — _trabaja en_ → Departamento (1).

```
Departamento (id_depto, nombre)
Empleado     (legajo, nombre, id_depto)
```

### Caso 3: FK ⊂ K — relación N:N o entidad débil

Factura (N) — _contiene_ → Producto (N), con atributo propio `cantidad`.

```
Factura       (nro_factura, fecha)
Producto      (cod_producto, descripcion)
DetalleFactura (nro_factura, cod_producto, cantidad)
```

## Actualizaciones via triggers

Se usan triggers INSTEAD OF

queda en manos del usuario del dbms la responsabilidad de implementar las actualizaciones necesarias y de la manera que las considere "adecuadas"

### Ejemplo

Si quiero que una actualizacion pegue en mas de una tabla

![](attachments/Pasted%20image%2020260811172049.png)

## Caso particular mySQL

![](attachments/Pasted%20image%2020260811172636.png)

## Ventajas de tener vistas

- simplifican la percepcion que los usuarios tienen de la db
- Presentan diferentes datos dependiendo el tipo de usuario
- Permiten definir consultas complejas/frecuentes para no tener que especificarlas cada vez que se utilizan
- Facilitan la independencia de los datos
- Permiten aplicar politicas de seguridad

## Vistas materializadas - PostgreSQL

Las vistas normales simplemente son una query que se guarda y se trae los datos al momento de acceder a la vista

Una *vista materializada* guarda el resultado (la vista) en disco

La idea es usarla para cuando se usa mucho y la query es lenta

```sql
CREATE MATERIALIZED VIEW <view_name>
AS <query>
WITH [NO] DATA
```

- WITH DATA: (default) carga la tabla con los datos
- WITH NO DATA: crea la estructura de la tabla (columnas, tipos) pero no ejecuta la query para popularla con datos



