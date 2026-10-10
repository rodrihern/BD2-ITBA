---
materia: bd2
tipo: apuntes
---

# Cassandra

Base de datos NoSQL **tabular** (orientada a columnas, ver [[6_NoSql#Columna]]), distribuida y masivamente escalable. Combina propiedades de una base **clave-valor** y una **orientada a columnas**.

- Origen: Facebook, para la búsqueda en el inbox. Open source en 2008, proyecto top-level de Apache en 2010.
- Inspirada en los papers de **Amazon Dynamo** (2007) y **Google BigTable** (2006). Hoy la mantiene **DataStax**.
- Escrita en Java, licencia Apache 2.0, multiplataforma.

---

## Bases de datos tabulares

| RDBMS         | Tabular                           |
| ------------- | --------------------------------- |
| Instancia     | Cluster                           |
| Base de datos | KeySpace                          |
| Tabla         | Familia de columnas               |
| Fila          | Fila                              |
| Columna       | Columna (distinta para cada fila) |

**Cuándo convienen:**
- Estructura libre: puede variar entre registros de la misma familia de columnas.
- Registrar eventos (logs de aplicaciones).
- Contar y clasificar accesos para después hacer análisis estadístico.
- Las familias de columnas pueden guardar datos, etiquetas, categorías, vínculos, y referenciar otros keyspaces.

**Cuándo NO convienen:**
- Sistemas que requieren transacciones **ACID** en lecturas y escrituras (ver [[5_Seguridad-Transacciones#ACID]]).
- Agregaciones (`SUM`, `AVG`), porque se hacen del lado del cliente.
- Prototipos tempranos donde todavía no se conocen los patrones de consulta.
- Cuando hay que cambiar el esquema de consultas existentes: es caro.

> [!NOTE]
> La slide 6 dice que las agregaciones son del lado del cliente, pero la slide 73 muestra `SELECT SUM(players) FROM plays`. Las dos cosas conviven: desde Cassandra 2.2 CQL tiene funciones de agregación nativas, pero (esto viene de afuera de la cátedra) se calculan en el coordinador y sobre grandes volúmenes o muchas particiones siguen siendo caras. Para el parcial quedate con la idea de la cátedra: **no es una base para análisis agregado**.

---

## Arquitectura y características

> [!IMPORTANT]
> **Teorema CAP:** Cassandra es **AP**. Da **tolerancia a particiones** y **disponibilidad** a cambio de ser **eventualmente consistente** (ver [[6_NoSql#Teorema CAP]] y [[6_NoSql#Transacciones BASE]]). El nivel de consistencia es **configurable, incluso por query**.

| Característica | Qué significa |
|---|---|
| Distribuida | Los datos se reparten entre los nodos del cluster |
| Alta disponibilidad | Si se cae un nodo, el servicio no se degrada |
| Sin punto único de falla | Los datos se replican automáticamente en varios nodos |
| Escala linealmente | El rendimiento crece proporcional a los nodos: si 2 nodos soportan 100.000 ops/s, 4 nodos soportan 200.000 |
| Escala horizontalmente | Se agregan nodos de hardware barato |
| Esquema dinámico | La estructura puede cambiar en tiempo de ejecución |
| Multi data center | Agrupa nodos lógicamente para tener los datos más cerca del usuario |

### Peer-to-peer

Cassandra usa una arquitectura **peer-to-peer (P2P)**: no hay maestro-esclavo ni nodos "principales" (a diferencia de los [[7_MongoDB#Replica sets|replica sets de MongoDB]], que tienen un primario).

- El cliente se conecta a **cualquier** nodo. Ese nodo actúa de **coordinador** de la query: determina qué nodos tienen los datos y deben responder.
- El **driver** decide qué nodo es coordinador, según una política predefinida.
- Topología de **anillo**: los datos se distribuyen alrededor del anillo para evitar cuellos de botella.

![](attachments/Pasted%20image%2020261010120025.png)

### Distribución por token

> [!IMPORTANT]
> Cada fila tiene un **token** único calculado con una **función hash** sobre su clave de partición. Los nodos se reparten equitativamente el rango de tokens
> $$[-2^{63},\ 2^{63}]$$
> y eso define el **nodo primario** de cada fila. Después Cassandra replica según el **factor de replicación** configurado.

> [!WARNING]
> En la slide 12 el rango aparece como "-263 a 263": es un error de extracción del exponente, es $-2^{63}$ a $2^{63}$.

### Elementos del cluster

| Elemento | Definición |
|---|---|
| **Nodo** | Instancia de Cassandra que almacena datos y responde lecturas/escrituras. Es responsable de una parte del conjunto de datos, según el algoritmo de partición |
| **Data center** | Conjunto de nodos agrupados lógicamente. Puede estar en un solo lugar o distribuido geográficamente. Sirve para separar cargas (ej. uno para análisis y otro para tiempo real) o replicar entre zonas |
| **Cluster** | Todos los nodos bajo una configuración común. Incluye uno o más data centers |

---

## Replicación y protocolo Gossip

- Uno o más nodos actúan como **réplicas** de un dato.
- Si algunas réplicas responden con un valor desactualizado, el coordinador devuelve al cliente el **más reciente** (el de timestamp más nuevo).
- Después hace una **reparación de lectura** (*read repair*) en segundo plano para actualizar las réplicas obsoletas.

> [!IMPORTANT]
> **Protocolo Gossip** ("chisme"): comunicación entre pares en segundo plano, inspirada en cómo se propagan las epidemias (el vecino le informa los cambios al vecino). Lo usan los nodos para comunicarse y **detectar nodos defectuosos** en el cluster.

---

## Almacenamiento: Commit log, MemTable y SSTable

| Componente | Dónde vive | Definición |
|---|---|---|
| **Commit log** | Disco | Mecanismo de recuperación ante fallas. **Toda** escritura se registra acá primero |
| **MemTable** | Memoria | Estructura residente en memoria con las escrituras de cada familia de columnas |
| **SSTable** | Disco | Archivo donde se vuelca la MemTable cuando se llena. **Inmutable** una vez creado |
| **Filtro de Bloom** | Memoria | Algoritmo rápido y **no determinista** para probar si un elemento pertenece a un conjunto. Es un tipo especial de caché; se consulta en cada lectura |

> [!TIP]
> El filtro de Bloom sirve para saltear SSTables: si dice "no está", seguro no está y no hace falta leer ese archivo. Si dice "está", *puede* que esté (falsos positivos sí, falsos negativos no). Esta precisión sobre falsos positivos/negativos viene de afuera de la cátedra.

### Escritura en un nodo

![](attachments/Pasted%20image%2020261010120031.png)

1. **Commit log:** se escribe toda la actividad para garantizar **durabilidad**. Si el sistema se apaga, se recupera desde acá.
2. **MemTable:** se guarda la escritura temporalmente en memoria.
3. **Flush a SSTable:** cuando la MemTable se llena, se vuelca a una SSTable en disco y se libera la memoria.
4. **Compactación** (periódica): fusiona SSTables para reducir su cantidad y elimina datos viejos. El objetivo es que las lecturas sean más eficientes.

> [!IMPORTANT]
> **Las escrituras son "baratas" y las lecturas son "caras".** Como las SSTables son inmutables, los datos de una misma partición pueden quedar repartidos en varias SSTables, y una lectura suele tener que buscar en varias.

> [!EXAMPLE]+ Escritura paso a paso
> Tabla `usuarios` con `PRIMARY KEY (mes, nombre)` (la de las slides 63–65).
>
> | Paso | Acción | Commit log | MemTable | SSTables en disco |
> |---|---|---|---|---|
> | 1 | `INSERT (5, 'Albert', 'ap67', ...)` | `Albert` | `Albert` | — |
> | 2 | `INSERT (5, 'John', 'uj89', ...)` | `Albert, John` | `Albert, John` | — |
> | 3 | MemTable llena → **flush** | (se puede descartar lo ya volcado) | vacía | `SST1 = {Albert, John}` |
> | 4 | `UPDATE` de John (`mes=5`) | `John'` | `John'` | `SST1` |
> | 5 | flush | — | vacía | `SST1 = {Albert, John}`, `SST2 = {John'}` |
> | 6 | Lectura de la partición `mes=5` | | | Hay que mirar **SST1 y SST2** y quedarse con el `John` de timestamp más nuevo → lectura cara |
> | 7 | **Compactación** | | | `SST3 = {Albert, John'}` (la versión vieja de John se descarta) |
>
> Después del paso 7 la misma lectura toca una sola SSTable.

### Escritura en el cluster

![](attachments/Pasted%20image%2020261010120035.png)

1. El **coordinador** envía la petición a **todas** las réplicas que contienen la fila.
2. Los datos se escriben en todos los nodos disponibles que reciben la petición.
3. El **nivel de consistencia** determina cuántos nodos tienen que responder para considerar la escritura exitosa.

> [!NOTE]
> "Escritura exitosa" en un nodo = escrita en el **Commit log y en la MemTable**. No hace falta que llegue a una SSTable.

### Borrado

- Las SSTables son inmutables, así que borrar una fila no la saca del disco: se marca con un **tombstone** (lápida).
- En la **compactación** se eliminan definitivamente las filas marcadas.
- **Tiempo de gracia:** si un nodo con la fila estaba caído al momento del borrado, todavía puede recibir la orden si se recupera dentro de un intervalo predefinido.
- **TTL** (*time to live*): los datos pueden tener fecha de caducidad, a partir de la cual se marcan como eliminados.

> [!WARNING]
> Si el nodo caído **no** vuelve dentro del tiempo de gracia, la fila no queda marcada y aparecen **inconsistencias entre réplicas** (el dato borrado puede "resucitar"). Por eso se recomienda mantenimiento periódico con la operación `repair`.

> [!NOTE]
> La slide 39 dice "LLTS"; casi seguro es un typo de **TTL**, que es como se llama en Cassandra (`INSERT ... USING TTL 86400`). La sintaxis viene de afuera de la cátedra.

### Lectura

En cada nodo que recibe la lectura:

1. Se busca por key en la **MemTable**.
2. Si los datos (o parte) no están, se va a las **SSTables** (con ayuda del filtro de Bloom).
3. Si no hubo compactación reciente, probablemente haya que leer **varias** SSTables → lentitud.

El coordinador obtiene el resultado con tres mecanismos:

| Mecanismo | Qué hace |
|---|---|
| **Direct read request** | El coordinador contacta a **un** nodo con la réplica y la recupera |
| **Digest request** | Contacta a tantas réplicas como pida el nivel de consistencia (las que responden **más rápido**) y compara contra lo devuelto por el direct read. Si hay inconsistencias gana el **timestamp más reciente**, y eso se devuelve al cliente. Después manda un digest al resto de las réplicas para verificar que también sean consistentes |
| **Background read repair request** | Si hay réplicas inconsistentes, las de timestamp más viejo se sobrescriben con la más actual |

> [!EXAMPLE]+ Lectura con QUORUM y RF = 3
> Réplicas de la fila: nodos 4, 5 y 8. El cliente se conecta al nodo 1 (coordinador). Nivel de consistencia `QUORUM` → hacen falta $\lfloor 3/2 \rfloor + 1 = 2$ respuestas.
>
> 1. **Direct read:** el nodo 1 le pide el dato completo al nodo 4 → `likes = 10, ts = 100`.
> 2. **Digest:** le pide un digest al nodo 8 (el que responde más rápido) → corresponde a `likes = 12, ts = 105`. Ya tiene 2 respuestas = quórum.
> 3. Hay inconsistencia: gana `ts = 105`. Se devuelve `likes = 12` al cliente.
> 4. Se manda digest al nodo 5, la réplica restante, para chequearla.
> 5. **Background read repair:** el nodo 4 (y el 5 si también estaba viejo) se sobrescriben con `likes = 12, ts = 105`.

---

## Consistencia configurable

> [!IMPORTANT]
> **Consistency level:**
> - Al **escribir**: en cuántas réplicas se debe escribir para confirmarle al cliente.
> - Al **leer**: cuántas réplicas deben responder para devolverle los datos al cliente.
>
> $$\text{Quorum} = \left\lfloor \frac{\sum \text{replication\_factors}}{2} \right\rfloor + 1$$

Niveles de consistencia en escritura (slide 46):

| Nivel | Descripción |
|---|---|
| `ANY` | Se escribe en al menos un nodo disponible |
| `ONE` / `TWO` / `THREE` | Se escribe en el commit log y la memtable de al menos 1 / 2 / 3 réplicas |
| `QUORUM` | Se escribe en un quórum de réplicas. Consistencia fuerte si se tolera cierto nivel de fallo. Se calcula como $(\text{replication\_factor}/2)+1$ |
| `LOCAL_QUORUM` | Quórum de réplicas en el **mismo data center** que el coordinador. Evita la latencia entre data centers |
| `EACH_QUORUM` | Consistencia fuerte: quórum de réplicas en **todos** los data centers |
| `ALL` | Se escribe en **todas** las réplicas del cluster. Máxima consistencia y **mínima disponibilidad** |

```sql
SELECT * FROM users WHERE state='TX' USING CONSISTENCY QUORUM;
```

> [!WARNING]
> Esa sintaxis (`USING CONSISTENCY`) es la de la cátedra pero es de versiones viejas de CQL y ya no existe. Hoy en `cqlsh` se fija con el comando `CONSISTENCY QUORUM;` antes de la query, o desde el driver. Esto viene de afuera de la cátedra.

> [!EXAMPLE]+ Cálculo de quórum
> | Configuración | Cálculo | Quórum | Réplicas que se pueden caer |
> |---|---|---|---|
> | RF = 3 | $\lfloor 3/2 \rfloor + 1$ | 2 | 1 |
> | RF = 5 | $\lfloor 5/2 \rfloor + 1$ | 3 | 2 |
> | RF = 6 | $\lfloor 6/2 \rfloor + 1$ | 4 | 2 |
> | 2 DCs con RF = 3 cada uno | $\lfloor (3+3)/2 \rfloor + 1$ | 4 | 2 en total |
> | `LOCAL_QUORUM`, DC con RF = 3 | $\lfloor 3/2 \rfloor + 1$ | 2 en el DC local | 1 en el DC local |

> [!TIP]
> Regla práctica (de afuera de la cátedra): si $R + W > RF$, donde $R$ y $W$ son las réplicas que responden en lectura y escritura, toda lectura se cruza con al menos una réplica que tiene la última escritura → **consistencia fuerte**. Por eso `QUORUM` en lectura y escritura con RF = 3 da $2 + 2 > 3$.

---

## Modelo de datos

![](attachments/Pasted%20image%2020261010120015.png)

Toda fila tiene una **row key** única y una serie de pares (nombre de columna, valor).

| Estructura | Definición |
|---|---|
| **Columna** | Unidad básica de almacenamiento. Tripla `(name, value, timestamp)` |
| **Supercolumna** | Columna cuyos valores son otras columnas (*subcolumns*), ordenadas e ilimitadas. **No tienen timestamp** y **no son recursivas** (un solo nivel). Son opcionales y **se desaconseja su uso** |
| **Fila** | Conjunto de columnas de una familia con valor asignado, referenciado por la **row key**. Equivale a la fila relacional |
| **Familia de columnas** | Contenedor de filas, cada una accedida por su row key. Se guarda en un archivo separado, ordenado por la row key. Equivale a la **tabla** |
| **Keyspace** | Agrupación de familias de columnas, normalmente una por aplicación. Equivale al **esquema** |
| **Cluster** | Las máquinas de una instancia de Cassandra. Puede tener varios keyspaces |

> [!NOTE]
> El **timestamp** lo provee la aplicación, por eso hay que tener el reloj sincronizado con el del cluster. Se usa para **resolver conflictos** (consistencia eventual: gana el más nuevo). Puede ser de cualquier tipo, pero por convención son los **microsegundos desde el 01/01/1970**.

Dos filas de la misma familia de columnas pueden tener columnas distintas (una con 50 y otra con 300), porque se accede siempre por la row key. Ejemplo de la slide 58:

| Key | Nombre_de_usuario | Fecha_de_registro | Teléfono |
|---|---|---|---|
| `9871r…12mj` | Braulio123 | 2015-04-01T20:21:01 | 898765432 |
| `911u…1pop` | Joaquín89 | 2014-04-02T10:01:01 | 123764436 |

**Tipos de columnas:** standard, counter, collection (`SET`, `LIST`, `MAP`), user-defined type, tuple, timestamp.

### Keyspace

```sql
CREATE KEYSPACE nombre WITH replication =
   {'class': 'SimpleStrategy', 'replication_factor': 3};
```

| Atributo | Significado |
|---|---|
| `replication_factor` | Cantidad de nodos que guardan una copia de cada dato (3 → 3 réplicas) |
| `SimpleStrategy` | Configuraciones simples o de **un solo** data center |
| `NetworkTopologyStrategy` | **Varios** data centers |

### Particionado

| Partitioner | Cómo distribuye |
|---|---|
| **RandomPartitioner** | Hash de la clave → distribución aleatoria y **uniforme** entre nodos |
| **OrderPreservingPartitioner** | Claves cercanas quedan en el mismo nodo o en nodos adyacentes: mantiene el **orden natural** de las claves |

> [!NOTE]
> De afuera de la cátedra: el partitioner por defecto hoy es `Murmur3Partitioner`, que también es por hash (como el Random). El order-preserving está desaconsejado porque genera **hot spots**.

---

## Clave primaria: partition key y clustering key

> [!IMPORTANT]
> La **row key** (clave primaria) tiene dos partes, y **el orden importa**:
> - **Partition key** (clave de partición): el **primer** componente. Las filas con el mismo valor se guardan en la **misma partición**, físicamente juntas. Define en qué nodo vive la fila.
> - **Clustering key** (clave de agrupamiento/ordenamiento): el **resto** de los componentes. Determina el **orden físico** de las filas dentro de la partición.
>
> Ambas pueden ser **compuestas**. Para que la partition key sea compuesta se usa un paréntesis extra.

| Definición | Partition key | Clustering key |
|---|---|---|
| `PRIMARY KEY (a)` | `a` | — |
| `PRIMARY KEY (a, b, c)` | `a` | `b, c` |
| `PRIMARY KEY ((a, b), c)` | `a, b` (compuesta) | `c` |
| `PRIMARY KEY ((idusuario, mes), nombre, registro)` | `idusuario, mes` | `nombre, registro` |

La tabla mínima posible es `CREATE TABLE t (k text PRIMARY KEY);`.

```sql
CREATE TABLE usuarios (
  idusuario text,
  nombre    text,
  mes       int,
  registro  time,
  PRIMARY KEY (mes, nombre)   -- partition key: mes, clustering key: nombre
);
```

> [!EXAMPLE]+ Asignación de espacio según la clave (slides 65–66)
> Mismos 4 registros, dos claves distintas.
>
> **Con `PRIMARY KEY (mes, nombre)`:** se agrupa por `mes` y dentro se ordena por `nombre`.
>
> | Partición | Mes | Nombre | Idusuario | Registro |
> |---|---|---|---|---|
> | 1 | 5 | Albert | ap67 | 21-05-2015 |
> | 1 | 5 | John | uj89 | 02-05-2013 |
> | 2 | 11 | John | jk98 | 09-11-2013 |
> | 2 | 11 | Peter | pk34 | 08-11-2013 |
>
> - Las réplicas de las filas de una partición están físicamente juntas.
> - Las distintas particiones pueden vivir en **nodos distintos**.
>
> **Con `PRIMARY KEY (idusuario)`:** `idusuario` es único, así que **cada partición tiene una sola fila** y no hay clustering key.
>
> | Partición | Mes | Nombre | Idusuario | Registro |
> |---|---|---|---|---|
> | 1 | 5 | Albert | ap67 | 21-05-2015 |
> | 2 | 5 | John | uj89 | 02-05-2013 |
> | 3 | 11 | John | jk98 | 09-11-2013 |
> | 4 | 11 | Peter | pk34 | 08-11-2013 |

### Índices

| Tipo | Definición |
|---|---|
| **Primario** | Único por fila. El partitioner y la estrategia de réplica asignan las filas a nodos según este índice. Como cada nodo sabe qué rango guarda, la consulta solo escanea las réplicas de las filas buscadas |
| **Secundario** | Índice sobre una columna. Se implementa como una **tabla oculta** separada de la original. **No se recomienda** para consultas que revisan mucho volumen para devolver pocos resultados |

Comparar con [[5_Seguridad-Transacciones#Indices|los índices en relacional]] y [[7_MongoDB#Índices|en MongoDB]].

---

## Modelado de datos

> [!IMPORTANT]
> En Cassandra el modelo se diseña **a partir de las consultas** (el patrón de acceso), no de las relaciones entre los datos. **No existen los JOIN ni las subqueries**: los datos están **desnormalizados**.

- Analizar primero qué queries se van a ejecutar.
- Elegir bien la **partition key**: con ella se distribuyen los datos en el cluster. Hay que repartirlos para evitar cuellos de botella.
- Minimizar la cantidad de particiones que hay que leer en cada consulta.
- Cassandra conviene cuando la **alta disponibilidad** es prioridad y cuando, con grandes volúmenes, importa más la velocidad de acceso que la normalización.

> [!TIP]
> Es lo opuesto a [[1_Bases-de-datos-relacionales#Derivación a esquema lógico|derivar un esquema relacional]]: allá normalizás y después escribís las queries; acá escribís las queries y armás una tabla por consulta, duplicando datos si hace falta. Comparalo también con el modelado embebido vs normalizado de [[7_MongoDB|MongoDB]].

---

## CQL

**Cassandra Query Language**: derivado reducido de SQL. Se usa desde la shell `cqlsh`, herramientas gráficas como DevCenter, o drivers para varios lenguajes.

```sql
CREATE KEYSPACE MiEspacioClaves
  WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 3 };

USE MiEspacioClaves;

CREATE COLUMNFAMILY MisColumnas (id text, Apellido text, Nombre text, PRIMARY KEY(id));

INSERT INTO MisColumnas (id, Apellido, Nombre) VALUES ('1', 'Perez', 'Juan');

SELECT * FROM MisColumnas;
```

```text
 id | nombre | apellido
----+--------+---------
  1 |  Juan  |  Perez

(1 fila)
```

> [!NOTE]
> `CREATE COLUMNFAMILY` es sinónimo viejo de `CREATE TABLE`. Además los nombres sin comillas se pasan a minúscula (por eso el resultado muestra `nombre` y `apellido`).

### Consultas

```sql
SELECT idusuario FROM usuarios WHERE idusuario = '1p67' ORDER BY nombre;
```

| Parte | Regla |
|---|---|
| `SELECT` | Proyección. `*` devuelve todas las columnas |
| `FROM` | **Una sola tabla** (no hay JOINs) |
| `WHERE` | Solo columnas de la **partition key** o con **índice secundario**. Si la partition key es compuesta, hay que incluir **todos** sus campos |
| `ORDER BY` | Solo sobre columnas de la **clustering key** |

> [!WARNING]
> Con la partition key compuesta `(idusuario, mes)`:
> ```sql
> -- ERROR: falta idusuario, que es parte de la partition key
> SELECT idusuario FROM usuarios WHERE mes = 9;
>
> -- OK: están todos los campos de la partition key
> SELECT idusuario FROM usuarios WHERE mes = 9 AND idusuario = 'agp88';
> ```
> Sin la partition key completa Cassandra no puede calcular el token, y por lo tanto no sabe en qué nodo buscar.

### ALLOW FILTERING

```sql
CREATE TABLE blogs (
  blogId  int,
  time1   int,
  time2   int,
  author  text,
  content text,
  PRIMARY KEY (blogId, time1, time2)
);

SELECT * FROM blogs;                              -- OK: trae todo
SELECT * FROM blogs WHERE time1 = 1418306451235;  -- ERROR
```

```text
Bad Request: Cannot execute this query as it might involve data filtering and thus may have
unpredictable performance. If you want to execute this query despite the performance
unpredictability, use ALLOW FILTERING.
```

La única forma de resolverla es leer **todas** las filas y filtrar las que tengan ese `time1` (no se filtró por la partition key `blogId`). Cassandra no sabe si eso es razonable, porque depende de la distribución de los datos:

- 1 millón de filas y el 95% cumple → es relativamente eficiente, usá `ALLOW FILTERING`.
- 1 millón de filas y solo 2 cumplen → carga 999.998 filas para nada. Si la query es frecuente, mejor crear un **índice** sobre `time1`.

> [!TIP]
> `ALLOW FILTERING` no es un error del motor sino un aviso: "esto puede ser carísimo, decidí vos". En el parcial, si aparece, justificá con la selectividad del filtro.

### Vistas materializadas

```sql
CREATE MATERIALIZED VIEW monkeySpecies_by_population AS
  SELECT * FROM monkeySpecies
  WHERE population IS NOT NULL AND species IS NOT NULL
  PRIMARY KEY (population, species)
  WITH comment = 'Allow query by population instead of species';
```

Permiten consultar la misma información con **otra partition key**. No se pueden actualizar directamente, pero los cambios en la tabla base se propagan a la vista. Comparar con las [[2_Vistas#Vistas materializadas - PostgreSQL|vistas materializadas de PostgreSQL]], donde hay que refrescarlas a mano.

### Funciones

- **Escalares:** toman valores y producen una salida.
  ```sql
  SELECT * FROM myTable WHERE date >= currentDate() - 2d;   -- últimos dos días
  ```
- **De agregación:** agregan varias filas del resultado de un `SELECT`.
  ```sql
  SELECT SUM(players) FROM plays;
  ```

### Triggers

```sql
CREATE TRIGGER myTrigger ON myTable
  USING 'org.apache.cassandra.triggers.AuditTrigger';   -- clase Java

DROP TRIGGER myTrigger;
```

- La lógica se escribe en cualquier lenguaje de la **JVM** y vive **fuera de la base**, en `lib/triggers` del directorio de instalación.
- Se carga al arrancar el cluster y existe en **todos** los nodos.
- Se dispara **antes** de la sentencia DML, lo que asegura la atomicidad.

Comparar con los triggers procedurales de [[4_restricciones-de-integridad#Procedural|PostgreSQL]].

---

## Seguridad

- Usuarios con login, password y permisos vía `GRANT` / `REVOKE` (igual que en [[5_Seguridad-Transacciones#Usuarios y permisos|relacional]]).
- Encriptación entre clientes y cluster, y entre nodos.
- Varias opciones de **backup**; conviene hacerlos seguido (ej. ante borrados accidentales).
- Herramientas externas (DataStax Enterprise) agregan autenticación externa, encriptación de tablas y auditoría.

```sql
-- Roles
CREATE ROLE alice WITH PASSWORD = 'password_a' AND LOGIN = true;
GRANT report_writer TO alice;
REVOKE report_writer FROM alice;
LIST ROLES;

-- Usuarios
CREATE USER alice WITH PASSWORD 'password_a' SUPERUSER;

-- Permisos
GRANT SELECT ON ALL KEYSPACES TO data_reader;
REVOKE SELECT ON ALL KEYSPACES FROM data_reader;
LIST ALL PERMISSIONS ON keyspace1.table1 OF bob;
```
