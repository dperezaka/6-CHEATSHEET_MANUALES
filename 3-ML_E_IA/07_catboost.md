# Manual de uso: CatBoost

> **Categoría:** Inteligencia artificial · **Utilidad:** Modelos predictivos con variables categóricas
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** Apache 2.0 · **Desarrollo:** Yandex

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer modelo: `CatBoostClassifier` y `CatBoostRegressor`](#4-primer-modelo-catboostclassifier-y-catboostregressor)
5. [El objeto `Pool`](#5-el-objeto-pool)
6. [Variables categóricas: cómo las trata CatBoost](#6-variables-categóricas-cómo-las-trata-catboost)
7. [Variables de texto y *embeddings*](#7-variables-de-texto-y-embeddings)
8. [Hiperparámetros](#8-hiperparámetros)
9. [Parada temprana y sobreajuste](#9-parada-temprana-y-sobreajuste)
10. [Funciones de pérdida y métricas](#10-funciones-de-pérdida-y-métricas)
11. [Validación cruzada y búsqueda de hiperparámetros](#11-validación-cruzada-y-búsqueda-de-hiperparámetros)
12. [Clases desbalanceadas y calibración](#12-clases-desbalanceadas-y-calibración)
13. [Interpretación: importancia, SHAP e interacciones](#13-interpretación-importancia-shap-e-interacciones)
14. [GPU y rendimiento](#14-gpu-y-rendimiento)
15. [Guardar y desplegar modelos](#15-guardar-y-desplegar-modelos)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**CatBoost** (*Categorical Boosting*) es una implementación de *gradient boosting* sobre árboles especializada en **datos con variables categóricas**. Las procesa directamente, sin codificación previa, mediante estadísticas ordenadas del objetivo que evitan la fuga de información. Además, sus valores por defecto están muy bien ajustados, por lo que suele dar resultados excelentes **sin apenas ajustar hiperparámetros**.

**Ventajas:**
- Uso directo de variables categóricas, incluidas las de **alta cardinalidad** (códigos postales, agencias, productos), sin *one-hot* ni *target encoding* manual.
- Admite también **texto** y ***embeddings*** como variables de entrada.
- Muy buen rendimiento «de serie»: los parámetros por defecto funcionan bien.
- **Boosting ordenado** y árboles simétricos: menos sobreajuste y predicción muy rápida.
- Herramientas de interpretación integradas (SHAP, interacciones, análisis de variables).
- Soporte de GPU, regresión multicuantil, ordenación y supervivencia.

**Limitaciones:**
- El entrenamiento en CPU suele ser más lento que el de LightGBM en conjuntos grandes y sobre todo numéricos.
- Los valores ausentes en variables categóricas deben sustituirse por una cadena.
- Archivos de modelo de mayor tamaño en algunos casos.

| Si necesita… | Opción |
|---|---|
| Muchas variables categóricas, buen rendimiento con poco ajuste | **CatBoost** |
| Máxima velocidad con millones de filas numéricas | LightGBM |
| Ecosistema muy maduro, amplio soporte de plataformas | XGBoost |
| Modelos sencillos e interpretables | scikit-learn |

---

## 2. Instalación e importación

```bash
pip install catboost
conda install -c conda-forge catboost
```

```bash
python -c "import catboost; print(catboost.__version__)"
```

Importaciones habituales:

```python
from catboost import CatBoostClassifier, CatBoostRegressor, CatBoostRanker, Pool, cv
```

> Para los gráficos interactivos en Jupyter (`plot=True`) instale también `ipywidgets`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Variable categórica | Columna con valores discretos sin orden numérico (país, canal). Se declara en `cat_features`. |
| *Target statistics* | Codificación de cada categoría con estadísticas del objetivo calculadas solo con las filas **anteriores** de una permutación aleatoria (evita fugas). |
| Combinaciones de categorías | CatBoost crea automáticamente combinaciones de categorías (p. ej. país × canal). |
| *Boosting* ordenado | Esquema de entrenamiento que reduce el sesgo de predicción en conjuntos pequeños. |
| Árbol simétrico (*oblivious*) | Todos los nodos de un mismo nivel usan la misma condición: árboles equilibrados, regulares y muy rápidos al predecir. |
| `Pool` | Estructura de datos que agrupa variables, objetivo, tipos de columnas y pesos. |
| *Overfitting detector* | Parada temprana integrada (`od_type`, `od_wait`, `early_stopping_rounds`). |

---

## 4. Primer modelo: `CatBoostClassifier` y `CatBoostRegressor`

```python
categoricas = ["pais", "canal", "producto"]

modelo = CatBoostClassifier(
    iterations=2000,
    learning_rate=0.05,
    depth=6,
    eval_metric="AUC",
    early_stopping_rounds=100,
    random_seed=42,
    verbose=200,                    # Mostrar progreso cada 200 iteraciones (0 = silencio)
)
modelo.fit(X_ent, y_ent, cat_features=categoricas, eval_set=(X_val, y_val), use_best_model=True)

modelo.predict(X_test)              # Clases
modelo.predict_proba(X_test)[:, 1]  # Probabilidades
modelo.get_best_iteration()
modelo.get_best_score()
```

Regresión:

```python
regresor = CatBoostRegressor(iterations=1500, loss_function="RMSE", depth=6, verbose=0)
regresor.fit(X_ent, y_ent, cat_features=categoricas, eval_set=(X_val, y_val))
```

| Clase | Uso |
|---|---|
| `CatBoostClassifier` | Clasificación binaria y multiclase |
| `CatBoostRegressor` | Regresión (incluye cuantiles, Poisson, Tweedie, multiobjetivo) |
| `CatBoostRanker` | Ordenación |

Son compatibles con la API de scikit-learn (`fit`, `predict`, `get_params`, `Pipeline`, `cross_val_score`).

---

## 5. El objeto `Pool`

Agrupa los datos con la información de tipos de columna. Es útil al reutilizar los mismos datos varias veces o al añadir texto, pesos o grupos:

```python
ent = Pool(data=X_ent, label=y_ent, cat_features=categoricas, text_features=["comentario"],
           weight=pesos, feature_names=list(X_ent.columns))
val = Pool(X_val, y_val, cat_features=categoricas, text_features=["comentario"])

modelo.fit(ent, eval_set=val)
modelo.predict_proba(val)
```

`cat_features` admite nombres de columna (con `DataFrame`) o índices de posición.

---

## 6. Variables categóricas: cómo las trata CatBoost

### 6.1 Uso

1. Las columnas categóricas deben ser de tipo **texto o entero** (no `float`).
2. Sustituya los valores ausentes por una cadena (`"desconocido"`): CatBoost **no** admite `NaN` en categóricas.
3. Declárelas en `cat_features`.

```python
df[categoricas] = df[categoricas].fillna("desconocido").astype(str)
```

### 6.2 Qué hace internamente

| Mecanismo | Descripción |
|---|---|
| *One-hot* para pocas categorías | Si una variable tiene ≤ `one_hot_max_size` valores (2 por defecto en CPU), se usa *one-hot*. |
| *Ordered target statistics* | Para el resto, cada fila se codifica con la media del objetivo de las filas **anteriores** de la misma categoría en una permutación aleatoria, suavizada con un valor previo. |
| Combinaciones | Durante la construcción de los árboles se crean combinaciones de categorías (controladas por `max_ctr_complexity`). |

Así se evita el principal problema del *target encoding* tradicional: que cada fila «vea» su propio valor objetivo, lo que provoca sobreajuste.

> No aplique *one-hot* ni *label encoding* antes de CatBoost: perdería sus ventajas. Pase las categorías en bruto.

---

## 7. Variables de texto y *embeddings*

### 7.1 Texto

CatBoost puede usar columnas de texto: las tokeniza, crea diccionarios de palabras y n-gramas, y genera variables numéricas (*Naive Bayes*, BM25, bolsa de palabras):

```python
modelo = CatBoostClassifier(iterations=1000, verbose=0)
modelo.fit(Pool(X, y, cat_features=categoricas, text_features=["descripcion"]))
```

Opciones avanzadas en `text_processing` (tokenizadores, diccionarios, generadores de variables).

> Las versiones recientes admiten texto en clasificación y regresión; en otras funciones de pérdida (ordenación, multiobjetivo), compruebe el soporte en la documentación de su versión.

### 7.2 *Embeddings*

Vectores numéricos (por ejemplo, generados por un modelo de lenguaje) como variable de entrada:

```python
Pool(X, y, embedding_features=["vector_descripcion"])     # Columna con listas o arrays de igual longitud
```

---

## 8. Hiperparámetros

| Parámetro | Efecto | Por defecto / habitual |
|---|---|---|
| `iterations` | Número máximo de árboles | 1000 (use parada temprana) |
| `learning_rate` | Peso de cada árbol | Automático según `iterations` y tamaño de datos |
| `depth` | Profundidad de los árboles simétricos | 6 (habitual 4–10) |
| `l2_leaf_reg` | Regularización L2 | 3 (habitual 1–10) |
| `random_strength` | Aleatoriedad en la elección de divisiones | 1 |
| `bagging_temperature` | Intensidad del muestreo bayesiano | 1 |
| `border_count` | Intervalos para variables numéricas | 254 en CPU |
| `one_hot_max_size` | Límite de categorías para usar *one-hot* | 2 |
| `max_ctr_complexity` | Máximo de categorías combinadas | 4 |
| `grow_policy` | `SymmetricTree` (defecto), `Depthwise`, `Lossguide` | — |
| `boosting_type` | `Ordered` o `Plain` | Automático |
| `thread_count` | Hilos (−1 = todos) | −1 |
| `task_type` | `"CPU"` o `"GPU"` | `"CPU"` |

> Empiece con los valores por defecto y parada temprana. Si necesita mejorar, ajuste en este orden: `learning_rate`/`iterations`, `depth`, `l2_leaf_reg`, `random_strength`, `bagging_temperature`.

---

## 9. Parada temprana y sobreajuste

```python
modelo = CatBoostClassifier(iterations=5000, learning_rate=0.03,
                            od_type="Iter", od_wait=150,          # o early_stopping_rounds=150
                            use_best_model=True,                   # Conserva solo hasta la mejor iteración
                            eval_metric="AUC")
modelo.fit(ent, eval_set=val)
modelo.get_evals_result()            # Historial de métricas
```

> `use_best_model=True` (por defecto si hay `eval_set`) recorta el modelo a la mejor iteración. Use un conjunto de validación distinto del de prueba final.

---

## 10. Funciones de pérdida y métricas

| Problema | `loss_function` | Métricas (`eval_metric`, `custom_metric`) |
|---|---|---|
| Binario | `Logloss`, `CrossEntropy` | `AUC`, `PRAUC`, `F1`, `Precision`, `Recall`, `BrierScore` |
| Multiclase | `MultiClass` | `MultiClass`, `Accuracy`, `TotalF1` |
| Multietiqueta | `MultiLogloss` | `HammingLoss` |
| Regresión | `RMSE`, `MAE`, `Huber:delta=1`, `MAPE` | `RMSE`, `MAE`, `R2`, `MAPE` |
| Cuantiles | `Quantile:alpha=0.9`, `MultiQuantile:alpha=0.1,0.5,0.9` | `Quantile` |
| Recuentos / importes | `Poisson`, `Tweedie:variance_power=1.5` | — |
| Varios objetivos de regresión | `MultiRMSE` | — |
| Ordenación | `YetiRank`, `PairLogit`, `QueryRMSE` | `NDCG`, `MAP` |
| Supervivencia | `SurvivalAft`, `Cox` | — |

```python
CatBoostClassifier(loss_function="Logloss", eval_metric="AUC", custom_metric=["PRAUC", "F1"])
modelo.eval_metrics(pool_test, ["AUC", "Logloss"])      # Métricas por iteración en otros datos
```

---

## 11. Validación cruzada y búsqueda de hiperparámetros

### 11.1 Validación cruzada nativa

```python
from catboost import cv

resultados = cv(Pool(X, y, cat_features=categoricas),
                {"iterations": 2000, "learning_rate": 0.05, "loss_function": "Logloss",
                 "eval_metric": "AUC", "verbose": 0},
                fold_count=5, stratified=True, early_stopping_rounds=100, logging_level="Silent")
resultados[["test-AUC-mean", "test-AUC-std"]].tail(1)
```

### 11.2 Búsqueda integrada

```python
modelo = CatBoostClassifier(iterations=500, verbose=0)
rejilla = {"depth": [4, 6, 8], "learning_rate": [0.03, 0.1], "l2_leaf_reg": [1, 3, 9]}
resultado = modelo.grid_search(rejilla, Pool(X, y, cat_features=categoricas), cv=3, plot=False, verbose=False)
resultado["params"]                 # Mejor combinación; el modelo queda entrenado con ella
```

También `randomized_search` y la integración con Optuna o `RandomizedSearchCV` de scikit-learn.

---

## 12. Clases desbalanceadas y calibración

```python
CatBoostClassifier(auto_class_weights="Balanced")        # o "SqrtBalanced"
CatBoostClassifier(class_weights={0: 1, 1: 5})
CatBoostClassifier(scale_pos_weight=5)
```

Las probabilidades de CatBoost con `Logloss` suelen estar **bien calibradas** (compruébelo por deciles, como en el ejemplo completo). Las ponderaciones de clase las descalibran: si necesita probabilidades reales, ajuste el umbral en lugar de ponderar.

---

## 13. Interpretación: importancia, SHAP e interacciones

```python
modelo.get_feature_importance(prettified=True)                        # PredictionValuesChange (DataFrame)
modelo.get_feature_importance(pool, type="LossFunctionChange")        # Impacto en la pérdida (necesita datos)
modelo.get_feature_importance(pool, type="ShapValues")                # SHAP por fila (última columna: valor base)
modelo.get_feature_importance(type="Interaction")                     # Pares de variables que interactúan

modelo.calc_feature_statistics(X, y, feature="antelacion_dias", plot=True)   # Objetivo y predicción por tramos
modelo.plot_tree(tree_idx=0, pool=pool)                               # Requiere graphviz
```

| Tipo de importancia | Significado |
|---|---|
| `PredictionValuesChange` | Cuánto cambia la predicción, de media, al variar la variable (por defecto) |
| `LossFunctionChange` | Cuánto empeora la pérdida sin la variable (más fiable; más lento) |
| `ShapValues` | Contribución de cada variable a cada predicción individual |
| `Interaction` | Fuerza de las interacciones entre pares de variables |

---

## 14. GPU y rendimiento

```python
CatBoostClassifier(task_type="GPU", devices="0")           # GPU NVIDIA
```

| Consejo | Detalle |
|---|---|
| GPU | Aceleraciones notables en conjuntos de más de ~100 000 filas |
| `border_count` menor | Entrenamiento más rápido |
| `boosting_type="Plain"` | Más rápido que `Ordered` en conjuntos grandes |
| `max_ctr_complexity=1` | Desactiva combinaciones de categorías (más rápido) |
| Predicción | Muy rápida gracias a los árboles simétricos; `thread_count` controla el paralelismo |

---

## 15. Guardar y desplegar modelos

```python
modelo.save_model("modelo.cbm")                              # Formato nativo binario
modelo = CatBoostClassifier().load_model("modelo.cbm")

modelo.save_model("modelo.json", format="json")              # Legible
modelo.save_model("modelo.onnx", format="onnx")              # ONNX (sin variables categóricas)
modelo.save_model("modelo.cpp", format="cpp")                # Código C++ / "python" para Python puro
```

CatBoost incluye además aplicadores para C/C++, Java y otros lenguajes, lo que facilita integrar el modelo en sistemas existentes.

---

## 16. Ejemplo práctico completo

**Predicción de cancelaciones de reservas hoteleras**: genera 60 000 reservas con nueve variables categóricas (algunas de alta cardinalidad, como 300 agencias y 70 países), variables numéricas, **una variable de texto** con las peticiones del cliente y valores ausentes. Entrena con `Pool`, parada temprana y `use_best_model`; compara el modelo **con y sin texto**; comprueba la **calibración** de las probabilidades por deciles; interpreta con importancia, **interacciones** y **valores SHAP**; y estima las llegadas esperadas de un hotel y mes para apoyar la política de sobreventa. Las categorías se usan **sin ninguna codificación previa**.

Requisitos: `pip install catboost scikit-learn pandas`

```python
"""
Predicción de cancelaciones de reservas hoteleras con CatBoost.

Datos con muchas variables categóricas (algunas de alta cardinalidad, como la
agencia o el país), una variable de texto (peticiones del cliente) y valores
ausentes, usados SIN codificación previa. Entrena con Pool, parada temprana y
el mejor modelo, compara con una versión sin texto, interpreta con SHAP e
interacciones y usa el modelo para decidir la política de sobreventa.
"""
import time

import numpy as np
import pandas as pd
from catboost import CatBoostClassifier, Pool, __version__ as version_catboost
from sklearn.metrics import brier_score_loss, log_loss, roc_auc_score
from sklearn.model_selection import train_test_split

SEMILLA = 11
CATEGORICAS = ["hotel", "pais", "agencia", "canal", "tipo_habitacion", "regimen", "tipo_deposito",
               "mes_llegada", "segmento"]
TEXTO = ["peticiones"]


# ------------------------------------------------------------------ Datos sintéticos
def generar_reservas(n: int = 60_000) -> pd.DataFrame:
    rng = np.random.default_rng(SEMILLA)
    paises = ["ES", "PT", "FR", "GB", "DE", "IT", "US", "NL", "BE", "IE"] + [f"P{i:02d}" for i in range(60)]
    p_pais = np.r_[[0.35, 0.15, 0.1, 0.1, 0.07, 0.05, 0.03, 0.03, 0.02, 0.02], np.full(60, 0.08 / 60)]
    agencias = [f"AG{i:03d}" for i in range(300)]
    riesgo_agencia = dict(zip(agencias, rng.normal(0, 0.8, 300)))           # Cada agencia, su riesgo
    peticiones_posibles = {"cuna": -0.2, "cama extra": -0.1, "parking": -0.3, "vistas al mar": 0.1,
                           "llegada tardía": 0.0, "aniversario": -0.6, "habitación tranquila": 0.0,
                           "reembolso si cancelo": 1.2, "confirmar precio": 0.7, "sin gluten": -0.2}
    df = pd.DataFrame({
        "hotel": rng.choice(["Urbano Madrid", "Playa Málaga", "Resort Canarias"], n, p=[0.45, 0.35, 0.2]),
        "pais": rng.choice(paises, n, p=p_pais / p_pais.sum()),
        "agencia": rng.choice(agencias, n),
        "canal": rng.choice(["web propia", "OTA", "agencia", "corporativo", "teléfono"], n,
                            p=[0.3, 0.4, 0.15, 0.1, 0.05]),
        "tipo_habitacion": rng.choice(list("ABCDEF"), n, p=[0.4, 0.2, 0.15, 0.12, 0.08, 0.05]),
        "regimen": rng.choice(["solo alojamiento", "desayuno", "media pensión", "todo incluido"], n),
        "tipo_deposito": rng.choice(["sin depósito", "reembolsable", "no reembolsable"], n,
                                    p=[0.75, 0.1, 0.15]),
        "mes_llegada": rng.choice(["ene", "feb", "mar", "abr", "may", "jun", "jul", "ago", "sep", "oct",
                                   "nov", "dic"], n),
        "segmento": rng.choice(["ocio", "negocios", "grupo"], n, p=[0.7, 0.2, 0.1]),
        "antelacion_dias": rng.gamma(1.5, 40, n).round(),
        "noches": rng.integers(1, 15, n),
        "adultos": rng.integers(1, 4, n),
        "ninos": rng.choice([0, 1, 2], n, p=[0.75, 0.15, 0.1]),
        "precio_noche": rng.lognormal(4.6, 0.35, n).round(2),
        "cancelaciones_previas": rng.choice([0, 1, 2, 3], n, p=[0.9, 0.06, 0.03, 0.01]),
        "cambios_reserva": rng.poisson(0.3, n),
    })
    textos, efecto_texto = [], np.zeros(n)
    for i in range(n):
        k = rng.choice([0, 1, 2], p=[0.5, 0.35, 0.15])
        elegidas = rng.choice(list(peticiones_posibles), k, replace=False)
        textos.append(", ".join(elegidas))
        efecto_texto[i] = sum(peticiones_posibles[p] for p in elegidas)
    df["peticiones"] = textos
    logit = (-1.6 + 0.012 * df["antelacion_dias"] + 1.2 * (df["canal"] == "OTA")
             - 1.0 * (df["canal"] == "corporativo") - 2.0 * (df["tipo_deposito"] == "no reembolsable")
             + 0.9 * df["cancelaciones_previas"] - 0.35 * df["cambios_reserva"]
             + df["agencia"].map(riesgo_agencia) * (df["canal"] == "agencia")
             + 0.5 * df["pais"].isin(["PT", "ES"]) - 0.4 * (df["segmento"] == "negocios")
             + 0.004 * (df["precio_noche"] - 100) + efecto_texto + rng.normal(0, 0.3, n))
    df["cancela"] = (rng.random(n) < 1 / (1 + np.exp(-logit))).astype(int)
    df.loc[rng.random(n) < 0.05, "pais"] = np.nan                     # Valores ausentes
    df.loc[df["canal"] != "agencia", "agencia"] = "sin agencia"
    df[CATEGORICAS] = df[CATEGORICAS].fillna("desconocido")           # CatBoost no admite NaN en categóricas
    return df


def entrenar(X_ent, y_ent, X_val, y_val, usar_texto: bool) -> CatBoostClassifier:
    columnas = [c for c in X_ent.columns if usar_texto or c not in TEXTO]
    pool_ent = Pool(X_ent[columnas], y_ent, cat_features=CATEGORICAS,
                    text_features=TEXTO if usar_texto else None)
    pool_val = Pool(X_val[columnas], y_val, cat_features=CATEGORICAS,
                    text_features=TEXTO if usar_texto else None)
    modelo = CatBoostClassifier(iterations=3000, learning_rate=0.06, depth=6, l2_leaf_reg=3,
                                loss_function="Logloss", eval_metric="AUC", od_type="Iter", od_wait=150,
                                use_best_model=True, random_seed=SEMILLA, verbose=0, thread_count=-1)
    modelo.fit(pool_ent, eval_set=pool_val)
    return modelo


def main() -> None:
    print(f"CatBoost {version_catboost}")
    df = generar_reservas()
    X, y = df.drop(columns="cancela"), df["cancela"]
    X_ent, X_test, y_ent, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=SEMILLA)
    X_ent, X_val, y_ent, y_val = train_test_split(X_ent, y_ent, test_size=0.2, stratify=y_ent,
                                                  random_state=SEMILLA)
    print(f"Reservas: {len(df):,} · tasa de cancelación {y.mean():.1%} · "
          f"agencias distintas {X['agencia'].nunique()} · países {X['pais'].nunique()}\n")

    # 1. Con y sin la variable de texto
    modelos = {}
    for usar_texto in (False, True):
        inicio = time.perf_counter()
        m = entrenar(X_ent, y_ent, X_val, y_val, usar_texto)
        columnas = m.feature_names_
        prob = m.predict_proba(X_test[columnas])[:, 1]
        nombre = "con texto" if usar_texto else "sin texto"
        modelos[nombre] = (m, prob)
        print(f"Modelo {nombre:<10} árboles {m.get_best_iteration() + 1:4d} · AUC {roc_auc_score(y_test, prob):.4f} "
              f"· log loss {log_loss(y_test, prob):.4f} · Brier {brier_score_loss(y_test, prob):.4f} "
              f"· {time.perf_counter() - inicio:.0f} s")
    modelo, prob = modelos["con texto"]

    # 2. Calibración: probabilidad media prevista frente a tasa real por decil
    deciles = pd.qcut(prob, 10, labels=False)
    calibracion = pd.DataFrame({"prevista": prob, "real": y_test.to_numpy(), "decil": deciles}) \
        .groupby("decil").mean()
    print("\nCalibración (decil: prevista → real): " + " · ".join(
        f"{d + 1}: {r.prevista:.0%}→{r.real:.0%}" for d, r in calibracion.iloc[::3].iterrows()))

    # 3. Importancia de variables e interacciones
    importancia = modelo.get_feature_importance(prettified=True)
    print("\nImportancia (PredictionValuesChange):")
    for _, fila in importancia.head(8).iterrows():
        print(f"  {fila['Feature Id']:<24}{fila['Importances']:6.1f}  {'█' * int(fila['Importances'] / 2)}")
    nombres = modelo.feature_names_
    interacciones = modelo.get_feature_importance(type="Interaction")[:3]
    print("Interacciones principales: " + " · ".join(
        f"{nombres[int(a)]} × {nombres[int(b)]} ({v:.1f})" for a, b, v in interacciones))

    # 4. Explicación de una reserva concreta con valores SHAP
    columnas = modelo.feature_names_
    pool_test = Pool(X_test[columnas], cat_features=CATEGORICAS, text_features=TEXTO)
    shap = modelo.get_feature_importance(pool_test, type="ShapValues")
    i = int(np.argmax(prob))
    print(f"\nReserva con mayor riesgo (prob. {prob[i]:.1%}, cancelada: {'sí' if y_test.iloc[i] else 'no'}):")
    detalle = pd.Series(shap[i, :-1], index=columnas).sort_values(key=abs, ascending=False)
    for variable, valor in detalle.head(5).items():
        print(f"  {variable:<22}= {str(X_test.iloc[i][variable])[:28]:<30}{valor:+.2f}")

    # 5. Uso de negocio: ocupación esperada para fijar la sobreventa
    llegadas = X_test.assign(prob=prob).query("hotel == 'Playa Málaga' and mes_llegada == 'ago'")
    esperadas = (1 - llegadas["prob"]).sum()
    print(f"\nPlaya Málaga · agosto · {len(llegadas)} reservas en el conjunto de prueba: "
          f"llegadas esperadas {esperadas:.0f} (reales {int((1 - y_test[llegadas.index]).sum())}) · "
          f"cancelaciones esperadas {len(llegadas) - esperadas:.0f}")

    modelo.save_model("modelo_cancelaciones.cbm")
    cargado = CatBoostClassifier().load_model("modelo_cancelaciones.cbm")
    print(f"Modelo guardado y recargado · predicciones idénticas: "
          f"{np.allclose(cargado.predict_proba(X_test[columnas])[:, 1], prob)}")


if __name__ == "__main__":
    main()
```

Resultado:

```
CatBoost 1.2.10
Reservas: 60,000 · tasa de cancelación 40.5% · agencias distintas 301 · países 71

Modelo sin texto  árboles  216 · AUC 0.7727 · log loss 0.5545 · Brier 0.1880 · 16 s
Modelo con texto  árboles  205 · AUC 0.7853 · log loss 0.5426 · Brier 0.1831 · 15 s

Calibración (decil: prevista → real): 1: 7%→6% · 4: 27%→27% · 7: 49%→50% · 10: 83%→85%

Importancia (PredictionValuesChange):
  canal                     25.1  ████████████
  antelacion_dias           18.0  █████████
  tipo_deposito             17.9  ████████
  peticiones                11.9  █████
  cancelaciones_previas     10.5  █████
  pais                       4.1  ██
  precio_noche               2.8  █
  agencia                    2.5  █
Interacciones principales: canal × tipo_deposito (2.6) · canal × antelacion_dias (2.4) · antelacion_dias × cancelaciones_previas (2.2)

Reserva con mayor riesgo (prob. 98.8%, cancelada: sí):
  cancelaciones_previas = 3                             +1.68
  antelacion_dias       = 274.0                         +1.51
  tipo_deposito         = sin depósito                  +0.31
  peticiones            = reembolso si cancelo, parkin  +0.30
  regimen               = desayuno                      +0.22

Playa Málaga · agosto · 364 reservas en el conjunto de prueba: llegadas esperadas 224 (reales 228) · cancelaciones esperadas 140
Modelo guardado y recargado · predicciones idénticas: True
```

> La variable de texto mejora la AUC de 0,773 a 0,785: CatBoost extrae por sí mismo la señal de frases como «reembolso si cancelo». Las probabilidades están bien calibradas (en el decil de mayor riesgo se prevé un 83 % de cancelaciones y cancela un 85 %), lo que permite usarlas directamente para estimar la ocupación: para las 364 reservas de agosto se esperaban 224 llegadas y llegaron 228.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `CatBoostError: Invalid type for cat_feature ... =nan : cat_features must be integer or string` | Valores ausentes o `float` en una variable categórica. | `df[col].fillna("desconocido").astype(str)`. |
| `CatBoostError: ... features data: Feature 'x' is Categorical in model but numeric in data` | Tipos distintos entre entrenamiento y predicción. | Use las mismas columnas y tipos (o el mismo `Pool`). |
| Rendimiento pobre con categorías | Se aplicó *one-hot* o *label encoding* antes. | Pase las categorías en bruto con `cat_features`. |
| `CatBoostError: Text features are not supported for ...` | Texto en una función de pérdida que no lo admite. | Use clasificación o convierta el texto a *embeddings*. |
| Demasiados mensajes en consola | `verbose` por defecto. | `verbose=0` o `logging_level="Silent"`. |
| Se crea la carpeta `catboost_info` | Registros de entrenamiento por defecto. | `train_dir="ruta"` o `allow_writing_files=False`. |
| `get_best_iteration()` devuelve `None` | Se entrenó sin `eval_set`. | Proporcione un conjunto de validación. |
| Entrenamiento lento | Muchas categorías combinadas o datos grandes en CPU. | `max_ctr_complexity=1`, `boosting_type="Plain"`, GPU. |
| Probabilidades descalibradas | Se usó `auto_class_weights` o `scale_pos_weight`. | Entrene sin ponderar y ajuste el umbral. |

---

## 18. Buenas prácticas

- Pase las **variables categóricas en bruto** (texto) y declárelas en `cat_features`; sustituya solo los ausentes.
- Empiece con los **parámetros por defecto** y parada temprana antes de ajustar nada.
- Use `Pool` para mantener juntos datos, tipos de columna y pesos.
- Aproveche las variables de **texto** cuando aporten información (comentarios, descripciones, motivos).
- Revise la **calibración** si va a usar las probabilidades para estimar cantidades (ocupación, pérdidas esperadas).
- Interprete con `LossFunctionChange` y SHAP, y revise las interacciones principales con el negocio.
- Guarde el modelo con `save_model` en formato `.cbm` junto con la lista de columnas y sus tipos.
- Compare con LightGBM y XGBoost si el tiempo de entrenamiento es crítico.

---

## 19. Referencia rápida

```python
from catboost import CatBoostClassifier, CatBoostRegressor, Pool, cv

m = CatBoostClassifier(iterations=3000, learning_rate=0.05, depth=6, l2_leaf_reg=3, loss_function="Logloss",
                       eval_metric="AUC", early_stopping_rounds=150, use_best_model=True, random_seed=42, verbose=0)
p = Pool(X, y, cat_features=[...], text_features=[...], embedding_features=[...], weight=w)
m.fit(p, eval_set=Pool(Xv, yv, cat_features=[...])) · m.predict_proba(X)[:, 1] · m.get_best_iteration()
CatBoostRegressor(loss_function="RMSE" | "MAE" | "Quantile:alpha=0.9" | "MultiQuantile:alpha=0.1,0.9" | "Tweedie:variance_power=1.5")

cv(p, params, fold_count=5, stratified=True, early_stopping_rounds=100) · m.grid_search(rejilla, p, cv=3)
auto_class_weights="Balanced" · task_type="GPU"
m.get_feature_importance(prettified=True) · get_feature_importance(p, type="ShapValues" | "LossFunctionChange" | "Interaction")
m.save_model("m.cbm") · CatBoostClassifier().load_model("m.cbm") · save_model("m.json", format="json")
```

---

## 20. Ejercicios propuestos

1. Compare CatBoost (categorías en bruto), LightGBM (categorías nativas) y XGBoost (categorías nativas) en el ejemplo completo: AUC y tiempo de entrenamiento.
2. Repita el ejemplo aplicando *one-hot* a las categorías antes de CatBoost y mida cuánto empeora el modelo.
3. Sustituya la variable de texto por *embeddings* generados con un modelo de Hugging Face y compare resultados.
4. Entrene un `CatBoostRegressor` con `MultiQuantile` para prever el precio por noche con intervalos P10–P90.
5. Use las probabilidades del ejemplo para calcular, por hotel y mes, cuántas reservas adicionales podrían aceptarse manteniendo un riesgo de sobreventa inferior al 5 %.

---

## 21. Referencias

- Documentación oficial: https://catboost.ai/docs/
- Variables categóricas: https://catboost.ai/docs/en/features/categorical-features
- Variables de texto: https://catboost.ai/docs/en/features/text-features
- Parámetros: https://catboost.ai/docs/en/references/training-parameters/
- Prokhorenkova, L. et al. (2018). *CatBoost: unbiased boosting with categorical features*. NeurIPS.
