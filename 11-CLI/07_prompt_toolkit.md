# Manual de uso: prompt_toolkit

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Construir interfaces interactivas de bajo nivel en terminal
> **Tipo:** Librería de terceros (`pip install prompt_toolkit`) · **Versión de referencia:** 3.0

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [La función `prompt`](#4-la-función-prompt)
5. [`PromptSession`: sesiones e historial](#5-promptsession-sesiones-e-historial)
6. [Autocompletado](#6-autocompletado)
7. [Sugerencias automáticas](#7-sugerencias-automáticas)
8. [Validación](#8-validación)
9. [Resaltado de sintaxis](#9-resaltado-de-sintaxis)
10. [Texto con formato y estilos](#10-texto-con-formato-y-estilos)
11. [Barra inferior, prompt derecho y marcadores](#11-barra-inferior-prompt-derecho-y-marcadores)
12. [Atajos de teclado](#12-atajos-de-teclado)
13. [Entrada multilínea y modos de edición](#13-entrada-multilínea-y-modos-de-edición)
14. [Confirmaciones, selección y diálogos](#14-confirmaciones-selección-y-diálogos)
15. [Barras de progreso](#15-barras-de-progreso)
16. [asyncio y `patch_stdout`](#16-asyncio-y-patch_stdout)
17. [Aplicaciones a pantalla completa](#17-aplicaciones-a-pantalla-completa)
18. [Pruebas](#18-pruebas)
19. [Ejemplo práctico completo](#19-ejemplo-práctico-completo)
20. [Errores frecuentes](#20-errores-frecuentes)
21. [Buenas prácticas](#21-buenas-prácticas)
22. [Referencia rápida](#22-referencia-rápida)
23. [Ejercicios propuestos](#23-ejercicios-propuestos)
24. [Referencias](#24-referencias)

---

## 1. Descripción

**prompt_toolkit** es una librería para construir interfaces de terminal interactivas. Ofrece desde un sustituto avanzado de `input()` (con historial, autocompletado, resaltado, validación y atajos) hasta aplicaciones completas a pantalla completa con distribución en paneles. Es la base de herramientas como **IPython**, **ptpython**, **pgcli**, **mycli**, **questionary** e **InquirerPy**.

**Ventajas:**
- Control total del teclado, la pantalla y el ratón.
- Historial persistente, búsqueda en el historial (Ctrl+R) y sugerencias tipo *fish shell*.
- Autocompletado síncrono, asíncrono o en hilo, con menú en columnas.
- Resaltado de sintaxis mediante Pygments.
- Modos de edición Emacs y Vi.
- Compatible con `asyncio`.
- Multiplataforma (Windows, macOS, Linux), sin dependencias pesadas.

**Limitaciones:**
- API de bajo nivel: tareas sencillas requieren más código que con questionary o InquirerPy.
- Las aplicaciones a pantalla completa exigen conocer su sistema de *layout*; para aplicaciones TUI complejas, Textual es más productivo.
- Requiere una terminal interactiva.

| Si necesita… | Opción |
|---|---|
| REPL propio, shell interactiva, historial, atajos personalizados | **prompt_toolkit** |
| Preguntas tipo formulario con poco código | questionary / InquirerPy |
| Aplicación TUI completa con widgets y CSS | Textual |
| Salida con formato (tablas, paneles) | Rich |

---

## 2. Instalación e importación

```bash
pip install prompt_toolkit
pip install pygments            # Opcional: resaltado de sintaxis
```

Importaciones habituales:

```python
from prompt_toolkit import prompt, PromptSession, print_formatted_text, HTML, ANSI
from prompt_toolkit.history import FileHistory, InMemoryHistory
from prompt_toolkit.auto_suggest import AutoSuggestFromHistory
from prompt_toolkit.completion import WordCompleter, NestedCompleter, PathCompleter, FuzzyCompleter
from prompt_toolkit.validation import Validator, ValidationError
from prompt_toolkit.key_binding import KeyBindings
from prompt_toolkit.styles import Style
from prompt_toolkit.patch_stdout import patch_stdout
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `prompt()` | Función que sustituye a `input()` con funciones avanzadas. |
| `PromptSession` | Objeto reutilizable que conserva historial y configuración entre llamadas. |
| `Buffer` | Texto editable y posición del cursor. |
| `Completer` | Genera sugerencias de autocompletado (`Completion`). |
| `AutoSuggest` | Sugerencia en gris al final de la línea (aceptar con →). |
| `Validator` | Comprueba la entrada antes de aceptarla. |
| `Lexer` | Colorea el texto mientras se escribe. |
| `KeyBindings` | Asociación de teclas a funciones. |
| Texto con formato | `HTML(...)`, `ANSI(...)` o lista de tuplas `(estilo, texto)`. |
| `Application` | Aplicación completa con *layout*, teclas y estilo. |
| *Layout* | Árbol de contenedores (`HSplit`, `VSplit`, `Window`) que forman la pantalla. |

---

## 4. La función `prompt`

```python
from prompt_toolkit import prompt

nombre = prompt("Nombre: ")
clave = prompt("Contraseña: ", is_password=True)
ciudad = prompt("Ciudad: ", default="Madrid")
```

A diferencia de `input()`, ya incluye: edición con flechas, Inicio/Fin, Ctrl+A/Ctrl+E, borrar palabra (Ctrl+W), deshacer (Ctrl+_), pegado de varias líneas y soporte Unicode completo.

| Excepción | Cuándo |
|---|---|
| `KeyboardInterrupt` | El usuario pulsa Ctrl+C. |
| `EOFError` | El usuario pulsa Ctrl+D con la línea vacía. |

```python
try:
    texto = prompt("> ")
except KeyboardInterrupt:
    print("Cancelado")
except EOFError:
    print("Fin")
```

---

## 5. `PromptSession`: sesiones e historial

Para preguntar varias veces (un REPL), cree una **sesión**: conserva el historial y evita reconstruir la configuración.

```python
from prompt_toolkit import PromptSession
from prompt_toolkit.history import FileHistory
from prompt_toolkit.auto_suggest import AutoSuggestFromHistory

sesion = PromptSession(
    history=FileHistory(".mi_repl_historial"),     # Persistente entre ejecuciones
    auto_suggest=AutoSuggestFromHistory(),
    enable_history_search=True,                    # ↑ busca entradas que empiecen igual
)

while True:
    try:
        texto = sesion.prompt("mi-app> ")
    except KeyboardInterrupt:
        continue                                   # Ctrl+C: descarta la línea
    except EOFError:
        break                                      # Ctrl+D: sale
    procesar(texto)
```

| Historial | Uso |
|---|---|
| `InMemoryHistory()` | Solo durante la ejecución (por defecto en `PromptSession`). |
| `FileHistory(ruta)` | Guarda en un archivo. |
| `ThreadedHistory(FileHistory(ruta))` | Carga el historial en segundo plano (archivos grandes). |

Teclas útiles: **↑/↓** recorren el historial, **Ctrl+R** busca hacia atrás, **Ctrl+S** hacia delante.

> Los parámetros de `PromptSession(...)` se pueden sobrescribir en cada llamada a `sesion.prompt(...)`.

---

## 6. Autocompletado

### 6.1 Lista de palabras

```python
from prompt_toolkit.completion import WordCompleter

comandos = WordCompleter(
    ["ayuda", "listar", "crear", "eliminar", "salir"],
    ignore_case=True,
    meta_dict={"listar": "Muestra los registros", "crear": "Añade un registro"},
)
texto = prompt("> ", completer=comandos, complete_while_typing=True)
```

Pulse **Tab** para abrir el menú; con `complete_while_typing=True` aparece al escribir.

### 6.2 Comandos anidados

```python
from prompt_toolkit.completion import NestedCompleter

completador = NestedCompleter.from_nested_dict({
    "mostrar": {"version": None, "usuarios": None, "tablas": {"--detalle": None}},
    "conectar": {"produccion": None, "pruebas": None},
    "salir": None,
})
```

### 6.3 Rutas de archivos

```python
from prompt_toolkit.completion import PathCompleter

ruta = prompt("Archivo: ", completer=PathCompleter(expanduser=True,
              file_filter=lambda n: n.endswith((".csv", ".xlsx")) or "." not in n))
```

### 6.4 Búsqueda difusa

```python
from prompt_toolkit.completion import FuzzyCompleter, FuzzyWordCompleter

completador = FuzzyCompleter(WordCompleter(lista_de_tablas))
completador = FuzzyWordCompleter(lista_de_tablas)            # Equivalente directo
```

### 6.5 Completador propio

```python
from prompt_toolkit.completion import Completer, Completion

class CompletadorUsuarios(Completer):
    def __init__(self, repositorio):
        self.repo = repositorio

    def get_completions(self, documento, evento):
        palabra = documento.get_word_before_cursor()
        for usuario in self.repo.buscar(palabra):
            yield Completion(
                usuario.login,
                start_position=-len(palabra),          # Sustituye la palabra escrita
                display=usuario.login,
                display_meta=usuario.nombre_completo,
                style="fg:ansigreen" if usuario.activo else "fg:ansired",
            )
```

| Recurso del `Document` | Contenido |
|---|---|
| `text` | Todo el texto. |
| `text_before_cursor` | Texto hasta el cursor. |
| `get_word_before_cursor()` | Palabra que se está escribiendo. |
| `cursor_position` | Posición del cursor. |
| `current_line`, `line_count` | Línea actual y número de líneas. |

### 6.6 Combinar y ejecutar en segundo plano

```python
from prompt_toolkit.completion import merge_completers, ThreadedCompleter

completador = merge_completers([comandos, CompletadorUsuarios(repo)])
sesion = PromptSession(completer=ThreadedCompleter(completador))     # No bloquea la escritura
# o bien: PromptSession(completer=..., complete_in_thread=True)
```

### 6.7 Estilo del menú

```python
from prompt_toolkit.shortcuts import CompleteStyle

prompt("> ", completer=c, complete_style=CompleteStyle.MULTI_COLUMN)   # COLUMN | MULTI_COLUMN | READLINE_LIKE
```

---

## 7. Sugerencias automáticas

Muestran en gris la continuación más probable; se acepta con **→** o **Ctrl+E** (Alt+F acepta solo una palabra).

```python
from prompt_toolkit.auto_suggest import AutoSuggestFromHistory, AutoSuggest, Suggestion

prompt("> ", history=historial, auto_suggest=AutoSuggestFromHistory())
```

Sugerencia propia:

```python
class SugerenciaComandos(AutoSuggest):
    COMANDOS = ["listar usuarios", "listar pedidos", "crear usuario"]

    def get_suggestion(self, buffer, documento):
        texto = documento.text
        for c in self.COMANDOS:
            if texto and c.startswith(texto):
                return Suggestion(c[len(texto):])
        return None
```

---

## 8. Validación

### 8.1 Desde una función

```python
from prompt_toolkit.validation import Validator

validador = Validator.from_callable(
    lambda t: t.isdigit() and 1 <= int(t) <= 65535,
    error_message="Introduzca un puerto entre 1 y 65535",
    move_cursor_to_end=True,
)
puerto = int(prompt("Puerto: ", validator=validador, validate_while_typing=False))
```

### 8.2 Clase propia

```python
import re
from prompt_toolkit.validation import Validator, ValidationError

class ValidadorEmail(Validator):
    def validate(self, documento):
        texto = documento.text
        if not re.match(r"^[\w.+-]+@[\w-]+\.[\w.-]+$", texto):
            posicion = texto.find("@") if "@" in texto else len(texto)
            raise ValidationError(message="Email no válido", cursor_position=posicion)
```

> Con `validate_while_typing=True` (por defecto) el error se muestra mientras se escribe; con `False`, solo al pulsar Enter.

---

## 9. Resaltado de sintaxis

```python
from prompt_toolkit.lexers import PygmentsLexer
from pygments.lexers.sql import SqlLexer
from pygments.lexers.python import PythonLexer

consulta = prompt("SQL> ", lexer=PygmentsLexer(SqlLexer))
codigo = prompt(">>> ", lexer=PygmentsLexer(PythonLexer), multiline=True)
```

Aplicar un tema de Pygments:

```python
from prompt_toolkit.styles import style_from_pygments_cls
from pygments.styles import get_style_by_name

estilo = style_from_pygments_cls(get_style_by_name("monokai"))
prompt("SQL> ", lexer=PygmentsLexer(SqlLexer), style=estilo, include_default_pygments_style=False)
```

Lexer propio sencillo:

```python
from prompt_toolkit.lexers import Lexer

class LexerComandos(Lexer):
    VALIDOS = {"listar", "crear", "eliminar"}

    def lex_document(self, documento):
        def linea(n):
            palabras = documento.lines[n].split(" ")
            resultado = []
            for i, p in enumerate(palabras):
                estilo = ("fg:ansigreen bold" if p in self.VALIDOS else "fg:ansired") if i == 0 else ""
                resultado.append((estilo, p + (" " if i < len(palabras) - 1 else "")))
            return resultado
        return linea
```

---

## 10. Texto con formato y estilos

### 10.1 Imprimir con formato

```python
from prompt_toolkit import print_formatted_text, HTML, ANSI
from prompt_toolkit.formatted_text import FormattedText

print_formatted_text(HTML("<b>Negrita</b> <i>cursiva</i> <u>subrayado</u>"))
print_formatted_text(HTML("<ansired>Error:</ansired> <style fg='#ffaa00' bg='#222222'>aviso</style>"))
print_formatted_text(HTML("<estado>OK</estado>"), style=Style.from_dict({"estado": "#2CA02C bold"}))
print_formatted_text(ANSI("\x1b[32mTexto con códigos ANSI\x1b[0m"))
print_formatted_text(FormattedText([("#ff0066", "Hola "), ("italic", "mundo")]))
```

> En `HTML`, escape el texto variable con `html.escape()` o use `HTML("<b>{}</b>").format(valor)`, que escapa automáticamente.

### 10.2 Prompt con formato

```python
mensaje = HTML("<usuario>ana</usuario>@<host>servidor</host> <ruta>~/datos</ruta>$ ")
estilo = Style.from_dict({
    "usuario": "#2CA02C bold",
    "host": "#1F78B4",
    "ruta": "#E07B00",
    "": "#ffffff",                 # Texto que escribe el usuario
})
prompt(mensaje, style=estilo)
```

### 10.3 Clases de estilo integradas

| Clase | Elemento |
|---|---|
| `completion-menu.completion` / `.current` | Elementos del menú de autocompletado. |
| `completion-menu.meta.completion` | Descripción de cada opción. |
| `auto-suggestion` | Sugerencia automática. |
| `bottom-toolbar` | Barra inferior. |
| `rprompt` | Prompt derecho. |
| `validation-toolbar` | Mensaje de error de validación. |
| `placeholder` | Texto de marcador. |

```python
estilo = Style.from_dict({
    "completion-menu.completion": "bg:#1F4E78 #ffffff",
    "completion-menu.completion.current": "bg:#E07B00 #000000 bold",
    "bottom-toolbar": "bg:#222222 #aaaaaa",
    "auto-suggestion": "#666666 italic",
})
```

Formato de los valores: `#rrggbb`, nombres ANSI (`ansired`, `ansibrightblue`…), `bg:color`, `bold`, `italic`, `underline`, `reverse`, `blink`, `hidden`, `noinherit`.

---

## 11. Barra inferior, prompt derecho y marcadores

```python
from datetime import datetime

def barra_inferior():
    return HTML(f" <b>F2</b> formato · <b>Ctrl+D</b> salir · {datetime.now():%H:%M:%S}")

sesion.prompt(
    "> ",
    bottom_toolbar=barra_inferior,       # Función: se recalcula en cada refresco
    rprompt=HTML("<ansigray>[producción]</ansigray>"),
    placeholder=HTML("<ansigray>Escriba un comando o 'ayuda'</ansigray>"),
    refresh_interval=1,                  # Refrescar cada segundo (reloj)
    show_frame=True,                     # Recuadro alrededor de la entrada (3.0.52+)
)
```

---

## 12. Atajos de teclado

### 12.1 Definir atajos

```python
from prompt_toolkit.key_binding import KeyBindings

kb = KeyBindings()

@kb.add("c-t")
def _(evento):
    """Ctrl+T: inserta la fecha actual."""
    evento.current_buffer.insert_text(datetime.now().strftime("%Y-%m-%d"))

@kb.add("f5")
def _(evento):
    """F5: ejecuta inmediatamente el texto actual."""
    evento.current_buffer.validate_and_handle()

@kb.add("escape", "c")            # Alt+C (Esc seguido de C)
def _(evento):
    evento.current_buffer.text = ""

@kb.add("c-x", "c-s")             # Secuencia Ctrl+X, Ctrl+S
def _(evento):
    guardar(evento.current_buffer.text)

texto = prompt("> ", key_bindings=kb)
```

| Notación | Tecla |
|---|---|
| `"c-a"` … `"c-z"` | Ctrl + letra |
| `"escape", "x"` | Alt + X |
| `"f1"` … `"f24"` | Teclas de función |
| `"up"`, `"down"`, `"left"`, `"right"` | Flechas |
| `"tab"`, `"s-tab"`, `"enter"`, `"escape"`, `"backspace"`, `"delete"` | Teclas especiales |
| `"home"`, `"end"`, `"pageup"`, `"pagedown"` | Navegación |
| `"<any>"` | Cualquier tecla |

### 12.2 Acceso al estado desde el atajo

| Atributo de `evento` | Uso |
|---|---|
| `evento.current_buffer` | Texto, cursor, `insert_text()`, `delete_before_cursor()`. |
| `evento.app` | Aplicación: `exit(result=...)`, `invalidate()`. |
| `evento.app.exit(result=valor)` | Termina el `prompt` devolviendo un valor. |

### 12.3 Atajos condicionales

```python
from prompt_toolkit.filters import Condition, has_completions

modo_seguro = {"activo": True}

@kb.add("c-k", filter=Condition(lambda: not modo_seguro["activo"]))
def _(evento):
    evento.current_buffer.text = ""

@kb.add("enter", filter=has_completions)
def _(evento):
    """Enter con el menú abierto: acepta la sugerencia sin enviar la línea."""
    b = evento.current_buffer
    b.apply_completion(b.complete_state.current_completion or b.complete_state.completions[0])
```

### 12.4 Ejecutar código sin romper la pantalla

```python
from prompt_toolkit.application import run_in_terminal

@kb.add("f1")
def _(evento):
    run_in_terminal(lambda: print("Ayuda: escriba 'listar' o 'crear'"))
```

---

## 13. Entrada multilínea y modos de edición

```python
codigo = prompt(
    ">>> ",
    multiline=True,                                       # Enter = nueva línea; Esc+Enter o Alt+Enter = enviar
    prompt_continuation=lambda ancho, n, wrap: "... ".rjust(ancho),
)
```

Enviar con Enter solo cuando la sentencia está completa (p. ej. termina en `;`):

```python
from prompt_toolkit.filters import Condition
from prompt_toolkit.application import get_app

@Condition
def sentencia_incompleta():
    return not get_app().current_buffer.text.rstrip().endswith(";")

sesion.prompt("SQL> ", multiline=sentencia_incompleta)
```

Modos de edición:

```python
from prompt_toolkit.enums import EditingMode

prompt("> ", vi_mode=True)
prompt("> ", editing_mode=EditingMode.EMACS)     # Por defecto
prompt("> ", enable_open_in_editor=True)         # Ctrl+X Ctrl+E (Emacs) o 'v' (Vi) abre $EDITOR
prompt("> ", mouse_support=True)                 # Clic para mover el cursor y elegir en menús
```

---

## 14. Confirmaciones, selección y diálogos

### 14.1 Confirmación y selección simples

```python
from prompt_toolkit.shortcuts import confirm, choice

if confirm("¿Eliminar el registro?"):        # Responde con y/n
    ...

entorno = choice(                             # Lista con flechas (3.0.52+)
    message="Entorno:",
    options=[("dev", "Desarrollo"), ("pre", "Preproducción"), ("pro", "Producción")],
    default="pre",
)
```

### 14.2 Diálogos a pantalla completa

```python
from prompt_toolkit.shortcuts import (message_dialog, input_dialog, yes_no_dialog,
                                      button_dialog, radiolist_dialog, checkboxlist_dialog)

message_dialog(title="Información", text="Proceso completado.").run()

nombre = input_dialog(title="Nuevo usuario", text="Nombre:").run()             # None si cancela
clave = input_dialog(title="Acceso", text="Contraseña:", password=True).run()

if yes_no_dialog(title="Confirmar", text="¿Desea continuar?", yes_text="Sí", no_text="No").run():
    ...

accion = button_dialog(title="Archivo existente", text="¿Qué desea hacer?",
                       buttons=[("Sobrescribir", "o"), ("Renombrar", "r"), ("Cancelar", None)]).run()

formato = radiolist_dialog(title="Exportar", text="Formato:",
                           values=[("csv", "CSV"), ("json", "JSON"), ("xlsx", "Excel")]).run()

modulos = checkboxlist_dialog(title="Instalación", text="Módulos:",
                              values=[("auth", "Autenticación"), ("pagos", "Pagos")],
                              default_values=["auth"]).run()
```

### 14.3 Diálogo de progreso

```python
from prompt_toolkit.shortcuts import progress_dialog

def tarea(fijar_porcentaje, escribir_log):
    for i in range(101):
        fijar_porcentaje(i)
        if i % 25 == 0:
            escribir_log(f"Paso {i}\n")
        time.sleep(0.02)

progress_dialog(title="Importando", text="Procesando registros…", run_callback=tarea).run()
```

---

## 15. Barras de progreso

```python
from prompt_toolkit.shortcuts import ProgressBar
from prompt_toolkit.formatted_text import HTML

with ProgressBar(title=HTML("<b>Descargando</b>"),
                 bottom_toolbar=HTML(" <b>Ctrl+C</b> cancelar")) as pb:
    for archivo in pb(archivos, label="Archivos"):
        descargar(archivo)
```

Varias barras en paralelo:

```python
import threading

with ProgressBar() as pb:
    def trabajo(nombre, n):
        for _ in pb(range(n), label=nombre):
            time.sleep(0.01)

    hilos = [threading.Thread(target=trabajo, args=(f"Lote {i}", 200 + i * 100)) for i in range(3)]
    for h in hilos: h.start()
    for h in hilos: h.join()
```

> Para barras de progreso sencillas en bucles, tqdm o Rich son más directos.

---

## 16. asyncio y `patch_stdout`

### 16.1 Prompt asíncrono

```python
import asyncio
from prompt_toolkit import PromptSession

async def main():
    sesion = PromptSession()
    while True:
        try:
            texto = await sesion.prompt_async("> ")
        except (EOFError, KeyboardInterrupt):
            return
        await procesar(texto)

asyncio.run(main())
```

### 16.2 Imprimir mientras se escribe (`patch_stdout`)

Si otras tareas imprimen mientras el usuario escribe, `patch_stdout` evita que la salida se mezcle con la línea de entrada:

```python
from prompt_toolkit.patch_stdout import patch_stdout

async def notificaciones():
    while True:
        await asyncio.sleep(3)
        print("🔔 Nuevo mensaje recibido")

async def main():
    sesion = PromptSession()
    tarea = asyncio.create_task(notificaciones())
    with patch_stdout():
        while True:
            try:
                texto = await sesion.prompt_async("chat> ")
            except (EOFError, KeyboardInterrupt):
                break
            print(f"Enviado: {texto}")
    tarea.cancel()
```

---

## 17. Aplicaciones a pantalla completa

### 17.1 Estructura

```python
from prompt_toolkit import Application
from prompt_toolkit.key_binding import KeyBindings
from prompt_toolkit.layout import Layout, HSplit, VSplit, Window
from prompt_toolkit.layout.controls import FormattedTextControl
from prompt_toolkit.widgets import Frame, TextArea, Label

salida = TextArea(text="", read_only=True, scrollbar=True)
entrada = TextArea(height=1, prompt="> ", multiline=False)

def al_enviar(buffer):
    salida.text += f"Recibido: {buffer.text}\n"
    salida.buffer.cursor_position = len(salida.text)      # Desplaza al final
    return False                                          # False: vacía la entrada

entrada.accept_handler = al_enviar

contenedor = HSplit([
    Window(FormattedTextControl(HTML("<b> Consola</b> · Ctrl+Q salir")), height=1, style="reverse"),
    VSplit([
        Frame(salida, title="Salida"),
        Frame(Label(text="Panel lateral"), title="Info", width=30),
    ]),
    Frame(entrada, title="Comando"),
])

kb = KeyBindings()

@kb.add("c-q")
def _(evento):
    evento.app.exit()

@kb.add("tab")
def _(evento):
    evento.app.layout.focus_next()

app = Application(layout=Layout(contenedor, focused_element=entrada), key_bindings=kb,
                  full_screen=True, mouse_support=True)
app.run()
```

### 17.2 Elementos principales

| Elemento | Uso |
|---|---|
| `HSplit([...])` | Apila elementos verticalmente. |
| `VSplit([...])` | Coloca elementos en horizontal. |
| `Window(control, width, height)` | Zona que muestra un control. |
| `FormattedTextControl(texto)` | Texto con formato (puede ser una función). |
| `BufferControl(buffer)` | Texto editable. |
| `ConditionalContainer(c, filter)` | Muestra u oculta según una condición. |
| `FloatContainer` + `Float` | Elementos superpuestos (ventanas emergentes, menús). |
| `widgets.TextArea` | Área de texto completa (editable o de solo lectura). |
| `widgets.Frame`, `Box`, `Label`, `Button`, `Checkbox`, `RadioList`, `Dialog`, `MenuContainer` | Widgets listos para usar. |

### 17.3 Refrescar y tareas en segundo plano

```python
app.invalidate()                                 # Volver a dibujar
app.create_background_task(actualizar_cada_segundo())   # Corrutina asociada a la app
await app.run_async()                            # Ejecutar dentro de asyncio
```

> Para aplicaciones TUI con muchos widgets, tablas o formularios, valore **Textual**: ofrece un modelo de componentes y estilos CSS más productivo.

---

## 18. Pruebas

prompt_toolkit permite simular la entrada de teclado y descartar la salida:

```python
from prompt_toolkit.application import create_app_session
from prompt_toolkit.input import create_pipe_input
from prompt_toolkit.output import DummyOutput
from prompt_toolkit import PromptSession

def test_prompt():
    with create_pipe_input() as entrada:
        entrada.send_text("hola mundo\r")
        with create_app_session(input=entrada, output=DummyOutput()):
            resultado = PromptSession().prompt("> ")
    assert resultado == "hola mundo"

def test_ctrl_c():
    with create_pipe_input() as entrada:
        entrada.send_text("\x03")
        with create_app_session(input=entrada, output=DummyOutput()):
            with pytest.raises(KeyboardInterrupt):
                PromptSession().prompt("> ")
```

| Secuencia | Tecla |
|---|---|
| `"\r"` | Enter |
| `"\t"` | Tab |
| `"\x03"` / `"\x04"` | Ctrl+C / Ctrl+D |
| `"\x1b[A"` / `"\x1b[B"` | Flecha arriba / abajo |
| `"\x1b\r"` | Alt+Enter |

> Cree la `PromptSession` (o el objeto que la contiene) **dentro** de `create_app_session`: la sesión toma la entrada y la salida activas en el momento de crearse. Si la crea antes, seguirá leyendo de la terminal real y la prueba se quedará esperando.

> Si el código lanza **varias** preguntas seguidas, envíe la entrada de cada una de forma escalonada (p. ej. desde un hilo con pausas) o pruebe cada pregunta por separado.

---

## 19. Ejemplo práctico completo

**Cliente SQL interactivo para SQLite**: historial persistente, sugerencias, autocompletado de palabras clave, tablas y columnas (con búsqueda difusa), resaltado SQL, sentencias multilínea terminadas en `;`, comandos especiales, barra inferior dinámica, atajos de teclado y salida en tabla o vertical.

```python
"""
sqlrepl.py — Cliente SQL interactivo para SQLite.
Requiere: pip install prompt_toolkit pygments
Uso: python sqlrepl.py [base_de_datos.db]
"""
import sqlite3
import sys
import time
from pathlib import Path

from prompt_toolkit import HTML, PromptSession, print_formatted_text
from prompt_toolkit.application import get_app
from prompt_toolkit.auto_suggest import AutoSuggestFromHistory
from prompt_toolkit.completion import Completer, Completion, FuzzyCompleter, merge_completers, WordCompleter
from prompt_toolkit.filters import Condition
from prompt_toolkit.history import FileHistory
from prompt_toolkit.key_binding import KeyBindings
from prompt_toolkit.lexers import PygmentsLexer
from prompt_toolkit.styles import Style
from pygments.lexers.sql import SqlLexer

PALABRAS_CLAVE = [
    "SELECT", "FROM", "WHERE", "AND", "OR", "NOT", "IN", "LIKE", "BETWEEN", "IS NULL", "ORDER BY",
    "GROUP BY", "HAVING", "LIMIT", "OFFSET", "JOIN", "LEFT JOIN", "INNER JOIN", "ON", "AS", "DISTINCT",
    "INSERT INTO", "VALUES", "UPDATE", "SET", "DELETE FROM", "CREATE TABLE", "DROP TABLE", "ALTER TABLE",
    "COUNT", "SUM", "AVG", "MIN", "MAX", "CASE", "WHEN", "THEN", "ELSE", "END", "PRAGMA",
]
COMANDOS = {
    ".tablas": "Lista las tablas",
    ".esquema": "Muestra el esquema de una tabla: .esquema <tabla>",
    ".modo": "Alterna entre salida en tabla y vertical",
    ".ayuda": "Muestra esta ayuda",
    ".salir": "Sale del programa",
}

ESTILO = Style.from_dict({
    "prompt": "#1F78B4 bold",
    "continuacion": "#808080",
    "bottom-toolbar": "bg:#1F2933 #CBD2D9",
    "bottom-toolbar.clave": "bg:#1F2933 #F5A623 bold",
    "completion-menu.completion": "bg:#243B53 #ffffff",
    "completion-menu.completion.current": "bg:#F5A623 #000000",
    "completion-menu.meta.completion": "bg:#334E68 #BCCCDC",
    "auto-suggestion": "#627D98 italic",
    "cabecera": "#1F78B4 bold",
    "nulo": "#808080 italic",
    "ok": "#2CA02C",
    "error": "#C00000 bold",
})


# ------------------------------------------------------------------ Autocompletado del esquema
class CompletadorEsquema(Completer):
    """Sugiere tablas y columnas leídas de la base de datos."""

    def __init__(self, conexion: sqlite3.Connection):
        self.conexion = conexion
        self.recargar()

    def recargar(self) -> None:
        self.tablas: dict[str, list[str]] = {}
        for (tabla,) in self.conexion.execute("SELECT name FROM sqlite_master WHERE type IN ('table','view')"):
            self.tablas[tabla] = [c[1] for c in self.conexion.execute(f'PRAGMA table_info("{tabla}")')]

    def get_completions(self, documento, evento):
        palabra = documento.get_word_before_cursor(WORD=True)
        if palabra.startswith("."):
            for cmd, ayuda in COMANDOS.items():
                if cmd.startswith(palabra):
                    yield Completion(cmd, -len(palabra), display_meta=ayuda)
            return
        for tabla, columnas in self.tablas.items():
            yield Completion(tabla, -len(palabra), display_meta="tabla", style="fg:#F5A623")
            for col in columnas:
                yield Completion(col, -len(palabra), display_meta=f"columna de {tabla}")


# ------------------------------------------------------------------ Aplicación
class ClienteSQL:
    def __init__(self, ruta_bd: str):
        self.ruta = ruta_bd
        self.conexion = sqlite3.connect(ruta_bd)
        self.vertical = False
        self.ultima_duracion = 0.0
        self.ultimas_filas = 0
        self.esquema = CompletadorEsquema(self.conexion)
        self.sesion = PromptSession(
            history=FileHistory(str(Path.home() / ".sqlrepl_historial")),
            auto_suggest=AutoSuggestFromHistory(),
            completer=FuzzyCompleter(merge_completers([
                WordCompleter(PALABRAS_CLAVE, ignore_case=True),
                self.esquema,
            ])),
            complete_while_typing=True,
            lexer=PygmentsLexer(SqlLexer),
            style=ESTILO,
            multiline=Condition(self._sentencia_incompleta),
            prompt_continuation=lambda ancho, n, wrap: HTML("<continuacion>{}</continuacion>").format("...> ".rjust(ancho)),
            bottom_toolbar=self._barra,
            key_bindings=self._atajos(),
            enable_history_search=True,
        )

    # ---------- Interfaz
    @staticmethod
    def _sentencia_incompleta() -> bool:
        texto = get_app().current_buffer.text.strip()
        return bool(texto) and not texto.startswith(".") and not texto.endswith(";")

    def _barra(self):
        modo = "vertical" if self.vertical else "tabla"
        return HTML(
            f" <b>{Path(self.ruta).name}</b> · {len(self.esquema.tablas)} tablas · modo: {modo} · "
            f"última: {self.ultimas_filas} filas en {self.ultima_duracion * 1000:.1f} ms   "
            "<clave>F2</clave> modo  <clave>F5</clave> recargar  <clave>Ctrl+D</clave> salir"
        )

    def _atajos(self) -> KeyBindings:
        kb = KeyBindings()

        @kb.add("f2")
        def _(evento):
            self.vertical = not self.vertical
            evento.app.invalidate()

        @kb.add("f5")
        def _(evento):
            self.esquema.recargar()
            evento.app.invalidate()

        @kb.add("c-l")
        def _(evento):
            evento.app.renderer.clear()

        return kb

    # ---------- Salida
    def _mostrar(self, cursor: sqlite3.Cursor) -> None:
        if cursor.description is None:
            print_formatted_text(HTML("<ok>✔ {} fila(s) afectada(s)</ok>").format(cursor.rowcount), style=ESTILO)
            self.ultimas_filas = max(cursor.rowcount, 0)
            return
        columnas = [d[0] for d in cursor.description]
        filas = cursor.fetchmany(500)
        self.ultimas_filas = len(filas)
        if not filas:
            print_formatted_text(HTML("<nulo>(sin resultados)</nulo>"), style=ESTILO)
            return
        texto = [["NULL" if v is None else str(v) for v in fila] for fila in filas]
        if self.vertical:
            ancho = max(map(len, columnas))
            for i, fila in enumerate(texto, 1):
                print_formatted_text(HTML("<cabecera>── Fila {} ──</cabecera>").format(i), style=ESTILO)
                for col, valor in zip(columnas, fila):
                    print(f"  {col.rjust(ancho)} │ {valor}")
        else:
            anchos = [min(40, max(len(c), *(len(f[i]) for f in texto))) for i, c in enumerate(columnas)]
            linea = "─┼─".join("─" * a for a in anchos)
            print_formatted_text(HTML("<cabecera>{}</cabecera>").format(
                " │ ".join(c.ljust(a)[:a] for c, a in zip(columnas, anchos))), style=ESTILO)
            print(linea)
            for fila in texto:
                print(" │ ".join(v.ljust(a)[:a] for v, a in zip(fila, anchos)))
        aviso = " (límite de 500)" if len(filas) == 500 else ""
        print_formatted_text(HTML("<nulo>{} fila(s){}</nulo>").format(len(filas), aviso), style=ESTILO)

    def _error(self, mensaje: str) -> None:
        print_formatted_text(HTML("<error>Error:</error> {}").format(mensaje), style=ESTILO)

    # ---------- Comandos
    def _comando(self, texto: str) -> bool:
        partes = texto.split()
        cmd = partes[0].lower()
        if cmd == ".salir":
            return False
        if cmd == ".tablas":
            for tabla, cols in self.esquema.tablas.items():
                print(f"  {tabla} ({len(cols)} columnas)")
        elif cmd == ".esquema":
            if len(partes) < 2 or partes[1] not in self.esquema.tablas:
                self._error("indique una tabla existente: .esquema <tabla>")
            else:
                sql = self.conexion.execute("SELECT sql FROM sqlite_master WHERE name = ?", (partes[1],)).fetchone()[0]
                print(sql)
        elif cmd == ".modo":
            self.vertical = not self.vertical
            print(f"Modo de salida: {'vertical' if self.vertical else 'tabla'}")
        elif cmd == ".ayuda":
            for c, ayuda in COMANDOS.items():
                print(f"  {c:<10} {ayuda}")
            print("  Las sentencias SQL terminan en ';' y pueden ocupar varias líneas.")
        else:
            self._error(f"comando desconocido {cmd!r}. Escriba .ayuda")
        return True

    def ejecutar(self, texto: str) -> None:
        inicio = time.perf_counter()
        try:
            with self.conexion:
                cursor = self.conexion.execute(texto.rstrip().rstrip(";"))
                self._mostrar(cursor)
        except sqlite3.Error as e:
            self._error(str(e))
        finally:
            self.ultima_duracion = time.perf_counter() - inicio
        if texto.lstrip().upper().startswith(("CREATE", "DROP", "ALTER")):
            self.esquema.recargar()

    # ---------- Bucle principal
    def ejecutar_repl(self) -> None:
        print_formatted_text(HTML("<prompt>sqlrepl</prompt> · SQLite {} · escriba <b>.ayuda</b>").format(
            sqlite3.sqlite_version), style=ESTILO)
        while True:
            try:
                texto = self.sesion.prompt(HTML("<prompt>sql&gt; </prompt>")).strip()
            except KeyboardInterrupt:
                continue
            except EOFError:
                break
            if not texto:
                continue
            if texto.startswith("."):
                if not self._comando(texto):
                    break
            else:
                self.ejecutar(texto)
        self.conexion.close()
        print("Hasta luego.")


def crear_demo(ruta: str) -> None:
    if Path(ruta).exists():
        return
    with sqlite3.connect(ruta) as c:
        c.executescript("""
            CREATE TABLE clientes (id INTEGER PRIMARY KEY, nombre TEXT, ciudad TEXT, alta DATE);
            CREATE TABLE pedidos (id INTEGER PRIMARY KEY, cliente_id INTEGER REFERENCES clientes(id),
                                  importe REAL, fecha DATE);
            INSERT INTO clientes (nombre, ciudad, alta) VALUES
                ('Ana Pérez', 'Madrid', '2025-01-10'), ('Luis Gómez', 'Sevilla', '2025-03-02'),
                ('Marta Ruiz', 'Valencia', '2025-06-21');
            INSERT INTO pedidos (cliente_id, importe, fecha) VALUES
                (1, 120.5, '2026-10-01'), (1, 80.0, '2026-11-03'), (2, 45.9, '2026-11-15'), (3, 300.0, '2026-11-20');
        """)


if __name__ == "__main__":
    ruta = sys.argv[1] if len(sys.argv) > 1 else "demo.db"
    crear_demo(ruta)
    ClienteSQL(ruta).ejecutar_repl()
```

Sesión de ejemplo:

```
sql> SELECT c.nombre, SUM(p.importe) AS total
...>   FROM clientes c JOIN pedidos p ON p.cliente_id = c.id
...>  GROUP BY c.nombre ORDER BY total DESC;
nombre     │ total
───────────┼──────
Marta Ruiz │ 300.0
Ana Pérez  │ 200.5
Luis Gómez │ 45.9
3 fila(s)
sql> .esquema pedidos
```

---

## 20. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `NoConsoleScreenBufferError` / `Input is not a terminal` | Ejecución sin terminal (IDE, tubería, CI). | Ejecute en una terminal real o use `create_pipe_input` en pruebas. |
| La salida de otros hilos o tareas rompe la línea de entrada | Se imprime mientras el prompt está activo. | `with patch_stdout():`. |
| `RuntimeError: This event loop is already running` | `prompt()` síncrono dentro de `asyncio`. | `await sesion.prompt_async()`. |
| Enter no envía en modo multilínea | `multiline=True` usa Esc+Enter para enviar. | Use una `Condition` o un atajo propio para Enter. |
| El texto variable dentro de `HTML(...)` produce errores de XML | Caracteres `<`, `&` sin escapar. | `HTML("...{}...").format(valor)` o `html.escape()`. |
| El autocompletado bloquea la escritura | El completador es lento (consulta a BD, red). | `complete_in_thread=True` o `ThreadedCompleter`. |
| No aparece el resaltado | Falta Pygments. | `pip install pygments`. |
| La barra inferior no se actualiza | Es texto fijo, no una función, o falta refresco. | Pase una función y use `refresh_interval` o `app.invalidate()`. |
| Las teclas Alt no funcionan en macOS | La terminal no envía Meta. | Active «Option como Meta» en la terminal. |
| En pruebas el `prompt` se queda esperando | La `PromptSession` se creó antes de `create_app_session`. | Cree la sesión dentro del bloque `with`. |
| En pruebas solo responde la primera pregunta | Toda la entrada simulada se consume de una vez. | Envíe la entrada escalonada o pruebe cada pregunta por separado. |

---

## 21. Buenas prácticas

- Use `PromptSession` para cualquier bucle de preguntas: mantiene historial y configuración.
- Guarde el historial en la carpeta del usuario (`Path.home()`), no en la del proyecto.
- Gestione siempre `KeyboardInterrupt` (descartar línea) y `EOFError` (salir) en los REPL.
- Ejecute los completadores costosos en un hilo y limite el número de sugerencias.
- Escape siempre el texto variable en `HTML`.
- Centralice los estilos en un único `Style.from_dict`.
- Muestre los atajos disponibles en la barra inferior.
- Use `patch_stdout` cuando haya tareas en segundo plano que impriman.
- Para formularios sencillos, use questionary o InquirerPy; reserve prompt_toolkit para REPL y necesidades a medida.

---

## 22. Referencia rápida

```python
prompt(msg, default=, is_password=, multiline=, completer=, validator=, lexer=, style=, key_bindings=,
       auto_suggest=, history=, bottom_toolbar=, rprompt=, placeholder=, vi_mode=, mouse_support=)
sesion = PromptSession(...); sesion.prompt(msg) · await sesion.prompt_async(msg)
FileHistory(ruta) · InMemoryHistory() · AutoSuggestFromHistory()
WordCompleter([...], meta_dict=) · NestedCompleter.from_nested_dict({...}) · PathCompleter()
FuzzyCompleter(c) · merge_completers([...]) · ThreadedCompleter(c) · Completer.get_completions → Completion
Validator.from_callable(f, error_message=) · ValidationError(message=, cursor_position=)
PygmentsLexer(SqlLexer) · Style.from_dict({"clase": "fg bg bold"})
print_formatted_text(HTML("<b>x</b>") | ANSI("...") | FormattedText([(estilo, texto)]), style=)
kb = KeyBindings(); @kb.add("c-t") / ("escape", "x") / ("f5", filter=Condition(f))
confirm("¿?") · choice(message, options=[(valor, texto)]) · *_dialog(...).run() · ProgressBar()
patch_stdout() · run_in_terminal(f) · get_app()
Application(layout=Layout(HSplit/VSplit([...])), key_bindings=, full_screen=True).run()
create_pipe_input() + create_app_session(input=, output=DummyOutput())   # Pruebas
```

---

## 23. Ejercicios propuestos

1. Cree una calculadora REPL con historial persistente, validación de expresiones y una barra inferior que muestre el último resultado.
2. Implemente una mini *shell* con `NestedCompleter` para los comandos `ls`, `cd`, `cat` y autocompletado de rutas en sus argumentos.
3. Añada al cliente SQL un comando `.exportar <archivo.csv>` que guarde el resultado de la última consulta.
4. Construya un chat simulado con `asyncio`: una tarea genera mensajes aleatorios y el usuario escribe con `prompt_async` dentro de `patch_stdout`.
5. Desarrolle una aplicación a pantalla completa con dos paneles: una lista de archivos a la izquierda y su contenido a la derecha, navegable con el teclado.

---

## 24. Referencias

- Documentación oficial: https://python-prompt-toolkit.readthedocs.io/
- Repositorio y ejemplos: https://github.com/prompt-toolkit/python-prompt-toolkit/tree/main/examples
- Pygments (lexers y temas): https://pygments.org/
