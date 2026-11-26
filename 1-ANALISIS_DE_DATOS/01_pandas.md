# Manual de uso: Pandas

> **Categoría:** Ciencia de datos · **Utilidad:** Análisis y manipulación de datos tabulares
> **Tipo:** Librería de terceros (`pip install pandas`) · **Versión de referencia:** 2.2 – 3.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Crear `Series` y `DataFrame`](#4-crear-series-y-dataframe)
5. [Leer y escribir datos](#5-leer-y-escribir-datos)
6. [Explorar un `DataFrame`](#6-explorar-un-dataframe)
7. [Seleccionar datos: `[]`, `loc`, `iloc`](#7-seleccionar-datos--loc-iloc)
8. [Filtrar filas](#8-filtrar-filas)
9. [Crear y transformar columnas](#9-crear-y-transformar-columnas)
10. [Tipos de datos](#10-tipos-de-datos)
11. [Valores ausentes y duplicados](#11-valores-ausentes-y-duplicados)
12. [Texto (`.str`)](#12-texto-str)
13. [Fechas y series temporales](#13-fechas-y-series-temporales)
14. [Ordenar y clasificar](#14-ordenar-y-clasificar)
15. [Agrupar y agregar (`groupby`)](#15-agrupar-y-agregar-groupby)
16. [Tablas dinámicas y reestructuración](#16-tablas-dinámicas-y-reestructuración)
17. [Combinar datos: `merge` y `concat`](#17-combinar-datos-merge-y-concat)
18. [Ventanas móviles y acumulados](#18-ventanas-móviles-y-acumulados)
19. [Gráficos rápidos](#19-gráficos-rápidos)
20. [Rendimiento y memoria](#20-rendimiento-y-memoria)
21. [Cambios importantes en pandas 3.0](#21-cambios-importantes-en-pandas-30)
22. [Ejemplo práctico completo](#22-ejemplo-práctico-completo)
23. [Errores frecuentes](#23-errores-frecuentes)
24. [Buenas prácticas](#24-buenas-prácticas)
25. [Referencia rápida](#25-referencia-rápida)
26. [Ejercicios propuestos](#26-ejercicios-propuestos)
27. [Referencias](#27-referencias)

---

## 1. Descripción

**pandas** es la librería de referencia en Python para el **análisis de datos tabulares**: tablas con filas y columnas como las de una hoja de cálculo o una base de datos. Permite leer datos de CSV, Excel, SQL, JSON o Parquet; limpiarlos; transformarlos; agruparlos; combinarlos; analizar series temporales y exportar los resultados, todo con operaciones vectorizadas muy eficientes construidas sobre **NumPy** (y opcionalmente **Apache Arrow**).

**Ventajas:**
- Estructuras potentes y expresivas: `Series` y `DataFrame`.
- Lectura y escritura de casi cualquier formato.
- Operaciones vectorizadas mucho más rápidas que los bucles de Python.
- Agrupaciones, tablas dinámicas, uniones y series temporales de alto nivel.
- Integración con todo el ecosistema: NumPy, Matplotlib, Seaborn, scikit-learn, SQLAlchemy, Streamlit…

**Limitaciones:**
- Trabaja en memoria: los datos deben caber en RAM (con margen).
- Muchas formas de hacer lo mismo; la API es amplia.
- Para volúmenes muy grandes o máximo rendimiento, existen **Polars**, **DuckDB** o **Dask**.

| Si necesita… | Opción |
|---|---|
| Analizar y transformar datos tabulares que caben en memoria | **pandas** |
| Cálculo numérico con matrices | NumPy |
| Datos que no caben en memoria o máximo rendimiento | Polars / DuckDB / Dask |
| Leer y dar formato a Excel con estilos | OpenPyXL (pandas lo usa internamente) |
| Visualizar los datos | Matplotlib / Seaborn / Plotly |

---

## 2. Instalación e importación

```bash
pip install pandas
pip install pyarrow openpyxl         # Recomendados: Parquet/Arrow y Excel
```

```python
import pandas as pd
import numpy as np

pd.set_option("display.max_columns", 50)
pd.set_option("display.width", 120)
pd.set_option("display.float_format", "{:,.2f}".format)
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Series` | Columna de datos con un índice (array unidimensional etiquetado). |
| `DataFrame` | Tabla de columnas (`Series`) que comparten índice. |
| Índice (`Index`) | Etiquetas de las filas (números, textos, fechas). |
| `dtype` | Tipo de datos de una columna (`int64`, `float64`, `str`, `datetime64`, `category`…). |
| `NaN` / `NA` / `NaT` | Valores ausentes. |
| Vectorización | Operar con columnas completas en lugar de recorrer filas. |
| Máscara booleana | `Series` de `True`/`False` para filtrar filas. |
| *Copy-on-Write* | Cada `DataFrame` derivado se comporta como una copia independiente (por defecto en pandas 3). |

```
            fecha     cliente  importe     ← columnas
0      2026-01-03   Acme S.L.   120.50
1      2026-01-04   Globex       89.90
↑ índice
```

---

## 4. Crear `Series` y `DataFrame`

```python
s = pd.Series([10, 20, 30], index=["a", "b", "c"], name="ventas")

df = pd.DataFrame({
    "producto": ["Teclado", "Ratón", "Monitor"],
    "precio": [59.9, 19.5, 189.0],
    "stock": [12, 40, 5],
})

pd.DataFrame([{"producto": "Hub", "precio": 29.9}, {"producto": "Cable", "precio": 4.5}])   # Lista de diccionarios
pd.DataFrame(np.random.default_rng(0).normal(size=(5, 3)), columns=["x", "y", "z"])         # Desde NumPy
pd.DataFrame.from_dict({"a": 1, "b": 2}, orient="index", columns=["valor"])
```

---

## 5. Leer y escribir datos

### 5.1 CSV

```python
df = pd.read_csv(
    "ventas.csv",
    sep=";",                                  # Separador (CSV español habitual)
    decimal=",",                              # Separador decimal
    thousands=".",                            # Separador de miles
    encoding="utf-8",                         # o "latin-1"/"cp1252" para archivos antiguos de Windows
    parse_dates=["fecha"],                    # Convertir a fecha
    dayfirst=True,                            # 03/01/2026 = 3 de enero
    dtype={"codigo_postal": "str"},           # Mantener ceros iniciales
    usecols=["fecha", "cliente", "importe"],  # Solo algunas columnas
    na_values=["", "N/D", "-"],               # Textos que significan «sin dato»
    nrows=1000,                               # Solo las primeras filas
)

df.to_csv("salida.csv", sep=";", decimal=",", index=False, encoding="utf-8-sig")   # utf-8-sig: Excel lo abre bien
```

Archivos grandes por bloques:

```python
total = 0
for bloque in pd.read_csv("enorme.csv", chunksize=100_000):
    total += bloque["importe"].sum()
```

### 5.2 Excel

```python
df = pd.read_excel("informe.xlsx", sheet_name="Ventas", skiprows=2, usecols="A:F")
hojas = pd.read_excel("informe.xlsx", sheet_name=None)     # Diccionario {nombre_hoja: DataFrame}

with pd.ExcelWriter("resultado.xlsx", engine="openpyxl") as escritor:
    df.to_excel(escritor, sheet_name="Datos", index=False)
    resumen.to_excel(escritor, sheet_name="Resumen")
```

### 5.3 Otros formatos

```python
pd.read_json("datos.json")                    ; df.to_json("datos.json", orient="records", force_ascii=False, indent=2)
pd.json_normalize(lista_de_dicts_anidados)    # JSON anidado → tabla plana
pd.read_parquet("datos.parquet")              ; df.to_parquet("datos.parquet")    # Rápido y compacto (pyarrow)
pd.read_sql("SELECT * FROM ventas", conexion) ; df.to_sql("ventas", conexion, if_exists="replace", index=False)
pd.read_html("https://…/tabla.html")          # Lista de tablas HTML (requiere lxml)
pd.read_clipboard()                           # Lo copiado desde Excel
df.to_markdown()                              # Requiere tabulate
```

> **Parquet** es el formato recomendado para guardar datos intermedios: conserva los tipos, ocupa poco y se lee muy rápido.

---

## 6. Explorar un `DataFrame`

```python
df.head(10) ; df.tail() ; df.sample(5, random_state=1)
df.shape                      # (filas, columnas)
df.columns ; df.index ; df.dtypes
df.info()                     # Tipos, nulos y memoria
df.describe()                 # Estadísticos de columnas numéricas
df.describe(include="all")
df["cliente"].value_counts()                    # Frecuencias
df["cliente"].value_counts(normalize=True)      # Proporciones
df["cliente"].nunique() ; df["cliente"].unique()
df.isna().sum()                                 # Nulos por columna
df.corr(numeric_only=True)                      # Correlaciones
df.memory_usage(deep=True).sum() / 1e6          # MB
```

---

## 7. Seleccionar datos: `[]`, `loc`, `iloc`

```python
df["precio"]                          # Una columna → Series
df[["producto", "precio"]]            # Varias columnas → DataFrame
df.precio                             # Atajo (solo si el nombre es un identificador válido)

df.loc[0]                             # Fila por etiqueta del índice
df.loc[0:2, ["producto", "precio"]]   # Filas 0 a 2 (incluida) y columnas por nombre
df.loc[df["stock"] < 10, "producto"]  # Filtro + columna
df.iloc[0]                            # Fila por posición
df.iloc[:3, 1:]                       # Primeras 3 filas, desde la 2.ª columna (posición 3 excluida)
df.at[0, "precio"]                    # Un valor por etiqueta (rápido)
df.iat[0, 1]                          # Un valor por posición
```

| Selector | Usa | Rango final |
|---|---|---|
| `loc` | Etiquetas | Incluido |
| `iloc` | Posiciones enteras | Excluido |

Modificar valores (siempre con `loc` en una sola operación):

```python
df.loc[df["stock"] == 0, "estado"] = "agotado"
df.loc[3] = ["Hub USB", 29.9, 20]        # Añadir/sustituir fila con etiqueta 3
```

Índices:

```python
df = df.set_index("producto")
df = df.reset_index()
df.index.name = "id"
```

---

## 8. Filtrar filas

```python
df[df["precio"] > 50]
df[(df["precio"] > 20) & (df["stock"] > 0)]          # Y (paréntesis obligatorios)
df[(df["categoria"] == "Monitores") | (df["precio"] < 10)]   # O
df[~df["descatalogado"]]                              # Negación
df[df["cliente"].isin(["Acme", "Globex"])]
df[df["precio"].between(10, 100)]
df[df["producto"].str.contains("usb", case=False, na=False)]
df[df["email"].notna()]
df.query("precio > 20 and stock > 0 and categoria in @cats")   # @ → variable de Python
df.nlargest(5, "importe") ; df.nsmallest(3, "precio")
```

> Use `&`, `|` y `~` (no `and`, `or`, `not`) y ponga cada condición entre paréntesis.

---

## 9. Crear y transformar columnas

```python
df["total"] = df["precio"] * df["stock"]
df["iva"] = df["precio"] * 0.21
df["con_iva"] = (df["precio"] * 1.21).round(2)
df = df.assign(margen=lambda d: d["precio"] - d["coste"], margen_pct=lambda d: d["margen"] / d["precio"])

# Condiciones
df["tipo"] = np.where(df["precio"] > 100, "premium", "estándar")
df["nivel"] = np.select([df["stock"] == 0, df["stock"] < 10], ["agotado", "bajo"], default="normal")
df["rango"] = pd.cut(df["precio"], bins=[0, 20, 100, np.inf], labels=["barato", "medio", "caro"])
df["cuartil"] = pd.qcut(df["importe"], q=4, labels=["Q1", "Q2", "Q3", "Q4"])

# Mapear valores
df["pais"] = df["codigo_pais"].map({"ES": "España", "FR": "Francia"})
df["categoria"] = df["categoria"].replace({"Perifericos": "Periféricos"})

# Aplicar funciones (más lento: úselo solo si no existe una operación vectorizada)
df["iniciales"] = df["cliente"].apply(lambda t: "".join(p[0] for p in t.split()))
df["resumen"] = df.apply(lambda fila: f"{fila['producto']} ({fila['stock']} uds.)", axis=1)

# Renombrar, eliminar y reordenar
df = df.rename(columns={"precio": "precio_eur"})
df.columns = df.columns.str.lower().str.replace(" ", "_")
df = df.drop(columns=["iva", "tipo"])
df = df[["producto", "precio_eur", "stock"]]
```

---

## 10. Tipos de datos

```python
df["cantidad"] = df["cantidad"].astype("int64")
df["importe"] = pd.to_numeric(df["importe"], errors="coerce")   # Lo no numérico → NaN
df["fecha"] = pd.to_datetime(df["fecha"], format="%d/%m/%Y", errors="coerce")
df["categoria"] = df["categoria"].astype("category")              # Pocos valores repetidos: ahorra memoria
df["activo"] = df["activo"].map({"Sí": True, "No": False})
df = df.convert_dtypes()                                          # Tipos que admiten nulos (Int64, boolean, string)
```

| Tipo | Uso |
|---|---|
| `int64` / `Int64` | Enteros (`Int64` admite valores nulos). |
| `float64` | Decimales. |
| `bool` / `boolean` | Lógicos. |
| `str` (pandas 3) / `object` (pandas 2) | Texto. |
| `category` | Texto con pocos valores distintos. |
| `datetime64[…]`, `timedelta64[…]` | Fechas y duraciones. |

> Para importes monetarios exactos, trabaje en céntimos (enteros) o redondee al presentar: `float` tiene errores de redondeo binario.

---

## 11. Valores ausentes y duplicados

```python
df.isna().sum()
df.dropna()                                   # Elimina filas con algún nulo
df.dropna(subset=["email"])                   # Solo si falta el email
df.dropna(thresh=3)                           # Mantener filas con al menos 3 valores
df["stock"] = df["stock"].fillna(0)
df["ciudad"] = df["ciudad"].fillna("Desconocida")
df["precio"] = df["precio"].fillna(df["precio"].median())
df["precio"] = df.groupby("categoria")["precio"].transform(lambda s: s.fillna(s.median()))
df["temperatura"] = df["temperatura"].ffill()        # Arrastrar el último valor (también bfill)
df["valor"] = df["valor"].interpolate()

df.duplicated().sum()
df = df.drop_duplicates()
df = df.drop_duplicates(subset=["email"], keep="last")
```

---

## 12. Texto (`.str`)

```python
s = df["cliente"]
s.str.strip() ; s.str.lower() ; s.str.upper() ; s.str.title()
s.str.len()
s.str.contains("s.l.", case=False, regex=False)
s.str.startswith("Acme") ; s.str.endswith("S.A.")
s.str.replace(r"\s+", " ", regex=True)
s.str.split(" ", n=1, expand=True)               # Separar en columnas
s.str.extract(r"(?P<nombre>\w+)\s+(?P<apellido>\w+)")   # Grupos con nombre → columnas
s.str.zfill(5)                                   # '28' → '00028'
s.str.slice(0, 3) ; s.str[:3]
s.str.normalize("NFKD").str.encode("ascii", "ignore").str.decode("ascii")   # Quitar tildes
```

---

## 13. Fechas y series temporales

```python
df["fecha"] = pd.to_datetime(df["fecha"], dayfirst=True)
df["anio"] = df["fecha"].dt.year
df["mes"] = df["fecha"].dt.month
df["dia_semana"] = df["fecha"].dt.day_name(locale="es_ES.UTF-8")    # Requiere la configuración regional instalada
df["trimestre"] = df["fecha"].dt.quarter
df["periodo"] = df["fecha"].dt.to_period("M")                       # 2026-01
df["fin_de_semana"] = df["fecha"].dt.dayofweek >= 5
df["dias"] = (pd.Timestamp("2026-12-31") - df["fecha"]).dt.days

pd.date_range("2026-01-01", "2026-12-31", freq="D")
pd.date_range("2026-01-01", periods=12, freq="ME")      # Fin de mes (pandas ≥ 2.2: "ME", no "M")

# Remuestreo (requiere índice de fechas o 'on=')
serie = df.set_index("fecha")["importe"]
serie.resample("W").sum()          # Semanal
serie.resample("ME").agg(["sum", "mean", "count"])
serie.resample("QE").sum()          # Trimestral
df.resample("ME", on="fecha")["importe"].sum()

serie.loc["2026-03"]                # Todo marzo
serie.loc["2026-03-01":"2026-03-15"]
serie.shift(1) ; serie.pct_change() ; serie.diff()
```

| Frecuencia | Significado |
|---|---|
| `D`, `B` | Día natural, día laborable. |
| `W`, `W-MON` | Semana (domingo), semana (lunes). |
| `ME`, `MS` | Fin, inicio de mes. |
| `QE`, `YE` | Fin de trimestre, de año. |
| `h`, `min`, `s` | Hora, minuto, segundo (en minúscula desde pandas 2.2). |

Zonas horarias:

```python
df["fecha"] = df["fecha"].dt.tz_localize("Europe/Madrid").dt.tz_convert("UTC")
```

---

## 14. Ordenar y clasificar

```python
df.sort_values("importe", ascending=False)
df.sort_values(["categoria", "precio"], ascending=[True, False])
df.sort_index()
df["ranking"] = df["importe"].rank(ascending=False, method="dense").astype(int)
df["ranking_cat"] = df.groupby("categoria")["importe"].rank(ascending=False)
```

---

## 15. Agrupar y agregar (`groupby`)

```python
df.groupby("categoria")["importe"].sum()
df.groupby(["region", "categoria"])["importe"].agg(["sum", "mean", "count"])

# Agregación con nombres (recomendada)
resumen = df.groupby("cliente").agg(
    pedidos=("pedido_id", "nunique"),
    unidades=("cantidad", "sum"),
    facturado=("importe", "sum"),
    ticket_medio=("importe", "mean"),
    ultima_compra=("fecha", "max"),
).sort_values("facturado", ascending=False)

# transform: devuelve un valor por fila (mismo tamaño que el original)
df["total_cliente"] = df.groupby("cliente")["importe"].transform("sum")
df["pct_cliente"] = df["importe"] / df["total_cliente"]

# filter: conservar grupos que cumplan una condición
df.groupby("cliente").filter(lambda g: g["importe"].sum() > 1000)

# Primeras filas de cada grupo
df.sort_values("importe", ascending=False).groupby("categoria").head(3)

# Recorrer grupos
for categoria, grupo in df.groupby("categoria"):
    print(categoria, len(grupo))
```

Funciones de agregación: `sum`, `mean`, `median`, `min`, `max`, `count` (sin nulos), `size` (con nulos), `nunique`, `std`, `var`, `first`, `last`, `quantile`, o cualquier función propia.

> Use `observed=True` al agrupar columnas `category` para no generar combinaciones vacías, y `as_index=False` para obtener las claves como columnas.

---

## 16. Tablas dinámicas y reestructuración

```python
tabla = pd.pivot_table(
    df, index="region", columns="categoria", values="importe",
    aggfunc="sum", fill_value=0, margins=True, margins_name="Total",
)

pd.crosstab(df["region"], df["canal"], normalize="index")      # Tabla de frecuencias / proporciones

# Ancho ↔ largo
largo = tabla.reset_index().melt(id_vars="region", var_name="categoria", value_name="importe")
ancho = largo.pivot(index="region", columns="categoria", values="importe")

df.set_index(["region", "categoria"])["importe"].unstack()      # Índice múltiple → columnas
tabla.stack()                                                    # Columnas → índice múltiple
df.explode("etiquetas")                                         # Lista en una celda → una fila por elemento
```

---

## 17. Combinar datos: `merge` y `concat`

```python
ventas.merge(clientes, on="cliente_id", how="left")             # Como un JOIN de SQL
ventas.merge(productos, left_on="cod", right_on="codigo", how="inner", suffixes=("_venta", "_prod"))
ventas.merge(clientes, on="cliente_id", how="left", validate="many_to_one", indicator=True)
```

| `how` | Resultado |
|---|---|
| `inner` | Solo coincidencias. |
| `left` | Todas las filas de la izquierda. |
| `right` | Todas las de la derecha. |
| `outer` | Todas. |
| `cross` | Producto cartesiano. |

```python
pd.concat([enero, febrero, marzo], ignore_index=True)          # Apilar filas
pd.concat([df1, df2], axis=1)                                   # Pegar columnas por índice
pd.concat(pd.read_csv(f) for f in Path("datos").glob("*.csv"))  # Unir muchos archivos
```

> `validate=` detecta claves duplicadas inesperadas y `indicator=True` añade la columna `_merge` para ver qué filas no han cruzado.

Unión por proximidad de fechas: `pd.merge_asof(ventas, tipos_cambio, on="fecha", direction="backward")`.

---

## 18. Ventanas móviles y acumulados

```python
serie.rolling(7).mean()                  # Media móvil de 7 periodos
serie.rolling("30D").sum()               # Ventana de 30 días (índice de fechas)
serie.expanding().max()                  # Máximo histórico
serie.ewm(span=10).mean()                # Media exponencial
df["acumulado"] = df.groupby("cliente")["importe"].cumsum()
df["variacion"] = df.groupby("cliente")["importe"].pct_change()
```

---

## 19. Gráficos rápidos

pandas integra Matplotlib para gráficos exploratorios:

```python
df.plot(x="fecha", y="importe", kind="line", title="Ventas")
df["categoria"].value_counts().plot.bar()
df["precio"].plot.hist(bins=30)
tabla.plot.barh(stacked=True, figsize=(10, 6))
df.plot.scatter(x="precio", y="unidades")
```

Para gráficos más elaborados, use Matplotlib, Seaborn o Plotly directamente.

---

## 20. Rendimiento y memoria

- **Vectorice**: evite `iterrows()` y `apply(axis=1)` cuando exista una operación por columnas.
- Use `category` para textos con pocos valores distintos y tipos numéricos más pequeños (`downcast`).
- Lea solo las columnas necesarias (`usecols`) y guarde en Parquet.
- Use `pd.read_csv(..., engine="pyarrow")` y tipos Arrow (`dtype_backend="pyarrow"`) para acelerar la lectura.
- Para datos muy grandes, procese por bloques (`chunksize`) o pase a Polars/DuckDB.

```python
df["cantidad"] = pd.to_numeric(df["cantidad"], downcast="integer")
df = pd.read_csv("grande.csv", engine="pyarrow", dtype_backend="pyarrow")
```

Comparativa orientativa sobre 1 millón de filas:

| Operación | Tiempo relativo |
|---|---|
| `df["a"] * df["b"]` | 1× |
| `df.apply(lambda f: f["a"] * f["b"], axis=1)` | ~1000× más lento |
| Bucle con `iterrows()` | ~2000× más lento |

---

## 21. Cambios importantes en pandas 3.0

| Cambio | Consecuencia |
|---|---|
| ***Copy-on-Write* siempre activo** | Cada objeto derivado se comporta como una copia. La «asignación encadenada» `df["a"][0] = 1` **no modifica** `df` (lanza `ChainedAssignmentError`). Use `df.loc[0, "a"] = 1`. |
| **Tipo de texto `str` por defecto** | Las columnas de texto ya no son `object`, sino `str` (respaldado por PyArrow si está instalado). Compruebe con `pd.api.types.is_string_dtype()`. |
| Resolución de fechas por defecto en microsegundos | `datetime64[us]` en lugar de `[ns]` al crear fechas a partir de texto. |
| Eliminación de funciones obsoletas | `applymap` → `map`; `fillna(method="ffill")` → `ffill()`; frecuencias `"M"`, `"Q"`, `"Y"`, `"H"` → `"ME"`, `"QE"`, `"YE"`, `"h"`. |

> El código escrito siguiendo las recomendaciones de pandas 2.2 sin avisos de obsolescencia funciona, en general, sin cambios en pandas 3.

---

## 22. Ejemplo práctico completo

**Informe de ventas**: genera un conjunto de datos realista (pedidos con errores típicos: textos con espacios, mayúsculas inconsistentes, importes con coma decimal, fechas en distintos formatos, duplicados y nulos), lo limpia, lo cruza con un maestro de clientes, calcula indicadores, rankings, evolución mensual con medias móviles, tabla dinámica por región y categoría, segmentación de clientes (RFM simplificado) y exporta un Excel de varias hojas y un Parquet.

```python
"""
informe_ventas.py — Limpieza, análisis y exportación de ventas con pandas.
Requiere: pip install pandas numpy openpyxl pyarrow
Uso: python informe_ventas.py [--salida informe_ventas.xlsx]
"""
import argparse
from pathlib import Path

import numpy as np
import pandas as pd

pd.set_option("display.width", 120)
pd.set_option("display.max_columns", 20)


# ------------------------------------------------------------------ Datos de ejemplo (con errores típicos)
def generar_datos(n: int = 3000, semilla: int = 42) -> tuple[pd.DataFrame, pd.DataFrame]:
    rng = np.random.default_rng(semilla)
    clientes = pd.DataFrame({
        "cliente_id": range(1, 41),
        "nombre": [f"Cliente {i:02d}" for i in range(1, 41)],
        "region": rng.choice(["Norte", "Sur", "Este", "Oeste", "Centro"], 40),
        "alta": pd.to_datetime("2024-01-01") + pd.to_timedelta(rng.integers(0, 700, 40), unit="D"),
    })
    categorias = {"Periféricos": (15, 120), "Monitores": (120, 450), "Almacenamiento": (40, 200),
                  "Redes": (25, 180), "Accesorios": (5, 60)}
    cat = rng.choice(list(categorias), n, p=[0.3, 0.15, 0.2, 0.15, 0.2])
    precio = np.array([rng.uniform(*categorias[c]) for c in cat]).round(2)
    fechas = pd.to_datetime("2025-01-01") + pd.to_timedelta(rng.integers(0, 365 + 300, n), unit="D")
    ventas = pd.DataFrame({
        "pedido_id": np.arange(10_000, 10_000 + n),
        "fecha": fechas.strftime("%d/%m/%Y"),
        "cliente_id": rng.integers(1, 43, n),            # 41 y 42 no existen en el maestro
        "categoria": cat,
        "cantidad": rng.integers(1, 8, n).astype(float),
        "precio_unitario": [f"{p:.2f}".replace(".", ",") for p in precio],   # Texto con coma decimal
        "canal": rng.choice(["web", "Web ", "TIENDA", "tienda", "teléfono"], n),
    })
    # Errores habituales
    ventas.loc[rng.choice(n, 40, replace=False), "cantidad"] = np.nan
    ventas.loc[rng.choice(n, 15, replace=False), "precio_unitario"] = "N/D"
    ventas.loc[rng.choice(n, 10, replace=False), "fecha"] = "2025-13-45"      # Fecha imposible
    ventas.loc[:9, "categoria"] = " perifericos "
    ventas = pd.concat([ventas, ventas.sample(25, random_state=1)], ignore_index=True)   # Duplicados
    return ventas, clientes


# ------------------------------------------------------------------ Limpieza
def limpiar(ventas: pd.DataFrame) -> tuple[pd.DataFrame, dict]:
    registro = {"filas_iniciales": len(ventas)}
    df = ventas.drop_duplicates()
    registro["duplicados_eliminados"] = registro["filas_iniciales"] - len(df)

    df = df.assign(
        fecha=pd.to_datetime(df["fecha"], format="%d/%m/%Y", errors="coerce"),
        precio_unitario=pd.to_numeric(df["precio_unitario"].str.replace(",", ".", regex=False), errors="coerce"),
        canal=df["canal"].str.strip().str.lower().replace({"teléfono": "telefono"}),
        categoria=df["categoria"].str.strip().str.title().replace({"Perifericos": "Periféricos"}),
    )
    registro["fechas_no_validas"] = int(df["fecha"].isna().sum())
    registro["precios_no_validos"] = int(df["precio_unitario"].isna().sum())
    registro["cantidades_vacias"] = int(df["cantidad"].isna().sum())

    df = df.dropna(subset=["fecha", "precio_unitario"])
    mediana_cat = df.groupby("categoria")["cantidad"].transform("median")
    df = df.assign(cantidad=df["cantidad"].fillna(mediana_cat).astype("int64"),
                   categoria=df["categoria"].astype("category"),
                   canal=df["canal"].astype("category"))
    df["importe"] = (df["cantidad"] * df["precio_unitario"]).round(2)
    registro["filas_finales"] = len(df)
    return df, registro


# ------------------------------------------------------------------ Análisis
def analizar(df: pd.DataFrame, clientes: pd.DataFrame) -> dict[str, pd.DataFrame]:
    datos = df.merge(clientes, on="cliente_id", how="left", validate="many_to_one", indicator=True)
    huerfanos = datos.loc[datos["_merge"] == "left_only", "cliente_id"].unique()
    datos = datos.drop(columns="_merge").fillna({"nombre": "Sin ficha", "region": "Desconocida"})

    kpis = pd.DataFrame({
        "indicador": ["Pedidos", "Unidades", "Facturación (€)", "Ticket medio (€)", "Clientes activos",
                      "Clientes sin ficha"],
        "valor": [datos["pedido_id"].nunique(), datos["cantidad"].sum(), datos["importe"].sum().round(2),
                  datos["importe"].mean().round(2), datos["cliente_id"].nunique(), len(huerfanos)],
    })

    por_categoria = (datos.groupby("categoria", observed=True)
                     .agg(pedidos=("pedido_id", "count"), unidades=("cantidad", "sum"),
                          facturacion=("importe", "sum"), precio_medio=("precio_unitario", "mean"))
                     .assign(cuota=lambda d: d["facturacion"] / d["facturacion"].sum())
                     .sort_values("facturacion", ascending=False).round(3))

    mensual = (datos.set_index("fecha").resample("ME")["importe"].agg(["sum", "count"])
               .rename(columns={"sum": "facturacion", "count": "pedidos"}))
    mensual["media_movil_3m"] = mensual["facturacion"].rolling(3).mean()
    mensual["variacion_mensual"] = mensual["facturacion"].pct_change()
    mensual["acumulado_anual"] = mensual.groupby(mensual.index.year)["facturacion"].cumsum()
    mensual.index = mensual.index.strftime("%Y-%m")

    dinamica = pd.pivot_table(datos, index="region", columns="categoria", values="importe",
                              aggfunc="sum", fill_value=0, margins=True, margins_name="Total",
                              observed=True).round(0)

    canales = pd.crosstab(datos["region"], datos["canal"], values=datos["importe"], aggfunc="sum",
                          normalize="index").round(3)

    # Segmentación RFM simplificada: recencia, frecuencia, valor monetario
    fecha_ref = datos["fecha"].max() + pd.Timedelta(days=1)
    rfm = datos.groupby(["cliente_id", "nombre", "region"]).agg(
        recencia_dias=("fecha", lambda f: (fecha_ref - f.max()).days),
        frecuencia=("pedido_id", "nunique"),
        valor=("importe", "sum"),
    ).reset_index()
    rfm["R"] = pd.qcut(rfm["recencia_dias"].rank(method="first"), 4, labels=[4, 3, 2, 1]).astype(int)
    rfm["F"] = pd.qcut(rfm["frecuencia"].rank(method="first"), 4, labels=[1, 2, 3, 4]).astype(int)
    rfm["M"] = pd.qcut(rfm["valor"].rank(method="first"), 4, labels=[1, 2, 3, 4]).astype(int)
    rfm["puntuacion"] = rfm[["R", "F", "M"]].sum(axis=1)
    rfm["segmento"] = pd.cut(rfm["puntuacion"], bins=[0, 6, 9, 12],
                             labels=["En riesgo", "Habituales", "Mejores clientes"])
    rfm = rfm.sort_values(["puntuacion", "valor"], ascending=False)
    rfm["ranking"] = range(1, len(rfm) + 1)

    top_productos_region = (datos.groupby(["region", "categoria"], observed=True)["importe"].sum()
                            .reset_index().sort_values("importe", ascending=False)
                            .groupby("region").head(2).sort_values(["region", "importe"],
                                                                   ascending=[True, False]))
    return {"KPIs": kpis, "Categorías": por_categoria, "Mensual": mensual, "Región x categoría": dinamica,
            "Canal por región": canales, "Clientes RFM": rfm, "Top por región": top_productos_region,
            "_datos": datos}


# ------------------------------------------------------------------ Exportación
def exportar(resultados: dict[str, pd.DataFrame], registro: dict, ruta: Path) -> None:
    with pd.ExcelWriter(ruta, engine="openpyxl") as escritor:
        for nombre, tabla in resultados.items():
            if nombre.startswith("_"):
                continue
            indice = not isinstance(tabla.index, pd.RangeIndex)
            tabla.to_excel(escritor, sheet_name=nombre[:31], index=indice)
        pd.Series(registro, name="valor").to_frame().to_excel(escritor, sheet_name="Calidad de datos")
        for hoja in escritor.book.worksheets:                       # Ancho de columnas aproximado
            for columna in hoja.columns:
                ancho = max(len(str(c.value)) if c.value is not None else 0 for c in columna)
                hoja.column_dimensions[columna[0].column_letter].width = min(ancho + 2, 40)
    resultados["_datos"].to_parquet(ruta.with_suffix(".parquet"), index=False)


def main() -> int:
    p = argparse.ArgumentParser(description="Informe de ventas con pandas.")
    p.add_argument("--salida", type=Path, default=Path("informe_ventas.xlsx"))
    args = p.parse_args()

    ventas, clientes = generar_datos()
    ventas.to_csv("ventas_brutas.csv", sep=";", index=False, encoding="utf-8-sig")   # Para inspeccionarlas
    df, registro = limpiar(ventas)
    resultados = analizar(df, clientes)
    exportar(resultados, registro, args.salida)

    print("Calidad de datos:", registro, "\n")
    print(resultados["KPIs"].to_string(index=False), "\n")
    print(resultados["Categorías"], "\n")
    print(resultados["Mensual"].tail(4), "\n")
    print(resultados["Clientes RFM"]["segmento"].value_counts(), "\n")
    print(f"✔ Informe guardado en {args.salida} y datos limpios en {args.salida.with_suffix('.parquet')}")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 23. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `KeyError: 'columna'` | Nombre incorrecto o con espacios ocultos. | `df.columns.tolist()`; `df.columns = df.columns.str.strip()`. |
| `ChainedAssignmentError` / `SettingWithCopyWarning` | Asignación encadenada `df[df.a > 0]["b"] = 1`. | `df.loc[df.a > 0, "b"] = 1`. |
| `ValueError: The truth value of a Series is ambiguous` | `and`/`or` con `Series` o `if serie:`. | `&`, `|`, `~` con paréntesis; `.any()`, `.all()`. |
| `TypeError: can only concatenate str…` / sumas que concatenan | Columna numérica leída como texto. | `pd.to_numeric(..., errors="coerce")`, `decimal=","`. |
| Fechas mal interpretadas (mes y día cambiados) | Formato ambiguo. | `format="%d/%m/%Y"` o `dayfirst=True`. |
| `UnicodeDecodeError` al leer CSV | Codificación distinta de UTF-8. | `encoding="latin-1"` o `"cp1252"`. |
| Se pierden los ceros iniciales | Código leído como número. | `dtype={"codigo": "str"}`. |
| `merge` multiplica filas | Clave duplicada en ambas tablas. | `validate="many_to_one"`; revise duplicados. |
| `ValueError: Invalid frequency: M` | Alias de frecuencia antiguos. | `"ME"`, `"QE"`, `"YE"`, `"h"`. |
| `AttributeError: 'DataFrame' object has no attribute 'applymap'` | Eliminado en pandas 3. | `df.map(func)`. |
| `MemoryError` | Datos demasiado grandes. | `usecols`, `chunksize`, tipos más pequeños, Parquet, Polars/DuckDB. |
| Excel con caracteres extraños al abrir un CSV | Falta el BOM de UTF-8. | `encoding="utf-8-sig"`. |

---

## 24. Buenas prácticas

- Inspeccione los datos al cargarlos (`info()`, `describe()`, `isna().sum()`, `value_counts()`).
- Fije los tipos al leer (`dtype`, `parse_dates`, `decimal`) en lugar de corregirlos después.
- Use operaciones vectorizadas; reserve `apply` para casos sin alternativa.
- Modifique valores con `loc` en una sola operación (compatible con *Copy-on-Write*).
- Encadene transformaciones legibles con `assign`, `pipe` y paréntesis.
- Use agregaciones con nombre en `groupby` y `validate=` en `merge`.
- Documente la limpieza (qué se elimina y por qué) y registre cuántas filas se pierden.
- Guarde los datos intermedios en Parquet y los informes en Excel/CSV.
- Separe en funciones las fases de carga, limpieza, análisis y exportación.

---

## 25. Referencia rápida

```python
pd.read_csv(r, sep=";", decimal=",", parse_dates=[], dtype={}, usecols=[]) · read_excel · read_parquet · read_sql · read_json
df.to_csv(index=False) · to_excel(writer, sheet_name=) · to_parquet · to_sql · to_json(orient="records")
df.head() · info() · describe() · shape · dtypes · value_counts() · isna().sum() · nunique()
df["c"] · df[["a","b"]] · df.loc[filas, cols] · df.iloc[i, j] · df.query("a > 1 and b in @lista")
df[(c1) & (c2) | ~(c3)] · isin() · between() · str.contains() · nlargest()
df.assign(x=lambda d: ...) · np.where · np.select · pd.cut · pd.qcut · map · replace · astype · to_numeric · to_datetime
dropna · fillna · ffill · drop_duplicates · rename · drop(columns=) · sort_values · rank
df.groupby(k).agg(nombre=("col", "func")) · transform · filter · pivot_table · crosstab · melt · pivot · stack/unstack
merge(on=, how=, validate=) · concat · merge_asof · resample("ME") · rolling(7) · shift · pct_change · cumsum
s.str.* · s.dt.* · df.plot.*
```

---

## 26. Ejercicios propuestos

1. Cargue un CSV de gastos personales, limpie las categorías y calcule el gasto por mes y categoría con una tabla dinámica.
2. Combine dos archivos (pedidos y clientes) y encuentre los clientes que no han comprado en los últimos 90 días.
3. A partir del ejemplo completo, calcule la facturación por día de la semana y por hora (genere también una hora aleatoria).
4. Detecte valores atípicos de importe por categoría con el rango intercuartílico y expórtelos a una hoja aparte.
5. Compare el tiempo de calcular una columna con `apply(axis=1)` frente a la versión vectorizada sobre un millón de filas.

---

## 27. Referencias

- Documentación oficial: https://pandas.pydata.org/docs/
- Guía de usuario: https://pandas.pydata.org/docs/user_guide/index.html
- Novedades de pandas 3.0: https://pandas.pydata.org/docs/whatsnew/
- *Copy-on-Write*: https://pandas.pydata.org/docs/user_guide/copy_on_write.html
- Hoja de referencia oficial: https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf
