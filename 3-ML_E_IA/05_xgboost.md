# Manual de uso: XGBoost

> **Categoría:** Inteligencia artificial · **Utilidad:** Modelos predictivos basados en árboles
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** Apache 2.0 · **Núcleo:** C++ con interfaces Python, R, Java, Scala

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [API de scikit-learn: `XGBClassifier` y `XGBRegressor`](#4-api-de-scikit-learn-xgbclassifier-y-xgbregressor)
5. [API nativa: `DMatrix` y `xgb.train`](#5-api-nativa-dmatrix-y-xgbtrain)
6. [Hiperparámetros](#6-hiperparámetros)
7. [Parada temprana y validación cruzada](#7-parada-temprana-y-validación-cruzada)
8. [Variables categóricas y valores ausentes](#8-variables-categóricas-y-valores-ausentes)
9. [Clases desbalanceadas](#9-clases-desbalanceadas)
10. [Objetivos y métricas](#10-objetivos-y-métricas)
11. [Restricciones monótonas y de interacción](#11-restricciones-monótonas-y-de-interacción)
12. [Ajuste de hiperparámetros](#12-ajuste-de-hiperparámetros)
13. [Interpretación: importancia y SHAP](#13-interpretación-importancia-y-shap)
14. [GPU, rendimiento y grandes volúmenes](#14-gpu-rendimiento-y-grandes-volúmenes)
15. [Guardar y desplegar modelos](#15-guardar-y-desplegar-modelos)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**XGBoost** (*eXtreme Gradient Boosting*) es una implementación optimizada de **gradient boosting** sobre árboles de decisión. Construye cientos o miles de árboles pequeños de forma secuencial, cada uno corrigiendo los errores de los anteriores. Es uno de los algoritmos más usados para **datos tabulares**, en competiciones (Kaggle) y en producción (riesgo de crédito, fraude, precios, demanda, *marketing*).

**Ventajas:**
- Rendimiento predictivo excelente en datos tabulares, normalmente superior a modelos lineales y bosques aleatorios.
- Regularización integrada (L1, L2, poda) que limita el sobreajuste.
- Trata los **valores ausentes** de forma nativa y admite **variables categóricas**.
- Muy rápido (algoritmo de histogramas, multihilo, GPU) y escalable (Spark, Dask).
- Restricciones monótonas e interpretación con valores SHAP integrados.
- Compatible con scikit-learn (*pipelines*, validación cruzada, búsqueda de hiperparámetros).

**Limitaciones:**
- Muchos hiperparámetros: requiere ajuste para obtener el máximo rendimiento.
- No es adecuado para imágenes, audio o texto en bruto (use redes neuronales).
- No extrapola: fuera del rango de los datos de entrenamiento, la predicción se mantiene constante.
- Menos interpretable que un modelo lineal o un árbol único (se compensa con SHAP).

| Si necesita… | Opción |
|---|---|
| Máximo rendimiento en datos tabulares, ecosistema muy maduro | **XGBoost** |
| Entrenamiento más rápido en conjuntos muy grandes | LightGBM |
| Muchas variables categóricas con poco preprocesado | CatBoost |
| Modelo sencillo e interpretable | scikit-learn (regresión logística, árbol) |
| Imágenes, texto, audio | PyTorch / TensorFlow / Transformers |

---

## 2. Instalación e importación

```bash
pip install xgboost                   # Incluye soporte de GPU NVIDIA en Linux y Windows
conda install -c conda-forge xgboost
```

Comprobar la instalación:

```bash
python -c "import xgboost as xgb; print(xgb.__version__)"
```

Importaciones habituales:

```python
import xgboost as xgb
import numpy as np
import pandas as pd
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| *Boosting* | Se entrenan modelos débiles (árboles poco profundos) en secuencia; cada uno corrige los errores residuales del conjunto anterior. |
| Árbol / ronda (*boosting round*) | Cada iteración añade un árbol (`n_estimators` = número máximo de árboles). |
| Tasa de aprendizaje (`learning_rate`, `eta`) | Peso de cada árbol nuevo. Valores bajos requieren más árboles pero generalizan mejor. |
| Objetivo (`objective`) | Función de pérdida que se optimiza (`binary:logistic`, `reg:squarederror`…). |
| Métrica de evaluación (`eval_metric`) | Métrica que se vigila en validación (`auc`, `logloss`, `rmse`…). |
| Parada temprana | Detener el entrenamiento cuando la métrica de validación deja de mejorar. |
| `DMatrix` | Estructura de datos optimizada de la API nativa. |
| *Booster* | Modelo entrenado (conjunto de árboles). |
| Ganancia (*gain*) | Mejora de la pérdida que aporta una división; base de la importancia de variables. |

**Cómo funciona (simplificado):**

```
Predicción inicial (p. ej. la media)
Para cada ronda:
    1. Calcular el gradiente de la pérdida para cada muestra (cuánto y en qué dirección se equivoca)
    2. Ajustar un árbol que prediga esos gradientes
    3. Sumar el árbol al modelo, multiplicado por learning_rate
Predicción final = suma de todos los árboles
```

---

## 4. API de scikit-learn: `XGBClassifier` y `XGBRegressor`

### 4.1 Clasificación

```python
modelo = xgb.XGBClassifier(
    n_estimators=1000,
    learning_rate=0.05,
    max_depth=6,
    subsample=0.8,
    colsample_bytree=0.8,
    early_stopping_rounds=50,       # Parada temprana (requiere eval_set)
    eval_metric="auc",
    random_state=42,
    n_jobs=-1,
)
modelo.fit(X_ent, y_ent, eval_set=[(X_val, y_val)], verbose=100)

modelo.predict(X_test)               # Clases
modelo.predict_proba(X_test)[:, 1]   # Probabilidad de la clase positiva
modelo.best_iteration                # Mejor ronda (0-indexada)
```

### 4.2 Regresión

```python
regresor = xgb.XGBRegressor(n_estimators=500, learning_rate=0.05, max_depth=5,
                            early_stopping_rounds=30, eval_metric="rmse")
regresor.fit(X_ent, y_ent, eval_set=[(X_val, y_val)], verbose=False)
regresor.predict(X_test)
```

### 4.3 Otras clases

| Clase | Uso |
|---|---|
| `XGBClassifier` | Clasificación binaria y multiclase |
| `XGBRegressor` | Regresión (también cuantiles, Poisson, Tweedie…) |
| `XGBRanker` | Ordenación (*learning to rank*): buscadores, recomendadores |
| `XGBRFClassifier` / `XGBRFRegressor` | Bosque aleatorio con el motor de XGBoost |

Al ser compatibles con scikit-learn, funcionan en `Pipeline`, `cross_validate`, `GridSearchCV`, etc.

---

## 5. API nativa: `DMatrix` y `xgb.train`

Ofrece más control y funciones adicionales (validación cruzada nativa, objetivos personalizados):

```python
dtrain = xgb.DMatrix(X_ent, label=y_ent, enable_categorical=True)
dval = xgb.DMatrix(X_val, label=y_val, enable_categorical=True)

parametros = {"objective": "binary:logistic", "eval_metric": "auc", "eta": 0.05,
              "max_depth": 6, "subsample": 0.8, "colsample_bytree": 0.8, "tree_method": "hist"}

booster = xgb.train(parametros, dtrain, num_boost_round=1000,
                    evals=[(dtrain, "ent"), (dval, "val")],
                    early_stopping_rounds=50, verbose_eval=100)

booster.predict(xgb.DMatrix(X_test, enable_categorical=True),
                iteration_range=(0, booster.best_iteration + 1))
```

| API scikit-learn | API nativa |
|---|---|
| `n_estimators` | `num_boost_round` |
| `learning_rate` | `eta` (también acepta `learning_rate`) |
| `modelo.get_booster()` | `booster` |

---

## 6. Hiperparámetros

### 6.1 Principales

| Parámetro | Efecto | Valores habituales |
|---|---|---|
| `n_estimators` | Número máximo de árboles | 100–5000 (con parada temprana) |
| `learning_rate` | Contribución de cada árbol | 0,01–0,3 |
| `max_depth` | Profundidad de cada árbol (complejidad) | 3–10 |
| `min_child_weight` | Peso mínimo en una hoja (evita hojas con pocas muestras) | 1–20 |
| `subsample` | Fracción de filas por árbol | 0,5–1 |
| `colsample_bytree` | Fracción de columnas por árbol | 0,5–1 |
| `gamma` (`min_split_loss`) | Ganancia mínima para dividir | 0–5 |
| `reg_lambda` / `reg_alpha` | Regularización L2 / L1 de los pesos de las hojas | 0–10 |
| `tree_method` | Algoritmo de construcción | `"hist"` (por defecto) |
| `max_bin` | Número de intervalos de los histogramas | 256 (por defecto) |
| `device` | `"cpu"` o `"cuda"` | — |

### 6.2 Cómo controlar el sobreajuste

| Síntoma | Ajuste |
|---|---|
| Gran diferencia entre entrenamiento y validación | ↓ `max_depth`, ↑ `min_child_weight`, ↑ `reg_lambda`, ↓ `subsample`/`colsample_bytree` |
| El modelo no aprende lo suficiente | ↑ `max_depth`, ↑ `n_estimators`, ↓ regularización |
| Entrenamiento lento | ↑ `learning_rate` (con menos árboles), ↓ `max_bin`, GPU |

> Regla práctica: fije `learning_rate` bajo (0,05) con muchos árboles y parada temprana; ajuste después `max_depth`, `min_child_weight` y el muestreo.

---

## 7. Parada temprana y validación cruzada

### 7.1 Parada temprana

```python
modelo = xgb.XGBClassifier(n_estimators=5000, learning_rate=0.03, early_stopping_rounds=100, eval_metric="logloss")
modelo.fit(X_ent, y_ent, eval_set=[(X_val, y_val)], verbose=False)
print(modelo.best_iteration, modelo.best_score)
modelo.evals_result()                     # Historial de la métrica por ronda
```

Al predecir, XGBoost usa automáticamente los árboles hasta `best_iteration`.

> El conjunto de validación usado para la parada temprana **no** debe ser el de prueba final: estaría influyendo en el modelo.

### 7.2 Validación cruzada nativa

```python
resultados = xgb.cv(parametros, dtrain, num_boost_round=2000, nfold=5, stratified=True,
                    early_stopping_rounds=50, seed=42)
resultados.tail(1)             # DataFrame con media y desviación por ronda
n_optimo = len(resultados)
```

---

## 8. Variables categóricas y valores ausentes

### 8.1 Categorías nativas

```python
for col in ["provincia", "canal", "producto"]:
    df[col] = df[col].astype("category")              # Tipo category de pandas

modelo = xgb.XGBClassifier(enable_categorical=True, tree_method="hist", max_cat_to_onehot=4)
modelo.fit(df[variables], df["objetivo"])
```

XGBoost agrupa categorías de forma óptima en cada división, sin necesidad de *one-hot*. Las categorías deben ser **las mismas** (mismo orden en `cat.categories`) en entrenamiento y predicción.

### 8.2 Valores ausentes

XGBoost los gestiona automáticamente: en cada división aprende hacia qué rama enviar las muestras con valor ausente. **No es necesario imputar.**

```python
xgb.DMatrix(X, missing=np.nan)          # Valor que representa ausente (por defecto NaN)
```

---

## 9. Clases desbalanceadas

| Técnica | Cómo |
|---|---|
| Ponderar la clase positiva | `scale_pos_weight = n_negativos / n_positivos` (o su raíz cuadrada) |
| Pesos por muestra | `modelo.fit(X, y, sample_weight=pesos)` |
| Métrica adecuada | `eval_metric="aucpr"` (área bajo la curva precisión-sensibilidad) |
| Ajustar el umbral | Elegir el umbral según el coste de cada error, no 0,5 |
| Estratificar | `stratify=y` en las particiones y `stratified=True` en `xgb.cv` |

> `scale_pos_weight` mejora la detección de la clase minoritaria pero **descalibra las probabilidades** (las infla). Si necesita probabilidades reales, entrene sin ponderar y ajuste el umbral, o calibre con `CalibratedClassifierCV`.

---

## 10. Objetivos y métricas

| Problema | `objective` | Métricas (`eval_metric`) |
|---|---|---|
| Clasificación binaria | `binary:logistic` | `logloss`, `auc`, `aucpr`, `error` |
| Multiclase | `multi:softprob` (con `num_class`) | `mlogloss`, `merror` |
| Regresión | `reg:squarederror` | `rmse`, `mae` |
| Regresión robusta | `reg:pseudohubererror`, `reg:absoluteerror` | `mae`, `mphe` |
| Cuantiles | `reg:quantileerror` + `quantile_alpha` | `quantile` |
| Recuentos | `count:poisson` | `poisson-nloglik` |
| Importes con muchos ceros (siniestros) | `reg:tweedie` | `tweedie-nloglik` |
| Tiempo hasta un evento | `survival:cox`, `survival:aft` | `cox-nloglik`, `aft-nloglik` |
| Ordenación | `rank:ndcg`, `rank:pairwise` | `ndcg`, `map` |

**Intervalos de predicción con varios cuantiles a la vez:**

```python
modelo = xgb.XGBRegressor(objective="reg:quantileerror", quantile_alpha=np.array([0.1, 0.5, 0.9]))
modelo.fit(X_ent, y_ent)
p10, p50, p90 = modelo.predict(X_test).T
```

---

## 11. Restricciones monótonas y de interacción

### 11.1 Restricciones monótonas

Obligan a que la predicción solo aumente (`1`) o solo disminuya (`-1`) al crecer una variable. Aportan coherencia con el conocimiento del negocio y suelen ser un requisito regulatorio (riesgo de crédito, tarificación):

```python
modelo = xgb.XGBClassifier(monotone_constraints={"deuda_ingresos": 1, "antiguedad_laboral": -1})
```

### 11.2 Restricciones de interacción

Limitan qué variables pueden combinarse en un mismo árbol:

```python
modelo = xgb.XGBRegressor(interaction_constraints=[["edad", "ingresos"], ["provincia", "canal"]])
```

---

## 12. Ajuste de hiperparámetros

### 12.1 Con scikit-learn

```python
from scipy.stats import loguniform, randint, uniform
from sklearn.model_selection import RandomizedSearchCV

espacio = {"max_depth": randint(3, 10), "min_child_weight": randint(1, 20),
           "subsample": uniform(0.5, 0.5), "colsample_bytree": uniform(0.5, 0.5),
           "reg_lambda": loguniform(1e-2, 10), "learning_rate": loguniform(0.01, 0.2)}
busqueda = RandomizedSearchCV(xgb.XGBClassifier(n_estimators=400), espacio, n_iter=40,
                              scoring="roc_auc", cv=5, n_jobs=1, random_state=0)
busqueda.fit(X_ent, y_ent)
```

> XGBoost ya usa varios hilos: combine `n_jobs=1` en la búsqueda con `n_jobs=-1` en el modelo (o al revés) para no saturar la CPU.

### 12.2 Con Optuna (optimización bayesiana)

```python
import optuna

def objetivo(trial):
    parametros = {"max_depth": trial.suggest_int("max_depth", 3, 10),
                  "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.2, log=True),
                  "subsample": trial.suggest_float("subsample", 0.5, 1.0),
                  "min_child_weight": trial.suggest_int("min_child_weight", 1, 20)}
    m = xgb.XGBClassifier(n_estimators=2000, early_stopping_rounds=50, eval_metric="auc", **parametros)
    m.fit(X_ent, y_ent, eval_set=[(X_val, y_val)], verbose=False)
    return m.best_score

estudio = optuna.create_study(direction="maximize")
estudio.optimize(objetivo, n_trials=50)
```

---

## 13. Interpretación: importancia y SHAP

### 13.1 Importancia de variables

```python
modelo.feature_importances_                                       # Según importance_type del modelo
modelo.get_booster().get_score(importance_type="gain")            # gain, weight, cover, total_gain
xgb.plot_importance(modelo, importance_type="gain", max_num_features=15)   # Requiere matplotlib
```

| Tipo | Significado |
|---|---|
| `gain` | Mejora media de la pérdida en las divisiones que usan la variable (**recomendado**) |
| `weight` | Número de veces que se usa la variable para dividir |
| `cover` | Número medio de muestras afectadas por esas divisiones |

### 13.2 Valores SHAP integrados

Descomponen cada predicción en la contribución de cada variable (en log-odds para clasificación):

```python
contribuciones = modelo.get_booster().predict(xgb.DMatrix(X_test, enable_categorical=True),
                                              pred_contribs=True)
# Matriz (n_muestras, n_variables + 1): la última columna es el valor base
```

Con la librería **SHAP** (`pip install shap`) se obtienen gráficos de resumen y de dependencia:

```python
import shap
explicador = shap.TreeExplainer(modelo)
valores = explicador(X_test)
shap.plots.beeswarm(valores)
```

### 13.3 Ver los árboles

```python
modelo.get_booster().trees_to_dataframe()       # Todas las divisiones en un DataFrame
xgb.plot_tree(modelo, tree_idx=0)               # Requiere graphviz
```

---

## 14. GPU, rendimiento y grandes volúmenes

```python
modelo = xgb.XGBClassifier(device="cuda", tree_method="hist")    # GPU NVIDIA
```

| Técnica | Cuándo |
|---|---|
| `tree_method="hist"` | Siempre (por defecto); rápido y con poca memoria |
| `device="cuda"` | Conjuntos grandes; aceleraciones de 5–20× |
| `xgb.QuantileDMatrix` | Reduce memoria al construir los histogramas |
| Memoria externa (`xgb.DataIter`) | Datos que no caben en memoria |
| `xgboost.dask` / `xgboost.spark` | Entrenamiento distribuido en clústeres |

---

## 15. Guardar y desplegar modelos

```python
modelo.save_model("modelo.json")            # Formato JSON (o .ubj binario), portable entre versiones
nuevo = xgb.XGBClassifier()
nuevo.load_model("modelo.json")
```

> Prefiera `save_model` en JSON/UBJ frente a `pickle`/`joblib`: es estable entre versiones de XGBoost y no ejecuta código al cargar. Si el modelo forma parte de un `Pipeline` de scikit-learn, guarde el *pipeline* con `joblib` y fije la versión de las librerías.

Opciones de despliegue: API REST (FastAPI), servicios gestionados en la nube, conversión a ONNX (`onnxmltools`) o compilación a código nativo (Treelite / TL2cgen).

---

## 16. Ejemplo práctico completo

**Detección de fraude en pagos con tarjeta**: genera 200 000 operaciones con un 2,5 % de fraude y variables numéricas y categóricas; usa **validación cruzada nativa** con parada temprana; compara tres ponderaciones de la clase minoritaria con `XGBClassifier`, **categorías nativas** y **restricciones monótonas**; elige el **umbral que minimiza el coste** (fraude no detectado frente a coste de revisión); evalúa en prueba; explica el modelo con importancia por ganancia y **contribuciones SHAP** de la operación más sospechosa; y guarda el modelo en JSON.

Requisitos: `pip install xgboost scikit-learn pandas`

```python
"""
Detección de fraude en pagos con tarjeta con XGBoost.

Datos muy desbalanceados (≈2,5 % de fraude) con variables numéricas y categóricas.
Entrena con la API de scikit-learn, categorías nativas, parada temprana,
ponderación de la clase minoritaria y restricciones monótonas; ajusta los
hiperparámetros con validación cruzada nativa; elige el umbral por coste;
explica las predicciones con contribuciones SHAP y guarda el modelo en JSON.
"""
import time

import numpy as np
import pandas as pd
import xgboost as xgb
from sklearn.metrics import average_precision_score, confusion_matrix, roc_auc_score
from sklearn.model_selection import train_test_split

SEMILLA = 7
COSTE_FRAUDE_NO_DETECTADO = 1.0     # Se pierde el importe de la operación
COSTE_REVISION = 3.0                # € por operación bloqueada y revisada manualmente


# ------------------------------------------------------------------ Datos sintéticos
def generar_pagos(n: int = 200_000, semilla: int = SEMILLA) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    comercio = rng.choice(["supermercado", "gasolinera", "online", "restaurante", "electrónica",
                           "viajes", "joyería", "farmacia"], n,
                          p=[0.25, 0.12, 0.25, 0.15, 0.07, 0.06, 0.03, 0.07])
    canal = rng.choice(["chip", "contactless", "online", "banda"], n, p=[0.35, 0.35, 0.27, 0.03])
    pais = rng.choice(["ES", "FR", "PT", "DE", "US", "otros"], n, p=[0.82, 0.05, 0.04, 0.03, 0.03, 0.03])
    importe = np.round(rng.lognormal(3.4, 1.0, n), 2)
    hora = rng.integers(0, 24, n)
    distancia_km = rng.exponential(15, n) * np.where(pais == "ES", 1, 40)
    ops_24h = rng.poisson(2.5, n)
    antiguedad_tarjeta = rng.integers(1, 120, n)
    ratio_importe = importe / rng.lognormal(3.4, 0.4, n)        # Importe / gasto medio del cliente
    logit = (-8.0 + 2.2 * np.log1p(ratio_importe) + 1.3 * (canal == "online") + 2.2 * (canal == "banda")
             + 1.8 * np.isin(comercio, ["electrónica", "joyería", "viajes"]) + 0.9 * (pais != "ES")
             + 1.5 * ((hora >= 1) & (hora <= 5)) + 0.5 * np.clip(ops_24h - 3, 0, None)
             + 0.6 * np.log1p(distancia_km / 50) - 0.02 * antiguedad_tarjeta)
    fraude = rng.random(n) < 1 / (1 + np.exp(-logit))
    df = pd.DataFrame({"importe": importe, "hora": hora, "distancia_km": distancia_km.round(1),
                       "ops_24h": ops_24h, "antiguedad_tarjeta_meses": antiguedad_tarjeta,
                       "ratio_importe": ratio_importe.round(3), "comercio": comercio, "canal": canal,
                       "pais": pais, "fraude": fraude.astype(int)})
    for col in ["comercio", "canal", "pais"]:
        df[col] = df[col].astype("category")                      # Categorías nativas en XGBoost
    return df


def coste_total(y, prob, importes, umbral) -> float:
    bloqueadas = prob >= umbral
    no_detectado = importes[(y == 1) & ~bloqueadas].sum() * COSTE_FRAUDE_NO_DETECTADO
    return no_detectado + bloqueadas.sum() * COSTE_REVISION


def main() -> None:
    print(f"XGBoost {xgb.__version__}")
    df = generar_pagos()
    X, y = df.drop(columns="fraude"), df["fraude"]
    X_ent, X_test, y_ent, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=SEMILLA)
    X_ent, X_val, y_ent, y_val = train_test_split(X_ent, y_ent, test_size=0.2, stratify=y_ent,
                                                  random_state=SEMILLA)
    print(f"Operaciones: {len(df):,} · fraude {y.mean():.2%} · "
          f"entrenamiento {len(y_ent):,} / validación {len(y_val):,} / prueba {len(y_test):,}\n")

    # 1. Validación cruzada nativa para fijar el número de árboles
    peso_positivos = (y_ent == 0).sum() / (y_ent == 1).sum()
    parametros = {"objective": "binary:logistic", "eval_metric": "aucpr", "tree_method": "hist",
                  "learning_rate": 0.05, "max_depth": 5, "subsample": 0.8, "colsample_bytree": 0.8,
                  "min_child_weight": 5, "reg_lambda": 1.0, "seed": SEMILLA}
    dtrain = xgb.DMatrix(X_ent, label=y_ent, enable_categorical=True)
    inicio = time.perf_counter()
    cv = xgb.cv(parametros, dtrain, num_boost_round=2000, nfold=5, stratified=True,
                early_stopping_rounds=50, seed=SEMILLA)
    n_arboles = len(cv)
    print(f"CV 5 particiones: {n_arboles} árboles · PR AUC {cv['test-aucpr-mean'].iloc[-1]:.4f} "
          f"± {cv['test-aucpr-std'].iloc[-1]:.4f} · {time.perf_counter() - inicio:.0f} s")

    # 2. Comparar ponderaciones de la clase minoritaria con la API de scikit-learn
    restricciones = {"ratio_importe": 1, "ops_24h": 1}          # Más importe/operaciones ⇒ no menos riesgo
    modelos = {}
    for nombre, peso in [("sin ponderar", 1.0), ("√ ratio", np.sqrt(peso_positivos)),
                         ("ratio completo", peso_positivos)]:
        modelo = xgb.XGBClassifier(
            n_estimators=2000, learning_rate=0.05, max_depth=5, subsample=0.8, colsample_bytree=0.8,
            min_child_weight=5, scale_pos_weight=peso, monotone_constraints=restricciones,
            enable_categorical=True, tree_method="hist", eval_metric="aucpr",
            early_stopping_rounds=50, random_state=SEMILLA, n_jobs=-1)
        modelo.fit(X_ent, y_ent, eval_set=[(X_val, y_val)], verbose=False)
        prob_val = modelo.predict_proba(X_val)[:, 1]
        modelos[nombre] = modelo
        print(f"  scale_pos_weight {nombre:<15} ({peso:5.1f}): árboles {modelo.best_iteration + 1:4d} · "
              f"PR AUC val {average_precision_score(y_val, prob_val):.4f}")
    modelo = max(modelos.values(),
                 key=lambda m: average_precision_score(y_val, m.predict_proba(X_val)[:, 1]))

    # 3. Umbral que minimiza el coste esperado (en validación)
    prob_val = modelo.predict_proba(X_val)[:, 1]
    umbrales = np.linspace(0.01, 0.99, 99)
    costes = [coste_total(y_val.values, prob_val, X_val["importe"].values, u) for u in umbrales]
    umbral = umbrales[int(np.argmin(costes))]

    # 4. Evaluación final en prueba
    prob = modelo.predict_proba(X_test)[:, 1]
    pred = (prob >= umbral).astype(int)
    vn, fp, fn, vp = confusion_matrix(y_test, pred).ravel()
    fraude_total = X_test["importe"][y_test == 1].sum()
    fraude_evitado = X_test["importe"][(y_test == 1) & (pred == 1)].sum()
    print(f"\nPrueba · ROC AUC {roc_auc_score(y_test, prob):.4f} · PR AUC "
          f"{average_precision_score(y_test, prob):.4f} (azar: {y_test.mean():.4f})")
    print(f"Umbral por coste: {umbral:.2f} · fraudes detectados {vp}/{vp + fn} ({vp / (vp + fn):.0%}) · "
          f"operaciones legítimas bloqueadas {fp} ({fp / (vn + fp):.2%})")
    print(f"Importe defraudado evitado: {fraude_evitado:,.0f} € de {fraude_total:,.0f} € · coste total "
          f"{coste_total(y_test.values, prob, X_test['importe'].values, umbral):,.0f} € (sin modelo: "
          f"{fraude_total:,.0f} €)")

    # 5. Importancia global y explicación individual (contribuciones SHAP nativas)
    importancia = pd.Series(modelo.get_booster().get_score(importance_type="gain")).sort_values(ascending=False)
    print("\nImportancia (ganancia media):")
    for variable, valor in (importancia / importancia.sum()).items():
        print(f"  {variable:<26}{valor:6.1%}  {'█' * int(valor * 60)}")

    dtest = xgb.DMatrix(X_test, enable_categorical=True)
    contribuciones = modelo.get_booster().predict(dtest, pred_contribs=True,
                                                   iteration_range=(0, modelo.best_iteration + 1))
    i = int(np.argmax(prob))
    print(f"\nOperación más sospechosa (prob. {prob[i]:.1%}, fraude real: {'sí' if y_test.iloc[i] else 'no'}):")
    detalle = pd.Series(contribuciones[i, :-1], index=X_test.columns).sort_values(key=abs, ascending=False)
    for variable, valor in detalle.head(5).items():
        print(f"  {variable:<26}= {str(X_test.iloc[i][variable]):<14} {valor:+.2f} (log-odds)")

    # 6. Guardar y cargar (formato JSON, portable entre versiones)
    modelo.save_model("modelo_fraude.json")
    cargado = xgb.XGBClassifier()
    cargado.load_model("modelo_fraude.json")
    print(f"\nModelo guardado y recargado · predicciones idénticas: "
          f"{np.allclose(cargado.predict_proba(X_test)[:, 1], prob)}")


if __name__ == "__main__":
    main()
```

Resultado:

```
XGBoost 3.4.1
Operaciones: 200,000 · fraude 2.52% · entrenamiento 128,000 / validación 32,000 / prueba 40,000

CV 5 particiones: 138 árboles · PR AUC 0.4184 ± 0.0128 · 14 s
  scale_pos_weight sin ponderar    (  1.0): árboles  178 · PR AUC val 0.4196
  scale_pos_weight √ ratio         (  6.2): árboles  190 · PR AUC val 0.4210
  scale_pos_weight ratio completo  ( 38.8): árboles  262 · PR AUC val 0.4211

Prueba · ROC AUC 0.9253 · PR AUC 0.4369 (azar: 0.0251)
Umbral por coste: 0.49 · fraudes detectados 830/1006 (83%) · operaciones legítimas bloqueadas 5049 (12.95%)
Importe defraudado evitado: 136,820 € de 145,927 € · coste total 26,744 € (sin modelo: 145,927 €)

Importancia (ganancia media):
  ratio_importe              26.5%  ███████████████
  pais                       17.7%  ██████████
  canal                      12.7%  ███████
  distancia_km                9.1%  █████
  comercio                    9.0%  █████
  ops_24h                     7.4%  ████
  hora                        6.6%  ███
  importe                     5.5%  ███
  antiguedad_tarjeta_meses    5.5%  ███

Operación más sospechosa (prob. 99.7%, fraude real: sí):
  ratio_importe             = 9.028          +1.92 (log-odds)
  distancia_km              = 1790.4         +1.39 (log-odds)
  ops_24h                   = 7              +1.36 (log-odds)
  hora                      = 3              +0.80 (log-odds)
  pais                      = FR             +0.57 (log-odds)

Modelo guardado y recargado · predicciones idénticas: True
```

> Con fraude poco frecuente, la exactitud no sirve como métrica: un modelo que nunca detecta fraude acertaría el 97,5 %. La PR AUC del modelo (0,44) es 17 veces la del azar (0,025). El umbral óptimo bloquea un 13 % de operaciones legítimas porque, con los costes supuestos, revisar una operación (3 €) es mucho más barato que el importe medio defraudado; con otros costes el umbral sería distinto.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ValueError: DataFrame.dtypes for data must be int, float, bool or category` | Columnas de texto (`object`). | Conviértalas a `category` y use `enable_categorical=True`. |
| `TypeError: fit() got an unexpected keyword argument 'early_stopping_rounds'` | En XGBoost ≥ 2.0, `early_stopping_rounds` y `eval_metric` van en el **constructor**. | `XGBClassifier(early_stopping_rounds=50, eval_metric="auc")`. |
| `ValueError: Must have at least 1 validation dataset for early stopping` | Parada temprana sin `eval_set`. | Pase `eval_set=[(X_val, y_val)]` a `fit`. |
| `feature_names mismatch` | Columnas distintas o en otro orden al predecir. | Use exactamente las mismas columnas que en el entrenamiento. |
| Predicciones incorrectas con categorías | Categorías distintas entre entrenamiento y predicción. | Fije las categorías con `pd.CategoricalDtype(categories=[...])`. |
| `ValueError: Invalid classes inferred from unique values of y` | Etiquetas que no son 0..n−1. | Codifique con `LabelEncoder`. |
| Sobreajuste (entrenamiento ≫ validación) | Árboles profundos, sin regularización. | Parada temprana, ↓ `max_depth`, ↑ `min_child_weight`, muestreo. |
| Probabilidades infladas | `scale_pos_weight` alto. | Entrene sin ponderar o calibre las probabilidades. |
| Validación demasiado optimista | Se usó el conjunto de prueba para la parada temprana. | Separe validación y prueba. |
| La CPU queda saturada en la búsqueda | Paralelismo duplicado. | `n_jobs=1` en el modelo o en la búsqueda. |

---

## 18. Buenas prácticas

- Use **parada temprana** con un conjunto de validación independiente del de prueba.
- Empiece con valores razonables (`learning_rate=0.05`, `max_depth=6`, `subsample=0.8`, `colsample_bytree=0.8`) y ajuste después.
- Use **categorías nativas** y deje que XGBoost gestione los valores ausentes.
- En problemas desbalanceados, vigile `aucpr` y ajuste el umbral según el coste real de los errores.
- Aplique **restricciones monótonas** cuando el sentido de una relación sea conocido o exigido por normativa.
- Explique el modelo con **SHAP** y revise que las variables más importantes tengan sentido de negocio.
- Guarde los modelos con `save_model` en JSON/UBJ junto con la lista de variables y las categorías.
- Compare siempre con una referencia sencilla y, si el rendimiento es crítico, con LightGBM y CatBoost.

---

## 19. Referencia rápida

```python
import xgboost as xgb

m = xgb.XGBClassifier(n_estimators=1000, learning_rate=0.05, max_depth=6, subsample=0.8, colsample_bytree=0.8,
                      min_child_weight=1, reg_lambda=1, scale_pos_weight=1, enable_categorical=True,
                      early_stopping_rounds=50, eval_metric="aucpr", device="cpu", n_jobs=-1, random_state=42)
m.fit(X, y, eval_set=[(X_val, y_val)], verbose=False) · m.predict_proba(X)[:, 1] · m.best_iteration
xgb.XGBRegressor(objective="reg:squarederror" | "reg:quantileerror" | "count:poisson" | "reg:tweedie")
monotone_constraints={"col": 1} · interaction_constraints=[["a", "b"]]

d = xgb.DMatrix(X, label=y, enable_categorical=True) · xgb.train(params, d, num_boost_round, evals, early_stopping_rounds)
xgb.cv(params, d, nfold=5, stratified=True, early_stopping_rounds=50)

m.feature_importances_ · m.get_booster().get_score(importance_type="gain") · booster.predict(d, pred_contribs=True)
m.save_model("m.json") · m.load_model("m.json")
```

---

## 20. Ejercicios propuestos

1. Compare `XGBClassifier`, `RandomForestClassifier` y `LogisticRegression` en el ejemplo de abandono de clientes del manual de scikit-learn.
2. Entrene un `XGBRegressor` para el precio de viviendas (`fetch_california_housing`) y obtenga intervalos de predicción del 80 % con regresión por cuantiles.
3. Optimice los hiperparámetros del ejemplo completo con Optuna y compare la PR AUC con la configuración inicial.
4. Elimine la restricción monótona del ejemplo y compruebe con SHAP si la relación entre `ratio_importe` y el riesgo deja de ser coherente en algún tramo.
5. Analice cómo cambia el umbral óptimo y el coste total si el coste de revisión pasa de 3 € a 10 €.

---

## 21. Referencias

- Documentación oficial: https://xgboost.readthedocs.io/
- Parámetros: https://xgboost.readthedocs.io/en/stable/parameter.html
- Variables categóricas: https://xgboost.readthedocs.io/en/stable/tutorials/categorical.html
- Introducción a los árboles potenciados: https://xgboost.readthedocs.io/en/stable/tutorials/model.html
- Chen, T. y Guestrin, C. (2016). *XGBoost: A Scalable Tree Boosting System*. KDD.
