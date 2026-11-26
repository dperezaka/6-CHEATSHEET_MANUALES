# Manual de uso: Dask

> **Categoría:** Ciencia de datos · **Utilidad:** Procesamiento paralelo y de grandes volúmenes de datos
> **Tipo:** Librería de terceros (`pip install "dask[complete]"`) · **Versión de referencia:** 2024.x o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Dask DataFrame: crear y leer datos](#4-dask-dataframe-crear-y-leer-datos)
5. [Operaciones con DataFrames](#5-operaciones-con-dataframes)
6. [Particiones e índice](#6-particiones-e-índice)
7. [`map_partitions` y el parámetro `meta`](#7-map_partitions-y-el-parámetro-meta)
8. [Ejecutar: `compute`, `persist` y cálculo compartido](#8-ejecutar-compute-persist-y-cálculo-compartido)
9. [Escritura de resultados](#9-escritura-de-resultados)
10. [Planificadores (*schedulers*)](#10-planificadores-schedulers)
11. [Dask Distributed: clúster local y panel de control](#11-dask-distributed-clúster-local-y-panel-de-control)
12. [`dask.delayed`: paralelizar código propio](#12-daskdelayed-paralelizar-código-propio)
13. [Futures: tareas en tiempo real](#13-futures-tareas-en-tiempo-real)
14. [Dask Array](#14-dask-array)
15. [Dask Bag: datos semiestructurados](#15-dask-bag-datos-semiestructurados)
16. [Diagnóstico y rendimiento](#16-diagnóstico-y-rendimiento)
17. [Despliegue en clústeres y la nube](#17-despliegue-en-clústeres-y-la-nube)
18. [Integración con el ecosistema](#18-integración-con-el-ecosistema)
19. [Ejemplo práctico completo](#19-ejemplo-práctico-completo)
20. [Errores frecuentes](#20-errores-frecuentes)
21. [Buenas prácticas](#21-buenas-prácticas)
22. [Referencia rápida](#22-referencia-rápida)
23. [Ejercicios propuestos](#23-ejercicios-propuestos)
24. [Referencias](#24-referencias)

---

## 1. Descripción

**Dask** escala el ecosistema de Python (pandas, NumPy, scikit-learn) a **datos mayores que la memoria** y a **varios núcleos o máquinas**. Divide los datos en trozos (particiones o bloques), construye un **grafo de tareas** de forma diferida y lo ejecuta en paralelo con un planificador, desde un portátil hasta un clúster de cientos de nodos.

**Ventajas:**
- API casi idéntica a pandas y NumPy: curva de aprendizaje mínima.
- Procesa datos que no caben en memoria trabajando por partes.
- Escala del portátil al clúster con el mismo código.
- Paraleliza código Python arbitrario (`delayed`, *futures*).
- Panel de control en tiempo real para entender el rendimiento.
- Optimizador de consultas en Dask DataFrame (desde 2024).

**Limitaciones:**
- Sobrecarga de planificación: con datos que caben holgadamente en memoria, pandas o Polars suelen ser más rápidos.
- No implementa toda la API de pandas (operaciones que requieren orden global o acceso posicional).
- Operaciones con reorganización de datos (*shuffle*: `set_index`, `merge` de tablas grandes, `groupby().apply`) son costosas.
- Depurar errores en código diferido es más difícil.

| Si necesita… | Opción |
|---|---|
| pandas/NumPy con datos mayores que la memoria o en clúster | **Dask** |
| Máxima velocidad en un equipo con datos hasta decenas de GB | Polars, DuckDB |
| Grandes plataformas de datos empresariales (JVM, SQL distribuido) | Apache Spark |
| Paralelismo genérico de tareas y aprendizaje automático distribuido | Ray |
| Paralelizar un bucle sencillo en un equipo | `concurrent.futures`, joblib |

---

## 2. Instalación e importación

```bash
pip install "dask[complete]"           # DataFrame, Array, Bag, Distributed, diagnóstico
pip install "dask[dataframe]"          # Solo DataFrame (con pandas y PyArrow)
pip install "dask[distributed]"        # Planificador distribuido y panel de control
```

```python
import dask
import dask.dataframe as dd
import dask.array as da
import dask.bag as db
from dask import delayed
from dask.distributed import Client, LocalCluster
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Colección | `dd.DataFrame`, `da.Array`, `db.Bag`: «envoltorios» de muchas piezas pandas/NumPy/listas. |
| Partición / bloque (*chunk*) | Cada trozo, procesado por una tarea. |
| Evaluación diferida | Las operaciones construyen un plan; nada se calcula hasta `compute()`. |
| Grafo de tareas | Dependencias entre tareas que el planificador ejecuta en paralelo. |
| Planificador | Decide dónde y cuándo se ejecuta cada tarea (hilos, procesos o clúster). |
| `compute()` | Ejecuta y devuelve un objeto en memoria (pandas/NumPy). |
| `persist()` | Ejecuta y mantiene el resultado distribuido en memoria, sin traerlo. |

```python
ddf = dd.read_csv("ventas_*.csv")          # Instantáneo: solo lee metadatos
total = ddf["importe"].sum()               # Instantáneo: añade tareas al plan
total.compute()                            # Aquí se lee y calcula todo, en paralelo
```

---

## 4. Dask DataFrame: crear y leer datos

```python
ddf = dd.read_csv("logs/2026-*.csv", sep=";", dtype={"cp": "string", "codigo": "int32"},
                  parse_dates=["fecha"], blocksize="64MB", assume_missing=True)
ddf = dd.read_parquet("datos/ventas/", columns=["fecha", "region", "importe"],
                      filters=[("anio", "==", 2026), ("importe", ">", 0)])      # Lee solo lo necesario
ddf = dd.read_parquet("s3://bucket/ventas/*.parquet", storage_options={"anon": False})
ddf = dd.read_json("eventos/*.jsonl", lines=True, blocksize="32MB")
ddf = dd.read_sql_table("pedidos", "postgresql://usuario:clave@servidor/bd", index_col="id", npartitions=20)
ddf = dd.from_pandas(df_pandas, npartitions=8)
ddf = dd.from_delayed([cargar(ruta) for ruta in rutas], meta=esquema)       # Lectores propios

ddf.npartitions ; ddf.columns ; ddf.dtypes
ddf.head()                                  # Calcula solo la primera partición
len(ddf)                                    # Recorre todos los datos
```

> Indique `dtype` al leer CSV: Dask infiere los tipos con una muestra del primer bloque y falla si más adelante aparecen valores distintos.

---

## 5. Operaciones con DataFrames

La mayoría de operaciones de pandas funcionan igual:

```python
ddf = ddf[ddf["estado"] != "cancelado"]
ddf = ddf.assign(importe=ddf["unidades"] * ddf["precio"], mes=ddf["fecha"].dt.to_period("M").astype(str))
ddf["region"] = ddf["region"].str.strip().str.title()
ddf = ddf.dropna(subset=["cliente"]).fillna({"descuento": 0})

ddf.groupby("region")["importe"].agg(["sum", "mean", "count"])
ddf.groupby(["region", "mes"]).agg(ventas=("importe", "sum"), pedidos=("id", "count"))
ddf.groupby("region")["importe"].median()                       # Requiere reorganizar datos (más costoso)
ddf["cliente"].nunique() ; ddf["importe"].quantile([0.5, 0.95]) ; ddf.nlargest(10, "importe")
ddf["canal"].value_counts()
ddf.describe()

dd.merge(ddf, clientes_pandas, on="cliente_id", how="left")    # Tabla pequeña: unión por difusión (rápida)
dd.merge(ddf, otro_ddf, on="cliente_id")                        # Dos tablas grandes: reorganización
dd.concat([ddf_2025, ddf_2026])
ddf.categorize(columns=["region"]).pivot_table(index="mes", columns="region", values="importe", aggfunc="sum")
```

Agregaciones personalizadas eficientes (por fragmentos):

```python
rango = dd.Aggregation(
    name="rango",
    chunk=lambda s: (s.max(), s.min()),                 # En cada partición
    agg=lambda maximos, minimos: (maximos.max(), minimos.min()),   # Combina particiones
    finalize=lambda maximo, minimo: maximo - minimo,
)
ddf.groupby("region")["importe"].agg(rango)
```

---

## 6. Particiones e índice

```python
ddf = ddf.repartition(npartitions=32)
ddf = ddf.repartition(partition_size="100MB")         # Objetivo: 100–300 MB por partición en memoria
ddf.map_partitions(len).compute()                     # Filas por partición

ddf = ddf.set_index("fecha", sorted=True)             # sorted=True si ya está ordenado: evita reorganizar
ddf.divisions                                          # Límites del índice de cada partición
ddf.loc["2026-03-01":"2026-03-31"]                     # Selección eficiente: solo las particiones necesarias
ddf["importe"].resample("1D").sum()                    # Requiere índice temporal ordenado
ddf.get_partition(0).compute()
```

| Tamaño de partición | Problema |
|---|---|
| Muy pequeño (< 10 MB) | Demasiadas tareas: la planificación domina el tiempo. |
| Muy grande (> 1 GB) | Riesgo de quedarse sin memoria; menos paralelismo. |

---

## 7. `map_partitions` y el parámetro `meta`

Aplica una función de pandas a cada partición. Es la vía para usar cualquier código de pandas:

```python
def enriquecer(df: pd.DataFrame) -> pd.DataFrame:
    df = df.copy()
    df["dispositivo"] = np.where(df["agente"].str.contains("Mobile"), "Móvil", "Escritorio")
    df["lenta"] = df["latencia_ms"] > 800
    return df

meta = ddf._meta.assign(dispositivo=pd.Series(dtype="string"), lenta=pd.Series(dtype="bool"))
ddf = ddf.map_partitions(enriquecer, meta=meta)
```

`meta` describe el resultado (columnas y tipos) sin ejecutarlo. Si se omite, Dask ejecuta la función sobre datos ficticios para adivinarlo, lo que puede fallar o producir tipos erróneos.

```python
ddf.groupby("endpoint")["latencia_ms"].apply(lambda s: s.quantile(0.95), meta=("latencia_ms", "float64"))
```

---

## 8. Ejecutar: `compute`, `persist` y cálculo compartido

```python
resultado = ddf.groupby("region")["importe"].sum().compute()     # → pandas.Series

# Varios resultados en una sola pasada (comparten lectura y pasos intermedios)
total, por_region, maximo = dask.compute(ddf["importe"].sum(),
                                         ddf.groupby("region")["importe"].sum(),
                                         ddf["importe"].max())

limpio = ddf[ddf["importe"] > 0].persist()     # Calcula una vez y conserva en memoria para varios análisis
```

> No llame a `compute()` repetidamente dentro de bucles: cada llamada relee los datos. Construya todo el plan y calcule una vez (o use `persist`).

---

## 9. Escritura de resultados

```python
ddf.to_parquet("salida/ventas/", partition_on=["anio", "mes"], write_index=False,
               compression="zstd", overwrite=True)                 # Un archivo por partición
ddf.to_csv("salida/csv/ventas-*.csv", index=False)                  # «*» se sustituye por el número de partición
ddf.to_csv("salida/ventas.csv", single_file=True, index=False)
ddf.to_sql("pedidos", uri, if_exists="append", chunksize=10_000)
resumen.compute().to_excel("resumen.xlsx")                          # Resultados pequeños: a pandas y escribir
```

---

## 10. Planificadores (*schedulers*)

| Planificador | Cuándo usarlo |
|---|---|
| `"threads"` (defecto en DataFrame y Array) | Código que libera el GIL: pandas, NumPy, PyArrow. |
| `"processes"` (defecto en Bag) | Código Python puro (texto, diccionarios) limitado por el GIL. |
| `"synchronous"` | Depuración: un solo hilo, permite `pdb` y trazas claras. |
| `distributed` (`Client`) | Recomendado incluso en un equipo: panel de control, memoria controlada, *futures*. |

```python
total.compute(scheduler="synchronous")                  # Depurar
dask.config.set(scheduler="processes")                  # Global
with dask.config.set(scheduler="threads", num_workers=4):
    total.compute()
```

> Con el planificador de procesos o `LocalCluster`, proteja el punto de entrada del script con `if __name__ == "__main__":` (obligatorio en Windows y macOS).

---

## 11. Dask Distributed: clúster local y panel de control

```python
from dask.distributed import Client, LocalCluster

if __name__ == "__main__":
    cluster = LocalCluster(n_workers=4, threads_per_worker=2, memory_limit="4GB")
    cliente = Client(cluster)                   # A partir de aquí, compute() usa el clúster
    print(cliente.dashboard_link)               # http://127.0.0.1:8787/status
    ...
    cliente.close(); cluster.close()

with LocalCluster(n_workers=4) as cluster, Client(cluster) as cliente:   # Cierre automático
    ...

cliente = Client("tcp://planificador:8786")    # Conectarse a un clúster existente
cliente.upload_file("utilidades.py")           # Enviar módulos propios a los trabajadores
cliente.restart()                              # Liberar memoria de los trabajadores
```

El **panel de control** muestra el progreso de las tareas, el uso de memoria de cada trabajador y el flujo de datos: es la herramienta principal para detectar cuellos de botella.

```python
from dask.distributed import performance_report
with performance_report(filename="informe_dask.html"):
    resultado.compute()                         # Guarda el panel en un HTML para analizarlo después
```

---

## 12. `dask.delayed`: paralelizar código propio

Convierte funciones normales en tareas diferidas para paralelizar flujos que no encajan en DataFrame o Array:

```python
from dask import delayed
import dask

@delayed
def cargar(ruta): ...
@delayed
def limpiar(datos): ...
@delayed
def resumir(lista): ...

limpios = [limpiar(cargar(r)) for r in rutas]     # Nada se ejecuta todavía
informe = resumir(limpios)
informe.visualize("grafo.png")                     # Dibujar el grafo (requiere graphviz)
resultado = informe.compute()                      # Carga y limpieza en paralelo, luego el resumen

resultados = dask.compute(*[procesar(x) for x in elementos])   # Lista de tareas independientes
```

> No llame a funciones `delayed` dentro de otras funciones `delayed` ni haga `compute()` dentro de ellas: construya el grafo completo desde fuera.

---

## 13. Futures: tareas en tiempo real

Con `distributed`, las tareas empiezan inmediatamente y se puede reaccionar a medida que terminan:

```python
from dask.distributed import as_completed

futuros = [cliente.submit(simular, parametros, pure=False) for parametros in combinaciones]
for futuro in as_completed(futuros):
    resultado = futuro.result()
    if resultado["error"] < mejor:
        mejor = resultado["error"]

resultados = cliente.gather(cliente.map(procesar, elementos))
datos_grandes = cliente.scatter(tabla_referencia, broadcast=True)    # Enviar una vez a todos los trabajadores
```

---

## 14. Dask Array

Matrices de NumPy divididas en bloques:

```python
x = da.random.default_rng(0).normal(size=(200_000, 1_000), chunks=(10_000, 1_000))
x.mean(axis=0).compute()
(x.T @ x).compute()                                   # Producto matricial por bloques
x = da.from_zarr("datos.zarr") ; da.to_zarr(x, "salida.zarr")      # Almacenamiento por bloques
x = da.from_array(np.memmap("enorme.dat", dtype="float32", shape=(10**6, 500)), chunks=(50_000, 500))
conteos, bordes = da.histogram(valores, bins=50, range=(0, 5000))
x.rechunk((50_000, 100))                              # Cambiar la forma de los bloques
```

Para imágenes, datos climáticos o científicos multidimensionales con etiquetas, **xarray** usa Dask Array internamente.

---

## 15. Dask Bag: datos semiestructurados

Colecciones de objetos Python (JSON, líneas de texto), al estilo de operaciones *map/filter/reduce*:

```python
import json
eventos = db.read_text("eventos/*.jsonl").map(json.loads)
compras = eventos.filter(lambda e: e["tipo"] == "compra")
compras.pluck("importe").sum().compute()
compras.map(lambda e: (e["pais"], e["importe"])).foldby(lambda t: t[0], lambda acc, t: acc + t[1], 0).compute()
compras.frequencies().topk(10, key=1)
ddf = compras.map(lambda e: {"pais": e["pais"], "importe": e["importe"]}).to_dataframe()
```

---

## 16. Diagnóstico y rendimiento

```python
from dask.diagnostics import ProgressBar, Profiler, ResourceProfiler
with ProgressBar():                         # Barra de progreso (planificadores locales)
    resultado.compute()

expresion = ddf.groupby("region")["importe"].sum()
expresion.pprint()                          # Plan de la consulta
expresion.optimize().pprint()               # Plan optimizado (columnas y filtros llevados a la lectura)
expresion.explain()                         # Representación gráfica (requiere graphviz)
```

Recomendaciones de rendimiento:

| Problema | Acción |
|---|---|
| Lectura lenta | Parquet en lugar de CSV; seleccionar columnas y filtros al leer. |
| Demasiadas tareas | Particiones mayores (`repartition`), menos archivos pequeños. |
| Memoria agotada | Particiones menores, `persist` solo lo necesario, más trabajadores o memoria. |
| Reorganizaciones (*shuffle*) costosas | Evitar `set_index` innecesarios; unir con tablas pequeñas en pandas. |
| Código Python puro lento con hilos | Planificador de procesos o `distributed`. |

---

## 17. Despliegue en clústeres y la nube

| Entorno | Paquete |
|---|---|
| Kubernetes | `dask-kubernetes` |
| HPC (SLURM, PBS, SGE) | `dask-jobqueue` |
| Hadoop/YARN | `dask-yarn` |
| AWS, GCP, Azure | `dask-cloudprovider`, Coiled (servicio gestionado) |
| Manual | `dask scheduler` y `dask worker tcp://planificador:8786` |

El código de análisis no cambia: solo cambia cómo se crea el `Client`.

---

## 18. Integración con el ecosistema

```python
import dask_ml                                              # pip install dask-ml
from dask_ml.model_selection import GridSearchCV            # Búsqueda de hiperparámetros distribuida
from dask_ml.preprocessing import StandardScaler

import joblib
with joblib.parallel_backend("dask"):                       # scikit-learn usa el clúster de Dask
    modelo.fit(X, y)

import xgboost as xgb
xgb.dask.DaskXGBRegressor().fit(X_dask, y_dask)            # Entrenamiento distribuido

pdf = ddf.compute() ; ddf = dd.from_pandas(pdf, npartitions=8)
pl_df = pl.from_pandas(resultado.compute())                 # Resultado agregado a Polars
```

---

## 19. Ejemplo práctico completo

**Análisis de registros de acceso de un servicio web** (2,4 millones de peticiones en 30 archivos diarios), un caso típico en el que los datos crecen día a día y dejan de caber cómodamente en memoria:

1. Genera los registros en CSV (uno por día) y un archivo JSON-lines de despliegues.
2. **Ingesta**: lectura con tipos declarados, limpieza y enriquecimiento con `map_partitions`, conversión a **Parquet particionado por día**.
3. **Análisis** sobre Parquet leyendo solo columnas necesarias: tráfico por hora, tasa de errores y latencias p50/p95 por *endpoint*, IP con más peticiones, histograma de latencias con Dask Array; todo calculado con una sola llamada a `dask.compute`.
4. **Correlación con despliegues** usando `dask.delayed` para analizar cada despliegue en paralelo.
5. Se ejecuta con un `LocalCluster` (opción `--cluster`) o con el planificador de hilos, y compara el tiempo con pandas.

```python
"""
registros_web.py — Ingesta y análisis de registros de acceso con Dask.
Requiere: pip install "dask[complete]" pandas pyarrow
Uso: python registros_web.py [--cluster]
"""
from __future__ import annotations

import argparse
import json
import logging
import time
from datetime import datetime, timedelta
from pathlib import Path

import dask
import dask.array as da
import dask.dataframe as dd
import numpy as np
import pandas as pd
from dask import delayed

ORIGEN = Path("registros_csv")
PARQUET = Path("registros_parquet")
DESPLIEGUES = Path("despliegues.jsonl")
DIAS = 30
PETICIONES_DIA = 80_000
ENDPOINTS = {"/api/productos": 45, "/api/carrito": 70, "/api/pago": 220, "/api/buscar": 160,
             "/api/usuario": 55, "/": 30, "/api/recomendaciones": 380}
TIPOS = {"ip": "string", "metodo": "category", "endpoint": "string", "estado": "int16",
         "latencia_ms": "float32", "bytes": "int32", "agente": "string"}


# ============================================================ 1. Datos de ejemplo
def generar_datos() -> None:
    ORIGEN.mkdir(exist_ok=True)
    rng = np.random.default_rng(1)
    inicio = datetime(2026, 10, 1)
    ips = np.array([f"10.{rng.integers(0, 255)}.{rng.integers(0, 255)}.{rng.integers(1, 255)}" for _ in range(5000)])
    ips[:3] = ["203.0.113.7", "198.51.100.23", "192.0.2.99"]                      # Bots muy activos
    peso_ip = np.r_[[0.03, 0.02, 0.015], np.full(4997, (1 - 0.065) / 4997)]
    despliegues = []
    for d in range(DIAS):
        dia = inicio + timedelta(days=d)
        n = PETICIONES_DIA + int(rng.integers(-8000, 8000))
        perfil = np.r_[np.full(7, 0.2), np.full(5, 1.2), np.full(6, 1.4), np.full(6, 0.5)]   # Tráfico relativo por hora
        hora = rng.choice(24, n, p=perfil / perfil.sum())
        momentos = [dia + timedelta(hours=int(h), seconds=int(s)) for h, s in zip(hora, rng.integers(0, 3600, n))]
        endpoint = rng.choice(list(ENDPOINTS), n, p=[.28, .18, .06, .22, .12, .1, .04])
        base = np.array([ENDPOINTS[e] for e in endpoint])
        latencia = base * rng.lognormal(0, 0.45, n)
        estado = np.where(rng.random(n) < 0.012, 500, np.where(rng.random(n) < 0.03, 404, 200))
        if d in (11, 23):                                                     # Despliegues defectuosos
            mala_hora = 14 if d == 11 else 10
            afectado = (hora >= mala_hora) & (hora < mala_hora + 3) & (endpoint == "/api/pago")
            latencia[afectado] *= 3.5
            estado[afectado & (rng.random(n) < 0.18)] = 503
            despliegues.append({"momento": (dia + timedelta(hours=mala_hora)).isoformat(), "servicio": "pagos",
                                "version": f"2.{d}.0"})
        elif d % 4 == 0:
            despliegues.append({"momento": (dia + timedelta(hours=9)).isoformat(), "servicio": "catalogo",
                                "version": f"5.{d}.1"})
        df = pd.DataFrame({
            "momento": pd.to_datetime(momentos).strftime("%d/%m/%Y %H:%M:%S"),
            "ip": rng.choice(ips, n, p=peso_ip), "metodo": rng.choice(["GET", "POST"], n, p=[.8, .2]),
            "endpoint": endpoint, "estado": estado, "latencia_ms": latencia.round(1),
            "bytes": rng.integers(300, 60_000, n),
            "agente": rng.choice(["Mozilla/5.0 (Windows)", "Mozilla/5.0 (iPhone; Mobile)", "Mozilla/5.0 (Android; Mobile)",
                                  "python-requests/2.32", "Googlebot/2.1"], n, p=[.42, .3, .2, .05, .03]),
        })
        df.to_csv(ORIGEN / f"acceso_{dia:%Y%m%d}.csv", index=False, sep=";")
    DESPLIEGUES.write_text("\n".join(json.dumps(x) for x in despliegues), encoding="utf-8")


# ============================================================ 2. Ingesta: CSV → Parquet
def enriquecer(df: pd.DataFrame) -> pd.DataFrame:
    """Se aplica a cada partición (un DataFrame de pandas)."""
    df = df.copy()
    df["momento"] = pd.to_datetime(df["momento"], format="%d/%m/%Y %H:%M:%S")
    df["dia"] = df["momento"].dt.strftime("%Y-%m-%d")
    df["hora"] = df["momento"].dt.hour.astype("int8")
    df["error"] = df["estado"] >= 500
    agente = df["agente"].fillna("")
    df["dispositivo"] = np.select([agente.str.contains("bot|requests", case=False), agente.str.contains("Mobile")],
                                  ["Bot", "Móvil"], default="Escritorio")
    return df


def ingesta() -> None:
    crudo = dd.read_csv(ORIGEN / "acceso_*.csv", sep=";", dtype=TIPOS | {"momento": "string"}, blocksize="16MB")
    meta = enriquecer(crudo._meta)                     # Esquema del resultado sin ejecutar nada
    limpio = crudo.map_partitions(enriquecer, meta=meta)
    limpio.to_parquet(PARQUET, partition_on=["dia"], write_index=False, compression="zstd", overwrite=True)


# ============================================================ 3. Análisis
def p95(serie: pd.Series) -> float:
    return float(serie.quantile(0.95))


def analizar() -> dict:
    ddf = dd.read_parquet(PARQUET, columns=["momento", "hora", "endpoint", "estado", "latencia_ms",
                                            "error", "ip", "dispositivo", "bytes"])
    ddf = ddf.repartition(partition_size="64MB").persist()   # Agrupa los 30 archivos diarios y conserva en memoria

    por_endpoint = ddf.groupby("endpoint").agg(peticiones=("estado", "count"), errores=("error", "sum"),
                                                latencia_media=("latencia_ms", "mean"),
                                                latencia_p50=("latencia_ms", "median"))
    p95_endpoint = ddf.groupby("endpoint")["latencia_ms"].apply(p95, meta=("latencia_ms", "float64"))
    trafico_hora = ddf.groupby("hora")["estado"].count()
    dispositivos = ddf["dispositivo"].value_counts()
    top_ips = ddf["ip"].value_counts().nlargest(5)
    gb_servidos = ddf["bytes"].sum() / 1e9

    t0 = time.perf_counter()
    (por_endpoint, p95_endpoint, trafico_hora, dispositivos, top_ips, gb_servidos, total) = dask.compute(
        por_endpoint, p95_endpoint, trafico_hora, dispositivos, top_ips, gb_servidos, ddf.shape[0])
    segundos = time.perf_counter() - t0
    # Dask Array: histograma por bloques (se calcula aparte: no mezclar colecciones de distinto tipo en un compute)
    conteos, bordes = da.histogram(ddf["latencia_ms"].to_dask_array(), bins=40, range=(0, 2000))
    conteos = conteos.compute()

    por_endpoint["latencia_p95"] = p95_endpoint
    por_endpoint["tasa_error"] = por_endpoint["errores"] / por_endpoint["peticiones"]
    return {"ddf": ddf, "total": total, "segundos": segundos, "endpoints": por_endpoint.sort_values("peticiones", ascending=False),
            "horas": trafico_hora.sort_index(), "dispositivos": dispositivos, "top_ips": top_ips,
            "gb": gb_servidos, "histograma": (conteos, bordes)}


# ============================================================ 4. Despliegues con dask.delayed
@delayed
def impacto_despliegue(ddf_dia: pd.DataFrame, despliegue: dict) -> dict:
    momento = pd.Timestamp(despliegue["momento"])
    pago = ddf_dia[ddf_dia["endpoint"] == "/api/pago"]
    antes = pago[(pago["momento"] >= momento - pd.Timedelta(hours=3)) & (pago["momento"] < momento)]
    despues = pago[(pago["momento"] >= momento) & (pago["momento"] < momento + pd.Timedelta(hours=3))]
    return {"momento": momento.strftime("%d/%m %H:%M"), "servicio": despliegue["servicio"], "version": despliegue["version"],
            "p95_antes": antes["latencia_ms"].quantile(0.95), "p95_despues": despues["latencia_ms"].quantile(0.95),
            "error_antes": (antes["estado"] >= 500).mean(), "error_despues": (despues["estado"] >= 500).mean()}


def analizar_despliegues() -> pd.DataFrame:
    despliegues = [json.loads(l) for l in DESPLIEGUES.read_text(encoding="utf-8").splitlines()]
    tareas = []
    for d in despliegues:
        dia = d["momento"][:10]
        datos_dia = dd.read_parquet(PARQUET, columns=["momento", "endpoint", "latencia_ms", "estado"],
                                    filters=[("dia", "==", dia)]).to_delayed()       # Solo la partición de ese día
        tareas.append(impacto_despliegue(delayed(pd.concat)(datos_dia), d))
    resultado = pd.DataFrame(dask.compute(*tareas))
    resultado["alerta"] = (resultado["p95_despues"] > 2 * resultado["p95_antes"]) | (resultado["error_despues"] > 0.05)
    return resultado


# ============================================================ 5. Comparación con pandas
def comparar_con_pandas() -> float:
    t0 = time.perf_counter()
    df = pd.concat((pd.read_csv(r, sep=";", dtype=TIPOS) for r in sorted(ORIGEN.glob("acceso_*.csv"))), ignore_index=True)
    df.groupby("endpoint")["latencia_ms"].quantile(0.95)
    return time.perf_counter() - t0


def informe(r: dict, despliegues: pd.DataFrame, t_ingesta: float, t_pandas: float) -> None:
    num = lambda v, d=0: f"{v:,.{d}f}".replace(",", "X").replace(".", ",").replace("X", ".")
    print(f"\nPeticiones analizadas: {num(r['total'])} · datos servidos: {num(r['gb'], 2)} GB")
    print(f"Ingesta CSV→Parquet: {t_ingesta:.1f} s · análisis (una pasada con dask.compute): {r['segundos']:.2f} s")
    print(f"Referencia pandas (leer CSV + p95 por endpoint): {t_pandas:.1f} s")

    print("\nRendimiento por endpoint")
    tabla = r["endpoints"][["peticiones", "tasa_error", "latencia_p50", "latencia_p95"]].copy()
    tabla[["latencia_p50", "latencia_p95"]] = tabla[["latencia_p50", "latencia_p95"]].astype(float).round(1)
    tabla["tasa_error"] = (tabla["tasa_error"] * 100).round(2).astype(str) + " %"
    print(tabla.to_string())
    lento = r["endpoints"]["latencia_p95"].idxmax()
    print(f"→ Endpoint más lento (p95): {lento}")

    horas = r["horas"]
    print(f"\nHora punta: {horas.idxmax()}:00 ({num(horas.max())} peticiones) · valle: {horas.idxmin()}:00 ({num(horas.min())})")
    print("Dispositivos:", ", ".join(f"{k} {v / r['total']:.1%}" for k, v in r["dispositivos"].items()).replace(".", ","))
    print("IPs más activas (posibles bots):", ", ".join(f"{ip} ({num(n)})" for ip, n in r["top_ips"].items()))
    conteos, bordes = r["histograma"]
    acumulado = np.cumsum(conteos) / conteos.sum()
    print(f"Histograma (Dask Array): el {acumulado[np.searchsorted(bordes[1:], 500)]:.1%} de las peticiones tarda ≤ 500 ms".replace(".", ","))

    print("\nImpacto de los despliegues en /api/pago (3 h antes y después)")
    vista = despliegues.copy()
    vista[["p95_antes", "p95_despues"]] = vista[["p95_antes", "p95_despues"]].round(0).astype(int)
    for c in ("error_antes", "error_despues"):
        vista[c] = (vista[c] * 100).round(1)
    vista = vista.rename(columns={"error_antes": "% error antes", "error_despues": "% error después"})
    print(vista.to_string(index=False))
    for _, d in despliegues[despliegues["alerta"]].iterrows():
        factor = f"{d['p95_despues'] / d['p95_antes']:.1f}".replace(".", ",")
        errores = f"{d['error_despues']:.1%}".replace(".", ",")
        print(f"⚠ Despliegue {d['version']} ({d['servicio']}, {d['momento']}): p95 ×{factor}, errores {errores}")


def main() -> int:
    parser = argparse.ArgumentParser()
    parser.add_argument("--cluster", action="store_true", help="Usar un LocalCluster de dask.distributed")
    args = parser.parse_args()

    if not any(ORIGEN.glob("acceso_*.csv")):
        print("Generando registros de ejemplo…")
        generar_datos()
    print(f"{len(list(ORIGEN.glob('*.csv')))} archivos CSV ({sum(p.stat().st_size for p in ORIGEN.glob('*.csv')) / 1e6:.0f} MB)")

    cliente = None
    if args.cluster:
        from dask.distributed import Client, LocalCluster
        cliente = Client(LocalCluster(n_workers=4, threads_per_worker=1, memory_limit="2GB",
                                            silence_logs=logging.ERROR))                 # Oculta avisos informativos
        print(f"Clúster local: {len(cliente.scheduler_info()['workers'])} trabajadores · panel en {cliente.dashboard_link}")

    try:
        t0 = time.perf_counter()
        ingesta()
        t_ingesta = time.perf_counter() - t0
        resultados = analizar()
        despliegues = analizar_despliegues()
        t_pandas = comparar_con_pandas()
        informe(resultados, despliegues, t_ingesta, t_pandas)
        resultados["endpoints"].to_csv("resumen_endpoints.csv", sep=";", decimal=",")
        despliegues.to_csv("impacto_despliegues.csv", sep=";", decimal=",", index=False)
        print("\n✔ resumen_endpoints.csv e impacto_despliegues.csv guardados")
    finally:
        if cliente:
            cliente.close()
    return 0


if __name__ == "__main__":                 # Imprescindible con procesos y LocalCluster
    raise SystemExit(main())
```

> Con 2,4 millones de filas, pandas aún cabe en memoria y puede ser tan rápido como Dask: el ejemplo ilustra el **flujo de trabajo**, que sigue funcionando igual cuando los datos son 100 veces mayores o se ejecutan en un clúster.

---

## 20. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ValueError: Mismatched dtypes found in pd.read_csv` | Tipos inferidos con el primer bloque no coinciden con los demás. | `dtype={...}` explícito o `assume_missing=True`. |
| `ValueError: Metadata inference failed` / resultados con tipos erróneos | `meta` incorrecto o ausente en `map_partitions`/`apply`. | Proporcione `meta` con columnas y tipos exactos. |
| `KilledWorker` / `MemoryError` | Particiones demasiado grandes, `persist` excesivo o *shuffle* enorme. | Reduzca particiones, libere `persist`, aumente memoria o trabajadores. |
| `RuntimeError: An attempt has been made to start a new process before … bootstrapping` | Falta `if __name__ == "__main__":` con procesos o `LocalCluster`. | Proteja el punto de entrada. |
| Muy lento con muchos archivos pequeños | Miles de tareas diminutas. | Agrupe archivos, `repartition(partition_size=...)`. |
| `NotImplementedError` con una operación de pandas | No soportada en Dask (orden global, `iloc` por filas…). | `map_partitions`, reformular, o `compute()` tras reducir datos. |
| El cálculo se repite varias veces | `compute()` en bucles o resultados no persistidos. | `dask.compute(a, b, c)` o `persist()`. |
| `len(ddf)` o `ddf.shape[0]` tardan | Requieren recorrer todos los datos. | Úselo con moderación. |
| `TypeError: Truth of Delayed objects is not supported` | Uso de un valor diferido en un `if`. | Calcule antes o mueva la lógica dentro de la función `delayed`. |
| `ValueError: Missing dependency ('read_parquet-…')` tras `persist()` | Incidencia de algunas versiones al persistir directamente un `read_parquet` con selección de columnas. | Añada un paso antes de persistir (`repartition(...)` o `map_partitions`) o actualice Dask. |
| Error opaco en tareas paralelas | Trazas de varios hilos/procesos. | `compute(scheduler="synchronous")` para depurar. |

---

## 21. Buenas prácticas

- Use Dask solo cuando haga falta: si los datos caben con holgura en memoria, pandas o Polars son más simples y rápidos.
- Convierta los datos a **Parquet** particionado una vez y analice desde ahí, leyendo solo columnas y filas necesarias.
- Mantenga particiones de 100–300 MB y evite miles de archivos pequeños.
- Reduzca pronto (filtrar, seleccionar, agregar) y traiga a memoria solo resultados pequeños.
- Calcule varios resultados a la vez con `dask.compute` y use `persist` con criterio.
- Declare tipos y `meta`; use `category` para textos repetidos.
- Utilice `dask.distributed` y su panel de control incluso en un solo equipo.
- Depure con el planificador síncrono y pruebe la lógica primero con una partición en pandas.

---

## 22. Referencia rápida

```python
dd.read_csv(patron, dtype=, blocksize=) · dd.read_parquet(ruta, columns=, filters=) · dd.from_pandas(df, npartitions=)
ddf[cond] · ddf.assign(...) · ddf.groupby(...).agg(...) · dd.merge(a, b, on=) · dd.concat · ddf.nlargest · value_counts
ddf.map_partitions(f, meta=) · groupby(...).apply(f, meta=) · dd.Aggregation(name, chunk, agg, finalize)
ddf.repartition(partition_size="100MB") · ddf.set_index(col, sorted=True) · ddf.npartitions · ddf.divisions
x.compute() · dask.compute(a, b) · ddf.persist() · x.pprint() · x.optimize() · ProgressBar()
ddf.to_parquet(ruta, partition_on=) · ddf.to_csv("parte-*.csv")
Client(LocalCluster(n_workers=, memory_limit=)) · cliente.dashboard_link · performance_report(filename=)
@delayed · dask.compute(*tareas) · cliente.submit(f, x) · cliente.map · as_completed · cliente.scatter
da.from_array(x, chunks=) · da.random.default_rng().normal(size=, chunks=) · da.histogram · db.read_text().map(json.loads)
dask.config.set(scheduler="threads"|"processes"|"synchronous")
```

---

## 23. Ejercicios propuestos

1. Añada al ejemplo la detección de sesiones de usuario (inactividad > 30 min por IP) usando `set_index` por IP y `map_partitions`.
2. Genere 300 días de registros y compare el tiempo y la memoria máxima de pandas, Dask (hilos), Dask (`LocalCluster`) y Polars en modo *streaming*.
3. Con `dask.delayed`, procese en paralelo 200 archivos Excel heterogéneos de delegaciones, normalícelos y consolídelos en Parquet.
4. Use *futures* y `as_completed` para lanzar 500 simulaciones de Montecarlo y detenerlas cuando el error estimado baje de un umbral.
5. Entrene un modelo de scikit-learn con `joblib.parallel_backend("dask")` sobre un `LocalCluster` y analice el informe de rendimiento.

---

## 24. Referencias

- Documentación oficial: https://docs.dask.org/
- Dask DataFrame: https://docs.dask.org/en/stable/dataframe.html
- Buenas prácticas: https://docs.dask.org/en/stable/best-practices.html
- Dask Distributed: https://distributed.dask.org/
- Tutorial oficial: https://tutorial.dask.org/
