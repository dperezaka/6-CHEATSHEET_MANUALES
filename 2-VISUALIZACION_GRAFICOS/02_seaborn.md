# Manual de uso: Seaborn

> **Categoría:** Visualización de datos · **Utilidad:** Gráficos estadísticos más elaborados
> **Tipo:** Librería externa (requiere instalación con `pip`). Construida sobre Matplotlib y pandas.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer gráfico](#4-primer-gráfico)
5. [Preparar los datos: formato largo](#5-preparar-los-datos-formato-largo)
6. [Gráficos relacionales](#6-gráficos-relacionales)
7. [Gráficos de distribución](#7-gráficos-de-distribución)
8. [Gráficos categóricos](#8-gráficos-categóricos)
9. [Gráficos de regresión](#9-gráficos-de-regresión)
10. [Matrices: mapas de calor y agrupamiento](#10-matrices-mapas-de-calor-y-agrupamiento)
11. [Gráficos multipanel](#11-gráficos-multipanel)
12. [Temas, contextos y paletas](#12-temas-contextos-y-paletas)
13. [Personalizar con Matplotlib](#13-personalizar-con-matplotlib)
14. [Interfaz de objetos (`seaborn.objects`)](#14-interfaz-de-objetos-seabornobjects)
15. [Guardar y exportar](#15-guardar-y-exportar)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Seaborn** es una librería de visualización estadística construida sobre **Matplotlib** y diseñada para trabajar directamente con **DataFrames de pandas**. Con una sola línea genera gráficos que en Matplotlib requerirían decenas: agrupa, calcula medias e intervalos de confianza, ajusta regresiones y asigna colores y leyendas automáticamente.

**Ventajas:**
- Sintaxis concisa: se indican columnas del DataFrame, no listas de valores.
- Cálculos estadísticos integrados (medias, intervalos de confianza, densidades, regresiones).
- Estética cuidada por defecto y paletas de color bien diseñadas.
- Gráficos multipanel (*facets*) por categorías con un solo parámetro.
- Totalmente compatible con Matplotlib: todo se puede ajustar con `ax` y `fig`.

**Limitaciones:**
- Gráficos estáticos (sin *tooltips* ni zoom interactivo).
- Menos control fino que Matplotlib para diseños muy personalizados (aunque se puede combinar).
- No está pensada para dashboards ni aplicaciones web.

| Si necesita… | Opción |
|---|---|
| Análisis exploratorio y gráficos estadísticos rápidos | **Seaborn** |
| Control absoluto de cada elemento | Matplotlib |
| Interactividad (zoom, *tooltips*, filtros) | Plotly / Bokeh / Altair |
| Paneles web | Dash / Streamlit |

---

## 2. Instalación e importación

```bash
pip install seaborn                 # Instala también matplotlib, pandas y numpy
pip install "seaborn[stats]"        # Añade scipy y statsmodels (regresiones avanzadas)
```

Comprobar la versión:

```bash
python -c "import seaborn as sns; print(sns.__version__)"
```

Importaciones habituales:

```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sns
```

> El alias `sns` es una convención universal. Siempre se importa también `matplotlib.pyplot` para mostrar, guardar y ajustar los gráficos.

Conjuntos de datos de ejemplo (requieren conexión a internet la primera vez):

```python
print(sns.get_dataset_names())          # 'tips', 'penguins', 'flights', 'titanic', 'iris'…
propinas = sns.load_dataset("tips")
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Datos en formato largo (*tidy*) | Una fila por observación y una columna por variable. Es el formato que espera Seaborn. |
| Variables semánticas | Parámetros que asignan columnas a propiedades visuales: `x`, `y`, `hue` (color), `size` (tamaño), `style` (marcador/línea). |
| Función de ejes (*axes-level*) | Dibuja en un único `Axes` de Matplotlib y lo devuelve. Ej.: `scatterplot`, `histplot`, `boxplot`. Acepta `ax=`. |
| Función de figura (*figure-level*) | Crea su propia figura, con posibilidad de varios paneles (`col`, `row`). Ej.: `relplot`, `displot`, `catplot`. Devuelve un objeto `FacetGrid`. |
| Estimador | Función estadística que resume los datos (por defecto, la media) en `barplot`, `pointplot`, `lineplot`. |
| Intervalo de error | Banda o barra de incertidumbre (`errorbar="ci"`, `"sd"`, `"se"`, `"pi"`). |
| Tema | Conjunto de estilo, contexto (tamaño) y paleta aplicados globalmente con `sns.set_theme()`. |

**Familias de funciones:**

| Familia | Función de figura | Funciones de ejes |
|---|---|---|
| Relacional | `relplot` | `scatterplot`, `lineplot` |
| Distribución | `displot` | `histplot`, `kdeplot`, `ecdfplot`, `rugplot` |
| Categórica | `catplot` | `stripplot`, `swarmplot`, `boxplot`, `violinplot`, `boxenplot`, `barplot`, `pointplot`, `countplot` |
| Regresión | `lmplot` | `regplot`, `residplot` |
| Matriz | — | `heatmap`, `clustermap` (figura propia) |

> Regla práctica: use funciones **de ejes** cuando vaya a componer el gráfico dentro de una figura propia (`fig, ax = plt.subplots()`), y funciones **de figura** cuando quiera dividir automáticamente en paneles por categoría.

---

## 4. Primer gráfico

```python
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns

sns.set_theme(style="whitegrid")

ventas = pd.DataFrame({
    "mes": ["Ene", "Feb", "Mar", "Abr"] * 2,
    "canal": ["Online"] * 4 + ["Tienda"] * 4,
    "importe": [50, 60, 55, 70, 70, 75, 73, 90],
})

fig, ax = plt.subplots(figsize=(8, 4))
sns.barplot(data=ventas, x="mes", y="importe", hue="canal", ax=ax)
ax.set_title("Ventas por canal")
ax.set_ylabel("Miles de €")
plt.show()
```

Con una sola línea Seaborn ha agrupado por mes y canal, asignado colores y creado la leyenda.

---

## 5. Preparar los datos: formato largo

Seaborn funciona mejor con datos **largos**. Si los datos están en formato **ancho** (una columna por serie), conviértalos con `melt`:

```python
ancho = pd.DataFrame({"mes": ["Ene", "Feb", "Mar"],
                      "Online": [50, 60, 55],
                      "Tienda": [70, 75, 73]})

largo = ancho.melt(id_vars="mes", var_name="canal", value_name="importe")
#    mes   canal  importe
# 0  Ene  Online       50
# 1  Feb  Online       60
# ...
```

| Formato | Aspecto | Uso en Seaborn |
|---|---|---|
| Largo | `mes · canal · importe` | Recomendado: permite `hue="canal"`. |
| Ancho | `mes · Online · Tienda` | Funciona en algunos gráficos (`sns.lineplot(data=ancho)`), con menos control. |

Operación inversa (largo → ancho), útil para mapas de calor:

```python
tabla = largo.pivot_table(index="canal", columns="mes", values="importe", aggfunc="sum")
```

---

## 6. Gráficos relacionales

### 6.1 Dispersión (`scatterplot`)

```python
sns.scatterplot(data=df, x="precio", y="unidades",
                hue="categoria",              # Color por categoría
                size="margen",                # Tamaño según valor
                style="canal",                # Forma del marcador
                sizes=(20, 200), alpha=0.7, ax=ax)
```

### 6.2 Líneas (`lineplot`)

```python
sns.lineplot(data=df, x="fecha", y="importe", hue="region", marker="o", ax=ax)
```

Si hay varias observaciones por valor de `x`, `lineplot` dibuja **la media y una banda de confianza del 95 %**:

```python
sns.lineplot(data=df, x="mes", y="importe", errorbar="sd")        # Desviación típica
sns.lineplot(data=df, x="mes", y="importe", errorbar=None)        # Sin banda
sns.lineplot(data=df, x="mes", y="importe", estimator="sum")      # Suma en vez de media
```

| `errorbar` | Significado |
|---|---|
| `("ci", 95)` | Intervalo de confianza del 95 % (por *bootstrap*). Por defecto. |
| `"sd"` | ± una desviación típica. |
| `"se"` | ± un error estándar. |
| `("pi", 50)` | Intervalo de percentiles (rango central del 50 %). |
| `None` | Sin barra ni banda. |

### 6.3 Varios paneles (`relplot`)

```python
g = sns.relplot(data=df, x="precio", y="unidades", hue="canal",
                col="region", col_wrap=2,          # Un panel por región, 2 por fila
                kind="scatter", height=3.5, aspect=1.2)
g.set_axis_labels("Precio (€)", "Unidades")
g.set_titles("Región: {col_name}")
```

---

## 7. Gráficos de distribución

### 7.1 Histograma (`histplot`)

```python
sns.histplot(data=df, x="importe", bins=30, kde=True, ax=ax)                    # Con curva de densidad
sns.histplot(data=df, x="importe", hue="canal", multiple="stack", ax=ax)        # Apilado
sns.histplot(data=df, x="importe", hue="canal", element="step", stat="density", common_norm=False)
```

| Parámetro | Opciones |
|---|---|
| `stat` | `"count"`, `"frequency"`, `"probability"`, `"percent"`, `"density"` |
| `multiple` | `"layer"`, `"stack"`, `"dodge"`, `"fill"` |
| `element` | `"bars"`, `"step"`, `"poly"` |
| `common_norm` | `False` normaliza cada grupo por separado (para comparar formas). |
| `log_scale` | `True` para datos muy asimétricos. |

### 7.2 Densidad (`kdeplot`)

```python
sns.kdeplot(data=df, x="importe", hue="canal", fill=True, alpha=0.4, ax=ax)
sns.kdeplot(data=df, x="precio", y="unidades", fill=True, cmap="Blues", ax=ax)   # Bivariante
```

### 7.3 Distribución acumulada (`ecdfplot`)

```python
sns.ecdfplot(data=df, x="importe", hue="canal", ax=ax)
```

> La ECDF muestra qué proporción de valores está por debajo de cada umbral. Es más fiable que el histograma para comparar grupos porque no depende del número de intervalos.

### 7.4 Varios paneles (`displot`)

```python
sns.displot(data=df, x="importe", col="region", kind="hist", kde=True, height=3)
sns.displot(data=df, x="precio", y="unidades", kind="kde")
```

### 7.5 Conjunta y marginal (`jointplot`)

```python
sns.jointplot(data=df, x="precio", y="unidades", hue="canal", kind="scatter")
sns.jointplot(data=df, x="precio", y="unidades", kind="hex")          # Hexágonos (muchos puntos)
sns.jointplot(data=df, x="precio", y="unidades", kind="reg")          # Con regresión
```

---

## 8. Gráficos categóricos

### 8.1 Observaciones individuales

```python
sns.stripplot(data=df, x="region", y="importe", hue="canal", dodge=True, alpha=0.5, ax=ax)
sns.swarmplot(data=df, x="region", y="importe", size=3, ax=ax)        # Sin solapamiento (< 1000 puntos)
```

### 8.2 Distribución por categoría

```python
sns.boxplot(data=df, x="region", y="importe", hue="canal", ax=ax)
sns.violinplot(data=df, x="region", y="importe", hue="canal", split=True, inner="quart", ax=ax)
sns.boxenplot(data=df, x="region", y="importe", ax=ax)          # Para muestras grandes
```

Combinación habitual (caja + puntos):

```python
sns.boxplot(data=df, x="region", y="importe", color="white", showfliers=False, ax=ax)
sns.stripplot(data=df, x="region", y="importe", alpha=0.3, size=3, ax=ax)
```

### 8.3 Estimaciones (`barplot`, `pointplot`, `countplot`)

```python
sns.barplot(data=df, x="region", y="importe", estimator="sum", errorbar=None, ax=ax)
sns.pointplot(data=df, x="mes", y="importe", hue="canal", dodge=0.3, ax=ax)
sns.countplot(data=df, x="region", order=df["region"].value_counts().index, ax=ax)
```

> `barplot` muestra por defecto la **media** con su intervalo de confianza, no la suma. Use `estimator="sum"` si necesita totales.

### 8.4 Orden y orientación

```python
orden = ["Norte", "Sur", "Este", "Oeste"]
sns.boxplot(data=df, x="region", y="importe", order=orden)
sns.barplot(data=df, x="importe", y="region")       # Horizontal: la variable numérica en x
```

### 8.5 Varios paneles (`catplot`)

```python
g = sns.catplot(data=df, x="region", y="importe", hue="canal", col="trimestre",
                kind="bar", height=3.5, aspect=0.9, errorbar=None)
g.set_xticklabels(rotation=30)
```

---

## 9. Gráficos de regresión

```python
sns.regplot(data=df, x="precio", y="unidades", ax=ax)                          # Lineal con IC
sns.regplot(data=df, x="precio", y="unidades", order=2, ax=ax)                 # Polinómica
sns.regplot(data=df, x="precio", y="unidades", lowess=True, ax=ax)             # Suavizado (requiere statsmodels)
sns.regplot(data=df, x="precio", y="unidades", logistic=True, ax=ax)           # Logística (y binaria)
sns.regplot(data=df, x="precio", y="unidades", scatter_kws={"alpha": 0.3}, line_kws={"color": "red"})

sns.lmplot(data=df, x="precio", y="unidades", hue="canal", col="region", height=3)
sns.residplot(data=df, x="precio", y="unidades", ax=ax)                        # Residuos
```

> Seaborn **dibuja** la regresión, pero no devuelve los coeficientes. Para obtenerlos use `numpy.polyfit`, `scipy.stats.linregress` o `statsmodels`.

---

## 10. Matrices: mapas de calor y agrupamiento

### 10.1 Mapa de calor (`heatmap`)

```python
tabla = df.pivot_table(index="dia_semana", columns="mes", values="importe", aggfunc="sum")

sns.heatmap(tabla, annot=True, fmt=".0f", cmap="Blues", linewidths=0.5,
            cbar_kws={"label": "Importe (€)"}, ax=ax)
```

Matriz de correlación con la mitad superior oculta:

```python
corr = df.select_dtypes("number").corr()
mascara = np.triu(np.ones_like(corr, dtype=bool))
sns.heatmap(corr, mask=mascara, annot=True, fmt=".2f", cmap="RdBu_r",
            vmin=-1, vmax=1, center=0, square=True, ax=ax)
```

| Parámetro | Efecto |
|---|---|
| `annot=True` | Muestra el valor en cada celda. |
| `fmt` | Formato del valor (`".0f"`, `".1%"`, `"d"`). |
| `center` | Valor central en paletas divergentes. |
| `vmin`, `vmax` | Rango fijo de la escala de color. |
| `mask` | Oculta celdas (matriz booleana). |

### 10.2 Mapa de calor con agrupamiento (`clustermap`)

```python
sns.clustermap(tabla, cmap="vlag", standard_scale=1, figsize=(8, 6))   # Requiere scipy
```

Reordena filas y columnas por similitud y añade dendrogramas: útil para detectar grupos de clientes o productos parecidos.

---

## 11. Gráficos multipanel

### 11.1 `FacetGrid`: el mismo gráfico por subgrupos

```python
g = sns.FacetGrid(df, col="region", row="canal", height=2.8, aspect=1.3, margin_titles=True)
g.map_dataframe(sns.histplot, x="importe", bins=20)
g.set_axis_labels("Importe (€)", "Tickets")
g.set_titles(col_template="{col_name}", row_template="{row_name}")
g.add_legend()
g.figure.suptitle("Distribución por región y canal", y=1.03)
```

El objeto devuelto por `relplot`, `displot`, `catplot` y `lmplot` es también un `FacetGrid`:

```python
g = sns.catplot(...)
for ax in g.axes.flat:                       # Recorrer los paneles
    ax.axhline(100, ls="--", color="gray")
g.refline(y=100)                             # Atajo para líneas de referencia
```

### 11.2 `pairplot` y `PairGrid`: todas las variables contra todas

```python
sns.pairplot(df, vars=["precio", "unidades", "margen"], hue="canal", corner=True, diag_kind="kde")

g = sns.PairGrid(df, vars=["precio", "unidades", "margen"], hue="canal")
g.map_upper(sns.scatterplot, alpha=0.5)
g.map_lower(sns.kdeplot)
g.map_diag(sns.histplot)
g.add_legend()
```

### 11.3 Combinar con `plt.subplots`

```python
fig, axs = plt.subplots(1, 3, figsize=(15, 4), layout="constrained")
sns.histplot(data=df, x="importe", ax=axs[0])
sns.boxplot(data=df, x="region", y="importe", ax=axs[1])
sns.scatterplot(data=df, x="precio", y="unidades", ax=axs[2])
```

> Las funciones de figura (`relplot`, `catplot`…) **no** aceptan `ax=`: crean su propia figura. Para componerlas en una figura propia, use sus equivalentes de ejes.

---

## 12. Temas, contextos y paletas

### 12.1 Tema global

```python
sns.set_theme(style="whitegrid", context="notebook", palette="deep", font_scale=1.1)
```

| Parámetro | Opciones |
|---|---|
| `style` | `"darkgrid"` (defecto), `"whitegrid"`, `"dark"`, `"white"`, `"ticks"` |
| `context` | `"paper"`, `"notebook"` (defecto), `"talk"`, `"poster"`: escala textos y líneas |
| `palette` | Nombre de paleta o lista de colores |
| `rc` | Diccionario de `rcParams` de Matplotlib |

```python
sns.set_theme(style="ticks", rc={"axes.spines.top": False, "axes.spines.right": False})
sns.despine()                                     # Quitar bordes superior y derecho del gráfico actual
sns.despine(left=True, bottom=True)
with sns.axes_style("white"):                     # Estilo temporal
    ...
```

### 12.2 Paletas

```python
sns.color_palette("deep")                         # Cualitativa (por defecto)
sns.color_palette("Blues", n_colors=5)            # Secuencial
sns.color_palette("vlag", as_cmap=True)           # Divergente como colormap
sns.color_palette("colorblind")                   # Apta para daltonismo
sns.color_palette(["#1F4E78", "#2E75B6", "#9DC3E6"])     # Personalizada
sns.light_palette("#1F4E78", n_colors=6)
sns.blend_palette(["#C00000", "white", "#1F4E78"], as_cmap=True)

sns.palplot(sns.color_palette("Set2"))            # Visualizar una paleta
```

| Tipo de dato | Paletas recomendadas |
|---|---|
| Categorías sin orden | `deep`, `muted`, `colorblind`, `Set2`, `tab10` |
| Valores ordenados | `rocket`, `mako`, `Blues`, `viridis`, `crest` |
| Valores alrededor de un centro | `vlag`, `icefire`, `RdBu_r`, `coolwarm` |

Colores fijos por categoría (coherentes entre gráficos):

```python
colores = {"Online": "#2E75B6", "Tienda": "#C00000"}
sns.barplot(data=df, x="region", y="importe", hue="canal", palette=colores)
```

---

## 13. Personalizar con Matplotlib

Las funciones de ejes devuelven un `Axes`; las de figura, un `FacetGrid` con `.figure` y `.axes`. A partir de ahí se usa todo Matplotlib:

```python
ax = sns.barplot(data=df, x="region", y="importe", errorbar=None)
ax.set(title="Ventas por región", xlabel="", ylabel="Importe (€)")
ax.bar_label(ax.containers[0], fmt="%.0f", padding=3)              # Valores sobre las barras
ax.yaxis.set_major_formatter(lambda v, _: f"{v:,.0f} €".replace(",", "."))
ax.tick_params(axis="x", rotation=30)
```

Mover la leyenda (`sns.move_legend`):

```python
sns.move_legend(ax, "upper left", bbox_to_anchor=(1, 1), title="Canal", frameon=False)
sns.move_legend(ax, "lower center", bbox_to_anchor=(0.5, 1), ncol=3, title=None)
```

Títulos y ejes en un `FacetGrid`:

```python
g.set(ylim=(0, None))
g.set_axis_labels("Mes", "Importe (€)")
g.figure.suptitle("Título general", y=1.02, fontweight="bold")
g.figure.subplots_adjust(top=0.88)
```

---

## 14. Interfaz de objetos (`seaborn.objects`)

Desde la versión 0.12, Seaborn incluye una interfaz declarativa, inspirada en la «gramática de gráficos», que combina **datos + marcas + transformaciones** de forma modular:

```python
import seaborn.objects as so

(
    so.Plot(df, x="mes", y="importe", color="canal")
    .add(so.Line(), so.Agg("sum"))                     # Línea con la suma por mes
    .add(so.Dot(), so.Agg("sum"))
    .facet(col="region", wrap=2)
    .label(title="{}", x="Mes", y="Importe (€)", color="Canal")
    .layout(size=(10, 6))
    .save("ventas.png", dpi=150, bbox_inches="tight")
)
```

| Elemento | Ejemplos |
|---|---|
| Marcas | `so.Dot`, `so.Line`, `so.Bar`, `so.Bars`, `so.Area`, `so.Band`, `so.Range`, `so.Text` |
| Transformaciones | `so.Agg`, `so.Est`, `so.Hist`, `so.KDE`, `so.PolyFit`, `so.Perc` |
| Ajustes de posición | `so.Dodge`, `so.Stack`, `so.Jitter`, `so.Norm` |

```python
so.Plot(df, x="importe", color="canal").add(so.Bars(), so.Hist(bins=30), so.Stack())
so.Plot(df, x="precio", y="unidades").add(so.Dot(alpha=0.3)).add(so.Line(color="red"), so.PolyFit(order=1))
```

> La interfaz de objetos es moderna y muy flexible, pero aún se considera en evolución. Para código de producción estable, las funciones clásicas siguen siendo la opción más documentada.

---

## 15. Guardar y exportar

```python
fig.savefig("grafico.png", dpi=200, bbox_inches="tight")       # Funciones de ejes
g = sns.catplot(...); g.savefig("panel.pdf")                    # Funciones de figura (FacetGrid)
sns.clustermap(...).savefig("clusters.png", dpi=150)
```

| Formato | Uso recomendado |
|---|---|
| PNG (`dpi` 150–300) | Presentaciones, web, correo |
| PDF / SVG | Informes e impresión (vectorial) |

En servidores sin pantalla, active el backend `Agg` antes de importar `pyplot` (véase el manual de Matplotlib).

---

## 16. Ejemplo práctico completo

**Análisis exploratorio de clientes de una tienda online**: genera un conjunto de datos realista de pedidos, aplica un tema corporativo y produce un informe de seis gráficos (distribución del importe por canal, comparación por región, relación descuento-importe con regresión, evolución mensual con banda de confianza, mapa de calor día-hora y correlaciones), además de un panel por segmento con `FacetGrid`. Exporta todo a PNG y un resumen estadístico a CSV.

```python
"""
Análisis exploratorio de pedidos con Seaborn.
Requiere: pip install seaborn
"""
from pathlib import Path

import matplotlib

matplotlib.use("Agg")
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import numpy as np
import pandas as pd
import seaborn as sns

SALIDA = Path(__file__).resolve().parent / "salida_seaborn"
COLORES_CANAL = {"Web": "#2E75B6", "App": "#1F4E78", "Marketplace": "#C55A11"}
ORDEN_REGIONES = ["Norte", "Centro", "Sur", "Este"]
DIAS = ["Lun", "Mar", "Mié", "Jue", "Vie", "Sáb", "Dom"]


# ------------------------------------------------------------------ Datos
def generar_pedidos(n: int = 6_000, semilla: int = 11) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    canal = rng.choice(list(COLORES_CANAL), n, p=[0.5, 0.35, 0.15])
    region = rng.choice(ORDEN_REGIONES, n, p=[0.3, 0.35, 0.2, 0.15])
    segmento = rng.choice(["Nuevo", "Recurrente", "Premium"], n, p=[0.45, 0.4, 0.15])
    fecha = pd.Timestamp("2025-01-01") + pd.to_timedelta(rng.integers(0, 365, n), unit="D")
    hora = (rng.normal(18, 3.5, n).round() % 24).astype(int)

    base = rng.lognormal(3.9, 0.55, n)
    base *= np.select([segmento == "Premium", segmento == "Recurrente"], [1.8, 1.2], 1.0)
    base *= np.where(canal == "App", 1.1, 1.0)
    descuento = np.clip(rng.gamma(2, 4, n), 0, 40).round()
    importe = base * (1 + descuento / 60) + rng.normal(0, 5, n)
    articulos = np.maximum(1, (importe / 25 + rng.normal(0, 1, n)).round()).astype(int)

    return pd.DataFrame({
        "fecha": fecha, "mes": fecha.month, "dia_semana": fecha.dayofweek, "hora": hora,
        "canal": canal, "region": region, "segmento": segmento,
        "descuento_pct": descuento, "articulos": articulos,
        "importe": np.maximum(importe, 5).round(2),
    })


# ------------------------------------------------------------------ Estilo
def configurar_tema() -> None:
    sns.set_theme(style="whitegrid", context="notebook", font_scale=0.95,
                  rc={"axes.spines.top": False, "axes.spines.right": False,
                      "axes.titleweight": "bold", "axes.titlelocation": "left",
                      "grid.alpha": 0.4, "savefig.dpi": 180, "savefig.bbox": "tight"})


def euros(valor: float, _=None) -> str:
    return f"{valor:,.0f} €".replace(",", ".")


# ------------------------------------------------------------------ Gráficos individuales
def distribucion_por_canal(ax: plt.Axes, df: pd.DataFrame) -> None:
    sns.kdeplot(data=df, x="importe", hue="canal", palette=COLORES_CANAL, fill=True,
                common_norm=False, alpha=0.25, linewidth=1.5, clip=(0, None), ax=ax)
    ax.set_xlim(0, df["importe"].quantile(0.98))
    ax.xaxis.set_major_formatter(mticker.FuncFormatter(euros))
    ax.set(title="Distribución del importe por canal", xlabel="Importe del pedido", ylabel="Densidad")
    sns.move_legend(ax, "upper right", title=None, frameon=False)


def comparacion_regiones(ax: plt.Axes, df: pd.DataFrame) -> None:
    sns.boxplot(data=df, x="region", y="importe", hue="segmento", order=ORDEN_REGIONES,
                hue_order=["Nuevo", "Recurrente", "Premium"], palette="Blues",
                showfliers=False, linewidth=1, ax=ax)
    ax.yaxis.set_major_formatter(mticker.FuncFormatter(euros))
    ax.set(title="Importe por región y segmento", xlabel="", ylabel="Importe del pedido")
    sns.move_legend(ax, "upper left", ncol=3, title=None, frameon=False)


def descuento_vs_importe(ax: plt.Axes, df: pd.DataFrame) -> None:
    muestra = df.sample(1_500, random_state=1)
    sns.regplot(data=muestra, x="descuento_pct", y="importe", x_jitter=0.4,
                scatter_kws={"alpha": 0.2, "s": 12, "color": "#7F7F7F"},
                line_kws={"color": "#C00000", "linewidth": 2}, ax=ax)
    pendiente, origen = np.polyfit(df["descuento_pct"], df["importe"], 1)
    ax.text(0.98, 0.95, f"Pendiente: {pendiente:+.2f} € por punto de descuento",
            transform=ax.transAxes, ha="right", va="top", fontsize=9,
            bbox={"facecolor": "white", "edgecolor": "#D9D9D9", "boxstyle": "round"})
    ax.set_ylim(0, df["importe"].quantile(0.98))
    ax.yaxis.set_major_formatter(mticker.FuncFormatter(euros))
    ax.set(title="Relación entre descuento e importe", xlabel="Descuento aplicado (%)", ylabel="Importe")


def evolucion_mensual(ax: plt.Axes, df: pd.DataFrame) -> None:
    sns.lineplot(data=df, x="mes", y="importe", hue="canal", palette=COLORES_CANAL,
                 errorbar="se", marker="o", markersize=5, ax=ax)
    ax.set_xticks(range(1, 13), ["E", "F", "M", "A", "M", "J", "J", "A", "S", "O", "N", "D"])
    ax.yaxis.set_major_formatter(mticker.FuncFormatter(euros))
    ax.set(title="Ticket medio mensual (± error estándar)", xlabel="", ylabel="Ticket medio")
    sns.move_legend(ax, "lower center", bbox_to_anchor=(0.5, -0.32), ncol=3, title=None, frameon=False)


def calor_dia_hora(ax: plt.Axes, df: pd.DataFrame) -> None:
    tabla = (df[df["hora"].between(8, 23)]
             .pivot_table(index="dia_semana", columns="hora", values="importe", aggfunc="count", fill_value=0)
             .reindex(range(7), fill_value=0))
    sns.heatmap(tabla, cmap="mako_r", linewidths=0.4, linecolor="white",
                cbar_kws={"label": "Nº de pedidos", "shrink": 0.8}, ax=ax)
    ax.set_yticklabels(DIAS, rotation=0)
    ax.set(title="Pedidos por día y hora", xlabel="Hora", ylabel="")


def correlaciones(ax: plt.Axes, df: pd.DataFrame) -> None:
    corr = df[["importe", "articulos", "descuento_pct", "hora"]].corr()
    corr.index = corr.columns = ["Importe", "Artículos", "Descuento", "Hora"]
    mascara = np.triu(np.ones_like(corr, dtype=bool), k=1)
    sns.heatmap(corr, mask=mascara, annot=True, fmt=".2f", cmap="vlag", vmin=-1, vmax=1,
                center=0, square=True, cbar=False, linewidths=1, ax=ax)
    ax.set_title("Correlaciones")
    ax.tick_params(axis="x", rotation=30)


# ------------------------------------------------------------------ Informes
def informe_general(df: pd.DataFrame) -> Path:
    fig, axs = plt.subplot_mosaic([["dist", "dist", "region", "region"],
                                   ["desc", "desc", "evol", "evol"],
                                   ["calor", "calor", "corr", "corr"]],
                                  figsize=(15, 14), layout="constrained",
                                  gridspec_kw={"height_ratios": [1, 1, 0.9]})
    distribucion_por_canal(axs["dist"], df)
    comparacion_regiones(axs["region"], df)
    descuento_vs_importe(axs["desc"], df)
    evolucion_mensual(axs["evol"], df)
    calor_dia_hora(axs["calor"], df)
    correlaciones(axs["corr"], df)

    total, pedidos = df["importe"].sum(), len(df)
    fig.suptitle(f"Análisis de pedidos 2025 · {pedidos:,} pedidos · {euros(total)}".replace(",", "."),
                 x=0.01, ha="left", fontsize=17, fontweight="bold")
    ruta = SALIDA / "analisis_pedidos.png"
    fig.savefig(ruta)
    plt.close(fig)
    return ruta


def panel_por_segmento(df: pd.DataFrame) -> Path:
    g = sns.displot(data=df, x="importe", hue="canal", col="segmento", row="region",
                    col_order=["Nuevo", "Recurrente", "Premium"], row_order=ORDEN_REGIONES,
                    palette=COLORES_CANAL, kind="hist", stat="percent", common_norm=False,
                    element="step", bins=25, height=2.3, aspect=1.4, facet_kws={"margin_titles": True})
    g.set(xlim=(0, df["importe"].quantile(0.97)))
    g.set_axis_labels("Importe (€)", "% de pedidos")
    g.set_titles(col_template="{col_name}", row_template="{row_name}")
    g.refline(x=df["importe"].median(), color="#7F7F7F", linestyle=":")
    g.figure.suptitle("Distribución del importe por región, segmento y canal", y=1.02, fontweight="bold")
    ruta = SALIDA / "panel_segmentos.png"
    g.savefig(ruta)
    plt.close(g.figure)
    return ruta


def resumen_estadistico(df: pd.DataFrame) -> Path:
    resumen = (df.groupby(["region", "canal"], observed=True)["importe"]
               .agg(pedidos="count", total="sum", media="mean", mediana="median", desviacion="std")
               .round(2).reset_index())
    ruta = SALIDA / "resumen.csv"
    resumen.to_csv(ruta, index=False, sep=";", decimal=",", encoding="utf-8-sig")
    return ruta


def main() -> None:
    configurar_tema()
    SALIDA.mkdir(exist_ok=True)
    df = generar_pedidos()
    for ruta in (informe_general(df), panel_por_segmento(df), resumen_estadistico(df)):
        print(f"Generado: {ruta.name}")


if __name__ == "__main__":
    main()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ValueError: Could not interpret value 'x' for parameter 'x'` | El nombre de columna no existe (mayúsculas, espacios, tildes). | Revise `df.columns` y escriba el nombre exacto. |
| `TypeError: ... got an unexpected keyword argument 'ax'` | Se pasó `ax=` a una función de figura (`relplot`, `catplot`…). | Use la función de ejes equivalente (`scatterplot`, `barplot`…). |
| `URLError` al usar `load_dataset` | Sin conexión a internet. | Cargue los datos desde un archivo local. |
| Las barras no muestran el total esperado | `barplot` usa la **media** por defecto. | `estimator="sum"`. |
| Barras de error inesperadas | Intervalo de confianza por defecto. | `errorbar=None`. |
| Gráfico muy lento con muchos datos | El *bootstrap* del intervalo de confianza es costoso. | `errorbar=None` o `errorbar="se"`; use una muestra (`df.sample`). |
| `swarmplot`: *UserWarning: x% of the points cannot be placed* | Demasiados puntos para un enjambre. | Use `stripplot` con `alpha` o `violinplot`. |
| `FutureWarning: Passing palette without assigning hue` | Se pasó `palette` sin `hue`. | Añada `hue=` con la misma variable que `x` y `legend=False`. |
| La leyenda tapa los datos | Posición automática. | `sns.move_legend(ax, "upper left", bbox_to_anchor=(1, 1))`. |
| El título del `FacetGrid` se superpone a los paneles | `suptitle` sin margen. | `g.figure.suptitle(..., y=1.03)`. |
| El orden de las categorías es alfabético o aleatorio | Seaborn usa el orden de aparición. | `order=[...]`, `hue_order=[...]` o una columna `Categorical` ordenada. |
| Los colores cambian entre gráficos | El ciclo de paleta depende de los datos presentes. | Pase un diccionario `palette={"A": "#...", ...}`. |
| `heatmap` muestra números con muchos decimales | Formato por defecto. | `annot=True, fmt=".1f"`. |
| `clustermap` / `lowess` fallan | Faltan `scipy` o `statsmodels`. | `pip install "seaborn[stats]"`. |

---

## 18. Buenas prácticas

- Prepare los datos en **formato largo** antes de graficar: la mayor parte del trabajo con Seaborn es manipular el DataFrame.
- Defina el tema **una sola vez** al principio (`sns.set_theme(...)`).
- Use **funciones de ejes** para componer informes y **funciones de figura** para explorar por subgrupos.
- Fije el orden de las categorías (`order`, `hue_order`) y los colores (`palette` como diccionario) para que todos los gráficos sean coherentes.
- Indique qué representa la barra de error (`errorbar=`) o desactívela; no deje la configuración por defecto sin explicarla.
- Con más de unos miles de puntos, use `alpha`, `kind="hex"`, `kdeplot` o una muestra para evitar la saturación.
- Para comparar distribuciones de grupos de distinto tamaño, normalice (`stat="percent"`, `common_norm=False`).
- Use paletas aptas para daltonismo (`colorblind`, `mako`, `rocket`, `vlag`).
- Complete los gráficos con Matplotlib: títulos explicativos, unidades, formato de números y fuente de los datos.
- Calcule y reporte los estadísticos aparte (pandas, SciPy); el gráfico los ilustra pero no los sustituye.

---

## 19. Referencia rápida

```python
sns.set_theme(style="whitegrid", context="notebook", palette="deep")

# Relacionales            → relplot(kind="scatter"|"line")
sns.scatterplot(data=df, x=, y=, hue=, size=, style=, ax=)
sns.lineplot(data=df, x=, y=, hue=, errorbar=, estimator=, ax=)
# Distribución            → displot(kind="hist"|"kde"|"ecdf")
sns.histplot(data=df, x=, hue=, bins=, kde=, stat=, multiple=) / kdeplot / ecdfplot / jointplot
# Categóricos             → catplot(kind="strip"|"box"|"violin"|"bar"|"point"|"count")
sns.boxplot / violinplot / stripplot / swarmplot / barplot(estimator="sum") / pointplot / countplot
# Regresión               → lmplot
sns.regplot(data=df, x=, y=, order=, lowess=, logistic=) / residplot
# Matrices
sns.heatmap(tabla, annot=True, fmt=".1f", cmap=, center=) / sns.clustermap(tabla)
# Multipanel
sns.FacetGrid(df, col=, row=).map_dataframe(f, x=) / sns.pairplot(df, hue=) / sns.PairGrid

sns.move_legend(ax, "upper left", bbox_to_anchor=(1, 1)) / sns.despine() / sns.color_palette("mako", 5)
g.set_axis_labels(x, y) / g.set_titles("{col_name}") / g.refline(y=) / g.savefig("g.png")
df.melt(id_vars=, var_name=, value_name=)  /  df.pivot_table(index=, columns=, values=, aggfunc=)
```

---

## 20. Ejercicios propuestos

1. Con un conjunto de datos de empleados (departamento, antigüedad, salario), compare la distribución salarial por departamento con `violinplot` y `stripplot` superpuestos.
2. Transforme una tabla ancha de ventas mensuales por producto a formato largo y represéntela con `lineplot`, incluyendo una banda de desviación típica.
3. Cree un `pairplot` de cuatro variables numéricas coloreado por categoría y comente las relaciones más fuertes.
4. Construya una matriz de correlación con máscara triangular y paleta divergente centrada en cero.
5. Amplíe el ejemplo completo con un análisis de cohortes: mapa de calor del porcentaje de clientes que repiten compra por mes de alta y mes de actividad.

---

## 21. Referencias

- Documentación oficial: https://seaborn.pydata.org/
- Tutorial oficial: https://seaborn.pydata.org/tutorial.html
- Galería de ejemplos: https://seaborn.pydata.org/examples/index.html
- Referencia de la API: https://seaborn.pydata.org/api.html
- Paletas de color: https://seaborn.pydata.org/tutorial/color_palettes.html
