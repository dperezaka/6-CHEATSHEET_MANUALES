# Manual de uso: QuantLib

> **Categoría:** Finanzas cuantitativas · **Utilidad:** Matemáticas financieras
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** BSD modificada · **Núcleo:** C++ con interfaz Python (SWIG)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Fechas, periodos y calendarios](#4-fechas-periodos-y-calendarios)
5. [Convenciones de cómputo de días y calendarios de pagos](#5-convenciones-de-cómputo-de-días-y-calendarios-de-pagos)
6. [Tipos de interés](#6-tipos-de-interés)
7. [Cotizaciones, *handles* y el patrón observador](#7-cotizaciones-handles-y-el-patrón-observador)
8. [Curvas de tipos](#8-curvas-de-tipos)
9. [Índices de referencia](#9-índices-de-referencia)
10. [Bonos](#10-bonos)
11. [Swaps de tipos de interés](#11-swaps-de-tipos-de-interés)
12. [Opciones sobre acciones](#12-opciones-sobre-acciones)
13. [Análisis de riesgo y sensibilidades](#13-análisis-de-riesgo-y-sensibilidades)
14. [Integración con pandas y otras herramientas](#14-integración-con-pandas-y-otras-herramientas)
15. [Rendimiento y organización del código](#15-rendimiento-y-organización-del-código)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**QuantLib** es la librería de código abierto de referencia para **finanzas cuantitativas**: modelado, valoración y gestión de riesgos de instrumentos financieros. Está escrita en C++ y se usa desde Python mediante el paquete `QuantLib`. La utilizan bancos, gestoras, aseguradoras, auditoras y universidades.

**Qué permite hacer:**
- Gestionar fechas financieras: calendarios de mercado, días hábiles, convenciones de cómputo de días.
- Construir curvas de tipos de interés a partir de cotizaciones de mercado (*bootstrapping*).
- Valorar bonos, swaps, FRA, *caps*/*floors*, *swaptions*, opciones sobre acciones, divisas y materias primas, instrumentos de crédito e inflación.
- Calcular rentabilidades, duraciones, convexidades, griegas y sensibilidades.
- Aplicar modelos estocásticos: Black-Scholes, Heston, Hull-White, Vasicek, SABR, entre otros.
- Usar métodos numéricos: árboles binomiales, diferencias finitas y Monte Carlo.

**Ventajas:**
- Cobertura muy amplia de instrumentos y modelos, revisados por la comunidad durante más de 20 años.
- Rendimiento de C++.
- Manejo riguroso de convenciones de mercado (calendarios, ajustes de fechas, bases de cálculo).
- Arquitectura reactiva: al cambiar una cotización, todos los valores dependientes se recalculan.

**Limitaciones:**
- Curva de aprendizaje pronunciada: requiere conocer tanto la librería como los conceptos financieros.
- La documentación está orientada a C++; la interfaz Python tiene poca documentación propia.
- Los mensajes de error de la capa SWIG (`Wrong number or type of arguments`) son poco descriptivos.
- La API evoluciona: algunas firmas cambian entre versiones.
- No incluye datos de mercado: debe proporcionarlos desde su propia fuente.

| Si necesita… | Opción |
|---|---|
| Valoración profesional de renta fija, derivados y curvas | **QuantLib** |
| Cálculos financieros sencillos (VAN, TIR, cuotas de préstamos) | `numpy-financial` |
| Análisis de series de precios y carteras | pandas, `PyPortfolioOpt` |
| Backtesting de estrategias | `backtrader`, `vectorbt` |
| Modelos propios con diferenciación automática | JAX |

---

## 2. Instalación e importación

```bash
pip install QuantLib            # Ruedas precompiladas para Windows, macOS y Linux
```

> El nombre del paquete es **`QuantLib`** (con mayúsculas). El antiguo paquete `QuantLib-Python` está obsoleto.

Comprobar la instalación:

```bash
python -c "import QuantLib as ql; print(ql.__version__)"
```

Importación habitual:

```python
import QuantLib as ql
```

> Por convención, siempre se usa el alias `ql`. Los nombres de las clases coinciden con los de la API de C++, por lo que la documentación de C++ sirve también como referencia en Python.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Fecha de evaluación | «Hoy» para todos los cálculos: `ql.Settings.instance().evaluationDate`. **Debe fijarse siempre.** |
| `Date` / `Period` | Fecha y plazo (`ql.Period(6, ql.Months)`, `ql.Period("5Y")`). |
| `Calendar` | Calendario de días hábiles de un mercado (TARGET, Reino Unido, EE. UU.…). |
| `DayCounter` | Convención de cómputo de días (Act/360, 30/360, Act/Act…). |
| `Schedule` | Calendario de fechas de pago de un instrumento. |
| `Quote` | Cotización de mercado que puede cambiar (`SimpleQuote`). |
| `Handle` | Referencia a un objeto que puede sustituirse sin reconstruir los dependientes. |
| `TermStructure` | Estructura temporal: curva de tipos, de volatilidad o de crédito. |
| `RateHelper` | Instrumento de mercado usado para construir una curva por *bootstrapping*. |
| `Index` | Índice de referencia (Euríbor, €STR, SOFR…). |
| `Instrument` | Producto financiero: bono, swap, opción… |
| `PricingEngine` | Método de valoración que se asigna a un instrumento (analítico, árbol, Monte Carlo…). |
| `StochasticProcess` | Proceso que describe la dinámica del subyacente (Black-Scholes, Heston…). |

**Flujo de trabajo típico:**

```
Fecha de evaluación ─► Datos de mercado (Quotes) ─► Curvas (TermStructures)
        ─► Instrumento ─► Motor de valoración ─► NPV, rentabilidad, sensibilidades
```

---

## 4. Fechas, periodos y calendarios

### 4.1 Fechas

```python
hoy = ql.Date(26, 11, 2026)                    # Día, mes, año
ql.Date(26, ql.November, 2026)                 # Con constante de mes
ql.Date("2026-12-31", "%Y-%m-%d")              # Desde texto
ql.Date.todaysDate()                           # Fecha del sistema

hoy.dayOfMonth(), hoy.month(), hoy.year()
hoy.weekday()                                  # 1 = domingo … 7 = sábado
hoy.ISO()                                      # '2026-11-26'
hoy.serialNumber()                             # Número de serie (compatible con Excel)
ql.Date.isLeap(2028)                           # True
ql.Date.endOfMonth(hoy)
```

### 4.2 Aritmética de fechas

```python
hoy + 30                                       # 30 días naturales
hoy + ql.Period(6, ql.Months)                  # May 26th, 2027
hoy + ql.Period("1Y")
ql.Date(31, 12, 2026) - hoy                    # Días entre fechas (int)
```

### 4.3 Conversión con Python

```python
import datetime as dt

hoy.to_date()                                  # datetime.date(2026, 11, 26)
ql.Date.from_date(dt.date(2027, 1, 15))        # Date(15, 1, 2027)
```

### 4.4 Periodos

```python
ql.Period(3, ql.Months); ql.Period("18M"); ql.Period("2Y"); ql.Period("1W"); ql.Period("2D")
ql.Period(ql.Semiannual)                       # 6M a partir de una frecuencia
```

| Unidad | Constante |
|---|---|
| Días | `ql.Days` |
| Semanas | `ql.Weeks` |
| Meses | `ql.Months` |
| Años | `ql.Years` |

### 4.5 Calendarios

```python
cal = ql.TARGET()                              # Calendario de la zona euro (TARGET2)
cal.isBusinessDay(ql.Date(25, 12, 2026))       # False
cal.adjust(ql.Date(25, 12, 2026))              # December 28th, 2026 (siguiente hábil)
cal.advance(hoy, 2, ql.Days)                   # 2 días hábiles después (fecha valor)
cal.advance(hoy, ql.Period("3M"), ql.ModifiedFollowing)
cal.businessDaysBetween(hoy, ql.Date(31, 12, 2026))
cal.holidayList(ql.Date(1, 1, 2027), ql.Date(31, 12, 2027))
```

Calendarios habituales:

| Calendario | Uso |
|---|---|
| `ql.TARGET()` | Mercado monetario y de derivados en euros |
| `ql.UnitedKingdom()`, `ql.UnitedStates(ql.UnitedStates.GovernmentBond)` | Mercados británico y estadounidense |
| `ql.JointCalendar(c1, c2)` | Días hábiles en ambos mercados |
| `ql.WeekendsOnly()`, `ql.NullCalendar()` | Solo fines de semana / sin festivos |

> QuantLib no incluye un calendario nacional de España: en operaciones en euros se usa TARGET. Para festivos locales, cree un `ql.BespokeCalendar` y añada los festivos con `addHoliday()`.

### 4.6 Convenciones de ajuste

| Convención | Si la fecha cae en día inhábil… |
|---|---|
| `ql.Following` | Siguiente día hábil |
| `ql.ModifiedFollowing` | Siguiente hábil, salvo que cambie de mes (entonces el anterior) — **la más habitual** |
| `ql.Preceding` | Día hábil anterior |
| `ql.ModifiedPreceding` | Anterior, salvo que cambie de mes |
| `ql.Unadjusted` | No se ajusta |

---

## 5. Convenciones de cómputo de días y calendarios de pagos

### 5.1 Convenciones de cómputo de días (*day count*)

Determinan la fracción de año entre dos fechas:

```python
dc = ql.Actual360()
dc.yearFraction(hoy, hoy + 180)               # 0.5
dc.dayCount(hoy, hoy + ql.Period("1Y"))       # Días según la convención
```

| Convención | Clase | Uso típico |
|---|---|---|
| Act/360 | `ql.Actual360()` | Mercado monetario en euros, Euríbor, €STR |
| Act/365 (Fixed) | `ql.Actual365Fixed()` | Curvas cero, libra esterlina, opciones |
| 30/360 | `ql.Thirty360(ql.Thirty360.BondBasis)` | Pata fija de swaps en euros, bonos corporativos |
| 30E/360 | `ql.Thirty360(ql.Thirty360.European)` | Eurobonos |
| Act/Act (ICMA) | `ql.ActualActual(ql.ActualActual.ISMA)` | Bonos de deuda pública europea |
| Act/Act (ISDA) | `ql.ActualActual(ql.ActualActual.ISDA)` | Swaps, algunos bonos |

### 5.2 Calendarios de pagos (`Schedule`)

```python
calendario_pagos = ql.Schedule(
    ql.Date(15, 3, 2026),           # Fecha de inicio
    ql.Date(15, 3, 2031),           # Fecha de vencimiento
    ql.Period(ql.Annual),           # Frecuencia
    ql.TARGET(),                    # Calendario
    ql.Unadjusted,                  # Ajuste de las fechas de pago
    ql.Unadjusted,                  # Ajuste de la fecha final
    ql.DateGeneration.Backward,     # Generación desde el vencimiento hacia atrás
    False,                          # Fin de mes
)
list(calendario_pagos)               # Lista de fechas
```

Forma abreviada con argumentos con nombre:

```python
calendario_pagos = ql.MakeSchedule(inicio, vencimiento, ql.Period("6M"),
                                   calendar=ql.TARGET(), convention=ql.ModifiedFollowing,
                                   backwards=True)
```

| Regla de generación | Descripción |
|---|---|
| `Backward` | Desde el vencimiento hacia atrás (el periodo irregular queda al principio). Habitual en bonos. |
| `Forward` | Desde el inicio hacia delante. |
| `Zero` | Sin fechas intermedias. |
| `ThirdWednesday`, `IMM`, `CDS` | Fechas estándar de futuros y CDS. |

### 5.3 Frecuencias

`ql.Annual`, `ql.Semiannual`, `ql.Quarterly`, `ql.Monthly`, `ql.Once`, `ql.NoFrequency`.

---

## 6. Tipos de interés

La clase `InterestRate` combina un tipo con su convención de cómputo, capitalización y frecuencia:

```python
r = ql.InterestRate(0.05, ql.Actual365Fixed(), ql.Compounded, ql.Annual)

r.compoundFactor(2.0)                     # 1.1025 → (1 + 5 %)²
r.discountFactor(2.0)                     # 0.907
r.equivalentRate(ql.Continuous, ql.NoFrequency, 1.0).rate()   # 0.04879 → tipo continuo equivalente
ql.InterestRate.impliedRate(1.1025, ql.Actual365Fixed(), ql.Compounded, ql.Annual, 2.0)
```

| Capitalización | Fórmula del factor | Uso |
|---|---|---|
| `ql.Simple` | 1 + r·t | Mercado monetario (< 1 año) |
| `ql.Compounded` | (1 + r/f)^(f·t) | Bonos, TIR |
| `ql.Continuous` | e^(r·t) | Modelos teóricos, curvas cero |
| `ql.SimpleThenCompounded` | Simple hasta un periodo, compuesta después | Algunas curvas de mercado |

---

## 7. Cotizaciones, *handles* y el patrón observador

QuantLib es **reactivo**: los objetos observan a los datos de los que dependen y se recalculan cuando estos cambian.

### 7.1 Cotizaciones

```python
tipo = ql.SimpleQuote(0.03)
tipo.value()                      # 0.03
tipo.setValue(0.035)              # Todos los objetos que dependan de él se marcan para recalcular
```

### 7.2 *Handles*

Un *handle* es una referencia indirecta: los objetos que lo usan no necesitan reconstruirse si se cambia el objeto al que apunta.

```python
handle_tipo = ql.QuoteHandle(tipo)
curva = ql.FlatForward(hoy, handle_tipo, ql.Actual365Fixed())
curva.discount(1.0)               # 0.97045
tipo.setValue(0.04)
curva.discount(1.0)               # 0.96079 → actualizada automáticamente
```

| Clase | Contiene |
|---|---|
| `ql.QuoteHandle` | Una cotización |
| `ql.YieldTermStructureHandle` | Una curva de tipos |
| `ql.RelinkableYieldTermStructureHandle` | Curva que puede sustituirse con `linkTo()` |
| `ql.BlackVolTermStructureHandle` | Una superficie de volatilidad |

```python
curva_relinkable = ql.RelinkableYieldTermStructureHandle()
indice = ql.Euribor6M(curva_relinkable)
curva_relinkable.linkTo(curva_nueva)       # El índice usa ahora la nueva curva
```

> Este mecanismo permite construir el modelo **una sola vez** y analizar escenarios cambiando solo las cotizaciones.

### 7.3 Cotizaciones derivadas

```python
base = ql.SimpleQuote(0.025)
diferencial = ql.SimpleQuote(0.0)
cotizacion = ql.CompositeQuote(ql.QuoteHandle(base), ql.QuoteHandle(diferencial), lambda a, b: a + b)
```

---

## 8. Curvas de tipos

### 8.1 Curva plana

```python
curva = ql.FlatForward(hoy, 0.03, ql.Actual365Fixed())                # Tipo fijo
curva = ql.FlatForward(2, ql.TARGET(), ql.QuoteHandle(tipo), ql.Actual365Fixed())  # Con liquidación
```

### 8.2 Curva a partir de tipos cero o factores de descuento

```python
fechas = [hoy, hoy + ql.Period("1Y"), hoy + ql.Period("5Y")]
curva_cero = ql.ZeroCurve(fechas, [0.020, 0.022, 0.026], ql.Actual365Fixed(),
                          ql.TARGET(), ql.Linear(), ql.Continuous)
curva_fd = ql.DiscountCurve(fechas, [1.0, 0.978, 0.878], ql.Actual365Fixed())
```

### 8.3 Consultas sobre una curva

```python
curva.discount(fecha)                                             # Factor de descuento
curva.discount(2.5)                                               # Por fracción de año
curva.zeroRate(fecha, ql.Actual365Fixed(), ql.Continuous).rate()  # Tipo cero
curva.forwardRate(f1, f2, ql.Actual360(), ql.Simple).rate()       # Tipo a plazo entre f1 y f2
curva.referenceDate(); curva.maxDate()
curva.enableExtrapolation()                                       # Permitir consultas más allá del último nodo
```

### 8.4 Construcción por *bootstrapping*

A partir de cotizaciones de mercado (depósitos, FRA, futuros, swaps), QuantLib obtiene la curva que reproduce exactamente sus precios:

```python
euribor6m = ql.Euribor6M()
helpers = [
    ql.DepositRateHelper(ql.QuoteHandle(ql.SimpleQuote(0.0230)), ql.Period("6M"), 2,
                         ql.TARGET(), ql.ModifiedFollowing, False, ql.Actual360()),
    ql.SwapRateHelper(ql.QuoteHandle(ql.SimpleQuote(0.0257)), ql.Period("5Y"), ql.TARGET(),
                      ql.Annual, ql.Unadjusted, ql.Thirty360(ql.Thirty360.BondBasis), euribor6m),
    # ... más plazos
]
curva = ql.PiecewiseLogCubicDiscount(2, ql.TARGET(), helpers, ql.Actual365Fixed())
```

| Helper | Instrumento |
|---|---|
| `DepositRateHelper` | Depósitos interbancarios |
| `FraRateHelper` | FRA (*forward rate agreements*) |
| `FuturesRateHelper` | Futuros sobre tipos de interés |
| `SwapRateHelper` | Swaps de tipo fijo contra variable |
| `OISRateHelper` | Swaps sobre índices a un día (€STR, SOFR) |
| `FixedRateBondHelper` | Bonos (curvas de deuda pública) |

| Clase de curva | Interpolación |
|---|---|
| `PiecewiseLogCubicDiscount` | Cúbica sobre el logaritmo del factor de descuento (suave, habitual) |
| `PiecewiseLogLinearDiscount` | Lineal sobre el log del descuento (tipos *forward* constantes por tramos) |
| `PiecewiseLinearZero` | Lineal sobre tipos cero |
| `PiecewiseFlatForward` | *Forward* plano por tramos |

> Desde la reforma de los índices de referencia, la práctica de mercado es descontar con la curva **€STR (OIS)** y proyectar el Euríbor con su propia curva (enfoque multicurva). Para simplificar, el ejemplo completo usa una única curva.

---

## 9. Índices de referencia

```python
euribor6m = ql.Euribor6M(curva_handle)        # Proyecta los tipos futuros con la curva
estr = ql.Estr(curva_handle)
sofr = ql.Sofr(curva_handle)

euribor6m.name()                               # 'Euribor6M Actual/360'
euribor6m.fixingDate(fecha_inicio_periodo)
euribor6m.addFixing(ql.Date(24, 11, 2026), 0.0228)     # Fijación histórica
euribor6m.fixing(ql.Date(24, 11, 2026))                 # 0.0228
ql.IndexManager.instance().clearHistories()             # Borrar fijaciones almacenadas
```

> Si un instrumento necesita una fijación pasada que no se ha cargado, QuantLib lanza `Missing Euribor6M Actual/360 fixing for ...`. Cargue el histórico con `addFixing` o `addFixings`.

---

## 10. Bonos

### 10.1 Bono a tipo fijo

```python
calendario_pagos = ql.MakeSchedule(ql.Date(15, 3, 2024), ql.Date(15, 3, 2031), ql.Period("1Y"),
                                   calendar=ql.TARGET(), convention=ql.Unadjusted, backwards=True)
bono = ql.FixedRateBond(
    2,                                          # Días de liquidación
    100.0,                                      # Nominal (precio en % del nominal)
    calendario_pagos,
    [0.03],                                     # Cupón (lista: puede variar por periodo)
    ql.ActualActual(ql.ActualActual.ISMA),      # Base de cálculo del cupón
)
bono.setPricingEngine(ql.DiscountingBondEngine(curva_handle))
```

### 10.2 Precio, cupón corrido y rentabilidad

```python
bono.NPV()                      # Valor actual en la fecha de evaluación
bono.cleanPrice()               # Precio ex-cupón (como cotiza en mercado)
bono.dirtyPrice()               # Precio con cupón corrido (el que se paga)
bono.accruedAmount()            # Cupón corrido
bono.settlementDate()

# Rentabilidad (TIR) a partir del precio
tir = bono.bondYield(ql.BondPrice(99.5, ql.BondPrice.Clean),
                     ql.ActualActual(ql.ActualActual.ISMA), ql.Compounded, ql.Annual)

# Precio a partir de una rentabilidad
bono.cleanPrice(0.03, ql.ActualActual(ql.ActualActual.ISMA), ql.Compounded, ql.Annual)
```

> En versiones recientes de QuantLib, `bondYield` recibe el precio como `ql.BondPrice(valor, tipo)`. Las versiones antiguas aceptaban directamente un número.

### 10.3 Flujos de caja

```python
for flujo in bono.cashflows():
    print(flujo.date().ISO(), round(flujo.amount(), 4))
```

### 10.4 Duración, convexidad y DV01

```python
tipo = ql.InterestRate(tir, ql.ActualActual(ql.ActualActual.ISMA), ql.Compounded, ql.Annual)

ql.BondFunctions.duration(bono, tipo, ql.Duration.Modified)     # Duración modificada
ql.BondFunctions.duration(bono, tipo, ql.Duration.Macaulay)     # Duración de Macaulay
ql.BondFunctions.convexity(bono, tipo)
ql.BondFunctions.basisPointValue(bono, tipo)    # Variación de precio por 1 pb (con signo negativo)
```

### 10.5 Otros tipos de bonos

| Clase | Instrumento |
|---|---|
| `ql.ZeroCouponBond` | Cupón cero (letras del Tesoro) |
| `ql.FloatingRateBond` | Bono a tipo variable (Euríbor + diferencial) |
| `ql.AmortizingFixedRateBond` | Bono amortizable |
| `ql.CallableFixedRateBond` | Bono con opción de amortización anticipada |
| `ql.CPIBond` | Bono ligado a la inflación |

```python
letra = ql.ZeroCouponBond(2, ql.TARGET(), 100.0, hoy + ql.Period("1Y"))
```

---

## 11. Swaps de tipos de interés

### 11.1 Crear un swap

```python
swap = ql.MakeVanillaSwap(
    ql.Period("5Y"), euribor6m, 0.025, ql.Period("0D"),
    nominal=1_000_000,
    swapType=ql.Swap.Payer,                         # Paga fijo, recibe variable
    fixedLegTenor=ql.Period("1Y"),
    fixedLegDayCount=ql.Thirty360(ql.Thirty360.BondBasis),
)
swap.setPricingEngine(ql.DiscountingSwapEngine(curva_handle))
```

| Tipo | Significado |
|---|---|
| `ql.Swap.Payer` | Paga tipo fijo y recibe variable (se beneficia si suben los tipos) |
| `ql.Swap.Receiver` | Recibe fijo y paga variable |

### 11.2 Resultados

```python
swap.NPV()                    # Valor actual
swap.fairRate()               # Tipo fijo que hace NPV = 0
swap.fairSpread()             # Diferencial sobre el variable que hace NPV = 0
swap.fixedLegNPV(); swap.floatingLegNPV()
swap.fixedLegBPS()            # Valor de 1 pb en la pata fija
for flujo in swap.leg(0):     # Pata fija (0) o variable (1)
    print(flujo.date(), flujo.amount())
```

> `MakeVanillaSwap` admite argumentos con nombre en minúscula (`nominal`, `swapType`, `fixedLegTenor`, `settlementDays`, `effectiveDate`…). Un nombre incorrecto produce `TypeError: unexpected keyword argument`.

---

## 12. Opciones sobre acciones

### 12.1 Componentes

```python
spot = ql.SimpleQuote(100.0)
curva_libre_riesgo = ql.YieldTermStructureHandle(ql.FlatForward(hoy, 0.025, ql.Actual365Fixed()))
dividendos = ql.YieldTermStructureHandle(ql.FlatForward(hoy, 0.015, ql.Actual365Fixed()))
volatilidad = ql.BlackVolTermStructureHandle(ql.BlackConstantVol(hoy, ql.TARGET(), 0.25, ql.Actual365Fixed()))

proceso = ql.BlackScholesMertonProcess(ql.QuoteHandle(spot), dividendos, curva_libre_riesgo, volatilidad)
```

### 12.2 Opción europea

```python
pago = ql.PlainVanillaPayoff(ql.Option.Call, 105.0)         # ql.Option.Put para venta
ejercicio = ql.EuropeanExercise(hoy + ql.Period("1Y"))
opcion = ql.VanillaOption(pago, ejercicio)
opcion.setPricingEngine(ql.AnalyticEuropeanEngine(proceso))

opcion.NPV()
opcion.delta(); opcion.gamma(); opcion.vega(); opcion.theta(); opcion.rho()
opcion.impliedVolatility(precio_mercado, proceso)
```

> `vega()` y `rho()` se expresan por unidad de volatilidad o tipo (100 %): divida entre 100 para obtener la sensibilidad por punto porcentual.

### 12.3 Opción americana y otros motores

```python
americana = ql.VanillaOption(pago, ql.AmericanExercise(hoy, hoy + ql.Period("1Y")))
americana.setPricingEngine(ql.BinomialVanillaEngine(proceso, "crr", 500))      # Árbol binomial
americana.setPricingEngine(ql.FdBlackScholesVanillaEngine(proceso, 200, 200))  # Diferencias finitas
```

| Motor | Método | Ejercicio |
|---|---|---|
| `AnalyticEuropeanEngine` | Fórmula de Black-Scholes | Europeo |
| `BinomialVanillaEngine` | Árboles (`"crr"`, `"jr"`, `"lr"`…) | Europeo y americano |
| `FdBlackScholesVanillaEngine` | Diferencias finitas | Europeo y americano |
| `MCEuropeanEngine` | Monte Carlo | Europeo |
| `AnalyticHestonEngine` | Modelo de Heston | Europeo |

### 12.4 Fórmula de Black directa

```python
ql.blackFormula(ql.Option.Call, 105.0, 100.0, 0.20, 0.98)     # strike, forward, desv. típica, FD
```

---

## 13. Análisis de riesgo y sensibilidades

### 13.1 Desplazamiento paralelo de la curva

```python
desplazamiento = ql.SimpleQuote(0.0)
curva_desplazada = ql.ZeroSpreadedTermStructure(curva_handle, ql.QuoteHandle(desplazamiento))
motor = ql.DiscountingBondEngine(ql.YieldTermStructureHandle(curva_desplazada))
bono.setPricingEngine(motor)

base = bono.NPV()
desplazamiento.setValue(0.0001)               # +1 pb
dv01 = bono.NPV() - base
desplazamiento.setValue(0.0)
```

### 13.2 Sensibilidad por cotizaciones de mercado

Como la curva se construye a partir de `SimpleQuote`, basta con modificar cada cotización para obtener la sensibilidad a cada plazo (*key rate duration*):

```python
for plazo, cotizacion in cotizaciones_swap.items():
    valor_inicial = cotizacion.value()
    cotizacion.setValue(valor_inicial + 0.0001)
    print(plazo, cartera_npv() - base)
    cotizacion.setValue(valor_inicial)
```

### 13.3 Griegas por diferencias finitas

```python
def griega_numerica(opcion, cotizacion, h=0.01):
    v0 = cotizacion.value()
    cotizacion.setValue(v0 + h); arriba = opcion.NPV()
    cotizacion.setValue(v0 - h); abajo = opcion.NPV()
    cotizacion.setValue(v0)
    return (arriba - abajo) / (2 * h)
```

---

## 14. Integración con pandas y otras herramientas

### 14.1 Flujos de caja en un `DataFrame`

```python
import pandas as pd

flujos = pd.DataFrame(
    [(cf.date().to_date(), cf.amount(), curva.discount(cf.date())) for cf in bono.cashflows()
     if cf.date() > hoy],
    columns=["fecha", "importe", "factor_descuento"],
)
flujos["valor_actual"] = flujos["importe"] * flujos["factor_descuento"]
```

### 14.2 Curva para gráficos

```python
fechas = [hoy + ql.Period(m, ql.Months) for m in range(1, 241)]
curva_df = pd.DataFrame({
    "fecha": [f.to_date() for f in fechas],
    "tipo_cero": [curva.zeroRate(f, ql.Actual365Fixed(), ql.Continuous).rate() for f in fechas],
})
curva_df.plot(x="fecha", y="tipo_cero")
```

### 14.3 Datos de mercado

QuantLib no proporciona cotizaciones. Fuentes habituales: proveedores profesionales (Bloomberg, Refinitiv/LSEG), el Banco de España y el BCE (curvas de deuda pública y €STR), EMMI (Euríbor) o archivos internos. Cárguelas con pandas y conviértalas en `SimpleQuote`.

---

## 15. Rendimiento y organización del código

- **Construya los objetos una sola vez** y use `SimpleQuote.setValue()` para los escenarios: QuantLib solo recalcula lo necesario.
- Los objetos de QuantLib pueden ser costosos de construir (curvas con muchos helpers, motores de diferencias finitas): reutilícelos.
- QuantLib **no es seguro entre hilos** por defecto (`Settings` es global). Para paralelizar, use procesos (`multiprocessing`, `concurrent.futures.ProcessPoolExecutor`).
- Encapsule cada parte en funciones o clases: datos de mercado, construcción de curvas, creación de instrumentos y análisis.

```
valoracion/
├── mercado.py           # Carga de cotizaciones → SimpleQuote
├── curvas.py            # Construcción de curvas (helpers, bootstrapping)
├── instrumentos.py      # Fábricas de bonos, swaps, opciones
├── riesgo.py            # Sensibilidades y escenarios
└── informes.py          # Exportación a Excel / pandas
```

---

## 16. Ejemplo práctico completo

**Valoración de una cartera de renta fija y derivados**: construye la curva del euro a partir de depósitos y swaps contra Euríbor 6M (*bootstrapping*), valora tres bonos a tipo fijo (precio, cupón corrido, TIR, duración modificada, convexidad y DV01), un swap de tipos de interés (NPV y tipo justo), una opción *put* europea y otra americana con sus griegas y volatilidad implícita, y analiza la sensibilidad de la cartera ante desplazamientos paralelos de la curva aprovechando el patrón observador.

Requisitos: `pip install QuantLib`

> Las cotizaciones son **ilustrativas**, no datos reales de mercado.

```python
"""
Valoración de una cartera de renta fija y derivados con QuantLib.

1. Construye la curva de tipos del euro (depósitos + swaps contra Euríbor 6M).
2. Valora bonos a tipo fijo: precio, TIR, duración, convexidad y DV01.
3. Valora un swap de tipo de interés y calcula su tipo fijo justo.
4. Valora una opción europea y una americana sobre acciones con sus griegas.
5. Analiza la sensibilidad de la cartera a un desplazamiento paralelo de la curva.
"""
import QuantLib as ql

# ------------------------------------------------------------------ Configuración
HOY = ql.Date(26, 11, 2026)
ql.Settings.instance().evaluationDate = HOY
CALENDARIO = ql.TARGET()
LIQUIDACION_DIAS = 2
NOMINAL = 1_000_000

DEPOSITOS = {"1M": 0.0215, "3M": 0.0222, "6M": 0.0230}              # Tipos de mercado
SWAPS = {"1Y": 0.0235, "2Y": 0.0240, "3Y": 0.0246, "5Y": 0.0257,
         "7Y": 0.0266, "10Y": 0.0278, "15Y": 0.0290, "20Y": 0.0294}

BONOS = [  # (nombre, cupón, vencimiento)
    ("Bono 3 años 2,50 %", 0.0250, ql.Date(30, 11, 2029)),
    ("Bono 7 años 3,00 %", 0.0300, ql.Date(31, 7, 2033)),
    ("Bono 15 años 3,40 %", 0.0340, ql.Date(30, 4, 2041)),
]


# ------------------------------------------------------------------ 1. Curva de tipos
def construir_curva(desplazamiento: ql.SimpleQuote):
    """Curva bootstrap con un diferencial paralelo ajustable (para sensibilidades)."""
    curva_proyeccion = ql.RelinkableYieldTermStructureHandle()
    euribor6m = ql.Euribor6M(curva_proyeccion)
    helpers = []
    for plazo, tipo in DEPOSITOS.items():
        cotizacion = ql.QuoteHandle(ql.CompositeQuote(
            ql.QuoteHandle(ql.SimpleQuote(tipo)), ql.QuoteHandle(desplazamiento), lambda a, b: a + b))
        helpers.append(ql.DepositRateHelper(cotizacion, ql.Period(plazo), LIQUIDACION_DIAS,
                                            CALENDARIO, ql.ModifiedFollowing, False, ql.Actual360()))
    for plazo, tipo in SWAPS.items():
        cotizacion = ql.QuoteHandle(ql.CompositeQuote(
            ql.QuoteHandle(ql.SimpleQuote(tipo)), ql.QuoteHandle(desplazamiento), lambda a, b: a + b))
        helpers.append(ql.SwapRateHelper(cotizacion, ql.Period(plazo), CALENDARIO, ql.Annual,
                                         ql.Unadjusted, ql.Thirty360(ql.Thirty360.BondBasis), euribor6m))
    curva = ql.PiecewiseLogCubicDiscount(LIQUIDACION_DIAS, CALENDARIO, helpers, ql.Actual365Fixed())
    curva.enableExtrapolation()
    curva_proyeccion.linkTo(curva)
    return curva, euribor6m


# ------------------------------------------------------------------ 2. Bonos
def crear_bono(cupon: float, vencimiento: ql.Date) -> ql.FixedRateBond:
    emision = vencimiento - ql.Period(20, ql.Years)
    calendario_pagos = ql.MakeSchedule(emision, vencimiento, ql.Period(ql.Annual), calendar=CALENDARIO,
                                       convention=ql.Unadjusted, backwards=True)
    return ql.FixedRateBond(LIQUIDACION_DIAS, 100.0, calendario_pagos, [cupon],
                            ql.ActualActual(ql.ActualActual.ISMA))


def analizar_bono(bono: ql.FixedRateBond):
    precio_limpio = bono.cleanPrice()
    tir = bono.bondYield(ql.BondPrice(precio_limpio, ql.BondPrice.Clean),
                         ql.ActualActual(ql.ActualActual.ISMA), ql.Compounded, ql.Annual)
    tipo = ql.InterestRate(tir, ql.ActualActual(ql.ActualActual.ISMA), ql.Compounded, ql.Annual)
    duracion = ql.BondFunctions.duration(bono, tipo, ql.Duration.Modified)
    convexidad = ql.BondFunctions.convexity(bono, tipo)
    dv01 = -ql.BondFunctions.basisPointValue(bono, tipo)     # QuantLib lo devuelve con signo negativo
    return precio_limpio, bono.accruedAmount(), tir, duracion, convexidad, dv01


# ------------------------------------------------------------------ 3. Swap
def crear_swap(euribor6m, tipo_fijo: float, plazo: str = "5Y") -> ql.VanillaSwap:
    return ql.MakeVanillaSwap(ql.Period(plazo), euribor6m, tipo_fijo, ql.Period(0, ql.Days),
                              nominal=NOMINAL, swapType=ql.Swap.Payer,
                              fixedLegTenor=ql.Period("1Y"),
                              fixedLegDayCount=ql.Thirty360(ql.Thirty360.BondBasis))


# ------------------------------------------------------------------ 4. Opciones
def valorar_opciones(curva_handle):
    spot = ql.SimpleQuote(100.0)
    vol = ql.BlackConstantVol(HOY, CALENDARIO, 0.25, ql.Actual365Fixed())
    dividendos = ql.FlatForward(HOY, 0.015, ql.Actual365Fixed())
    proceso = ql.BlackScholesMertonProcess(ql.QuoteHandle(spot), ql.YieldTermStructureHandle(dividendos),
                                           curva_handle, ql.BlackVolTermStructureHandle(vol))
    pago = ql.PlainVanillaPayoff(ql.Option.Put, 100.0)
    vencimiento = HOY + ql.Period(1, ql.Years)

    europea = ql.VanillaOption(pago, ql.EuropeanExercise(vencimiento))
    europea.setPricingEngine(ql.AnalyticEuropeanEngine(proceso))
    americana = ql.VanillaOption(pago, ql.AmericanExercise(HOY, vencimiento))
    americana.setPricingEngine(ql.BinomialVanillaEngine(proceso, "crr", 500))
    return europea, americana, proceso


# ------------------------------------------------------------------ Informe
def main() -> None:
    desplazamiento = ql.SimpleQuote(0.0)
    curva, euribor6m = construir_curva(desplazamiento)
    curva_handle = ql.YieldTermStructureHandle(curva)
    motor_bonos = ql.DiscountingBondEngine(curva_handle)

    print(f"Fecha de valoración: {HOY.ISO()}\n")
    print("CURVA CERO (continua, Act/365)")
    for plazo in ("6M", "1Y", "2Y", "5Y", "10Y", "20Y"):
        fecha = CALENDARIO.advance(HOY, ql.Period(plazo))
        cero = curva.zeroRate(fecha, ql.Actual365Fixed(), ql.Continuous).rate()
        print(f"  {plazo:>4}: {cero:6.3%}   FD {curva.discount(fecha):.5f}")

    print(f"\n{'BONO':<22}{'Precio':>9}{'Cupón c.':>9}{'TIR':>8}{'Dur.mod':>9}{'Convex.':>9}{'DV01':>8}")
    bonos = []
    for nombre, cupon, vencimiento in BONOS:
        bono = crear_bono(cupon, vencimiento)
        bono.setPricingEngine(motor_bonos)
        bonos.append(bono)
        p, cc, tir, dur, conv, dv01 = analizar_bono(bono)
        print(f"{nombre:<22}{p:>9.3f}{cc:>9.3f}{tir:>8.3%}{dur:>9.2f}{conv:>9.1f}{dv01:>8.4f}")

    swap = crear_swap(euribor6m, 0.0250)
    swap.setPricingEngine(ql.DiscountingSwapEngine(curva_handle))
    print(f"\nSWAP 5 AÑOS pagador fijo 2,50 % · nominal {NOMINAL:,} €")
    print(f"  Valor (NPV): {swap.NPV():>12,.2f} €")
    print(f"  Tipo fijo justo: {swap.fairRate():.4%}")
    print(f"  Valor de la pata fija / variable: {swap.fixedLegNPV():,.0f} € / {swap.floatingLegNPV():,.0f} €")

    europea, americana, proceso = valorar_opciones(curva_handle)
    print("\nPUT SOBRE ACCIÓN (S = K = 100, σ = 25 %, 1 año)")
    print(f"  Europea:  {europea.NPV():.4f}  Δ {europea.delta():+.4f}  Γ {europea.gamma():.4f}  "
          f"Vega {europea.vega() / 100:.4f}")
    print(f"  Americana: {americana.NPV():.4f}  (prima de ejercicio anticipado "
          f"{americana.NPV() - europea.NPV():.4f})")
    vol_implicita = europea.impliedVolatility(europea.NPV() + 0.50, proceso)
    print(f"  Volatilidad implícita si el mercado cotiza 0,50 más: {vol_implicita:.2%}")

    # ------------------------------------------------------------------ 5. Sensibilidad
    def valor_cartera():
        return sum(b.dirtyPrice() / 100 * NOMINAL for b in bonos) + swap.NPV()

    base = valor_cartera()
    print(f"\nSENSIBILIDAD DE LA CARTERA (3 bonos × {NOMINAL:,} € + swap) · valor {base:,.0f} €")
    for pb in (-100, -50, -10, 10, 50, 100):
        desplazamiento.setValue(pb / 10_000)        # Toda la curva se recalcula automáticamente
        print(f"  {pb:+5d} pb: {valor_cartera() - base:>+13,.0f} €")
    desplazamiento.setValue(0.0)


if __name__ == "__main__":
    main()
```

Resultado:

```
Fecha de valoración: 2026-11-26

CURVA CERO (continua, Act/365)
    6M: 2.317%   FD 0.98883
    1Y: 2.323%   FD 0.97729
    2Y: 2.368%   FD 0.95386
    5Y: 2.544%   FD 0.88073
   10Y: 2.761%   FD 0.75877
   20Y: 2.931%   FD 0.55642

BONO                     Precio Cupón c.     TIR  Dur.mod  Convex.    DV01
Bono 3 años 2,50 %      100.114    0.000  2.460%     2.86     11.1  0.0286
Bono 7 años 3,00 %      102.128    1.003  2.645%     5.94     43.1  0.0612
Bono 15 años 3,40 %     105.962    1.993  2.887%    11.19    155.5  0.1207

SWAP 5 AÑOS pagador fijo 2,50 % · nominal 1,000,000 €
  Valor (NPV):     3,376.29 €
  Tipo fijo justo: 2.5727%
  Valor de la pata fija / variable: -116,029 € / 119,405 €

PUT SOBRE ACCIÓN (S = K = 100, σ = 25 %, 1 año)
  Europea:  9.3744  Δ -0.4311  Γ 0.0155  Vega 0.3882
  Americana: 9.4038  (prima de ejercicio anticipado 0.0294)
  Volatilidad implícita si el mercado cotiza 0,50 más: 26.29%

SENSIBILIDAD DE LA CARTERA (3 bonos × 1,000,000 € + swap) · valor 3,115,383 €
   -100 pb:      +176,055 €
    -50 pb:       +85,344 €
    -10 pb:       +16,658 €
    +10 pb:       -16,458 €
    +50 pb:       -80,351 €
   +100 pb:      -156,056 €
```

> La asimetría entre subidas y bajadas (+176 055 € frente a −156 056 €) refleja la **convexidad** de los bonos. El swap pagador compensa parcialmente la pérdida de los bonos cuando suben los tipos. El cupón corrido del primer bono es cero porque la fecha de liquidación coincide con una fecha de pago de cupón.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| Resultados distintos cada día / valores inesperados | No se fijó la fecha de evaluación (se usa la del sistema). | `ql.Settings.instance().evaluationDate = fecha` al inicio. |
| `TypeError: Wrong number or type of arguments for overloaded function` | Argumentos en orden o tipo incorrecto (p. ej. `int` donde se espera `Period`, número en lugar de `BondPrice`). | Revise la firma que muestra el mensaje («Possible C/C++ prototypes»). |
| `RuntimeError: Missing Euribor6M Actual/360 fixing for ...` | El instrumento necesita una fijación pasada no cargada. | Añada el histórico con `indice.addFixing(fecha, valor)`. |
| `RuntimeError: time (...) is past max curve time` | Consulta más allá del último nodo de la curva. | Añada plazos más largos o `curva.enableExtrapolation()`. |
| `RuntimeError: 1st iteration: failed at 1st alive instrument` | Fallo del *bootstrapping*: cotizaciones incoherentes o fecha de evaluación posterior a algún instrumento. | Revise las cotizaciones, plazos duplicados y la fecha de evaluación. |
| `RuntimeError: no pricing engine set` | No se asignó un motor al instrumento. | `instrumento.setPricingEngine(motor)`. |
| `RuntimeError: null term structure set to this instance` | *Handle* vacío (p. ej. índice creado sin curva). | Pase la curva al crear el índice o enlácela con `linkTo()`. |
| `TypeError: ... got an unexpected keyword argument` | Nombre de argumento incorrecto en funciones `Make...`. | Use los nombres en minúscula (`nominal`, `swapType`…). |
| La TIR no coincide con la de la terminal de mercado | Convenciones distintas (base, frecuencia, liquidación). | Use exactamente las convenciones del folleto del bono. |
| `vega` 100 veces mayor de lo esperado | Se expresa por unidad de volatilidad. | Divida entre 100 para obtenerla por punto porcentual. |
| El NPV no cambia al modificar un dato | Se modificó un valor copiado, no la `SimpleQuote` enlazada. | Mantenga referencias a las cotizaciones y use `setValue()`. |
| Resultados erróneos en paralelo con hilos | `Settings` y la caché de índices son globales. | Use procesos independientes. |

---

## 18. Buenas prácticas

- **Fije siempre la fecha de evaluación** al inicio de cada cálculo y regístrela en los informes.
- Use las **convenciones exactas** del instrumento (calendario, ajuste, base de cálculo, días de liquidación): son la principal fuente de discrepancias.
- Construya las curvas con `SimpleQuote` y *handles* para poder analizar escenarios sin reconstruir objetos.
- Separe datos de mercado, curvas, instrumentos y análisis en módulos distintos.
- **Valide** los resultados con casos de referencia: la curva debe reproducir las cotizaciones de entrada (`swap.fairRate()` ≈ cotización del swap).
- Mantenga un histórico de fijaciones de los índices y cárguelo antes de valorar instrumentos vivos.
- Documente cada supuesto (interpolación, extrapolación, curva de descuento) en el código o en el informe.
- Fije la versión de QuantLib en `requirements.txt`: las firmas pueden cambiar entre versiones.
- Para paralelizar escenarios, use procesos en lugar de hilos.
- Consulte la documentación de C++ y los ejemplos oficiales de Python cuando dude sobre una clase.

---

## 19. Referencia rápida

```python
import QuantLib as ql
ql.Settings.instance().evaluationDate = ql.Date(26, 11, 2026)

# Fechas
ql.Date(d, m, a) · ql.Period("6M") · fecha + ql.Period(1, ql.Years) · fecha.ISO() · fecha.to_date()
ql.TARGET() · cal.adjust(f) · cal.advance(f, n, ql.Days) · cal.isBusinessDay(f)
ql.Actual360() · ql.Actual365Fixed() · ql.Thirty360(ql.Thirty360.BondBasis) · ql.ActualActual(ql.ActualActual.ISMA)
ql.MakeSchedule(inicio, fin, ql.Period("1Y"), calendar=..., convention=..., backwards=True)

# Mercado y curvas
q = ql.SimpleQuote(0.03); q.setValue(0.031); ql.QuoteHandle(q)
ql.FlatForward(hoy, 0.03, dc) · ql.ZeroCurve(fechas, tipos, dc) · ql.DiscountCurve(fechas, fds, dc)
ql.DepositRateHelper(...) · ql.SwapRateHelper(...) · ql.PiecewiseLogCubicDiscount(2, cal, helpers, dc)
curva.discount(f) · curva.zeroRate(f, dc, ql.Continuous).rate() · curva.forwardRate(f1, f2, dc, ql.Simple).rate()
ql.YieldTermStructureHandle(curva) · ql.RelinkableYieldTermStructureHandle() · ql.Euribor6M(handle)

# Instrumentos
ql.FixedRateBond(2, 100, sched, [cupon], dc) + ql.DiscountingBondEngine(h)
bono.cleanPrice() · dirtyPrice() · accruedAmount() · bondYield(ql.BondPrice(p, ql.BondPrice.Clean), dc, comp, freq)
ql.BondFunctions.duration(bono, tipo, ql.Duration.Modified) · convexity · basisPointValue
ql.MakeVanillaSwap(ql.Period("5Y"), indice, tipo, nominal=..., swapType=ql.Swap.Payer) + ql.DiscountingSwapEngine(h)
swap.NPV() · swap.fairRate() · swap.fairSpread()
ql.VanillaOption(ql.PlainVanillaPayoff(tipo, K), ql.EuropeanExercise(f)) + ql.AnalyticEuropeanEngine(proceso)
opcion.NPV() · delta() · gamma() · vega() · theta() · impliedVolatility(precio, proceso)
```

---

## 20. Ejercicios propuestos

1. Construya un calendario de pagos semestral para un préstamo de 10 años con el calendario TARGET y la convención *Modified Following*, y compare las fechas con y sin ajuste.
2. Valore una letra del Tesoro a 12 meses (cupón cero) a partir de su rentabilidad y obtenga el precio con base Act/360.
3. Construya la curva €STR a partir de cotizaciones OIS (`OISRateHelper`) y compare sus tipos cero con la curva del ejemplo.
4. Calcule la sensibilidad de la cartera del ejemplo a cada plazo de la curva (*key rate duration*) desplazando las cotizaciones una a una.
5. Valore una opción *call* europea con el modelo de Heston y compare su precio con el de Black-Scholes para distintos precios de ejercicio, representando la sonrisa de volatilidad implícita.

---

## 21. Referencias

- Sitio oficial: https://www.quantlib.org/
- Documentación de referencia (C++): https://www.quantlib.org/reference/
- Documentación de la interfaz Python: https://quantlib-python-docs.readthedocs.io/
- Repositorio de la interfaz Python (SWIG) y ejemplos: https://github.com/lballabio/QuantLib-SWIG
- Luigi Ballabio, *Implementing QuantLib* — https://www.implementingquantlib.com/
- Goutham Balaraman y Luigi Ballabio, *QuantLib Python Cookbook* (Leanpub).
