# Manual de uso: TensorFlow

> **Categoría:** Inteligencia artificial · **Utilidad:** Redes neuronales y Deep Learning
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** Apache 2.0 · **Desarrollo:** Google

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Tensores](#4-tensores)
5. [Variables y diferenciación automática (`GradientTape`)](#5-variables-y-diferenciación-automática-gradienttape)
6. [Grafos con `tf.function`](#6-grafos-con-tffunction)
7. [Canalizaciones de datos con `tf.data`](#7-canalizaciones-de-datos-con-tfdata)
8. [Modelos con Keras dentro de TensorFlow](#8-modelos-con-keras-dentro-de-tensorflow)
9. [Bucles de entrenamiento personalizados](#9-bucles-de-entrenamiento-personalizados)
10. [Supervisión con TensorBoard](#10-supervisión-con-tensorboard)
11. [GPU, precisión mixta y entrenamiento distribuido](#11-gpu-precisión-mixta-y-entrenamiento-distribuido)
12. [Guardar y exportar modelos](#12-guardar-y-exportar-modelos)
13. [Despliegue: TF Serving, LiteRT (TFLite) y TensorFlow.js](#13-despliegue-tf-serving-litert-tflite-y-tensorflowjs)
14. [Reproducibilidad y depuración](#14-reproducibilidad-y-depuración)
15. [Ecosistema](#15-ecosistema)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**TensorFlow** es una plataforma de código abierto para construir, entrenar y desplegar modelos de **aprendizaje profundo** (*Deep Learning*). Proporciona operaciones con tensores aceleradas por GPU/TPU, diferenciación automática, compilación a grafos optimizados, canalizaciones de datos eficientes y un ecosistema completo de despliegue en servidores, móviles, navegadores y dispositivos embebidos.

Desde TensorFlow 2, la API de alto nivel para definir y entrenar modelos es **Keras** (`tf.keras`), que se trata en detalle en su propio manual. Este manual se centra en las capacidades propias de TensorFlow: tensores, `GradientTape`, `tf.function`, `tf.data`, entrenamiento personalizado, distribución y despliegue.

**Ventajas:**
- Ecosistema de **producción** muy maduro: TF Serving, LiteRT (antes TensorFlow Lite), TensorFlow.js, TFX.
- Canalizaciones de datos de alto rendimiento (`tf.data`).
- Compilación a grafos y XLA para acelerar el cálculo.
- Soporte de GPU y TPU, y entrenamiento distribuido.
- Formato de exportación estándar (*SavedModel*) independiente del código Python.

**Limitaciones:**
- La investigación y los modelos de lenguaje recientes se publican mayoritariamente en PyTorch.
- La depuración dentro de `tf.function` es menos intuitiva que el código Python normal.
- Soporte de GPU nativo solo en Linux (en Windows, mediante WSL2).
- Paquete de gran tamaño y dependencias exigentes de versiones de CUDA.

| Si necesita… | Opción |
|---|---|
| Despliegue en producción, móviles o navegador; canalizaciones de datos a gran escala | **TensorFlow** |
| Investigación, modelos de lenguaje, comunidad de IA generativa | PyTorch |
| Definir y entrenar redes de forma sencilla | Keras (sobre TensorFlow, JAX o PyTorch) |
| Machine Learning clásico sobre tablas | scikit-learn, XGBoost, LightGBM |
| Cálculo numérico diferenciable con estilo NumPy | JAX |

---

## 2. Instalación e importación

```bash
pip install tensorflow                  # CPU en Windows/macOS; CPU en Linux
pip install "tensorflow[and-cuda]"      # Linux con GPU NVIDIA (instala las librerías CUDA)
pip install tensorboard                 # Visualización (ya no siempre se instala con TensorFlow)
```

> Compruebe en la documentación oficial qué versiones de Python admite cada versión de TensorFlow. En Windows, para usar GPU, instale TensorFlow dentro de **WSL2**.

Comprobar la instalación y los dispositivos:

```python
import tensorflow as tf
print(tf.__version__)
print(tf.config.list_physical_devices())       # [PhysicalDevice(name='/physical_device:CPU:0', ...), GPU...]
```

Importaciones habituales:

```python
import numpy as np
import tensorflow as tf
from tensorflow import keras        # Equivale a «import keras» (Keras 3)
```

Reducir los mensajes informativos de arranque (antes de importar TensorFlow):

```python
import os
os.environ["TF_CPP_MIN_LOG_LEVEL"] = "2"     # 0 = todo, 1 = sin INFO, 2 = sin WARNING, 3 = sin ERROR
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Tensor (`tf.Tensor`) | Array multidimensional **inmutable** con un tipo (`dtype`) y una forma (`shape`). |
| Variable (`tf.Variable`) | Tensor **modificable**; almacena los pesos de los modelos. |
| Ejecución inmediata (*eager*) | Las operaciones se ejecutan al momento, como en NumPy. Modo por defecto. |
| Grafo | Representación optimizada del cálculo, generada con `tf.function`. |
| `GradientTape` | Registra operaciones para calcular gradientes automáticamente. |
| `tf.data.Dataset` | Canalización eficiente de lectura, transformación y agrupación de datos. |
| Capa / modelo | Bloques de la red neuronal (Keras). |
| Época | Pasada completa por los datos de entrenamiento. |
| Lote (*batch*) | Subconjunto de muestras procesado en cada paso de optimización. |
| *SavedModel* | Formato de exportación con el grafo y los pesos, utilizable sin el código Python. |

**Flujo de trabajo típico:**

```
Datos ─► tf.data (leer, transformar, lotes) ─► Modelo (Keras) ─► Entrenamiento (fit o bucle propio)
      ─► Evaluación ─► Exportación (SavedModel / LiteRT / TF.js) ─► Despliegue
```

---

## 4. Tensores

### 4.1 Crear tensores

```python
a = tf.constant([[1.0, 2.0], [3.0, 4.0]])
tf.zeros((3, 4)); tf.ones((2, 2)); tf.eye(3); tf.range(10); tf.linspace(0.0, 1.0, 5)
tf.random.normal((3, 3), mean=0.0, stddev=1.0, seed=42)
tf.convert_to_tensor(np.array([1, 2, 3]))

a.shape          # TensorShape([2, 2])
a.dtype          # tf.float32
a.numpy()        # Conversión a NumPy
```

### 4.2 Operaciones

```python
a + b; a * b; a @ b                      # Elemento a elemento y producto matricial
tf.matmul(a, b); tf.transpose(a); tf.reshape(a, (4,)); tf.concat([a, b], axis=0)
tf.reduce_sum(a); tf.reduce_mean(a, axis=0); tf.reduce_max(a); tf.argmax(a, axis=1)
tf.nn.relu(x); tf.nn.softmax(x); tf.math.log(x); tf.sqrt(x)
tf.where(a > 2, a, 0.0)
tf.one_hot([0, 2], depth=3)
tf.cast(tf.constant([1, 2]), tf.float32)  # Conversión de tipo
```

> TensorFlow **no convierte tipos automáticamente**: `tf.constant([1, 2]) + tf.constant([1.0, 2.0])` lanza `InvalidArgumentError`. Use `tf.cast`.

### 4.3 Tipos de tensores especiales

| Tipo | Uso |
|---|---|
| `tf.RaggedTensor` | Filas de longitud variable (frases, secuencias) |
| `tf.SparseTensor` | Tensores dispersos (mayoría de ceros) |
| `tf.string` | Texto: `tf.strings.split`, `tf.strings.lower`… |

---

## 5. Variables y diferenciación automática (`GradientTape`)

### 5.1 Variables

```python
w = tf.Variable(tf.random.normal((3, 1)), name="pesos")
w.assign(tf.zeros((3, 1)))
w.assign_add(tf.ones((3, 1)))
```

### 5.2 Gradientes

```python
x = tf.Variable(2.0)
with tf.GradientTape() as cinta:
    y = x ** 3
cinta.gradient(y, x)          # 12.0 → dy/dx = 3x²
```

Regresión lineal desde cero:

```python
w, b = tf.Variable(0.0), tf.Variable(0.0)
optimizador = tf.keras.optimizers.SGD(learning_rate=0.1)

for paso in range(200):
    with tf.GradientTape() as cinta:
        pred = w * x_datos + b
        perdida = tf.reduce_mean((pred - y_datos) ** 2)
    grads = cinta.gradient(perdida, [w, b])
    optimizador.apply_gradients(zip(grads, [w, b]))
```

| Opción | Uso |
|---|---|
| `tf.GradientTape(persistent=True)` | Calcular varios gradientes con la misma cinta |
| `cinta.watch(tensor)` | Derivar respecto a un tensor que no es `Variable` |
| `tf.stop_gradient(x)` | Tratar `x` como constante |
| Cintas anidadas | Derivadas de segundo orden |

---

## 6. Grafos con `tf.function`

`tf.function` convierte una función de Python en un **grafo** de TensorFlow: más rápido, exportable y ejecutable sin Python.

```python
@tf.function
def paso(x):
    return tf.reduce_sum(x ** 2)

paso(tf.ones(3))       # 1.ª llamada: traza (ejecuta el Python y construye el grafo)
paso(tf.ones(3))       # Siguientes: ejecuta el grafo
```

### 6.1 Trazado y retrazado

La función se **retraza** (se reconstruye el grafo) cuando cambian la forma o el tipo de los tensores, o **cualquier valor** de un argumento Python:

```python
@tf.function
def g(x):
    print("Trazando")          # Solo se muestra al trazar
    tf.print("Ejecutando", x)  # Se muestra en cada ejecución
    return x + 1

g(tf.constant(1)); g(tf.constant(2))     # Una sola traza (mismo tipo y forma)
g(3); g(4)                               # Una traza por cada valor Python distinto ✗
```

> Pase **tensores**, no números de Python, a las funciones decoradas. Para fijar la firma: `@tf.function(input_signature=[tf.TensorSpec([None, 10], tf.float32)])`.

### 6.2 AutoGraph: control de flujo

Dentro de `tf.function`, `if` y `for`/`while` sobre **tensores** se convierten automáticamente en operaciones del grafo (`tf.cond`, `tf.while_loop`):

```python
@tf.function
def recortar(x):
    if tf.reduce_sum(x) > 10:      # Condición sobre un tensor: se convierte en tf.cond
        return x / 2
    return x
```

### 6.3 Compilación XLA

```python
@tf.function(jit_compile=True)       # Fusiona operaciones; mejora notable en GPU
def paso_rapido(x): ...
```

---

## 7. Canalizaciones de datos con `tf.data`

### 7.1 Crear un `Dataset`

```python
ds = tf.data.Dataset.from_tensor_slices((X, y))               # Desde arrays de NumPy o tensores
ds = tf.data.Dataset.from_tensor_slices(dict(df))             # Desde un DataFrame
ds = tf.data.TFRecordDataset(["datos-000.tfrecord"])           # Formato binario de TensorFlow
ds = tf.data.Dataset.list_files("imagenes/*.jpg")
ds = tf.keras.utils.image_dataset_from_directory("imagenes/", image_size=(224, 224), batch_size=32)
ds = tf.keras.utils.text_dataset_from_directory("textos/")
ds = tf.keras.utils.timeseries_dataset_from_array(serie, objetivos, sequence_length=24)
```

### 7.2 Transformaciones

```python
ds = (ds
      .shuffle(10_000, seed=42)                         # Barajar (búfer de 10 000 elementos)
      .map(preprocesar, num_parallel_calls=tf.data.AUTOTUNE)   # Transformar en paralelo
      .filter(lambda x, y: y >= 0)
      .batch(64)                                        # Agrupar en lotes
      .cache()                                          # Guardar en memoria tras la primera época
      .prefetch(tf.data.AUTOTUNE))                      # Preparar el siguiente lote mientras se entrena

for x_lote, y_lote in ds.take(1):
    print(x_lote.shape)
```

| Método | Función |
|---|---|
| `map` | Aplicar una función a cada elemento |
| `filter` | Descartar elementos |
| `shuffle` | Barajar (el búfer debe ser grande respecto al conjunto) |
| `batch` / `padded_batch` | Agrupar (con relleno para secuencias de longitud variable) |
| `cache` | Evitar recalcular en cada época |
| `prefetch` | Solapar preparación de datos y entrenamiento |
| `window` + `flat_map` | Ventanas deslizantes (series temporales) |
| `interleave` | Leer varios archivos en paralelo |
| `take`, `skip`, `repeat` | Seleccionar o repetir |

> Orden recomendado: `shuffle` → `map` → `batch` → `cache` (si cabe en memoria) → `prefetch`.

---

## 8. Modelos con Keras dentro de TensorFlow

TensorFlow utiliza **Keras 3** como API de modelos (`tf.keras` equivale a `keras`). Para un tratamiento completo consulte el manual de Keras.

```python
modelo = tf.keras.Sequential([
    tf.keras.Input(shape=(20,)),
    tf.keras.layers.Dense(64, activation="relu"),
    tf.keras.layers.Dropout(0.2),
    tf.keras.layers.Dense(1, activation="sigmoid"),
])
modelo.compile(optimizer="adam", loss="binary_crossentropy", metrics=["auc"])
modelo.fit(ds_entrenamiento, validation_data=ds_validacion, epochs=20,
           callbacks=[tf.keras.callbacks.EarlyStopping(patience=3, restore_best_weights=True)])
modelo.evaluate(ds_prueba)
modelo.predict(x_nuevos)
```

`fit` acepta directamente objetos `tf.data.Dataset`.

---

## 9. Bucles de entrenamiento personalizados

Cuando se necesita un control total (GAN, pérdidas complejas, aprendizaje por refuerzo, recorte de gradientes personalizado), se escribe el bucle con `GradientTape`:

```python
optimizador = tf.keras.optimizers.Adam(1e-3)
perdida_fn = tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True)
exactitud = tf.keras.metrics.SparseCategoricalAccuracy()

@tf.function
def paso_entrenamiento(x, y):
    with tf.GradientTape() as cinta:
        logits = modelo(x, training=True)            # training=True activa Dropout/BatchNorm
        perdida = perdida_fn(y, logits) + sum(modelo.losses)   # + regularización
    grads = cinta.gradient(perdida, modelo.trainable_variables)
    optimizador.apply_gradients(zip(grads, modelo.trainable_variables))
    exactitud.update_state(y, logits)
    return perdida

for epoca in range(10):
    exactitud.reset_state()
    for x, y in ds_entrenamiento:
        perdida = paso_entrenamiento(x, y)
    print(epoca, float(perdida), float(exactitud.result()))
```

> Alternativa intermedia: sobrescribir `train_step` en una subclase de `keras.Model` para personalizar el paso y seguir usando `fit()` con sus *callbacks*.

---

## 10. Supervisión con TensorBoard

```bash
pip install tensorboard
```

```python
# Con fit
modelo.fit(..., callbacks=[tf.keras.callbacks.TensorBoard(log_dir="logs/exp1", histogram_freq=1)])

# En un bucle propio
escritor = tf.summary.create_file_writer("logs/exp2")
with escritor.as_default():
    tf.summary.scalar("perdida", perdida, step=epoca)
    tf.summary.histogram("pesos", modelo.layers[0].kernel, step=epoca)
```

```bash
tensorboard --logdir logs          # Abrir http://localhost:6006
```

---

## 11. GPU, precisión mixta y entrenamiento distribuido

### 11.1 GPU

TensorFlow usa la GPU automáticamente si está disponible.

```python
gpus = tf.config.list_physical_devices("GPU")
for gpu in gpus:
    tf.config.experimental.set_memory_growth(gpu, True)   # No reservar toda la memoria al inicio

with tf.device("/CPU:0"):          # Forzar un dispositivo
    resultado = tf.matmul(a, b)
```

### 11.2 Precisión mixta

En GPU modernas, calcular en `float16` acelera el entrenamiento y reduce la memoria:

```python
tf.keras.mixed_precision.set_global_policy("mixed_float16")
# La última capa debe producir float32 para estabilidad numérica:
salida = tf.keras.layers.Dense(10, dtype="float32")(x)
```

### 11.3 Entrenamiento distribuido

```python
estrategia = tf.distribute.MirroredStrategy()        # Varias GPU en una máquina
print(estrategia.num_replicas_in_sync)
with estrategia.scope():
    modelo = crear_modelo()
    modelo.compile(...)
modelo.fit(ds, epochs=10)
```

| Estrategia | Uso |
|---|---|
| `MirroredStrategy` | Varias GPU en una máquina |
| `MultiWorkerMirroredStrategy` | Varias máquinas |
| `TPUStrategy` | TPU (Google Cloud, Colab) |

---

## 12. Guardar y exportar modelos

| Formato | Cómo | Para qué |
|---|---|---|
| `.keras` | `modelo.save("modelo.keras")` / `tf.keras.models.load_model(...)` | Guardar y reanudar en Python (arquitectura + pesos + optimizador) |
| Pesos | `modelo.save_weights("m.weights.h5")` | Solo los pesos |
| *SavedModel* | `modelo.export("directorio")` / `tf.saved_model.load(...)` | Servir en producción, convertir a LiteRT o TF.js |
| *Checkpoints* | `tf.train.Checkpoint(modelo=m, optimizador=o).save("ckpt/c")` | Bucles personalizados, entrenamientos largos |

```python
modelo.export("modelo_servicio", verbose=False)
cargado = tf.saved_model.load("modelo_servicio")
cargado.serve(x)                      # Firma de inferencia
```

> En Keras 3, `modelo.save()` solo admite el formato `.keras`. Para obtener un *SavedModel*, use `modelo.export()`.

---

## 13. Despliegue: TF Serving, LiteRT (TFLite) y TensorFlow.js

### 13.1 TensorFlow Serving (servidor)

```bash
docker run -p 8501:8501 \
  -v "$PWD/modelo_servicio:/models/demanda/1" -e MODEL_NAME=demanda tensorflow/serving
curl -X POST http://localhost:8501/v1/models/demanda:predict -d '{"instances": [...]}'
```

### 13.2 LiteRT (antes TensorFlow Lite): móviles y dispositivos

```python
conversor = tf.lite.TFLiteConverter.from_saved_model("modelo_servicio")
conversor.optimizations = [tf.lite.Optimize.DEFAULT]       # Cuantización: modelo ~4 veces menor
with open("modelo.tflite", "wb") as f:
    f.write(conversor.convert())
```

Inferencia con el intérprete LiteRT (`pip install ai-edge-litert`):

```python
from ai_edge_litert.interpreter import Interpreter

interprete = Interpreter(model_path="modelo.tflite")
interprete.allocate_tensors()
entrada = interprete.get_input_details()[0]
salida = interprete.get_output_details()[0]
interprete.set_tensor(entrada["index"], x.astype(np.float32))
interprete.invoke()
resultado = interprete.get_tensor(salida["index"])
```

> `tf.lite.Interpreter` está en desuso y será eliminado: use el paquete **`ai-edge-litert`**. Las capas recurrentes (LSTM, GRU) suelen requerir formas estáticas para la conversión (véase el ejemplo completo).

### 13.3 TensorFlow.js (navegador)

```bash
pip install tensorflowjs
tensorflowjs_converter --input_format=tf_saved_model modelo_servicio modelo_web/
```

---

## 14. Reproducibilidad y depuración

```python
tf.keras.utils.set_random_seed(42)                # Python, NumPy y TensorFlow a la vez
tf.config.experimental.enable_op_determinism()    # Operaciones deterministas (más lento)

tf.config.run_functions_eagerly(True)             # Ejecutar tf.function como Python (depurar)
tf.debugging.check_numerics(x, "x contiene NaN o Inf")
tf.debugging.assert_shapes([(x, ("N", 72, 6))])
tf.debugging.enable_check_numerics()              # Detectar NaN/Inf en todas las operaciones
```

---

## 15. Ecosistema

| Componente | Uso |
|---|---|
| **Keras** | API de alto nivel para modelos |
| **TensorBoard** | Visualización del entrenamiento |
| **TensorFlow Datasets** (`tensorflow-datasets`) | Cientos de conjuntos de datos listos para `tf.data` |
| **Kaggle Models / KerasHub** | Modelos preentrenados |
| **TF Serving** | Servidor de modelos de alto rendimiento |
| **LiteRT** | Inferencia en móviles, microcontroladores y dispositivos |
| **TensorFlow.js** | Inferencia y entrenamiento en navegador y Node.js |
| **TFX** | Canalizaciones de ML en producción (validación, entrenamiento, despliegue) |
| **TensorFlow Probability** | Modelos probabilísticos y estadística bayesiana |

---

## 16. Ejemplo práctico completo

**Previsión de la demanda eléctrica horaria**: genera dos años de demanda horaria con estacionalidad diaria y semanal y efecto de la temperatura; construye ventanas de 72 horas con `tf.data` (ventanas, barajado, lotes, caché y *prefetch*); entrena una red Conv1D + LSTM con un **bucle de entrenamiento propio** (`GradientTape` + `tf.function`, recorte de gradientes, parada temprana); compara con una referencia ingenua (misma hora del día anterior); exporta el modelo como *SavedModel* y lo convierte a LiteRT con cuantización.

Requisitos: `pip install tensorflow numpy ai-edge-litert`

```python
"""
Previsión de la demanda eléctrica horaria con TensorFlow.

Genera una serie horaria de demanda con estacionalidad diaria y semanal y efecto
de la temperatura, construye ventanas con tf.data, entrena una red con un bucle
de entrenamiento propio (GradientTape + tf.function) con parada temprana,
compara con una referencia ingenua, exporta el modelo (SavedModel) y lo
convierte a TensorFlow Lite.
"""
import os
os.environ.setdefault("TF_CPP_MIN_LOG_LEVEL", "2")      # Menos mensajes informativos

import time

import numpy as np
import tensorflow as tf

tf.keras.utils.set_random_seed(42)

VENTANA = 72            # Horas de historia que ve el modelo
HORIZONTE = 24          # Horas a predecir
LOTE = 64
EPOCAS_MAX = 40
PACIENCIA = 5


# ------------------------------------------------------------------ Datos
def generar_serie(horas: int = 24 * 365 * 2) -> tuple[np.ndarray, np.ndarray]:
    """Demanda (MW) y temperatura (°C) horarias de dos años."""
    rng = np.random.default_rng(0)
    t = np.arange(horas)
    hora, dia_semana, dia_anio = t % 24, (t // 24) % 7, (t // 24) % 365
    temperatura = 15 - 9 * np.cos(2 * np.pi * (dia_anio - 15) / 365) \
        + 4 * np.sin(2 * np.pi * (hora - 9) / 24) + rng.normal(0, 1.5, horas)
    perfil_diario = 1 + 0.25 * np.exp(-((hora - 13) / 3.5) ** 2) + 0.30 * np.exp(-((hora - 20.5) / 2) ** 2)
    laborable = np.where(dia_semana < 5, 1.0, 0.85)
    climatizacion = 0.012 * np.abs(temperatura - 18) ** 1.4
    demanda = 28_000 * perfil_diario * laborable * (1 + climatizacion) + rng.normal(0, 600, horas)
    return demanda.astype("float32"), temperatura.astype("float32")


def caracteristicas(demanda, temperatura) -> np.ndarray:
    t = np.arange(len(demanda))
    hora, dia_semana = t % 24, (t // 24) % 7
    return np.stack([demanda, temperatura,
                     np.sin(2 * np.pi * hora / 24), np.cos(2 * np.pi * hora / 24),
                     np.sin(2 * np.pi * dia_semana / 7), np.cos(2 * np.pi * dia_semana / 7)],
                    axis=1).astype("float32")


def crear_ventanas(datos: np.ndarray, barajar: bool) -> tf.data.Dataset:
    """(VENTANA horas de todas las variables) → (HORIZONTE horas de demanda normalizada)."""
    ds = tf.data.Dataset.from_tensor_slices(datos)
    ds = ds.window(VENTANA + HORIZONTE, shift=1, drop_remainder=True)
    ds = ds.flat_map(lambda w: w.batch(VENTANA + HORIZONTE))
    ds = ds.map(lambda w: (w[:VENTANA], w[VENTANA:, 0]), num_parallel_calls=tf.data.AUTOTUNE)
    if barajar:
        ds = ds.shuffle(5_000, seed=42)
    return ds.batch(LOTE).cache().prefetch(tf.data.AUTOTUNE)


# ------------------------------------------------------------------ Modelo
def crear_modelo(n_variables: int) -> tf.keras.Model:
    entrada = tf.keras.Input(shape=(VENTANA, n_variables), name="historia")
    x = tf.keras.layers.Conv1D(32, kernel_size=6, activation="relu")(entrada)
    x = tf.keras.layers.LSTM(64)(x)
    x = tf.keras.layers.Dense(64, activation="relu")(x)
    salida = tf.keras.layers.Dense(HORIZONTE, name="prevision")(x)
    return tf.keras.Model(entrada, salida, name="prevision_demanda")


def main() -> None:
    print(f"TensorFlow {tf.__version__} · dispositivos: "
          f"{[d.device_type for d in tf.config.list_physical_devices()]}")
    demanda, temperatura = generar_serie()
    datos = caracteristicas(demanda, temperatura)

    # División temporal (nunca aleatoria en series): 70 % / 15 % / 15 %
    n = len(datos)
    ent, val, test = datos[: int(n * 0.7)], datos[int(n * 0.7): int(n * 0.85)], datos[int(n * 0.85):]
    media, desv = ent.mean(axis=0), ent.std(axis=0)            # Solo con entrenamiento
    normalizar = lambda d: (d - media) / desv
    ds_ent, ds_val, ds_test = (crear_ventanas(normalizar(ent), True),
                               crear_ventanas(normalizar(val), False),
                               crear_ventanas(normalizar(test), False))

    modelo = crear_modelo(datos.shape[1])
    optimizador = tf.keras.optimizers.Adam(learning_rate=1e-3)
    perdida_fn = tf.keras.losses.Huber()
    mae_val = tf.keras.metrics.MeanAbsoluteError()

    @tf.function                                                # Compila el paso a un grafo
    def paso_entrenamiento(x, y):
        with tf.GradientTape() as cinta:
            pred = modelo(x, training=True)
            perdida = perdida_fn(y, pred)
        gradientes = cinta.gradient(perdida, modelo.trainable_variables)
        gradientes, _ = tf.clip_by_global_norm(gradientes, 1.0)
        optimizador.apply_gradients(zip(gradientes, modelo.trainable_variables))
        return perdida

    @tf.function
    def paso_validacion(x, y):
        mae_val.update_state(y, modelo(x, training=False))

    print(f"Parámetros del modelo: {modelo.count_params():,}\n")
    mejor, sin_mejora, mejores_pesos = np.inf, 0, None
    inicio = time.perf_counter()
    for epoca in range(1, EPOCAS_MAX + 1):
        perdidas = [paso_entrenamiento(x, y) for x, y in ds_ent]
        mae_val.reset_state()
        for x, y in ds_val:
            paso_validacion(x, y)
        mae_mw = float(mae_val.result()) * desv[0]
        print(f"Época {epoca:2d} · pérdida {float(tf.reduce_mean(perdidas)):.4f} · MAE val {mae_mw:,.0f} MW")
        if mae_mw < mejor - 1:
            mejor, sin_mejora, mejores_pesos = mae_mw, 0, modelo.get_weights()
        else:
            sin_mejora += 1
            if sin_mejora >= PACIENCIA:
                print(f"Parada temprana: {PACIENCIA} épocas sin mejora")
                break
    modelo.set_weights(mejores_pesos)
    print(f"Entrenamiento: {time.perf_counter() - inicio:.0f} s\n")

    # Evaluación en prueba frente a la referencia «misma hora del día anterior»
    pred = np.concatenate([modelo(x, training=False).numpy() for x, _ in ds_test]) * desv[0] + media[0]
    real = np.concatenate([y.numpy() for _, y in ds_test]) * desv[0] + media[0]
    ingenua = np.concatenate([x[:, -24:, 0].numpy() for x, _ in ds_test]) * desv[0] + media[0]
    mae_modelo, mae_ingenua = np.abs(pred - real).mean(), np.abs(ingenua - real).mean()
    mape = np.mean(np.abs(pred - real) / real)
    print(f"Prueba · MAE modelo {mae_modelo:,.0f} MW (MAPE {mape:.2%}) · "
          f"MAE referencia ingenua {mae_ingenua:,.0f} MW · mejora {1 - mae_modelo / mae_ingenua:.0%}")

    # Exportar: SavedModel para servir y TensorFlow Lite para dispositivos
    modelo.export("modelo_demanda", verbose=False)               # Directorio SavedModel
    recargado = tf.saved_model.load("modelo_demanda")
    x_ej = next(iter(ds_test))[0][:1]
    diferencia = np.abs(recargado.serve(x_ej).numpy() - modelo(x_ej).numpy()).max()
    print(f"SavedModel exportado y recargado (diferencia máxima {diferencia:.1e})")

    # LiteRT (TFLite) necesita formas estáticas para la LSTM: se exporta con lote fijo de 1
    n_var = datos.shape[1]
    modelo.export("modelo_demanda_lite", verbose=False,
                  input_signature=[tf.TensorSpec([1, VENTANA, n_var], tf.float32)])
    conversor = tf.lite.TFLiteConverter.from_saved_model("modelo_demanda_lite")
    conversor.optimizations = [tf.lite.Optimize.DEFAULT]          # Cuantización de pesos
    tflite = conversor.convert()
    with open("modelo_demanda.tflite", "wb") as f:
        f.write(tflite)

    # Comprobar el modelo LiteRT con su intérprete (pip install ai-edge-litert)
    from ai_edge_litert.interpreter import Interpreter
    interprete = Interpreter(model_content=tflite)
    interprete.allocate_tensors()
    entrada, salida = interprete.get_input_details()[0], interprete.get_output_details()[0]
    interprete.set_tensor(entrada["index"], x_ej.numpy())
    interprete.invoke()
    error_lite = np.abs(interprete.get_tensor(salida["index"]) - modelo(x_ej).numpy()).max() * desv[0]
    print(f"Modelo LiteRT: {len(tflite) / 1024:.0f} KB · diferencia máxima tras cuantizar {error_lite:.0f} MW")


if __name__ == "__main__":
    main()
```

Resultado (CPU de 2 núcleos):

```
TensorFlow 2.21.0 · dispositivos: ['CPU']
Parámetros del modelo: 31,736

Época  1 · pérdida 0.1771 · MAE val 1,873 MW
Época  2 · pérdida 0.0902 · MAE val 1,547 MW
Época  3 · pérdida 0.0639 · MAE val 1,429 MW
...
Época 37 · pérdida 0.0454 · MAE val 1,273 MW
Época 38 · pérdida 0.0453 · MAE val 1,273 MW
Parada temprana: 5 épocas sin mejora
Entrenamiento: 169 s

Prueba · MAE modelo 1,343 MW (MAPE 3.76%) · MAE referencia ingenua 2,866 MW · mejora 53%
SavedModel exportado y recargado (diferencia máxima 0.0e+00)
Modelo LiteRT: 50 KB · diferencia máxima tras cuantizar 109 MW
```

> El modelo reduce el error un 53 % respecto a la referencia ingenua, con un error porcentual del 3,8 %. Para LiteRT se exporta una versión con tamaño de lote fijo (1): con lote variable, el conversor falla al procesar la LSTM (`'tf.TensorListReserve' op requires element_shape to be static`). La cuantización reduce el tamaño a costa de una pequeña diferencia en las predicciones (109 MW como máximo, frente a un error medio de 1 343 MW).

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `InvalidArgumentError: cannot compute AddV2 as input #1 was expected to be a int32 tensor but is a float tensor` | Tipos distintos en una operación. | Convierta con `tf.cast`. |
| `ValueError: Input 0 of layer ... is incompatible with the layer: expected shape=(None, 20), found shape=(20,)` | Falta la dimensión de lote. | Use `x[None, ...]` o `tf.expand_dims(x, 0)`. |
| Entrenamiento muy lento con `tf.function` | Retrazado continuo (argumentos Python o formas cambiantes). | Pase tensores, fije `input_signature` o use `reduce_retracing=True`. |
| `print` dentro de `tf.function` solo se muestra una vez | Se ejecuta durante el trazado. | Use `tf.print`. |
| Pérdida `NaN` | Tasa de aprendizaje alta, datos sin normalizar, `log(0)`. | Normalice, reduzca la tasa, recorte gradientes, use `check_numerics`. |
| `ResourceExhaustedError: OOM` | Falta memoria de GPU. | Reduzca el lote, active `set_memory_growth`, use precisión mixta. |
| No se detecta la GPU | Instalación sin CUDA, Windows nativo o versiones incompatibles. | `pip install "tensorflow[and-cuda]"` en Linux/WSL2; revise el driver. |
| `TBNotInstalledError` | TensorBoard no instalado. | `pip install tensorboard`. |
| `ConverterError ... TensorListReserve` al convertir a LiteRT | LSTM/GRU con forma dinámica. | Exporte con `input_signature` de forma fija (`modelo.export(dir, input_signature=[tf.TensorSpec([1, ...])])`) o habilite `SELECT_TF_OPS`. |
| `ValueError: Invalid filepath extension for saving` | `modelo.save()` sin extensión `.keras` (Keras 3). | Use `.keras` o `modelo.export()` para *SavedModel*. |
| Resultados distintos en cada ejecución | Aleatoriedad no fijada. | `set_random_seed` y `enable_op_determinism`. |
| `AutoGraph could not transform <lambda>` | Varias *lambdas* en la misma línea dentro de `tf.data`. | Defina funciones con nombre o una *lambda* por línea. |

---

## 18. Buenas prácticas

- Use **`tf.data`** con `prefetch(tf.data.AUTOTUNE)` y `num_parallel_calls` para que la GPU no espere a los datos.
- Normalice las entradas con estadísticas calculadas **solo** sobre el conjunto de entrenamiento.
- En series temporales, divida los datos **por tiempo**, nunca aleatoriamente.
- Empiece con `model.fit()` y pase a bucles propios solo cuando lo necesite.
- Decore los pasos de entrenamiento con **`tf.function`** y evite argumentos Python que provoquen retrazados.
- Compare siempre con una **referencia sencilla** (ingenua, modelo lineal, *gradient boosting*).
- Use parada temprana y guarde los mejores pesos.
- Registre los experimentos con TensorBoard (o MLflow) incluyendo hiperparámetros y versión de los datos.
- Exporte con `modelo.export()` y verifique que el modelo recargado produce las mismas predicciones.
- Fije las versiones de TensorFlow, CUDA y Python del entorno de producción.

---

## 19. Referencia rápida

```python
import tensorflow as tf
tf.constant(x) · tf.Variable(x) · t.numpy() · tf.cast(t, tf.float32) · tf.reshape · tf.concat · tf.reduce_mean
with tf.GradientTape() as c: perdida = ...   →   c.gradient(perdida, vars)
@tf.function · @tf.function(input_signature=[tf.TensorSpec([None, n], tf.float32)]) · tf.print(...)

tf.data.Dataset.from_tensor_slices((X, y)).shuffle(n).map(f, num_parallel_calls=tf.data.AUTOTUNE).batch(64).cache().prefetch(tf.data.AUTOTUNE)
tf.keras.utils.image_dataset_from_directory · text_dataset_from_directory · timeseries_dataset_from_array

opt = tf.keras.optimizers.Adam(1e-3); opt.apply_gradients(zip(grads, modelo.trainable_variables))
modelo.fit(ds, validation_data=ds_val, epochs=n, callbacks=[...]) · modelo.evaluate · modelo.predict

modelo.save("m.keras") · tf.keras.models.load_model("m.keras") · modelo.export("dir") · tf.saved_model.load("dir")
tf.lite.TFLiteConverter.from_saved_model("dir").convert()

tf.config.list_physical_devices("GPU") · tf.distribute.MirroredStrategy() · tf.keras.mixed_precision.set_global_policy("mixed_float16")
tf.keras.utils.set_random_seed(42) · tf.config.run_functions_eagerly(True) · tf.summary.create_file_writer("logs")
```

---

## 20. Ejercicios propuestos

1. Implemente una regresión logística desde cero con `tf.Variable` y `GradientTape`, y compare los coeficientes con los de scikit-learn.
2. Construya una canalización `tf.data` que lea imágenes de un directorio, aplique aumento de datos aleatorio y mida con TensorBoard el tiempo de cada paso con y sin `prefetch`.
3. Entrene un clasificador de MNIST (`tf.keras.datasets.mnist`) con `fit` y con un bucle personalizado, y compare tiempos con y sin `tf.function`.
4. Convierta el modelo de MNIST a LiteRT con cuantización y mida la diferencia de tamaño y exactitud.
5. Amplíe el ejemplo completo añadiendo la previsión de temperatura como entrada futura conocida y analice cuánto mejora el error.

---

## 21. Referencias

- Documentación oficial: https://www.tensorflow.org/
- Guías: https://www.tensorflow.org/guide
- `tf.data`: https://www.tensorflow.org/guide/data
- `tf.function` y rendimiento: https://www.tensorflow.org/guide/function
- LiteRT: https://ai.google.dev/edge/litert
- Tutoriales: https://www.tensorflow.org/tutorials
