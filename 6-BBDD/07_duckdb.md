# Manual de uso: DuckDB

> **Categoría:** Bases de datos · **Utilidad:** Consultas analíticas sobre datos
> **Tipo:** Librería externa · Base de datos analítica embebida (sin servidor)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primeros pasos](#4-primeros-pasos)
5. [Consultar archivos directamente](#5-consultar-archivos-directamente)
6. [Integración con Pandas, Polars y Arrow](#6-integración-con-pandas-polars-y-arrow)
7. [Bases de datos persistentes y tablas](#7-bases-de-datos-persistentes-y-tablas)
8. [Parámetros y resultados](#8-parámetros-y-resultados)
9. [SQL analítico](#9-sql-analítico)
10. [Extensiones de SQL de DuckDB](#10-extensiones-de-sql-de-duckdb)
11. [Exportar datos](#11-exportar-datos)
12. [API relacional de Python](#12-api-relacional-de-python)
13. [Extensiones: Excel, HTTP, otras bases de datos](#13-extensiones-excel-http-otras-bases-de-datos)
14. [Rendimiento y configuración](#14-rendimiento-y-configuración)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**DuckDB** es una base de datos **analítica** (OLAP) que se ejecuta dentro del propio proceso de Python, sin servidor, como SQLite. Está optimizada para **consultas analíticas sobre grandes volúmenes**: agregaciones, uniones, filtros y funciones de ventana sobre millones o miles de millones de filas.

**Características destacadas:**
- Consulta directamente archivos **CSV, Parquet, JSON y Excel**, sin importarlos.
- Consulta **DataFrames de Pandas/Polars** con SQL, sin copiarlos.
- Motor columnar y vectorizado que usa todos los núcleos del procesador.
- Procesa datos **más grandes que la memoria** (usa disco cuando es necesario).
- SQL completo y muy cómodo (`GROUP BY ALL`, `EXCLUDE`, `PIVOT`, `QUALIFY`…).
- Instalación con `pip`, sin dependencias.

**DuckDB frente a otras herramientas:**

| Herramienta | Mejor para |
|---|---|
| **DuckDB** | Análisis, informes, ETL, ciencia de datos sobre archivos y DataFrames. |
| SQLite | Aplicaciones con muchas lecturas/escrituras pequeñas (transaccional). |
| PostgreSQL / MySQL | Aplicaciones multiusuario con servidor. |
| Pandas | Manipulación en memoria con API de Python. DuckDB es más rápido en agregaciones y uniones grandes y no necesita cargar todo en memoria. |

---

## 2. Instalación e importación

```bash
pip install duckdb
```

```python
import duckdb

print(duckdb.__version__)
```

También existe una consola de línea de comandos (`duckdb`) y una interfaz web local (`duckdb -ui`).

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Conexión | `duckdb.connect()` en memoria o `duckdb.connect("archivo.duckdb")` persistente. |
| Conexión por defecto | `duckdb.sql(...)` usa una conexión global en memoria. |
| Relación (`DuckDBPyRelation`) | Resultado de una consulta; se evalúa de forma diferida (*lazy*). |
| Escaneo de archivos | `FROM 'ventas.csv'` o `read_csv(...)` leen el archivo al ejecutar la consulta. |
| Escaneo de DataFrames | Un DataFrame de Python puede usarse en SQL por su nombre de variable. |
| Almacenamiento columnar | Los datos se guardan y procesan por columnas (muy eficiente para análisis). |

---

## 4. Primeros pasos

### 4.1 Consulta rápida

```python
duckdb.sql("SELECT 42 AS respuesta").show()

resultado = duckdb.sql("SELECT range AS n, n * n AS cuadrado FROM range(5)")
print(resultado)                 # Muestra una tabla formateada
```

### 4.2 Obtener los resultados

```python
rel = duckdb.sql("SELECT * FROM range(5) t(n)")

rel.fetchall()          # Lista de tuplas
rel.fetchall()[0]       # Primera fila
rel.df()                # DataFrame de Pandas
rel.pl()                # DataFrame de Polars
rel.arrow()             # Tabla de PyArrow
rel.fetchnumpy()        # Diccionario de arrays NumPy
rel.columns, rel.types  # Nombres y tipos de las columnas
```

> **DuckDB 2.0:** las relaciones (resultado de `sql()`) ya **no** tienen `fetchone()` ni `fetchmany()`, porque cada llamada vuelve a ejecutar la consulta. Use `fetchall()`, `df()`, `limit(1).fetchall()`, o `con.execute(...)` (sección 8), que sí ofrece `fetchone()` y `fetchmany()`.

### 4.3 Conexión explícita

```python
con = duckdb.connect()                       # En memoria
con = duckdb.connect("analitica.duckdb")     # Persistente en archivo
con.sql("SELECT 1").show()
con.close()

with duckdb.connect("analitica.duckdb") as con:
    ...
```

> Use una conexión explícita en scripts y aplicaciones; `duckdb.sql()` es cómodo para exploración rápida.

---

## 5. Consultar archivos directamente

### 5.1 CSV

```python
duckdb.sql("SELECT * FROM 'ventas.csv' LIMIT 5").show()

duckdb.sql("""
    SELECT region, SUM(importe) AS total
    FROM read_csv('ventas.csv', delim=';', decimal_separator=',', header=true, dateformat='%d/%m/%Y')
    GROUP BY region
    ORDER BY total DESC
""").show()
```

DuckDB **detecta automáticamente** el separador, los tipos de columna y la cabecera. Opciones útiles de `read_csv`:

| Opción | Uso |
|---|---|
| `delim` / `sep` | Separador (`';'`, `'\t'`). |
| `header` | Si la primera fila es cabecera. |
| `decimal_separator` | `','` para números en formato español. |
| `dateformat` / `timestampformat` | Formato de fechas (`'%d/%m/%Y'`). |
| `columns` | Nombres y tipos explícitos: `{'id': 'INTEGER', 'fecha': 'DATE'}`. |
| `encoding` | `'utf-8'` (por defecto), `'latin-1'`, `'utf-16'`. |
| `ignore_errors` | Omite filas con errores. |
| `skip` | Filas a saltar al inicio. |
| `filename` | Añade una columna con el nombre del archivo de origen. |
| `all_varchar` | Lee todo como texto (para revisar datos problemáticos). |

Ver cómo interpreta DuckDB un archivo:

```python
duckdb.sql("DESCRIBE SELECT * FROM 'ventas.csv'").show()
duckdb.sql("SELECT * FROM sniff_csv('ventas.csv')").show()
```

### 5.2 Varios archivos a la vez

```python
duckdb.sql("SELECT COUNT(*) FROM 'datos/ventas_2026_*.csv'")                    # Comodines
duckdb.sql("SELECT * FROM read_csv(['enero.csv', 'febrero.csv'], union_by_name=true)")
duckdb.sql("SELECT filename, COUNT(*) FROM read_csv('logs/*.csv', filename=true) GROUP BY filename")
duckdb.sql("SELECT * FROM 'datos/**/*.parquet'")                                 # Recursivo
```

`union_by_name=true` combina archivos con columnas en distinto orden o con columnas adicionales.

### 5.3 Parquet (formato recomendado para análisis)

```python
duckdb.sql("SELECT * FROM 'ventas.parquet' WHERE fecha >= DATE '2026-01-01'")
duckdb.sql("SELECT * FROM read_parquet('ventas/*.parquet', hive_partitioning=true) WHERE anio = 2026")
duckdb.sql("SELECT * FROM parquet_metadata('ventas.parquet')")
```

> Parquet es columnar y comprimido: las consultas solo leen las columnas y bloques necesarios. Convertir CSV grandes a Parquet suele acelerar las consultas entre 10 y 100 veces.

### 5.4 JSON

```python
duckdb.sql("SELECT * FROM 'pedidos.json'")                         # Lista de objetos o JSON por líneas
duckdb.sql("SELECT * FROM read_json('eventos/*.jsonl', format='newline_delimited')")
duckdb.sql("""
    SELECT cliente.nombre AS cliente, UNNEST(lineas) AS linea
    FROM 'pedidos.json'
""")
```

### 5.5 Crear una vista sobre archivos

```python
con.sql("CREATE VIEW ventas AS SELECT * FROM read_csv('datos/ventas_*.csv', union_by_name=true)")
con.sql("SELECT region, SUM(importe) FROM ventas GROUP BY region")
```

---

## 6. Integración con Pandas, Polars y Arrow

### 6.1 Consultar un DataFrame con SQL

```python
import pandas as pd

df = pd.DataFrame({
    "region": ["Norte", "Sur", "Norte", "Este", "Sur"],
    "producto": ["A", "B", "A", "C", "A"],
    "importe": [120.5, 80.0, 99.9, 300.0, 45.0],
})

resumen = duckdb.sql("""
    SELECT region, COUNT(*) AS pedidos, ROUND(SUM(importe), 2) AS total
    FROM df
    GROUP BY region
    ORDER BY total DESC
""").df()
```

> DuckDB encuentra el DataFrame por el **nombre de la variable** (`df`) y lo lee sin copiarlo.

### 6.2 Registrar DataFrames con un nombre

```python
con = duckdb.connect()
con.register("clientes", df_clientes)
con.register("pedidos", df_pedidos)

con.sql("""
    SELECT c.nombre, SUM(p.total) AS total
    FROM pedidos p JOIN clientes c USING (cliente_id)
    GROUP BY c.nombre
""").df()
```

### 6.3 Crear una tabla a partir de un DataFrame

```python
con.sql("CREATE TABLE ventas AS SELECT * FROM df")
con.sql("INSERT INTO ventas SELECT * FROM df_nuevas")
```

### 6.4 Polars y Arrow

```python
import polars as pl

df_pl = pl.read_csv("ventas.csv")
duckdb.sql("SELECT region, SUM(importe) FROM df_pl GROUP BY ALL").pl()

tabla_arrow = duckdb.sql("SELECT * FROM 'ventas.parquet'").arrow()
```

### 6.5 Combinar archivos y DataFrames en la misma consulta

```python
objetivos = pd.DataFrame({"region": ["Norte", "Sur", "Este"], "objetivo": [50000, 40000, 30000]})

duckdb.sql("""
    SELECT v.region, SUM(v.importe) AS ventas, o.objetivo,
           ROUND(100 * SUM(v.importe) / o.objetivo, 1) AS cumplimiento
    FROM 'ventas.parquet' v
    JOIN objetivos o USING (region)
    GROUP BY v.region, o.objetivo

""").df()
```

---

## 7. Bases de datos persistentes y tablas

### 7.1 Crear tablas

```python
con = duckdb.connect("tienda.duckdb")

con.sql("""
    CREATE TABLE IF NOT EXISTS productos (
        codigo    VARCHAR PRIMARY KEY,
        nombre    VARCHAR NOT NULL,
        categoria VARCHAR,
        precio    DECIMAL(10, 2) CHECK (precio >= 0),
        etiquetas VARCHAR[],
        creado    TIMESTAMP DEFAULT current_timestamp
    )
""")

con.sql("CREATE OR REPLACE TABLE ventas AS SELECT * FROM read_csv('ventas.csv')")
```

### 7.2 Tipos de datos principales

| Tipo | Descripción |
|---|---|
| `INTEGER`, `BIGINT`, `HUGEINT` | Enteros |
| `DOUBLE` | Coma flotante |
| `DECIMAL(p, s)` | Decimal exacto (importes) |
| `VARCHAR` | Texto |
| `BOOLEAN` | Verdadero/falso |
| `DATE`, `TIME`, `TIMESTAMP`, `TIMESTAMPTZ`, `INTERVAL` | Fechas y horas |
| `UUID`, `BLOB`, `JSON` | Otros |
| `TIPO[]` (LIST) | Lista de valores |
| `STRUCT(a INT, b VARCHAR)` | Estructura con campos |
| `MAP(VARCHAR, INT)` | Diccionario |
| `ENUM` | Conjunto fijo de valores |

### 7.3 Insertar, actualizar y eliminar

```python
con.execute("INSERT INTO productos (codigo, nombre, categoria, precio, etiquetas) VALUES (?, ?, ?, ?, ?)",
            ["TEC-01", "Teclado", "Periféricos", 59.90, ["gaming", "oferta"]])

con.executemany("INSERT INTO productos (codigo, nombre, precio) VALUES (?, ?, ?)",
                [("RAT-01", "Ratón", 24.5), ("CAB-HD", "Cable HDMI", 9.9)])

con.sql("""
    INSERT INTO productos (codigo, nombre, precio) VALUES ('TEC-01', 'Teclado RGB', 64.90)
    ON CONFLICT (codigo) DO UPDATE SET nombre = EXCLUDED.nombre, precio = EXCLUDED.precio
""")

con.sql("UPDATE productos SET precio = precio * 1.05 WHERE categoria = 'Periféricos'")
con.sql("DELETE FROM productos WHERE precio = 0")
```

> DuckDB está optimizado para **cargas masivas** (`INSERT ... SELECT`, `COPY`, `CREATE TABLE AS`). Insertar fila a fila en bucles es lento; agrupe los datos en un DataFrame o archivo y cárguelos de una vez.

### 7.4 Transacciones

```python
con.begin()
try:
    con.sql("DELETE FROM ventas WHERE fecha >= '2026-11-01'")
    con.sql("INSERT INTO ventas SELECT * FROM 'ventas_noviembre.csv'")
    con.commit()
except Exception:
    con.rollback()
    raise
```

### 7.5 Información del esquema

```python
con.sql("SHOW TABLES").show()
con.sql("DESCRIBE productos").show()
con.sql("SUMMARIZE ventas").show()          # Estadísticas de todas las columnas
con.sql("SELECT * FROM duckdb_tables()").show()
```

> **Concurrencia:** un archivo `.duckdb` puede abrirlo **un solo proceso** en modo escritura (o varios en solo lectura: `duckdb.connect("archivo.duckdb", read_only=True)`). No está pensado para múltiples usuarios escribiendo a la vez.

---

## 8. Parámetros y resultados

### 8.1 Consultas con parámetros

```python
con.execute("SELECT * FROM ventas WHERE region = ? AND importe > ?", ["Norte", 100]).df()
con.execute("SELECT * FROM ventas WHERE region = $region", {"region": "Sur"}).fetchall()
con.sql("SELECT * FROM ventas WHERE fecha >= $desde", params={"desde": "2026-01-01"})
```

> Use parámetros para valores procedentes del usuario. `execute` devuelve un resultado que se lee con `fetchone`, `fetchmany`, `fetchall` o `df`; `sql` devuelve una relación.

> **DuckDB 2.0:** tras `con.execute(...)`, debe **terminar de leer el resultado** (por ejemplo, con `fetchall()`) antes de ejecutar otra consulta en la misma conexión; si solo lee una parte (`fetchone()`), la siguiente operación falla con `ResourceInUseException`. Para obtener un único valor, lo más sencillo es `con.sql("...").fetchall()[0]`.

### 8.2 Leer por lotes

```python
cur = con.execute("SELECT * FROM 'enorme.parquet'")
while lote := cur.fetchmany(100_000):          # Hasta que devuelva una lista vacía
    procesar(lote)

# Por lotes de Arrow (requiere pip install pyarrow)
for lote in con.sql("SELECT * FROM 'enorme.parquet'").to_arrow_reader(100_000):
    procesar(lote.to_pandas())
```

---

## 9. SQL analítico

### 9.1 Agregaciones

```sql
SELECT region,
       COUNT(*)                       AS pedidos,
       COUNT(DISTINCT cliente_id)     AS clientes,
       SUM(importe)                   AS ventas,
       AVG(importe)                   AS ticket_medio,
       MEDIAN(importe)                AS mediana,
       QUANTILE_CONT(importe, 0.9)    AS p90,
       STDDEV(importe)                AS desviacion,
       MIN(fecha), MAX(fecha)
FROM ventas
GROUP BY region;
```

### 9.2 Agregaciones condicionales y multinivel

```sql
SELECT region,
       SUM(importe) FILTER (WHERE canal = 'web')    AS ventas_web,
       SUM(importe) FILTER (WHERE canal = 'tienda') AS ventas_tienda,
       COUNT(*) FILTER (WHERE importe > 500)        AS pedidos_grandes
FROM ventas
GROUP BY region;

-- Subtotales por región y total general
SELECT region, categoria, SUM(importe) AS ventas
FROM ventas
GROUP BY ROLLUP (region, categoria)
ORDER BY region NULLS LAST, categoria NULLS LAST;
```

### 9.3 Fechas

```sql
SELECT date_trunc('month', fecha)              AS mes,
       strftime(fecha, '%Y-%m')                AS mes_texto,
       year(fecha), month(fecha), dayofweek(fecha), week(fecha),
       fecha + INTERVAL 30 DAY                 AS vencimiento,
       date_diff('day', fecha, current_date)   AS antiguedad_dias
FROM ventas;

-- Calendario completo (para rellenar días sin ventas)
SELECT CAST(d AS DATE) AS dia FROM generate_series(DATE '2026-01-01', DATE '2026-12-31', INTERVAL 1 DAY) t(d);
```

### 9.4 Funciones de ventana

```sql
SELECT fecha, region, ventas,
       SUM(ventas) OVER (PARTITION BY region ORDER BY fecha)                         AS acumulado,
       AVG(ventas) OVER (PARTITION BY region ORDER BY fecha ROWS 6 PRECEDING)        AS media_movil_7d,
       ventas - LAG(ventas) OVER (PARTITION BY region ORDER BY fecha)                AS diferencia,
       RANK() OVER (PARTITION BY region ORDER BY ventas DESC)                        AS puesto,
       ROUND(100.0 * ventas / SUM(ventas) OVER (PARTITION BY region), 2)             AS pct_region
FROM ventas_diarias;
```

### 9.5 CTE y subconsultas

```sql
WITH mensual AS (
    SELECT date_trunc('month', fecha) AS mes, SUM(importe) AS ventas
    FROM ventas GROUP BY 1
)
SELECT mes, ventas,
       ROUND(100 * (ventas / LAG(ventas) OVER (ORDER BY mes) - 1), 1) AS variacion_pct
FROM mensual
ORDER BY mes;
```

---

## 10. Extensiones de SQL de DuckDB

DuckDB añade sintaxis que simplifica mucho el SQL analítico.

### 10.1 `FROM` primero y `SELECT` implícito

```sql
FROM 'ventas.csv' LIMIT 10;                       -- Equivale a SELECT * FROM ... LIMIT 10
FROM ventas SELECT region, importe WHERE importe > 100;
```

### 10.2 `GROUP BY ALL` y `ORDER BY ALL`

```sql
SELECT region, categoria, SUM(importe) AS total
FROM ventas
GROUP BY ALL            -- Agrupa por todas las columnas que no son agregaciones
ORDER BY ALL;
```

### 10.3 `EXCLUDE`, `REPLACE` y `COLUMNS`

```sql
SELECT * EXCLUDE (dni, telefono) FROM clientes;
SELECT * REPLACE (ROUND(importe, 2) AS importe) FROM ventas;
SELECT MAX(COLUMNS('importe_.*')) FROM ventas;          -- Aplicar a todas las columnas que coinciden
SELECT COLUMNS(lambda c: c LIKE 'fecha%') FROM ventas;   -- Sintaxis lambda actual (la flecha -> está obsoleta)
```

### 10.4 `QUALIFY`: filtrar por funciones de ventana

```sql
-- El pedido más grande de cada cliente
SELECT * FROM pedidos
QUALIFY ROW_NUMBER() OVER (PARTITION BY cliente_id ORDER BY total DESC) = 1;
```

### 10.5 `PIVOT` y `UNPIVOT`

```sql
-- Ventas por región (filas) y mes (columnas)
PIVOT ventas
ON strftime(fecha, '%Y-%m')
USING SUM(importe)
GROUP BY region;

-- De columnas a filas
UNPIVOT ventas_mensuales
ON COLUMNS(* EXCLUDE region)
INTO NAME mes VALUE ventas;
```

### 10.6 Otras utilidades

```sql
SELECT DISTINCT ON (cliente_id) * FROM pedidos ORDER BY cliente_id, fecha DESC;   -- Último por cliente
SELECT * FROM ventas USING SAMPLE 10%;                                            -- Muestra aleatoria
SELECT nombre ILIKE '%lópez%', regexp_extract(email, '@(.+)$', 1) AS dominio FROM clientes;
SELECT list(producto) AS productos, string_agg(producto, ', ') FROM lineas GROUP BY pedido_id;
SELECT etiqueta, COUNT(*)                                                          -- Expandir listas
FROM (SELECT UNNEST(etiquetas) AS etiqueta FROM productos) GROUP BY ALL;          -- (UNNEST no puede ir en el GROUP BY)
SELECT {'nombre': 'Ana', 'edad': 30} AS persona;                                  -- STRUCT literal
SELECT * FROM ventas ASOF JOIN tipos_cambio USING (moneda, fecha);                 -- Unión por fecha más próxima
```

---

## 11. Exportar datos

```python
con.sql("COPY (SELECT * FROM ventas WHERE region = 'Norte') TO 'norte.csv' (HEADER, DELIMITER ';')")
con.sql("COPY ventas TO 'ventas.parquet' (FORMAT parquet, COMPRESSION zstd)")
con.sql("COPY ventas TO 'ventas_particionado' (FORMAT parquet, PARTITION_BY (anio, mes), OVERWRITE_OR_IGNORE)")
con.sql("COPY ventas TO 'ventas.json' (FORMAT json, ARRAY true)")

con.sql("EXPORT DATABASE 'copia_bd' (FORMAT parquet)")      # Exporta todas las tablas y el esquema
con.sql("IMPORT DATABASE 'copia_bd'")
```

Desde la API de relaciones:

```python
rel = con.sql("SELECT * FROM ventas")
rel.write_csv("ventas.csv")
rel.write_parquet("ventas.parquet")
rel.to_table("ventas_copia")
```

Exportar a Excel (extensión `excel`, ver sección 13) o a través de Pandas:

```python
con.sql("SELECT * FROM resumen").df().to_excel("resumen.xlsx", index=False)
```

---

## 12. API relacional de Python

Permite construir consultas encadenando métodos (evaluación diferida):

```python
rel = duckdb.read_csv("ventas.csv")              # También read_parquet, read_json, from_df(df)

resultado = (
    rel.filter("importe > 50")
       .aggregate("region, SUM(importe) AS total, COUNT(*) AS pedidos", "region")
       .order("total DESC")
       .limit(5)
)
resultado.show()
df = resultado.df()

rel.project("region, importe * 1.21 AS importe_iva")
rel.join(duckdb.read_csv("regiones.csv"), "region")
rel.describe().show()                            # Estadísticas
rel.columns, rel.types, rel.shape
```

Las relaciones pueden mezclarse con SQL:

```python
grandes = rel.filter("importe > 500")
duckdb.sql("SELECT region, COUNT(*) FROM grandes GROUP BY ALL")
```

### 12.1 Funciones de Python en SQL

```python
import unicodedata

def sin_tildes(texto: str) -> str:
    return unicodedata.normalize("NFKD", texto).encode("ascii", "ignore").decode()

con.create_function("sin_tildes", sin_tildes, ["VARCHAR"], "VARCHAR")
con.sql("SELECT sin_tildes('Ñandú López')").fetchall()     # [('Nandu Lopez',)]
```

> Los tipos pueden indicarse como texto (`"VARCHAR"`) o con objetos de tipo: `from duckdb.sqltypes import VARCHAR` en DuckDB 2.0 (en versiones 1.x el módulo se llamaba `duckdb.typing`).

> Las funciones de Python son más lentas que las nativas; úselas solo cuando no exista una función SQL equivalente (DuckDB incluye `strip_accents()`).

---

## 13. Extensiones: Excel, HTTP, otras bases de datos

DuckDB se amplía con extensiones que se instalan y cargan bajo demanda:

```python
con.install_extension("httpfs");  con.load_extension("httpfs")
# Equivalente en SQL: INSTALL httpfs; LOAD httpfs;
```

| Extensión | Uso |
|---|---|
| `httpfs` | Leer archivos por HTTP(S) y de S3, Azure, Google Cloud. |
| `excel` | Leer y escribir `.xlsx` (`read_xlsx`, `COPY ... TO 'x.xlsx'`). |
| `spatial` | Datos geográficos (GeoJSON, Shapefile, funciones `ST_*`). |
| `postgres` / `mysql` / `sqlite` | Consultar estas bases de datos directamente. |
| `json`, `parquet` | Incluidas por defecto. |
| `fts` | Búsqueda de texto completo. |
| `delta`, `iceberg` | Formatos de *data lake*. |

### 13.1 Archivos remotos

```python
con.sql("SELECT * FROM 'https://ejemplo.com/datos/ventas.parquet' LIMIT 10")
```

### 13.2 Excel

```python
con.sql("INSTALL excel; LOAD excel;")
con.sql("SELECT * FROM read_xlsx('presupuesto.xlsx', sheet = 'Hoja1')")
con.sql("COPY (SELECT * FROM resumen) TO 'resumen.xlsx' WITH (FORMAT xlsx, HEADER true)")
```

### 13.3 Consultar otras bases de datos

```python
con.sql("INSTALL postgres; LOAD postgres;")
con.sql("ATTACH 'dbname=tienda user=app password=secreto host=localhost' AS pg (TYPE postgres, READ_ONLY)")
con.sql("SELECT region, SUM(total) FROM pg.public.pedidos GROUP BY ALL")

con.sql("ATTACH 'empresa.db' AS lite (TYPE sqlite)")                 # SQLite
con.sql("CREATE TABLE historico AS SELECT * FROM lite.pedidos")       # Copiar a DuckDB
```

> Muy útil para hacer análisis pesados sobre datos de la base de datos de producción sin cargarla, o para combinar datos de varias fuentes en una sola consulta.

---

## 14. Rendimiento y configuración

```python
con.sql("SET threads = 8")                       # Núcleos a usar (por defecto, todos)
con.sql("SET memory_limit = '4GB'")              # Límite de memoria
con.sql("SET temp_directory = '/tmp/duckdb'")    # Disco para datos que no caben en memoria
con.sql("SET preserve_insertion_order = false")  # Menos memoria en cargas grandes
con.sql("SELECT * FROM duckdb_settings() WHERE name LIKE '%memory%'")
```

Analizar una consulta:

```python
con.sql("EXPLAIN ANALYZE SELECT region, SUM(importe) FROM 'ventas.parquet' GROUP BY ALL").show()
```

Recomendaciones:
- Convierta CSV grandes a **Parquet** y consulte Parquet.
- Seleccione solo las columnas necesarias (evite `SELECT *` en archivos grandes).
- Filtre lo antes posible; DuckDB aplica los filtros al leer Parquet (*predicate pushdown*).
- Cargue datos con operaciones masivas, no fila a fila.
- Para datos muy grandes, particione los Parquet por fecha u otra columna frecuente en filtros.

Ejemplo de diferencia de rendimiento:

```python
import time, numpy as np, pandas as pd

n = 10_000_000
df = pd.DataFrame({"grupo": np.random.randint(0, 1000, n), "valor": np.random.rand(n)})

t = time.perf_counter(); df.groupby("grupo")["valor"].agg(["sum", "mean", "count"]); t_pd = time.perf_counter() - t
t = time.perf_counter(); duckdb.sql("SELECT grupo, SUM(valor), AVG(valor), COUNT(*) FROM df GROUP BY grupo").df(); t_dd = time.perf_counter() - t
print(f"Pandas: {t_pd:.2f} s · DuckDB: {t_dd:.2f} s")
```

---

## 15. Ejemplo práctico completo

Análisis de ventas de un año a partir de **varios CSV mensuales** (con formato español), un **DataFrame** de objetivos y un CSV de productos: validación de datos, conversión a Parquet, base de datos persistente, KPIs, tablas dinámicas, rankings con `QUALIFY`, medias móviles, análisis de clientes (RFM simplificado) y exportación a Excel.

```python
"""
Análisis de ventas con DuckDB.
Requisitos: pip install duckdb pandas openpyxl numpy
"""
import random
from datetime import date, timedelta
from pathlib import Path

import duckdb
import pandas as pd

RAIZ = Path(__file__).resolve().parent
DATOS = RAIZ / "datos_ventas"
BD = RAIZ / "analitica.duckdb"
REGIONES = ["Norte", "Sur", "Este", "Oeste", "Centro"]
CANALES = ["web", "tienda", "telefono"]


# ------------------------------------------------------------ Datos de ejemplo
def generar_datos() -> None:
    DATOS.mkdir(exist_ok=True)
    random.seed(2026)
    productos = [
        ("TEC-01", "Teclado mecánico", "Periféricos", 59.90), ("RAT-01", "Ratón inalámbrico", "Periféricos", 24.50),
        ("MON-27", "Monitor 27\"", "Pantallas", 219.00), ("POR-14", "Portátil 14\"", "Equipos", 899.00),
        ("AUR-BT", "Auriculares BT", "Audio", 79.00), ("CAB-HD", "Cable HDMI", "Accesorios", 9.90),
        ("WEB-HD", "Webcam HD", "Periféricos", 49.00), ("ALT-BT", "Altavoz BT", "Audio", 39.90),
    ]
    pd.DataFrame(productos, columns=["codigo", "nombre", "categoria", "precio"]).to_csv(
        DATOS / "productos.csv", index=False)

    pedido = 1
    for mes in range(1, 12):
        filas = []
        inicio = date(2026, mes, 1)
        dias = ((inicio.replace(month=mes % 12 + 1) if mes < 12 else date(2027, 1, 1)) - inicio).days
        for _ in range(random.randint(700, 1100)):
            codigo, _, _, precio = random.choices(productos, weights=[6, 8, 3, 1, 4, 9, 3, 4])[0]
            uds = random.choice([1, 1, 1, 2, 2, 3, 5])
            fecha = inicio + timedelta(days=random.randrange(dias))
            importe = f"{uds * precio * random.choice([1, 1, 1, 0.9]):.2f}".replace(".", ",")
            if random.random() < 0.004:
                importe = "N/D"                                   # Datos erróneos intencionados
            filas.append([pedido, fecha.strftime("%d/%m/%Y"), f"C{random.randint(1, 400):04d}",
                          random.choice(REGIONES), random.choice(CANALES), codigo, uds, importe])
            pedido += 1
        pd.DataFrame(filas, columns=["pedido", "fecha", "cliente", "region", "canal", "producto",
                                     "unidades", "importe"]).to_csv(
            DATOS / f"ventas_2026_{mes:02d}.csv", sep=";", index=False)


# ------------------------------------------------------------ Carga y validación
def cargar(con: duckdb.DuckDBPyConnection) -> None:
    patron = str(DATOS / "ventas_2026_*.csv")

    errores = con.sql(f"""
        SELECT filename, pedido, importe
        FROM read_csv('{patron}', delim=';', all_varchar=true, filename=true)
        WHERE TRY_CAST(replace(importe, ',', '.') AS DECIMAL(12,2)) IS NULL
    """).df()
    if not errores.empty:
        print(f"⚠ {len(errores)} filas con importe no válido (se excluyen):")
        print(errores.assign(filename=errores["filename"].map(lambda f: Path(f).name)).head().to_string(index=False))

    con.sql(f"""
        CREATE OR REPLACE TABLE ventas AS
        SELECT pedido::INTEGER                                            AS pedido,
               strptime(fecha, '%d/%m/%Y')::DATE                          AS fecha,
               cliente, region, canal, producto,
               unidades::INTEGER                                          AS unidades,
               CAST(replace(importe, ',', '.') AS DECIMAL(12,2))          AS importe
        FROM read_csv('{patron}', delim=';', all_varchar=true)
        WHERE TRY_CAST(replace(importe, ',', '.') AS DECIMAL(12,2)) IS NOT NULL
    """)
    con.sql(f"CREATE OR REPLACE TABLE productos AS SELECT * FROM read_csv('{DATOS / 'productos.csv'}')")
    con.sql(f"COPY ventas TO '{DATOS / 'ventas_2026.parquet'}' (FORMAT parquet, COMPRESSION zstd)")

    n, desde, hasta = con.sql("SELECT COUNT(*), MIN(fecha), MAX(fecha) FROM ventas").fetchall()[0]
    print(f"✔ {n:,} ventas cargadas ({desde} → {hasta})".replace(",", "."))


# ------------------------------------------------------------ Análisis
def kpis(con) -> pd.DataFrame:
    return con.sql("""
        SELECT COUNT(DISTINCT pedido)                     AS pedidos,
               COUNT(DISTINCT cliente)                    AS clientes,
               SUM(unidades)                              AS unidades,
               ROUND(SUM(importe), 2)                     AS facturacion,
               ROUND(AVG(importe), 2)                     AS ticket_medio,
               ROUND(SUM(importe) FILTER (WHERE canal = 'web') / SUM(importe) * 100, 1) AS pct_web
        FROM ventas
    """).df()


def evolucion_mensual(con) -> pd.DataFrame:
    return con.sql("""
        WITH m AS (
            SELECT date_trunc('month', fecha) AS mes, SUM(importe) AS ventas, COUNT(*) AS pedidos
            FROM ventas GROUP BY ALL
        )
        SELECT strftime(mes, '%Y-%m') AS mes, pedidos, ROUND(ventas, 2) AS ventas,
               ROUND(100 * (ventas / LAG(ventas) OVER (ORDER BY mes) - 1), 1) AS variacion_pct,
               ROUND(SUM(ventas) OVER (ORDER BY mes), 2) AS acumulado
        FROM m ORDER BY m.mes
    """).df()


def pivot_region_mes(con) -> pd.DataFrame:
    return con.sql("""
        PIVOT (SELECT region, strftime(fecha, '%m') AS mes, importe FROM ventas)
        ON mes USING ROUND(SUM(importe), 0)
        GROUP BY region ORDER BY region
    """).df()


def top_producto_por_region(con) -> pd.DataFrame:
    return con.sql("""
        SELECT v.region, p.nombre AS producto, SUM(v.unidades) AS unidades, ROUND(SUM(v.importe), 2) AS ventas
        FROM ventas v JOIN productos p ON p.codigo = v.producto
        GROUP BY v.region, p.nombre                    -- QUALIFY no admite GROUP BY ALL
        QUALIFY ROW_NUMBER() OVER (PARTITION BY v.region ORDER BY SUM(v.importe) DESC) = 1
        ORDER BY ventas DESC
    """).df()


def cumplimiento_objetivos(con, objetivos: pd.DataFrame) -> pd.DataFrame:
    return con.sql("""
        SELECT v.region, ROUND(SUM(v.importe), 0) AS ventas, o.objetivo,
               ROUND(100 * SUM(v.importe) / o.objetivo, 1) AS cumplimiento_pct
        FROM ventas v JOIN objetivos o USING (region)
        GROUP BY v.region, o.objetivo ORDER BY cumplimiento_pct DESC
    """).df()


def media_movil(con) -> pd.DataFrame:
    return con.sql("""
        WITH diario AS (
            SELECT d::DATE AS dia, COALESCE(SUM(v.importe), 0) AS ventas
            FROM generate_series(DATE '2026-01-01', DATE '2026-11-30', INTERVAL 1 DAY) t(d)
            LEFT JOIN ventas v ON v.fecha = t.d
            GROUP BY ALL
        )
        SELECT dia, ROUND(ventas, 2) AS ventas,
               ROUND(AVG(ventas) OVER (ORDER BY dia ROWS 6 PRECEDING), 2) AS media_7d
        FROM diario ORDER BY dia
    """).df()


def segmentacion_clientes(con) -> pd.DataFrame:
    return con.sql("""
        WITH rfm AS (
            SELECT cliente,
                   date_diff('day', MAX(fecha), DATE '2026-11-30') AS recencia,
                   COUNT(DISTINCT pedido)                          AS frecuencia,
                   SUM(importe)                                    AS valor
            FROM ventas GROUP BY cliente
        ),
        puntuado AS (
            SELECT *, NTILE(3) OVER (ORDER BY recencia DESC) AS r,
                      NTILE(3) OVER (ORDER BY frecuencia)     AS f,
                      NTILE(3) OVER (ORDER BY valor)          AS m
            FROM rfm
        )
        SELECT CASE WHEN r = 3 AND f = 3 AND m = 3 THEN 'Campeones'
                    WHEN r >= 2 AND f >= 2           THEN 'Leales'
                    WHEN r = 1 AND f >= 2            THEN 'En riesgo'
                    WHEN r = 3 AND f = 1             THEN 'Nuevos'
                    ELSE 'Ocasionales' END            AS segmento,
               COUNT(*) AS clientes, ROUND(AVG(valor), 2) AS valor_medio, ROUND(SUM(valor), 2) AS valor_total
        FROM puntuado GROUP BY ALL ORDER BY valor_total DESC
    """).df()


# ------------------------------------------------------------ Principal
def main() -> None:
    if not DATOS.exists():
        generar_datos()

    objetivos = pd.DataFrame({"region": REGIONES, "objetivo": [320_000, 330_000, 310_000, 325_000, 335_000]})

    with duckdb.connect(str(BD)) as con:
        con.register("objetivos", objetivos)
        cargar(con)

        resultados = {
            "KPIs": kpis(con),
            "Evolución mensual": evolucion_mensual(con),
            "Región x mes": pivot_region_mes(con),
            "Top producto región": top_producto_por_region(con),
            "Objetivos": cumplimiento_objetivos(con, objetivos),
            "Media móvil 7 días": media_movil(con),
            "Segmentos clientes": segmentacion_clientes(con),
        }

        consulta_parquet = con.sql(f"""
            SELECT canal, ROUND(SUM(importe), 2) AS ventas
            FROM '{DATOS / "ventas_2026.parquet"}' GROUP BY ALL ORDER BY ventas DESC
        """).df()
        resultados["Canales (Parquet)"] = consulta_parquet

    pd.set_option("display.width", 140)
    for nombre in ("KPIs", "Evolución mensual", "Top producto región", "Objetivos", "Segmentos clientes"):
        print(f"\n=== {nombre} ===")
        print(resultados[nombre].to_string(index=False))

    salida = RAIZ / "informe_ventas_2026.xlsx"
    with pd.ExcelWriter(salida) as excel:
        for nombre, df in resultados.items():
            df.to_excel(excel, sheet_name=nombre[:31], index=False)
    print(f"\nInforme exportado: {salida.name}")


if __name__ == "__main__":
    main()
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `IOException: Could not set lock on file` | El archivo `.duckdb` está abierto por otro proceso. | Cierre la otra conexión o abra en `read_only=True`. |
| `ConversionException: Could not convert string 'N/D' to DOUBLE` | Datos no válidos en el archivo. | Use `TRY_CAST`, `ignore_errors=true` o `all_varchar=true` para inspeccionar. |
| Números con coma decimal leídos como texto | Formato español. | `decimal_separator=','` o `replace(col, ',', '.')`. |
| Fechas `dd/mm/aaaa` mal interpretadas | Formato no detectado. | `dateformat='%d/%m/%Y'` o `strptime(col, '%d/%m/%Y')`. |
| `CatalogException: Table with name df does not exist` | La variable del DataFrame no está en el ámbito o tiene otro nombre. | Use `con.register("nombre", df)`. |
| `BinderException: column must appear in the GROUP BY clause` | Columna no agregada ni agrupada. | Añádala al `GROUP BY` o use `GROUP BY ALL`. |
| `Cannot mix aggregates with non-aggregated columns` con `GROUP BY ALL` | Una expresión combina un agregado y una columna sin agregar (`SUM(x) / objetivo`). | Enumere las columnas en `GROUP BY` explícitamente. |
| `OutOfMemoryException` | Consulta muy grande. | `SET memory_limit`, `SET temp_directory`, seleccione menos columnas. |
| Inserciones fila a fila muy lentas | DuckDB está optimizado para cargas masivas. | Cargue desde DataFrame o archivo con `INSERT ... SELECT`. |
| `Extension "httpfs" not found` | Extensión no instalada o sin acceso a Internet. | `INSTALL httpfs; LOAD httpfs;` (requiere conexión la primera vez). |
| Resultados distintos al combinar CSV con columnas diferentes | Columnas en distinto orden. | `union_by_name=true`. |
| `AttributeError: Relations no longer have fetchone()` | DuckDB 2.0: las relaciones no tienen `fetchone`/`fetchmany`. | Use `fetchall()[0]`, `limit(1)` o `con.execute(...)`. |
| `ResourceInUseException: connection has an unfinished result from execute()` | DuckDB 2.0: resultado de `execute()` leído solo en parte. | Lea todo el resultado o use `con.sql(...).fetchall()`. |
| `ModuleNotFoundError: No module named 'duckdb.typing'` | En DuckDB 2.0 el módulo se llama `duckdb.sqltypes`. | Use `duckdb.sqltypes` o tipos como texto (`"VARCHAR"`). |
| `Deprecated lambda arrow (->) detected` | Sintaxis antigua de lambdas. | Use `lambda x: ...`. |
| `Combining QUALIFY with GROUP BY ALL is not supported yet` | Limitación actual. | Enumere las columnas en `GROUP BY`. |
| Columna numérica leída como `VARCHAR` | Algún valor no numérico en el archivo (p. ej. «N/D»). | `TRY_CAST` al cargar o `columns={...}` con tipos explícitos. |

---

## 17. Buenas prácticas

- Use DuckDB para **análisis y ETL**; para aplicaciones transaccionales con muchos usuarios, use PostgreSQL/MySQL.
- Consulte los archivos directamente para explorar; convierta a **Parquet** los datos que vaya a analizar repetidamente.
- Valide los datos de entrada con `TRY_CAST` y `all_varchar` antes de cargarlos.
- Use parámetros (`?`, `$nombre`) para valores procedentes del usuario.
- Aproveche la sintaxis de DuckDB (`GROUP BY ALL`, `QUALIFY`, `PIVOT`, `EXCLUDE`) para un SQL más claro.
- Cargue datos con operaciones masivas, nunca en bucles fila a fila.
- Use un archivo `.duckdb` por proceso de escritura; abra en solo lectura para consultas paralelas.
- Combine DuckDB con Pandas/Polars: SQL para las agregaciones pesadas y DataFrames para la presentación.
- Revise el plan con `EXPLAIN ANALYZE` en consultas lentas.

---

## 18. Referencia rápida

```python
duckdb.sql("SELECT ...").show() | .df() | .pl() | .arrow() | .fetchall()
con.execute(sql, params).fetchone() | .fetchmany(n) | .fetchall()     # Leer el resultado completo (2.0)
con = duckdb.connect("bd.duckdb", read_only=False); con.close()
con.sql(sql, params={...}) / con.execute(sql, [p1, p2]).fetchall() / con.executemany(sql, filas)
con.register("nombre", df) / con.begin() / con.commit() / con.rollback()

FROM 'archivo.csv' | 'archivo.parquet' | 'archivo.json' | 'carpeta/*.csv'
read_csv('f.csv', delim=';', decimal_separator=',', dateformat='%d/%m/%Y', union_by_name=true, filename=true)
read_parquet('f/*.parquet', hive_partitioning=true) / read_json('f.jsonl')
COPY (consulta) TO 'salida.parquet' (FORMAT parquet) / TO 'salida.csv' (HEADER, DELIMITER ';')
CREATE TABLE t AS SELECT ... / INSERT INTO t SELECT ... / DESCRIBE t / SUMMARIZE t

GROUP BY ALL · ORDER BY ALL · SELECT * EXCLUDE (...) · QUALIFY ... · PIVOT ... ON ... USING ...
FILTER (WHERE ...) · TRY_CAST · strptime · strftime · date_trunc · generate_series · UNNEST · list()
INSTALL ext; LOAD ext;   (httpfs, excel, postgres, mysql, sqlite, spatial)
SET threads = n; SET memory_limit = '4GB'; EXPLAIN ANALYZE ...
```

---

## 19. Ejercicios propuestos

1. Consulte con SQL una carpeta con 12 CSV mensuales sin importarlos y obtenga las ventas por mes y región.
2. Convierta un CSV de varios millones de filas a Parquet y compare el tiempo de una misma consulta sobre ambos formatos.
3. Cree una tabla dinámica (`PIVOT`) con productos en filas y meses en columnas, y expórtela a Excel.
4. Combine en una sola consulta un DataFrame de Pandas, un archivo Parquet y una tabla de SQLite adjuntada con `ATTACH`.
5. Calcule para cada cliente su primer pedido, último pedido, número de pedidos y días entre compras usando funciones de ventana.

---

## 20. Referencias

- Documentación oficial: https://duckdb.org/docs/
- API de Python: https://duckdb.org/docs/api/python/overview
- Sintaxis SQL ampliada («Friendly SQL»): https://duckdb.org/docs/sql/dialect/friendly_sql
- Extensiones: https://duckdb.org/docs/extensions/overview
