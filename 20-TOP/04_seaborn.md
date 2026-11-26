# Manual de uso: Seaborn

> **Categoría:** Ciencia de datos · **Utilidad:** Visualización estadística
> **Tipo:** Librería de terceros (`pip install seaborn`) · **Versión de referencia:** 0.13

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Datos en formato «largo»](#4-datos-en-formato-largo)
5. [Temas, contextos y paletas](#5-temas-contextos-y-paletas)
6. [Relaciones: `scatterplot`, `lineplot`, `relplot`](#6-relaciones-scatterplot-lineplot-relplot)
7. [Distribuciones: `histplot`, `kdeplot`, `ecdfplot`, `displot`](#7-distribuciones-histplot-kdeplot-ecdfplot-displot)
8. [Categorías: `boxplot`, `violinplot`, `barplot`, `catplot`…](#8-categorías-boxplot-violinplot-barplot-catplot)
9. [Regresión: `regplot`, `lmplot`, `residplot`](#9-regresión-regplot-lmplot-residplot)
10. [Matrices: `heatmap` y `clustermap`](#10-matrices-heatmap-y-clustermap)
11. [Varias variables: `pairplot` y `jointplot`](#11-varias-variables-pairplot-y-jointplot)
12. [Facetas con `FacetGrid` y funciones de figura](#12-facetas-con-facetgrid-y-funciones-de-figura)
13. [Intervalos de confianza y barras de error](#13-intervalos-de-confianza-y-barras-de-error)
14. [Personalizar con Matplotlib](#14-personalizar-con-matplotlib)
15. [La interfaz de objetos (`seaborn.objects`)](#15-la-interfaz-de-objetos-seabornobjects)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Seaborn** es una librería de **visualización estadística** construida sobre Matplotlib y diseñada para trabajar directamente con **DataFrames de pandas**. Con una sola línea genera gráficos que agregan datos, calculan intervalos de confianza, ajustan regresiones, estiman densidades y dividen la información en paneles por grupos, con una estética cuidada por defecto.

**Ventajas:**
- Sintaxis declarativa: se indican columnas (`x=`, `y=`, `hue=`) y Seaborn hace el resto.
- Estadística integrada: medias con intervalos de confianza, KDE, regresiones, ECDF.
- Paneles por grupos (*facetas*) con un parámetro (`col=`, `row=`).
- Temas y paletas atractivos y coherentes.
- Totalmente compatible con Matplotlib para el ajuste fino.

**Limitaciones:**
- Gráficos estáticos (sin interactividad).
- Menos control directo que Matplotlib en diseños muy personalizados.
- La función `load_dataset()` descarga los datos de ejemplo de Internet.

| Si necesita… | Opción |
|---|---|
| Exploración estadística rápida de un DataFrame | **Seaborn** |
| Control total del diseño de un gráfico | Matplotlib |
| Gráficos interactivos | Plotly |
| Contrastes de hipótesis y modelos | SciPy / statsmodels |

---

## 2. Instalación e importación

```bash
pip install seaborn            # Instala también pandas y Matplotlib
```

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

sns.set_theme(style="whitegrid", context="notebook", palette="deep")
propinas = sns.load_dataset("tips")      # Conjunto de ejemplo (requiere conexión a Internet)
```

> Conjuntos de ejemplo disponibles con `sns.get_dataset_names()`: `tips`, `penguins`, `titanic`, `flights`, `diamonds`, `iris`… En equipos sin Internet, use sus propios DataFrames.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Variables semánticas | Columnas asignadas a propiedades visuales: `x`, `y`, `hue` (color), `size`, `style`, `col`, `row`. |
| Función de ejes (*axes-level*) | Dibuja en un `Axes` de Matplotlib (`scatterplot`, `histplot`, `boxplot`…). Acepta `ax=`. |
| Función de figura (*figure-level*) | Crea su propia figura con facetas (`relplot`, `displot`, `catplot`, `lmplot`). Devuelve un `FacetGrid`. |
| Estimación | Agregación automática (media por defecto) con intervalo de confianza. |
| Formato largo (*tidy*) | Una fila por observación y una columna por variable. |
| Tema / contexto | Estilo global y escala de los elementos (papel, cuaderno, charla, póster). |

| Familia | Función de figura | Funciones de ejes |
|---|---|---|
| Relaciones | `relplot` | `scatterplot`, `lineplot` |
| Distribuciones | `displot` | `histplot`, `kdeplot`, `ecdfplot`, `rugplot` |
| Categorías | `catplot` | `stripplot`, `swarmplot`, `boxplot`, `violinplot`, `boxenplot`, `pointplot`, `barplot`, `countplot` |
| Regresión | `lmplot` | `regplot`, `residplot` |

---

## 4. Datos en formato «largo»

Seaborn funciona mejor con datos en formato largo:

```python
# Ancho (una columna por año) → difícil de usar con hue
ancho = pd.DataFrame({"region": ["Norte", "Sur"], "2025": [120, 95], "2026": [135, 90]})

# Largo (una fila por región y año) → ideal para Seaborn
largo = ancho.melt(id_vars="region", var_name="anio", value_name="ventas")
sns.barplot(largo, x="region", y="ventas", hue="anio")
```

---

## 5. Temas, contextos y paletas

```python
sns.set_theme(style="whitegrid")       # darkgrid (defecto), whitegrid, dark, white, ticks
sns.set_context("talk")                # paper < notebook (defecto) < talk < poster
sns.set_context("notebook", font_scale=1.1, rc={"lines.linewidth": 2})
sns.despine()                          # Quitar bordes superior y derecho
sns.despine(left=True, bottom=True, offset=5)

with sns.axes_style("white"):          # Estilo temporal
    sns.histplot(datos)

sns.set_palette("Set2")
sns.color_palette("viridis", 5)        # Lista de colores
sns.color_palette("Blues", as_cmap=True)
sns.diverging_palette(220, 20, as_cmap=True)
sns.light_palette("#1F4E78", 6) ; sns.dark_palette("seagreen", 4)
sns.palplot(sns.color_palette("husl", 8))   # Previsualizar
```

| Tipo de paleta | Uso | Ejemplos |
|---|---|---|
| Cualitativa | Categorías sin orden. | `deep`, `muted`, `Set2`, `tab10`, `colorblind` |
| Secuencial | Valores ordenados. | `rocket`, `mako`, `viridis`, `Blues` |
| Divergente | Valores alrededor de un centro. | `vlag`, `icefire`, `RdBu`, `coolwarm` |

> La paleta `colorblind` es una buena opción por defecto para que los gráficos sean legibles por personas con daltonismo.

---

## 6. Relaciones: `scatterplot`, `lineplot`, `relplot`

```python
sns.scatterplot(df, x="superficie", y="precio", hue="barrio", size="habitaciones",
                style="tipo", sizes=(20, 200), alpha=0.7)

sns.lineplot(ventas, x="mes", y="importe", hue="region", marker="o",
             errorbar=("ci", 95))           # Media por mes con IC del 95 % (si hay varias filas por mes)
sns.lineplot(ventas, x="mes", y="importe", estimator="sum", errorbar=None)

sns.relplot(df, x="superficie", y="precio", hue="tipo", col="ciudad", col_wrap=3,
            kind="scatter", height=3.5, aspect=1.1)
sns.relplot(ventas, x="fecha", y="importe", kind="line", row="canal", facet_kws={"sharey": False})
```

---

## 7. Distribuciones: `histplot`, `kdeplot`, `ecdfplot`, `displot`

```python
sns.histplot(df, x="importe", bins=30, kde=True)
sns.histplot(df, x="importe", hue="canal", element="step", stat="density", common_norm=False)
sns.histplot(df, x="importe", hue="canal", multiple="stack")       # layer | dodge | stack | fill
sns.histplot(df, x="importe", log_scale=True)
sns.histplot(df, x="edad", y="ingresos", cbar=True)                # Histograma 2D

sns.kdeplot(df, x="importe", hue="canal", fill=True, alpha=0.3, bw_adjust=0.8, cut=0)
sns.kdeplot(df, x="x", y="y", levels=8, fill=True, cmap="mako")    # Densidad 2D
sns.ecdfplot(df, x="tiempo_entrega", hue="transportista")           # Distribución acumulada
sns.rugplot(df, x="importe", height=0.03)

sns.displot(df, x="importe", col="region", kind="hist", kde=True, col_wrap=3)
sns.displot(df, x="importe", hue="canal", kind="kde", fill=True)
```

| `stat` | Eje Y |
|---|---|
| `count` (defecto) | Número de observaciones. |
| `frequency` | Número dividido por el ancho de clase. |
| `probability` / `percent` | Proporción / porcentaje. |
| `density` | Densidad (el área total suma 1). |

> Con grupos de distinto tamaño, use `stat="density"` y `common_norm=False` para comparar formas.

---

## 8. Categorías: `boxplot`, `violinplot`, `barplot`, `catplot`…

```python
# Observaciones individuales
sns.stripplot(df, x="region", y="importe", jitter=0.25, alpha=0.5)
sns.swarmplot(df, x="region", y="importe", size=3)                  # Sin solapamientos (pocos datos)

# Distribuciones por categoría
sns.boxplot(df, x="region", y="importe", hue="canal", showfliers=False)
sns.violinplot(df, x="region", y="importe", hue="canal", split=True, inner="quart")
sns.boxenplot(df, x="region", y="importe")                          # Más cuantiles (muchos datos)

# Estimaciones (media + intervalo)
sns.barplot(df, x="region", y="importe", estimator="mean", errorbar=("ci", 95))
sns.pointplot(df, x="mes", y="importe", hue="canal", dodge=True, markers=["o", "s"])
sns.countplot(df, x="region", order=df["region"].value_counts().index)   # Frecuencias

sns.catplot(df, x="region", y="importe", hue="canal", col="anio", kind="box", height=4)
```

Opciones comunes: `order=`, `hue_order=`, `orient="h"` (o asignar la categoría a `y`), `palette=`, `dodge=`, `native_scale=True` (escala numérica real en el eje categórico), `log_scale=`, `legend=False`.

> Desde la versión 0.13, `palette=` sin `hue=` genera un aviso: indique `hue` explícitamente (`hue="region", legend=False`).

Combinar una distribución y las observaciones:

```python
ax = sns.boxplot(df, x="region", y="importe", color="white", showfliers=False)
sns.stripplot(df, x="region", y="importe", ax=ax, alpha=0.4, size=3)
```

---

## 9. Regresión: `regplot`, `lmplot`, `residplot`

```python
sns.regplot(df, x="publicidad", y="ventas", ci=95, scatter_kws={"alpha": 0.5}, line_kws={"color": "red"})
sns.regplot(df, x="x", y="y", order=2)                  # Polinómica
sns.regplot(df, x="x", y="y", lowess=True)             # Suavizado no paramétrico (requiere statsmodels)
sns.regplot(df, x="precio", y="compra", logistic=True)  # Variable binaria (requiere statsmodels)
sns.regplot(df, x="x", y="y", robust=True)              # Resistente a atípicos (requiere statsmodels)
sns.residplot(df, x="publicidad", y="ventas", lowess=True)   # Residuos

sns.lmplot(df, x="publicidad", y="ventas", hue="canal", col="region", height=3.5)
```

> Seaborn dibuja la recta pero **no devuelve** sus coeficientes. Para obtenerlos, use `numpy.polyfit`, `scipy.stats.linregress` o statsmodels.

---

## 10. Matrices: `heatmap` y `clustermap`

```python
corr = df.select_dtypes("number").corr()
mascara = np.triu(np.ones_like(corr, dtype=bool))       # Ocultar el triángulo superior
sns.heatmap(corr, mask=mascara, annot=True, fmt=".2f", cmap="vlag", center=0, vmin=-1, vmax=1,
            square=True, linewidths=0.5, cbar_kws={"shrink": 0.7})

tabla = df.pivot_table(index="dia_semana", columns="hora", values="pedidos", aggfunc="mean")
sns.heatmap(tabla, cmap="rocket_r", annot=True, fmt=".0f")

sns.clustermap(tabla, standard_scale=1, cmap="mako", figsize=(8, 8))   # Agrupa filas y columnas similares
```

---

## 11. Varias variables: `pairplot` y `jointplot`

```python
sns.pairplot(df, vars=["precio", "superficie", "antiguedad"], hue="tipo", corner=True,
             diag_kind="kde", plot_kws={"alpha": 0.5, "s": 15})

sns.jointplot(df, x="superficie", y="precio", kind="reg")    # scatter | kde | hist | hex | reg | resid
sns.jointplot(df, x="superficie", y="precio", hue="tipo", kind="kde")

g = sns.JointGrid(df, x="superficie", y="precio")
g.plot(sns.scatterplot, sns.histplot)
```

---

## 12. Facetas con `FacetGrid` y funciones de figura

Las funciones de figura (`relplot`, `displot`, `catplot`, `lmplot`) devuelven un `FacetGrid`:

```python
g = sns.catplot(df, x="canal", y="importe", col="region", col_wrap=3, kind="bar", height=3, aspect=1.2)
g.set_titles("{col_name}")                                # Títulos de cada panel
g.set_axis_labels("Canal", "Importe medio (€)")
g.set(ylim=(0, None))
g.figure.suptitle("Importe medio por canal y región", y=1.03)
g.tick_params(axis="x", rotation=30)
g.refline(y=df["importe"].mean(), color="gray", linestyle="--")
for ax in g.axes.flat:                                    # Acceso a cada Axes
    ax.yaxis.set_major_formatter("{x:.0f} €")
g.savefig("facetas.png", dpi=150)
```

`FacetGrid` manual:

```python
g = sns.FacetGrid(df, col="region", row="canal", margin_titles=True, height=2.5)
g.map_dataframe(sns.histplot, x="importe", bins=20)
g.add_legend()
```

> Las funciones de figura **no** aceptan `ax=`: crean su propia figura. Para combinar varios gráficos en una figura propia de Matplotlib, use las funciones de ejes (`scatterplot`, `histplot`…) con `ax=`.

---

## 13. Intervalos de confianza y barras de error

El parámetro `errorbar` controla la incertidumbre que se muestra en `barplot`, `pointplot` y `lineplot`:

| Valor | Muestra | Tipo |
|---|---|---|
| `("ci", 95)` (defecto) | Intervalo de confianza del 95 % de la media (*bootstrap*). | Incertidumbre del estimador |
| `("se", 1)` | Error estándar. | Incertidumbre del estimador |
| `("sd", 1)` | Desviación típica. | Dispersión de los datos |
| `("pi", 50)` | Intervalo de percentiles (p. ej., 25–75). | Dispersión de los datos |
| `None` | Sin barras. | — |

```python
sns.barplot(df, x="region", y="importe", errorbar=("sd", 1), capsize=0.2)
sns.lineplot(df, x="mes", y="importe", errorbar=("pi", 80))
sns.barplot(df, x="region", y="importe", estimator="median", errorbar=("ci", 90), n_boot=2000, seed=1)
```

> Indique siempre en el pie del gráfico qué representan las barras de error: «IC 95 %» y «desviación típica» transmiten cosas distintas.

---

## 14. Personalizar con Matplotlib

Las funciones de ejes devuelven un `Axes` de Matplotlib:

```python
fig, axs = plt.subplots(1, 2, figsize=(12, 4.5), layout="constrained")
ax = sns.histplot(df, x="importe", ax=axs[0], color="#1F4E78")
ax.set(title="Distribución del importe", xlabel="Importe (€)", ylabel="Pedidos")
ax.axvline(df["importe"].median(), color="orange", linestyle="--", label="Mediana")
ax.legend()

ax2 = sns.barplot(df, x="region", y="importe", ax=axs[1], hue="region", legend=False, palette="crest")
for contenedor in ax2.containers:
    ax2.bar_label(contenedor, fmt="%.0f €", padding=3, fontsize=8)
sns.move_legend(ax, "upper left", bbox_to_anchor=(1, 1))     # Mover leyendas generadas por Seaborn
fig.savefig("analisis.png", dpi=150)
```

---

## 15. La interfaz de objetos (`seaborn.objects`)

Interfaz declarativa (inspirada en la «gramática de gráficos») para construir gráficos por capas:

```python
import seaborn.objects as so

(
    so.Plot(df, x="publicidad", y="ventas", color="canal")
    .add(so.Dot(alpha=0.5))
    .add(so.Line(), so.PolyFit(order=1))          # Recta de ajuste por canal
    .facet(col="region", wrap=3)
    .label(x="Inversión publicitaria (k€)", y="Ventas (k€)", color="Canal")
    .theme({"axes.facecolor": "white"})
    .save("capas.png", dpi=150, bbox_inches="tight")
)

so.Plot(df, x="region", y="importe").add(so.Bar(), so.Agg("mean")).add(so.Range(), so.Est(errorbar="se"))
so.Plot(df, x="importe").add(so.Bars(), so.Hist(bins=30))
```

> La interfaz de objetos está en desarrollo activo; para el uso diario, las funciones clásicas siguen siendo las más estables y documentadas.

---

## 16. Ejemplo práctico completo

**Análisis visual de la calidad del servicio de entregas**: genera un conjunto de datos realista de pedidos (región, transportista, canal, distancia, tiempo de entrega, importe y valoración del cliente), y produce un informe de seis figuras: distribución de tiempos por transportista (histograma + ECDF), comparación por región y transportista (violín y medias con IC), relación distancia–tiempo con regresión por transportista, matriz de correlación, mapa de calor día × franja horaria y panel de facetas de la valoración. Calcula además estadísticos de apoyo y guarda un resumen en CSV.

```python
"""
analisis_entregas.py — Análisis estadístico visual con Seaborn.
Requiere: pip install seaborn pandas numpy matplotlib
Uso: python analisis_entregas.py [--salida informe_entregas]
"""
import argparse
import warnings
from pathlib import Path

import matplotlib

matplotlib.use("Agg")
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sns

warnings.filterwarnings("ignore", category=matplotlib.MatplotlibDeprecationWarning)   # Avisos internos de Seaborn 0.13 con Matplotlib ≥ 3.11

DIAS = ["Lun", "Mar", "Mié", "Jue", "Vie", "Sáb", "Dom"]
FRANJAS = ["8-11", "11-14", "14-17", "17-20", "20-23"]
TRANSPORTISTAS = {"RápidoExpress": (1.00, 0.12), "LogiSur": (1.18, 0.20), "EnvíaYa": (0.92, 0.30)}


def generar_datos(n: int = 6000, semilla: int = 11) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    region = rng.choice(["Norte", "Sur", "Este", "Oeste", "Centro"], n, p=[0.2, 0.2, 0.2, 0.15, 0.25])
    transportista = rng.choice(list(TRANSPORTISTAS), n, p=[0.45, 0.30, 0.25])
    canal = rng.choice(["Web", "App", "Tienda"], n, p=[0.5, 0.35, 0.15])
    distancia = rng.gamma(2.2, 18, n) + np.where(region == "Centro", 0, 15)
    factor, ruido = np.array([TRANSPORTISTAS[t] for t in transportista]).T
    dia = rng.integers(0, 7, n)
    franja = rng.choice(FRANJAS, n, p=[0.15, 0.3, 0.25, 0.2, 0.1])
    horas = (18 + 0.35 * distancia) * factor * rng.lognormal(0, ruido) + np.where(dia >= 5, 14, 0)
    importe = rng.lognormal(3.9, 0.6, n).round(2)
    # Valoración (1-5) que empeora con el tiempo de entrega
    base = 5.2 - 0.022 * horas + rng.normal(0, 0.6, n) + np.where(canal == "App", 0.15, 0)
    valoracion = np.clip(np.round(base), 1, 5).astype(int)
    return pd.DataFrame({
        "region": region, "transportista": transportista, "canal": canal,
        "distancia_km": distancia.round(1), "horas_entrega": horas.round(1),
        "dia": pd.Categorical(np.array(DIAS)[dia], categories=DIAS, ordered=True),
        "franja": pd.Categorical(franja, categories=FRANJAS, ordered=True),
        "importe": importe, "valoracion": valoracion,
        "a_tiempo": horas <= 48,
    })


def guardar(fig_o_grid, ruta: Path) -> None:
    fig = getattr(fig_o_grid, "figure", fig_o_grid)
    fig.savefig(ruta, dpi=140, bbox_inches="tight")
    plt.close(fig)


# ------------------------------------------------------------------ Figuras
def fig_distribuciones(df: pd.DataFrame, salida: Path) -> None:
    fig, axs = plt.subplots(1, 2, figsize=(13, 4.8), layout="constrained")
    sns.histplot(df, x="horas_entrega", hue="transportista", stat="density", common_norm=False,
                 element="step", fill=True, alpha=0.25, bins=50, ax=axs[0])
    axs[0].axvline(48, color="black", linestyle="--", linewidth=1)
    axs[0].text(49, axs[0].get_ylim()[1] * 0.92, "Plazo comprometido: 48 h", fontsize=9)
    axs[0].set(title="Distribución del tiempo de entrega", xlabel="Horas", ylabel="Densidad", xlim=(0, 140))
    sns.ecdfplot(df, x="horas_entrega", hue="transportista", ax=axs[1], linewidth=2)
    axs[1].axvline(48, color="black", linestyle="--", linewidth=1)
    axs[1].yaxis.set_major_formatter("{x:.0%}")
    axs[1].set(title="Proporción acumulada de pedidos entregados", xlabel="Horas", ylabel="Pedidos entregados",
               xlim=(0, 140))
    sns.move_legend(axs[1], "lower right", title=None)
    fig.suptitle("1. ¿Cuánto tardan los pedidos?", fontsize=15, fontweight="bold", x=0.01, ha="left")
    guardar(fig, salida / "1_distribuciones.png")


def fig_comparacion(df: pd.DataFrame, salida: Path) -> None:
    orden = df.groupby("region")["horas_entrega"].median().sort_values().index
    fig, axs = plt.subplots(1, 2, figsize=(14, 5), layout="constrained", width_ratios=[1.4, 1])
    sns.violinplot(df, x="region", y="horas_entrega", hue="transportista", order=orden, inner="quart",
                   cut=0, density_norm="width", linewidth=0.8, ax=axs[0])
    axs[0].set(title="Distribución por región y transportista", xlabel=None, ylabel="Horas", ylim=(0, 150))
    sns.move_legend(axs[0], "upper left", ncols=3, title=None, frameon=False)
    sns.pointplot(df, x="transportista", y="a_tiempo", hue="canal", dodge=0.35, errorbar=("ci", 95),
                  capsize=0.08, markers=["o", "s", "D"], ax=axs[1])
    axs[1].yaxis.set_major_formatter("{x:.0%}")
    axs[1].set(title="Entregas a tiempo (media e IC 95 %)", xlabel=None, ylabel="% dentro de 48 h")
    fig.suptitle("2. Comparación entre regiones, transportistas y canales", fontsize=15, fontweight="bold",
                 x=0.01, ha="left")
    guardar(fig, salida / "2_comparacion.png")


def fig_regresion(df: pd.DataFrame, salida: Path) -> None:
    muestra = df.sample(1500, random_state=1)
    g = sns.lmplot(muestra, x="distancia_km", y="horas_entrega", hue="transportista", col="transportista",
                   height=4, aspect=1.05, scatter_kws={"alpha": 0.25, "s": 10}, line_kws={"linewidth": 2.5},
                   facet_kws={"sharey": True}, legend=False)
    for ax, (nombre, grupo) in zip(g.axes.flat, df.groupby("transportista", sort=True)):
        pendiente, origen = np.polyfit(grupo["distancia_km"], grupo["horas_entrega"], 1)
        r = np.corrcoef(grupo["distancia_km"], grupo["horas_entrega"])[0, 1]
        ax.text(0.03, 0.95, f"horas = {origen:.1f} + {pendiente:.2f}·km\nr = {r:.2f}", transform=ax.transAxes,
                va="top", fontsize=9, bbox={"facecolor": "white", "alpha": 0.8, "edgecolor": "none"})
    g.set_titles("{col_name}")
    g.set_axis_labels("Distancia (km)", "Horas de entrega")
    g.set(ylim=(0, 150))
    g.figure.suptitle("3. Relación entre distancia y tiempo de entrega", fontsize=15, fontweight="bold",
                      x=0.01, ha="left", y=1.05)
    guardar(g, salida / "3_regresion.png")


def fig_correlaciones(df: pd.DataFrame, salida: Path) -> None:
    numericas = df[["distancia_km", "horas_entrega", "importe", "valoracion"]].assign(
        a_tiempo=df["a_tiempo"].astype(int), fin_de_semana=df["dia"].isin(["Sáb", "Dom"]).astype(int))
    corr = numericas.corr(method="spearman")
    mascara = np.triu(np.ones_like(corr, dtype=bool), k=1)
    fig, ax = plt.subplots(figsize=(7.5, 6), layout="constrained")
    sns.heatmap(corr, mask=mascara, annot=True, fmt=".2f", cmap="vlag", center=0, vmin=-1, vmax=1, square=True,
                linewidths=0.6, cbar_kws={"shrink": 0.75, "label": "ρ de Spearman"}, ax=ax)
    ax.set_title("4. Correlaciones (Spearman)", fontsize=14, fontweight="bold", loc="left")
    guardar(fig, salida / "4_correlaciones.png")


def fig_calor(df: pd.DataFrame, salida: Path) -> None:
    tabla = df.pivot_table(index="dia", columns="franja", values="a_tiempo", aggfunc="mean", observed=False)
    fig, ax = plt.subplots(figsize=(8, 5), layout="constrained")
    sns.heatmap(tabla, annot=True, fmt=".0%", cmap="RdYlGn", vmin=0.3, vmax=0.95, linewidths=0.5,
                cbar_kws={"format": "{x:.0%}", "label": "Entregas a tiempo"}, ax=ax)
    ax.set(xlabel="Franja horaria del pedido", ylabel=None)
    ax.set_title("5. Entregas a tiempo según día y hora del pedido", fontsize=14, fontweight="bold", loc="left")
    guardar(fig, salida / "5_mapa_calor.png")


def fig_valoracion(df: pd.DataFrame, salida: Path) -> None:
    df = df.assign(tramo_entrega=pd.cut(df["horas_entrega"], [0, 24, 48, 72, np.inf],
                                        labels=["< 24 h", "24-48 h", "48-72 h", "> 72 h"]))
    # Porcentaje de cada valoración dentro de cada canal y tramo (se calcula antes de dibujar)
    reparto = (df.groupby(["canal", "tramo_entrega"], observed=True)["valoracion"]
               .value_counts(normalize=True).rename("porcentaje").reset_index())
    g = sns.catplot(reparto, x="tramo_entrega", y="porcentaje", hue="valoracion", col="canal", kind="bar",
                    palette="RdYlGn", height=4, aspect=1.1, errorbar=None)
    for ax in g.axes.flat:
        ax.yaxis.set_major_formatter("{x:.0%}")
    g.set_titles("Canal: {col_name}")
    g.set_axis_labels("Tiempo de entrega", "% de pedidos del tramo")
    g.legend.set_title("Valoración")
    g.figure.suptitle("6. Valoración del cliente según el tiempo de entrega", fontsize=15, fontweight="bold",
                      x=0.01, ha="left", y=1.05)
    guardar(g, salida / "6_valoracion.png")


def resumen(df: pd.DataFrame, salida: Path) -> pd.DataFrame:
    tabla = (df.groupby("transportista")
             .agg(pedidos=("horas_entrega", "size"), mediana_horas=("horas_entrega", "median"),
                  p90_horas=("horas_entrega", lambda s: s.quantile(0.9)), a_tiempo=("a_tiempo", "mean"),
                  valoracion_media=("valoracion", "mean"))
             .round(2).sort_values("a_tiempo", ascending=False))
    tabla.to_csv(salida / "resumen_transportistas.csv", sep=";", decimal=",", encoding="utf-8-sig")
    return tabla


def main() -> int:
    p = argparse.ArgumentParser(description="Análisis visual de entregas con Seaborn.")
    p.add_argument("--salida", type=Path, default=Path("informe_entregas"))
    args = p.parse_args()
    args.salida.mkdir(exist_ok=True)

    sns.set_theme(style="whitegrid", context="notebook", palette="colorblind",
                  rc={"axes.spines.top": False, "axes.spines.right": False, "axes.titleweight": "bold"})
    df = generar_datos()
    for figura in (fig_distribuciones, fig_comparacion, fig_regresion, fig_correlaciones, fig_calor, fig_valoracion):
        figura(df, args.salida)
    print(resumen(df, args.salida).to_string(), "\n")
    print(f"Correlación tiempo–valoración (Spearman): "
          f"{df['horas_entrega'].corr(df['valoracion'], method='spearman'):.2f}")
    print(f"✔ 6 figuras y resumen CSV en {args.salida}/")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `URLError` / `HTTPError` en `load_dataset` | Sin conexión a Internet. | Use sus propios datos o descargue el CSV previamente. |
| `ValueError: Could not interpret value 'x' for parameter 'x'` | La columna no existe en el DataFrame. | Revise `df.columns` (mayúsculas, espacios). |
| `FutureWarning: Passing palette without assigning hue` | `palette=` sin `hue=` (0.13). | Añada `hue="misma_columna", legend=False`. |
| `TypeError: … got an unexpected keyword argument 'ax'` | Se pasó `ax=` a una función de figura (`catplot`, `relplot`…). | Use la función de ejes equivalente (`boxplot`, `scatterplot`…). |
| `ci=` muestra un aviso de obsolescencia | Sustituido por `errorbar=` (0.12+). | `errorbar=("ci", 95)` o `errorbar=None`. |
| `MatplotlibDeprecationWarning: vert: bool was deprecated` | Seaborn 0.13 con Matplotlib 3.11 o superior (aviso interno). | Es inofensivo; puede filtrarse con `warnings.filterwarnings`. |
| Las barras muestran la media y no la suma | `barplot` estima la media por defecto. | `estimator="sum"` o agregue antes y use `errorbar=None`. |
| Categorías en orden alfabético no deseado | Orden por defecto. | `order=[…]` o columna `Categorical` ordenada. |
| Leyenda tapando los datos | Posición automática. | `sns.move_legend(ax, "upper left", bbox_to_anchor=(1, 1))`. |
| `swarmplot` avisa de puntos que no caben | Demasiadas observaciones. | Use `stripplot`, `violinplot` o una muestra. |
| Gráficos lentos con millones de filas | KDE, *bootstrap* o puntos individuales. | `errorbar=None`, muestree, agregue antes o use `histplot`. |

---

## 18. Buenas prácticas

- Prepare los datos en formato largo y con tipos correctos (`Categorical` ordenadas para categorías con orden).
- Elija la familia de gráfico según la pregunta: distribución, relación, comparación entre categorías o regresión.
- Muestre las observaciones o la distribución además de las medias cuando haya pocos datos.
- Indique qué representan las barras de error y compare grupos de distinto tamaño con densidades normalizadas.
- Use paletas accesibles (`colorblind`, `viridis`) y un orden lógico de categorías.
- Configure el estilo una vez con `sns.set_theme(...)`.
- Use funciones de ejes para composiciones propias y funciones de figura para facetas rápidas.
- Ajuste los detalles finales (formatos, anotaciones, títulos) con Matplotlib.
- Complemente los gráficos con estadísticos calculados (medianas, percentiles, coeficientes).

---

## 19. Referencia rápida

```python
sns.set_theme(style="whitegrid", context="notebook", palette="colorblind") · sns.despine() · sns.color_palette("viridis", 5)
sns.scatterplot · lineplot · relplot(kind="scatter"|"line", col=, row=, hue=, size=, style=)
sns.histplot(x=, hue=, stat="density", common_norm=False, kde=True) · kdeplot · ecdfplot · displot(kind=)
sns.boxplot · violinplot · boxenplot · stripplot · swarmplot · barplot(errorbar=("ci", 95)) · pointplot · countplot · catplot(kind=)
sns.regplot · lmplot · residplot · heatmap(annot=True, fmt=, cmap=, center=) · clustermap · pairplot · jointplot
g = sns.catplot(...); g.set_titles("{col_name}"); g.set_axis_labels(); g.figure.suptitle(); g.savefig()
sns.move_legend(ax, "upper left", bbox_to_anchor=(1, 1)) · ax.bar_label(ax.containers[0])
import seaborn.objects as so; so.Plot(df, x=, y=).add(so.Dot()).add(so.Line(), so.PolyFit())
```

---

## 20. Ejercicios propuestos

1. Con un DataFrame de alumnos y notas, compare la distribución por asignatura con `violinplot` y superponga las observaciones con `stripplot`.
2. Cree un `pairplot` de las variables numéricas de un conjunto de datos de viviendas coloreado por barrio, solo con el triángulo inferior.
3. Represente la evolución mensual de las ventas por región con `relplot(kind="line")` en paneles con `col_wrap=3`.
4. Amplíe el ejemplo completo con una figura que compare el importe medio por canal con `barplot` mostrando la desviación típica y explique la diferencia con el IC del 95 %.
5. Reproduzca uno de los gráficos del ejemplo con la interfaz `seaborn.objects`.

---

## 21. Referencias

- Documentación oficial: https://seaborn.pydata.org/
- Tutorial: https://seaborn.pydata.org/tutorial.html
- Galería de ejemplos: https://seaborn.pydata.org/examples/index.html
- Barras de error: https://seaborn.pydata.org/tutorial/error_bars.html
- Interfaz de objetos: https://seaborn.pydata.org/tutorial/objects_interface.html
