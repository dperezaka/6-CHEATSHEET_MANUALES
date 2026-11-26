# Manual de uso: JAX

> **Categoría:** Computación numérica y aprendizaje automático · **Utilidad:** Cálculo numérico y diferenciación automática
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** Apache 2.0 · **Desarrollo:** Google

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [`jax.numpy`: arrays inmutables](#4-jaxnumpy-arrays-inmutables)
5. [Diferenciación automática](#5-diferenciación-automática)
6. [Compilación con `jit`](#6-compilación-con-jit)
7. [Control de flujo dentro de `jit`](#7-control-de-flujo-dentro-de-jit)
8. [Vectorización automática con `vmap`](#8-vectorización-automática-con-vmap)
9. [Números aleatorios](#9-números-aleatorios)
10. [Pytrees: estructuras de parámetros](#10-pytrees-estructuras-de-parámetros)
11. [Precisión numérica y tipos](#11-precisión-numérica-y-tipos)
12. [Optimización y resolución numérica](#12-optimización-y-resolución-numérica)
13. [GPU, TPU y paralelismo](#13-gpu-tpu-y-paralelismo)
14. [Depuración y rendimiento](#14-depuración-y-rendimiento)
15. [Ecosistema](#15-ecosistema)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**JAX** es una librería para cálculo numérico de alto rendimiento que combina una API casi idéntica a NumPy con **transformaciones de funciones** componibles:

| Transformación | Qué hace |
|---|---|
| `grad` | Calcula derivadas (diferenciación automática). |
| `jit` | Compila la función con **XLA** para CPU, GPU o TPU. |
| `vmap` | Vectoriza automáticamente una función escrita para un solo elemento. |
| `shard_map` / *sharding* | Distribuye el cálculo entre varios dispositivos. |

Estas transformaciones se pueden anidar libremente: `jit(vmap(grad(f)))` es válido.

**Ventajas:**
- Derivadas exactas de cualquier orden de funciones Python/NumPy, sin derivar a mano.
- El mismo código se ejecuta en CPU, GPU y TPU sin cambios.
- Compilación XLA con fusión de operaciones: muy eficiente en cálculos con arrays grandes.
- Estilo funcional que facilita la reproducibilidad y la paralelización.
- Base de un ecosistema amplio: Flax, Optax, Equinox, NumPyro, BlackJAX.

**Limitaciones:**
- Exige un estilo **funcional**: arrays inmutables, funciones puras y sin efectos secundarios.
- El control de flujo dependiente de los datos dentro de `jit` requiere primitivas específicas (`lax.cond`, `lax.scan`…).
- Coste de compilación inicial y recompilación si cambian las formas de los arrays.
- Por defecto trabaja en **32 bits** (`float32`).
- El soporte GPU oficial se limita a Linux (en Windows, mediante WSL2).

| Si necesita… | Opción |
|---|---|
| Derivadas, optimización y cálculo científico diferenciable | **JAX** |
| Acelerar bucles Python sobre arrays en CPU | Numba |
| Redes neuronales con ecosistema amplio y estilo imperativo | PyTorch |
| Cálculo matricial sencillo sin derivadas | NumPy |
| Matemáticas simbólicas (derivadas exactas como expresiones) | SymPy |

---

## 2. Instalación e importación

```bash
pip install -U jax                    # Solo CPU (Windows, macOS, Linux)
pip install -U "jax[cuda12]"          # GPU NVIDIA con CUDA 12 (Linux)
pip install -U "jax[tpu]"             # TPU de Google Cloud
```

Comprobar la instalación y el dispositivo disponible:

```python
import jax
print(jax.__version__)
print(jax.devices())                  # [CpuDevice(id=0)] o [CudaDevice(id=0), ...]
print(jax.default_backend())          # 'cpu', 'gpu' o 'tpu'
```

Importaciones habituales:

```python
import jax
import jax.numpy as jnp
from jax import jit, grad, vmap, value_and_grad, lax, random
from functools import partial
```

> Por convención, `jnp` es el NumPy de JAX y `np` el NumPy estándar. Ambos pueden convivir: use `np` para la preparación de datos en Python y `jnp` dentro de las funciones que se transforman.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `jax.Array` | Array de JAX. Similar a `np.ndarray`, pero **inmutable** y ubicado en un dispositivo (CPU/GPU/TPU). |
| Función pura | Su resultado depende solo de sus argumentos y no modifica nada externo. Requisito para las transformaciones. |
| Trazado (*tracing*) | Para transformar una función, JAX la ejecuta con valores abstractos (*tracers*) que registran las operaciones. |
| Jaxpr | Representación intermedia de la función trazada. Se consulta con `jax.make_jaxpr`. |
| XLA | Compilador que genera código optimizado para el dispositivo. |
| Clave aleatoria | Estado explícito del generador de números aleatorios (`random.key`). |
| Pytree | Estructura anidada de listas, tuplas, diccionarios y arrays que JAX recorre automáticamente. |
| Valor estático | Argumento de `jit` que se trata como constante de compilación (`static_argnames`). |
| Ejecución asíncrona | Las operaciones devuelven el control antes de terminar; `block_until_ready()` espera al resultado. |

**Ejemplo de trazado:**

```python
def f(x):
    return jnp.sin(x) * x ** 2

print(jax.make_jaxpr(f)(1.0))
# { lambda ; a:f32[]. let b:f32[] = sin a; c:f32[] = integer_pow[y=2] a; d:f32[] = mul b c in (d,) }
```

---

## 4. `jax.numpy`: arrays inmutables

### 4.1 Uso similar a NumPy

```python
x = jnp.arange(10.0)
m = jnp.linspace(0, 1, 6).reshape(2, 3)
y = jnp.exp(-x) * jnp.sin(x)
total = jnp.sum(y); media = jnp.mean(y); maximo = jnp.max(y)
producto = m @ m.T
inversa = jnp.linalg.inv(producto + jnp.eye(2))
filtrado = jnp.where(x > 5, x, 0.0)
```

La gran mayoría de funciones de `numpy` y muchas de `scipy` (`jax.scipy.stats`, `jax.scipy.special`, `jax.scipy.linalg`) tienen equivalente.

### 4.2 Inmutabilidad: la sintaxis `.at[]`

Los arrays de JAX **no se pueden modificar**. En lugar de asignar, se crea un array nuevo:

```python
x = jnp.zeros(5)
x[0] = 1.0                         # ✗ TypeError: JAX arrays are immutable

x = x.at[0].set(1.0)               # ✓ Devuelve un array nuevo
x = x.at[1:3].add(2.0)             # Sumar
x = x.at[4].multiply(3.0)          # Multiplicar
x = x.at[jnp.array([0, 2])].max(5.0)
```

> Dentro de `jit`, XLA optimiza estas operaciones para que se realicen en el sitio cuando es posible: no hay copias reales.

### 4.3 Diferencias con NumPy

| Situación | NumPy | JAX |
|---|---|---|
| Asignación por índice | `a[0] = 1` | `a = a.at[0].set(1)` |
| Tipo decimal por defecto | `float64` | `float32` |
| Índice fuera de rango | `IndexError` | Se recorta al límite **sin error** |
| Números aleatorios | Estado global | Clave explícita |
| Ejecución | Síncrona | Asíncrona |
| Conversión a NumPy | — | `np.asarray(a)` o `jax.device_get(a)` |

---

## 5. Diferenciación automática

### 5.1 `grad`: gradiente de una función escalar

```python
def f(x):
    return jnp.sin(x) * x ** 2

df = grad(f)                  # Nueva función: f'(x)
df(1.0)                       # 2.2232442
d2f = grad(grad(f))           # Segunda derivada
```

Con varios argumentos, `argnums` indica respecto a cuáles derivar:

```python
def perdida(w, b, x, y):
    return jnp.mean((x @ w + b - y) ** 2)

grad(perdida)(w, b, x, y)                        # Respecto a w (argnums=0)
dw, db = grad(perdida, argnums=(0, 1))(w, b, x, y)
```

### 5.2 Valor y gradiente a la vez

```python
valor, gradiente = value_and_grad(perdida)(w, b, x, y)
```

Para devolver información adicional que no se deriva, use `has_aux`:

```python
def perdida_con_info(w, x, y):
    pred = x @ w
    return jnp.mean((pred - y) ** 2), {"predicciones": pred}

(valor, info), g = value_and_grad(perdida_con_info, has_aux=True)(w, x, y)
```

### 5.3 Jacobianas y hessianas

```python
jax.jacfwd(f)(x)          # Jacobiana en modo directo (eficiente con pocas entradas)
jax.jacrev(f)(x)          # Jacobiana en modo inverso (eficiente con pocas salidas)
jax.hessian(f)(x)         # Hessiana
jax.jvp(f, (x,), (v,))    # Producto jacobiana-vector
jax.vjp(f, x)             # Producto vector-jacobiana
```

| Situación | Herramienta |
|---|---|
| Función escalar, muchos parámetros (pérdidas) | `grad` (modo inverso) |
| Muchas salidas, pocas entradas | `jacfwd` |
| Pocas salidas, muchas entradas | `jacrev` |
| Matriz de segundas derivadas | `hessian` (= `jacfwd(jacrev(f))`) |

### 5.4 Detener el gradiente

```python
objetivo = jax.lax.stop_gradient(prediccion_red_objetivo)   # Se trata como constante al derivar
```

### 5.5 Derivadas personalizadas

Útil para estabilidad numérica o para funciones con derivada conocida:

```python
@jax.custom_jvp
def log1pexp(x):
    return jnp.log1p(jnp.exp(x))

@log1pexp.defjvp
def _log1pexp_jvp(primals, tangents):
    (x,), (t,) = primals, tangents
    return log1pexp(x), t * jax.nn.sigmoid(x)       # Derivada estable

grad(log1pexp)(100.0)          # 1.0 (sin la regla personalizada daría nan)
```

> `grad` solo funciona con salidas **escalares** y entradas de tipo decimal (`float`/`complex`). Derivar respecto a enteros produce un error.

---

## 6. Compilación con `jit`

### 6.1 Uso básico

```python
@jit
def normalizar(x):
    return (x - x.mean()) / x.std()

x = random.normal(random.key(0), (1_000_000,))
normalizar(x)                         # 1.ª llamada: traza y compila
normalizar(x).block_until_ready()     # Siguientes: ejecuta el código compilado
```

`jit` fusiona las operaciones y evita crear arrays intermedios. La ganancia es mayor cuantas más operaciones contenga la función.

### 6.2 Cuándo se recompila

JAX compila una versión por cada combinación de **forma (*shape*) y tipo** de los argumentos:

```python
normalizar(jnp.ones(100))     # Compila para f32[100]
normalizar(jnp.ones(100))     # Reutiliza
normalizar(jnp.ones(200))     # Recompila para f32[200]
```

> Si las formas cambian constantemente (p. ej. lotes de tamaño variable), rellene los arrays hasta tamaños fijos (*padding*) para evitar recompilaciones continuas.

### 6.3 Argumentos estáticos

Los valores que determinan la estructura del cálculo (tamaños, número de pasos, opciones) deben declararse estáticos:

```python
@partial(jit, static_argnames=("n_pasos", "metodo"))
def simular(x0, n_pasos, metodo="euler"):
    ...

simular(x0, n_pasos=100)            # Cambiar n_pasos provoca recompilación
```

Los argumentos estáticos deben ser **hashables** (enteros, cadenas, tuplas; no arrays ni listas).

### 6.4 Efectos secundarios

Durante el trazado, el código Python se ejecuta **una sola vez**. Los efectos secundarios no se repiten:

```python
@jit
def f(x):
    print("Trazando", x)              # Solo se muestra al compilar: Traced<ShapedArray(float32[])>
    jax.debug.print("x = {}", x)      # ✓ Se muestra en cada ejecución con el valor real
    return x * 2
```

> No modifique variables globales ni listas externas dentro de funciones transformadas: los cambios se pierden o se aplican solo durante el trazado.

---

## 7. Control de flujo dentro de `jit`

### 7.1 El problema

Dentro de `jit`, los valores son abstractos: Python no puede decidir un `if` en función de ellos.

```python
@jit
def valor_absoluto(x):
    if x > 0:                 # ✗ TracerBoolConversionError
        return x
    return -x
```

### 7.2 Soluciones

| Necesidad | Primitiva |
|---|---|
| Selección elemento a elemento | `jnp.where(cond, a, b)` |
| `if`/`else` con dos ramas | `lax.cond(pred, f_verdadero, f_falso, operando)` |
| Varias ramas (`switch`) | `lax.switch(indice, [f0, f1, f2], operando)` |
| Bucle con número fijo de iteraciones | `lax.fori_loop(inicio, fin, cuerpo, valor_inicial)` |
| Bucle con acumulación de resultados | `lax.scan(f, carry, xs)` |
| Bucle con condición de parada | `lax.while_loop(cond, cuerpo, valor_inicial)` |

```python
jnp.where(x > 0, x, -x)

lax.cond(x > 0, lambda v: v, lambda v: -v, x)

lax.fori_loop(0, 10, lambda i, acum: acum + i, 0)              # 45
```

### 7.3 `lax.scan`: bucles con estado

Es la primitiva más útil para simulaciones, series temporales y entrenamiento:

```python
def paso(saldo, movimiento):              # (estado, elemento) -> (nuevo_estado, salida)
    nuevo = saldo * 1.001 + movimiento
    return nuevo, nuevo

movimientos = jnp.array([100.0, -50.0, 200.0, -30.0])
saldo_final, historial = lax.scan(paso, 1000.0, movimientos)
```

### 7.4 `lax.while_loop`: iterar hasta converger

```python
def newton(f, x0, tol=1e-6, max_iter=50):
    df = grad(f)
    def condicion(estado):
        x, i = estado
        return (jnp.abs(f(x)) > tol) & (i < max_iter)
    def cuerpo(estado):
        x, i = estado
        return x - f(x) / df(x), i + 1
    return lax.while_loop(condicion, cuerpo, (x0, 0))

raiz, iteraciones = jit(lambda x0: newton(lambda x: x**3 - 2*x - 5, x0))(2.0)   # 2.0945513
```

> `while_loop` no admite diferenciación en modo inverso (`grad`). Si necesita derivar a través del bucle, use `scan` o `fori_loop` con un número fijo de iteraciones.

### 7.5 Los bucles Python normales

Un bucle `for` de Python con un número de iteraciones **conocido** funciona dentro de `jit`, pero se «desenrolla»: 1000 iteraciones generan 1000 copias del cuerpo y una compilación muy lenta. Use `lax.scan` o `lax.fori_loop`.

---

## 8. Vectorización automática con `vmap`

### 8.1 Concepto

Escriba la función para **un solo ejemplo**; `vmap` la convierte en una función que procesa un lote completo, sin bucles:

```python
def prediccion(w, x):                 # x: un único cliente (vector)
    return jnp.dot(w, x)

w = jnp.ones(3)
lote = jnp.ones((1000, 3))

prediccion_lote = vmap(prediccion, in_axes=(None, 0))   # w fijo, x recorre el eje 0
prediccion_lote(w, lote)                                # Forma (1000,)
```

### 8.2 `in_axes` y `out_axes`

| Valor | Significado |
|---|---|
| `0` | Recorrer el primer eje de ese argumento |
| `1` | Recorrer el segundo eje |
| `None` | Argumento compartido (no se vectoriza) |

```python
vmap(f, in_axes=(0, None, 1), out_axes=0)
```

### 8.3 Casos de uso típicos

```python
# Gradiente por ejemplo (en lugar del gradiente medio)
gradientes = vmap(grad(perdida_ejemplo), in_axes=(None, 0, 0))(params, xs, ys)

# Evaluar un modelo con muchos conjuntos de parámetros
resultados = vmap(modelo, in_axes=(0, None))(params_apilados, datos)

# Matriz de distancias por pares
dist = vmap(vmap(lambda a, b: jnp.linalg.norm(a - b), (None, 0)), (0, None))(puntos, puntos)
```

> `vmap` se combina con `jit`: `jit(vmap(f))` compila una única función vectorizada.

---

## 9. Números aleatorios

JAX no usa un estado global: cada llamada recibe una **clave** explícita. Esto garantiza reproducibilidad y paralelización segura.

```python
clave = random.key(42)                     # Clave inicial (semilla)

clave, subclave = random.split(clave)      # Dividir antes de cada uso
x = random.normal(subclave, (3, 2))

claves = random.split(clave, 10)           # 10 claves independientes
```

Distribuciones habituales:

```python
random.uniform(k, (n,), minval=0, maxval=1)
random.normal(k, (n,))
random.randint(k, (n,), 0, 10)
random.bernoulli(k, p=0.3, shape=(n,))
random.choice(k, a, shape=(5,), replace=False)
random.permutation(k, n)
random.multivariate_normal(k, media, covarianza, (n,))
```

> **Regla fundamental:** nunca reutilice una misma clave para dos sorteos distintos: obtendría números idénticos o correlacionados. Divida siempre con `random.split`.

> `random.key()` es la API moderna; `random.PRNGKey()` es la heredada y sigue funcionando.

---

## 10. Pytrees: estructuras de parámetros

Las transformaciones de JAX aceptan estructuras anidadas (diccionarios, listas, tuplas, `NamedTuple`, *dataclasses* registradas) cuyas hojas son arrays:

```python
params = {
    "capa1": {"w": jnp.ones((3, 4)), "b": jnp.zeros(4)},
    "capa2": {"w": jnp.ones((4, 1)), "b": jnp.zeros(1)},
}

grads = grad(perdida)(params, x, y)          # Mismo formato que params
params = jax.tree.map(lambda p, g: p - 0.01 * g, params, grads)   # Descenso de gradiente

jax.tree.leaves(params)                      # Lista de arrays
jax.tree.structure(params)                   # Estructura
```

Clases propias como pytrees:

```python
from dataclasses import dataclass

@jax.tree_util.register_dataclass
@dataclass
class Parametros:
    w: jax.Array
    b: jax.Array

grad(lambda p: (p.w * 2).sum() + p.b)(Parametros(jnp.ones(2), jnp.array(1.0)))
```

> Las `NamedTuple` son pytrees automáticamente y resultan una opción sencilla y legible.

---

## 11. Precisión numérica y tipos

### 11.1 Activar 64 bits

```python
import jax
jax.config.update("jax_enable_x64", True)    # Al inicio del programa, antes de crear arrays
```

O mediante variable de entorno: `JAX_ENABLE_X64=1`.

| Uso | Precisión recomendada |
|---|---|
| Aprendizaje automático, redes neuronales | `float32` (o `bfloat16` en GPU/TPU) |
| Cálculo científico, finanzas, métodos iterativos sensibles | `float64` |

> En GPU de consumo, `float64` puede ser 10–30 veces más lento que `float32`.

### 11.2 Tipos débiles (*weak types*)

Los escalares de Python (`2.0`) no fuerzan el tipo del resultado: `jnp.ones(3, dtype=jnp.float16) * 2.0` sigue siendo `float16`. Por eso algunos resultados muestran `weak_type=True`.

---

## 12. Optimización y resolución numérica

### 12.1 Descenso de gradiente manual

```python
@jit
def paso(params, x, y, tasa=0.01):
    valor, g = value_and_grad(perdida)(params, x, y)
    return jax.tree.map(lambda p, gi: p - tasa * gi, params, g), valor

for i in range(1000):
    params, valor = paso(params, x, y)
```

### 12.2 Con Optax (optimizadores profesionales)

```python
# pip install optax
import optax

optimizador = optax.adam(learning_rate=1e-3)
estado = optimizador.init(params)

@jit
def paso(params, estado, x, y):
    valor, g = value_and_grad(perdida)(params, x, y)
    actualizaciones, estado = optimizador.update(g, estado, params)
    return optax.apply_updates(params, actualizaciones), estado, valor
```

### 12.3 Minimización con `jax.scipy.optimize`

```python
from jax.scipy.optimize import minimize

resultado = minimize(lambda p: jnp.sum((p - jnp.array([1.0, 2.0])) ** 2),
                     jnp.zeros(2), method="BFGS")
resultado.x, resultado.success              # [1., 2.], True
```

> Para problemas más completos (mínimos cuadrados, restricciones, ecuaciones), consulte **Optimistix** y **JAXopt**.

### 12.4 Álgebra lineal y estadística

```python
jnp.linalg.solve(A, b); jnp.linalg.eigh(S); jnp.linalg.cholesky(S); jnp.linalg.svd(M)
jax.scipy.stats.norm.cdf(x); jax.scipy.stats.norm.logpdf(x, loc, scale)
jax.scipy.special.logsumexp(x); jax.nn.softmax(x); jax.nn.sigmoid(x)
```

---

## 13. GPU, TPU y paralelismo

### 13.1 Colocación en dispositivos

JAX coloca los arrays en el dispositivo por defecto (la GPU si está disponible). No es necesario mover los datos manualmente en la mayoría de casos.

```python
x = jnp.ones(1000)                       # En el dispositivo por defecto
x.devices()                              # {CudaDevice(id=0)}
x_cpu = jax.device_put(x, jax.devices("cpu")[0])
x_np = np.asarray(x)                     # Copia a la memoria del host como NumPy
```

### 13.2 Ejecución asíncrona

```python
resultado = f(x)                     # Devuelve inmediatamente
resultado.block_until_ready()        # Espera a que termine (necesario para medir tiempos)
```

### 13.3 Varios dispositivos (*sharding*)

```python
from jax.sharding import Mesh, NamedSharding, PartitionSpec as P

malla = jax.make_mesh((jax.device_count(),), ("datos",))
reparto = NamedSharding(malla, P("datos"))
x = jax.device_put(jnp.ones((8 * jax.device_count(), 1024)), reparto)   # Filas repartidas
w = jnp.ones((1024, 64))                                                # Pesos replicados

@jit
def f(x, w):
    return jnp.tanh(x) @ w            # Cada dispositivo procesa sus filas; JAX coordina el resto
```

> Para simular varios dispositivos en CPU durante el desarrollo: `XLA_FLAGS=--xla_force_host_platform_device_count=8`.

---

## 14. Depuración y rendimiento

### 14.1 Herramientas de depuración

```python
jax.debug.print("x = {x}, media = {m}", x=x, m=x.mean())   # print dentro de jit/vmap
jax.debug.breakpoint()                                     # Punto de interrupción dentro de jit

jax.config.update("jax_disable_jit", True)   # Ejecutar sin compilar (depurar con pdb)
jax.config.update("jax_debug_nans", True)    # Error en cuanto aparezca un NaN
jax.make_jaxpr(f)(x)                         # Ver las operaciones trazadas
```

### 14.2 Medir tiempos correctamente

```python
import time

f_jit = jit(f)
f_jit(x).block_until_ready()                 # Compilar
inicio = time.perf_counter()
f_jit(x).block_until_ready()                 # Medir sin compilación y esperando al resultado
print(time.perf_counter() - inicio)
```

En Jupyter: `%timeit f_jit(x).block_until_ready()`.

### 14.3 Caché de compilación persistente

```python
jax.config.update("jax_compilation_cache_dir", "/ruta/cache_jax")
```

### 14.4 Consejos de rendimiento

- Aplique `jit` a la función **más externa** posible (el paso completo de entrenamiento o simulación), no a operaciones pequeñas.
- Evite transferir datos entre dispositivo y Python dentro de bucles (`float(x)`, `np.asarray(x)`, `print(x)` fuerzan sincronización).
- Mantenga formas de array constantes.
- Sustituya bucles Python por `lax.scan`, `fori_loop` o `vmap`.

---

## 15. Ecosistema

| Librería | Uso |
|---|---|
| **Optax** | Optimizadores (Adam, SGD, programación de la tasa de aprendizaje, recorte de gradientes). |
| **Flax (NNX)** | Redes neuronales (Google). |
| **Equinox** | Redes neuronales y modelos como pytrees, estilo minimalista. |
| **Optimistix / Lineax / Diffrax** | Optimización, sistemas lineales y ecuaciones diferenciales. |
| **NumPyro / BlackJAX** | Programación probabilística e inferencia bayesiana (MCMC). |
| **Orbax** | Guardado y carga de puntos de control (*checkpoints*). |
| **Chex** | Utilidades de pruebas y aserciones para código JAX. |

---

## 16. Ejemplo práctico completo

**Modelo de riesgo de impago (*credit scoring*)**: genera una cartera sintética de 20 000 clientes, entrena una regresión logística con regularización L2 y el optimizador Adam implementado desde cero, **entrena 9 modelos con distintos niveles de regularización a la vez** mediante `vmap`, elige el mejor con un conjunto de validación, evalúa con AUC y exactitud, y usa `grad` para obtener la sensibilidad de la probabilidad de impago de un cliente a cada variable. Combina `jit`, `vmap`, `grad`, `lax.scan`, claves aleatorias y pytrees (`NamedTuple`).

Requisitos: `pip install jax`

```python
"""
Modelo de riesgo de impago con JAX.

Entrena una regresión logística con regularización L2 mediante el optimizador
Adam (implementado desde cero), selecciona el mejor nivel de regularización
entrenando todos los candidatos a la vez con vmap y evalúa el modelo.
"""
import time
from functools import partial
from typing import NamedTuple

import jax
import jax.numpy as jnp
from jax import random

VARIABLES = ("ingresos", "deuda_ingresos", "antigüedad", "retrasos", "edad")
PESOS_REALES = jnp.array([-1.2, 1.8, -0.6, 1.4, -0.3])
SESGO_REAL = -1.0
N_CLIENTES = 20_000
LAMBDAS = jnp.logspace(-4, 0, 9)                 # Candidatos de regularización L2
PASOS = 1_500
TASA = 0.05


# ------------------------------------------------------------------ Datos sintéticos
def generar_datos(clave, n):
    k_x, k_y = random.split(clave)
    x = random.normal(k_x, (n, len(VARIABLES)))
    prob = jax.nn.sigmoid(x @ PESOS_REALES + SESGO_REAL)
    y = random.bernoulli(k_y, prob).astype(jnp.float32)
    return x, y


# ------------------------------------------------------------------ Modelo
class Parametros(NamedTuple):            # Un NamedTuple es un pytree: grad/jit lo recorren solos
    w: jax.Array
    b: jax.Array


def predecir(params: Parametros, x):
    return jax.nn.sigmoid(x @ params.w + params.b)


def perdida(params: Parametros, x, y, lam):
    logits = x @ params.w + params.b
    entropia = jnp.mean(jnp.logaddexp(0.0, logits) - y * logits)   # Entropía cruzada estable
    return entropia + lam * jnp.sum(params.w ** 2)


# ------------------------------------------------------------------ Optimizador Adam
class EstadoAdam(NamedTuple):
    m: Parametros
    v: Parametros
    t: jax.Array


def adam_iniciar(params):
    ceros = jax.tree.map(jnp.zeros_like, params)
    return EstadoAdam(m=ceros, v=ceros, t=jnp.array(0))


def adam_paso(params, grads, estado, tasa, b1=0.9, b2=0.999, eps=1e-8):
    t = estado.t + 1
    m = jax.tree.map(lambda m, g: b1 * m + (1 - b1) * g, estado.m, grads)
    v = jax.tree.map(lambda v, g: b2 * v + (1 - b2) * g ** 2, estado.v, grads)
    params = jax.tree.map(
        lambda p, m, v: p - tasa * (m / (1 - b1 ** t)) / (jnp.sqrt(v / (1 - b2 ** t)) + eps),
        params, m, v)
    return params, EstadoAdam(m, v, t)


# ------------------------------------------------------------------ Entrenamiento
@partial(jax.jit, static_argnames="pasos")
def entrenar(x, y, lam, pasos=PASOS):
    params = Parametros(w=jnp.zeros(x.shape[1]), b=jnp.array(0.0))

    def paso(carry, _):
        params, estado = carry
        valor, grads = jax.value_and_grad(perdida)(params, x, y, lam)
        params, estado = adam_paso(params, grads, estado, TASA)
        return (params, estado), valor

    (params, _), historial = jax.lax.scan(paso, (params, adam_iniciar(params)), None, length=pasos)
    return params, historial


# Entrena un modelo por cada valor de lambda, en paralelo (vectorizado)
entrenar_varios = jax.jit(jax.vmap(entrenar, in_axes=(None, None, 0)))


# ------------------------------------------------------------------ Métricas
@jax.jit
def auc(prob, y):
    """Área bajo la curva ROC (estadístico de Mann-Whitney)."""
    rangos = jnp.argsort(jnp.argsort(prob)) + 1
    n_pos = y.sum()
    n_neg = y.shape[0] - n_pos
    return (jnp.sum(rangos * y) - n_pos * (n_pos + 1) / 2) / (n_pos * n_neg)


@jax.jit
def exactitud(prob, y, umbral=0.5):
    return jnp.mean((prob >= umbral) == (y == 1))


def main() -> None:
    print(f"Dispositivo: {jax.devices()[0].platform.upper()} · JAX {jax.__version__}")
    clave = random.key(2026)
    k_ent, k_val, k_test = random.split(clave, 3)
    x_ent, y_ent = generar_datos(k_ent, N_CLIENTES)
    x_val, y_val = generar_datos(k_val, N_CLIENTES // 4)
    x_test, y_test = generar_datos(k_test, N_CLIENTES // 4)
    print(f"Clientes de entrenamiento: {N_CLIENTES:,} · tasa de impago {y_ent.mean():.1%}\n")

    # 1. Entrenar todos los candidatos a la vez
    inicio = time.perf_counter()
    params_todos, historiales = entrenar_varios(x_ent, y_ent, LAMBDAS)
    jax.block_until_ready(params_todos)
    print(f"{len(LAMBDAS)} modelos × {PASOS} pasos entrenados en {time.perf_counter() - inicio:.2f} s "
          "(incluye compilación)")

    # 2. Elegir lambda con el conjunto de validación (vmap sobre los parámetros)
    prob_val = jax.vmap(predecir, in_axes=(0, None))(params_todos, x_val)
    auc_val = jax.vmap(auc, in_axes=(0, None))(prob_val, y_val)
    print(f"\n{'lambda':>10}{'pérdida final':>16}{'AUC validación':>17}")
    for lam, hist, a in zip(LAMBDAS, historiales, auc_val):
        print(f"{float(lam):>10.4f}{float(hist[-1]):>16.4f}{float(a):>17.4f}")

    mejor = int(jnp.argmax(auc_val))
    params = jax.tree.map(lambda p: p[mejor], params_todos)
    print(f"\nMejor lambda: {float(LAMBDAS[mejor]):.4f}")

    # 3. Evaluación final en el conjunto de prueba
    prob_test = predecir(params, x_test)
    print(f"AUC prueba: {float(auc(prob_test, y_test)):.4f} · "
          f"exactitud: {float(exactitud(prob_test, y_test)):.1%}")

    print(f"\n{'Variable':<16}{'Peso real':>11}{'Estimado':>11}")
    for nombre, real, est in zip(VARIABLES, PESOS_REALES, params.w):
        print(f"{nombre:<16}{float(real):>11.2f}{float(est):>11.2f}")
    print(f"{'(sesgo)':<16}{SESGO_REAL:>11.2f}{float(params.b):>11.2f}")

    # 4. Sensibilidad de la probabilidad de impago de un cliente a cada variable
    cliente = x_test[jnp.argmin(jnp.abs(prob_test - 0.5))]      # Cliente en la zona de duda
    sensibilidad = jax.grad(lambda c: predecir(params, c))(cliente)
    print(f"\nCliente de ejemplo · probabilidad de impago {float(predecir(params, cliente)):.1%}")
    for nombre, s in zip(VARIABLES, sensibilidad):
        print(f"  ∂P/∂{nombre:<15}{float(s):+.4f}")


if __name__ == "__main__":
    main()
```

Resultado orientativo (CPU):

```
Dispositivo: CPU · JAX 0.12.1
Clientes de entrenamiento: 20,000 · tasa de impago 37.5%

9 modelos × 1500 pasos entrenados en 1.21 s (incluye compilación)

    lambda   pérdida final   AUC validación
    0.0001          0.3720           0.9010
    0.0003          0.3736           0.9010
    ...
    1.0000          0.6391           0.9007

Mejor lambda: 0.0003
AUC prueba: 0.9043 · exactitud: 83.0%

Variable          Peso real   Estimado
ingresos              -1.20      -1.20
deuda_ingresos         1.80       1.77
antigüedad            -0.60      -0.61
retrasos               1.40       1.36
edad                  -0.30      -0.28
(sesgo)               -1.00      -1.01

Cliente de ejemplo · probabilidad de impago 50.0%
  ∂P/∂ingresos       -0.2995
  ∂P/∂deuda_ingresos +0.4423
  ∂P/∂antigüedad     -0.1526
  ∂P/∂retrasos       +0.3410
  ∂P/∂edad           -0.0704
```

> El modelo recupera los pesos reales con gran precisión. Con muchos datos y pocas variables la regularización apenas afecta a la AUC; su efecto es mayor con pocos clientes o muchas variables (véase el ejercicio 5).

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TypeError: JAX arrays are immutable` | Asignación por índice (`x[0] = 1`). | Use `x = x.at[0].set(1)`. |
| `TracerBoolConversionError` | `if`/`while` de Python sobre un valor trazado dentro de `jit`. | Use `jnp.where`, `lax.cond` o `lax.while_loop`; o declare el argumento estático. |
| `ConcretizationTypeError` | Se necesita un valor concreto (p. ej. una forma) a partir de un valor trazado. | Declare el argumento con `static_argnames` o reestructure el cálculo. |
| `NonConcreteBooleanIndexError` | Indexado con máscara booleana (`x[x > 0]`) dentro de `jit`: forma variable. | Use `jnp.where(x > 0, x, 0)` o `jnp.sum(x, where=x > 0)`. |
| `TypeError: grad requires real- or complex-valued inputs` | Se deriva respecto a un entero. | Convierta a decimal: `jnp.array(3.0)`. |
| `TypeError: Gradient only defined for scalar-output functions` | `grad` sobre una función que devuelve un array. | Reduzca a escalar (`sum`, `mean`) o use `jacrev`/`vmap(grad(...))`. |
| Números aleatorios idénticos en cada llamada | Se reutiliza la misma clave. | Divida la clave con `random.split` en cada uso. |
| Resultados ligeramente distintos a NumPy | JAX usa `float32` por defecto. | Active `jax_enable_x64` si necesita doble precisión. |
| La función se vuelve muy lenta tras unas llamadas | Recompilación por cambios de forma o de argumentos estáticos. | Use formas fijas (*padding*) y limite los valores estáticos distintos. |
| Compilación extremadamente lenta | Bucle `for` de Python largo dentro de `jit` (se desenrolla). | Sustitúyalo por `lax.scan` o `lax.fori_loop`. |
| Tiempos medidos irreales (demasiado rápidos) | Ejecución asíncrona. | Llame a `.block_until_ready()` antes de parar el cronómetro. |
| `print` dentro de `jit` muestra `Traced<ShapedArray...>` | `print` se ejecuta solo durante el trazado. | Use `jax.debug.print`. |
| Índices fuera de rango devuelven valores sin error | JAX recorta los índices al límite. | Valide los índices antes o use `mode="fill"` en `.at[].get()`. |
| Aparecen `NaN` en los gradientes | Operaciones como `sqrt(0)`, `log(0)` o `0/0` en la derivada. | Active `jax_debug_nans`; use `jnp.where` seguro o `custom_jvp`. |
| `CUDA backend failed to initialize` | Instalación GPU incorrecta o drivers incompatibles. | Reinstale con `pip install -U "jax[cuda12]"` y actualice el driver de NVIDIA. |

---

## 18. Buenas prácticas

- Escriba **funciones puras**: todo lo que la función necesita entra por argumentos y todo lo que produce sale por `return`.
- Aplique `jit` al **nivel más alto** posible (el paso completo) y no a funciones diminutas.
- Use `vmap` en lugar de bucles sobre lotes; escriba el código pensando en un único ejemplo.
- Gestione la aleatoriedad **pasando claves** de forma explícita y dividiéndolas siempre.
- Organice los parámetros como **pytrees** (`NamedTuple`, diccionarios o *dataclasses* registradas).
- Mantenga constantes las **formas** de los arrays dentro de los bucles de entrenamiento o simulación.
- Active `float64` en cálculo científico y financiero; mantenga `float32` en aprendizaje automático.
- Prepare y limpie los datos con NumPy/pandas **fuera** de JAX y convierta a `jnp` solo para el cálculo.
- Mida siempre con `block_until_ready()` y separando la compilación.
- Use librerías del ecosistema (Optax, Flax, Equinox) en lugar de reimplementar optimizadores y capas en proyectos reales.
- Verifique los gradientes con `jax.test_util.check_grads` o comparando con diferencias finitas.

---

## 19. Referencia rápida

```python
import jax, jax.numpy as jnp
from jax import jit, grad, vmap, value_and_grad, lax, random

jnp.array / zeros / ones / arange / linspace / where / sum / mean / linalg.*   # Como NumPy
x.at[i].set(v) / .add(v) / .multiply(v) / .min(v) / .max(v)                   # «Modificar» arrays

grad(f) / grad(f, argnums=(0, 1)) / value_and_grad(f, has_aux=True)           # Derivadas
jax.jacfwd / jax.jacrev / jax.hessian / jax.jvp / jax.vjp / lax.stop_gradient
jit(f) / partial(jit, static_argnames=("n",))                                 # Compilar
vmap(f, in_axes=(None, 0), out_axes=0)                                        # Vectorizar

lax.cond(p, f_v, f_f, x) / lax.switch(i, [f0, f1], x)                         # Condicionales
lax.fori_loop(a, b, cuerpo, init) / lax.scan(f, carry, xs) / lax.while_loop(c, cuerpo, init)

k = random.key(0); k, sk = random.split(k); random.normal(sk, (n,))           # Aleatoriedad
jax.tree.map(f, arbol) / jax.tree.leaves(arbol)                               # Pytrees

jax.config.update("jax_enable_x64", True) / ("jax_debug_nans", True) / ("jax_disable_jit", True)
jax.debug.print("{}", x) / jax.make_jaxpr(f)(x) / y.block_until_ready() / jax.devices()
```

---

## 20. Ejercicios propuestos

1. Calcule con `grad` las «griegas» (delta, gamma y vega) de una opción europea con la fórmula de Black-Scholes y compárelas con sus expresiones analíticas.
2. Implemente una regresión lineal con descenso de gradiente y compare el resultado con la solución cerrada `jnp.linalg.lstsq`.
3. Simule 100 000 trayectorias de un paseo aleatorio con `lax.scan` y `vmap`, y calcule la distribución del máximo alcanzado.
4. Entrene una red neuronal de dos capas para clasificar dígitos (conjunto `load_digits` de scikit-learn) usando Optax.
5. Repita el ejemplo completo con solo 300 clientes de entrenamiento y 20 variables de ruido adicionales, y analice cómo cambia la elección del parámetro de regularización.

---

## 21. Referencias

- Documentación oficial: https://docs.jax.dev/
- Guía rápida: https://docs.jax.dev/en/latest/quickstart.html
- Errores comunes («The Sharp Bits»): https://docs.jax.dev/en/latest/notebooks/Common_Gotchas_in_JAX.html
- Repositorio: https://github.com/jax-ml/jax
- Optax: https://optax.readthedocs.io/
