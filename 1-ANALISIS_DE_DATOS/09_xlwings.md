# Manual de uso: xlwings

> **Categoría:** Ofimática · **Utilidad:** Automatización de Excel desde Python (y Python desde Excel)
> **Tipo:** Librería de terceros (`pip install xlwings`) · **Versión de referencia:** 0.30 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y requisitos](#2-instalación-y-requisitos)
3. [Modelo de objetos](#3-modelo-de-objetos)
4. [Aplicaciones y libros](#4-aplicaciones-y-libros)
5. [Hojas](#5-hojas)
6. [Rangos: leer y escribir](#6-rangos-leer-y-escribir)
7. [Conversores: pandas, NumPy, fechas y dimensiones](#7-conversores-pandas-numpy-fechas-y-dimensiones)
8. [Formato](#8-formato)
9. [Tablas de Excel, nombres y fórmulas](#9-tablas-de-excel-nombres-y-fórmulas)
10. [Gráficos e imágenes](#10-gráficos-e-imágenes)
11. [Rendimiento](#11-rendimiento)
12. [Llamar a Python desde Excel (RunPython)](#12-llamar-a-python-desde-excel-runpython)
13. [Funciones definidas por el usuario (UDF)](#13-funciones-definidas-por-el-usuario-udf)
14. [Acceso a la API nativa de Excel (`.api`)](#14-acceso-a-la-api-nativa-de-excel-api)
15. [Macros VBA desde Python](#15-macros-vba-desde-python)
16. [Exportar a PDF](#16-exportar-a-pdf)
17. [xlwings Lite, Server y Reader](#17-xlwings-lite-server-y-reader)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**xlwings** controla **Microsoft Excel en ejecución** desde Python: abre libros, lee y escribe rangos, aplica formato, crea gráficos, ejecuta macros y recalcula, exactamente como lo haría un usuario o VBA. También funciona en sentido inverso: permite **llamar a Python desde Excel** (botones, macros) y crear **funciones de hoja** escritas en Python.

**Ventajas:**
- Trabaja con Excel «real»: se calculan las fórmulas, se conservan macros, tablas dinámicas, segmentaciones y cualquier elemento del libro.
- Integración natural con pandas y NumPy (escribe un DataFrame con una línea).
- Sustituto moderno de VBA: Python para la lógica, Excel como interfaz.
- Ve los cambios en directo mientras desarrolla.

**Limitaciones:**
- **Requiere Excel instalado** (Windows o macOS); no funciona en Linux ni en servidores sin Excel (salvo con xlwings Server/Lite).
- Más lento que OpenPyXL para generar archivos masivos sin interfaz.
- Microsoft desaconseja automatizar Office en servicios desatendidos de servidor.
- Algunas funciones (Reader, ciertos módulos avanzados) requieren licencia de pago.

| Si necesita… | Opción |
|---|---|
| Automatizar un Excel abierto, recalcular, macros, plantillas complejas | **xlwings** |
| Crear o modificar `.xlsx` sin Excel (servidores, Linux) | OpenPyXL, XlsxWriter |
| Leer datos de Excel para analizarlos | pandas (`read_excel`) |
| Python dentro de Excel en Microsoft 365 | «Python in Excel» (`=PY()`), xlwings Lite |
| Excel en Windows con control total COM | pywin32 (más bajo nivel) |

---

## 2. Instalación y requisitos

```bash
pip install xlwings
xlwings addin install          # Complemento de Excel (cinta «xlwings») para RunPython y UDF
xlwings quickstart mi_proyecto # Crea mi_proyecto.xlsm + mi_proyecto.py ya conectados
```

| Sistema | Requisitos |
|---|---|
| Windows | Excel de escritorio. Funcionalidad completa (incluidas UDF). |
| macOS | Excel de escritorio. Sin UDF; conceder permisos de automatización a la terminal/IDE. |
| Linux / servidores | Sin Excel: use xlwings Server, xlwings Lite u OpenPyXL. |

```python
import xlwings as xw
```

---

## 3. Modelo de objetos

La jerarquía refleja la de Excel/VBA:

```
App (instancia de Excel)
 └── Book (libro)
      └── Sheet (hoja)
           └── Range (rango) · Table · Chart · Picture · Shape
```

```python
xw.apps            # Instancias de Excel abiertas
xw.books           # Libros de la instancia activa
xw.sheets          # Hojas del libro activo
xw.Range("A1")     # Rango de la hoja activa (cómodo en uso interactivo)
```

---

## 4. Aplicaciones y libros

```python
libro = xw.Book()                                 # Libro nuevo en la instancia activa (o abre Excel)
libro = xw.Book("ventas.xlsx")                    # Abrir (o conectarse si ya está abierto)
libro = xw.books["ventas.xlsx"]                   # Conectar con un libro ya abierto

# Control explícito: instancia propia, invisible, que se cierra al terminar
with xw.App(visible=False, add_book=False) as app:
    app.display_alerts = False                    # Sin avisos («¿Desea guardar?», vínculos…)
    libro = app.books.open("plantilla.xlsx", update_links=False, read_only=False)
    ...
    libro.save("informe_noviembre.xlsx")          # Guardar como
    libro.close()
                                                  # Al salir del bloque, Excel se cierra

libro.fullname, libro.name, libro.sheets, libro.app
libro.save()                                      # Guardar en el mismo archivo
```

> Use siempre `with xw.App(...)` o `try/finally` con `app.quit()` en procesos automáticos: un error sin cerrar Excel deja procesos ocultos (`EXCEL.EXE`) bloqueando archivos.

---

## 5. Hojas

```python
hoja = libro.sheets["Datos"]          # Por nombre
hoja = libro.sheets[0]                # Por posición
nueva = libro.sheets.add("Resumen", after=libro.sheets[-1])
hoja.name = "Datos 2026"
hoja.copy(after=libro.sheets[0], name="Copia")
hoja.activate() ; hoja.visible = False ; hoja.delete()
hoja.clear() ; hoja.clear_contents()  # Todo / solo valores
hoja.used_range.address               # Rango usado
hoja.autofit()                        # Ajustar anchos y altos
[s.name for s in libro.sheets]
```

---

## 6. Rangos: leer y escribir

```python
h = libro.sheets["Datos"]
h["A1"].value = "Informe de ventas"
h["A2"].value = [["Producto", "Unidades"], ["Silla", 12], ["Mesa", 4]]   # Lista de listas → bloque
h["D1"].value = [1, 2, 3]                                                 # Lista → fila
h["F1"].options(transpose=True).value = [1, 2, 3]                         # Lista → columna

valor = h["B3"].value                                  # Escalar
fila = h["A3:B3"].value                                # Lista
bloque = h["A2:B4"].value                              # Lista de listas
tabla = h["A2"].expand().value                         # Hasta la última celda contigua ("table", "down", "right")

h["A1:C10"].clear_contents()
h["E2"].formula = "=SUM(B3:B4)"
h["E2"].formula2 = "=FILTER(A3:B4,B3:B4>5)"            # Fórmulas de matriz dinámica
h.range((2, 1), (10, 3))                              # Por (fila, columna)
h.cells(1, 1)
h["A1"].offset(1, 2) ; h["A1"].resize(5, 3) ; h["A1"].end("down") ; h["A1"].current_region
ultima_fila = h.range("A" + str(h.cells.last_cell.row)).end("up").row
```

---

## 7. Conversores: pandas, NumPy, fechas y dimensiones

```python
import pandas as pd

h["A1"].options(index=False).value = df                              # Escribir un DataFrame
df = h["A1"].options(pd.DataFrame, header=1, index=False, expand="table").value   # Leer
serie = h["A1"].options(pd.Series, expand="table").value
arr = h["A1:C100"].options(np.array, ndim=2).value
h["A1"].options(ndim=2).value                                         # Siempre lista de listas
h["A1"].options(numbers=int).value                                    # Enteros en lugar de float
h["A1"].options(dates=dt.date).value                                  # Fechas como date
h["A1"].options(empty="").value                                       # Celdas vacías como ""
```

| Opción | Efecto |
|---|---|
| `pd.DataFrame`, `pd.Series`, `np.array`, `dict` | Tipo del resultado. |
| `expand="table"` | Detecta el tamaño del bloque automáticamente. |
| `index`, `header` | Incluir/leer índice y cabeceras. |
| `ndim=1/2` | Fuerza dimensiones (evita escalares o listas planas inesperadas). |
| `transpose=True` | Transponer al leer/escribir. |
| `numbers`, `dates`, `empty` | Conversión de tipos de celda. |

> Escribir **un bloque completo** es mucho más rápido que escribir celda a celda: cada asignación es una llamada a Excel.

---

## 8. Formato

```python
r = h["A1:F1"]
r.font.bold = True ; r.font.size = 12 ; r.font.color = "#FFFFFF" ; r.font.name = "Calibri"
r.color = "#1F4E78"                                   # Relleno (hex o tupla RGB)
r.number_format = "#,##0.00 €"                       # Formato numérico (notación de Excel local)
h["B:B"].number_format = "dd/mm/yyyy"
r.column_width = 18 ; r.row_height = 24
r.wrap_text = True
r.merge() ; r.unmerge()
h["A1"].add_hyperlink("https://www.ejemplo.es", text_to_display="Ver web")
h["B2"].note.text = "Dato provisional"               # Nota (comentario clásico)
h.autofit()

# Lo que no expone la API de xlwings se hace con .api (ver sección 14)
r.api.HorizontalAlignment = -4108                     # xlCenter (Windows)
```

---

## 9. Tablas de Excel, nombres y fórmulas

```python
tabla = h.tables.add(source=h["A1"].expand(), name="TablaVentas", table_style_name="TableStyleMedium2")
tabla.update(df_nuevo, index=False)                    # Sustituir los datos manteniendo formato y fórmulas
tabla.data_body_range ; tabla.header_row_range ; tabla.show_totals = True

libro.names.add("Objetivo", "=Parametros!$B$2")       # Nombre definido
h["H2"].formula = "=SUM(TablaVentas[Importe])/Objetivo"
libro.app.calculate()                                  # Recalcular todo
valor = h["H2"].value                                  # Resultado calculado por Excel
```

Ventaja clave frente a OpenPyXL: **Excel calcula las fórmulas** y Python puede leer el resultado inmediatamente.

---

## 10. Gráficos e imágenes

```python
grafico = h.charts.add(left=h["H2"].left, top=h["H2"].top, width=480, height=280)
grafico.set_source_data(h["A1"].expand())
grafico.chart_type = "column_clustered"               # "line", "pie", "bar_stacked", "xy_scatter"…
grafico.name = "VentasMensuales"

import matplotlib.pyplot as plt
fig, ax = plt.subplots(figsize=(6, 3))
ax.plot(df["mes"], df["ventas"])
h.pictures.add(fig, name="Tendencia", update=True, anchor=h["H20"])   # update=True sustituye la existente
h.pictures.add("logo.png", left=h["A1"].left, top=h["A1"].top, width=120)
```

---

## 11. Rendimiento

```python
app = libro.app
app.screen_updating = False          # No redibujar la pantalla
app.calculation = "manual"           # No recalcular tras cada escritura
app.enable_events = False            # No disparar eventos VBA (Worksheet_Change…)
try:
    h["A1"].options(index=False).value = df_grande        # Escribir en bloque
finally:
    app.calculation = "automatic"
    app.screen_updating = True
    app.enable_events = True
```

| Lento | Rápido |
|---|---|
| Bucle que escribe celda a celda | Una asignación con lista de listas o DataFrame |
| Leer celdas sueltas en un bucle | Leer el bloque completo y procesarlo en Python |
| Recalcular tras cada cambio | `calculation = "manual"` y `app.calculate()` al final |

---

## 12. Llamar a Python desde Excel (RunPython)

Con el complemento instalado, una macro VBA o un botón puede ejecutar Python:

```vba
Sub ActualizarInforme()
    RunPython "import cierre_mensual; cierre_mensual.desde_excel()"
End Sub
```

```python
# cierre_mensual.py (en la misma carpeta que el libro .xlsm)
import xlwings as xw

def desde_excel():
    libro = xw.Book.caller()                   # El libro que llamó a la macro
    hoja = libro.sheets["Panel"]
    hoja["B2"].value = "Actualizado"

if __name__ == "__main__":
    xw.Book("cierre_mensual.xlsm").set_mock_caller()   # Probar desde Python como si se llamara desde Excel
    desde_excel()
```

`xlwings quickstart` genera el par `.xlsm` + `.py` ya configurado. La configuración (intérprete, rutas) se ajusta en la pestaña *xlwings* de la cinta o en una hoja `xlwings.conf`.

---

## 13. Funciones definidas por el usuario (UDF)

Funciones de hoja escritas en Python (solo Windows):

```python
import xlwings as xw
import pandas as pd

@xw.func
def variacion(real: float, previsto: float) -> float:
    """Variación relativa entre real y previsto."""
    return (real - previsto) / previsto if previsto else None

@xw.func
@xw.arg("datos", pd.DataFrame, index=False, header=True)
@xw.ret(index=False, header=True, expand="table")
def resumen_por(datos, columna):
    """Agrupa y suma: =resumen_por(A1:D100; "region")"""
    return datos.groupby(columna, as_index=False).sum(numeric_only=True)
```

Tras pulsar *Import Functions* en la cinta *xlwings*, se usan como cualquier función: `=variacion(B2;C2)`. Las funciones que devuelven tablas se expanden automáticamente (matrices dinámicas).

---

## 14. Acceso a la API nativa de Excel (`.api`)

`.api` da acceso al objeto subyacente (COM en Windows, AppleScript en macOS) para todo lo que xlwings no envuelve:

```python
from xlwings.constants import HAlign, BordersIndex, LineStyle

h["A1:F1"].api.HorizontalAlignment = HAlign.xlHAlignCenter
borde = h["A1:F20"].api.Borders(BordersIndex.xlEdgeBottom)
borde.LineStyle = LineStyle.xlContinuous
h.api.PageSetup.Orientation = 2                               # Horizontal (xlLandscape)
libro.api.RefreshAll()                                        # Actualizar consultas y tablas dinámicas
h["A1"].api.AutoFilter(Field=3, Criteria1=">1000")
```

> El código con `.api` es específico de cada plataforma (las llamadas de Windows no funcionan igual en macOS).

---

## 15. Macros VBA desde Python

```python
macro = libro.macro("Modulo1.CalcularComisiones")
resultado = macro(2026, "Norte")                              # Ejecuta la macro VBA con argumentos
```

Útil para migrar de VBA a Python de forma gradual, reutilizando macros existentes.

---

## 16. Exportar a PDF

```python
libro.to_pdf("informe.pdf", include=["Resumen", "Gráficos"])      # Hojas seleccionadas
libro.sheets["Resumen"].to_pdf("resumen.pdf")
libro.to_pdf("informe.pdf", layout="membrete.pdf")                # Fondo/membrete (PDF de plantilla)
```

Las hojas cuyo nombre empieza por `#` se excluyen por defecto (útil para hojas auxiliares).

---

## 17. xlwings Lite, Server y Reader

| Variante | Descripción |
|---|---|
| **xlwings Lite** | Complemento de Excel (incluido Excel en la web) que ejecuta Python en el navegador, sin instalar Python. Usa `@script` y `@func`. |
| **xlwings Server** | Ejecuta el código Python en un servidor; Excel (escritorio, web o Google Sheets) actúa de interfaz. No requiere Excel en el servidor. |
| **xlwings Reader** | Lee archivos de Excel sin Excel con la sintaxis de xlwings (`xw.Book(ruta, mode="r")`); requiere licencia. |

```python
# Estilo xlwings Lite / Server
import xlwings as xw

@xw.script
def actualizar(book: xw.Book):
    hoja = book.sheets["Panel"]
    hoja["A1"].value = "Hola desde Python"
```

---

## 18. Ejemplo práctico completo

**Automatización del cierre mensual de gastos por centro de coste**. Un proceso que antes se hacía a mano en Excel:

1. Lee los asientos contables del mes (CSV exportado del ERP) y el **presupuesto** desde la hoja `Presupuesto` del libro de control.
2. Calcula en Python (pandas) el real por centro y cuenta, la **desviación** frente al presupuesto y las alertas.
3. Escribe en Excel: hoja de datos como **tabla de Excel**, hoja `Resumen` con fórmulas que **Excel recalcula** (y Python verifica), formato, gráfico nativo y gráfico de Matplotlib.
4. Exporta el resumen a **PDF** para la dirección y guarda el libro con el nombre del mes.
5. Expone `desde_excel()` para lanzarlo con un botón (RunPython) y dos **UDF** reutilizables en la hoja.

El código separa la **lógica** (pandas, comprobable sin Excel) de la **capa de Excel** (xlwings). Con `--sin-excel` se ejecuta solo la lógica y se escriben CSV, lo que permite probarla en cualquier equipo o en integración continua.

```python
"""
cierre_mensual.py — Cierre mensual de gastos con pandas + xlwings.
Requiere: pip install xlwings pandas matplotlib   (y Microsoft Excel en Windows o macOS)
Uso:
    python cierre_mensual.py                 # Genera datos de ejemplo, abre Excel y produce el informe
    python cierre_mensual.py --sin-excel     # Solo la lógica (sin Excel): escribe CSV de resultados
Desde Excel (botón):  RunPython "import cierre_mensual; cierre_mensual.desde_excel()"
"""
from __future__ import annotations

import argparse
import sys
from dataclasses import dataclass
from datetime import date
from pathlib import Path

import numpy as np
import pandas as pd

CARPETA = Path(__file__).resolve().parent
ASIENTOS = CARPETA / "asientos_2026_11.csv"
CONTROL = CARPETA / "control_gastos.xlsx"
MES = date(2026, 11, 1)
CENTROS = ["Comercial", "Marketing", "Operaciones", "Tecnología", "Administración"]
CUENTAS = {"621": "Arrendamientos", "623": "Servicios profesionales", "627": "Publicidad",
           "628": "Suministros", "629": "Viajes y dietas", "640": "Sueldos y salarios"}
UMBRAL_ALERTA = 0.10                     # Desviación relativa que genera alerta


# ============================================================ Lógica (sin Excel)
@dataclass
class ResultadoCierre:
    detalle: pd.DataFrame                # Asientos limpios
    resumen: pd.DataFrame                # Real vs. presupuesto por centro y cuenta
    por_centro: pd.DataFrame             # Totales por centro
    alertas: pd.DataFrame


def generar_ejemplo() -> pd.DataFrame:
    """Crea el CSV de asientos (como lo exportaría el ERP) y devuelve el presupuesto."""
    rng = np.random.default_rng(11)
    base = {c: rng.uniform(4_000, 30_000, len(CUENTAS)).round(-2) for c in CENTROS}
    presupuesto = pd.DataFrame([(c, cta, base[c][i]) for c in CENTROS for i, cta in enumerate(CUENTAS)],
                               columns=["centro", "cuenta", "presupuesto"])
    filas = []
    for _, p in presupuesto.iterrows():
        factor = rng.normal(1.0, 0.08) * (1.35 if (p.centro, p.cuenta) in {("Marketing", "627"), ("Comercial", "629")} else 1)
        n = rng.integers(3, 12)
        importes = rng.dirichlet(np.ones(n)) * p.presupuesto * factor
        for imp in importes:
            filas.append({"fecha": MES.replace(day=int(rng.integers(1, 29))).strftime("%d/%m/%Y"),
                          "asiento": int(rng.integers(10_000, 99_999)), "cuenta": p.cuenta,
                          "centro": f" {p.centro.upper()} " if rng.random() < 0.1 else p.centro,
                          "concepto": f"{CUENTAS[p.cuenta]} {p.centro}",
                          "importe": f"{imp:.2f}".replace(".", ",")})
    pd.DataFrame(filas).to_csv(ASIENTOS, sep=";", index=False, encoding="utf-8-sig")
    return presupuesto


def leer_asientos(ruta: Path) -> pd.DataFrame:
    df = pd.read_csv(ruta, sep=";", decimal=",", dtype={"cuenta": str}, encoding="utf-8-sig")
    df["fecha"] = pd.to_datetime(df["fecha"], format="%d/%m/%Y")
    df["centro"] = df["centro"].str.strip().str.capitalize().replace({"Tecnologia": "Tecnología"})
    df["nombre_cuenta"] = df["cuenta"].map(CUENTAS)
    desconocidos = df[~df["centro"].isin(CENTROS) | df["nombre_cuenta"].isna()]
    if not desconocidos.empty:
        raise ValueError(f"Asientos con centro o cuenta desconocidos:\n{desconocidos.head()}")
    return df


def calcular_cierre(asientos: pd.DataFrame, presupuesto: pd.DataFrame) -> ResultadoCierre:
    real = asientos.groupby(["centro", "cuenta"], as_index=False)["importe"].sum().rename(columns={"importe": "real"})
    resumen = presupuesto.merge(real, on=["centro", "cuenta"], how="outer", validate="1:1").fillna(0)
    resumen["nombre_cuenta"] = resumen["cuenta"].map(CUENTAS)
    resumen["desviacion"] = resumen["real"] - resumen["presupuesto"]
    resumen["desviacion_pct"] = np.where(resumen["presupuesto"] != 0, resumen["desviacion"] / resumen["presupuesto"], np.nan)
    resumen = resumen[["centro", "cuenta", "nombre_cuenta", "presupuesto", "real", "desviacion", "desviacion_pct"]]
    resumen = resumen.sort_values(["centro", "cuenta"], ignore_index=True)

    por_centro = resumen.groupby("centro", as_index=False)[["presupuesto", "real", "desviacion"]].sum()
    por_centro["desviacion_pct"] = por_centro["desviacion"] / por_centro["presupuesto"]
    por_centro = por_centro.set_index("centro").loc[CENTROS].reset_index()

    alertas = resumen[resumen["desviacion_pct"] > UMBRAL_ALERTA].sort_values("desviacion", ascending=False)
    return ResultadoCierre(asientos, resumen.round(2), por_centro.round(4), alertas.round(4))


# ============================================================ Funciones de hoja (UDF, Windows)
import xlwings as xw  # noqa: E402  (se importa aquí para que la lógica anterior no dependa de xlwings)


@xw.func
def desviacion_pct(real: float, presupuesto: float):
    """=desviacion_pct(real; presupuesto) → desviación relativa, o vacío si no hay presupuesto."""
    return (real - presupuesto) / presupuesto if presupuesto else None


@xw.func
@xw.arg("importes", np.array, ndim=1)
@xw.arg("pesos", np.array, ndim=1)
def media_ponderada(importes, pesos):
    """=media_ponderada(B2:B20; C2:C20)"""
    return float(np.average(importes, weights=pesos))


# ============================================================ Capa de Excel
AZUL, AZUL_CLARO, ROJO, VERDE = "#1F4E78", "#DDEBF7", "#C00000", "#548235"


def preparar_libro_control(app: xw.App, presupuesto: pd.DataFrame) -> xw.Book:
    """Crea el libro de control con el presupuesto si no existe (en la realidad ya lo mantiene Finanzas)."""
    if CONTROL.exists():
        return app.books.open(CONTROL)
    libro = app.books.add()
    hoja = libro.sheets[0]
    hoja.name = "Presupuesto"
    hoja["A1"].options(index=False).value = presupuesto
    hoja.tables.add(source=hoja["A1"].expand(), name="TablaPresupuesto")
    hoja["C:C"].number_format = "#,##0.00"
    hoja.autofit()
    libro.save(CONTROL)
    return libro


def leer_presupuesto(libro: xw.Book) -> pd.DataFrame:
    df = libro.sheets["Presupuesto"]["A1"].options(pd.DataFrame, index=False, expand="table").value
    df["cuenta"] = df["cuenta"].astype(str).str.replace(r"\.0$", "", regex=True)   # Excel devuelve números como float
    return df


def hoja_nueva(libro: xw.Book, nombre: str) -> xw.Sheet:
    if nombre in [s.name for s in libro.sheets]:
        libro.sheets[nombre].delete()
    return libro.sheets.add(nombre, after=libro.sheets[-1])


def escribir_datos(libro: xw.Book, r: ResultadoCierre) -> None:
    hoja = hoja_nueva(libro, "Asientos")
    datos = r.detalle[["fecha", "asiento", "centro", "cuenta", "nombre_cuenta", "concepto", "importe"]]
    hoja["A1"].options(index=False).value = datos
    hoja.tables.add(source=hoja["A1"].expand(), name="TablaAsientos", table_style_name="TableStyleLight9")
    hoja["A:A"].number_format = "dd/mm/yyyy"
    hoja["G:G"].number_format = "#,##0.00 €"
    hoja.autofit()


def escribir_resumen(libro: xw.Book, r: ResultadoCierre) -> float:
    hoja = hoja_nueva(libro, "Resumen")
    hoja["A1"].value = f"Cierre de gastos · {MES:%m/%Y}"
    hoja["A1"].font.size = 16
    hoja["A1"].font.bold = True
    hoja["A1"].font.color = AZUL
    hoja["A2"].value = f"Alerta si la desviación supera el {UMBRAL_ALERTA:.0%}"

    cabecera = ["Centro", "Presupuesto", "Real", "Desviación", "Desv. %"]
    hoja["A4"].value = cabecera
    for i, centro in enumerate(CENTROS, start=5):
        hoja[f"A{i}"].value = centro
        hoja[f"B{i}"].formula = f'=SUMIFS(TablaPresupuesto[presupuesto],TablaPresupuesto[centro],A{i})'
        hoja[f"C{i}"].formula = f'=SUMIFS(TablaAsientos[importe],TablaAsientos[centro],A{i})'
        hoja[f"D{i}"].formula = f"=C{i}-B{i}"
        hoja[f"E{i}"].formula = f"=IF(B{i}=0,\"\",D{i}/B{i})"
    total = 5 + len(CENTROS)
    hoja[f"A{total}"].value = "Total"
    for col in "BCD":
        hoja[f"{col}{total}"].formula = f"=SUM({col}5:{col}{total - 1})"
    hoja[f"E{total}"].formula = f"=D{total}/B{total}"

    encabezado = hoja["A4:E4"]
    encabezado.color = AZUL
    encabezado.font.color = "#FFFFFF"
    encabezado.font.bold = True
    hoja[f"A{total}:E{total}"].font.bold = True
    hoja[f"A{total}:E{total}"].color = AZUL_CLARO
    hoja[f"B5:D{total}"].number_format = "#,##0 €"
    hoja[f"E5:E{total}"].number_format = "0.0%"

    libro.app.calculate()                                                     # Excel calcula las fórmulas
    desviaciones = hoja[f"E5:E{total - 1}"].options(ndim=1).value
    for fila, valor in enumerate(desviaciones, start=5):                      # Formato condicional «manual»
        if isinstance(valor, float) and valor > UMBRAL_ALERTA:
            hoja[f"E{fila}"].font.color = ROJO
            hoja[f"E{fila}"].font.bold = True
        elif isinstance(valor, float) and valor < 0:
            hoja[f"E{fila}"].font.color = VERDE

    grafico = hoja.charts.add(left=hoja["G4"].left, top=hoja["G4"].top, width=430, height=250)
    grafico.set_source_data(hoja[f"A4:C{total - 1}"])
    grafico.chart_type = "column_clustered"
    grafico.name = "PresupuestoVsReal"

    fila_alertas = total + 3
    hoja[f"A{fila_alertas}"].value = "Partidas con desviación superior al umbral"
    hoja[f"A{fila_alertas}"].font.bold = True
    if r.alertas.empty:
        hoja[f"A{fila_alertas + 1}"].value = "Sin alertas este mes"
    else:
        vista = r.alertas[["centro", "nombre_cuenta", "presupuesto", "real", "desviacion", "desviacion_pct"]]
        hoja[f"A{fila_alertas + 1}"].options(index=False).value = vista
        hoja[f"C{fila_alertas + 2}:E{fila_alertas + 1 + len(vista)}"].number_format = "#,##0 €"
        hoja[f"F{fila_alertas + 2}:F{fila_alertas + 1 + len(vista)}"].number_format = "0.0%"
        hoja[f"A{fila_alertas + 1}:F{fila_alertas + 1}"].font.bold = True
    hoja.autofit()
    return hoja[f"C{total}"].value                                            # Total real calculado por Excel


def insertar_grafico_matplotlib(libro: xw.Book, r: ResultadoCierre) -> None:
    import matplotlib
    matplotlib.use("Agg")
    import matplotlib.pyplot as plt
    datos = r.por_centro.sort_values("desviacion_pct")
    fig, ax = plt.subplots(figsize=(6, 3.2), layout="constrained")
    colores = [ROJO if v > UMBRAL_ALERTA else (VERDE if v < 0 else "#7F7F7F") for v in datos["desviacion_pct"]]
    ax.barh(datos["centro"], datos["desviacion_pct"] * 100, color=colores)
    ax.axvline(UMBRAL_ALERTA * 100, color=ROJO, linestyle="--", linewidth=1)
    ax.set_xlabel("Desviación sobre presupuesto (%)")
    ax.set_title("Desviación por centro de coste")
    hoja = libro.sheets["Resumen"]
    hoja.pictures.add(fig, name="DesviacionCentros", update=True, anchor=hoja["G22"])
    plt.close(fig)


def ejecutar_en_excel(libro: xw.Book, presupuesto: pd.DataFrame | None = None) -> Path:
    app = libro.app
    app.screen_updating = False
    app.calculation = "manual"
    try:
        presupuesto = presupuesto if presupuesto is not None else leer_presupuesto(libro)
        r = calcular_cierre(leer_asientos(ASIENTOS), presupuesto)
        escribir_datos(libro, r)
        total_excel = escribir_resumen(libro, r)
        insertar_grafico_matplotlib(libro, r)
        total_python = r.resumen["real"].sum()
        if abs(total_excel - total_python) > 0.01:                            # Conciliación Python ↔ Excel
            raise RuntimeError(f"Descuadre: Excel {total_excel:.2f} ≠ Python {total_python:.2f}")
        libro.sheets["Resumen"].activate()
    finally:
        app.calculation = "automatic"
        app.screen_updating = True
    destino = CARPETA / f"cierre_gastos_{MES:%Y_%m}.xlsx"
    libro.save(destino)
    libro.to_pdf(destino.with_suffix(".pdf"), include=["Resumen"])
    return destino


def desde_excel() -> None:
    """Punto de entrada para un botón en Excel: RunPython "import cierre_mensual; cierre_mensual.desde_excel()"."""
    libro = xw.Book.caller()
    destino = ejecutar_en_excel(libro)
    libro.sheets["Resumen"]["A2"].value = f"Generado: {destino.name}"


# ============================================================ Ejecución
def main() -> int:
    parser = argparse.ArgumentParser(description="Cierre mensual de gastos")
    parser.add_argument("--sin-excel", action="store_true", help="Ejecutar solo la lógica (sin Excel)")
    args = parser.parse_args()

    presupuesto = generar_ejemplo()
    if args.sin_excel:
        r = calcular_cierre(leer_asientos(ASIENTOS), presupuesto)
        r.resumen.to_csv(CARPETA / "resumen_cierre.csv", sep=";", decimal=",", index=False)
        print(r.por_centro.to_string(index=False))
        print(f"\nAlertas ({len(r.alertas)}):")
        print(r.alertas[["centro", "nombre_cuenta", "desviacion", "desviacion_pct"]].to_string(index=False))
        print(f"UDF desviacion_pct(110, 100) = {desviacion_pct(110, 100):.2f}")
        return 0

    if sys.platform not in ("win32", "darwin"):
        print("xlwings necesita Microsoft Excel (Windows o macOS). Use --sin-excel en este sistema.")
        return 1
    with xw.App(visible=False, add_book=False) as app:                       # Excel se cierra al terminar
        app.display_alerts = False
        libro = preparar_libro_control(app, presupuesto)
        destino = ejecutar_en_excel(libro, leer_presupuesto(libro))
        libro.close()
    print(f"✔ Informe guardado: {destino.name} y {destino.with_suffix('.pdf').name}")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Arquitectura del ejemplo:

| Capa | Funciones | Depende de Excel |
|---|---|---|
| Lógica | `leer_asientos`, `calcular_cierre`, `generar_ejemplo` | No (pandas) |
| Excel | `preparar_libro_control`, `escribir_*`, `insertar_grafico_matplotlib`, `ejecutar_en_excel` | Sí |
| Entradas | `main()` (línea de comandos), `desde_excel()` (botón), UDF | Sí (salvo `--sin-excel`) |

> **Conciliación**: el total que calcula Excel con `SUMIFS` se compara con el de pandas antes de guardar. Si no cuadran (filtros, centros mal escritos, fórmulas rotas), el proceso se detiene.

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `XlwingsError: Make sure to have "appscript" and "psutil"…` / no encuentra Excel | Excel no instalado o plataforma no compatible (Linux). | Instale Excel; en Linux use OpenPyXL o xlwings Server/Lite. |
| `com_error: (-2147352567, 'Exception occurred.' …)` | Error de Excel (ruta inválida, hoja protegida, libro abierto en modo lectura). | Revise el mensaje interno, rutas absolutas y protección. |
| Procesos `EXCEL.EXE` ocultos que bloquean archivos | Script terminado sin cerrar Excel. | `with xw.App(...)` o `try/finally: app.quit()`; en último caso `app.kill()`. |
| Excel se queda esperando un diálogo | Avisos de guardado, vínculos o compatibilidad. | `app.display_alerts = False`, `update_links=False`. |
| Lectura devuelve escalar o lista plana en lugar de tabla | Dimensiones automáticas. | `.options(ndim=2)` o `.options(pd.DataFrame, expand="table")`. |
| Números leídos como `float` (`621.0`) | Excel almacena todos los números como doble precisión. | `.options(numbers=int)` o convertir en pandas. |
| Muy lento | Escritura celda a celda, recálculo y pantalla activos. | Escriba en bloque; `screen_updating=False`, `calculation="manual"`. |
| `RunPython` no encuentra el módulo | El `.py` no está junto al libro o en `PYTHONPATH`. | Misma carpeta o configure `PYTHONPATH` en la cinta *xlwings*. |
| Las UDF no aparecen | No se importaron o no es Windows. | *Import Functions* en la cinta (solo Windows). |
| `LicenseError: xlwings Reader requires a license key` | Uso de `mode="r"` sin licencia. | Use pandas/OpenPyXL para leer sin Excel, o configure la licencia. |
| macOS solicita permisos o falla | Falta permiso de automatización. | Ajustes del Sistema → Privacidad → Automatización: permitir controlar Excel. |

---

## 20. Buenas prácticas

- Separe la lógica de negocio (pandas, comprobable sin Excel) de la capa de Excel.
- Abra Excel con `with xw.App(visible=False, add_book=False)` en procesos automáticos y desactive alertas.
- Lea y escriba **bloques completos**; desactive actualización de pantalla y recálculo durante escrituras masivas.
- Use tablas de Excel y nombres definidos: las fórmulas sobreviven a cambios de tamaño de los datos.
- Aproveche que Excel calcula: lea los resultados de las fórmulas y **concílielos** con los cálculos en Python.
- Trabaje sobre copias o guarde con otro nombre; no sobrescriba la plantilla.
- Mantenga el código de `.api` al mínimo y aislado (es específico de la plataforma).
- Para procesos de servidor sin Excel, use OpenPyXL o xlwings Server.

---

## 21. Referencia rápida

```python
with xw.App(visible=False, add_book=False) as app: libro = app.books.open(ruta) · app.display_alerts = False
xw.Book() · xw.Book(ruta) · xw.books["x.xlsx"] · xw.Book.caller() · libro.save(ruta) · libro.close() · libro.to_pdf(ruta, include=)
libro.sheets["Hoja"] · libro.sheets.add("Nueva", after=) · hoja.autofit() · hoja.used_range · hoja.clear_contents()
hoja["A1"].value = valor | lista | lista_de_listas · hoja["A1"].expand().value · hoja.range((f, c), (f2, c2))
hoja["A1"].options(index=False).value = df · hoja["A1"].options(pd.DataFrame, expand="table", index=False).value
rango.formula / formula2 · rango.number_format · rango.color · rango.font.bold/size/color · rango.column_width
hoja.tables.add(source=, name=, table_style_name=) · tabla.update(df) · libro.names.add(nombre, "=Hoja!$A$1")
hoja.charts.add(left=, top=, width=, height=) · chart.set_source_data(rango) · chart.chart_type = "line"
hoja.pictures.add(fig, name=, update=True, anchor=) · app.calculate() · app.screen_updating · app.calculation = "manual"
@xw.func · @xw.arg("x", pd.DataFrame) · @xw.ret(expand="table") · libro.macro("Modulo.Macro")(args) · rango.api
```

---

## 22. Ejercicios propuestos

1. Añada al cierre una hoja por centro de coste con sus asientos y un gráfico de evolución de los últimos 12 meses.
2. Convierta una macro VBA existente de su organización a Python con xlwings, manteniendo el botón en el libro.
3. Cree una UDF que consulte una API de tipos de cambio y devuelva una tabla que se expanda en la hoja.
4. Automatice la actualización de un libro con consultas de Power Query (`libro.api.RefreshAll()`) y exporte el resultado a PDF cada lunes.
5. Implemente el mismo informe con OpenPyXL y compare tiempos, funcionalidades y requisitos de despliegue.

---

## 23. Referencias

- Documentación oficial: https://docs.xlwings.org/
- Guía rápida: https://docs.xlwings.org/en/latest/quickstart.html
- Conversores y opciones: https://docs.xlwings.org/en/latest/converters.html
- RunPython y UDF: https://docs.xlwings.org/en/latest/vba.html · https://docs.xlwings.org/en/latest/udfs.html
- xlwings Lite: https://lite.xlwings.org/
