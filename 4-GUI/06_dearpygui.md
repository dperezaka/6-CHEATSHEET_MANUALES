# Manual de uso: Dear PyGui

> **Categoría:** Interfaces gráficas (GUI) · **Utilidad:** Interfaces gráficas rápidas y modernas
> **Tipo:** Librería externa · Renderizado por GPU, modo inmediato (basado en Dear ImGui)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Estructura de una aplicación](#4-estructura-de-una-aplicación)
5. [Ventanas y contenedores](#5-ventanas-y-contenedores)
6. [Widgets](#6-widgets)
7. [Callbacks, valores y etiquetas](#7-callbacks-valores-y-etiquetas)
8. [Disposición de los elementos](#8-disposición-de-los-elementos)
9. [Tablas](#9-tablas)
10. [Gráficos (plots)](#10-gráficos-plots)
11. [Menús, diálogos y emergentes](#11-menús-diálogos-y-emergentes)
12. [Temas y estilos](#12-temas-y-estilos)
13. [Fuentes y caracteres en español](#13-fuentes-y-caracteres-en-español)
14. [Teclado, ratón y eventos](#14-teclado-ratón-y-eventos)
15. [Datos en tiempo real y concurrencia](#15-datos-en-tiempo-real-y-concurrencia)
16. [Herramientas de desarrollo](#16-herramientas-de-desarrollo)
17. [Distribución](#17-distribución)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**Dear PyGui** (DPG) es un framework de interfaces gráficas para Python basado en **Dear ImGui** y acelerado por GPU (DirectX en Windows, Metal en macOS, OpenGL en Linux). Está pensado para herramientas **rápidas, interactivas y con muchos datos**: paneles de control, monitorización en tiempo real, visualización científica, herramientas internas, editores y depuradores.

**Características:**
- Rendimiento muy alto: decenas de miles de puntos en gráficos que se actualizan en tiempo real.
- Gráficos integrados (ImPlot): líneas, barras, dispersión, áreas, mapas de calor, velas, etc.
- Tablas con ordenación, redimensionado y desplazamiento.
- Temas personalizables, modo oscuro por defecto.
- API sencilla basada en funciones y gestores de contexto (`with`).
- Licencia MIT.

**Cuándo usar Dear PyGui:**

| Situación | ¿Dear PyGui? |
|---|---|
| Panel de datos en tiempo real, monitorización, telemetría | ✔ Excelente |
| Herramientas internas para ingeniería o ciencia de datos | ✔ Muy adecuado |
| Prototipos rápidos con gráficos interactivos | ✔ Muy adecuado |
| Aplicación de gestión con aspecto nativo del sistema | ✘ Mejor PySide6/PyQt6 |
| Aplicación móvil | ✘ Mejor Kivy |
| Interfaces muy accesibles (lectores de pantalla) | ✘ Soporte limitado |

> Dear PyGui dibuja su propia interfaz: el aspecto es el mismo en todos los sistemas, más cercano a una herramienta técnica o un editor de videojuegos que a una aplicación de oficina.

---

## 2. Instalación e importación

```bash
pip install dearpygui
```

Requiere una tarjeta gráfica (o controlador) compatible con DirectX 11 (Windows), Metal (macOS) u OpenGL 3.3 (Linux).

```python
import dearpygui.dearpygui as dpg
```

Ver una demostración con todos los componentes:

```python
import dearpygui.dearpygui as dpg
import dearpygui.demo as demo

dpg.create_context()
dpg.create_viewport(title="Demo", width=1200, height=800)
demo.show_demo()
dpg.setup_dearpygui()
dpg.show_viewport()
dpg.start_dearpygui()
dpg.destroy_context()
```

> La demo es la mejor referencia práctica: cada sección muestra el código correspondiente.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Contexto | `create_context()` / `destroy_context()`: inicio y fin de Dear PyGui. |
| *Viewport* | La ventana real del sistema operativo. |
| Ventana (`window`) | Ventana interna de DPG dentro del viewport (se puede mover, redimensionar, cerrar). |
| Ventana principal | `set_primary_window`: una ventana que ocupa todo el viewport. |
| *Item* | Cualquier elemento (botón, texto, tabla, serie de un gráfico…). |
| `tag` | Identificador de un item (texto o número) para consultarlo o modificarlo. |
| Contenedor | Item que contiene otros (`window`, `group`, `child_window`, `table`…). |
| Pila de contenedores | Los items creados dentro de un bloque `with` se añaden a ese contenedor. |
| Callback | Función que se ejecuta al interactuar con un item. |
| Registro | Contenedores especiales: temas, fuentes, texturas, manejadores, valores. |
| Bucle de renderizado | `start_dearpygui()` (automático) o bucle manual con `render_dearpygui_frame()`. |

> A diferencia de Tkinter o Qt, no se crean objetos de clase: se llama a **funciones** que devuelven un identificador.

---

## 4. Estructura de una aplicación

```python
import dearpygui.dearpygui as dpg

dpg.create_context()                                   # 1. Crear el contexto

with dpg.window(label="Saludo", width=400, height=200, tag="principal"):   # 2. Crear la interfaz
    dpg.add_input_text(label="Nombre", tag="nombre", hint="Escriba su nombre")
    dpg.add_button(label="Saludar", callback=lambda: dpg.set_value(
        "resultado", f"Hola, {dpg.get_value('nombre') or 'desconocido'}"))
    dpg.add_text("", tag="resultado")

dpg.create_viewport(title="Mi aplicación", width=600, height=400)   # 3. Ventana del sistema
dpg.setup_dearpygui()
dpg.show_viewport()
dpg.set_primary_window("principal", True)              # 4. (Opcional) ocupar todo el viewport
dpg.start_dearpygui()                                  # 5. Bucle de renderizado
dpg.destroy_context()                                  # 6. Liberar recursos
```

### 4.1 Bucle manual (para actualizar datos en cada fotograma)

```python
dpg.setup_dearpygui()
dpg.show_viewport()
while dpg.is_dearpygui_running():
    actualizar_datos()                   # Código propio (debe ser rápido)
    dpg.render_dearpygui_frame()
dpg.destroy_context()
```

### 4.2 Opciones del viewport

```python
dpg.create_viewport(title="Panel", width=1280, height=800, min_width=800, min_height=500,
                    resizable=True, vsync=True, small_icon="icono.ico", large_icon="icono.ico")
dpg.set_viewport_title("Nuevo título")
dpg.maximize_viewport()
dpg.toggle_viewport_fullscreen()
dpg.get_viewport_client_width(), dpg.get_viewport_client_height()
dpg.set_viewport_resize_callback(lambda: print("Redimensionado"))
dpg.set_exit_callback(lambda: print("Cerrando"))
```

---

## 5. Ventanas y contenedores

### 5.1 Ventanas

```python
with dpg.window(label="Herramientas", tag="herramientas", width=300, height=400, pos=(20, 20),
                no_close=True, no_collapse=True, no_resize=False, no_move=False):
    ...

with dpg.window(label="Aviso", modal=True, show=False, tag="aviso", no_title_bar=False,
                autosize=True):
    dpg.add_text("Operación completada")
    dpg.add_button(label="Cerrar", callback=lambda: dpg.configure_item("aviso", show=False))

dpg.configure_item("aviso", show=True)        # Mostrar
dpg.hide_item("herramientas"); dpg.show_item("herramientas")
```

### 5.2 Contenedores

| Contenedor | Uso |
|---|---|
| `group` | Agrupar items (vertical u horizontal). |
| `child_window` | Zona con tamaño propio y barra de desplazamiento. |
| `collapsing_header` | Sección plegable. |
| `tab_bar` + `tab` | Pestañas. |
| `tree_node` | Árbol plegable. |
| `table` | Tabla (también útil para maquetar). |
| `popup` / `tooltip` | Ventanas emergentes y textos de ayuda. |
| `menu_bar` / `menu` | Menús. |

```python
with dpg.window(label="Panel"):
    with dpg.tab_bar():
        with dpg.tab(label="Datos"):
            with dpg.child_window(height=200, border=True):
                for i in range(50):
                    dpg.add_text(f"Registro {i}")
        with dpg.tab(label="Ajustes"):
            with dpg.collapsing_header(label="Avanzado", default_open=False):
                dpg.add_checkbox(label="Modo depuración")
```

### 5.3 Añadir items a un contenedor desde fuera

```python
dpg.add_text("Nuevo elemento", parent="panel_lista")
dpg.add_button(label="Antes", parent="panel_lista", before="otro_item")

dpg.push_container_stack("panel_lista")
dpg.add_text("Añadido con la pila de contenedores")
dpg.pop_container_stack()
```

---

## 6. Widgets

### 6.1 Texto

```python
dpg.add_text("Texto normal")
dpg.add_text("Texto en color", color=(230, 126, 34))
dpg.add_text("Texto con viñeta", bullet=True)
dpg.add_text("Texto largo que se ajusta al ancho indicado", wrap=300)
dpg.add_separator()
dpg.add_spacer(height=10)
```

### 6.2 Botones

```python
dpg.add_button(label="Guardar", width=120, height=32, callback=guardar)
dpg.add_button(label="⟳", small=True)
dpg.add_button(arrow=True, direction=dpg.mvDir_Right)
dpg.add_selectable(label="Elemento seleccionable", callback=al_seleccionar)
```

### 6.3 Entrada de datos

```python
dpg.add_input_text(label="Nombre", hint="Obligatorio", width=250, tag="nombre")
dpg.add_input_text(label="Contraseña", password=True)
dpg.add_input_text(label="Notas", multiline=True, height=100, width=-1)     # width=-1: todo el ancho
dpg.add_input_text(label="Código", uppercase=True, no_spaces=True, decimal=False)
dpg.add_input_int(label="Unidades", default_value=1, min_value=0, min_clamped=True, step=1)
dpg.add_input_float(label="Precio", default_value=9.99, format="%.2f", step=0.5)
dpg.add_drag_float(label="Ganancia", default_value=1.0, speed=0.01, min_value=0, max_value=10)
dpg.add_slider_int(label="Volumen", default_value=50, min_value=0, max_value=100)
dpg.add_slider_float(label="Umbral", default_value=0.5, max_value=1, format="%.2f")
```

### 6.4 Selección

```python
dpg.add_checkbox(label="Activo", default_value=True)
dpg.add_radio_button(["Diario", "Semanal", "Mensual"], default_value="Semanal", horizontal=True)
dpg.add_combo(["Madrid", "Sevilla", "Valencia"], label="Ciudad", default_value="Madrid", width=150)
dpg.add_listbox(["Norte", "Sur", "Este", "Oeste"], label="Región", num_items=4)
dpg.add_color_edit(label="Color", default_value=(37, 99, 235, 255))
dpg.add_date_picker(level=dpg.mvDatePickerLevel_Day,
                    default_value={"month_day": 24, "month": 10, "year": 126})   # month 0-11, year desde 1900
```

### 6.5 Progreso e indicadores

```python
dpg.add_progress_bar(default_value=0.35, overlay="35 %", width=300, tag="progreso")
dpg.add_loading_indicator(style=1, radius=2)
```

### 6.6 Imágenes

```python
ancho, alto, canales, datos = dpg.load_image("logo.png")
with dpg.texture_registry():
    dpg.add_static_texture(ancho, alto, datos, tag="tex_logo")
dpg.add_image("tex_logo", width=160, height=160 * alto / ancho)
```

---

## 7. Callbacks, valores y etiquetas

### 7.1 Identificadores (`tag`)

```python
dpg.add_input_text(tag="email")          # Tag de texto (recomendado)
id_boton = dpg.add_button(label="OK")     # Sin tag: devuelve un identificador numérico
dpg.generate_uuid()                      # Generar un identificador numérico propio
dpg.does_item_exist("email")
```

### 7.2 Leer y modificar valores

```python
dpg.get_value("email")
dpg.set_value("email", "ana@ejemplo.com")
dpg.get_values(["nombre", "email"])
```

### 7.3 Configurar items

```python
dpg.configure_item("guardar", label="Guardando…", enabled=False)
dpg.get_item_configuration("guardar")
dpg.set_item_label("guardar", "Guardar")
dpg.enable_item("guardar"); dpg.disable_item("guardar")
dpg.delete_item("panel", children_only=True)     # Vaciar un contenedor
dpg.delete_item("panel")                          # Eliminar el item
dpg.get_item_children("panel", slot=1)            # Hijos (slot 1: items normales)
```

### 7.4 Callbacks

Firma completa de un callback: `(sender, app_data, user_data)`.

| Argumento | Contenido |
|---|---|
| `sender` | Tag o identificador del item que lo disparó. |
| `app_data` | Dato asociado (valor nuevo, ruta del archivo seleccionado…). |
| `user_data` | Dato propio indicado al crear el item. |

```python
def al_cambiar(sender, app_data, user_data):
    print(f"{sender} cambió a {app_data} (dato propio: {user_data})")

dpg.add_slider_int(label="Unidades", callback=al_cambiar, user_data="producto_A")
dpg.add_input_text(label="Buscar", callback=filtrar, on_enter=True)    # Solo al pulsar Intro
dpg.add_button(label="Eliminar", callback=eliminar, user_data=registro_id)
```

> Se puede definir el callback con menos argumentos (`def f(): ...` o `def f(sender): ...`): DPG pasa solo los que la función acepta.

### 7.5 Valores compartidos entre items

```python
with dpg.value_registry():
    dpg.add_float_value(default_value=0.5, tag="umbral")

dpg.add_slider_float(label="Umbral", source="umbral")       # Ambos muestran y modifican el mismo valor
dpg.add_input_float(label="Umbral (exacto)", source="umbral")
```

---

## 8. Disposición de los elementos

### 8.1 Grupos horizontales

```python
with dpg.group(horizontal=True, horizontal_spacing=10):
    dpg.add_button(label="Aceptar")
    dpg.add_button(label="Cancelar")
```

### 8.2 Ancho de los widgets

```python
dpg.add_input_text(width=200)        # Píxeles
dpg.add_input_text(width=-1)         # Todo el ancho disponible
dpg.add_input_text(width=-100)       # Todo el ancho menos 100 px (para dejar sitio a la etiqueta)
```

> Las etiquetas (`label`) se muestran a la **derecha** del widget. Para formularios con la etiqueta a la izquierda, use un texto y el widget en un grupo horizontal, o una tabla de dos columnas.

### 8.3 Formulario con tabla de maquetación

```python
with dpg.table(header_row=False, borders_innerH=False, borders_outerH=False,
               borders_innerV=False, borders_outerV=False):
    dpg.add_table_column(width_fixed=True, init_width_or_weight=110)
    dpg.add_table_column(width_stretch=True)
    for etiqueta, tag in [("Nombre", "f_nombre"), ("Email", "f_email"), ("Teléfono", "f_tel")]:
        with dpg.table_row():
            dpg.add_text(etiqueta)
            dpg.add_input_text(tag=tag, width=-1)
```

### 8.4 Posición y tamaño absolutos

```python
dpg.add_button(label="Fijo", pos=(20, 60))
dpg.set_item_pos("ventana", (100, 100))
dpg.set_item_width("ventana", 500)
dpg.get_item_rect_size("ventana")
```

### 8.5 Diseño adaptable al tamaño del viewport

```python
def ajustar():
    ancho = dpg.get_viewport_client_width()
    dpg.set_item_width("panel_izquierdo", int(ancho * 0.3))

dpg.set_viewport_resize_callback(ajustar)
```

---

## 9. Tablas

```python
productos = [("TEC-01", "Teclado", 59.90, 120), ("RAT-01", "Ratón", 24.50, 8), ("MON-27", "Monitor", 219.0, 35)]

with dpg.table(tag="tabla", header_row=True, resizable=True, sortable=True, callback=ordenar_tabla,
               policy=dpg.mvTable_SizingStretchProp, row_background=True, borders_innerH=True,
               borders_outerH=True, borders_innerV=True, borders_outerV=True,
               scrollY=True, freeze_rows=1, height=300):
    dpg.add_table_column(label="Código", width_fixed=True, init_width_or_weight=80)
    dpg.add_table_column(label="Producto")
    dpg.add_table_column(label="Precio")
    dpg.add_table_column(label="Stock", default_sort=True)

    for codigo, nombre, precio, stock in productos:
        with dpg.table_row():
            dpg.add_text(codigo)
            dpg.add_text(nombre)
            dpg.add_text(f"{precio:.2f}")
            dpg.add_text(str(stock), color=(231, 76, 60) if stock < 10 else (220, 220, 220))
```

### 9.1 Ordenar al pulsar la cabecera

La tabla no ordena sola: el callback recibe la columna y la dirección.

```python
def ordenar_tabla(sender, especificacion):
    if not especificacion:
        return
    columna_id, direccion = especificacion[0]            # direccion: 1 ascendente, -1 descendente
    columnas = dpg.get_item_children(sender, slot=0)
    indice = columnas.index(columna_id)
    filas = dpg.get_item_children(sender, slot=1)

    def clave(fila):
        celda = dpg.get_item_children(fila, slot=1)[indice]
        valor = dpg.get_value(celda)
        try:
            return (0, float(valor))
        except (TypeError, ValueError):
            return (1, str(valor).lower())

    dpg.reorder_items(sender, 1, sorted(filas, key=clave, reverse=direccion < 0))
```

### 9.2 Filas seleccionables

```python
with dpg.table_row():
    dpg.add_selectable(label="TEC-01", span_columns=True, callback=al_seleccionar_fila, user_data="TEC-01")
    dpg.add_text("Teclado")
```

### 9.3 Reconstruir la tabla con nuevos datos

```python
def rellenar(tabla: str, filas: list[tuple]) -> None:
    for fila in dpg.get_item_children(tabla, slot=1):
        dpg.delete_item(fila)
    for datos in filas:
        with dpg.table_row(parent=tabla):
            for valor in datos:
                dpg.add_text(str(valor))
```

### 9.4 Filtro de texto integrado

```python
dpg.add_input_text(label="Filtrar", user_data="tabla",
                   callback=lambda s, a, u: dpg.set_value(u, a))
with dpg.table(tag="tabla"):
    ...
    with dpg.table_row(filter_key="teclado mecánico"):     # Texto por el que se filtra la fila
        ...
```

---

## 10. Gráficos (plots)

Dear PyGui incluye **ImPlot**: gráficos interactivos (zoom con la rueda, desplazamiento arrastrando, menú contextual con el botón derecho) y muy rápidos.

### 10.1 Gráfico de líneas

```python
import math

x = [i / 10 for i in range(500)]
y = [math.sin(v) for v in x]

with dpg.plot(label="Señal", height=300, width=-1):
    dpg.add_plot_legend()
    dpg.add_plot_axis(dpg.mvXAxis, label="Tiempo (s)", tag="eje_x")
    with dpg.plot_axis(dpg.mvYAxis, label="Amplitud", tag="eje_y"):
        dpg.add_line_series(x, y, label="sen(x)", tag="serie_seno")
```

### 10.2 Actualizar una serie

```python
dpg.set_value("serie_seno", [nuevos_x, nuevos_y])
dpg.fit_axis_data("eje_x")
dpg.fit_axis_data("eje_y")
dpg.set_axis_limits("eje_y", -1.5, 1.5)            # Límites fijos
dpg.set_axis_limits_auto("eje_y")                 # Volver al modo automático
```

### 10.3 Tipos de series

| Función | Gráfico |
|---|---|
| `add_line_series` | Líneas |
| `add_scatter_series` | Dispersión |
| `add_bar_series` | Barras |
| `add_stair_series` | Escalones |
| `add_area_series` / `add_shade_series` | Áreas |
| `add_stem_series` | Tallos |
| `add_histogram_series` | Histograma |
| `add_heat_series` | Mapa de calor |
| `add_pie_series` | Sectores |
| `add_candle_series` | Velas (finanzas) |
| `add_error_series` | Barras de error |
| `add_text_point` | Texto en una posición |

### 10.4 Barras con etiquetas en el eje

```python
regiones = ["Norte", "Sur", "Este", "Oeste"]
ventas = [13800, 11200, 13900, 10100]

with dpg.plot(label="Ventas por región", height=280, width=-1):
    eje_x = dpg.add_plot_axis(dpg.mvXAxis, no_gridlines=True)
    dpg.set_axis_ticks(eje_x, tuple((r, i) for i, r in enumerate(regiones)))
    with dpg.plot_axis(dpg.mvYAxis, label="€"):
        dpg.add_bar_series(list(range(len(regiones))), ventas, weight=0.6)
```

### 10.5 Eje de tiempo

```python
import time
with dpg.plot(height=250, width=-1):
    dpg.add_plot_axis(dpg.mvXAxis, scale=dpg.mvPlotScale_Time)   # Timestamps Unix (antes: time=True)
    with dpg.plot_axis(dpg.mvYAxis):
        dpg.add_line_series([time.time() - 60 * i for i in range(60)][::-1], valores)
```

### 10.6 Elementos interactivos en el gráfico

```python
dpg.add_drag_line(label="Umbral", color=(255, 80, 80, 255), default_value=0.8, vertical=False,
                  callback=lambda s: print("Umbral:", dpg.get_value(s)))
dpg.add_drag_point(label="Punto", default_value=(1.0, 0.5))
```

### 10.7 Colores y estilo de una serie

```python
with dpg.theme(tag="tema_linea_roja"):
    with dpg.theme_component(dpg.mvLineSeries):
        dpg.add_theme_color(dpg.mvPlotCol_Line, (231, 76, 60), category=dpg.mvThemeCat_Plots)
        dpg.add_theme_style(dpg.mvPlotStyleVar_LineWeight, 2.0, category=dpg.mvThemeCat_Plots)
dpg.bind_item_theme("serie_seno", "tema_linea_roja")
```

---

## 11. Menús, diálogos y emergentes

### 11.1 Barra de menús

```python
with dpg.viewport_menu_bar():                     # Menú del viewport (o dpg.menu_bar() dentro de una ventana)
    with dpg.menu(label="Archivo"):
        dpg.add_menu_item(label="Abrir…", shortcut="Ctrl+O", callback=lambda: dpg.show_item("dialogo_abrir"))
        dpg.add_menu_item(label="Exportar CSV", callback=exportar)
        dpg.add_separator()
        dpg.add_menu_item(label="Salir", callback=dpg.stop_dearpygui)
    with dpg.menu(label="Ver"):
        dpg.add_menu_item(label="Modo oscuro", check=True, default_value=True, callback=cambiar_tema)
```

> El texto `shortcut` solo se muestra: el atajo se implementa con un manejador de teclado (sección 14).

### 11.2 Selector de archivos

```python
def al_elegir(sender, app_data):
    ruta = app_data["file_path_name"]
    print("Archivo:", ruta)

with dpg.file_dialog(directory_selector=False, show=False, callback=al_elegir, tag="dialogo_abrir",
                     width=700, height=400, default_path="."):
    dpg.add_file_extension(".csv", color=(46, 204, 113, 255))
    dpg.add_file_extension(".*")
```

> Para usar el diálogo nativo del sistema operativo, puede combinar DPG con `tkinter.filedialog` (creando una ventana Tk oculta) o librerías como `plyer`.

### 11.3 Ventana modal de confirmación

```python
def confirmar(mensaje: str, al_aceptar) -> None:
    tag = dpg.generate_uuid()

    def cerrar(aceptado: bool):
        dpg.delete_item(tag)
        if aceptado:
            al_aceptar()

    ancho, alto = 320, 120
    with dpg.window(label="Confirmar", modal=True, tag=tag, no_resize=True, width=ancho, height=alto,
                    pos=((dpg.get_viewport_client_width() - ancho) // 2,
                         (dpg.get_viewport_client_height() - alto) // 2)):
        dpg.add_text(mensaje, wrap=ancho - 20)
        dpg.add_spacer(height=8)
        with dpg.group(horizontal=True):
            dpg.add_button(label="Aceptar", width=90, callback=lambda: cerrar(True))
            dpg.add_button(label="Cancelar", width=90, callback=lambda: cerrar(False))
```

### 11.4 Emergentes y ayudas

```python
dpg.add_button(label="Opciones", tag="btn_opciones")
with dpg.popup("btn_opciones", mousebutton=dpg.mvMouseButton_Left):
    dpg.add_selectable(label="Duplicar")
    dpg.add_selectable(label="Eliminar")

dpg.add_input_float(label="IVA", tag="iva", default_value=21)
with dpg.tooltip("iva"):
    dpg.add_text("Porcentaje de IVA aplicado a los precios")
```

---

## 12. Temas y estilos

Los temas se definen con colores (`add_theme_color`) y estilos (`add_theme_style`) por tipo de componente.

### 12.1 Tema global

```python
with dpg.theme() as tema_global:
    with dpg.theme_component(dpg.mvAll):
        dpg.add_theme_color(dpg.mvThemeCol_WindowBg, (24, 26, 31), category=dpg.mvThemeCat_Core)
        dpg.add_theme_color(dpg.mvThemeCol_FrameBg, (40, 44, 52), category=dpg.mvThemeCat_Core)
        dpg.add_theme_color(dpg.mvThemeCol_Button, (37, 99, 235), category=dpg.mvThemeCat_Core)
        dpg.add_theme_color(dpg.mvThemeCol_ButtonHovered, (59, 130, 246), category=dpg.mvThemeCat_Core)
        dpg.add_theme_color(dpg.mvThemeCol_ButtonActive, (29, 78, 216), category=dpg.mvThemeCat_Core)
        dpg.add_theme_color(dpg.mvThemeCol_Header, (37, 99, 235, 120), category=dpg.mvThemeCat_Core)
        dpg.add_theme_style(dpg.mvStyleVar_FrameRounding, 5, category=dpg.mvThemeCat_Core)
        dpg.add_theme_style(dpg.mvStyleVar_WindowRounding, 8, category=dpg.mvThemeCat_Core)
        dpg.add_theme_style(dpg.mvStyleVar_FramePadding, 8, 6, category=dpg.mvThemeCat_Core)
        dpg.add_theme_style(dpg.mvStyleVar_ItemSpacing, 8, 8, category=dpg.mvThemeCat_Core)

dpg.bind_theme(tema_global)
```

### 12.2 Tema para un item concreto

```python
with dpg.theme(tag="boton_peligro"):
    with dpg.theme_component(dpg.mvButton):
        dpg.add_theme_color(dpg.mvThemeCol_Button, (192, 57, 43))
        dpg.add_theme_color(dpg.mvThemeCol_ButtonHovered, (231, 76, 60))

dpg.add_button(label="Eliminar", tag="btn_eliminar")
dpg.bind_item_theme("btn_eliminar", "boton_peligro")
```

### 12.3 Estilo de items desactivados

```python
with dpg.theme(tag="tema_desactivado"):
    with dpg.theme_component(dpg.mvButton, enabled_state=False):
        dpg.add_theme_color(dpg.mvThemeCol_Button, (80, 80, 80))
        dpg.add_theme_color(dpg.mvThemeCol_Text, (150, 150, 150))
```

### 12.4 Tema claro

Dear PyGui usa tema oscuro por defecto. Para un tema claro, defina los colores principales (`WindowBg`, `ChildBg`, `FrameBg`, `Text`, `Button`, `Header`, `TableHeaderBg`, `TableRowBgAlt`…) con tonos claros. Puede inspeccionar todos los colores disponibles con `dpg.show_style_editor()`.

---

## 13. Fuentes y caracteres en español

La fuente por defecto es pequeña y no incluye algunos símbolos (por ejemplo, **€**). Cargue una fuente TrueType que contenga esos caracteres:

```python
with dpg.font_registry():
    dpg.add_font("C:/Windows/Fonts/segoeui.ttf", 18, tag="fuente_normal")    # Ruta a una fuente .ttf
    fuente_titulo = dpg.add_font("C:/Windows/Fonts/segoeuib.ttf", 26)

dpg.bind_font("fuente_normal")                     # Fuente global
dpg.bind_item_font("titulo", fuente_titulo)        # Fuente de un item
```

> **Versiones:** desde Dear PyGui 2.x los caracteres se cargan automáticamente según se usan; `add_font_range_hint`, `add_font_range` y `add_font_chars` siguen existiendo pero no hacen nada (avisan con `DeprecationWarning`). En versiones 1.x sí son necesarios:
>
> ```python
> with dpg.font("segoeui.ttf", 18):
>     dpg.add_font_range_hint(dpg.mvFontRangeHint_Default)
>     dpg.add_font_range(0x00A0, 0x00FF)    # á é í ó ú ñ ¿ ¡ ü
>     dpg.add_font_chars([0x20AC])          # €
> ```

Rutas de fuentes habituales:

| Sistema | Ejemplo |
|---|---|
| Windows | `C:/Windows/Fonts/segoeui.ttf`, `arial.ttf` |
| macOS | `/System/Library/Fonts/Supplemental/Arial.ttf` |
| Linux | `/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf` |

> Para distribuir la aplicación, incluya el archivo `.ttf` junto al programa (por ejemplo, fuentes libres como Roboto, Inter o DejaVu).

Escalado global (útil en pantallas de alta resolución):

```python
dpg.set_global_font_scale(1.25)
```

---

## 14. Teclado, ratón y eventos

### 14.1 Manejadores globales

```python
def al_pulsar_tecla(sender, tecla):
    ctrl = dpg.is_key_down(dpg.mvKey_LControl) or dpg.is_key_down(dpg.mvKey_RControl)
    if ctrl and tecla == dpg.mvKey_S:
        guardar()
    elif tecla == dpg.mvKey_Escape:
        dpg.hide_item("panel_detalle")

with dpg.handler_registry():
    dpg.add_key_press_handler(callback=al_pulsar_tecla)
    dpg.add_mouse_click_handler(button=dpg.mvMouseButton_Right, callback=menu_contextual)
    dpg.add_mouse_wheel_handler(callback=lambda s, delta: print("Rueda:", delta))
```

### 14.2 Manejadores de un item

```python
with dpg.item_handler_registry(tag="manejadores_tabla"):
    dpg.add_item_clicked_handler(callback=lambda s, a: print("Clic en", a[1]))
    dpg.add_item_hover_handler(callback=lambda s, a: dpg.set_value("estado", "Sobre la tabla"))

dpg.bind_item_handler_registry("tabla", "manejadores_tabla")
```

### 14.3 Estado del ratón

```python
dpg.get_mouse_pos()                       # Posición relativa a la ventana
dpg.get_plot_mouse_pos()                  # Coordenadas dentro del gráfico
dpg.is_item_hovered("boton")
dpg.is_mouse_button_down(dpg.mvMouseButton_Left)
```

### 14.4 Arrastrar y soltar

```python
dpg.add_button(label="Arrástrame", tag="origen", payload_type="producto", drag_callback=None)
with dpg.drag_payload(parent="origen", drag_data="TEC-01", payload_type="producto"):
    dpg.add_text("TEC-01")
dpg.add_text("Suelte aquí", drop_callback=lambda s, a: print("Recibido:", a), payload_type="producto")
```

---

## 15. Datos en tiempo real y concurrencia

### 15.1 Actualizar en cada fotograma

```python
import time, math, collections

tiempos = collections.deque(maxlen=500)
valores = collections.deque(maxlen=500)

while dpg.is_dearpygui_running():
    t = time.time()
    tiempos.append(t)
    valores.append(math.sin(t * 2))
    dpg.set_value("serie", [list(tiempos), list(valores)])
    dpg.fit_axis_data("eje_x")
    dpg.render_dearpygui_frame()
```

### 15.2 Adquisición en un hilo + cola

Las lecturas lentas (puertos serie, red, bases de datos) deben ejecutarse en otro hilo. Los datos se pasan al bucle principal mediante una cola:

```python
import queue, threading, random

cola = queue.Queue()
parar = threading.Event()

def adquirir():
    while not parar.is_set():
        cola.put((time.time(), random.gauss(20, 0.5)))      # Simula un sensor
        time.sleep(0.05)

threading.Thread(target=adquirir, daemon=True).start()

while dpg.is_dearpygui_running():
    while not cola.empty():
        t, v = cola.get_nowait()
        tiempos.append(t)
        valores.append(v)
    dpg.set_value("serie", [list(tiempos), list(valores)])
    dpg.render_dearpygui_frame()

parar.set()
```

> Dear PyGui permite llamar a muchas funciones (`set_value`, `configure_item`) desde otros hilos, pero centralizar las actualizaciones en el bucle principal evita problemas y mantiene el código predecible.

### 15.3 Callbacks lentos

Si un callback tarda, la interfaz se congela. Lance el trabajo en un hilo:

```python
def exportar_callback():
    dpg.configure_item("btn_exportar", enabled=False, label="Exportando…")

    def trabajo():
        exportar_datos_grandes()
        dpg.configure_item("btn_exportar", enabled=True, label="Exportar")

    threading.Thread(target=trabajo, daemon=True).start()
```

### 15.4 Callbacks en el bucle principal (manual)

```python
dpg.configure_app(manual_callback_management=True)
...
while dpg.is_dearpygui_running():
    dpg.run_callbacks(dpg.get_callback_queue())
    dpg.render_dearpygui_frame()
```

### 15.5 Ejecutar código tras un número de fotogramas

```python
dpg.set_frame_callback(2, lambda: dpg.set_item_width("panel", dpg.get_viewport_client_width() // 3))
```

> Útil para operaciones que requieren que la interfaz ya esté dibujada (tamaños reales de los items).

---

## 16. Herramientas de desarrollo

Dear PyGui incluye ventanas de ayuda muy útiles durante el desarrollo:

```python
dpg.show_documentation()      # Documentación interactiva
dpg.show_style_editor()       # Editor de estilo y colores en vivo
dpg.show_item_registry()      # Árbol de todos los items y sus tags
dpg.show_metrics()            # Rendimiento (FPS, tiempos)
dpg.show_font_manager()       # Fuentes cargadas
dpg.show_debug()              # Depuración
dpg.show_about()
```

---

## 17. Distribución

```bash
pip install pyinstaller
pyinstaller --onefile --windowed --name PanelSensores --add-data "fuentes;fuentes" main.py
```

> En Linux y macOS, el separador de `--add-data` es `:` en lugar de `;`. Incluya las fuentes `.ttf` y localice los recursos con `sys._MEIPASS` (ver manual de Tkinter, sección 15).

---

## 18. Ejemplo práctico completo

**Monitor de sensores en tiempo real**: simula varios sensores de temperatura y humedad en un hilo de adquisición; muestra gráficos de líneas que se actualizan en vivo (con ventana temporal configurable y línea de umbral arrastrable), tarjetas con el último valor, tabla de alarmas ordenable, controles de pausa/reanudar, frecuencia y selección de sensores, menú, exportación a CSV, atajos de teclado, tema personalizado y soporte de caracteres en español.

```python
"""
Monitor de sensores en tiempo real con Dear PyGui.
Requisitos: pip install dearpygui
"""
import collections
import csv
import math
import os
import queue
import random
import threading
import time
from datetime import datetime
from pathlib import Path

import dearpygui.dearpygui as dpg

SENSORES = {"Sala A": (21.0, 45.0), "Sala B": (23.5, 50.0), "Almacén": (17.0, 60.0), "CPD": (19.5, 35.0)}
COLORES = [(59, 130, 246), (16, 185, 129), (245, 158, 11), (239, 68, 68)]
MAX_PUNTOS = 3000
FUENTES_CANDIDATAS = ["C:/Windows/Fonts/segoeui.ttf", "/System/Library/Fonts/Supplemental/Arial.ttf",
                      "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf"]


# ------------------------------------------------------------------ Estado
class Estado:
    def __init__(self):
        self.cola: "queue.Queue[tuple]" = queue.Queue()
        self.parar = threading.Event()
        self.pausado = threading.Event()
        self.intervalo = 0.1
        self.ventana_seg = 60
        self.umbral = 25.0
        self.t: dict[str, collections.deque] = {s: collections.deque(maxlen=MAX_PUNTOS) for s in SENSORES}
        self.temp: dict[str, collections.deque] = {s: collections.deque(maxlen=MAX_PUNTOS) for s in SENSORES}
        self.hum: dict[str, collections.deque] = {s: collections.deque(maxlen=MAX_PUNTOS) for s in SENSORES}
        self.alarmas: list[tuple] = []
        self.en_alarma: set[str] = set()
        self.lecturas = 0


estado = Estado()


# ------------------------------------------------------------------ Adquisición (hilo)
def adquirir() -> None:
    inicio = time.time()
    while not estado.parar.is_set():
        if not estado.pausado.is_set():
            ahora = time.time()
            for i, (sensor, (t_base, h_base)) in enumerate(SENSORES.items()):
                deriva = 3.5 * math.sin((ahora - inicio) / (20 + 7 * i)) + (4 if sensor == "CPD" and
                                                                             int(ahora - inicio) % 45 > 35 else 0)
                temperatura = t_base + deriva + random.gauss(0, 0.15)
                humedad = h_base - deriva * 1.5 + random.gauss(0, 0.4)
                estado.cola.put((sensor, ahora, temperatura, humedad))
        time.sleep(estado.intervalo)


# ------------------------------------------------------------------ Actualización de la interfaz (hilo principal)
def procesar_cola() -> None:
    nuevas = 0
    while True:
        try:
            sensor, t, temp, hum = estado.cola.get_nowait()
        except queue.Empty:
            break
        estado.t[sensor].append(t)
        estado.temp[sensor].append(temp)
        estado.hum[sensor].append(hum)
        estado.lecturas += 1
        nuevas += 1
        comprobar_alarma(sensor, t, temp)
    if nuevas:
        actualizar_vista()


def comprobar_alarma(sensor: str, t: float, temp: float) -> None:
    if temp > estado.umbral and sensor not in estado.en_alarma:
        estado.en_alarma.add(sensor)
        estado.alarmas.append((datetime.fromtimestamp(t).strftime("%H:%M:%S"), sensor, round(temp, 2), "Supera umbral"))
        anadir_fila_alarma(estado.alarmas[-1])
    elif temp <= estado.umbral - 0.5 and sensor in estado.en_alarma:       # Histéresis
        estado.en_alarma.discard(sensor)
        estado.alarmas.append((datetime.fromtimestamp(t).strftime("%H:%M:%S"), sensor, round(temp, 2), "Normalizado"))
        anadir_fila_alarma(estado.alarmas[-1])


def actualizar_vista() -> None:
    ahora = time.time()
    desde = ahora - estado.ventana_seg
    for i, sensor in enumerate(SENSORES):
        if not estado.t[sensor]:
            continue
        visible = dpg.get_value(f"chk_{i}")
        dpg.configure_item(f"serie_t_{i}", show=visible)
        dpg.configure_item(f"serie_h_{i}", show=visible)
        tiempos = list(estado.t[sensor])
        inicio = next((k for k, v in enumerate(tiempos) if v >= desde), 0)
        dpg.set_value(f"serie_t_{i}", [tiempos[inicio:], list(estado.temp[sensor])[inicio:]])
        dpg.set_value(f"serie_h_{i}", [tiempos[inicio:], list(estado.hum[sensor])[inicio:]])

        ultima = estado.temp[sensor][-1]
        dpg.set_value(f"kpi_valor_{i}", f"{ultima:.1f} °C")
        dpg.set_value(f"kpi_hum_{i}", f"{estado.hum[sensor][-1]:.0f} % HR")
        dpg.configure_item(f"kpi_valor_{i}", color=(239, 68, 68) if sensor in estado.en_alarma else (235, 235, 235))

    for eje in ("eje_x_t", "eje_x_h"):
        dpg.set_axis_limits(eje, desde, ahora)
    dpg.set_value("estado_lecturas", f"Lecturas: {estado.lecturas:,}".replace(",", "."))
    dpg.set_value("estado_alarmas", f"Sensores en alarma: {len(estado.en_alarma)}")


def anadir_fila_alarma(fila: tuple) -> None:
    hora, sensor, temp, tipo = fila
    color = (239, 68, 68) if tipo == "Supera umbral" else (16, 185, 129)
    with dpg.table_row(parent="tabla_alarmas", before=(dpg.get_item_children("tabla_alarmas", 1) or [0])[0]):
        dpg.add_text(hora)
        dpg.add_text(sensor)
        dpg.add_text(f"{temp:.2f}")
        dpg.add_text(tipo, color=color)


# ------------------------------------------------------------------ Callbacks
def alternar_pausa() -> None:
    if estado.pausado.is_set():
        estado.pausado.clear()
        dpg.configure_item("btn_pausa", label="Pausar")
        dpg.set_value("estado_modo", "● Adquiriendo")
    else:
        estado.pausado.set()
        dpg.configure_item("btn_pausa", label="Reanudar")
        dpg.set_value("estado_modo", "‖ En pausa")


def cambiar_umbral(sender) -> None:
    estado.umbral = round(dpg.get_value(sender), 1)
    dpg.set_value("lbl_umbral", f"Umbral: {estado.umbral:.1f} °C")
    dpg.set_value("linea_umbral", estado.umbral)
    dpg.set_value("slider_umbral", estado.umbral)


def limpiar() -> None:
    for s in SENSORES:
        for d in (estado.t, estado.temp, estado.hum):
            d[s].clear()
    estado.alarmas.clear()
    estado.en_alarma.clear()
    estado.lecturas = 0
    for fila in dpg.get_item_children("tabla_alarmas", 1):
        dpg.delete_item(fila)


def exportar(ruta: str | None = None) -> str:
    ruta = ruta or f"lecturas_{datetime.now():%Y%m%d_%H%M%S}.csv"
    with open(ruta, "w", encoding="utf-8-sig", newline="") as f:
        w = csv.writer(f, delimiter=";")
        w.writerow(["sensor", "fecha_hora", "temperatura", "humedad"])
        for s in SENSORES:
            for t, temp, hum in zip(list(estado.t[s]), list(estado.temp[s]), list(estado.hum[s])):
                w.writerow([s, datetime.fromtimestamp(t).isoformat(sep=" ", timespec="milliseconds"),
                            f"{temp:.2f}".replace(".", ","), f"{hum:.1f}".replace(".", ",")])
    dpg.set_value("estado_mensaje", f"Exportado: {Path(ruta).name}")
    return ruta


def ordenar_alarmas(sender, especificacion) -> None:
    if not especificacion:
        return
    columna, direccion = especificacion[0]
    indice = dpg.get_item_children(sender, 0).index(columna)
    filas = dpg.get_item_children(sender, 1)
    clave = lambda fila: str(dpg.get_value(dpg.get_item_children(fila, 1)[indice]))
    dpg.reorder_items(sender, 1, sorted(filas, key=clave, reverse=direccion < 0))


def teclas(sender, tecla) -> None:
    if tecla == dpg.mvKey_Spacebar:
        alternar_pausa()
    elif tecla == dpg.mvKey_E and (dpg.is_key_down(dpg.mvKey_LControl) or dpg.is_key_down(dpg.mvKey_RControl)):
        exportar()


# ------------------------------------------------------------------ Construcción de la interfaz
def configurar_fuente_y_tema() -> None:
    fuente = next((f for f in FUENTES_CANDIDATAS if os.path.exists(f)), None)
    if fuente:
        with dpg.font_registry():
            dpg.add_font(fuente, 17, tag="fuente")       # DPG 2.x carga los caracteres automáticamente
            dpg.add_font(fuente, 30, tag="fuente_grande")
        dpg.bind_font("fuente")

    with dpg.theme() as tema:
        with dpg.theme_component(dpg.mvAll):
            for col, valor in [(dpg.mvThemeCol_WindowBg, (17, 24, 39)), (dpg.mvThemeCol_ChildBg, (31, 41, 55)),
                               (dpg.mvThemeCol_FrameBg, (55, 65, 81)), (dpg.mvThemeCol_Button, (37, 99, 235)),
                               (dpg.mvThemeCol_ButtonHovered, (59, 130, 246)),
                               (dpg.mvThemeCol_ButtonActive, (29, 78, 216)),
                               (dpg.mvThemeCol_Header, (37, 99, 235, 110)),
                               (dpg.mvThemeCol_CheckMark, (96, 165, 250)),
                               (dpg.mvThemeCol_SliderGrab, (96, 165, 250))]:
                dpg.add_theme_color(col, valor, category=dpg.mvThemeCat_Core)
            for var, *valor in [(dpg.mvStyleVar_FrameRounding, 5), (dpg.mvStyleVar_ChildRounding, 8),
                                (dpg.mvStyleVar_FramePadding, 8, 5), (dpg.mvStyleVar_ItemSpacing, 8, 8)]:
                dpg.add_theme_style(var, *valor, category=dpg.mvThemeCat_Core)
    dpg.bind_theme(tema)

    for i, color in enumerate(COLORES):
        with dpg.theme(tag=f"tema_serie_{i}"):
            with dpg.theme_component(dpg.mvLineSeries):
                dpg.add_theme_color(dpg.mvPlotCol_Line, color, category=dpg.mvThemeCat_Plots)
                dpg.add_theme_style(dpg.mvPlotStyleVar_LineWeight, 1.8, category=dpg.mvThemeCat_Plots)


def construir() -> None:
    with dpg.viewport_menu_bar():
        with dpg.menu(label="Archivo"):
            dpg.add_menu_item(label="Exportar CSV", shortcut="Ctrl+E", callback=lambda: exportar())
            dpg.add_menu_item(label="Limpiar datos", callback=limpiar)
            dpg.add_separator()
            dpg.add_menu_item(label="Salir", callback=dpg.stop_dearpygui)
        with dpg.menu(label="Herramientas"):
            dpg.add_menu_item(label="Editor de estilo", callback=dpg.show_style_editor)
            dpg.add_menu_item(label="Métricas", callback=dpg.show_metrics)

    with dpg.window(tag="principal"):
        dpg.add_spacer(height=18)
        # --- Tarjetas
        with dpg.group(horizontal=True):
            for i, sensor in enumerate(SENSORES):
                with dpg.child_window(width=210, height=96, border=False):
                    dpg.add_text(sensor, color=COLORES[i])
                    dpg.add_text("—", tag=f"kpi_valor_{i}")
                    if dpg.does_item_exist("fuente_grande"):
                        dpg.bind_item_font(f"kpi_valor_{i}", "fuente_grande")
                    dpg.add_text("", tag=f"kpi_hum_{i}", color=(156, 163, 175))

        with dpg.group(horizontal=True):
            # --- Panel de control
            with dpg.child_window(width=260, tag="panel_control"):
                dpg.add_text("Control")
                dpg.add_separator()
                dpg.add_button(label="Pausar", tag="btn_pausa", width=-1, callback=alternar_pausa)
                dpg.add_button(label="Exportar CSV", width=-1, callback=lambda: exportar())
                dpg.add_spacer(height=6)
                dpg.add_text("Ventana temporal (s)")
                dpg.add_slider_int(default_value=60, min_value=10, max_value=300, width=-1,
                                   callback=lambda s, a: setattr(estado, "ventana_seg", a))
                dpg.add_text("Frecuencia de muestreo (ms)")
                dpg.add_combo(["50", "100", "250", "500", "1000"], default_value="100", width=-1,
                              callback=lambda s, a: setattr(estado, "intervalo", int(a) / 1000))
                dpg.add_text("Umbral: 25.0 °C", tag="lbl_umbral")
                dpg.add_slider_float(tag="slider_umbral", default_value=estado.umbral, min_value=15, max_value=35,
                                     format="%.1f °C", width=-1, callback=cambiar_umbral)
                dpg.add_spacer(height=6)
                dpg.add_text("Sensores visibles")
                for i, sensor in enumerate(SENSORES):
                    dpg.add_checkbox(label=sensor, default_value=True, tag=f"chk_{i}")
                dpg.add_spacer(height=10)
                dpg.add_text("Atajos: Espacio = pausa · Ctrl+E = exportar", wrap=230, color=(156, 163, 175))

            # --- Gráficos y alarmas
            with dpg.child_window(border=False):
                with dpg.plot(label="Temperatura", height=260, width=-1):
                    dpg.add_plot_legend(location=dpg.mvPlot_Location_NorthWest)
                    dpg.add_plot_axis(dpg.mvXAxis, scale=dpg.mvPlotScale_Time, tag="eje_x_t")
                    with dpg.plot_axis(dpg.mvYAxis, label="°C", tag="eje_y_t"):
                        for i, sensor in enumerate(SENSORES):
                            dpg.add_line_series([], [], label=sensor, tag=f"serie_t_{i}")
                            dpg.bind_item_theme(f"serie_t_{i}", f"tema_serie_{i}")
                    dpg.set_axis_limits("eje_y_t", 10, 32)
                    dpg.add_drag_line(tag="linea_umbral", label="Umbral", vertical=False,
                                      default_value=estado.umbral, color=(239, 68, 68, 255), callback=cambiar_umbral)

                with dpg.plot(label="Humedad relativa", height=200, width=-1):
                    dpg.add_plot_axis(dpg.mvXAxis, scale=dpg.mvPlotScale_Time, tag="eje_x_h")
                    with dpg.plot_axis(dpg.mvYAxis, label="% HR", tag="eje_y_h"):
                        for i, sensor in enumerate(SENSORES):
                            dpg.add_line_series([], [], label=sensor, tag=f"serie_h_{i}")
                            dpg.bind_item_theme(f"serie_h_{i}", f"tema_serie_{i}")
                    dpg.set_axis_limits("eje_y_h", 20, 80)

                dpg.add_text("Registro de alarmas")
                with dpg.table(tag="tabla_alarmas", header_row=True, sortable=True, callback=ordenar_alarmas,
                               row_background=True, borders_innerH=True, borders_outerH=True,
                               scrollY=True, height=-30, policy=dpg.mvTable_SizingStretchProp):
                    dpg.add_table_column(label="Hora")
                    dpg.add_table_column(label="Sensor")
                    dpg.add_table_column(label="Temp. (°C)")
                    dpg.add_table_column(label="Evento")

                with dpg.group(horizontal=True, horizontal_spacing=30):
                    dpg.add_text("● Adquiriendo", tag="estado_modo", color=(16, 185, 129))
                    dpg.add_text("Lecturas: 0", tag="estado_lecturas")
                    dpg.add_text("Sensores en alarma: 0", tag="estado_alarmas")
                    dpg.add_text("", tag="estado_mensaje", color=(156, 163, 175))

    with dpg.handler_registry():
        dpg.add_key_press_handler(callback=teclas)


def main() -> None:
    dpg.create_context()
    configurar_fuente_y_tema()
    construir()
    dpg.create_viewport(title="Monitor de sensores", width=1280, height=860, min_width=1000, min_height=700)
    dpg.setup_dearpygui()
    dpg.show_viewport()
    dpg.set_primary_window("principal", True)

    hilo = threading.Thread(target=adquirir, daemon=True)
    hilo.start()
    try:
        while dpg.is_dearpygui_running():
            procesar_cola()
            dpg.render_dearpygui_frame()
    finally:
        estado.parar.set()
        hilo.join(timeout=1)
        dpg.destroy_context()


if __name__ == "__main__":
    main()
```

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `SystemError: ... Item not found: <tag>` | Se usa un tag que no existe (o aún no se ha creado). | Revise los tags; compruebe con `does_item_exist`. |
| `Alias already exists` | Dos items con el mismo `tag`. | Use tags únicos o `generate_uuid()`. |
| `Parent could not be deduced` | Se creó un item fuera de cualquier contenedor. | Créelo dentro de un `with dpg.window()` o indique `parent=`. |
| La ventana no aparece | Falta `create_viewport` / `setup_dearpygui` / `show_viewport` o el orden es incorrecto. | Siga el orden de la sección 4. |
| Error al crear el viewport en máquinas virtuales o servidores | Sin aceleración gráfica (OpenGL 3.3 / DirectX 11). | Actualice los controladores o use un equipo con GPU compatible. |
| Aparecen `?` en lugar de tildes, ñ o € | La fuente no incluye esos caracteres (o, en DPG 1.x, faltan los rangos). | Cargue una fuente TTF que los contenga; en 1.x añada los rangos (sección 13). |
| La interfaz se congela | Callback o código del bucle demasiado lento. | Mueva el trabajo a un hilo y use una cola. |
| El gráfico no se actualiza | Datos con formato incorrecto. | `set_value(serie, [lista_x, lista_y])` con listas del mismo tamaño. |
| La tabla no se ordena al pulsar la cabecera | DPG no ordena automáticamente. | Implemente el callback de ordenación (sección 9.1). |
| `DeprecationWarning: time keyword deprecated` | Uso de `time=True` en un eje (API 1.x). | Use `scale=dpg.mvPlotScale_Time`. |
| Un tema no se aplica | Componente o categoría incorrectos. | Use el `theme_component` del tipo de item y `mvThemeCat_Plots` para gráficos. |
| `set_primary_window` no ocupa toda la ventana | Se llamó antes de `setup_dearpygui`. | Llámelo después de `show_viewport()`. |
| Los cambios de `configure_item` no surten efecto en otro hilo | Condiciones de carrera. | Actualice la interfaz desde el bucle principal. |

---

## 20. Buenas prácticas

- Siga siempre la secuencia: contexto → interfaz → viewport → setup → show → bucle → destruir contexto.
- Use **tags de texto descriptivos** y únicos para los items que necesite consultar o modificar.
- Organice la construcción de la interfaz en funciones (cabecera, panel de control, gráficos…).
- Mantenga el bucle principal ligero: la adquisición y el trabajo pesado, en hilos; la actualización de la interfaz, en el hilo principal.
- Limite el número de puntos de los gráficos (`deque(maxlen=…)`) y actualice solo lo visible.
- Cargue desde el principio una fuente TTF que incluya tildes, ñ, € y °.
- Defina un tema global coherente y temas específicos solo para excepciones (botones de peligro, series).
- Use las herramientas integradas (`show_item_registry`, `show_style_editor`, `show_metrics`) durante el desarrollo.
- Para aplicaciones de gestión con formularios complejos y aspecto nativo, valore PySide6.

---

## 21. Referencia rápida

```python
dpg.create_context(); ...; dpg.create_viewport(title, width, height); dpg.setup_dearpygui()
dpg.show_viewport(); dpg.set_primary_window("tag", True); dpg.start_dearpygui(); dpg.destroy_context()
while dpg.is_dearpygui_running(): ...; dpg.render_dearpygui_frame()

with dpg.window(label, tag, width, height, modal, show): ...
with dpg.group(horizontal=True) / child_window / tab_bar / tab / collapsing_header / table / popup / tooltip
dpg.add_text / add_button / add_input_text / add_input_int / add_input_float / add_slider_int / add_slider_float
dpg.add_checkbox / add_radio_button / add_combo / add_listbox / add_progress_bar / add_color_edit / add_date_picker

dpg.get_value(tag) / set_value(tag, v) / configure_item(tag, **opc) / delete_item(tag, children_only=True)
dpg.show_item / hide_item / enable_item / disable_item / does_item_exist / get_item_children(tag, slot)
callback(sender, app_data, user_data)

with dpg.table(sortable, callback, resizable, scrollY, policy): add_table_column(label); with table_row(): ...
with dpg.plot(label, height, width=-1): add_plot_legend(); add_plot_axis(dpg.mvXAxis, scale=dpg.mvPlotScale_Time)
    with plot_axis(dpg.mvYAxis): add_line_series(x, y, tag) / add_bar_series / add_scatter_series
dpg.set_value(serie, [x, y]) / fit_axis_data(eje) / set_axis_limits(eje, a, b) / add_drag_line

with dpg.theme(): with theme_component(dpg.mvAll): add_theme_color(...); add_theme_style(...)
dpg.bind_theme(t) / bind_item_theme(tag, t)
with dpg.font_registry(): f = dpg.add_font(ruta, tam)  →  dpg.bind_font(f)
with dpg.handler_registry(): add_key_press_handler(callback) · dpg.is_key_down(dpg.mvKey_LControl)
dpg.viewport_menu_bar() / menu / add_menu_item · dpg.file_dialog + add_file_extension
dpg.show_item_registry() / show_style_editor() / show_metrics() / show_documentation()
```

---

## 22. Ejercicios propuestos

1. Cree una calculadora de préstamos con sliders (importe, interés, plazo) que actualice en vivo la cuota y un gráfico de amortización.
2. Cargue un CSV en una tabla ordenable con filtro de texto y muestre un histograma de una columna numérica elegida en un combo.
3. Lea datos de un puerto serie (o simúlelos) en un hilo y represéntelos en tiempo real con una ventana temporal configurable.
4. Diseñe un tema claro completo y añada al ejemplo una opción de menú para alternar entre tema claro y oscuro.
5. Añada al monitor un gráfico de barras con la temperatura media de cada sensor en el último minuto.

---

## 23. Referencias

- Documentación oficial: https://dearpygui.readthedocs.io/
- Repositorio y ejemplos: https://github.com/hoffstadt/DearPyGui
- Demo integrada: `dearpygui.demo.show_demo()`
