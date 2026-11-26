# Manual de uso: Numba

> **Categoría:** Computación numérica y rendimiento · **Utilidad:** Acelerar cálculos en Python
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** BSD

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primeros pasos con `@njit`](#4-primeros-pasos-con-njit)
5. [Opciones de compilación](#5-opciones-de-compilación)
6. [Tipos y firmas](#6-tipos-y-firmas)
7. [Qué admite Numba (Python y NumPy)](#7-qué-admite-numba-python-y-numpy)
8. [Paralelización con `prange`](#8-paralelización-con-prange)
9. [Funciones universales: `@vectorize` y `@guvectorize`](#9-funciones-universales-vectorize-y-guvectorize)
10. [Estructuras de datos: listas, diccionarios y clases](#10-estructuras-de-datos-listas-diccionarios-y-clases)
11. [Funciones como argumentos y extensión de Numba](#11-funciones-como-argumentos-y-extensión-de-numba)
12. [Integración con código externo](#12-integración-con-código-externo)
13. [Medir el rendimiento correctamente](#13-medir-el-rendimiento-correctamente)
14. [Depuración y diagnóstico](#14-depuración-y-diagnóstico)
15. [Computación en GPU](#15-computación-en-gpu)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Numba** es un compilador *just-in-time* (JIT) que traduce funciones de Python y NumPy a código máquina optimizado mediante **LLVM**. Basta con añadir un decorador a una función numérica para obtener un rendimiento comparable al de C o Fortran, sin abandonar Python.

**Ventajas:**
- Aceleraciones de 10× a 1000× en bucles numéricos, sin reescribir el código en otro lenguaje.
- Se integra con NumPy: trabaja directamente sobre arrays sin copias.
- Paralelización multinúcleo con un solo parámetro (`parallel=True`).
- Permite escribir funciones universales (*ufuncs*) propias con la misma eficiencia que las de NumPy.
- Caché en disco del código compilado.

**Limitaciones:**
- Solo compila un **subconjunto** de Python: tipos numéricos, arrays de NumPy, tuplas, cadenas y estructuras tipadas. No admite pandas, objetos arbitrarios ni la mayoría de librerías externas dentro de las funciones compiladas.
- La primera llamada tiene un coste de compilación (de décimas de segundo a varios segundos).
- Los mensajes de error de tipado pueden resultar extensos y poco intuitivos.
- No aporta mejoras si el código ya está bien vectorizado con NumPy y no hay bucles.

| Si necesita… | Opción |
|---|---|
| Acelerar bucles numéricos sobre arrays | **Numba** |
| Operaciones de arrays sin bucles | NumPy vectorizado |
| Diferenciación automática, GPU/TPU, aprendizaje automático | JAX / PyTorch |
| Extensiones compiladas distribuibles o interfaz con C/C++ | Cython / pybind11 |
| Procesamiento de datos tabulares | pandas / Polars |

---

## 2. Instalación e importación

```bash
pip install numba                  # Instala también llvmlite y, si falta, NumPy
conda install numba                # Alternativa con conda
```

Comprobar la instalación y el entorno:

```bash
python -c "import numba; print(numba.__version__)"
numba -s                           # Informe completo: CPU, hilos, soporte GPU, versiones
```

Importaciones habituales:

```python
import numpy as np
import numba as nb
from numba import njit, prange, vectorize, guvectorize, objmode, types
from numba.typed import List, Dict
from numba.experimental import jitclass
```

> Numba depende estrechamente de las versiones de Python y NumPy. Si actualiza alguno de ellos, actualice también Numba y consulte la tabla de compatibilidad de la documentación oficial.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Compilación JIT | La función se compila a código máquina en la **primera llamada**, según los tipos de los argumentos. |
| Modo *nopython* | Compilación completa sin intérprete de Python. Es el modo por defecto desde Numba 0.59 y el único que aporta rendimiento real. |
| `@njit` | Abreviatura de `@jit(nopython=True)`. Decorador principal. |
| Especialización | Numba genera una versión compilada distinta por cada combinación de tipos de entrada. |
| Firma (*signature*) | Descripción de los tipos de entrada y salida, p. ej. `"float64(float64[:])"`. |
| Inferencia de tipos | Numba deduce el tipo de cada variable; si no puede, lanza un `TypingError`. |
| `prange` | Versión paralela de `range` para repartir iteraciones entre núcleos. |
| *ufunc* | Función universal que se aplica elemento a elemento sobre arrays (`@vectorize`). |
| Caché | Guarda el código compilado en disco para evitar recompilar entre ejecuciones. |

**Flujo de trabajo:**

```
Código Python ─► Análisis de bytecode ─► Inferencia de tipos ─► LLVM IR ─► Código máquina
                                          (primera llamada con cada tipo)
```

---

## 4. Primeros pasos con `@njit`

### 4.1 Función mínima

```python
import numpy as np
from numba import njit

@njit
def suma_cuadrados(a):
    total = 0.0
    for x in a:
        total += x * x
    return total

datos = np.random.rand(10_000_000)
suma_cuadrados(datos)        # 1.ª llamada: compila y ejecuta
suma_cuadrados(datos)        # Siguientes: solo ejecuta (código máquina)
```

### 4.2 Cuándo merece la pena

Numba es especialmente eficaz cuando el código contiene:

- **Bucles** explícitos sobre arrays (sobre todo bucles anidados).
- Lógica condicional dentro de los bucles, difícil de vectorizar con NumPy.
- Algoritmos iterativos o con dependencias entre pasos (simulaciones, recursiones, métodos numéricos).
- Funciones pequeñas llamadas millones de veces.

```python
# Difícil de vectorizar: cada valor depende del anterior
@njit
def suavizado_exponencial(serie, alfa):
    resultado = np.empty_like(serie)
    resultado[0] = serie[0]
    for i in range(1, serie.shape[0]):
        resultado[i] = alfa * serie[i] + (1 - alfa) * resultado[i - 1]
    return resultado
```

### 4.3 Llamadas entre funciones compiladas

Una función `@njit` puede llamar a otras funciones `@njit`; Numba las compila y, a menudo, las integra (*inline*) en la llamante.

```python
@njit
def distancia(x1, y1, x2, y2):
    return np.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2)

@njit
def longitud_ruta(xs, ys):
    total = 0.0
    for i in range(xs.shape[0] - 1):
        total += distancia(xs[i], ys[i], xs[i + 1], ys[i + 1])
    return total
```

> Una función `@njit` **no puede** llamar a funciones de Python normales (no decoradas). Use `objmode` (sección 12.1) si es imprescindible.

### 4.4 Ejecutar la versión original

```python
suma_cuadrados.py_func(datos)       # Ejecuta la función Python sin compilar
```

Útil para comparar resultados y rendimiento.

---

## 5. Opciones de compilación

```python
@njit(cache=True, fastmath=True, parallel=True, nogil=True, boundscheck=False, error_model="numpy")
def funcion(...):
    ...
```

| Opción | Efecto | Cuándo usarla |
|---|---|---|
| `cache=True` | Guarda el código compilado en `__pycache__` y lo reutiliza entre ejecuciones. | Siempre en scripts y aplicaciones que se ejecutan a menudo. |
| `fastmath=True` | Permite optimizaciones matemáticas no estrictas (reordenar sumas, ignorar `NaN`/`inf`). | Cálculos donde una diferencia mínima de redondeo es aceptable. |
| `parallel=True` | Activa `prange` y la paralelización automática de operaciones de arrays. | Bucles con iteraciones independientes y mucho trabajo. |
| `nogil=True` | Libera el GIL durante la ejecución. | Ejecutar la función desde varios hilos de Python (`threading`). |
| `boundscheck=True` | Comprueba los índices de los arrays (lanza `IndexError`). | Depuración; por defecto está desactivado. |
| `error_model="numpy"` | La división por cero devuelve `inf`/`nan` en lugar de lanzar `ZeroDivisionError`. | Cálculos vectoriales donde se prefiere el comportamiento de NumPy. |
| `inline="always"` | Integra la función en el código de la llamante. | Funciones auxiliares muy pequeñas. |

> **Atención con `fastmath`:** puede alterar resultados que dependan de `NaN` o de la precisión exacta (p. ej. `np.isnan` puede dejar de funcionar como se espera). No lo use en código financiero o de validación sin comprobarlo.

### 5.1 Variables de entorno útiles

| Variable | Efecto |
|---|---|
| `NUMBA_DISABLE_JIT=1` | Desactiva la compilación: todas las funciones se ejecutan como Python normal (depuración). |
| `NUMBA_NUM_THREADS=4` | Número máximo de hilos para el código paralelo. |
| `NUMBA_CACHE_DIR=/ruta` | Carpeta de la caché (útil si el código está en una carpeta de solo lectura). |
| `NUMBA_THREADING_LAYER=omp` | Capa de hilos: `tbb`, `omp` o `workqueue`. |
| `NUMBA_OPT=3` | Nivel de optimización de LLVM (0 a 3). |

---

## 6. Tipos y firmas

### 6.1 Compilación diferida (por defecto)

Sin firma, Numba compila una versión por cada combinación de tipos con la que se llama a la función:

```python
@njit
def sumar(a, b):
    return a + b

sumar(1, 2)          # Compila para (int64, int64)
sumar(1.0, 2.5)      # Compila para (float64, float64)
print(sumar.signatures)
```

### 6.2 Compilación anticipada con firma

Con una firma explícita, la compilación se realiza al **definir** la función y solo se aceptan esos tipos:

```python
@njit("float64(float64[:])")
def media(a):
    return a.sum() / a.shape[0]

@njit(["int64(int64, int64)", "float64(float64, float64)"])   # Varias firmas
def maximo(a, b):
    return a if a > b else b
```

| Notación | Tipo |
|---|---|
| `int32`, `int64`, `float32`, `float64`, `boolean`, `complex128` | Escalares |
| `float64[:]` | Array 1D (cualquier disposición en memoria) |
| `float64[::1]` | Array 1D contiguo (C) — permite más optimizaciones |
| `float64[:, :]` / `float64[:, ::1]` | Array 2D / 2D contiguo |
| `types.unicode_type` | Cadena de texto |
| `types.UniTuple(float64, 3)` | Tupla homogénea de 3 elementos |
| `void` | Sin valor de retorno |

### 6.3 Consultar el tipo que asigna Numba

```python
nb.typeof(np.ones(3))        # array(float64, 1d, C)
nb.typeof(3.5)               # float64
nb.typeof((1, 2.0))          # Tuple(int64, float64)
```

### 6.4 Estabilidad de tipos

Una variable debe tener **un único tipo** en toda la función. Este código falla:

```python
@njit
def incorrecto(x):
    resultado = 0               # int64
    if x > 0:
        resultado = 1.5         # float64: Numba unifica a float64 (válido)
    else:
        resultado = "negativo"  # unicode: TypingError, no se puede unificar
    return resultado
```

> Inicialice las variables con el tipo definitivo (`0.0` en lugar de `0` para acumuladores decimales) y devuelva siempre el mismo tipo.

---

## 7. Qué admite Numba (Python y NumPy)

### 7.1 Python

| Admitido | No admitido (en modo *nopython*) |
|---|---|
| `int`, `float`, `complex`, `bool`, `str` | Objetos de clases propias (salvo `jitclass`) |
| `tuple`, `namedtuple` | `pandas.DataFrame`, `Series` |
| `list` y `dict` homogéneos (mejor `typed.List`/`typed.Dict`) | Listas o diccionarios con tipos mezclados |
| `if`, `for`, `while`, `break`, `continue`, `return` | `try`/`except` complejos (solo formas básicas) |
| `range`, `enumerate`, `zip`, `len`, `min`, `max`, `abs`, `sum` | Generadores con `send`, `async`/`await` |
| `math`, `cmath`, `random` (funciones básicas) | Llamadas a funciones Python no compiladas |
| `raise` con excepciones con argumentos constantes | Formato de cadenas avanzado (f-strings con formato) |
| Funciones anidadas y *closures* simples | `print` con formato complejo (sí admite `print` básico) |

### 7.2 NumPy

Numba admite la mayor parte de la API numérica de NumPy:

```python
@njit
def ejemplos_numpy(a, b):
    c = np.zeros((3, 3)); d = np.empty_like(a); e = np.arange(10); f = np.linspace(0, 1, 5)
    g = a + b * 2.0; h = np.sqrt(a); i = np.exp(-a); j = np.where(a > 0, a, 0.0)
    k = a.sum(); m = a.mean(); n = a.std(); o = np.max(a); p = np.argmax(a)
    q = np.sort(a); r = np.cumsum(a); s = np.dot(a, b); t = a[a > 0.5]
    u = np.linalg.inv(c + np.eye(3)); v = np.linalg.solve(c + np.eye(3), np.ones(3))
    w = np.random.normal(0.0, 1.0, size=10)
    return k
```

Limitaciones habituales:

- Muchas funciones solo aceptan los argumentos principales (p. ej. `a.sum(axis=...)` solo con `axis` entero).
- No se admite `np.random.default_rng()` (*Generator*): use las funciones heredadas `np.random.*` dentro de `@njit`.
- Las funciones de `np.linalg` requieren SciPy instalado (usan BLAS/LAPACK) y arrays contiguos de tipo `float` o `complex`.

> Ante la duda, consulte la lista completa: *Supported NumPy features* en la documentación oficial.

### 7.3 Números aleatorios

```python
@njit
def simular(n):
    np.random.seed(42)                 # La semilla debe fijarse DENTRO de la función compilada
    return np.random.standard_normal(n)
```

> El generador de Numba es independiente del de NumPy: `np.random.seed()` llamado desde Python **no** afecta a las funciones compiladas.

---

## 8. Paralelización con `prange`

### 8.1 Bucles paralelos

```python
from numba import njit, prange

@njit(parallel=True)
def media_por_fila(m):
    resultado = np.empty(m.shape[0])
    for i in prange(m.shape[0]):           # Las iteraciones se reparten entre los núcleos
        resultado[i] = m[i].mean()
    return resultado
```

### 8.2 Reducciones

Numba reconoce automáticamente las reducciones de suma, producto, mínimo y máximo sobre una variable escalar:

```python
@njit(parallel=True)
def suma_paralela(a):
    total = 0.0
    for i in prange(a.shape[0]):
        total += a[i]                      # Reducción segura: cada hilo acumula por separado
    return total
```

### 8.3 Condiciones de carrera

Cada iteración debe escribir en posiciones **distintas**. Este patrón produce resultados incorrectos:

```python
@njit(parallel=True)
def histograma_incorrecto(datos, n_cubetas):
    conteo = np.zeros(n_cubetas, dtype=np.int64)
    for i in prange(datos.shape[0]):
        conteo[int(datos[i] * n_cubetas)] += 1     # ✗ Varios hilos escriben en la misma celda
    return conteo
```

Solución: un histograma parcial por bloque y suma final:

```python
@njit(parallel=True)
def histograma(datos, n_cubetas, n_bloques=16):
    parciales = np.zeros((n_bloques, n_cubetas), dtype=np.int64)
    tam = (datos.shape[0] + n_bloques - 1) // n_bloques
    for b in prange(n_bloques):
        for i in range(b * tam, min((b + 1) * tam, datos.shape[0])):
            parciales[b, min(int(datos[i] * n_cubetas), n_cubetas - 1)] += 1
    return parciales.sum(axis=0)
```

### 8.4 Control del número de hilos

```python
nb.config.NUMBA_NUM_THREADS     # Máximo disponible
nb.get_num_threads()            # Hilos en uso
nb.set_num_threads(4)           # Limitar (≤ NUMBA_NUM_THREADS)
```

### 8.5 Paralelización automática de expresiones

Con `parallel=True`, Numba también paraleliza operaciones de arrays sin bucles explícitos:

```python
@njit(parallel=True)
def normalizar(a):
    return (a - a.mean()) / a.std()       # Operaciones fusionadas y paralelizadas
```

> La paralelización tiene un coste fijo de arranque. Para arrays pequeños (menos de unos miles de elementos) o bucles con poco trabajo, la versión secuencial suele ser más rápida.

---

## 9. Funciones universales: `@vectorize` y `@guvectorize`

### 9.1 `@vectorize`: de escalar a array

Escriba la operación para **un elemento**; Numba crea una *ufunc* de NumPy que admite arrays, *broadcasting* y los métodos `reduce`/`accumulate`:

```python
from numba import vectorize, float64

@vectorize([float64(float64, float64)])
def hipotenusa(a, b):
    return (a * a + b * b) ** 0.5

hipotenusa(np.array([3.0, 5.0]), np.array([4.0, 12.0]))   # array([ 5., 13.])
hipotenusa(np.ones((3, 3)), 2.0)                          # Broadcasting
```

| `target` | Ejecución |
|---|---|
| `"cpu"` (defecto) | Un núcleo |
| `"parallel"` | Varios núcleos (requiere firma) |
| `"cuda"` | GPU NVIDIA (ver sección 15) |

```python
@vectorize(["float64(float64)"], target="parallel")
def funcion_costosa(x):
    return np.exp(-x) * np.sin(x) ** 2
```

### 9.2 `@guvectorize`: operaciones sobre subarrays

Para funciones que trabajan con **vectores o matrices** en lugar de escalares. Se indica la «firma de dimensiones» y el resultado se escribe en el último argumento:

```python
from numba import guvectorize

@guvectorize(["void(float64[:], int64, float64[:])"], "(n),()->(n)")
def media_movil(x, ventana, salida):
    acumulado = 0.0
    for i in range(x.shape[0]):
        acumulado += x[i]
        if i >= ventana:
            acumulado -= x[i - ventana]
        salida[i] = acumulado / min(i + 1, ventana)

media_movil(np.arange(6.0), 3)                   # array([0. , 0.5, 1. , 2. , 3. , 4. ])
media_movil(np.random.rand(1000, 250), 20)       # Se aplica a cada una de las 1000 filas
```

---

## 10. Estructuras de datos: listas, diccionarios y clases

### 10.1 Listas tipadas (`typed.List`)

Las listas de Python que se pasan a una función compilada se convierten («reflejan») con un coste alto y están en desuso. Use `numba.typed.List`:

```python
from numba.typed import List

valores = List([1.5, 2.0, 3.25])                 # Tipo inferido: float64
vacia = List.empty_list(types.float64)

@njit
def filtrar_positivos(datos):
    resultado = List.empty_list(types.float64)
    for x in datos:
        if x > 0:
            resultado.append(x)
    return resultado
```

Dentro de una función compilada, `[]` y `list.append` funcionan con normalidad siempre que todos los elementos sean del mismo tipo.

### 10.2 Diccionarios tipados (`typed.Dict`)

```python
from numba.typed import Dict

precios = Dict.empty(key_type=types.unicode_type, value_type=types.float64)
precios["TEC-01"] = 59.90
precios["RAT-02"] = 24.50

@njit
def total_inventario(precios, stock):
    total = 0.0
    for codigo, precio in precios.items():
        total += precio * stock.get(codigo, 0)
    return total
```

### 10.3 Clases compiladas (`jitclass`)

```python
from numba import float64
from numba.experimental import jitclass

especificacion = [
    ("saldo", float64),
    ("movimientos", types.ListType(float64)),
]

@jitclass(especificacion)
class Cuenta:
    def __init__(self, saldo):
        self.saldo = saldo
        self.movimientos = List.empty_list(float64)

    def ingresar(self, importe):
        self.saldo += importe
        self.movimientos.append(importe)

    @property
    def n_movimientos(self):
        return len(self.movimientos)

cuenta = Cuenta(100.0)
cuenta.ingresar(50.0)
cuenta.saldo                  # 150.0
```

> `jitclass` sigue en el módulo `experimental`: no admite herencia y su creación desde Python es lenta. Para la mayoría de casos, es preferible trabajar con arrays y funciones.

### 10.4 Arrays estructurados

Alternativa eficiente a las clases para colecciones de registros:

```python
dtype_pedido = np.dtype([("id", np.int64), ("importe", np.float64), ("urgente", np.bool_)])
pedidos = np.zeros(1000, dtype=dtype_pedido)

@njit
def importe_urgente(pedidos):
    total = 0.0
    for p in pedidos:
        if p.urgente:
            total += p.importe
    return total
```

---

## 11. Funciones como argumentos y extensión de Numba

### 11.1 Pasar funciones compiladas

```python
@njit
def aplicar(f, x):
    return f(x)

@njit
def doble(x):
    return 2 * x

aplicar(doble, 3)          # 6
```

> Cada función distinta que se pasa provoca una nueva compilación de `aplicar`. Para integradores o resolutores genéricos, es una técnica muy útil.

### 11.2 *Closures* (funciones generadoras)

```python
def crear_polinomio(coeficientes):
    coefs = np.asarray(coeficientes, dtype=np.float64)

    @njit
    def evaluar(x):
        resultado = 0.0
        for c in coefs[::-1]:              # Método de Horner
            resultado = resultado * x + c
        return resultado
    return evaluar

p = crear_polinomio([1, 0, 2])             # 1 + 2x²
p(3.0)                                     # 19.0
```

> Las variables globales y las capturadas se **congelan** en el momento de compilar: si cambian después, la función compilada no lo detecta.

### 11.3 Implementar funciones para Numba con `@overload`

Permite que una función Python tenga una implementación compilable:

```python
from numba.extending import overload

def recortar(x, minimo, maximo):
    return min(max(x, minimo), maximo)     # Versión Python normal

@overload(recortar)
def _recortar_numba(x, minimo, maximo):
    def implementacion(x, minimo, maximo):
        return min(max(x, minimo), maximo)
    return implementacion

@njit
def usar(x):
    return recortar(x, 0, 10)
```

---

## 12. Integración con código externo

### 12.1 Volver a Python dentro de una función compilada (`objmode`)

```python
from numba import objmode

@njit
def proceso(datos):
    resultado = datos.sum()
    with objmode(mediana="float64"):           # Se declara el tipo de las variables de salida
        mediana = float(np.median(datos))      # Código Python normal (lento)
        print("Mediana:", round(mediana, 3))
    return resultado, mediana
```

> Úselo con moderación. Aunque el bloque se ejecuta como Python, Numba analiza toda la función: las f-strings con formato (`{x:.3f}`) no se admiten ni siquiera dentro de `objmode`.
>
> Además, cada entrada en `objmode` tiene un coste. Es útil para registrar, mostrar progreso o llamar ocasionalmente a una librería no compatible.

### 12.2 *Callbacks* nativos para SciPy (`@cfunc`)

`@cfunc` genera una función C. Combinada con `scipy.LowLevelCallable`, acelera integradores y resolutores que llaman a la función millones de veces:

```python
from numba import cfunc
from scipy import LowLevelCallable
from scipy.integrate import quad

@cfunc("float64(float64)")
def integrando(x):
    return np.exp(-x * x)

quad(LowLevelCallable(integrando.ctypes), -np.inf, np.inf)    # (1.7724538509055159, ...)
```

### 12.3 Liberar el GIL y usar hilos de Python

```python
from concurrent.futures import ThreadPoolExecutor

@njit(nogil=True)
def procesar_bloque(bloque):
    return np.sqrt(bloque).sum()

bloques = np.array_split(np.random.rand(10_000_000), 8)
with ThreadPoolExecutor(max_workers=8) as ejecutor:
    resultados = list(ejecutor.map(procesar_bloque, bloques))
```

---

## 13. Medir el rendimiento correctamente

### 13.1 Separar compilación y ejecución

```python
import time

funcion(datos)                       # Calentamiento: compila
inicio = time.perf_counter()
funcion(datos)
print(f"{time.perf_counter() - inicio:.4f} s")
```

En Jupyter / IPython:

```python
funcion(datos)          # Compilar primero
%timeit funcion(datos)
```

### 13.2 Comparar siempre con NumPy

| Situación | Ganancia esperada con Numba |
|---|---|
| Bucle Python puro sobre un array grande | Muy alta (50×–500×) |
| Código ya vectorizado con NumPy (una operación) | Nula o mínima |
| Varias operaciones NumPy encadenadas con arrays temporales | Moderada (2×–5×) al fusionarlas en un bucle |
| Algoritmos con dependencias entre iteraciones | Muy alta |
| Funciones con E/S, pandas o cadenas complejas | No aplicable |

### 13.3 Disposición de memoria

Recorra los arrays en el orden en que están almacenados (por filas en arrays C):

```python
@njit
def suma_rapida(m):
    total = 0.0
    for i in range(m.shape[0]):          # Fila externa
        for j in range(m.shape[1]):      # Columna interna: acceso contiguo
            total += m[i, j]
    return total
```

> Evite crear arrays dentro de bucles calientes (`np.zeros` en cada iteración): reserve la memoria una vez fuera del bucle y reutilícela.

---

## 14. Depuración y diagnóstico

### 14.1 Desactivar la compilación

```bash
NUMBA_DISABLE_JIT=1 python script.py        # Linux / macOS
set NUMBA_DISABLE_JIT=1 && python script.py # Windows (cmd)
```

Con la compilación desactivada puede usar `pdb`, puntos de interrupción del IDE y `print` con formato.

### 14.2 Leer un `TypingError`

```
numba.core.errors.TypingError: Failed in nopython mode pipeline (step: nopython frontend)
Cannot unify int64 and unicode_type for 'resultado.2', defined at script.py (6)
```

Lea el mensaje **desde el final**: la última línea con número de archivo indica dónde está el problema. Las causas más comunes son tipos incompatibles en una variable, una función no admitida o un argumento de tipo no soportado.

### 14.3 Inspeccionar la compilación

```python
funcion.signatures                   # Firmas compiladas
funcion.inspect_types()              # Tipo inferido de cada variable, línea a línea
funcion.parallel_diagnostics(level=1)  # Qué bucles se han paralelizado (con parallel=True)
funcion.inspect_llvm()               # Código intermedio LLVM (avanzado)
funcion.inspect_asm()                # Ensamblador generado (avanzado)
```

### 14.4 Comprobación de índices

```python
@njit(boundscheck=True)              # O NUMBA_BOUNDSCHECK=1
def acceso(a, i):
    return a[i]                      # Lanza IndexError en lugar de leer memoria inválida
```

> Sin `boundscheck`, un índice fuera de rango puede devolver valores aleatorios o cerrar el intérprete sin mensaje de error.

---

## 15. Computación en GPU

Numba puede compilar *kernels* para GPU NVIDIA con CUDA. Desde Numba 0.61, el soporte CUDA integrado está en desuso y se distribuye en el paquete **`numba-cuda`**, mantenido por NVIDIA:

```bash
pip install numba-cuda[cu12]
```

```python
from numba import cuda

@cuda.jit
def sumar_vectores(a, b, salida):
    i = cuda.grid(1)                       # Índice global del hilo
    if i < salida.shape[0]:
        salida[i] = a[i] + b[i]

n = 1_000_000
a_gpu = cuda.to_device(np.random.rand(n))
b_gpu = cuda.to_device(np.random.rand(n))
salida_gpu = cuda.device_array(n)

hilos = 256
bloques = (n + hilos - 1) // hilos
sumar_vectores[bloques, hilos](a_gpu, b_gpu, salida_gpu)
resultado = salida_gpu.copy_to_host()
```

> La transferencia de datos entre CPU y GPU es costosa: mantenga los datos en la GPU durante todo el proceso. Para operaciones de arrays de alto nivel en GPU, valore **CuPy** o **JAX**.

---

## 16. Ejemplo práctico completo

**Simulación de Monte Carlo de una cartera de acciones**: genera 200 000 escenarios de precios correlacionados (movimiento browniano geométrico con descomposición de Cholesky), calcula el valor en riesgo (VaR) y el déficit esperado (ES) al 99 %, y compara el rendimiento de Python puro, NumPy vectorizado y Numba en paralelo. Usa caché de compilación, `prange`, `fastmath` y mide la compilación por separado.

Requisitos: `pip install numba numpy`

```python
"""
Simulación de Monte Carlo de una cartera de acciones con Numba.

Calcula el valor en riesgo (VaR) y el déficit esperado (ES) de una cartera
simulando trayectorias de precios con movimiento browniano geométrico
correlacionado, y compara el rendimiento de tres implementaciones.
"""
import time

import numpy as np
from numba import njit, prange, get_num_threads

# ------------------------------------------------------------------ Parámetros
ACTIVOS = ("TEC", "BAN", "ENE")
PRECIOS = np.array([150.0, 42.0, 18.5])          # Precio actual (€)
TITULOS = np.array([200.0, 1_500.0, 3_000.0])    # Títulos en cartera
MU = np.array([0.08, 0.05, 0.04])                # Rentabilidad anual esperada
SIGMA = np.array([0.30, 0.22, 0.18])             # Volatilidad anual
CORR = np.array([[1.0, 0.45, 0.30],
                 [0.45, 1.0, 0.50],
                 [0.30, 0.50, 1.0]])
DIAS = 10                                        # Horizonte (días hábiles)
N_SIM = 200_000                                  # Escenarios
NIVEL = 0.99                                     # Nivel de confianza


# ------------------------------------------------------------------ 1. Python puro
def simular_python(precios, titulos, mu, sigma, chol, dias, n_sim, semilla):
    rng = np.random.default_rng(semilla)
    dt = 1.0 / 252.0
    n = len(precios)
    valor_inicial = sum(precios[j] * titulos[j] for j in range(n))
    pyg = np.empty(n_sim)
    for s in range(n_sim):
        p = list(precios)
        for _ in range(dias):
            z = rng.standard_normal(n)
            for j in range(n):
                e = sum(chol[j, k] * z[k] for k in range(j + 1))
                p[j] *= np.exp((mu[j] - 0.5 * sigma[j] ** 2) * dt + sigma[j] * np.sqrt(dt) * e)
        pyg[s] = sum(p[j] * titulos[j] for j in range(n)) - valor_inicial
    return pyg


# ------------------------------------------------------------------ 2. NumPy vectorizado
def simular_numpy(precios, titulos, mu, sigma, chol, dias, n_sim, semilla):
    rng = np.random.default_rng(semilla)
    dt = 1.0 / 252.0
    z = rng.standard_normal((n_sim, dias, len(precios))) @ chol.T
    deriva = (mu - 0.5 * sigma**2) * dt
    log_rend = (deriva + sigma * np.sqrt(dt) * z).sum(axis=1)
    precios_finales = precios * np.exp(log_rend)
    return precios_finales @ titulos - precios @ titulos


# ------------------------------------------------------------------ 3. Numba en paralelo
@njit(parallel=True, fastmath=True, cache=True)
def simular_numba(precios, titulos, mu, sigma, chol, dias, n_sim, semilla):
    np.random.seed(semilla)                      # Con prange, cada hilo usa su propio generador
    dt = 1.0 / 252.0
    n = precios.shape[0]
    deriva = (mu - 0.5 * sigma**2) * dt
    vol = sigma * np.sqrt(dt)
    valor_inicial = 0.0
    for j in range(n):
        valor_inicial += precios[j] * titulos[j]

    pyg = np.empty(n_sim)
    for s in prange(n_sim):
        log_p = np.zeros(n)
        z = np.empty(n)
        for _ in range(dias):
            for j in range(n):
                z[j] = np.random.standard_normal()
            for j in range(n):
                e = 0.0
                for k in range(j + 1):
                    e += chol[j, k] * z[k]
                log_p[j] += deriva[j] + vol[j] * e
        valor = 0.0
        for j in range(n):
            valor += precios[j] * np.exp(log_p[j]) * titulos[j]
        pyg[s] = valor - valor_inicial
    return pyg


@njit(cache=True)
def var_es(pyg, nivel):
    """VaR y ES históricos de una distribución de pérdidas y ganancias."""
    ordenado = np.sort(pyg)
    corte = int((1.0 - nivel) * ordenado.shape[0])
    var = -ordenado[corte]
    es = -ordenado[:corte].mean()
    return var, es


# ------------------------------------------------------------------ Utilidades
def cronometrar(funcion, *args, repeticiones=3):
    mejor = float("inf")
    for _ in range(repeticiones):
        inicio = time.perf_counter()
        resultado = funcion(*args)
        mejor = min(mejor, time.perf_counter() - inicio)
    return resultado, mejor


def main() -> None:
    chol = np.linalg.cholesky(CORR)
    args = (PRECIOS, TITULOS, MU, SIGMA, chol, DIAS)
    valor = PRECIOS @ TITULOS
    print(f"Cartera: {', '.join(ACTIVOS)} · valor actual {valor:,.2f} €")
    print(f"Horizonte {DIAS} días · {N_SIM:,} escenarios · {get_num_threads()} hilos\n")

    # Compilación (primera llamada) y medición por separado
    inicio = time.perf_counter()
    simular_numba(*args, 1_000, 1)
    var_es(np.zeros(100), NIVEL)
    print(f"Compilación / carga de caché: {time.perf_counter() - inicio:.2f} s\n")

    n_py = N_SIM // 100                          # Python puro es demasiado lento para N_SIM
    _, t_py = cronometrar(simular_python, *args, n_py, 1, repeticiones=1)
    t_py *= N_SIM / n_py                         # Extrapolado
    pyg_np, t_np = cronometrar(simular_numpy, *args, N_SIM, 1)
    pyg_nb, t_nb = cronometrar(simular_numba, *args, N_SIM, 1)

    print(f"{'Implementación':<22}{'Tiempo (s)':>12}{'Aceleración':>14}")
    for nombre, t in [("Python puro (estim.)", t_py), ("NumPy vectorizado", t_np), ("Numba paralelo", t_nb)]:
        print(f"{nombre:<22}{t:>12.3f}{t_py / t:>13.0f}x")

    print(f"\n{'Método':<10}{'VaR 99 %':>14}{'ES 99 %':>14}")
    for nombre, pyg in [("NumPy", pyg_np), ("Numba", pyg_nb)]:
        var, es = var_es(pyg, NIVEL)
        print(f"{nombre:<10}{var:>12,.2f} €{es:>12,.2f} €")


if __name__ == "__main__":
    main()
```

Resultado orientativo (2 núcleos; los tiempos varían según el equipo):

```
Cartera: TEC, BAN, ENE · valor actual 148,500.00 €
Horizonte 10 días · 200,000 escenarios · 2 hilos

Compilación / carga de caché: 3.66 s

Implementación          Tiempo (s)   Aceleración
Python puro (estim.)        16.132            1x
NumPy vectorizado            0.186           87x
Numba paralelo               0.110          147x

Método          VaR 99 %       ES 99 %
NumPy        11,391.15 €   12,975.29 €
Numba        11,381.24 €   12,961.89 €
```

> En la segunda ejecución, gracias a `cache=True`, la compilación se sustituye por la carga desde caché (≈ 0,3 s). Las pequeñas diferencias de VaR entre NumPy y Numba se deben a que usan generadores aleatorios distintos. Además, la versión NumPy necesita reservar un array de `N_SIM × DIAS × activos`, mientras que Numba trabaja escenario a escenario con memoria constante.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TypingError: Cannot unify int64 and unicode_type` | Una variable recibe tipos incompatibles. | Use un único tipo por variable; inicialice con el tipo correcto. |
| `TypingError: Unknown attribute ... of type module` / `Untyped global name` | Se llama a una función o librería no admitida (pandas, función Python sin decorar). | Decore la función con `@njit`, pase arrays NumPy o use `objmode`. |
| `TypingError: No implementation of function ... found for signature` | Argumentos no admitidos por esa función de NumPy (p. ej. `axis` como tupla). | Consulte la lista de funciones admitidas; reescriba con un bucle. |
| `NumbaDeprecationWarning: reflected list` | Se pasa una lista de Python a una función compilada. | Use `np.array` o `numba.typed.List`. |
| La primera llamada es muy lenta | Tiempo de compilación. | Es normal; use `cache=True` y mida a partir de la segunda llamada. |
| No hay mejora de rendimiento | El código ya era NumPy vectorizado o el trabajo es muy pequeño. | Compare con NumPy; aplique Numba solo a bucles costosos. |
| Resultados distintos con `parallel=True` | Condición de carrera: varios hilos escriben en la misma posición. | Escriba en posiciones independientes o use reducciones parciales. |
| Resultados aleatorios no reproducibles | La semilla se fijó desde Python o el código es paralelo. | Llame a `np.random.seed()` dentro de la función; con `prange` cada hilo tiene su propio generador. |
| La función no detecta cambios en una variable global | Las globales se congelan al compilar. | Pase el valor como argumento. |
| Resultados `NaN` incorrectos o comparaciones erróneas | `fastmath=True` asume que no hay `NaN`/`inf`. | Desactive `fastmath` o use `fastmath={"nsz", "arcp", "contract", "afn", "reassoc"}` (sin `nnan`/`ninf`). |
| El programa se cierra sin mensaje / valores basura | Índice fuera de rango sin comprobación. | Active `boundscheck=True` mientras depura. |
| `ZeroDivisionError` dentro de la función compilada | Modelo de errores de Python. | Compruebe el divisor o use `error_model="numpy"`. |
| Error al instalar o importar tras actualizar NumPy | Versiones incompatibles de Numba/NumPy/Python. | Actualice Numba o instale versiones compatibles. |

---

## 18. Buenas prácticas

- **Mida antes de optimizar:** identifique el cuello de botella (`cProfile`, `line_profiler`) y aplique Numba solo a esas funciones.
- Use siempre **`@njit`** (modo *nopython*); si no compila, simplifique el código en lugar de forzar el modo objeto.
- Escriba funciones **pequeñas y puras** que reciban arrays NumPy y escalares, y devuelvan arrays o escalares.
- Separe la **lógica numérica** (compilada) de la **E/S, validación y presentación** (Python normal).
- Active **`cache=True`** en el código de producción para evitar recompilaciones.
- Mantenga la **estabilidad de tipos**: un tipo por variable, acumuladores decimales inicializados con `0.0`.
- Reserve la memoria **fuera** de los bucles y reutilícela; recorra los arrays en orden de memoria.
- Use `parallel=True` solo cuando haya suficiente trabajo por iteración y las iteraciones sean independientes.
- Use **`fastmath`** solo tras comprobar que no altera resultados relevantes.
- Verifique siempre los resultados comparándolos con la versión original (`funcion.py_func`) mediante pruebas automáticas (`np.testing.assert_allclose`).
- Fije las versiones de Numba, NumPy y Python en `requirements.txt`.

---

## 19. Referencia rápida

```python
from numba import njit, prange, vectorize, guvectorize, cfunc, objmode, types
from numba.typed import List, Dict
from numba.experimental import jitclass

@njit                                             # Compilación JIT (modo nopython)
@njit(cache=True, fastmath=True, parallel=True, nogil=True, boundscheck=True, error_model="numpy")
@njit("float64(float64[:])")                      # Firma explícita (compilación anticipada)
for i in prange(n): ...                           # Bucle paralelo (con parallel=True)

@vectorize(["float64(float64)"], target="parallel")           # ufunc escalar → array
@guvectorize(["void(float64[:], float64[:])"], "(n)->(n)")    # ufunc sobre subarrays
@cfunc("float64(float64)")                                    # Función C (callbacks SciPy)
with objmode(x="float64"): ...                                # Volver a Python dentro de @njit

List.empty_list(types.float64) / Dict.empty(key_type=..., value_type=...) / @jitclass(spec)
f.py_func / f.signatures / f.inspect_types() / f.parallel_diagnostics(level=1)
nb.typeof(x) / nb.get_num_threads() / nb.set_num_threads(n)
NUMBA_DISABLE_JIT=1 · NUMBA_NUM_THREADS=n · NUMBA_CACHE_DIR=ruta · numba -s
```

---

## 20. Ejercicios propuestos

1. Implemente el cálculo del conjunto de Mandelbrot en Python puro, NumPy y Numba (`prange`), y compare los tiempos para una imagen de 2000 × 2000 píxeles.
2. Escriba una *ufunc* con `@vectorize` que calcule la cuota de un préstamo (sistema francés) a partir del capital, el tipo de interés y el plazo, y aplíquela a una rejilla de combinaciones.
3. Programe el algoritmo *k-means* con Numba (asignación paralela de puntos a centroides) y compare el resultado con `sklearn.cluster.KMeans`.
4. Cree una función `@guvectorize` que calcule la volatilidad móvil de 1000 series de precios de forma simultánea.
5. Amplíe el ejemplo completo para calcular el VaR con distintos horizontes (1, 5, 10 y 20 días) y represente los resultados con Matplotlib.

---

## 21. Referencias

- Documentación oficial: https://numba.readthedocs.io/
- Guía de inicio en 5 minutos: https://numba.readthedocs.io/en/stable/user/5minguide.html
- Funciones de NumPy admitidas: https://numba.readthedocs.io/en/stable/reference/numpysupported.html
- Consejos de rendimiento: https://numba.readthedocs.io/en/stable/user/performance-tips.html
- Numba CUDA (NVIDIA): https://nvidia.github.io/numba-cuda/
