# Manual de uso: CustomTkinter

> **Categoría:** Interfaces gráficas (GUI) · **Utilidad:** Interfaces modernas basadas en Tkinter
> **Tipo:** Librería externa · Requiere conocimientos básicos de **Tkinter**

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Apariencia, temas y escalado](#4-apariencia-temas-y-escalado)
5. [La ventana principal](#5-la-ventana-principal)
6. [Widgets](#6-widgets)
7. [Contenedores](#7-contenedores)
8. [Colores, fuentes e imágenes](#8-colores-fuentes-e-imágenes)
9. [Diseño de la interfaz](#9-diseño-de-la-interfaz)
10. [Ventanas secundarias y diálogos](#10-ventanas-secundarias-y-diálogos)
11. [Combinar con Tkinter y ttk](#11-combinar-con-tkinter-y-ttk)
12. [Componentes reutilizables](#12-componentes-reutilizables)
13. [Tareas largas](#13-tareas-largas)
14. [Temas personalizados](#14-temas-personalizados)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**CustomTkinter** es una librería construida sobre Tkinter que ofrece widgets con **diseño moderno**: esquinas redondeadas, modo claro/oscuro automático, temas de color y escalado correcto en pantallas de alta resolución.

**Ventajas:**
- Aspecto actual sin necesidad de CSS ni frameworks pesados.
- Modo claro, oscuro o el del sistema, con cambio en tiempo real.
- Escalado automático en pantallas HiDPI (125 %, 150 %, 4K).
- Misma lógica que Tkinter: `pack`, `grid`, `place`, variables, eventos y `mainloop`.
- Se puede combinar con widgets de Tkinter y `ttk` cuando haga falta (por ejemplo, tablas).

**Limitaciones:**
- No incluye tabla (`Treeview`) ni menús propios: se usan los de Tkinter.
- Rendimiento algo menor que Tkinter puro con cientos de widgets.

---

## 2. Instalación e importación

```bash
pip install customtkinter
pip install pillow              # Para imágenes (CTkImage)
```

```python
import customtkinter as ctk
```

Ver una demostración de todos los widgets:

```python
import customtkinter
customtkinter.run_showroom()     # Disponible en versiones recientes
```

---

## 3. Conceptos clave

| Tkinter | CustomTkinter |
|---|---|
| `tk.Tk()` | `ctk.CTk()` |
| `tk.Toplevel` | `ctk.CTkToplevel` |
| `ttk.Frame` | `ctk.CTkFrame` |
| `ttk.Label` | `ctk.CTkLabel` |
| `ttk.Button` | `ctk.CTkButton` |
| `ttk.Entry` | `ctk.CTkEntry` |
| `tk.Text` | `ctk.CTkTextbox` |
| `ttk.Combobox` | `ctk.CTkComboBox` / `ctk.CTkOptionMenu` |
| `ttk.Checkbutton` | `ctk.CTkCheckBox` / `ctk.CTkSwitch` |
| `ttk.Radiobutton` | `ctk.CTkRadioButton` |
| `ttk.Scale` | `ctk.CTkSlider` |
| `ttk.Progressbar` | `ctk.CTkProgressBar` |
| `ttk.Notebook` | `ctk.CTkTabview` |
| `ttk.Scrollbar` | `ctk.CTkScrollbar` |
| `simpledialog.askstring` | `ctk.CTkInputDialog` |

**Diferencias importantes con Tkinter:**
- Los tamaños (`width`, `height`) se indican en **píxeles**, no en caracteres.
- Los colores se definen con `fg_color`, `text_color`, `hover_color`, `border_color`…
- Cada color puede ser una **tupla `(claro, oscuro)`** que cambia según el modo de apariencia.
- Las imágenes usan `CTkImage` (con versión clara y oscura opcional).
- Las opciones se cambian con `configure(...)` y se leen con `cget(...)`.

---

## 4. Apariencia, temas y escalado

Configúrelos **antes** de crear la ventana principal.

### 4.1 Modo de apariencia

```python
ctk.set_appearance_mode("system")     # "light", "dark" o "system" (sigue al sistema operativo)
ctk.get_appearance_mode()             # "Light" o "Dark"
```

Se puede cambiar en tiempo real:

```python
ctk.CTkSwitch(app, text="Modo oscuro",
              command=lambda: ctk.set_appearance_mode("dark" if switch.get() else "light"))
```

### 4.2 Tema de color

```python
ctk.set_default_color_theme("blue")         # "blue" (por defecto), "green", "dark-blue"
ctk.set_default_color_theme("mi_tema.json") # Tema propio (sección 14)
```

### 4.3 Escalado

```python
ctk.set_widget_scaling(1.1)        # Tamaño de los widgets (110 %)
ctk.set_window_scaling(1.0)        # Tamaño de las ventanas
ctk.deactivate_automatic_dpi_awareness()   # Desactivar el escalado automático (raro)
```

Selector de escala en la aplicación:

```python
ctk.CTkOptionMenu(app, values=["90%", "100%", "110%", "125%"],
                  command=lambda v: ctk.set_widget_scaling(int(v[:-1]) / 100))
```

---

## 5. La ventana principal

```python
import customtkinter as ctk

ctk.set_appearance_mode("system")
ctk.set_default_color_theme("blue")

app = ctk.CTk()
app.title("Mi aplicación")
app.geometry("900x600")
app.minsize(700, 450)
app.mainloop()
```

Con clase (recomendado):

```python
class App(ctk.CTk):
    def __init__(self):
        super().__init__()
        self.title("Mi aplicación")
        self.geometry("900x600")
        self.grid_columnconfigure(0, weight=1)
        self.grid_rowconfigure(0, weight=1)
        ctk.CTkLabel(self, text="Hola", font=ctk.CTkFont(size=24, weight="bold")).grid(row=0, column=0)


if __name__ == "__main__":
    App().mainloop()
```

Las opciones de ventana de Tkinter siguen disponibles: `resizable`, `attributes`, `protocol("WM_DELETE_WINDOW", f)`, `after`, `bind`, `iconbitmap`…

---

## 6. Widgets

### 6.1 Etiqueta

```python
ctk.CTkLabel(app, text="Clientes", font=ctk.CTkFont(size=22, weight="bold"))
ctk.CTkLabel(app, text="Texto secundario", text_color=("gray40", "gray60"))
ctk.CTkLabel(app, text="Ajustado", wraplength=300, justify="left", anchor="w")
ctk.CTkLabel(app, text="", image=icono)                     # Solo imagen
ctk.CTkLabel(app, text=" Usuarios", image=icono, compound="left")
```

### 6.2 Botón

```python
ctk.CTkButton(app, text="Guardar", command=guardar)
ctk.CTkButton(app, text="Eliminar", fg_color="#C0392B", hover_color="#922B21", command=eliminar)
ctk.CTkButton(app, text="Cancelar", fg_color="transparent", border_width=2,
              text_color=("gray10", "gray90"), command=cancelar)          # Botón secundario
ctk.CTkButton(app, text="", image=icono, width=36, height=36, command=f) # Botón de icono
ctk.CTkButton(app, text="Exportar", corner_radius=20, height=40)

boton.configure(state="disabled")         # Desactivar
boton.configure(text="Guardando…")
```

### 6.3 Campo de texto

```python
entrada = ctk.CTkEntry(app, placeholder_text="Buscar cliente…", width=250)
clave = ctk.CTkEntry(app, placeholder_text="Contraseña", show="•")

entrada.get()
entrada.delete(0, "end")
entrada.insert(0, "Texto")
entrada.bind("<Return>", lambda e: buscar())
entrada.configure(border_color="#C0392B")        # Resaltar un error
```

> El texto de ejemplo (`placeholder_text`) solo se muestra si **no** se usa `textvariable`.

### 6.4 Texto multilínea

```python
caja = ctk.CTkTextbox(app, width=400, height=200, wrap="word", font=ctk.CTkFont(family="Consolas", size=12))
caja.insert("0.0", "Contenido inicial\n")
texto = caja.get("0.0", "end")
caja.delete("0.0", "end")
caja.configure(state="disabled")           # Solo lectura
```

### 6.5 Casilla, interruptor y opciones

```python
acepta = ctk.CTkCheckBox(app, text="Acepto las condiciones", command=al_cambiar)
acepta.get()            # 1 o 0
acepta.select(); acepta.deselect()

switch = ctk.CTkSwitch(app, text="Notificaciones", onvalue="si", offvalue="no")
switch.get()            # "si" / "no"

tipo = ctk.StringVar(value="particular")
ctk.CTkRadioButton(app, text="Particular", variable=tipo, value="particular")
ctk.CTkRadioButton(app, text="Empresa", variable=tipo, value="empresa")
```

### 6.6 Listas desplegables

```python
# Menú de opciones (solo selección)
menu = ctk.CTkOptionMenu(app, values=["Madrid", "Sevilla", "Valencia"], command=lambda v: print(v))
menu.set("Madrid")
menu.get()
menu.configure(values=["Bilbao", "Zaragoza"])        # Cambiar opciones

# Combo (permite escribir)
combo = ctk.CTkComboBox(app, values=["Alta", "Media", "Baja"], command=lambda v: print(v))
combo.set("Media")
```

### 6.7 Botón segmentado

```python
periodo = ctk.CTkSegmentedButton(app, values=["Día", "Semana", "Mes", "Año"],
                                 command=lambda v: actualizar_grafico(v))
periodo.set("Mes")
```

### 6.8 Deslizador y barra de progreso

```python
deslizador = ctk.CTkSlider(app, from_=0, to=100, number_of_steps=20,
                           command=lambda v: etiqueta.configure(text=f"{v:.0f} %"))
deslizador.set(50)

progreso = ctk.CTkProgressBar(app, width=300)
progreso.set(0.35)                         # Valor entre 0 y 1
indeterminado = ctk.CTkProgressBar(app, mode="indeterminate")
indeterminado.start(); indeterminado.stop()
```

### 6.9 Variables de control

Funcionan igual que en Tkinter:

```python
nombre = ctk.StringVar(value="Ana")
ctk.CTkEntry(app, textvariable=nombre)
ctk.CTkLabel(app, textvariable=nombre)
nombre.trace_add("write", lambda *_: validar())
```

---

## 7. Contenedores

### 7.1 Marco (`CTkFrame`)

```python
tarjeta = ctk.CTkFrame(app, corner_radius=12)
tarjeta.pack(padx=20, pady=20, fill="both", expand=True)

transparente = ctk.CTkFrame(app, fg_color="transparent")       # Solo para agrupar
con_borde = ctk.CTkFrame(app, border_width=1, border_color=("gray80", "gray25"))
```

### 7.2 Marco con desplazamiento

```python
lista = ctk.CTkScrollableFrame(app, label_text="Tareas", width=300, height=400)
lista.pack(fill="both", expand=True)
for i in range(50):
    ctk.CTkCheckBox(lista, text=f"Tarea {i + 1}").pack(anchor="w", pady=3, padx=10)
```

### 7.3 Pestañas (`CTkTabview`)

```python
pestanas = ctk.CTkTabview(app, command=lambda: print(pestanas.get()))
pestanas.pack(fill="both", expand=True, padx=20, pady=20)

pestanas.add("General")
pestanas.add("Avanzado")
pestanas.set("General")

ctk.CTkLabel(pestanas.tab("General"), text="Opciones generales").pack(pady=10)
```

---

## 8. Colores, fuentes e imágenes

### 8.1 Colores

```python
ctk.CTkButton(app, fg_color="#1F4E78", hover_color="#163A5A", text_color="white")
ctk.CTkFrame(app, fg_color=("#FFFFFF", "#2B2B2B"))         # (modo claro, modo oscuro)
ctk.CTkLabel(app, text_color=("#333333", "#DDDDDD"))
```

| Opción | Uso |
|---|---|
| `fg_color` | Color principal (fondo del botón, del marco…). |
| `bg_color` | Color detrás del widget (esquinas redondeadas). Normalmente `"transparent"`. |
| `hover_color` | Color al pasar el ratón. |
| `text_color` / `text_color_disabled` | Color del texto. |
| `border_color`, `border_width` | Borde. |
| `button_color`, `progress_color` | Partes específicas (deslizador, barra de progreso, menús). |

> Defina los colores como tuplas `(claro, oscuro)` para que la interfaz se vea bien en ambos modos.

### 8.2 Fuentes

```python
titulo = ctk.CTkFont(family="Segoe UI", size=24, weight="bold")
normal = ctk.CTkFont(size=13)
mono = ctk.CTkFont(family="Consolas", size=12)

ctk.CTkLabel(app, text="Panel", font=titulo)
titulo.configure(size=28)                  # Actualiza todos los widgets que la usan
```

> Use `CTkFont` en lugar de tuplas `("Arial", 12)` para que la fuente respete el escalado.

### 8.3 Imágenes e iconos

```python
from PIL import Image

icono_usuario = ctk.CTkImage(
    light_image=Image.open("iconos/usuario_oscuro.png"),   # Se ve sobre fondo claro
    dark_image=Image.open("iconos/usuario_claro.png"),     # Se ve sobre fondo oscuro
    size=(20, 20),
)
ctk.CTkButton(app, text="Usuarios", image=icono_usuario, anchor="w")

logo = ctk.CTkImage(Image.open("logo.png"), size=(160, 50))
ctk.CTkLabel(app, text="", image=logo).pack()
```

> `CTkImage` escala correctamente en pantallas HiDPI. No use `tk.PhotoImage` ni `ImageTk.PhotoImage` con widgets CTk.

---

## 9. Diseño de la interfaz

Los gestores `pack`, `grid` y `place` funcionan igual que en Tkinter.

### 9.1 Diseño con barra lateral

```python
class App(ctk.CTk):
    def __init__(self):
        super().__init__()
        self.geometry("1000x620")
        self.grid_columnconfigure(1, weight=1)
        self.grid_rowconfigure(0, weight=1)

        lateral = ctk.CTkFrame(self, width=200, corner_radius=0)
        lateral.grid(row=0, column=0, sticky="nsw")
        lateral.grid_rowconfigure(5, weight=1)

        ctk.CTkLabel(lateral, text="MiEmpresa", font=ctk.CTkFont(size=20, weight="bold")).grid(
            row=0, column=0, padx=20, pady=(20, 30))
        for i, seccion in enumerate(["Inicio", "Clientes", "Pedidos", "Informes"], start=1):
            ctk.CTkButton(lateral, text=seccion, fg_color="transparent", anchor="w",
                          text_color=("gray10", "gray90"), hover_color=("gray75", "gray25"),
                          command=lambda s=seccion: self.mostrar(s)).grid(row=i, column=0, padx=10, pady=3, sticky="ew")

        ctk.CTkOptionMenu(lateral, values=["System", "Light", "Dark"],
                          command=ctk.set_appearance_mode).grid(row=6, column=0, padx=20, pady=20)

        self.contenido = ctk.CTkFrame(self, fg_color="transparent")
        self.contenido.grid(row=0, column=1, sticky="nsew", padx=20, pady=20)
```

### 9.2 Cambiar de vista

```python
def mostrar(self, nombre: str) -> None:
    for vista in self.vistas.values():
        vista.grid_forget()
    self.vistas[nombre].grid(row=0, column=0, sticky="nsew")
```

### 9.3 Formularios con `grid`

```python
formulario = ctk.CTkFrame(app)
formulario.grid_columnconfigure(1, weight=1)
for fila, (etiqueta, widget) in enumerate([
    ("Nombre", ctk.CTkEntry(formulario)),
    ("Email", ctk.CTkEntry(formulario)),
    ("Ciudad", ctk.CTkOptionMenu(formulario, values=["Madrid", "Sevilla"])),
]):
    ctk.CTkLabel(formulario, text=etiqueta).grid(row=fila, column=0, padx=10, pady=6, sticky="w")
    widget.grid(row=fila, column=1, padx=10, pady=6, sticky="ew")
```

---

## 10. Ventanas secundarias y diálogos

### 10.1 `CTkToplevel`

```python
class VentanaDetalle(ctk.CTkToplevel):
    def __init__(self, padre, cliente: dict):
        super().__init__(padre)
        self.title(f"Cliente · {cliente['nombre']}")
        self.geometry("400x300")
        self.transient(padre)
        self.after(100, self.grab_set)            # Modal (con un pequeño retraso, ver nota)
        self.after(100, self.lift)
        ctk.CTkLabel(self, text=cliente["nombre"], font=ctk.CTkFont(size=18, weight="bold")).pack(pady=20)
        ctk.CTkButton(self, text="Cerrar", command=self.destroy).pack()
```

> En algunos sistemas, `CTkToplevel` aparece detrás de la ventana principal o `grab_set()` falla si se llama inmediatamente. Usar `after(100, ...)` para `lift()`, `focus()` y `grab_set()` lo evita.

### 10.2 Diálogo de entrada

```python
dialogo = ctk.CTkInputDialog(text="Nombre del nuevo proyecto:", title="Nuevo proyecto")
nombre = dialogo.get_input()           # None si se cancela
```

### 10.3 Mensajes y diálogos de archivo

CustomTkinter no incluye cuadros de mensaje propios. Opciones:

```python
from tkinter import messagebox, filedialog

messagebox.showinfo("Información", "Datos guardados", parent=app)
ruta = filedialog.askopenfilename(parent=app, filetypes=[("CSV", "*.csv")])
```

O la librería externa **CTkMessagebox** (`pip install CTkMessagebox`), con aspecto coherente:

```python
from CTkMessagebox import CTkMessagebox

respuesta = CTkMessagebox(title="Eliminar", message="¿Eliminar el registro?", icon="warning",
                          option_1="Cancelar", option_2="Eliminar").get()
```

### 10.4 Notificación temporal propia

```python
def notificar(app, texto: str, color: str = "#2E7D32", ms: int = 2500) -> None:
    aviso = ctk.CTkLabel(app, text=f"  {texto}  ", fg_color=color, text_color="white",
                         corner_radius=8, font=ctk.CTkFont(size=13, weight="bold"))
    aviso.place(relx=0.5, rely=0.95, anchor="s")
    app.after(ms, aviso.destroy)
```

---

## 11. Combinar con Tkinter y ttk

CustomTkinter no incluye tabla ni barra de menús. Use los widgets de Tkinter adaptando sus colores al modo actual.

### 11.1 Tabla (`ttk.Treeview`) con estilo coherente

```python
from tkinter import ttk


def estilo_tabla(widget) -> None:
    oscuro = ctk.get_appearance_mode() == "Dark"
    fondo = "#2B2B2B" if oscuro else "#FFFFFF"
    texto = "#DCE4EE" if oscuro else "#1A1A1A"
    cabecera = "#1F538D" if oscuro else "#3B8ED0"
    estilo = ttk.Style(widget)
    estilo.theme_use("clam")
    estilo.configure("Treeview", background=fondo, fieldbackground=fondo, foreground=texto,
                     rowheight=28, borderwidth=0, font=("Segoe UI", 10))
    estilo.map("Treeview", background=[("selected", cabecera)], foreground=[("selected", "white")])
    estilo.configure("Treeview.Heading", background=cabecera, foreground="white", relief="flat",
                     font=("Segoe UI", 10, "bold"))
    estilo.map("Treeview.Heading", background=[("active", cabecera)])


tabla = ttk.Treeview(marco, columns=("nombre", "email"), show="headings")
estilo_tabla(app)
```

Vuelva a llamar a `estilo_tabla` cuando cambie el modo de apariencia.

### 11.2 Barra de menús

```python
import tkinter as tk

barra = tk.Menu(app)
archivo = tk.Menu(barra, tearoff=False)
archivo.add_command(label="Salir", command=app.destroy)
barra.add_cascade(label="Archivo", menu=archivo)
app.configure(menu=barra)
```

> La barra de menús nativa no sigue el modo oscuro en Windows. Muchas aplicaciones CTk sustituyen el menú por botones en una barra lateral o superior.

### 11.3 Gráficos de Matplotlib

```python
from matplotlib.figure import Figure
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

oscuro = ctk.get_appearance_mode() == "Dark"
figura = Figure(figsize=(5, 3), dpi=100, facecolor="#2B2B2B" if oscuro else "#FFFFFF")
ax = figura.add_subplot()
ax.set_facecolor(figura.get_facecolor())
ax.tick_params(colors="#DCE4EE" if oscuro else "#333333")

lienzo = FigureCanvasTkAgg(figura, master=tarjeta)
lienzo.get_tk_widget().pack(fill="both", expand=True, padx=10, pady=10)
```

---

## 12. Componentes reutilizables

Cree clases que hereden de `CTkFrame` para encapsular partes de la interfaz.

### 12.1 Tarjeta de indicador (KPI)

```python
class TarjetaKPI(ctk.CTkFrame):
    def __init__(self, padre, titulo: str, valor: str = "—", color: str = "#3B8ED0"):
        super().__init__(padre, corner_radius=12)
        ctk.CTkFrame(self, width=6, fg_color=color, corner_radius=3).pack(side="left", fill="y", padx=(10, 0), pady=12)
        cuerpo = ctk.CTkFrame(self, fg_color="transparent")
        cuerpo.pack(side="left", fill="both", expand=True, padx=12, pady=12)
        ctk.CTkLabel(cuerpo, text=titulo, text_color=("gray40", "gray60"), anchor="w").pack(fill="x")
        self.valor = ctk.CTkLabel(cuerpo, text=valor, font=ctk.CTkFont(size=26, weight="bold"), anchor="w")
        self.valor.pack(fill="x")

    def actualizar(self, valor: str) -> None:
        self.valor.configure(text=valor)
```

### 12.2 Campo de formulario con validación

```python
class Campo(ctk.CTkFrame):
    def __init__(self, padre, etiqueta: str, validador=None, **kw):
        super().__init__(padre, fg_color="transparent")
        self.validador = validador
        ctk.CTkLabel(self, text=etiqueta, anchor="w").pack(fill="x")
        self.entrada = ctk.CTkEntry(self, **kw)
        self.entrada.pack(fill="x")
        self.error = ctk.CTkLabel(self, text="", text_color="#E74C3C", anchor="w", font=ctk.CTkFont(size=11))
        self.error.pack(fill="x")
        self._borde = self.entrada.cget("border_color")

    def valor(self) -> str:
        return self.entrada.get().strip()

    def validar(self) -> bool:
        mensaje = self.validador(self.valor()) if self.validador else None
        self.error.configure(text=mensaje or "")
        self.entrada.configure(border_color="#E74C3C" if mensaje else self._borde)
        return mensaje is None
```

---

## 13. Tareas largas

Como en Tkinter, las tareas lentas deben ejecutarse en otro hilo y la interfaz debe actualizarse desde el hilo principal con `after`:

```python
import threading

def descargar(self):
    self.boton.configure(state="disabled", text="Descargando…")
    self.progreso.configure(mode="indeterminate")
    self.progreso.start()

    def trabajo():
        try:
            datos = obtener_datos_lentos()
            self.after(0, self.descarga_terminada, datos, None)
        except Exception as e:
            self.after(0, self.descarga_terminada, None, e)

    threading.Thread(target=trabajo, daemon=True).start()

def descarga_terminada(self, datos, error):
    self.progreso.stop()
    self.progreso.configure(mode="determinate")
    self.progreso.set(1 if error is None else 0)
    self.boton.configure(state="normal", text="Descargar")
    ...
```

> Programar con `self.after(0, ...)` desde el hilo secundario hace que la función se ejecute en el hilo de la interfaz. Para muchos mensajes (por ejemplo, progreso), use una `queue.Queue` revisada periódicamente.

---

## 14. Temas personalizados

Un tema es un archivo JSON con los colores de cada tipo de widget. Lo más sencillo es copiar uno de los incluidos y modificarlo:

```python
import customtkinter, os, shutil
origen = os.path.join(os.path.dirname(customtkinter.__file__), "assets", "themes", "blue.json")
shutil.copy(origen, "mi_tema.json")
```

Fragmento del archivo:

```json
{
  "CTkButton": {
    "corner_radius": 8,
    "border_width": 0,
    "fg_color": ["#1F4E78", "#2E6DA4"],
    "hover_color": ["#163A5A", "#245683"],
    "border_color": ["#3E454A", "#949A9F"],
    "text_color": ["#FFFFFF", "#FFFFFF"],
    "text_color_disabled": ["gray74", "gray60"]
  },
  "CTkFont": {
    "Windows": {"family": "Segoe UI", "size": 13, "weight": "normal"},
    "macOS": {"family": "SF Display", "size": 13, "weight": "normal"},
    "Linux": {"family": "Roboto", "size": 13, "weight": "normal"}
  }
}
```

```python
ctk.set_default_color_theme("mi_tema.json")
```

> Cada color es una lista `[modo_claro, modo_oscuro]`. Existen temas de la comunidad (por ejemplo, el repositorio *CTkThemesPack*).

---

## 15. Ejemplo práctico completo

**Panel de control de tareas** con: barra lateral de navegación, selector de modo y escala, vista de inicio con tarjetas KPI, vista de tareas con alta, filtros, búsqueda y lista desplazable, vista de ajustes, persistencia en JSON, validación, notificaciones y una operación de sincronización simulada en segundo plano.

```python
"""
Panel de tareas con CustomTkinter.
Requisitos: pip install customtkinter
"""
import json
import random
import threading
import time
import uuid
from dataclasses import asdict, dataclass, field
from datetime import date, datetime, timedelta
from pathlib import Path

import customtkinter as ctk

RUTA_DATOS = Path(__file__).resolve().parent / "tareas.json"
PRIORIDADES = {"Alta": "#E74C3C", "Media": "#F39C12", "Baja": "#27AE60"}


# ------------------------------------------------------------------ Modelo
@dataclass
class Tarea:
    titulo: str
    prioridad: str = "Media"
    vence: str = field(default_factory=lambda: date.today().isoformat())
    hecha: bool = False
    id: str = field(default_factory=lambda: uuid.uuid4().hex[:8])

    @property
    def vencida(self) -> bool:
        return not self.hecha and date.fromisoformat(self.vence) < date.today()


class Almacen:
    def __init__(self, ruta: Path):
        self.ruta = ruta
        self.tareas: list[Tarea] = []
        self.cargar()

    def cargar(self) -> None:
        if self.ruta.exists():
            self.tareas = [Tarea(**t) for t in json.loads(self.ruta.read_text(encoding="utf-8"))]
        else:
            hoy = date.today()
            self.tareas = [
                Tarea("Preparar informe mensual", "Alta", (hoy + timedelta(days=1)).isoformat()),
                Tarea("Revisar facturas pendientes", "Media", (hoy - timedelta(days=2)).isoformat()),
                Tarea("Llamar a proveedores", "Baja", (hoy + timedelta(days=5)).isoformat()),
                Tarea("Actualizar inventario", "Media", hoy.isoformat(), hecha=True),
            ]
            self.guardar()

    def guardar(self) -> None:
        self.ruta.write_text(json.dumps([asdict(t) for t in self.tareas], ensure_ascii=False, indent=2),
                             encoding="utf-8")

    def anadir(self, tarea: Tarea) -> None:
        self.tareas.append(tarea)
        self.guardar()

    def eliminar(self, id_: str) -> None:
        self.tareas = [t for t in self.tareas if t.id != id_]
        self.guardar()

    def alternar(self, id_: str) -> None:
        for t in self.tareas:
            if t.id == id_:
                t.hecha = not t.hecha
        self.guardar()


# ------------------------------------------------------------------ Componentes
class TarjetaKPI(ctk.CTkFrame):
    def __init__(self, padre, titulo: str, color: str):
        super().__init__(padre, corner_radius=12)
        ctk.CTkFrame(self, width=6, fg_color=color, corner_radius=3).pack(side="left", fill="y", padx=(10, 0), pady=12)
        cuerpo = ctk.CTkFrame(self, fg_color="transparent")
        cuerpo.pack(side="left", fill="both", expand=True, padx=12, pady=12)
        ctk.CTkLabel(cuerpo, text=titulo, text_color=("gray40", "gray60"), anchor="w").pack(fill="x")
        self.valor = ctk.CTkLabel(cuerpo, text="0", font=ctk.CTkFont(size=28, weight="bold"), anchor="w")
        self.valor.pack(fill="x")


class FilaTarea(ctk.CTkFrame):
    def __init__(self, padre, tarea: Tarea, al_alternar, al_eliminar):
        super().__init__(padre, corner_radius=8, fg_color=("gray92", "gray17"))
        self.grid_columnconfigure(1, weight=1)
        casilla = ctk.CTkCheckBox(self, text="", width=24, command=lambda: al_alternar(tarea.id))
        if tarea.hecha:
            casilla.select()
        casilla.grid(row=0, column=0, rowspan=2, padx=(12, 6), pady=10)

        fuente = ctk.CTkFont(size=14, overstrike=tarea.hecha)
        ctk.CTkLabel(self, text=tarea.titulo, font=fuente, anchor="w",
                     text_color=("gray50", "gray55") if tarea.hecha else ("gray10", "gray90")).grid(
            row=0, column=1, sticky="ew", pady=(8, 0))
        vence = date.fromisoformat(tarea.vence).strftime("%d/%m/%Y")
        detalle = f"Vence: {vence}" + ("  ·  ⚠ vencida" if tarea.vencida else "")
        ctk.CTkLabel(self, text=detalle, anchor="w", font=ctk.CTkFont(size=11),
                     text_color="#E74C3C" if tarea.vencida else ("gray45", "gray60")).grid(
            row=1, column=1, sticky="ew", pady=(0, 8))

        ctk.CTkLabel(self, text=f" {tarea.prioridad} ", fg_color=PRIORIDADES[tarea.prioridad],
                     text_color="white", corner_radius=6, font=ctk.CTkFont(size=11, weight="bold")).grid(
            row=0, column=2, rowspan=2, padx=8)
        ctk.CTkButton(self, text="✕", width=30, height=30, fg_color="transparent",
                      hover_color=("gray80", "gray25"), text_color=("gray30", "gray70"),
                      command=lambda: al_eliminar(tarea.id)).grid(row=0, column=3, rowspan=2, padx=(0, 10))


# ------------------------------------------------------------------ Vistas
class VistaInicio(ctk.CTkFrame):
    def __init__(self, padre, app: "App"):
        super().__init__(padre, fg_color="transparent")
        self.app = app
        ctk.CTkLabel(self, text="Resumen", font=ctk.CTkFont(size=26, weight="bold")).pack(anchor="w")
        self.saludo = ctk.CTkLabel(self, text="", text_color=("gray40", "gray60"))
        self.saludo.pack(anchor="w", pady=(0, 20))

        rejilla = ctk.CTkFrame(self, fg_color="transparent")
        rejilla.pack(fill="x")
        self.kpis = {}
        for i, (clave, titulo, color) in enumerate([("total", "Tareas totales", "#3B8ED0"),
                                                    ("pendientes", "Pendientes", "#F39C12"),
                                                    ("vencidas", "Vencidas", "#E74C3C"),
                                                    ("hechas", "Completadas", "#27AE60")]):
            rejilla.grid_columnconfigure(i, weight=1, uniform="kpi")
            self.kpis[clave] = TarjetaKPI(rejilla, titulo, color)
            self.kpis[clave].grid(row=0, column=i, sticky="ew", padx=6)

        ctk.CTkLabel(self, text="Progreso", font=ctk.CTkFont(size=16, weight="bold")).pack(anchor="w", pady=(30, 6))
        self.barra = ctk.CTkProgressBar(self, height=14)
        self.barra.pack(fill="x")
        self.porcentaje = ctk.CTkLabel(self, text="")
        self.porcentaje.pack(anchor="e")

    def refrescar(self) -> None:
        t = self.app.almacen.tareas
        hechas = sum(x.hecha for x in t)
        valores = {"total": len(t), "pendientes": len(t) - hechas,
                   "vencidas": sum(x.vencida for x in t), "hechas": hechas}
        for clave, v in valores.items():
            self.kpis[clave].valor.configure(text=str(v))
        ratio = hechas / len(t) if t else 0
        self.barra.set(ratio)
        self.porcentaje.configure(text=f"{ratio:.0%} completado")
        hora = datetime.now().hour
        saludo = "Buenos días" if hora < 14 else "Buenas tardes" if hora < 21 else "Buenas noches"
        self.saludo.configure(text=f"{saludo}. Hoy es {date.today():%d/%m/%Y}.")


class VistaTareas(ctk.CTkFrame):
    def __init__(self, padre, app: "App"):
        super().__init__(padre, fg_color="transparent")
        self.app = app
        self.grid_columnconfigure(0, weight=1)
        self.grid_rowconfigure(3, weight=1)

        ctk.CTkLabel(self, text="Tareas", font=ctk.CTkFont(size=26, weight="bold")).grid(row=0, column=0, sticky="w")

        alta = ctk.CTkFrame(self)
        alta.grid(row=1, column=0, sticky="ew", pady=(15, 10))
        alta.grid_columnconfigure(0, weight=1)
        self.titulo = ctk.CTkEntry(alta, placeholder_text="Nueva tarea…", height=36)
        self.titulo.grid(row=0, column=0, sticky="ew", padx=(10, 6), pady=10)
        self.titulo.bind("<Return>", lambda _: self.anadir())
        self.prioridad = ctk.CTkOptionMenu(alta, values=list(PRIORIDADES), width=100)
        self.prioridad.set("Media")
        self.prioridad.grid(row=0, column=1, padx=6)
        self.dias = ctk.CTkOptionMenu(alta, values=["Hoy", "Mañana", "En 3 días", "En 1 semana"], width=120)
        self.dias.set("Mañana")
        self.dias.grid(row=0, column=2, padx=6)
        ctk.CTkButton(alta, text="Añadir", width=90, height=36, command=self.anadir).grid(row=0, column=3, padx=(6, 10))

        filtros = ctk.CTkFrame(self, fg_color="transparent")
        filtros.grid(row=2, column=0, sticky="ew", pady=(0, 8))
        self.filtro = ctk.CTkSegmentedButton(filtros, values=["Todas", "Pendientes", "Vencidas", "Hechas"],
                                             command=lambda _: self.refrescar())
        self.filtro.set("Pendientes")
        self.filtro.pack(side="left")
        self.busqueda = ctk.StringVar()
        self.busqueda.trace_add("write", lambda *_: self.refrescar())
        ctk.CTkEntry(filtros, textvariable=self.busqueda, width=200).pack(side="right")
        ctk.CTkLabel(filtros, text="Buscar:").pack(side="right", padx=6)

        self.lista = ctk.CTkScrollableFrame(self)
        self.lista.grid(row=3, column=0, sticky="nsew")
        self.lista.grid_columnconfigure(0, weight=1)

    def anadir(self) -> None:
        texto = self.titulo.get().strip()
        if len(texto) < 3:
            self.titulo.configure(border_color="#E74C3C")
            self.app.notificar("El título debe tener al menos 3 caracteres", "#C0392B")
            return
        self.titulo.configure(border_color=("#979DA2", "#565B5E"))
        dias = {"Hoy": 0, "Mañana": 1, "En 3 días": 3, "En 1 semana": 7}[self.dias.get()]
        self.app.almacen.anadir(Tarea(texto, self.prioridad.get(), (date.today() + timedelta(days=dias)).isoformat()))
        self.titulo.delete(0, "end")
        self.app.notificar("Tarea añadida")
        self.app.refrescar()

    def refrescar(self) -> None:
        for w in self.lista.winfo_children():
            w.destroy()
        filtro, texto = self.filtro.get(), self.busqueda.get().lower().strip()
        orden = {"Alta": 0, "Media": 1, "Baja": 2}
        tareas = sorted(self.app.almacen.tareas, key=lambda t: (t.hecha, orden[t.prioridad], t.vence))
        visibles = [t for t in tareas
                    if (filtro == "Todas" or (filtro == "Pendientes" and not t.hecha)
                        or (filtro == "Vencidas" and t.vencida) or (filtro == "Hechas" and t.hecha))
                    and texto in t.titulo.lower()]
        if not visibles:
            ctk.CTkLabel(self.lista, text="No hay tareas que mostrar", text_color=("gray50", "gray55")).grid(pady=30)
        for i, t in enumerate(visibles):
            FilaTarea(self.lista, t, self.alternar, self.eliminar).grid(row=i, column=0, sticky="ew", pady=3, padx=4)

    def alternar(self, id_: str) -> None:
        self.app.almacen.alternar(id_)
        self.app.refrescar()

    def eliminar(self, id_: str) -> None:
        self.app.almacen.eliminar(id_)
        self.app.notificar("Tarea eliminada", "#7F8C8D")
        self.app.refrescar()


class VistaAjustes(ctk.CTkFrame):
    def __init__(self, padre, app: "App"):
        super().__init__(padre, fg_color="transparent")
        self.app = app
        ctk.CTkLabel(self, text="Ajustes", font=ctk.CTkFont(size=26, weight="bold")).pack(anchor="w", pady=(0, 20))
        tarjeta = ctk.CTkFrame(self)
        tarjeta.pack(fill="x")
        tarjeta.grid_columnconfigure(1, weight=1)

        ctk.CTkLabel(tarjeta, text="Apariencia").grid(row=0, column=0, padx=15, pady=12, sticky="w")
        modo = ctk.CTkSegmentedButton(tarjeta, values=["Light", "Dark", "System"], command=ctk.set_appearance_mode)
        modo.set("System")
        modo.grid(row=0, column=1, padx=15, sticky="e")

        ctk.CTkLabel(tarjeta, text="Escala de la interfaz").grid(row=1, column=0, padx=15, pady=12, sticky="w")
        escala = ctk.CTkOptionMenu(tarjeta, values=["90%", "100%", "110%", "125%"],
                                   command=lambda v: ctk.set_widget_scaling(int(v[:-1]) / 100))
        escala.set("100%")
        escala.grid(row=1, column=1, padx=15, sticky="e")

        ctk.CTkLabel(tarjeta, text="Sincronizar con el servidor").grid(row=2, column=0, padx=15, pady=12, sticky="w")
        self.boton_sync = ctk.CTkButton(tarjeta, text="Sincronizar", command=self.sincronizar)
        self.boton_sync.grid(row=2, column=1, padx=15, sticky="e")
        self.progreso = ctk.CTkProgressBar(tarjeta)
        self.progreso.set(0)
        self.progreso.grid(row=3, column=0, columnspan=2, padx=15, pady=(0, 15), sticky="ew")

    def sincronizar(self) -> None:
        self.boton_sync.configure(state="disabled", text="Sincronizando…")

        def trabajo():
            for i in range(1, 21):
                time.sleep(random.uniform(0.03, 0.1))         # Simula la red
                self.after(0, self.progreso.set, i / 20)
            self.after(0, fin)

        def fin():
            self.boton_sync.configure(state="normal", text="Sincronizar")
            self.app.notificar(f"Sincronizado a las {datetime.now():%H:%M:%S}")

        threading.Thread(target=trabajo, daemon=True).start()

    def refrescar(self) -> None:
        pass


# ------------------------------------------------------------------ Aplicación
class App(ctk.CTk):
    def __init__(self):
        super().__init__()
        self.title("Panel de tareas")
        self.geometry("1000x640")
        self.minsize(820, 520)
        self.almacen = Almacen(RUTA_DATOS)
        self.grid_columnconfigure(1, weight=1)
        self.grid_rowconfigure(0, weight=1)

        lateral = ctk.CTkFrame(self, width=200, corner_radius=0)
        lateral.grid(row=0, column=0, sticky="ns")
        lateral.grid_rowconfigure(10, weight=1)
        ctk.CTkLabel(lateral, text="✔ Tareas", font=ctk.CTkFont(size=22, weight="bold")).grid(
            row=0, column=0, padx=20, pady=(24, 30))

        contenedor = ctk.CTkFrame(self, fg_color="transparent")
        contenedor.grid(row=0, column=1, sticky="nsew", padx=24, pady=24)
        contenedor.grid_columnconfigure(0, weight=1)
        contenedor.grid_rowconfigure(0, weight=1)

        self.vistas = {"Inicio": VistaInicio(contenedor, self), "Tareas": VistaTareas(contenedor, self),
                       "Ajustes": VistaAjustes(contenedor, self)}
        self.botones = {}
        for i, nombre in enumerate(self.vistas, start=1):
            b = ctk.CTkButton(lateral, text=nombre, anchor="w", height=38, fg_color="transparent",
                              text_color=("gray10", "gray90"), hover_color=("gray75", "gray25"),
                              command=lambda n=nombre: self.mostrar(n))
            b.grid(row=i, column=0, padx=12, pady=3, sticky="ew")
            self.botones[nombre] = b

        ctk.CTkLabel(lateral, text="v1.0", text_color=("gray50", "gray55")).grid(row=11, column=0, pady=15)
        self.bind("<Control-n>", lambda _: (self.mostrar("Tareas"), self.vistas["Tareas"].titulo.focus()))
        self.refrescar()
        self.mostrar("Inicio")

    def mostrar(self, nombre: str) -> None:
        for n, vista in self.vistas.items():
            vista.grid_forget()
            self.botones[n].configure(fg_color="transparent")
        self.vistas[nombre].grid(row=0, column=0, sticky="nsew")
        self.botones[nombre].configure(fg_color=("gray75", "gray25"))

    def refrescar(self) -> None:
        for vista in self.vistas.values():
            vista.refrescar()

    def notificar(self, texto: str, color: str = "#2E7D32") -> None:
        aviso = ctk.CTkLabel(self, text=f"  {texto}  ", fg_color=color, text_color="white", corner_radius=8,
                             height=34, font=ctk.CTkFont(size=13, weight="bold"))
        aviso.place(relx=0.58, rely=0.97, anchor="s")
        self.after(2200, aviso.destroy)


if __name__ == "__main__":
    ctk.set_appearance_mode("system")
    ctk.set_default_color_theme("blue")
    App().mainloop()
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'customtkinter'` | Librería no instalada. | `pip install customtkinter`. |
| `ValueError: ['font'] are not supported arguments` o similar | Opción de Tkinter no existente en el widget CTk. | Consulte las opciones del widget CTk (p. ej. `fg_color` en lugar de `bg`). |
| `Warning: Given image is not CTkImage` / imagen borrosa | Uso de `PhotoImage` en un widget CTk. | Use `ctk.CTkImage`. |
| El texto de ejemplo no aparece | Se usa `textvariable` junto con `placeholder_text`. | Elija uno de los dos. |
| La ventana secundaria queda detrás de la principal | Comportamiento de `CTkToplevel` en algunos sistemas. | `self.after(100, self.lift)` y `self.after(100, self.grab_set)`. |
| Los colores de un `ttk.Treeview` no cambian con el modo oscuro | Los widgets de Tkinter no siguen el tema de CTk. | Reaplique el estilo al cambiar de modo. |
| `width` no tiene efecto en un widget con `pack(fill="x")` | El gestor de geometría estira el widget. | Quite `fill` o use `grid` con `weight`. |
| Interfaz lenta con muchos elementos | Cientos de widgets CTk (cada uno se dibuja en un canvas). | Pagine, use `CTkScrollableFrame` con menos elementos o un `ttk.Treeview` para tablas grandes. |
| La interfaz se congela | Tarea larga en el hilo principal. | Hilos + `after`. |
| `TclError` al cambiar opciones desde otro hilo | Modificación de widgets fuera del hilo principal. | Use `self.after(0, funcion, args)`. |

---

## 17. Buenas prácticas

- Configure apariencia, tema y escala **antes** de crear `CTk()`.
- Defina los colores como tuplas `(claro, oscuro)` y pruebe la interfaz en ambos modos.
- Use `CTkFont` y `CTkImage` para respetar el escalado.
- Organice la aplicación en clases: ventana principal, vistas y componentes reutilizables.
- Use `grid` con `weight` para interfaces que se adapten al tamaño de la ventana.
- No mezcle estilos: si necesita widgets de Tkinter (tablas, menús), adapte sus colores al tema.
- Muestre el estado de las operaciones (botones desactivados, barras de progreso, notificaciones).
- Ejecute las tareas lentas en hilos y actualice la interfaz con `after`.
- Persista la configuración del usuario (modo, escala, tamaño de ventana).

---

## 18. Referencia rápida

```python
ctk.set_appearance_mode("system" | "light" | "dark"); ctk.set_default_color_theme("blue" | "green" | "dark-blue" | "tema.json")
ctk.set_widget_scaling(1.1); ctk.set_window_scaling(1.0)

app = ctk.CTk(); app.title(); app.geometry("900x600"); app.mainloop()
ctk.CTkLabel(text, font=ctk.CTkFont(size, weight), text_color, image)
ctk.CTkButton(text, command, fg_color, hover_color, border_width, corner_radius, image, state)
ctk.CTkEntry(placeholder_text, show, textvariable, width) · ctk.CTkTextbox(wrap)
ctk.CTkCheckBox / CTkSwitch / CTkRadioButton(variable, value) · .get() .select() .deselect()
ctk.CTkOptionMenu(values, command) / CTkComboBox / CTkSegmentedButton · .set() .get()
ctk.CTkSlider(from_, to, number_of_steps, command) · ctk.CTkProgressBar() .set(0..1) .start() .stop()
ctk.CTkFrame(fg_color="transparent", corner_radius) / CTkScrollableFrame(label_text) / CTkTabview() .add(n) .tab(n)
ctk.CTkToplevel(padre) / ctk.CTkInputDialog(text, title).get_input()
ctk.CTkImage(light_image=Image.open(...), dark_image=..., size=(w, h))
widget.configure(opción=valor) / widget.cget("opción")
```

---

## 19. Ejercicios propuestos

1. Cree una pantalla de inicio de sesión con validación de campos, opción «recordarme» y cambio de modo claro/oscuro.
2. Desarrolle un conversor de divisas con `CTkSegmentedButton` para elegir la moneda y actualización en vivo.
3. Construya un reproductor de lista de tareas que guarde los datos en SQLite en lugar de JSON.
4. Integre un `ttk.Treeview` con estilo adaptado al modo oscuro para mostrar un CSV de clientes.
5. Amplíe el ejemplo completo con una vista de estadísticas que incluya un gráfico de Matplotlib con colores coherentes con el tema.

---

## 20. Referencias

- Documentación oficial: https://customtkinter.tomschimansky.com/
- Repositorio y wiki: https://github.com/TomSchimansky/CustomTkinter
- CTkMessagebox: https://github.com/Akascape/CTkMessagebox
