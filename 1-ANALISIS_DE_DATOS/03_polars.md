# Manual de uso: Polars

> **Categoría:** Ciencia de datos · **Utilidad:** Análisis de datos rápido, alternativa a pandas
> **Tipo:** Librería de terceros (`pip install polars`) · **Versión de referencia:** 1.0 o superior (ejemplos verificados en 2.x)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Creación de DataFrames y tipos de datos](#4-creación-de-dataframes-y-tipos-de-datos)
5. [Lectura y escritura de archivos](#5-lectura-y-escritura-de-archivos)
6. [Expresiones: el núcleo de Polars](#6-expresiones-el-núcleo-de-polars)
7. [Seleccionar, filtrar y crear columnas](#7-seleccionar-filtrar-y-crear-columnas)
8. [Texto, fechas y valores nulos](#8-texto-fechas-y-valores-nulos)
9. [Agregaciones con `group_by`](#9-agregaciones-con-group_by)
10. [Funciones de ventana (`over`)](#10-funciones-de-ventana-over)
11. [Uniones (*joins*)](#11-uniones-joins)
12. [Reestructuración: pivotar, despivotar y concatenar](#12-reestructuración-pivotar-despivotar-y-concatenar)
13. [Series temporales](#13-series-temporales)
14. [API diferida (*lazy*) y optimizador](#14-api-diferida-lazy-y-optimizador)
15. [Datos mayores que la memoria: *streaming*](#15-datos-mayores-que-la-memoria-streaming)
16. [Tipos anidados: listas y estructuras](#16-tipos-anidados-listas-y-estructuras)
17. [SQL en Polars](#17-sql-en-polars)
18. [Interoperabilidad con pandas, NumPy y Arrow](#18-interoperabilidad-con-pandas-numpy-y-arrow)
19. [Equivalencias pandas → Polars](#19-equivalencias-pandas--polars)
20. [Ejemplo práctico completo](#20-ejemplo-práctico-completo)
21. [Errores frecuentes](#21-errores-frecuentes)
22. [Buenas prácticas](#22-buenas-prácticas)
23. [Referencia rápida](#23-referencia-rápida)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Referencias](#25-referencias)

---

## 1. Descripción

**Polars** es una librería de DataFrames escrita en **Rust** sobre el formato columnar **Apache Arrow**. Ofrece una API de expresiones declarativa, ejecución **multihilo** automática y un **optimizador de consultas** (modo *lazy*) que la hacen, habitualmente, entre 5 y 30 veces más rápida que pandas con menor consumo de memoria.

**Ventajas:**
- Muy alto rendimiento: usa todos los núcleos y evita copias innecesarias.
- Evaluación diferida con optimizaciones automáticas (filtrado y selección de columnas en origen, eliminación de cálculos innecesarios).
- Procesa datos mayores que la memoria mediante *streaming*.
- API coherente y explícita: sin índice, sin `SettingWithCopyWarning`, tipos estrictos.
- Lectura y escritura rápidas de CSV, Parquet, JSON, Excel, bases de datos y almacenamiento en la nube.

**Limitaciones:**
- Ecosistema más joven: algunas librerías (statsmodels, algunas de visualización) esperan pandas o NumPy (la conversión es inmediata).
- API diferente de pandas: requiere aprender el estilo de expresiones.
- La API evoluciona rápido: entre versiones mayores hay funciones renombradas (consulte las notas de versión).
- No tiene índice de filas: las operaciones por etiqueta se hacen con filtros y uniones.

| Si necesita… | Opción |
|---|---|
| Análisis rápido de datos medianos y grandes en un equipo | **Polars** |
| Máxima compatibilidad con el ecosistema y tutoriales | pandas |
| Datos distribuidos en un clúster | Dask, Spark |
| SQL analítico sobre archivos | DuckDB |
| Formatos columnar y Parquet a bajo nivel | PyArrow |

---

## 2. Instalación e importación

```bash
pip install polars
pip install "polars[pyarrow,pandas,numpy,excel,plot]"     # Dependencias opcionales de interoperabilidad
pip install polars-lts-cpu                                  # Para CPU antiguas sin instrucciones AVX2
```

```python
import polars as pl
import polars.selectors as cs
pl.show_versions()
pl.Config.set_tbl_rows(20)               # Filas mostradas al imprimir
pl.Config.set_thousands_separator(".")   # Formato de presentación
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `DataFrame` | Tabla en memoria (modo inmediato, *eager*). |
| `LazyFrame` | Plan de consulta que se ejecuta al llamar a `.collect()` (modo diferido, *lazy*). |
| `Series` | Una columna con nombre y tipo. |
| `Expr` (expresión) | Descripción de un cálculo sobre columnas: `pl.col("precio") * 1.21`. |
| Contexto | Método donde se evalúan las expresiones: `select`, `with_columns`, `filter`, `group_by().agg()`. |
| Sin índice | Las filas se identifican por su contenido; no hay `loc` ni índices jerárquicos. |

Las expresiones de un mismo contexto se ejecutan **en paralelo**:

```python
df.with_columns(
    (pl.col("unidades") * pl.col("precio")).alias("importe"),
    pl.col("cliente").str.to_uppercase(),
    pl.col("fecha").dt.year().alias("anio"),
)                                         # Tres cálculos simultáneos
```

---

## 4. Creación de DataFrames y tipos de datos

```python
from datetime import date

df = pl.DataFrame({
    "fecha": [date(2026, 1, 5), date(2026, 1, 9), date(2026, 2, 3)],
    "region": ["Norte", "Sur", "Norte"],
    "unidades": [10, 4, 7],
    "precio": [19.9, 250.0, 19.9],
})
df = pl.DataFrame(filas, schema=["id", "nombre"], orient="row")        # Desde lista de tuplas
df = pl.DataFrame(lista_de_diccionarios)
df = pl.DataFrame({"x": [1, 2]}, schema={"x": pl.Int32})              # Tipos explícitos

df.shape, df.height, df.width, df.columns, df.schema, df.dtypes
df.head(5) ; df.tail(3) ; df.sample(10, seed=1) ; df.glimpse() ; df.describe()
df.estimated_size("mb")
```

| Tipo | Uso |
|---|---|
| `Int8`…`Int64`, `UInt8`…`UInt64` | Enteros (con nulos sin convertir a decimal, a diferencia de pandas). |
| `Float32`, `Float64` | Decimales. |
| `Decimal(precision, scale)` | Decimales exactos (importes). |
| `String` | Texto (UTF-8). |
| `Categorical`, `Enum([...])` | Categorías (dinámicas / fijas y ordenadas). |
| `Boolean` | Lógico. |
| `Date`, `Datetime("us", "Europe/Madrid")`, `Duration`, `Time` | Fechas y tiempos. |
| `List(tipo)`, `Array(tipo, n)`, `Struct` | Anidados. |
| `Null` | Columna sin valores. |

```python
df = df.with_columns(pl.col("unidades").cast(pl.Int32), pl.col("region").cast(pl.Categorical))
df.with_columns(pl.col("texto").cast(pl.Int64, strict=False))         # Valores no convertibles → null
```

---

## 5. Lectura y escritura de archivos

```python
df = pl.read_csv("ventas.csv", separator=";", decimal_comma=True, try_parse_dates=True,
                 schema_overrides={"cp": pl.String}, null_values=["", "NA", "-"], encoding="utf8")
df = pl.read_parquet("ventas.parquet", columns=["fecha", "importe"])
df = pl.read_excel("ventas.xlsx", sheet_name="Datos")                  # Requiere fastexcel
df = pl.read_json("datos.json") ; df = pl.read_ndjson("eventos.jsonl")
df = pl.read_database_uri("SELECT * FROM pedidos", "postgresql://usuario:clave@servidor/bd")
df = pl.read_database("SELECT * FROM pedidos", conexion_sqlalchemy)

lf = pl.scan_csv("datos/*.csv")                    # Lectura diferida (patrones glob)
lf = pl.scan_parquet("datos/anio=*/mes=*/*.parquet", hive_partitioning=True)
lf = pl.scan_parquet("s3://bucket/ventas/*.parquet", storage_options={"aws_region": "eu-west-1"})

df.write_parquet("salida.parquet", compression="zstd")
df.write_parquet("particionado/", partition_by=["anio", "mes"])
df.write_csv("salida.csv", separator=";", decimal_comma=True)
df.write_excel("salida.xlsx", worksheet="Resumen", autofit=True, table_style="Table Style Medium 2")   # Requiere xlsxwriter
df.write_database("tabla", conexion, if_table_exists="replace")
```

> Prefiera **Parquet** para almacenar datos intermedios: conserva los tipos, ocupa mucho menos y permite leer solo las columnas y filas necesarias.

---

## 6. Expresiones: el núcleo de Polars

```python
pl.col("precio")                         # Una columna
pl.col("precio", "unidades")             # Varias
pl.col(pl.Float64)                       # Por tipo
pl.col("^venta_.*$")                     # Expresión regular (entre ^ y $)
pl.all() ; pl.exclude("id")
pl.lit(0.21)                             # Valor literal
pl.len()                                 # Número de filas (en select o agg)

cs.numeric() ; cs.string() ; cs.temporal() ; cs.starts_with("venta_") ; cs.by_name("a", "b")
cs.numeric() - cs.by_name("id")          # Operaciones de conjuntos con selectores

(pl.col("importe") * (1 + pl.lit(0.21))).round(2).alias("importe_iva")
pl.col("importe").sum() ; .mean() ; .median() ; .std() ; .quantile(0.9) ; .n_unique() ; .max()
pl.col("importe").name.suffix("_eur")    # Renombrar resultados
pl.when(pl.col("importe") > 1000).then(pl.lit("Alto"))
  .when(pl.col("importe") > 100).then(pl.lit("Medio"))
  .otherwise(pl.lit("Bajo")).alias("tramo")
pl.col("estado").replace({"E": "Enviado", "P": "Pendiente"})
pl.col("estado").replace_strict({"E": 1, "P": 2}, default=0)        # Cambia el tipo
pl.col("importe").is_between(10, 100) ; pl.col("region").is_in(["Norte", "Sur"])
pl.col("importe").clip(0, 10_000) ; pl.col("x").abs() ; pl.col("x").log1p()
```

---

## 7. Seleccionar, filtrar y crear columnas

```python
df.select("fecha", "importe")                          # Solo esas columnas (y las calculadas)
df.select(pl.col("importe").sum(), pl.len().alias("n"))
df.with_columns(importe=pl.col("unidades") * pl.col("precio"))   # Añadir o sustituir columnas
df.filter(pl.col("region") == "Norte", pl.col("importe") > 100)  # Varias condiciones = AND
df.filter((pl.col("region") == "Norte") | pl.col("vip"))
df.filter(~pl.col("cliente").is_in(bloqueados))
df.drop("auxiliar") ; df.rename({"imp": "importe"})
df.sort("importe", descending=True, nulls_last=True) ; df.sort(["region", "fecha"])
df.top_k(10, by="importe") ; df.bottom_k(5, by="margen")
df.unique(subset=["cliente"], keep="first", maintain_order=True)
df.filter(pl.col("pedido").is_duplicated())             # Filas con clave repetida
df.with_row_index("n", offset=1)                       # Número de fila
df.slice(100, 20) ; df.row(0, named=True) ; df.item(0, "importe")
df["importe"]                                           # Series (acceso directo, solo lectura práctica)
```

> Polars no modifica DataFrames en el sitio: cada operación devuelve uno nuevo (sin copiar datos innecesariamente). Asigne el resultado: `df = df.with_columns(...)`.

---

## 8. Texto, fechas y valores nulos

```python
# Texto (.str)
pl.col("nombre").str.strip_chars().str.to_titlecase()
pl.col("email").str.to_lowercase().str.contains(r"@empresa\.es$")
pl.col("nif").str.replace_all(r"[\s\-]", "")
pl.col("ref").str.slice(0, 3) ; pl.col("ref").str.len_chars() ; pl.col("ref").str.zfill(8)
pl.col("ruta").str.split("/").list.last()
pl.col("texto").str.extract(r"(\d{5})", 1)             # Código postal
pl.col("importe_txt").str.replace(".", "", literal=True).str.replace(",", ".").cast(pl.Float64)
pl.concat_str([pl.col("nombre"), pl.col("apellido")], separator=" ")

# Fechas (.dt)
pl.col("fecha_txt").str.to_date("%d/%m/%Y", strict=False)       # Inválidas → null
pl.col("momento").str.to_datetime("%Y-%m-%d %H:%M:%S")
pl.col("fecha").dt.year() ; .dt.month() ; .dt.weekday() ; .dt.quarter() ; .dt.iso_year() ; .dt.week()
pl.col("fecha").dt.truncate("1mo")                     # Primer día del mes
pl.col("fecha").dt.month_end() ; pl.col("fecha").dt.offset_by("3d") ; pl.col("fecha").dt.strftime("%d/%m/%Y")
(pl.col("entrega") - pl.col("pedido")).dt.total_days()
pl.col("momento").dt.convert_time_zone("Europe/Madrid")
pl.date_range(date(2026, 1, 1), date(2026, 12, 31), "1d", eager=True)

# Nulos
df.null_count()
pl.col("descuento").fill_null(0) ; pl.col("precio").fill_null(strategy="forward")
pl.col("x").fill_nan(None)                             # NaN (decimal) ≠ null (ausente)
pl.coalesce("telefono_movil", "telefono_fijo", pl.lit("Sin teléfono"))
df.drop_nulls(subset=["cliente"]) ; pl.col("x").is_null()
```

---

## 9. Agregaciones con `group_by`

```python
resumen = (df.group_by("region", "canal")
             .agg(ventas=pl.col("importe").sum(),
                  pedidos=pl.len(),
                  ticket=pl.col("importe").mean(),
                  clientes=pl.col("cliente").n_unique(),
                  mayor=pl.col("importe").max(),
                  ventas_web=pl.col("importe").filter(pl.col("canal") == "Web").sum(),   # Agregado condicional
                  productos=pl.col("producto").unique().sort())                        # Lista
             .sort("ventas", descending=True))

df.group_by("region", maintain_order=True).agg(pl.col("importe").sum())   # Conserva el orden de aparición
df.group_by("cliente").agg(pl.all().sort_by("fecha").last())             # Último registro de cada cliente
df.group_by("region").agg(pl.col("importe").top_k(3).alias("tres_mayores"))
```

`group_by` no garantiza el orden de los grupos: ordene el resultado o use `maintain_order=True`.

---

## 10. Funciones de ventana (`over`)

Calculan un agregado por grupo **sin reducir filas** (equivalente a `groupby().transform` de pandas o a `OVER (PARTITION BY …)` en SQL):

```python
df.with_columns(
    total_region=pl.col("importe").sum().over("region"),
    cuota=pl.col("importe") / pl.col("importe").sum().over("region"),
    puesto=pl.col("importe").rank("dense", descending=True).over("region"),
    acumulado=pl.col("importe").cum_sum().over("cliente", order_by="fecha"),
    anterior=pl.col("importe").shift(1).over("cliente", order_by="fecha"),
    media_movil=pl.col("importe").rolling_mean(3).over("cliente", order_by="fecha"),
)
df.filter(pl.col("importe") == pl.col("importe").max().over("region"))   # Fila máxima de cada grupo
```

---

## 11. Uniones (*joins*)

```python
pedidos.join(clientes, on="cliente_id", how="left", validate="m:1")     # Comprueba la cardinalidad
pedidos.join(clientes, left_on="cliente", right_on="id", how="inner", suffix="_cli")
pedidos.join(devoluciones, on="pedido_id", how="anti")                 # Pedidos SIN devolución
pedidos.join(devoluciones, on="pedido_id", how="semi")                 # Pedidos CON devolución
a.join(b, on="clave", how="full", coalesce=True)                       # Externa completa
tallas.join(colores, how="cross")                                       # Producto cartesiano

# Unión por el valor más cercano anterior (tipos de cambio, precios vigentes)
ventas.sort("fecha").join_asof(tarifas.sort("desde"), left_on="fecha", right_on="desde",
                               by="producto", strategy="backward")
# Unión por condiciones de desigualdad
pedidos.join_where(promociones, pl.col("fecha") >= pl.col("inicio"), pl.col("fecha") <= pl.col("fin"))
```

| `how` | Resultado |
|---|---|
| `inner` | Solo coincidencias. |
| `left` / `right` | Todas las filas de un lado. |
| `full` | Todas las filas de ambos lados. |
| `semi` / `anti` | Filtra la izquierda por existencia / no existencia en la derecha. |
| `cross` | Todas las combinaciones. |

---

## 12. Reestructuración: pivotar, despivotar y concatenar

```python
ancho = df.pivot(on="mes", index="region", values="importe", aggregate_function="sum", sort_columns=True)
largo = ancho.unpivot(index="region", variable_name="mes", value_name="importe")

pl.concat([enero, febrero])                               # Vertical (mismo esquema)
pl.concat([a, b], how="vertical_relaxed")                 # Ajusta tipos compatibles
pl.concat([a, b], how="diagonal")                         # Columnas distintas → nulos
pl.concat([izq, der], how="horizontal")
df.explode("etiquetas")                                   # Una fila por elemento de la lista
df.partition_by("region", as_dict=True)                   # {("Norte",): DataFrame, …}
df.transpose(include_header=True)
```

---

## 13. Series temporales

```python
semanal = (df.sort("fecha")
             .group_by_dynamic("fecha", every="1w", period="1w", closed="left", group_by="region")
             .agg(ventas=pl.col("importe").sum(), pedidos=pl.len()))

mensual = df.sort("fecha").group_by_dynamic("fecha", every="1mo").agg(pl.col("importe").sum())

df.sort("fecha").with_columns(                            # Ventanas móviles basadas en tiempo
    ventas_30d=pl.col("importe").rolling_sum_by("fecha", window_size="30d"),
    media_7d=pl.col("importe").rolling_mean_by("fecha", window_size="7d"),
)
diario = df.sort("fecha").upsample("fecha", every="1d").with_columns(pl.col("importe").fill_null(0))
```

| Cadena | Significado |
|---|---|
| `"1d"`, `"1w"`, `"1mo"`, `"1q"`, `"1y"` | Día, semana, mes, trimestre, año. |
| `"1h"`, `"15m"`, `"30s"` | Horas, minutos, segundos. |
| `"1mo_saturating"` | Mes que se ajusta a fin de mes (31/01 + 1 mes = 28/02). |

---

## 14. API diferida (*lazy*) y optimizador

```python
consulta = (
    pl.scan_parquet("ventas/*.parquet")               # No lee nada todavía
      .filter(pl.col("fecha") >= date(2026, 1, 1))
      .with_columns(importe=pl.col("unidades") * pl.col("precio"))
      .group_by("region")
      .agg(pl.col("importe").sum())
      .sort("importe", descending=True)
)
print(consulta.explain())                            # Plan optimizado
resultado = consulta.collect()                       # Ejecuta
df.lazy() ; lf.collect_schema()                      # Pasar a diferido · esquema sin ejecutar
a, b = pl.collect_all([consulta_a, consulta_b])      # Varias consultas compartiendo trabajo
```

Optimizaciones automáticas:

| Optimización | Efecto |
|---|---|
| *Predicate pushdown* | Los filtros se aplican al leer el archivo (menos filas). |
| *Projection pushdown* | Solo se leen las columnas usadas. |
| *Slice pushdown* | `head()` deja de leer cuando tiene suficientes filas. |
| Eliminación de subexpresiones comunes | Los cálculos repetidos se hacen una vez. |
| Simplificación de expresiones | Constantes precalculadas, reordenación de operaciones. |

> Regla práctica: use **`scan_*` + `lazy` + `collect()`** para procesos con archivos; el modo inmediato es cómodo para exploración interactiva.

---

## 15. Datos mayores que la memoria: *streaming*

```python
(pl.scan_csv("logs/*.csv")
   .filter(pl.col("estado") >= 500)
   .group_by("servicio")
   .agg(pl.len().alias("errores"))
   .collect(engine="streaming"))                     # Procesa por lotes

(pl.scan_parquet("enorme/*.parquet")
   .with_columns(pl.col("importe").round(2))
   .sink_parquet("resultado.parquet"))               # Escribe sin cargar todo en memoria
lf.sink_csv("resultado.csv")
```

---

## 16. Tipos anidados: listas y estructuras

```python
agrupado = df.group_by("cliente").agg(pl.col("producto").alias("productos"))
agrupado.with_columns(
    n=pl.col("productos").list.len(),
    primero=pl.col("productos").list.first(),
    distintos=pl.col("productos").list.unique().list.len(),
    tiene_silla=pl.col("productos").list.contains("Silla"),
)
df.with_columns(direccion=pl.struct("calle", "cp", "ciudad"))           # Estructura
df.select(pl.col("direccion").struct.field("ciudad"))
df.unnest("direccion")                                                   # Estructura → columnas
pl.read_json("api.json").unnest("datos").explode("lineas")              # JSON anidado
```

---

## 17. SQL en Polars

```python
df.sql("SELECT region, SUM(importe) AS ventas FROM self GROUP BY region ORDER BY ventas DESC")

ctx = pl.SQLContext(pedidos=pedidos, clientes=clientes.lazy())
resultado = ctx.execute("""
    SELECT c.segmento, COUNT(*) AS pedidos, ROUND(AVG(p.importe), 2) AS ticket
    FROM pedidos p JOIN clientes c ON p.cliente_id = c.id
    WHERE p.fecha >= '2026-01-01'
    GROUP BY c.segmento
""", eager=True)
```

---

## 18. Interoperabilidad con pandas, NumPy y Arrow

```python
pdf = df.to_pandas()                                   # use_pyarrow_extension_array=True: sin copia
df = pl.from_pandas(pdf)
arr = df.select(cs.numeric()).to_numpy()
tabla = df.to_arrow() ; df = pl.from_arrow(tabla)      # Sin copia (mismo formato en memoria)
df.to_dicts() ; df.rows() ; df["columna"].to_list()
df.plot.bar(x="region", y="ventas")                    # Gráficos (Altair) con polars[plot]
```

---

## 19. Equivalencias pandas → Polars

| pandas | Polars |
|---|---|
| `df[df.x > 5]` | `df.filter(pl.col("x") > 5)` |
| `df[["a", "b"]]` | `df.select("a", "b")` |
| `df["c"] = df.a * 2` | `df = df.with_columns(c=pl.col("a") * 2)` |
| `df.assign(...)` | `df.with_columns(...)` |
| `df.groupby("g").agg({"x": "sum"})` | `df.group_by("g").agg(pl.col("x").sum())` |
| `df.groupby("g")["x"].transform("sum")` | `pl.col("x").sum().over("g")` |
| `df.merge(o, on="k", how="left")` | `df.join(o, on="k", how="left")` |
| `pd.concat([a, b])` | `pl.concat([a, b])` |
| `df.pivot_table(...)` | `df.pivot(on=, index=, values=, aggregate_function=)` |
| `df.melt(...)` | `df.unpivot(...)` |
| `np.where(c, a, b)` | `pl.when(c).then(a).otherwise(b)` |
| `df.apply(f, axis=1)` | Expresiones nativas (o `map_elements` como último recurso) |
| `df.resample("ME", on="fecha")` | `df.group_by_dynamic("fecha", every="1mo")` |
| `df.rolling("30D", on="fecha")` | `pl.col(...).rolling_sum_by("fecha", "30d")` |
| `pd.cut` / `pd.qcut` | `bin_intervals` / `bin_ranks`, `bin_quantiles` (antes `cut`/`qcut`) |
| `df.isna().sum()` | `df.null_count()` |
| `df.drop_duplicates()` | `df.unique()` |
| `df.reset_index()` | No aplica (no hay índice) |

---

## 20. Ejemplo práctico completo

**Análisis de una red logística con 1,5 millones de envíos**. El script:

1. Genera los datos: envíos en **Parquet particionado por mes** (estilo Hive) y un CSV «sucio» de transportistas exportado desde un ERP (separador `;`, decimales con coma, espacios, fechas en texto, duplicados).
2. Limpia los transportistas y **valida** la unión (`validate="m:1"`).
3. Construye una consulta **diferida** sobre todos los archivos y muestra cómo el optimizador aplica filtros y columnas en la lectura.
4. Calcula indicadores: plazos, puntualidad y coste por kg por transportista y región; *ranking* con funciones de ventana; evolución semanal con `group_by_dynamic` y media móvil; matriz pivotada; segmentación en terciles; rutas problemáticas por SQL.
5. Compara el tiempo con el mismo cálculo en pandas y exporta los resultados a Parquet y Excel.

```python
"""
red_logistica.py — Análisis de 1,5 millones de envíos con Polars (lazy, ventanas, series temporales, SQL).
Requiere: pip install polars pyarrow pandas xlsxwriter
Uso: python red_logistica.py
"""
from __future__ import annotations

import time
from datetime import date, timedelta
from pathlib import Path

import numpy as np
import polars as pl
import polars.selectors as cs

DATOS = Path("datos_logistica")
SALIDA = Path("resultados_logistica")
N_ENVIOS = 1_500_000
REGIONES = ["Norte", "Centro", "Este", "Sur", "Noroeste"]

pl.Config.set_tbl_rows(12)
pl.Config.set_tbl_hide_dataframe_shape(True)
pl.Config.set_float_precision(2)
pl.Config.set_tbl_width_chars(140)
pl.Config.set_fmt_str_lengths(30)


def titulo(texto: str) -> None:
    print(f"\n{texto}\n{'─' * len(texto)}")


# ============================================================ 1. Datos de ejemplo
def generar_datos() -> None:
    rng = np.random.default_rng(42)
    inicio = date(2025, 7, 1)
    dias = rng.integers(0, 365, N_ENVIOS)
    transportista = rng.choice([f"T{i:02d}" for i in range(1, 9)], N_ENVIOS, p=[.2, .18, .15, .12, .1, .1, .08, .07])
    region = rng.choice(REGIONES, N_ENVIOS, p=[.22, .3, .2, .18, .1])
    peso = np.round(rng.lognormal(1.6, 0.9, N_ENVIOS), 2)
    calidad = {f"T{i:02d}": q for i, q in enumerate([0.4, 0.8, 0.2, 1.4, 0.6, 1.0, 2.0, 0.3], start=1)}
    plazo_base = np.select([region == "Centro", region == "Noroeste"], [1, 3], default=2)
    factor_region = {"Centro": 0.7, "Norte": 1.0, "Este": 0.9, "Sur": 1.2, "Noroeste": 1.7}
    especialista = {("T03", "Noroeste"): 2.5, ("T05", "Sur"): 0.4, ("T02", "Centro"): 0.5}   # Puntos fuertes/débiles
    lam = [calidad[t] * factor_region[r] * especialista.get((t, r), 1.0) for t, r in zip(transportista, region)]
    retraso = rng.poisson(lam)
    envios = pl.DataFrame({
        "envio_id": np.arange(1, N_ENVIOS + 1),
        "fecha_envio": pl.Series([inicio + timedelta(days=int(d)) for d in range(365)])[dias],
        "transportista": transportista,
        "region_destino": region,
        "peso_kg": peso,
        "plazo_prometido": plazo_base,
        "dias_transito": plazo_base + retraso - (rng.random(N_ENVIOS) < 0.15),
        "incidencia": rng.random(N_ENVIOS) < np.array([calidad[t] for t in transportista]) * 0.02,
    }).with_columns(pl.col("dias_transito").clip(lower_bound=0))
    envios = envios.with_columns(anio=pl.col("fecha_envio").dt.year(), mes=pl.col("fecha_envio").dt.month())
    envios.write_parquet(DATOS / "envios", partition_by=["anio", "mes"])        # anio=2025/mes=7/….parquet

    nombres = ["Rápidos del Norte", "TransIbérica", "Logística Ágil", "Envíos Económicos",
               "Mensajería Centro", "Cargas del Sur", "Paquetería Low Cost", "Express Premium"]
    tarifas = ["0,62", "0,85", "0,98", "0,41", "0,77", "0,70", "0,35", "1,25"]
    filas = ["codigo;nombre;tarifa_kg;alta;activo"]
    for i, (n, t) in enumerate(zip(nombres, tarifas), start=1):
        filas.append(f" T{i:02d} ;  {n}  ;{t};{(i % 28) + 1:02d}/0{(i % 9) + 1}/2020;{'Sí' if i != 8 else 'sí'}")
    filas.append(" T04 ;Envíos Económicos;0,41;01/05/2020;Sí")                     # Duplicado
    (DATOS / "transportistas.csv").write_text("\n".join(filas), encoding="utf-8")


# ============================================================ 2. Limpieza de transportistas
def cargar_transportistas() -> pl.DataFrame:
    crudo = pl.read_csv(DATOS / "transportistas.csv", separator=";", schema_overrides={"tarifa_kg": pl.String})
    limpio = (crudo
              .with_columns(cs.string().str.strip_chars())
              .with_columns(tarifa_kg=pl.col("tarifa_kg").str.replace(",", ".").cast(pl.Float64),
                            alta=pl.col("alta").str.to_date("%d/%m/%Y", strict=False),
                            activo=pl.col("activo").str.to_lowercase() == "sí")
              .unique(subset="codigo", keep="first", maintain_order=True))
    print(f"Transportistas: {crudo.height} filas leídas → {limpio.height} tras limpiar duplicados")
    return limpio


# ============================================================ 3. Consulta diferida
def consulta_base(transportistas: pl.DataFrame) -> pl.LazyFrame:
    return (pl.scan_parquet(DATOS / "envios" / "**" / "*.parquet", hive_partitioning=True)
            .join(transportistas.lazy().select("codigo", "nombre", "tarifa_kg"),
                  left_on="transportista", right_on="codigo", how="left", validate="m:1")
            .with_columns(
                coste=(pl.col("peso_kg") * pl.col("tarifa_kg")).round(2),
                retraso=(pl.col("dias_transito") - pl.col("plazo_prometido")).clip(lower_bound=0),
                fecha_entrega=pl.col("fecha_envio").dt.offset_by(pl.format("{}d", pl.col("dias_transito"))),
            )
            .with_columns(a_tiempo=pl.col("retraso") == 0))


def mostrar_optimizacion(lf: pl.LazyFrame) -> None:
    titulo("Plan optimizado (filtro y columnas aplicados en la lectura)")
    consulta = lf.filter(pl.col("mes") == 12).select("transportista", "coste")
    plan = consulta.explain()
    for linea in plan.splitlines():
        if any(clave in linea for clave in ("Parquet SCAN", "SELECTION", "PROJECT", "FILTER")):
            print("  ", linea.strip()[:110])
    print("   → Solo se abre la partición mes=12 (poda de particiones Hive) y solo las columnas necesarias.")


# ============================================================ 4. Análisis
def indicadores(lf: pl.LazyFrame) -> dict[str, pl.DataFrame]:
    por_transportista = (lf.group_by("transportista", "nombre")
                           .agg(envios=pl.len(),
                                kg=pl.col("peso_kg").sum(),
                                coste=pl.col("coste").sum(),
                                puntualidad=pl.col("a_tiempo").mean(),
                                retraso_medio=pl.col("retraso").filter(~pl.col("a_tiempo")).mean(),
                                incidencias=pl.col("incidencia").mean(),
                                p95_transito=pl.col("dias_transito").quantile(0.95))
                           .with_columns(coste_kg=pl.col("coste") / pl.col("kg"),
                                         cuota=pl.col("envios") / pl.col("envios").sum())
                           .with_columns(segmento_coste=pl.col("coste_kg").bin_ranks(3, labels=["Económico", "Medio", "Caro"]),
                                         segmento_calidad=pl.col("puntualidad").bin_ranks(3, labels=["Baja", "Media", "Alta"]))
                           .sort("envios", descending=True))

    ranking_region = (lf.group_by("region_destino", "transportista")
                        .agg(envios=pl.len(), puntualidad=pl.col("a_tiempo").mean())
                        .filter(pl.col("envios") >= 5_000)
                        .with_columns(puesto=pl.col("puntualidad").rank("ordinal", descending=True).over("region_destino"),
                                      diferencia_media=pl.col("puntualidad") - pl.col("puntualidad").mean().over("region_destino"))
                        .filter(pl.col("puesto") <= 2)
                        .sort("region_destino", "puesto"))

    ultima_fecha = lf.select(pl.col("fecha_envio").max()).collect().item()
    semanal = (lf.sort("fecha_envio")
                 .group_by_dynamic("fecha_envio", every="1w", closed="left")
                 .agg(envios=pl.len(), puntualidad=pl.col("a_tiempo").mean(), coste=pl.col("coste").sum())
                 .with_columns(puntualidad_4s=pl.col("puntualidad").rolling_mean(4),
                               variacion_envios=pl.col("envios").pct_change())
                 .filter(pl.col("fecha_envio").dt.offset_by("6d") <= ultima_fecha))      # Solo semanas completas

    matriz = (lf.group_by("transportista", "region_destino")
                .agg(pl.col("a_tiempo").mean())
                .collect()
                .pivot(on="region_destino", index="transportista", values="a_tiempo", sort_columns=True)
                .sort("transportista"))

    resultados = dict(zip(["transportistas", "ranking_region", "semanal"],
                          pl.collect_all([por_transportista, ranking_region, semanal])))   # Ejecución conjunta
    resultados["matriz_puntualidad"] = matriz
    return resultados


def rutas_problematicas(lf: pl.LazyFrame) -> pl.DataFrame:
    ctx = pl.SQLContext(envios=lf)
    return ctx.execute("""
        SELECT transportista, region_destino,
               COUNT(*)                                        AS envios,
               ROUND(AVG(CASE WHEN a_tiempo THEN 1.0 ELSE 0 END) * 100, 1) AS pct_a_tiempo,
               ROUND(AVG(retraso), 2)                          AS retraso_medio
        FROM envios
        GROUP BY transportista, region_destino
        HAVING COUNT(*) > 10000 AND AVG(CASE WHEN a_tiempo THEN 1.0 ELSE 0 END) < 0.55
        ORDER BY pct_a_tiempo
    """, eager=True)


# ============================================================ 5. Comparación con pandas
def comparar_con_pandas(transportistas: pl.DataFrame) -> None:
    import pandas as pd
    titulo("Rendimiento: mismo cálculo en pandas y Polars")

    t0 = time.perf_counter()
    envios = pd.read_parquet(DATOS / "envios", partitioning=None)       # Las columnas anio/mes ya están en los archivos
    tr = transportistas.to_pandas()
    m = envios.merge(tr[["codigo", "tarifa_kg"]], left_on="transportista", right_on="codigo", how="left")
    m["coste"] = m["peso_kg"] * m["tarifa_kg"]
    m["a_tiempo"] = m["dias_transito"] <= m["plazo_prometido"]
    r_pd = m.groupby(["transportista", "region_destino"], observed=True).agg(coste=("coste", "sum"), puntualidad=("a_tiempo", "mean"))
    t_pd = time.perf_counter() - t0

    t0 = time.perf_counter()
    r_pl = (pl.scan_parquet(DATOS / "envios" / "**" / "*.parquet")
              .join(transportistas.lazy().select("codigo", "tarifa_kg"), left_on="transportista", right_on="codigo", how="left")
              .group_by("transportista", "region_destino")
              .agg(coste=(pl.col("peso_kg") * pl.col("tarifa_kg")).sum(),
                   puntualidad=(pl.col("dias_transito") <= pl.col("plazo_prometido")).mean())
              .collect())
    t_pl = time.perf_counter() - t0
    iguales = np.isclose(r_pd["coste"].sum(), r_pl["coste"].sum())
    print(f"pandas: {t_pd:.2f} s · Polars: {t_pl:.2f} s · {t_pd / t_pl:.1f}× más rápido · resultados iguales: {iguales}")


def main() -> int:
    DATOS.mkdir(exist_ok=True)
    SALIDA.mkdir(exist_ok=True)
    if not (DATOS / "envios").exists():
        t0 = time.perf_counter()
        generar_datos()
        print(f"Datos generados en {time.perf_counter() - t0:.1f} s "
              f"({len(list((DATOS / 'envios').rglob('*.parquet')))} archivos Parquet)")

    transportistas = cargar_transportistas()
    lf = consulta_base(transportistas)
    print(f"Esquema de la consulta: {len(lf.collect_schema())} columnas · {lf.select(pl.len()).collect().item():,} envíos")
    mostrar_optimizacion(lf)

    t0 = time.perf_counter()
    r = indicadores(lf)
    print(f"\nIndicadores calculados en {time.perf_counter() - t0:.2f} s")

    titulo("Transportistas")
    print(r["transportistas"].select("transportista", "nombre", "envios", "puntualidad", "retraso_medio",
                                     "coste_kg", "segmento_coste", "segmento_calidad"))
    titulo("Los dos más puntuales de cada región")
    print(r["ranking_region"])
    titulo("Matriz de puntualidad transportista × región")
    print(r["matriz_puntualidad"])
    titulo("Últimas semanas")
    print(r["semanal"].tail(5))
    titulo("Rutas problemáticas (SQL)")
    print(rutas_problematicas(lf))

    for nombre, df in r.items():
        df.write_parquet(SALIDA / f"{nombre}.parquet")
    excel = SALIDA / "informe_logistica.xlsx"
    import xlsxwriter
    with xlsxwriter.Workbook(excel) as libro:
        for nombre, df in r.items():
            df.write_excel(libro, worksheet=nombre[:31], autofit=True, table_style="Table Style Medium 2",
                           float_precision=3, dtype_formats={pl.Date: "dd/mm/yyyy"})
    print(f"\n✔ Resultados en «{SALIDA}» (Parquet y {excel.name})")

    comparar_con_pandas(transportistas)
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 21. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ColumnNotFoundError` | Nombre de columna incorrecto o creada en el mismo contexto. | Revise `df.columns`; use la columna nueva en un `with_columns` posterior. |
| `InvalidOperationError: conversion from str to i64 failed` | `cast` estricto con valores no convertibles. | `cast(..., strict=False)` o limpie el texto antes. |
| `ComputeError: could not parse … as dtype` al leer CSV | Inferencia de tipos con pocas filas. | `infer_schema_length=10000`, `schema_overrides` o leer como texto. |
| `SchemaError` al concatenar | Tipos o columnas distintas. | `how="vertical_relaxed"` o `how="diagonal"`, o castee antes. |
| `DuplicateError: column with name 'x' has more than one occurrence` | Dos expresiones producen el mismo nombre. | `.alias()` distintos. |
| `ComputeError: join keys did not fulfill m:1 validation` | Duplicados en la tabla de dimensiones. | Elimine duplicados (`unique`) o revise la clave. |
| `TypeError: 'Expr' object is not iterable` / comparaciones con `and`/`or` | Operadores de Python en lugar de los de Polars. | `&`, `\|`, `~` y paréntesis. |
| `PolarsInefficientMapWarning` | `map_elements` con algo que se puede expresar nativamente. | Use la expresión sugerida en el aviso. |
| `AttributeError: 'DataFrame' object has no attribute 'groupby'` / `melt` / `apply` | Nombres de pandas o de versiones antiguas de Polars. | `group_by`, `unpivot`, expresiones. |
| `DeprecationWarning: cut/qcut is deprecated` | Renombrado en Polars 2. | `bin_intervals`, `bin_ranks`, `bin_quantiles`. |
| Orden de grupos distinto en cada ejecución | `group_by` en paralelo. | `.sort(...)` o `maintain_order=True`. |
| Falta memoria con archivos grandes | Lectura inmediata completa. | `scan_*`, filtros tempranos, `collect(engine="streaming")`, `sink_parquet`. |

---

## 22. Buenas prácticas

- Piense en **expresiones**, no en bucles: evite `map_elements` y `iter_rows` salvo casos excepcionales.
- Use el modo **lazy** (`scan_*` → `collect`) en procesos con archivos y deje que el optimizador trabaje.
- Filtre y seleccione columnas lo antes posible; almacene datos intermedios en **Parquet**.
- Declare los tipos al leer (`schema_overrides`) y use `Enum`/`Categorical` para textos repetidos.
- Valide las uniones con `validate=` y compruebe nulos tras las uniones izquierdas.
- Agrupe varias consultas con `pl.collect_all` para compartir lecturas.
- Convierta a pandas/NumPy solo en el borde (gráficos o librerías que lo requieran).
- Fije la versión en `requirements.txt` y revise las notas de versión al actualizar: la API evoluciona.

---

## 23. Referencia rápida

```python
pl.read_csv/read_parquet/read_excel · pl.scan_csv/scan_parquet(hive_partitioning=True) · df.write_parquet/write_csv/write_excel
df.select(...) · df.with_columns(nombre=expr) · df.filter(c1, c2) · df.sort(col, descending=) · df.unique(subset=) · df.top_k
pl.col("x") · pl.lit(v) · pl.len() · pl.all() · cs.numeric() · .alias() · .cast(tipo, strict=False) · pl.when().then().otherwise()
.str.strip_chars/contains/replace_all/to_date · .dt.year/truncate/offset_by · .fill_null · pl.coalesce · .is_in · .is_between
df.group_by("g").agg(x=pl.col("v").sum(), n=pl.len()) · expr.over("g", order_by=) · .rank() · .cum_sum() · .shift()
a.join(b, on=, how="left", validate="m:1") · join_asof · join_where · pl.concat([...], how=) · pivot · unpivot · explode
group_by_dynamic("fecha", every="1mo") · rolling_mean_by("fecha", "7d") · upsample · bin_intervals/bin_ranks/bin_quantiles
lf = df.lazy() · lf.explain() · lf.collect(engine="streaming") · lf.sink_parquet() · pl.collect_all([...]) · df.sql("... FROM self")
df.to_pandas() · pl.from_pandas() · df.to_numpy() · df.to_arrow()
```

---

## 24. Ejercicios propuestos

1. Añada al ejemplo una tabla de tarifas con vigencia por fechas y calcule el coste con `join_asof` según la fecha de cada envío.
2. Detecte días anómalos (envíos fuera de ±3 desviaciones de la media móvil de 28 días) por región.
3. Reescriba en Polars un análisis existente de pandas de su organización y compare tiempos y legibilidad.
4. Procese con `collect(engine="streaming")` un CSV de varios GB y escriba el resultado con `sink_parquet`, midiendo la memoria máxima.
5. Construya una cohorte de clientes (mes de primera compra × meses transcurridos) con `over`, `pivot` y porcentajes de retención.

---

## 25. Referencias

- Documentación y guía de usuario: https://docs.pola.rs/
- Referencia de la API de Python: https://docs.pola.rs/api/python/stable/reference/
- Migración desde pandas: https://docs.pola.rs/user-guide/migration/pandas/
- Notas de versión y cambios incompatibles: https://github.com/pola-rs/polars/releases
