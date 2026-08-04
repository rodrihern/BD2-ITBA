
# Bases de datos relacionales

Capitulo 1 de un libro: https://campus.itba.edu.ar/ultra/courses/_35943_1/outline/file/_1180213_1?courseId=_35943_1

*DBMS*: database management system (postgres, mySQL, mongoDB)

Las relacionales son las de bd 1, con tablas que se relacionan entre si

Se usan cuando uno quiere persistir informacion, se pueden usar en la nube, on premise, ...

Se usan desde sistemas chicos y grandes

## Vision de los datos

Provee **abstraccion** a niveles:
- Fisico: lo guarda en bits, vos no pensas en bits
- logico: las relaciones
- vistas: algunos usuarios solo pueden acceder (ver) un subconjunto de los datos

## Modelo

El modelo de ER es de los 70' que esta buenisimo y es muy simple, pero a la vez es muy rigido. Ahora hay otras maneras tambien de hacerlo

## Usuarios

Lo de siempre, roles, distintos permisos, administrador

## Postgresql

Es open source y es el que mejor implementa el estandar de sql

osea es una bomba

![](notas/attachments/Pasted%20image%2020260804104825.png)


## Modelos de datos

representamos en tuplas

tiene poder expresivo

es abstracta

sirve para esconder tambien datos que hoy no son relevantes

![](notas/attachments/Pasted%20image%2020260804105946.png)

Queremos hacer modelos **sencillos** y lo suficientemente **genericos**

## Modelo ER

tiene 3 elementos:
- Entidad
- relacion
- atributos

![](notas/attachments/Pasted%20image%2020260804110119.png)

### Entidades

Modelan objetos o cosas

Tenemos:
- fuertes -> existen por si solas
- debiles -> necesitan de otra para existir (ej: una provincia necesita que exista un pais)

Tienen estas propiedades inherentes:
1. Tiene sentido que exista
2. Cada ejemplar debe poder distinguirse de los demas
3. Todos los ejemplares deben tener las mismas propiedades (atributos)

### Atributos

Son los datos relativos a una entidad

- Presencia: si es requerido o no
- Cardinalidad: cuantas instancias de un atributo se pueden tener (univaludado, multivaluado)
- Rol: 
	- identificador principal
	- identificador alternativo
	- Descriptor
- Composicion: si el atributo tiene atributos
- Origen: nativo o derivado

### Relacion

Caracteristicas:
- Nombre: para identificarla
- orden: numero de entidades que participan
- cardinalidad 1:N N:M

pueden ser n-arias (en general se trata de que sean binarias o no mayor a 3)


![](attachments/Pasted%20image%2020260804111740.png)

