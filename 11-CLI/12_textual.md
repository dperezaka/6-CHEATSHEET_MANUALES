# Manual de uso: Textual

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Aplicaciones TUI completas en terminal, con CSS, widgets reactivos y ratón
> **Tipo:** Librería de terceros (`pip install textual`) · **Versión de referencia:** 8.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera aplicación](#4-primera-aplicación)
5. [Componer la interfaz: `compose` y contenedores](#5-componer-la-interfaz-compose-y-contenedores)
6. [Widgets principales](#6-widgets-principales)
7. [Estilos con Textual CSS](#7-estilos-con-textual-css)
8. [Eventos, mensajes y el decorador `@on`](#8-eventos-mensajes-y-el-decorador-on)
9. [Atajos de teclado y acciones](#9-atajos-de-teclado-y-acciones)
10. [Atributos reactivos](#10-atributos-reactivos)
11. [Consultar y modificar widgets (DOM)](#11-consultar-y-modificar-widgets-dom)
12. [Widgets personalizados](#12-widgets-personalizados)
13. [Pantallas y diálogos modales](#13-pantallas-y-diálogos-modales)
14. [Tareas en segundo plano: *workers* y temporizadores](#14-tareas-en-segundo-plano-workers-y-temporizadores)
15. [Notificaciones, temas y paleta de comandos](#15-notificaciones-temas-y-paleta-de-comandos)
16. [Herramientas de desarrollo](#16-herramientas-de-desarrollo)
17. [Pruebas con `run_test` y `Pilot`](#17-pruebas-con-run_test-y-pilot)
18. [Distribución y ejecución en el navegador](#18-distribución-y-ejecución-en-el-navegador)
19. [Ejemplo práctico completo](#19-ejemplo-práctico-completo)
20. [Errores frecuentes](#20-errores-frecuentes)
21. [Buenas prácticas](#21-buenas-prácticas)
22. [Referencia rápida](#22-referencia-rápida)
23. [Ejercicios propuestos](#23-ejercicios-propuestos)
24. [Referencias](#24-referencias)

---

## 1. Descripción

**Textual** es un *framework* para crear **aplicaciones de interfaz de usuario en terminal** (TUI) con Python. Ofrece un modelo similar al de las aplicaciones web o de escritorio: widgets (botones, tablas, formularios, pestañas, árboles), diseño con **CSS** propio, eventos, atajos de teclado, soporte de **ratón**, pantallas y diálogos, tareas en segundo plano y animaciones. Es del mismo autor que **Rich**, cuyo motor de renderizado utiliza.

**Ventajas:**
- Interfaces ricas y modernas en la terminal, incluso por SSH.
- Diseño separado de la lógica mediante Textual CSS (archivos `.tcss`).
- Amplio catálogo de widgets listos para usar.
- Modelo reactivo: la interfaz se actualiza al cambiar un atributo.
- Programación asíncrona nativa y *workers* para tareas largas.
- Herramientas de desarrollo: consola de depuración, recarga de CSS en vivo.
- Pruebas automatizadas con un «piloto» que simula teclado y ratón.
- Las aplicaciones pueden servirse en el navegador sin cambios (`textual serve`).

**Limitaciones:**
- Curva de aprendizaje mayor que una CLI clásica (eventos, DOM, CSS, asincronía).
- Depende de las capacidades de la terminal (colores, Unicode, tamaño).
- No apropiada para scripts que deban ejecutarse sin interacción.
- Evoluciona con rapidez: consulte las notas de versión al actualizar.

| Si necesita… | Opción |
|---|---|
| Aplicación de terminal completa con widgets, ratón y diseño | **Textual** |
| Salida con formato sin interacción | Rich |
| Preguntas encadenadas tipo formulario | questionary / InquirerPy |
| REPL o *shell* propia | prompt_toolkit |
| Interfaz clásica de consola en Unix con mínima dependencia | urwid |
| Aplicación gráfica de escritorio | Tkinter / PySide6 |

---

## 2. Instalación e importación

```bash
pip install textual
pip install textual-dev          # Herramientas de desarrollo (consola, recarga en vivo)
python -m textual                # Demostración de widgets y capacidades
```

Importaciones habituales:

```python
from textual import on, work
from textual.app import App, ComposeResult
from textual.binding import Binding
from textual.containers import Horizontal, Vertical, Grid, VerticalScroll, Container
from textual.reactive import reactive
from textual.screen import Screen, ModalScreen
from textual.widgets import Header, Footer, Button, Input, Label, Static, DataTable
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `App` | Clase principal de la aplicación. |
| `compose()` | Método que declara (con `yield`) los widgets de la interfaz. |
| Widget | Componente visual: `Button`, `Input`, `DataTable`… |
| Contenedor | Widget que agrupa y distribuye otros (`Horizontal`, `Vertical`, `Grid`). |
| DOM | Árbol de widgets de la aplicación, consultable con selectores CSS. |
| Textual CSS | Hojas de estilo (`.tcss`) que definen diseño y aspecto. |
| Evento / mensaje | Notificación de algo ocurrido (`Button.Pressed`, `Input.Changed`, `Key`). |
| Manejador | Método `on_<mensaje>` o decorado con `@on(...)` que responde a un mensaje. |
| Acción | Operación con nombre (`action_guardar`) que se puede asociar a teclas. |
| *Binding* | Asociación de una tecla a una acción, mostrada en el `Footer`. |
| Reactivo | Atributo que, al cambiar, redibuja el widget y llama a `watch_<nombre>`. |
| `Screen` | Pantalla completa; las aplicaciones pueden apilar varias. |
| *Worker* | Tarea en segundo plano (corrutina o hilo) gestionada por Textual. |

---

## 4. Primera aplicación

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Button, Static


class Saludo(App):
    """Aplicación mínima con un botón."""

    CSS = """
    Screen { align: center middle; }
    #mensaje { width: 40; content-align: center middle; height: 3; border: round $accent; }
    Button { margin-top: 1; }
    """
    BINDINGS = [("q", "quit", "Salir"), ("d", "toggle_dark", "Modo oscuro")]

    def compose(self) -> ComposeResult:
        yield Header(show_clock=True)
        yield Static("Pulse el botón", id="mensaje")
        yield Button("Saludar", id="saludar", variant="primary")
        yield Footer()

    def on_button_pressed(self, evento: Button.Pressed) -> None:
        self.query_one("#mensaje", Static).update("¡Hola desde Textual!")

    def action_toggle_dark(self) -> None:
        self.theme = "textual-light" if self.theme == "textual-dark" else "textual-dark"


if __name__ == "__main__":
    Saludo().run()
```

```bash
python saludo.py
```

- **Tab/Shift+Tab** mueve el foco entre widgets; **Enter** o clic pulsa el botón.
- **Ctrl+Q** cierra cualquier aplicación Textual; **Ctrl+P** abre la paleta de comandos.

---

## 5. Componer la interfaz: `compose` y contenedores

### 5.1 `compose` con contenedores anidados

```python
from textual.containers import Horizontal, Vertical, VerticalScroll

def compose(self) -> ComposeResult:
    yield Header()
    with Horizontal():
        with Vertical(id="lateral"):
            yield Label("Filtros")
            yield Input(placeholder="Buscar…", id="busqueda")
        with VerticalScroll(id="principal"):
            yield DataTable(id="tabla")
    yield Footer()
```

### 5.2 Contenedores disponibles

| Contenedor | Distribución |
|---|---|
| `Vertical` / `Horizontal` | Apila en columna / en fila (ocupan todo el espacio disponible). |
| `VerticalGroup` / `HorizontalGroup` | Igual, pero con la altura justa del contenido. |
| `VerticalScroll` / `HorizontalScroll` / `ScrollableContainer` | Con barras de desplazamiento. |
| `Grid` | Rejilla (definida en CSS con `grid-size`). |
| `ItemGrid` | Rejilla que ajusta columnas al ancho disponible. |
| `Container` | Genérico. |
| `Center`, `Middle`, `CenterMiddle`, `Right` | Alineación. |

### 5.3 Añadir y quitar widgets en ejecución

```python
await self.query_one("#lista").mount(Label("Nuevo elemento"))
self.query_one("#lista").mount(Label("Al principio"), before=0)
self.query_one("#aviso").remove()
await self.query_one("#lista").remove_children()
```

---

## 6. Widgets principales

### 6.1 Texto y presentación

```python
Label("Texto simple")
Static("[b]Texto con marcado Rich[/b]", id="info")      # .update("…") para cambiarlo
Markdown("# Título\n- elemento")
Pretty({"clave": "valor"})
Digits("12:45")                                          # Números grandes
Rule()                                                   # Separador
Link("Documentación", url="https://textual.textualize.io")
```

### 6.2 Entrada de datos

```python
from textual.validation import Number, Length, Regex, Function

Input(placeholder="Nombre", id="nombre", validators=[Length(minimum=2)])
Input(placeholder="Edad", type="integer", validators=[Number(minimum=0, maximum=120)])
Input(placeholder="Email", validators=[Regex(r"^[\w.+-]+@[\w-]+\.[\w.-]+$", failure_description="Email no válido")])
Input(password=True, placeholder="Contraseña")
MaskedInput(template="9999-99-99;_", placeholder="AAAA-MM-DD")
TextArea("texto inicial", language="python", show_line_numbers=True)   # Editor multilínea
Checkbox("Acepto las condiciones", value=False)
Switch(value=True)
RadioSet("Particular", "Empresa", id="tipo")
Select([("Madrid", "mad"), ("Sevilla", "sev")], prompt="Ciudad", allow_blank=False, value="mad")
SelectionList(("Email", "email", True), ("SMS", "sms"))   # Selección múltiple
Button("Guardar", variant="success")                      # default | primary | success | warning | error
```

Validar un `Input`:

```python
@on(Input.Changed, "#email")
def validar_email(self, evento: Input.Changed) -> None:
    if evento.validation_result and not evento.validation_result.is_valid:
        self.query_one("#error", Label).update(evento.validation_result.failure_descriptions[0])
    else:
        self.query_one("#error", Label).update("")
```

### 6.3 Listas, tablas y árboles

```python
# Tabla
tabla = DataTable(zebra_stripes=True, cursor_type="row")
tabla.add_columns("Código", "Producto", "Precio")
tabla.add_row("TEC-01", "Teclado", "59,90 €", key="TEC-01")
tabla.update_cell("TEC-01", "Precio", "54,90 €")     # requiere claves de columna (add_column(..., key=))
tabla.remove_row("TEC-01")
tabla.clear()
tabla.sort("Producto", reverse=False)

# Lista
ListView(ListItem(Label("Uno")), ListItem(Label("Dos")))
OptionList("Opción A", "Opción B", "Opción C")

# Árboles
arbol = Tree("Proyecto")
src = arbol.root.add("src", expand=True)
src.add_leaf("main.py")
DirectoryTree("./")                                    # Explorador de archivos
```

### 6.4 Navegación y agrupación

```python
with TabbedContent(initial="clientes"):
    with TabPane("Clientes", id="clientes"):
        yield DataTable()
    with TabPane("Pedidos", id="pedidos"):
        yield DataTable()

with Collapsible(title="Opciones avanzadas", collapsed=True):
    yield Switch()

with ContentSwitcher(initial="vista1"):                # Muestra un hijo cada vez
    yield Static("Vista 1", id="vista1")
    yield Static("Vista 2", id="vista2")
```

### 6.5 Progreso, registro y visualización

```python
ProgressBar(total=100, show_eta=True)                 # .advance(1) / .update(progress=50)
LoadingIndicator()                                    # O widget.loading = True en cualquier widget
RichLog(highlight=True, markup=True)                  # .write("…"), admite objetos Rich
Log()                                                 # .write_line("…"), más ligero
Sparkline([1, 4, 2, 8, 5, 7])
Placeholder("Zona pendiente de diseño")
```

---

## 7. Estilos con Textual CSS

### 7.1 Dónde escribir el CSS

```python
class MiApp(App):
    CSS_PATH = "estilos.tcss"        # Archivo externo (recomendado; admite recarga en vivo)
    # CSS = "…"                      # O CSS en línea

class MiWidget(Static):
    DEFAULT_CSS = """
    MiWidget { padding: 1; border: tall $primary; }
    """
```

### 7.2 Selectores

```css
Button { margin: 1 2; }                       /* Por tipo */
#guardar { background: $success; }            /* Por id */
.peligro { color: $error; }                   /* Por clase */
#lateral Input { width: 100%; }               /* Descendiente */
Button:hover { text-style: bold; }            /* Pseudoclases: hover, focus, disabled, dark, light… */
Input.-invalid { border: tall $error; }       /* Clase interna de estado */
```

Clases desde Python:

```python
widget.add_class("peligro")
widget.remove_class("peligro")
widget.toggle_class("seleccionado")
widget.set_class(condicion, "resaltado")
```

### 7.3 Propiedades más usadas

| Propiedad | Ejemplo |
|---|---|
| `width`, `height` | `30`, `50%`, `1fr`, `auto`, `100vw` |
| `min-width`, `max-height` | `20`, `80%` |
| `margin`, `padding` | `1`, `1 2`, `0 1 0 2` |
| `border`, `outline` | `round $accent`, `tall green`, `heavy red`, `none` |
| `background`, `color` | `$surface`, `#1F4E78`, `red 50%` |
| `text-style` | `bold`, `italic`, `reverse`, `underline` |
| `text-align`, `content-align` | `center`, `center middle` |
| `align` | Alinea los hijos: `center middle` |
| `layout` | `vertical`, `horizontal`, `grid` |
| `grid-size`, `grid-columns`, `grid-gutter` | `3 2`, `1fr 2fr`, `1` |
| `column-span`, `row-span` | `2` |
| `dock` | `top`, `bottom`, `left`, `right` |
| `display` | `block`, `none` |
| `visibility` | `visible`, `hidden` |
| `overflow` | `auto`, `hidden`, `scroll` |
| `opacity`, `tint` | `50%`, `red 20%` |
| `transition` | `offset 300ms in_out_cubic` |

### 7.4 Rejilla

```css
#panel {
    layout: grid;
    grid-size: 3 2;              /* 3 columnas, 2 filas */
    grid-columns: 1fr 1fr 2fr;
    grid-gutter: 1 2;
}
#grafico { column-span: 2; }
```

### 7.5 Variables de tema

Use variables en lugar de colores fijos para que la aplicación funcione con cualquier tema: `$primary`, `$secondary`, `$accent`, `$success`, `$warning`, `$error`, `$background`, `$surface`, `$panel`, `$foreground`, `$text-muted`, `$boost` y sus variantes (`$primary-lighten-2`, `$error-darken-1`…).

### 7.6 Anidamiento

```css
#formulario {
    padding: 1 2;
    & Input { margin-bottom: 1; }
    & .botones { align-horizontal: right; }
}
```

---

## 8. Eventos, mensajes y el decorador `@on`

### 8.1 Manejadores por nombre

El nombre se forma con `on_` + nombre del widget + nombre del mensaje en minúsculas con guiones bajos:

```python
def on_button_pressed(self, evento: Button.Pressed) -> None: ...
def on_input_submitted(self, evento: Input.Submitted) -> None: ...
def on_data_table_row_selected(self, evento: DataTable.RowSelected) -> None: ...
def on_mount(self) -> None: ...          # Al montar la app o el widget
def on_key(self, evento) -> None: ...    # Cualquier tecla
def on_resize(self, evento) -> None: ...
```

### 8.2 `@on` con selectores (recomendado)

```python
from textual import on

@on(Button.Pressed, "#guardar")
def guardar(self) -> None: ...

@on(Button.Pressed, ".peligro")
def confirmar_peligro(self, evento: Button.Pressed) -> None: ...

@on(Input.Changed, "#busqueda")
def filtrar(self, evento: Input.Changed) -> None:
    self.aplicar_filtro(evento.value)
```

### 8.3 Mensajes frecuentes

| Widget | Mensajes |
|---|---|
| `Button` | `Pressed` |
| `Input` | `Changed`, `Submitted`, `Blurred` |
| `Checkbox` / `Switch` | `Changed` |
| `Select` | `Changed` |
| `RadioSet` | `Changed` |
| `DataTable` | `RowSelected`, `RowHighlighted`, `CellSelected`, `HeaderSelected` |
| `ListView` | `Selected`, `Highlighted` |
| `Tree` / `DirectoryTree` | `NodeSelected`, `NodeExpanded` / `FileSelected` |
| `TabbedContent` | `TabActivated` |

### 8.4 Mensajes propios

```python
from textual.message import Message

class Contador(Static):
    class Cambiado(Message):
        def __init__(self, valor: int) -> None:
            self.valor = valor
            super().__init__()

    def incrementar(self) -> None:
        self.valor += 1
        self.post_message(self.Cambiado(self.valor))

# En la app
def on_contador_cambiado(self, evento: Contador.Cambiado) -> None:
    self.notify(f"Nuevo valor: {evento.valor}")
```

Los mensajes **suben** por el DOM (del widget a sus contenedores y a la app). `evento.stop()` detiene la propagación; `evento.prevent_default()` evita el comportamiento predeterminado.

### 8.5 Manejadores asíncronos

Cualquier manejador puede ser `async def`; Textual lo espera sin bloquear la interfaz:

```python
async def on_mount(self) -> None:
    await self.cargar_datos()
```

---

## 9. Atajos de teclado y acciones

```python
from textual.binding import Binding

class MiApp(App):
    BINDINGS = [
        ("q", "quit", "Salir"),
        ("n", "nuevo", "Nuevo"),
        Binding("ctrl+s", "guardar", "Guardar", priority=True),       # Funciona aunque un Input tenga el foco
        Binding("delete", "eliminar", "Eliminar", show=True),
        Binding("f1", "app.push_screen('ayuda')", "Ayuda"),
        Binding("escape", "limpiar", show=False),
    ]

    def action_nuevo(self) -> None: ...
    def action_guardar(self) -> None: ...
    def action_eliminar(self) -> None: ...
```

- Las acciones son métodos `action_<nombre>`.
- `priority=True` hace que el atajo tenga prioridad sobre el widget con foco.
- Los widgets también pueden tener sus propios `BINDINGS` (activos cuando tienen el foco).
- Las acciones pueden invocarse desde marcado: `Static("[@click=app.nuevo]Crear[/]")`.

Habilitar o deshabilitar acciones dinámicamente:

```python
def check_action(self, accion: str, parametros) -> bool | None:
    if accion == "eliminar":
        return self.hay_seleccion        # False: oculta; None: muestra deshabilitada
    return True
# Tras cambiar el estado: self.refresh_bindings()
```

---

## 10. Atributos reactivos

```python
from textual.reactive import reactive, var

class Temporizador(Static):
    segundos = reactive(0)                       # Al cambiar, se redibuja y se llama a watch_segundos
    en_marcha = reactive(False, bindings=True)   # bindings=True: refresca los atajos al cambiar
    interno = var(0)                             # Reactivo sin redibujar

    def render(self) -> str:
        m, s = divmod(self.segundos, 60)
        return f"{m:02d}:{s:02d}"

    def watch_segundos(self, anterior: int, nuevo: int) -> None:
        if nuevo >= 60:
            self.add_class("largo")

    def validate_segundos(self, valor: int) -> int:     # Valida/ajusta antes de asignar
        return max(0, valor)
```

| Método | Cuándo se llama |
|---|---|
| `watch_<atributo>(nuevo)` o `(anterior, nuevo)` | Tras cambiar el valor. |
| `validate_<atributo>(valor)` | Antes de asignar; devuelve el valor corregido. |
| `compute_<atributo>()` | Valor calculado a partir de otros reactivos. |

```python
class Factura(Static):
    base = reactive(0.0)
    iva = reactive(0.21)
    total = reactive(0.0)

    def compute_total(self) -> float:
        return round(self.base * (1 + self.iva), 2)
```

Vincular reactivos entre padre e hijo:

```python
yield Temporizador().data_bind(segundos=MiApp.tiempo)
```

---

## 11. Consultar y modificar widgets (DOM)

```python
self.query_one("#nombre", Input).value                 # Un widget (error si no existe o hay varios)
self.query_one(DataTable)                               # Por tipo
self.query("Button")                                    # Todos los botones (DOMQuery)
self.query(".peligro").add_class("oculto")
self.query("Input").first().focus()
for boton in self.query(Button):
    boton.disabled = True
self.query_exactly_one("#unico")                       # Error si hay más de uno
```

Propiedades útiles de cualquier widget:

| Propiedad / método | Uso |
|---|---|
| `id`, `classes` | Identificadores. |
| `styles.display = "none"` | Ocultar desde código. |
| `disabled = True` | Deshabilitar. |
| `loading = True` | Muestra un indicador de carga sobre el widget. |
| `focus()`, `has_focus` | Foco. |
| `tooltip = "texto"` | Información emergente. |
| `border_title`, `border_subtitle` | Texto en el borde. |
| `refresh()` | Redibujar. |
| `scroll_visible()` | Desplazar hasta hacerlo visible. |

---

## 12. Widgets personalizados

### 12.1 Composición de widgets existentes

```python
class TarjetaKPI(Vertical):
    DEFAULT_CSS = """
    TarjetaKPI { border: round $primary; height: auto; padding: 0 1; width: 1fr; }
    TarjetaKPI .valor { text-style: bold; color: $accent; }
    """

    def __init__(self, titulo: str, valor: str, **kwargs) -> None:
        super().__init__(**kwargs)
        self.titulo, self.valor = titulo, valor

    def compose(self) -> ComposeResult:
        yield Label(self.titulo)
        yield Label(self.valor, classes="valor")

    def actualizar(self, valor: str) -> None:
        self.query_one(".valor", Label).update(valor)
```

### 12.2 Widget con `render`

```python
from rich.text import Text

class Semaforo(Widget):
    estado = reactive("verde")

    def render(self) -> Text:
        colores = {"verde": "green", "ambar": "yellow", "rojo": "red"}
        return Text("● " + self.estado.upper(), style=f"bold {colores[self.estado]}")
```

> `render` puede devolver texto, `Text` o cualquier objeto Rich (tablas, paneles…).

---

## 13. Pantallas y diálogos modales

### 13.1 Pantallas

```python
class PantallaAyuda(Screen):
    BINDINGS = [("escape", "app.pop_screen", "Volver")]

    def compose(self) -> ComposeResult:
        yield Markdown(TEXTO_AYUDA)
        yield Footer()


class MiApp(App):
    SCREENS = {"ayuda": PantallaAyuda}

    def action_ayuda(self) -> None:
        self.push_screen("ayuda")          # pop_screen() para volver; switch_screen() para sustituir
```

### 13.2 Diálogo modal que devuelve un valor

```python
class Confirmar(ModalScreen[bool]):
    DEFAULT_CSS = """
    Confirmar { align: center middle; }
    #dialogo { width: 50; height: auto; border: thick $warning; background: $surface; padding: 1 2; }
    #dialogo Horizontal { height: auto; align-horizontal: right; margin-top: 1; }
    """

    def __init__(self, pregunta: str) -> None:
        super().__init__()
        self.pregunta = pregunta

    def compose(self) -> ComposeResult:
        with Vertical(id="dialogo"):
            yield Label(self.pregunta)
            with Horizontal():
                yield Button("Cancelar", id="no")
                yield Button("Aceptar", id="si", variant="warning")

    @on(Button.Pressed)
    def responder(self, evento: Button.Pressed) -> None:
        self.dismiss(evento.button.id == "si")
```

Uso con *callback*:

```python
def action_eliminar(self) -> None:
    def al_responder(confirmado: bool | None) -> None:
        if confirmado:
            self.eliminar_seleccion()
    self.push_screen(Confirmar("¿Eliminar el registro?"), al_responder)
```

Uso con `await` (dentro de un *worker*):

```python
@work
async def action_eliminar(self) -> None:
    if await self.push_screen_wait(Confirmar("¿Eliminar el registro?")):
        self.eliminar_seleccion()
```

---

## 14. Tareas en segundo plano: *workers* y temporizadores

### 14.1 Por qué

Textual ejecuta la interfaz en un bucle de eventos. Una función lenta (red, disco, cálculo) en un manejador **congela** la aplicación. Los *workers* la ejecutan en segundo plano.

### 14.2 `@work` asíncrono (E/S con librerías async)

```python
from textual import work

@work(exclusive=True)                    # exclusive: cancela el worker anterior del mismo grupo
async def buscar(self, texto: str) -> None:
    resultados = await api_async.buscar(texto)
    self.mostrar(resultados)             # En workers async se puede tocar la UI directamente
```

### 14.3 `@work(thread=True)` (código bloqueante)

```python
from textual.worker import get_current_worker

@work(thread=True, exclusive=True)
def exportar(self, ruta: str) -> None:
    worker = get_current_worker()
    filas = self.repo.todas()
    for i, fila in enumerate(filas, 1):
        if worker.is_cancelled:
            return
        escribir(fila)
        self.call_from_thread(self.query_one(ProgressBar).update, progress=i)   # UI solo vía call_from_thread
    self.call_from_thread(self.notify, f"{len(filas)} filas exportadas")
```

> Desde un hilo, **nunca** modifique widgets directamente: use `self.call_from_thread(funcion, *args)`.

### 14.4 Estado de los *workers*

```python
from textual.worker import Worker, WorkerState

def on_worker_state_changed(self, evento: Worker.StateChanged) -> None:
    if evento.state == WorkerState.ERROR:
        self.notify(f"Error: {evento.worker.error}", severity="error")
    elif evento.state == WorkerState.SUCCESS:
        self.log(f"Resultado: {evento.worker.result}")
```

### 14.5 Temporizadores

```python
def on_mount(self) -> None:
    self.reloj = self.set_interval(1, self.actualizar_reloj)      # Repetitivo
    self.set_timer(5, self.ocultar_aviso)                          # Una sola vez
    self.call_later(self.enfocar_busqueda)                         # Tras el refresco actual
# self.reloj.pause() · self.reloj.resume() · self.reloj.stop()
```

---

## 15. Notificaciones, temas y paleta de comandos

### 15.1 Notificaciones

```python
self.notify("Registro guardado")
self.notify("Sin conexión con el servidor", title="Aviso", severity="warning", timeout=5)
self.notify("No se pudo guardar", severity="error")
self.bell()
```

### 15.2 Temas

```python
self.theme = "nord"        # textual-dark, textual-light, nord, gruvbox, dracula, tokyo-night,
                           # monokai, catppuccin-mocha, solarized-light, flexoki…
```

Tema propio:

```python
from textual.theme import Theme

corporativo = Theme(name="corporativo", primary="#1F4E78", secondary="#2E75B6", accent="#E07B00",
                    success="#2CA02C", warning="#F5A623", error="#C00000",
                    background="#101820", surface="#1A2633", panel="#22313F", dark=True)

def on_mount(self) -> None:
    self.register_theme(corporativo)
    self.theme = "corporativo"
```

### 15.3 Paleta de comandos

**Ctrl+P** abre una paleta con búsqueda difusa de comandos (cambiar tema, salir, captura de pantalla…). Se pueden añadir comandos propios:

```python
from textual.app import SystemCommand

def get_system_commands(self, screen):
    yield from super().get_system_commands(screen)
    yield SystemCommand("Exportar CSV", "Exporta los datos visibles", self.action_exportar)
```

### 15.4 Otras utilidades de la aplicación

```python
self.copy_to_clipboard(texto)
self.save_screenshot("captura.svg")
with self.suspend():                  # Suspende la TUI para ejecutar un programa externo (p. ej., un editor)
    subprocess.run(["vim", "notas.txt"])
self.exit(resultado)                  # App.run() devuelve 'resultado'
```

---

## 16. Herramientas de desarrollo

Con `textual-dev` instalado:

```bash
textual run --dev mi_app.py            # Recarga el CSS al guardar y conecta con la consola
textual console                        # En otra terminal: muestra self.log(...), print() y eventos
textual console -x EVENT               # Excluir grupos de mensajes
textual colors                         # Previsualizar variables de color del tema
textual keys                           # Ver qué tecla envía la terminal
textual borders / easing               # Catálogos de bordes y curvas de animación
```

Registrar mensajes de depuración:

```python
self.log("Valor actual:", self.valor)
self.log(tabla=tabla_rich)             # Admite objetos Rich
```

---

## 17. Pruebas con `run_test` y `Pilot`

```python
import pytest
from mi_app import MiApp

@pytest.mark.asyncio                    # pip install pytest-asyncio
async def test_saludo():
    app = MiApp()
    async with app.run_test(size=(100, 30)) as pilot:
        await pilot.click("#saludar")
        assert str(app.query_one("#mensaje").render()) == "¡Hola desde Textual!"

async def test_formulario():
    app = MiApp()
    async with app.run_test() as pilot:
        await pilot.click("#nombre")
        await pilot.press(*"Ana", "tab")
        await pilot.press("ctrl+s")
        await pilot.pause()                     # Espera a que se procesen los mensajes
        assert app.guardados == ["Ana"]
```

| Método de `Pilot` | Acción |
|---|---|
| `press("a", "enter", "ctrl+s")` | Teclas. |
| `click("#selector")` / `click(Button, offset=(2, 1))` | Clic. |
| `hover(...)`, `double_click(...)` | Ratón. |
| `pause(delay=None)` | Esperar a que se procesen los mensajes. |
| `resize_terminal(ancho, alto)` | Cambiar el tamaño. |
| `wait_for_scheduled_animations()` | Esperar animaciones. |

Pruebas de instantánea visual: plugin `pytest-textual-snapshot` (`snap_compare`).

---

## 18. Distribución y ejecución en el navegador

```toml
[project]
name = "inventario-tui"
dependencies = ["textual>=1.0"]

[project.scripts]
inventario = "inventario.app:main"
```

```python
def main() -> None:
    Inventario().run()
```

Servir la aplicación en el navegador (con `textual-dev`):

```bash
textual serve "python -m inventario.app"     # http://localhost:8000
```

> Asegúrese de que el CSS externo (`CSS_PATH`) se incluye en el paquete (por ejemplo, con `package-data`).

---

## 19. Ejemplo práctico completo

**Gestor de inventario TUI**: tabla con búsqueda en vivo y filtro por categoría, tarjetas de indicadores reactivas, formulario modal con validación para crear y editar productos, confirmación de borrado, atajos en el pie, exportación a CSV en un hilo con barra de progreso, notificaciones, tema corporativo y comando propio en la paleta. Almacena los datos en JSON.

```python
"""
inventario.py — Gestor de inventario en terminal con Textual.
Requiere: pip install textual
Uso: python inventario.py
"""
import csv
import json
import time
from dataclasses import asdict, dataclass
from pathlib import Path

from textual import on, work
from textual.app import App, ComposeResult, SystemCommand
from textual.binding import Binding
from textual.containers import Horizontal, Vertical
from textual.reactive import reactive
from textual.screen import ModalScreen
from textual.theme import Theme
from textual.validation import Length, Number
from textual.widgets import (Button, DataTable, Footer, Header, Input, Label, ProgressBar, Select,
                             Static)
from textual.worker import get_current_worker

RUTA_DATOS = Path("inventario.json")
CATEGORIAS = ["Periféricos", "Monitores", "Almacenamiento", "Redes", "Accesorios"]
STOCK_MINIMO = 10


# ------------------------------------------------------------------ Datos
@dataclass
class Producto:
    codigo: str
    nombre: str
    categoria: str
    precio: float
    stock: int


class Repositorio:
    def __init__(self, ruta: Path):
        self.ruta = ruta
        if ruta.exists():
            self.productos = {p["codigo"]: Producto(**p) for p in json.loads(ruta.read_text(encoding="utf-8"))}
        else:
            self.productos = {p.codigo: p for p in self._ejemplos()}
            self.guardar()

    @staticmethod
    def _ejemplos() -> list[Producto]:
        base = [("Teclado mecánico", "Periféricos", 89.9), ("Ratón inalámbrico", "Periféricos", 29.5),
                ("Monitor 27\" 4K", "Monitores", 329.0), ("Monitor 24\" FHD", "Monitores", 139.0),
                ("SSD NVMe 1 TB", "Almacenamiento", 74.9), ("Disco externo 2 TB", "Almacenamiento", 69.0),
                ("Router Wi-Fi 6", "Redes", 119.0), ("Switch 8 puertos", "Redes", 34.9),
                ("Soporte portátil", "Accesorios", 24.0), ("Hub USB-C", "Accesorios", 39.9)]
        return [Producto(f"P{i:03d}", n, c, p, (i * 7) % 45) for i, (n, c, p) in enumerate(base, 1)]

    def guardar(self) -> None:
        datos = [asdict(p) for p in self.productos.values()]
        self.ruta.write_text(json.dumps(datos, ensure_ascii=False, indent=2), encoding="utf-8")

    def siguiente_codigo(self) -> str:
        numeros = [int(c[1:]) for c in self.productos if c[1:].isdigit()]
        return f"P{max(numeros, default=0) + 1:03d}"


# ------------------------------------------------------------------ Widgets
class TarjetaKPI(Vertical):
    DEFAULT_CSS = """
    TarjetaKPI { border: round $primary; height: 5; padding: 0 1; width: 1fr; }
    TarjetaKPI .titulo { color: $text-muted; }
    TarjetaKPI .valor { text-style: bold; color: $accent; }
    TarjetaKPI.alerta { border: round $error; }
    TarjetaKPI.alerta .valor { color: $error; }
    """
    valor = reactive("—")

    def __init__(self, titulo: str, **kwargs) -> None:
        super().__init__(**kwargs)
        self.titulo = titulo

    def compose(self) -> ComposeResult:
        yield Label(self.titulo, classes="titulo")
        yield Label(self.valor, classes="valor")

    def watch_valor(self, valor: str) -> None:
        if self.is_mounted:
            self.query_one(".valor", Label).update(valor)


class FormularioProducto(ModalScreen[Producto | None]):
    DEFAULT_CSS = """
    FormularioProducto { align: center middle; }
    #formulario { width: 64; height: auto; border: thick $primary; background: $surface; padding: 1 2; }
    #formulario Label { margin-top: 1; }
    #formulario .error { color: $error; height: 1; margin: 0; }
    #botones { height: auto; align-horizontal: right; margin-top: 1; }
    #botones Button { margin-left: 1; }
    """
    BINDINGS = [("escape", "cancelar", "Cancelar")]

    def __init__(self, producto: Producto, nuevo: bool) -> None:
        super().__init__()
        self.producto, self.nuevo = producto, nuevo

    def compose(self) -> ComposeResult:
        p = self.producto
        with Vertical(id="formulario"):
            yield Static(f"[b]{'Nuevo producto' if self.nuevo else 'Editar ' + p.codigo}[/b]")
            yield Label("Nombre")
            yield Input(p.nombre, id="nombre", validators=[Length(minimum=3, failure_description="Mínimo 3 caracteres")])
            yield Label("Categoría")
            yield Select([(c, c) for c in CATEGORIAS], value=p.categoria, allow_blank=False, id="categoria")
            with Horizontal(classes="fila"):
                with Vertical():
                    yield Label("Precio (€)")
                    yield Input(f"{p.precio:.2f}", id="precio", type="number",
                                validators=[Number(minimum=0.01, failure_description="Precio no válido")])
                with Vertical():
                    yield Label("Stock")
                    yield Input(str(p.stock), id="stock", type="integer",
                                validators=[Number(minimum=0, failure_description="Stock no válido")])
            yield Label("", id="error", classes="error")
            with Horizontal(id="botones"):
                yield Button("Cancelar", id="cancelar")
                yield Button("Guardar", id="guardar", variant="primary")

    def on_mount(self) -> None:
        self.query_one("#nombre", Input).focus()

    @on(Button.Pressed, "#guardar")
    @on(Input.Submitted)
    def guardar(self) -> None:
        for entrada in self.query(Input):
            resultado = entrada.validate(entrada.value)
            if resultado and not resultado.is_valid:
                self.query_one("#error", Label).update(f"⚠ {resultado.failure_descriptions[0]}")
                entrada.focus()
                return
        self.dismiss(Producto(
            codigo=self.producto.codigo,
            nombre=self.query_one("#nombre", Input).value.strip(),
            categoria=self.query_one("#categoria", Select).value,
            precio=round(float(self.query_one("#precio", Input).value), 2),
            stock=int(self.query_one("#stock", Input).value),
        ))

    @on(Button.Pressed, "#cancelar")
    def action_cancelar(self) -> None:
        self.dismiss(None)


class Confirmar(ModalScreen[bool]):
    DEFAULT_CSS = """
    Confirmar { align: center middle; }
    #caja { width: 56; height: auto; border: thick $error; background: $surface; padding: 1 2; }
    #caja Horizontal { height: auto; align-horizontal: right; margin-top: 1; }
    #caja Button { margin-left: 1; }
    """
    BINDINGS = [("escape", "app.pop_screen", "Cancelar")]

    def __init__(self, pregunta: str) -> None:
        super().__init__()
        self.pregunta = pregunta

    def compose(self) -> ComposeResult:
        with Vertical(id="caja"):
            yield Label(self.pregunta)
            with Horizontal():
                yield Button("Cancelar", id="no")
                yield Button("Eliminar", id="si", variant="error")

    @on(Button.Pressed)
    def responder(self, evento: Button.Pressed) -> None:
        self.dismiss(evento.button.id == "si")


# ------------------------------------------------------------------ Aplicación
class Inventario(App):
    TITLE = "Inventario"
    SUB_TITLE = "Gestión de productos"
    CSS = """
    #kpis { height: 5; margin: 0 1; }
    #filtros { height: 3; margin: 1 1 0 1; }
    #busqueda { width: 1fr; }
    #categoria-filtro { width: 30; margin-left: 1; }
    #tabla { margin: 0 1; height: 1fr; }
    #progreso { margin: 0 1; display: none; }
    #progreso.visible { display: block; }
    """
    BINDINGS = [
        Binding("n", "nuevo", "Nuevo"),
        Binding("e", "editar", "Editar"),
        Binding("delete", "eliminar", "Eliminar"),
        Binding("ctrl+e", "exportar", "Exportar CSV", priority=True),
        Binding("/", "buscar", "Buscar"),
        Binding("q", "quit", "Salir"),
    ]
    COLUMNAS = [("codigo", "Código"), ("nombre", "Producto"), ("categoria", "Categoría"),
                ("precio", "Precio"), ("stock", "Stock"), ("valor", "Valor")]

    # init=False: el watcher no se ejecuta al leer el valor por primera vez
    filtro_texto = reactive("", init=False)
    filtro_categoria = reactive("", init=False)

    def __init__(self, ruta: Path = RUTA_DATOS) -> None:
        super().__init__()
        self.repo = Repositorio(ruta)

    # ---------- Construcción
    def compose(self) -> ComposeResult:
        yield Header(show_clock=True)
        with Horizontal(id="kpis"):
            yield TarjetaKPI("Productos", id="kpi-productos")
            yield TarjetaKPI("Unidades en stock", id="kpi-unidades")
            yield TarjetaKPI("Valor del inventario", id="kpi-valor")
            yield TarjetaKPI(f"Stock bajo (< {STOCK_MINIMO})", id="kpi-bajo")
        with Horizontal(id="filtros"):
            yield Input(placeholder="Buscar por código o nombre…", id="busqueda")
            yield Select([(c, c) for c in CATEGORIAS], prompt="Todas las categorías", id="categoria-filtro")
        yield DataTable(id="tabla", zebra_stripes=True, cursor_type="row")
        yield ProgressBar(id="progreso", show_eta=False)
        yield Footer()

    def on_mount(self) -> None:
        self.register_theme(Theme(name="corporativo", primary="#2E75B6", secondary="#1F4E78",
                                  accent="#F5A623", success="#2CA02C", warning="#F5A623", error="#E5484D",
                                  background="#0F1720", surface="#16212C", panel="#1D2B38", dark=True))
        self.theme = "corporativo"
        tabla = self.query_one(DataTable)
        for clave, titulo in self.COLUMNAS:
            tabla.add_column(titulo, key=clave)
        self.refrescar()
        tabla.focus()

    def get_system_commands(self, screen):
        yield from super().get_system_commands(screen)
        yield SystemCommand("Exportar CSV", "Exporta los productos visibles a CSV", self.action_exportar)
        yield SystemCommand("Restablecer filtros", "Muestra todos los productos", self.limpiar_filtros)

    # ---------- Datos en pantalla
    def productos_visibles(self) -> list[Producto]:
        texto = self.filtro_texto.lower()
        return sorted(
            (p for p in self.repo.productos.values()
             if (not texto or texto in p.codigo.lower() or texto in p.nombre.lower())
             and (not self.filtro_categoria or p.categoria == self.filtro_categoria)),
            key=lambda p: p.codigo)

    def refrescar(self, seleccionar: str | None = None) -> None:
        tabla = self.query_one(DataTable)
        tabla.clear()
        visibles = self.productos_visibles()
        for p in visibles:
            stock = f"[b red]{p.stock}[/]" if p.stock < STOCK_MINIMO else str(p.stock)
            tabla.add_row(p.codigo, p.nombre, p.categoria, f"{p.precio:,.2f} €", stock,
                          f"{p.precio * p.stock:,.2f} €", key=p.codigo)
        if seleccionar and seleccionar in self.repo.productos and any(p.codigo == seleccionar for p in visibles):
            tabla.move_cursor(row=tabla.get_row_index(seleccionar))
        self.actualizar_kpis(visibles)

    def actualizar_kpis(self, visibles: list[Producto]) -> None:
        bajo = sum(p.stock < STOCK_MINIMO for p in visibles)
        self.query_one("#kpi-productos", TarjetaKPI).valor = f"{len(visibles)} de {len(self.repo.productos)}"
        self.query_one("#kpi-unidades", TarjetaKPI).valor = f"{sum(p.stock for p in visibles):,}"
        self.query_one("#kpi-valor", TarjetaKPI).valor = f"{sum(p.precio * p.stock for p in visibles):,.2f} €"
        kpi_bajo = self.query_one("#kpi-bajo", TarjetaKPI)
        kpi_bajo.valor = str(bajo)
        kpi_bajo.set_class(bajo > 0, "alerta")

    def seleccionado(self) -> Producto | None:
        tabla = self.query_one(DataTable)
        if tabla.row_count == 0:
            return None
        clave = tabla.coordinate_to_cell_key(tabla.cursor_coordinate).row_key.value
        return self.repo.productos.get(clave)

    # ---------- Filtros
    @on(Input.Changed, "#busqueda")
    def al_buscar(self, evento: Input.Changed) -> None:
        self.filtro_texto = evento.value.strip()

    @on(Select.Changed, "#categoria-filtro")
    def al_filtrar(self, evento: Select.Changed) -> None:
        self.filtro_categoria = "" if evento.value == Select.BLANK else str(evento.value)

    def watch_filtro_texto(self) -> None:
        self.refrescar()

    def watch_filtro_categoria(self) -> None:
        self.refrescar()

    def limpiar_filtros(self) -> None:
        self.query_one("#busqueda", Input).value = ""
        self.query_one("#categoria-filtro", Select).clear()

    @on(DataTable.RowSelected)
    def al_seleccionar_fila(self) -> None:
        self.action_editar()

    # ---------- Acciones
    def action_buscar(self) -> None:
        self.query_one("#busqueda", Input).focus()

    def action_nuevo(self) -> None:
        nuevo = Producto(self.repo.siguiente_codigo(), "", CATEGORIAS[0], 0.0, 0)
        self.push_screen(FormularioProducto(nuevo, nuevo=True), self._guardar_producto)

    def action_editar(self) -> None:
        if producto := self.seleccionado():
            self.push_screen(FormularioProducto(producto, nuevo=False), self._guardar_producto)

    def _guardar_producto(self, producto: Producto | None) -> None:
        if producto is None:
            return
        es_nuevo = producto.codigo not in self.repo.productos
        self.repo.productos[producto.codigo] = producto
        self.repo.guardar()
        self.refrescar(seleccionar=producto.codigo)
        self.notify(f"{producto.nombre} {'creado' if es_nuevo else 'actualizado'}", title=producto.codigo)
        if producto.stock < STOCK_MINIMO:
            self.notify(f"Stock bajo: {producto.stock} unidades", severity="warning")

    @work
    async def action_eliminar(self) -> None:
        producto = self.seleccionado()
        if producto is None:
            return
        if await self.push_screen_wait(Confirmar(f"¿Eliminar «{producto.nombre}» ({producto.codigo})?")):
            del self.repo.productos[producto.codigo]
            self.repo.guardar()
            self.refrescar()
            self.notify(f"{producto.codigo} eliminado", severity="warning")

    def action_exportar(self) -> None:
        visibles = self.productos_visibles()
        if not visibles:
            self.notify("No hay productos que exportar", severity="error")
            return
        barra = self.query_one("#progreso", ProgressBar)
        barra.update(total=len(visibles), progress=0)
        barra.add_class("visible")
        self.exportar_csv(visibles, Path(f"inventario_{time.strftime('%Y%m%d_%H%M%S')}.csv"))

    @work(thread=True, exclusive=True, group="exportacion")
    def exportar_csv(self, productos: list[Producto], ruta: Path) -> None:
        worker = get_current_worker()
        barra = self.query_one("#progreso", ProgressBar)
        with ruta.open("w", newline="", encoding="utf-8-sig") as f:
            escritor = csv.writer(f, delimiter=";")
            escritor.writerow(["Código", "Producto", "Categoría", "Precio", "Stock"])
            for p in productos:
                if worker.is_cancelled:
                    return
                escritor.writerow([p.codigo, p.nombre, p.categoria, f"{p.precio:.2f}".replace(".", ","), p.stock])
                time.sleep(0.05)                                   # Simula un proceso más lento
                self.call_from_thread(barra.advance, 1)
        self.call_from_thread(barra.remove_class, "visible")
        self.call_from_thread(self.notify, f"{len(productos)} productos exportados a {ruta.name}",
                              title="Exportación completada")


def main() -> None:
    Inventario().run()


if __name__ == "__main__":
    main()
```

Prueba automatizada del ejemplo:

```python
# test_inventario.py — pip install pytest pytest-asyncio
import pytest
from inventario import Inventario

@pytest.mark.asyncio
async def test_crear_y_filtrar(tmp_path):
    app = Inventario(ruta=tmp_path / "inv.json")
    async with app.run_test(size=(120, 40)) as pilot:
        await pilot.press("n")                                  # Abre el formulario
        await pilot.press(*"Webcam HD")                         # El foco está en «Nombre»
        app.screen.query_one("#precio").value = "45.50"         # Rellenar campos directamente
        app.screen.query_one("#stock").value = "12"
        await pilot.click("#guardar")
        await pilot.pause()
        assert any(p.nombre == "Webcam HD" for p in app.repo.productos.values())

        await pilot.press("/", *"webcam")
        await pilot.pause()
        assert app.query_one("DataTable").row_count == 1
```

---

## 20. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `NoMatches: No nodes match '#x'` | El widget no existe todavía o el selector es incorrecto. | Consulte tras `on_mount`; revise el `id`. |
| `TooManyMatches` en `query_one` | Varios widgets coinciden. | Use un `id` o un selector más específico. |
| La interfaz se congela | Código lento en un manejador. | `@work(thread=True)` o funciones `async`. |
| `RuntimeError` / comportamiento errático al tocar widgets desde un hilo | Modificación de la UI fuera del bucle principal. | `self.call_from_thread(...)`. |
| `push_screen_wait` lanza `NoActiveWorker` | Se llamó fuera de un *worker*. | Decore el método con `@work` o use el *callback* de `push_screen`. |
| Un atajo no responde con un `Input` enfocado | El `Input` consume la tecla. | `Binding(..., priority=True)` o use combinaciones con Ctrl. |
| El CSS no se aplica | Selector incorrecto o especificidad menor que el estilo por defecto. | Use `textual run --dev` y la consola; aumente la especificidad. |
| `watch_` falla al iniciar o se ejecuta dos veces (p. ej. `DuplicateKey` en una tabla) | Por defecto (`init=True`) el *watcher* se llama al inicializar el reactivo, incluso en su primera lectura. | Declare el reactivo con `reactive(valor, init=False)` o compruebe `self.is_mounted`. |
| Colores pobres o caracteres extraños | Terminal sin color verdadero o fuente sin Unicode. | Windows Terminal, iTerm2, kitty… y una fuente moderna (Nerd Font). |
| `print()` no muestra nada | La TUI ocupa la pantalla. | `self.log(...)` y `textual console`. |

---

## 21. Buenas prácticas

- Separe la lógica de negocio (clases de datos y servicios) de los widgets.
- Ponga el CSS en archivos `.tcss` (`CSS_PATH`) y use variables de tema (`$primary`, `$error`).
- Prefiera `@on(Mensaje, "selector")` a manejadores genéricos con condiciones sobre `id`.
- Use reactivos para el estado que afecta a la interfaz y deje que `watch_` actualice la vista.
- Ejecute toda operación lenta en un *worker*; desde hilos, toque la UI solo con `call_from_thread`.
- Defina `BINDINGS` con descripciones claras: el `Footer` sirve de ayuda contextual.
- Use `ModalScreen` para formularios y confirmaciones, y devuelva el resultado con `dismiss()`.
- Escriba pruebas con `run_test` y `Pilot` para los flujos principales.
- Desarrolle con `textual run --dev` y `textual console` abiertos.

---

## 22. Referencia rápida

```python
class MiApp(App):
    CSS_PATH = "app.tcss"; TITLE = "…"; BINDINGS = [("q", "quit", "Salir"), Binding("ctrl+s", "guardar", "Guardar", priority=True)]
    def compose(self) -> ComposeResult: yield Header(); with Horizontal(): yield ...; yield Footer()
    def on_mount(self): ...
    @on(Button.Pressed, "#id") def f(self, evento): ...
    def action_guardar(self): ...
MiApp().run()

Contenedores: Vertical Horizontal VerticalScroll Grid Container Center Middle VerticalGroup HorizontalGroup
Widgets: Static Label Button Input MaskedInput TextArea Checkbox Switch RadioSet Select SelectionList
         DataTable ListView OptionList Tree DirectoryTree TabbedContent/TabPane Collapsible ContentSwitcher
         ProgressBar LoadingIndicator RichLog Log Markdown Digits Sparkline Placeholder
x = reactive(v) → watch_x / validate_x / compute_x · var(v) · widget.data_bind(...)
self.query_one("#id", Tipo) · self.query(".clase") · mount() · remove() · add_class/remove_class/set_class
push_screen(Pantalla(), callback) · await push_screen_wait(...) (en @work) · dismiss(valor) · pop_screen()
@work(exclusive=True) async · @work(thread=True) + call_from_thread · set_interval / set_timer
notify(msg, severity="warning") · theme = "nord" · register_theme(Theme(...)) · suspend() · exit(valor)
async with app.run_test() as pilot: await pilot.press(...); await pilot.click("#id"); await pilot.pause()
textual run --dev app.py · textual console · textual serve "python app.py"
```

---

## 23. Ejercicios propuestos

1. Cree un cronómetro con `Digits`, botones Iniciar/Parar/Reiniciar, atributos reactivos y `set_interval`.
2. Desarrolle un visor de archivos con `DirectoryTree` a la izquierda y el contenido a la derecha (`TextArea` en modo lectura con resaltado según la extensión).
3. Añada al inventario una pestaña (`TabbedContent`) con un historial de movimientos de stock en un `RichLog`.
4. Implemente un cliente de una API pública que cargue datos con `@work` asíncrono, muestre `loading = True` mientras espera y gestione los errores con notificaciones.
5. Escriba una batería de pruebas con `Pilot` para el inventario que cubra creación, edición, borrado con confirmación y filtros.

---

## 24. Referencias

- Documentación oficial: https://textual.textualize.io/
- Guía: https://textual.textualize.io/guide/
- Galería de widgets: https://textual.textualize.io/widget_gallery/
- Repositorio y ejemplos: https://github.com/Textualize/textual
- Rich (motor de renderizado): https://rich.readthedocs.io/
