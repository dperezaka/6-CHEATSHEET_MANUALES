# Manual de uso: scikit-learn

> **Categoría:** Inteligencia artificial · **Utilidad:** Machine Learning tradicional
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** BSD · **Importación:** `sklearn`

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [La API común: `fit`, `predict`, `transform`](#4-la-api-común-fit-predict-transform)
5. [Preparación de los datos](#5-preparación-de-los-datos)
6. [Preprocesado](#6-preprocesado)
7. [Pipelines y `ColumnTransformer`](#7-pipelines-y-columntransformer)
8. [Modelos de clasificación y regresión](#8-modelos-de-clasificación-y-regresión)
9. [Aprendizaje no supervisado](#9-aprendizaje-no-supervisado)
10. [Validación cruzada](#10-validación-cruzada)
11. [Búsqueda de hiperparámetros](#11-búsqueda-de-hiperparámetros)
12. [Métricas de evaluación](#12-métricas-de-evaluación)
13. [Umbral de decisión y clases desbalanceadas](#13-umbral-de-decisión-y-clases-desbalanceadas)
14. [Interpretación de modelos](#14-interpretación-de-modelos)
15. [Guardar y desplegar modelos](#15-guardar-y-desplegar-modelos)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**scikit-learn** es la librería de referencia en Python para **Machine Learning tradicional** sobre datos tabulares: clasificación, regresión, *clustering*, reducción de dimensionalidad, preprocesado, selección de modelos y evaluación. Todos sus componentes comparten una API uniforme, lo que permite cambiar de algoritmo modificando una sola línea.

**Ventajas:**
- API consistente y muy bien diseñada (`fit` / `predict` / `transform`).
- Amplio catálogo de algoritmos probados y documentados.
- Herramientas completas para el flujo de trabajo: *pipelines*, validación cruzada, búsqueda de hiperparámetros, métricas.
- Documentación excelente, con guías y ejemplos.
- Integración con NumPy, pandas, SciPy y Matplotlib.

**Limitaciones:**
- No está pensada para *Deep Learning* ni para GPU.
- Los datos deben caber en memoria.
- Para *gradient boosting* de máximo rendimiento suelen preferirse XGBoost, LightGBM o CatBoost (aunque `HistGradientBoosting` es muy competitivo).
- No procesa directamente texto, imágenes o audio sin extracción previa de características.

| Si necesita… | Opción |
|---|---|
| ML clásico sobre datos tabulares, preprocesado, evaluación | **scikit-learn** |
| Máximo rendimiento en datos tabulares grandes | XGBoost / LightGBM / CatBoost |
| Redes neuronales, imágenes, texto, audio | PyTorch / TensorFlow-Keras |
| Modelos de lenguaje preentrenados | Hugging Face Transformers |
| Modelos estadísticos con inferencia (p-valores, intervalos) | statsmodels |

---

## 2. Instalación e importación

```bash
pip install scikit-learn
pip install scikit-learn pandas matplotlib       # Entorno habitual
```

> El paquete se instala como **`scikit-learn`** pero se importa como **`sklearn`**. El paquete `sklearn` de PyPI está obsoleto y no debe instalarse.

```bash
python -c "import sklearn; sklearn.show_versions()"
```

Importaciones habituales (se importa cada componente desde su submódulo):

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split, cross_validate, GridSearchCV
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier, HistGradientBoostingClassifier
from sklearn.metrics import classification_report, roc_auc_score
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `X` | Matriz de características (filas = muestras, columnas = variables). Array de NumPy o `DataFrame`. |
| `y` | Variable objetivo (etiqueta o valor a predecir). |
| Estimador | Cualquier objeto con método `fit` (modelos y transformadores). |
| Transformador | Estimador con `transform`: escaladores, codificadores, imputadores, PCA. |
| Predictor | Estimador con `predict`: clasificadores y regresores. |
| Hiperparámetro | Parámetro que se fija antes de entrenar (`C`, `max_depth`). Se pasa al constructor. |
| Atributo aprendido | Resultado del entrenamiento; termina en guion bajo (`coef_`, `classes_`, `feature_importances_`). |
| Pipeline | Cadena de transformadores y un estimador final que se comporta como un único modelo. |
| Fuga de datos (*data leakage*) | Uso de información del conjunto de prueba durante el entrenamiento. El error más grave en ML. |
| Validación cruzada | Evaluación repetida sobre distintas particiones de los datos. |

**Tipos de problemas:**

| Problema | `y` | Ejemplos |
|---|---|---|
| Clasificación binaria | 2 clases | Abandono sí/no, fraude, impago |
| Clasificación multiclase | > 2 clases | Tipo de producto, categoría de incidencia |
| Regresión | Número continuo | Precio, demanda, tiempo de entrega |
| *Clustering* | Sin `y` | Segmentación de clientes |
| Reducción de dimensionalidad | Sin `y` | Visualización, compresión de variables |

---

## 4. La API común: `fit`, `predict`, `transform`

```python
modelo = LogisticRegression(C=1.0, max_iter=1000)    # 1. Crear con hiperparámetros
modelo.fit(X_ent, y_ent)                             # 2. Entrenar
pred = modelo.predict(X_test)                        # 3. Predecir clases
prob = modelo.predict_proba(X_test)                  # Probabilidades por clase
modelo.score(X_test, y_test)                         # Exactitud (clasificación) o R² (regresión)
modelo.coef_, modelo.classes_                        # Atributos aprendidos

escalador = StandardScaler()
X_ent_esc = escalador.fit_transform(X_ent)           # Aprende media y desviación y transforma
X_test_esc = escalador.transform(X_test)             # ⚠ Solo transform en prueba
```

| Método | Disponible en | Función |
|---|---|---|
| `fit(X, y)` | Todos | Aprende de los datos |
| `predict(X)` | Predictores | Predicción |
| `predict_proba(X)` | Muchos clasificadores | Probabilidad de cada clase |
| `decision_function(X)` | SVM, modelos lineales | Puntuación sin calibrar |
| `transform(X)` | Transformadores | Transforma los datos |
| `fit_transform(X)` | Transformadores | `fit` + `transform` en un paso |
| `score(X, y)` | Predictores | Métrica por defecto |
| `get_params()` / `set_params()` | Todos | Consultar / cambiar hiperparámetros |

Cambiar de algoritmo solo requiere cambiar la clase:

```python
for modelo in [LogisticRegression(max_iter=1000), RandomForestClassifier(), HistGradientBoostingClassifier()]:
    modelo.fit(X_ent, y_ent)
    print(type(modelo).__name__, modelo.score(X_test, y_test))
```

---

## 5. Preparación de los datos

### 5.1 Conjuntos de datos incluidos

```python
from sklearn.datasets import load_iris, load_diabetes, load_breast_cancer, make_classification

X, y = load_iris(return_X_y=True, as_frame=True)          # DataFrame y Series
X, y = make_classification(n_samples=1000, n_features=20, weights=[0.9], random_state=0)   # Sintéticos
```

### 5.2 División entrenamiento / prueba

```python
X_ent, X_test, y_ent, y_test = train_test_split(
    X, y,
    test_size=0.2,          # 20 % para prueba
    stratify=y,             # Mantener la proporción de clases
    random_state=42,        # Reproducibilidad
)
```

> El conjunto de prueba se reserva **hasta el final**. Toda decisión (preprocesado, modelo, hiperparámetros) se toma con el conjunto de entrenamiento y validación cruzada.

### 5.3 Salida en pandas

```python
import sklearn
sklearn.set_config(transform_output="pandas")     # Los transformadores devuelven DataFrames
# o por transformador: StandardScaler().set_output(transform="pandas")
```

---

## 6. Preprocesado

### 6.1 Escalado de variables numéricas

| Clase | Transformación | Cuándo |
|---|---|---|
| `StandardScaler` | Media 0, desviación 1 | Por defecto en modelos lineales, SVM, KNN, redes |
| `MinMaxScaler` | Rango [0, 1] | Variables acotadas |
| `RobustScaler` | Mediana y rango intercuartílico | Datos con valores atípicos |
| `PowerTransformer`, `QuantileTransformer` | Aproxima una distribución normal | Variables muy asimétricas |

> Los modelos basados en **árboles** (bosques, *boosting*) no necesitan escalado.

### 6.2 Variables categóricas

```python
OneHotEncoder(handle_unknown="ignore", sparse_output=False)    # Una columna por categoría
OneHotEncoder(min_frequency=0.01, handle_unknown="infrequent_if_exist")   # Agrupa categorías raras
OrdinalEncoder(categories=[["bajo", "medio", "alto"]])          # Categorías con orden natural
TargetEncoder(cv=KFold(5, shuffle=True, random_state=0))       # Alta cardinalidad (códigos postales…)
```

### 6.3 Valores ausentes

```python
SimpleImputer(strategy="median")              # mean, median, most_frequent, constant
SimpleImputer(strategy="constant", fill_value="desconocido")
KNNImputer(n_neighbors=5)                     # A partir de muestras similares
SimpleImputer(strategy="median", add_indicator=True)   # Añade columna «era ausente»
```

> `HistGradientBoosting*` acepta valores ausentes de forma nativa, sin imputar.

### 6.4 Otras transformaciones

```python
PolynomialFeatures(degree=2, include_bias=False)        # Interacciones y potencias
FunctionTransformer(np.log1p, feature_names_out="one-to-one")   # Función propia
KBinsDiscretizer(n_bins=5, encode="ordinal")            # Discretizar en intervalos
SplineTransformer(n_knots=5)                            # Relaciones no lineales suaves
```

---

## 7. Pipelines y `ColumnTransformer`

### 7.1 Por qué usar *pipelines*

Un *pipeline* encadena preprocesado y modelo en un único objeto:

- **Evita la fuga de datos**: en validación cruzada, el preprocesado se ajusta solo con los datos de entrenamiento de cada partición.
- Permite buscar hiperparámetros del preprocesado y del modelo a la vez.
- Se guarda y despliega como un solo objeto: el modelo recibe datos «en bruto».

```python
modelo = make_pipeline(StandardScaler(), LogisticRegression(max_iter=1000))
modelo.fit(X_ent, y_ent)
modelo.predict(X_test)
```

### 7.2 Tratamiento distinto por columna

```python
from sklearn.compose import ColumnTransformer

numericas = ["edad", "ingresos", "antiguedad"]
categoricas = ["provincia", "canal"]

preprocesado = ColumnTransformer([
    ("num", Pipeline([("imputar", SimpleImputer(strategy="median")),
                      ("escalar", StandardScaler())]), numericas),
    ("cat", Pipeline([("imputar", SimpleImputer(strategy="most_frequent")),
                      ("codificar", OneHotEncoder(handle_unknown="ignore"))]), categoricas),
], remainder="drop")                         # "passthrough" para conservar el resto

modelo = Pipeline([("prep", preprocesado), ("clasificador", RandomForestClassifier())])
```

Selección automática de columnas por tipo:

```python
from sklearn.compose import make_column_transformer, make_column_selector

make_column_transformer(
    (StandardScaler(), make_column_selector(dtype_include="number")),
    (OneHotEncoder(handle_unknown="ignore"), make_column_selector(dtype_include=object)),
)
```

### 7.3 Acceder a los pasos

```python
modelo.named_steps["clasificador"].feature_importances_
modelo["prep"].get_feature_names_out()       # Nombres de las columnas generadas
modelo.set_params(clasificador__n_estimators=500)   # Notación paso__parámetro
```

---

## 8. Modelos de clasificación y regresión

### 8.1 Catálogo principal

| Familia | Clasificación | Regresión | Notas |
|---|---|---|---|
| Lineales | `LogisticRegression` | `LinearRegression`, `Ridge`, `Lasso`, `ElasticNet` | Rápidos, interpretables; requieren escalado |
| Vecinos | `KNeighborsClassifier` | `KNeighborsRegressor` | Sencillos; lentos con muchos datos |
| SVM | `SVC`, `LinearSVC` | `SVR` | Buenos con pocos datos; escalan mal |
| Árbol | `DecisionTreeClassifier` | `DecisionTreeRegressor` | Interpretables; tienden a sobreajustar |
| Bosque aleatorio | `RandomForestClassifier` | `RandomForestRegressor` | Robustos, pocos ajustes |
| *Boosting* | `HistGradientBoostingClassifier` | `HistGradientBoostingRegressor` | Alto rendimiento, rápido, admite ausentes y categóricas |
| Bayesianos | `GaussianNB`, `MultinomialNB` | — | Muy rápidos; texto |
| Redes neuronales | `MLPClassifier` | `MLPRegressor` | Básicas (para redes reales: PyTorch/Keras) |
| Ensamblados | `VotingClassifier`, `StackingClassifier` | `VotingRegressor`, `StackingRegressor` | Combinan modelos |

### 8.2 Hiperparámetros más relevantes

| Modelo | Hiperparámetros |
|---|---|
| `LogisticRegression` | `C` (inversa de la regularización), `class_weight`, `max_iter` |
| `RandomForest*` | `n_estimators`, `max_depth`, `min_samples_leaf`, `max_features`, `n_jobs` |
| `HistGradientBoosting*` | `learning_rate`, `max_iter`, `max_leaf_nodes`, `l2_regularization`, `early_stopping`, `categorical_features` |
| `SVC` | `C`, `kernel`, `gamma` |
| `KNeighbors*` | `n_neighbors`, `weights` |

### 8.3 Árbol de decisión legible

```python
from sklearn.tree import DecisionTreeClassifier, export_text

arbol = DecisionTreeClassifier(max_depth=2).fit(X_ent, y_ent)
print(export_text(arbol, feature_names=list(X.columns)))
```

```
|--- petal width (cm) <= 0.80
|   |--- class: 0
|--- petal width (cm) >  0.80
|   |--- petal width (cm) <= 1.75
|   |   |--- class: 1
|   |--- petal width (cm) >  1.75
|   |   |--- class: 2
```

---

## 9. Aprendizaje no supervisado

### 9.1 *Clustering*

```python
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.metrics import silhouette_score

X_esc = StandardScaler().fit_transform(X)
kmeans = KMeans(n_clusters=4, n_init="auto", random_state=0).fit(X_esc)
kmeans.labels_                 # Grupo de cada muestra
kmeans.cluster_centers_
silhouette_score(X_esc, kmeans.labels_)        # Calidad: −1 a 1 (mayor es mejor)

DBSCAN(eps=0.5, min_samples=10).fit_predict(X_esc)   # Grupos de forma arbitraria; −1 = ruido
```

Elegir el número de grupos:

```python
for k in range(2, 9):
    etiquetas = KMeans(k, n_init="auto", random_state=0).fit_predict(X_esc)
    print(k, round(silhouette_score(X_esc, etiquetas), 3))
```

### 9.2 Reducción de dimensionalidad

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=0.95)            # Componentes que explican el 95 % de la varianza
X_red = pca.fit_transform(X_esc)
pca.explained_variance_ratio_
```

Para visualización no lineal: `sklearn.manifold.TSNE`.

### 9.3 Detección de anomalías

```python
from sklearn.ensemble import IsolationForest

detector = IsolationForest(contamination=0.01, random_state=0).fit(X)
detector.predict(X)                     # 1 = normal, −1 = anómalo
```

---

## 10. Validación cruzada

### 10.1 Evaluación robusta

```python
from sklearn.model_selection import cross_val_score, cross_validate, StratifiedKFold

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
puntuaciones = cross_val_score(modelo, X_ent, y_ent, cv=cv, scoring="roc_auc")
print(f"{puntuaciones.mean():.3f} ± {puntuaciones.std():.3f}")

resultados = cross_validate(modelo, X_ent, y_ent, cv=cv, scoring=["roc_auc", "f1"],
                            return_train_score=True, n_jobs=-1)
```

### 10.2 Estrategias de partición

| Clase | Uso |
|---|---|
| `KFold` | Regresión, datos independientes |
| `StratifiedKFold` | Clasificación (mantiene la proporción de clases) |
| `GroupKFold` | Varias filas por entidad (cliente, paciente): las filas de un grupo no se reparten |
| `TimeSeriesSplit` | Series temporales: entrena con el pasado y valida con el futuro |
| `RepeatedStratifiedKFold` | Estimaciones más estables con pocos datos |

> En datos temporales, **nunca** mezcle aleatoriamente: use `TimeSeriesSplit` o una partición por fecha.

### 10.3 Predicciones fuera de muestra

```python
from sklearn.model_selection import cross_val_predict
prob_oof = cross_val_predict(modelo, X_ent, y_ent, cv=cv, method="predict_proba")[:, 1]
```

---

## 11. Búsqueda de hiperparámetros

### 11.1 Búsqueda exhaustiva (`GridSearchCV`)

```python
from sklearn.model_selection import GridSearchCV

pipe = make_pipeline(StandardScaler(), SVC())
rejilla = {"svc__C": [0.1, 1, 10], "svc__kernel": ["linear", "rbf"]}
busqueda = GridSearchCV(pipe, rejilla, cv=5, scoring="accuracy", n_jobs=-1)
busqueda.fit(X_ent, y_ent)

busqueda.best_params_          # {'svc__C': 1, 'svc__kernel': 'rbf'}
busqueda.best_score_
busqueda.best_estimator_       # Pipeline reentrenado con todos los datos de entrenamiento
pd.DataFrame(busqueda.cv_results_)
```

### 11.2 Búsqueda aleatoria y sucesiva

```python
from scipy.stats import loguniform, randint
from sklearn.model_selection import RandomizedSearchCV

distribuciones = {"clasificador__learning_rate": loguniform(1e-3, 0.3),
                  "clasificador__max_leaf_nodes": randint(8, 64)}
RandomizedSearchCV(pipe, distribuciones, n_iter=40, cv=5, scoring="roc_auc", random_state=0)
```

Con muchos candidatos, `HalvingRandomSearchCV` (requiere `from sklearn.experimental import enable_halving_search_cv`) descarta pronto los peores.

> Para búsquedas avanzadas (optimización bayesiana), **Optuna** se integra fácilmente con scikit-learn.

---

## 12. Métricas de evaluación

### 12.1 Clasificación

```python
from sklearn.metrics import (accuracy_score, precision_score, recall_score, f1_score, roc_auc_score,
                             average_precision_score, confusion_matrix, classification_report, log_loss)

print(classification_report(y_test, pred, digits=3))
confusion_matrix(y_test, pred)
roc_auc_score(y_test, prob)
```

| Métrica | Pregunta que responde | Cuándo usarla |
|---|---|---|
| Exactitud (*accuracy*) | ¿Qué proporción acierta? | Clases equilibradas |
| Precisión | De los marcados como positivos, ¿cuántos lo son? | Coste alto de falsos positivos |
| Sensibilidad (*recall*) | De los positivos reales, ¿cuántos detecta? | Coste alto de falsos negativos (fraude, enfermedad) |
| F1 | Media armónica de precisión y sensibilidad | Clases desbalanceadas |
| ROC AUC | ¿Ordena bien a positivos sobre negativos? | Comparar modelos independientemente del umbral |
| PR AUC (`average_precision`) | Como ROC AUC, centrado en la clase positiva | Clases muy desbalanceadas |
| *Log loss* | ¿Son buenas las probabilidades? | Cuando importan las probabilidades |

### 12.2 Regresión

```python
from sklearn.metrics import mean_absolute_error, root_mean_squared_error, r2_score, mean_absolute_percentage_error
```

| Métrica | Interpretación |
|---|---|
| MAE | Error medio absoluto, en las unidades de `y` |
| RMSE | Penaliza más los errores grandes |
| MAPE | Error porcentual medio (problemático si `y` ≈ 0) |
| R² | Proporción de varianza explicada (1 = perfecto, 0 = predecir la media) |

### 12.3 Nombres de métricas para `scoring`

`"accuracy"`, `"f1"`, `"roc_auc"`, `"average_precision"`, `"neg_log_loss"`, `"neg_mean_absolute_error"`, `"neg_root_mean_squared_error"`, `"r2"`. Las métricas de error llevan el prefijo `neg_` porque scikit-learn siempre **maximiza**.

### 12.4 Visualizaciones

```python
from sklearn.metrics import ConfusionMatrixDisplay, RocCurveDisplay, PrecisionRecallDisplay

ConfusionMatrixDisplay.from_estimator(modelo, X_test, y_test)
RocCurveDisplay.from_estimator(modelo, X_test, y_test)
```

---

## 13. Umbral de decisión y clases desbalanceadas

### 13.1 El umbral de 0,5 no siempre es el adecuado

`predict()` asigna la clase positiva cuando la probabilidad ≥ 0,5. Si un falso negativo cuesta más que un falso positivo, conviene un umbral menor.

```python
from sklearn.model_selection import TunedThresholdClassifierCV, FixedThresholdClassifier

ajustado = TunedThresholdClassifierCV(modelo, scoring="f1", cv=5).fit(X_ent, y_ent)
ajustado.best_threshold_

FixedThresholdClassifier(modelo, threshold=0.3)    # Umbral fijo decidido por negocio
```

Con una métrica de negocio propia:

```python
from sklearn.metrics import make_scorer

def beneficio(y_real, y_pred):
    return (ganancia_por_acierto * ((y_pred == 1) & (y_real == 1)).sum() - coste_por_accion * (y_pred == 1).sum())

TunedThresholdClassifierCV(modelo, scoring=make_scorer(beneficio))
```

### 13.2 Clases desbalanceadas

| Técnica | Cómo |
|---|---|
| Pesos de clase | `class_weight="balanced"` en el modelo |
| Métricas adecuadas | F1, PR AUC, *recall* en lugar de exactitud |
| Ajuste del umbral | `TunedThresholdClassifierCV` |
| Remuestreo | Librería `imbalanced-learn` (SMOTE, submuestreo) |

### 13.3 Calibración de probabilidades

```python
from sklearn.calibration import CalibratedClassifierCV
calibrado = CalibratedClassifierCV(modelo, method="isotonic", cv=5).fit(X_ent, y_ent)
```

---

## 14. Interpretación de modelos

```python
# Coeficientes (modelos lineales, sobre variables escaladas)
pd.Series(modelo[-1].coef_[0], index=modelo[:-1].get_feature_names_out()).sort_values()

# Importancia por impureza (árboles) — sesgada hacia variables con muchas categorías
modelo[-1].feature_importances_

# Importancia por permutación — recomendada, independiente del modelo
from sklearn.inspection import permutation_importance
r = permutation_importance(modelo, X_test, y_test, scoring="roc_auc", n_repeats=10, random_state=0)
pd.Series(r.importances_mean, index=X_test.columns).sort_values(ascending=False)

# Dependencia parcial: efecto de una variable sobre la predicción
from sklearn.inspection import PartialDependenceDisplay
PartialDependenceDisplay.from_estimator(modelo, X_test, ["antiguedad_meses"])
```

> Para explicaciones por predicción individual, use la librería **SHAP**.

---

## 15. Guardar y desplegar modelos

```python
import joblib

joblib.dump(pipeline, "modelo.joblib")             # Guarda preprocesado + modelo
modelo = joblib.load("modelo.joblib")
modelo.predict(nuevos_datos)                        # DataFrame con las mismas columnas originales
```

> ⚠ Cargue el modelo con **la misma versión** de scikit-learn con que se guardó, y **nunca** cargue archivos `.joblib`/`.pkl` de origen no fiable: pueden ejecutar código arbitrario. Para un formato portable y seguro, valore **skops** o **ONNX** (`skl2onnx`).

**Despliegue habitual:** API REST (FastAPI) que carga el *pipeline* al iniciar y llama a `predict_proba` con los datos recibidos.

---

## 16. Ejemplo práctico completo

**Predicción del abandono de clientes (*churn*)** de una operadora de telecomunicaciones: datos sintéticos con variables numéricas, categóricas y ordinales y valores ausentes; preprocesado en un `Pipeline` con `ColumnTransformer`; comparación de tres modelos con validación cruzada estratificada; búsqueda aleatoria de hiperparámetros; **ajuste del umbral de decisión para maximizar el beneficio** de una campaña de retención; evaluación final; importancia por permutación; y guardado del modelo completo para puntuar clientes nuevos.

Requisitos: `pip install scikit-learn pandas joblib` (scikit-learn 1.5 o superior por `TunedThresholdClassifierCV`)

```python
"""
Predicción del abandono de clientes (churn) con scikit-learn.

Flujo completo: datos con variables numéricas y categóricas y valores ausentes,
preprocesado en un Pipeline, comparación de modelos con validación cruzada,
búsqueda de hiperparámetros, ajuste del umbral de decisión según el coste de
negocio, evaluación final, interpretación y guardado del modelo.
"""
import joblib
import numpy as np
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.ensemble import HistGradientBoostingClassifier, RandomForestClassifier
from sklearn.impute import SimpleImputer
from sklearn.inspection import permutation_importance
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, confusion_matrix, make_scorer, roc_auc_score
from sklearn.model_selection import (RandomizedSearchCV, StratifiedKFold, TunedThresholdClassifierCV,
                                     cross_validate, train_test_split)
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, OrdinalEncoder, StandardScaler

SEMILLA = 42
COSTE_CAMPANA = 30          # € por cliente al que se ofrece una retención
VALOR_CLIENTE = 300         # € que se pierden si un cliente abandona
EFICACIA_CAMPANA = 0.40     # Proporción de clientes en riesgo que la campaña retiene


# ------------------------------------------------------------------ Datos
def generar_clientes(n: int = 8_000, semilla: int = SEMILLA) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    contrato = rng.choice(["mensual", "anual", "bianual"], n, p=[0.55, 0.30, 0.15])
    antiguedad = np.where(contrato == "mensual", rng.gamma(2.0, 8, n), rng.gamma(4.0, 10, n)).clip(1, 72).round()
    cuota = rng.normal(55, 18, n).clip(15, 120).round(2)
    tarifa = rng.choice(["Básica", "Fibra", "Fibra+TV", "Total"], n, p=[0.3, 0.35, 0.2, 0.15])
    incidencias = rng.poisson(np.where(tarifa == "Fibra+TV", 1.6, 0.9))
    pago = rng.choice(["domiciliación", "tarjeta", "transferencia"], n, p=[0.6, 0.3, 0.1])
    edad = rng.normal(45, 14, n).clip(18, 90).round()
    logit = (-1.2 + 1.6 * (contrato == "mensual") - 0.8 * (contrato == "bianual")
             - 0.035 * antiguedad + 0.025 * (cuota - 55) + 0.45 * incidencias
             + 0.5 * (pago == "transferencia") - 0.01 * (edad - 45) + rng.normal(0, 0.5, n))
    df = pd.DataFrame({
        "contrato": contrato, "antiguedad_meses": antiguedad, "cuota_mensual": cuota,
        "region": rng.choice(["Norte", "Sur", "Este", "Oeste", "Centro"], n), "tarifa": tarifa,
        "incidencias": incidencias, "forma_pago": pago, "edad": edad,
        "abandona": (rng.random(n) < 1 / (1 + np.exp(-logit))).astype(int),
    })
    df.loc[rng.random(n) < 0.04, "edad"] = np.nan              # Valores ausentes
    df.loc[rng.random(n) < 0.02, "forma_pago"] = np.nan
    return df


NUMERICAS = ["antiguedad_meses", "cuota_mensual", "incidencias", "edad"]
CATEGORICAS = ["region", "tarifa", "forma_pago"]
ORDINALES = ["contrato"]


# ------------------------------------------------------------------ Preprocesado
def crear_preprocesador() -> ColumnTransformer:
    numericas = Pipeline([("imputar", SimpleImputer(strategy="median")), ("escalar", StandardScaler())])
    categoricas = Pipeline([("imputar", SimpleImputer(strategy="most_frequent")),
                            ("codificar", OneHotEncoder(handle_unknown="ignore", sparse_output=False))])
    ordinales = OrdinalEncoder(categories=[["mensual", "anual", "bianual"]])
    return ColumnTransformer([("num", numericas, NUMERICAS), ("cat", categoricas, CATEGORICAS),
                              ("ord", ordinales, ORDINALES)], verbose_feature_names_out=False)


def beneficio(y_real, y_pred) -> float:
    """Beneficio neto de la campaña de retención por cliente analizado."""
    vp = np.sum((y_pred == 1) & (y_real == 1))     # Clientes en riesgo contactados
    contactados = np.sum(y_pred == 1)
    return (vp * VALOR_CLIENTE * EFICACIA_CAMPANA - contactados * COSTE_CAMPANA) / len(y_real)


def main() -> None:
    df = generar_clientes()
    X, y = df.drop(columns="abandona"), df["abandona"]
    X_ent, X_test, y_ent, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=SEMILLA)
    print(f"Clientes: {len(df):,} · tasa de abandono {y.mean():.1%} · "
          f"entrenamiento {len(X_ent):,} / prueba {len(X_test):,}\n")

    # 1. Comparar modelos con validación cruzada
    cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=SEMILLA)
    modelos = {
        "Regresión logística": LogisticRegression(max_iter=1000),
        "Bosque aleatorio": RandomForestClassifier(n_estimators=300, min_samples_leaf=5,
                                                   n_jobs=-1, random_state=SEMILLA),
        "Gradient Boosting": HistGradientBoostingClassifier(random_state=SEMILLA),
    }
    print(f"{'Modelo':<22}{'ROC AUC':>16}{'F1':>16}")
    for nombre, modelo in modelos.items():
        pipe = Pipeline([("prep", crear_preprocesador()), ("modelo", modelo)])
        r = cross_validate(pipe, X_ent, y_ent, cv=cv, scoring=["roc_auc", "f1"], n_jobs=-1)
        print(f"{nombre:<22}{r['test_roc_auc'].mean():>10.3f} ±{r['test_roc_auc'].std():.3f}"
              f"{r['test_f1'].mean():>10.3f} ±{r['test_f1'].std():.3f}")

    # 2. Búsqueda de hiperparámetros sobre el mejor candidato
    pipe = Pipeline([("prep", crear_preprocesador()), ("modelo", LogisticRegression(max_iter=1000))])
    busqueda = RandomizedSearchCV(
        pipe,
        {"modelo__C": np.logspace(-3, 2, 30), "modelo__class_weight": [None, "balanced"]},
        n_iter=20, scoring="roc_auc", cv=cv, n_jobs=-1, random_state=SEMILLA)
    busqueda.fit(X_ent, y_ent)
    mejores = busqueda.best_params_
    print(f"\nMejores hiperparámetros: C = {mejores['modelo__C']:.3g}, "
          f"class_weight = {mejores['modelo__class_weight']} · AUC CV {busqueda.best_score_:.3f}")

    # 3. Ajustar el umbral de decisión para maximizar el beneficio de negocio
    ajustado = TunedThresholdClassifierCV(busqueda.best_estimator_, scoring=make_scorer(beneficio),
                                          cv=cv, n_jobs=-1)
    ajustado.fit(X_ent, y_ent)
    print(f"Umbral óptimo de decisión: {ajustado.best_threshold_:.2f} (en lugar de 0.50)")

    # 4. Evaluación final en el conjunto de prueba
    prob = ajustado.predict_proba(X_test)[:, 1]
    pred = ajustado.predict(X_test)
    pred_05 = (prob >= 0.5).astype(int)
    print(f"\nROC AUC en prueba: {roc_auc_score(y_test, prob):.3f}")
    print(classification_report(y_test, pred, target_names=["Permanece", "Abandona"], digits=3))
    print("Matriz de confusión (filas = real, columnas = predicción):")
    print(pd.DataFrame(confusion_matrix(y_test, pred), index=["Permanece", "Abandona"],
                       columns=["Permanece", "Abandona"]))
    print(f"\nBeneficio por cliente · umbral 0,50: {beneficio(y_test.values, pred_05):.2f} € · "
          f"umbral ajustado: {beneficio(y_test.values, pred):.2f} €")

    # 5. Interpretación: importancia por permutación sobre las columnas originales
    imp = permutation_importance(ajustado, X_test, y_test, scoring="roc_auc",
                                 n_repeats=10, random_state=SEMILLA, n_jobs=-1)
    importancias = pd.Series(imp.importances_mean, index=X_test.columns).sort_values(ascending=False)
    print("\nImportancia de las variables (caída de AUC al permutarlas):")
    for variable, valor in importancias.items():
        print(f"  {variable:<18}{valor:>7.3f}  {'█' * int(valor * 200)}")

    # 6. Guardar y reutilizar el modelo completo (preprocesado + modelo + umbral)
    joblib.dump(ajustado, "modelo_abandono.joblib")
    cargado = joblib.load("modelo_abandono.joblib")
    nuevo = pd.DataFrame([{"contrato": "mensual", "antiguedad_meses": 3, "cuota_mensual": 85.0,
                           "region": "Sur", "tarifa": "Fibra+TV", "incidencias": 4,
                           "forma_pago": "transferencia", "edad": np.nan}])
    print(f"\nCliente nuevo · probabilidad de abandono {cargado.predict_proba(nuevo)[0, 1]:.1%} · "
          f"¿contactar? {'Sí' if cargado.predict(nuevo)[0] else 'No'}")


if __name__ == "__main__":
    main()
```

Resultado:

```
Clientes: 8,000 · tasa de abandono 37.5% · entrenamiento 6,400 / prueba 1,600

Modelo                         ROC AUC              F1
Regresión logística        0.819 ±0.010     0.658 ±0.011
Bosque aleatorio           0.809 ±0.011     0.640 ±0.010
Gradient Boosting          0.802 ±0.008     0.643 ±0.014

Mejores hiperparámetros: C = 45.2, class_weight = None · AUC CV 0.819
Umbral óptimo de decisión: 0.23 (en lugar de 0.50)

ROC AUC en prueba: 0.825
              precision    recall  f1-score   support

   Permanece      0.925     0.557     0.695      1000
    Abandona      0.556     0.925     0.695       600

    accuracy                          0.695      1600
   macro avg      0.741     0.741     0.695      1600
weighted avg      0.787     0.695     0.695      1600

Matriz de confusión (filas = real, columnas = predicción):
           Permanece  Abandona
Permanece        557       443
Abandona          45       555

Beneficio por cliente · umbral 0,50: 18.26 € · umbral ajustado: 22.91 €

Importancia de las variables (caída de AUC al permutarlas):
  contrato            0.142  ████████████████████████████
  antiguedad_meses    0.074  ██████████████
  incidencias         0.029  █████
  cuota_mensual       0.027  █████
  edad                0.003  
  forma_pago          0.001  
  region              0.001  
  tarifa              0.000  

Cliente nuevo · probabilidad de abandono 96.6% · ¿contactar? Sí
```

> El modelo sencillo (regresión logística) supera a los de árboles porque los datos se generaron con una relación casi lineal: **compare siempre varios modelos** antes de elegir. Ajustar el umbral a 0,23 detecta el 92,5 % de los clientes que abandonan a cambio de más contactos, y aumenta el beneficio de la campaña un 25 % frente al umbral por defecto.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| Resultados excelentes en validación y malos en producción | Fuga de datos: preprocesado ajustado con todos los datos, variables que contienen el resultado, filas del mismo cliente en entrenamiento y prueba. | Use *pipelines*, revise las variables y use `GroupKFold`/`TimeSeriesSplit` si procede. |
| `ValueError: could not convert string to float` | Variables categóricas sin codificar. | Use `OneHotEncoder`/`OrdinalEncoder` en un `ColumnTransformer`. |
| `ValueError: Input X contains NaN` | Valores ausentes en un modelo que no los admite. | `SimpleImputer` o `HistGradientBoosting*`. |
| `ValueError: Found unknown categories` | Categoría nueva en prueba o producción. | `OneHotEncoder(handle_unknown="ignore")`. |
| `ValueError: X has N features, but ... is expecting M features` | Columnas distintas entre entrenamiento y predicción. | Use el mismo *pipeline* con las columnas originales. |
| `ConvergenceWarning` en `LogisticRegression` | Datos sin escalar o pocas iteraciones. | Escale las variables y aumente `max_iter`. |
| Exactitud del 95 % pero el modelo no detecta nada | Clases desbalanceadas: predice siempre la mayoritaria. | Use F1/PR AUC, `class_weight="balanced"` y ajuste el umbral. |
| Error al usar `best_params_` en otro *pipeline* | Los nombres incluyen el paso (`modelo__C`). | Use `best_estimator_` o la notación `paso__parámetro`. |
| `InconsistentVersionWarning` al cargar un modelo | Versión de scikit-learn distinta. | Use la misma versión o reentrene. |
| Modelo KNN/SVM con resultados pobres | Variables sin escalar. | Añada `StandardScaler` al *pipeline*. |
| Resultados no reproducibles | Falta `random_state`. | Fíjelo en particiones y modelos. |
| `ModuleNotFoundError: No module named 'sklearn'` | No instalado, o se instaló el paquete `sklearn` obsoleto. | `pip install scikit-learn`. |

---

## 18. Buenas prácticas

- **Separe el conjunto de prueba al principio** y úselo una sola vez, al final.
- Incluya **todo el preprocesado en un `Pipeline`**: evita fugas de datos y simplifica el despliegue.
- Empiece con un **modelo de referencia sencillo** (`DummyClassifier`, regresión logística) para saber si los modelos complejos aportan algo.
- Use **validación cruzada** con la estrategia adecuada (estratificada, por grupos o temporal).
- Elija la **métrica según el problema de negocio**, no por costumbre; ajuste el umbral cuando los errores tengan costes distintos.
- Fije `random_state` en particiones y modelos para que los resultados sean reproducibles.
- Revise la importancia de las variables: una variable con importancia desmesurada suele indicar una fuga de datos.
- Guarde el *pipeline* completo con la versión de las librerías (`requirements.txt`) y los metadatos del entrenamiento (fecha, datos, métricas).
- Vigile el modelo en producción: la distribución de los datos cambia con el tiempo (*data drift*).

---

## 19. Referencia rápida

```python
from sklearn.model_selection import train_test_split, StratifiedKFold, cross_validate, GridSearchCV, RandomizedSearchCV
X_ent, X_test, y_ent, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

# Preprocesado
StandardScaler() · MinMaxScaler() · OneHotEncoder(handle_unknown="ignore") · OrdinalEncoder() · TargetEncoder()
SimpleImputer(strategy="median") · ColumnTransformer([(nombre, transf, columnas), ...]) · make_column_selector
Pipeline([("prep", ...), ("modelo", ...)]) · make_pipeline(...) · sklearn.set_config(transform_output="pandas")

# Modelos
LogisticRegression · RandomForestClassifier · HistGradientBoostingClassifier · SVC · KNeighborsClassifier
LinearRegression · Ridge · Lasso · RandomForestRegressor · HistGradientBoostingRegressor
KMeans · DBSCAN · PCA · IsolationForest

# Entrenar, predecir, evaluar
m.fit(X, y) · m.predict(X) · m.predict_proba(X)[:, 1] · m.score(X, y)
cross_validate(m, X, y, cv=cv, scoring=["roc_auc", "f1"]) · GridSearchCV(m, rejilla, cv=5).fit(X, y).best_params_
classification_report · confusion_matrix · roc_auc_score · f1_score · mean_absolute_error · root_mean_squared_error · r2_score
TunedThresholdClassifierCV(m, scoring=...) · permutation_importance(m, X, y) · joblib.dump(m, "m.joblib")
```

---

## 20. Ejercicios propuestos

1. Con `load_breast_cancer`, compare cinco clasificadores mediante validación cruzada y represente sus curvas ROC.
2. Construya un modelo de regresión que prediga el precio de viviendas (`fetch_california_housing`) con un *pipeline* que incluya `SplineTransformer` y compare `Ridge` con `HistGradientBoostingRegressor`.
3. Segmente clientes a partir de un conjunto de datos de compras (frecuencia, importe, antigüedad) con `KMeans`, elija el número de grupos con la silueta y describa cada segmento.
4. Provoque deliberadamente una fuga de datos (escalar antes de dividir, o incluir una variable posterior al evento) y mida cómo cambia la evaluación.
5. Amplíe el ejemplo completo para comparar el beneficio de la campaña con distintos costes de contacto y represente cómo varía el umbral óptimo.

---

## 21. Referencias

- Documentación oficial: https://scikit-learn.org/stable/
- Guía de usuario: https://scikit-learn.org/stable/user_guide.html
- Elegir el estimador adecuado: https://scikit-learn.org/stable/machine_learning_map.html
- Errores comunes y prácticas recomendadas: https://scikit-learn.org/stable/common_pitfalls.html
- Galería de ejemplos: https://scikit-learn.org/stable/auto_examples/index.html
