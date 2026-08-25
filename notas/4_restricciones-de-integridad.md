
# Restricciones de integridad

Mecanismos para mantener a la base de datos en un estado consistente

## Clasificacion 

### Segun su naturaleza

- **inherente**: por definicion del modelo de datos (una tabla no puede tener 2 filas exactamente iguales)
- **implicitas**: vienen del modelo de datos y se especifican en la creacion (un NOT NULL por ejemplo, o que la fk exista en la otra tabla)
- **Explicitas**: restricciones adicionales (como un CHECK porcentaje <= 100)

### Segun los estados involucrados

- **de estado**: este estado no puede ser (*no* puede ser < 0) 
- **de transicion de estados**: de este estado no puede pasar a este otro (de CANCELLED no puede pasar a COMPLETED, el sueldo no puede disminuir)

## Referenciales

cuando tengo una FK en una tabla (*referenciante*) hacia una tabla con su primary key (*referenciada*) tengo que tener en cuenta que los cambios en la tabla referenciada "impactan" en la referenciante (no va a cambiar ninguna fila de la referenciante pero logicamente algo cambia pues por algo hay una fk bro).


### Acciones referenciales

- NO ACTION: no lo permite, se chequea al final
- RESTRICT: no lo permite, se chequea primero
- CASCADE: se propaga y borra todos los referenciantes
- SET NULL: setea en unll todos los referenciantes
- SET DEFAULT: poloca el valor por defecto en todos lso referencianes

todo esto siempre y cuando no viole las restricciones del modelo (por ejemplo lo del set null tiene que permitir null)

### Tipos de matching

Afectan cuando las FK es compuesta. 

si ningun campo es null y hay coincidencia con una tupla de la tabla referenciada es todos los tipos a la vez.

Sino va a ser de cada tipo cuando

- MATCH SIMPLE: al menos una es NULL
- MATCH PARTIAL: al menos una en NULL pero que las que estan presentes coinciden con al menos una fila existente
- MATCH FULL: todas en NULL

ejemplos:

![](attachments/Pasted%20image%2020260825164830.png)

## Restricciones del administrador

Hay 2 tipos:
- **declarativa**
- **procedural**

la **procedural** es la de los triggers, procedimientos, funciones y la **declarativa** es la de los constraint y los check

>[!note]
>ojo en los check que con los null dan UNKNOWN y eso lo toma como true

si yo ya tengo data, la declarativa mira los datos ya almacenados. La procedural no, solo se va a ejecutar con datos nuevos

a la hora de aplicar una restriccion hay que intentar agregarla con este orden de prioridades:
1. de DBMS
2. declarativa
3. procedural

### Declarativa

Se pueden crear dominios para usarlos como tipo de dato. Vendrian a ser como una especie de `typedef` de sql

```sql
CREATE DOMAIN NomDominio  
AS TipoDato [ DEFAULT ValorDefecto ]  
[ [CONSTRAINT NomRestriccion] CHECK (condición);
```



Se ejecutan las restricciones segun esta jerarquia
1. **dominio/atributo**: tipo de dato o checks
2. **tupla**: mira varios atributos, constraints tipo fechaInicio < fechaFin
3. **tabla**: miran varias tuplas, UNIQUE, PRIMARY KEY
4. **db**: mira varias tablas. Assertions en postgres



### Procedural

sintaxis de triggers para postgres

```sql
CREATE [ CONSTRAINT ] TRIGGER nombre_del_trigger  
{ BEFORE | AFTER | INSTEAD OF }  
{ INSERT [ OR ] UPDATE [ OF nombre_columna [, ... ] ]  
[ OR ] DELETE [ OR ] TRUNCATE }  
ON nombre_tabla_o_vista  
[ FOR [ EACH ] { ROW | STATEMENT } ] 
[ WHEN ( condición ) ]
EXECUTE PROCEDURE nombre_función;
```

Se usan mucho para las de transicion de estado, por ejemplo para aplicar la regla de "que el sueldo de un empleado no se reduzca" o tambien para actualizacion automatica de datos derivados

si un insert dispara un trigger, y este falla, se hace un rollback de todo y el insert nunca se hace

![](attachments/Pasted%20image%2020260825190055.png)

