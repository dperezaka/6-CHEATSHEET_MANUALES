# Manual de uso: Rich

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Formatear la salida en terminal y pedir datos básicos
> **Tipo:** Librería de terceros (`pip install rich`) · **Versión de referencia:** 14-15

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [La consola y `print`](#4-la-consola-y-print)
5. [Marcado, estilos y temas](#5-marcado-estilos-y-temas)
6. [Texto (`Text`)](#6-texto-text)
7. [Tablas](#7-tablas)
8. [Paneles, reglas, columnas y alineación](#8-paneles-reglas-columnas-y-alineación)
9. [Árboles](#9-árboles)
10. [Código, Markdown y JSON](#10-código-markdown-y-json)
11. [Estructuras de Python e inspección](#11-estructuras-de-python-e-inspección)
12. [Tracebacks y logging](#12-tracebacks-y-logging)
13. [Barras de progreso](#13-barras-de-progreso)
14. [Indicador de estado (`status`)](#14-indicador-de-estado-status)
15. [Pantallas en vivo (`Live`) y `Layout`](#15-pantallas-en-vivo-live-y-layout)
16. [Pedir datos: `Prompt`, `Confirm`, `IntPrompt`](#16-pedir-datos-prompt-confirm-intprompt)
17. [Exportar a HTML, SVG y texto](#17-exportar-a-html-svg-y-texto)
18. [Objetos personalizados con `__rich__`](#18-objetos-personalizados-con-__rich__)
19. [Pruebas y entornos sin terminal](#19-pruebas-y-entornos-sin-terminal)
20. [Ejemplo práctico completo](#20-ejemplo-práctico-completo)
21. [Errores frecuentes](#21-errores-frecuentes)
22. [Buenas prácticas](#22-buenas-prácticas)
23. [Referencia rápida](#23-referencia-rápida)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Referencias](#25-referencias)

---

## 1. Descripción

**Rich** es una librería para producir salida enriquecida en la terminal: colores, estilos, tablas, paneles, árboles, código con resaltado, Markdown, barras de progreso, indicadores de actividad, pantallas que se actualizan en vivo y tracebacks legibles. Incluye también preguntas básicas (`Prompt`, `Confirm`, `IntPrompt`). Typer la usa para su ayuda y sus mensajes de error.

**Ventajas:**
- Sintaxis de marcado muy sencilla: `"[bold red]Error[/]"`.
- Gran variedad de componentes listos para usar.
- Detecta las capacidades de la terminal (colores, ancho, Unicode) y se adapta.
- Funciona en Windows, macOS, Linux y Jupyter.
- Tracebacks y logging con formato muy legible.
- Exportación de la salida a HTML, SVG o texto.

**Limitaciones:**
- No es un framework de aplicaciones interactivas: para TUIs completas, use **Textual** (del mismo autor).
- Las preguntas son básicas (sin menús con flechas): para eso, questionary o InquirerPy.
- Añade un pequeño coste de importación en scripts muy breves.

| Si necesita… | Opción |
|---|---|
| Salida con formato, tablas, progreso, logs | **Rich** |
| Barra de progreso mínima en un bucle | tqdm |
| Menús y formularios interactivos | questionary / InquirerPy |
| Aplicación de terminal interactiva completa | Textual |
| Solo colores ANSI multiplataforma | colorama |

---

## 2. Instalación e importación

```bash
pip install rich
python -m rich                  # Demostración de capacidades en su terminal
```

Importaciones habituales:

```python
from rich import print
from rich.console import Console
from rich.table import Table
from rich.panel import Panel
from rich.progress import Progress, track
from rich.prompt import Prompt, Confirm, IntPrompt
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Console` | Objeto central: imprime, mide la terminal, gestiona estilos y exportación. |
| Marcado (*markup*) | Etiquetas entre corchetes dentro del texto: `[bold]`, `[red]`, `[link=…]`. |
| Estilo | Combinación de color, fondo y atributos: `"bold white on red"`. |
| *Renderable* | Cualquier objeto que Rich sabe dibujar: `Table`, `Panel`, `Text`, `Tree`… |
| Tema | Diccionario de nombres de estilo reutilizables (`"error"`, `"ok"`). |
| `Live` | Zona de la pantalla que se redibuja periódicamente. |
| Resaltado automático | Rich colorea números, rutas, URLs, booleanos, etc. al imprimir. |

---

## 4. La consola y `print`

### 4.1 `print` de Rich

```python
from rich import print

print("Hola, [bold magenta]mundo[/bold magenta]! :rocket:")
print({"nombre": "Ana", "edad": 34, "activo": True})        # Estructuras con formato y color
print([1, 2, 3], "texto", 3.14)
```

### 4.2 `Console` (recomendado en aplicaciones)

```python
from rich.console import Console

consola = Console()
consola_error = Console(stderr=True, style="bold red")

consola.print("Proceso [green]completado[/]", justify="center")
consola.print("Texto largo…", overflow="ellipsis", no_wrap=True)
consola.print("[dim]Sin resaltado automático[/]", highlight=False)
consola.print("Literal con [corchetes]", markup=False)
consola_error.print("No se encontró el archivo")

consola.log("Mensaje con hora y archivo:línea")      # Útil para depuración
consola.rule("[bold]Sección[/]")                      # Línea horizontal con título
consola.print_json('{"a": 1, "b": [1, 2]}')
consola.input("Nombre: ")                            # Como input(), admite marcado
```

| Parámetro de `Console` | Efecto |
|---|---|
| `stderr=True` | Escribe en la salida de error. |
| `width`, `height` | Fija las dimensiones (útil en pruebas o ficheros). |
| `force_terminal=True` | Fuerza colores aunque la salida no sea una terminal. |
| `no_color=True` | Desactiva colores (también con la variable `NO_COLOR`). |
| `color_system` | `"auto"`, `"standard"`, `"256"`, `"truecolor"`, `None`. |
| `theme` | Tema de estilos propio. |
| `record=True` | Guarda la salida para exportarla. |
| `file` | Escribir en un archivo u otro flujo. |
| `highlight`, `markup`, `emoji` | Activan o desactivan funciones automáticas. |
| `quiet=True` | No imprime nada. |

### 4.3 Información de la terminal

```python
consola.width, consola.height
consola.is_terminal            # False si se redirige a un archivo o tubería
consola.color_system
consola.clear()
consola.bell()
```

### 4.4 Paginador y pantalla alternativa

```python
with consola.pager(styles=True):
    consola.print(texto_muy_largo)

with consola.screen():          # Pantalla alternativa: al salir se restaura la original
    consola.print("Pantalla temporal")
    time.sleep(2)
```

---

## 5. Marcado, estilos y temas

### 5.1 Sintaxis de marcado

```python
consola.print("[bold]negrita[/bold] [italic]cursiva[/italic] [underline]subrayado[/]")
consola.print("[red]rojo[/] [#FF8800]naranja hex[/] [rgb(0,128,255)]azul rgb[/] [color(208)]256 colores[/]")
consola.print("[white on red] fondo [/] [bold yellow on blue] combinado [/]")
consola.print("[strike]tachado[/] [dim]tenue[/] [reverse]invertido[/] [blink]parpadeo[/]")
consola.print("[link=https://rich.readthedocs.io]Documentación[/link]")
consola.print(":white_check_mark: :warning: :x: :rocket:")             # Emojis por nombre
```

- `[/]` cierra la última etiqueta abierta.
- Para escribir un corchete literal: `\[texto]` o `rich.markup.escape(texto)`.

```python
from rich.markup import escape
consola.print(f"Usuario: [bold]{escape(nombre_usuario)}[/]")     # Evita que '[...]' del usuario se interprete
```

### 5.2 Estilos como parámetro

```python
consola.print("Error grave", style="bold white on red")
consola.print("Aviso", style="yellow italic")
```

Formato: `[atributos] [color] [on color_fondo]`. Atributos: `bold`, `dim`, `italic`, `underline`, `strike`, `reverse`, `blink`, `conceal`, `overline`. Negación: `not bold`.

### 5.3 Temas

```python
from rich.theme import Theme

tema = Theme({
    "info": "cyan",
    "ok": "bold green",
    "aviso": "bold yellow",
    "error": "bold white on red",
    "dato": "magenta",
    "repr.number": "bold bright_blue",          # Sobrescribe estilos internos de Rich
})
consola = Console(theme=tema)

consola.print("[ok]✔ Copia completada[/ok]")
consola.print("Registros procesados: [dato]1.250[/]")
consola.print("Fallo de conexión", style="error")
```

Lista de colores disponibles: `python -m rich.color`. Estilos predefinidos: `python -m rich.theme`.

---

## 6. Texto (`Text`)

`Text` permite construir texto con estilos por tramos, sin marcado:

```python
from rich.text import Text

t = Text("Estado: ")
t.append("ACTIVO", style="bold green")
t.append(" desde 2026-11-01", style="dim")
consola.print(t)

t = Text.from_markup("[bold]Total:[/] 1.250 €")
t = Text("Error en línea 42 del archivo config.toml")
t.highlight_words(["Error"], style="bold red")
t.highlight_regex(r"\d+", style="cyan")
t.stylize("underline", 0, 5)                    # Estilo por posición
consola.print(t)

Text("Centrado", justify="center")
Text("Texto muy largo…", overflow="ellipsis", no_wrap=True)
```

---

## 7. Tablas

### 7.1 Tabla básica

```python
from rich.table import Table

tabla = Table(title="Ventas por región", caption="Datos de noviembre de 2026")
tabla.add_column("Región", style="cyan", no_wrap=True)
tabla.add_column("Pedidos", justify="right")
tabla.add_column("Importe", justify="right", style="green")

tabla.add_row("Norte", "1.204", "45.320,50 €")
tabla.add_row("Sur", "980", "38.110,00 €")
tabla.add_row("Este", "1.502", "[bold]61.900,75 €[/]")

consola.print(tabla)
```

### 7.2 Opciones de la tabla

```python
from rich import box

tabla = Table(
    title="Inventario",
    box=box.ROUNDED,                 # SIMPLE, MINIMAL, HEAVY, DOUBLE, SQUARE, MARKDOWN, None…
    show_header=True,
    header_style="bold white on #1F4E78",
    show_lines=False,                # Línea entre filas
    row_styles=["", "on grey11"],    # Filas alternas
    show_footer=True,
    expand=True,                     # Ocupa todo el ancho
    highlight=True,
    padding=(0, 1),
)
tabla.add_column("Producto", footer="Total", ratio=3)
tabla.add_column("Stock", justify="right", footer="1.240", min_width=6)
tabla.add_column("Estado", justify="center", max_width=12)

tabla.add_row("Teclado", "120", "[green]OK[/]")
tabla.add_row("Ratón", "4", "[yellow]Bajo[/]", style="italic")
tabla.add_section()                  # Separador de sección
tabla.add_row("Monitor", "0", "[red]Agotado[/]", end_section=True)
```

| Parámetro de `add_column` | Efecto |
|---|---|
| `justify` | `left`, `center`, `right`, `full`. |
| `style`, `header_style`, `footer_style` | Estilos de la columna. |
| `width`, `min_width`, `max_width`, `ratio` | Control del ancho. |
| `no_wrap`, `overflow` | `fold`, `crop`, `ellipsis`. |
| `vertical` | `top`, `middle`, `bottom`. |

### 7.3 Tablas a partir de datos

```python
def tabla_desde(registros: list[dict], titulo: str = "") -> Table:
    tabla = Table(title=titulo, header_style="bold cyan")
    for columna in registros[0]:
        tabla.add_column(columna.capitalize(),
                         justify="right" if isinstance(registros[0][columna], (int, float)) else "left")
    for r in registros:
        tabla.add_row(*(f"{v:,.2f}" if isinstance(v, float) else str(v) for v in r.values()))
    return tabla
```

### 7.4 Tabla como cuadrícula (sin bordes)

```python
cuadricula = Table.grid(expand=True, padding=(0, 2))
cuadricula.add_column(justify="left")
cuadricula.add_column(justify="right")
cuadricula.add_row("[bold]Servidor[/]", "srv-01")
cuadricula.add_row("[bold]Estado[/]", "[green]operativo[/]")
```

> Las celdas admiten cualquier *renderable*: texto, otras tablas, paneles, barras…

---

## 8. Paneles, reglas, columnas y alineación

```python
from rich.panel import Panel
from rich.columns import Columns
from rich.align import Align
from rich.padding import Padding
from rich.rule import Rule
from rich.console import Group

consola.print(Panel("Contenido del panel", title="Título", subtitle="v1.0",
                    border_style="blue", padding=(1, 2), expand=False))
consola.print(Panel.fit("[bold]Ajustado al contenido[/]", border_style="green"))

consola.print(Rule("Resultados", style="cyan", align="left"))

tarjetas = [Panel(f"[bold]{n}[/]\n{v}", expand=True) for n, v in
            [("Usuarios", "1.204"), ("Pedidos", "3.950"), ("Ingresos", "98.210 €")]]
consola.print(Columns(tarjetas, equal=True, expand=True))

consola.print(Align.center("[bold]Texto centrado[/]", vertical="middle"))
consola.print(Padding("Con margen", (1, 4), style="on grey15"))

# Agrupar varios elementos en uno solo (p. ej., dentro de un panel)
consola.print(Panel(Group(Rule("Parte 1"), "Texto", Rule("Parte 2"), tabla), title="Informe"))
```

---

## 9. Árboles

```python
from rich.tree import Tree

arbol = Tree("[bold]mi_proyecto/[/]", guide_style="dim")
src = arbol.add("📁 src", style="cyan")
src.add("main.py")
modulos = src.add("📁 modulos")
modulos.add("datos.py").add("[dim]3 clases, 12 funciones[/]")
arbol.add("📄 README.md")
arbol.add("📄 pyproject.toml")
consola.print(arbol)
```

Árbol de una carpeta real:

```python
from pathlib import Path

def arbol_carpeta(ruta: Path, nodo: Tree | None = None, profundidad: int = 2) -> Tree:
    nodo = nodo or Tree(f"[bold]{ruta.name}/[/]")
    if profundidad == 0:
        return nodo
    for hijo in sorted(ruta.iterdir(), key=lambda p: (p.is_file(), p.name.lower())):
        if hijo.name.startswith("."):
            continue
        if hijo.is_dir():
            arbol_carpeta(hijo, nodo.add(f"[cyan]{hijo.name}/[/]"), profundidad - 1)
        else:
            nodo.add(f"{hijo.name} [dim]({hijo.stat().st_size:,} B)[/]")
    return nodo
```

---

## 10. Código, Markdown y JSON

```python
from rich.syntax import Syntax
from rich.markdown import Markdown
from rich.json import JSON

codigo = '''def saludar(nombre: str) -> str:
    return f"Hola, {nombre}"'''
consola.print(Syntax(codigo, "python", theme="monokai", line_numbers=True,
                     highlight_lines={2}, word_wrap=True))
consola.print(Syntax.from_path("config.toml", line_numbers=True))

consola.print(Markdown("""
# Informe
- **Estado:** correcto
- Ver [documentación](https://rich.readthedocs.io)

```python
print("bloques de código con resaltado")
```
"""))

consola.print(JSON('{"usuario": "ana", "roles": ["admin", "editor"], "activo": true}'))
consola.print(JSON.from_data({"a": 1, "b": None}, indent=4))
```

---

## 11. Estructuras de Python e inspección

```python
from rich.pretty import pprint, Pretty
from rich import inspect

pprint(configuracion, expand_all=True, max_length=10)       # Listas largas truncadas
consola.print(Panel(Pretty(configuracion), title="Configuración"))

inspect(objeto, methods=True)            # Atributos, métodos y documentación de un objeto
inspect(str, methods=True, help=True)
```

Instalar el *pretty printer* en el intérprete interactivo:

```python
from rich import pretty
pretty.install()
```

---

## 12. Tracebacks y logging

### 12.1 Tracebacks

```python
from rich.traceback import install

install(show_locals=True, width=120, suppress=[click])   # Al principio del programa
```

Todas las excepciones no capturadas se muestran con código resaltado y, opcionalmente, variables locales. Para mostrar una excepción capturada:

```python
try:
    procesar()
except Exception:
    consola.print_exception(show_locals=False, max_frames=5)
```

> Desactive `show_locals` en producción si las variables pueden contener datos sensibles.

### 12.2 Logging

```python
import logging
from rich.logging import RichHandler

logging.basicConfig(
    level="INFO",
    format="%(message)s",
    datefmt="[%X]",
    handlers=[RichHandler(rich_tracebacks=True, markup=True, show_path=False)],
)
log = logging.getLogger("app")

log.info("Conectando con [bold]db01[/]…")
log.warning("Respuesta lenta: %.1f s", 3.2)
log.error("Fallo al guardar", exc_info=True)
```

Combinar log en consola (Rich) y en archivo (texto plano):

```python
archivo = logging.FileHandler("app.log", encoding="utf-8")
archivo.setFormatter(logging.Formatter("%(asctime)s %(levelname)s %(name)s: %(message)s"))
logging.basicConfig(level="INFO", handlers=[RichHandler(), archivo], format="%(message)s")
```

---

## 13. Barras de progreso

### 13.1 `track` (lo más sencillo)

```python
from rich.progress import track

for archivo in track(archivos, description="Procesando…"):
    procesar(archivo)
```

### 13.2 `Progress` con varias tareas

```python
from rich.progress import Progress

with Progress() as progreso:
    descarga = progreso.add_task("[cyan]Descargando", total=1000)
    proceso = progreso.add_task("[green]Procesando", total=500)
    while not progreso.finished:
        progreso.update(descarga, advance=7)
        progreso.update(proceso, advance=3)
        time.sleep(0.01)
```

### 13.3 Columnas personalizadas

```python
from rich.progress import (Progress, SpinnerColumn, TextColumn, BarColumn, TaskProgressColumn,
                           MofNCompleteColumn, TimeElapsedColumn, TimeRemainingColumn,
                           DownloadColumn, TransferSpeedColumn)

progreso = Progress(
    SpinnerColumn(),
    TextColumn("[bold blue]{task.description}"),
    BarColumn(bar_width=None),
    MofNCompleteColumn(),
    TaskProgressColumn(),
    TimeElapsedColumn(),
    TextColumn("·"),
    TimeRemainingColumn(),
    transient=True,                     # La barra desaparece al terminar
    expand=True,
)
```

Descargas con tamaño y velocidad:

```python
with Progress(TextColumn("{task.fields[archivo]}"), BarColumn(), DownloadColumn(),
              TransferSpeedColumn(), TimeRemainingColumn()) as progreso:
    tarea = progreso.add_task("descarga", total=tamano_total, archivo="datos.zip")
    for bloque in respuesta.iter_bytes(65536):
        destino.write(bloque)
        progreso.update(tarea, advance=len(bloque))
```

### 13.4 Total desconocido, campos extra y archivos

```python
tarea = progreso.add_task("Buscando…", total=None)            # Barra indeterminada (pulsante)
progreso.update(tarea, total=250)                               # Fijar el total cuando se conozca
progreso.update(tarea, description="Analizando", completed=10)
progreso.add_task("Lote", total=100, servidor="srv-01")        # Campo extra: {task.fields[servidor]}
progreso.console.print("Mensaje que no rompe las barras")      # Imprimir mientras hay barras

from rich.progress import open as progreso_open
with progreso_open("grande.csv", "rb", description="Leyendo") as f:
    datos = f.read()
```

### 13.5 Progreso con hilos

```python
from concurrent.futures import ThreadPoolExecutor

with Progress() as progreso:
    def trabajo(nombre, n):
        tarea = progreso.add_task(nombre, total=n)
        for _ in range(n):
            time.sleep(0.01)
            progreso.advance(tarea)

    with ThreadPoolExecutor(max_workers=4) as ex:
        for i in range(4):
            ex.submit(trabajo, f"Lote {i}", 100 + i * 50)
```

> `Progress` es seguro entre hilos.

---

## 14. Indicador de estado (`status`)

```python
with consola.status("[bold green]Conectando con el servidor…", spinner="dots") as estado:
    conectar()
    estado.update("[bold green]Descargando datos…")
    descargar()
    consola.log("Datos descargados")         # Se puede imprimir mientras gira
consola.print("[green]✔ Listo[/]")
```

Spinners disponibles: `python -m rich.spinner` (`dots`, `line`, `arc`, `bouncingBar`, `earth`, `moon`…).

---

## 15. Pantallas en vivo (`Live`) y `Layout`

### 15.1 `Live`

Redibuja un elemento sin desplazar la pantalla:

```python
from rich.live import Live

def generar_tabla(datos) -> Table:
    tabla = Table(title="Servidores")
    tabla.add_column("Nombre")
    tabla.add_column("CPU", justify="right")
    for nombre, cpu in datos.items():
        color = "red" if cpu > 80 else "yellow" if cpu > 50 else "green"
        tabla.add_row(nombre, f"[{color}]{cpu:.0f} %[/]")
    return tabla

with Live(generar_tabla(medir()), refresh_per_second=4, console=consola) as live:
    for _ in range(40):
        time.sleep(0.25)
        live.update(generar_tabla(medir()))
```

| Parámetro | Efecto |
|---|---|
| `refresh_per_second` | Frecuencia de refresco. |
| `transient=True` | Borra el contenido al salir. |
| `screen=True` | Usa la pantalla completa (pantalla alternativa). |
| `auto_refresh=False` | Refrescar solo con `live.refresh()` o `live.update(..., refresh=True)`. |
| `vertical_overflow` | `"visible"`, `"crop"`, `"ellipsis"`. |

### 15.2 `Layout`

Divide la pantalla en regiones:

```python
from rich.layout import Layout

layout = Layout()
layout.split_column(
    Layout(name="cabecera", size=3),
    Layout(name="cuerpo", ratio=1),
    Layout(name="pie", size=3),
)
layout["cuerpo"].split_row(
    Layout(name="lateral", ratio=1),
    Layout(name="principal", ratio=3),
)

layout["cabecera"].update(Panel(Align.center("[bold]Panel de control[/]"), style="on #1F4E78"))
layout["lateral"].update(Panel(arbol, title="Navegación"))
layout["principal"].update(tabla)
layout["pie"].update(Panel("Ctrl+C para salir", style="dim"))

with Live(layout, screen=True, refresh_per_second=4):
    while True:
        layout["principal"].update(generar_tabla(medir()))
        time.sleep(0.5)
```

> `Layout["nombre"].visible = False` oculta una región; `size` fija líneas o columnas; `ratio` reparte el espacio sobrante; `minimum_size` impone un mínimo.

---

## 16. Pedir datos: `Prompt`, `Confirm`, `IntPrompt`

```python
from rich.prompt import Prompt, Confirm, IntPrompt, FloatPrompt

nombre = Prompt.ask("Nombre del proyecto", default="mi-app")
clave = Prompt.ask("Contraseña", password=True)
entorno = Prompt.ask("Entorno", choices=["dev", "pre", "pro"], default="dev")
replicas = IntPrompt.ask("Réplicas", default=2)
cpu = FloatPrompt.ask("CPU por réplica", default=0.5)
seguir = Confirm.ask("¿Desplegar ahora?", default=False)
```

- Con `choices`, solo acepta esas opciones y repite la pregunta si la respuesta no es válida.
- `IntPrompt`/`FloatPrompt` repiten la pregunta hasta recibir un número.
- `Confirm` acepta `y`/`n`.

Validación propia (rangos, formatos):

```python
from rich.prompt import IntPrompt, InvalidResponse

class PuertoPrompt(IntPrompt):
    def process_response(self, valor: str) -> int:
        numero = super().process_response(valor)
        if not 1 <= numero <= 65535:
            raise InvalidResponse("[red]El puerto debe estar entre 1 y 65535[/]")
        return numero

puerto = PuertoPrompt.ask("Puerto", default=8080)
```

Confirmación con letras en español:

```python
class Confirmar(Confirm):
    choices = ["s", "n"]
    validate_error_message = "[prompt.invalid]Responda s o n"

    def process_response(self, valor: str) -> bool:
        valor = valor.strip().lower()
        if valor not in self.choices:
            raise InvalidResponse(self.validate_error_message)
        return valor == "s"

Confirmar.ask("¿Continuar?", default=True)
```

> Para menús con flechas, selección múltiple o autocompletado, combine Rich con **questionary** o **InquirerPy**.

---

## 17. Exportar a HTML, SVG y texto

```python
consola = Console(record=True, width=100)
consola.print(tabla)
consola.print(Panel("Resumen"))

consola.save_html("informe.html", inline_styles=True)
consola.save_svg("informe.svg", title="Informe de ventas")
consola.save_text("informe.txt")

html = consola.export_html(clear=False)         # Como texto, sin borrar el registro
```

Capturar la salida en una variable:

```python
with consola.capture() as captura:
    consola.print(tabla)
texto = captura.get()
```

Escribir directamente en un archivo:

```python
with open("informe.txt", "w", encoding="utf-8") as f:
    Console(file=f, width=120).print(tabla)
```

---

## 18. Objetos personalizados con `__rich__`

Cualquier clase puede definir cómo se muestra con Rich:

```python
from dataclasses import dataclass

@dataclass
class Pedido:
    id: int
    cliente: str
    importe: float
    pagado: bool

    def __rich__(self):
        estado = "[green]pagado[/]" if self.pagado else "[red]pendiente[/]"
        return Panel(f"Cliente: [bold]{self.cliente}[/]\nImporte: {self.importe:,.2f} €\nEstado: {estado}",
                     title=f"Pedido #{self.id}", expand=False)

consola.print(Pedido(1024, "Ana Pérez", 149.9, False))
```

Para control total del ancho disponible, implemente `__rich_console__(self, console, options)` y `__rich_measure__`.

---

## 19. Pruebas y entornos sin terminal

```python
from io import StringIO
from rich.console import Console

def test_tabla():
    salida = StringIO()
    consola = Console(file=salida, width=80, force_terminal=False, color_system=None)
    consola.print(generar_tabla({"srv-01": 20}))
    assert "srv-01" in salida.getvalue()
```

Comportamiento fuera de una terminal (redirección a archivo, CI):

| Situación | Comportamiento de Rich |
|---|---|
| Salida redirigida (`> archivo`) | Sin colores ni animaciones; ancho de 80 columnas. |
| Variable `NO_COLOR` definida | Sin colores. |
| Variable `FORCE_COLOR` definida | Fuerza colores. |
| `TERM=dumb` | Sin estilos. |
| Variables `COLUMNS` / `LINES` | Fijan el tamaño. |
| Jupyter | Renderiza HTML en la celda. |

---

## 20. Ejemplo práctico completo

**Monitor de despliegue**: simula el despliegue de varios servicios mostrando una pantalla en vivo con `Layout` (cabecera, tabla de estado, barras de progreso y registro de eventos), pregunta la configuración con `Prompt`, usa un tema propio, registra eventos con `RichHandler` y exporta un informe final en HTML.

```python
"""
despliegue.py — Simulador de despliegue con panel en vivo.
Requiere: pip install rich
Uso: python despliegue.py [--auto]
"""
import logging
import random
import sys
import time
from collections import deque
from dataclasses import dataclass, field
from datetime import datetime

from rich import box
from rich.align import Align
from rich.console import Console, Group
from rich.layout import Layout
from rich.live import Live
from rich.logging import RichHandler
from rich.panel import Panel
from rich.progress import BarColumn, MofNCompleteColumn, Progress, SpinnerColumn, TextColumn, TimeElapsedColumn
from rich.prompt import Confirm, IntPrompt, Prompt
from rich.table import Table
from rich.text import Text
from rich.theme import Theme
from rich.traceback import install

TEMA = Theme({
    "ok": "bold green", "aviso": "bold yellow", "error": "bold red",
    "info": "cyan", "tenue": "dim", "titulo": "bold white on #1F4E78",
})
consola = Console(theme=TEMA, record=True)
install(console=consola, show_locals=False)

logging.basicConfig(level="INFO", format="%(message)s", datefmt="[%X]",
                    handlers=[RichHandler(console=Console(stderr=True, theme=TEMA), markup=True, show_path=False)])
log = logging.getLogger("despliegue")

ICONOS = {"pendiente": "⏳", "desplegando": "🔄", "verificando": "🔍", "ok": "✅", "error": "❌"}
ESTILOS = {"pendiente": "tenue", "desplegando": "info", "verificando": "aviso", "ok": "ok", "error": "error"}


@dataclass
class Servicio:
    nombre: str
    replicas: int
    estado: str = "pendiente"
    progreso: int = 0
    inicio: float | None = None
    fin: float | None = None

    @property
    def duracion(self) -> str:
        if self.inicio is None:
            return "—"
        return f"{(self.fin or time.monotonic()) - self.inicio:5.1f} s"


@dataclass
class Despliegue:
    entorno: str
    version: str
    servicios: list[Servicio]
    tasa_fallo: float
    eventos: deque = field(default_factory=lambda: deque(maxlen=12))

    def evento(self, mensaje: str, estilo: str = "") -> None:
        self.eventos.append((datetime.now().strftime("%H:%M:%S"), mensaje, estilo))


# ------------------------------------------------------------------ Vistas
def vista_cabecera(d: Despliegue) -> Panel:
    texto = Text.assemble(("🚀 Despliegue ", "bold"), (d.version, "bold cyan"), " → ",
                          (d.entorno.upper(), "bold magenta"), ("   ", ""),
                          (datetime.now().strftime("%d/%m/%Y %H:%M:%S"), "tenue"))
    return Panel(Align.center(texto), box=box.HEAVY, style="on grey11")


def vista_tabla(d: Despliegue) -> Panel:
    tabla = Table(box=box.SIMPLE_HEAVY, expand=True, header_style="bold cyan", row_styles=["", "on grey7"])
    tabla.add_column("Servicio", ratio=3)
    tabla.add_column("Réplicas", justify="right")
    tabla.add_column("Estado", ratio=2)
    tabla.add_column("Progreso", justify="right")
    tabla.add_column("Tiempo", justify="right")
    for s in d.servicios:
        tabla.add_row(s.nombre, str(s.replicas),
                      Text(f"{ICONOS[s.estado]} {s.estado}", style=ESTILOS[s.estado]),
                      f"{s.progreso:3d} %", s.duracion)
    return Panel(tabla, title="[bold]Servicios[/]", border_style="blue")


def vista_resumen(d: Despliegue, progreso: Progress) -> Panel:
    cuenta = {e: sum(s.estado == e for s in d.servicios) for e in ICONOS}
    resumen = Table.grid(padding=(0, 2))
    resumen.add_column(justify="right", style="bold")
    resumen.add_column()
    for estado in ("ok", "error", "desplegando", "verificando", "pendiente"):
        resumen.add_row(str(cuenta[estado]), Text(f"{ICONOS[estado]} {estado}", style=ESTILOS[estado]))
    return Panel(Group(progreso, "", resumen), title="[bold]Progreso global[/]", border_style="green")


def vista_eventos(d: Despliegue) -> Panel:
    lineas = Text()
    for hora, mensaje, estilo in d.eventos:
        lineas.append(f"{hora} ", style="tenue")
        lineas.append(mensaje + "\n", style=estilo)
    return Panel(lineas, title="[bold]Eventos[/]", border_style="magenta")


def construir_layout() -> Layout:
    layout = Layout()
    layout.split_column(Layout(name="cabecera", size=3), Layout(name="cuerpo"), Layout(name="eventos", size=14))
    layout["cuerpo"].split_row(Layout(name="tabla", ratio=2), Layout(name="resumen", ratio=1))
    return layout


def refrescar(layout: Layout, d: Despliegue, progreso: Progress) -> None:
    layout["cabecera"].update(vista_cabecera(d))
    layout["tabla"].update(vista_tabla(d))
    layout["resumen"].update(vista_resumen(d, progreso))
    layout["eventos"].update(vista_eventos(d))


# ------------------------------------------------------------------ Simulación
def desplegar(d: Despliegue) -> bool:
    progreso = Progress(SpinnerColumn(), TextColumn("{task.description}"), BarColumn(bar_width=None),
                        MofNCompleteColumn(), TimeElapsedColumn(), expand=True)
    global_ = progreso.add_task("Servicios", total=len(d.servicios))
    layout = construir_layout()
    d.evento(f"Inicio del despliegue de {d.version} en {d.entorno}", "info")

    with Live(layout, console=consola, refresh_per_second=8, screen=False, transient=False):
        for s in d.servicios:
            s.estado, s.inicio = "desplegando", time.monotonic()
            d.evento(f"Desplegando {s.nombre} ({s.replicas} réplicas)", "info")
            while s.progreso < 100:
                s.progreso = min(100, s.progreso + random.randint(4, 15))
                if s.progreso >= 80 and s.estado == "desplegando":
                    s.estado = "verificando"
                    d.evento(f"Verificando salud de {s.nombre}", "aviso")
                refrescar(layout, d, progreso)
                time.sleep(0.08)
            s.fin = time.monotonic()
            if random.random() < d.tasa_fallo:
                s.estado = "error"
                d.evento(f"✖ {s.nombre}: la verificación de salud ha fallado", "error")
                log.error("Fallo en [bold]%s[/]", s.nombre)
            else:
                s.estado = "ok"
                d.evento(f"✔ {s.nombre} desplegado en {s.duracion.strip()}", "ok")
            progreso.advance(global_)
            refrescar(layout, d, progreso)

        errores = [s for s in d.servicios if s.estado == "error"]
        d.evento("Despliegue finalizado" + (f" con {len(errores)} error(es)" if errores else " correctamente"),
                 "error" if errores else "ok")
        refrescar(layout, d, progreso)
        time.sleep(0.5)
    return not errores


# ------------------------------------------------------------------ Configuración e informe
def configurar(automatico: bool) -> Despliegue:
    nombres = ["auth-api", "usuarios-api", "pedidos-api", "pagos-worker", "catalogo-api",
               "notificaciones-worker", "gateway", "informes-cron"]
    if automatico:
        entorno, version, cantidad, tasa = "pre", "v2.8.0", 6, 0.15
    else:
        consola.print(Panel.fit("[bold]Configuración del despliegue[/]", border_style="cyan"))
        entorno = Prompt.ask("Entorno", choices=["dev", "pre", "pro"], default="pre")
        version = Prompt.ask("Versión", default="v2.8.0")
        cantidad = IntPrompt.ask(f"Número de servicios (1-{len(nombres)})", default=6)
        cantidad = max(1, min(len(nombres), cantidad))
        tasa = 0.0 if entorno == "dev" else 0.15
        if entorno == "pro" and not Confirm.ask("[aviso]Va a desplegar en PRODUCCIÓN. ¿Continuar?[/]",
                                                default=False):
            consola.print("[aviso]Despliegue cancelado.[/]")
            sys.exit(1)
    servicios = [Servicio(n, random.randint(1, 4)) for n in nombres[:cantidad]]
    return Despliegue(entorno, version, servicios, tasa)


def informe(d: Despliegue, correcto: bool) -> None:
    consola.rule("[bold]Resumen final[/]")
    tabla = Table(box=box.ROUNDED, header_style="bold", show_footer=True)
    tabla.add_column("Servicio", footer="Total")
    tabla.add_column("Estado", footer=f"{sum(s.estado == 'ok' for s in d.servicios)}/{len(d.servicios)} OK")
    tabla.add_column("Duración", justify="right")
    for s in d.servicios:
        tabla.add_row(s.nombre, Text(f"{ICONOS[s.estado]} {s.estado}", style=ESTILOS[s.estado]), s.duracion)
    consola.print(tabla)
    if correcto:
        consola.print(Panel.fit("[ok]Despliegue completado sin errores[/]", border_style="green"))
    else:
        consola.print(Panel.fit("[error]Despliegue con errores: revise los servicios marcados[/]",
                                border_style="red"))
    ruta = f"despliegue_{d.version}_{d.entorno}.html"
    consola.save_html(ruta)
    consola.print(f"[tenue]Informe guardado en[/] [link=file://{ruta}]{ruta}[/link]")


def main() -> int:
    automatico = "--auto" in sys.argv or not consola.is_terminal
    despliegue = configurar(automatico)
    try:
        correcto = desplegar(despliegue)
    except KeyboardInterrupt:
        consola.print("\n[aviso]Despliegue interrumpido por el usuario.[/]")
        return 130
    informe(despliegue, correcto)
    return 0 if correcto else 1


if __name__ == "__main__":
    sys.exit(main())
```

---

## 21. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `MarkupError: closing tag '[/x]' … doesn't match any open tag` | Corchetes en datos variables interpretados como marcado. | `escape(valor)` o `markup=False`. |
| Los colores no aparecen en un archivo o CI | Rich detecta que no es una terminal. | `Console(force_terminal=True)` o variable `FORCE_COLOR`. |
| Las barras de progreso se duplican o rompen | Se usó `print()` normal dentro de `Progress`/`Live`. | Use `progreso.console.print()` o `live.console.print()`. |
| `LiveError: Only one live display may be active at once` | `Progress`/`status` dentro de otro `Live`. | Integre la barra dentro del *layout* del `Live` existente. |
| La tabla se corta o se ve desordenada | Terminal estrecha. | `overflow="fold"`, `ratio`, `min_width` o `expand=True`. |
| Caracteres extraños en Windows antiguo | Consola sin soporte Unicode/ANSI. | Use Windows Terminal; `safe_box=True`. |
| `Prompt.ask` no termina en CI | Espera entrada del usuario. | Detecte `console.is_terminal` y use valores por defecto. |
| El `Layout` no ocupa toda la pantalla | `Live` sin `screen=True`. | `Live(layout, screen=True)`. |
| Exportación HTML vacía | Consola sin `record=True`. | `Console(record=True)`. |
| Datos sensibles en tracebacks | `show_locals=True`. | Desactívelo en producción. |

---

## 22. Buenas prácticas

- Cree **una** `Console` (y otra para `stderr`) y reutilícela en toda la aplicación.
- Defina un tema con nombres semánticos (`ok`, `error`, `aviso`) en lugar de colores repartidos por el código.
- Escape siempre los datos de usuario que se mezclen con marcado.
- Separe la construcción de las vistas (funciones que devuelven `Table`, `Panel`…) de su impresión.
- Use `RichHandler` para los logs y `traceback.install()` durante el desarrollo.
- Dentro de `Progress` o `Live`, imprima siempre a través de su consola.
- Respete `NO_COLOR` y no dependa del color para transmitir información crítica (añada iconos o texto).
- Detecte si hay terminal (`console.is_terminal`) antes de animaciones o preguntas.
- Para menús interactivos use questionary/InquirerPy; para aplicaciones completas, Textual.

---

## 23. Referencia rápida

```python
from rich import print, inspect; from rich.console import Console
c = Console(theme=Theme({...}), record=True, stderr=False, width=None)
c.print("[bold red on white]x[/] [link=url]y[/]", style=, justify=, highlight=, markup=)
c.log() · c.rule("t") · c.print_json() · c.input() · c.status("…") · c.pager() · c.screen() · c.capture()
Text("x", style=).append() .highlight_regex() · escape(texto)
Table(title=, box=box.ROUNDED, expand=, show_lines=, row_styles=); .add_column(justify=, style=, ratio=); .add_row()
Table.grid() · Panel(x, title=, border_style=) · Panel.fit() · Columns([...]) · Align.center() · Group(...)
Tree("raiz").add("hijo") · Syntax(code, "python", line_numbers=True) · Markdown(md) · JSON(txt)
track(it, description=) · Progress(SpinnerColumn(), BarColumn(), ...) → add_task / update / advance
Live(renderable, refresh_per_second=, screen=) → live.update() · Layout().split_column/split_row
Prompt.ask(t, choices=, default=, password=) · IntPrompt · FloatPrompt · Confirm.ask
RichHandler(rich_tracebacks=True) · traceback.install(show_locals=) · c.print_exception()
c.save_html() / save_svg() / save_text() · __rich__ / __rich_console__
```

---

## 24. Ejercicios propuestos

1. Escriba un script que muestre el contenido de un CSV como tabla Rich, con alineación automática de números y filas alternas.
2. Cree un visor de carpetas que muestre un `Tree` con tamaños y colores por tipo de archivo.
3. Implemente un monitor de recursos (CPU, memoria, disco) con `Live` y `Layout` que se refresque cada segundo (puede usar `psutil`).
4. Sustituya los `print` de un script existente por `logging` con `RichHandler` y añada `traceback.install()`.
5. Amplíe el ejemplo completo para que, en caso de error, pregunte con `Confirm` si se desea revertir los servicios ya desplegados y muestre el progreso de la reversión.

---

## 25. Referencias

- Documentación oficial: https://rich.readthedocs.io/
- Repositorio: https://github.com/Textualize/rich
- Galería de ejemplos: https://github.com/Textualize/rich/tree/master/examples
- Textual (aplicaciones TUI del mismo autor): https://textual.textualize.io/
