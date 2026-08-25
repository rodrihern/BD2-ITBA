---
materia: bd2
tipo: apuntes
---

# Bases de datos relacionales

> [!quote] Bibliografía y referencias
> **Lectura obligatoria:** Silberschatz, A.; Korth, H. F.; Sudarshan, S. — *Fundamentos de bases de datos*. **Capítulo 1: Introducción**.
> Archivo local: [[Silberschatz BD_Cap1.pdf]]

> [!IMPORTANT] Definición
> Un **SGBD** (Sistema Gestor de Bases de Datos, en inglés *DBMS: Database Management System*) es una colección de datos interrelacionados junto con un conjunto de programas para acceder a dichos datos. Ejemplos: PostgreSQL, MySQL, MongoDB.

- **Colección de datos** (la *base de datos* propiamente dicha): contiene información relevante para la organización.
- **Objetivo principal:** proporcionar una forma **práctica y eficiente** de almacenar y recuperar información.
- Se diseñan para gestionar **grandes volúmenes** de información, lo que implica poder:
	- Definir estructuras para almacenar información.
	- Contar con mecanismos para manipular dicha información.

Las bases de datos **relacionales** son las de BD1: tablas que se relacionan entre sí. Se usan cuando se quiere **persistir** información, tanto en la nube como *on premise*, y aparecen desde sistemas chicos hasta grandes.

---

## Aplicaciones de los SGBD

Los SGBD están presentes en gran parte de los sistemas de información:

- Bancos
- Líneas aéreas
- Universidades
- Tarjetas de crédito
- Telecomunicaciones
- Sistemas de ventas o de compras
- Cadenas de producción o gestión de almacenes

---

## Aplicaciones de BD vs. sistemas de archivos

En un sistema de procesamiento de archivos típico, los registros permanentes se almacenan en varios archivos y se escriben distintos programas de aplicación para extraer y añadir registros. Esto ocasiona varios inconvenientes que los SGBD vienen a resolver:

- Redundancia e inconsistencia de datos.
- Dificultad en el acceso a los datos.
- Aislamiento de datos.
- Problemas de integridad.
- Problemas de atomicidad.
- Anomalías en el acceso concurrente.
- Problemas de seguridad.

---

## Visión de los datos

Uno de los propósitos principales de un SGBD es proporcionar una **visión abstracta** de los datos, escondiendo los detalles de cómo se almacenan y mantienen. Esta **abstracción** se organiza en niveles:

| Nivel | Descripción |
|-------|-------------|
| **Físico** | Nivel más bajo: describe *cómo* se almacenan realmente los datos (estructuras de bajo nivel, bits). El usuario no piensa en bits. |
| **Lógico** | Describe *qué* datos se almacenan y qué relaciones existen entre ellos. El usuario no se preocupa por cómo están almacenados. |
| **Vistas** | Nivel más alto: algunos usuarios sólo acceden a un **subconjunto** del total de los datos. |

---

## Modelos de datos

Por debajo de la estructura de la base de datos está el **modelo de datos**: una colección de herramientas conceptuales para describir los datos, las relaciones entre ellos, su semántica y sus restricciones.

Un modelo de datos:

- Representamos la información en **tuplas**.
- Tiene **poder expresivo** para representar cómo se relacionan los datos.
- Es **abstracto**, para ser mínimamente perturbable ante los cambios del mundo real.
- Sirve para **esconder** datos que hoy no son relevantes.

> [!TIP] Objetivo del modelado
> Queremos modelos **sencillos** y a la vez lo suficientemente **genéricos**.

![[Pasted image 20260804105946.png]]

Modelos de datos más usados:

- **Entidad-Relación (E/R):** representación gráfica conveniente para ver datos, relaciones y restricciones.
- **Relacional:** ampliamente usado para almacenar datos en las bases de datos.
- Otros: orientado a objetos, relacional orientado a objetos y modelos semiestructurados.

---

## Lenguajes de bases de datos

- **LDD (Lenguaje de Definición de Datos):** especifica el **esquema** de la base de datos mediante un conjunto de definiciones.
- **LMD (Lenguaje de Manipulación de Datos):** permite a los usuarios acceder o manipular los datos.
	- **Declarativos / no procedimentales:** el usuario especifica sólo *qué* datos necesita (ampliamente usados hoy).
	- **Procedimentales:** el usuario especifica además *cómo* obtenerlos.
