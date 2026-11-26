# Manual de uso: scikit-learn

> **Categoría:** Ciencia de datos · **Utilidad:** *Machine learning* clásico
> **Tipo:** Librería de terceros (`pip install scikit-learn`) · **Versión de referencia:** 1.5 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Flujo de trabajo de un proyecto](#4-flujo-de-trabajo-de-un-proyecto)
5. [La API común: `fit`, `predict`, `transform`](#5-la-api-común-fit-predict-transform)
6. [Datos de ejemplo y división en entrenamiento y prueba](#6-datos-de-ejemplo-y-división-en-entrenamiento-y-prueba)
7. [Preprocesamiento](#7-preprocesamiento)
8. [*Pipelines* y `ColumnTransformer`](#8-pipelines-y-columntransformer)
9. [Modelos de clasificación](#9-modelos-de-clasificación)
10. [Modelos de regresión](#10-modelos-de-regresión)
11. [Métricas de evaluación](#11-métricas-de-evaluación)
12. [Validación cruzada](#12-validación-cruzada)
13. [Ajuste de hiperparámetros](#13-ajuste-de-hiperparámetros)
14. [Clases desequilibradas y umbral de decisión](#14-clases-desequilibradas-y-umbral-de-decisión)
15. [Interpretación de modelos](#15-interpretación-de-modelos)
16. [Aprendizaje no supervisado](#16-aprendizaje-no-supervisado)
17. [Guardar y cargar modelos](#17-guardar-y-cargar-modelos)
18. [Fugas de información (*data leakage*)](#18-fugas-de-información-data-leakage)
19. [Ejemplo práctico completo](#19-ejemplo-práctico-completo)
20. [Errores frecuentes](#20-errores-frecuentes)
21. [Buenas prácticas](#21-buenas-prácticas)
22. [Referencia rápida](#22-referencia-rápida)
23. [Ejercicios propuestos](#23-ejercicios-propuestos)
24. [Referencias](#24-referencias)

---

## 1. Descripción

**scikit-learn** es la librería de referencia de *machine learning* clásico en Python. Ofrece, con una **API uniforme**, algoritmos de clasificación, regresión, *clustering* y reducción de dimensionalidad, junto con herramientas de preprocesamiento, *pipelines*, validación cruzada, búsqueda de hiperparámetros, métricas e interpretación de modelos. Está construida sobre NumPy y SciPy y se integra de forma natural con pandas.

**Ventajas:**
- API consistente: todos los modelos se usan igual (`fit`, `predict`).
- Amplísimo catálogo de algoritmos probados y bien documentados.
- *Pipelines* que encadenan preprocesamiento y modelo, evitando errores y fugas de información.
- Validación cruzada, búsqueda de hiperparámetros y métricas integradas.
- Excelente documentación con guías teóricas y ejemplos.

**Limitaciones:**
- No está pensada para *deep learning* (redes neuronales grandes, imágenes, texto complejo).
- Sin soporte de GPU.
- Trabaja con datos en memoria.
- Algunos algoritmos no escalan a decenas de millones de filas.

| Si necesita… | Opción |
|---|---|
| Modelos clásicos sobre datos tabulares | **scikit-learn** |
| *Gradient boosting* de máximo rendimiento | XGBoost / LightGBM / CatBoost (compatibles con la API de scikit-learn) |
| Redes neuronales, visión, lenguaje | PyTorch / TensorFlow / Hugging Face |
| Estadística inferencial (p-valores, intervalos) | statsmodels |

---

## 2. Instalación e importación

```bash
pip install scikit-learn pandas matplotlib
```

```python
import sklearn
from sklearn.model_selection import train_test_split, cross_validate, GridSearchCV
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report

sklearn.set_config(transform_output="pandas")     # Los transformadores devuelven DataFrames
```

> El paquete se instala como `scikit-learn` y se importa como `sklearn`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `X` | Matriz de características (*features*): una fila por ejemplo, una columna por variable. |
| `y` | Variable objetivo (*target*) que se quiere predecir. |
| Estimador | Cualquier objeto con `fit()`: modelos y transformadores. |
| Transformador | Estimador con `transform()` (escalar, codificar, imputar). |
| Predictor | Estimador con `predict()` (clasificadores, regresores). |
| Clasificación | Predecir una categoría (abandona / no abandona). |
| Regresión | Predecir un número (precio, demanda). |
| *Clustering* | Agrupar ejemplos sin etiquetas. |
| Hiperparámetro | Configuración del modelo fijada antes de entrenar (`max_depth`, `C`). |
| Sobreajuste (*overfitting*) | El modelo memoriza el entrenamiento y generaliza mal. |
| Validación cruzada | Evaluar el modelo en varias particiones para estimar su rendimiento real. |
| *Pipeline* | Cadena de pasos de preprocesamiento + modelo tratada como un único estimador. |

---

## 4. Flujo de trabajo de un proyecto

```
1. Definir el problema y la métrica   →  ¿Qué se predice? ¿Qué error es más caro?
2. Cargar y explorar los datos        →  pandas: tipos, nulos, distribuciones, desequilibrio
3. Separar entrenamiento y prueba     →  train_test_split (prueba reservada hasta el final)
4. Construir un pipeline              →  preprocesamiento + modelo
5. Modelo de referencia (baseline)    →  DummyClassifier / DummyRegressor
6. Comparar modelos con validación cruzada
7. Ajustar hiperparámetros            →  GridSearchCV / RandomizedSearchCV
8. Evaluar una sola vez en prueba     →  métricas, matriz de confusión, curvas
9. Interpretar                        →  importancia por permutación, dependencia parcial
10. Guardar y desplegar               →  joblib / skops + monitorización
```

---

## 5. La API común: `fit`, `predict`, `transform`

```python
modelo = LogisticRegression(max_iter=1000)    # 1. Crear con hiperparámetros
modelo.fit(X_train, y_train)                  # 2. Entrenar
y_pred = modelo.predict(X_test)               # 3. Predecir
probabilidades = modelo.predict_proba(X_test) # Probabilidad de cada clase (clasificadores)
modelo.score(X_test, y_test)                  # Exactitud (clasificación) o R² (regresión)

escalador = StandardScaler()
X_train_esc = escalador.fit_transform(X_train)   # Aprende media y desviación del entrenamiento
X_test_esc = escalador.transform(X_test)         # Aplica lo aprendido (¡sin volver a ajustar!)

modelo.get_params() ; modelo.set_params(C=0.5)
modelo.classes_ ; modelo.coef_ ; modelo.feature_names_in_     # Atributos aprendidos (terminan en _)
```

> Los atributos que terminan en guion bajo (`coef_`, `classes_`) solo existen después de llamar a `fit()`.

---

## 6. Datos de ejemplo y división en entrenamiento y prueba

```python
from sklearn.datasets import load_iris, load_breast_cancer, load_diabetes, make_classification, make_regression

X, y = load_breast_cancer(return_X_y=True, as_frame=True)          # Incluidos (sin Internet)
X, y = make_classification(n_samples=2000, n_features=10, weights=[0.9], random_state=0)   # Sintéticos
# fetch_california_housing(), fetch_openml("titanic", version=1) → se descargan de Internet

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42,
    stratify=y,                 # Mantiene la proporción de clases en ambos conjuntos
)
```

Series temporales: **no** baraje; entrene con el pasado y evalúe con el futuro (`shuffle=False` o `TimeSeriesSplit`).

---

## 7. Preprocesamiento

### 7.1 Variables numéricas

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler, PowerTransformer, KBinsDiscretizer
StandardScaler()          # Media 0, desviación 1 (modelos lineales, SVM, KNN, redes)
MinMaxScaler()            # Rango [0, 1]
RobustScaler()            # Mediana y rango intercuartílico (resistente a atípicos)
PowerTransformer()        # Hace las distribuciones más simétricas
KBinsDiscretizer(n_bins=5, encode="ordinal")
```

> Los árboles y los modelos basados en árboles (Random Forest, *gradient boosting*) **no** necesitan escalado.

### 7.2 Variables categóricas

```python
from sklearn.preprocessing import OneHotEncoder, OrdinalEncoder, TargetEncoder
OneHotEncoder(handle_unknown="ignore", min_frequency=20, sparse_output=False)   # Una columna por categoría
OrdinalEncoder(categories=[["bajo", "medio", "alto"]])                          # Categorías con orden
TargetEncoder()                      # Sustituye cada categoría por la media del objetivo (alta cardinalidad)
```

### 7.3 Valores ausentes

```python
from sklearn.impute import SimpleImputer, KNNImputer
SimpleImputer(strategy="median")                         # mean | median | most_frequent | constant
SimpleImputer(strategy="constant", fill_value="desconocido")
SimpleImputer(strategy="median", add_indicator=True)     # Añade columna «estaba vacío»
KNNImputer(n_neighbors=5)
```

### 7.4 Otras transformaciones

```python
from sklearn.preprocessing import PolynomialFeatures, FunctionTransformer, SplineTransformer
PolynomialFeatures(degree=2, include_bias=False)
FunctionTransformer(np.log1p, feature_names_out="one-to-one")
from sklearn.feature_selection import SelectKBest, mutual_info_classif, RFECV, VarianceThreshold
SelectKBest(mutual_info_classif, k=10)
from sklearn.feature_extraction.text import TfidfVectorizer      # Texto → números
TfidfVectorizer(max_features=5000, ngram_range=(1, 2))
```

---

## 8. *Pipelines* y `ColumnTransformer`

Encadenar preprocesamiento y modelo en un *pipeline* garantiza que las transformaciones se aprenden **solo con los datos de entrenamiento** de cada partición y se aplican igual al predecir.

```python
from sklearn.compose import ColumnTransformer, make_column_selector
from sklearn.pipeline import Pipeline

numericas = ["edad", "antiguedad_meses", "gasto_mensual"]
categoricas = ["plan", "region", "metodo_pago"]

preprocesado = ColumnTransformer([
    ("num", Pipeline([("imputar", SimpleImputer(strategy="median")),
                      ("escalar", StandardScaler())]), numericas),
    ("cat", Pipeline([("imputar", SimpleImputer(strategy="most_frequent")),
                      ("onehot", OneHotEncoder(handle_unknown="ignore"))]), categoricas),
], remainder="drop")                       # "passthrough" para conservar las demás columnas

modelo = Pipeline([("prep", preprocesado), ("clf", LogisticRegression(max_iter=1000))])
modelo.fit(X_train, y_train)
modelo.predict(X_test)

modelo.named_steps["clf"].coef_                      # Acceder a un paso
modelo[:-1].get_feature_names_out()                  # Nombres tras el preprocesado
modelo.set_params(clf__C=0.1)                        # Parámetro de un paso: <paso>__<parámetro>
```

Selección automática por tipo:

```python
ColumnTransformer([
    ("num", StandardScaler(), make_column_selector(dtype_include="number")),
    ("cat", OneHotEncoder(handle_unknown="ignore"), make_column_selector(dtype_include=["object", "string", "category"])),
])
```

Ver el *pipeline* como diagrama (Jupyter): basta con mostrar el objeto; en HTML: `sklearn.utils.estimator_html_repr(modelo)`.

---

## 9. Modelos de clasificación

```python
from sklearn.dummy import DummyClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.neighbors import KNeighborsClassifier
from sklearn.svm import SVC
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, HistGradientBoostingClassifier, ExtraTreesClassifier
from sklearn.naive_bayes import GaussianNB
from sklearn.neural_network import MLPClassifier

DummyClassifier(strategy="most_frequent")                         # Referencia mínima
LogisticRegression(C=1.0, l1_ratio=0, max_iter=1000, class_weight="balanced")
KNeighborsClassifier(n_neighbors=15)
SVC(kernel="rbf", C=1.0, probability=True)
DecisionTreeClassifier(max_depth=5, min_samples_leaf=20)
RandomForestClassifier(n_estimators=300, max_features="sqrt", min_samples_leaf=5, n_jobs=-1, random_state=0)
HistGradientBoostingClassifier(learning_rate=0.1, max_iter=300, early_stopping=True, random_state=0)
```

| Modelo | Ventajas | Inconvenientes |
|---|---|---|
| Regresión logística | Rápida, interpretable, probabilidades calibradas. | Relaciones lineales (salvo transformaciones). |
| KNN | Sencillo, sin entrenamiento. | Lento al predecir; sensible a la escala. |
| SVM | Potente con pocas muestras y muchas variables. | Lento con muchos datos; requiere escalado. |
| Árbol de decisión | Muy interpretable. | Sobreajusta con facilidad. |
| Random Forest | Robusto, poco ajuste necesario. | Modelos grandes; menos interpretables. |
| `HistGradientBoosting` | Gran rendimiento en tabulares, maneja nulos y categorías. | Más hiperparámetros. |

> **Cambio reciente:** en `LogisticRegression`, el parámetro `penalty` está obsoleto desde la versión 1.8. Use `l1_ratio=0` (L2, por defecto), `l1_ratio=1` (L1, con `solver="saga"` o `"liblinear"`), un valor intermedio para *elastic net*, y `C=np.inf` para no regularizar.

`HistGradientBoosting` con categorías nativas:

```python
X["plan"] = X["plan"].astype("category")
HistGradientBoostingClassifier(categorical_features="from_dtype")
```

---

## 10. Modelos de regresión

```python
from sklearn.dummy import DummyRegressor
from sklearn.linear_model import LinearRegression, Ridge, Lasso, ElasticNet, HuberRegressor, PoissonRegressor
from sklearn.ensemble import RandomForestRegressor, HistGradientBoostingRegressor
from sklearn.svm import SVR

Ridge(alpha=1.0) ; Lasso(alpha=0.01) ; ElasticNet(alpha=0.01, l1_ratio=0.5)
HistGradientBoostingRegressor(loss="squared_error")        # "absolute_error", "poisson", "quantile"
HistGradientBoostingRegressor(loss="quantile", quantile=0.9)   # Percentil 90 (intervalos de predicción)

from sklearn.compose import TransformedTargetRegressor
TransformedTargetRegressor(regressor=Ridge(), func=np.log1p, inverse_func=np.expm1)   # Objetivo asimétrico (precios)
```

---

## 11. Métricas de evaluación

### 11.1 Clasificación

```python
from sklearn.metrics import (accuracy_score, precision_score, recall_score, f1_score, roc_auc_score,
                             average_precision_score, confusion_matrix, classification_report, log_loss,
                             balanced_accuracy_score, ConfusionMatrixDisplay, RocCurveDisplay,
                             PrecisionRecallDisplay)

print(classification_report(y_test, y_pred, digits=3))
confusion_matrix(y_test, y_pred)
roc_auc_score(y_test, modelo.predict_proba(X_test)[:, 1])
ConfusionMatrixDisplay.from_estimator(modelo, X_test, y_test, normalize="true", cmap="Blues")
RocCurveDisplay.from_estimator(modelo, X_test, y_test)
PrecisionRecallDisplay.from_estimator(modelo, X_test, y_test)
```

```
                  Predicho: No    Predicho: Sí
Real: No     │  Verdaderos neg. │  Falsos positivos │
Real: Sí     │  Falsos negativos│  Verdaderos pos.  │
```

| Métrica | Pregunta que responde | Úsela cuando… |
|---|---|---|
| Exactitud (*accuracy*) | ¿Qué proporción acierta? | Clases equilibradas. |
| Precisión | De los que predice «Sí», ¿cuántos lo son? | Los falsos positivos son caros. |
| Sensibilidad (*recall*) | De los «Sí» reales, ¿cuántos detecta? | Los falsos negativos son caros. |
| F1 | Equilibrio entre precisión y sensibilidad. | Clases desequilibradas. |
| ROC AUC | ¿Ordena bien positivos frente a negativos? | Comparar modelos independientemente del umbral. |
| *Average precision* (PR AUC) | Calidad en la clase positiva. | Clase positiva muy minoritaria. |
| *Log loss* / Brier | Calidad de las probabilidades. | Se usan las probabilidades directamente. |

### 11.2 Regresión

```python
from sklearn.metrics import mean_absolute_error, root_mean_squared_error, r2_score, mean_absolute_percentage_error
mean_absolute_error(y_test, y_pred)        # MAE: error medio en las unidades del objetivo
root_mean_squared_error(y_test, y_pred)    # RMSE: penaliza más los errores grandes
r2_score(y_test, y_pred)                   # R²: proporción de varianza explicada
mean_absolute_percentage_error(y_test, y_pred)
from sklearn.metrics import PredictionErrorDisplay
PredictionErrorDisplay.from_predictions(y_test, y_pred, kind="actual_vs_predicted")
```

---

## 12. Validación cruzada

```python
from sklearn.model_selection import cross_val_score, cross_validate, StratifiedKFold, KFold, TimeSeriesSplit, GroupKFold

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=0)
puntuaciones = cross_val_score(modelo, X_train, y_train, cv=cv, scoring="roc_auc", n_jobs=-1)
print(f"AUC: {puntuaciones.mean():.3f} ± {puntuaciones.std():.3f}")

resultado = cross_validate(modelo, X_train, y_train, cv=cv, n_jobs=-1,
                           scoring={"auc": "roc_auc", "f1": "f1", "recall": "recall"},
                           return_train_score=True)            # Compare train y test para detectar sobreajuste
```

| Estrategia | Uso |
|---|---|
| `KFold` | Regresión, datos independientes. |
| `StratifiedKFold` | Clasificación (mantiene proporciones). |
| `GroupKFold` / `StratifiedGroupKFold` | Varias filas por cliente/paciente: el mismo grupo no puede estar en entrenamiento y validación. |
| `TimeSeriesSplit` | Series temporales (siempre entrena con el pasado). |
| `RepeatedStratifiedKFold` | Estimaciones más estables con pocos datos. |

Nombres de métricas para `scoring`: `accuracy`, `balanced_accuracy`, `f1`, `precision`, `recall`, `roc_auc`, `average_precision`, `neg_log_loss`, `r2`, `neg_mean_absolute_error`, `neg_root_mean_squared_error`. Métrica propia: `make_scorer(func, greater_is_better=…)`.

---

## 13. Ajuste de hiperparámetros

```python
from sklearn.model_selection import GridSearchCV, RandomizedSearchCV
from scipy.stats import loguniform, randint

rejilla = {
    "clf__C": [0.01, 0.1, 1, 10],
    "prep__num__imputar__strategy": ["mean", "median"],
}
busqueda = GridSearchCV(modelo, rejilla, cv=cv, scoring="roc_auc", n_jobs=-1, refit=True)
busqueda.fit(X_train, y_train)
busqueda.best_params_ ; busqueda.best_score_
mejor = busqueda.best_estimator_
pd.DataFrame(busqueda.cv_results_).sort_values("rank_test_score").head()

aleatoria = RandomizedSearchCV(
    Pipeline([("prep", preprocesado), ("clf", HistGradientBoostingClassifier(random_state=0))]),
    {"clf__learning_rate": loguniform(0.01, 0.3), "clf__max_leaf_nodes": randint(15, 63),
     "clf__min_samples_leaf": randint(10, 100), "clf__l2_regularization": loguniform(1e-3, 10)},
    n_iter=40, cv=cv, scoring="roc_auc", n_jobs=-1, random_state=0)
```

> Comparar varios modelos en la misma búsqueda: use una lista de rejillas con `"clf": [LogisticRegression(), RandomForestClassifier()]`.

Validación cruzada anidada (estimación honesta del rendimiento tras ajustar): `cross_val_score(GridSearchCV(...), X, y, cv=cv_externo)`.

---

## 14. Clases desequilibradas y umbral de decisión

```python
LogisticRegression(class_weight="balanced")
RandomForestClassifier(class_weight="balanced_subsample")
HistGradientBoostingClassifier(class_weight="balanced")
from sklearn.utils.class_weight import compute_sample_weight
modelo.fit(X_train, y_train, clf__sample_weight=compute_sample_weight("balanced", y_train))
```

Por defecto, `predict()` usa un umbral de 0,5 sobre la probabilidad. Ajustarlo según el coste de cada error suele ser más eficaz que remuestrear:

```python
from sklearn.model_selection import TunedThresholdClassifierCV, FixedThresholdClassifier

ajustado = TunedThresholdClassifierCV(modelo, scoring="f1", cv=5)      # Busca el umbral que maximiza F1
ajustado.fit(X_train, y_train)
ajustado.best_threshold_

# Umbral por coste de negocio: un falso negativo cuesta 5 veces más que un falso positivo
from sklearn.metrics import make_scorer, confusion_matrix
def beneficio(y, y_pred):
    tn, fp, fn, tp = confusion_matrix(y, y_pred).ravel()
    return 100 * tp - 20 * fp - 100 * fn
TunedThresholdClassifierCV(modelo, scoring=make_scorer(beneficio), cv=5)

FixedThresholdClassifier(modelo, threshold=0.3)                         # Umbral fijo
```

Calibrar probabilidades: `CalibratedClassifierCV(modelo, method="isotonic", cv=5)`; comprobar con `CalibrationDisplay`.

---

## 15. Interpretación de modelos

```python
from sklearn.inspection import permutation_importance, PartialDependenceDisplay

r = permutation_importance(modelo, X_test, y_test, scoring="roc_auc", n_repeats=10, random_state=0, n_jobs=-1)
importancias = pd.Series(r.importances_mean, index=X_test.columns).sort_values(ascending=False)

PartialDependenceDisplay.from_estimator(modelo, X_test, ["antiguedad_meses", "gasto_mensual"], kind="both")

# Modelos lineales: coeficientes (con variables escaladas)
coefs = pd.Series(modelo[-1].coef_[0], index=modelo[:-1].get_feature_names_out()).sort_values()

# Árboles: reglas legibles
from sklearn.tree import export_text, plot_tree
print(export_text(arbol, feature_names=list(nombres)))
```

> La importancia por permutación se calcula sobre datos de **prueba** y no favorece a las variables con muchos valores distintos, a diferencia de `feature_importances_` de los árboles. Para explicaciones por predicción individual: librería **SHAP**.

---

## 16. Aprendizaje no supervisado

```python
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering, HDBSCAN
from sklearn.decomposition import PCA
from sklearn.manifold import TSNE
from sklearn.metrics import silhouette_score
from sklearn.ensemble import IsolationForest

X_esc = StandardScaler().fit_transform(X)
for k in range(2, 9):                                           # Elegir k
    etiquetas = KMeans(n_clusters=k, n_init="auto", random_state=0).fit_predict(X_esc)
    print(k, silhouette_score(X_esc, etiquetas))

segmentos = make_pipeline(StandardScaler(), KMeans(n_clusters=4, random_state=0)).fit_predict(X)
HDBSCAN(min_cluster_size=30).fit_predict(X_esc)                 # Número de grupos automático y ruido (-1)

pca = PCA(n_components=0.95)                                    # Componentes que explican el 95 % de la varianza
X_red = pca.fit_transform(X_esc); pca.explained_variance_ratio_

anomalias = IsolationForest(contamination=0.01, random_state=0).fit_predict(X)   # -1 = anómalo
```

---

## 17. Guardar y cargar modelos

```python
import joblib
joblib.dump(mejor, "modelo_abandono.joblib")
modelo = joblib.load("modelo_abandono.joblib")
modelo.predict(nuevos_clientes)                       # DataFrame con las mismas columnas originales
```

- Guarde el ***pipeline* completo** (preprocesado + modelo), no solo el modelo.
- Cargue con la **misma versión** de scikit-learn (y de Python) con la que se guardó; anote las versiones junto al archivo.
- **No cargue archivos `joblib`/`pickle` de origen desconocido**: pueden ejecutar código. Alternativa segura: la librería **skops** (`skops.io.dump` / `load` con lista de tipos de confianza).
- Para otros entornos (Java, C#, navegador): exportación a **ONNX** con `skl2onnx`.

---

## 18. Fugas de información (*data leakage*)

Una fuga ocurre cuando el modelo «ve» información que no tendrá al predecir. Produce métricas excelentes en validación y malas en producción.

| Fuga | Ejemplo | Cómo evitarla |
|---|---|---|
| Preprocesar antes de dividir | `StandardScaler().fit(X)` con todos los datos. | Use *pipelines*; ajuste solo con entrenamiento. |
| Variables del futuro | «Fecha de baja» para predecir la baja. | Use solo información disponible en el momento de la predicción. |
| Duplicados o mismo cliente en ambos conjuntos | Varias filas por cliente repartidas. | `GroupKFold`, eliminar duplicados. |
| Seleccionar variables con todos los datos | `SelectKBest` antes de la validación cruzada. | Inclúyalo dentro del *pipeline*. |
| Ajustar viendo el conjunto de prueba | Probar muchas configuraciones contra `X_test`. | Use validación cruzada; prueba solo al final. |
| Barajar series temporales | Entrenar con datos posteriores a los de validación. | `TimeSeriesSplit`. |

---

## 19. Ejemplo práctico completo

**Predicción del abandono de clientes (*churn*)**: genera un conjunto de datos realista con variables numéricas y categóricas, nulos y desequilibrio de clases; define un *pipeline* con `ColumnTransformer`; compara un modelo de referencia, regresión logística, Random Forest y *gradient boosting* con validación cruzada; ajusta hiperparámetros; elige el umbral de decisión que maximiza el beneficio de una campaña de retención; evalúa una sola vez en prueba; interpreta con importancia por permutación; guarda el *pipeline* y puntúa nuevos clientes.

```python
"""
abandono_clientes.py — Modelo de abandono de clientes con scikit-learn.
Requiere: pip install scikit-learn pandas numpy scipy joblib
Uso: python abandono_clientes.py
"""
import json
import time
from pathlib import Path

import joblib
import numpy as np
import pandas as pd
import sklearn
from scipy.stats import loguniform, randint
from sklearn.compose import ColumnTransformer
from sklearn.dummy import DummyClassifier
from sklearn.ensemble import HistGradientBoostingClassifier, RandomForestClassifier
from sklearn.impute import SimpleImputer
from sklearn.inspection import permutation_importance
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import (average_precision_score, classification_report, confusion_matrix, make_scorer,
                             roc_auc_score)
from sklearn.model_selection import (RandomizedSearchCV, StratifiedKFold, TunedThresholdClassifierCV,
                                     cross_validate, train_test_split)
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler

sklearn.set_config(transform_output="pandas")
SEMILLA = 42
NUMERICAS = ["edad", "antiguedad_meses", "cuota_mensual", "consumo_gb", "llamadas_soporte",
             "dias_desde_ultimo_uso", "retrasos_pago"]
CATEGORICAS = ["plan", "contrato", "metodo_pago", "region"]

# Economía de la campaña de retención: ofrecer un descuento a quien el modelo marque como «abandona»
VALOR_CLIENTE = 300        # € que se conservan si se retiene a un cliente que iba a abandonar
COSTE_OFERTA = 40          # € que cuesta la oferta (se paga también si el cliente no iba a irse)
EXITO_OFERTA = 0.35        # Proporción de clientes en riesgo que aceptan la oferta y se quedan


# ------------------------------------------------------------------ Datos
def generar_datos(n: int = 8000) -> pd.DataFrame:
    rng = np.random.default_rng(SEMILLA)
    df = pd.DataFrame({
        "edad": rng.integers(18, 80, n),
        "antiguedad_meses": rng.gamma(2.0, 14, n).round().clip(1, 120),
        "plan": rng.choice(["Básico", "Estándar", "Premium"], n, p=[0.45, 0.4, 0.15]),
        "contrato": rng.choice(["Mensual", "Anual", "Bianual"], n, p=[0.55, 0.3, 0.15]),
        "metodo_pago": rng.choice(["Tarjeta", "Domiciliación", "Transferencia"], n, p=[0.4, 0.45, 0.15]),
        "region": rng.choice(["Norte", "Sur", "Este", "Oeste", "Centro"], n),
        "consumo_gb": rng.lognormal(2.5, 0.8, n).round(1),
        "llamadas_soporte": rng.poisson(1.2, n),
        "dias_desde_ultimo_uso": rng.exponential(6, n).round(),
        "retrasos_pago": rng.poisson(0.3, n),
    })
    df["cuota_mensual"] = (df["plan"].map({"Básico": 20, "Estándar": 35, "Premium": 60})
                           * rng.normal(1, 0.08, n)).round(2)
    # Probabilidad de abandono (relaciones no lineales e interacciones realistas)
    logit = (-2.4 + 1.3 * (df["contrato"] == "Mensual") - 0.9 * (df["contrato"] == "Bianual")
             - 0.025 * df["antiguedad_meses"] + 0.45 * df["llamadas_soporte"]
             + 0.07 * df["dias_desde_ultimo_uso"] + 0.6 * df["retrasos_pago"]
             + 0.5 * (df["metodo_pago"] == "Transferencia") + 0.012 * (df["cuota_mensual"] - 35)
             - 0.35 * np.log1p(df["consumo_gb"]) + 0.6 * ((df["edad"] < 25) & (df["plan"] == "Premium")))
    df["abandona"] = (rng.random(n) < 1 / (1 + np.exp(-logit))).astype(int)
    for col, p in [("consumo_gb", 0.05), ("metodo_pago", 0.03), ("dias_desde_ultimo_uso", 0.04)]:
        df.loc[rng.random(n) < p, col] = np.nan
    df.insert(0, "cliente_id", [f"C{i:05d}" for i in range(n)])
    return df


# ------------------------------------------------------------------ Pipelines
def preprocesado(escalar: bool) -> ColumnTransformer:
    pasos_num = [("imputar", SimpleImputer(strategy="median", add_indicator=True))]
    if escalar:
        pasos_num.append(("escalar", StandardScaler()))
    return ColumnTransformer([
        ("num", Pipeline(pasos_num), NUMERICAS),
        ("cat", Pipeline([("imputar", SimpleImputer(strategy="most_frequent")),
                          ("onehot", OneHotEncoder(handle_unknown="ignore", sparse_output=False))]), CATEGORICAS),
    ], verbose_feature_names_out=False)


def candidatos() -> dict[str, Pipeline]:
    return {
        "Referencia (clase mayoritaria)": Pipeline([("prep", preprocesado(False)),
                                                    ("clf", DummyClassifier(strategy="prior"))]),
        "Regresión logística": Pipeline([("prep", preprocesado(True)),
                                         ("clf", LogisticRegression(max_iter=2000, class_weight="balanced"))]),
        "Random Forest": Pipeline([("prep", preprocesado(False)),
                                   ("clf", RandomForestClassifier(n_estimators=300, min_samples_leaf=10,
                                                                  class_weight="balanced_subsample",
                                                                  n_jobs=-1, random_state=SEMILLA))]),
        "Gradient boosting": Pipeline([("prep", preprocesado(False)),
                                       ("clf", HistGradientBoostingClassifier(random_state=SEMILLA))]),
    }


def beneficio_campania(y_real, y_pred) -> float:
    """Beneficio de ofrecer la oferta a los marcados como «abandona»."""
    tn, fp, fn, tp = confusion_matrix(y_real, y_pred, labels=[0, 1]).ravel()
    return tp * (EXITO_OFERTA * VALOR_CLIENTE - COSTE_OFERTA) - fp * COSTE_OFERTA


# ------------------------------------------------------------------ Proceso
def main() -> int:
    inicio = time.perf_counter()
    df = generar_datos()
    print(f"Clientes: {len(df):,} · Tasa de abandono: {df['abandona'].mean():.1%}\n".replace(",", "."))

    X, y = df[NUMERICAS + CATEGORICAS], df["abandona"]
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, stratify=y, random_state=SEMILLA)
    cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=SEMILLA)

    # 1. Comparación de modelos con validación cruzada
    print("1. Comparación de modelos (validación cruzada, 5 particiones)")
    filas = []
    for nombre, pipe in candidatos().items():
        r = cross_validate(pipe, X_train, y_train, cv=cv, n_jobs=-1, return_train_score=True,
                           scoring={"auc": "roc_auc", "ap": "average_precision"})
        filas.append({"modelo": nombre, "AUC": r["test_auc"].mean(), "± AUC": r["test_auc"].std(),
                      "AP": r["test_ap"].mean(), "AUC entrenamiento": r["train_auc"].mean()})
    comparacion = pd.DataFrame(filas).set_index("modelo").round(3)
    print(comparacion.to_string(), "\n")

    # 2. Ajuste de hiperparámetros del mejor candidato (según la validación cruzada)
    rejillas = {
        "Regresión logística": {"clf__C": loguniform(1e-3, 1e2), "clf__l1_ratio": [0.0, 0.5, 1.0],
                                "clf__solver": ["saga"]},
        "Random Forest": {"clf__min_samples_leaf": randint(2, 60), "clf__max_features": ["sqrt", 0.5, 0.8],
                          "clf__max_depth": [None, 8, 16]},
        "Gradient boosting": {"clf__learning_rate": loguniform(0.02, 0.3), "clf__max_leaf_nodes": randint(8, 48),
                              "clf__min_samples_leaf": randint(20, 150), "clf__l2_regularization": loguniform(1e-3, 10),
                              "clf__max_iter": randint(100, 400)},
    }
    mejor_nombre = comparacion.drop(index="Referencia (clase mayoritaria)")["AUC"].idxmax()
    print(f"2. Búsqueda aleatoria de hiperparámetros ({mejor_nombre})")
    busqueda = RandomizedSearchCV(candidatos()[mejor_nombre], rejillas[mejor_nombre], n_iter=20, cv=cv,
                                  scoring="roc_auc", n_jobs=-1, random_state=SEMILLA)
    busqueda.fit(X_train, y_train)
    print(f"   Mejor AUC (CV): {busqueda.best_score_:.3f} (antes del ajuste: {comparacion.loc[mejor_nombre, 'AUC']:.3f})")
    print("   Parámetros:", {k.split("__")[1]: (round(float(v), 4) if isinstance(v, float) else v)
                            for k, v in busqueda.best_params_.items()}, "\n")

    # 3. Umbral que maximiza el beneficio de la campaña
    #    (con class_weight="balanced" las probabilidades salen «infladas», por eso el umbral óptimo suele subir)
    print("3. Ajuste del umbral de decisión según el beneficio esperado")
    modelo = TunedThresholdClassifierCV(busqueda.best_estimator_, scoring=make_scorer(beneficio_campania),
                                        cv=cv, n_jobs=-1, store_cv_results=True)
    modelo.fit(X_train, y_train)
    print(f"   Umbral óptimo: {modelo.best_threshold_:.2f} (en lugar de 0,50)\n".replace(".", ",", 1))

    # 4. Evaluación final (única) en el conjunto de prueba
    print("4. Evaluación en el conjunto de prueba")
    probabilidades = modelo.predict_proba(X_test)[:, 1]
    y_pred = modelo.predict(X_test)
    y_pred_05 = (probabilidades >= 0.5).astype(int)
    print(f"   ROC AUC: {roc_auc_score(y_test, probabilidades):.3f} · "
          f"Average precision: {average_precision_score(y_test, probabilidades):.3f}")
    print(classification_report(y_test, y_pred, target_names=["Se queda", "Abandona"], digits=3))
    tn, fp, fn, tp = confusion_matrix(y_test, y_pred).ravel()
    print(f"   Matriz de confusión: VN={tn} FP={fp} FN={fn} VP={tp}")
    b_opt, b_05 = beneficio_campania(y_test, y_pred), beneficio_campania(y_test, y_pred_05)
    b_todos = beneficio_campania(y_test, np.ones_like(y_test))
    miles = lambda v: f"{v:,.0f}".replace(",", ".")
    print(f"   Beneficio de la campaña: umbral óptimo {miles(b_opt)} € · umbral 0,5 {miles(b_05)} € · "
          f"ofrecerla a todos {miles(b_todos)} €\n")

    # 5. Interpretación
    print("5. Importancia de las variables (permutación sobre prueba, caída del AUC)")
    imp = permutation_importance(modelo.estimator_, X_test, y_test, scoring="roc_auc", n_repeats=10,
                                 random_state=SEMILLA, n_jobs=-1)
    importancias = (pd.DataFrame({"importancia": imp.importances_mean, "desviacion": imp.importances_std},
                                 index=X_test.columns).sort_values("importancia", ascending=False))
    for nombre, fila in importancias.head(8).iterrows():
        print(f"   {nombre:<22} {fila['importancia']:.4f} ± {fila['desviacion']:.4f} {'█' * int(fila['importancia'] * 400)}")

    # 6. Guardar y puntuar nuevos clientes
    salida = Path("modelo_abandono")
    salida.mkdir(exist_ok=True)
    joblib.dump(modelo, salida / "pipeline.joblib")
    (salida / "metadatos.json").write_text(json.dumps({
        "sklearn": sklearn.__version__, "umbral": float(modelo.best_threshold_),
        "auc_prueba": float(roc_auc_score(y_test, probabilidades)), "variables": NUMERICAS + CATEGORICAS,
    }, indent=2, ensure_ascii=False), encoding="utf-8")

    cargado = joblib.load(salida / "pipeline.joblib")
    nuevos = generar_datos(5).drop(columns="abandona").sample(5, random_state=1)
    nuevos["prob_abandono"] = cargado.predict_proba(nuevos[NUMERICAS + CATEGORICAS])[:, 1].round(3)
    nuevos["ofrecer_descuento"] = cargado.predict(nuevos[NUMERICAS + CATEGORICAS]).astype(bool)
    print("\n6. Puntuación de nuevos clientes con el modelo guardado")
    print(nuevos[["cliente_id", "contrato", "antiguedad_meses", "llamadas_soporte", "prob_abandono",
                  "ofrecer_descuento"]].to_string(index=False))
    print(f"\n✔ Modelo guardado en {salida}/ · Tiempo total: {time.perf_counter() - inicio:.1f} s")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 20. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ValueError: could not convert string to float` | Columnas de texto sin codificar. | `OneHotEncoder`/`OrdinalEncoder` en un `ColumnTransformer`. |
| `ValueError: Input X contains NaN` | Modelo que no admite nulos. | `SimpleImputer` en el *pipeline* (o `HistGradientBoosting`). |
| `ConvergenceWarning` en regresión logística | No converge en las iteraciones permitidas. | Escale las variables y aumente `max_iter`. |
| `FutureWarning: 'penalty' was deprecated` | API de `LogisticRegression` actualizada (1.8). | Use `l1_ratio` y `C` en lugar de `penalty`. |
| `ValueError: Found unknown categories` | Categoría nueva al predecir. | `OneHotEncoder(handle_unknown="ignore")`. |
| `NotFittedError` | Se usó el estimador sin `fit()`. | Entrene antes de predecir. |
| Métricas excelentes en validación y malas en producción | Fuga de información o distinta distribución. | Revise la sección 18; use *pipelines*. |
| Exactitud del 95 % que no sirve | Clases desequilibradas (95 % de una clase). | Use AUC, F1, *average precision* y compare con `DummyClassifier`. |
| `ValueError: X has N features, but … is expecting M features` | Columnas distintas al predecir. | Use DataFrames con las mismas columnas y el *pipeline* completo. |
| Advertencias al cargar un modelo | Versión de scikit-learn diferente. | Use la misma versión; reentrene si es necesario. |
| `Invalid parameter 'C' for estimator Pipeline` | Falta el prefijo del paso. | `clf__C`. |

---

## 21. Buenas prácticas

- Defina la métrica según el problema de negocio antes de entrenar.
- Separe un conjunto de prueba al principio y no lo use hasta la evaluación final.
- Encapsule todo el preprocesamiento en un `Pipeline` con `ColumnTransformer`.
- Compare siempre con un modelo de referencia (`DummyClassifier`/`DummyRegressor`).
- Use validación cruzada estratificada (o por grupos / temporal según los datos).
- Ajuste el umbral de decisión según el coste de los errores; no asuma 0,5.
- Fije `random_state` para resultados reproducibles.
- Interprete el modelo (permutación, dependencia parcial) y revise que tenga sentido.
- Guarde el *pipeline* completo con las versiones de las librerías y monitorice el rendimiento en producción.

---

## 22. Referencia rápida

```python
train_test_split(X, y, test_size=0.2, stratify=y, random_state=0)
Pipeline([("prep", ColumnTransformer([("num", StandardScaler(), num), ("cat", OneHotEncoder(handle_unknown="ignore"), cat)])),
          ("clf", modelo)]) · make_pipeline(...) · pipe.fit(X, y) · predict · predict_proba · named_steps · set_params(clf__C=1)
SimpleImputer · StandardScaler · MinMaxScaler · OneHotEncoder · OrdinalEncoder · TargetEncoder · PolynomialFeatures
LogisticRegression(C=, l1_ratio=) · RandomForestClassifier · HistGradientBoostingClassifier · Ridge · Lasso · KMeans · PCA
cross_val_score / cross_validate(pipe, X, y, cv=StratifiedKFold(5, shuffle=True), scoring="roc_auc")
GridSearchCV / RandomizedSearchCV(pipe, params, cv=, scoring=).fit() → best_params_, best_estimator_
TunedThresholdClassifierCV(pipe, scoring=) · FixedThresholdClassifier · CalibratedClassifierCV
classification_report · confusion_matrix · roc_auc_score · f1_score · mean_absolute_error · r2_score · *Display.from_estimator
permutation_importance · PartialDependenceDisplay · joblib.dump/load · sklearn.set_config(transform_output="pandas")
```

---

## 23. Ejercicios propuestos

1. Con `load_breast_cancer`, compare regresión logística, SVM y Random Forest con validación cruzada y muestre las matrices de confusión.
2. Prediga el precio de viviendas de un conjunto sintético (`make_regression` o datos propios) con `TransformedTargetRegressor` y compare MAE y RMSE.
3. Segmente clientes con K-Means a partir de variables de gasto y frecuencia, elija el número de grupos con la silueta y describa cada segmento.
4. Modifique el ejemplo completo para usar `GroupKFold` suponiendo varias filas por cliente.
5. Cambie los costes de la campaña en el ejemplo y observe cómo se desplaza el umbral óptimo y el beneficio.

---

## 24. Referencias

- Documentación oficial: https://scikit-learn.org/stable/
- Guía de usuario: https://scikit-learn.org/stable/user_guide.html
- Elegir el estimador adecuado: https://scikit-learn.org/stable/machine_learning_map.html
- Errores comunes y fugas de información: https://scikit-learn.org/stable/common_pitfalls.html
- Persistencia de modelos: https://scikit-learn.org/stable/model_persistence.html
