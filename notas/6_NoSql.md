# Bases de datos no sql

Sirve para perfeccionar las bd con exceso de datos que necesitan un gran indexado de documentos.

Con muchos datos y muchas trnasacciones se vuelven lentas las relacionales.

## Escalabilidad

Vertical -> mas hardware en la misma maquina
Horizontal -> mas maquinas

la escalabilidad horizontal es **elastica** (uso mas o menos compus segun el consumo)

Las no sql responden a la demanda de escalabilidad horizontal

Hay diferentes DBs noSQL para diferentes proyectos

## Ventajas

Rapidas, escalabilidad horizontal, enormes cantidades de datos, se ejecutan en clusters de maquinas baratas

## Desventajas

Les falta madurez, no son del todo ACID compliant, problemas de compatibilidad

## Teorema CAP

Es sobre sistemas distribuidos pero aplica aca

Solo se puede cumplir 2 al mismo tiempo:
1. **C**onsistency
2. **A**vailability 
3. **P**artition tolerance 

![](attachments/Pasted%20image%2020260929154242.png)

que no sea consistente la cantidad de likes en un post por un tiempo no es un problema, en transacciones con plata si.

## Transacciones BASE

![](attachments/Pasted%20image%2020260929154334.png)

Consistencia vs Disponibilidad

## Categorias

![](attachments/Pasted%20image%2020260929154516.png)

### Orientadas a grafo

- info en nodos
- normalizada
- no es necesario definir la cantidad de atributos
- schema free, registros de longitud variable
![](attachments/Pasted%20image%2020260929155835.png)

### Columna

- guarda valores en columnas
- los datos son almacenados como secciones de las columnas de datos
![](attachments/Pasted%20image%2020260929155913.png)

### Clave valor

- conjunto de duplas key-value
- existen contenedores
- validacion de datos en la app cliente

![](attachments/Pasted%20image%2020260929160031.png)

### Orientada a documentos

- Almacena los datos en documentos
- son duplas clave-documento
- Los documentos dentro de una misma coleccion pueden tener datos diferentes

![](attachments/Pasted%20image%2020260929160129.png)

## SQL vs NoSQL

![](attachments/Pasted%20image%2020260929160153.png)