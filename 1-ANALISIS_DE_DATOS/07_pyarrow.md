# Manual de uso: PyArrow

> **Categoría:** Ingeniería de datos · **Utilidad:** Datos columnares en memoria y formatos Parquet, Arrow/Feather y CSV
> **Tipo:** Librería de terceros (`pip install pyarrow`) · **Versión de referencia:** 15 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Tipos de datos y esquemas](#4-tipos-de-datos-y-esquemas)
5. [Arrays, tablas y lotes](#5-arrays-tablas-y-lotes)
6. [Cálculo con `pyarrow.compute`](#6-cálculo-con-pyarrowcompute)
7. [Agrupar, unir y ordenar tablas](#7-agrupar-unir-y-ordenar-tablas)
8. [Lectura y escritura de CSV](#8-lectura-y-escritura-de-csv)
9. [Parquet](#9-parquet)
10. [Metadatos y estadísticas de Parquet](#10-metadatos-y-estadísticas-de-parquet)
11. [Datasets: colecciones de archivos particionados](#11-datasets-colecciones-de-archivos-particionados)
12. [Arrow IPC y Feather](#12-arrow-ipc-y-feather)
13. [Sistemas de archivos y almacenamiento en la nube](#13-sistemas-de-archivos-y-almacenamiento-en-la-nube)
14. [Interoperabilidad: pandas, Polars, NumPy, DuckDB](#14-interoperabilidad-pandas-polars-numpy-duckdb)
15. [Evolución del esquema](#15-evolución-del-esquema)
16. [Memoria y rendimiento](#16-memoria-y-rendimiento)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**PyArrow** es la implementación para Python de **Apache Arrow**, un formato estándar de datos **columnares en memoria** compartido por pandas, Polars, DuckDB, Spark, Dask y muchas bases de datos. Incluye:

- Estructuras de datos columnares eficientes (`Array`, `Table`, `RecordBatch`) con tipos estrictos y valores nulos nativos.
- Funciones de cálculo vectorizadas en C++ (`pyarrow.compute`).
- Lectura y escritura muy rápidas de **Parquet**, **Arrow IPC/Feather**, **CSV**, JSON y ORC.
- **Datasets**: consultas sobre colecciones de archivos particionados, locales o en la nube (S3, GCS, Azure).
- Intercambio de datos entre librerías y procesos **sin copias** (*zero-copy*).

**Ventajas:**
- Rendimiento muy alto y bajo consumo de memoria.
- Estándar de intercambio: el mismo dato sirve a pandas, Polars, DuckDB, Spark…
- Tipos ricos (decimales, fechas con zona horaria, listas, estructuras, diccionarios).
- Lectura selectiva (columnas, filas y particiones) en Parquet.

**Limitaciones:**
- API de bajo nivel: no pretende sustituir a pandas o Polars en el análisis diario.
- Las tablas son **inmutables**: cada transformación crea una nueva.
- Funciones de cálculo menos ergonómicas que las de un DataFrame.

| Si necesita… | Opción |
|---|---|
| Leer/escribir Parquet, gestionar datasets particionados, intercambiar datos | **PyArrow** |
| Análisis tabular cómodo | pandas (con *backend* Arrow) o Polars |
| SQL sobre Parquet y Arrow | DuckDB |
| Procesamiento distribuido | Dask, Spark |

---

## 2. Instalación e importación

```bash
pip install pyarrow
```

```python
import pyarrow as pa
import pyarrow.compute as pc
import pyarrow.parquet as pq
import pyarrow.dataset as ds
import pyarrow.csv as pcsv
import pyarrow.feather as feather
import pyarrow.ipc as ipc
import pyarrow.fs as pafs
print(pa.__version__)
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Array` | Columna de valores del mismo tipo, contigua en memoria, con mapa de nulos. |
| `ChunkedArray` | Columna formada por varios `Array` (trozos). Las columnas de una `Table` son de este tipo. |
| `RecordBatch` | Conjunto de arrays de igual longitud (un bloque de filas). |
| `Table` | Colección de `ChunkedArray` con un `Schema`: equivalente a un DataFrame inmutable. |
| `Schema` | Nombres, tipos, nulabilidad y metadatos de las columnas. |
| `Dataset` | Vista lógica sobre muchos archivos (y particiones) que se consulta sin cargarlos enteros. |
| Formato columnar | Los valores de una columna se almacenan juntos: óptimo para analítica y compresión. |

---

## 4. Tipos de datos y esquemas

```python
esquema = pa.schema([
    pa.field("pedido_id", pa.int64(), nullable=False),
    pa.field("fecha", pa.date32()),
    pa.field("momento", pa.timestamp("ms", tz="Europe/Madrid")),
    pa.field("tienda", pa.dictionary(pa.int16(), pa.string())),      # Categórica
    pa.field("importe", pa.decimal128(12, 2)),                       # Decimal exacto
    pa.field("unidades", pa.int32()),
    pa.field("etiquetas", pa.list_(pa.string())),
    pa.field("direccion", pa.struct([("cp", pa.string()), ("ciudad", pa.string())])),
], metadata={"origen": "ERP", "version": "3"})

esquema.names ; esquema.types ; esquema.field("importe").type ; esquema.metadata
esquema.set(1, pa.field("fecha", pa.timestamp("s")))              # Nuevo esquema con un campo cambiado
```

| Tipo Arrow | Python / pandas |
|---|---|
| `int8`…`int64`, `uint8`…`uint64` | Enteros (con nulos sin pasar a `float`). |
| `float32`, `float64` | Decimales binarios. |
| `decimal128(p, s)` | `Decimal` exacto. |
| `string` / `large_string` / `string_view` | Texto (límites de 2 GB y > 2 GB por bloque / vista eficiente). |
| `bool_` | Lógico. |
| `date32`, `timestamp(unidad, tz)`, `duration`, `time64` | Fechas y tiempos. |
| `dictionary(indices, valores)` | Categórica (como `category` de pandas). |
| `list_`, `struct`, `map_` | Anidados. |
| `null` | Sin valores. |

---

## 5. Arrays, tablas y lotes

```python
precios = pa.array([19.9, None, 250.0], type=pa.float64())
precios.null_count, len(precios), precios.type, precios.to_pylist()

tabla = pa.table({"producto": ["Silla", "Mesa", "Lámpara"], "precio": precios, "stock": [10, 0, 7]})
tabla = pa.Table.from_pylist([{"a": 1, "b": "x"}, {"a": 2, "b": "y"}])
tabla = pa.Table.from_pandas(df, preserve_index=False)
tabla = pa.Table.from_batches([lote1, lote2])

tabla.num_rows, tabla.num_columns, tabla.schema, tabla.column_names, tabla.nbytes
tabla["precio"] ; tabla.column(0) ; tabla.select(["producto", "precio"]) ; tabla.slice(0, 2)
tabla.append_column("iva", pc.multiply(tabla["precio"], 0.21))
tabla.set_column(1, "precio", pc.fill_null(tabla["precio"], 0.0))
tabla.drop_columns(["stock"]) ; tabla.rename_columns(["p", "precio", "s"])
tabla.cast(nuevo_esquema)                                     # Cambiar tipos
pa.concat_tables([t1, t2], promote_options="default")         # Unir tablas (rellena columnas ausentes con nulos)
tabla.combine_chunks()                                        # Un único trozo por columna
tabla.to_pydict() ; tabla.to_pylist() ; tabla.to_batches(max_chunksize=10_000)
```

---

## 6. Cálculo con `pyarrow.compute`

```python
import pyarrow.compute as pc

pc.sum(tabla["precio"]).as_py() ; pc.mean(...) ; pc.min_max(...) ; pc.stddev(...) ; pc.quantile(x, q=[0.5, 0.95])
pc.count(x, mode="only_null") ; pc.count_distinct(x) ; pc.value_counts(x) ; pc.unique(x)
pc.add(a, b) ; pc.subtract ; pc.multiply ; pc.divide ; pc.round(x, 2) ; pc.abs ; pc.power
pc.greater(x, 100) ; pc.equal ; pc.is_in(x, value_set=pa.array(["A", "B"])) ; pc.is_null(x) ; pc.and_(m1, m2)
pc.if_else(pc.greater(x, 100), "alto", "normal") ; pc.fill_null(x, 0) ; pc.coalesce(a, b)
pc.cumulative_sum(x) ; pc.rank(x, sort_keys="descending")

# Texto
pc.utf8_upper(t) ; pc.utf8_trim_whitespace(t) ; pc.utf8_length(t) ; pc.utf8_slice_codeunits(t, 0, 3)
pc.match_substring(t, "error") ; pc.match_substring_regex(t, r"^\d{5}$") ; pc.replace_substring_regex(t, r"\s+", " ")
pc.split_pattern(t, ";") ; pc.binary_join_element_wise(a, b, " ")

# Fechas
pc.year(f) ; pc.month(f) ; pc.day_of_week(f) ; pc.iso_week(f) ; pc.strftime(f, "%d/%m/%Y")
pc.strptime(texto, format="%d/%m/%Y", unit="s", error_is_null=True)
pc.floor_temporal(momento, unit="month") ; pc.days_between(inicio, fin)

# Filtros: arrays booleanos o expresiones
tabla.filter(pc.greater(tabla["stock"], 0))
tabla.filter((pc.field("stock") > 0) & (pc.field("precio") < 100))          # Expresión (también para datasets)
```

Lista completa: `pc.list_functions()`; ayuda de una función: `help(pc.utf8_trim)`.

---

## 7. Agrupar, unir y ordenar tablas

```python
resumen = tabla.group_by(["tienda", "mes"]).aggregate([
    ("importe", "sum"), ("importe", "mean"), ("pedido_id", "count"),
    ("cliente", "count_distinct"), ("importe", "max"), ("producto", "list"),
])                                                     # Columnas resultado: importe_sum, importe_mean…

tabla.join(productos, keys="producto_id", join_type="left outer")    # "inner", "full outer", "left anti"…
tabla.join(otra, keys="id", right_keys="codigo", right_suffix="_otra")
tabla.sort_by([("tienda", "ascending"), ("importe", "descending")])
pc.top_k_unstable(tabla["importe"], k=10)
```

Para análisis complejos, convierta a Polars o DuckDB (sin copia) y vuelva a Arrow si lo necesita.

---

## 8. Lectura y escritura de CSV

El lector CSV de Arrow es multihilo y mucho más rápido que el de Python o pandas clásico:

```python
tabla = pcsv.read_csv(
    "ventas.csv",
    read_options=pcsv.ReadOptions(encoding="latin-1", skip_rows=0, block_size=1 << 24),
    parse_options=pcsv.ParseOptions(delimiter=";", quote_char='"', invalid_row_handler=lambda fila: "skip"),
    convert_options=pcsv.ConvertOptions(
        column_types={"cp": pa.string(), "importe": pa.decimal128(12, 2)},
        decimal_point=",",
        null_values=["", "NA", "-"],
        timestamp_parsers=["%d/%m/%Y", "%d/%m/%Y %H:%M"],
        include_columns=["fecha", "tienda", "importe"],
        strings_can_be_null=True,
    ),
)

lector = pcsv.open_csv("enorme.csv")                 # Lectura en streaming por lotes
for lote in lector:
    procesar(lote)

pcsv.write_csv(tabla, "salida.csv", write_options=pcsv.WriteOptions(delimiter=";", include_header=True))
```

---

## 9. Parquet

**Parquet** es el formato de almacenamiento columnar de referencia: comprimido, tipado, con estadísticas por bloque que permiten saltar datos que no cumplen un filtro.

```python
pq.write_table(tabla, "ventas.parquet", compression="zstd", compression_level=3,
               row_group_size=250_000, use_dictionary=["tienda", "producto"], write_statistics=True)

tabla = pq.read_table("ventas.parquet", columns=["fecha", "importe"],
                      filters=[("tienda", "in", ["Madrid", "Bilbao"]), ("importe", ">", 0)])
pq.read_schema("ventas.parquet")

archivo = pq.ParquetFile("enorme.parquet")
for lote in archivo.iter_batches(batch_size=100_000, columns=["importe"]):      # Streaming
    ...
archivo.read_row_group(0)

with pq.ParquetWriter("incremental.parquet", esquema, compression="zstd") as escritor:
    for lote in generar_lotes():
        escritor.write_table(lote)                    # Escribir por partes sin cargar todo
```

| Compresión | Características |
|---|---|
| `snappy` (defecto) | Muy rápida, compresión moderada. |
| `zstd` | Muy buena relación compresión/velocidad (recomendada). |
| `gzip`, `brotli` | Mayor compresión, más lentas. |
| `none` | Sin compresión. |

> Tamaño de grupo de filas (*row group*) recomendado: 100.000–1.000.000 filas o 128–512 MB. Grupos muy pequeños empeoran la compresión y la lectura.

---

## 10. Metadatos y estadísticas de Parquet

```python
meta = pq.ParquetFile("ventas.parquet").metadata
meta.num_rows, meta.num_row_groups, meta.created_by, meta.serialized_size
grupo = meta.row_group(0)
columna = grupo.column(3)
columna.path_in_schema, columna.compression, columna.total_compressed_size, columna.total_uncompressed_size
columna.statistics.min, columna.statistics.max, columna.statistics.null_count   # Permiten saltar el grupo con filtros

tabla = tabla.replace_schema_metadata({"generado_por": "proceso_nocturno", "fecha": "2026-11-25"})
pq.read_schema("ventas.parquet").metadata                                       # Metadatos propios (bytes)
```

Ordenar los datos por la columna que más se filtra (fecha, tienda) antes de escribir hace que las estadísticas min/max sean muy selectivas.

---

## 11. Datasets: colecciones de archivos particionados

```python
# Escribir un dataset particionado al estilo Hive: anio=2026/mes=11/part-0.parquet
ds.write_dataset(tabla, "lago/ventas", format="parquet",
                 partitioning=["anio", "mes"], partitioning_flavor="hive",
                 existing_data_behavior="overwrite_or_ignore",       # Añadir sin borrar lo existente
                 basename_template="lote-20261125-{i}.parquet",
                 file_options=ds.ParquetFileFormat().make_write_options(compression="zstd"),
                 max_rows_per_file=1_000_000,
                 preserve_order=True)                                # Conservar el orden de las filas (escritura multihilo)

dataset = ds.dataset("lago/ventas", format="parquet", partitioning="hive")
dataset.schema ; dataset.files ; dataset.count_rows(filter=pc.field("mes") == 11)
tabla = dataset.to_table(columns=["fecha", "tienda", "importe"],
                         filter=(pc.field("anio") == 2026) & (pc.field("tienda") == "Bilbao"))
for lote in dataset.to_batches(columns=["importe"], batch_size=500_000):       # Streaming
    ...
dataset.head(10)
fragmentos = list(dataset.get_fragments(filter=pc.field("mes") == 11))         # Archivos que se leerían
```

| `existing_data_behavior` | Efecto |
|---|---|
| `"error"` (defecto) | Falla si el directorio tiene datos. |
| `"overwrite_or_ignore"` | Escribe archivos nuevos; sobrescribe solo los de igual nombre. |
| `"delete_matching"` | Borra las particiones que se van a escribir y las reemplaza. |

Los filtros sobre columnas de partición **descartan directorios completos** sin abrirlos (poda de particiones); los filtros sobre otras columnas usan las estadísticas de cada grupo de filas.

---

## 12. Arrow IPC y Feather

El formato **Arrow IPC** (archivos `.arrow`/`.feather`) guarda la representación de memoria tal cual: lectura casi instantánea, incluso con *memory mapping*.

```python
feather.write_feather(tabla, "cache.arrow", compression="lz4")
tabla = feather.read_table("cache.arrow", memory_map=True)

# Flujo (stream) para enviar datos entre procesos o por red
sink = pa.BufferOutputStream()
with ipc.new_stream(sink, tabla.schema) as escritor:
    escritor.write_table(tabla)
buffer = sink.getvalue()                          # Bytes listos para enviar
recibida = ipc.open_stream(buffer).read_all()
```

| Formato | Uso recomendado |
|---|---|
| Parquet | Almacenamiento a largo plazo, data lakes, intercambio entre sistemas. |
| Arrow IPC / Feather | Cachés locales, intercambio rápido entre procesos o lenguajes. |
| CSV | Intercambio con herramientas que no admiten otros formatos. |

---

## 13. Sistemas de archivos y almacenamiento en la nube

```python
local = pafs.LocalFileSystem()
local.get_file_info(pafs.FileSelector("lago", recursive=True))

s3 = pafs.S3FileSystem(region="eu-west-1")                   # Credenciales del entorno o argumentos
dataset = ds.dataset("mi-bucket/ventas/", filesystem=s3, format="parquet", partitioning="hive")
pq.write_table(tabla, "mi-bucket/salida/ventas.parquet", filesystem=s3)

gcs = pafs.GcsFileSystem()
fs, ruta = pafs.FileSystem.from_uri("s3://mi-bucket/ventas")  # Desde una URI
```

---

## 14. Interoperabilidad: pandas, Polars, NumPy, DuckDB

```python
import pandas as pd
df = tabla.to_pandas()                                   # Tipos NumPy clásicos
df = tabla.to_pandas(types_mapper=pd.ArrowDtype)         # pandas con backend Arrow (sin pérdida de tipos)
df = pd.read_parquet("ventas.parquet", dtype_backend="pyarrow")
tabla = pa.Table.from_pandas(df, preserve_index=False)

import polars as pl
pl_df = pl.from_arrow(tabla) ; tabla = pl_df.to_arrow()   # Sin copia

import duckdb
duckdb.sql("SELECT tienda, SUM(importe) FROM tabla GROUP BY tienda").arrow()   # DuckDB consulta la tabla Arrow directamente

np_array = tabla["precio"].to_numpy()                    # Sin copia si no hay nulos
```

---

## 15. Evolución del esquema

Los archivos de distintas épocas pueden tener esquemas diferentes (columnas nuevas, tipos cambiados):

```python
esquema_objetivo = pa.schema([...])                                # Esquema «oficial» actual
dataset = ds.dataset("lago/ventas", schema=esquema_objetivo, partitioning="hive")   # Columnas ausentes → nulos

esquemas = [pq.read_schema(f) for f in archivos]
unificado = pa.unify_schemas(esquemas, promote_options="permissive")   # int32 + int64 → int64, etc.
tabla = tabla.cast(unificado) ; pa.concat_tables(tablas, promote_options="permissive")
```

---

## 16. Memoria y rendimiento

```python
tabla.nbytes                         # Memoria ocupada por los datos
pa.total_allocated_bytes()           # Memoria total reservada por Arrow
pa.set_cpu_count(8) ; pa.cpu_count()
pa.set_io_thread_count(16)           # Hilos de E/S (útil con almacenamiento en la nube)
```

- Seleccione columnas y filtre **al leer**, no después.
- Use `dictionary` para textos repetidos y `zstd` en Parquet.
- Procese por lotes (`iter_batches`, `to_batches`, `open_csv`) cuando los datos no quepan en memoria.
- Evite convertir a listas de Python (`to_pylist`) con muchos datos.

---

## 17. Ejemplo práctico completo

**Mini lago de datos de ventas** que reproduce un flujo real de ingeniería de datos:

1. Recibe exportaciones diarias en CSV de tres tiendas, en formato español (`;`, decimales con coma, fechas `dd/mm/aaaa`). Una tienda cambió su sistema a mitad de mes y **añadió una columna** y cambió el nombre de otra (deriva de esquema).
2. **Ingesta**: lectura con `pyarrow.csv` y tipos explícitos, normalización al esquema oficial, **validación** con `pyarrow.compute` (nulos, rangos, duplicados) y cuarentena de filas inválidas.
3. Escribe un **dataset Parquet particionado** por año y mes (Hive), con compresión zstd, datos ordenados y metadatos propios; después añade una **carga incremental** sin reescribir lo existente.
4. **Consulta** el lago con proyección y filtros: muestra qué archivos se leen gracias a la poda de particiones y las estadísticas de los grupos de filas; agrega, une con el catálogo y calcula indicadores.
5. Compara tamaño y tiempo de lectura de CSV, Parquet y Feather, envía un resultado por **Arrow IPC** y lo abre en pandas (backend Arrow) y Polars sin copias.

```python
"""
lago_ventas.py — Ingesta, validación y consulta de un lago de datos Parquet con PyArrow.
Requiere: pip install pyarrow pandas   (opcional: polars)
Uso: python lago_ventas.py
"""
from __future__ import annotations

import random
import shutil
import time
from datetime import date, timedelta
from decimal import Decimal
from pathlib import Path

import pyarrow as pa
import pyarrow.compute as pc
import pyarrow.csv as pcsv
import pyarrow.dataset as ds
import pyarrow.feather as feather
import pyarrow.ipc as ipc
import pyarrow.parquet as pq

ENTRADA = Path("exportaciones")
LAGO = Path("lago/ventas")
CUARENTENA = Path("lago/cuarentena")
TIENDAS = ["Madrid", "Bilbao", "Sevilla"]
PRODUCTOS = {"P01": ("Portátil", 899.0), "P02": ("Monitor", 219.0), "P03": ("Teclado", 49.0),
             "P04": ("Ratón", 29.0), "P05": ("Silla", 189.0), "P06": ("Auriculares", 59.0)}

ESQUEMA = pa.schema([
    pa.field("ticket", pa.string(), nullable=False),
    pa.field("fecha", pa.date32(), nullable=False),
    pa.field("tienda", pa.dictionary(pa.int8(), pa.string())),
    pa.field("producto_id", pa.string()),
    pa.field("unidades", pa.int32()),
    pa.field("precio", pa.decimal128(10, 2)),
    pa.field("canal", pa.dictionary(pa.int8(), pa.string())),
    pa.field("anio", pa.int16()),
    pa.field("mes", pa.int8()),
], metadata={"propietario": "Departamento de Datos", "version_esquema": "2"})


def num(v, d=0) -> str:
    return f"{v:,.{d}f}".replace(",", "X").replace(".", ",").replace("X", ".")


# ============================================================ 1. Exportaciones de ejemplo
def generar_exportaciones(inicio: date, dias: int, semilla: int = 3) -> None:
    rng = random.Random(semilla)
    ENTRADA.mkdir(exist_ok=True)
    for d in range(dias):
        dia = inicio + timedelta(days=d)
        for tienda in TIENDAS:
            nuevo_sistema = tienda == "Bilbao" and dia >= date(2026, 10, 15)     # Deriva de esquema
            cab = ["ticket", "fecha", "producto", "uds", "precio_unitario"] + (["canal"] if nuevo_sistema else [])
            filas = [";".join(cab)]
            for i in range(rng.randint(900, 1300)):
                pid = rng.choice(list(PRODUCTOS))
                precio = PRODUCTOS[pid][1] * rng.uniform(0.92, 1.0)
                fila = [f"{tienda[:3].upper()}-{dia:%Y%m%d}-{i:05d}", dia.strftime("%d/%m/%Y"), pid,
                        str(rng.choice([1, 1, 1, 2, 3])), f"{precio:.2f}".replace(".", ",")]
                if rng.random() < 0.003:
                    fila[3] = "-1"                                  # Error: unidades negativas
                if rng.random() < 0.002:
                    fila[2] = ""                                    # Error: producto vacío
                if nuevo_sistema:
                    fila.append(rng.choice(["Tienda", "Web", "Web", "Teléfono"]))
                filas.append(";".join(fila))
            if rng.random() < 0.3:
                filas.append(filas[-1])                             # Ticket duplicado (reenvío)
            (ENTRADA / f"{tienda.lower()}_{dia:%Y%m%d}.csv").write_text("\n".join(filas), encoding="utf-8")


# ============================================================ 2. Ingesta y validación
def leer_exportacion(ruta: Path) -> pa.Table:
    tienda = ruta.stem.split("_")[0].capitalize()
    tabla = pcsv.read_csv(
        ruta,
        parse_options=pcsv.ParseOptions(delimiter=";"),
        convert_options=pcsv.ConvertOptions(
            column_types={"ticket": pa.string(), "producto": pa.string(), "uds": pa.int32(),
                          "precio_unitario": pa.decimal128(10, 2), "fecha": pa.timestamp("s")},
            timestamp_parsers=["%d/%m/%Y"], decimal_point=",", strings_can_be_null=True),
    )
    fecha = pc.cast(tabla["fecha"], pa.date32())                    # Los formatos personalizados se leen como timestamp
    canal = tabla["canal"] if "canal" in tabla.column_names else pa.nulls(tabla.num_rows, pa.string())
    normalizada = pa.table({
        "ticket": tabla["ticket"], "fecha": fecha,
        "tienda": pa.array([tienda] * tabla.num_rows, pa.string()),
        "producto_id": tabla["producto"], "unidades": tabla["uds"], "precio": tabla["precio_unitario"],
        "canal": pc.fill_null(canal, "Tienda"),                     # Antes del cambio: todo era venta en tienda
        "anio": pc.cast(pc.year(fecha), pa.int16()), "mes": pc.cast(pc.month(fecha), pa.int8()),
    })
    return normalizada.cast(ESQUEMA)                                # Esquema oficial (diccionarios, metadatos)


def validar(tabla: pa.Table) -> tuple[pa.Table, pa.Table]:
    validos_producto = pc.is_in(tabla["producto_id"], value_set=pa.array(list(PRODUCTOS)))
    reglas = pc.and_(pc.and_(validos_producto, pc.greater(tabla["unidades"], 0)),
                     pc.greater(tabla["precio"], pa.scalar(Decimal("0"), pa.decimal128(10, 2))))
    reglas = pc.fill_null(reglas, False)
    buenas, malas = tabla.filter(reglas), tabla.filter(pc.invert(reglas))
    # Duplicados: conservar la primera aparición de cada ticket
    indices = pa.table({"ticket": buenas["ticket"], "i": pa.array(range(buenas.num_rows))})
    primeros = indices.group_by("ticket").aggregate([("i", "min")])["i_min"]
    unicas = buenas.take(pc.sort_indices(primeros))
    return unicas, malas.append_column("motivo", pa.array(["regla de validación"] * malas.num_rows))


def ingerir(rutas: list[Path], etiqueta: str) -> dict:
    tablas = [leer_exportacion(r) for r in rutas]
    datos = pa.concat_tables(tablas)
    validas, invalidas = validar(datos)
    duplicados = datos.num_rows - invalidas.num_rows - validas.num_rows
    validas = validas.sort_by([("fecha", "ascending"), ("ticket", "ascending")])   # Mejora las estadísticas min/max
    validas = validas.replace_schema_metadata({**ESQUEMA.metadata, b"lote": etiqueta.encode()})
    ds.write_dataset(validas, LAGO, format="parquet", partitioning=["anio", "mes"], partitioning_flavor="hive",
                     schema=validas.schema, existing_data_behavior="overwrite_or_ignore",
                     basename_template=f"{etiqueta}-{{i}}.parquet", max_rows_per_group=20_000,
                     preserve_order=True,                     # Mantiene el orden por fecha dentro de cada archivo
                     file_options=ds.ParquetFileFormat().make_write_options(compression="zstd"))
    if invalidas.num_rows:
        CUARENTENA.mkdir(parents=True, exist_ok=True)
        pq.write_table(invalidas, CUARENTENA / f"{etiqueta}.parquet")
    return {"archivos": len(rutas), "filas": datos.num_rows, "validas": validas.num_rows,
            "invalidas": invalidas.num_rows, "duplicadas": duplicados}


# ============================================================ 3. Consultas
def consultar() -> pa.Table:
    lago = ds.dataset(LAGO, format="parquet", partitioning="hive")
    print(f"\nLago: {len(lago.files)} archivos · {num(lago.count_rows())} filas · esquema con {len(lago.schema)} columnas")

    filtro = (pc.field("mes") == 11) & (pc.field("tienda") == "Bilbao") & (pc.field("canal") == "Web")
    leidos = [Path(f.path).relative_to(LAGO).as_posix() for f in lago.get_fragments(filter=filtro)]
    print(f"Filtro mes=11 + Bilbao + Web → solo se abren {len(leidos)} de {len(lago.files)} archivos: {leidos}")

    t0 = time.perf_counter()
    tabla = lago.to_table(columns=["fecha", "tienda", "producto_id", "unidades", "precio", "canal", "mes"])
    # Para analizar, decodificar las columnas diccionario (cada archivo trae su propio diccionario y
    # algunas operaciones, como ordenar o unir, no las admiten)
    tabla = tabla.cast(pa.schema([pa.field(f.name, f.type.value_type) if pa.types.is_dictionary(f.type) else f
                                  for f in tabla.schema]))
    importe = pc.multiply(pc.cast(tabla["unidades"], pa.decimal128(10, 0)), tabla["precio"])
    tabla = tabla.append_column("importe", importe)
    catalogo = pa.table({"producto_id": list(PRODUCTOS), "producto": [v[0] for v in PRODUCTOS.values()]})
    tabla = tabla.join(catalogo, keys="producto_id", join_type="left outer")

    por_tienda_mes = (tabla.group_by(["tienda", "mes"])
                           .aggregate([("importe", "sum"), ("unidades", "sum"), ("fecha", "count_distinct")])
                           .sort_by([("tienda", "ascending"), ("mes", "ascending")]))
    print(f"Consulta completa en {time.perf_counter() - t0:.2f} s\n")
    print(f"{'Tienda':<9}{'Mes':>4}{'Ventas':>16}{'Unidades':>10}{'Días':>6}")
    for fila in por_tienda_mes.to_pylist():
        print(f"{fila['tienda']:<9}{fila['mes']:>4}{num(fila['importe_sum'], 2):>14} €{num(fila['unidades_sum']):>10}"
              f"{fila['fecha_count_distinct']:>6}")

    bilbao = tabla.filter((pc.field("tienda") == "Bilbao") & (pc.field("fecha") >= date(2026, 10, 15)))
    canales = bilbao.group_by("canal").aggregate([("importe", "sum")])
    total = pc.sum(canales["importe_sum"]).as_py()
    print("\nBilbao desde el nuevo sistema (15/10), ventas por canal:")
    for fila in canales.sort_by([("importe_sum", "descending")]).to_pylist():
        cuota = f"{fila['importe_sum'] / total:.1%}".replace(".", ",")
        print(f"  {fila['canal']:<10} {num(fila['importe_sum'], 2):>14} € ({cuota})")

    ranking = (tabla.group_by("producto").aggregate([("importe", "sum"), ("unidades", "sum")])
                    .sort_by([("importe_sum", "descending")]))
    print("\nProductos por ventas:", ", ".join(f"{p} {num(v, 0)} €" for p, v in
                                              zip(ranking["producto"].to_pylist(), ranking["importe_sum"].to_pylist())))
    return por_tienda_mes


def inspeccionar_parquet() -> None:
    archivo = next(LAGO.rglob("*.parquet"))
    meta = pq.ParquetFile(archivo).metadata
    print(f"\nMetadatos de {archivo.relative_to(LAGO)}: {num(meta.num_rows)} filas en {meta.num_row_groups} grupos")
    idx = meta.schema.names.index("fecha")
    for g in range(min(3, meta.num_row_groups)):
        col = meta.row_group(g).column(idx)
        print(f"  grupo {g}: fecha {col.statistics.min} → {col.statistics.max} · "
              f"{col.total_compressed_size / 1024:.1f} KB comprimidos")
    print(f"  Metadatos propios: {pq.read_schema(archivo).metadata.get(b'lote', b'').decode()}")


def comparar_formatos() -> None:
    tabla = ds.dataset(LAGO, partitioning="hive").to_table()
    destino = Path("comparacion")
    destino.mkdir(exist_ok=True)
    pcsv.write_csv(tabla.cast(pa.schema([pa.field(f.name, pa.string() if pa.types.is_dictionary(f.type) else f.type)
                                         for f in tabla.schema])), destino / "ventas.csv")
    pq.write_table(tabla, destino / "ventas.parquet", compression="zstd")
    feather.write_feather(tabla, destino / "ventas.arrow", compression="lz4")
    print(f"\n{'Formato':<9}{'Tamaño':>10}{'Lectura':>10}")
    lectores = {"CSV": lambda p: pcsv.read_csv(p), "Parquet": pq.read_table, "Feather": lambda p: feather.read_table(p, memory_map=True)}
    for nombre, ext in [("CSV", "csv"), ("Parquet", "parquet"), ("Feather", "arrow")]:
        ruta = destino / f"ventas.{ext}"
        t0 = time.perf_counter()
        lectores[nombre](ruta)
        print(f"{nombre:<9}{ruta.stat().st_size / 1e6:>8.2f} MB{(time.perf_counter() - t0) * 1000:>8.1f} ms")


def intercambio(resumen: pa.Table) -> None:
    sink = pa.BufferOutputStream()
    with ipc.new_stream(sink, resumen.schema) as escritor:
        escritor.write_table(resumen)
    mensaje = sink.getvalue()
    recibido = ipc.open_stream(mensaje).read_all()
    print(f"\nArrow IPC: resumen serializado en {mensaje.size} bytes y recibido íntegro: {recibido.equals(resumen)}")
    import pandas as pd
    df = recibido.to_pandas(types_mapper=pd.ArrowDtype)
    print(f"pandas (backend Arrow): {dict(df.dtypes.astype(str))}")
    try:
        import polars as pl
        print(f"Polars: {pl.from_arrow(recibido).shape} filas × columnas, sin copia de datos")
    except ImportError:
        pass


def main() -> int:
    for carpeta in (ENTRADA, Path("lago"), Path("comparacion")):
        shutil.rmtree(carpeta, ignore_errors=True)

    generar_exportaciones(date(2026, 10, 1), dias=45)                    # 1/10 → 14/11
    rutas = sorted(ENTRADA.glob("*.csv"))
    print(f"Exportaciones recibidas: {len(rutas)} archivos CSV")
    r = ingerir(rutas, "carga-inicial")
    print(f"Carga inicial: {num(r['filas'])} filas → {num(r['validas'])} válidas, "
          f"{r['invalidas']} a cuarentena, {r['duplicadas']} duplicadas descartadas")

    generar_exportaciones(date(2026, 11, 15), dias=3, semilla=99)       # Llegan tres días más
    nuevas = sorted(p for p in ENTRADA.glob("*.csv") if p.stem.split("_")[1] >= "20261115")
    r = ingerir(nuevas, "incremental-20261117")
    print(f"Carga incremental: {r['archivos']} archivos · {num(r['validas'])} filas añadidas sin reescribir el lago")

    resumen = consultar()
    inspeccionar_parquet()
    comparar_formatos()
    intercambio(resumen)
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `CSV conversion error to date32[day]: invalid value '01/10/2026'` | `timestamp_parsers` solo se aplica a columnas `timestamp`. | Lea la columna como `pa.timestamp("s")` y conviértala con `pc.cast(..., pa.date32())`. |
| `ArrowInvalid: CSV conversion error to int64: invalid value 'N/D'` | Valor no convertible en una columna tipada. | Añádalo a `null_values` o lea la columna como `string` y limpie. |
| `ArrowInvalid: Schema at index 1 was different` | Concatenar tablas con esquemas distintos. | `cast` al esquema común o `promote_options="permissive"`. |
| `ArrowTypeError: int or Decimal object expected, got float` | Crear decimales a partir de `float`. | Use `Decimal("1.5")` o convierta con `pc.cast` desde `float64`. |
| `ArrowNotImplementedError: Function 'x' has no kernel matching input types` | Operación no definida para ese tipo (p. ej., aritmética con diccionarios o decimales y enteros). | Convierta antes (`pc.cast`, `dictionary_decode`). |
| `FileExistsError` / `ArrowInvalid` al escribir un dataset | `existing_data_behavior="error"` y el directorio tiene datos. | `"overwrite_or_ignore"` o `"delete_matching"`. |
| Se sobrescriben archivos en cargas incrementales | Mismo `basename_template` en cada carga. | Incluya un identificador único (fecha, lote) en el nombre. |
| Las particiones se leen como texto o no aparecen como columnas | Falta `partitioning="hive"` al abrir el dataset. | `ds.dataset(ruta, partitioning="hive")`. |
| Lectura lenta pese a usar filtros | Datos sin ordenar o grupos de filas enormes; filtro sobre columnas sin estadísticas. | Ordene antes de escribir, ajuste `row_group_size`, particione por la columna filtrada. |
| `ArrowTypeError: Unsupported type for RecordBatch sorting: dictionary<…>` | Ordenar por una columna de tipo diccionario. | Ordene por otra columna o decodifique antes (`pc.dictionary_decode` / `cast` a `string`). |
| `ArrowNotImplementedError: Unifying differing dictionaries` | Columnas diccionario con diccionarios distintos en cada trozo (varios archivos). | `tabla.unify_dictionaries()` o convertir a `string`. |
| `ArrowCapacityError` | Más de 2 GB de texto en un trozo de `string`. | `large_string` o trocear. |
| Tipos distintos al pasar a pandas (enteros con nulos → `float`) | Conversión a tipos NumPy. | `to_pandas(types_mapper=pd.ArrowDtype)`. |

---

## 19. Buenas prácticas

- Defina un **esquema oficial** y conviértale cada entrada (`cast`): detecta pronto los cambios de origen.
- Valide los datos en la ingesta y envíe las filas erróneas a cuarentena en lugar de descartarlas en silencio.
- Almacene en **Parquet + zstd**, particionado por las columnas de filtro más habituales (fecha, región), sin crear miles de particiones diminutas.
- Ordene los datos por la columna de filtro principal antes de escribir.
- Lea siempre seleccionando columnas y filtros; procese por lotes si los datos no caben en memoria.
- Use `dictionary` para textos repetidos y `decimal128` para importes.
- Guarde metadatos (origen, versión del esquema, lote) en el esquema para trazabilidad.
- Use Arrow como formato de intercambio entre pandas, Polars y DuckDB para evitar copias y pérdidas de tipo.

---

## 20. Referencia rápida

```python
pa.schema([pa.field(n, tipo)]) · pa.table({...}) · pa.array(valores, tipo) · tabla.cast(esquema) · pa.concat_tables(...)
tabla.select · filter(pc.field("x") > 0) · append_column · set_column · drop_columns · sort_by · group_by().aggregate · join
pc.sum/mean/min_max/quantile/value_counts · pc.add/multiply/divide/round · pc.if_else · pc.fill_null · pc.is_in
pc.utf8_upper/utf8_trim_whitespace/match_substring_regex · pc.year/month/strftime/strptime/floor_temporal
pcsv.read_csv(ruta, parse_options=, convert_options=) · pcsv.open_csv · pcsv.write_csv
pq.write_table(t, ruta, compression="zstd", row_group_size=) · pq.read_table(ruta, columns=, filters=) · pq.ParquetFile().metadata
ds.write_dataset(t, ruta, partitioning=, partitioning_flavor="hive", existing_data_behavior=) · ds.dataset(ruta, partitioning="hive")
dataset.to_table(columns=, filter=) · to_batches · count_rows · get_fragments(filter=)
feather.write_feather / read_table(memory_map=True) · ipc.new_stream / open_stream
t.to_pandas(types_mapper=pd.ArrowDtype) · pa.Table.from_pandas(df) · pl.from_arrow(t) · duckdb.sql("... FROM t")
```

---

## 21. Ejercicios propuestos

1. Añada a la ingesta la detección automática de deriva de esquema: compare el esquema de cada CSV con el último recibido y registre las diferencias.
2. Implemente una compactación mensual: lea todos los archivos pequeños de una partición y reescríbalos en uno solo con `delete_matching`.
3. Mida cómo cambian el tamaño y el tiempo de lectura con distintos `row_group_size`, compresiones y orden de los datos.
4. Publique el dataset en un almacenamiento S3 compatible (MinIO local) y consúltelo con PyArrow y con DuckDB.
5. Cree un servicio que reciba tablas en formato Arrow IPC por HTTP (FastAPI) y las añada al lago.

---

## 22. Referencias

- Documentación de PyArrow: https://arrow.apache.org/docs/python/
- Funciones de cálculo: https://arrow.apache.org/docs/python/api/compute.html
- Datasets: https://arrow.apache.org/docs/python/dataset.html
- Parquet en PyArrow: https://arrow.apache.org/docs/python/parquet.html
- Especificación de Apache Parquet: https://parquet.apache.org/docs/