- Acceso desde programas de aplicación mediante APIs como **ODBC** y **JDBC**.

---

## Usuarios y administradores

Los usuarios se catalogan en clases, cada una con su tipo de interfaz:

- **Usuarios normales.**
- **Programadores de aplicaciones.**
- **Administradores de bases de datos (DBA).** Funciones:
	- Definición de esquemas y organización física.
	- Definición de estructuras y métodos de acceso.
	- Autorizaciones para acceder a los datos (roles, permisos).
	- Mantenimiento de la base de datos.

---

## Módulos de un SGBD

Un sistema de bases de datos se compone de varios subsistemas:

- **Gestor de transacciones:** asegura que la base permanezca en un estado **consistente** a pesar de fallos, y que las transacciones concurrentes se ejecuten **sin conflictos**.
- **Procesador de consultas:** compila y ejecuta las instrucciones LDD y LMD.
- **Gestor de almacenamiento:** interfaz entre los datos de bajo nivel almacenados y las consultas/programas de aplicación.

---

## Arquitecturas de aplicaciones

Las aplicaciones de bases de datos suelen dividirse en una parte **front end** (máquina cliente) y una parte **back end** (servidor):

- **Dos capas:** el frontal se comunica directamente con la base de datos del servidor.
- **Tres capas:** el servidor se divide en un **servidor de aplicaciones** y un **servidor de bases de datos**.

---

## PostgreSQL

- SGBD **open-source**, con origen en el *‘Proyecto Ingres’* de la Universidad de Berkeley.
- Primera versión liberada en **1997**.
- *Cross-platform*, escrito en **C**.
- Es el que **mejor implementa el estándar SQL**.
- Usado por organizaciones como Yahoo, MySpace y Skype.

![[Pasted image 20260804104825.png]]

---

## Modelo Entidad-Relación

### Representación de la información

La **estructura** y el **contexto** le dan significado a los datos, posibilitando su entendimiento. El lenguaje natural es nuestra forma primaria de comunicación, pero **no es el mejor medio**: a veces conviene una representación especializada (fórmulas matemáticas, mapas, partituras).

Cuando trabajamos con conjuntos de descripciones, usamos una notación simplificada que sólo contiene los valores. Esa notación se denomina **tupla** o **registro**:

```text
<LU; Nombre y Apellido; Fecha nacimiento; carrera>   (elementos / interpretación)
<123456; Juan Torres; 23/08/1995; Ing en Sistemas>   (tupla / registro)
```

### El modelo conceptual

El **Modelo de Entidades y Relaciones (MER / E-R)** fue propuesto por **Chen** (1976 y 1977). Es de los años 70', muy simple pero también bastante rígido; hoy existen otras maneras de modelar.

> [!NOTE] Chen
> "El Modelo E/R puede ser usado como una base para una vista unificada de los datos", adoptando "el enfoque más natural del mundo real que consiste en entidades e interrelaciones". Autores posteriores lo extendieron (**MER Extendido — MERExt**).

El MER distingue **tres elementos** estáticos:

| Elemento | Descripción |
|----------|-------------|
| **Entidad** | Objeto real o abstracto sobre el cual se desea almacenar información. |
| **Relación** (interrelación) | Asociación o vinculación entre entidades. |
| **Atributos** | Características de entidades y relaciones que proveen detalles descriptivos. |

![[Pasted image 20260804110119.png]]

---

### Entidades

Modelan objetos o cosas. Están formadas por **ejemplares** o **instancias**, cada una representando un objeto del mundo real distinguible de los demás.

Dos categorías:

- **Fuertes (o regulares):** sus ejemplares existen por sí mismos (ej. `ALUMNO`).
- **Débiles:** la identificación y existencia de un ejemplar **dependen** de la de otra entidad. Ej.: una `PROVINCIA` depende del `PAÍS`; un `RENGLON_REMITO` depende del `REMITE`.

