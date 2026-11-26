# Manual de uso: SymPy

> **Categoría:** Matemáticas y cálculo · **Utilidad:** Matemáticas simbólicas y álgebra
> **Tipo:** Librería externa (requiere instalación con `pip`). Escrita íntegramente en Python.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Símbolos y expresiones](#4-símbolos-y-expresiones)
5. [Simplificación y manipulación algebraica](#5-simplificación-y-manipulación-algebraica)
6. [Resolución de ecuaciones](#6-resolución-de-ecuaciones)
7. [Cálculo: límites, derivadas, integrales y series](#7-cálculo-límites-derivadas-integrales-y-series)
8. [Ecuaciones diferenciales](#8-ecuaciones-diferenciales)
9. [Álgebra lineal (matrices)](#9-álgebra-lineal-matrices)
10. [Evaluación numérica y `lambdify`](#10-evaluación-numérica-y-lambdify)
11. [Teoría de números, combinatoria y sumas](#11-teoría-de-números-combinatoria-y-sumas)
12. [Transformadas, lógica y conjuntos](#12-transformadas-lógica-y-conjuntos)
13. [Presentación: LaTeX, impresión y gráficos](#13-presentación-latex-impresión-y-gráficos)
14. [Generación de código y física](#14-generación-de-código-y-física)
15. [Rendimiento y buenas costumbres](#15-rendimiento-y-buenas-costumbres)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**SymPy** es una librería de **cálculo simbólico** (álgebra computacional, CAS) para Python. A diferencia del cálculo numérico, que trabaja con aproximaciones (`0.3333…`), SymPy manipula expresiones matemáticas **exactas**: `1/3`, `√2`, `π`, `x² + 2x + 1`. Permite simplificar, factorizar, derivar, integrar, resolver ecuaciones y sistemas, calcular límites y series, trabajar con matrices simbólicas, resolver ecuaciones diferenciales y generar código o LaTeX a partir de los resultados.

**Ventajas:**
- Resultados exactos y fórmulas cerradas, no solo números.
- Escrita en Python puro: sin dependencias complejas (solo `mpmath`).
- Cubre álgebra, cálculo, álgebra lineal, ecuaciones diferenciales, teoría de números, combinatoria, lógica, geometría y física.
- Exporta a LaTeX, MathML, C, Fortran, JavaScript y funciones NumPy (`lambdify`).
- Se integra con Jupyter (salida matemática formateada), NumPy, SciPy y Matplotlib.
- Licencia BSD: uso libre, también comercial.

**Limitaciones:**
- Mucho más lenta que el cálculo numérico: no está pensada para procesar grandes volúmenes de datos.
- Algunas integrales, ecuaciones o simplificaciones no tienen forma cerrada o SymPy no la encuentra.
- Las expresiones pueden crecer enormemente (expresiones «hinchadas») en problemas grandes.

| Si necesita… | Opción |
|---|---|
| Fórmulas exactas, derivación/integración simbólica, álgebra | **SymPy** |
| Cálculo numérico, optimización, integración numérica, estadística | SciPy / NumPy |
| Derivadas automáticas de código numérico | JAX |
| Operaciones matemáticas básicas sobre números | `math` |
| Cálculo simbólico muy intensivo | SageMath, Mathematica, Maple |

---

## 2. Instalación e importación

```bash
pip install sympy
```

Comprobar la versión:

```bash
python -c "import sympy; print(sympy.__version__)"
```

Importaciones habituales:

```python
import sympy as sp                    # Recomendado en scripts y proyectos
from sympy import symbols, sin, cos, exp, pi, sqrt, Rational, diff, integrate, solve   # Importación selectiva
from sympy import *                   # Cómodo en Jupyter y sesiones interactivas
```

Consola interactiva preparada (símbolos `x`, `y`, `z`, `t`, `k`, `m`, `n`, `f`, `g`, `h` ya definidos):

```bash
isympy
```

En Jupyter, para mostrar fórmulas con formato matemático:

```python
sp.init_printing()
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Símbolo (`Symbol`) | Variable matemática (`x`, `y`) que no tiene valor numérico. |
| Expresión | Combinación de símbolos, números y funciones: `x**2 + sin(y)`. Es un **árbol** de objetos. |
| Exactitud | `Rational(1, 3)`, `sqrt(2)`, `pi` se mantienen exactos hasta que se pide una aproximación. |
| Inmutabilidad | Las expresiones no se modifican: cada operación devuelve una expresión nueva. |
| Supuestos (*assumptions*) | Propiedades de un símbolo (`positive`, `real`, `integer`) que permiten simplificar más. |
| Evaluación | `subs()` sustituye valores; `evalf()`/`N()` obtiene un número decimal. |
| `lambdify` | Convierte una expresión en una función numérica rápida (NumPy). |
| `Eq` | Ecuación (`Eq(a, b)`); `==` compara estructuras, **no** crea ecuaciones. |

**El árbol de una expresión:**

```python
expr = x**2 + 2*x + 1
sp.srepr(expr)          # "Add(Pow(Symbol('x'), Integer(2)), Mul(Integer(2), Symbol('x')), Integer(1))"
expr.args               # (1, 2*x, x**2)
expr.func               # <class 'sympy.core.add.Add'>
expr.free_symbols       # {x}
```

---

## 4. Símbolos y expresiones

### 4.1 Definir símbolos

```python
import sympy as sp

x, y, z = sp.symbols("x y z")
t = sp.Symbol("t")
a, b, c = sp.symbols("a:c")                     # a, b, c
x1, x2, x3 = sp.symbols("x1:4")                 # x1, x2, x3
alfa, beta = sp.symbols("alpha beta")           # Se muestran como letras griegas en LaTeX
```

### 4.2 Supuestos

```python
r = sp.Symbol("r", positive=True)
n = sp.Symbol("n", integer=True, nonnegative=True)
w = sp.Symbol("w", real=True)

sp.sqrt(r**2)            # r          (sin el supuesto sería sqrt(r**2))
sp.sqrt(w**2)            # Abs(w)
r.is_positive            # True
(n + 1).is_integer       # True
```

| Supuesto | Significado |
|---|---|
| `real`, `complex` | Número real / complejo. |
| `positive`, `negative`, `nonnegative`, `nonzero` | Signo. |
| `integer`, `rational`, `prime`, `even`, `odd` | Tipo de número. |
| `finite` | Finito. |

> Indique los supuestos que conoce del problema (longitudes positivas, índices enteros): SymPy simplificará mucho mejor.

### 4.3 Números exactos

```python
sp.Rational(1, 3) + sp.Rational(1, 6)    # 1/2
sp.Integer(1) / 3                         # 1/3  (exacto)
1 / 3                                     # 0.333…  (¡float de Python!)
sp.sqrt(8)                                # 2*sqrt(2)
sp.pi, sp.E, sp.I, sp.oo                  # π, e, unidad imaginaria, infinito
sp.nsimplify(0.333333333333)              # 1/3     (de decimal a exacto)
sp.S(1) / 3                               # Forma corta de Integer(1)/3
sp.S("1/3 + x")                           # Convertir texto a expresión (sympify)
```

> **Cuidado con `1/2` en Python:** `x + 1/2` crea `x + 0.5` (decimal). Use `x + sp.Rational(1, 2)` o `x + sp.S(1)/2`.

### 4.4 Funciones matemáticas

```python
sp.sin(x), sp.cos(x), sp.tan(x), sp.asin(x), sp.atan2(y, x)
sp.exp(x), sp.log(x), sp.log(x, 10), sp.sqrt(x), sp.root(x, 3), sp.Abs(x)
sp.sinh(x), sp.factorial(n), sp.binomial(n, 2), sp.gamma(x), sp.floor(x), sp.ceiling(x)
sp.Piecewise((0, x < 0), (x, x <= 1), (1, True))      # Función a trozos
sp.Heaviside(x), sp.DiracDelta(x), sp.Max(x, y), sp.Min(x, 0)
```

Funciones no definidas (para ecuaciones diferenciales o fórmulas genéricas):

```python
f = sp.Function("f")
f(x).diff(x)               # Derivative(f(x), x)
```

### 4.5 Sustitución y evaluación

```python
expr = x**2 + 3*x*y
expr.subs(x, 2)                          # 6*y + 4
expr.subs({x: 2, y: sp.Rational(1, 2)})  # 7
expr.subs(x, sp.sin(t))                  # Sustituir por otra expresión
sp.pi.evalf(30)                          # 3.14159265358979323846264338328
sp.N(sp.sqrt(2), 10)                     # 1.414213562
expr.evalf(subs={x: 1.5, y: 2})          # 11.25
```

---

## 5. Simplificación y manipulación algebraica

| Función | Efecto | Ejemplo → resultado |
|---|---|---|
| `simplify` | Intenta obtener la forma «más simple» (heurística). | `sin(x)**2 + cos(x)**2` → `1` |
| `expand` | Desarrolla productos y potencias. | `(x + 1)**3` → `x**3 + 3*x**2 + 3*x + 1` |
| `factor` | Factoriza polinomios. | `x**3 - 1` → `(x - 1)*(x**2 + x + 1)` |
| `collect` | Agrupa por potencias de una variable. | `collect(a*x + b*x + c, x)` → `c + x*(a + b)` |
| `cancel` | Simplifica fracciones racionales. | `(x**2 - 1)/(x - 1)` → `x + 1` |
| `apart` | Fracciones simples. | `1/(x**2 - 1)` → `-1/(2*(x + 1)) + 1/(2*(x - 1))` |
| `together` | Une en una fracción. | `1/x + 1/(x + 1)` → `(2*x + 1)/(x*(x + 1))` |
| `trigsimp` | Simplificación trigonométrica. | `sin(x)*cos(x)` → `sin(2*x)/2` |
| `expand_trig` | Desarrolla fórmulas trigonométricas. | `sin(2*x)` → `2*sin(x)*cos(x)` |
| `powsimp`, `expand_power_exp` | Potencias. | `x**a * x**b` → `x**(a + b)` |
| `expand_log`, `logcombine` | Logaritmos (requieren supuestos positivos). | `log(a*b)` → `log(a) + log(b)` |
| `radsimp`, `nsimplify` | Radicales / decimal a exacto. | `1/(1 + sqrt(2))` → `-1 + sqrt(2)` |

```python
expr = (x**2 + 2*x + 1) / (x + 1)
sp.simplify(expr)                      # x + 1
sp.factor(x**2 - 5*x + 6)              # (x - 3)*(x - 2)
sp.expand((x + y)**2)                  # x**2 + 2*x*y + y**2
sp.collect(sp.expand((x + 2)*(x + a)), x)
```

> `simplify()` prueba muchas estrategias y puede ser lenta o no dar la forma deseada. Si sabe qué transformación necesita, use la función específica (`factor`, `cancel`, `trigsimp`…).

Comparar expresiones:

```python
(x + 1)**2 == x**2 + 2*x + 1                       # False: compara estructura
sp.simplify((x + 1)**2 - (x**2 + 2*x + 1)) == 0    # True: comparación matemática
((x + 1)**2).equals(x**2 + 2*x + 1)                # True
```

Polinomios:

```python
p = sp.Poly(3*x**3 - 2*x + 5, x)
p.degree(), p.coeffs(), p.all_coeffs()     # 3, [3, -2, 5], [3, 0, -2, 5]
sp.div(x**3 + 1, x + 1)                    # (x**2 - x + 1, 0)  cociente y resto
sp.gcd(x**2 - 1, x**2 - 2*x + 1)           # x - 1
sp.roots(x**3 - 6*x**2 + 11*x - 6)         # {3: 1, 2: 1, 1: 1}  raíz: multiplicidad
```

---

## 6. Resolución de ecuaciones

### 6.1 `solve` y `solveset`

```python
sp.solve(x**2 - 5*x + 6, x)                     # [2, 3]   (se iguala a 0)
sp.solve(sp.Eq(x**2, 4), x)                     # [-2, 2]
sp.solve(a*x**2 + b*x + c, x)                   # Fórmula general
sp.solveset(x**2 + 1, x)                        # {-I, I}  (dominio complejo)
sp.solveset(x**2 + 1, x, domain=sp.S.Reals)     # EmptySet
sp.solveset(sp.sin(x), x, domain=sp.S.Reals)    # Union(ImageSet(…2*n*pi…), ImageSet(…2*n*pi + pi…))
sp.solveset(x**2 - 4 > 0, x, sp.S.Reals)        # Union(Interval.open(-oo, -2), Interval.open(2, oo))
```

| Función | Devuelve | Cuándo usarla |
|---|---|---|
| `solve` | Lista o diccionarios. | Uso general, sistemas, despejar variables. |
| `solveset` | Conjunto matemático (incluye infinitas soluciones e intervalos). | Una ecuación o inecuación con dominio explícito. |
| `linsolve` | Conjunto de soluciones de sistemas lineales. | Sistemas lineales. |
| `nonlinsolve` | Conjunto de soluciones de sistemas no lineales. | Sistemas polinómicos. |
| `nsolve` | Solución numérica cerca de un punto inicial. | Cuando no hay solución cerrada. |

### 6.2 Sistemas

```python
sp.solve([x + y - 3, x - y - 1], [x, y])                 # {x: 2, y: 1}
sp.solve([x + y - 2, x*y - 1], [x, y], dict=True)         # [{x: 1, y: 1}]
sp.linsolve([x + y - 3, x - y - 1], x, y)                 # {(2, 1)}
sp.nonlinsolve([x**2 + y**2 - 4, x - y], x, y)            # {(-sqrt(2), -sqrt(2)), (sqrt(2), sqrt(2))}
```

### 6.3 Despejar una variable de una fórmula

```python
v, v0, g, t = sp.symbols("v v0 g t", positive=True)
sp.solve(sp.Eq(v, v0 - g*t), t)                           # [(-v + v0)/g]
```

### 6.4 Soluciones numéricas

```python
sp.nsolve(sp.cos(x) - x, x, 1)                            # 0.739085133215161
sp.nsolve([x**2 + y**2 - 4, sp.exp(x) - y], [x, y], [1, 1])
```

### 6.5 Recurrencias

```python
n = sp.Symbol("n", integer=True)
a = sp.Function("a")
sp.rsolve(a(n + 2) - a(n + 1) - a(n), a(n), {a(0): 0, a(1): 1})   # Fórmula de Binet (Fibonacci)
```

---

## 7. Cálculo: límites, derivadas, integrales y series

### 7.1 Límites

```python
sp.limit(sp.sin(x) / x, x, 0)                 # 1
sp.limit(1 / x, x, 0, dir="+")                # oo
sp.limit((1 + 1/x)**x, x, sp.oo)              # E
```

### 7.2 Derivadas

```python
sp.diff(x**3 * sp.sin(x), x)                  # x**3*cos(x) + 3*x**2*sin(x)
sp.diff(x**4, x, 2)                           # 12*x**2  (segunda derivada)
sp.diff(x**2 * y**3, x, y)                    # 6*x*y**2  (derivada mixta)
expr = x**2 * y
[expr.diff(v) for v in (x, y)]                # Gradiente: [2*x*y, x**2]
sp.Derivative(x**2, x)                        # Sin evaluar (útil para mostrar); .doit() la calcula
sp.idiff(x**2 + y**2 - 1, y, x)               # Derivación implícita: -x/y
```

Puntos críticos y clasificación:

```python
f = x**3 - 3*x
criticos = sp.solve(f.diff(x), x)                        # [-1, 1]
[(c, f.diff(x, 2).subs(x, c)) for c in criticos]         # [(-1, -6), (1, 6)] → máximo en -1, mínimo en 1
```

Gradiente, hessiana y jacobiana:

```python
f = x**2 + x*y + y**2
sp.Matrix([f]).jacobian([x, y])               # Matrix([[2*x + y, x + 2*y]])
sp.hessian(f, (x, y))                         # Matrix([[2, 1], [1, 2]])
```

### 7.3 Integrales

```python
sp.integrate(x * sp.exp(x), x)                    # (x - 1)*exp(x)   (indefinida, sin constante)
sp.integrate(x**2, (x, 0, 3))                     # 9
sp.integrate(sp.exp(-x**2), (x, -sp.oo, sp.oo))   # sqrt(pi)
sp.integrate(x*y, (x, 0, 1), (y, 0, 2))           # 1  (integral doble)
sp.Integral(sp.sin(x)**2, x)                      # Sin evaluar; .doit() / .evalf()
sp.Integral(sp.exp(-x**3), (x, 0, 1)).evalf()     # Numérica si no hay primitiva cerrada
```

> Si SymPy no encuentra la primitiva, devuelve un objeto `Integral` sin evaluar. Puede calcular su valor numérico con `.evalf()`.

### 7.4 Series y sumas

```python
sp.series(sp.exp(x), x, 0, 5)                  # 1 + x + x**2/2 + x**3/6 + x**4/24 + O(x**5)
sp.series(sp.exp(x), x, 0, 5).removeO()        # Polinomio de Taylor (sin el término O)
sp.series(sp.sin(x)/x, x, 0, 6)
k = sp.Symbol("k", integer=True, positive=True)
sp.summation(1 / k**2, (k, 1, sp.oo))           # pi**2/6
sp.summation(k, (k, 1, n))                      # n**2/2 + n/2
sp.product(k, (k, 1, 5))                        # 120
```

---

## 8. Ecuaciones diferenciales

### 8.1 Ecuaciones diferenciales ordinarias (`dsolve`)

```python
t = sp.Symbol("t")
y = sp.Function("y")

sp.dsolve(y(t).diff(t) + 2*y(t), y(t))                         # Eq(y(t), C1*exp(-2*t))
sp.dsolve(sp.Eq(y(t).diff(t), -2*y(t)), y(t), ics={y(0): 5})   # Eq(y(t), 5*exp(-2*t))
sp.dsolve(y(t).diff(t, 2) + y(t), y(t))                        # Eq(y(t), C1*sin(t) + C2*cos(t))

# Con condiciones iniciales sobre la derivada
sol = sp.dsolve(y(t).diff(t, 2) + 4*y(t), y(t), ics={y(0): 1, y(t).diff(t).subs(t, 0): 0})
sol.rhs                                                         # cos(2*t)
```

Sistemas de EDO:

```python
x_, y_ = sp.symbols("x y", cls=sp.Function)
sp.dsolve([sp.Eq(x_(t).diff(t), y_(t)), sp.Eq(y_(t).diff(t), -x_(t))])
```

Comprobar una solución:

```python
ecuacion = y(t).diff(t, 2) + y(t)
sp.checkodesol(ecuacion, sp.dsolve(ecuacion, y(t)))            # (True, 0)
sp.classify_ode(ecuacion, y(t))                                # Métodos aplicables
```

### 8.2 Ecuaciones en derivadas parciales

```python
u = sp.Function("u")
x, y = sp.symbols("x y")
sp.pdsolve(u(x, y).diff(x) + u(x, y).diff(y), u(x, y))        # Eq(u(x, y), F(x - y))
```

> Para EDO sin solución cerrada o con datos numéricos, use `scipy.integrate.solve_ivp` (véase el manual de SciPy), por ejemplo combinado con `lambdify`.

---

## 9. Álgebra lineal (matrices)

```python
M = sp.Matrix([[2, 1], [1, 2]])
v = sp.Matrix([1, 2])                      # Vector columna

M.T, M.det(), M.inv(), M.rank(), M.trace()          # Traspuesta, determinante (3), inversa, rango, traza
M * v                                       # Producto matriz-vector
M**2                                        # Potencia
M.eigenvals()                               # {3: 1, 1: 1}  valor propio: multiplicidad
M.eigenvects()                              # Vectores propios
P, D = M.diagonalize()                      # M = P*D*P⁻¹
M.rref()                                    # Forma escalonada reducida y columnas pivote
M.nullspace(), M.columnspace()
M.LUsolve(sp.Matrix([3, 3]))                # Resolver M·x = b
sp.eye(3), sp.zeros(2, 3), sp.ones(2), sp.diag(1, 2, 3)
```

Matrices simbólicas:

```python
a, b, c, d = sp.symbols("a b c d")
A = sp.Matrix([[a, b], [c, d]])
A.det()                                     # a*d - b*c
A.inv()                                     # Inversa en función de a, b, c, d
A.charpoly().as_expr()                      # Polinomio característico
```

Sistemas lineales en forma matricial:

```python
A, b_ = sp.linear_eq_to_matrix([x + 2*y - 4, 3*x - y - 5], [x, y])
A.solve(b_)                                 # Matrix([[2], [1]])
```

---

## 10. Evaluación numérica y `lambdify`

Evaluar una expresión con `subs`/`evalf` en muchos puntos es muy lento. `lambdify` convierte la expresión en una función Python que usa NumPy:

```python
import numpy as np

expr = sp.sin(x) * sp.exp(-x / 5)
f = sp.lambdify(x, expr, modules="numpy")
xs = np.linspace(0, 20, 1_000_000)
ys = f(xs)                                  # Vectorizado: milisegundos

g = sp.lambdify((x, y), x**2 + y**2)        # Varias variables
g(3, 4)                                     # 25

sp.lambdify(x, expr, modules="math")        # Sin NumPy (escalares)
sp.lambdify(x, expr, modules="scipy")       # Funciones especiales de SciPy
sp.lambdify(x, expr, modules="jax")         # Para JAX (diferenciable y compilable)
```

Uso típico: deducir una fórmula con SymPy y resolver numéricamente con SciPy.

```python
from scipy.optimize import minimize
coste = (x - 2)**2 + (y + 1)**2 + x*y
f = sp.lambdify([(x, y)], coste)                          # Recibe un vector
grad = sp.lambdify([(x, y)], [coste.diff(v) for v in (x, y)])
minimize(f, x0=[0, 0], jac=grad).x
```

Precisión arbitraria con `mpmath`:

```python
sp.N(sp.pi, 100)                            # 100 cifras
sp.N(sp.sqrt(2) - 1.4142135623730951, 50)
```

---

## 11. Teoría de números, combinatoria y sumas

```python
sp.isprime(97)                     # True
sp.factorint(360)                  # {2: 3, 3: 2, 5: 1}  → 2³·3²·5
list(sp.primerange(1, 20))         # [2, 3, 5, 7, 11, 13, 17, 19]
sp.nextprime(100), sp.prime(10)    # 101, 29
sp.gcd(48, 180), sp.lcm(4, 6)      # 12, 12
sp.divisors(28)                    # [1, 2, 4, 7, 14, 28]
sp.totient(36)                     # 12
sp.mod_inverse(3, 11)              # 4
from sympy.ntheory import discrete_log
sp.binomial(10, 3), sp.factorial(10), sp.fibonacci(30)
from sympy.utilities.iterables import permutations, combinations, partitions
sp.diophantine(3*x + 5*y - 7)      # Soluciones enteras de ecuaciones diofánticas
```

---

## 12. Transformadas, lógica y conjuntos

### 12.1 Transformadas

```python
s, k = sp.symbols("s k")
sp.laplace_transform(sp.sin(t), t, s)             # (1/(s**2 + 1), 0, True)  (resultado, abscisa, condiciones)
sp.laplace_transform(sp.sin(t), t, s, noconds=True)   # 1/(s**2 + 1)
sp.inverse_laplace_transform(1 / (s + 2), s, t)   # exp(-2*t)*Heaviside(t)
sp.fourier_transform(sp.exp(-x**2), x, k)         # sqrt(pi)*exp(-pi**2*k**2)
```

### 12.2 Lógica

```python
from sympy.logic.boolalg import And, Or, Not, Implies, simplify_logic
p, q, r = sp.symbols("p q r")
simplify_logic((p & q) | (p & ~q))                # p
sp.satisfiable(p & ~q)                            # {p: True, q: False}
```

### 12.3 Conjuntos e intervalos

```python
I = sp.Interval(0, 5)                    # [0, 5]
J = sp.Interval.open(3, 8)               # (3, 8)
I.intersect(J), I.union(J), I - J
sp.FiniteSet(1, 2, 3).contains(2)
sp.S.Reals, sp.S.Integers, sp.S.Naturals
```

---

## 13. Presentación: LaTeX, impresión y gráficos

### 13.1 Impresión

```python
sp.pprint(sp.Integral(sp.sqrt(1/x), x))   # Impresión «bonita» en texto
sp.latex(sp.Integral(x**2, x))            # '\\int x^{2}\\, dx'
sp.latex(expr, mode="equation")
sp.mathml(expr)
str(expr), repr(expr), sp.srepr(expr)
```

En Jupyter, `sp.init_printing()` muestra las expresiones con formato matemático automáticamente.

Nombres en LaTeX personalizados:

```python
v0 = sp.Symbol("v_0")                          # Subíndice
theta_dot = sp.Symbol(r"\dot{\theta}")         # Cualquier código LaTeX
```

### 13.2 Gráficos rápidos

```python
sp.plot(sp.sin(x) / x, (x, -10, 10), title="sinc", ylabel="y")
sp.plot(x**2, x**3, (x, -2, 2), legend=True)
sp.plot_implicit(sp.Eq(x**2 + y**2, 4))
sp.plotting.plot3d(sp.sin(x) * sp.cos(y), (x, -3, 3), (y, -3, 3))
sp.plot_parametric(sp.cos(t), sp.sin(2*t), (t, 0, 2*sp.pi))
```

> Para gráficos de calidad, use `lambdify` + Matplotlib: más control y rapidez.

---

## 14. Generación de código y física

### 14.1 Generar código

```python
from sympy.utilities.codegen import codegen
sp.ccode(sp.sin(x)**2 + sp.sqrt(y))          # 'sqrt(y) + pow(sin(x), 2)'
sp.fcode(expr), sp.jscode(expr), sp.pycode(expr), sp.octave_code(expr), sp.rust_code(expr)
codegen(("funcion", x**2 + y), "C99", "mi_modulo")   # Archivos .c y .h

# Simplificar antes con subexpresiones comunes
reemplazos, reducida = sp.cse([sp.sin(x)**2 + sp.cos(x), sp.sin(x)**3])
```

### 14.2 Módulo de física

```python
from sympy.physics.units import meter, second, kilogram, newton, convert_to, kilometer, hour
convert_to(100 * kilometer / hour, meter / second)        # 250*meter/(9*second)

from sympy.physics.mechanics import dynamicsymbols, LagrangesMethod, Particle, Point, ReferenceFrame
from sympy.physics.vector import ReferenceFrame
from sympy.physics.quantum import Ket, Bra
```

| Submódulo | Uso |
|---|---|
| `sympy.physics.units` | Unidades y conversiones (para unidades en cálculos numéricos, véase Pint). |
| `sympy.physics.mechanics` | Mecánica clásica: Lagrange, Kane, sistemas multicuerpo. |
| `sympy.physics.vector` | Vectores y marcos de referencia. |
| `sympy.physics.quantum`, `optics`, `continuum_mechanics` | Mecánica cuántica, óptica, vigas. |
| `sympy.stats` | Variables aleatorias simbólicas (`Normal`, `E`, `variance`, `P`). |
| `sympy.geometry` | Puntos, rectas, círculos, polígonos. |

Ejemplo de estadística simbólica:

```python
from sympy.stats import Normal, E, variance, P
X = Normal("X", 0, 1)
P(X > 1).evalf()                       # 0.158655253931457
E(X**2)                                # 1
```

---

## 15. Rendimiento y buenas costumbres

| Problema | Solución |
|---|---|
| `simplify` lenta | Use funciones específicas (`factor`, `cancel`, `trigsimp`) o simplifique por partes. |
| Expresiones enormes | Añada supuestos, sustituya valores numéricos antes, use `cse`. |
| Evaluar en muchos puntos | `lambdify` con NumPy, nunca `subs` en un bucle. |
| `solve` no termina | Pruebe `solveset`, `nsolve` o reduzca el problema (sustituya parámetros). |
| Integral sin forma cerrada | `Integral(...).evalf()` o `scipy.integrate.quad`. |
| Repetir un cálculo costoso | Guárdelo: `sp.srepr`/`pickle` o `functools.lru_cache`. |

---

## 16. Ejemplo práctico completo

**Estudio analítico de un sistema masa-muelle-amortiguador y diseño óptimo de un depósito.** El script:

1. Deduce la ecuación del movimiento, la resuelve simbólicamente para los tres regímenes (subamortiguado, crítico y sobreamortiguado) y calcula la frecuencia natural, el factor de amortiguamiento y el tiempo de establecimiento.
2. Obtiene la función de transferencia con la transformada de Laplace.
3. Resuelve un problema de optimización: dimensiones de un depósito cilíndrico de volumen fijo con coste mínimo (tapas más caras que el lateral), verificando que es un mínimo.
4. Estudia el error de aproximar la respuesta por su polinomio de Taylor.
5. Convierte las fórmulas en funciones numéricas (`lambdify`), genera gráficos y un informe en Markdown con las fórmulas en LaTeX.

```python
"""
Estudio simbólico con SymPy: oscilador amortiguado y depósito de coste mínimo.
Requiere: pip install sympy numpy matplotlib
"""
from pathlib import Path

import matplotlib

matplotlib.use("Agg")
import matplotlib.pyplot as plt
import numpy as np
import sympy as sp

SALIDA = Path(__file__).resolve().parent / "salida_sympy"


# ------------------------------------------------------------------ 1. Oscilador amortiguado
def estudio_oscilador() -> dict:
    t = sp.Symbol("t", nonnegative=True)
    m, k, c, x0 = sp.symbols("m k c x_0", positive=True)
    x = sp.Function("x")

    # Segunda ley de Newton: m·x'' + c·x' + k·x = 0
    ecuacion = sp.Eq(m * x(t).diff(t, 2) + c * x(t).diff(t) + k * x(t), 0)
    wn = sp.sqrt(k / m)                                   # Frecuencia natural
    zeta = c / (2 * sp.sqrt(k * m))                       # Factor de amortiguamiento

    # Parámetros base: m = 1 kg, k = 16 N/m → ωn = 4 rad/s; x(0) = x0, x'(0) = 0
    valores = {m: 1, k: 16}
    casos = {"subamortiguado": 2, "crítico": 8, "sobreamortiguado": 20}     # Valores de c (N·s/m)
    soluciones = {}
    for nombre, valor_c in casos.items():
        ecuacion_caso = ecuacion.subs({**valores, c: valor_c})
        sol = sp.dsolve(ecuacion_caso, x(t), ics={x(0): x0, x(t).diff(t).subs(t, 0): 0})
        soluciones[nombre] = {"c": valor_c, "zeta": zeta.subs({**valores, c: valor_c}),
                              "x": sp.simplify(sol.rhs)}
        assert sp.checkodesol(ecuacion_caso, sol)[0], f"Solución incorrecta en el caso {nombre}"

    # Tiempo de establecimiento (criterio del 2 %) para el caso subamortiguado: ts ≈ 4 / (ζ·ωn)
    z_sub = soluciones["subamortiguado"]["zeta"]
    ts = sp.Rational(4) / (z_sub * wn.subs(valores))

    # Función de transferencia con fuerza de entrada F(t): X(s)/F(s)
    s = sp.Symbol("s")
    X, F = sp.symbols("X F")
    laplace = sp.Eq(m * s**2 * X + c * s * X + k * X, F)               # Condiciones iniciales nulas
    G = sp.simplify(sp.solve(laplace, X)[0] / F)
    polos = sp.solve(sp.denom(G), s)

    return {"ecuacion": ecuacion, "wn": wn, "zeta": zeta, "soluciones": soluciones, "t": t, "x0": x0,
            "ts": ts, "G": G, "polos": polos}


# ------------------------------------------------------------------ 2. Depósito de coste mínimo
def deposito_optimo() -> dict:
    r, h, V, p_lat, p_tapa = sp.symbols("r h V p_l p_t", positive=True)
    volumen = sp.Eq(sp.pi * r**2 * h, V)
    h_de_r = sp.solve(volumen, h)[0]                                   # h = V / (π r²)
    coste = p_lat * 2 * sp.pi * r * h + p_tapa * 2 * sp.pi * r**2      # Lateral + dos tapas
    coste_r = sp.simplify(coste.subs(h, h_de_r))

    r_opt = sp.solve(sp.diff(coste_r, r), r)[0]
    segunda = sp.simplify(sp.diff(coste_r, r, 2).subs(r, r_opt))
    assert segunda.is_positive, "El punto crítico no es un mínimo"
    h_opt = sp.simplify(h_de_r.subs(r, r_opt))
    relacion = sp.simplify(h_opt / r_opt)                              # h/r en el óptimo

    datos = {V: 2, p_lat: 30, p_tapa: 45}                              # 2 m³; €/m² de lateral y tapas
    return {"coste_r": coste_r, "r_opt": r_opt, "h_opt": h_opt, "relacion": relacion, "segunda": segunda,
            "numerico": {"r": float(r_opt.subs(datos)), "h": float(h_opt.subs(datos)),
                         "coste": float(coste_r.subs(datos).subs(r, r_opt.subs(datos)))},
            "simbolos": (r, V, p_lat, p_tapa), "datos": datos}


# ------------------------------------------------------------------ 3. Error de Taylor
def error_taylor(sol_sub: sp.Expr, t: sp.Symbol, x0: sp.Symbol) -> dict:
    respuesta = sol_sub.subs(x0, 1)
    resultados = {}
    for orden in (2, 4, 6):
        aprox = sp.series(respuesta, t, 0, orden + 1).removeO()
        error = sp.Abs(respuesta - aprox).subs(t, sp.Rational(1, 4))
        resultados[orden] = {"polinomio": sp.expand(aprox), "error_t025": float(error.evalf())}
    return resultados


# ------------------------------------------------------------------ 4. Gráficos e informe
def graficos(osc: dict, dep: dict) -> None:
    t = osc["t"]
    tiempo = np.linspace(0, 4, 600)
    fig, axs = plt.subplots(1, 2, figsize=(13, 4.5), layout="constrained")
    for nombre, datos in osc["soluciones"].items():
        f = sp.lambdify(t, datos["x"].subs(osc["x0"], 1), "numpy")
        valores = np.broadcast_to(f(tiempo), tiempo.shape)
        axs[0].plot(tiempo, valores, lw=2, label=f"{nombre} (c = {datos['c']}, ζ = {float(datos['zeta']):.2f})")
    axs[0].axhline(0, color="gray", lw=0.8)
    axs[0].set(title="Respuesta libre del oscilador (x₀ = 1 m)", xlabel="t (s)", ylabel="x (m)")
    axs[0].legend(frameon=False)

    r, V, p_lat, p_tapa = dep["simbolos"]
    coste = sp.lambdify(r, dep["coste_r"].subs(dep["datos"]), "numpy")
    radios = np.linspace(0.3, 1.6, 300)
    axs[1].plot(radios, coste(radios), lw=2, color="#1F4E78")
    axs[1].scatter([dep["numerico"]["r"]], [dep["numerico"]["coste"]], color="#C00000", zorder=3,
                   label=f"Óptimo: r = {dep['numerico']['r']:.3f} m, coste = {dep['numerico']['coste']:.0f} €")
    axs[1].set(title="Coste del depósito de 2 m³ según el radio", xlabel="r (m)", ylabel="Coste (€)")
    axs[1].legend(frameon=False)
    for ax in axs:
        ax.grid(alpha=0.3)
    fig.savefig(SALIDA / "graficos.png", dpi=150)
    plt.close(fig)


def informe(osc: dict, dep: dict, taylor: dict) -> str:
    L = sp.latex
    lineas = [
        "# Informe de cálculo simbólico", "",
        "## 1. Oscilador masa-muelle-amortiguador", "",
        f"Ecuación del movimiento: $${L(osc['ecuacion'])}$$",
        f"Frecuencia natural: $\\omega_n = {L(osc['wn'])}$ · Factor de amortiguamiento: $\\zeta = {L(osc['zeta'])}$", "",
        "| Régimen | c (N·s/m) | ζ | x(t) |", "|---|---|---|---|",
    ]
    for nombre, datos in osc["soluciones"].items():
        lineas.append(f"| {nombre} | {datos['c']} | ${L(datos['zeta'])}$ | ${L(datos['x'])}$ |")
    lineas += [
        "", f"Tiempo de establecimiento (2 %, caso subamortiguado): $t_s = {L(osc['ts'])}$ s",
        "", f"Función de transferencia: $$G(s) = \\frac{{X(s)}}{{F(s)}} = {L(osc['G'])}$$",
        f"Polos: ${', '.join(L(p) for p in osc['polos'])}$", "",
        "## 2. Depósito cilíndrico de coste mínimo", "",
        f"Coste en función del radio: $$C(r) = {L(dep['coste_r'])}$$",
        f"Radio óptimo: $$r^* = {L(dep['r_opt'])}$$",
        f"Altura óptima: $$h^* = {L(dep['h_opt'])}$$ · Relación $h^*/r^* = {L(dep['relacion'])}$",
        f"Segunda derivada en el óptimo: ${L(dep['segunda'])} > 0$ → mínimo.", "",
        f"Para V = 2 m³, 30 €/m² el lateral y 45 €/m² las tapas: r = {dep['numerico']['r']:.3f} m, "
        f"h = {dep['numerico']['h']:.3f} m, coste = {dep['numerico']['coste']:.2f} €.", "",
        "## 3. Aproximación de Taylor (caso subamortiguado, x₀ = 1)", "",
        "| Orden | Polinomio | Error en t = 0,25 s |", "|---|---|---|",
    ]
    for orden, datos in taylor.items():
        lineas.append(f"| {orden} | ${L(datos['polinomio'])}$ | {datos['error_t025']:.2e} |")
    lineas += ["", "![Gráficos](graficos.png)", ""]
    return "\n".join(lineas)


def main() -> None:
    SALIDA.mkdir(exist_ok=True)
    osc = estudio_oscilador()
    dep = deposito_optimo()
    taylor = error_taylor(osc["soluciones"]["subamortiguado"]["x"], osc["t"], osc["x0"])
    graficos(osc, dep)
    (SALIDA / "informe.md").write_text(informe(osc, dep, taylor), encoding="utf-8")

    print("Oscilador (subamortiguado):", osc["soluciones"]["subamortiguado"]["x"])
    print("Tiempo de establecimiento:", osc["ts"], "s")
    print("Función de transferencia:", osc["G"])
    print("Depósito: r* =", dep["r_opt"], "| h*/r* =", dep["relacion"])
    print("Depósito (2 m³):", {k: round(v, 3) for k, v in dep["numerico"].items()})
    print("Errores de Taylor en t = 0,25 s:", {o: f"{d['error_t025']:.1e}" for o, d in taylor.items()})
    print(f"Informe y gráficos en: {SALIDA}")


if __name__ == "__main__":
    main()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `NameError: name 'x' is not defined` | No se creó el símbolo. | `x = sp.symbols("x")`. |
| Aparecen decimales (`0.5*x`) en lugar de fracciones | Se usó `1/2` de Python. | `sp.Rational(1, 2)` o `sp.S(1)/2`. |
| `sqrt(x**2)` no se simplifica a `x` | Sin supuestos, `x` puede ser negativo o complejo. | `x = sp.Symbol("x", positive=True)`. |
| `==` devuelve `False` con expresiones equivalentes | Compara estructura, no valor matemático. | `sp.simplify(a - b) == 0` o `a.equals(b)`. |
| `solve` devuelve `[]` | No hay solución en el dominio o SymPy no la encuentra. | `solveset` con dominio; `nsolve` para numérica. |
| `solve` tarda muchísimo | Problema grande o no polinómico. | Sustituya parámetros numéricos; `nsolve`. |
| `integrate` devuelve `Integral(...)` | No existe primitiva cerrada (o no la encuentra). | `.evalf()` o integración numérica. |
| `TypeError: can't convert expression to float` | Se pasó una expresión simbólica a `math` o NumPy. | `float(expr.evalf())` o `lambdify`. |
| `TypeError: cannot determine truth value of Relational` | `if x > 0:` con `x` simbólico. | Use supuestos o `sp.Piecewise`; evalúe antes. |
| `lambdify` devuelve un escalar con un arreglo | La expresión es constante. | `np.broadcast_to(f(xs), xs.shape)` o `np.full_like`. |
| `SympifyError` | Texto no interpretable en `sympify`. | Revise la sintaxis (`**` para potencias, `*` explícito). |
| Variables de Python «sobrescriben» símbolos | `x = 3` tras definir `x` como símbolo. | Use nombres distintos para valores numéricos. |

---

## 18. Buenas prácticas

- Importe como `import sympy as sp` en scripts; reserve `from sympy import *` para sesiones interactivas.
- Defina los símbolos con los **supuestos** del problema (positivos, reales, enteros).
- Mantenga los números exactos (`Rational`, `sqrt`, `pi`) hasta el final y evalúe con `evalf`/`N` solo cuando necesite decimales.
- Use `Eq` para ecuaciones y `subs` con diccionarios para sustituir varios valores.
- Prefiera funciones de simplificación específicas a `simplify` en expresiones grandes.
- Verifique los resultados: `checkodesol`, sustituir soluciones en la ecuación, comparar con valores numéricos.
- Pase a numérico con `lambdify` para evaluar, graficar u optimizar.
- Genere LaTeX directamente de las expresiones para informes y documentación.
- Combine SymPy (deducción de fórmulas) con SciPy/NumPy (cálculo intensivo).

---

## 19. Referencia rápida

```python
import sympy as sp
x, y = sp.symbols("x y", real=True);  f = sp.Function("f");  sp.Rational(1, 3);  sp.pi, sp.E, sp.I, sp.oo
expr.subs({x: 1}) / expr.evalf(n) / sp.N(expr, n) / sp.sympify("x**2 + 1") / expr.free_symbols
sp.simplify / expand / factor / collect(expr, x) / cancel / apart / together / trigsimp / nsimplify
sp.solve(expr, x) / sp.solve([e1, e2], [x, y], dict=True) / sp.solveset(e, x, sp.S.Reals) / sp.linsolve / sp.nsolve(e, x, x0)
sp.limit(e, x, 0) / sp.diff(e, x, n) / sp.integrate(e, x) / sp.integrate(e, (x, a, b)) / sp.series(e, x, 0, n).removeO()
sp.summation(e, (k, 1, sp.oo)) / sp.dsolve(ode, f(x), ics={f(0): 1}) / sp.checkodesol / sp.rsolve
sp.Matrix([[..]]).det() / inv() / eigenvals() / eigenvects() / diagonalize() / rref() / sp.hessian(f, (x, y))
sp.laplace_transform(e, t, s, noconds=True) / inverse_laplace_transform / fourier_transform
sp.isprime / factorint / primerange / gcd / lcm / binomial / factorial
sp.lambdify((x, y), expr, "numpy") / sp.latex(expr) / sp.pprint(expr) / sp.ccode(expr) / sp.plot(expr, (x, a, b))
```

---

## 20. Ejercicios propuestos

1. Calcule la derivada, los puntos críticos, los puntos de inflexión y las asíntotas de `f(x) = (x² - 1)/(x² - 4)`.
2. Resuelva simbólicamente el tiro parabólico: alcance máximo en función del ángulo y ángulo óptimo.
3. Obtenga los valores y vectores propios de una matriz 3×3 simbólica simétrica y compruebe la diagonalización.
4. Resuelva el circuito RLC serie con `dsolve` y obtenga la condición de amortiguamiento crítico en función de R, L y C.
5. Amplíe el ejemplo completo para añadir una fuerza externa sinusoidal y estudiar la resonancia (amplitud en función de la frecuencia).

---

## 21. Referencias

- Documentación oficial: https://docs.sympy.org/latest/
- Tutorial introductorio: https://docs.sympy.org/latest/tutorials/intro-tutorial/index.html
- Guía de resolución de ecuaciones: https://docs.sympy.org/latest/guides/solving/index.html
- SymPy Live / Gamma (pruebas en línea): https://www.sympygamma.com/
- Meurer, A. et al. (2017). *SymPy: symbolic computing in Python*. PeerJ Computer Science 3:e103.
