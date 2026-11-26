# Manual de uso: Keras

> **Categoría:** Inteligencia artificial · **Utilidad:** Construcción sencilla de redes neuronales
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** Apache 2.0 · **Versión de referencia:** Keras 3 (multibackend)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación, importación y elección del backend](#2-instalación-importación-y-elección-del-backend)
3. [Conceptos clave](#3-conceptos-clave)
4. [Tres formas de construir modelos](#4-tres-formas-de-construir-modelos)
5. [Capas principales](#5-capas-principales)
6. [Compilar: optimizador, pérdida y métricas](#6-compilar-optimizador-pérdida-y-métricas)
7. [Entrenar, evaluar y predecir](#7-entrenar-evaluar-y-predecir)
8. [Callbacks](#8-callbacks)
9. [Capas de preprocesado](#9-capas-de-preprocesado)
10. [Datos de entrada: arrays, `tf.data` y `DataLoader`](#10-datos-de-entrada-arrays-tfdata-y-dataloader)
11. [Regularización y prevención del sobreajuste](#11-regularización-y-prevención-del-sobreajuste)
12. [Aprendizaje por transferencia y KerasHub](#12-aprendizaje-por-transferencia-y-kerashub)
13. [Capas, modelos y pasos de entrenamiento personalizados](#13-capas-modelos-y-pasos-de-entrenamiento-personalizados)
14. [Guardar, cargar y exportar](#14-guardar-cargar-y-exportar)
15. [Código independiente del backend: `keras.ops`](#15-código-independiente-del-backend-kerasops)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Keras** es una API de alto nivel para construir y entrenar redes neuronales con muy poco código. Su lema es «diseñada para humanos»: modelos legibles, mensajes de error claros y un flujo de trabajo estándar (`compile` → `fit` → `evaluate` → `predict`).

Desde **Keras 3**, la librería es **multibackend**: el mismo código se ejecuta sobre **TensorFlow, JAX o PyTorch**. En TensorFlow sigue disponible como `tf.keras`.

**Ventajas:**
- Curva de aprendizaje muy suave: una red se define y entrena en pocas líneas.
- API funcional para modelos con varias entradas, salidas o ramas.
- *Callbacks*, métricas y capas de preprocesado listas para usar.
- Libertad para elegir backend sin reescribir el modelo.
- Modelos preentrenados en **KerasHub** (lenguaje, visión, audio).

**Limitaciones:**
- Menos flexible que PyTorch puro para investigación con arquitecturas muy dinámicas (aunque admite personalización completa).
- Algunas capas de preprocesado de texto dependen de TensorFlow internamente.
- El comportamiento y el rendimiento pueden variar ligeramente entre backends.

| Si necesita… | Opción |
|---|---|
| Redes neuronales con poco código y API estándar | **Keras** |
| Control total del entrenamiento e investigación | PyTorch |
| Ecosistema de despliegue (LiteRT, TF.js, TF Serving) | Keras con backend TensorFlow |
| Modelos de lenguaje preentrenados | Hugging Face Transformers / KerasHub |
| Datos tabulares sin redes neuronales | scikit-learn, XGBoost, LightGBM |

---

## 2. Instalación, importación y elección del backend

```bash
pip install keras tensorflow          # Keras + backend TensorFlow (opción por defecto)
pip install keras jax                 # Backend JAX
pip install keras torch               # Backend PyTorch
```

**Elegir el backend** (antes de importar Keras):

```python
import os
os.environ["KERAS_BACKEND"] = "jax"      # "tensorflow" (por defecto), "jax" o "torch"
import keras
print(keras.__version__, keras.config.backend())
```

O de forma permanente en `~/.keras/keras.json` (`"backend": "torch"`), o con la variable de entorno del sistema `KERAS_BACKEND`.

| Backend | Puntos fuertes |
|---|---|
| TensorFlow | Despliegue (SavedModel, LiteRT, TF.js), `tf.data`, capas de texto |
| JAX | Velocidad en GPU/TPU con compilación XLA |
| PyTorch | Integración con el ecosistema PyTorch y Hugging Face |

Importaciones habituales:

```python
import keras
from keras import layers, ops
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Capa (`Layer`) | Bloque que transforma tensores y puede tener pesos (`Dense`, `Conv2D`…). |
| Modelo (`Model`) | Grafo de capas con métodos de entrenamiento e inferencia. |
| `compile` | Configura optimizador, función de pérdida y métricas. |
| `fit` | Entrena el modelo durante un número de épocas. |
| *Callback* | Objeto que actúa durante el entrenamiento (parada temprana, guardado, registro). |
| Época / lote | Pasada completa por los datos / grupo de muestras por paso. |
| Capa de preprocesado | Capa que transforma datos en bruto (normaliza, codifica texto o categorías). `adapt()` la ajusta a los datos. |
| `keras.ops` | Operaciones matemáticas comunes a todos los backends. |
| Formato `.keras` | Archivo único con arquitectura, pesos y estado del optimizador. |

---

## 4. Tres formas de construir modelos

### 4.1 Modelo secuencial

Para pilas lineales de capas (una entrada, una salida):

```python
modelo = keras.Sequential([
    keras.Input(shape=(10,)),
    layers.Dense(32, activation="relu"),
    layers.Dense(3, activation="softmax"),
])
modelo.summary()
```

### 4.2 API funcional (recomendada)

Para varias entradas o salidas, ramas y conexiones residuales:

```python
entrada_num = keras.Input(shape=(8,), name="numericas")
entrada_img = keras.Input(shape=(64, 64, 3), name="imagen")

x1 = layers.Dense(32, activation="relu")(entrada_num)
x2 = layers.Conv2D(16, 3, activation="relu")(entrada_img)
x2 = layers.GlobalAveragePooling2D()(x2)

x = layers.Concatenate()([x1, x2])
precio = layers.Dense(1, name="precio")(x)
categoria = layers.Dense(5, activation="softmax", name="categoria")(x)

modelo = keras.Model(inputs=[entrada_num, entrada_img], outputs=[precio, categoria])
keras.utils.plot_model(modelo, "modelo.png", show_shapes=True)     # Requiere pydot y graphviz
```

### 4.3 Subclases de `Model`

Para lógica dinámica en la propagación:

```python
class Red(keras.Model):
    def __init__(self):
        super().__init__()
        self.oculta = layers.Dense(64, activation="relu")
        self.salida = layers.Dense(1)

    def call(self, x, training=False):
        return self.salida(self.oculta(x))
```

| Forma | Cuándo usarla |
|---|---|
| `Sequential` | Prototipos y redes simples |
| Funcional | La mayoría de modelos reales: multientrada, multisalida, residuales |
| Subclases | Arquitecturas con lógica Python dentro de `call` |

---

## 5. Capas principales

| Tipo | Capas |
|---|---|
| Densas | `Dense`, `Activation`, `Dropout` |
| Convolucionales | `Conv1D`, `Conv2D`, `Conv3D`, `SeparableConv2D`, `Conv2DTranspose` |
| *Pooling* | `MaxPooling2D`, `AveragePooling2D`, `GlobalAveragePooling2D` |
| Recurrentes | `LSTM`, `GRU`, `SimpleRNN`, `Bidirectional` |
| Atención | `MultiHeadAttention`, `Attention` |
| Normalización | `BatchNormalization`, `LayerNormalization` |
| Embeddings | `Embedding` |
| Forma | `Flatten`, `Reshape`, `Permute`, `RepeatVector` |
| Combinación | `Concatenate`, `Add`, `Multiply`, `Average` |
| Aumento de datos | `RandomFlip`, `RandomRotation`, `RandomZoom`, `RandomContrast` |

Activaciones habituales: `"relu"`, `"gelu"`, `"sigmoid"`, `"softmax"`, `"tanh"`, `"silu"`.

---

## 6. Compilar: optimizador, pérdida y métricas

```python
modelo.compile(
    optimizer=keras.optimizers.Adam(learning_rate=1e-3),
    loss="sparse_categorical_crossentropy",
    metrics=["accuracy"],
)
```

| Problema | Última capa | Pérdida |
|---|---|---|
| Binario | `Dense(1, activation="sigmoid")` | `"binary_crossentropy"` |
| Multiclase, etiquetas enteras | `Dense(n, activation="softmax")` | `"sparse_categorical_crossentropy"` |
| Multiclase, etiquetas *one-hot* | `Dense(n, activation="softmax")` | `"categorical_crossentropy"` |
| Multietiqueta | `Dense(n, activation="sigmoid")` | `"binary_crossentropy"` |
| Regresión | `Dense(1)` | `"mse"`, `"mae"`, `"huber"` |

Optimizadores: `Adam`, `AdamW`, `SGD(momentum=0.9)`, `RMSprop`, `Lion`. Programación de la tasa:

```python
programa = keras.optimizers.schedules.CosineDecay(initial_learning_rate=1e-3, decay_steps=10_000)
keras.optimizers.AdamW(learning_rate=programa, weight_decay=1e-4)
```

Métricas: `"accuracy"`, `keras.metrics.AUC()`, `Precision()`, `Recall()`, `MeanAbsoluteError()`, `RootMeanSquaredError()`.

Modelos con varias salidas: pérdidas y pesos por salida.

```python
modelo.compile(optimizer="adam",
               loss={"precio": "mse", "categoria": "sparse_categorical_crossentropy"},
               loss_weights={"precio": 1.0, "categoria": 0.5})
```

---

## 7. Entrenar, evaluar y predecir

```python
historial = modelo.fit(
    X_ent, y_ent,
    validation_split=0.2,              # o validation_data=(X_val, y_val)
    epochs=50,
    batch_size=64,
    class_weight={0: 1.0, 1: 3.0},     # Clases desbalanceadas
    callbacks=[...],
    verbose=1,                         # 0 silencio, 1 barra, 2 una línea por época
)

historial.history["val_loss"]          # Evolución de las métricas
modelo.evaluate(X_test, y_test)        # [pérdida, métricas...]
modelo.predict(X_nuevos, batch_size=256)
```

Gráfico de la curva de aprendizaje:

```python
import pandas as pd
pd.DataFrame(historial.history)[["loss", "val_loss"]].plot()
```

> Si la pérdida de validación sube mientras la de entrenamiento baja, el modelo está **sobreajustando** (sección 11).

---

## 8. Callbacks

```python
callbacks = [
    keras.callbacks.EarlyStopping(monitor="val_loss", patience=10, restore_best_weights=True),
    keras.callbacks.ReduceLROnPlateau(monitor="val_loss", factor=0.5, patience=4),
    keras.callbacks.ModelCheckpoint("mejor.keras", monitor="val_loss", save_best_only=True),
    keras.callbacks.CSVLogger("entrenamiento.csv"),
    keras.callbacks.TensorBoard(log_dir="logs"),
]
modelo.fit(..., callbacks=callbacks)
```

*Callback* propio:

```python
class Aviso(keras.callbacks.Callback):
    def on_epoch_end(self, epoca, logs=None):
        if logs["val_loss"] < 0.01:
            print(f"Objetivo alcanzado en la época {epoca}")
            self.model.stop_training = True
```

---

## 9. Capas de preprocesado

Integran la transformación de los datos en el propio flujo del modelo. Se ajustan con `adapt()` sobre los datos de **entrenamiento**.

| Capa | Función |
|---|---|
| `Normalization` | Estandariza variables numéricas (media 0, varianza 1) |
| `Discretization` | Agrupa valores numéricos en intervalos |
| `StringLookup` / `IntegerLookup` | Categorías → índices (o *one-hot*) |
| `CategoryEncoding` | Índices → *one-hot* / *multi-hot* |
| `Hashing` | Categorías de alta cardinalidad sin vocabulario |
| `TextVectorization` | Texto → secuencias de índices, bolsa de palabras o TF-IDF |
| `Rescaling`, `Resizing`, `CenterCrop` | Imágenes |
| `RandomFlip`, `RandomRotation`, `RandomZoom` | Aumento de datos (solo activos en entrenamiento) |

```python
normalizacion = layers.Normalization()
normalizacion.adapt(X_ent)                         # Calcula media y varianza

ciudades = layers.StringLookup(output_mode="one_hot")
ciudades.adapt(df_ent["ciudad"].to_numpy(str))

texto = layers.TextVectorization(max_tokens=20_000, output_sequence_length=200)
texto.adapt(textos_entrenamiento)
```

> `StringLookup` y `TextVectorization` se ejecutan internamente con TensorFlow. Con los backends JAX o PyTorch, aplíquelas en una canalización `tf.data` (fuera del modelo) y entregue al modelo tensores numéricos, como en el ejemplo completo.

Para datos tabulares, `keras.utils.FeatureSpace` configura el preprocesado de todas las columnas de una vez.

---

## 10. Datos de entrada: arrays, `tf.data` y `DataLoader`

`fit`, `evaluate` y `predict` aceptan:

| Tipo de entrada | Backend |
|---|---|
| Arrays de NumPy (numéricos) | Todos |
| Diccionarios de arrays (modelos multientrada, por nombre de `Input`) | Todos |
| `tf.data.Dataset` | Todos |
| `torch.utils.data.DataLoader` | Todos |
| `keras.utils.PyDataset` (generador propio) | Todos |

```python
modelo.fit({"numericas": X_num, "imagen": X_img}, {"precio": y_p, "categoria": y_c}, epochs=10)
```

Utilidades de carga:

```python
keras.utils.image_dataset_from_directory("imagenes/", image_size=(224, 224), batch_size=32)
keras.utils.text_dataset_from_directory("textos/")
keras.utils.timeseries_dataset_from_array(serie, objetivos, sequence_length=48)
```

> Los arrays de NumPy con texto (`dtype=object` o `str`) no se aceptan directamente en `fit` (`ValueError: Invalid dtype: object`). Use un `tf.data.Dataset`.

---

## 11. Regularización y prevención del sobreajuste

| Técnica | Cómo |
|---|---|
| Parada temprana | `EarlyStopping(restore_best_weights=True)` |
| *Dropout* | `layers.Dropout(0.3)` |
| Penalización L2 | `layers.Dense(64, kernel_regularizer=keras.regularizers.L2(1e-4))` |
| Decaimiento de pesos | `keras.optimizers.AdamW(weight_decay=1e-4)` |
| Aumento de datos | Capas `Random*` |
| Normalización por lotes | `layers.BatchNormalization()` |
| Más datos / modelo más pequeño | Reducir capas y unidades |

---

## 12. Aprendizaje por transferencia y KerasHub

### 12.1 Modelos de visión preentrenados

```python
base = keras.applications.EfficientNetB0(include_top=False, weights="imagenet",
                                         input_shape=(224, 224, 3), pooling="avg")
base.trainable = False                                   # Congelar

entrada = keras.Input((224, 224, 3))
x = layers.RandomFlip("horizontal")(entrada)
x = base(x, training=False)                              # Mantener BatchNorm en modo inferencia
salida = layers.Dense(5, activation="softmax")(layers.Dropout(0.2)(x))
modelo = keras.Model(entrada, salida)
modelo.compile(optimizer="adam", loss="sparse_categorical_crossentropy", metrics=["accuracy"])

# Ajuste fino tras unas épocas: descongelar y reducir la tasa de aprendizaje
base.trainable = True
modelo.compile(optimizer=keras.optimizers.Adam(1e-5), loss="sparse_categorical_crossentropy")
```

### 12.2 KerasHub

```bash
pip install keras-hub
```

```python
import keras_hub

clasificador = keras_hub.models.TextClassifier.from_preset("bert_base_en_uncased", num_classes=2)
clasificador.fit(textos, etiquetas, epochs=2)
```

KerasHub ofrece modelos de lenguaje (Gemma, Llama, BERT…), visión y audio con preprocesado incluido. Los pesos se descargan la primera vez.

---

## 13. Capas, modelos y pasos de entrenamiento personalizados

### 13.1 Capa propia

```python
class Escala(layers.Layer):
    def build(self, forma_entrada):
        self.w = self.add_weight(shape=(forma_entrada[-1],), initializer="ones", trainable=True)

    def call(self, x):
        return x * self.w
```

### 13.2 Pérdida o métrica propia

```python
def perdida_asimetrica(y_real, y_pred):
    error = y_real - y_pred
    return ops.mean(ops.where(error > 0, 2.0 * error, -error))     # Penaliza más infraestimar

modelo.compile(optimizer="adam", loss=perdida_asimetrica)
```

### 13.3 Personalizar `fit`

Sobrescribiendo `train_step` se conserva todo lo que ofrece `fit()` (callbacks, métricas, distribución). La implementación depende del backend (con TensorFlow usa `tf.GradientTape`; con PyTorch, `loss.backward()`); consulte la guía *Customizing what happens in fit()* de su backend.

---

## 14. Guardar, cargar y exportar

```python
modelo.save("modelo.keras")                              # Arquitectura + pesos + optimizador
modelo = keras.saving.load_model("modelo.keras")         # o keras.models.load_model

modelo.save_weights("modelo.weights.h5")                 # Solo pesos
modelo.load_weights("modelo.weights.h5")

modelo.export("modelo_servicio")                         # SavedModel (backend TensorFlow) para servir / LiteRT
modelo.export("modelo.onnx", format="onnx")              # ONNX
```

> En Keras 3, `save()` exige la extensión `.keras`. Las capas y funciones propias deben registrarse con `@keras.saving.register_keras_serializable()` o pasarse en `custom_objects` al cargar.

---

## 15. Código independiente del backend: `keras.ops`

`keras.ops` ofrece operaciones con la misma interfaz en los tres backends (equivalentes a NumPy y a funciones de redes neuronales):

```python
from keras import ops

ops.mean(x); ops.matmul(a, b); ops.exp(x); ops.where(c, a, b); ops.concatenate([a, b])
ops.softmax(x); ops.relu(x); ops.convert_to_numpy(t); ops.convert_to_tensor(arr)
```

> En capas y pérdidas propias use `keras.ops` en lugar de `tf.*`, `torch.*` o `jnp.*` para que el código funcione con cualquier backend.

---

## 16. Ejemplo práctico completo

**Tasación de viviendas con un modelo multientrada**: genera 12 000 anuncios sintéticos con variables numéricas (superficie, habitaciones, baños, antigüedad), categóricas (ciudad, tipo) y una **descripción en texto** («vivienda con terraza, piscina…»). Construye un modelo con la **API funcional** que combina normalización, *embeddings* de categorías y un vector TF-IDF del texto; entrena con *callbacks* (parada temprana, reducción de la tasa, punto de control); compara con una referencia de €/m² por ciudad; y guarda, recarga y usa el modelo para tasar viviendas nuevas. El modelo funciona sin cambios con los tres backends.

Requisitos: `pip install keras tensorflow pandas` (más `jax` o `torch` para esos backends)

```python
"""
Tasación de viviendas con Keras 3: modelo multientrada con la API funcional.

Combina variables numéricas, categóricas y la descripción en texto del anuncio.
Usa capas de preprocesado de Keras (Normalization dentro del modelo; StringLookup
y TextVectorization en una canalización tf.data), callbacks (parada temprana, reducción de la tasa de
aprendizaje, puntos de control), evalúa frente a una referencia y guarda el
modelo en formato .keras. El modelo funciona con los backends TensorFlow, JAX o PyTorch.
"""
import os
os.environ.setdefault("KERAS_BACKEND", "tensorflow")     # "jax" o "torch" también son válidos
os.environ.setdefault("TF_CPP_MIN_LOG_LEVEL", "2")

import keras
import numpy as np
import pandas as pd
import tensorflow as tf                     # Solo para tf.data y las capas de texto
from keras import layers

keras.utils.set_random_seed(42)
CIUDADES = {"Madrid": 4200, "Barcelona": 4000, "Valencia": 2100, "Sevilla": 2000,
            "Málaga": 2900, "Bilbao": 3000, "Zaragoza": 1700}
TIPOS = {"piso": 1.0, "ático": 1.25, "chalet": 1.15, "estudio": 0.95, "dúplex": 1.1}
EXTRAS = {"terraza": 0.06, "piscina": 0.08, "garaje": 0.05, "reformado": 0.07, "ascensor": 0.04,
          "vistas": 0.05, "trastero": 0.02, "exterior": 0.03, "para reformar": -0.12, "bajo": -0.06}
NUMERICAS = ["metros", "habitaciones", "banos", "antiguedad"]


# ------------------------------------------------------------------ Datos sintéticos
def generar_viviendas(n: int = 12_000, semilla: int = 0) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    ciudad = rng.choice(list(CIUDADES), n)
    tipo = rng.choice(list(TIPOS), n, p=[0.55, 0.1, 0.12, 0.13, 0.1])
    metros = np.where(tipo == "estudio", rng.normal(40, 8, n),
                      np.where(tipo == "chalet", rng.normal(180, 40, n), rng.normal(90, 25, n))).clip(25, 400)
    habitaciones = np.clip(np.round(metros / 30 + rng.normal(0, 0.6, n)), 1, 7)
    banos = np.clip(np.round(habitaciones / 2 + rng.normal(0, 0.4, n)), 1, 4)
    antiguedad = rng.integers(0, 80, n)
    descripciones, factores = [], []
    for _ in range(n):
        extras = rng.choice(list(EXTRAS), rng.integers(1, 5), replace=False)
        factores.append(1 + sum(EXTRAS[e] for e in extras))
        descripciones.append("vivienda con " + ", ".join(extras) + rng.choice(
            [" en zona tranquila", " cerca del metro", " junto a colegios", ""]))
    precio_m2 = (np.array([CIUDADES[c] for c in ciudad]) * np.array([TIPOS[t] for t in tipo])
                 * np.array(factores) * (1 - 0.003 * antiguedad) * rng.lognormal(0, 0.08, n))
    return pd.DataFrame({"ciudad": ciudad, "tipo": tipo, "metros": metros.round(),
                         "habitaciones": habitaciones, "banos": banos, "antiguedad": antiguedad,
                         "descripcion": descripciones, "precio": (precio_m2 * metros).round(-3)})


def a_dataset(df: pd.DataFrame, prep: dict, lote: int = 128, barajar: bool = False) -> tf.data.Dataset:
    """Canalización tf.data: las capas de texto/categorías se aplican aquí (válido con cualquier backend)."""
    entradas = {c: df[c].to_numpy("float32")[:, None] for c in NUMERICAS}
    entradas |= {c: df[c].to_numpy(str) for c in ["ciudad", "tipo", "descripcion"]}
    ds = tf.data.Dataset.from_tensor_slices((entradas, np.log(df["precio"].to_numpy("float32"))))
    if barajar:
        ds = ds.shuffle(len(df), seed=42)

    def preprocesar(x, y):
        x = dict(x)
        for col in ["ciudad", "tipo"]:
            x[col] = tf.cast(prep[col](x[col]), tf.int32)    # Texto → índice entero
        x["descripcion"] = prep["descripcion"](x["descripcion"])   # Texto → vector TF-IDF
        return x, y

    return ds.batch(lote).map(preprocesar, num_parallel_calls=tf.data.AUTOTUNE).prefetch(tf.data.AUTOTUNE)


# ------------------------------------------------------------------ Preprocesado y modelo
def crear_preprocesado(entrenamiento: pd.DataFrame) -> dict:
    """Capas de preprocesado de Keras ajustadas (adapt) solo con el conjunto de entrenamiento."""
    prep = {}
    for col in ["ciudad", "tipo"]:
        prep[col] = layers.StringLookup(name=f"indice_{col}")
        prep[col].adapt(entrenamiento[col].to_numpy(str))
    prep["descripcion"] = layers.TextVectorization(max_tokens=200, ngrams=2, output_mode="tf_idf")
    prep["descripcion"].adapt(entrenamiento["descripcion"].to_numpy(str))
    return prep


def crear_modelo(entrenamiento: pd.DataFrame, prep: dict) -> keras.Model:
    entradas, ramas = {}, []

    for col in NUMERICAS:                                      # Numéricas: normalización
        entradas[col] = keras.Input(shape=(1,), name=col)
        normalizacion = layers.Normalization(name=f"norm_{col}")
        normalizacion.adapt(entrenamiento[col].to_numpy("float32")[:, None])
        ramas.append(normalizacion(entradas[col]))

    for col, dim in [("ciudad", 4), ("tipo", 3)]:              # Categóricas: embedding
        entradas[col] = keras.Input(shape=(), dtype="int32", name=col)
        ramas.append(layers.Embedding(prep[col].vocabulary_size(), dim, name=f"emb_{col}")(entradas[col]))

    n_terminos = prep["descripcion"].vocabulary_size()         # Texto: vector TF-IDF
    entradas["descripcion"] = keras.Input(shape=(n_terminos,), name="descripcion")
    ramas.append(layers.Dense(16, activation="relu")(entradas["descripcion"]))

    x = layers.Concatenate()(ramas)
    x = layers.Dense(128, activation="relu")(x)
    x = layers.Dropout(0.1)(x)
    x = layers.Dense(64, activation="relu")(x)
    log_precio = layers.Dense(1, name="log_precio")(x)        # Se predice log(precio)
    return keras.Model(entradas, log_precio, name="tasador")


def main() -> None:
    print(f"Keras {keras.__version__} · backend: {keras.backend.backend()}")
    df = generar_viviendas()
    ent = df.sample(frac=0.7, random_state=1)
    resto = df.drop(ent.index)
    val, test = resto.iloc[: len(resto) // 2], resto.iloc[len(resto) // 2:]
    print(f"Viviendas · entrenamiento {len(ent):,} · validación {len(val):,} · prueba {len(test):,}")

    prep = crear_preprocesado(ent)
    ds_ent, ds_val, ds_test = a_dataset(ent, prep, barajar=True), a_dataset(val, prep), a_dataset(test, prep)
    modelo = crear_modelo(ent, prep)
    modelo.compile(optimizer=keras.optimizers.Adam(1e-3), loss="huber",
                   metrics=[keras.metrics.MeanAbsoluteError(name="mae_log")])
    print(f"Parámetros: {modelo.count_params():,}\n")

    callbacks = [
        keras.callbacks.EarlyStopping(monitor="val_loss", patience=10, restore_best_weights=True),
        keras.callbacks.ReduceLROnPlateau(monitor="val_loss", factor=0.5, patience=4),
        keras.callbacks.ModelCheckpoint("mejor_tasador.keras", monitor="val_loss", save_best_only=True),
    ]
    historial = modelo.fit(ds_ent, validation_data=ds_val, epochs=80, callbacks=callbacks,
                           shuffle=False, verbose=0)               # Ya se baraja en tf.data
    h = historial.history
    print(f"Épocas: {len(h['loss'])} · mejor val_loss {min(h['val_loss']):.5f}")

    # Evaluación en euros frente a una referencia: precio medio por m² de la ciudad
    pred = np.exp(modelo.predict(ds_test, verbose=0).ravel())
    real = test["precio"].to_numpy()
    m2_ciudad = (ent["precio"] / ent["metros"]).groupby(ent["ciudad"]).mean()
    referencia = test["ciudad"].map(m2_ciudad).to_numpy() * test["metros"].to_numpy()
    for nombre, p in [("Modelo Keras", pred), ("Referencia €/m² ciudad", referencia)]:
        error = np.abs(p - real)
        print(f"{nombre:<24} MAE {error.mean():>9,.0f} € · error mediano {np.median(error / real):6.1%}")

    # Guardar, recargar y tasar viviendas nuevas
    modelo.save("tasador.keras")
    cargado = keras.saving.load_model("tasador.keras")
    nuevas = pd.DataFrame([
        {"ciudad": "Málaga", "tipo": "ático", "metros": 95, "habitaciones": 3, "banos": 2,
         "antiguedad": 10, "descripcion": "vivienda con terraza, vistas, piscina cerca del metro"},
        {"ciudad": "Málaga", "tipo": "ático", "metros": 95, "habitaciones": 3, "banos": 2,
         "antiguedad": 10, "descripcion": "vivienda con para reformar, bajo"},
    ]).assign(precio=1.0)                                     # Precio ficticio: solo se usa la entrada
    for (_, v), p in zip(nuevas.iterrows(), np.exp(cargado.predict(a_dataset(nuevas, prep), verbose=0).ravel())):
        print(f"\n{v['tipo'].capitalize()} en {v['ciudad']}, {v['metros']} m² · «{v['descripcion']}»\n"
              f"  Tasación: {p:,.0f} € ({p / v['metros']:,.0f} €/m²)")


if __name__ == "__main__":
    main()
```

Resultado (backend TensorFlow):

```
Keras 3.15.1 · backend: tensorflow
Viviendas · entrenamiento 8,400 · validación 1,800 · prueba 1,800
Parámetros: 14,559

Épocas: 30 · mejor val_loss 0.00835
Modelo Keras             MAE    26,829 € · error mediano   8.1%
Referencia €/m² ciudad   MAE    36,060 € · error mediano  11.6%

Ático en Málaga, 95 m² · «vivienda con terraza, vistas, piscina cerca del metro»
  Tasación: 379,022 € (3,990 €/m²)

Ático en Málaga, 95 m² · «vivienda con para reformar, bajo»
  Tasación: 266,789 € (2,808 €/m²)
```

El mismo código con los tres backends (cambiando solo `KERAS_BACKEND`):

| Backend | Épocas | MAE en prueba | Error mediano |
|---|---|---|---|
| TensorFlow | 30 | 26,829 € | 8,1 % |
| PyTorch | 44 | 26,810 € | 8,2 % |
| JAX | 29 | 33,934 € | 10,3 % |

> Las diferencias entre backends se deben a que cada uno genera números aleatorios de forma distinta (inicialización de pesos, *dropout*), y la pérdida de validación de este modelo oscila de una época a otra. Para conclusiones firmes, repita el entrenamiento con varias semillas. El modelo capta el efecto de la descripción: el mismo ático vale unos 110 000 € menos si el anuncio indica «para reformar, bajo».

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ValueError: Input 0 of layer "dense" is incompatible with the layer: expected axis -1 ... to have value 10` | La forma de los datos no coincide con `Input(shape=...)`. | Revise `X.shape` y la forma declarada. |
| `ValueError: A Concatenate layer requires inputs with matching shapes except for the concatenation axis` | Ramas con distinto número de dimensiones (p. ej. `(None, 1, 16)` y `(None, 4)`). | Use `Flatten` o ajuste la forma de las entradas. |
| `ValueError: Invalid dtype: object` | Arrays de texto en `fit`. | Use `tf.data.Dataset` o preprocese el texto antes. |
| Pérdida `nan` | Tasa de aprendizaje alta o datos sin normalizar. | Normalice, reduzca la tasa, use `clipnorm=1.0` en el optimizador. |
| Exactitud estancada en el azar | Activación o pérdida incorrectas (p. ej. `softmax` con `binary_crossentropy`). | Revise la tabla de la sección 6. |
| `ValueError: Invalid filepath extension for saving` | `save()` sin `.keras`. | Use `modelo.keras` o `export()`. |
| Error al cargar un modelo con capas propias | Objetos personalizados no registrados. | `@keras.saving.register_keras_serializable()` o `custom_objects`. |
| El backend no cambia | `KERAS_BACKEND` se fijó después de `import keras`. | Fíjelo antes de cualquier importación de Keras. |
| Resultados distintos en cada ejecución | Aleatoriedad. | `keras.utils.set_random_seed(42)`. |
| Aviso `shuffle=True was passed, but will be ignored` | Se pasa un `tf.data.Dataset` a `fit`. | Baraje en el `Dataset` y use `shuffle=False`. |

---

## 18. Buenas prácticas

- Use la **API funcional** por defecto; recurra a subclases solo si necesita lógica dinámica.
- Ajuste las capas de preprocesado (`adapt`) **solo con datos de entrenamiento**.
- Elija la combinación correcta de **última capa y pérdida** para el tipo de problema.
- Use siempre **conjunto de validación**, `EarlyStopping(restore_best_weights=True)` y `ModelCheckpoint`.
- Compare con una **referencia sencilla** antes de complicar la red.
- Para datos tabulares, compare también con *gradient boosting*: suele ser igual o mejor con menos esfuerzo.
- Escriba capas y pérdidas con `keras.ops` para mantener la independencia del backend.
- Guarde los modelos en formato `.keras` y registre los objetos personalizados.
- Fije la semilla y registre versión de Keras, backend e hiperparámetros.

---

## 19. Referencia rápida

```python
import os; os.environ["KERAS_BACKEND"] = "tensorflow"   # o "jax" / "torch"
import keras; from keras import layers, ops

keras.Sequential([keras.Input((n,)), layers.Dense(64, activation="relu"), layers.Dense(1)])
e = keras.Input((n,)); s = layers.Dense(1)(layers.Dense(64, activation="relu")(e)); keras.Model(e, s)
class M(keras.Model): __init__ + call(self, x, training=False)

m.compile(optimizer=keras.optimizers.Adam(1e-3), loss="binary_crossentropy", metrics=["accuracy", keras.metrics.AUC()])
h = m.fit(X, y, validation_split=0.2, epochs=50, batch_size=64, callbacks=[...]); h.history
m.evaluate(X, y) · m.predict(X) · m.summary() · m.count_params()

keras.callbacks.EarlyStopping(patience=10, restore_best_weights=True) · ReduceLROnPlateau · ModelCheckpoint · TensorBoard
layers.Normalization().adapt(X) · StringLookup · TextVectorization · Embedding · Dropout · BatchNormalization
layers.Conv2D · MaxPooling2D · GlobalAveragePooling2D · LSTM · GRU · MultiHeadAttention · Concatenate

m.save("m.keras") · keras.saving.load_model("m.keras") · m.save_weights("m.weights.h5") · m.export("dir")
keras.utils.set_random_seed(42) · keras.config.backend() · keras.applications.* · keras_hub.models.*
```

---

## 20. Ejercicios propuestos

1. Entrene un clasificador de dígitos (`keras.datasets.mnist`) con un modelo `Sequential` denso y otro convolucional, y compare exactitud y parámetros.
2. Construya un modelo funcional con dos salidas que prediga a la vez el precio y el tipo de vivienda a partir de los datos del ejemplo.
3. Implemente una pérdida asimétrica con `keras.ops` que penalice más infraestimar el precio y analice cómo cambian las tasaciones.
4. Aplique aprendizaje por transferencia con `EfficientNetB0` a un conjunto de imágenes propio, primero con la base congelada y después con ajuste fino.
5. Ejecute el ejemplo completo con cinco semillas distintas en cada backend y calcule la media y la desviación del error.

---

## 21. Referencias

- Documentación oficial: https://keras.io/
- Guías para desarrolladores: https://keras.io/guides/
- Ejemplos de código: https://keras.io/examples/
- Capas de preprocesado: https://keras.io/api/layers/preprocessing_layers/
- KerasHub: https://keras.io/keras_hub/