> [!IMPORTANT] Las tres propiedades inherentes de las entidades
> 1. **Tiene sentido que exista** (existencia propia), evaluada en el contexto del sistema.
> 2. Cada ejemplar debe poder **distinguirse** de los demás.
> 3. Todos los ejemplares deben tener las **mismas propiedades** (atributos).

---

### Atributos

Son los datos relativos a una entidad o relación. Cada atributo tiene un **dominio de definición** (entero, cadena, fecha, etc.) y toma un valor dentro de ese dominio. Se colocan junto a la entidad que describen.

Sus características:

| Característica | Valores posibles |
|---------------|------------------|
| **Presencia** | Obligatorio (siempre hay valor) / Opcional (puede estar ausente). |
| **Cardinalidad** | Univaluado (un valor, ej. Edad) / Multivaluado (varios valores, ej. Teléfonos). |
| **Rol** | Identificador principal (IP) / Identificador alternativo / Descriptor. |
| **Composición** | Simple / Compuesto (ej. Dirección → calle, número, piso, dpto.). |
| **Origen** | Nativo / Derivado (se calcula a partir de otros, ej. edad desde la fecha de nacimiento). |

> [!NOTE] Sobre el identificador principal
> Cada conjunto de entidades debe tener **al menos un identificador principal**. Puede ser un atributo **compuesto**, pero **no** debería ser multivaluado ni opcional.

![](attachments/Pasted%20image%2020260804173347.png)

---

### Relaciones (interrelaciones)

Una **relación** es una asociación o correspondencia entre conjuntos de entidades, materializada en asociaciones entre instancias.

> [!EXAMPLE] Ejemplo de relación
> `CURSA` vincula las entidades `ALUMNO` y `MATERIA`. Un ejemplar concreto es la vinculación entre *"123, Carlos Sánchez"* y *"EdD, Estructuras de Datos"*, ya que satisface la frase *"Carlos Sánchez cursa la materia Estructuras de Datos"*.

**Características de una relación:**

- **Nombre:** único, para identificarla.
- **Grado u orden:** número de entidades que participan.
	- $n=1$ → **unaria**; $n=2$ → **binaria**; $n=3$ → **ternaria**; …
- **Cardinalidad** (tipo de correspondencia): $1:1$, $1:N$, $N:N$.
- Puede tener **atributos propios**.

En general se busca que las relaciones sean **binarias**, y no de orden mayor a 3.

![[Pasted image 20260804111740.png]]

---

### Cardinalidades

La lectura de cardinalidades se denomina **Look-Across (LA)** o **Chen-Style**: se lee sobre la línea de la *entidad destino*. Responde: *¿cuántos ejemplares de $E_1$ pueden relacionarse con cada ejemplar de $E_2$, como máximo y como mínimo?*

- **Cardinalidad máxima:** máximo número de ejemplares con los que se puede relacionar.
	- como máximo **1**, o como máximo **N** (muchos, cantidad variable), o como máximo **a** (cantidad fija).
- **Cardinalidad mínima:** mínimo número de ejemplares con los que debe relacionarse.
	- **0** → **opcionalidad**.
	- **1** → **obligatoriedad** (relación mandatoria).

> [!EXAMPLE]+ Relaciones según su tipo
> - **Unaria (reflexiva/recursiva):** cada pieza *forma-parte-de* otra u otras piezas.
> - **Binaria 1:N:** cada carrera pertenece a un único departamento; cada departamento posee muchas carreras.
> - **Binaria N:N:** cada alumno practica varios deportes; cada deporte es practicado por varios alumnos.
> - **Opcional vs. obligatoria:** con cardinalidad mínima $(1,N)$ un alumno *debe* practicar al menos un deporte; con $(0,N)$ *podría* no practicar ninguno.

---

### Relación entidad débil — entidad fuerte

- Una **entidad débil** sólo puede identificarse unívocamente en el contexto de una **entidad fuerte** (o propietaria).
- Están vinculadas por una relación binaria **$(1,1):(*,N)$**. Del lado fuerte la cardinalidad es siempre **1**.
- La entidad débil tiene **dependencia de existencia y de identificación** respecto de la fuerte.

### Jerarquías (relaciones ES-UN / ISA)

Relacionan un **supertipo** con uno o más **subtipos**:

