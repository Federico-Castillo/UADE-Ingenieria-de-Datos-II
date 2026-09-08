# Clase 6 — 8/9

Bases de datos de Columnas. Laboratorio 3 — Apache Cassandra: CQL, modelado
orientado a consultas, colecciones, tipos definidos por el usuario (UDT),
tuplas y vectores.

## Resumen

Laboratorio práctico sobre [Apache Cassandra](https://cassandra.apache.org/doc/latest/),
una base de datos NoSQL distribuida orientada a columnas. A diferencia de las
bases relacionales, Cassandra se organiza en:

- **Keyspaces**: similares a una base de datos (agrupan tablas y definen la
  estrategia de replicación).
- **Tables**: similares a las tablas relacionales.
- **Partition Keys**: determinan en qué nodo/partición física se almacenan
  los datos.
- **Clustering Keys**: determinan el orden de los datos dentro de una misma
  partición.

### Comandos básicos (`cqlsh`)

```sql
-- Ingresar al cliente CQL
cqlsh

-- Consultar la versión del cliente, del cluster y del protocolo CQL
SHOW VERSION;

-- Listar los keyspaces (de sistema y de usuario)
DESCRIBE KEYSPACES;

-- Crear un keyspace
CREATE KEYSPACE nombre
WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1};

-- Seleccionar un keyspace existente
USE nombre_keyspace;
```

### Tipos de datos básicos

| Tipo | Tamaño / rango | Descripción |
|---|---|---|
| `tinyint` | 1 byte (-128 a 127) | Entero de 8 bits firmado. |
| `smallint` | 2 bytes (-32.768 a 32.767) | Entero de 16 bits firmado. |
| `int` | 4 bytes | Entero de 32 bits firmado (estándar). |
| `bigint` | 8 bytes | Entero de 64 bits firmado. |
| `varint` | Variable | Entero de precisión arbitraria. |
| `float` / `double` | 4 / 8 bytes | Punto flotante simple / doble precisión. |
| `decimal` | Variable | Decimal de precisión exacta (moneda). |
| `counter` | 8 bytes | Solo se incrementa/decrementa; no admite `INSERT` directo. |
| `boolean` | — | `true` / `false`. |
| `blob` | — | Datos binarios sin estructurar (hex). |
| `inet` | — | Dirección IPv4 o IPv6. |
| `timestamp` | 8 bytes | Fecha y hora (ms desde epoch Unix); admite formato ISO. |
| `date` | 4 bytes | Fecha sin hora (`YYYY-MM-DD`). |
| `time` | — | Hora sin fecha, precisión de nanosegundos. |
| `duration` | — | Lapso en meses, días y nanosegundos (ej. `3d2h`, `1mo2d`). |
| `uuid` | 128 bits | Identificador único (RFC 4122); evita colisiones de PK en clústeres. |
| `timeuuid` | — | UUID v1, incluye timestamp de generación → permite orden cronológico. |
| `text` / `varchar` | — | Cadena UTF-8 (tipo recomendado; ambos son equivalentes). |
| `ascii` | — | Cadena limitada a caracteres ASCII. |

### Colecciones

Permiten guardar múltiples valores en una sola columna, sin normalizar en
tablas secundarias:

- **`list<type>`**: colección ordenada, admite duplicados, preserva el orden
  de inserción. Ej.: `emails list<text>` → `['a@test.com', 'b@test.com']`.
- **`set<type>`**: colección sin duplicados. Ej.: `tags set<text>` →
  `{'java', 'nosql', 'cassandra'}`.
- **`map<type1, type2>`**: pares clave-valor. Ej.: `scores map<text, int>` →
  `{'math': 90, 'physics': 85}`.

### Tipos avanzados y personalizados

**User-Defined Types (UDT)**: permiten definir una estructura propia con
`CREATE TYPE`, útil para agrupar campos relacionados en una sola columna.

```sql
CREATE TYPE direccion (
    calle text,
    ciudad text,
    codigo_postal int
);

CREATE TABLE usuarios (
    id uuid PRIMARY KEY,
    nombre text,
    domicilio frozen<direccion>
);
```

Preguntas planteadas en el laboratorio para discutir: ¿cuándo conviene usar
un UDT?, ¿qué hace el modificador `frozen`?, ¿qué ventajas tiene su uso?

**Tuplas**: colección de valores anónimos, de longitud fija y tipos
heterogéneos (`tuple<...>`), implícitamente `frozen`.

Ejemplo de modelado (LMS tipo Moodle, entregas de trabajos prácticos):

```sql
CREATE TABLE entregas_tp (
    materia_id uuid,
    materia_nombre text,
    alumno_id uuid,
    alumno_nombre text,
    tp_id int,
    fecha_entrega timestamp,
    ubicacion_gps tuple<double, double>,
    calificacion tuple<decimal, text>,
    PRIMARY KEY ((materia_id, tp_id), alumno_id)
);

INSERT INTO entregas_tp (
    materia_id, materia_nombre, tp_id, alumno_id, alumno_nombre,
    fecha_entrega, ubicacion_gps, calificacion
) VALUES (
    a3b8e8f0-1234-11ed-a100-0242ac120002,
    'Ingeniería de Datos II',
    1,
    c9d8e7f0-5678-11ed-a100-0242ac120002,
    'Cacho Gomez',
    toTimestamp(now()),
    (-34.6037, -58.3816), -- (Latitud, Longitud de CABA)
    (8.50, 'Aprobado')
);

-- Se intenta actualizar solo el segundo valor de la tupla:
UPDATE entregas_tp
SET calificacion[1] = 'Promocionado'
WHERE materia_id = a3b8e8f0-1234-11ed-a100-0242ac120002
  AND tp_id = 1
  AND alumno_id = c9d8e7f0-5678-11ed-a100-0242ac120002;
```

Ejercicio de discusión: ¿por qué no funciona el `UPDATE` anterior sobre un
elemento de la tupla? ¿Cómo debería escribirse correctamente?

### Vectores (`VECTOR`)

Un vector es una lista ordenada de números decimales de longitud fija
(`VECTOR<FLOAT, n>`), donde cada posición representa una dimensión en un
espacio abstracto. Se usan para almacenar **embeddings**: representaciones
numéricas de texto, imágenes o audio generadas por modelos de IA (OpenAI,
Hugging Face, Cohere, etc.), donde conceptos similares quedan cerca en el
espacio vectorial. En producción suelen tener 768, 1536 o 3072 dimensiones.

A diferencia de una consulta SQL tradicional (coincidencia exacta), con
`VECTOR` se busca por **similitud semántica**:

1. Se convierte la consulta del usuario a un vector.
2. Se calcula la distancia geométrica (coseno, euclidiana) entre ese vector
   y los almacenados.
3. La base devuelve los registros conceptualmente más cercanos, aunque no
   compartan las mismas palabras.

Aplicaciones típicas: **búsqueda semántica**, **motores de recomendación** y
arquitecturas **RAG** (Retrieval-Augmented Generation, para dar contexto de
negocio a un LLM).

```sql
SELECT article_id, title,
       similarity_cosine(embedding, [0.15, 0.42, 0.88, 0.30, 0.20]) AS score
FROM articles
ORDER BY embedding ANN OF [0.15, 0.42, 0.88, 0.30, 0.20]
LIMIT 3;
```

`ANN OF [...]` le indica al motor que use el índice **HNSW** (Hierarchical
Navigable Small World) para estimar rápidamente los vectores más cercanos
sin comparar contra todas las filas.

### Modelado de consultas (ejercicios)

**A) Liga de fútbol** — modelar equipos, jugadores, temporadas, partidos y
goleadores, diseñando tablas orientadas a satisfacer estas consultas:

