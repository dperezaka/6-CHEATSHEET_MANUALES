# Manual de uso: SciPy

> **Categoría:** Ciencia de datos · **Utilidad:** Cálculo científico y técnico
> **Tipo:** Librería de terceros (`pip install scipy`) · **Versión de referencia:** 1.11 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Mapa de submódulos](#3-mapa-de-submódulos)
4. [Constantes y funciones especiales](#4-constantes-y-funciones-especiales)
5. [Optimización (`scipy.optimize`)](#5-optimización-scipyoptimize)
6. [Ajuste de curvas](#6-ajuste-de-curvas)
7. [Raíces de ecuaciones](#7-raíces-de-ecuaciones)
8. [Programación lineal y entera](#8-programación-lineal-y-entera)
9. [Integración y ecuaciones diferenciales (`scipy.integrate`)](#9-integración-y-ecuaciones-diferenciales-scipyintegrate)
10. [Interpolación (`scipy.interpolate`)](#10-interpolación-scipyinterpolate)
11. [Estadística (`scipy.stats`)](#11-estadística-scipystats)
12. [Contrastes de hipótesis](#12-contrastes-de-hipótesis)
13. [Procesamiento de señales (`scipy.signal`) y FFT](#13-procesamiento-de-señales-scipysignal-y-fft)
14. [Álgebra lineal (`scipy.linalg`)](#14-álgebra-lineal-scipylinalg)
15. [Matrices dispersas (`scipy.sparse`)](#15-matrices-dispersas-scipysparse)
16. [Datos espaciales (`scipy.spatial`)](#16-datos-espaciales-scipyspatial)
17. [Imágenes n-dimensionales (`scipy.ndimage`)](#17-imágenes-n-dimensionales-scipyndimage)
18. [Agrupamiento jerárquico (`scipy.cluster`)](#18-agrupamiento-jerárquico-scipycluster)
19. [Ejemplo práctico completo](#19-ejemplo-práctico-completo)
20. [Errores frecuentes](#20-errores-frecuentes)
21. [Buenas prácticas](#21-buenas-prácticas)
22. [Referencia rápida](#22-referencia-rápida)
23. [Ejercicios propuestos](#23-ejercicios-propuestos)
24. [Referencias](#24-referencias)

---

## 1. Descripción

**SciPy** (*Scientific Python*) amplía NumPy con algoritmos de **cálculo científico y técnico**: optimización, ajuste de curvas, resolución de ecuaciones, integración y ecuaciones diferenciales, interpolación, estadística y contrastes de hipótesis, procesamiento de señales, álgebra lineal avanzada, matrices dispersas, geometría computacional y procesamiento de imágenes. Sus rutinas se apoyan en bibliotecas numéricas maduras y muy optimizadas (LAPACK, BLAS, QUADPACK, HiGHS…).

**Ventajas:**
- Algoritmos robustos, probados y eficientes en una sola librería.
- Integración total con NumPy (trabaja con *arrays*).
- Amplia cobertura: ingeniería, física, finanzas, estadística, investigación operativa.
- Documentación con fundamentos matemáticos y referencias.

**Limitaciones:**
- API de bajo nivel: hay que conocer el problema matemático que se resuelve.
- Estadística orientada a contrastes y distribuciones; para modelos (regresión con p-valores, series temporales) es más completo **statsmodels**.
- Optimización con restricciones complejas o a gran escala: **Pyomo**, **CVXPY**, **OR-Tools**.
- No admite GPU (salvo con el estándar *array API* en algunos módulos).

| Si necesita… | Opción |
|---|---|
| Optimización, integración, señales, estadística, interpolación | **SciPy** |
| Operaciones básicas con *arrays* | NumPy |
| Modelos estadísticos con informes (OLS, GLM, ARIMA) | statsmodels |
| *Machine learning* | scikit-learn |
| Cálculo simbólico (derivadas exactas, ecuaciones) | SymPy |

---

## 2. Instalación e importación

```bash
pip install scipy numpy matplotlib
```

Se importan los submódulos, no el paquete completo:

```python
import numpy as np
from scipy import optimize, integrate, interpolate, stats, signal, linalg, sparse, spatial, ndimage, special
```

---

## 3. Mapa de submódulos

| Submódulo | Contenido |
|---|---|
| `scipy.constants` | Constantes físicas y conversiones de unidades. |
| `scipy.special` | Funciones especiales (gamma, beta, Bessel, error, logística…). |
| `scipy.optimize` | Minimización, raíces, ajuste de curvas, mínimos cuadrados, programación lineal y entera. |
| `scipy.integrate` | Integrales numéricas y ecuaciones diferenciales ordinarias. |
| `scipy.interpolate` | Interpolación y *splines* 1D y N-D. |
| `scipy.stats` | Distribuciones, estadística descriptiva, contrastes, *bootstrap*. |
| `scipy.signal` | Filtros, convolución, espectros, detección de picos. |
| `scipy.fft` | Transformada rápida de Fourier. |
| `scipy.linalg` | Álgebra lineal avanzada (descomposiciones, exponencial de matrices). |
| `scipy.sparse` | Matrices dispersas y sus algoritmos (`sparse.linalg`, `sparse.csgraph`). |
| `scipy.spatial` | Árboles KD, distancias, envolventes convexas, triangulaciones, rotaciones. |
| `scipy.ndimage` | Filtros y morfología sobre imágenes N-dimensionales. |
| `scipy.cluster` | Agrupamiento jerárquico y *k-means* básico. |
| `scipy.io` | Lectura/escritura de MATLAB (`.mat`), WAV, Matrix Market. |

---

## 4. Constantes y funciones especiales

```python
from scipy import constants as c
c.c, c.g, c.h, c.k, c.N_A          # Luz, gravedad, Planck, Boltzmann, Avogadro
c.kilo, c.mega, c.hour, c.day, c.inch, c.mile, c.bar, c.atm
c.convert_temperature(100, "Celsius", "Kelvin")
c.physical_constants["electron mass"]          # (valor, unidad, incertidumbre)

from scipy import special
special.gamma(5) ; special.factorial(5, exact=True) ; special.comb(10, 3, exact=True) ; special.perm(10, 3)
special.erf(0.5) ; special.expit(2.0) ; special.logit(0.88) ; special.softmax([1, 2, 3])
special.logsumexp([1000, 1001])                # Suma de exponenciales sin desbordamiento
special.binom(10, 3) ; special.jv(0, 2.5)      # Bessel
```

---

## 5. Optimización (`scipy.optimize`)

### 5.1 Minimizar funciones

```python
from scipy.optimize import minimize, minimize_scalar

def rosenbrock(x):
    return (1 - x[0]) ** 2 + 100 * (x[1] - x[0] ** 2) ** 2

res = minimize(rosenbrock, x0=[-1.0, 2.0], method="BFGS")
res.x, res.fun, res.success, res.message, res.nit

minimize_scalar(lambda x: (x - 2) ** 2 + 1, bounds=(0, 5), method="bounded")
```

### 5.2 Restricciones y límites

```python
# Maximizar el volumen de una caja con superficie 10 m² (minimizar el negativo)
volumen = lambda v: -(v[0] * v[1] * v[2])
restricciones = [{"type": "eq", "fun": lambda v: 2 * (v[0]*v[1] + v[1]*v[2] + v[0]*v[2]) - 10}]
res = minimize(volumen, x0=[1, 1, 1], method="SLSQP", bounds=[(0.1, None)] * 3, constraints=restricciones)

from scipy.optimize import LinearConstraint, NonlinearConstraint, Bounds
minimize(f, x0, method="trust-constr", bounds=Bounds([0, 0], [10, 5]),
         constraints=[LinearConstraint([[1, 1]], lb=-np.inf, ub=8)])
```

| Método | Uso |
|---|---|
| `BFGS`, `L-BFGS-B` | Sin restricciones (o con límites, `L-BFGS-B`); funciones suaves. |
| `Nelder-Mead`, `Powell` | Sin derivadas; funciones ruidosas o no derivables. |
| `SLSQP` | Restricciones de igualdad y desigualdad. |
| `trust-constr` | Restricciones generales, problemas grandes. |
| `differential_evolution`, `dual_annealing`, `shgo`, `basinhopping` | Optimización global (muchos mínimos locales). |

```python
from scipy.optimize import differential_evolution
differential_evolution(f, bounds=[(-5, 5), (-5, 5)], seed=0, polish=True)
```

> Indique siempre un buen punto de partida (`x0`) y compruebe `res.success`. Los métodos locales encuentran el mínimo **más cercano**, no necesariamente el global.

---

## 6. Ajuste de curvas

### 6.1 `curve_fit`

```python
from scipy.optimize import curve_fit

def exponencial(t, a, k, c):
    return a * np.exp(-k * t) + c

t = np.linspace(0, 10, 40)
y = exponencial(t, 5, 0.6, 1) + np.random.default_rng(0).normal(0, 0.15, t.size)

parametros, covarianza = curve_fit(exponencial, t, y, p0=[4, 0.5, 0.5], bounds=([0, 0, -5], [20, 5, 5]))
errores = np.sqrt(np.diag(covarianza))        # Desviación típica de cada parámetro
for nombre, v, e in zip("akc", parametros, errores):
    print(f"{nombre} = {v:.3f} ± {e:.3f}")

residuos = y - exponencial(t, *parametros)
r2 = 1 - np.sum(residuos**2) / np.sum((y - y.mean())**2)
```

Con incertidumbres conocidas en cada punto: `curve_fit(f, x, y, sigma=error_y, absolute_sigma=True)`.

### 6.2 `least_squares` (robusto frente a atípicos)

```python
from scipy.optimize import least_squares

def residuos(p, t, y):
    return exponencial(t, *p) - y

res = least_squares(residuos, x0=[4, 0.5, 0.5], args=(t, y), loss="soft_l1", f_scale=0.3)
res.x, res.cost
```

---

## 7. Raíces de ecuaciones

```python
from scipy.optimize import root_scalar, root, fsolve, brentq

# Tipo de interés que hace que el VAN sea 0 (TIR)
flujos = np.array([-1000, 300, 400, 400, 300])
van = lambda r: np.sum(flujos / (1 + r) ** np.arange(flujos.size))
tir = root_scalar(van, bracket=[0, 1], method="brentq").root      # bracket: f cambia de signo

brentq(lambda x: x**3 - 2*x - 5, 2, 3)

# Sistemas de ecuaciones no lineales
def sistema(v):
    x, y = v
    return [x**2 + y**2 - 4, np.exp(x) + y - 1]
root(sistema, x0=[1, -1]).x
```

> Los métodos con intervalo (`brentq`, `bisect`) son los más fiables en una variable: garantizan encontrar una raíz si la función cambia de signo en el intervalo.

---

## 8. Programación lineal y entera

### 8.1 `linprog`

```python
from scipy.optimize import linprog

# Maximizar beneficio 40·mesas + 30·sillas (linprog minimiza → coeficientes negativos)
c = [-40, -30]
A_ub = [[2, 1],      # Horas de carpintería: 2 por mesa, 1 por silla ≤ 100
        [1, 1]]      # Horas de pintura:     1 por mesa, 1 por silla ≤ 80
b_ub = [100, 80]
res = linprog(c, A_ub=A_ub, b_ub=b_ub, bounds=[(0, None), (0, 40)], method="highs")
res.x, -res.fun
res.ineqlin.marginals          # Precios sombra: valor de una hora más de cada recurso
```

### 8.2 Programación entera mixta (`milp`)

```python
from scipy.optimize import milp, LinearConstraint, Bounds
res = milp(c=np.array([-40, -30]), constraints=LinearConstraint(A_ub, ub=b_ub),
           integrality=np.ones(2), bounds=Bounds(0, [np.inf, 40]))   # 1 = variable entera
```

### 8.3 Asignación óptima

```python
from scipy.optimize import linear_sum_assignment
costes = np.array([[4, 1, 3], [2, 0, 5], [3, 2, 2]])          # coste[trabajador, tarea]
filas, columnas = linear_sum_assignment(costes)                # Asignación de coste mínimo
costes[filas, columnas].sum()
```

---

## 9. Integración y ecuaciones diferenciales (`scipy.integrate`)

### 9.1 Integrales

```python
from scipy import integrate

valor, error = integrate.quad(lambda x: np.exp(-x**2), -np.inf, np.inf)    # √π
integrate.quad(lambda x, a: a * x**2, 0, 1, args=(3,))
integrate.dblquad(lambda y, x: x * y, 0, 1, 0, 2)                           # Integral doble
integrate.trapezoid(y, x) ; integrate.simpson(y, x=x)                       # A partir de datos muestreados
integrate.cumulative_trapezoid(velocidad, tiempo, initial=0)               # Acumulada (posición)
```

### 9.2 Ecuaciones diferenciales ordinarias con `solve_ivp`

```python
from scipy.integrate import solve_ivp

# Modelo epidemiológico SIR
def sir(t, y, beta, gamma):
    s, i, r = y
    return [-beta * s * i, beta * s * i - gamma * i, gamma * i]

sol = solve_ivp(sir, t_span=(0, 160), y0=[0.999, 0.001, 0], args=(0.3, 0.1),
                t_eval=np.linspace(0, 160, 321), method="RK45", rtol=1e-8)
s, i, r = sol.y
pico = sol.t[i.argmax()]

# Detener la integración cuando ocurra un evento
def impacto(t, y):            # Altura = 0
    return y[0]
impacto.terminal, impacto.direction = True, -1
caida = solve_ivp(lambda t, y: [y[1], -9.81], (0, 10), [100, 0], events=impacto, dense_output=True)
caida.t_events[0]             # Instante del impacto
caida.sol(1.5)                # Solución continua en cualquier instante
```

| Método | Uso |
|---|---|
| `RK45` (defecto), `DOP853` | Problemas no rígidos. |
| `Radau`, `BDF`, `LSODA` | Problemas rígidos (escalas de tiempo muy distintas, química, circuitos). |

---

## 10. Interpolación (`scipy.interpolate`)

```python
from scipy import interpolate

x = np.array([0, 1, 2, 4, 7]); y = np.array([0, 0.8, 0.9, -0.7, 0.6])
xs = np.linspace(0, 7, 200)

np.interp(xs, x, y)                                            # Lineal (NumPy)
interpolate.CubicSpline(x, y, bc_type="natural")(xs)          # Spline cúbico
interpolate.PchipInterpolator(x, y)(xs)                       # Conserva la monotonía (sin oscilaciones)
interpolate.Akima1DInterpolator(x, y)(xs)
interpolate.make_interp_spline(x, y, k=3)(xs)
interpolate.make_smoothing_spline(x_ruido, y_ruido, lam=None)(xs)   # Suavizado (datos con ruido)

# 2D: datos dispersos → rejilla
from scipy.interpolate import griddata, RBFInterpolator
zi = griddata(puntos, valores, (rejilla_x, rejilla_y), method="cubic")
rbf = RBFInterpolator(puntos, valores, kernel="thin_plate_spline", smoothing=0.1)
```

> `interp1d` es una API heredada; para código nuevo use `np.interp`, `CubicSpline`, `PchipInterpolator` o `make_interp_spline`.

---

## 11. Estadística (`scipy.stats`)

### 11.1 Distribuciones

```python
from scipy import stats

normal = stats.norm(loc=100, scale=15)                   # Distribución «congelada»
normal.pdf(115) ; normal.cdf(130) ; normal.sf(130)       # Densidad, acumulada, cola superior (1 - cdf)
normal.ppf(0.95)                                          # Percentil 95 (cuantil)
normal.interval(0.95)                                     # Intervalo central del 95 %
normal.mean(), normal.std(), normal.rvs(size=1000, random_state=0)

stats.binom(n=20, p=0.3).pmf(5)                          # Discretas: pmf
stats.poisson(mu=4).cdf(2)
stats.expon(scale=10) ; stats.lognorm(s=0.5, scale=np.exp(3)) ; stats.t(df=12) ; stats.chi2(df=3)
stats.beta(a=2, b=5) ; stats.gamma(a=2, scale=3) ; stats.weibull_min(c=1.5) ; stats.uniform(0, 10)

parametros = stats.lognorm.fit(datos, floc=0)            # Ajustar una distribución a datos
stats.fit(stats.gamma, datos)                            # API genérica de ajuste
```

| Método | Significado |
|---|---|
| `pdf` / `pmf` | Densidad / probabilidad puntual. |
| `cdf` / `sf` | Probabilidad acumulada / de superar el valor. |
| `ppf` / `isf` | Inversa de `cdf` / de `sf` (cuantiles). |
| `rvs` | Muestras aleatorias. |
| `interval`, `mean`, `std`, `stats`, `entropy` | Resúmenes. |

### 11.2 Estadística descriptiva

```python
stats.describe(datos)
stats.skew(datos) ; stats.kurtosis(datos)
stats.zscore(datos) ; stats.iqr(datos) ; stats.median_abs_deviation(datos)
stats.trim_mean(datos, 0.1) ; stats.mode(datos) ; stats.percentileofscore(datos, 120)
stats.sem(datos)                                       # Error estándar de la media
stats.pearsonr(x, y) ; stats.spearmanr(x, y) ; stats.kendalltau(x, y)
stats.linregress(x, y)                                 # Pendiente, ordenada, r, p, error estándar
```

### 11.3 Intervalos de confianza

```python
r = stats.ttest_1samp(datos, popmean=0)
r.confidence_interval(confidence_level=0.95)           # IC de la media

bs = stats.bootstrap((datos,), np.median, confidence_level=0.95, n_resamples=5000, random_state=0)
bs.confidence_interval                                 # IC de cualquier estadístico
```

---

## 12. Contrastes de hipótesis

```python
# Normalidad
stats.shapiro(datos) ; stats.normaltest(datos) ; stats.anderson(datos)

# Dos grupos
stats.ttest_ind(a, b, equal_var=False)          # t de Welch (medias, datos aproximadamente normales)
stats.mannwhitneyu(a, b)                         # No paramétrico
stats.ttest_rel(antes, despues)                  # Datos emparejados
stats.wilcoxon(antes, despues)                   # Emparejados no paramétrico

# Más de dos grupos
stats.f_oneway(g1, g2, g3)                       # ANOVA de un factor
stats.kruskal(g1, g2, g3)                        # Kruskal-Wallis
stats.tukey_hsd(g1, g2, g3)                      # Comparaciones múltiples por pares

# Proporciones y tablas
stats.chi2_contingency([[120, 380], [150, 350]])       # Independencia en tablas de contingencia
stats.fisher_exact([[8, 2], [1, 5]])                   # Tablas 2×2 con pocos datos
stats.binomtest(k=58, n=100, p=0.5)                    # Proporción frente a un valor

# Pruebas de permutación (sin supuestos de distribución)
stats.permutation_test((a, b), lambda x, y: np.mean(x) - np.mean(y), n_resamples=10_000, random_state=0)

# Corrección por comparaciones múltiples
stats.false_discovery_control(p_valores, method="bh")  # p-valores ajustados (Benjamini-Hochberg)
```

| Situación | Contraste habitual |
|---|---|
| Media de un grupo frente a un valor | `ttest_1samp` / `wilcoxon` |
| Dos grupos independientes | `ttest_ind(equal_var=False)` / `mannwhitneyu` |
| Dos medidas del mismo sujeto | `ttest_rel` / `wilcoxon` |
| Tres o más grupos | `f_oneway` + `tukey_hsd` / `kruskal` |
| Dos variables categóricas | `chi2_contingency` / `fisher_exact` |
| Correlación | `pearsonr` / `spearmanr` |

> Un p-valor bajo indica que los datos son poco compatibles con la hipótesis nula, **no** el tamaño ni la importancia práctica del efecto. Informe siempre del tamaño del efecto y de su intervalo de confianza.

---

## 13. Procesamiento de señales (`scipy.signal`) y FFT

```python
from scipy import signal
from scipy.fft import rfft, rfftfreq

fs = 500                                                       # Frecuencia de muestreo (Hz)
t = np.arange(0, 2, 1 / fs)
senal = np.sin(2*np.pi*5*t) + 0.5*np.sin(2*np.pi*50*t) + np.random.default_rng(0).normal(0, 0.3, t.size)

# Filtro paso bajo Butterworth de orden 4 a 15 Hz, aplicado sin desfase
sos = signal.butter(4, 15, btype="lowpass", fs=fs, output="sos")
filtrada = signal.sosfiltfilt(sos, senal)
signal.butter(4, [45, 55], btype="bandstop", fs=fs, output="sos")     # Eliminar 50 Hz (red eléctrica)

signal.savgol_filter(senal, window_length=31, polyorder=3)            # Suavizado que conserva picos
signal.medfilt(senal, kernel_size=5)                                  # Elimina picos aislados

picos, props = signal.find_peaks(filtrada, height=0.5, distance=fs // 10, prominence=0.3)

frecuencias = rfftfreq(t.size, 1 / fs)
espectro = np.abs(rfft(senal)) / t.size * 2
f, densidad = signal.welch(senal, fs=fs, nperseg=256)                 # Densidad espectral (más estable)
f, tt, Sxx = signal.spectrogram(senal, fs=fs)

signal.detrend(senal) ; signal.resample(senal, 500) ; signal.decimate(senal, 4)
signal.correlate(a, b, mode="full") ; signal.fftconvolve(a, kernel, mode="same")
```

> Use `output="sos"` + `sosfiltfilt` para filtros IIR: es numéricamente más estable que `b, a` + `filtfilt`.

---

## 14. Álgebra lineal (`scipy.linalg`)

Complementa a `numpy.linalg` con más descomposiciones y opciones:

```python
from scipy import linalg
linalg.solve(A, b, assume_a="pos")          # Indicar la estructura (simétrica definida positiva) acelera
linalg.lu(A) ; linalg.qr(A) ; linalg.cholesky(A) ; linalg.schur(A) ; linalg.svd(A)
linalg.eigh(A, subset_by_index=[0, 2])      # Solo algunos valores propios
linalg.expm(A)                              # Exponencial de matriz (sistemas dinámicos lineales)
linalg.null_space(A) ; linalg.orth(A) ; linalg.pinv(A)
linalg.solve_banded((1, 1), bandas, b) ; linalg.solve_triangular(L, b, lower=True)
linalg.toeplitz([1, 2, 3]) ; linalg.block_diag(A, B)
```

---

## 15. Matrices dispersas (`scipy.sparse`)

Para matrices con la mayoría de elementos a cero (redes, sistemas de ecuaciones grandes, texto):

```python
from scipy import sparse
from scipy.sparse.linalg import spsolve, eigsh
from scipy.sparse.csgraph import shortest_path, connected_components

filas = [0, 1, 2, 2]; cols = [1, 2, 0, 3]; datos = [4, 1, 2, 7]
m = sparse.coo_array((datos, (filas, cols)), shape=(4, 4)).tocsr()    # Construir en COO, operar en CSR
m.nnz ; m.toarray() ; m @ v
sparse.eye_array(1000, format="csr") ; sparse.random_array((1000, 1000), density=0.001)
sparse.save_npz("m.npz", m) ; sparse.load_npz("m.npz")

x = spsolve(A_dispersa, b)
dist, previos = shortest_path(grafo, directed=False, return_predecessors=True)   # Rutas en grafos
n, etiquetas = connected_components(grafo)
```

| Formato | Uso |
|---|---|
| COO | Construcción a partir de coordenadas. |
| CSR / CSC | Operaciones aritméticas, productos, filas / columnas. |
| LIL / DOK | Modificación elemento a elemento. |

> Las clases `*_array` (`csr_array`) son la API recomendada; las antiguas `*_matrix` se mantienen por compatibilidad. Con `*_array`, `*` multiplica elemento a elemento y `@` es el producto matricial, igual que en NumPy.

---

## 16. Datos espaciales (`scipy.spatial`)

```python
from scipy.spatial import KDTree, distance, ConvexHull, Delaunay, Voronoi
from scipy.spatial.transform import Rotation

arbol = KDTree(puntos)
dist, idx = arbol.query(consulta, k=3)                  # 3 vecinos más cercanos
arbol.query_ball_point([0.5, 0.5], r=0.1)               # Puntos dentro de un radio

distance.euclidean(a, b) ; distance.cityblock(a, b) ; distance.cosine(a, b)
D = distance.cdist(A, B, metric="euclidean")            # Distancias entre dos conjuntos
d = distance.pdist(A) ; distance.squareform(d)          # Distancias dentro de un conjunto

hull = ConvexHull(puntos); hull.volume, hull.vertices   # En 2D, volume = área
Delaunay(puntos).find_simplex(nuevos_puntos)
Rotation.from_euler("xyz", [90, 0, 45], degrees=True).apply(vectores)
```

Distancia geodésica entre coordenadas (fórmula del semiverseno):

```python
def haversine_km(lat1, lon1, lat2, lon2):
    lat1, lon1, lat2, lon2 = map(np.radians, (lat1, lon1, lat2, lon2))
    a = np.sin((lat2 - lat1) / 2) ** 2 + np.cos(lat1) * np.cos(lat2) * np.sin((lon2 - lon1) / 2) ** 2
    return 6371 * 2 * np.arcsin(np.sqrt(a))
```

---

## 17. Imágenes n-dimensionales (`scipy.ndimage`)

```python
from scipy import ndimage
ndimage.gaussian_filter(imagen, sigma=2) ; ndimage.median_filter(imagen, size=3)
ndimage.sobel(imagen, axis=0)                           # Bordes
ndimage.rotate(imagen, 30, reshape=False) ; ndimage.zoom(imagen, 0.5) ; ndimage.shift(imagen, (5, -3))
mascara = imagen > 0.5
etiquetas, n = ndimage.label(mascara)                    # Componentes conectados
ndimage.find_objects(etiquetas) ; ndimage.center_of_mass(mascara, etiquetas, range(1, n + 1))
ndimage.binary_opening(mascara, iterations=2) ; ndimage.binary_fill_holes(mascara)
ndimage.distance_transform_edt(mascara)
```

> Para visión artificial completa (cámaras, detección, vídeo), use OpenCV; para análisis científico de imágenes, scikit-image.

---

## 18. Agrupamiento jerárquico (`scipy.cluster`)

```python
from scipy.cluster.hierarchy import linkage, fcluster, dendrogram
Z = linkage(datos_escalados, method="ward")
grupos = fcluster(Z, t=4, criterion="maxclust")          # 4 grupos
dendrogram(Z, labels=nombres, leaf_rotation=90)          # Requiere Matplotlib
```

---

## 19. Ejemplo práctico completo

**Laboratorio de análisis técnico**: cinco casos resueltos con SciPy en un único script que genera un informe de texto y una figura de resumen:

1. **Ajuste de curvas**: curva de descarga de una batería con modelo exponencial, incertidumbre de los parámetros y predicción del tiempo hasta el 20 %.
2. **Ecuaciones diferenciales**: modelo SIR de difusión de un producto; estimación de sus parámetros a partir de datos con `least_squares` y cálculo del pico.
3. **Señales**: limpieza de la señal de un sensor (filtro de banda eliminada de 50 Hz y paso bajo), detección de picos y espectro.
4. **Optimización**: plan de producción óptimo con programación entera (`milp`) y precios sombra con `linprog`.
5. **Estadística**: prueba A/B de una web (tasa de conversión e importe) con χ², t de Welch, Mann-Whitney y *bootstrap*.

```python
"""
laboratorio_scipy.py — Cinco casos de cálculo científico con SciPy.
Requiere: pip install scipy numpy matplotlib
Uso: python laboratorio_scipy.py
"""
from pathlib import Path

import matplotlib

matplotlib.use("Agg")
import matplotlib.pyplot as plt
import numpy as np
from scipy import integrate, optimize, signal, stats
from scipy.fft import rfft, rfftfreq

RNG = np.random.default_rng(2026)
SALIDA = Path("laboratorio")
SALIDA.mkdir(exist_ok=True)


def titulo(texto: str) -> None:
    print(f"\n{texto}\n{'─' * len(texto)}")


# ------------------------------------------------------------------ 1. Ajuste de curvas
def caso_bateria(ax) -> None:
    titulo("1. Descarga de una batería (ajuste de curvas)")
    horas = np.linspace(0, 30, 31)
    modelo = lambda t, a, k, c: a * np.exp(-k * t) + c
    carga = modelo(horas, 82, 0.09, 15) + RNG.normal(0, 1.2, horas.size)

    p, cov = optimize.curve_fit(modelo, horas, carga, p0=[80, 0.1, 10], bounds=([0, 0, 0], [150, 2, 50]))
    err = np.sqrt(np.diag(cov))
    residuos = carga - modelo(horas, *p)
    r2 = 1 - residuos.var() / carga.var()
    for nombre, v, e in zip(["Amplitud", "Constante k", "Asíntota"], p, err):
        print(f"   {nombre:<12} {v:8.3f} ± {e:.3f}")
    print(f"   R² = {r2:.4f}")

    t20 = optimize.root_scalar(lambda t: modelo(t, *p) - 20, bracket=[0, 200]).root
    muestras = RNG.multivariate_normal(p, cov, 2000)                     # Propagación de la incertidumbre
    validas = muestras[muestras[:, 2] < 20]
    t20_mc = [optimize.brentq(lambda t: modelo(t, *m) - 20, 0, 1000) for m in validas[:500]]
    print(f"   Tiempo hasta el 20 %: {t20:.1f} h (IC 95 %: {np.percentile(t20_mc, 2.5):.1f}–"
          f"{np.percentile(t20_mc, 97.5):.1f} h)")

    ts = np.linspace(0, 45, 300)
    ax.scatter(horas, carga, s=14, color="gray", label="Medidas")
    ax.plot(ts, modelo(ts, *p), color="#1F4E78", label="Ajuste")
    ax.axhline(20, color="#C00000", linestyle="--", linewidth=1)
    ax.axvline(t20, color="#C00000", linestyle=":", linewidth=1)
    ax.set(title="1. Descarga de batería", xlabel="Horas", ylabel="Carga (%)")
    ax.legend(fontsize=8)


# ------------------------------------------------------------------ 2. Ecuaciones diferenciales
def sir(t, y, beta, gamma):
    s, i, r = y
    return [-beta * s * i, beta * s * i - gamma * i, gamma * i]


def caso_sir(ax) -> None:
    titulo("2. Difusión de un producto (modelo SIR + estimación de parámetros)")
    dias = np.arange(0, 60)
    real = integrate.solve_ivp(sir, (0, 59), [0.995, 0.005, 0], args=(0.42, 0.14), t_eval=dias)
    observados = np.clip(real.y[1] * RNG.lognormal(0, 0.08, dias.size), 0, None)   # «Usuarios activos» con ruido

    def residuos(param):
        beta, gamma = param
        s = integrate.solve_ivp(sir, (0, 59), [0.995, 0.005, 0], args=(beta, gamma), t_eval=dias, rtol=1e-7)
        return s.y[1] - observados

    est = optimize.least_squares(residuos, x0=[0.3, 0.1], bounds=([0, 0], [2, 1]))
    beta, gamma = est.x
    J = est.jac
    cov = np.linalg.inv(J.T @ J) * (est.fun @ est.fun) / (dias.size - 2)
    eb, eg = np.sqrt(np.diag(cov))
    print(f"   β = {beta:.3f} ± {eb:.3f} · γ = {gamma:.3f} ± {eg:.3f} · R₀ = β/γ = {beta / gamma:.2f}")

    pred = integrate.solve_ivp(sir, (0, 120), [0.995, 0.005, 0], args=(beta, gamma), dense_output=True)
    tt = np.linspace(0, 120, 600)
    s, i, r = pred.sol(tt)
    print(f"   Pico de usuarios activos: día {tt[i.argmax()]:.0f} ({i.max():.1%} de la población)")
    print(f"   Alcance final estimado: {r[-1]:.1%}")

    ax.scatter(dias, observados, s=10, color="gray", label="Observado")
    for serie, nombre, color in [(s, "Potenciales", "#7F7F7F"), (i, "Activos", "#E07B00"), (r, "Pasados", "#1F4E78")]:
        ax.plot(tt, serie, label=nombre, color=color)
    ax.set(title="2. Modelo SIR ajustado", xlabel="Días", ylabel="Proporción")
    ax.legend(fontsize=8)


# ------------------------------------------------------------------ 3. Señales
def caso_senal(ax_t, ax_f) -> None:
    titulo("3. Señal de un sensor (filtrado, picos y espectro)")
    fs = 1000
    t = np.arange(0, 3, 1 / fs)
    pulsos = sum(np.exp(-((t - c) ** 2) / (2 * 0.02 ** 2)) for c in np.arange(0.25, 3, 0.4))
    senal = pulsos + 0.4 * np.sin(2 * np.pi * 50 * t) + 0.15 * np.sin(2 * np.pi * 120 * t) + RNG.normal(0, 0.12, t.size)

    sos_notch = signal.butter(4, [47, 53], btype="bandstop", fs=fs, output="sos")
    sos_bajo = signal.butter(4, 25, btype="lowpass", fs=fs, output="sos")
    limpia = signal.sosfiltfilt(sos_bajo, signal.sosfiltfilt(sos_notch, senal))
    picos, props = signal.find_peaks(limpia, height=0.5, distance=0.2 * fs, prominence=0.4)
    intervalos = np.diff(t[picos])
    print(f"   Picos detectados: {picos.size} · intervalo medio {intervalos.mean():.3f} s "
          f"(frecuencia {60 / intervalos.mean():.0f} por minuto)")

    f = rfftfreq(t.size, 1 / fs)
    espectro, espectro_limpio = (np.abs(rfft(x)) / t.size * 2 for x in (senal, limpia))
    idx, alt = signal.find_peaks(np.where(f > 20, espectro, 0), height=0.05)   # Interferencias por encima de 20 Hz
    interferencias = f[idx[np.argsort(alt["peak_heights"])[::-1]]]
    print(f"   Interferencias detectadas en la señal original: {', '.join(f'{x:.0f} Hz' for x in interferencias)}")
    snr = 10 * np.log10(np.var(pulsos) / np.var(limpia - pulsos))
    print(f"   Relación señal/ruido tras filtrar: {snr:.1f} dB")

    ax_t.plot(t, senal, color="#B8C4D0", linewidth=0.6, label="Original")
    ax_t.plot(t, limpia, color="#1F4E78", label="Filtrada")
    ax_t.plot(t[picos], limpia[picos], "v", color="#C00000", label="Picos")
    ax_t.set(title="3a. Señal del sensor", xlabel="s", xlim=(0, 1.5))
    ax_t.legend(fontsize=8, ncols=3)
    ax_f.semilogy(f, espectro + 1e-6, color="#B8C4D0", label="Original")
    ax_f.semilogy(f, espectro_limpio + 1e-6, color="#1F4E78", label="Filtrada")
    ax_f.set(title="3b. Espectro", xlabel="Hz", xlim=(0, 200), ylim=(1e-5, 1))
    ax_f.legend(fontsize=8)


# ------------------------------------------------------------------ 4. Optimización
def caso_produccion(ax) -> None:
    titulo("4. Plan de producción (programación lineal entera)")
    productos = ["Mesa", "Silla", "Estantería", "Escritorio"]
    beneficio = np.array([45, 20, 32, 60])
    recursos = ["Carpintería (h)", "Pintura (h)", "Madera (m²)", "Montaje (h)"]
    consumo = np.array([[3.0, 1.0, 2.0, 4.0],
                        [1.0, 0.5, 1.0, 1.5],
                        [2.0, 0.8, 2.5, 3.0],
                        [1.0, 0.5, 1.5, 2.0]])
    disponible = np.array([240, 100, 200, 120])
    minimos = np.array([10, 40, 0, 5])                      # Pedidos comprometidos
    maximos = np.array([60, 120, 40, 30])                   # Demanda máxima

    res = optimize.milp(c=-beneficio, integrality=np.ones(4),
                        constraints=optimize.LinearConstraint(consumo, ub=disponible),
                        bounds=optimize.Bounds(minimos, maximos))
    plan = np.round(res.x).astype(int)
    print(f"   Estado: {res.message}")
    for nombre, u in zip(productos, plan):
        print(f"   {nombre:<11} {u:4d} unidades")
    print(f"   Beneficio máximo: {beneficio @ plan:,.0f} €".replace(",", "."))
    uso = consumo @ plan
    for nombre, u, d in zip(recursos, uso, disponible):
        print(f"   {nombre:<16} {u:6.1f} / {d} ({u / d:.0%})")

    rel = optimize.linprog(-beneficio, A_ub=consumo, b_ub=disponible, bounds=list(zip(minimos, maximos)), method="highs")
    sombra = -rel.ineqlin.marginals
    print("   Precio sombra (beneficio extra por unidad adicional de recurso, versión continua):")
    for nombre, v in zip(recursos, sombra):
        print(f"     {nombre:<16} {v:6.2f} €")

    y = np.arange(len(recursos))
    ax.barh(y, disponible, color="#E5E9EF", label="Disponible")
    ax.barh(y, uso, color="#1F4E78", label="Usado")
    ax.set_yticks(y, recursos)
    ax.set(title="4. Uso de recursos del plan óptimo")
    ax.legend(fontsize=8)


# ------------------------------------------------------------------ 5. Estadística
def caso_ab(ax) -> None:
    titulo("5. Prueba A/B de una web (contrastes de hipótesis)")
    n_a, n_b = 4800, 4750
    conv_a, conv_b = RNG.binomial(n_a, 0.052), RNG.binomial(n_b, 0.061)
    tabla = np.array([[conv_a, n_a - conv_a], [conv_b, n_b - conv_b]])
    chi2, p_chi, *_ = stats.chi2_contingency(tabla)
    pa, pb = conv_a / n_a, conv_b / n_b
    ee = np.sqrt(pa * (1 - pa) / n_a + pb * (1 - pb) / n_b)
    ic = (pb - pa) + np.array([-1, 1]) * stats.norm.ppf(0.975) * ee
    print(f"   Conversión A {pa:.2%} · B {pb:.2%} · diferencia {pb - pa:+.2%} "
          f"(IC 95 %: {ic[0]:+.2%} a {ic[1]:+.2%}) · χ² p = {p_chi:.4f}")

    importe_a = RNG.lognormal(4.0, 0.6, conv_a)
    importe_b = RNG.lognormal(4.05, 0.6, conv_b)
    print(f"   Normalidad del importe (Shapiro, muestra A): p = {stats.shapiro(importe_a[:500]).pvalue:.2e} → no normal")
    welch = stats.ttest_ind(importe_b, importe_a, equal_var=False)
    mw = stats.mannwhitneyu(importe_b, importe_a)
    bs = stats.bootstrap((importe_b, importe_a), lambda x, y: np.median(x) - np.median(y),
                         n_resamples=3000, random_state=1, vectorized=False)
    print(f"   Importe medio A {importe_a.mean():.2f} € · B {importe_b.mean():.2f} € · t de Welch p = {welch.pvalue:.3f}")
    print(f"   Mann-Whitney p = {mw.pvalue:.3f} · diferencia de medianas (bootstrap, IC 95 %): "
          f"{bs.confidence_interval.low:+.2f} € a {bs.confidence_interval.high:+.2f} €")
    ingreso_a, ingreso_b = pa * importe_a.mean(), pb * importe_b.mean()
    print(f"   Ingreso esperado por visita: A {ingreso_a:.2f} € · B {ingreso_b:.2f} € ({ingreso_b / ingreso_a - 1:+.1%})")
    decision = "Implantar B" if ic[0] > 0 else "Seguir recogiendo datos"
    print(f"   Decisión sugerida: {decision}")

    x = np.linspace(0.03, 0.08, 400)
    for conv, n, nombre, color in [(conv_a, n_a, "A", "#7F7F7F"), (conv_b, n_b, "B", "#1F4E78")]:
        ax.plot(x, stats.beta(conv + 1, n - conv + 1).pdf(x), label=f"Variante {nombre}", color=color)
    ax.set(title="5. Distribución de la tasa de conversión", xlabel="Tasa de conversión")
    ax.xaxis.set_major_formatter(matplotlib.ticker.PercentFormatter(1, decimals=1))
    ax.legend(fontsize=8)


def main() -> int:
    plt.rcParams.update({"font.size": 9, "axes.spines.top": False, "axes.spines.right": False,
                         "axes.grid": True, "grid.alpha": 0.3})
    fig, axs = plt.subplot_mosaic([["bat", "sir", "ab"], ["sen_t", "sen_f", "prod"]], figsize=(16, 9),
                                  layout="constrained")
    caso_bateria(axs["bat"])
    caso_sir(axs["sir"])
    caso_senal(axs["sen_t"], axs["sen_f"])
    caso_produccion(axs["prod"])
    caso_ab(axs["ab"])
    fig.suptitle("Laboratorio de análisis con SciPy", fontsize=15, fontweight="bold", x=0.01, ha="left")
    fig.savefig(SALIDA / "resumen.png", dpi=140)
    print(f"\n✔ Figura de resumen guardada en {SALIDA / 'resumen.png'}")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 20. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `OptimizeWarning: Covariance of the parameters could not be estimated` | Modelo mal planteado o mal punto inicial en `curve_fit`. | Proporcione `p0` razonable y `bounds`; revise el modelo. |
| `RuntimeError: Optimal parameters not found: Number of calls … reached maxfev` | No converge. | Mejore `p0`, escale los datos, aumente `maxfev`. |
| `res.success == False` en `minimize` | Mínimo no alcanzado. | Cambie de método o de punto inicial; revise la escala de las variables. |
| `ValueError: f(a) and f(b) must have different signs` | El intervalo de `brentq` no contiene una raíz. | Elija otro intervalo (represente la función). |
| `IntegrationWarning` en `quad` | Integrando con singularidades u oscilaciones. | Divida el intervalo, use `points=` o `limit=`. |
| `solve_ivp` muy lento | Problema rígido con `RK45`. | `method="Radau"`, `"BDF"` o `"LSODA"`. |
| Filtro inestable o con resultados extraños | Coeficientes `b, a` de orden alto. | `output="sos"` + `sosfiltfilt`. |
| Frecuencia de corte incorrecta | Se omitió `fs` (frecuencias normalizadas). | Indique `fs=` en el diseño del filtro. |
| `AttributeError: module 'scipy.integrate' has no attribute 'trapz'` (o `simps`) | Funciones eliminadas. | `trapezoid`, `simpson`. |
| `AttributeError: … 'binom_test'` | Eliminada. | `stats.binomtest(k, n, p).pvalue`. |
| `linprog` da un máximo negativo inesperado | `linprog` minimiza. | Cambie el signo del objetivo y del resultado. |
| p-valores «significativos» en muchos contrastes | Comparaciones múltiples. | `stats.false_discovery_control` o Bonferroni. |

---

## 21. Buenas prácticas

- Identifique el tipo de problema (minimización, raíz, ajuste, EDO…) antes de elegir la función.
- Proporcione puntos iniciales, límites y escalas razonables; compruebe siempre `success` y los mensajes.
- Represente gráficamente las funciones, los ajustes y los residuos para validar los resultados.
- Informe de la incertidumbre (errores de parámetros, intervalos de confianza), no solo del valor central.
- Use semillas (`random_state`, `default_rng`) para resultados reproducibles.
- En señales, indique siempre la frecuencia de muestreo y use filtros sin desfase (`sosfiltfilt`).
- Elija el contraste estadístico según el diseño (independiente/emparejado, normal/no normal, número de grupos) y corrija por comparaciones múltiples.
- Prefiera las APIs actuales (`*_array` en `sparse`, `make_interp_spline`, `trapezoid`).

---

## 22. Referencia rápida

```python
optimize.minimize(f, x0, method=, bounds=, constraints=) · minimize_scalar · differential_evolution
optimize.curve_fit(f, x, y, p0=, bounds=, sigma=) → p, cov · least_squares(res, x0, loss="soft_l1")
optimize.root_scalar(f, bracket=[a, b]) · brentq · root(sistema, x0) · linprog(c, A_ub, b_ub, bounds) · milp · linear_sum_assignment
integrate.quad(f, a, b) · dblquad · trapezoid · simpson · cumulative_trapezoid · solve_ivp(f, (t0, t1), y0, t_eval=, method=, events=)
interpolate.CubicSpline · PchipInterpolator · make_interp_spline · make_smoothing_spline · griddata · RBFInterpolator
stats.norm(loc, scale).pdf/cdf/ppf/rvs/interval · fit · describe · pearsonr · spearmanr · linregress · bootstrap
stats.ttest_ind(equal_var=False) · ttest_rel · mannwhitneyu · wilcoxon · f_oneway · kruskal · tukey_hsd · chi2_contingency · binomtest · shapiro
signal.butter(N, Wn, btype=, fs=, output="sos") · sosfiltfilt · savgol_filter · find_peaks · welch · spectrogram · scipy.fft.rfft/rfftfreq
linalg.solve · expm · lu · cholesky · sparse.csr_array · sparse.linalg.spsolve · csgraph.shortest_path
spatial.KDTree.query · distance.cdist · ConvexHull · ndimage.gaussian_filter · label · cluster.hierarchy.linkage/fcluster
```

---

## 23. Ejercicios propuestos

1. Ajuste un modelo logístico de crecimiento (`K / (1 + exp(-r·(t - t0)))`) a datos de adopción de un producto y estime cuándo se alcanzará el 90 % del mercado.
2. Resuelva el movimiento de un péndulo amortiguado con `solve_ivp` y compare la solución con la aproximación lineal.
3. Diseñe un filtro paso banda para extraer una frecuencia de 8–12 Hz de una señal con ruido y represente el antes y el después.
4. Resuelva un problema de dieta (coste mínimo cumpliendo requisitos nutricionales) con `linprog` e interprete los precios sombra.
5. Compare tres versiones de una campaña de correo con `chi2_contingency` y `tukey_hsd` sobre el importe, y corrija los p-valores por comparaciones múltiples.

---

## 24. Referencias

- Documentación oficial: https://docs.scipy.org/doc/scipy/
- Guía de usuario: https://docs.scipy.org/doc/scipy/tutorial/index.html
- Contrastes de hipótesis en `scipy.stats`: https://docs.scipy.org/doc/scipy/tutorial/stats/hypothesis_tests.html
- Optimización: https://docs.scipy.org/doc/scipy/tutorial/optimize.html
- Procesamiento de señales: https://docs.scipy.org/doc/scipy/tutorial/signal.html
