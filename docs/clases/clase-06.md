# Clase 6 — 8/9

Bases de datos de Columnas. Laboratorio 3 — Apache Cassandra: CQL, modelado
orientado a consultas, colecciones, tipos definidos por el usuario (UDT),
tuplas y vectores.

**Material:** [Laboratorio 3 — Cassandra](../materiales/archivos/clase-06-laboratorio_Cassandra.pdf){: target="_blank" }

## Resumen

Laboratorio práctico sobre [Apache Cassandra](https://cassandra.apache.org/doc/latest/){: target="_blank" },
una base de datos NoSQL distribuida orientada a columnas. A diferencia de las
bases relacionales, Cassandra se organiza en:

- **Keyspaces**: similares a una base de datos (agrupan tablas y definen la
  estrategia de replicación).
- **Tables**: similares a las tablas relacionales.
- **Partition Keys**: determinan en qué nodo/partición física se almacenan
  los datos.
- **Clustering Keys**: determinan el orden de los datos dentro de una misma
  partición.

??? note "Comandos básicos (`cqlsh`)"
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

??? abstract "Tipos de datos básicos"
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

??? info "Colecciones"
    Permiten guardar múltiples valores en una sola columna, sin normalizar en
    tablas secundarias:

    - **`list<type>`**: colección ordenada, admite duplicados, preserva el
      orden de inserción. Ej.: `emails list<text>` →
      `['a@test.com', 'b@test.com']`.
    - **`set<type>`**: colección sin duplicados. Ej.: `tags set<text>` →
      `{'java', 'nosql', 'cassandra'}`.
    - **`map<type1, type2>`**: pares clave-valor. Ej.: `scores map<text, int>`
      → `{'math': 90, 'physics': 85}`.

??? tip "Tipos avanzados y personalizados"
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

??? example "Vectores (`VECTOR`)"
    Un vector es una lista ordenada de números decimales de longitud fija
    (`VECTOR<FLOAT, n>`), donde cada posición representa una dimensión en un
    espacio abstracto. Se usan para almacenar **embeddings**: representaciones
    numéricas de texto, imágenes o audio generadas por modelos de IA (OpenAI,
    Hugging Face, Cohere, etc.), donde conceptos similares quedan cerca en el
    espacio vectorial. En producción suelen tener 768, 1536 o 3072 dimensiones.

    A diferencia de una consulta SQL tradicional (coincidencia exacta), con
    `VECTOR` se busca por **similitud semántica**:

    1. Se convierte la consulta del usuario a un vector.
    2. Se calcula la distancia geométrica (coseno, euclidiana) entre ese
       vector y los almacenados.
    3. La base devuelve los registros conceptualmente más cercanos, aunque no
       compartan las mismas palabras.

    Aplicaciones típicas: **búsqueda semántica**, **motores de recomendación**
    y arquitecturas **RAG** (Retrieval-Augmented Generation, para dar contexto
    de negocio a un LLM).

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

??? question "Modelado de consultas (ejercicios, resueltos)"
    **A) Liga de fútbol** — modelar equipos, jugadores, temporadas, partidos y
    goleadores, diseñando tablas orientadas a satisfacer estas consultas.
    En Cassandra el modelado es "query-first": una tabla desnormalizada por
    cada patrón de consulta, no por entidad.

    ```sql
    -- 1) Partidos por torneo, orden descendente por fecha
    CREATE TABLE partidos_por_torneo (
        torneo_id uuid,
        fecha timestamp,
        partido_id uuid,
        equipo_local text,
        equipo_visitante text,
        goles_local int,
        goles_visitante int,
        temporada text,
        PRIMARY KEY (torneo_id, fecha, partido_id)
    ) WITH CLUSTERING ORDER BY (fecha DESC, partido_id ASC);

    -- 2) Partidos por equipo, historial completo
    CREATE TABLE partidos_por_equipo (
        equipo_id uuid,
        fecha timestamp,
        partido_id uuid,
        rival text,
        condicion text, -- 'local' o 'visitante'
        goles_favor int,
        goles_contra int,
        temporada text,
        PRIMARY KEY (equipo_id, fecha, partido_id)
    ) WITH CLUSTERING ORDER BY (fecha DESC, partido_id ASC);

    -- 3) Partidos por equipo y temporada
    CREATE TABLE partidos_por_equipo_temporada (
        equipo_id uuid,
        temporada text,
        fecha timestamp,
        partido_id uuid,
        rival text,
        goles_favor int,
        goles_contra int,
        PRIMARY KEY ((equipo_id, temporada), fecha, partido_id)
    ) WITH CLUSTERING ORDER BY (fecha DESC, partido_id ASC);

    -- 4) Últimos 10 partidos de una temporada
    CREATE TABLE partidos_por_temporada (
        temporada text,
        fecha timestamp,
        partido_id uuid,
        equipo_local text,
        equipo_visitante text,
        goles_local int,
        goles_visitante int,
        PRIMARY KEY (temporada, fecha, partido_id)
    ) WITH CLUSTERING ORDER BY (fecha DESC, partido_id ASC);

    SELECT * FROM partidos_por_temporada WHERE temporada = '2025' LIMIT 10;

    -- 5) Ranking de goleadores por temporada
    CREATE TABLE ranking_goleadores (
        temporada text,
        goles int,
        jugador_id uuid,
        jugador_nombre text,
        equipo text,
        PRIMARY KEY (temporada, goles, jugador_id)
    ) WITH CLUSTERING ORDER BY (goles DESC, jugador_id ASC);

    SELECT jugador_nombre, equipo, goles
    FROM ranking_goleadores
    WHERE temporada = '2025'
    LIMIT 10;
    ```

    Notas de diseño: `ranking_goleadores` guarda `goles` como parte de la
    clustering key (no como `counter`) para que Cassandra devuelva las filas
    ya ordenadas; como una clustering key no se puede `UPDATE`, cada gol
    implica borrar la fila vieja del jugador e insertar una nueva con el
    valor actualizado (lo mantiene la aplicación, no la base).

    **B) Recomendaciones de películas con IA** — modelo completo, con
    colección, mapa, UDT e índices:

    ```sql
    CREATE TYPE director (
        nombre text,
        nacionalidad text
    );

    CREATE TABLE movies (
        movie_id uuid PRIMARY KEY,
        title text,
        genre text,
        year int,
        actors set<text>,
        info map<text, text>,
        director frozen<director>,
        embedding vector<float, 8>
    );

    CREATE CUSTOM INDEX IF NOT EXISTS idx_movies_embedding
    ON movies (embedding) USING 'StorageAttachedIndex'
    WITH OPTIONS = {'similarity_function': 'COSINE'};

    -- Índices adicionales (StorageAttachedIndex) para filtrar por columnas
    -- normales sin recurrir a ALLOW FILTERING
    CREATE CUSTOM INDEX IF NOT EXISTS idx_movies_genre
    ON movies (genre) USING 'StorageAttachedIndex';

    CREATE CUSTOM INDEX IF NOT EXISTS idx_movies_year
    ON movies (year) USING 'StorageAttachedIndex';
    ```

    Carga de las 10 películas con embeddings ficticios:

    ```sql
    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'Inception', 'Sci-Fi', 2010,
        {'Leonardo DiCaprio', 'Joseph Gordon-Levitt'},
        {'idioma': 'inglés', 'duracion': '148 min'},
        {nombre: 'Christopher Nolan', nacionalidad: 'Reino Unido'},
        [0.12, 0.45, 0.91, 0.33, 0.18, 0.50, 0.05, 0.82]);

    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'The Matrix', 'Sci-Fi', 1999,
        {'Keanu Reeves', 'Carrie-Anne Moss'},
        {'idioma': 'inglés', 'duracion': '136 min'},
        {nombre: 'Lana Wachowski', nacionalidad: 'Estados Unidos'},
        [0.14, 0.40, 0.85, 0.36, 0.20, 0.48, 0.08, 0.78]);

    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'Interstellar', 'Sci-Fi', 2014,
        {'Matthew McConaughey', 'Anne Hathaway'},
        {'idioma': 'inglés', 'duracion': '169 min'},
        {nombre: 'Christopher Nolan', nacionalidad: 'Reino Unido'},
        [0.10, 0.47, 0.88, 0.30, 0.22, 0.52, 0.06, 0.80]);

    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'The Godfather', 'Drama', 1972,
        {'Marlon Brando', 'Al Pacino'},
        {'idioma': 'inglés', 'duracion': '175 min'},
        {nombre: 'Francis Ford Coppola', nacionalidad: 'Estados Unidos'},
        [0.70, 0.15, 0.20, 0.60, 0.55, 0.10, 0.65, 0.25]);

    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'Pulp Fiction', 'Crime', 1994,
        {'John Travolta', 'Uma Thurman'},
        {'idioma': 'inglés', 'duracion': '154 min'},
        {nombre: 'Quentin Tarantino', nacionalidad: 'Estados Unidos'},
        [0.65, 0.20, 0.25, 0.58, 0.50, 0.15, 0.60, 0.30]);

    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'Whiplash', 'Drama', 2014,
        {'Miles Teller', 'J.K. Simmons'},
        {'idioma': 'inglés', 'duracion': '106 min'},
        {nombre: 'Damien Chazelle', nacionalidad: 'Estados Unidos'},
        [0.68, 0.18, 0.22, 0.62, 0.52, 0.12, 0.63, 0.28]);

    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'Toy Story', 'Animation', 1995,
        {'Tom Hanks', 'Tim Allen'},
        {'idioma': 'inglés', 'duracion': '81 min'},
        {nombre: 'John Lasseter', nacionalidad: 'Estados Unidos'},
        [0.30, 0.85, 0.40, 0.10, 0.75, 0.35, 0.20, 0.55]);

    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'Coco', 'Animation', 2017,
        {'Anthony Gonzalez', 'Gael García Bernal'},
        {'idioma': 'español', 'duracion': '105 min'},
        {nombre: 'Lee Unkrich', nacionalidad: 'Estados Unidos'},
        [0.32, 0.82, 0.42, 0.12, 0.72, 0.38, 0.18, 0.58]);

    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'The Dark Knight', 'Action', 2008,
        {'Christian Bale', 'Heath Ledger'},
        {'idioma': 'inglés', 'duracion': '152 min'},
        {nombre: 'Christopher Nolan', nacionalidad: 'Reino Unido'},
        [0.55, 0.30, 0.60, 0.45, 0.28, 0.65, 0.40, 0.20]);

    INSERT INTO movies (movie_id, title, genre, year, actors, info, director, embedding)
    VALUES (uuid(), 'Parasite', 'Thriller', 2019,
        {'Song Kang-ho', 'Lee Sun-kyun'},
        {'idioma': 'coreano', 'duracion': '132 min'},
        {nombre: 'Bong Joon-ho', nacionalidad: 'Corea del Sur'},
        [0.58, 0.28, 0.58, 0.48, 0.30, 0.62, 0.42, 0.22]);
    ```

    Consultas pedidas:

    ```sql
    -- Consultar todas las películas
    SELECT title, genre, year FROM movies;

    -- Buscar por género (usa el índice SAI idx_movies_genre)
    SELECT title, year FROM movies WHERE genre = 'Sci-Fi';

    -- Buscar por año (usa el índice SAI idx_movies_year)
    SELECT title, genre FROM movies WHERE year = 2014;

    -- Recomendación: películas más similares a "Inception" por embedding
    SELECT title, genre, year,
           similarity_cosine(embedding, [0.12, 0.45, 0.91, 0.33, 0.18, 0.50, 0.05, 0.82]) AS score
    FROM movies
    ORDER BY embedding ANN OF [0.12, 0.45, 0.91, 0.33, 0.18, 0.50, 0.05, 0.82]
    LIMIT 5;
    ```

    Esta última consulta no filtra por igualdad: `ORDER BY embedding ANN OF
    [...]` le pide al motor que recorra el índice HNSW y devuelva, en orden,
    las películas cuyo `embedding` está geométricamente más cerca del vector
    de "Inception" (aproximación por vecinos más cercanos, no un escaneo
    exacto de toda la tabla), mientras que `similarity_cosine(...)` en el
    `SELECT` solo calcula y muestra ese puntaje de similitud para cada fila
    devuelta. En este caso, al compartir género y una época similar de
    valores en el embedding, se espera que "The Matrix" e "Interstellar"
    aparezcan entre las más cercanas.

