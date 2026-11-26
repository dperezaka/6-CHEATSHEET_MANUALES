# Manual de uso: LightGBM

> **Categoría:** Inteligencia artificial · **Utilidad:** Machine Learning de alto rendimiento
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** MIT · **Desarrollo:** Microsoft

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [API de scikit-learn: `LGBMClassifier` y `LGBMRegressor`](#4-api-de-scikit-learn-lgbmclassifier-y-lgbmregressor)
5. [API nativa: `Dataset` y `lgb.train`](#5-api-nativa-dataset-y-lgbtrain)
6. [Hiperparámetros](#6-hiperparámetros)
7. [Callbacks: parada temprana y registro](#7-callbacks-parada-temprana-y-registro)
8. [Variables categóricas y valores ausentes](#8-variables-categóricas-y-valores-ausentes)
9. [Objetivos y métricas](#9-objetivos-y-métricas)
10. [Validación cruzada y ajuste de hiperparámetros](#10-validación-cruzada-y-ajuste-de-hiperparámetros)
11. [Previsión de series temporales con modelos globales](#11-previsión-de-series-temporales-con-modelos-globales)
12. [Restricciones, árboles lineales y otras opciones avanzadas](#12-restricciones-árboles-lineales-y-otras-opciones-avanzadas)
13. [Interpretación del modelo](#13-interpretación-del-modelo)
14. [Rendimiento, GPU y entrenamiento distribuido](#14-rendimiento-gpu-y-entrenamiento-distribuido)
15. [Guardar y desplegar modelos](#15-guardar-y-desplegar-modelos)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**LightGBM** (*Light Gradient Boosting Machine*) es una implementación de *gradient boosting* sobre árboles de decisión diseñada para la **velocidad y la eficiencia en memoria**. Es especialmente adecuada para conjuntos de datos grandes (millones de filas) y es una de las herramientas más usadas en previsión de demanda, riesgo, *marketing* y competiciones de datos tabulares.

**Qué la hace rápida:**
- **Histogramas**: agrupa los valores continuos en intervalos (`max_bin`), lo que reduce cálculo y memoria.
- **Crecimiento por hojas (*leaf-wise*)**: en cada paso divide la hoja que más reduce la pérdida, en lugar de crecer nivel a nivel. Logra más precisión con menos árboles (pero puede sobreajustar si no se controla).
- **GOSS** (*Gradient-based One-Side Sampling*): prioriza las muestras con gradientes grandes.
- **EFB** (*Exclusive Feature Bundling*): agrupa variables dispersas que rara vez coinciden.

**Ventajas:**
- Entrenamiento muy rápido y bajo consumo de memoria.
- Soporte nativo de variables categóricas y valores ausentes.
- Gran variedad de objetivos (regresión, clasificación, cuantiles, Poisson, Tweedie, ordenación).
- Compatible con scikit-learn, y con entrenamiento distribuido (Dask, Spark) y GPU.

**Limitaciones:**
- Tiende a sobreajustar en conjuntos pequeños (menos de unos pocos miles de filas) si no se limita `num_leaves` y `min_data_in_leaf`.
- Muchos hiperparámetros, algunos con varios alias.
- Como todos los modelos de árboles, no extrapola fuera del rango de los datos de entrenamiento.

| Si necesita… | Opción |
|---|---|
| Máxima velocidad con millones de filas | **LightGBM** |
| Ecosistema muy maduro y amplio soporte | XGBoost |
| Muchas variables categóricas y buen rendimiento sin ajuste | CatBoost |
| Conjuntos pequeños e interpretabilidad | scikit-learn (modelos lineales, árboles) |

---

## 2. Instalación e importación

```bash
pip install lightgbm
conda install -c conda-forge lightgbm
```

> En macOS puede necesitar OpenMP: `brew install libomp`.

```bash
python -c "import lightgbm as lgb; print(lgb.__version__)"
```

Importaciones habituales:

```python
import lightgbm as lgb
import numpy as np
import pandas as pd
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| *Boosting* | Árboles entrenados en secuencia; cada uno corrige los errores del conjunto anterior. |
| Hoja / `num_leaves` | Número máximo de hojas de cada árbol: principal control de la complejidad. |
| Crecimiento por hojas | Se divide la hoja con mayor ganancia, aunque el árbol quede desequilibrado. |
| `Dataset` | Estructura de datos de la API nativa; precalcula los histogramas. |
| *Booster* | Modelo entrenado. |
| Iteración / ronda | Cada árbol añadido (`num_boost_round` o `n_estimators`). |
| *Callback* | Función que actúa durante el entrenamiento (parada temprana, registro). |
| Alias | Muchos parámetros tienen varios nombres (`learning_rate` = `eta` = `shrinkage_rate`). |

**Crecimiento por niveles frente a por hojas:**

```
Por niveles (XGBoost por defecto)       Por hojas (LightGBM)
          ●                                     ●
       ●     ●                               ●     ●
     ●  ●   ●  ●                                 ●   ●
                                                    ●  ●
Árbol equilibrado                        Profundiza donde más ganancia hay
```

---

## 4. API de scikit-learn: `LGBMClassifier` y `LGBMRegressor`

```python
modelo = lgb.LGBMClassifier(
    n_estimators=2000,
    learning_rate=0.05,
    num_leaves=31,
    min_child_samples=50,
    subsample=0.8, subsample_freq=1,
    colsample_bytree=0.8,
    reg_lambda=1.0,
    random_state=42,
    verbose=-1,                         # Silenciar mensajes
)
modelo.fit(
    X_ent, y_ent,
    eval_X=X_val, eval_y=y_val,         # Conjunto de validación
    eval_metric="auc",
    callbacks=[lgb.early_stopping(100), lgb.log_evaluation(200)],
)
modelo.predict_proba(X_test)[:, 1]
modelo.best_iteration_
modelo.best_score_["valid_0"]["auc"]
```

> Desde LightGBM 4.7, `eval_set=[(X_val, y_val)]` está en desuso: use `eval_X` y `eval_y` (tuplas para varios conjuntos).

| Clase | Uso |
|---|---|
| `LGBMClassifier` | Clasificación binaria y multiclase |
| `LGBMRegressor` | Regresión (incluidos cuantiles, Poisson, Tweedie) |
| `LGBMRanker` | Ordenación (*learning to rank*) |

Los nombres de la API de scikit-learn son alias de los nativos: `n_estimators` → `num_iterations`, `min_child_samples` → `min_data_in_leaf`, `subsample` → `bagging_fraction`, `colsample_bytree` → `feature_fraction`, `reg_lambda` → `lambda_l2`.

---

## 5. API nativa: `Dataset` y `lgb.train`

```python
d_ent = lgb.Dataset(X_ent, label=y_ent, categorical_feature="auto")
d_val = lgb.Dataset(X_val, label=y_val, reference=d_ent)      # Reutiliza los histogramas

parametros = {"objective": "binary", "metric": ["auc", "binary_logloss"],
              "learning_rate": 0.05, "num_leaves": 31, "min_data_in_leaf": 50,
              "feature_fraction": 0.8, "bagging_fraction": 0.8, "bagging_freq": 1,
              "verbosity": -1, "seed": 42}

booster = lgb.train(parametros, d_ent, num_boost_round=3000,
                    valid_sets=[d_ent, d_val], valid_names=["ent", "val"],
                    callbacks=[lgb.early_stopping(100), lgb.log_evaluation(200)])

booster.predict(X_test, num_iteration=booster.best_iteration)
```

Opciones del `Dataset`: `weight=` (pesos por muestra), `group=` (consultas en ordenación), `init_score=` (predicción inicial), `free_raw_data=False` (conservar los datos originales).

---

## 6. Hiperparámetros

### 6.1 Principales

| Parámetro (nativo / scikit-learn) | Efecto | Valores habituales |
|---|---|---|
| `num_leaves` | Complejidad de cada árbol | 15–255 (por defecto 31) |
| `max_depth` | Límite de profundidad (−1 = sin límite) | −1 o 6–12 |
| `min_data_in_leaf` / `min_child_samples` | Muestras mínimas por hoja | 20–500 según tamaño |
| `learning_rate` | Peso de cada árbol | 0,01–0,1 |
| `num_iterations` / `n_estimators` | Número máximo de árboles | Con parada temprana |
| `feature_fraction` / `colsample_bytree` | Fracción de variables por árbol | 0,6–1 |
| `bagging_fraction` + `bagging_freq` | Fracción de filas y frecuencia del muestreo | 0,6–1 y 1 |
| `lambda_l1`, `lambda_l2` | Regularización | 0–10 |
| `min_gain_to_split` | Ganancia mínima para dividir | 0–1 |
| `max_bin` | Intervalos de los histogramas | 63–511 (por defecto 255) |
| `cat_smooth`, `min_data_per_group` | Regularización de categorías | — |

> **Regla clave:** `num_leaves` debe ser menor que `2^max_depth`. Para evitar sobreajuste, controle sobre todo `num_leaves` y `min_data_in_leaf`.

### 6.2 Guía de ajuste por objetivo

| Objetivo | Ajuste |
|---|---|
| Más precisión | ↓ `learning_rate` con más iteraciones, ↑ `num_leaves`, ↑ `max_bin` |
| Menos sobreajuste | ↓ `num_leaves`, ↑ `min_data_in_leaf`, muestreo de filas y columnas, ↑ `lambda_l2`, `max_depth` |
| Más velocidad | ↑ `learning_rate`, ↓ `max_bin`, muestreo, `num_threads` = núcleos físicos |

---

## 7. Callbacks: parada temprana y registro

```python
historial = {}
callbacks = [
    lgb.early_stopping(stopping_rounds=100, first_metric_only=True, verbose=False),
    lgb.log_evaluation(period=200),          # Mostrar la métrica cada 200 iteraciones
    lgb.record_evaluation(historial),        # Guardar el historial en un diccionario
    lgb.reset_parameter(learning_rate=lambda i: 0.1 * 0.995 ** i),   # Tasa variable
]
booster = lgb.train(parametros, d_ent, valid_sets=[d_val], callbacks=callbacks)
lgb.plot_metric(historial)                    # Requiere matplotlib
```

> En LightGBM 4, la parada temprana y el registro se configuran **solo** mediante *callbacks* (los antiguos argumentos `early_stopping_rounds` y `verbose_eval` de `train` se eliminaron).

---

## 8. Variables categóricas y valores ausentes

### 8.1 Categorías nativas

```python
for col in ["tienda", "producto", "region"]:
    df[col] = df[col].astype("category")

d = lgb.Dataset(df[variables], df["objetivo"], categorical_feature="auto")   # Detecta el tipo category
# o explícitamente: categorical_feature=["tienda", "producto"]
```

LightGBM busca la mejor partición de las categorías en dos grupos en cada división, sin *one-hot*. Es más eficaz y rápido que la codificación *one-hot* con muchas categorías.

> Las categorías deben tener **los mismos valores y orden** en entrenamiento y predicción. Si crea los `category` por separado en cada conjunto, fije las categorías con `pd.CategoricalDtype(categories=lista)`.

### 8.2 Valores ausentes

Se gestionan automáticamente (`use_missing=True`): el modelo aprende la mejor rama para los ausentes. Con `zero_as_missing=True`, los ceros también se tratan como ausentes.

---

## 9. Objetivos y métricas

| Problema | `objective` | Métricas (`metric`) |
|---|---|---|
| Regresión | `regression` (L2), `regression_l1`, `huber`, `fair` | `l2`, `rmse`, `l1` (MAE), `mape` |
| Cuantiles | `quantile` + `alpha` | `quantile` |
| Recuentos | `poisson` | `poisson`, `l1` |
| Ventas o importes con ceros y asimetría | `tweedie` + `tweedie_variance_power` (1–2) | `tweedie`, `l1` |
| Valores positivos asimétricos | `gamma` | `gamma` |
| Binario | `binary` | `auc`, `binary_logloss`, `average_precision` |
| Multiclase | `multiclass` + `num_class` | `multi_logloss`, `multi_error` |
| Ordenación | `lambdarank` | `ndcg`, `map` |

**Intervalos de predicción** con dos modelos de cuantiles:

```python
p10 = lgb.LGBMRegressor(objective="quantile", alpha=0.1).fit(X_ent, y_ent).predict(X_test)
p90 = lgb.LGBMRegressor(objective="quantile", alpha=0.9).fit(X_ent, y_ent).predict(X_test)
```

Clases desbalanceadas: `is_unbalance=True` o `scale_pos_weight`, y métricas `average_precision` / `auc`.

---

## 10. Validación cruzada y ajuste de hiperparámetros

### 10.1 Validación cruzada nativa

```python
resultado = lgb.cv(parametros, lgb.Dataset(X, y), num_boost_round=3000, nfold=5, stratified=True,
                   callbacks=[lgb.early_stopping(100, verbose=False)], seed=42)
len(resultado["valid auc-mean"])          # Número óptimo de iteraciones
resultado["valid auc-mean"][-1]
```

Para series temporales, pase particiones propias con `folds=TimeSeriesSplit(n_splits=5)`.

### 10.2 Búsqueda de hiperparámetros

Con scikit-learn (`RandomizedSearchCV`) o con **Optuna**, que incluye una integración específica (`optuna-integration`, `LightGBMTunerCV`) que ajusta los parámetros por etapas:

```python
import optuna

def objetivo(trial):
    p = {"objective": "binary", "metric": "auc", "verbosity": -1,
         "num_leaves": trial.suggest_int("num_leaves", 15, 255, log=True),
         "min_data_in_leaf": trial.suggest_int("min_data_in_leaf", 20, 500, log=True),
         "feature_fraction": trial.suggest_float("feature_fraction", 0.5, 1.0),
         "lambda_l2": trial.suggest_float("lambda_l2", 1e-3, 10, log=True),
         "learning_rate": 0.05}
    b = lgb.train(p, d_ent, 3000, valid_sets=[d_val], callbacks=[lgb.early_stopping(100, verbose=False)])
    return b.best_score["valid_0"]["auc"]

optuna.create_study(direction="maximize").optimize(objetivo, n_trials=50)
```

---

## 11. Previsión de series temporales con modelos globales

LightGBM es la técnica dominante en previsión de demanda con muchas series (tiendas, productos, almacenes): se entrena **un único modelo global** con todas las series, en lugar de un modelo por serie.

### 11.1 Variables habituales

| Tipo | Ejemplos |
|---|---|
| Calendario | Día de la semana, día del mes, mes, semana del año, festivos, vísperas |
| Retardos (*lags*) | Ventas hace 7, 14, 28, 364 días |
| Ventanas móviles | Media, desviación, máximo de los últimos 7 / 28 días |
| Identificadores | Tienda, producto, familia, región (como categorías) |
| Variables externas | Promociones, precio, tiempo meteorológico, eventos |

```python
serie = df.sort_values("fecha").groupby(["tienda", "producto"])["ventas"]
df["ventas_hace_7d"] = serie.shift(7)
df["media_28d"] = serie.transform(lambda s: s.shift(7).rolling(28).mean())
```

### 11.2 Reglas para evitar fugas de información

- Los retardos deben ser **≥ horizonte de previsión** (si se prevé a 7 días vista, no se puede usar la venta de ayer).
- Las ventanas móviles se calculan sobre valores **ya desplazados** (`shift` antes de `rolling`).
- Divida los datos **por fecha** (entrenamiento → validación → prueba), nunca aleatoriamente.

### 11.3 Métricas de previsión

| Métrica | Uso |
|---|---|
| **WMAPE** = Σ\|error\| / Σ real | Estándar en *retail*: robusta con ceros |
| Sesgo = (Σ previsto − Σ real) / Σ real | Detecta infraestimación o sobreestimación sistemática |
| MAE / RMSE | Error en unidades |
| Cobertura de intervalos | Proporción de valores reales dentro del intervalo P10–P90 |

---

## 12. Restricciones, árboles lineales y otras opciones avanzadas

```python
lgb.LGBMRegressor(monotone_constraints=[1, 0, -1, ...])          # Una entrada por variable
lgb.LGBMRegressor(interaction_constraints=[[0, 1], [2, 3]])       # Grupos de variables que pueden interactuar
lgb.LGBMRegressor(linear_tree=True)       # Hojas con regresión lineal: extrapola mejor en tendencias
lgb.LGBMClassifier(boosting_type="dart")  # Dropout en los árboles: a veces generaliza mejor (más lento)
lgb.LGBMClassifier(data_sample_strategy="goss")   # Muestreo GOSS
```

---

## 13. Interpretación del modelo

```python
pd.Series(booster.feature_importance("gain"), index=booster.feature_name()).sort_values()
modelo.feature_importances_                         # API scikit-learn ("split" por defecto)
lgb.plot_importance(booster, importance_type="gain", max_num_features=20)

contribuciones = booster.predict(X_test, pred_contrib=True)   # Valores SHAP (última columna = valor base)
booster.trees_to_dataframe()                                  # Estructura de los árboles
lgb.plot_tree(booster, tree_index=0)                          # Requiere graphviz
```

> `importance_type="split"` (por defecto) cuenta usos; `"gain"` mide la mejora real de la pérdida y suele ser más informativa. Para análisis detallados, use la librería **SHAP**.

---

## 14. Rendimiento, GPU y entrenamiento distribuido

| Técnica | Detalle |
|---|---|
| `num_threads` | Fije el número de **núcleos físicos** (0 = automático) |
| `max_bin` menor | Más rápido, ligera pérdida de precisión |
| `Dataset.save_binary("d.bin")` | Evita reconstruir los histogramas en entrenamientos repetidos |
| `device_type="gpu"` / `"cuda"` | Requiere una compilación de LightGBM con soporte de GPU |
| `lightgbm.dask` | Entrenamiento distribuido con Dask |
| SynapseML | Entrenamiento en Apache Spark |

---

## 15. Guardar y desplegar modelos

```python
booster.save_model("modelo.txt", num_iteration=booster.best_iteration)    # Texto legible
booster = lgb.Booster(model_file="modelo.txt")
modelo.booster_.save_model("modelo.txt")          # Desde la API de scikit-learn
booster.model_to_string(); booster.dump_model()   # Cadena / diccionario JSON
```

El formato de texto es estable entre versiones. Para producción, también puede convertirse a ONNX (`onnxmltools`) o compilarse con Treelite / TL2cgen.

---

## 16. Ejemplo práctico completo

**Previsión de ventas diarias por tienda y producto**: genera dos años y medio de ventas de 750 series (25 tiendas × 30 productos, más de 650 000 filas) con tendencia, estacionalidad semanal y anual, promociones y festivos. Construye variables de calendario, **retardos y medias móviles sin fuga de información**, entrena **un modelo global** con objetivo Tweedie, categorías nativas y *callbacks*, lo compara con dos referencias ingenuas, mide el error agregado por tienda y semana, estima **intervalos P10–P90** con regresión por cuantiles y muestra la importancia de las variables.

Requisitos: `pip install lightgbm pandas numpy`

```python
"""
Previsión de ventas diarias por tienda y producto con LightGBM.

Un único modelo global para 750 series (25 tiendas × 30 productos):
variables de calendario, retardos y medias móviles, categorías nativas,
promociones y festivos. Entrena con la API nativa (Dataset + callbacks de
parada temprana y registro), compara con referencias ingenuas, estima
intervalos de predicción con regresión por cuantiles y explica el modelo.
"""
import time

import lightgbm as lgb
import numpy as np
import pandas as pd

SEMILLA = 3
N_TIENDAS, N_PRODUCTOS = 25, 30
INICIO, FIN = "2024-01-01", "2026-06-30"
CORTE_VALIDACION, CORTE_PRUEBA = "2026-03-01", "2026-05-01"     # Divisiones temporales
FESTIVOS = pd.to_datetime(["2024-01-01", "2024-01-06", "2024-03-29", "2024-05-01", "2024-08-15",
                           "2024-10-12", "2024-11-01", "2024-12-06", "2024-12-25", "2025-01-01",
                           "2025-01-06", "2025-04-18", "2025-05-01", "2025-08-15", "2025-10-12",
                           "2025-11-01", "2025-12-06", "2025-12-25", "2026-01-01", "2026-01-06",
                           "2026-04-03", "2026-05-01"])


# ------------------------------------------------------------------ Datos sintéticos
def generar_ventas() -> pd.DataFrame:
    rng = np.random.default_rng(SEMILLA)
    fechas = pd.date_range(INICIO, FIN, freq="D")
    tiendas = pd.DataFrame({"tienda": [f"T{i:02d}" for i in range(N_TIENDAS)],
                            "tamano_tienda": rng.lognormal(0, 0.4, N_TIENDAS),
                            "region": rng.choice(["Norte", "Sur", "Este", "Centro"], N_TIENDAS)})
    productos = pd.DataFrame({"producto": [f"P{j:02d}" for j in range(N_PRODUCTOS)],
                              "familia": rng.choice(["alimentación", "bebidas", "limpieza", "higiene"],
                                                    N_PRODUCTOS),
                              "nivel_producto": rng.lognormal(1.2, 0.6, N_PRODUCTOS),
                              "estacionalidad": rng.uniform(-0.4, 0.4, N_PRODUCTOS)})
    df = (pd.DataFrame({"fecha": fechas}).merge(tiendas, how="cross").merge(productos, how="cross"))
    dia_semana = df["fecha"].dt.dayofweek.to_numpy()
    dia_anio = df["fecha"].dt.dayofyear.to_numpy()
    df["promocion"] = (rng.random(len(df)) < 0.06).astype(int)
    df["festivo"] = df["fecha"].isin(FESTIVOS).astype(int)
    tendencia = 1 + 0.08 * (df["fecha"] - df["fecha"].min()).dt.days.to_numpy() / 365
    efecto_semana = np.array([0.85, 0.9, 0.95, 1.0, 1.2, 1.45, 0.6])[dia_semana]
    estacional = 1 + df["estacionalidad"].to_numpy() * np.sin(2 * np.pi * (dia_anio - 80) / 365)
    media = (df["tamano_tienda"] * df["nivel_producto"]).to_numpy() * tendencia * efecto_semana \
        * estacional * np.where(df["promocion"] == 1, 1.8, 1.0) * np.where(df["festivo"] == 1, 0.3, 1.0)
    df["ventas"] = rng.poisson(media).astype("float32")
    return df.drop(columns=["tamano_tienda", "nivel_producto", "estacionalidad"])


# ------------------------------------------------------------------ Variables
def crear_variables(df: pd.DataFrame) -> pd.DataFrame:
    df = df.sort_values(["tienda", "producto", "fecha"]).copy()
    df["dia_semana"] = df["fecha"].dt.dayofweek
    df["dia_mes"] = df["fecha"].dt.day
    df["mes"] = df["fecha"].dt.month
    df["semana_anio"] = df["fecha"].dt.isocalendar().week.astype(int)
    serie = df.groupby(["tienda", "producto"], observed=True)["ventas"]
    # Retardos ≥ 7 días: el modelo puede prever con una semana de antelación
    for retardo in (7, 14, 28, 364):
        df[f"ventas_hace_{retardo}d"] = serie.shift(retardo)
    for ventana in (7, 28):
        df[f"media_{ventana}d"] = serie.transform(lambda s: s.shift(7).rolling(ventana).mean())
    df["media_misma_dia_4sem"] = serie.transform(
        lambda s: (s.shift(7) + s.shift(14) + s.shift(21) + s.shift(28)) / 4)
    for col in ["tienda", "producto", "familia", "region"]:
        df[col] = df[col].astype("category")
    return df.dropna(subset=["ventas_hace_28d", "media_28d"])


VARIABLES = ["tienda", "producto", "familia", "region", "promocion", "festivo", "dia_semana", "dia_mes",
             "mes", "semana_anio", "ventas_hace_7d", "ventas_hace_14d", "ventas_hace_28d",
             "ventas_hace_364d", "media_7d", "media_28d", "media_misma_dia_4sem"]


def wmape(real, pred) -> float:
    """Error absoluto ponderado: suma de errores / suma de ventas."""
    return np.abs(real - pred).sum() / real.sum()


def main() -> None:
    print(f"LightGBM {lgb.__version__}")
    inicio = time.perf_counter()
    df = crear_variables(generar_ventas())
    ent = df[df["fecha"] < CORTE_VALIDACION]
    val = df[(df["fecha"] >= CORTE_VALIDACION) & (df["fecha"] < CORTE_PRUEBA)]
    test = df[df["fecha"] >= CORTE_PRUEBA]
    print(f"Filas: {len(df):,} · series {N_TIENDAS * N_PRODUCTOS:,} · entrenamiento {len(ent):,} · "
          f"validación {len(val):,} · prueba {len(test):,} · preparación {time.perf_counter() - inicio:.0f} s\n")

    # 1. Modelo principal: objetivo Tweedie (recuentos con muchos ceros y asimetría)
    parametros = {"objective": "tweedie", "tweedie_variance_power": 1.2, "metric": "mae",
                  "learning_rate": 0.08, "num_leaves": 63, "min_data_in_leaf": 100,
                  "feature_fraction": 0.8, "bagging_fraction": 0.8, "bagging_freq": 1,
                  "lambda_l2": 1.0, "verbosity": -1, "seed": SEMILLA, "num_threads": 0}
    d_ent = lgb.Dataset(ent[VARIABLES], ent["ventas"], categorical_feature="auto")
    d_val = lgb.Dataset(val[VARIABLES], val["ventas"], reference=d_ent)
    historial = {}
    inicio = time.perf_counter()
    modelo = lgb.train(parametros, d_ent, num_boost_round=3000, valid_sets=[d_val], valid_names=["val"],
                       callbacks=[lgb.early_stopping(100, verbose=False), lgb.log_evaluation(200),
                                  lgb.record_evaluation(historial)])
    print(f"Mejor iteración: {modelo.best_iteration} · MAE val {modelo.best_score['val']['l1']:.3f} · "
          f"entrenamiento {time.perf_counter() - inicio:.0f} s\n")

    # 2. Evaluación frente a referencias ingenuas
    pred = modelo.predict(test[VARIABLES], num_iteration=modelo.best_iteration)
    real = test["ventas"].to_numpy()
    print(f"{'Método':<34}{'WMAPE':>8}{'Sesgo':>9}")
    for nombre, p in [("LightGBM", pred),
                      ("Mismo día semana anterior", test["ventas_hace_7d"].to_numpy()),
                      ("Media mismo día últimas 4 sem.", test["media_misma_dia_4sem"].to_numpy())]:
        print(f"{nombre:<34}{wmape(real, p):>8.1%}{(p.sum() - real.sum()) / real.sum():>+9.1%}")

    # Error agregado: tienda-semana (nivel de decisión de reposición)
    agregado = (test.assign(pred=pred).groupby(["tienda", pd.Grouper(key="fecha", freq="W")], observed=True)
                [["ventas", "pred"]].sum())
    print(f"WMAPE agregando por tienda y semana: {wmape(agregado['ventas'], agregado['pred']):.1%}")

    # 3. Intervalos de predicción: modelos de cuantiles P10 y P90
    cuantiles = {}
    for alfa in (0.1, 0.9):
        p = {**parametros, "objective": "quantile", "alpha": alfa, "metric": "quantile"}
        m = lgb.train(p, d_ent, num_boost_round=modelo.best_iteration)
        cuantiles[alfa] = m.predict(test[VARIABLES])
    cobertura = np.mean((real >= cuantiles[0.1]) & (real <= cuantiles[0.9]))
    print(f"Intervalo P10–P90: cobertura real {cobertura:.1%} (objetivo 80 %)\n")

    # 4. Importancia de variables
    importancia = pd.Series(modelo.feature_importance("gain"), index=modelo.feature_name())
    importancia = (importancia / importancia.sum()).sort_values(ascending=False)
    print("Importancia (ganancia):")
    for variable, valor in importancia.head(8).items():
        print(f"  {variable:<24}{valor:6.1%}  {'█' * int(valor * 60)}")

    # 5. Previsión de ejemplo con intervalo
    fila = test[(test["tienda"] == "T07") & (test["producto"] == "P03")].iloc[-7:]
    idx = fila.index
    posiciones = test.index.get_indexer(idx)
    print("\nTienda T07 · producto P03 · última semana de prueba")
    print(f"{'Fecha':<12}{'Real':>6}{'Previsión':>11}{'P10–P90':>12}")
    for (_, f), i in zip(fila.iterrows(), posiciones):
        print(f"{f['fecha']:%Y-%m-%d}  {f['ventas']:>4.0f}{pred[i]:>11.1f}"
              f"{cuantiles[0.1][i]:>6.0f}–{cuantiles[0.9][i]:<4.0f}{'  promoción' if f['promocion'] else ''}")

    modelo.save_model("modelo_ventas.txt", num_iteration=modelo.best_iteration)


if __name__ == "__main__":
    main()
```

Resultado:

```
LightGBM 4.7.0
Filas: 658,500 · series 750 · entrenamiento 567,000 · validación 45,750 · prueba 45,750 · preparación 3 s

[200]	val's l1: 1.61492
[400]	val's l1: 1.61037
[600]	val's l1: 1.61002
Mejor iteración: 508 · MAE val 1.610 · entrenamiento 17 s

Método                               WMAPE    Sesgo
LightGBM                             33.7%    -3.7%
Mismo día semana anterior            51.7%    +0.1%
Media mismo día últimas 4 sem.       41.0%    -0.2%
WMAPE agregando por tienda y semana: 4.3%
Intervalo P10–P90: cobertura real 78.2% (objetivo 80 %)

Importancia (ganancia):
  media_misma_dia_4sem     59.3%  ███████████████████████████████████
  media_28d                27.8%  ████████████████
  dia_semana                3.6%  ██
  promocion                 3.3%  █
  festivo                   2.2%  █
  producto                  1.1%  
  media_7d                  0.9%  
  tienda                    0.8%  

Tienda T07 · producto P03 · última semana de prueba
Fecha         Real  Previsión     P10–P90
2026-06-24     1        2.3     1–5   
2026-06-25     1        2.5     1–5   
2026-06-26     3        3.1     1–5   
2026-06-27     1        4.0     2–6   
2026-06-28     4        2.6     1–5     promoción
2026-06-29     5        3.9     2–7     promoción
2026-06-30     3        2.2     1–4   
```

> El modelo reduce el error un 18 % respecto a la mejor referencia ingenua. El error por producto y día es alto (34 %) porque las ventas diarias de un producto en una tienda son pequeñas y muy ruidosas, pero al agregar por tienda y semana, que es el nivel de las decisiones de reposición, baja al 4,3 %. El sesgo del −3,7 % indica que el modelo infraestima ligeramente la tendencia de crecimiento: los árboles no extrapolan, y conviene vigilar esta métrica en producción.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TypeError: train() got an unexpected keyword argument 'early_stopping_rounds'` | API anterior a la versión 4. | Use `callbacks=[lgb.early_stopping(100)]`. |
| `LGBMDeprecationWarning: The argument 'eval_set' is deprecated` | LightGBM ≥ 4.7. | Use `eval_X=` y `eval_y=`. |
| `ValueError: pandas dtypes must be int, float or bool` | Columnas de texto (`object`). | Conviértalas a `category`. |
| `Warning: No further splits with positive gain` | Árboles que no pueden dividirse (pocos datos o `min_data_in_leaf` alto). | Normal al final; si es continuo, revise los datos y los parámetros. |
| `The number of features in data is not the same as it was in training data` | Columnas distintas al predecir. | Use las mismas columnas en el mismo orden. |
| Predicciones erróneas con categorías | Categorías creadas por separado en cada conjunto. | Use un `CategoricalDtype` común. |
| Sobreajuste en conjuntos pequeños | `num_leaves` alto, `min_data_in_leaf` bajo. | ↓ `num_leaves`, ↑ `min_data_in_leaf`, `max_depth`. |
| Evaluación excelente en series temporales y mala en producción | Retardos menores que el horizonte o división aleatoria. | Retardos ≥ horizonte y partición por fecha. |
| `[Warning] Found whitespace in feature_names` | Nombres de columnas con espacios o caracteres especiales. | Renombre las columnas. |
| Mensajes informativos excesivos | Nivel de detalle por defecto. | `verbosity=-1` (nativo) o `verbose=-1` (scikit-learn). |

---

## 18. Buenas prácticas

- Controle la complejidad sobre todo con **`num_leaves` y `min_data_in_leaf`**, y use parada temprana.
- Use **categorías nativas** para identificadores y variables categóricas de alta cardinalidad.
- En previsión, entrene **modelos globales** y construya retardos y ventanas que respeten el horizonte.
- Elija el **objetivo según la distribución** de la variable (Tweedie o Poisson para ventas y recuentos; cuantiles para intervalos).
- Evalúe al **nivel de agregación de la decisión** (tienda-semana, almacén-mes) además del nivel de detalle.
- Compare siempre con referencias ingenuas y vigile el **sesgo**.
- Fije `seed` y `deterministic=True` si necesita resultados reproducibles exactos.
- Guarde el modelo con `save_model` junto con la lista de variables y las categorías.

---

## 19. Referencia rápida

```python
import lightgbm as lgb

m = lgb.LGBMRegressor(n_estimators=3000, learning_rate=0.05, num_leaves=31, min_child_samples=50,
                      subsample=0.8, subsample_freq=1, colsample_bytree=0.8, reg_lambda=1, objective="tweedie", verbose=-1)
m.fit(X, y, eval_X=X_val, eval_y=y_val, eval_metric="l1", callbacks=[lgb.early_stopping(100), lgb.log_evaluation(200)])
m.predict(X) · m.best_iteration_ · m.feature_importances_ · m.booster_

d = lgb.Dataset(X, y, categorical_feature="auto"); dv = lgb.Dataset(Xv, yv, reference=d)
b = lgb.train(params, d, num_boost_round=3000, valid_sets=[dv], callbacks=[lgb.early_stopping(100), lgb.record_evaluation(h)])
lgb.cv(params, d, nfold=5, stratified=True, callbacks=[...]) · b.predict(X, num_iteration=b.best_iteration)

objective: regression · regression_l1 · huber · quantile (alpha) · poisson · tweedie · gamma · binary · multiclass · lambdarank
b.feature_importance("gain") · b.predict(X, pred_contrib=True) · b.save_model("m.txt") · lgb.Booster(model_file="m.txt")
```

---

## 20. Ejercicios propuestos

1. Compare LightGBM y XGBoost en el ejemplo de fraude del manual de XGBoost: tiempo de entrenamiento y PR AUC.
2. Modifique el ejemplo completo para prever a 1 día vista (retardos desde 1 día) y compare el error con la versión a 7 días.
3. Añada al ejemplo una variable de precio y una restricción monótona decreciente sobre ella.
4. Pruebe `linear_tree=True` en el ejemplo y analice si se reduce el sesgo de la tendencia.
5. Calibre los intervalos de predicción para que la cobertura real se acerque al 80 % (por ejemplo, ajustando los cuantiles con el conjunto de validación).

---

## 21. Referencias

- Documentación oficial: https://lightgbm.readthedocs.io/
- Parámetros: https://lightgbm.readthedocs.io/en/latest/Parameters.html
- Ajuste de parámetros: https://lightgbm.readthedocs.io/en/latest/Parameters-Tuning.html
- Repositorio: https://github.com/microsoft/LightGBM
- Ke, G. et al. (2017). *LightGBM: A Highly Efficient Gradient Boosting Decision Tree*. NeurIPS.
