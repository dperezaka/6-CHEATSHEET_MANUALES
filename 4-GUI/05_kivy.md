# Manual de uso: Kivy

> **Categoría:** Interfaces gráficas (GUI) · **Utilidad:** Aplicaciones multiplataforma y táctiles
> **Tipo:** Librería externa · Escritorio (Windows, macOS, Linux) y móvil (Android, iOS)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera aplicación](#4-primera-aplicación)
5. [El lenguaje KV](#5-el-lenguaje-kv)
6. [Widgets](#6-widgets)
7. [Layouts](#7-layouts)
8. [Propiedades y eventos](#8-propiedades-y-eventos)
9. [Varias pantallas: ScreenManager](#9-varias-pantallas-screenmanager)
10. [Listas eficientes: RecycleView](#10-listas-eficientes-recycleview)
11. [Ventanas emergentes y notificaciones](#11-ventanas-emergentes-y-notificaciones)
12. [Gráficos, animaciones y gestos](#12-gráficos-animaciones-y-gestos)
13. [Tiempo, tareas y concurrencia](#13-tiempo-tareas-y-concurrencia)
14. [Almacenamiento y configuración](#14-almacenamiento-y-configuración)
15. [Diseño adaptable (tamaños y densidad)](#15-diseño-adaptable-tamaños-y-densidad)
16. [KivyMD: Material Design](#16-kivymd-material-design)
17. [Empaquetar para escritorio y móvil](#17-empaquetar-para-escritorio-y-móvil)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**Kivy** es un framework de código abierto (licencia MIT) para crear aplicaciones con interfaces **táctiles y multiplataforma**. Un mismo código puede ejecutarse en Windows, macOS, Linux, **Android** e **iOS**.

**Características:**
- Diseñado para pantallas táctiles: gestos, multitáctil, deslizamientos.
- Renderizado acelerado por GPU (OpenGL ES 2), con animaciones fluidas.
- Lenguaje declarativo **KV** para separar el diseño de la lógica.
- Propiedades reactivas: la interfaz se actualiza sola al cambiar los datos.
- Licencia MIT: uso comercial libre.

**Cuándo usar Kivy:**

| Situación | ¿Kivy? |
|---|---|
| Aplicación para Android/iOS escrita en Python | ✔ Muy adecuado |
| Quioscos, pantallas táctiles, TPV, Raspberry Pi | ✔ Muy adecuado |
| Juegos sencillos e interfaces muy personalizadas | ✔ Adecuado |
| Aplicación de escritorio de gestión con aspecto nativo | ✘ Mejor PySide6/PyQt6 |
| Herramienta interna sencilla | ✘ Mejor Tkinter/CustomTkinter |

> Kivy **no** usa los controles nativos del sistema: dibuja su propia interfaz, con un aspecto uniforme en todas las plataformas.

---

## 2. Instalación e importación

```bash
pip install "kivy[base]"            # Kivy con dependencias básicas
pip install "kivy[base,media]"      # + audio y vídeo
```

> Use un entorno virtual. Si no existen paquetes precompilados para su versión de Python, Kivy intentará compilarse (lo que requiere herramientas adicionales). Compruebe las versiones de Python compatibles en la documentación oficial.

Comprobar la instalación:

```bash
python -c "import kivy; print(kivy.__version__)"
```

Importaciones habituales:

```python
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.gridlayout import GridLayout
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.uix.textinput import TextInput
from kivy.uix.screenmanager import ScreenManager, Screen
from kivy.properties import StringProperty, NumericProperty, ListProperty, ObjectProperty, BooleanProperty
from kivy.lang import Builder
from kivy.clock import Clock
from kivy.core.window import Window
from kivy.metrics import dp, sp
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `App` | Clase de la aplicación. Su método `build()` devuelve el widget raíz. |
| Widget | Elemento visual (`Label`, `Button`…). Todo en Kivy es un widget. |
| Layout | Widget que organiza a sus hijos (`BoxLayout`, `GridLayout`…). |
| Lenguaje KV | Lenguaje declarativo para definir la interfaz (archivo `.kv` o cadena). |
| Propiedades Kivy | Atributos reactivos (`StringProperty`…) que notifican sus cambios. |
| Eventos | `on_press`, `on_text`, `on_touch_down`… |
| `Clock` | Planificador de tareas (temporizadores, ejecuciones diferidas). |
| `ScreenManager` | Gestiona varias pantallas con transiciones. |
| `dp` / `sp` | Unidades independientes de la densidad de pantalla. |
| Canvas | Instrucciones de dibujo de cada widget. |

**Sistema de coordenadas:** el origen `(0, 0)` está en la **esquina inferior izquierda** de la ventana.

**Tamaño de los widgets:** por defecto, los layouts asignan el tamaño mediante `size_hint` (proporción del espacio disponible). Para un tamaño fijo, hay que desactivarlo: `size_hint_y=None, height=dp(48)`.

---

## 4. Primera aplicación

### 4.1 Solo con Python

```python
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button


class Saludo(BoxLayout):
    def __init__(self, **kwargs):
        super().__init__(orientation="vertical", padding=20, spacing=10, **kwargs)
        self.entrada = TextInput(hint_text="Escriba su nombre", multiline=False, size_hint_y=None, height=44)
        self.resultado = Label(text="")
        boton = Button(text="Saludar", size_hint_y=None, height=48)
        boton.bind(on_press=self.saludar)
        for w in (self.entrada, boton, self.resultado):
            self.add_widget(w)

    def saludar(self, *_):
        self.resultado.text = f"Hola, {self.entrada.text.strip() or 'desconocido'}"


class SaludoApp(App):
    def build(self):
        self.title = "Saludo"
        return Saludo()


if __name__ == "__main__":
    SaludoApp().run()
```

### 4.2 Con lenguaje KV (recomendado)

**`main.py`**

```python
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout


class Saludo(BoxLayout):
    def saludar(self):
        nombre = self.ids.entrada.text.strip() or "desconocido"
        self.ids.resultado.text = f"Hola, {nombre}"


class SaludoApp(App):
    pass


if __name__ == "__main__":
    SaludoApp().run()
```

**`saludo.kv`** (Kivy lo carga automáticamente: nombre de la clase `App` sin «App», en minúsculas)

```yaml
Saludo:

<Saludo>:
    orientation: "vertical"
    padding: dp(20)
    spacing: dp(10)

    TextInput:
        id: entrada
        hint_text: "Escriba su nombre"
        multiline: False
        size_hint_y: None
        height: dp(44)
        on_text_validate: root.saludar()

    Button:
        text: "Saludar"
        size_hint_y: None
        height: dp(48)
        on_press: root.saludar()

    Label:
        id: resultado
        font_size: sp(20)
```

> La regla `Saludo:` (sin `< >`) define el widget raíz; `<Saludo>:` define cómo es **cualquier** instancia de la clase `Saludo`.

---

## 5. El lenguaje KV

### 5.1 Sintaxis básica

```yaml
#:import dp kivy.metrics.dp
#:set COLOR_PRIMARIO (0.15, 0.39, 0.92, 1)

<MiPantalla>:                       # Regla de clase
    orientation: "vertical"         # Propiedad
    Label:                          # Widget hijo
        text: "Título"
        color: COLOR_PRIMARIO
    Button:
        text: "Aceptar"
        on_release: root.aceptar()  # Evento → método de la clase raíz de la regla
```

| Palabra clave | Significado |
|---|---|
| `root` | Widget raíz de la regla actual. |
| `self` | Widget actual. |
| `app` | Instancia de la aplicación (`App.get_running_app()`). |
| `id` | Identificador; accesible en Python con `self.ids.nombre`. |
| `#:import` | Importar un módulo o nombre. |
| `#:set` | Definir una constante. |
| `#:include` | Incluir otro archivo `.kv`. |

### 5.2 Enlaces reactivos

Las expresiones en KV se **reevalúan automáticamente** cuando cambian las propiedades que usan:

```yaml
<Contador>:
    Label:
        text: f"Pulsaciones: {root.contador}"          # Se actualiza al cambiar root.contador
    Button:
        text: "Desactivado" if root.contador >= 10 else "Sumar"
        disabled: root.contador >= 10
        on_press: root.contador += 1
    Slider:
        id: volumen
        min: 0
        max: 100
    Label:
        text: f"Volumen: {int(volumen.value)} %"
```

```python
from kivy.properties import NumericProperty

class Contador(BoxLayout):
    contador = NumericProperty(0)
```

### 5.3 Cargar KV desde Python

```python
from kivy.lang import Builder

Builder.load_file("pantallas/ajustes.kv")

KV = """
BoxLayout:
    Label:
        text: "Definido en una cadena"
"""
raiz = Builder.load_string(KV)
```

> Cargue cada archivo `.kv` **una sola vez**; cargarlo dos veces duplica las reglas.

### 5.4 Estilos reutilizables

```yaml
<BotonPrimario@Button>:              # Clase dinámica: hereda de Button
    background_normal: ""
    background_color: 0.15, 0.39, 0.92, 1
    color: 1, 1, 1, 1
    size_hint_y: None
    height: dp(48)
    font_size: sp(16)

<Formulario>:
    BotonPrimario:
        text: "Guardar"
```

---

## 6. Widgets

### 6.1 Texto

```yaml
Label:
    text: "[b]Negrita[/b] y [color=#E74C3C]rojo[/color]"
    markup: True
    font_size: sp(18)
    halign: "left"
    valign: "middle"
    text_size: self.size                     # Necesario para que halign/valign funcionen y el texto se ajuste
    shorten: True                            # Recortar con «…» si no cabe
```

### 6.2 Botones

```yaml
Button:
    text: "Enviar"
    on_press: print("pulsado")
    on_release: root.enviar()               # Preferible: se ejecuta al soltar

ToggleButton:
    text: "Wifi"
    group: "conexion"                       # Botones excluyentes
    state: "down"                           # "down" = activado
```

### 6.3 Entrada de datos

```yaml
TextInput:
    hint_text: "Importe"
    input_filter: "float"                   # "int", "float" o una función
    multiline: False
    write_tab: False                        # Tab pasa al siguiente campo
    password: False
    on_text_validate: root.guardar()        # Al pulsar Intro

CheckBox:
    active: True
    on_active: root.al_marcar(self.active)

Switch:
    active: False

Slider:
    min: 0
    max: 100
    step: 5
    value: 50

Spinner:                                    # Lista desplegable
    text: "Madrid"
    values: ["Madrid", "Sevilla", "Valencia"]
    on_text: root.ciudad = self.text

ProgressBar:
    max: 100
    value: 40
```

### 6.4 Imágenes

```yaml
Image:
    source: "imagenes/logo.png"
    fit_mode: "contain"                     # Kivy 2.2+: "scale-down", "fill", "contain", "cover"

AsyncImage:
    source: "https://ejemplo.com/foto.jpg"  # Carga en segundo plano
```

### 6.5 Otros widgets

| Widget | Uso |
|---|---|
| `ScrollView` | Contenedor desplazable. |
| `TabbedPanel` | Pestañas. |
| `Popup` / `ModalView` | Ventanas emergentes. |
| `DropDown` | Menú desplegable. |
| `FileChooserListView` / `FileChooserIconView` | Selector de archivos. |
| `Carousel` | Páginas deslizables. |
| `Accordion` | Secciones plegables. |
| `VideoPlayer` | Reproductor de vídeo. |
| `Camera` | Cámara. |
| `RstDocument` | Texto con formato reStructuredText. |

---

## 7. Layouts

### 7.1 `BoxLayout` (fila o columna)

```yaml
BoxLayout:
    orientation: "vertical"                 # o "horizontal"
    padding: dp(16)
    spacing: dp(8)

    Label:
        size_hint_y: 0.2                    # 20 % del alto
    Label:
        size_hint_y: 0.8
    Button:
        size_hint_y: None                   # Alto fijo
        height: dp(48)
```

### 7.2 `GridLayout` (rejilla)

```yaml
GridLayout:
    cols: 3
    spacing: dp(6)
    row_default_height: dp(60)
    row_force_default: True
    Button:
        text: "7"
    Button:
        text: "8"
    Button:
        text: "9"
```

### 7.3 `AnchorLayout`, `FloatLayout`, `RelativeLayout`, `StackLayout`

```yaml
AnchorLayout:                    # Ancla a una posición
    anchor_x: "right"
    anchor_y: "bottom"
    Button:
        size_hint: None, None
        size: dp(56), dp(56)
        text: "+"

FloatLayout:                     # Posición relativa libre
    Button:
        size_hint: 0.3, 0.1
        pos_hint: {"center_x": 0.5, "top": 0.95}

StackLayout:                     # Coloca los hijos uno tras otro, saltando de línea (etiquetas)
    orientation: "lr-tb"
```

### 7.4 Desplazamiento

```yaml
ScrollView:
    do_scroll_x: False
    BoxLayout:
        orientation: "vertical"
        size_hint_y: None
        height: self.minimum_height         # ¡Imprescindible para que haga scroll!
        spacing: dp(6)
        # ... muchos hijos con height fijo
```

### 7.5 `size_hint` y `pos_hint`

| Propiedad | Significado |
|---|---|
| `size_hint: 0.5, 1` | 50 % del ancho y 100 % del alto del padre. |
| `size_hint: None, None` + `size: dp(100), dp(40)` | Tamaño fijo. |
| `size_hint_x: None` + `width: dp(120)` | Solo ancho fijo. |
| `pos_hint: {"center_x": .5, "center_y": .5}` | Centrado (en `FloatLayout`). |
| `pos_hint: {"x": 0, "top": 1}` | Arriba a la izquierda. |

---

## 8. Propiedades y eventos

### 8.1 Propiedades Kivy

```python
from kivy.properties import (StringProperty, NumericProperty, BooleanProperty, ListProperty,
                             DictProperty, ObjectProperty, OptionProperty, AliasProperty)


class Tarjeta(BoxLayout):
    titulo = StringProperty("")
    importe = NumericProperty(0)
    pagado = BooleanProperty(False)
    etiquetas = ListProperty([])
    estado = OptionProperty("pendiente", options=["pendiente", "enviado", "entregado"])
    color_fondo = ListProperty([1, 1, 1, 1])

    def on_pagado(self, instancia, valor):              # Se ejecuta al cambiar «pagado»
        self.color_fondo = [0.85, 0.95, 0.85, 1] if valor else [1, 1, 1, 1]
```

> Las propiedades Kivy se declaran **a nivel de clase**. Notifican sus cambios, se pueden enlazar en KV y disparan automáticamente el método `on_<nombre>`.

### 8.2 Enlazar cambios desde Python

```python
tarjeta.bind(importe=lambda instancia, valor: print("Nuevo importe:", valor))
tarjeta.bind(pagado=self.actualizar_totales)
tarjeta.unbind(pagado=self.actualizar_totales)
```

### 8.3 Eventos táctiles

```python
class Lienzo(Widget):
    def on_touch_down(self, touch):
        if self.collide_point(*touch.pos):              # Solo si el toque es sobre este widget
            print("Toque en", touch.pos, "doble:", touch.is_double_tap)
            return True                                 # Consumir el evento
        return super().on_touch_down(touch)

    def on_touch_move(self, touch): ...
    def on_touch_up(self, touch): ...
```

> Los eventos táctiles llegan a **todos** los widgets; compruebe `collide_point` y devuelva `True` si lo ha gestionado.

### 8.4 Teclado

```python
from kivy.core.window import Window

class MiApp(App):
    def build(self):
        Window.bind(on_keyboard=self.al_teclear)
        ...

    def al_teclear(self, ventana, tecla, scancode, codepoint, modificadores):
        if tecla == 27:                         # Esc / botón «Atrás» en Android
            return self.volver()                # Devolver True evita cerrar la aplicación
        if "ctrl" in modificadores and codepoint == "s":
            self.guardar()
            return True
```

### 8.5 Eventos propios

```python
from kivy.event import EventDispatcher

class Carrito(EventDispatcher):
    __events__ = ("on_actualizado",)

    def anadir(self, producto):
        ...
        self.dispatch("on_actualizado", producto)

    def on_actualizado(self, producto):         # Manejador por defecto (obligatorio)
        pass
```

---

## 9. Varias pantallas: ScreenManager

```python
from kivy.uix.screenmanager import ScreenManager, Screen, SlideTransition, FadeTransition


class PantallaInicio(Screen):
    pass


class PantallaDetalle(Screen):
    producto = StringProperty("")
```

```yaml
<PantallaInicio>:
    name: "inicio"
    BoxLayout:
        orientation: "vertical"
        Button:
            text: "Ver detalle"
            on_release:
                app.root.get_screen("detalle").producto = "Teclado"
                app.root.transition.direction = "left"
                app.root.current = "detalle"

<PantallaDetalle>:
    name: "detalle"
    BoxLayout:
        orientation: "vertical"
        Label:
            text: f"Producto: {root.producto}"
        Button:
            text: "Volver"
            on_release:
                app.root.transition.direction = "right"
                app.root.current = "inicio"
```

```python
class TiendaApp(App):
    def build(self):
        gestor = ScreenManager(transition=SlideTransition(duration=0.25))
        gestor.add_widget(PantallaInicio())
        gestor.add_widget(PantallaDetalle())
        return gestor
```

Eventos de pantalla: `on_pre_enter`, `on_enter`, `on_pre_leave`, `on_leave` (útiles para cargar datos al entrar).

Transiciones: `SlideTransition`, `FadeTransition`, `SwapTransition`, `WipeTransition`, `NoTransition`, `CardTransition`.

---

## 10. Listas eficientes: RecycleView

`RecycleView` muestra listas largas creando solo los widgets visibles y reutilizándolos al desplazarse. Úselo en lugar de añadir cientos de widgets a un `ScrollView`.

```python
from kivy.uix.recycleview import RecycleView
from kivy.uix.boxlayout import BoxLayout
from kivy.properties import StringProperty, NumericProperty


class FilaProducto(BoxLayout):
    nombre = StringProperty("")
    precio = NumericProperty(0)
    indice = NumericProperty(0)


class ListaProductos(RecycleView):
    def cargar(self, productos: list[dict]):
        self.data = [{"nombre": p["nombre"], "precio": p["precio"], "indice": i}
                     for i, p in enumerate(productos)]
```

```yaml
<FilaProducto>:
    size_hint_y: None
    height: dp(56)
    padding: dp(12), 0
    Label:
        text: root.nombre
        text_size: self.size
        halign: "left"
        valign: "middle"
    Label:
        text: f"{root.precio:.2f} €"
        size_hint_x: None
        width: dp(90)

<ListaProductos>:
    viewclass: "FilaProducto"
    RecycleBoxLayout:
        orientation: "vertical"
        default_size: None, dp(56)
        default_size_hint: 1, None
        size_hint_y: None
        height: self.minimum_height
        spacing: dp(2)
```

> Para modificar la lista, cambie `self.data` (una lista de diccionarios). Cada diccionario asigna las propiedades de una fila. Como las filas se reutilizan, **todo** el estado visual debe venir de `data`, no guardarse en el widget.

---

## 11. Ventanas emergentes y notificaciones

### 11.1 `Popup`

```python
from kivy.uix.popup import Popup

def confirmar(titulo: str, mensaje: str, al_aceptar) -> None:
    contenido = BoxLayout(orientation="vertical", padding=dp(12), spacing=dp(12))
    contenido.add_widget(Label(text=mensaje))
    botones = BoxLayout(size_hint_y=None, height=dp(44), spacing=dp(8))
    popup = Popup(title=titulo, content=contenido, size_hint=(0.85, None), height=dp(220), auto_dismiss=False)
    cancelar = Button(text="Cancelar", on_release=popup.dismiss)
    aceptar = Button(text="Aceptar", on_release=lambda *_: (popup.dismiss(), al_aceptar()))
    botones.add_widget(cancelar)
    botones.add_widget(aceptar)
    contenido.add_widget(botones)
    popup.open()
```

O en KV:

```yaml
<PopupAviso@Popup>:
    title: "Aviso"
    size_hint: 0.8, 0.4
    BoxLayout:
        orientation: "vertical"
        Label:
            text: "Operación completada"
        Button:
            text: "Cerrar"
            size_hint_y: None
            height: dp(44)
            on_release: root.dismiss()
```

```python
from kivy.factory import Factory
Factory.PopupAviso().open()
```

### 11.2 Mensaje temporal (*toast*)

```python
from kivy.animation import Animation

def toast(raiz, texto: str, segundos: float = 2.0) -> None:
    etiqueta = Label(text=texto, size_hint=(None, None), size=(dp(280), dp(44)),
                     pos_hint={"center_x": 0.5, "y": 0.05}, opacity=0)
    with etiqueta.canvas.before:
        from kivy.graphics import Color, RoundedRectangle
        Color(0.1, 0.1, 0.1, 0.85)
        fondo = RoundedRectangle(radius=[dp(10)])
    etiqueta.bind(pos=lambda *a: setattr(fondo, "pos", etiqueta.pos),
                  size=lambda *a: setattr(fondo, "size", etiqueta.size))
    raiz.add_widget(etiqueta)
    (Animation(opacity=1, d=0.2) + Animation(d=segundos) + Animation(opacity=0, d=0.3)).start(etiqueta)
    Clock.schedule_once(lambda *_: raiz.remove_widget(etiqueta), segundos + 0.6)
```

> `toast` necesita que `raiz` sea un `FloatLayout` (o derivado) para usar `pos_hint`.

---

## 12. Gráficos, animaciones y gestos

### 12.1 Canvas: fondos, bordes y formas

```yaml
<Tarjeta>:
    canvas.before:
        Color:
            rgba: 1, 1, 1, 1
        RoundedRectangle:
            pos: self.pos
            size: self.size
            radius: [dp(12)]
        Color:
            rgba: 0.15, 0.39, 0.92, 1
        Line:
            rounded_rectangle: (*self.pos, *self.size, dp(12))
            width: 1.2
```

> `canvas.before` dibuja **detrás** del contenido; `canvas.after`, delante. Enlace `pos` y `size` para que el dibujo siga al widget al redimensionarse.

Colores en Kivy: tuplas RGBA con valores entre 0 y 1. Conversión desde hexadecimal:

```python
from kivy.utils import get_color_from_hex
get_color_from_hex("#2563EB")         # [0.145, 0.388, 0.922, 1.0]
```

### 12.2 Fondo de la ventana

```python
from kivy.core.window import Window
Window.clearcolor = (0.96, 0.97, 0.98, 1)
```

### 12.3 Animaciones

```python
from kivy.animation import Animation

anim = Animation(opacity=0, d=0.3) + Animation(opacity=1, d=0.3)     # Secuencia
anim &= Animation(size=(dp(200), dp(60)), d=0.6, t="out_back")       # En paralelo
anim.bind(on_complete=lambda *_: print("Fin"))
anim.start(widget)
anim.repeat = True
Animation.cancel_all(widget)
```

Transiciones (`t`): `linear`, `in_quad`, `out_quad`, `in_out_cubic`, `out_back`, `out_bounce`, `out_elastic`…

### 12.4 Gestos: deslizar para eliminar

```python
class FilaDeslizable(BoxLayout):
    def on_touch_down(self, touch):
        if self.collide_point(*touch.pos):
            touch.ud["inicio_x"] = touch.x              # ud: diccionario propio del toque
        return super().on_touch_down(touch)

    def on_touch_up(self, touch):
        inicio = touch.ud.get("inicio_x")
        if inicio is not None and inicio - touch.x > dp(100):     # Deslizamiento a la izquierda
            self.eliminar()
            return True
        return super().on_touch_up(touch)
```

---

## 13. Tiempo, tareas y concurrencia

### 13.1 `Clock`

```python
from kivy.clock import Clock

Clock.schedule_once(self.cargar_datos, 0)          # En el siguiente fotograma (tras construir la interfaz)
Clock.schedule_once(self.ocultar_aviso, 3)         # A los 3 segundos
evento = Clock.schedule_interval(self.actualizar, 1)   # Cada segundo
evento.cancel()                                    # Cancelar

def actualizar(self, dt):                          # dt = tiempo transcurrido
    ...
```

> Las funciones programadas reciben el argumento `dt`.

### 13.2 Tareas lentas en un hilo

La interfaz de Kivy solo debe modificarse desde el hilo principal. Use un hilo para el trabajo y `@mainthread` (o `Clock.schedule_once`) para actualizar la interfaz:

```python
import threading
from kivy.clock import mainthread


class Pantalla(Screen):
    def descargar(self):
        self.ids.estado.text = "Descargando…"
        threading.Thread(target=self._trabajo, daemon=True).start()

    def _trabajo(self):
        datos = obtener_datos_de_la_api()        # Lento
        self._mostrar(datos)

    @mainthread
    def _mostrar(self, datos):
        self.ids.estado.text = f"{len(datos)} registros"
```

### 13.3 Peticiones HTTP sin bloquear

```python
from kivy.network.urlrequest import UrlRequest

UrlRequest("https://api.ejemplo.com/productos",
           on_success=lambda req, resultado: self.cargar(resultado),
           on_failure=lambda req, r: print("Error", r),
           on_error=lambda req, e: print("Error de red", e),
           timeout=10)
```

---

## 14. Almacenamiento y configuración

### 14.1 Carpeta de datos del usuario

```python
from pathlib import Path

class MiApp(App):
    def build(self):
        self.ruta_datos = Path(self.user_data_dir) / "datos.json"     # Carpeta escribible en cada plataforma
        ...
```

> En Android e iOS, la carpeta de la aplicación es de solo lectura: guarde siempre los datos en `user_data_dir`.

### 14.2 `JsonStore`

```python
from kivy.storage.jsonstore import JsonStore

almacen = JsonStore(str(Path(app.user_data_dir) / "ajustes.json"))
almacen.put("usuario", nombre="Ana", tema="oscuro")
if almacen.exists("usuario"):
    almacen.get("usuario")["tema"]
almacen.delete("usuario")
```

### 14.3 SQLite

`sqlite3` funciona en todas las plataformas, incluidas Android e iOS:

```python
import sqlite3
conexion = sqlite3.connect(Path(self.user_data_dir) / "app.db")
```

### 14.4 Configuración de la aplicación

```python
class MiApp(App):
    def build_config(self, config):
        config.setdefaults("general", {"moneda": "€", "tema": "claro"})

    def build_settings(self, settings):
        settings.add_json_panel("General", self.config, data="""[
            {"type": "options", "title": "Tema", "section": "general", "key": "tema",
             "options": ["claro", "oscuro"]},
            {"type": "string", "title": "Moneda", "section": "general", "key": "moneda"}
        ]""")
```

`app.open_settings()` (o la tecla F1 en escritorio) muestra el panel de ajustes generado automáticamente.

### 14.5 Configuración de la ventana (escritorio)

```python
from kivy.config import Config
Config.set("graphics", "width", "420")
Config.set("graphics", "height", "760")
Config.set("input", "mouse", "mouse,multitouch_on_demand")    # Quitar los puntos rojos del clic derecho
# ⚠ Debe ejecutarse ANTES de importar otros módulos de Kivy
```

O tras arrancar:

```python
Window.size = (420, 760)
Window.minimum_width, Window.minimum_height = 360, 600
```

---

## 15. Diseño adaptable (tamaños y densidad)

### 15.1 Unidades

| Unidad | Uso |
|---|---|
| `dp(n)` | Píxeles independientes de la densidad: tamaños, márgenes, alturas. |
| `sp(n)` | Como `dp`, pero respeta el tamaño de letra elegido por el usuario: fuentes. |
| Píxeles sin unidad | Evítelos: se verán diminutos en móviles de alta densidad. |

### 15.2 Adaptar la disposición al ancho

```yaml
<Panel>:
    GridLayout:
        cols: 1 if root.width < dp(600) else 2      # Una columna en móvil, dos en tableta/escritorio
```

```python
class Principal(BoxLayout):
    def on_size(self, *_):
        self.orientation = "vertical" if self.width < dp(600) else "horizontal"
```

### 15.3 Teclado virtual en móviles

```python
Window.softinput_mode = "below_target"        # Desplaza la interfaz para que el campo activo sea visible
```

---

## 16. KivyMD: Material Design

**KivyMD** añade componentes con el estilo Material Design de Google (barras superiores, tarjetas, botones flotantes, cuadros de diálogo, navegación inferior, temas claro/oscuro).

```bash
pip install kivymd
```

```python
from kivymd.app import MDApp
from kivy.lang import Builder

KV = """
MDScreen:
    md_bg_color: self.theme_cls.backgroundColor
    MDButton:
        style: "filled"
        pos_hint: {"center_x": .5, "center_y": .5}
        on_release: app.saludar()
        MDButtonText:
            text: "Saludar"
"""


class Demo(MDApp):
    def build(self):
        self.theme_cls.theme_style = "Light"
        self.theme_cls.primary_palette = "Blue"
        return Builder.load_string(KV)

    def saludar(self):
        print("Hola")


Demo().run()
```

> La API de KivyMD ha cambiado mucho entre versiones (1.x y 2.x). Consulte la documentación correspondiente a la versión instalada.

---

## 17. Empaquetar para escritorio y móvil

### 17.1 Escritorio

```bash
pip install pyinstaller
pyinstaller --name MiApp --windowed --onefile main.py
```

Incluya los archivos `.kv`, imágenes y fuentes (`--add-data "app.kv:."`). Consulte la guía de empaquetado de Kivy para su sistema.

### 17.2 Android con Buildozer

Buildozer funciona en **Linux** o macOS (en Windows, mediante **WSL**):

```bash
pip install buildozer
buildozer init                       # Crea buildozer.spec
buildozer -v android debug           # Compila el APK (la primera vez descarga el SDK/NDK)
buildozer android deploy run logcat  # Instala en un móvil conectado y muestra el registro
```

Ajustes clave de `buildozer.spec`:

```ini
title = Mis Gastos
package.name = misgastos
package.domain = org.miempresa
source.include_exts = py,png,jpg,kv,json,ttf
requirements = python3,kivy
orientation = portrait
android.permissions = INTERNET
version = 1.0
```

> Añada a `requirements` todas las librerías que use (por ejemplo, `kivymd`, `requests`). Las librerías con partes en C necesitan una «receta» de python-for-android.

### 17.3 iOS

Se compila con **kivy-ios** en un Mac con Xcode:

```bash
pip install kivy-ios
toolchain build kivy
toolchain create MiApp ~/proyectos/miapp
```

---

## 18. Ejemplo práctico completo

**Mis Gastos**: aplicación móvil/escritorio con dos pantallas (lista y alta), `RecycleView` con filas deslizables para eliminar, filtro por categoría con botones, total reactivo, formulario con validación y selector de categoría, persistencia en `user_data_dir` (JSON), notificaciones tipo *toast*, botón «Atrás»/Esc y diseño en KV con estilos reutilizables.

**`main.py`**

```python
"""
Mis Gastos — Kivy.
Requisitos: pip install "kivy[base]"
Ejecutar: python main.py   (el archivo misgastos.kv debe estar en la misma carpeta)
"""
import json
from datetime import date
from pathlib import Path

from kivy.animation import Animation
from kivy.app import App
from kivy.clock import Clock
from kivy.core.window import Window
from kivy.graphics import Color, RoundedRectangle
from kivy.metrics import dp
from kivy.properties import ListProperty, NumericProperty, StringProperty
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.recycleview import RecycleView
from kivy.uix.screenmanager import Screen
from kivy.utils import get_color_from_hex

CATEGORIAS = {"Comida": "#F59E0B", "Transporte": "#2563EB", "Ocio": "#7C3AED",
              "Hogar": "#16A34A", "Otros": "#64748B"}


def euros(v: float) -> str:
    return f"{v:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


# ------------------------------------------------------------------ Datos
class Almacen:
    def __init__(self, ruta: Path):
        self.ruta = ruta
        self.gastos: list[dict] = []
        if ruta.exists():
            self.gastos = json.loads(ruta.read_text(encoding="utf-8"))
        else:
            hoy = date.today().isoformat()
            self.gastos = [
                {"id": 1, "concepto": "Supermercado", "categoria": "Comida", "importe": 54.30, "fecha": hoy},
                {"id": 2, "concepto": "Abono transporte", "categoria": "Transporte", "importe": 20.00, "fecha": hoy},
                {"id": 3, "concepto": "Cine", "categoria": "Ocio", "importe": 9.50, "fecha": hoy},
            ]
            self.guardar()

    def guardar(self) -> None:
        self.ruta.parent.mkdir(parents=True, exist_ok=True)
        self.ruta.write_text(json.dumps(self.gastos, ensure_ascii=False, indent=2), encoding="utf-8")

    def anadir(self, concepto: str, categoria: str, importe: float) -> None:
        nuevo_id = max((g["id"] for g in self.gastos), default=0) + 1
        self.gastos.insert(0, {"id": nuevo_id, "concepto": concepto, "categoria": categoria,
                               "importe": round(importe, 2), "fecha": date.today().isoformat()})
        self.guardar()

    def eliminar(self, id_: int) -> dict | None:
        for i, g in enumerate(self.gastos):
            if g["id"] == id_:
                eliminado = self.gastos.pop(i)
                self.guardar()
                return eliminado
        return None


# ------------------------------------------------------------------ Widgets
class FilaGasto(BoxLayout):
    id_gasto = NumericProperty(0)
    concepto = StringProperty("")
    categoria = StringProperty("")
    importe_texto = StringProperty("")
    fecha = StringProperty("")
    color = ListProperty([0.5, 0.5, 0.5, 1])

    def on_touch_down(self, touch):
        if self.collide_point(*touch.pos):
            touch.ud["fila"] = self.id_gasto
            touch.ud["x0"] = touch.x
        return super().on_touch_down(touch)

    def on_touch_up(self, touch):
        if touch.ud.get("fila") == self.id_gasto and touch.ud["x0"] - touch.x > dp(90):
            App.get_running_app().eliminar_gasto(self.id_gasto)
            return True
        return super().on_touch_up(touch)


class ListaGastos(RecycleView):
    pass


class PantallaLista(Screen):
    filtro = StringProperty("Todas")
    total_texto = StringProperty("")

    def on_kv_post(self, *_):              # Los ids ya están disponibles
        self.refrescar()

    def on_pre_enter(self, *_):
        self.refrescar()

    def on_filtro(self, *_):
        self.refrescar()

    def refrescar(self) -> None:
        if "lista" not in self.ids:         # La pantalla inicial recibe on_pre_enter antes de tener ids
            return
        gastos = App.get_running_app().almacen.gastos
        visibles = [g for g in gastos if self.filtro == "Todas" or g["categoria"] == self.filtro]
        self.ids.lista.data = [{
            "id_gasto": g["id"], "concepto": g["concepto"], "categoria": g["categoria"],
            "importe_texto": euros(g["importe"]),
            "fecha": date.fromisoformat(g["fecha"]).strftime("%d/%m/%Y"),
            "color": get_color_from_hex(CATEGORIAS.get(g["categoria"], "#64748B")),
        } for g in visibles]
        self.total_texto = euros(sum(g["importe"] for g in visibles))
        self.ids.vacio.opacity = 0 if visibles else 1


class PantallaAlta(Screen):
    categoria = StringProperty("Comida")
    error = StringProperty("")

    def on_pre_enter(self, *_):
        if "concepto" not in self.ids:
            return
        self.ids.concepto.text = ""
        self.ids.importe.text = ""
        self.error = ""
        Clock.schedule_once(lambda *_: setattr(self.ids.concepto, "focus", True), 0.3)

    def guardar(self) -> None:
        concepto = self.ids.concepto.text.strip()
        try:
            importe = float(self.ids.importe.text.replace(",", "."))
        except ValueError:
            importe = 0
        if len(concepto) < 2:
            self.error = "Indique un concepto."
        elif importe <= 0:
            self.error = "Indique un importe mayor que cero."
        else:
            app = App.get_running_app()
            app.almacen.anadir(concepto, self.categoria, importe)
            app.ir_a("lista", "right")
            app.toast(f"Gasto de {euros(importe)} añadido")


# ------------------------------------------------------------------ Aplicación
class MisGastosApp(App):
    categorias = ListProperty(list(CATEGORIAS))

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        # Los datos se crean aquí porque el archivo .kv se carga ANTES de build()
        # y sus pantallas ya los necesitan al construirse.
        self.almacen = Almacen(Path(self.user_data_dir) / "gastos.json")

    def build(self):
        self.title = "Mis Gastos"
        Window.clearcolor = get_color_from_hex("#F3F4F6")
        Window.bind(on_keyboard=self._tecla)
        # Sin return: la interfaz es la regla raíz de misgastos.kv (cargado automáticamente)

    def ir_a(self, pantalla: str, direccion: str = "left") -> None:
        gestor = self.root.ids.gestor
        gestor.transition.direction = direccion
        gestor.current = pantalla

    def eliminar_gasto(self, id_: int) -> None:
        eliminado = self.almacen.eliminar(id_)
        if eliminado:
            self.root.ids.gestor.get_screen("lista").refrescar()
            self.toast(f"«{eliminado['concepto']}» eliminado")

    def color_categoria(self, nombre: str):
        return get_color_from_hex(CATEGORIAS.get(nombre, "#64748B"))

    def toast(self, texto: str, segundos: float = 1.8) -> None:
        etiqueta = Label(text=texto, size_hint=(None, None), size=(dp(300), dp(44)),
                         pos_hint={"center_x": 0.5, "y": 0.04}, opacity=0)
        with etiqueta.canvas.before:
            Color(0.07, 0.09, 0.15, 0.9)
            fondo = RoundedRectangle(radius=[dp(12)])
        etiqueta.bind(pos=lambda *_: setattr(fondo, "pos", etiqueta.pos),
                      size=lambda *_: setattr(fondo, "size", etiqueta.size))
        self.root.add_widget(etiqueta)
        (Animation(opacity=1, d=0.15) + Animation(d=segundos) + Animation(opacity=0, d=0.3)).start(etiqueta)
        Clock.schedule_once(lambda *_: self.root.remove_widget(etiqueta), segundos + 0.6)

    def _tecla(self, ventana, tecla, *args):
        if tecla == 27 and self.root.ids.gestor.current != "lista":     # Esc / Atrás
            self.ir_a("lista", "right")
            return True
        return False


if __name__ == "__main__":
    MisGastosApp().run()
```

**`misgastos.kv`**

```yaml
#:import dp kivy.metrics.dp
#:import sp kivy.metrics.sp
#:import hex kivy.utils.get_color_from_hex
#:import SlideTransition kivy.uix.screenmanager.SlideTransition

# ---------- Estilos reutilizables
<Texto@Label>:
    color: hex("#111827")
    text_size: self.size
    halign: "left"
    valign: "middle"

<BotonPrimario@Button>:
    background_normal: ""
    background_down: ""
    background_color: hex("#2563EB") if self.state == "normal" else hex("#1D4ED8")
    color: 1, 1, 1, 1
    font_size: sp(16)
    bold: True
    size_hint_y: None
    height: dp(50)

<Chip@ToggleButton>:
    background_normal: ""
    background_down: ""
    background_color: hex("#2563EB") if self.state == "down" else hex("#E5E7EB")
    color: (1, 1, 1, 1) if self.state == "down" else hex("#374151")
    font_size: sp(13)
    size_hint: None, None
    size: self.texture_size[0] + dp(28), dp(34)
    allow_no_selection: False

<Campo@TextInput>:
    multiline: False
    write_tab: False
    size_hint_y: None
    height: dp(48)
    font_size: sp(16)
    padding: dp(12), dp(13)
    background_normal: ""
    background_active: ""
    background_color: 1, 1, 1, 1

# ---------- Fila de la lista
<FilaGasto>:
    size_hint_y: None
    height: dp(68)
    padding: dp(14), dp(8)
    spacing: dp(12)
    canvas.before:
        Color:
            rgba: 1, 1, 1, 1
        RoundedRectangle:
            pos: self.x + dp(10), self.y + dp(3)
            size: self.width - dp(20), self.height - dp(6)
            radius: [dp(10)]
    Widget:
        size_hint_x: None
        width: dp(10)
        canvas:
            Color:
                rgba: root.color
            Ellipse:
                pos: self.center_x - dp(5), self.center_y - dp(5)
                size: dp(10), dp(10)
    BoxLayout:
        orientation: "vertical"
        Texto:
            text: root.concepto
            font_size: sp(16)
            bold: True
        Texto:
            text: f"{root.categoria} · {root.fecha}"
            font_size: sp(12)
            color: hex("#6B7280")
    Texto:
        text: root.importe_texto
        size_hint_x: None
        width: dp(110)
        halign: "right"
        font_size: sp(16)
        bold: True

<ListaGastos>:
    viewclass: "FilaGasto"
    RecycleBoxLayout:
        orientation: "vertical"
        default_size: None, dp(68)
        default_size_hint: 1, None
        size_hint_y: None
        height: self.minimum_height

# ---------- Pantallas
<PantallaLista>:
    name: "lista"
    BoxLayout:
        orientation: "vertical"
        BoxLayout:
            size_hint_y: None
            height: dp(110)
            orientation: "vertical"
            padding: dp(18), dp(12)
            canvas.before:
                Color:
                    rgba: hex("#2563EB")
                Rectangle:
                    pos: self.pos
                    size: self.size
            Label:
                text: "Total" if root.filtro == "Todas" else f"Total · {root.filtro}"
                color: 1, 1, 1, 0.8
                text_size: self.size
                halign: "left"
            Label:
                text: root.total_texto
                font_size: sp(30)
                bold: True
                color: 1, 1, 1, 1
                text_size: self.size
                halign: "left"
        ScrollView:
            size_hint_y: None
            height: dp(54)
            do_scroll_y: False
            StackLayout:
                id: chips
                orientation: "lr-tb"
                size_hint_x: None
                width: self.minimum_width
                padding: dp(12), dp(10)
                spacing: dp(8)
                Chip:
                    text: "Todas"
                    group: "filtro"
                    state: "down"
                    on_press: root.filtro = "Todas"
                Chip:
                    text: "Comida"
                    group: "filtro"
                    on_press: root.filtro = self.text
                Chip:
                    text: "Transporte"
                    group: "filtro"
                    on_press: root.filtro = self.text
                Chip:
                    text: "Ocio"
                    group: "filtro"
                    on_press: root.filtro = self.text
                Chip:
                    text: "Hogar"
                    group: "filtro"
                    on_press: root.filtro = self.text
                Chip:
                    text: "Otros"
                    group: "filtro"
                    on_press: root.filtro = self.text
        FloatLayout:
            ListaGastos:
                id: lista
                pos_hint: {"x": 0, "y": 0}
            Label:
                id: vacio
                text: "No hay gastos en esta categoría"
                color: hex("#9CA3AF")
                opacity: 0
                pos_hint: {"center_x": .5, "center_y": .6}
            Label:
                text: "Desliza una fila a la izquierda para eliminarla"
                color: hex("#9CA3AF")
                font_size: sp(11)
                size_hint_y: None
                height: dp(20)
                pos_hint: {"center_x": .5, "y": .0}
        BoxLayout:
            size_hint_y: None
            height: dp(74)
            padding: dp(14), dp(12)
            BotonPrimario:
                text: "+  Añadir gasto"
                on_release: app.ir_a("alta", "left")

<PantallaAlta>:
    name: "alta"
    BoxLayout:
        orientation: "vertical"
        padding: dp(18)
        spacing: dp(12)
        BoxLayout:
            size_hint_y: None
            height: dp(44)
            Button:
                text: "‹ Volver"
                size_hint_x: None
                width: dp(90)
                background_normal: ""
                background_color: 0, 0, 0, 0
                color: hex("#2563EB")
                on_release: app.ir_a("lista", "right")
            Texto:
                text: "Nuevo gasto"
                font_size: sp(20)
                bold: True
                halign: "center"
            Widget:
                size_hint_x: None
                width: dp(90)
        Texto:
            text: "Concepto"
            size_hint_y: None
            height: dp(22)
        Campo:
            id: concepto
            hint_text: "Ej.: Supermercado"
            on_text_validate: importe.focus = True
        Texto:
            text: "Importe (€)"
            size_hint_y: None
            height: dp(22)
        Campo:
            id: importe
            hint_text: "0,00"
            input_filter: lambda texto, deshacer: "".join(c for c in texto if c in "0123456789,.")
            on_text_validate: root.guardar()
        Texto:
            text: "Categoría"
            size_hint_y: None
            height: dp(22)
        GridLayout:
            cols: 3
            spacing: dp(8)
            size_hint_y: None
            height: self.minimum_height
            row_default_height: dp(42)
            row_force_default: True
            Chip:
                text: "Comida"
                group: "categoria"
                state: "down"
                size_hint: 1, None
                on_press: root.categoria = self.text
            Chip:
                text: "Transporte"
                group: "categoria"
                size_hint: 1, None
                on_press: root.categoria = self.text
            Chip:
                text: "Ocio"
                group: "categoria"
                size_hint: 1, None
                on_press: root.categoria = self.text
            Chip:
                text: "Hogar"
                group: "categoria"
                size_hint: 1, None
                on_press: root.categoria = self.text
            Chip:
                text: "Otros"
                group: "categoria"
                size_hint: 1, None
                on_press: root.categoria = self.text
        Label:
            text: root.error
            color: hex("#DC2626")
            size_hint_y: None
            height: dp(24)
        Widget:
        BotonPrimario:
            text: "Guardar"
            on_release: root.guardar()

# ---------- Raíz
FloatLayout:
    ScreenManager:
        id: gestor
        transition: SlideTransition(duration=0.22)
        PantallaLista:
        PantallaAlta:
```

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `[CRITICAL] [App] Unable to get a Window` / errores de OpenGL | Controladores gráficos antiguos o máquina virtual sin aceleración. | Actualice los controladores; en Windows pruebe `KIVY_GL_BACKEND=angle_sdl2`. |
| La ventana sale vacía | No se cargó el archivo `.kv` (nombre incorrecto) o `build()` no devuelve nada sin KV raíz. | Nombre el archivo como la clase `App` sin «App», en minúsculas. |
| `AttributeError: 'super' object has no attribute '__getattr__'` | Acceso a `self.ids.x` que no existe o antes de que la interfaz esté construida (p. ej. en el `on_pre_enter` de la primera pantalla). | Revise los `id`; compruebe `if "x" in self.ids`, o acceda en `on_kv_post` o con `Clock.schedule_once`. |
| `AttributeError` en una pantalla por un atributo creado en `build()` | El archivo `.kv` automático se carga **antes** de `build()`. | Cree los datos en `__init__` de la `App` o cárguelos en `on_start`. |
| Reglas duplicadas / widgets repetidos | El archivo `.kv` se cargó dos veces (automática y manualmente). | Cárguelo una sola vez. |
| El `ScrollView` no se desplaza | El hijo tiene `size_hint_y` por defecto. | `size_hint_y: None` y `height: self.minimum_height`. |
| `halign` no tiene efecto en un `Label` | Falta `text_size`. | `text_size: self.size`. |
| Un widget ocupa todo el espacio ignorando `height` | `size_hint_y` sigue activo. | `size_hint_y: None`. |
| El fondo dibujado en `canvas` no sigue al widget | No se enlazaron `pos` y `size`. | En KV use `pos: self.pos` y `size: self.size`. |
| Elementos de `RecycleView` con datos cruzados | Estado guardado en el widget reciclado. | Todo el estado debe venir de `data`. |
| La interfaz se congela | Tarea larga en el hilo principal. | Hilo + `@mainthread`. |
| Texto diminuto en el móvil | Tamaños en píxeles. | Use `dp()` y `sp()`. |
| Clic derecho dibuja un punto rojo (escritorio) | Emulación multitáctil. | `Config.set("input", "mouse", "mouse,multitouch_on_demand")` antes de importar Kivy. |
| No se pueden guardar datos en Android | Se escribe en la carpeta de la aplicación. | Use `App.user_data_dir`. |
| La app se cierra con el botón «Atrás» de Android | Esc (27) no gestionado. | Capture `on_keyboard` y devuelva `True`. |

---

## 20. Buenas prácticas

- Separe el diseño (archivos `.kv`) de la lógica (Python).
- Use **propiedades Kivy** y enlaces en KV para una interfaz reactiva, en lugar de actualizar widgets a mano.
- Use `dp()` y `sp()` para todos los tamaños.
- Use `RecycleView` para listas de más de unas decenas de elementos.
- Defina estilos reutilizables con clases dinámicas (`<BotonPrimario@Button>`).
- Organice las pantallas con `ScreenManager` y cargue datos en `on_pre_enter`.
- Ejecute las tareas lentas en hilos y actualice la interfaz con `@mainthread`.
- Guarde los datos en `user_data_dir` para que funcione en todas las plataformas.
- Gestione la tecla Esc/«Atrás» en aplicaciones móviles.
- Pruebe en distintos tamaños de ventana (`Window.size`) y, si es posible, en un dispositivo real.

---

## 21. Referencia rápida

```python
class MiApp(App):
    def build(self): return WidgetRaiz()          # o archivo mi.kv
MiApp().run()

BoxLayout(orientation) / GridLayout(cols) / FloatLayout / AnchorLayout / StackLayout / ScrollView
Label(text, markup, font_size=sp()) / Button(on_release) / TextInput(hint_text, input_filter, multiline)
CheckBox / Switch / Slider / Spinner(values) / ToggleButton(group) / ProgressBar / Image(source) / Popup
size_hint: None, None · size: dp(w), dp(h) · pos_hint: {"center_x": .5}

StringProperty / NumericProperty / BooleanProperty / ListProperty / ObjectProperty → on_<prop>()
widget.bind(prop=func) · self.ids.nombre · App.get_running_app() · Factory.Clase()
ScreenManager + Screen(name) · gestor.current = "nombre" · on_pre_enter / on_enter
RecycleView: viewclass + data = [dict, ...]
Clock.schedule_once(f, s) / schedule_interval(f, s) · @mainthread · Animation(prop=v, d=s).start(w)
canvas.before: Color / Rectangle / RoundedRectangle / Line / Ellipse
dp() / sp() · Window.size / clearcolor / bind(on_keyboard=f) · App.user_data_dir · JsonStore
KV: root / self / app / id / #:import / #:set / <Clase@Base>
```

---

## 22. Ejercicios propuestos

1. Cree una calculadora táctil con `GridLayout` cuyo tamaño de botones se adapte a la ventana.
2. Desarrolle una lista de la compra con `RecycleView`, marcado de elementos comprados y persistencia con `JsonStore`.
3. Añada al ejemplo completo una tercera pantalla de estadísticas con un gráfico de barras dibujado con el `canvas`.
4. Consulte una API pública con `UrlRequest` y muestre los resultados en una lista con imágenes asíncronas.
5. Empaquete el ejemplo completo como APK con Buildozer y pruébelo en un móvil Android.

---

## 23. Referencias

- Documentación oficial: https://kivy.org/doc/stable/
- Lenguaje KV: https://kivy.org/doc/stable/guide/lang.html
- Buildozer: https://buildozer.readthedocs.io/
- KivyMD: https://kivymd.readthedocs.io/
