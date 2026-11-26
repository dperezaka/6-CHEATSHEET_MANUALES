# Manual de uso: Statsmodels

> **Categoría:** Ciencia de datos · **Utilidad:** Análisis estadístico, modelos econométricos y series temporales
> **Tipo:** Librería de terceros (`pip install statsmodels`) · **Versión de referencia:** 0.14 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Dos interfaces: fórmulas y matrices](#3-dos-interfaces-fórmulas-y-matrices)
4. [Regresión lineal (OLS)](#4-regresión-lineal-ols)
5. [Interpretar el resumen](#5-interpretar-el-resumen)
6. [Fórmulas: variables categóricas, interacciones y transformaciones](#6-fórmulas-variables-categóricas-interacciones-y-transformaciones)
7. [Diagnóstico del modelo](#7-diagnóstico-del-modelo)
8. [Errores estándar robustos](#8-errores-estándar-robustos)
9. [Predicción e intervalos](#9-predicción-e-intervalos)
10. [ANOVA y comparación de modelos](#10-anova-y-comparación-de-modelos)
11. [Regresión logística y otros modelos lineales generalizados](#11-regresión-logística-y-otros-modelos-lineales-generalizados)
12. [Otros modelos: WLS, cuantílica, mixtos](#12-otros-modelos-wls-cuantílica-mixtos)
13. [Contrastes estadísticos y potencia](#13-contrastes-estadísticos-y-potencia)
14. [Series temporales: exploración](#14-series-temporales-exploración)
15. [Series temporales: modelos ARIMA/SARIMAX](#15-series-temporales-modelos-arimasarimax)
16. [Suavizado exponencial (ETS y Holt-Winters)](#16-suavizado-exponencial-ets-y-holt-winters)
17. [Evaluación de previsiones](#17-evaluación-de-previsiones)
18. [Gráficos de statsmodels](#18-gráficos-de-statsmodels)
19. [Ejemplo práctico completo](#19-ejemplo-práctico-completo)
20. [Errores frecuentes](#20-errores-frecuentes)
21. [Buenas prácticas](#21-buenas-prácticas)
22. [Referencia rápida](#22-referencia-rápida)
23. [Ejercicios propuestos](#23-ejercicios-propuestos)
24. [Referencias](#24-referencias)

---

## 1. Descripción

**Statsmodels** es la librería de Python para **estadística inferencial y econometría**. Estima modelos (regresión lineal, logística, de Poisson, mixtos, cuantílica…), proporciona informes completos con coeficientes, errores estándar, p-valores e intervalos de confianza, ofrece contrastes de hipótesis y diagnóstico, y modela **series temporales** (ARIMA, SARIMAX, suavizado exponencial, descomposición estacional).

**Ventajas:**
- Informes estadísticos completos, similares a R o Stata.
- Fórmulas estilo R (`y ~ x1 + C(region) + np.log(precio)`) con DataFrames de pandas.
- Amplio catálogo de diagnósticos y errores estándar robustos.
- Modelos de series temporales con intervalos de previsión.

**Limitaciones:**
- Orientada a **explicar e inferir**, no a maximizar la precisión predictiva (para eso, scikit-learn).
- Algunos modelos son lentos con millones de filas.
- No incluye selección automática de modelos ARIMA (use `pmdarima` o `statsforecast`).

| Si necesita… | Opción |
|---|---|
| Entender el efecto de variables, con p-valores e intervalos | **Statsmodels** |
| Modelos predictivos y validación cruzada | scikit-learn |
| Contrastes estadísticos rápidos y distribuciones | SciPy (`scipy.stats`) |
| Previsión automática de muchas series | statsforecast, Prophet |
| Estadística bayesiana | PyMC, Bambi |

---

## 2. Instalación e importación

```bash
pip install statsmodels pandas matplotlib
```

```python
import numpy as np
import pandas as pd
import statsmodels.api as sm                    # Interfaz de matrices, herramientas y contrastes
import statsmodels.formula.api as smf           # Interfaz de fórmulas
```

---

## 3. Dos interfaces: fórmulas y matrices

```python
# Fórmulas (recomendada con DataFrames): incluye la constante automáticamente
modelo = smf.ols("ventas ~ precio + publicidad + C(region)", data=df).fit()

# Matrices: debe añadir la constante usted mismo
X = sm.add_constant(df[["precio", "publicidad"]])
modelo = sm.OLS(df["ventas"], X).fit()
```

| Función con fórmula (`smf`) | Modelo |
|---|---|
| `ols`, `wls`, `gls` | Lineal por mínimos cuadrados (ordinarios, ponderados, generalizados). |
| `logit`, `probit` | Respuesta binaria. |
| `mnlogit` | Respuesta multinomial. |
| `poisson`, `negativebinomial` | Recuentos. |
| `glm` | Modelos lineales generalizados (cualquier familia y enlace). |
| `quantreg` | Regresión cuantílica. |
| `mixedlm` | Modelos de efectos mixtos. |

---

## 4. Regresión lineal (OLS)

```python
modelo = smf.ols("ventas ~ precio + publicidad + promocion", data=df).fit()
print(modelo.summary())

modelo.params            # Coeficientes
modelo.bse               # Errores estándar
modelo.pvalues
modelo.conf_int(0.05)    # Intervalos de confianza al 95 %
modelo.rsquared, modelo.rsquared_adj, modelo.aic, modelo.bic, modelo.nobs
modelo.fittedvalues, modelo.resid
modelo.f_pvalue          # Significación global

tabla = pd.DataFrame({"coef": modelo.params, "ee": modelo.bse, "p": modelo.pvalues}).join(
    modelo.conf_int().rename(columns={0: "ic_inf", 1: "ic_sup"}))
```

---

## 5. Interpretar el resumen

| Elemento | Significado |
|---|---|
| `coef` | Variación esperada de la respuesta por cada unidad de la variable, manteniendo las demás constantes. |
| `std err` | Incertidumbre del coeficiente. |
| `t`, `P>\|t\|` | Contraste de que el coeficiente es 0. |
| `[0.025 0.975]` | Intervalo de confianza del 95 %. |
| `R-squared` / `Adj. R-squared` | Proporción de variabilidad explicada (ajustada por número de variables). |
| `F-statistic`, `Prob (F-statistic)` | Significación conjunta del modelo. |
| `AIC`, `BIC` | Comparación de modelos (menor es mejor). |
| `Durbin-Watson` | Autocorrelación de los residuos (≈ 2 sin autocorrelación). |
| `Omnibus`, `Jarque-Bera` | Normalidad de los residuos. |
| `Cond. No.` | Valores altos (> 1000) sugieren multicolinealidad o escalas muy dispares. |

> Un coeficiente significativo no implica causalidad. La interpretación causal exige un diseño adecuado (experimento, variables de control, identificación).

---

## 6. Fórmulas: variables categóricas, interacciones y transformaciones

```python
smf.ols("ventas ~ C(region)", df)                                  # Variables ficticias (la primera, referencia)
smf.ols("ventas ~ C(region, Treatment(reference='Centro'))", df)   # Elegir la categoría de referencia
smf.ols("ventas ~ precio * promocion", df)                         # precio + promocion + precio:promocion
smf.ols("ventas ~ precio:promocion", df)                           # Solo la interacción
smf.ols("np.log(ventas) ~ np.log(precio)", df)                     # Log-log: coeficiente = elasticidad
smf.ols("ventas ~ precio + I(precio ** 2)", df)                    # I(): aritmética literal
smf.ols("ventas ~ scale(publicidad)", df)                          # Estandarizar
smf.ols("ventas ~ bs(temperatura, df=4)", df)                      # Spline (no linealidad)
smf.ols("ventas ~ precio - 1", df)                                 # Sin constante
smf.ols("ventas ~ Q('gasto en TV')", df)                           # Nombres con espacios
```

| Sintaxis | Significado |
|---|---|
| `a + b` | Efectos principales. |
| `a:b` | Interacción. |
| `a * b` | `a + b + a:b`. |
| `C(x)` | Tratar como categórica. |
| `I(expr)` | Expresión aritmética literal. |
| `- 1` | Eliminar la constante. |

---

## 7. Diagnóstico del modelo

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor
from statsmodels.stats.diagnostic import het_breuschpagan, linear_reset
from statsmodels.stats.stattools import durbin_watson, jarque_bera

X = modelo.model.exog
vif = pd.Series([variance_inflation_factor(X, i) for i in range(1, X.shape[1])],
                index=modelo.model.exog_names[1:])           # > 5–10: multicolinealidad

lm, p_bp, f, p_f = het_breuschpagan(modelo.resid, X)       # p < 0,05 → heterocedasticidad
jb, p_jb, asimetria, curtosis = jarque_bera(modelo.resid)  # p < 0,05 → residuos no normales
durbin_watson(modelo.resid)                                 # ≈ 2: sin autocorrelación
linear_reset(modelo, power=2).pvalue                        # p < 0,05 → posible forma funcional incorrecta

influencia = modelo.get_influence()
cook = influencia.cooks_distance[0]                         # > 4/n: observaciones influyentes
residuos_est = influencia.resid_studentized_external       # |r| > 3: posibles atípicos
```

| Supuesto | Diagnóstico | Remedio |
|---|---|---|
| Linealidad | Residuos frente a ajustados, RESET | Transformaciones, términos cuadráticos, *splines*. |
| Homocedasticidad | Breusch-Pagan, gráfico de residuos | Errores robustos (HC3), WLS, transformar la respuesta. |
| Independencia | Durbin-Watson, Ljung-Box | Errores HAC, modelos de series temporales. |
| Normalidad de residuos | Jarque-Bera, gráfico Q-Q | Muestras grandes: poco relevante; transformar. |
| Sin multicolinealidad | VIF, número de condición | Eliminar o combinar variables, regularización. |
| Sin observaciones influyentes | Distancia de Cook | Revisar datos; modelos robustos (`RLM`). |

---

## 8. Errores estándar robustos

```python
smf.ols(formula, df).fit(cov_type="HC3")                                   # Heterocedasticidad
smf.ols(formula, df).fit(cov_type="HAC", cov_kwds={"maxlags": 4})          # + autocorrelación (series)
smf.ols(formula, df).fit(cov_type="cluster", cov_kwds={"groups": df["tienda"]})   # Datos agrupados
sm.RLM(y, X, M=sm.robust.norms.HuberT()).fit()                             # Regresión robusta a atípicos
```

Los coeficientes no cambian; cambian los errores estándar, los p-valores y los intervalos.

---

## 9. Predicción e intervalos

```python
nuevos = pd.DataFrame({"precio": [9.5, 10.5], "publicidad": [2000, 3000], "promocion": [1, 0]})
modelo.predict(nuevos)                                       # Valores esperados

pred = modelo.get_prediction(nuevos).summary_frame(alpha=0.05)
# mean, mean_ci_lower/upper: intervalo de la MEDIA esperada
# obs_ci_lower/upper:        intervalo de PREDICCIÓN para una nueva observación (más amplio)
```

> En modelos con `np.log(y)`, la predicción está en escala logarítmica: aplique `np.exp` y, para la media, una corrección (p. ej., multiplicar por `exp(σ²/2)`).

---

## 10. ANOVA y comparación de modelos

```python
from statsmodels.stats.anova import anova_lm

sm.stats.anova_lm(modelo, typ=2)                       # Contribución de cada término (tipo II)
reducido = smf.ols("ventas ~ precio", df).fit()
completo = smf.ols("ventas ~ precio + C(region) + promocion", df).fit()
anova_lm(reducido, completo)                           # ¿Mejora significativamente el modelo completo?
completo.compare_lr_test(reducido)                     # Razón de verosimilitudes
completo.f_test("promocion = 0, C(region)[T.Norte] = 0")
completo.t_test("precio = -2")                         # Contraste sobre un valor concreto

from statsmodels.stats.multicomp import pairwise_tukeyhsd
pairwise_tukeyhsd(df["ventas"], df["region"]).summary()   # Comparaciones múltiples de medias
```

---

## 11. Regresión logística y otros modelos lineales generalizados

```python
logit = smf.logit("compra ~ edad + C(canal) + descuento", data=df).fit(disp=False)
print(logit.summary())

odds = pd.DataFrame({"OR": np.exp(logit.params), "IC_inf": np.exp(logit.conf_int()[0]),
                     "IC_sup": np.exp(logit.conf_int()[1]), "p": logit.pvalues})    # Razones de probabilidades
logit.get_margeff(at="overall").summary()     # Efecto marginal medio en puntos de probabilidad
logit.predict(nuevos)                          # Probabilidades
logit.prsquared                                # Pseudo-R² de McFadden

poisson = smf.poisson("reclamaciones ~ antiguedad + C(producto)", df).fit(disp=False)
np.exp(poisson.params)                         # Ratios de tasas (IRR)
smf.poisson("eventos ~ x", df, offset=np.log(df["exposicion"])).fit()   # Tasas por exposición

glm = smf.glm("importe ~ edad + C(segmento)", df, family=sm.families.Gamma(sm.families.links.Log())).fit()
smf.glm("exito ~ x", df, family=sm.families.Binomial()).fit()
smf.negativebinomial("visitas ~ x", df).fit(disp=False)                 # Sobredispersión
```

| Respuesta | Familia / modelo |
|---|---|
| Continua, simétrica | `ols` / `Gaussian` |
| Binaria (sí/no) | `logit` / `Binomial` |
| Recuentos | `poisson`; `negativebinomial` si varianza > media |
| Positiva y asimétrica (importes, tiempos) | `Gamma` con enlace logarítmico |
| Proporciones | `Binomial` con pesos o `Beta` (`BetaModel`) |

---

## 12. Otros modelos: WLS, cuantílica, mixtos

```python
smf.wls("y ~ x", df, weights=1 / df["varianza"]).fit()                 # Mínimos cuadrados ponderados
smf.quantreg("tiempo_entrega ~ distancia + C(transportista)", df).fit(q=0.9)   # Percentil 90 (no la media)
smf.mixedlm("ventas ~ precio + promocion", df, groups=df["tienda"], re_formula="~precio").fit()   # Efectos aleatorios
```

---

## 13. Contrastes estadísticos y potencia

```python
from statsmodels.stats.proportion import proportions_ztest, proportion_confint, confint_proportions_2indep
from statsmodels.stats.power import TTestIndPower, NormalIndPower
from statsmodels.stats.multitest import multipletests
from statsmodels.stats.weightstats import ttest_ind, DescrStatsW

z, p = proportions_ztest(count=[310, 352], nobs=[5000, 5000])                  # Dos proporciones
proportion_confint(310, 5000, method="wilson")                                  # IC de una proporción
confint_proportions_2indep(352, 5000, 310, 5000, compare="diff")               # IC de la diferencia
t, p, gl = ttest_ind(grupo_a, grupo_b, usevar="unequal")
DescrStatsW(datos, weights=pesos).tconfint_mean()                              # Media ponderada con IC

rechazar, p_ajustados, *_ = multipletests(p_valores, alpha=0.05, method="fdr_bh")   # o "holm", "bonferroni"

# Tamaño muestral necesario
from statsmodels.stats.proportion import proportion_effectsize
efecto = proportion_effectsize(0.07, 0.062)                                     # h de Cohen
NormalIndPower().solve_power(effect_size=efecto, alpha=0.05, power=0.8, ratio=1)   # n por grupo
TTestIndPower().solve_power(effect_size=0.3, alpha=0.05, power=0.8)
```

---

## 14. Series temporales: exploración

```python
from statsmodels.tsa.seasonal import STL, seasonal_decompose
from statsmodels.tsa.stattools import adfuller, kpss, acf, pacf
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf

serie = df.set_index("fecha")["ventas"].asfreq("MS")        # Frecuencia explícita (mensual, inicio de mes)

stl = STL(serie, period=12, robust=True).fit()
stl.trend, stl.seasonal, stl.resid
fuerza_estacional = 1 - stl.resid.var() / (stl.seasonal + stl.resid).var()

adf = adfuller(serie, autolag="AIC", result_object=True)    # H0: raíz unitaria (no estacionaria)
adf.statistic, adf.pvalue                                   # p < 0,05 → estacionaria
kp = kpss(serie, nlags="auto", result_object=True)          # H0: estacionaria (contraste complementario)
sm.stats.acorr_ljungbox(residuos, lags=[12], return_df=True)    # Autocorrelación de residuos

plot_acf(serie.diff().dropna(), lags=24) ; plot_pacf(serie.diff().dropna(), lags=24)
```

> Versiones recientes de statsmodels avisan de que `adfuller` y `kpss` devolverán un objeto de resultados: use `result_object=True` y acceda a `.statistic` y `.pvalue`.

---

## 15. Series temporales: modelos ARIMA/SARIMAX

```python
from statsmodels.tsa.statespace.sarimax import SARIMAX
from statsmodels.tsa.arima.model import ARIMA

modelo = SARIMAX(serie, order=(1, 1, 1), seasonal_order=(0, 1, 1, 12),
                 exog=df[["promocion"]], enforce_stationarity=False, enforce_invertibility=False)
ajuste = modelo.fit(disp=False)
print(ajuste.summary())
ajuste.plot_diagnostics(figsize=(10, 7))

prevision = ajuste.get_forecast(steps=12, exog=futuro[["promocion"]])
tabla = prevision.summary_frame(alpha=0.05)          # mean, mean_ci_lower, mean_ci_upper
ajuste.aic ; ajuste.resid

ARIMA(serie, order=(2, 1, 0)).fit()                   # Sin estacionalidad ni regresores
```

| Parámetro | Significado |
|---|---|
| `order=(p, d, q)` | Autorregresivo, diferencias, media móvil. |
| `seasonal_order=(P, D, Q, s)` | Igual para la parte estacional; `s` = periodo (12 mensual, 7 diario semanal). |
| `exog` | Variables externas (promociones, festivos, precios); deben conocerse también en el futuro. |
| `trend` | `"c"` constante, `"t"` tendencia lineal. |

Selección de órdenes: compare el AIC de varias combinaciones y valide con datos reservados (sección 17).

---

## 16. Suavizado exponencial (ETS y Holt-Winters)

```python
from statsmodels.tsa.holtwinters import ExponentialSmoothing
from statsmodels.tsa.exponential_smoothing.ets import ETSModel

hw = ExponentialSmoothing(serie, trend="add", damped_trend=True, seasonal="mul", seasonal_periods=12).fit()
hw.forecast(12)

ets = ETSModel(serie, error="add", trend="add", seasonal="add", seasonal_periods=12).fit(disp=False)
ets.get_prediction(start=len(serie), end=len(serie) + 11).summary_frame(alpha=0.05)   # Con intervalos
```

ETS (*Error, Trend, Seasonal*) es la versión con modelo estadístico completo: permite intervalos de previsión y comparación por AIC.

---

## 17. Evaluación de previsiones

```python
entrenamiento, prueba = serie[:-12], serie[-12:]                 # Reservar el último año
ajuste = SARIMAX(entrenamiento, order=(1, 1, 1), seasonal_order=(0, 1, 1, 12)).fit(disp=False)
prev = ajuste.forecast(12)

mae = (prueba - prev).abs().mean()
mape = ((prueba - prev).abs() / prueba).mean()
rmse = np.sqrt(((prueba - prev) ** 2).mean())
ingenuo = serie.shift(12)[-12:]                                  # Referencia: mismo mes del año anterior
mase = mae / (entrenamiento.diff(12).abs().mean())               # < 1: mejor que el ingenuo estacional
```

Compare siempre con un modelo **ingenuo** (último valor o mismo periodo del año anterior): si su modelo no lo mejora, no aporta valor.

---

## 18. Gráficos de statsmodels

```python
import statsmodels.graphics.api as smg
sm.qqplot(modelo.resid, line="45", fit=True)                 # Normalidad de residuos
sm.graphics.plot_partregress_grid(modelo)                    # Regresión parcial de cada variable
sm.graphics.plot_regress_exog(modelo, "precio")             # Diagnóstico de una variable
sm.graphics.influence_plot(modelo, criterion="cooks")
stl.plot() ; ajuste.plot_diagnostics()
```

---

## 19. Ejemplo práctico completo

**Estudio analítico para la dirección comercial de una cadena de tiendas**, en tres partes:

1. **¿Qué impulsa las ventas?** Modelo log-log de ventas semanales por tienda: **elasticidad precio**, efecto de promociones y publicidad, diferencias por tipo de tienda y estacionalidad; diagnóstico completo (VIF, Breusch-Pagan, Jarque-Bera, Cook), errores agrupados por tienda, ANOVA y **simulación de escenarios de precio** con intervalos.
2. **Campaña de cupones**: regresión logística de la redención con razones de probabilidades y efectos marginales; prueba A/B de dos diseños de cupón con IC de la diferencia y **cálculo del tamaño muestral** necesario; corrección por comparaciones múltiples entre segmentos.
3. **Previsión de la demanda mensual**: descomposición STL, contraste ADF, comparación SARIMAX (con promociones como regresor) frente a ETS y frente al modelo ingenuo en el último año, y previsión a 12 meses con intervalos; guarda un gráfico y un Excel.

```python
"""
estudio_comercial.py — Inferencia, regresión logística y previsión con statsmodels.
Requiere: pip install statsmodels pandas matplotlib openpyxl
Uso: python estudio_comercial.py
"""
from __future__ import annotations

import warnings
from pathlib import Path

import matplotlib

matplotlib.use("Agg")
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import statsmodels.api as sm
import statsmodels.formula.api as smf
from statsmodels.stats.diagnostic import het_breuschpagan
from statsmodels.stats.multitest import multipletests
from statsmodels.stats.outliers_influence import variance_inflation_factor
from statsmodels.stats.power import NormalIndPower
from statsmodels.stats.proportion import confint_proportions_2indep, proportion_effectsize, proportions_ztest
from statsmodels.stats.stattools import jarque_bera
from statsmodels.tsa.exponential_smoothing.ets import ETSModel
from statsmodels.tsa.seasonal import STL
from statsmodels.tsa.statespace.sarimax import SARIMAX
from statsmodels.tsa.stattools import adfuller

RNG = np.random.default_rng(2026)
SALIDA = Path("estudio_comercial")
SALIDA.mkdir(exist_ok=True)
pd.set_option("display.width", 140)
pd.set_option("display.max_columns", 12)


def titulo(texto: str) -> None:
    print(f"\n{texto}\n{'═' * len(texto)}")


def num(v: float, d: int = 2) -> str:
    return f"{v:,.{d}f}".replace(",", "X").replace(".", ",").replace("X", ".")


# ============================================================ 1. ¿Qué impulsa las ventas?
def datos_ventas() -> pd.DataFrame:
    tiendas = pd.DataFrame({"tienda": [f"T{i:02d}" for i in range(1, 41)],
                            "tipo": RNG.choice(["Centro comercial", "Calle", "Periferia"], 40, p=[.35, .4, .25])})
    tiendas["efecto_tienda"] = RNG.normal(0, 0.15, 40)
    semanas = pd.date_range("2024-01-01", periods=104, freq="W-MON")
    df = tiendas.merge(pd.DataFrame({"semana": semanas}), how="cross")
    n = len(df)
    df["precio"] = np.round(RNG.uniform(8.5, 12.5, n), 2)
    df["promocion"] = (RNG.random(n) < 0.25).astype(int)
    df["publicidad"] = np.round(RNG.gamma(2, 1500, n), 0)
    df["trimestre"] = "T" + df["semana"].dt.quarter.astype(str)
    efecto_tipo = df["tipo"].map({"Centro comercial": 0.25, "Calle": 0.0, "Periferia": -0.15})
    efecto_trim = df["trimestre"].map({"T1": -0.08, "T2": 0.0, "T3": -0.12, "T4": 0.22})
    log_ventas = (7.2 - 1.6 * np.log(df["precio"]) + 0.18 * df["promocion"] + 0.06 * np.log1p(df["publicidad"])
                  + efecto_tipo + efecto_trim + df["efecto_tienda"] + RNG.normal(0, 0.12 + 0.05 * df["promocion"], n))
    df["unidades"] = np.round(np.exp(log_ventas)).astype(int)
    return df


def parte_regresion() -> pd.DataFrame:
    titulo("1. Factores de las ventas semanales (OLS log-log)")
    df = datos_ventas()
    print(f"{len(df):,} observaciones · {df['tienda'].nunique()} tiendas · {df['semana'].nunique()} semanas".replace(",", "."))
    formula = ("np.log(unidades) ~ np.log(precio) + promocion + np.log1p(publicidad) "
               "+ C(tipo, Treatment('Calle')) + C(trimestre)")

    ols = smf.ols(formula, df).fit()
    X = ols.model.exog
    vif = {n: variance_inflation_factor(X, i) for i, n in enumerate(ols.model.exog_names) if i > 0}
    _, p_bp, _, _ = het_breuschpagan(ols.resid, X)
    _, p_jb, *_ = jarque_bera(ols.resid)
    cook = ols.get_influence().cooks_distance[0]
    print(f"Diagnóstico: VIF máx. {num(max(vif.values()))} · Breusch-Pagan p = {p_bp:.1e} "
          f"(heterocedasticidad) · Jarque-Bera p = {num(p_jb)} · {np.sum(cook > 4 / len(df))} obs. influyentes")

    # Errores agrupados por tienda: corrigen heterocedasticidad y correlación dentro de cada tienda
    modelo = smf.ols(formula, df).fit(cov_type="cluster", cov_kwds={"groups": pd.factorize(df["tienda"])[0]})
    nombres = {"np.log(precio)": "Elasticidad precio", "promocion": "Promoción (sí/no)",
               "np.log1p(publicidad)": "Elasticidad publicidad"}
    ic = modelo.conf_int()
    print(f"\nR² ajustado: {modelo.rsquared_adj:.3f}  (errores agrupados por tienda)")
    for clave, texto in nombres.items():
        coef, p = modelo.params[clave], modelo.pvalues[clave]
        print(f"  {texto:<24} {coef:+.3f}  IC95 [{ic.loc[clave, 0]:+.3f}, {ic.loc[clave, 1]:+.3f}]  p = {p:.1e}")
    print(f"  → Una promoción aumenta las unidades un {np.expm1(modelo.params['promocion']):.1%}".replace(".", ","))
    print(f"  → Subir el precio un 1 % reduce las unidades un {abs(modelo.params['np.log(precio)']):.2f} %".replace(".", ","))
    for tipo in ["Centro comercial", "Periferia"]:
        clave = f"C(tipo, Treatment('Calle'))[T.{tipo}]"
        print(f"  {tipo:<24} {np.expm1(modelo.params[clave]):+.1%} frente a tiendas de calle".replace(".", ","))

    anova = sm.stats.anova_lm(ols, typ=2)
    anova["% suma cuadrados"] = anova["sum_sq"] / anova["sum_sq"].sum()
    print("\nANOVA tipo II (contribución de cada factor):")
    print(anova[["sum_sq", "F", "PR(>F)", "% suma cuadrados"]].round(4).to_string())

    # Escenarios de precio para una tienda de calle, sin promoción, T2, publicidad 3.000 €
    escenarios = pd.DataFrame({"precio": [9.0, 9.5, 10.0, 10.5, 11.0], "promocion": 0, "publicidad": 3000,
                               "tipo": "Calle", "trimestre": "T2"})
    pred = modelo.get_prediction(escenarios).summary_frame(alpha=0.05)
    sigma2 = ols.mse_resid
    escenarios["unidades"] = np.exp(pred["mean"] + sigma2 / 2)                      # Corrección de retransformación
    escenarios["ic_inf"] = np.exp(pred["mean_ci_lower"] + sigma2 / 2)
    escenarios["ic_sup"] = np.exp(pred["mean_ci_upper"] + sigma2 / 2)
    escenarios["ingresos"] = escenarios["precio"] * escenarios["unidades"]
    print("\nEscenarios de precio (tienda de calle, semana sin promoción):")
    print(escenarios[["precio", "unidades", "ic_inf", "ic_sup", "ingresos"]].round(1).to_string(index=False))
    print("  → Con elasticidad > 1 en valor absoluto, bajar el precio aumenta los ingresos.")
    return escenarios


# ============================================================ 2. Campaña de cupones
def parte_cupones() -> dict:
    titulo("2. Campaña de cupones (logística, prueba A/B y potencia)")
    n = 12_000
    df = pd.DataFrame({"edad": RNG.integers(18, 75, n),
                       "segmento": RNG.choice(["Nuevo", "Ocasional", "Frecuente", "VIP"], n, p=[.25, .4, .28, .07]),
                       "canal": RNG.choice(["Email", "App", "SMS"], n, p=[.5, .35, .15]),
                       "diseno": RNG.choice(["A", "B"], n)})
    lineal = (-3.0 + 0.012 * (df["edad"] - 40) + df["segmento"].map({"Nuevo": 0, "Ocasional": .35, "Frecuente": .8, "VIP": 1.2})
              + df["canal"].map({"Email": 0, "App": .45, "SMS": -.2}) + (df["diseno"] == "B") * 0.16)
    df["redime"] = (RNG.random(n) < 1 / (1 + np.exp(-lineal))).astype(int)

    logit = smf.logit("redime ~ edad + C(segmento, Treatment('Nuevo')) + C(canal, Treatment('Email')) + C(diseno)",
                      df).fit(disp=False)
    ic = np.exp(logit.conf_int())
    tabla = pd.DataFrame({"OR": np.exp(logit.params), "IC_inf": ic[0], "IC_sup": ic[1], "p": logit.pvalues}).iloc[1:]
    tabla.index = [i.replace("C(segmento, Treatment('Nuevo'))[T.", "Segmento ").replace("C(canal, Treatment('Email'))[T.", "Canal ")
                   .replace("C(diseno)[T.", "Diseño ").rstrip("]") for i in tabla.index]
    print(f"Tasa de redención global: {df['redime'].mean():.2%} · pseudo-R² {logit.prsquared:.3f}".replace(".", ","))
    print("Razones de probabilidades (OR > 1 aumenta la redención):")
    print(tabla.round(3).to_string())
    margen = logit.get_margeff(at="overall").summary_frame()
    fila_b = [i for i in margen.index if "diseno" in i][0]
    print(f"Efecto marginal medio del diseño B: {margen.loc[fila_b, 'dy/dx'] * 100:+.2f} puntos porcentuales".replace(".", ","))

    # Prueba A/B directa
    resumen = df.groupby("diseno")["redime"].agg(["sum", "count"])
    z, p = proportions_ztest(resumen["sum"][["B", "A"]].values, resumen["count"][["B", "A"]].values)
    ic_inf, ic_sup = confint_proportions_2indep(resumen.loc["B", "sum"], resumen.loc["B", "count"],
                                                resumen.loc["A", "sum"], resumen.loc["A", "count"], compare="diff")
    pa, pb = resumen["sum"] / resumen["count"]
    print(f"\nA/B: A {pa:.2%} · B {pb:.2%} · diferencia {pb - pa:+.2%} (IC95 {ic_inf:+.2%} a {ic_sup:+.2%}) · p = {p:.4f}".replace(".", ","))
    efecto = proportion_effectsize(pb, pa)
    n_necesario = NormalIndPower().solve_power(effect_size=efecto, alpha=0.05, power=0.8)
    potencia_actual = NormalIndPower().solve_power(effect_size=efecto, nobs1=resumen.loc["A", "count"], alpha=0.05)
    print(f"Para detectar esa diferencia con potencia del 80 % hacen falta {num(np.ceil(n_necesario), 0)} clientes por diseño "
          f"(potencia con la muestra actual: {potencia_actual:.0%})")

    # Efecto del diseño B por segmento con corrección por comparaciones múltiples
    filas = []
    for seg, g in df.groupby("segmento"):
        r = g.groupby("diseno")["redime"].agg(["sum", "count"])
        _, p_seg = proportions_ztest(r["sum"][["B", "A"]].values, r["count"][["B", "A"]].values)
        filas.append((seg, r.loc["B", "sum"] / r.loc["B", "count"] - r.loc["A", "sum"] / r.loc["A", "count"], p_seg))
    segmentos = pd.DataFrame(filas, columns=["segmento", "diferencia_B_A", "p"])
    segmentos["p_ajustado_holm"] = multipletests(segmentos["p"], method="holm")[1]
    print("\nEfecto del diseño B por segmento (p ajustados por Holm):")
    print(segmentos.round(4).to_string(index=False))
    return {"tasa_A": pa, "tasa_B": pb, "p": p, "n_necesario": int(np.ceil(n_necesario))}


# ============================================================ 3. Previsión de demanda
def parte_prevision() -> pd.DataFrame:
    titulo("3. Previsión de la demanda mensual")
    fechas = pd.date_range("2018-01-01", "2026-10-01", freq="MS")
    t = np.arange(len(fechas))
    promo = pd.Series(((fechas.month.isin([3, 7, 11])) | (RNG.random(len(fechas)) < 0.08)).astype(int), index=fechas)
    estacional = 1 + 0.18 * np.sin(2 * np.pi * (fechas.month - 4) / 12) + 0.25 * (fechas.month == 12)
    ruido = np.zeros(len(fechas))
    for i in range(1, len(fechas)):
        ruido[i] = 0.5 * ruido[i - 1] + RNG.normal(0, 0.035)
    serie = pd.Series(np.round(20_000 * (1 + 0.006 * t) * estacional * (1 + 0.12 * promo) * np.exp(ruido)),
                      index=fechas, name="unidades").asfreq("MS")
    serie[fechas.year == 2020] *= np.where(fechas[fechas.year == 2020].month.isin([4, 5]), 0.55, 1)   # Cierre temporal
    pandemia = pd.Series(((fechas.year == 2020) & fechas.month.isin([4, 5])).astype(int), index=fechas)
    exog = pd.DataFrame({"promocion": promo, "cierre": pandemia})

    stl = STL(np.log(serie), period=12, robust=True).fit()
    fuerza = 1 - stl.resid.var() / (stl.seasonal + stl.resid).var()
    adf = adfuller(np.log(serie), autolag="AIC", result_object=True)
    adf_d = adfuller(np.log(serie).diff(12).dropna(), autolag="AIC", result_object=True)
    print(f"Observaciones: {len(serie)} meses · fuerza estacional (STL) {fuerza:.2f}")
    print(f"ADF nivel p = {adf.pvalue:.3f} (no estacionaria) · tras diferencia estacional p = {adf_d.pvalue:.3f}")

    corte = -12
    y_tr, y_te = np.log(serie[:corte]), serie[corte:]
    x_tr, x_te = exog[:corte], exog[corte:]
    with warnings.catch_warnings():
        warnings.simplefilter("ignore")
        sarimax = SARIMAX(y_tr, exog=x_tr, order=(1, 0, 0), seasonal_order=(0, 1, 1, 12), trend="c").fit(disp=False)
        ets = ETSModel(y_tr, error="add", trend="add", damped_trend=True, seasonal="add", seasonal_periods=12).fit(disp=False)
    previsiones = {"SARIMAX + promociones": np.exp(sarimax.forecast(12, exog=x_te)),
                   "ETS amortiguado": np.exp(ets.forecast(12)),
                   "Ingenuo estacional": serie.shift(12)[corte:]}
    escala = (serie[:corte] - serie[:corte].shift(12)).abs().mean()
    print(f"\nValidación en los últimos 12 meses ({y_te.index[0]:%m/%Y}–{y_te.index[-1]:%m/%Y}):")
    for nombre, p in previsiones.items():
        mae = (y_te - p).abs().mean()
        print(f"  {nombre:<24} MAPE {((y_te - p).abs() / y_te).mean():6.2%}   MASE {mae / escala:.2f}".replace(".", ","))
    print(f"Coeficiente de promoción (SARIMAX): {np.expm1(sarimax.params['promocion']):+.1%} de unidades".replace(".", ","))

    with warnings.catch_warnings():
        warnings.simplefilter("ignore")
        final = SARIMAX(np.log(serie), exog=exog, order=(1, 0, 0), seasonal_order=(0, 1, 1, 12), trend="c").fit(disp=False)
    futuras = pd.date_range(serie.index[-1] + pd.offsets.MonthBegin(), periods=12, freq="MS")
    exog_fut = pd.DataFrame({"promocion": futuras.month.isin([3, 7, 11]).astype(int), "cierre": 0}, index=futuras)
    resumen = np.exp(final.get_forecast(12, exog=exog_fut).summary_frame(alpha=0.05)[["mean", "mean_ci_lower", "mean_ci_upper"]])
    resumen.columns = ["prevision", "ic95_inf", "ic95_sup"]
    resumen.insert(0, "promocion", exog_fut["promocion"])
    print("\nPrevisión de los próximos 12 meses (unidades):")
    print(resumen.round(0).astype(int).to_string())
    crecimiento = f"{resumen['prevision'].sum() / serie[-12:].sum() - 1:+.1%}".replace(".", ",")
    print(f"Total previsto: {num(resumen['prevision'].sum(), 0)} unidades ({crecimiento} frente a los últimos 12 meses)")

    fig, ejes = plt.subplots(2, 1, figsize=(11, 7), layout="constrained")
    ejes[0].plot(serie[-48:], color="#1F4E78", label="Real")
    for (nombre, p), color in zip(previsiones.items(), ["#C55A11", "#548235", "#A6A6A6"]):
        ejes[0].plot(p, linestyle="--", color=color, label=nombre)
    ejes[0].set_title("Validación: último año reservado")
    ejes[0].legend(fontsize=8)
    ejes[1].plot(serie[-36:], color="#1F4E78", label="Real")
    ejes[1].plot(resumen["prevision"], color="#C00000", label="Previsión")
    ejes[1].fill_between(resumen.index, resumen["ic95_inf"], resumen["ic95_sup"], color="#C00000", alpha=0.15, label="IC 95 %")
    ejes[1].set_title("Previsión a 12 meses (SARIMAX con promociones)")
    ejes[1].legend(fontsize=8)
    for eje in ejes:
        eje.yaxis.set_major_formatter(matplotlib.ticker.FuncFormatter(lambda v, _: num(v, 0)))
        eje.grid(alpha=0.3)
    fig.savefig(SALIDA / "prevision_demanda.png", dpi=130)
    return resumen


def main() -> int:
    escenarios = parte_regresion()
    ab = parte_cupones()
    prevision = parte_prevision()
    with pd.ExcelWriter(SALIDA / "estudio_comercial.xlsx") as excel:
        escenarios.to_excel(excel, sheet_name="Escenarios precio", index=False)
        pd.DataFrame([ab]).to_excel(excel, sheet_name="Prueba AB", index=False)
        prevision.rename_axis("mes").reset_index().to_excel(excel, sheet_name="Prevision", index=False)
    print(f"\n✔ Gráfico y Excel guardados en «{SALIDA}»")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Lectura de los resultados:

- **Elasticidad precio ≈ −1,5** (demanda elástica): bajar el precio aumenta los ingresos, como confirman los escenarios. Los errores agrupados por tienda son necesarios porque el contraste de Breusch-Pagan detecta heterocedasticidad.
- **Prueba A/B**: la diferencia entre diseños no es significativa y el intervalo incluye el cero. El cálculo de potencia muestra que con 6.000 clientes por diseño solo había un 8 % de probabilidad de detectar una diferencia de ese tamaño: el experimento no puede concluir nada y haría falta una muestra mucho mayor o un efecto más grande.
- **Previsión**: los modelos mejoran al ingenuo estacional (MASE < 1), pero por poco. SARIMAX aporta además el efecto cuantificado de las promociones y permite planificarlas en la previsión.

---

## 20. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| Coeficientes sin constante con `sm.OLS` | La interfaz de matrices no añade la constante. | `sm.add_constant(X)` o use fórmulas. |
| `PatsyError: Error evaluating factor` | Nombre de columna con espacios o caracteres especiales. | `Q('nombre columna')` o renombrar. |
| `MissingDataError: exog contains inf or nans` | Valores ausentes o infinitos (p. ej., `log(0)`). | Limpie los datos; `np.log1p`; `missing="drop"`. |
| `PerfectSeparationError` / coeficientes enormes en logit | Una variable predice perfectamente la respuesta. | Revise la variable; agrupe categorías; regularización (`fit_regularized`). |
| `ConvergenceWarning` / `HessianInversionWarning` | El optimizador no converge. | Escale las variables, aumente `maxiter`, simplifique el modelo. |
| `ValueWarning: No frequency information was provided` | Serie sin frecuencia. | `serie.asfreq("MS")` (o la frecuencia adecuada). |
| Error al prever con `exog` | No se pasaron los regresores futuros. | `forecast(steps, exog=futuro)` con las mismas columnas. |
| `LinAlgError: Singular matrix` | Columnas colineales (p. ej., todas las ficticias más la constante). | Elimine variables redundantes; revise VIF. |
| p-valores demasiado optimistas | Heterocedasticidad, autocorrelación o datos agrupados. | `cov_type="HC3"`, `"HAC"` o `"cluster"`. |
| `FutureWarning` en `adfuller`/`kpss` | Cambio de tipo de resultado anunciado. | `result_object=True`. |
| Predicciones sesgadas en modelos con `log(y)` | Retransformación sin corrección. | Multiplique por `exp(σ²/2)` o use un GLM con enlace logarítmico. |

---

## 21. Buenas prácticas

- Plantee la pregunta (¿explicar o predecir?) antes de elegir el modelo.
- Explore los datos y represente las relaciones antes de modelizar.
- Diagnostique siempre los supuestos y use errores robustos cuando proceda.
- Informe de efectos con **intervalos de confianza** y en unidades comprensibles (porcentajes, euros), no solo p-valores.
- Corrija por comparaciones múltiples cuando contraste muchas hipótesis.
- Calcule la potencia y el tamaño muestral **antes** de un experimento.
- En series temporales, valide con datos fuera de muestra y compare con un modelo ingenuo.
- Documente las transformaciones y la categoría de referencia de cada variable categórica.

---

## 22. Referencia rápida

```python
smf.ols("y ~ x1 + C(cat) + np.log(x2)", df).fit(cov_type="HC3"|"cluster") · .summary() · .params · .pvalues · .conf_int()
.get_prediction(nuevos).summary_frame() · sm.stats.anova_lm(m, typ=2) · m.f_test("x1 = 0") · m.compare_lr_test(m0)
variance_inflation_factor(X, i) · het_breuschpagan(resid, X) · jarque_bera(resid) · durbin_watson(resid) · m.get_influence()
smf.logit(...).fit(disp=False) · np.exp(params) · .get_margeff().summary() · smf.poisson · smf.glm(family=sm.families.Gamma())
smf.quantreg(...).fit(q=0.9) · smf.mixedlm(..., groups=) · smf.wls(..., weights=)
proportions_ztest · confint_proportions_2indep · NormalIndPower().solve_power · multipletests(p, method="holm")
STL(serie, period=12).fit() · adfuller(serie, result_object=True).pvalue · acorr_ljungbox · plot_acf/plot_pacf
SARIMAX(y, exog=, order=(p,d,q), seasonal_order=(P,D,Q,s)).fit(disp=False).get_forecast(h, exog=).summary_frame()
ETSModel(y, error=, trend=, seasonal=, seasonal_periods=).fit() · ExponentialSmoothing(...).fit().forecast(h)
```

---

## 23. Ejercicios propuestos

1. Añada al modelo de ventas una interacción `precio × tipo de tienda` y contraste si la elasticidad difiere entre tipos.
2. Ajuste un modelo de efectos mixtos con pendiente aleatoria de precio por tienda y compárelo con el modelo de errores agrupados.
3. Modele el número de reclamaciones mensuales por tienda con Poisson y binomial negativa, y elija el adecuado según la sobredispersión.
4. Diseñe un experimento de precios: calcule el tamaño muestral necesario para detectar un cambio del 3 % en las ventas medias.
5. Implemente una validación cruzada temporal (origen móvil) para comparar SARIMAX y ETS en 24 meses.

---

## 24. Referencias

- Documentación oficial: https://www.statsmodels.org/stable/
- Guía de usuario por temas: https://www.statsmodels.org/stable/user-guide.html
- Fórmulas (patsy / formulaic): https://www.statsmodels.org/stable/example_formulas.html
- Series temporales: https://www.statsmodels.org/stable/tsa.html
- Hyndman y Athanasopoulos, *Forecasting: Principles and Practice*: https://otexts.com/fpp3/