## Preguntas del laboratorio

Todas las preguntas de discusión planteadas durante el laboratorio,
respondidas en un solo lugar:

??? question "¿Cuándo conviene usar un UDT?"
    Cuando un grupo de campos relacionados (por ejemplo los de una
    dirección) siempre se leen y escriben juntos y no necesitan consultarse
    ni indexarse por separado: agruparlos en un UDT evita crear una tabla
    adicional y simplifica el modelo.

??? question "¿Qué hace el modificador `frozen`?"
    Serializa el UDT (o colección) completo como un único valor binario e
    inmutable. Ya no se pueden actualizar campos individuales del UDT (solo
    reemplazar el valor entero con un nuevo `UPDATE`), pero a cambio
    Cassandra puede compararlo y "hashearlo" como un todo.

??? question "¿Qué ventajas tiene el uso de `frozen`?"
    Al estar `frozen`, el UDT puede anidarse dentro de colecciones
    (`list<frozen<direccion>>`) y usarse como parte de la primary key,
    además de reducir el overhead de almacenamiento frente a columnas
    individuales (se guarda como un solo valor en vez de una celda con
    metadata propia por cada campo).

??? question "¿Por qué no funciona `UPDATE ... SET calificacion[1] = ...` sobre una tupla?"
    Porque `tuple`, igual que un UDT, es un tipo implícitamente `frozen`: se
    trata como un valor atómico e indivisible. La sintaxis
    `columna[índice] = valor` para modificar un elemento puntual solo es
    válida sobre colecciones no congeladas (`list`, `set`, `map`), no sobre
    tuplas.