- **Jerarquía exclusiva:** una instancia del supertipo pertenece a un único subtipo.
- **Jerarquía compartida:** una instancia puede pertenecer a varios subtipos.
- Puede tener participación **total** o **parcial**.

---

### Construcción del MER (heurísticas de Chen)

> [!NOTE] No hay reglas fijas
> No existen reglas que indiquen cómo construir un modelo de datos; sólo principios generales aplicados junto al criterio del diseñador experimentado.

Heurísticas propuestas por Chen para interpretar frases en lenguaje natural:

- En general un **sustantivo** es una **entidad** (aunque también podría ser un atributo). Ej.: *"los **ALUMNOS** cursan **MATERIAS**"*.
- Un **verbo** o frase verbal puede indicar una **relación**. Ej.: *"los alumnos **CURSAN** materias"*.
- Luego se determina qué información registrar → los **atributos** de entidades y relaciones.

---

## Derivación a esquema lógico

El diseño lógico transforma el **Modelo E/R** en un **Esquema Lógico según el Modelo Relacional**, y de ahí al **esquema post-relacional** (tablas en SQL), aplicando **reglas de transformación**.

```mermaid
graph LR
    A[Modelo E/R] -->|Reglas de<br>transformación| B[Esquema Lógico<br>Modelo Relacional]
    B --> C[Esquema Post-relacional<br>tablas en SQL]
```

### Esquema de base de datos

- Una base de datos relacional consiste en un conjunto de **tablas** (relaciones), cada una con un conjunto de **tuplas**.
- La definición se realiza con sentencias **DDL**; el SGBD guarda su información en **tablas de metadatos**.
- El nombre de cada tabla debe ser **único** dentro del esquema.
- Una tabla tiene **columnas** (atributos, con su dominio) y **filas** (tuplas/registros).
	- **Comprensión** = cabecera/esquema de la tabla.
	- **Extensión** = valor o estado actual (las filas).
- En SQL **no existe un orden** para las filas: aparecen en orden aleatorio salvo que se especifique.

> [!IMPORTANT] Claves
> - **Clave (o clave alternativa):** subconjunto de columnas que **identifica** a cada tupla.
> - **Clave extranjera (FK, *foreign key*):** conjunto de columnas de una tabla $T$ que referencia la clave de otra tabla $T'$, imponiendo una **Restricción de Integridad Referencial**.

![[Pasted image 20260804112711.png]]

---

### Reglas de transformación de entidades

Del DERExt al Esquema Relacional, para cada **entidad**:

- Se crea una **tabla** por cada entidad, con el mismo nombre.
- El **identificador** de la entidad se transforma en la **clave** de la tabla.
- Todo atributo **simplemente valuado** se transforma en una columna.
- Los atributos **compuestos** se despliegan en sus componentes (como si fueran univaluados).
- Los atributos **obligatorios** llevan `NOT NULL`; los **opcionales**, igual pero sin esa leyenda.
- Los atributos **multivaluados** se proyectan en **otra tabla**, junto con la clave de la entidad o relación.

> [!EXAMPLE]+ Derivación de una entidad `ALUMNO`
> A partir de la entidad `ALUMNO` (con identificador `LU`, dirección compuesta y atributos obligatorios):
> ```sql
> CREATE TABLE ALUMNO (
>     LU        integer      NOT NULL,
>     Documento integer      NOT NULL,
>     Apellido  varchar(30)  NOT NULL,
>     Nombre    varchar(30)  NOT NULL,
>     Tutor     varchar(50),                 -- opcional
>     Calle     varchar(40)  NOT NULL,       -- componente de Dirección
>     Nro       integer      NOT NULL,       -- componente de Dirección
>     Ciudad    varchar(60)  NOT NULL,       -- componente de Dirección
>     CONSTRAINT PK_ALUMNO PRIMARY KEY (LU)
> );
> ```
> La clave primaria también puede definirse aparte:
> ```sql
> ALTER TABLE ALUMNO ADD CONSTRAINT PK_ALUMNO PRIMARY KEY (LU);
> ```

---

### Derivación de relaciones binarias 1:N (y unarias 1:N)

Los atributos identificadores de la entidad del **‘lado 1’** (la clave) se agregan como columnas en la tabla de la entidad del **‘lado N’** → constituyen una **clave extranjera**.

