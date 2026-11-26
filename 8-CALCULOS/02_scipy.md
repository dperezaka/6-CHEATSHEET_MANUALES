# Manual de uso: SciPy

> **Categoría:** Matemáticas y cálculo · **Utilidad:** Métodos científicos y numéricos
> **Tipo:** Librería externa (requiere instalación con `pip`). Construida sobre NumPy.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Constantes y funciones especiales](#4-constantes-y-funciones-especiales)
5. [Integración y ecuaciones diferenciales (`integrate`)](#5-integración-y-ecuaciones-diferenciales-integrate)
6. [Optimización y raíces (`optimize`)](#6-optimización-y-raíces-optimize)
7. [Ajuste de curvas](#7-ajuste-de-curvas)
8. [Interpolación (`interpolate`)](#8-interpolación-interpolate)
9. [Álgebra lineal (`linalg`) y matrices dispersas (`sparse`)](#9-álgebra-lineal-linalg-y-matrices-dispersas-sparse)
10. [Estadística (`stats`)](#10-estadística-stats)
11. [Procesamiento de señales (`signal`) y FFT (`fft`)](#11-procesamiento-de-señales-signal-y-fft-fft)
12. [Geometría y datos espaciales (`spatial`)](#12-geometría-y-datos-espaciales-spatial)
13. [Imágenes (`ndimage`) y agrupamiento (`cluster`)](#13-imágenes-ndimage-y-agrupamiento-cluster)
14. [Entrada y salida (`io`)](#14-entrada-y-salida-io)
15. [Rendimiento y precisión numérica](#15-rendimiento-y-precisión-numérica)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**SciPy** (*Scientific Python*) es la librería de referencia para **cálculo científico y numérico** en Python. Sobre los arreglos de **NumPy** ofrece algoritmos probados y eficientes (la mayoría escritos en C, C++ y Fortran) para integración, ecuaciones diferenciales, optimización, ajuste de curvas, interpolación, álgebra lineal, estadística, procesamiento de señales e imágenes, geometría computacional y transformadas de Fourier.

**Ventajas:**
- Algoritmos maduros, validados y rápidos (BLAS/LAPACK, QUADPACK, ODEPACK, HiGHS, FFTW-like…).
- Cobertura muy amplia de métodos científicos en una sola librería.
- Integración natural con NumPy, pandas, Matplotlib y scikit-learn.
- Estándar de facto en ciencia, ingeniería y análisis de datos.
- Licencia BSD.

**Limitaciones:**
- Trabaja con aproximaciones numéricas (no fórmulas exactas: para eso, SymPy).
- Sin diferenciación automática ni ejecución en GPU (para eso, JAX).
- Algunas funciones requieren entender bien los parámetros (tolerancias, métodos, condiciones iniciales) para obtener resultados fiables.

| Si necesita… | Opción |
|---|---|
| Integrar, optimizar, ajustar, interpolar, estadística, señales | **SciPy** |
| Fórmulas exactas y álgebra simbólica | SymPy |
| Acelerar bucles Python propios | Numba |
| Gradientes automáticos, GPU | JAX |
| Modelos estadísticos completos (regresión, series temporales) | statsmodels |
| Aprendizaje automático | scikit-learn |

---

## 2. Instalación e importación

```bash
pip install scipy            # Instala también NumPy
```

Comprobar la versión:

```bash
python -c "import scipy; print(scipy.__version__)"
```

SciPy se organiza en **submódulos** que se importan por separado:

```python
import numpy as np
from scipy import integrate, optimize, interpolate, linalg, stats, signal, fft, spatial, special, constants
```

| Submódulo | Contenido |
|---|---|
| `scipy.constants` | Constantes físicas y unidades. |
| `scipy.special` | Funciones especiales (gamma, Bessel, error…). |
| `scipy.integrate` | Integrales y ecuaciones diferenciales. |
| `scipy.optimize` | Minimización, raíces, ajuste, programación lineal. |
| `scipy.interpolate` | Interpolación y *splines*. |
| `scipy.linalg` | Álgebra lineal (más completa que `numpy.linalg`). |
| `scipy.sparse` | Matrices dispersas y sus solucionadores. |
| `scipy.stats` | Distribuciones, pruebas estadísticas, estadística descriptiva. |
| `scipy.signal` | Filtros, convolución, espectros, detección de picos. |
| `scipy.fft` | Transformadas de Fourier. |
| `scipy.spatial` | Distancias, árboles KD, envolventes convexas, triangulaciones. |
| `scipy.ndimage` | Procesamiento de imágenes n-dimensionales. |
| `scipy.cluster` | Agrupamiento jerárquico y k-medias. |
| `scipy.io` | Archivos MATLAB, WAV, Matrix Market, NetCDF… |

> No basta con `import scipy`: importe el submódulo que necesite (`from scipy import optimize`).

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Arreglo NumPy | Estructura de datos con la que trabaja SciPy (`np.ndarray`). |
| Función objetivo | Función Python que SciPy evalúa repetidamente (`f(x)` → número o arreglo). |
| Vectorización | Evaluar sobre arreglos completos en lugar de bucles: mucho más rápido. |
| Tolerancias (`rtol`, `atol`, `xtol`) | Precisión exigida a un método iterativo. |
| Objeto resultado | Muchas funciones devuelven un objeto con atributos: `x`, `fun`, `success`, `message`, `nit`… |
| Condición inicial / estimación inicial | Punto de partida de métodos iterativos (`x0`, `p0`, `y0`). |
| Distribución «congelada» | Distribución estadística con parámetros fijados: `stats.norm(loc=10, scale=2)`. |

**Patrón habitual:**

```python
resultado = optimize.minimize(funcion, x0=[0, 0])
if not resultado.success:
    raise RuntimeError(resultado.message)
print(resultado.x, resultado.fun)
```

> Compruebe siempre `success` (o el estado equivalente) y revise el mensaje: un método numérico puede terminar sin converger.

---

## 4. Constantes y funciones especiales

### 4.1 Constantes (`scipy.constants`)

```python
from scipy import constants as cte

cte.c          # 299792458.0     velocidad de la luz (m/s)
cte.g          # 9.80665         gravedad estándar (m/s²)
cte.h          # 6.62607015e-34  constante de Planck (J·s)
cte.k          # 1.380649e-23    constante de Boltzmann (J/K)
cte.N_A        # 6.02214076e+23  número de Avogadro (1/mol)
cte.R, cte.G, cte.e, cte.epsilon_0, cte.m_e
cte.physical_constants["electron mass"]         # (9.1093837139e-31, 'kg', 2.8e-40)  valor, unidad, incertidumbre
cte.find("Boltzmann")                           # Buscar constantes por nombre
cte.kilo, cte.milli, cte.minute, cte.hour, cte.inch, cte.atm, cte.bar, cte.calorie
cte.convert_temperature(25, "Celsius", "Kelvin")   # 298.15
```

### 4.2 Funciones especiales (`scipy.special`)

```python
from scipy import special

special.gamma(5)                   # 24.0  (= 4!)
special.factorial(5)               # 120.0
special.comb(10, 3, exact=True)    # 120   (combinaciones, entero exacto)
special.binom(10, 3)               # 120.0
special.erf(1)                     # 0.8427…  función de error
special.jv(0, 1.0)                 # 0.7652…  Bessel de primera especie
special.expit(0), special.logit(0.5)     # Sigmoide y su inversa: 0.5, 0.0
special.logsumexp([1000, 1000])          # 1000.69…  sin desbordamiento
special.zeta(2)                          # 1.6449…  = π²/6
special.ellipk(0.5), special.legendre(3), special.lambertw(1)
```

> Las funciones de `special` son *ufuncs*: aceptan arreglos y se aplican elemento a elemento.

---

## 5. Integración y ecuaciones diferenciales (`integrate`)

### 5.1 Integrales de funciones

```python
from scipy import integrate

valor, error = integrate.quad(lambda x: np.exp(-x**2), -np.inf, np.inf)   # (1.7724…, 1.4e-08) = √π
integrate.quad(lambda x, a: a * x**2, 0, 1, args=(3,))                      # Parámetros adicionales
integrate.dblquad(lambda y, x: x * y, 0, 1, 0, 2)                           # ¡Orden (y, x)! → (1.0, …)
integrate.tplquad(lambda z, y, x: x + y + z, 0, 1, 0, 1, 0, 1)
integrate.nquad(lambda x, y: x * y, [[0, 1], [0, 2]])                       # n dimensiones
integrate.quad_vec(lambda x: np.array([x, x**2]), 0, 1)                     # Funciones vectoriales
```

| Parámetro de `quad` | Uso |
|---|---|
| `args` | Parámetros adicionales de la función. |
| `epsabs`, `epsrel` | Tolerancias absoluta y relativa. |
| `limit` | Máximo de subintervalos (súbalo si avisa de que no converge). |
| `points` | Puntos problemáticos (discontinuidades) en el intervalo. |
| `weight`, `wvar` | Integrales oscilantes o con singularidades conocidas. |

### 5.2 Integrales de datos muestreados

```python
x = np.linspace(0, np.pi, 101)
y = np.sin(x)
integrate.simpson(y, x=x)                 # 2.0000000108  (regla de Simpson)
integrate.trapezoid(y, x)                 # 1.9998355     (regla del trapecio)
integrate.cumulative_trapezoid(y, x, initial=0)    # Integral acumulada (p. ej., velocidad → posición)
```

> En versiones recientes, `simps`, `trapz` y `cumtrapz` se han eliminado: use `simpson`, `trapezoid` y `cumulative_trapezoid`.

### 5.3 Ecuaciones diferenciales ordinarias (`solve_ivp`)

Toda EDO se expresa como sistema de primer orden `dy/dt = f(t, y)`:

```python
def oscilador(t, y, c, k):
    x, v = y                              # y[0] = posición, y[1] = velocidad
    return [v, -c * v - k * x]            # [dx/dt, dv/dt]

sol = integrate.solve_ivp(oscilador, t_span=(0, 10), y0=[1.0, 0.0], args=(0.5, 4.0),
                          t_eval=np.linspace(0, 10, 500), rtol=1e-8, atol=1e-10)
sol.t, sol.y[0], sol.y[1]                 # Tiempos, posición, velocidad
sol.success, sol.message, sol.nfev
```

| Método (`method=`) | Uso |
|---|---|
| `"RK45"` (por defecto) | Problemas generales no rígidos. |
| `"DOP853"` | Alta precisión. |
| `"Radau"`, `"BDF"`, `"LSODA"` | Problemas **rígidos** (escalas de tiempo muy diferentes: cinética química, circuitos). |

Eventos (detener la integración cuando ocurre algo):

```python
def toca_suelo(t, y):
    return y[0]                           # Se detecta el cruce por cero de la altura
toca_suelo.terminal = True                # Detener la integración
toca_suelo.direction = -1                 # Solo al bajar

sol = integrate.solve_ivp(lambda t, y: [y[1], -9.81], (0, 10), [10, 0], events=toca_suelo, dense_output=True)
sol.t_events[0]                           # Instante del impacto: [1.4278…]
sol.sol(0.5)                              # Solución continua en cualquier t (dense_output=True)
```

Problemas de contorno: `integrate.solve_bvp(f, condiciones, x, y_inicial)`.

---

## 6. Optimización y raíces (`optimize`)

### 6.1 Minimización

```python
from scipy import optimize

def f(p):
    x, y = p
    return (x - 1)**2 + 10 * (y - x**2)**2

res = optimize.minimize(f, x0=[0, 0])                                  # BFGS por defecto (sin restricciones)
res.x, res.fun, res.success, res.nit

optimize.minimize(f, [0, 0], method="Nelder-Mead")                     # Sin derivadas
optimize.minimize(f, [0, 0], method="L-BFGS-B", bounds=[(0, 2), (0, 2)])   # Con límites
optimize.minimize(f, [0, 0], jac=gradiente)                            # Con gradiente analítico (más rápido)
optimize.minimize_scalar(lambda x: (x - 2)**2, bounds=(0, 5), method="bounded")   # Una variable
optimize.minimize(optimize.rosen, [1.3, 0.7, 0.8, 1.9, 1.2], method="BFGS").x     # [1. 1. 1. 1. 1.]
```

Con restricciones:

```python
restricciones = [
    {"type": "ineq", "fun": lambda p: 10 - p[0] - 2 * p[1]},     # g(p) ≥ 0
    {"type": "eq", "fun": lambda p: p[0] - p[1] - 1},             # h(p) = 0
]
optimize.minimize(f, [0, 0], method="SLSQP", constraints=restricciones, bounds=[(0, None)] * 2)
```

| Método | Derivadas | Restricciones | Uso |
|---|---|---|---|
| `Nelder-Mead`, `Powell` | No | Límites (Powell) | Funciones ruidosas o no derivables. |
| `BFGS`, `CG` | Opcional | No | Funciones suaves. |
| `L-BFGS-B` | Opcional | Límites | Muchas variables con límites. |
| `SLSQP` | Opcional | Igualdad, desigualdad, límites | Problemas restringidos pequeños/medios. |
| `trust-constr` | Opcional | Todas | Problemas restringidos grandes. |

### 6.2 Optimización global

```python
limites = [(-5, 5), (-5, 5)]
optimize.differential_evolution(f, limites, seed=1)         # Evolutiva, robusta
optimize.dual_annealing(f, limites, seed=1)                 # Recocido simulado
optimize.shgo(f, limites)                                    # Homología simplicial
optimize.basinhopping(f, x0=[0, 0], niter=100)
```

### 6.3 Raíces de ecuaciones

```python
optimize.brentq(lambda x: x**3 - 2, 0, 2)                         # 1.2599…  (requiere cambio de signo)
optimize.root_scalar(lambda x: np.cos(x) - x, bracket=[0, 1]).root   # 0.7390851…
optimize.root_scalar(f, x0=1, fprime=df, method="newton")
optimize.fsolve(lambda p: [p[0] + p[1] - 3, p[0] * p[1] - 2], [0, 0])   # Sistemas
optimize.root(sistema, x0=[1, 1], method="hybr")
```

### 6.4 Programación lineal y entera

```python
# Maximizar 3x + 5y  ⇒  minimizar -3x - 5y
res = optimize.linprog(c=[-3, -5],
                       A_ub=[[1, 0], [0, 2], [3, 2]], b_ub=[4, 12, 18],
                       bounds=[(0, None), (0, None)])
res.x, -res.fun                           # [2. 6.], 36.0

# Programación lineal entera mixta
from scipy.optimize import milp, LinearConstraint, Bounds
res = milp(c=[-3, -5], constraints=LinearConstraint([[1, 0], [0, 2], [3, 2]], ub=[4, 12, 18]),
           integrality=[1, 1], bounds=Bounds(0, np.inf))
```

Problemas de asignación:

```python
filas, columnas = optimize.linear_sum_assignment(matriz_costes)     # Asignación óptima (algoritmo húngaro)
```

---

## 7. Ajuste de curvas

### 7.1 `curve_fit`: mínimos cuadrados no lineales

```python
def modelo(t, A, k, C):
    return A * np.exp(-k * t) + C

t = np.linspace(0, 10, 50)
y = modelo(t, 5, 0.6, 1) + np.random.default_rng(0).normal(0, 0.1, t.size)

parametros, covarianza = optimize.curve_fit(modelo, t, y, p0=[4, 0.5, 0], bounds=([0, 0, -5], [20, 5, 5]))
errores = np.sqrt(np.diag(covarianza))        # Desviación típica de cada parámetro
residuos = y - modelo(t, *parametros)
r2 = 1 - np.sum(residuos**2) / np.sum((y - y.mean())**2)
```

| Parámetro | Uso |
|---|---|
| `p0` | Estimación inicial (**muy importante** en modelos no lineales). |
| `bounds` | Límites de los parámetros. |
| `sigma`, `absolute_sigma` | Incertidumbre de cada medida (ajuste ponderado). |
| `maxfev` | Máximo de evaluaciones. |

### 7.2 `least_squares`: control avanzado y ajuste robusto

```python
def residuos(p, t, y):
    return modelo(t, *p) - y

res = optimize.least_squares(residuos, x0=[4, 0.5, 0], args=(t, y), loss="soft_l1")   # Robusto a valores atípicos
res.x
```

### 7.3 Ajustes lineales y polinómicos

```python
np.polyfit(x, y, deg=2)                               # Polinomio (NumPy)
from scipy import stats
stats.linregress(x, y)                                 # Recta con pendiente, ordenada, r, p, errores típicos
```

---

## 8. Interpolación (`interpolate`)

```python
from scipy import interpolate

x = np.array([0, 1, 2, 3, 4, 5])
y = np.array([0, 0.8, 0.9, 0.1, -0.8, -1.0])

spline = interpolate.CubicSpline(x, y)                    # Spline cúbico (curva suave)
spline(2.5), spline(2.5, 1)                               # Valor y primera derivada
spline.integrate(0, 5)
akima = interpolate.Akima1DInterpolator(x, y)             # Sin oscilaciones excesivas
pchip = interpolate.PchipInterpolator(x, y)               # Monótona: respeta subidas y bajadas
lineal = np.interp(2.5, x, y)                             # Lineal (NumPy, la más rápida)
bspline = interpolate.make_interp_spline(x, y, k=3)
suavizado = interpolate.make_smoothing_spline(x, y, lam=0.1)     # Suaviza datos con ruido
```

Datos en varias dimensiones:

```python
# Rejilla regular
interp = interpolate.RegularGridInterpolator((xs, ys), valores_2d, method="cubic")
interp([[1.5, 2.3], [0.2, 0.7]])

# Puntos dispersos
interpolate.griddata(puntos, valores, (rejilla_x, rejilla_y), method="cubic")
interpolate.RBFInterpolator(puntos, valores, kernel="thin_plate_spline")(nuevos_puntos)
```

> `interp2d` se eliminó: use `RegularGridInterpolator` (rejillas) o `griddata`/`RBFInterpolator` (puntos dispersos). `interp1d` sigue disponible, pero se considera heredada: prefiera `CubicSpline`, `make_interp_spline` o `np.interp`.

| Método | Cuándo |
|---|---|
| Lineal (`np.interp`) | Rapidez; datos densos. |
| `CubicSpline` | Curva suave con derivadas continuas. |
| `PchipInterpolator` | Datos monótonos (no inventa máximos/mínimos). |
| `make_smoothing_spline` | Datos con ruido (no pasa exactamente por los puntos). |

---

## 9. Álgebra lineal (`linalg`) y matrices dispersas (`sparse`)

### 9.1 `scipy.linalg`

```python
from scipy import linalg

A = np.array([[3, 2], [1, 2]])
b = np.array([2, 0])
linalg.solve(A, b)                         # [ 1.  -0.5]
linalg.det([[1, 2], [3, 4]])               # -2.0
linalg.inv(A)
valores, vectores = linalg.eig(A)
linalg.eigh([[2, 1], [1, 2]])              # Matrices simétricas: (array([1., 3.]), vectores)
linalg.svd(A), linalg.qr(A), linalg.lu(A), linalg.cholesky(A @ A.T)
linalg.norm(A), linalg.pinv(A), linalg.expm(A), linalg.null_space(A)
linalg.lstsq(A_rectangular, b)              # Mínimos cuadrados
linalg.solve(A, b, assume_a="pos")          # Más rápido si se conoce la estructura (simétrica definida positiva)
linalg.solve_banded(...), linalg.solve_triangular(...), linalg.toeplitz([1, 2, 3])
```

> `scipy.linalg` incluye todo `numpy.linalg` y más (exponencial de matrices, descomposiciones adicionales, matrices especiales). Use `solve` en lugar de `inv(A) @ b`: es más rápido y preciso.

### 9.2 Matrices dispersas

Para matrices grandes con mayoría de ceros (redes, elementos finitos, sistemas de ecuaciones grandes):

```python
from scipy import sparse
from scipy.sparse import linalg as splinalg

n = 100_000
diagonales = [np.full(n, -1.0), np.full(n, 2.0), np.full(n, -1.0)]
A = sparse.diags_array(diagonales, offsets=[-1, 0, 1], format="csr")     # Tridiagonal
b = np.ones(n)
x = splinalg.spsolve(A, b)                 # Solucionador directo
x, info = splinalg.cg(A, b, rtol=1e-8)     # Gradiente conjugado (iterativo)
splinalg.eigsh(A, k=5, which="SM")         # 5 valores propios más pequeños

M = sparse.coo_array((valores, (filas, columnas)), shape=(n, n)).tocsr()   # Construcción por coordenadas
A.nnz, A.toarray()                          # Elementos no nulos; convertir a densa (¡solo si es pequeña!)
```

| Formato | Uso |
|---|---|
| `coo` | Construir a partir de (fila, columna, valor). |
| `lil`, `dok` | Modificar elementos uno a uno. |
| `csr` | Operaciones aritméticas y productos por filas (el más usado). |
| `csc` | Operaciones por columnas, factorizaciones. |

> SciPy recomienda las clases `*_array` (`csr_array`, `coo_array`, `diags_array`) frente a las antiguas `*_matrix`: `*` es producto elemento a elemento y `@` producto matricial, igual que en NumPy.

Grafos: `scipy.sparse.csgraph` (caminos mínimos, componentes conexas, árbol de expansión mínima):

```python
from scipy.sparse.csgraph import shortest_path, connected_components, minimum_spanning_tree
distancias, predecesores = shortest_path(grafo, directed=False, return_predecessors=True)
```

---

## 10. Estadística (`stats`)

### 10.1 Distribuciones

```python
from scipy import stats

stats.norm.cdf(1.96)                       # 0.9750  P(Z ≤ 1.96)
stats.norm.ppf(0.975)                      # 1.9600  cuantil (inversa de la cdf)
stats.t.ppf(0.975, df=10)                  # 2.2281
stats.binom.pmf(3, n=10, p=0.5)            # 0.1172  P(X = 3)

normal = stats.norm(loc=170, scale=8)      # Distribución «congelada»: estatura media 170 cm, σ = 8
normal.cdf(180) - normal.cdf(160)          # P(160 < X < 180)
normal.mean(), normal.std(), normal.interval(0.95)
muestras = normal.rvs(size=1000, random_state=42)
parametros = stats.norm.fit(muestras)      # Estimar loc y scale a partir de datos
```

| Método | Significado |
|---|---|
| `pdf` / `pmf` | Densidad / probabilidad puntual. |
| `cdf` / `sf` | Probabilidad acumulada / complementaria (1 − cdf). |
| `ppf` / `isf` | Cuantil / cuantil complementario. |
| `rvs` | Muestras aleatorias. |
| `fit` | Estimar parámetros por máxima verosimilitud. |
| `mean`, `var`, `std`, `interval`, `moment`, `entropy` | Características. |

Distribuciones continuas: `norm`, `t`, `chi2`, `f`, `expon`, `gamma`, `beta`, `lognorm`, `weibull_min`, `uniform`, `cauchy`, `pareto`, `gumbel_r`… Discretas: `binom`, `poisson`, `geom`, `nbinom`, `hypergeom`, `bernoulli`, `randint`.

> Las versiones recientes incluyen una nueva interfaz de distribuciones (`stats.Normal(mu=0, sigma=1)`, `stats.make_distribution`) más rápida y con transformaciones; la interfaz clásica (`stats.norm`) sigue siendo la más documentada.

### 10.2 Estadística descriptiva

```python
datos = np.array([5.1, 4.9, 5.3, 5.0, 5.2])
stats.describe(datos)                      # n, mín/máx, media, varianza, asimetría, curtosis
stats.sem(datos)                           # Error típico de la media
stats.iqr(datos), stats.mode(datos), stats.skew(datos), stats.kurtosis(datos), stats.zscore(datos)
stats.trim_mean(datos, 0.1)                # Media recortada
stats.bootstrap((datos,), np.mean, confidence_level=0.95).confidence_interval
```

Intervalo de confianza de una media:

```python
media, error = datos.mean(), stats.sem(datos)
stats.t.interval(0.95, df=len(datos) - 1, loc=media, scale=error)
```

### 10.3 Pruebas de hipótesis

```python
a = np.array([5.1, 4.9, 5.3, 5.0, 5.2])
b = np.array([4.8, 4.7, 5.0, 4.9, 4.6])
stats.ttest_ind(a, b)                      # statistic=3.0, pvalue=0.0171  (medias de dos grupos)
stats.ttest_ind(a, b, equal_var=False)     # Welch (varianzas distintas)
stats.ttest_rel(antes, despues)            # Muestras emparejadas
stats.ttest_1samp(a, popmean=5)
stats.mannwhitneyu(a, b)                   # No paramétrica
stats.wilcoxon(antes, despues)
stats.f_oneway(g1, g2, g3)                 # ANOVA de un factor
stats.kruskal(g1, g2, g3)
stats.chi2_contingency(tabla)              # Independencia en tablas de contingencia
stats.shapiro(datos)                       # Normalidad
stats.kstest(datos, "norm", args=(media, desviacion))
stats.levene(a, b)                         # Igualdad de varianzas
stats.pearsonr(x, y), stats.spearmanr(x, y), stats.kendalltau(x, y)
stats.false_discovery_control(p_valores)   # Corrección por comparaciones múltiples
```

> Un p-valor pequeño (habitualmente < 0,05) indica evidencia contra la hipótesis nula. Informe también el tamaño del efecto y los intervalos de confianza, no solo el p-valor.

### 10.4 Regresión lineal simple

```python
r = stats.linregress([1, 2, 3, 4, 5], [2, 4, 5, 4, 5])
r.slope, r.intercept, r.rvalue**2, r.pvalue, r.stderr     # 0.6, 2.2, 0.6, 0.124, 0.283
```

---

## 11. Procesamiento de señales (`signal`) y FFT (`fft`)

### 11.1 Filtros digitales

```python
from scipy import signal

fs = 1000                                                    # Frecuencia de muestreo (Hz)
sos = signal.butter(4, 50, btype="lowpass", fs=fs, output="sos")   # Butterworth de orden 4, corte 50 Hz
filtrada = signal.sosfiltfilt(sos, senal)                    # Sin desfase (adelante y atrás)
sos_banda = signal.butter(4, [8, 12], btype="bandpass", fs=fs, output="sos")
b, a = signal.iirnotch(50, Q=30, fs=fs)                      # Eliminar una frecuencia (red eléctrica)
signal.filtfilt(b, a, senal)
signal.savgol_filter(senal, window_length=31, polyorder=3)   # Suavizado conservando picos
signal.medfilt(senal, kernel_size=5)                         # Filtro de mediana (ruido impulsivo)
```

> Use la salida `"sos"` (secciones de segundo orden): es numéricamente más estable que los coeficientes `b, a` en filtros de orden alto.

### 11.2 Análisis espectral y FFT

```python
from scipy import fft

N = len(senal)
espectro = fft.rfft(senal)                       # FFT de señal real
frecuencias = fft.rfftfreq(N, d=1 / fs)          # Frecuencias correspondientes
amplitud = 2 * np.abs(espectro) / N
frecuencia_dominante = frecuencias[np.argmax(amplitud[1:]) + 1]

f, Pxx = signal.welch(senal, fs=fs, nperseg=1024)          # Densidad espectral (más robusta que la FFT directa)
f, t, Sxx = signal.spectrogram(senal, fs=fs)               # Espectrograma
fft.fft2(imagen), fft.fftshift(espectro), fft.ifft(espectro)
```

### 11.3 Picos, convolución y remuestreo

```python
picos, propiedades = signal.find_peaks(senal, height=0.5, distance=50, prominence=0.2)
signal.convolve(a, b, mode="same"), signal.fftconvolve(a, b), signal.correlate(a, b)
signal.resample(senal, 500), signal.decimate(senal, 4)
signal.detrend(senal)                                       # Quitar tendencia lineal
analitica = signal.hilbert(senal); envolvente = np.abs(analitica)
```

### 11.4 Sistemas lineales

```python
sistema = signal.TransferFunction([1], [1, 0.5, 4])        # 1 / (s² + 0.5 s + 4)
t, y = signal.step(sistema)                                # Respuesta al escalón
w, mag, fase = signal.bode(sistema)                        # Diagrama de Bode
```

---

## 12. Geometría y datos espaciales (`spatial`)

```python
from scipy import spatial

spatial.distance.euclidean([0, 0], [3, 4])                 # 5.0
spatial.distance.cdist(A, B, metric="euclidean")           # Matriz de distancias entre dos conjuntos
spatial.distance.pdist(puntos, metric="cityblock")         # Distancias por pares (forma condensada)
spatial.distance.squareform(pdist_resultado)

arbol = spatial.KDTree(puntos)                             # Búsqueda rápida de vecinos
distancia, indice = arbol.query([0.5, 0.5], k=3)           # 3 vecinos más cercanos
arbol.query_ball_point([0.5, 0.5], r=0.1)                  # Todos los puntos en un radio

envolvente = spatial.ConvexHull(puntos); envolvente.volume, envolvente.vertices
tri = spatial.Delaunay(puntos); tri.simplices; tri.find_simplex([0.3, 0.4])
spatial.Voronoi(puntos)
spatial.transform.Rotation.from_euler("xyz", [90, 0, 45], degrees=True).as_matrix()   # Rotaciones 3D
```

Distancia geográfica aproximada (Haversine) con `cdist` y una función propia, o librerías como `geopy` para precisión geodésica.

---

## 13. Imágenes (`ndimage`) y agrupamiento (`cluster`)

### 13.1 `ndimage`

```python
from scipy import ndimage

ndimage.gaussian_filter(imagen, sigma=2)             # Desenfoque
ndimage.median_filter(imagen, size=3)                # Eliminar ruido «sal y pimienta»
ndimage.sobel(imagen, axis=0)                        # Bordes
ndimage.rotate(imagen, 30, reshape=False), ndimage.zoom(imagen, 2), ndimage.shift(imagen, (5, 0))
etiquetas, n = ndimage.label(mascara_binaria)        # Regiones conectadas
ndimage.center_of_mass(imagen, etiquetas, range(1, n + 1))
ndimage.binary_opening(mascara), ndimage.binary_fill_holes(mascara), ndimage.distance_transform_edt(mascara)
```

### 13.2 `cluster`

```python
from scipy.cluster import hierarchy, vq

Z = hierarchy.linkage(datos, method="ward")          # Agrupamiento jerárquico
grupos = hierarchy.fcluster(Z, t=3, criterion="maxclust")
hierarchy.dendrogram(Z)                              # Dibujar con Matplotlib
centroides, etiquetas = vq.kmeans2(datos, 3, seed=1, minit="++")
```

> Para aprendizaje automático general (k-medias avanzado, DBSCAN, clasificación) use scikit-learn.

---

## 14. Entrada y salida (`io`)

```python
from scipy import io

datos = io.loadmat("archivo.mat")                          # MATLAB (hasta v7.2; v7.3 es HDF5: use h5py)
io.savemat("salida.mat", {"matriz": A, "vector": v})
frecuencia, audio = io.wavfile.read("sonido.wav")
io.wavfile.write("salida.wav", 44100, (audio * 32767).astype(np.int16))
io.mmread("matriz.mtx"), io.mmwrite("matriz.mtx", A_dispersa)     # Matrix Market
io.readsav("datos.sav")                                    # IDL
```

---

## 15. Rendimiento y precisión numérica

| Recomendación | Motivo |
|---|---|
| Vectorice las funciones objetivo | Cada llamada a una función Python tiene coste; SciPy la evalúa muchas veces. |
| Proporcione gradientes/jacobianos (`jac=`) | Menos evaluaciones y mejor convergencia en optimización y EDO rígidas. |
| Escale las variables | Variables de órdenes de magnitud muy distintos dificultan la convergencia. |
| Use buenos valores iniciales | Los métodos locales encuentran el óptimo más cercano. |
| Elija el método adecuado | EDO rígidas → `Radau`/`BDF`; funciones ruidosas → `Nelder-Mead`; problemas globales → `differential_evolution`. |
| Ajuste tolerancias | `rtol`/`atol` por defecto pueden ser insuficientes (EDO) o excesivas. |
| Matrices dispersas | Para matrices grandes con muchos ceros, use `sparse`. |
| Acelere la función objetivo | Compílela con Numba si es costosa. |
| Paralelice | `differential_evolution(..., workers=-1)`, `vectorized=True` en `solve_ivp`. |

Precisión en coma flotante:

```python
0.1 + 0.2 == 0.3                    # False
np.isclose(0.1 + 0.2, 0.3)          # True
np.finfo(float).eps                 # 2.22e-16  precisión de máquina
```

---

## 16. Ejemplo práctico completo

**Caracterización de un sensor de temperatura a partir de un ensayo de enfriamiento.** Una pieza caliente se enfría según la ley de Newton, `T(t) = T∞ + (T₀ − T∞)·e^(−t/τ)`, pero el sensor que la mide no es instantáneo: responde como un sistema de primer orden con tiempo característico τₛ. El script simula un ensayo realista (ruido, interferencia de 50 Hz y valores atípicos) y realiza el análisis completo:

1. **Señal:** detecta la interferencia con el espectro de Welch, la elimina con un filtro *notch* y quita los valores atípicos con un filtro de mediana (para la gráfica se suaviza además con un Butterworth sin desfase).
2. **Ajuste:** ajusta con `curve_fit` el modelo completo «pieza + sensor», que estima a la vez T∞, T₀, τ y τₛ con intervalos de confianza, y lo compara con el modelo ingenuo que ignora el sensor.
3. **Validación:** analiza los residuos (normalidad de Shapiro-Wilk) y comprueba el modelo analítico resolviendo numéricamente el sistema de EDO con `solve_ivp`.
4. **Ingeniería:** calcula el tiempo hasta el 95 % del enfriamiento (`brentq`), la energía disipada (`quad`) y el máximo error dinámico del sensor (`minimize_scalar`).
5. **Informe:** guarda una figura con cuatro paneles y un resumen en texto.

```python
"""
Caracterización de un ensayo de enfriamiento con SciPy.
Requiere: pip install scipy numpy matplotlib
"""
from pathlib import Path

import matplotlib

matplotlib.use("Agg")
import matplotlib.pyplot as plt
import numpy as np
from scipy import integrate, optimize, signal, stats

SALIDA = Path(__file__).resolve().parent / "salida_scipy"
FS = 200.0                     # Frecuencia de muestreo (Hz)
DURACION = 600.0               # Segundos de ensayo
REAL = {"T_inf": 22.0, "T0": 85.0, "tau": 140.0, "tau_s": 6.0}
MASA, CALOR_ESPECIFICO = 0.8, 900.0          # kg, J/(kg·K): pieza de aluminio


# ------------------------------------------------------------------ Modelos
def enfriamiento(t, T_inf, T0, tau):
    """Temperatura real de la pieza (ley de Newton)."""
    return T_inf + (T0 - T_inf) * np.exp(-t / tau)


def lectura_sensor(t, T_inf, T0, tau, tau_s):
    """Lectura de un sensor de primer orden que parte de T_inf y se pone en contacto en t = 0.
    Solución analítica de  tau_s·dTs/dt = T(t) - Ts,  Ts(0) = T_inf."""
    return T_inf + (T0 - T_inf) * tau / (tau - tau_s) * (np.exp(-t / tau) - np.exp(-t / tau_s))


# ------------------------------------------------------------------ 1. Datos simulados
def simular_ensayo(semilla=11):
    rng = np.random.default_rng(semilla)
    t = np.arange(0, DURACION, 1 / FS)
    real = enfriamiento(t, REAL["T_inf"], REAL["T0"], REAL["tau"])
    sensor = signal.lti([1], [REAL["tau_s"], 1])                  # Función de transferencia 1/(τs·s + 1)
    _, medida, _ = signal.lsim(sensor, real, t, X0=[REAL["T_inf"] * REAL["tau_s"]])
    medida += rng.normal(0, 0.35, t.size)                         # Ruido blanco
    medida += 0.8 * np.sin(2 * np.pi * 50 * t)                    # Interferencia de la red eléctrica
    atipicos = rng.choice(t.size, 60, replace=False)
    medida[atipicos] += rng.choice([-1, 1], 60) * rng.uniform(4, 9, 60)   # Picos espurios
    return t, medida, real


# ------------------------------------------------------------------ 2. Procesado de la señal
def limpiar_senal(medida):
    f, Pxx = signal.welch(signal.detrend(medida), fs=FS, nperseg=4096)
    banda = f > 5
    f_interferencia = f[banda][np.argmax(Pxx[banda])]
    sin_atipicos = signal.medfilt(medida, kernel_size=9)
    b, a = signal.iirnotch(f_interferencia, Q=30, fs=FS)
    sin_red = signal.filtfilt(b, a, sin_atipicos)                # Base para el ajuste (sin suavizar)
    sos = signal.butter(4, 1.0, btype="lowpass", fs=FS, output="sos")
    suavizada = signal.sosfiltfilt(sos, sin_red)                 # Solo para visualizar
    return sin_red, suavizada, (f, Pxx, f_interferencia)


# ------------------------------------------------------------------ 3. Ajuste de modelos
def ajustar(t, datos):
    t_aj, y_aj = t[::FS_A_1HZ], datos[::FS_A_1HZ]                # Una muestra por segundo
    p0 = [y_aj[-1], y_aj[:20].max() + 5, (t_aj[-1] - t_aj[0]) / 4, 2.0]
    p, cov = optimize.curve_fit(lectura_sensor, t_aj, y_aj, p0=p0,
                                bounds=([-20, 0, 1, 0.1], [60, 200, 5000, 60]))
    gl = t_aj.size - len(p)
    ic95 = stats.t.ppf(0.975, gl) * np.sqrt(np.diag(cov))
    residuos = y_aj - lectura_sensor(t_aj, *p)
    r2 = 1 - np.sum(residuos**2) / np.sum((y_aj - y_aj.mean())**2)
    _, p_normalidad = stats.shapiro(residuos)

    # Modelo ingenuo: ignora el sensor y descarta los primeros 30 s
    ventana = t_aj > 30
    p_ing, _ = optimize.curve_fit(enfriamiento, t_aj[ventana], y_aj[ventana], p0=p[:3])
    return {"p": p, "ic95": ic95, "r2": r2, "rmse": np.sqrt(np.mean(residuos**2)), "residuos": residuos,
            "t_aj": t_aj, "p_normalidad": p_normalidad, "p_ingenuo": p_ing}


FS_A_1HZ = int(FS)


def validar_con_edo(p, t):
    """Resuelve numéricamente pieza + sensor y lo compara con la solución analítica."""
    T_inf, T0, tau, tau_s = p

    def sistema(_t, y):
        T, Ts = y
        return [-(T - T_inf) / tau, (T - Ts) / tau_s]

    sol = integrate.solve_ivp(sistema, (0, t[-1]), [T0, T_inf], t_eval=t[::FS_A_1HZ],
                              method="DOP853", rtol=1e-10, atol=1e-10)
    diferencia = max(np.max(np.abs(sol.y[0] - enfriamiento(sol.t, T_inf, T0, tau))),
                     np.max(np.abs(sol.y[1] - lectura_sensor(sol.t, *p))))
    return sol, diferencia


# ------------------------------------------------------------------ 4. Magnitudes de ingeniería
def magnitudes(p):
    T_inf, T0, tau, tau_s = p
    objetivo = T_inf + 0.05 * (T0 - T_inf)                      # 95 % del cambio total
    t95 = optimize.brentq(lambda tt: enfriamiento(tt, T_inf, T0, tau) - objetivo, 0, 20 * tau)

    potencia = lambda tt: MASA * CALOR_ESPECIFICO * (T0 - T_inf) / tau * np.exp(-tt / tau)   # -m·c·dT/dt
    energia, _ = integrate.quad(potencia, 0, t95)

    # Error dinámico: cuánto se queda «atrás» la lectura respecto a la temperatura real.
    # Se minimiza (real - lectura): el mínimo es el mayor exceso de la lectura sobre la realidad.
    error = lambda tt: enfriamiento(tt, T_inf, T0, tau) - lectura_sensor(tt, *p)
    res = optimize.minimize_scalar(error, bounds=(3 * tau_s, 300), method="bounded")
    return {"t95": t95, "energia_kJ": energia / 1000,
            "energia_teorica_kJ": MASA * CALOR_ESPECIFICO * 0.95 * (T0 - T_inf) / 1000,
            "t_error_max": res.x, "error_max": -res.fun}


# ------------------------------------------------------------------ 5. Informe
def figura(t, medida, suavizada, espectro, ajuste, sol_edo):
    f, Pxx, f_int = espectro
    T_inf, T0, tau, tau_s = ajuste["p"]
    fig, axs = plt.subplots(2, 2, figsize=(14, 9), layout="constrained")
    axs[0, 0].plot(t, medida, color="#BFBFBF", lw=0.5, label="Medida bruta")
    axs[0, 0].plot(t, suavizada, color="#1F4E78", lw=1.5, label="Señal filtrada")
    axs[0, 0].plot(t, enfriamiento(t, T_inf, T0, tau), "--", color="#C00000", lw=1.5,
                   label=f"Temperatura real estimada (τ = {tau:.1f} s)")
    axs[0, 0].set(title="Ensayo de enfriamiento", xlabel="t (s)", ylabel="T (°C)")
    axs[0, 0].legend(frameon=False)

    axs[0, 1].semilogy(f, Pxx, color="#1F4E78")
    axs[0, 1].axvline(f_int, color="#C00000", ls=":", label=f"Interferencia: {f_int:.1f} Hz")
    axs[0, 1].set(title="Densidad espectral (Welch)", xlabel="f (Hz)", ylabel="PSD (°C²/Hz)", xlim=(0, 100))
    axs[0, 1].legend(frameon=False)

    axs[1, 0].scatter(ajuste["t_aj"], ajuste["residuos"], s=4, color="#2E75B6", alpha=0.6)
    axs[1, 0].axhline(0, color="black", lw=0.8)
    axs[1, 0].set(title=f"Residuos del ajuste (R² = {ajuste['r2']:.5f}, RMSE = {ajuste['rmse']:.3f} °C)",
                  xlabel="t (s)", ylabel="Residuo (°C)")

    ventana = sol_edo.t <= 60
    axs[1, 1].plot(t[t <= 60], suavizada[t <= 60], color="#1F4E78", label="Lectura del sensor (filtrada)")
    axs[1, 1].plot(sol_edo.t[ventana], sol_edo.y[0][ventana], "o", ms=4, color="#C00000",
                   label="Temperatura de la pieza (EDO)")
    axs[1, 1].set(title=f"Transitorio inicial: τₛ estimado = {tau_s:.2f} s", xlabel="t (s)", ylabel="T (°C)")
    axs[1, 1].legend(frameon=False)
    for ax in axs.flat:
        ax.grid(alpha=0.3)
    fig.savefig(SALIDA / "ensayo.png", dpi=140)
    plt.close(fig)


def main():
    SALIDA.mkdir(exist_ok=True)
    t, medida, _ = simular_ensayo()
    datos, suavizada, espectro = limpiar_senal(medida)
    ajuste = ajustar(t, datos)
    sol_edo, diferencia_edo = validar_con_edo(ajuste["p"], t)
    mag = magnitudes(ajuste["p"])
    figura(t, medida, suavizada, espectro, ajuste, sol_edo)

    nombres = ["T∞ (°C)", "T₀ (°C)", "τ (s)", "τₛ (s)"]
    reales = [REAL["T_inf"], REAL["T0"], REAL["tau"], REAL["tau_s"]]
    lineas = ["RESULTADOS DEL ENSAYO", "=" * 60, f"Interferencia detectada: {espectro[2]:.2f} Hz", "",
              "Modelo pieza + sensor (curve_fit):"]
    for nombre, valor, ic, real in zip(nombres, ajuste["p"], ajuste["ic95"], reales):
        lineas.append(f"  {nombre:<8} = {valor:8.3f} ± {ic:.3f} (IC 95 %)   valor real: {real}")
    lineas += [f"  R² = {ajuste['r2']:.6f} · RMSE = {ajuste['rmse']:.3f} °C",
               f"  Normalidad de los residuos (Shapiro-Wilk): p = {ajuste['p_normalidad']:.3f}",
               f"Modelo ingenuo sin sensor: T₀ = {ajuste['p_ingenuo'][1]:.2f} °C, τ = {ajuste['p_ingenuo'][2]:.1f} s "
               f"(sesgo en T₀ por el retraso del sensor)", "",
               f"Diferencia máxima solución analítica vs. EDO numérica: {diferencia_edo:.2e} °C",
               f"Tiempo hasta el 95 % del enfriamiento: {mag['t95']:.1f} s",
               f"Energía disipada hasta t95: {mag['energia_kJ']:.2f} kJ (teórica {mag['energia_teorica_kJ']:.2f} kJ)",
               f"Error dinámico máximo del sensor: {mag['error_max']:.2f} °C en t = {mag['t_error_max']:.1f} s"]
    texto = "\n".join(lineas)
    (SALIDA / "resultados.txt").write_text(texto, encoding="utf-8")
    print(texto)


if __name__ == "__main__":
    main()
```

> **Lectura de los resultados:** el modelo ingenuo sobrestima T₀ porque el sensor «llega tarde» (un sensor de primer orden retrasa la señal unos τₛ segundos). Modelar el instrumento de medida junto con el fenómeno elimina ese sesgo: es una situación muy habitual en ingeniería experimental.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `AttributeError: module 'scipy' has no attribute 'optimize'` | Solo se importó `scipy`. | `from scipy import optimize`. |
| `ImportError: cannot import name 'simps'` / `trapz` / `cumtrapz` | Funciones eliminadas. | `simpson`, `trapezoid`, `cumulative_trapezoid`. |
| `NotImplementedError` con `interp2d` | Función eliminada. | `RegularGridInterpolator` o `griddata`. |
| `OptimizeWarning: Covariance of the parameters could not be estimated` | Parámetros redundantes o mal condicionados, o `p0` deficiente. | Revise el modelo, dé un `p0` razonable, escale los datos. |
| `RuntimeError: Optimal parameters not found: maxfev exceeded` | `curve_fit` no converge. | Mejor `p0`, `bounds`, aumente `maxfev`. |
| `minimize` devuelve un resultado malo con `success=True` | Mínimo local. | Varios `x0` u optimización global. |
| `ValueError: f(a) and f(b) must have different signs` | `brentq` sin cambio de signo en el intervalo. | Elija un intervalo que contenga la raíz. |
| `IntegrationWarning: The maximum number of subdivisions has been achieved` | Integrando oscilante o singular. | `limit=`, `points=`, divida el intervalo. |
| `solve_ivp` extremadamente lento | Problema rígido con método explícito. | `method="Radau"`, `"BDF"` o `"LSODA"`. |
| `dblquad` da resultados extraños | Orden de los argumentos: la función recibe `(y, x)`. | Respete el orden o use `nquad`. |
| `LinAlgError: Singular matrix` | Matriz no invertible. | Revise el sistema; use `lstsq` o `pinv`. |
| `MemoryError` con matrices grandes | Matriz densa enorme. | `scipy.sparse`. |
| Filtro inestable o que devuelve `nan` | Orden alto con coeficientes `b, a`. | `output="sos"` y `sosfiltfilt`. |
| p-valores incoherentes en pruebas | Supuestos no cumplidos (normalidad, independencia). | Compruebe supuestos o use pruebas no paramétricas. |

---

## 18. Buenas prácticas

- Importe los submódulos explícitamente (`from scipy import optimize, stats`).
- Vectorice las funciones y trabaje con arreglos NumPy.
- Compruebe `success`/`message` de cada resultado y valide con casos conocidos.
- Proporcione buenas estimaciones iniciales, límites razonables y, si es posible, gradientes.
- Elija el método según el problema (rígido, ruidoso, global, restringido).
- Cuantifique la incertidumbre: errores de los parámetros, intervalos de confianza, residuos.
- Use filtros en formato `sos` y `filtfilt`/`sosfiltfilt` para no desfasar la señal.
- Use matrices dispersas cuando la mayoría de elementos sean cero.
- Fije semillas (`random_state`, `seed`, `np.random.default_rng(semilla)`) para resultados reproducibles.
- Consulte las notas de versión al actualizar: SciPy elimina periódicamente funciones obsoletas.

---

## 19. Referencia rápida

```python
from scipy import integrate, optimize, interpolate, linalg, sparse, stats, signal, fft, spatial, special, constants
integrate.quad(f, a, b) / dblquad(f(y, x), a, b, c, d) / simpson(y, x=x) / trapezoid / cumulative_trapezoid
integrate.solve_ivp(f(t, y), (t0, tf), y0, t_eval=, method="RK45"|"Radau", events=, args=)
optimize.minimize(f, x0, method=, jac=, bounds=, constraints=) / minimize_scalar / differential_evolution(f, limites)
optimize.brentq(f, a, b) / root_scalar / fsolve / root / linprog / milp / linear_sum_assignment
optimize.curve_fit(modelo, x, y, p0=, bounds=) → (p, cov) / least_squares(residuos, x0, loss="soft_l1")
interpolate.CubicSpline(x, y) / PchipInterpolator / make_smoothing_spline / RegularGridInterpolator / griddata
linalg.solve / det / inv / eig / eigh / svd / lu / qr / cholesky / expm / lstsq / null_space
sparse.diags_array / csr_array / coo_array; sparse.linalg.spsolve / cg / eigsh; sparse.csgraph.shortest_path
stats.norm(loc, scale).pdf / cdf / ppf / rvs / interval; stats.describe / sem / ttest_ind / mannwhitneyu / f_oneway
stats.chi2_contingency / shapiro / pearsonr / spearmanr / linregress / bootstrap
signal.butter(n, fc, fs=, output="sos") + sosfiltfilt / iirnotch / savgol_filter / medfilt / find_peaks / welch / spectrogram
fft.rfft / rfftfreq / fft / ifft / fftshift;  spatial.KDTree / distance.cdist / ConvexHull / Delaunay / transform.Rotation
special.gamma / erf / comb / jv / logsumexp / expit;  constants.c / g / h / k / physical_constants
```

---

## 20. Ejercicios propuestos

1. Calcule el área bajo la curva normal estándar entre -1 y 1 con `quad` y compárela con `stats.norm.cdf`.
2. Simule el sistema depredador-presa de Lotka-Volterra con `solve_ivp` y represente las poblaciones y el diagrama de fases.
3. Ajuste un modelo logístico de crecimiento a datos de casos acumulados y obtenga intervalos de confianza de la tasa de crecimiento.
4. Resuelva un problema de planificación de producción con `linprog` (tres productos, tres recursos limitados) y analice cómo cambia el beneficio al variar un recurso.
5. Amplíe el ejemplo completo con un análisis *bootstrap* de τ, la detección automática del instante de contacto con `find_peaks` sobre la derivada y un informe en PDF.

---

## 21. Referencias

- Documentación oficial: https://docs.scipy.org/doc/scipy/
- Guía de usuario: https://docs.scipy.org/doc/scipy/tutorial/index.html
- Referencia de la API: https://docs.scipy.org/doc/scipy/reference/index.html
- Notas de versión (cambios y eliminaciones): https://docs.scipy.org/doc/scipy/release.html
- Virtanen, P. et al. (2020). *SciPy 1.0: fundamental algorithms for scientific computing in Python*. Nature Methods 17, 261–272.
- *Scipy Lecture Notes*: https://lectures.scientific-python.org/