??? question "¿Cómo debería escribirse correctamente ese `UPDATE`?"
    Reemplazando la tupla completa, no un elemento individual:

    ```sql
    UPDATE entregas_tp
    SET calificacion = (8.50, 'Promocionado')
    WHERE materia_id = a3b8e8f0-1234-11ed-a100-0242ac120002
      AND tp_id = 1
      AND alumno_id = c9d8e7f0-5678-11ed-a100-0242ac120002;
    ```

??? question "Para cada tabla de la Liga de fútbol, ¿qué columna(s) es la partition key?"
    - `partidos_por_torneo`: `torneo_id`.
    - `partidos_por_equipo`: `equipo_id`.
    - `partidos_por_equipo_temporada`: la clave compuesta
      `(equipo_id, temporada)`.
    - `partidos_por_temporada`: `temporada`.
    - `ranking_goleadores`: `temporada`.

    En los cinco casos la partition key es la columna (o combinación de
    columnas) por la que se filtra con igualdad en la consulta que la tabla
    resuelve; el resto de las columnas usadas para ordenar u acotar el
    resultado son clustering keys.

??? question "¿Qué cambio en la consulta invalidaría alguna de estas tablas?"
    Cualquier filtro que no coincida con la partition key definida. Por
    ejemplo, `partidos_por_torneo` (partition key `torneo_id`) no puede
    responder eficientemente "partidos de un equipo" — para eso existe
    `partidos_por_equipo`. Del mismo modo, si se necesitara "partidos de un
    equipo en un rango de fechas que cruza varias temporadas",
    `partidos_por_equipo_temporada` (partition key `(equipo_id, temporada)`)
    ya no alcanzaría, porque el rango caería en particiones distintas; ahí
    conviene volver a `partidos_por_equipo`, que sí tiene todo el historial
    del equipo en una sola partición. En Cassandra cada nuevo patrón de
    consulta que no calce con una partition key existente requiere, en
    general, una tabla desnormalizada nueva.

??? question "¿Cómo impacta esto en el tamaño de las particiones?"
    `partidos_por_equipo` acumula en una sola partición **todo** el
    historial del equipo, que puede crecer sin límite a lo largo de los
    años (partición "ancha"). `partidos_por_equipo_temporada` evita ese
    problema acotando la partición a una temporada. Algo similar aplica a
    `partidos_por_torneo`: si `torneo_id` identifica una edición puntual del
    torneo, la partición queda acotada; si en cambio identifica una
    competencia que se repite indefinidamente año tras año, la partición
    también crecería sin límite y convendría partir por
    `(torneo_id, temporada)` en su lugar.