> [!NOTE] Relaciones unarias 1:N
> En una relación unaria (recursiva), la FK **debe renombrarse** para distinguirla de la clave original.

### Derivación de relaciones N:N (unarias y binarias)

- Se crea una **nueva tabla** cuya clave es la **yuxtaposición** de los identificadores de las entidades participantes.
- El **nombre** de la tabla es el indicado en el rombo (puede renombrarse).
- Cada clave, por separado, es una **FK** referida a la tabla de la que proviene.

> [!EXAMPLE]+ Derivación de `ALUMNO` N:N `CARRERA`
> ```sql
> CREATE TABLE ALUMNO (
>     LU        integer      NOT NULL,
>     Apellido  varchar(30)  NOT NULL,
>     Nombre    varchar(30)  NOT NULL,
>     FechaNac  date,
>     CONSTRAINT PK_ALUMNO PRIMARY KEY (LU)
> );
>
> CREATE TABLE CARRERA (
>     IdCarrera     char(5)       NOT NULL,
>     NombreCarrera varchar(100)  NOT NULL,
>     PlanEstudio   char(6)       NOT NULL,
>     CONSTRAINT PK_CARRERA PRIMARY KEY (IdCarrera)
> );
>
> -- Tabla producto de la relación N:N
> CREATE TABLE ALUMNOSXCARRERA (
>     LU        integer  NOT NULL,
>     IdCarrera char(5)  NOT NULL,
>     CONSTRAINT PK_ALUMNOSXCARRERA PRIMARY KEY (LU, IdCarrera)
> );
>
> ALTER TABLE ALUMNOSXCARRERA ADD CONSTRAINT FK_AXC_CARRERA
>     FOREIGN KEY (IdCarrera) REFERENCES CARRERA(IdCarrera);
>
> ALTER TABLE ALUMNOSXCARRERA ADD CONSTRAINT FK_AXC_ALUMNO
>     FOREIGN KEY (LU) REFERENCES ALUMNO(LU);
> ```

---

### Derivación de atributos en relaciones

- Los atributos de una relación pueden ser del mismo tipo que los de una entidad.
- Si la relación es **designativa ($1:N$):** se incluyen en la tabla del **lado N**.
- Si la relación es **asociativa ($N:N$ o ternaria):** se derivan en la **tabla producto** de la relación.

### Derivación de jerarquías

- Se crea una tabla por la **entidad supertipo** (con los atributos comunes, incluido su identificador) y una tabla por cada **subtipo** (con sus atributos propios).
- La **clave** de cada tabla subtipo es la **clave del supertipo**.
- En jerarquías **exclusivas** se agrega el atributo **discriminante** (tipo) a la tabla del supertipo.

### Derivación de entidades débiles

- Tienen dependencia de existencia e identificación.
- Su clave se forma con el **identificador propio** (clave parcial) **más** la clave de la entidad **fuerte**.

---

### Sentencia CREATE TABLE

Sintaxis general en PostgreSQL:

```sql
CREATE TABLE [ IF NOT EXISTS ] nombre_tabla (
    { nombre_columna tipo_dato [ column_constraint [ ... ] ]
      | table_constraint }
    [, ... ]
);
```

**`column_constraint`** (restricción de columna):

```sql
[ CONSTRAINT constraint_name ]
{ NOT NULL
  | NULL
  | DEFAULT default_expr
  | UNIQUE index_parameters
  | PRIMARY KEY index_parameters
  | REFERENCES reftable [ ( refcolumn ) ] }
```

**`table_constraint`** (restricción de tabla):

```sql
[ CONSTRAINT constraint_name ]
{ CHECK ( expression )
  | UNIQUE ( column_name [, ... ] )
  | PRIMARY KEY ( column_name [, ... ] )
  | FOREIGN KEY ( column_name [, ... ] )
    REFERENCES reftable [ ( refcolumn [, ... ] ) ] }
```

> [!TIP] Tipos de datos
> Cada columna tiene un **tipo de dato** que limita el conjunto de valores posibles. La referencia completa de tipos de PostgreSQL está en su [documentación oficial](https://www.postgresql.org/docs/current/datatype.html).