- Partidos por torneo (orden descendente por fecha).
- Partidos por equipo (historial completo).
- Partidos por equipo y temporada.
- Últimos 10 partidos de una temporada.
- Ranking de goleadores.

Para cada consulta se debía analizar: ¿qué columna(s) es la partition key?,
¿qué cambio en la consulta invalidaría la tabla?, ¿cómo impacta en el tamaño
de las particiones?

**B) Recomendaciones de películas con IA** — modelar y poblar:

```sql
CREATE TABLE movies (
    movie_id UUID PRIMARY KEY,
    title TEXT,
    genre TEXT,
    year INT,
    embedding VECTOR<FLOAT, 8>
);

CREATE CUSTOM INDEX IF NOT EXISTS idx_movies_embedding
ON movies (embedding) USING 'StorageAttachedIndex'
WITH OPTIONS = {'similarity_function': 'COSINE'};

INSERT INTO movies (movie_id, title, genre, year, embedding)
VALUES (uuid(), 'Inception', 'Sci-Fi', 2010,
        [0.12, 0.45, 0.91, 0.33, 0.18, 0.50, 0.05, 0.82]);

SELECT movie_id, title, genre, year,
       similarity_cosine(embedding, [0.12, 0.45, 0.91, 0.33, 0.18, 0.50, 0.05, 0.82]) AS score
FROM movies
ORDER BY embedding ANN OF [0.12, 0.45, 0.91, 0.33, 0.18, 0.50, 0.05, 0.82]
LIMIT 5;
```

Tareas del ejercicio: insertar 10 películas con embeddings ficticios;
consultar todas; buscar por género y por año; agregar una colección
`SET<TEXT>` para actores; agregar un `MAP<TEXT,TEXT>` con información
adicional; incorporar un UDT para el director; crear el índice sobre el
vector; y explicar el funcionamiento de la sentencia `SELECT` con `ANN OF`.
