# Manual de uso: math

> **Categoría:** Matemáticas · **Utilidad:** Funciones matemáticas básicas
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Importación](#2-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Constantes](#4-constantes)
5. [Redondeo y partes de un número](#5-redondeo-y-partes-de-un-número)
6. [Potencias, raíces y logaritmos](#6-potencias-raíces-y-logaritmos)
7. [Trigonometría y ángulos](#7-trigonometría-y-ángulos)
8. [Funciones hiperbólicas y especiales](#8-funciones-hiperbólicas-y-especiales)
9. [Teoría de números y combinatoria](#9-teoría-de-números-y-combinatoria)
10. [Sumas, productos y distancias](#10-sumas-productos-y-distancias)
11. [Comparación de decimales y precisión](#11-comparación-de-decimales-y-precisión)
12. [Infinito, NaN y representación en coma flotante](#12-infinito-nan-y-representación-en-coma-flotante)
13. [Gestión de errores](#13-gestión-de-errores)
14. [Módulos relacionados: `cmath`, `decimal`, `fractions`, `statistics`](#14-módulos-relacionados-cmath-decimal-fractions-statistics)
15. [`math` frente a NumPy](#15-math-frente-a-numpy)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**`math`** es el módulo de la librería estándar de Python que proporciona funciones matemáticas para números **reales** (`int` y `float`): redondeo, potencias, logaritmos, trigonometría, combinatoria, teoría de números y utilidades de precisión numérica. Es una capa sobre la librería matemática de C, por lo que es rápido y fiable.

**Ventajas:**
- Incluido en Python: sin instalación ni dependencias.
- Rápido para operaciones con **escalares** (más que NumPy para valores individuales).
- Funciones de precisión numérica (`fsum`, `isclose`, `fma`) y enteros de tamaño arbitrario (`comb`, `factorial`, `isqrt`).
- Comportamiento estable y bien documentado en todas las plataformas.

**Limitaciones:**
- Solo trabaja con **escalares**: no admite listas ni arrays (use NumPy).
- No admite números complejos (use `cmath`).
- Usa coma flotante binaria (`float`): no es adecuado para importes monetarios exactos (use `decimal`).

| Si necesita… | Opción |
|---|---|
| Funciones matemáticas sobre números individuales | **math** |
| Números complejos | `cmath` |
| Cálculos monetarios exactos con redondeo controlado | `decimal` |
| Fracciones exactas | `fractions` |
| Media, mediana, desviación típica | `statistics` |
| Operaciones sobre arrays y vectores | NumPy |
| Matemáticas simbólicas | SymPy |

---

## 2. Importación

No requiere instalación.

```python
import math

math.sqrt(16)          # 4.0
```

Otras formas:

```python
from math import sqrt, pi, isclose       # Importar funciones concretas
import math as m                         # Alias (poco habitual)
```

> Se recomienda `import math` y usar el prefijo `math.`: deja claro el origen de cada función y evita confusiones con funciones de NumPy del mismo nombre.

Consultar la ayuda desde Python:

```python
help(math.comb)
dir(math)               # Lista de funciones y constantes
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `float` | Número en coma flotante de doble precisión (IEEE 754, 64 bits): unas 15–17 cifras significativas. |
| `int` | Entero de precisión arbitraria: no tiene límite de tamaño. |
| Valor devuelto | La mayoría de funciones devuelven `float`; las de redondeo y teoría de números (`floor`, `ceil`, `comb`, `gcd`, `isqrt`…) devuelven `int`. |
| Radianes | Las funciones trigonométricas trabajan en radianes, no en grados. |
| Error de dominio | `ValueError` si el argumento está fuera del dominio (`sqrt(-1)`, `log(0)`). |
| Error de rango | `OverflowError` si el resultado no cabe en un `float` (`exp(1000)`). |
| `inf` y `nan` | Infinito y «no es un número», valores especiales de IEEE 754. |
| Error de redondeo | `0.1 + 0.2` no es exactamente `0.3` porque 0,1 no tiene representación binaria exacta. |

---

## 4. Constantes

| Constante | Valor | Descripción |
|---|---|---|
| `math.pi` | 3.141592653589793 | π |
| `math.tau` | 6.283185307179586 | 2π (una vuelta completa en radianes) |
| `math.e` | 2.718281828459045 | Número de Euler |
| `math.inf` | `inf` | Infinito positivo (`-math.inf` para el negativo) |
| `math.nan` | `nan` | «No es un número» |

```python
area_circulo = math.pi * radio ** 2
longitud_circunferencia = math.tau * radio
```

---

## 5. Redondeo y partes de un número

### 5.1 Funciones de redondeo

| Función | Descripción | `2.7` | `-2.7` | `-2.5` |
|---|---|---|---|---|
| `math.floor(x)` | Entero inferior más próximo (hacia −∞) | 2 | -3 | -3 |
| `math.ceil(x)` | Entero superior más próximo (hacia +∞) | 3 | -2 | -2 |
| `math.trunc(x)` | Elimina la parte decimal (hacia 0) | 2 | -2 | -2 |
| `int(x)` | Igual que `trunc` | 2 | -2 | -2 |
| `round(x)` | Redondeo **bancario** (al par en caso de empate) | 3 | -3 | -2 |

```python
math.ceil(42 / 18)          # 3 → vehículos necesarios para 42 palés de 18 en 18
math.floor(1234.99)         # 1234
```

> **`round()` no es de `math`** y aplica redondeo al par: `round(2.5) == 2`, `round(3.5) == 4`. Además, `round(2.675, 2)` devuelve `2.67` porque 2,675 se almacena como 2,67499999… Para redondeo comercial use `decimal` (sección 14.2).

### 5.2 Redondeo a múltiplos

```python
def redondear_arriba(x: float, multiplo: float) -> float:
    return math.ceil(x / multiplo) * multiplo

redondear_arriba(23.2, 5)       # 25   (p. ej. tarifas por tramos de 5 kg)
redondear_arriba(1.31, 0.05)    # 1.35
```

### 5.3 Partes de un número

```python
math.modf(3.75)            # (0.75, 3.0)  → parte decimal y entera
math.fabs(-3)              # 3.0          → valor absoluto (siempre float); abs() mantiene el tipo
math.copysign(3, -0.0)     # -3.0         → magnitud de x con el signo de y
math.frexp(8.0)            # (0.5, 4)     → mantisa y exponente: 8 = 0,5 × 2⁴
math.ldexp(0.5, 4)         # 8.0          → inversa de frexp
```

### 5.4 Resto de la división

| Expresión | `-7, 3` | Signo del resultado |
|---|---|---|
| `x % y` | 2 | Del divisor (`y`) — recomendado con enteros |
| `math.fmod(x, y)` | -1.0 | Del dividendo (`x`) — recomendado con `float` |
| `math.remainder(x, y)` | -1.0 | Resto IEEE (al múltiplo más cercano) |
| `divmod(x, y)` | (-3, 2) | Cociente y resto a la vez |

```python
horas, minutos = divmod(155, 60)     # (2, 35)
```

---

## 6. Potencias, raíces y logaritmos

### 6.1 Potencias y raíces

```python
math.sqrt(16)          # 4.0             Raíz cuadrada (float)
math.isqrt(17)         # 4               Raíz cuadrada entera (exacta, para int de cualquier tamaño)
math.cbrt(27)          # 3.0000000000000004  Raíz cúbica (Python 3.11+)
math.pow(2, 10)        # 1024.0          Siempre devuelve float
2 ** 10                # 1024            Operador: mantiene int si ambos son int
pow(2, 10, 1000)       # 24              Potencia modular (función integrada)
math.exp(1)            # 2.718281828...  eˣ
math.exp2(10)          # 1024.0          2ˣ (Python 3.11+)
x ** (1 / n)           # Raíz n-ésima
```

> Para comprobar si un entero es un cuadrado perfecto, use `math.isqrt(n) ** 2 == n`; `math.sqrt` puede fallar con enteros grandes por la precisión limitada de `float`.

### 6.2 Logaritmos

```python
math.log(x)            # Logaritmo natural (ln)
math.log(x, base)      # Logaritmo en cualquier base: math.log(8, 2) → 3.0
math.log2(x)           # Base 2 (más preciso que log(x, 2))
math.log10(x)          # Base 10
math.log1p(x)          # ln(1 + x), preciso para x muy pequeño
math.expm1(x)          # eˣ − 1, preciso para x muy pequeño
```

**Aplicación:** plazo necesario para duplicar un capital al 4 % anual:

```python
anios = math.log(2) / math.log(1.04)          # 17.67 años
```

**Por qué existen `log1p` y `expm1`:** para tipos de interés diarios o tasas muy pequeñas, `math.log(1 + x)` pierde precisión porque `1 + x` se redondea antes de calcular el logaritmo.

```python
x = 1e-10
math.log(1 + x)        # 1.000000082690371e-10  (error en la 8.ª cifra)
math.log1p(x)          # 9.999999999500001e-11  (correcto)
```

### 6.3 Multiplicación y suma fusionadas (Python 3.13+)

```python
math.fma(x, y, z)      # x * y + z con un único redondeo (más preciso)
```

---

## 7. Trigonometría y ángulos

### 7.1 Conversión de unidades

```python
math.radians(180)      # 3.141592653589793
math.degrees(math.pi)  # 180.0
```

> Todas las funciones trigonométricas usan **radianes**. Olvidar la conversión es el error más habitual.

### 7.2 Funciones trigonométricas

| Función | Descripción |
|---|---|
| `sin(x)`, `cos(x)`, `tan(x)` | Seno, coseno y tangente |
| `asin(x)`, `acos(x)` | Inversas; dominio [-1, 1]; resultado en radianes |
| `atan(x)` | Arcotangente; resultado en (−π/2, π/2) |
| `atan2(y, x)` | Ángulo del punto (x, y) en (−π, π]; **tiene en cuenta el cuadrante** |
| `hypot(x, y, ...)` | √(x² + y² + …) sin desbordamiento |

```python
angulo = math.degrees(math.atan2(1, -1))      # 135.0  (atan(1 / -1) daría -45.0: cuadrante incorrecto)
math.hypot(3, 4)                              # 5.0
math.hypot(1, 2, 2)                           # 3.0 (varias dimensiones)
```

**Aplicación:** componentes de una fuerza o velocidad:

```python
velocidad, rumbo = 15.0, math.radians(30)
vx = velocidad * math.cos(rumbo)
vy = velocidad * math.sin(rumbo)
```

> `math.cos(math.pi / 2)` devuelve `6.123233995736766e-17`, no `0`: es el error de representación de π. Compare con `math.isclose(..., 0, abs_tol=1e-12)`.

---

## 8. Funciones hiperbólicas y especiales

### 8.1 Hiperbólicas

```python
math.sinh(x); math.cosh(x); math.tanh(x)
math.asinh(x); math.acosh(x); math.atanh(x)
```

Aplicación típica: la catenaria (forma de un cable suspendido) `y = a · cosh(x / a)`.

### 8.2 Funciones especiales

| Función | Descripción | Uso típico |
|---|---|---|
| `erf(x)` | Función de error | Probabilidades de la distribución normal |
| `erfc(x)` | 1 − erf(x), precisa para x grande | Colas de la normal |
| `gamma(x)` | Función Γ: `gamma(n) == factorial(n - 1)` | Estadística, factorial de no enteros |
| `lgamma(x)` | ln \|Γ(x)\| | Factoriales enormes sin desbordamiento |

**Función de distribución de la normal sin librerías externas:**

```python
def cdf_normal(x: float, media: float = 0.0, desviacion: float = 1.0) -> float:
    return 0.5 * (1 + math.erf((x - media) / (desviacion * math.sqrt(2))))

cdf_normal(1.96)            # 0.9750021048517796
```

> Para estadística más completa, use `statistics.NormalDist` (sección 14.4).

---

## 9. Teoría de números y combinatoria

Todas estas funciones trabajan con **enteros exactos** de cualquier tamaño.

### 9.1 Combinatoria

```python
math.factorial(5)          # 120
math.comb(49, 6)           # 13983816   Combinaciones: formas de elegir 6 de 49 (sin orden)
math.perm(5, 2)            # 20         Variaciones: formas de ordenar 2 de 5
math.perm(5)               # 120        Permutaciones de 5 elementos
```

| Pregunta | Función |
|---|---|
| ¿De cuántas formas elijo k elementos de n, sin importar el orden? | `comb(n, k)` |
| ¿De cuántas formas elijo y ordeno k elementos de n? | `perm(n, k)` |
| ¿De cuántas formas ordeno n elementos? | `factorial(n)` |

Probabilidad de acertar la lotería primitiva (6 de 49):

```python
probabilidad = 1 / math.comb(49, 6)       # 7.15e-08
```

### 9.2 Divisores y múltiplos

```python
math.gcd(12, 18, 24)       # 6    Máximo común divisor (varios argumentos desde Python 3.9)
math.lcm(4, 6, 10)         # 60   Mínimo común múltiplo (Python 3.9+)
```

**Aplicación:** simplificar una proporción o calcular cuándo coinciden dos ciclos:

```python
math.lcm(15, 20)           # 60 → dos rutas de 15 y 20 min coinciden cada 60 min
```

### 9.3 Raíz entera

```python
math.isqrt(10)             # 3  (⌊√10⌋, exacta)

def es_primo(n: int) -> bool:
    if n < 2:
        return False
    return all(n % d for d in range(2, math.isqrt(n) + 1))
```

---

## 10. Sumas, productos y distancias

### 10.1 Suma precisa: `fsum`

```python
valores = [0.1] * 10
math.fsum(valores)         # 1.0  (suma con precisión completa)
```

`math.fsum` controla el error de redondeo acumulado y garantiza el resultado correctamente redondeado. Es importante en sumas largas o con valores de magnitudes muy distintas:

```python
datos = [1e16, 1.0, -1e16]
sum(datos)                 # 0.0 en Python ≤ 3.11; 1.0 desde Python 3.12
math.fsum(datos)           # 1.0
```

> Desde Python 3.12, `sum()` usa un algoritmo compensado y es mucho más preciso con `float`, pero `math.fsum` sigue siendo la opción de máxima precisión.

### 10.2 Productos

```python
math.prod([1, 2, 3, 4])               # 24
math.prod([1.05, 1.03, 0.98])         # 1.05987 → rentabilidad acumulada
math.prod(range(1, 6), start=1)       # 120
math.sumprod([1, 2, 3], [4, 5, 6])    # 32 → producto escalar (Python 3.12+)
```

**Aplicación:** media ponderada con `sumprod`:

```python
notas, pesos = [7.5, 6.0, 9.0], [0.3, 0.3, 0.4]
media = math.sumprod(notas, pesos) / math.fsum(pesos)       # 7.65
```

### 10.3 Distancias

```python
math.dist((0, 0), (3, 4))             # 5.0  Distancia euclídea (2D, 3D o n dimensiones)
math.dist((1, 2, 3), (4, 6, 3))       # 5.0
math.hypot(3, 4)                      # 5.0  Norma de un vector
```

> `math.dist` calcula distancia en un plano. Para coordenadas geográficas (latitud/longitud) use la fórmula de Haversine (sección 16).

---

## 11. Comparación de decimales y precisión

### 11.1 Nunca compare `float` con `==`

```python
0.1 + 0.2 == 0.3                          # False
0.1 + 0.2                                 # 0.30000000000000004
math.isclose(0.1 + 0.2, 0.3)              # True
```

### 11.2 Tolerancias de `isclose`

```python
math.isclose(a, b, rel_tol=1e-9, abs_tol=0.0)
```

| Parámetro | Significado | Por defecto |
|---|---|---|
| `rel_tol` | Diferencia máxima **relativa** al mayor de los valores | `1e-9` |
| `abs_tol` | Diferencia máxima **absoluta** | `0.0` |

```python
math.isclose(1e-12, 0)                     # False: con abs_tol=0 nada es «cercano» a 0
math.isclose(1e-12, 0, abs_tol=1e-9)       # True
math.isclose(100.0, 100.4, rel_tol=0.01)   # True (diferencia del 0,4 % < 1 %)
math.isclose(cuadre, 0, abs_tol=0.005)     # Cuadre contable al céntimo
```

> **Al comparar con cero, indique siempre `abs_tol`.**

### 11.3 Precisión de `float`

```python
import sys
sys.float_info.epsilon     # 2.220446049250313e-16  Diferencia entre 1.0 y el siguiente float
sys.float_info.max         # 1.7976931348623157e+308
sys.float_info.dig         # 15  Cifras decimales fiables

math.ulp(1.0)              # 2.220446049250313e-16  Separación entre floats en ese valor (3.9+)
math.nextafter(1.0, 2.0)   # 1.0000000000000002     Siguiente float hacia 2.0 (3.9+)
```

---

## 12. Infinito, NaN y representación en coma flotante

```python
math.inf > 1e308                 # True
-math.inf < -1e308               # True
math.inf - math.inf              # nan
float("inf"), float("nan")       # Alternativa sin importar math

math.isinf(x)                    # ¿Es infinito?
math.isnan(x)                    # ¿Es NaN?
math.isfinite(x)                 # ¿Ni infinito ni NaN?
```

**NaN no es igual a nada, ni a sí mismo:**

```python
math.nan == math.nan             # False
x = math.nan
x == math.nan                    # False  → ✗ Nunca compruebe así
math.isnan(x)                    # True   → ✓
```

Uso habitual de `inf` como valor inicial en búsquedas de mínimos:

```python
mejor = math.inf
for ruta in rutas:
    if (coste := calcular(ruta)) < mejor:
        mejor = coste
```

---

## 13. Gestión de errores

| Excepción | Causa | Ejemplo |
|---|---|---|
| `ValueError: math domain error` | Argumento fuera del dominio de la función | `sqrt(-1)`, `log(0)`, `acos(2)` |
| `OverflowError: math range error` | Resultado demasiado grande para `float` | `exp(1000)` |
| `OverflowError: int too large to convert to float` | Entero demasiado grande para convertirlo | `sqrt(10**400)` |
| `TypeError` | Tipo no numérico o lista en lugar de escalar | `sqrt("4")`, `sqrt([1, 4])` |

```python
def raiz_segura(x: float) -> float | None:
    try:
        return math.sqrt(x)
    except ValueError:
        return None

def log_seguro(x: float, por_defecto: float = -math.inf) -> float:
    return math.log(x) if x > 0 else por_defecto
```

---

## 14. Módulos relacionados: `cmath`, `decimal`, `fractions`, `statistics`

### 14.1 `cmath`: números complejos

```python
import cmath

cmath.sqrt(-1)                   # 1j        (math.sqrt(-1) lanzaría ValueError)
cmath.polar(1 + 1j)              # (1.414..., 0.785...)  módulo y argumento
cmath.rect(2, math.pi / 2)       # ≈ 2j
cmath.exp(1j * math.pi)          # ≈ -1 (identidad de Euler)
```

### 14.2 `decimal`: cálculos monetarios

```python
from decimal import Decimal, ROUND_HALF_UP, getcontext

precio = Decimal("19.99")                             # Siempre desde cadena, no desde float
iva = (precio * Decimal("0.21")).quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)
Decimal("2.675").quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)    # 2.68 (round(2.675, 2) da 2.67)
getcontext().prec = 28                                # Cifras significativas
```

### 14.3 `fractions`: fracciones exactas

```python
from fractions import Fraction

Fraction(1, 3) + Fraction(1, 6)       # Fraction(1, 2)
Fraction("0.75")                      # Fraction(3, 4)
```

### 14.4 `statistics`: estadística descriptiva

```python
import statistics as st

st.mean(datos); st.median(datos); st.mode(datos); st.stdev(datos); st.quantiles(datos, n=4)
st.correlation(x, y); st.linear_regression(x, y)
st.NormalDist(mu=100, sigma=15).cdf(130)            # 0.977
```

---

## 15. `math` frente a NumPy

| Aspecto | `math` | NumPy |
|---|---|---|
| Datos | Escalares | Arrays (y escalares) |
| Instalación | Incluido | `pip install numpy` |
| Velocidad con un valor | Más rápido | Más lento (coste de llamada) |
| Velocidad con millones de valores | Lento (bucle Python) | Mucho más rápido |
| Errores de dominio | Lanza `ValueError` | Devuelve `nan` y emite un aviso |
| Enteros grandes | Precisión arbitraria | Limitados a 64 bits |

```python
math.sqrt(16)                          # Un valor → math
np.sqrt(np.array([1, 4, 9, 16]))      # Muchos valores → NumPy
[math.sqrt(x) for x in valores]       # Aceptable en listas pequeñas
```

> No mezcle `math` con arrays de NumPy: `math.sqrt(np.array([1, 4]))` lanza `TypeError`.

---

## 16. Ejemplo práctico completo

**Planificador de rutas de reparto**: a partir de las coordenadas de un almacén central y ocho clientes, calcula las distancias geodésicas con la fórmula de Haversine y el rumbo de cada destino, construye una ruta con la heurística del vecino más próximo y la mejora con el algoritmo 2-opt, y estima vehículos necesarios, consumo, coste y tiempo de reparto. Usa trigonometría (`radians`, `sin`, `cos`, `asin`, `atan2`, `degrees`), combinatoria (`factorial`, `comb`), redondeo (`ceil`), suma precisa (`fsum`) y comparación (`isclose`).

Solo requiere la librería estándar.

```python
"""
Planificador de rutas de reparto con el módulo math.

Calcula distancias geodésicas (fórmula de Haversine) entre almacenes, construye
una ruta con la heurística del vecino más próximo, la mejora con 2-opt, y
estima el coste, el número de vehículos y el tiempo de reparto.
Solo usa la librería estándar.
"""
import math
from itertools import pairwise

RADIO_TIERRA_KM = 6_371.0088              # Radio medio (IUGG)
FACTOR_CARRETERA = 1.25                   # Distancia por carretera ≈ 1,25 × línea recta
VELOCIDAD_MEDIA_KMH = 80.0
CONSUMO_L_100KM = 28.0                    # Camión de reparto
PRECIO_LITRO = 1.52                       # €/L
CAPACIDAD_VEHICULO = 18                   # Palés por vehículo
TIEMPO_DESCARGA_MIN = 25

# Almacén central y clientes: (nombre, latitud, longitud, palés)
ORIGEN = ("Madrid (central)", 40.4168, -3.7038, 0)
CLIENTES = [
    ("Zaragoza", 41.6488, -0.8891, 6),
    ("Valencia", 39.4699, -0.3763, 9),
    ("Sevilla", 37.3891, -5.9845, 7),
    ("Bilbao", 43.2630, -2.9350, 5),
    ("Valladolid", 41.6523, -4.7245, 4),
    ("Córdoba", 37.8882, -4.7794, 3),
    ("Murcia", 37.9922, -1.1307, 6),
    ("Toledo", 39.8628, -4.0273, 2),
]


# ------------------------------------------------------------------ Geometría
def haversine(lat1: float, lon1: float, lat2: float, lon2: float) -> float:
    """Distancia ortodrómica en km entre dos puntos (grados decimales)."""
    φ1, φ2 = math.radians(lat1), math.radians(lat2)
    Δφ = φ2 - φ1
    Δλ = math.radians(lon2 - lon1)
    a = math.sin(Δφ / 2) ** 2 + math.cos(φ1) * math.cos(φ2) * math.sin(Δλ / 2) ** 2
    return 2 * RADIO_TIERRA_KM * math.asin(math.sqrt(a))


def rumbo_inicial(lat1: float, lon1: float, lat2: float, lon2: float) -> float:
    """Rumbo inicial en grados (0 = norte, 90 = este)."""
    φ1, φ2 = math.radians(lat1), math.radians(lat2)
    Δλ = math.radians(lon2 - lon1)
    x = math.sin(Δλ) * math.cos(φ2)
    y = math.cos(φ1) * math.sin(φ2) - math.sin(φ1) * math.cos(φ2) * math.cos(Δλ)
    return (math.degrees(math.atan2(x, y)) + 360) % 360


def punto_cardinal(grados: float) -> str:
    puntos = ("N", "NE", "E", "SE", "S", "SO", "O", "NO")
    return puntos[round(grados / 45) % 8]


def matriz_distancias(puntos: list[tuple]) -> list[list[float]]:
    n = len(puntos)
    return [[haversine(puntos[i][1], puntos[i][2], puntos[j][1], puntos[j][2]) * FACTOR_CARRETERA
             for j in range(n)] for i in range(n)]


# ------------------------------------------------------------------ Rutas
def longitud(ruta: list[int], d: list[list[float]]) -> float:
    return math.fsum(d[a][b] for a, b in pairwise(ruta))


def vecino_mas_proximo(d: list[list[float]]) -> list[int]:
    pendientes = set(range(1, len(d)))
    ruta = [0]
    while pendientes:
        siguiente = min(pendientes, key=lambda j: d[ruta[-1]][j])
        ruta.append(siguiente)
        pendientes.remove(siguiente)
    return ruta + [0]


def mejora_2opt(ruta: list[int], d: list[list[float]]) -> list[int]:
    """Invierte tramos mientras se reduzca la longitud total."""
    mejorada = True
    while mejorada:
        mejorada = False
        for i in range(1, len(ruta) - 2):
            for j in range(i + 1, len(ruta) - 1):
                delta = (d[ruta[i - 1]][ruta[j]] + d[ruta[i]][ruta[j + 1]]
                         - d[ruta[i - 1]][ruta[i]] - d[ruta[j]][ruta[j + 1]])
                if delta < -1e-9:
                    ruta[i:j + 1] = reversed(ruta[i:j + 1])
                    mejorada = True
    return ruta


# ------------------------------------------------------------------ Informe
def formato_horas(horas: float) -> str:
    minutos_totales = math.ceil(horas * 60)
    h, m = divmod(minutos_totales, 60)
    return f"{h} h {m:02d} min"


def main() -> None:
    puntos = [ORIGEN] + CLIENTES
    d = matriz_distancias(puntos)
    n = len(CLIENTES)

    print(f"Clientes: {n} · rutas posibles: {math.factorial(n) // 2:,} "
          f"(combinaciones de 3 clientes para un vehículo: {math.comb(n, 3)})\n")

    print(f"{'Cliente':<12}{'Distancia':>11}{'Rumbo':>9}")
    for nombre, lat, lon, _ in CLIENTES:
        km = haversine(ORIGEN[1], ORIGEN[2], lat, lon)
        rumbo = rumbo_inicial(ORIGEN[1], ORIGEN[2], lat, lon)
        print(f"{nombre:<12}{km:>8.1f} km{rumbo:>6.0f}° {punto_cardinal(rumbo)}")

    inicial = vecino_mas_proximo(d)
    optima = mejora_2opt(inicial.copy(), d)
    km_inicial, km_final = longitud(inicial, d), longitud(optima, d)

    print(f"\nRuta vecino más próximo: {km_inicial:,.1f} km")
    print(f"Ruta tras 2-opt:         {km_final:,.1f} km "
          f"({(km_inicial - km_final) / km_inicial:.1%} de ahorro)")
    print("  " + " → ".join(puntos[i][0].split(" (")[0] for i in optima))

    pales = sum(c[3] for c in CLIENTES)
    vehiculos = math.ceil(pales / CAPACIDAD_VEHICULO)
    litros = km_final * CONSUMO_L_100KM / 100
    horas = km_final / VELOCIDAD_MEDIA_KMH + n * TIEMPO_DESCARGA_MIN / 60

    print(f"\nPalés: {pales} · vehículos necesarios: {vehiculos} (capacidad {CAPACIDAD_VEHICULO})")
    print(f"Combustible: {litros:,.1f} L · coste: {litros * PRECIO_LITRO:,.2f} €")
    print(f"Tiempo estimado (un vehículo): {formato_horas(horas)}")

    # Comprobación: la ida y la vuelta deben medir lo mismo
    assert all(math.isclose(d[i][j], d[j][i]) for i in range(n + 1) for j in range(n + 1))


if __name__ == "__main__":
    main()
```

Resultado:

```
Clientes: 8 · rutas posibles: 20,160 (combinaciones de 3 clientes para un vehículo: 56)

Cliente       Distancia    Rumbo
Zaragoza       272.9 km    59° NE
Valencia       302.6 km   109° E
Sevilla        390.2 km   211° SO
Bilbao         322.8 km    11° N
Valladolid     161.9 km   328° NO
Córdoba        296.1 km   199° S
Murcia         349.0 km   140° SE
Toledo          67.5 km   204° SO

Ruta vecino más próximo: 2,506.7 km
Ruta tras 2-opt:         2,366.1 km (5.6% de ahorro)
  Madrid → Toledo → Sevilla → Córdoba → Murcia → Valencia → Zaragoza → Bilbao → Valladolid → Madrid

Palés: 42 · vehículos necesarios: 3 (capacidad 18)
Combustible: 662.5 L · coste: 1,007.00 €
Tiempo estimado (un vehículo): 32 h 55 min
```

> `itertools.pairwise` requiere Python 3.10 o superior. Las distancias son en línea recta corregidas con un factor de carretera; para distancias reales por carretera se necesita un servicio de rutas.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `0.1 + 0.2 == 0.3` devuelve `False` | Representación binaria de los decimales. | Use `math.isclose()` o `decimal`. |
| `math.isclose(x, 0)` siempre `False` | La tolerancia relativa no sirve para comparar con cero. | Indique `abs_tol`: `isclose(x, 0, abs_tol=1e-9)`. |
| Resultados trigonométricos absurdos | Se pasaron grados en lugar de radianes. | Convierta con `math.radians()`. |
| `ValueError: math domain error` | `sqrt` de negativo, `log` de 0 o negativo, `acos`/`asin` fuera de [-1, 1]. | Valide el dominio; use `cmath` para complejos; recorte con `max(-1, min(1, x))` en `acos`. |
| `OverflowError: math range error` | `exp()` o `pow()` con resultado > 1,8e308. | Trabaje en escala logarítmica (`lgamma`, sumas de logaritmos). |
| `TypeError: must be real number, not list` | Se pasó una lista o un array. | Use un bucle/comprensión o NumPy. |
| `round(2.5)` devuelve 2 | Redondeo bancario (al par). | Use `Decimal.quantize(..., ROUND_HALF_UP)`. |
| `round(2.675, 2)` devuelve 2.67 | 2,675 se almacena como 2,67499… | Use `decimal` con valores creados desde cadena. |
| `x == math.nan` siempre `False` | NaN no es igual a nada. | Use `math.isnan(x)`. |
| `-7 % 3` da 2 y `math.fmod(-7, 3)` da -1.0 | Convenios de signo distintos. | Elija conscientemente: `%` para enteros, `fmod` para `float`. |
| `math.floor(-2.5)` da -3 | `floor` redondea hacia −∞, no hacia cero. | Use `math.trunc()` para eliminar decimales. |
| `AttributeError: module 'math' has no attribute 'cbrt'` | Función de una versión más reciente de Python. | Compruebe la versión mínima (tabla de la sección 19). |

---

## 18. Buenas prácticas

- Use `import math` y el prefijo `math.` para que el origen de cada función sea evidente.
- **Nunca compare decimales con `==`**: use `math.isclose()` con tolerancias adecuadas al problema.
- Use **`decimal`** para importes monetarios y **`math`** para cálculos científicos y técnicos.
- Convierta siempre los ángulos a radianes antes de usar funciones trigonométricas y documente las unidades en el nombre de la variable (`angulo_rad`, `rumbo_grados`).
- Prefiera `math.fsum` en sumas largas, `log1p`/`expm1` para valores pequeños, `hypot` en lugar de `sqrt(x**2 + y**2)` y `atan2` en lugar de `atan(y / x)`.
- Use las funciones enteras (`isqrt`, `comb`, `gcd`) cuando trabaje con enteros: son exactas.
- Valide los argumentos o capture `ValueError` y `OverflowError` en funciones expuestas a datos externos.
- Para colecciones grandes de valores, pase a NumPy.
- Defina las constantes del problema con nombre (`RADIO_TIERRA_KM`) en lugar de números sueltos en el código.

---

## 19. Referencia rápida

```python
import math

# Constantes
math.pi · math.tau · math.e · math.inf · math.nan

# Redondeo y partes
math.floor(x) · math.ceil(x) · math.trunc(x) · math.modf(x) · math.fabs(x) · math.copysign(x, y)
math.fmod(x, y) · math.remainder(x, y) · math.frexp(x) · math.ldexp(m, e)

# Potencias y logaritmos
math.sqrt(x) · math.isqrt(n) · math.cbrt(x) · math.pow(x, y) · math.exp(x) · math.exp2(x) · math.expm1(x)
math.log(x[, base]) · math.log2(x) · math.log10(x) · math.log1p(x) · math.fma(x, y, z)

# Trigonometría
math.sin · cos · tan · asin · acos · atan · atan2(y, x) · hypot(*coords) · radians(g) · degrees(r)
math.sinh · cosh · tanh · asinh · acosh · atanh

# Especiales
math.erf(x) · math.erfc(x) · math.gamma(x) · math.lgamma(x)

# Enteros y combinatoria
math.factorial(n) · math.comb(n, k) · math.perm(n, k) · math.gcd(*n) · math.lcm(*n)

# Sumas, productos y distancias
math.fsum(it) · math.prod(it) · math.sumprod(a, b) · math.dist(p, q)

# Comparación y coma flotante
math.isclose(a, b, rel_tol=1e-9, abs_tol=0) · isfinite(x) · isinf(x) · isnan(x) · ulp(x) · nextafter(x, y)
```

Versión mínima de Python de las funciones más recientes:

| Función | Versión |
|---|---|
| `comb`, `perm`, `prod`, `isqrt`, `dist`, `hypot` con n argumentos | 3.8 |
| `lcm`, `gcd` con n argumentos, `ulp`, `nextafter` | 3.9 |
| `cbrt`, `exp2` | 3.11 |
| `sumprod` | 3.12 |
| `fma` | 3.13 |

---

## 20. Ejercicios propuestos

1. Escriba una calculadora de préstamos que obtenga la cuota mensual (sistema francés) y, con logaritmos, el número de meses necesario para amortizar un préstamo dada una cuota máxima.
2. Implemente una función que convierta coordenadas cartesianas en polares y viceversa, y compruebe con `isclose` que la conversión de ida y vuelta devuelve el punto original.
3. Calcule la probabilidad de obtener exactamente k éxitos en n intentos (distribución binomial) con `math.comb` y compárela con `statistics.NormalDist` para n grande.
4. Programe la criba de Eratóstenes usando `math.isqrt` y compare su rendimiento con la función `es_primo` de la sección 9.3.
5. Amplíe el ejemplo completo para repartir los clientes entre los vehículos necesarios según su capacidad, de forma que cada uno tenga su propia ruta optimizada.

---

## 21. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/math.html
- `cmath`: https://docs.python.org/es/3/library/cmath.html
- `decimal`: https://docs.python.org/es/3/library/decimal.html
- Aritmética de coma flotante, problemas y limitaciones: https://docs.python.org/es/3/tutorial/floatingpoint.html
