# Manual de uso: NumPy

> **Categoría:** Ciencia de datos · **Utilidad:** Cálculo numérico y matemáticas con *arrays*
> **Tipo:** Librería de terceros (`pip install numpy`) · **Versión de referencia:** 2.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Crear *arrays*](#4-crear-arrays)
5. [Atributos y tipos de datos](#5-atributos-y-tipos-de-datos)
6. [Indexación y *slicing*](#6-indexación-y-slicing)
7. [Operaciones vectorizadas y funciones universales](#7-operaciones-vectorizadas-y-funciones-universales)
8. [*Broadcasting*](#8-broadcasting)
9. [Agregaciones y ejes](#9-agregaciones-y-ejes)
10. [Cambiar la forma, unir y dividir](#10-cambiar-la-forma-unir-y-dividir)
11. [Vistas y copias](#11-vistas-y-copias)
12. [Condiciones, búsqueda y ordenación](#12-condiciones-búsqueda-y-ordenación)
13. [Álgebra lineal](#13-álgebra-lineal)
14. [Números aleatorios](#14-números-aleatorios)
15. [Estadística](#15-estadística)
16. [Valores especiales: `nan` e `inf`](#16-valores-especiales-nan-e-inf)
17. [Fechas, texto y *arrays* estructurados](#17-fechas-texto-y-arrays-estructurados)
18. [Entrada y salida](#18-entrada-y-salida)
19. [Rendimiento](#19-rendimiento)
20. [Cambios importantes en NumPy 2](#20-cambios-importantes-en-numpy-2)
21. [Ejemplo práctico completo](#21-ejemplo-práctico-completo)
22. [Errores frecuentes](#22-errores-frecuentes)
23. [Buenas prácticas](#23-buenas-prácticas)
24. [Referencia rápida](#24-referencia-rápida)
25. [Ejercicios propuestos](#25-ejercicios-propuestos)
26. [Referencias](#26-referencias)

---

## 1. Descripción

**NumPy** (*Numerical Python*) es la base del cálculo científico en Python. Proporciona el ***array* multidimensional** (`ndarray`): una estructura homogénea y compacta en memoria sobre la que se realizan operaciones matemáticas **vectorizadas** implementadas en C, decenas o cientos de veces más rápidas que los bucles de Python. pandas, SciPy, scikit-learn, Matplotlib, OpenCV y la mayoría de librerías científicas se construyen sobre NumPy.

**Ventajas:**
- Rendimiento muy alto en operaciones sobre grandes volúmenes de números.
- Sintaxis matemática concisa: `a * b + c` opera elemento a elemento.
- *Broadcasting*: operar con *arrays* de formas distintas sin copiar datos.
- Álgebra lineal, números aleatorios, estadística, transformadas de Fourier.
- Interoperabilidad con todo el ecosistema (y con GPU mediante CuPy, JAX, PyTorch).

**Limitaciones:**
- Datos homogéneos: todos los elementos del *array* tienen el mismo tipo.
- No tiene etiquetas de filas o columnas (para eso, pandas).
- Los *arrays* tienen tamaño fijo; añadir elementos implica crear uno nuevo.
- Se ejecuta en CPU y en memoria.

| Si necesita… | Opción |
|---|---|
| Cálculo numérico con vectores y matrices | **NumPy** |
| Datos tabulares con columnas de distinto tipo | pandas |
| Optimización, integración, estadística avanzada, señales | SciPy |
| Cálculo en GPU o diferenciación automática | CuPy / JAX / PyTorch |

---

## 2. Instalación e importación

```bash
pip install numpy
```

```python
import numpy as np
print(np.__version__)
np.set_printoptions(precision=3, suppress=True)      # Formato de impresión
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `ndarray` | *Array* de N dimensiones con elementos del mismo tipo. |
| `shape` | Forma: tamaño de cada dimensión, p. ej. `(3, 4)`. |
| `ndim` | Número de dimensiones (ejes). |
| `dtype` | Tipo de los elementos (`int64`, `float64`, `bool`…). |
| Eje (*axis*) | Dimensión a lo largo de la que se opera: `axis=0` filas (vertical), `axis=1` columnas (horizontal). |
| Vectorización | Operar sobre *arrays* completos sin bucles de Python. |
| *ufunc* | Función universal que actúa elemento a elemento (`np.sqrt`, `np.exp`…). |
| *Broadcasting* | Reglas para operar con *arrays* de formas compatibles. |
| Vista | *Array* que comparte memoria con otro. |

```
                axis=1 →
            ┌──────────────┐
  axis=0 ↓  │  1   2   3   │   shape = (2, 3)
            │  4   5   6   │   ndim = 2
            └──────────────┘
```

---

## 4. Crear *arrays*

```python
np.array([1, 2, 3])                          # Desde una lista
np.array([[1, 2, 3], [4, 5, 6]], dtype=float)
np.zeros((3, 4)) ; np.ones(5) ; np.full((2, 2), 7.5) ; np.empty((2, 3))
np.zeros_like(a) ; np.ones_like(a)
np.eye(3)                                    # Identidad
np.diag([1, 2, 3])
np.arange(0, 10, 2)                          # [0 2 4 6 8] (fin excluido)
np.linspace(0, 1, 5)                         # [0. 0.25 0.5 0.75 1.] (fin incluido)
np.logspace(0, 3, 4)                         # [1 10 100 1000]
np.tile([1, 2], 3) ; np.repeat([1, 2], 3)
xx, yy = np.meshgrid(np.linspace(-1, 1, 5), np.linspace(-1, 1, 5))   # Rejillas 2D
np.fromfunction(lambda i, j: i * 10 + j, (3, 4), dtype=int)
```

> Use `linspace` cuando necesite un número exacto de puntos con decimales; `arange` con pasos decimales puede dar un elemento más o menos por errores de redondeo.

---

## 5. Atributos y tipos de datos

```python
a = np.array([[1.5, 2.0, 3.25], [4.0, 5.5, 6.0]])
a.shape     # (2, 3)
a.ndim      # 2
a.size      # 6
a.dtype     # float64
a.itemsize  # 8 bytes por elemento
a.nbytes    # 48 bytes
a.T         # Traspuesta (3, 2)

a.astype(np.int32)          # Conversión (crea un array nuevo)
np.array([1, 2], dtype=np.uint8)
```

| `dtype` | Rango / uso |
|---|---|
| `bool` | `True`/`False`. |
| `int8`, `int16`, `int32`, `int64` | Enteros con signo. |
| `uint8` … `uint64` | Enteros sin signo (`uint8`: imágenes, 0-255). |
| `float32`, `float64` | Decimales (simple y doble precisión). |
| `complex128` | Complejos. |
| `datetime64`, `timedelta64` | Fechas y duraciones. |
| `StringDType()` / `str_` | Texto (variable / fijo). |

```python
np.iinfo(np.int16)          # Límites de enteros
np.finfo(np.float32)        # Precisión de decimales
```

> Cuidado con el **desbordamiento**: `np.array([250], dtype=np.uint8) + 10` da `[4]` (no hay error). Use un tipo mayor antes de operar.

---

## 6. Indexación y *slicing*

```python
a = np.arange(1, 13).reshape(3, 4)
# [[ 1  2  3  4]
#  [ 5  6  7  8]
#  [ 9 10 11 12]]

a[0, 0]        # 1
a[-1, -1]      # 12
a[1]           # Fila 1 → [5 6 7 8]
a[:, 2]        # Columna 2 → [3 7 11]
a[0:2, 1:3]    # Submatriz [[2 3] [6 7]]
a[::2, ::-1]   # Filas alternas, columnas invertidas
a[..., 0]      # Primer elemento del último eje (útil en N dimensiones)

# Indexación booleana
a[a > 6]                       # [ 7  8  9 10 11 12]
a[(a % 2 == 0) & (a > 4)]      # [ 6  8 10 12]

# Indexación con listas de índices («fancy indexing»)
a[[0, 2]]                      # Filas 0 y 2
a[[0, 1, 2], [3, 2, 1]]        # Elementos (0,3), (1,2), (2,1) → [4 7 10]
a[np.ix_([0, 2], [1, 3])]      # Submatriz de filas 0,2 y columnas 1,3

# Asignación
a[a > 10] = 0
a[:, 0] = -1
```

> Una rebanada (`a[0:2]`) es una **vista**: modificarla modifica el original. La indexación booleana o con listas devuelve **copias**.

---

## 7. Operaciones vectorizadas y funciones universales

```python
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

a + b ; a - b ; a * b ; a / b ; a // b ; a % b ; a ** 2 ; -a
a == b ; a < 3                       # Comparaciones → arrays booleanos
a @ b                                # Producto escalar (vectores) o matricial

np.sqrt(a) ; np.exp(a) ; np.log(a) ; np.log10(a) ; np.log1p(a)
np.sin(a) ; np.cos(a) ; np.arctan2(y, x) ; np.deg2rad(180)
np.abs(a) ; np.sign(a) ; np.round(a / 3, 2) ; np.floor(x) ; np.ceil(x) ; np.trunc(x)
np.maximum(a, 2) ; np.minimum(a, b)          # Elemento a elemento
np.clip(a, 2, 3)                              # Limitar a [2, 3]
np.hypot(x, y)

np.add(a, b, out=resultado)                  # Escribir en un array existente (sin crear otro)
np.add.reduce(a) ; np.multiply.accumulate(a) ; np.add.outer(a, b)
```

Funciones propias vectorizadas:

```python
def tramo_iva(precio):
    return np.where(precio < 100, precio * 1.10, precio * 1.21)

tramo_iva(np.array([50, 150]))       # Operaciones con np.where: rápidas

np.vectorize(func_python)            # Comodidad, no velocidad (sigue siendo un bucle)
```

---

## 8. *Broadcasting*

NumPy puede operar con *arrays* de formas distintas «estirando» virtualmente las dimensiones de tamaño 1, sin copiar datos.

**Regla:** se comparan las formas desde la derecha; dos dimensiones son compatibles si son iguales o una de ellas es 1.

```python
m = np.arange(12).reshape(3, 4)       # (3, 4)
m + 10                                # Escalar: se suma a todo
m + np.array([1, 2, 3, 4])            # (3,4) + (4,)  → suma por columnas
m + np.array([[100], [200], [300]])   # (3,4) + (3,1) → suma por filas

# Normalizar cada columna
normalizada = (m - m.mean(axis=0)) / m.std(axis=0)

# Distancias entre todos los puntos (N, 2) y (M, 2)
p = np.random.default_rng(0).random((5, 2))
q = np.random.default_rng(1).random((3, 2))
distancias = np.sqrt(((p[:, None, :] - q[None, :, :]) ** 2).sum(axis=-1))   # (5, 3)
```

| Forma A | Forma B | Resultado |
|---|---|---|
| `(3, 4)` | `(4,)` | `(3, 4)` |
| `(3, 4)` | `(3, 1)` | `(3, 4)` |
| `(5, 1, 2)` | `(1, 3, 2)` | `(5, 3, 2)` |
| `(3, 4)` | `(3,)` | ✖ Error: alinee con `[:, None]` |

`None` (o `np.newaxis`) añade una dimensión de tamaño 1: `v[:, None]` convierte `(3,)` en `(3, 1)`.

---

## 9. Agregaciones y ejes

```python
m = np.array([[1, 2, 3], [4, 5, 6]])
m.sum()            # 21
m.sum(axis=0)      # [5 7 9]   (por columnas: «colapsa» las filas)
m.sum(axis=1)      # [ 6 15]   (por filas)
m.mean(axis=1, keepdims=True)    # Mantiene la dimensión → (2, 1), útil para broadcasting
m.min() ; m.max() ; m.argmin() ; m.argmax(axis=1)
m.prod() ; m.cumsum(axis=1) ; m.cumprod()
m.std(ddof=1) ; m.var()          # ddof=1: desviación muestral
np.median(m) ; np.percentile(m, [25, 75]) ; np.quantile(m, 0.9)
np.ptp(m)                        # Rango (max - min)
m.any() ; m.all() ; np.count_nonzero(m > 2)
```

---

## 10. Cambiar la forma, unir y dividir

```python
a = np.arange(12)
a.reshape(3, 4) ; a.reshape(-1, 6)          # -1: calcula esa dimensión
a.reshape(3, 4).ravel()                     # Aplanar (vista si es posible)
a.reshape(3, 4).flatten()                   # Aplanar (siempre copia)
a[:, np.newaxis] ; np.expand_dims(a, 0)     # Añadir dimensión
np.squeeze(x)                               # Quitar dimensiones de tamaño 1
m.T ; np.transpose(t, (2, 0, 1)) ; np.moveaxis(t, -1, 0) ; np.swapaxes(t, 0, 1)

np.concatenate([a, b])                      # Por un eje existente
np.vstack([fila1, fila2]) ; np.hstack([c1, c2]) ; np.column_stack([x, y])
np.stack([a, b], axis=0)                    # Por un eje nuevo
np.split(a, 3) ; np.array_split(a, 5) ; np.hsplit(m, 2) ; np.vsplit(m, 3)

np.append(a, [99]) ; np.insert(a, 1, 0) ; np.delete(a, [0, 2])   # Crean arrays nuevos (lentos en bucles)
```

> Para construir un *array* elemento a elemento, acumule en una lista de Python y conviértala al final con `np.array(lista)`: `np.append` dentro de un bucle copia todo cada vez.

---

## 11. Vistas y copias

```python
a = np.arange(10)
v = a[2:5]          # Vista
v[0] = 100          # ¡Modifica a!
c = a[2:5].copy()   # Copia independiente

np.shares_memory(a, v)   # True
v.base is a              # True
```

| Operación | Devuelve |
|---|---|
| *Slicing* (`a[1:5]`), `reshape`, `T`, `ravel` (normalmente) | Vista |
| Indexación booleana o con listas, `flatten`, `copy`, operaciones aritméticas | Copia |

---

## 12. Condiciones, búsqueda y ordenación

```python
x = np.array([3, -1, 7, 0, -5, 8])
np.where(x > 0, x, 0)                       # Sustituir negativos por 0
np.where(x > 0)                             # Índices donde se cumple → (array([0, 2, 5]),)
np.nonzero(x) ; np.argwhere(x > 0)
np.select([x < 0, x == 0], ["negativo", "cero"], default="positivo")
np.isin(x, [0, 7, 9])                       # Pertenencia
np.unique(x) ; np.unique(x, return_counts=True)
np.unique_counts(x)                         # Valores y frecuencias (NumPy 2)

np.sort(x) ; x.sort()                       # Copia / en el sitio
np.argsort(x)                               # Índices que ordenan
np.argsort(-x)[:3]                          # Índices de los 3 mayores
np.partition(x, 2)                          # Los 2 menores al principio (más rápido que ordenar)
np.searchsorted([10, 20, 30], 25)           # Posición de inserción → 2
np.digitize(edades, bins=[18, 30, 50, 65])  # Asignar a tramos

# Ordenar filas de una matriz por una columna
datos = np.array([[3, 30], [1, 10], [2, 20]])
datos[datos[:, 0].argsort()]
np.lexsort((secundaria, primaria))          # Ordenación por varias claves
```

---

## 13. Álgebra lineal

```python
A = np.array([[3, 1], [1, 2]], dtype=float)
b = np.array([9, 8], dtype=float)

A @ A                          # Producto matricial (equivale a np.matmul)
np.dot(u, v) ; np.vdot(u, v) ; np.cross(u, v) ; np.outer(u, v)
np.linalg.solve(A, b)          # Resolver Ax = b → [2. 3.]
np.linalg.inv(A)               # Inversa (use solve si solo quiere resolver un sistema)
np.linalg.det(A)
np.linalg.eig(A)               # Valores y vectores propios (eigh para matrices simétricas)
np.linalg.svd(A)               # Descomposición en valores singulares
np.linalg.norm(v) ; np.linalg.norm(A, "fro")
np.linalg.matrix_rank(A)
np.linalg.pinv(A)              # Pseudoinversa
np.linalg.cholesky(A) ; np.linalg.qr(A)
np.trace(A)
np.linalg.matrix_power(A, 3)

# Mínimos cuadrados: ajustar y = m·x + c
x = np.array([0, 1, 2, 3]); y = np.array([-1, 0.2, 0.9, 2.1])
X = np.column_stack([x, np.ones_like(x)])
(m, c), *_ = np.linalg.lstsq(X, y)

np.polyfit(x, y, deg=2)        # Coeficientes de un polinomio de grado 2
np.polyval(coef, nuevos_x)
np.polynomial.Polynomial.fit(x, y, 2)   # API moderna de polinomios
```

> Use `@` para el producto matricial; `*` multiplica **elemento a elemento**.

---

## 14. Números aleatorios

Use la API moderna basada en `Generator` (la antigua `np.random.rand`, `np.random.seed` se mantiene por compatibilidad):

```python
rng = np.random.default_rng(seed=42)        # Semilla: resultados reproducibles

rng.random(5)                                # Uniforme [0, 1)
rng.uniform(10, 20, size=(2, 3))
rng.integers(1, 7, size=10)                  # Dados: 1..6 (fin excluido)
rng.normal(loc=100, scale=15, size=1000)     # Normal
rng.lognormal(mean=0, sigma=0.5, size=100)
rng.poisson(lam=3, size=100) ; rng.exponential(scale=2, size=100) ; rng.binomial(n=10, p=0.3, size=100)
rng.triangular(left=5, mode=8, right=15, size=100)
rng.choice(["A", "B", "C"], size=10, p=[0.5, 0.3, 0.2])
rng.choice(100, size=10, replace=False)      # Muestra sin reemplazo
rng.permutation(10) ; rng.shuffle(arr)
rng.multivariate_normal(media, covarianza, size=500)

hijos = rng.spawn(4)                         # Generadores independientes (p. ej., para procesos paralelos)
```

---

## 15. Estadística

```python
datos = rng.normal(50, 10, 1000)
datos.mean() ; np.median(datos) ; datos.std(ddof=1)
np.percentile(datos, [5, 50, 95])
np.corrcoef(x, y)                          # Matriz de correlación
np.cov(x, y)                               # Covarianza
conteos, bordes = np.histogram(datos, bins=20)
np.histogram2d(x, y, bins=10)
np.bincount([0, 1, 1, 3])                  # Frecuencias de enteros → [1 2 0 1]
np.average(valores, weights=pesos)         # Media ponderada
np.convolve(senal, np.ones(5) / 5, mode="valid")   # Media móvil
np.diff(x) ; np.gradient(y, x)
np.trapezoid(y, x)                         # Integral numérica (antes np.trapz)
np.interp(nuevos_x, x, y)                  # Interpolación lineal
np.fft.rfft(senal) ; np.fft.rfftfreq(n, d=1/frecuencia)   # Transformada de Fourier
```

> Para contrastes de hipótesis, distribuciones y estadística avanzada, use `scipy.stats`.

---

## 16. Valores especiales: `nan` e `inf`

```python
x = np.array([1.0, np.nan, 3.0, np.inf])
np.isnan(x) ; np.isinf(x) ; np.isfinite(x)
np.nanmean(np.array([1.0, np.nan, 3.0]))      # 2.0: ignora nan
np.nansum(x) ; np.nanmax(x) ; np.nanpercentile(x, 50)
np.nan_to_num(x, nan=0.0, posinf=1e9)
np.nan == np.nan                              # False: use np.isnan

with np.errstate(divide="ignore", invalid="ignore"):
    r = a / b                                 # Sin avisos por divisiones entre 0

np.isclose(0.1 + 0.2, 0.3) ; np.allclose(a, b, rtol=1e-5)   # Comparar decimales
```

`nan` solo existe en tipos decimales: un *array* de enteros no puede contener `nan`. Para datos con huecos, use máscaras (`np.ma`) o pandas.

---

## 17. Fechas, texto y *arrays* estructurados

```python
dias = np.arange("2026-01-01", "2026-02-01", dtype="datetime64[D]")
np.busday_count("2026-01-01", "2026-02-01")                 # Días laborables
np.busday_offset("2026-11-20", 5, roll="forward")           # 5 días laborables después
(np.datetime64("2026-12-25") - np.datetime64("2026-11-25")).astype(int)   # 30

np.char.upper(np.array(["a", "b"])) ; np.strings.str_len(np.array(["hola"]))   # Operaciones de texto
np.array(["corto", "texto más largo"], dtype=np.dtypes.StringDType())          # Texto de longitud variable

tipo = np.dtype([("nombre", "U20"), ("edad", "i4"), ("altura", "f8")])
personas = np.array([("Ana", 34, 1.68), ("Luis", 41, 1.80)], dtype=tipo)
personas["edad"].mean() ; personas[personas["altura"] > 1.7]["nombre"]
```

> Para datos con columnas heterogéneas, pandas es casi siempre más práctico que los *arrays* estructurados.

---

## 18. Entrada y salida

```python
np.save("matriz.npy", m) ; m = np.load("matriz.npy")                 # Formato binario nativo
np.savez_compressed("datos.npz", x=x, y=y) ; d = np.load("datos.npz"); d["x"]
np.savetxt("datos.csv", m, delimiter=";", fmt="%.3f", header="a;b;c", comments="")
m = np.loadtxt("datos.csv", delimiter=";", skiprows=1)
m = np.genfromtxt("datos.csv", delimiter=";", names=True, missing_values="", filling_values=np.nan)
grande = np.load("enorme.npy", mmap_mode="r")                          # Sin cargarlo entero en memoria
```

---

## 19. Rendimiento

```python
import timeit
x = np.random.default_rng(0).random(1_000_000)

timeit.timeit(lambda: sum(v * v for v in x), number=1)     # Bucle de Python: ~0,2 s
timeit.timeit(lambda: (x * x).sum(), number=1)              # Vectorizado: ~0,002 s
timeit.timeit(lambda: x @ x, number=1)                      # BLAS: aún más rápido
```

- Evite bucles de Python sobre elementos: use operaciones vectorizadas, *broadcasting* y `np.where`.
- Prefiera operaciones en el sitio (`a += 1`, `out=`) con *arrays* grandes para ahorrar memoria.
- Use `float32` si la precisión lo permite (mitad de memoria).
- Evite `np.append`/`np.concatenate` repetidos dentro de bucles.
- Para bucles inevitables y complejos: **Numba** (`@njit`) o Cython.

---

## 20. Cambios importantes en NumPy 2

| Antes (1.x) | NumPy 2 |
|---|---|
| `np.float_`, `np.complex_`, `np.NaN`, `np.Inf` | `np.float64`, `np.complex128`, `np.nan`, `np.inf` |
| `np.trapz` | `np.trapezoid` |
| `np.in1d` | `np.isin` |
| `np.row_stack` | `np.vstack` |
| `np.product`, `np.cumproduct`, `np.alltrue`, `np.sometrue`, `np.round_` | `np.prod`, `np.cumprod`, `np.all`, `np.any`, `np.round` |
| Promoción de tipos dependiente del valor | Reglas NEP 50: `np.float32(3) + 3.0` da `float32` |
| `np.array(x, copy=False)` copiaba si era necesario | Ahora lanza error si no puede evitar la copia; use `np.asarray(x)` |

Nuevo: tipo de texto de longitud variable `StringDType`, funciones `np.strings.*`, `np.unique_counts`, `np.unique_values`, `.mT` (traspuesta de matrices apiladas) y compatibilidad con el estándar *Array API*.

> Para adaptar código antiguo, la herramienta **Ruff** incluye la regla `NPY201`, que detecta y corrige automáticamente la mayoría de usos eliminados.

---

## 21. Ejemplo práctico completo

**Análisis numérico de un proyecto con simulación Monte Carlo**: estima la duración y el coste de un proyecto con tareas inciertas (distribuciones triangulares), dependencias (camino crítico calculado de forma vectorizada para 100 000 escenarios), riesgos aleatorios, análisis de sensibilidad por correlación, ajuste por mínimos cuadrados de una curva de aprendizaje, resolución de un sistema lineal para repartir el presupuesto, y comparación de rendimiento entre bucle y versión vectorizada.

```python
"""
montecarlo_proyecto.py — Simulación Monte Carlo y cálculo numérico con NumPy.
Requiere: pip install numpy
Uso: python montecarlo_proyecto.py [--escenarios 100000] [--semilla 7]
"""
import argparse
import time

import numpy as np

np.set_printoptions(precision=2, suppress=True)

# Tareas: (nombre, optimista, más probable, pesimista) en días; coste diario por tarea en €
TAREAS = [
    ("Análisis", 5, 8, 15),
    ("Diseño", 6, 10, 18),
    ("Backend", 15, 22, 40),
    ("Frontend", 12, 18, 30),
    ("Integración", 4, 6, 12),
    ("Pruebas", 5, 8, 16),
    ("Despliegue", 1, 2, 5),
]
COSTE_DIA = np.array([550, 600, 700, 650, 700, 500, 800])
# Dependencias: cada tarea empieza cuando terminan todas sus predecesoras (índices)
PREDECESORAS = {0: [], 1: [0], 2: [1], 3: [1], 4: [2, 3], 5: [4], 6: [5]}
# Riesgos: (probabilidad, retraso_min, retraso_max, tarea afectada)
RIESGOS = [(0.25, 3, 10, 2), (0.15, 2, 6, 3), (0.10, 5, 15, 4)]


def simular(n: int, rng: np.random.Generator) -> dict[str, np.ndarray]:
    nombres = [t[0] for t in TAREAS]
    minimos, modas, maximos = (np.array(c, dtype=float) for c in zip(*(t[1:] for t in TAREAS)))

    # Duraciones: matriz (n escenarios × m tareas), sin bucles por escenario
    duraciones = rng.triangular(minimos, modas, maximos, size=(n, len(TAREAS)))

    # Riesgos: se activan con su probabilidad y añaden un retraso uniforme a una tarea
    for prob, rmin, rmax, tarea in RIESGOS:
        activo = rng.random(n) < prob
        duraciones[:, tarea] += np.where(activo, rng.uniform(rmin, rmax, n), 0.0)

    # Camino crítico vectorizado: fin[t] = max(fin de predecesoras) + duración[t]
    fin = np.zeros_like(duraciones)
    for t in range(len(TAREAS)):                      # Bucle sobre 7 tareas, no sobre 100 000 escenarios
        inicio = fin[:, PREDECESORAS[t]].max(axis=1) if PREDECESORAS[t] else 0.0
        fin[:, t] = inicio + duraciones[:, t]
    duracion_total = fin[:, -1]

    coste = duraciones @ COSTE_DIA                    # Producto matricial: coste de cada escenario
    critica_backend = fin[:, 2] >= fin[:, 3]          # ¿El backend determina el camino crítico?
    return {"nombres": np.array(nombres), "duraciones": duraciones, "duracion_total": duracion_total,
            "coste": coste, "critica_backend": critica_backend}


def informe_simulacion(r: dict[str, np.ndarray], plazo: float, presupuesto: float) -> None:
    d, c = r["duracion_total"], r["coste"]
    p = np.percentile(d, [10, 50, 80, 90, 95])
    pc = np.percentile(c, [10, 50, 80, 90, 95])
    print(f"Escenarios simulados: {d.size:,}".replace(",", "."))
    print(f"Duración (días)  media {d.mean():6.1f} · σ {d.std(ddof=1):5.1f} · "
          f"P10 {p[0]:.0f} · P50 {p[1]:.0f} · P80 {p[2]:.0f} · P90 {p[3]:.0f} · P95 {p[4]:.0f}")
    print(f"Coste (k€)       media {c.mean() / 1000:6.1f} · σ {c.std(ddof=1) / 1000:5.1f} · "
          f"P50 {pc[1] / 1000:.1f} · P80 {pc[2] / 1000:.1f} · P95 {pc[4] / 1000:.1f}")
    print(f"Probabilidad de cumplir el plazo de {plazo:.0f} días: {np.mean(d <= plazo):.1%}")
    print(f"Probabilidad de no superar {presupuesto / 1000:.0f} k€: {np.mean(c <= presupuesto):.1%}")
    print(f"Ambas cosas a la vez: {np.mean((d <= plazo) & (c <= presupuesto)):.1%}")
    print(f"El backend está en el camino crítico en el {r['critica_backend'].mean():.1%} de los escenarios")

    # Histograma textual de la duración
    conteos, bordes = np.histogram(d, bins=12)
    escala = 40 / conteos.max()
    print("\nDistribución de la duración total:")
    for n, a, b in zip(conteos, bordes[:-1], bordes[1:]):
        print(f"  {a:5.1f}–{b:5.1f} │{'█' * int(n * escala):<40}│ {n / d.size:5.1%}")

    # Sensibilidad: correlación de cada tarea con la duración total
    corr = np.array([np.corrcoef(r["duraciones"][:, i], d)[0, 1] for i in range(len(TAREAS))])
    orden = np.argsort(-corr)
    print("\nSensibilidad (correlación con la duración total):")
    for i in orden:
        print(f"  {r['nombres'][i]:<12} {corr[i]:5.2f} {'▇' * int(max(corr[i], 0) * 30)}")


def curva_aprendizaje(rng: np.random.Generator) -> None:
    """Ajuste por mínimos cuadrados de t(n) = a · n^b (modelo de Wright) linealizando con logaritmos."""
    n = np.arange(1, 21)
    real_a, real_b = 40.0, np.log2(0.85)               # Curva del 85 %
    t = real_a * n ** real_b * rng.lognormal(0, 0.05, n.size)
    X = np.column_stack([np.ones(n.size), np.log(n)])
    (log_a, b), residuo, *_ = np.linalg.lstsq(X, np.log(t))
    a = np.exp(log_a)
    tasa = 2 ** b
    prediccion = a * np.array([30, 50]) ** b
    print(f"\nCurva de aprendizaje ajustada: t(n) = {a:.1f}·n^{b:.3f} (tasa de aprendizaje {tasa:.1%})")
    print(f"  Horas estimadas para la unidad 30: {prediccion[0]:.1f} · unidad 50: {prediccion[1]:.1f}")


def reparto_presupuesto() -> None:
    """Sistema lineal: reparto entre personal, infraestructura y licencias con restricciones."""
    # x + y + z = 120 000 ; x = 3·(y + z) ; y − 2·z = 0
    A = np.array([[1, 1, 1], [1, -3, -3], [0, 1, -2]], dtype=float)
    b = np.array([120_000, 0, 0], dtype=float)
    x = np.linalg.solve(A, b)
    assert np.allclose(A @ x, b)
    print(f"\nReparto del presupuesto: personal {x[0]:,.0f} € · infraestructura {x[1]:,.0f} € · "
          f"licencias {x[2]:,.0f} €".replace(",", "."))


def comparar_rendimiento(rng: np.random.Generator, n: int = 200_000) -> None:
    a, b = rng.random(n), rng.random(n)
    inicio = time.perf_counter()
    lento = sum(x * y for x, y in zip(a, b))
    t_bucle = time.perf_counter() - inicio
    inicio = time.perf_counter()
    rapido = a @ b
    t_vector = time.perf_counter() - inicio
    assert np.isclose(lento, rapido)
    print(f"\nProducto escalar de {n:,} elementos: bucle {t_bucle * 1000:.1f} ms · "
          f"NumPy {t_vector * 1000:.3f} ms (×{t_bucle / max(t_vector, 1e-9):,.0f})".replace(",", "."))


def main() -> int:
    p = argparse.ArgumentParser(description="Simulación Monte Carlo de un proyecto con NumPy.")
    p.add_argument("--escenarios", type=int, default=100_000)
    p.add_argument("--semilla", type=int, default=7)
    p.add_argument("--plazo", type=float, default=70)
    p.add_argument("--presupuesto", type=float, default=58_000)
    args = p.parse_args()

    rng = np.random.default_rng(args.semilla)
    inicio = time.perf_counter()
    resultado = simular(args.escenarios, rng)
    print(f"Simulación completada en {time.perf_counter() - inicio:.2f} s\n")
    informe_simulacion(resultado, args.plazo, args.presupuesto)
    curva_aprendizaje(rng)
    reparto_presupuesto()
    comparar_rendimiento(rng)
    np.savez_compressed("simulacion.npz", **{k: v for k, v in resultado.items() if k != "nombres"})
    print("\n✔ Resultados guardados en simulacion.npz")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 22. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ValueError: operands could not be broadcast together with shapes (3,4) (3,)` | Formas incompatibles. | Añada una dimensión: `v[:, None]`. |
| `ValueError: The truth value of an array with more than one element is ambiguous` | `if array:` o `and`/`or` con *arrays*. | `.any()`, `.all()`, `&`, `|`. |
| `IndexError: index 5 is out of bounds` | Índice fuera de rango. | Revise `shape`. |
| `ValueError: cannot reshape array of size 10 into shape (3,4)` | El tamaño no coincide. | El producto de la nueva forma debe ser igual a `size`. |
| Resultados inesperados con enteros pequeños | Desbordamiento (`uint8`, `int16`). | `astype` a un tipo mayor antes de operar. |
| El *array* original cambia «solo» | Se modificó una vista. | Use `.copy()`. |
| `a * b` no da el producto matricial | `*` es elemento a elemento. | `a @ b`. |
| `AttributeError: module 'numpy' has no attribute 'float_'` (o `NaN`, `trapz`) | Eliminado en NumPy 2. | `np.float64`, `np.nan`, `np.trapezoid`. |
| `RuntimeWarning: invalid value encountered` / `divide by zero` | Operaciones con `nan`, 0 o logaritmos de negativos. | Valide los datos; `np.errstate`; funciones `nan*`. |
| Comparaciones de decimales que fallan | Errores de redondeo. | `np.isclose`, `np.allclose`. |
| Resultados aleatorios distintos en cada ejecución | Sin semilla. | `np.random.default_rng(semilla)`. |

---

## 23. Buenas prácticas

- Piense en operaciones sobre *arrays* completos, no en bucles sobre elementos.
- Use `np.random.default_rng(semilla)` y pase el generador a las funciones.
- Indique `axis=` explícitamente en las agregaciones.
- Elija el `dtype` adecuado y vigile desbordamientos y precisión.
- Use `@` para productos matriciales y `np.linalg.solve` en lugar de calcular inversas.
- Compare decimales con `np.isclose`/`np.allclose`.
- Recuerde qué operaciones devuelven vistas y use `.copy()` cuando necesite independencia.
- Documente las formas esperadas de los *arrays* (`# (n, 3)`) en funciones y comentarios.
- Para datos etiquetados o heterogéneos, pase a pandas.

---

## 24. Referencia rápida

```python
np.array · zeros · ones · full · eye · arange · linspace · meshgrid · *_like
a.shape · ndim · size · dtype · astype · reshape(-1, n) · ravel · T · copy()
a[i, j] · a[i:j, ::2] · a[a > 0] · a[[0, 2]] · np.ix_ · a[:, None]
+ - * / ** @ · np.sqrt exp log sin abs round clip maximum where select isin unique
a.sum(axis=0) · mean · std(ddof=1) · min · max · argmax · cumsum · median · percentile · quantile
concatenate · vstack · hstack · column_stack · stack · split · sort · argsort · partition · searchsorted · digitize
np.linalg.solve · inv · det · eig · svd · norm · lstsq · polyfit · polyval
rng = np.random.default_rng(42); rng.random · integers · normal · uniform · choice · triangular · permutation
np.isnan · nanmean · nan_to_num · isclose · allclose · errstate
np.save/load · savez_compressed · savetxt · loadtxt · genfromtxt · mmap_mode="r"
```

---

## 25. Ejercicios propuestos

1. Genere 1 000 000 de lanzamientos de dos dados y calcule la frecuencia de cada suma; compárela con la probabilidad teórica.
2. Implemente la normalización *min-max* y *z-score* de cada columna de una matriz usando *broadcasting*.
3. Calcule la matriz de distancias entre 500 puntos aleatorios en 2D sin bucles y encuentre el par más cercano.
4. Ajuste un polinomio de grado 3 a datos con ruido, calcule el error cuadrático medio y compárelo con grados 1 y 5.
5. Amplíe el ejemplo completo con un análisis de escenarios: ¿cuántos días se ganarían si se redujera un 20 % la duración pesimista del backend?

---

## 26. Referencias

- Documentación oficial: https://numpy.org/doc/stable/
- Guía para principiantes: https://numpy.org/doc/stable/user/absolute_beginners.html
- Guía de migración a NumPy 2: https://numpy.org/doc/stable/numpy_2_0_migration_guide.html
- Generación de números aleatorios: https://numpy.org/doc/stable/reference/random/index.html
- *Broadcasting*: https://numpy.org/doc/stable/user/basics.broadcasting.html
