# Manual de uso: urwid

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Interfaces de consola clásicas (TUI) en Linux/Unix
> **Tipo:** Librería de terceros (`pip install urwid`) · **Versión de referencia:** 2.6 – 5.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer programa](#4-primer-programa)
5. [Tipos de tamaño: *box*, *flow* y *fixed*](#5-tipos-de-tamaño-box-flow-y-fixed)
6. [Widgets básicos](#6-widgets-básicos)
7. [Decoradores de widgets](#7-decoradores-de-widgets)
8. [Contenedores: `Pile`, `Columns`, `Frame`, `GridFlow`, `Overlay`](#8-contenedores-pile-columns-frame-gridflow-overlay)
9. [Listas desplazables: `ListBox` y *walkers*](#9-listas-desplazables-listbox-y-walkers)
10. [Paleta de colores y atributos](#10-paleta-de-colores-y-atributos)
11. [Teclado, ratón y señales](#11-teclado-ratón-y-señales)
12. [El bucle principal (`MainLoop`)](#12-el-bucle-principal-mainloop)
13. [Widgets personalizados](#13-widgets-personalizados)
14. [Diálogos y ventanas emergentes](#14-diálogos-y-ventanas-emergentes)
15. [Temporizadores, procesos externos y asyncio](#15-temporizadores-procesos-externos-y-asyncio)
16. [Pruebas sin terminal](#16-pruebas-sin-terminal)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**urwid** es una librería veterana (desde 2004) para construir **interfaces de usuario en modo texto** en la consola: menús, formularios, listas desplazables, diálogos y paneles, con soporte de colores, ratón y redimensionado. Es la base de herramientas conocidas como **mitmproxy**, **bpython** (parcialmente), **alot**, **pudb** o **wicd-curses**.

Su modelo se basa en **widgets** que se combinan como piezas, un **bucle principal** que procesa teclado, ratón y eventos, y una **paleta** que asigna colores a nombres de atributo.

**Ventajas:**
- Madura, estable y probada en aplicaciones reales durante muchos años.
- Ligera: sin dependencias obligatorias.
- Funciona en terminales muy básicas y conexiones SSH lentas.
- Control fino del diseño y del redibujado.
- Integración con varios bucles de eventos: propio, `asyncio`, Twisted, Tornado, GLib, Trio, ZeroMQ.
- Soporte de Unicode y caracteres de doble ancho (CJK).

**Limitaciones:**
- Orientada a Linux/Unix; el soporte para Windows existe desde la versión 2.6, pero es más reciente y menos probado.
- API de bajo nivel: los diseños requieren más código que en Textual.
- Sin sistema de estilos tipo CSS ni widgets avanzados como tablas con columnas ordenables.
- Documentación más técnica y menos ejemplos modernos.

| Si necesita… | Opción |
|---|---|
| TUI clásica, ligera, en servidores Unix | **urwid** |
| TUI moderna con CSS, tablas y muchos widgets | Textual |
| REPL o *shell* interactiva | prompt_toolkit |
| Preguntas sencillas tipo formulario | questionary / InquirerPy |
| Control absoluto a muy bajo nivel | `curses` (librería estándar en Unix) |

---

## 2. Instalación e importación

```bash
pip install urwid
```

```python
import urwid
```

Comprobar la versión:

```bash
python -c "import urwid; print(urwid.__version__)"
```

> Muchos ejemplos antiguos de internet usan nombres ya obsoletos (`urwid.raw_display`, `'center'` como cadenas en lugar de constantes…). Siguen funcionando con avisos de obsolescencia, pero conviene seguir la API actual.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Widget | Elemento de la interfaz: texto, campo editable, botón, lista… |
| Contenedor | Widget que contiene otros: `Pile` (vertical), `Columns` (horizontal), `Frame`, `Overlay`. |
| Decorador | Widget que envuelve a otro para modificarlo: `Padding`, `Filler`, `AttrMap`, `LineBox`. |
| Tipo de tamaño | Cómo negocia un widget su tamaño: *box*, *flow* o *fixed* (sección 5). |
| Foco | Widget que recibe el teclado; se propaga por los contenedores. |
| `MainLoop` | Bucle que dibuja la pantalla y reparte teclado, ratón y temporizadores. |
| Paleta | Lista que asocia nombres de atributo (`"titulo"`) con colores. |
| Atributo | Nombre de estilo aplicado a un texto o widget. |
| Señal | Mecanismo de eventos de los widgets (`"click"`, `"change"`). |
| *Walker* | Objeto que proporciona los elementos de un `ListBox`. |
| `unhandled_input` | Función que recibe las teclas que ningún widget ha procesado. |

---

## 4. Primer programa

```python
import urwid

def salir_con_q(tecla):
    if tecla in ("q", "Q"):
        raise urwid.ExitMainLoop()

texto = urwid.Text(("titulo", "¡Hola, urwid!\nPulse Q para salir."), align=urwid.CENTER)
relleno = urwid.Filler(texto, valign=urwid.MIDDLE)
fondo = urwid.AttrMap(relleno, "fondo")

paleta = [
    ("titulo", "white,bold", "dark blue"),
    ("fondo", "", "dark blue"),
]

urwid.MainLoop(fondo, paleta, unhandled_input=salir_con_q).run()
```

Estructura:

```
AttrMap (colores de fondo)
└── Filler (centra verticalmente; convierte flow → box)
    └── Text (contenido)
```

---

## 5. Tipos de tamaño: *box*, *flow* y *fixed*

Es el concepto más importante de urwid y la causa más frecuente de errores.

| Tipo | Quién decide el tamaño | Ejemplos |
|---|---|---|
| **box** | El contenedor da **ancho y alto**. | `ListBox`, `Filler`, `Frame`, `SolidFill`, `Overlay` |
| **flow** | El contenedor da el **ancho**; el widget calcula su alto. | `Text`, `Edit`, `Button`, `CheckBox`, `Divider`, `Pile` de widgets *flow* |
| **fixed** | El widget decide **ancho y alto**. | `BigText`, `SelectableIcon` (según configuración) |

La pantalla principal (`MainLoop`) necesita un widget **box**. Por eso:

```python
urwid.MainLoop(urwid.Text("Hola"))                 # ✖ Error: Text es flow
urwid.MainLoop(urwid.Filler(urwid.Text("Hola")))   # ✔ Filler convierte flow en box
```

Adaptadores entre tipos:

| Adaptador | Conversión |
|---|---|
| `Filler(widget_flow, valign=, height=)` | flow → box |
| `BoxAdapter(widget_box, height=n)` | box → flow (con alto fijo) |
| `Padding(widget, width=)` | Ajusta el ancho (mantiene el tipo) |
| `Pile` con `("weight", n, w)` o `("pack", w)` | Mezcla box y flow en vertical |
| `Columns` con `("weight", n, w)`, `("fixed", n, w)`, `("pack", w)` | Reparte el ancho en horizontal |

```python
print(urwid.Text("x").sizing())        # {Sizing.FLOW, Sizing.FIXED} en versiones recientes
print(urwid.ListBox([]).sizing())      # {Sizing.BOX}
```

---

## 6. Widgets básicos

### 6.1 Texto

```python
urwid.Text("Texto simple")
urwid.Text("Centrado", align=urwid.CENTER)                    # LEFT, CENTER, RIGHT
urwid.Text("Texto largo…", wrap=urwid.WrapMode.ELLIPSIS)      # SPACE (defecto), ANY, CLIP, ELLIPSIS
urwid.Text([("negrita", "Estado: "), ("ok", "ACTIVO"), " desde hoy"])   # Marcado por tramos

t = urwid.Text("")
t.set_text("Nuevo contenido")
t.text                                                         # Contenido actual
```

### 6.2 Campo editable

```python
nombre = urwid.Edit("Nombre: ")
clave = urwid.Edit("Clave: ", mask="*")
notas = urwid.Edit("Notas:\n", multiline=True)
edad = urwid.IntEdit("Edad: ", default=30)

nombre.edit_text                      # Valor
nombre.set_edit_text("Ana")
nombre.set_edit_pos(len(nombre.edit_text))
edad.value()                          # Entero
```

### 6.3 Botones y casillas

```python
def al_pulsar(boton, dato):
    estado.set_text(f"Pulsado: {dato}")

boton = urwid.Button("Guardar", on_press=al_pulsar, user_data="guardar")

casilla = urwid.CheckBox("Recordar sesión", state=True)
casilla.get_state()

grupo = []
r1 = urwid.RadioButton(grupo, "Particular", state=True)
r2 = urwid.RadioButton(grupo, "Empresa")
seleccionado = next(r.label for r in grupo if r.state)
```

### 6.4 Otros

```python
urwid.Divider("─")                              # Línea separadora (flow)
urwid.Divider()                                 # Línea en blanco
urwid.SolidFill("░")                            # Relleno (box)
urwid.ProgressBar("progreso_vacio", "progreso_lleno", current=40, done=100)
urwid.BigText("12:30", urwid.HalfBlock5x4Font())   # Texto grande (fixed)
urwid.SelectableIcon("▶ Elemento", cursor_position=0)   # Texto que puede recibir el foco
```

---

## 7. Decoradores de widgets

```python
# Márgenes y ancho
urwid.Padding(widget, left=2, right=2)
urwid.Padding(widget, align=urwid.CENTER, width=(urwid.RELATIVE, 60))   # 60 % del ancho
urwid.Padding(widget, align=urwid.CENTER, width=40)                     # 40 columnas

# Relleno vertical (flow → box)
urwid.Filler(widget, valign=urwid.TOP, top=1, bottom=1)

# Colores: atributo normal y atributo con foco
urwid.AttrMap(widget, "normal", focus_map="seleccionado")

# Marco con título
urwid.LineBox(widget, title="Datos del cliente", title_align=urwid.LEFT)

# Contenedor que se puede sustituir en caliente
marcador = urwid.WidgetPlaceholder(urwid.Text("Cargando…"))
marcador.original_widget = otra_vista

# Deshabilitar la interacción
urwid.WidgetDisable(widget)
```

Botón con colores al recibir el foco (patrón muy habitual):

```python
def boton(etiqueta, funcion, dato=None):
    b = urwid.Button(etiqueta, on_press=funcion, user_data=dato)
    return urwid.AttrMap(b, "boton", focus_map="boton_foco")
```

---

## 8. Contenedores: `Pile`, `Columns`, `Frame`, `GridFlow`, `Overlay`

### 8.1 `Pile` (vertical)

```python
pila = urwid.Pile([
    urwid.Text("Formulario"),
    urwid.Divider(),
    urwid.Edit("Nombre: "),
    urwid.Edit("Email: "),
    ("weight", 1, urwid.ListBox(elementos)),      # Ocupa el alto sobrante (box)
    ("pack", urwid.Text("Pie")),                  # Alto natural (flow)
])
pila.focus_position = 2                           # Foco en el tercer elemento
```

### 8.2 `Columns` (horizontal)

```python
columnas = urwid.Columns([
    ("fixed", 20, menu_lateral),                  # 20 columnas
    ("weight", 3, contenido),                     # 3 partes del resto
    ("weight", 1, panel_info),                    # 1 parte del resto
], dividechars=1)                                 # Separación entre columnas

urwid.Columns([urwid.Text("Etiqueta"), ("pack", urwid.Text("ajustado"))])
```

### 8.3 `Frame` (cabecera, cuerpo y pie)

```python
marco = urwid.Frame(
    body=urwid.ListBox(elementos),                # box
    header=urwid.AttrMap(urwid.Text(" Mi aplicación"), "cabecera"),   # flow
    footer=urwid.AttrMap(urwid.Text(" F1 Ayuda  Q Salir"), "pie"),    # flow
    focus_part="body",
)
marco.focus_position = "footer"                   # Mover el foco a otra parte
```

### 8.4 `GridFlow` (rejilla que se ajusta al ancho)

```python
botones = urwid.GridFlow(
    [boton("Nuevo", f), boton("Editar", f), boton("Eliminar", f), boton("Salir", f)],
    cell_width=14, h_sep=2, v_sep=1, align=urwid.CENTER,
)
```

### 8.5 `Overlay` (superponer un widget sobre otro)

```python
dialogo = urwid.LineBox(urwid.Filler(urwid.Text("¿Seguro?")), title="Confirmar")
superposicion = urwid.Overlay(
    top_w=dialogo,
    bottom_w=vista_principal,
    align=urwid.CENTER, width=(urwid.RELATIVE, 50),
    valign=urwid.MIDDLE, height=7,
)
```

### 8.6 Desplazamiento de widgets no-lista (2.6+)

```python
contenido = urwid.Pile([...muchos widgets flow...])
desplazable = urwid.ScrollBar(urwid.Scrollable(contenido))     # box con barra de desplazamiento
```

---

## 9. Listas desplazables: `ListBox` y *walkers*

`ListBox` es el widget **box** principal para mostrar contenido desplazable. Recibe un *walker* (o una lista, que convierte automáticamente).

```python
elementos = urwid.SimpleFocusListWalker([
    urwid.AttrMap(urwid.Button(nombre, on_press=al_elegir, user_data=nombre), None, focus_map="seleccionado")
    for nombre in ["Madrid", "Barcelona", "Sevilla", "Valencia"]
])
lista = urwid.ListBox(elementos)
```

Modificar la lista en ejecución (el `ListBox` se actualiza solo):

```python
elementos.append(nuevo_widget)
elementos.insert(0, otro_widget)
del elementos[2]
elementos[:] = [crear_fila(x) for x in datos_filtrados]     # Sustituir todo
```

Foco y desplazamiento:

```python
widget, posicion = lista.get_focus()        # o lista.focus / lista.focus_position
lista.set_focus(0)
lista.set_focus_valign(urwid.MIDDLE)

def al_cambiar_foco():
    detalle.set_text(f"Elemento {lista.focus_position + 1} de {len(elementos)}")

urwid.connect_signal(elementos, "modified", al_cambiar_foco)
```

| *Walker* | Uso |
|---|---|
| `SimpleFocusListWalker(lista)` | Lista de widgets; avisa de cambios de foco (recomendado). |
| `SimpleListWalker(lista)` | Igual, sin notificar cambios de foco. |
| *Walker* propio (`ListWalker`) | Listas enormes o generadas bajo demanda. |
| `TreeWalker` + `TreeListBox` | Árboles desplegables. |

---

## 10. Paleta de colores y atributos

### 10.1 Formato de la paleta

```python
paleta = [
    # (nombre, primer_plano, fondo, mono, primer_plano_alto, fondo_alto)
    ("cabecera", "white,bold", "dark blue"),
    ("pie", "black", "light gray"),
    ("normal", "light gray", "black"),
    ("seleccionado", "black", "dark cyan"),
    ("error", "light red,bold", ""),
    ("ok", "light green", ""),
    ("boton", "black", "light gray"),
    ("boton_foco", "white,bold", "dark red"),
    ("acento", "", "", "", "#fa0,bold", "g11"),          # Colores de 256 / 24 bits
]
```

- **Colores básicos (16):** `black`, `dark red`, `dark green`, `brown`, `dark blue`, `dark magenta`, `dark cyan`, `light gray`, `dark gray`, `light red`, `light green`, `yellow`, `light blue`, `light magenta`, `light cyan`, `white`, y `default`.
- **Atributos:** `bold`, `underline`, `standout`, `blink`, `italics`, `strikethrough` (combinados con comas: `"white,bold"`).
- **Alta resolución:** `#rgb` (3 dígitos hexadecimales), `g0`…`g100` (grises), `h0`…`h255` (paleta de 256 colores) o `#rrggbb` con color verdadero.

### 10.2 Activar más colores

```python
bucle = urwid.MainLoop(vista, paleta)
bucle.screen.set_terminal_properties(colors=256)        # 16 (defecto), 88, 256 o 2**24
```

### 10.3 Aplicar atributos

```python
urwid.Text(("error", "No se pudo guardar"))
urwid.Text([("ok", "✔ "), "Guardado en ", ("negrita", "datos.json")])
urwid.AttrMap(widget, "normal", focus_map="seleccionado")
urwid.AttrMap(widget, {"normal": "atenuado", None: "atenuado"})       # Reasignar atributos internos
```

---

## 11. Teclado, ratón y señales

### 11.1 Nombres de teclas

| Tecla | Nombre en urwid |
|---|---|
| Letras y números | `"a"`, `"A"`, `"1"` |
| Enter, Esc, Tab | `"enter"`, `"esc"`, `"tab"`, `"shift tab"` |
| Flechas | `"up"`, `"down"`, `"left"`, `"right"` |
| Navegación | `"home"`, `"end"`, `"page up"`, `"page down"` |
| Funciones | `"f1"` … `"f12"` |
| Modificadores | `"ctrl s"`, `"meta x"` (Alt+X), `"shift up"` |
| Borrado | `"backspace"`, `"delete"` |

### 11.2 Teclas no procesadas por ningún widget

```python
def teclas_globales(tecla):
    if tecla in ("q", "Q", "f10"):
        raise urwid.ExitMainLoop()
    if tecla == "f1":
        mostrar_ayuda()
        return True                      # Indica que se ha procesado

urwid.MainLoop(vista, paleta, unhandled_input=teclas_globales)
```

> urwid **no** mueve el foco con Tab por defecto (se navega con las flechas). Si quiere navegación con Tab, captúrela en `unhandled_input` y cambie `focus_position` del contenedor, como hace el ejemplo completo.

> Las teclas que procesa un widget enfocado (por ejemplo, letras en un `Edit`) **no** llegan a `unhandled_input`. Use teclas de función o combinaciones con Ctrl para atajos globales.

### 11.3 Interceptar teclas antes que los widgets

```python
def filtro(teclas, crudas):
    if "ctrl s" in teclas:
        guardar()
        teclas = [t for t in teclas if t != "ctrl s"]
    return teclas

urwid.MainLoop(vista, paleta, input_filter=filtro)
```

### 11.4 Ratón

Con `handle_mouse=True` (por defecto), los clics enfocan y pulsan botones. Los eventos de ratón llegan a `unhandled_input` como tuplas:

```python
def teclas_globales(evento):
    if isinstance(evento, tuple):
        tipo, boton, columna, fila = evento      # ("mouse press", 1, 10, 4)
        ...
```

> Con el ratón activado, para seleccionar texto en la terminal suele ser necesario mantener **Mayús** pulsada.

### 11.5 Señales

```python
urwid.connect_signal(boton, "click", al_pulsar, user_args=[registro_id])
urwid.connect_signal(campo, "postchange", al_cambiar_texto)     # Edit: tras cambiar el texto
urwid.connect_signal(casilla, "postchange", al_marcar)          # CheckBox
urwid.disconnect_signal(boton, "click", al_pulsar, user_args=[registro_id])
```

Firmas: `al_pulsar(registro_id, boton)` con `user_args`; `al_cambiar_texto(edit, texto_anterior)`.

Señales propias:

```python
class Selector(urwid.WidgetWrap):
    signals = ["elegido"]

    def elegir(self, valor):
        urwid.emit_signal(self, "elegido", self, valor)

urwid.connect_signal(selector, "elegido", lambda w, v: estado.set_text(f"Elegido: {v}"))
```

---

## 12. El bucle principal (`MainLoop`)

```python
bucle = urwid.MainLoop(
    widget,                              # Widget box raíz
    palette=paleta,
    unhandled_input=teclas_globales,
    handle_mouse=True,
    pop_ups=False,                       # True si usa PopUpLauncher
    event_loop=None,                     # Bucle propio, asyncio, Twisted…
)
bucle.run()                              # Bloquea hasta ExitMainLoop
```

| Atributo / método | Uso |
|---|---|
| `bucle.widget = nueva_vista` | Cambiar la vista principal (navegación entre pantallas). |
| `bucle.set_alarm_in(segundos, f)` | Ejecutar `f(bucle, datos)` tras un tiempo. |
| `bucle.set_alarm_at(timestamp, f)` | Ejecutar a una hora concreta. |
| `bucle.remove_alarm(manejador)` | Cancelar. |
| `bucle.draw_screen()` | Forzar el redibujado. |
| `bucle.watch_file(fd, f)` | Reaccionar cuando hay datos en un descriptor. |
| `bucle.watch_pipe(f)` | Tubería para comunicarse desde otros hilos o procesos. |
| `bucle.screen` | Acceso a la pantalla (`set_terminal_properties`, `clear`). |
| `raise urwid.ExitMainLoop()` | Salir del bucle. |

Navegación entre «pantallas»:

```python
class Aplicacion:
    def __init__(self):
        self.bucle = urwid.MainLoop(self.vista_menu(), PALETA, unhandled_input=self.teclas)

    def ir_a(self, vista):
        self.bucle.widget = vista

    def vista_menu(self): ...
    def vista_clientes(self): ...
```

---

## 13. Widgets personalizados

### 13.1 Componer con `WidgetWrap` (recomendado)

```python
class FilaContacto(urwid.WidgetWrap):
    def __init__(self, contacto):
        self.contacto = contacto
        columnas = urwid.Columns([
            ("weight", 2, urwid.Text(contacto["nombre"])),
            ("weight", 2, urwid.Text(contacto["email"])),
            ("fixed", 12, urwid.Text(contacto["telefono"], align=urwid.RIGHT)),
        ], dividechars=1)
        super().__init__(urwid.AttrMap(columnas, "normal", focus_map="seleccionado"))

    def selectable(self) -> bool:
        return True                              # Puede recibir el foco

    def keypress(self, size, tecla):
        if tecla == "enter":
            urwid.emit_signal(self, "abrir", self.contacto)
            return None                          # Tecla procesada
        return tecla                             # Tecla no procesada: sigue subiendo

    signals = ["abrir"]
```

> `keypress` debe devolver `None` si procesa la tecla, o la propia tecla para que la gestione el contenedor o `unhandled_input`.

### 13.2 Widget dibujado a mano

```python
class Barra(urwid.Widget):
    _sizing = frozenset([urwid.Sizing.FLOW])

    def __init__(self, valor: float):
        super().__init__()
        self.valor = valor

    def rows(self, size, focus=False):
        return 1

    def render(self, size, focus=False):
        (ancho,) = size
        lleno = int(ancho * self.valor)
        return urwid.TextCanvas([("█" * lleno + "░" * (ancho - lleno)).encode()], maxcol=ancho)

    def actualizar(self, valor):
        self.valor = valor
        self._invalidate()                       # Pide redibujar
```

---

## 14. Diálogos y ventanas emergentes

### 14.1 Diálogo modal con `Overlay`

```python
class Dialogo:
    """Muestra un diálogo sobre la vista actual y la restaura al cerrarse."""

    def __init__(self, bucle: urwid.MainLoop):
        self.bucle = bucle

    def confirmar(self, mensaje: str, al_aceptar) -> None:
        anterior = self.bucle.widget

        def cerrar(_boton=None):
            self.bucle.widget = anterior

        def aceptar(_boton):
            cerrar()
            al_aceptar()

        botones = urwid.GridFlow([
            urwid.AttrMap(urwid.Button("Cancelar", cerrar), "boton", "boton_foco"),
            urwid.AttrMap(urwid.Button("Aceptar", aceptar), "boton", "boton_foco"),
        ], cell_width=12, h_sep=2, v_sep=0, align=urwid.CENTER)
        cuerpo = urwid.Pile([urwid.Text(mensaje, align=urwid.CENTER), urwid.Divider(), botones])
        caja = urwid.AttrMap(urwid.LineBox(urwid.Filler(cuerpo), title="Confirmar"), "dialogo")
        self.bucle.widget = urwid.Overlay(caja, anterior, align=urwid.CENTER, width=50,
                                          valign=urwid.MIDDLE, height=8)
```

### 14.2 `PopUpLauncher` (menús emergentes ligados a un widget)

```python
class BotonConMenu(urwid.PopUpLauncher):
    def __init__(self):
        super().__init__(urwid.Button("Opciones ▾"))
        urwid.connect_signal(self.original_widget, "click", lambda _: self.open_pop_up())

    def create_pop_up(self):
        opciones = urwid.Pile([urwid.Button(o, on_press=lambda _: self.close_pop_up())
                               for o in ("Exportar", "Imprimir", "Duplicar")])
        return urwid.AttrMap(urwid.LineBox(opciones), "dialogo")

    def get_pop_up_parameters(self):
        return {"left": 0, "top": 1, "overlay_width": 18, "overlay_height": 5}

# Requiere: urwid.MainLoop(..., pop_ups=True)
```

---

## 15. Temporizadores, procesos externos y asyncio

### 15.1 Temporizador periódico (reloj)

```python
import time

reloj = urwid.Text("", align=urwid.RIGHT)

def actualizar_reloj(bucle, _datos=None):
    reloj.set_text(time.strftime("%H:%M:%S"))
    bucle.set_alarm_in(1, actualizar_reloj)

bucle = urwid.MainLoop(vista, paleta)
bucle.set_alarm_in(0, actualizar_reloj)
bucle.run()
```

### 15.2 Actualizar desde otro hilo con `watch_pipe`

```python
import threading

registro = urwid.SimpleFocusListWalker([])

def recibir(datos: bytes) -> bool:
    for linea in datos.decode().splitlines():
        registro.append(urwid.Text(linea))
    return True                                   # False cierra la tubería

fd = bucle.watch_pipe(recibir)

def trabajo():
    import os
    for i in range(10):
        time.sleep(0.5)
        os.write(fd, f"Paso {i + 1} completado\n".encode())

threading.Thread(target=trabajo, daemon=True).start()
```

> Los widgets no son seguros entre hilos: comunique los cambios mediante `watch_pipe` en lugar de modificar widgets desde otro hilo.

### 15.3 Integración con asyncio

```python
import asyncio

async def consultar_periodicamente():
    while True:
        datos = await obtener_estado()
        estado.set_text(datos)
        await asyncio.sleep(5)

bucle_asyncio = asyncio.new_event_loop()
asyncio.set_event_loop(bucle_asyncio)
bucle_asyncio.create_task(consultar_periodicamente())
urwid.MainLoop(vista, paleta, event_loop=urwid.AsyncioEventLoop(loop=bucle_asyncio)).run()
```

---

## 16. Pruebas sin terminal

Los widgets se pueden **renderizar** y recibir **teclas** sin abrir la pantalla, lo que permite pruebas unitarias:

```python
def texto_renderizado(widget, ancho=40, alto=None) -> str:
    tamano = (ancho,) if alto is None else (ancho, alto)
    canvas = widget.render(tamano, focus=True)
    return "\n".join(linea.decode().rstrip() for linea in canvas.text)

def test_formulario():
    campo = urwid.Edit("Nombre: ")
    for tecla in "Ana":
        campo.keypress((40,), tecla)
    assert campo.edit_text == "Ana"
    assert "Nombre: Ana" in texto_renderizado(campo)

def test_lista_navegacion():
    lista = urwid.ListBox(urwid.SimpleFocusListWalker([urwid.SelectableIcon(f"Item {i}") for i in range(5)]))
    lista.keypress((20, 3), "down")
    assert lista.focus_position == 1
```

---

## 17. Ejemplo práctico completo

**Agenda de contactos TUI**: cabecera con reloj, lista filtrable de contactos en el panel izquierdo, ficha de detalle en el derecho que se actualiza al mover el foco, formulario de alta y edición con validación, confirmación de borrado en un diálogo superpuesto, barra de estado, atajos de teclado, soporte de ratón y almacenamiento en JSON.

```python
"""
agenda.py — Agenda de contactos en consola con urwid.
Requiere: pip install urwid
Uso: python agenda.py [agenda.json]
"""
import json
import re
import sys
import time
from dataclasses import asdict, dataclass
from pathlib import Path

import urwid

PALETA = [
    ("cabecera", "white,bold", "dark blue"),
    ("pie", "black", "light gray"),
    ("pie_tecla", "dark red,bold", "light gray"),
    ("normal", "light gray", "default"),
    ("seleccionado", "black", "dark cyan"),
    ("titulo", "light cyan,bold", "default"),
    ("etiqueta", "dark gray", "default"),
    ("valor", "white", "default"),
    ("busqueda", "white", "dark gray"),
    ("error", "light red,bold", "default"),
    ("ok", "light green", "default"),
    ("boton", "black", "light gray"),
    ("boton_foco", "white,bold", "dark red"),
    ("dialogo", "black", "light gray"),
    ("campo", "black", "white"),
    ("campo_foco", "black", "yellow"),
]
PATRON_EMAIL = re.compile(r"^[\w.+-]+@[\w-]+\.[\w.-]+$")


# ------------------------------------------------------------------ Datos
@dataclass
class Contacto:
    nombre: str
    email: str
    telefono: str = ""
    empresa: str = ""
    notas: str = ""


class Agenda:
    def __init__(self, ruta: Path):
        self.ruta = ruta
        if ruta.exists():
            self.contactos = [Contacto(**c) for c in json.loads(ruta.read_text(encoding="utf-8"))]
        else:
            self.contactos = [
                Contacto("Ana Pérez", "ana.perez@ejemplo.com", "600111222", "Acme", "Responsable de compras"),
                Contacto("Luis Gómez", "luis@ejemplo.com", "600333444", "Globex"),
                Contacto("Marta Ruiz", "marta.ruiz@ejemplo.com", "600555666", "Initech", "Prefiere email"),
                Contacto("Carlos Díaz", "cdiaz@ejemplo.com", "", "Acme"),
            ]
            self.guardar()

    def guardar(self) -> None:
        self.contactos.sort(key=lambda c: c.nombre.lower())
        self.ruta.write_text(json.dumps([asdict(c) for c in self.contactos], ensure_ascii=False, indent=2),
                             encoding="utf-8")

    def buscar(self, texto: str) -> list[Contacto]:
        t = texto.lower().strip()
        return [c for c in self.contactos
                if not t or t in c.nombre.lower() or t in c.email.lower() or t in c.empresa.lower()]


# ------------------------------------------------------------------ Widgets
class FilaContacto(urwid.WidgetWrap):
    signals = ["abrir"]

    def __init__(self, contacto: Contacto):
        self.contacto = contacto
        texto = urwid.Text([("valor", contacto.nombre), ("etiqueta", f"  {contacto.empresa}")],
                           wrap=urwid.WrapMode.ELLIPSIS)
        super().__init__(urwid.AttrMap(texto, "normal", focus_map="seleccionado"))

    def selectable(self) -> bool:
        return True

    def keypress(self, size, tecla):
        if tecla == "enter":
            urwid.emit_signal(self, "abrir", self.contacto)
            return None
        return tecla

    def mouse_event(self, size, evento, boton, col, fila, foco):
        if evento == "mouse press" and boton == 1 and foco:
            urwid.emit_signal(self, "abrir", self.contacto)
            return True
        return False


def boton(etiqueta: str, funcion, dato=None) -> urwid.AttrMap:
    return urwid.AttrMap(urwid.Button(etiqueta, on_press=funcion, user_data=dato), "boton", "boton_foco")


def campo(etiqueta: str, valor: str = "") -> tuple[urwid.Columns, urwid.Edit]:
    edit = urwid.Edit("", valor)
    fila = urwid.Columns([("fixed", 11, urwid.Text(("etiqueta", etiqueta))),
                          urwid.AttrMap(edit, "campo", "campo_foco")])
    return fila, edit


# ------------------------------------------------------------------ Aplicación
class AplicacionAgenda:
    def __init__(self, ruta: Path):
        self.agenda = Agenda(ruta)
        self.reloj = urwid.Text("", align=urwid.RIGHT)
        self.estado = urwid.Text("")
        self.busqueda = urwid.Edit(("busqueda", " Buscar: "))
        urwid.connect_signal(self.busqueda, "postchange", lambda *_: self.recargar_lista())

        self.filas = urwid.SimpleFocusListWalker([])
        urwid.connect_signal(self.filas, "modified", self.mostrar_detalle)
        self.lista = urwid.ListBox(self.filas)
        self.detalle = urwid.WidgetPlaceholder(urwid.Filler(urwid.Text("")))

        self.principal = self.construir_vista()
        self.bucle = urwid.MainLoop(self.principal, PALETA, unhandled_input=self.teclas_globales)
        self.recargar_lista()

    # ---------- Construcción
    def construir_vista(self) -> urwid.Frame:
        cabecera = urwid.AttrMap(urwid.Columns([urwid.Text(" 📇 Agenda de contactos"), self.reloj]),
                                 "cabecera")
        panel_izq = urwid.LineBox(urwid.Pile([("pack", self.busqueda), ("pack", urwid.Divider("─")),
                                              self.lista]), title="Contactos")
        panel_der = urwid.LineBox(self.detalle, title="Detalle")
        cuerpo = urwid.Columns([("weight", 2, panel_izq), ("weight", 3, panel_der)], dividechars=1)
        teclas = [("pie_tecla", " F2"), " Nuevo ", ("pie_tecla", "F3"), " Editar ",
                  ("pie_tecla", "F8"), " Eliminar ", ("pie_tecla", "F4"), " Buscar ",
                  ("pie_tecla", "F10"), " Salir  "]
        pie = urwid.AttrMap(urwid.Columns([urwid.Text(teclas), self.estado]), "pie")
        return urwid.Frame(cuerpo, header=cabecera, footer=pie)

    # ---------- Lista y detalle
    def recargar_lista(self, seleccionar: Contacto | None = None) -> None:
        resultados = self.agenda.buscar(self.busqueda.edit_text)
        nuevas = []
        for c in resultados:
            fila = FilaContacto(c)
            urwid.connect_signal(fila, "abrir", self.editar)
            nuevas.append(fila)
        self.filas[:] = nuevas                  # Ojo: tras sustituir el contenido, el foco queda al final
        if seleccionar in resultados:
            self.filas.set_focus(resultados.index(seleccionar))
        elif nuevas:
            self.filas.set_focus(0)
        self.mostrar_detalle()
        self.avisar(f"{len(resultados)} de {len(self.agenda.contactos)} contactos")

    def contacto_actual(self) -> Contacto | None:
        if not self.filas:
            return None
        return self.filas[self.lista.focus_position].contacto

    def mostrar_detalle(self) -> None:
        c = self.contacto_actual()
        if c is None:
            self.detalle.original_widget = urwid.Filler(urwid.Text(("etiqueta", "Sin contactos"), align=urwid.CENTER))
            return
        lineas = [urwid.Text(("titulo", c.nombre)), urwid.Divider()]
        for etiqueta, valor in [("Email", c.email), ("Teléfono", c.telefono or "—"),
                                ("Empresa", c.empresa or "—"), ("Notas", c.notas or "—")]:
            lineas.append(urwid.Columns([("fixed", 10, urwid.Text(("etiqueta", etiqueta))),
                                         urwid.Text(("valor", valor))]))
        self.detalle.original_widget = urwid.Filler(urwid.Padding(urwid.Pile(lineas), left=1, right=1),
                                                    valign=urwid.TOP)

    def avisar(self, mensaje: str, tipo: str = "pie") -> None:
        self.estado.set_text((tipo if tipo != "pie" else "pie", mensaje))
        self.estado.align = urwid.RIGHT

    # ---------- Formulario
    def formulario(self, contacto: Contacto | None) -> None:
        nuevo = contacto is None
        c = contacto or Contacto("", "")
        filas_campos, campos = [], {}
        for clave, etiqueta in [("nombre", "Nombre *"), ("email", "Email *"), ("telefono", "Teléfono"),
                                ("empresa", "Empresa"), ("notas", "Notas")]:
            fila, edit = campo(etiqueta, getattr(c, clave))
            filas_campos.append(fila)
            campos[clave] = edit
        error = urwid.Text("")

        def guardar(_boton=None):
            datos = {k: e.edit_text.strip() for k, e in campos.items()}
            if len(datos["nombre"]) < 2:
                error.set_text(("error", "El nombre es obligatorio (mínimo 2 caracteres)."))
                return
            if not PATRON_EMAIL.match(datos["email"]):
                error.set_text(("error", "El email no es válido."))
                return
            if datos["telefono"] and not datos["telefono"].replace(" ", "").isdigit():
                error.set_text(("error", "El teléfono solo puede contener dígitos."))
                return
            duplicado = next((x for x in self.agenda.contactos
                              if x.email.lower() == datos["email"].lower() and x is not contacto), None)
            if duplicado:
                error.set_text(("error", f"El email ya pertenece a {duplicado.nombre}."))
                return
            if nuevo:
                destino = Contacto(**datos)
                self.agenda.contactos.append(destino)
            else:
                destino = contacto
                for k, v in datos.items():
                    setattr(destino, k, v)
            self.agenda.guardar()
            self.cerrar_dialogo()
            self.recargar_lista(seleccionar=destino)
            self.avisar(f"✔ {destino.nombre} guardado", "ok")

        botones = urwid.GridFlow([boton("Cancelar", lambda _: self.cerrar_dialogo()), boton("Guardar", guardar)],
                                 cell_width=12, h_sep=2, v_sep=0, align=urwid.RIGHT)
        cuerpo = urwid.Pile([*filas_campos, urwid.Divider(), error, urwid.Divider(), botones])
        self.pila_dialogo = cuerpo                  # Para la navegación con Tab
        self.abrir_dialogo(cuerpo, "Nuevo contacto" if nuevo else "Editar contacto", ancho=60, alto=13)

    def editar(self, contacto: Contacto | None = None) -> None:
        contacto = contacto or self.contacto_actual()
        if contacto:
            self.formulario(contacto)

    def eliminar(self) -> None:
        c = self.contacto_actual()
        if c is None:
            return

        def confirmar(_boton):
            self.agenda.contactos.remove(c)
            self.agenda.guardar()
            self.cerrar_dialogo()
            self.recargar_lista()
            self.avisar(f"{c.nombre} eliminado", "error")

        cuerpo = urwid.Pile([
            urwid.Text(f"¿Eliminar a «{c.nombre}»?", align=urwid.CENTER),
            urwid.Divider(),
            urwid.GridFlow([boton("Cancelar", lambda _: self.cerrar_dialogo()), boton("Eliminar", confirmar)],
                           cell_width=12, h_sep=2, v_sep=0, align=urwid.CENTER),
        ])
        self.pila_dialogo = cuerpo
        self.abrir_dialogo(cuerpo, "Confirmar", ancho=46, alto=7)

    # ---------- Diálogos
    def abrir_dialogo(self, contenido: urwid.Widget, titulo: str, ancho: int, alto: int) -> None:
        caja = urwid.AttrMap(urwid.LineBox(urwid.Filler(contenido, valign=urwid.TOP), title=titulo), "dialogo")
        self.bucle.widget = urwid.Overlay(caja, self.principal, align=urwid.CENTER, width=ancho,
                                          valign=urwid.MIDDLE, height=alto)

    def cerrar_dialogo(self) -> None:
        self.bucle.widget = self.principal

    def dialogo_abierto(self) -> bool:
        return self.bucle.widget is not self.principal

    # ---------- Teclado y temporizador
    def teclas_globales(self, tecla) -> bool | None:
        if isinstance(tecla, tuple):                         # Eventos de ratón no procesados
            return None
        if self.dialogo_abierto():
            if tecla == "esc":
                self.cerrar_dialogo()
            elif tecla in ("tab", "shift tab"):
                self.mover_foco_dialogo(1 if tecla == "tab" else -1)
            return True
        acciones = {
            "f2": lambda: self.formulario(None),
            "f3": self.editar,
            "f8": self.eliminar,
            "delete": self.eliminar,
            "f4": self.enfocar_busqueda,
            "/": self.enfocar_busqueda,
            "esc": self.limpiar_busqueda,
        }
        if tecla in ("f10", "q", "Q"):
            raise urwid.ExitMainLoop()
        if tecla in acciones:
            acciones[tecla]()
            return True
        return None

    def mover_foco_dialogo(self, paso: int) -> None:
        """urwid no gestiona Tab por defecto: se mueve el foco al siguiente elemento seleccionable."""
        pila = getattr(self, "pila_dialogo", None)
        if pila is None:
            return
        seleccionables = [i for i, (w, _) in enumerate(pila.contents) if w.selectable()]
        if seleccionables:
            actual = pila.focus_position
            siguientes = [i for i in seleccionables if (i - actual) * paso > 0]
            pila.focus_position = (min(siguientes, key=lambda i: abs(i - actual)) if siguientes
                                   else (seleccionables[0] if paso > 0 else seleccionables[-1]))

    def enfocar_busqueda(self) -> None:
        self.principal.focus_position = "body"
        self.principal.body.focus_position = 0
        self.principal.body.contents[0][0].original_widget.focus_position = 0

    def limpiar_busqueda(self) -> None:
        self.busqueda.set_edit_text("")
        self.principal.body.contents[0][0].original_widget.focus_position = 2

    def tic(self, bucle, _datos=None) -> None:
        self.reloj.set_text(time.strftime("%d/%m/%Y %H:%M:%S "))
        bucle.set_alarm_in(1, self.tic)

    def ejecutar(self) -> None:
        self.bucle.set_alarm_in(0, self.tic)
        self.bucle.run()


if __name__ == "__main__":
    ruta = Path(sys.argv[1]) if len(sys.argv) > 1 else Path("agenda.json")
    AplicacionAgenda(ruta).ejecutar()
```

Uso:

```bash
python agenda.py
python agenda.py ~/trabajo/clientes.json
```

| Tecla | Acción |
|---|---|
| ↑ / ↓ | Moverse por la lista (el detalle se actualiza) |
| Enter / clic | Editar el contacto |
| F2 | Nuevo contacto |
| F3 | Editar |
| F8 / Supr | Eliminar (con confirmación) |
| F4 o `/` | Buscar · Esc limpia la búsqueda |
| Tab / Mayús+Tab o ↑ / ↓ | Cambiar de campo en el formulario |
| Esc | Cerrar el diálogo |
| F10 / Q | Salir |

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ValueError: … not a box widget` / `WidgetError: … sizing` | Se pasó un widget *flow* donde se espera *box* (p. ej. `MainLoop(Text(...))`). | Envuélvalo en `Filler` o use `("pack", w)` / `("weight", n, w)` en `Pile`. |
| `ListBox` dentro de `Pile` no se ve o falla | `ListBox` es *box* y se añadió sin peso. | `("weight", 1, listbox)` o `BoxAdapter(listbox, alto)`. |
| Los atajos globales no funcionan | El widget enfocado (un `Edit`) consume la tecla. | Use teclas de función o `input_filter`. |
| Los colores no cambian | El nombre del atributo no está en la paleta o tiene una errata. | Revise la paleta; los nombres distinguen mayúsculas. |
| Colores de 256 no se muestran | La pantalla está en modo 16 colores. | `bucle.screen.set_terminal_properties(colors=256)`. |
| El texto se corta o se reparte en varias líneas | Modo de ajuste por defecto. | `wrap=urwid.WrapMode.ELLIPSIS` o `CLIP`. |
| Tras recargar una lista, el foco aparece en el último elemento | Al sustituir el contenido del *walker* (`walker[:] = …`) el foco queda al final. | Llame a `walker.set_focus(0)` después. |
| Tab no cambia de campo | urwid navega con flechas; Tab no tiene acción por defecto. | Gestione `"tab"` en `unhandled_input` moviendo `focus_position`. |
| La lista no se actualiza al cambiar los datos | Se reasignó la variable en lugar de modificar el *walker*. | Modifique el *walker* en su sitio: `walker[:] = nuevos`. |
| Pantalla corrupta al modificar widgets desde un hilo | urwid no es seguro entre hilos. | Use `watch_pipe` o alarmas. |
| `DeprecationWarning` con nombres antiguos | API heredada (`urwid.raw_display`, cadenas `'center'`…). | Use `urwid.display.raw`, `urwid.CENTER`, etc. |
| No se puede seleccionar texto con el ratón | El ratón está capturado por la aplicación. | Mantenga Mayús al seleccionar o `handle_mouse=False`. |

---

## 19. Buenas prácticas

- Diseñe pensando en los **tipos de tamaño** (box/flow) antes de escribir código.
- Organice la aplicación en una clase con métodos para construir cada vista, como en el ejemplo.
- Defina la paleta en un único lugar con nombres semánticos (`error`, `seleccionado`, `cabecera`).
- Use `AttrMap(widget, normal, focus_map=…)` para que el foco sea siempre visible.
- Use `SimpleFocusListWalker` y modifíquelo en su sitio para actualizar listas.
- Reserve teclas de función o combinaciones para los atajos globales y muéstrelas en el pie.
- No modifique widgets desde otros hilos: use `watch_pipe`, alarmas o el bucle `asyncio`.
- Pruebe widgets aislados con `render()` y `keypress()` sin necesidad de terminal.
- Para aplicaciones nuevas con muchos widgets o estilos complejos, valore Textual.

---

## 20. Referencia rápida

```python
urwid.MainLoop(widget_box, palette=[(nombre, fg, bg)], unhandled_input=f, input_filter=f, pop_ups=False).run()
raise urwid.ExitMainLoop() · bucle.widget = vista · bucle.set_alarm_in(s, f) · bucle.watch_pipe(f)
Text(markup, align=urwid.CENTER, wrap=urwid.WrapMode.ELLIPSIS) · .set_text()
Edit(caption, edit_text, multiline=, mask="*") · .edit_text · IntEdit · Button(label, on_press, user_data)
CheckBox(label, state) · RadioButton(grupo, label) · Divider("─") · SolidFill · ProgressBar · BigText
Padding(w, align=, width=(urwid.RELATIVE, 60), left=, right=) · Filler(w, valign=) · AttrMap(w, attr, focus_map)
LineBox(w, title=) · WidgetPlaceholder(w) · BoxAdapter(w, h) · ScrollBar(Scrollable(w))
Pile([w, ("pack", w), ("weight", 1, w)]) · Columns([("fixed", n, w), ("weight", n, w)], dividechars=)
Frame(body, header=, footer=) · GridFlow([...], cell_width=, h_sep=, v_sep=, align=) · Overlay(top, bottom, ...)
ListBox(SimpleFocusListWalker([...])) · walker[:] = nuevos · listbox.focus_position
connect_signal(w, "click"|"postchange"|"modified", f, user_args=[...]) · emit_signal · signals = [...]
class X(urwid.WidgetWrap): selectable() · keypress(size, key) → None|key · mouse_event(...)
bucle.screen.set_terminal_properties(colors=256) · event_loop=urwid.AsyncioEventLoop(loop=...)
```

---

## 21. Ejercicios propuestos

1. Cree un menú principal con `ListBox` de botones que navegue a tres «pantallas» distintas cambiando `bucle.widget`.
2. Implemente un monitor de un archivo de log que muestre las nuevas líneas en un `ListBox` usando `watch_file` o un hilo con `watch_pipe`.
3. Añada a la agenda una exportación a CSV con una barra de progreso (`ProgressBar`) dentro de un diálogo.
4. Desarrolle un selector de archivos con `TreeListBox` que muestre las carpetas de forma desplegable.
5. Escriba pruebas unitarias para `FilaContacto` y para el formulario de la agenda usando `render()` y `keypress()`.

---

## 22. Referencias

- Documentación oficial: https://urwid.org/
- Tutorial: https://urwid.org/tutorial/
- Manual de referencia de widgets: https://urwid.org/reference/
- Repositorio y ejemplos: https://github.com/urwid/urwid
