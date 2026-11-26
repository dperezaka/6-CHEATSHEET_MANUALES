# Manual de uso: Rich

> **Categoría:** Utilidades generales · **Utilidad:** Mostrar información y tablas en terminales de forma atractiva
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Marcado y estilos](#5-marcado-y-estilos)
6. [Tablas](#6-tablas)
7. [Paneles, columnas y reglas](#7-paneles-columnas-y-reglas)
8. [Barras de progreso y estados](#8-barras-de-progreso-y-estados)
9. [Contenido especializado](#9-contenido-especializado)
10. [Entrada de datos](#10-entrada-de-datos)
11. [Operaciones avanzadas](#11-operaciones-avanzadas)
12. [Ejemplo práctico completo](#12-ejemplo-práctico-completo)
13. [Errores frecuentes](#13-errores-frecuentes)
14. [Buenas prácticas](#14-buenas-prácticas)
15. [Referencia rápida](#15-referencia-rápida)
16. [Ejercicios propuestos](#16-ejercicios-propuestos)
17. [Referencias](#17-referencias)

---

## 1. Descripción

**Rich** permite mostrar en la terminal texto con colores y estilos, tablas, paneles, barras de progreso, árboles, código con resaltado de sintaxis, Markdown, registros (*logs*) formateados y trazas de error legibles.

**Casos de uso habituales:**
- Herramientas de línea de comandos (CLI) profesionales.
- Scripts de automatización con salida clara y legible.
- Informes rápidos en consola.
- Depuración (inspección de objetos y trazas de error mejoradas).
- Paneles de estado en tiempo real.

Funciona en Windows (Windows Terminal recomendado), macOS y Linux, y también en Jupyter.

---

## 2. Instalación e importación

```bash
pip install rich
```

Ver una demostración de sus capacidades:

```bash
python -m rich
```

Importaciones habituales:

```python
from rich import print
from rich.console import Console
from rich.table import Table
from rich.panel import Panel
from rich.progress import track, Progress
from rich.prompt import Prompt, Confirm, IntPrompt
from rich.markdown import Markdown
from rich.syntax import Syntax
from rich.tree import Tree
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Console` | Objeto central que escribe en la terminal. |
| Marcado (*markup*) | Etiquetas tipo BBCode para dar estilo: `[bold red]texto[/]`. |
| Estilo | Combinación de color, fondo y atributos (`bold`, `italic`…). |
| *Renderable* | Cualquier elemento que Rich sabe dibujar (tabla, panel, árbol…). |
| `Live` | Pantalla que se actualiza en tiempo real. |

---

## 4. Operaciones básicas

### 4.1 `print` mejorado

```python
from rich import print

print("[bold green]Proceso completado[/bold green] :white_check_mark:")
print({"cliente": "Ana", "total": 59.8, "pagado": True})     # Resaltado automático
print([1, 2, 3], None, True)
```

### 4.2 Objeto `Console`

```python
from rich.console import Console

console = Console()

console.print("Texto normal")
console.print("Texto con estilo", style="bold white on blue")
console.print("[italic yellow]Advertencia:[/] revise la configuración")
console.print("Centrado", justify="center")
console.rule("[bold]Sección")                       # Línea separadora con título
console.log("Mensaje con hora y archivo de origen")  # Incluye marca de tiempo
console.print_json('{"id": 1, "activo": true}')     # JSON formateado
```

### 4.3 Inspeccionar objetos

```python
from rich import inspect

inspect([1, 2, 3], methods=True)
inspect(console)
```

### 4.4 Emojis

```python
console.print(":rocket: Despliegue iniciado  :warning: Atención  :x: Error")
```

---

## 5. Marcado y estilos

### 5.1 Sintaxis del marcado

```python
console.print("[bold]Negrita[/bold] [italic]cursiva[/italic] [underline]subrayado[/underline]")
console.print("[red]Rojo[/] [green]verde[/] [blue]azul[/]")
console.print("[white on red] FONDO ROJO [/]")
console.print("[bold magenta]Varios[/] [dim]atenuado[/] [strike]tachado[/] [reverse]invertido[/]")
console.print("[link=https://www.python.org]Enlace a Python[/link]")
console.print("[#1F4E78]Color hexadecimal[/] [rgb(255,128,0)]Color RGB[/]")
```

`[/]` cierra la última etiqueta abierta.

### 5.2 Atributos de estilo

| Atributo | Efecto |
|---|---|
| `bold` | Negrita |
| `dim` | Atenuado |
| `italic` | Cursiva |
| `underline` | Subrayado |
| `strike` | Tachado |
| `reverse` | Invierte colores |
| `blink` | Parpadeo |
| `on <color>` | Color de fondo |

Colores con nombre: `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`, y variantes `bright_*`, además de los 256 colores estándar (`python -m rich.color` para verlos).

### 5.3 Escapar corchetes

```python
from rich.markup import escape

nombre_usuario = "[admin]"
console.print(f"Usuario: {escape(nombre_usuario)}")
console.print("Lista: \\[1, 2, 3]")              # Corchete escapado manualmente
console.print("[1, 2, 3]", markup=False)         # Desactivar el marcado
```

### 5.4 Objeto `Text` (estilos por fragmentos)

```python
from rich.text import Text

texto = Text("Estado: ")
texto.append("CORRECTO", style="bold green")
texto.append(" · 12 elementos procesados", style="dim")
console.print(texto)

t = Text("Error en la línea 42 del archivo config.yaml")
t.highlight_words(["Error", "42"], style="bold red")
t.highlight_regex(r"\w+\.yaml", style="cyan")
console.print(t)
```

### 5.5 Temas personalizados

```python
from rich.theme import Theme

tema = Theme({
    "info": "cyan",
    "aviso": "bold yellow",
    "error": "bold white on red",
    "ok": "bold green",
})
console = Console(theme=tema)

console.print("Conexión establecida", style="ok")
console.print("[aviso]Espacio en disco bajo[/aviso]")
console.print("[error] FALLO [/error] No se pudo guardar")
```

---

## 6. Tablas

### 6.1 Tabla básica

```python
from rich.table import Table

tabla = Table(title="Ventas por región — Q4 2026")

tabla.add_column("Región", style="cyan", no_wrap=True)
tabla.add_column("Pedidos", justify="right")
tabla.add_column("Importe", justify="right", style="green")
tabla.add_column("Variación", justify="right")

tabla.add_row("Norte", "125", "13.800 €", "[green]+10,4 %[/]")
tabla.add_row("Sur", "98", "11.200 €", "[green]+14,3 %[/]")
tabla.add_row("Este", "143", "13.900 €", "[red]-2,8 %[/]")

console.print(tabla)
```

### 6.2 Opciones de la tabla

```python
from rich import box

tabla = Table(
    title="Inventario",
    caption="Datos a 24/11/2026",
    box=box.ROUNDED,                 # SIMPLE, MINIMAL, HEAVY, DOUBLE, SQUARE, MARKDOWN…
    show_header=True,
    header_style="bold white on #1F4E78",
    show_lines=False,                # Líneas entre filas
    row_styles=["", "dim"],          # Filas alternas
    expand=False,                    # Ocupar todo el ancho
    show_footer=True,
    title_justify="left",
)
```

### 6.3 Opciones de columna

```python
tabla.add_column(
    "Descripción",
    justify="left",          # left, center, right
    style="white",
    header_style="bold",
    min_width=10,
    max_width=40,
    overflow="ellipsis",     # fold, crop, ellipsis
    no_wrap=False,
    footer="Total",
    ratio=2,                 # Proporción del ancho si expand=True
)
```

### 6.4 Tabla a partir de datos

```python
datos = [
    {"producto": "Teclado", "stock": 15, "precio": 29.9},
    {"producto": "Ratón", "stock": 3, "precio": 14.5},
    {"producto": "Monitor", "stock": 0, "precio": 189.0},
]

tabla = Table(title="Inventario")
for col in ("Producto", "Stock", "Precio"):
    tabla.add_column(col, justify="right" if col != "Producto" else "left")

for d in datos:
    color = "red" if d["stock"] == 0 else "yellow" if d["stock"] < 5 else "green"
    tabla.add_row(d["producto"], f"[{color}]{d['stock']}[/]", f"{d['precio']:.2f} €")

console.print(tabla)
```

### 6.5 Mostrar un DataFrame de Pandas

```python
import pandas as pd

def tabla_desde_df(df: pd.DataFrame, titulo: str = "") -> Table:
    t = Table(title=titulo, header_style="bold cyan")
    for col in df.columns:
        es_num = pd.api.types.is_numeric_dtype(df[col])
        t.add_column(str(col), justify="right" if es_num else "left")
    for fila in df.itertuples(index=False):
        t.add_row(*[f"{v:,.2f}" if isinstance(v, float) else str(v) for v in fila])
    return t

console.print(tabla_desde_df(df, "Resumen"))
```

### 6.6 Tablas sin bordes (para alinear datos)

```python
rejilla = Table.grid(padding=(0, 2))
rejilla.add_column(style="bold")
rejilla.add_column()
rejilla.add_row("Usuario:", "ana.lopez")
rejilla.add_row("Rol:", "Administradora")
rejilla.add_row("Último acceso:", "24/11/2026 21:10")
console.print(rejilla)
```

---

## 7. Paneles, columnas y reglas

### 7.1 Paneles

```python
from rich.panel import Panel

console.print(Panel("Copia de seguridad completada correctamente.",
                    title="[bold green]Éxito", subtitle="21:45", border_style="green"))

console.print(Panel.fit("Texto ajustado al contenido", border_style="cyan"))

console.print(Panel(tabla, title="Informe", padding=(1, 2)))     # Panel con una tabla dentro
```

### 7.2 Columnas

```python
from rich.columns import Columns

tarjetas = [
    Panel(f"[bold]{valor}[/]\n[dim]{titulo}", expand=True)
    for titulo, valor in [("Ventas", "48.900 €"), ("Pedidos", "366"), ("Clientes", "214")]
]
console.print(Columns(tarjetas, equal=True, expand=True))
```

### 7.3 Alineación y espaciado

```python
from rich.align import Align
from rich.padding import Padding

console.print(Align.center("[bold]Título centrado[/]"))
console.print(Padding("Texto con margen", (1, 4)))
```

### 7.4 Reglas (separadores)

```python
console.rule()
console.rule("[bold blue]Resultados", style="blue")
console.rule("Fin", align="left")
```

### 7.5 Diseño de varias zonas (`Layout`)

```python
from rich.layout import Layout

pantalla = Layout()
pantalla.split_column(Layout(name="cabecera", size=3), Layout(name="cuerpo"), Layout(name="pie", size=3))
pantalla["cuerpo"].split_row(Layout(name="izquierda"), Layout(name="derecha", ratio=2))

pantalla["cabecera"].update(Panel(Align.center("[bold]Panel de control")))
pantalla["izquierda"].update(Panel("Menú"))
pantalla["derecha"].update(Panel(tabla))
pantalla["pie"].update(Panel("[dim]Pulse Ctrl+C para salir"))
console.print(pantalla)
```

---

## 8. Barras de progreso y estados

### 8.1 Progreso sencillo con `track`

```python
import time
from rich.progress import track

for archivo in track(range(50), description="Procesando archivos..."):
    time.sleep(0.05)
```

### 8.2 Varias tareas simultáneas

```python
from rich.progress import Progress

with Progress() as progreso:
    descarga = progreso.add_task("[cyan]Descargando...", total=100)
    proceso = progreso.add_task("[green]Procesando...", total=200)

    while not progreso.finished:
        progreso.update(descarga, advance=1.5)
        progreso.update(proceso, advance=1)
        time.sleep(0.02)
```

### 8.3 Personalizar las columnas de la barra

```python
from rich.progress import (
    Progress, SpinnerColumn, TextColumn, BarColumn, TaskProgressColumn,
    MofNCompleteColumn, TimeElapsedColumn, TimeRemainingColumn,
    DownloadColumn, TransferSpeedColumn,
)

progreso = Progress(
    SpinnerColumn(),
    TextColumn("[bold blue]{task.description}"),
    BarColumn(bar_width=40),
    MofNCompleteColumn(),
    TaskProgressColumn(),
    TimeElapsedColumn(),
    TimeRemainingColumn(),
)
```

Para descargas: `DownloadColumn()` y `TransferSpeedColumn()`.

### 8.4 Tareas de duración desconocida

```python
with Progress(SpinnerColumn(), TextColumn("{task.description}")) as p:
    p.add_task("Conectando con el servidor...", total=None)
    time.sleep(3)
```

### 8.5 Indicador de estado (*spinner*)

```python
with console.status("[bold green]Consultando la base de datos...", spinner="dots"):
    time.sleep(2)
console.print("[green]✔[/] Consulta completada")
```

Ver los *spinners* disponibles: `python -m rich.spinner`.

### 8.6 Actualización en vivo (`Live`)

```python
from rich.live import Live
import random

def generar_tabla() -> Table:
    t = Table(title="Monitor de servidores")
    t.add_column("Servidor")
    t.add_column("CPU", justify="right")
    t.add_column("Estado")
    for nombre in ("web-01", "web-02", "bd-01"):
        cpu = random.randint(5, 99)
        estado = "[red]ALTO" if cpu > 85 else "[green]OK"
        t.add_row(nombre, f"{cpu} %", estado)
    return t

with Live(generar_tabla(), refresh_per_second=4) as vivo:
    for _ in range(20):
        time.sleep(0.5)
        vivo.update(generar_tabla())
```

---

## 9. Contenido especializado

### 9.1 Árboles

```python
from rich.tree import Tree

arbol = Tree("📁 [bold]proyecto")
src = arbol.add("📁 src")
src.add("🐍 main.py")
src.add("🐍 config.py")
tests = arbol.add("📁 tests")
tests.add("🐍 test_main.py")
arbol.add("📄 README.md")
console.print(arbol)
```

Mostrar una carpeta real:

```python
from pathlib import Path

def arbol_carpeta(ruta: Path, nodo: Tree | None = None) -> Tree:
    nodo = nodo or Tree(f"📁 [bold]{ruta.name}")
    for elem in sorted(ruta.iterdir(), key=lambda p: (p.is_file(), p.name.lower())):
        if elem.name.startswith("."):
            continue
        if elem.is_dir():
            arbol_carpeta(elem, nodo.add(f"📁 {elem.name}"))
        else:
            nodo.add(f"📄 {elem.name} [dim]({elem.stat().st_size / 1024:.1f} KB)")
    return nodo

console.print(arbol_carpeta(Path(".")))
```

### 9.2 Código con resaltado de sintaxis

```python
from rich.syntax import Syntax

codigo = '''
def saludar(nombre: str) -> str:
    return f"Hola, {nombre}"
'''
console.print(Syntax(codigo, "python", theme="monokai", line_numbers=True))
console.print(Syntax.from_path("config.yaml", line_numbers=True))
```

### 9.3 Markdown

```python
from rich.markdown import Markdown

md = """
# Informe de ejecución
- **Archivos procesados:** 120
- **Errores:** 2

> Revise los registros para más detalles.
"""
console.print(Markdown(md))
```

### 9.4 JSON

```python
from rich.json import JSON
console.print(JSON('{"pedido": 1045, "lineas": [{"art": "A", "uds": 2}]}'))
```

### 9.5 Formateo de objetos (`pretty`)

```python
from rich.pretty import pprint
pprint(objeto_grande, max_length=5, expand_all=True)
```

---

## 10. Entrada de datos

```python
from rich.prompt import Prompt, IntPrompt, FloatPrompt, Confirm

nombre = Prompt.ask("Nombre del cliente")
entorno = Prompt.ask("Entorno", choices=["desarrollo", "pruebas", "produccion"], default="desarrollo")
clave = Prompt.ask("Contraseña", password=True)
unidades = IntPrompt.ask("Unidades", default=1)
precio = FloatPrompt.ask("Precio")
continuar = Confirm.ask("¿Desea continuar?", default=True)
```

`IntPrompt` y `FloatPrompt` repiten la pregunta automáticamente si la entrada no es válida.

> `Confirm` espera por defecto las respuestas `y` / `n`. Para usar `s` / `n` en español, cree una subclase:
>
> ```python
> class Confirmar(Confirm):
>     choices = ["s", "n"]
>     validate_error_message = "[prompt.invalid]Responda s o n"
>
>     def process_response(self, value: str) -> bool:
>         value = value.strip().lower()
>         if value not in self.choices:
>             raise InvalidResponse(self.validate_error_message)
>         return value == "s"
> ```
>
> (requiere `from rich.prompt import InvalidResponse`).

---

## 11. Operaciones avanzadas

### 11.1 Trazas de error mejoradas

```python
from rich.traceback import install
install(show_locals=True)            # Todas las excepciones no capturadas se mostrarán con formato
```

Dentro de un `except`:

```python
try:
    1 / 0
except ZeroDivisionError:
    console.print_exception(show_locals=False)
```

### 11.2 Integración con `logging`

```python
import logging
from rich.logging import RichHandler

logging.basicConfig(
    level="INFO",
    format="%(message)s",
    datefmt="[%X]",
    handlers=[RichHandler(rich_tracebacks=True, markup=True)],
)
log = logging.getLogger("app")

log.info("Servidor iniciado en el puerto [bold]8000[/]")
log.warning("Uso de memoria elevado")
log.error("No se pudo conectar con la base de datos")
```

### 11.3 Exportar la salida (HTML, SVG, texto)

```python
console = Console(record=True)
console.print(tabla)
console.save_html("informe.html")
console.save_svg("informe.svg", title="Informe de ventas")
console.save_text("informe.txt")
```

### 11.4 Escribir en archivo o capturar la salida

```python
with open("salida.txt", "w", encoding="utf-8") as f:
    Console(file=f, width=100).print(tabla)

with console.capture() as captura:
    console.print("[bold]Texto")
texto = captura.get()
```

### 11.5 Consola de errores y ancho fijo

```python
error_console = Console(stderr=True, style="bold red")
error_console.print("Error crítico")

Console(width=80)                     # Ancho fijo (útil en informes)
Console(force_terminal=True)          # Forzar colores (CI, Docker)
Console(no_color=True)                # Sin colores
```

### 11.6 Paginador y pantalla alternativa

```python
with console.pager(styles=True):
    console.print(tabla_larga)

with console.screen():
    console.print(Align.center("Pantalla completa"), justify="center")
    time.sleep(2)
```

### 11.7 Herramientas relacionadas

- **Typer**: crea CLIs con argumentos y opciones; usa Rich para la ayuda y los errores.
- **Textual**: framework para aplicaciones de terminal completas (TUI), del mismo autor.
- **rich-cli**: `pip install rich-cli` → `rich archivo.md`, `rich datos.csv`, `rich script.py`.

---

## 12. Ejemplo práctico completo

Herramienta de análisis de un CSV de ventas con resumen, tabla, barras de progreso, avisos y exportación del informe a HTML.

```python
import csv
import random
import time
from collections import defaultdict
from datetime import date, timedelta
from pathlib import Path

from rich import box
from rich.columns import Columns
from rich.console import Console
from rich.panel import Panel
from rich.progress import BarColumn, MofNCompleteColumn, Progress, SpinnerColumn, TextColumn
from rich.prompt import Confirm
from rich.table import Table
from rich.theme import Theme
from rich.traceback import install

install()
console = Console(record=True, theme=Theme({
    "ok": "bold green", "aviso": "bold yellow", "error": "bold red", "dato": "bold cyan",
}))

ARCHIVO = Path("ventas_demo.csv")


def euros(v: float) -> str:
    return f"{v:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def crear_datos_demo(filas: int = 400) -> None:
    rng = random.Random(42)
    regiones = ["Norte", "Sur", "Este", "Oeste"]
    productos = {"Teclado": 29.9, "Ratón": 14.5, "Monitor": 189.0, "Cable HDMI": 9.9, "Webcam": 59.0}
    inicio = date(2026, 10, 1)
    with ARCHIVO.open("w", newline="", encoding="utf-8") as f:
        w = csv.writer(f)
        w.writerow(["fecha", "region", "producto", "unidades", "precio"])
        for _ in range(filas):
            prod = rng.choice(list(productos))
            unidades = rng.choice([1, 1, 2, 3, 5, 0, -1])            # Incluye datos erróneos
            w.writerow([inicio + timedelta(days=rng.randint(0, 54)), rng.choice(regiones),
                        prod, unidades, productos[prod]])


def analizar() -> tuple[dict, dict, list[str]]:
    por_region = defaultdict(lambda: {"pedidos": 0, "unidades": 0, "importe": 0.0})
    por_producto = defaultdict(float)
    avisos: list[str] = []

    filas = list(csv.DictReader(ARCHIVO.open(encoding="utf-8")))
    with Progress(SpinnerColumn(), TextColumn("[bold blue]{task.description}"),
                  BarColumn(), MofNCompleteColumn(), console=console, transient=True) as p:
        tarea = p.add_task("Analizando registros", total=len(filas))
        for num, fila in enumerate(filas, start=2):
            p.advance(tarea)
            time.sleep(0.002)                                       # Simula trabajo
            uds = int(fila["unidades"])
            if uds <= 0:
                avisos.append(f"Línea {num}: unidades no válidas ({uds}) en {fila['producto']}")
                continue
            importe = uds * float(fila["precio"])
            r = por_region[fila["region"]]
            r["pedidos"] += 1
            r["unidades"] += uds
            r["importe"] += importe
            por_producto[fila["producto"]] += importe
    return por_region, por_producto, avisos


def mostrar(por_region: dict, por_producto: dict, avisos: list[str]) -> None:
    total = sum(r["importe"] for r in por_region.values())
    pedidos = sum(r["pedidos"] for r in por_region.values())

    console.rule("[bold]Informe de ventas · Oct–Nov 2026")

    # Tarjetas KPI
    kpis = [
        ("Facturación", euros(total), "green"),
        ("Pedidos válidos", f"{pedidos}", "cyan"),
        ("Ticket medio", euros(total / pedidos if pedidos else 0), "magenta"),
        ("Registros con error", f"{len(avisos)}", "red" if avisos else "green"),
    ]
    console.print(Columns(
        [Panel(f"[bold {c}]{v}[/]\n[dim]{t}", expand=True, border_style=c) for t, v, c in kpis],
        equal=True, expand=True,
    ))

    # Tabla por región
    t = Table(title="Ventas por región", box=box.ROUNDED, header_style="bold white on #1F4E78",
              row_styles=["", "dim"], show_footer=True)
    t.add_column("Región", footer="TOTAL", style="dato")
    t.add_column("Pedidos", justify="right", footer=str(pedidos))
    t.add_column("Unidades", justify="right",
                 footer=str(sum(r["unidades"] for r in por_region.values())))
    t.add_column("Importe", justify="right", footer=euros(total), style="ok")
    t.add_column("Cuota", justify="left")
    for region, r in sorted(por_region.items(), key=lambda x: x[1]["importe"], reverse=True):
        cuota = r["importe"] / total
        barra = "█" * round(cuota * 40)
        t.add_row(region, str(r["pedidos"]), str(r["unidades"]), euros(r["importe"]),
                  f"[blue]{barra}[/] {cuota:.0%}")
    console.print(t)

    # Ranking de productos
    ranking = Table(title="Ranking de productos", box=box.SIMPLE_HEAVY)
    ranking.add_column("#", justify="right", style="dim")
    ranking.add_column("Producto")
    ranking.add_column("Importe", justify="right")
    for i, (prod, imp) in enumerate(sorted(por_producto.items(), key=lambda x: x[1], reverse=True), 1):
        medalla = {1: "🥇", 2: "🥈", 3: "🥉"}.get(i, "")
        ranking.add_row(str(i), f"{medalla} {prod}", euros(imp))
    console.print(ranking)

    # Avisos
    if avisos:
        detalle = "\n".join(f"• {a}" for a in avisos[:8])
        if len(avisos) > 8:
            detalle += f"\n[dim]… y {len(avisos) - 8} más[/]"
        console.print(Panel(detalle, title="[aviso]⚠ Registros descartados", border_style="yellow"))
    else:
        console.print("[ok]✔ Todos los registros son válidos")


def main() -> None:
    if not ARCHIVO.exists():
        with console.status("[bold green]Generando datos de ejemplo..."):
            crear_datos_demo()
            time.sleep(0.5)
        console.print(f"[ok]✔[/] Datos de ejemplo creados en [dato]{ARCHIVO}[/]")

    mostrar(*analizar())

    if Confirm.ask("¿Exportar el informe a HTML?", default=True):
        console.save_html("informe_ventas.html")
        console.print("[ok]✔[/] Informe guardado en [dato]informe_ventas.html[/]")


if __name__ == "__main__":
    main()
```

---

## 13. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| Texto entre corchetes desaparece | Rich lo interpreta como marcado. | Use `escape()` o `markup=False`. |
| `MarkupError: closing tag ... doesn't match` | Etiquetas mal cerradas. | Revise las etiquetas o use `[/]`. |
| Se ven códigos como `←[32m` | Terminal antigua de Windows. | Use Windows Terminal o PowerShell moderno. |
| Sin colores en CI o al redirigir a archivo | Rich detecta que no es una terminal. | Use `Console(force_terminal=True)`. |
| La barra de progreso se rompe con `print()` | Se mezcla `print` normal con `Progress`. | Use `progress.console.print()` o `console.log()`. |
| Emojis mostrados como cuadrados | La fuente de la terminal no los admite. | Cambie de fuente o evite emojis. |
| La tabla se corta | Terminal estrecha. | Use `overflow="fold"`, `max_width` o reduzca columnas. |
| `print` de Rich sustituye al nativo | `from rich import print`. | Importe solo `Console` si quiere mantener el `print` original. |

---

## 14. Buenas prácticas

- Cree **una única** instancia de `Console` y reutilícela en todo el programa.
- Defina un **tema** con estilos semánticos (`ok`, `aviso`, `error`) en lugar de colores sueltos.
- Escape siempre el texto procedente del usuario o de archivos.
- No abuse de colores y emojis: priorice la legibilidad.
- Use `track`/`Progress` en procesos largos para informar del avance.
- Use `RichHandler` para que los logs sean legibles en desarrollo.
- Asegúrese de que la salida sigue siendo útil sin colores (archivos, CI).

---

## 15. Referencia rápida

```python
from rich import print, inspect
console = Console(theme=..., record=True)
console.print("[bold red]texto[/]", style="...", justify="center")
console.log(...) / console.rule("Título") / console.print_json(...) / console.status("...")

Table(title, box=box.ROUNDED, header_style, show_footer); .add_column(...); .add_row(...)
Table.grid(padding=(0, 2))
Panel(contenido, title, border_style) / Panel.fit(...)
Columns([...], equal=True) / Layout() / Align.center(...)
track(iterable, description) / Progress(...) / Live(renderable)
Tree("raíz").add("hijo") / Syntax(codigo, "python") / Markdown(texto) / JSON(texto)
Prompt.ask / IntPrompt.ask / Confirm.ask
install()  # trazas  ·  RichHandler()  # logging
console.save_html / save_svg / save_text
escape(texto)
```

---

## 16. Ejercicios propuestos

1. Muestre el contenido de un CSV en una tabla con colores según el valor de una columna.
2. Cree un script de copia de seguridad que muestre una barra de progreso por archivo y un resumen final en un panel.
3. Construya un monitor en tiempo real con `Live` que muestre el uso de CPU y memoria (use la librería `psutil`).
4. Represente como árbol la estructura de carpetas de un proyecto, excluyendo archivos ocultos y mostrando tamaños.
5. Configure `logging` con `RichHandler` en un proyecto existente.

---

## 17. Referencias

- Documentación oficial: https://rich.readthedocs.io/
- Repositorio: https://github.com/Textualize/rich
- Textual: https://textual.textualize.io/
