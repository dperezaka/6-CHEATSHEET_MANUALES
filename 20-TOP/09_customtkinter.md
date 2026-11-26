# Manual de uso: CustomTkinter

> **Categoría:** Interfaces gráficas · **Utilidad:** Interfaces de escritorio modernas sobre Tkinter
> **Tipo:** Librería de terceros (`pip install customtkinter`) · **Versión de referencia:** 5.2 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Primera aplicación](#3-primera-aplicación)
4. [Modo de apariencia, temas y escalado](#4-modo-de-apariencia-temas-y-escalado)
5. [Colores y fuentes](#5-colores-y-fuentes)
6. [Botones, etiquetas y entradas de texto](#6-botones-etiquetas-y-entradas-de-texto)
7. [Controles de selección](#7-controles-de-selección)
8. [Deslizadores y barras de progreso](#8-deslizadores-y-barras-de-progreso)
9. [Contenedores: marcos, marcos desplazables y pestañas](#9-contenedores-marcos-marcos-desplazables-y-pestañas)
10. [Ventanas secundarias y diálogos](#10-ventanas-secundarias-y-diálogos)
11. [Imágenes con `CTkImage`](#11-imágenes-con-ctkimage)
12. [Diseño con `grid`](#12-diseño-con-grid)
13. [Variables, eventos y atajos de teclado](#13-variables-eventos-y-atajos-de-teclado)
14. [Arquitectura orientada a objetos](#14-arquitectura-orientada-a-objetos)
15. [Tareas largas sin bloquear la interfaz](#15-tareas-largas-sin-bloquear-la-interfaz)
16. [Temas personalizados](#16-temas-personalizados)
17. [Gráficos con Matplotlib](#17-gráficos-con-matplotlib)
18. [Combinación con Tkinter y ttk](#18-combinación-con-tkinter-y-ttk)
19. [Distribución como ejecutable](#19-distribución-como-ejecutable)
20. [Ejemplo práctico completo](#20-ejemplo-práctico-completo)
21. [Errores frecuentes](#21-errores-frecuentes)
22. [Buenas prácticas](#22-buenas-prácticas)
23. [Referencia rápida](#23-referencia-rápida)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Referencias](#25-referencias)

---

## 1. Descripción

**CustomTkinter** es una librería de interfaces gráficas construida sobre **Tkinter** que ofrece *widgets* de aspecto moderno: esquinas redondeadas, modo claro y oscuro (incluido el del sistema), temas de color y escalado automático en pantallas de alta resolución. Sus clases (`CTkButton`, `CTkEntry`, `CTkFrame`…) se usan igual que las de Tkinter, por lo que lo aprendido con Tkinter (gestores de geometría, eventos, variables) es directamente aplicable.

**Ventajas:**
- Aspecto actual sin esfuerzo, coherente en Windows, macOS y Linux.
- Modo claro/oscuro y temas con una sola línea.
- Curva de aprendizaje mínima para quien conoce Tkinter.
- Ligera: solo depende de Tkinter (incluido en Python) y de `darkdetect`.
- Compatible con *widgets* de Tkinter/ttk y con integraciones como Matplotlib.

**Limitaciones:**
- Catálogo de *widgets* reducido: no incluye tablas (`Treeview`), menús de barra, calendarios ni árboles (se usan los de Tkinter/ttk o paquetes externos).
- Rendimiento inferior a Qt con cientos de *widgets* (cada uno se dibuja en un `Canvas`).
- Sin soporte de estilos tipo CSS ni animaciones avanzadas.
- Hereda las limitaciones de Tkinter (un solo hilo para la interfaz).

| Si necesita… | Opción |
|---|---|
| Aplicación de escritorio sencilla con aspecto moderno | **CustomTkinter** |
| Máxima simplicidad y cero dependencias | Tkinter / ttk |
| Aplicación profesional compleja (tablas, *docking*, multimedia) | PySide6 / PyQt6 |
| Interfaz web para datos sin escribir HTML | Streamlit |
| Aplicación multiplataforma con móvil | Flet, Kivy |

---

## 2. Instalación e importación

```bash
pip install customtkinter
```

Tkinter debe estar disponible en Python: viene incluido en los instaladores de Windows y macOS; en Linux puede requerir el paquete del sistema (`sudo apt install python3-tk`).

```python
import customtkinter as ctk
```

---

## 3. Primera aplicación

```python
import customtkinter as ctk

ctk.set_appearance_mode("system")          # "light", "dark" o "system"
ctk.set_default_color_theme("blue")        # "blue", "green", "dark-blue" o ruta a un JSON


class App(ctk.CTk):
    def __init__(self):
        super().__init__()
        self.title("Saludo")
        self.geometry("360x180")
        self.grid_columnconfigure(0, weight=1)

        self.entrada = ctk.CTkEntry(self, placeholder_text="Escriba su nombre")
        self.entrada.grid(row=0, column=0, padx=20, pady=(20, 10), sticky="ew")
        ctk.CTkButton(self, text="Saludar", command=self.saludar).grid(row=1, column=0, padx=20)
        self.resultado = ctk.CTkLabel(self, text="")
        self.resultado.grid(row=2, column=0, pady=10)

    def saludar(self):
        nombre = self.entrada.get().strip() or "desconocido"
        self.resultado.configure(text=f"Hola, {nombre}")


if __name__ == "__main__":
    App().mainloop()
```

| Tkinter | CustomTkinter |
|---|---|
| `tk.Tk()` | `ctk.CTk()` |
| `tk.Toplevel` | `ctk.CTkToplevel` |
| `tk.Button`, `tk.Label`, `tk.Entry` | `ctk.CTkButton`, `ctk.CTkLabel`, `ctk.CTkEntry` |
| `tk.Frame` | `ctk.CTkFrame` |
| `widget.config(...)` | `widget.configure(...)` |
| `bg=`, `fg=` | `fg_color=`, `text_color=` |

---

## 4. Modo de apariencia, temas y escalado

```python
ctk.set_appearance_mode("dark")            # Cambiable en tiempo de ejecución
ctk.get_appearance_mode()                  # "Light" o "Dark"
ctk.set_default_color_theme("green")       # Antes de crear widgets
ctk.set_widget_scaling(1.2)                # Tamaño de los widgets (120 %)
ctk.set_window_scaling(1.0)                # Tamaño de las ventanas
ctk.deactivate_automatic_dpi_awareness()   # Desactivar el escalado automático (antes de crear la ventana)
```

- El tema de color se aplica a los *widgets* creados **después** de llamarlo; establézcalo al inicio del programa.
- El modo de apariencia sí se puede cambiar en caliente: todos los *widgets* se redibujan.

---

## 5. Colores y fuentes

### 5.1 Colores

Los colores aceptan un nombre o código hexadecimal, o una **tupla `(claro, oscuro)`** que se adapta automáticamente al modo:

```python
ctk.CTkButton(app, text="Eliminar", fg_color="#C0392B", hover_color="#962D22", text_color="white")
ctk.CTkFrame(app, fg_color=("#F2F4F7", "#1F2329"), corner_radius=12, border_width=1,
             border_color=("#D0D5DD", "#3A3F47"))
ctk.CTkLabel(app, text="Aviso", text_color=("#B54708", "#FDB022"))
ctk.CTkFrame(app, fg_color="transparent")       # Hereda el fondo del contenedor
```

| Parámetro | Elemento |
|---|---|
| `fg_color` | Relleno principal del *widget*. |
| `bg_color` | Fondo detrás de las esquinas redondeadas (normalmente automático). |
| `hover_color` | Relleno al pasar el ratón. |
| `text_color`, `text_color_disabled` | Texto. |
| `border_color`, `border_width` | Borde. |
| `corner_radius` | Radio de las esquinas (0 = rectangular). |
| `button_color`, `progress_color` | Partes específicas (deslizadores, menús, barras). |

### 5.2 Fuentes

```python
titulo = ctk.CTkFont(family="Segoe UI", size=22, weight="bold")
ctk.CTkLabel(app, text="Panel", font=titulo)
ctk.CTkButton(app, text="Aceptar", font=("Arial", 13))       # Tupla también válida
titulo.configure(size=26)                                    # Actualiza todos los widgets que la usan
```

> Cree los objetos `CTkFont` **después** de la ventana principal (necesitan un intérprete Tk activo).

---

## 6. Botones, etiquetas y entradas de texto

### 6.1 `CTkButton` y `CTkLabel`

```python
boton = ctk.CTkButton(app, text="Guardar", width=120, height=36, command=guardar,
                      image=icono, compound="left", anchor="center")
boton.configure(state="disabled")             # "normal" / "disabled"
boton.invoke()                                # Ejecuta el comando por programa

etiqueta = ctk.CTkLabel(app, text="Estado: listo", anchor="w", justify="left", wraplength=300)
etiqueta.configure(text="Estado: guardado")
etiqueta.cget("text")
```

### 6.2 `CTkEntry`

```python
entrada = ctk.CTkEntry(app, placeholder_text="correo@empresa.com", width=260)
clave = ctk.CTkEntry(app, show="•")
entrada.get() ; entrada.delete(0, "end") ; entrada.insert(0, "texto")
entrada.configure(border_color="#C0392B")     # Resaltar un error de validación
entrada.bind("<Return>", lambda e: enviar())
```

> El *placeholder* se muestra solo si el campo está vacío y **no** tiene `textvariable`. Si necesita ambos, prescinda del *placeholder* o gestione la variable usted mismo.

### 6.3 `CTkTextbox` (texto multilínea)

```python
caja = ctk.CTkTextbox(app, width=400, height=200, wrap="word", font=("Consolas", 12))
caja.insert("end", "Primera línea\n")
contenido = caja.get("1.0", "end-1c")        # Todo el texto sin el salto final
caja.delete("1.0", "end")
caja.configure(state="disabled")             # Solo lectura (vuelva a "normal" para escribir)
caja.see("end")                              # Desplazar al final (registros)
caja.tag_config("error", foreground="red")   # Etiquetas de formato de tk.Text
```

---

## 7. Controles de selección

```python
# Casilla y conmutador (valor 1/0 por defecto, o personalizado)
acepta = ctk.CTkCheckBox(app, text="Acepto las condiciones", command=actualizar)
acepta.get() ; acepta.select() ; acepta.deselect()
modo_oscuro = ctk.CTkSwitch(app, text="Modo oscuro", onvalue="dark", offvalue="light",
                            command=lambda: ctk.set_appearance_mode(modo_oscuro.get()))

# Botones de opción (comparten variable)
prioridad = ctk.StringVar(value="media")
for valor in ("baja", "media", "alta"):
    ctk.CTkRadioButton(app, text=valor.capitalize(), variable=prioridad, value=valor).pack(anchor="w")

# Lista desplegable no editable y combinada (editable)
menu = ctk.CTkOptionMenu(app, values=["Madrid", "Sevilla", "Bilbao"], command=lambda v: print(v))
menu.set("Sevilla") ; menu.get()
combo = ctk.CTkComboBox(app, values=["Alimentación", "Transporte", "Ocio"])
combo.configure(values=["Nueva", "Lista"])    # Actualizar opciones

# Botón segmentado (selector tipo pestaña)
tipo = ctk.CTkSegmentedButton(app, values=["Gasto", "Ingreso"], command=cambiar_tipo)
tipo.set("Gasto")
```

| *Widget* | Uso |
|---|---|
| `CTkCheckBox` / `CTkSwitch` | Opción sí/no (el conmutador para ajustes que se aplican al momento). |
| `CTkRadioButton` | Una opción entre pocas, visibles a la vez. |
| `CTkOptionMenu` | Una opción entre muchas (lista cerrada). |
| `CTkComboBox` | Lista sugerida que también admite texto libre. |
| `CTkSegmentedButton` | 2–5 opciones excluyentes de forma compacta. |

---

## 8. Deslizadores y barras de progreso

```python
etiqueta = ctk.CTkLabel(app, text="50 %")
deslizador = ctk.CTkSlider(app, from_=0, to=100, number_of_steps=20,
                           command=lambda v: etiqueta.configure(text=f"{v:.0f} %"))
deslizador.set(50) ; deslizador.get()

barra = ctk.CTkProgressBar(app, width=300, progress_color="#12B76A")
barra.set(0.35)                                  # Valor entre 0 y 1

espera = ctk.CTkProgressBar(app, mode="indeterminate")
espera.start() ; espera.stop()
```

---

## 9. Contenedores: marcos, marcos desplazables y pestañas

```python
tarjeta = ctk.CTkFrame(app, corner_radius=12)
tarjeta.grid(row=0, column=0, padx=10, pady=10, sticky="nsew")

lista = ctk.CTkScrollableFrame(app, label_text="Tareas", width=320, height=240)
for i in range(30):
    ctk.CTkCheckBox(lista, text=f"Tarea {i + 1}").pack(anchor="w", pady=2)
# orientation="horizontal" para desplazamiento horizontal

pestanas = ctk.CTkTabview(app, width=500, height=300, command=lambda: print(pestanas.get()))
pestanas.add("General")
pestanas.add("Avanzado")
ctk.CTkLabel(pestanas.tab("General"), text="Opciones generales").pack(pady=10)
pestanas.set("Avanzado")                          # Pestaña activa
pestanas.delete("Avanzado")
```

> Los hijos de `CTkScrollableFrame` se colocan con `pack` o `grid` **dentro del propio marco**; no use `place` en él.

---

## 10. Ventanas secundarias y diálogos

```python
class VentanaDetalle(ctk.CTkToplevel):
    def __init__(self, padre, registro):
        super().__init__(padre)
        self.title("Detalle")
        self.geometry("320x200")
        self.transient(padre)                        # Siempre delante de la principal
        self.after(100, self.grab_set)               # Modal (con un breve retardo: la ventana debe existir)
        ctk.CTkLabel(self, text=str(registro)).pack(padx=20, pady=20)
        ctk.CTkButton(self, text="Cerrar", command=self.destroy).pack()

# Pedir un dato
dialogo = ctk.CTkInputDialog(text="Nombre de la categoría:", title="Nueva categoría")
valor = dialogo.get_input()                          # None si se cancela

# Mensajes y selección de archivos: los de Tkinter
from tkinter import messagebox, filedialog
messagebox.showinfo("Exportación", "Archivo guardado")
if messagebox.askyesno("Confirmar", "¿Eliminar el registro?"): ...
ruta = filedialog.asksaveasfilename(defaultextension=".csv", filetypes=[("CSV", "*.csv")])
```

Para evitar abrir varias copias de la misma ventana secundaria, guarde la referencia y compruebe `ventana.winfo_exists()` antes de crear otra.

---

## 11. Imágenes con `CTkImage`

`CTkImage` recibe imágenes de **Pillow** y las escala correctamente en pantallas de alta densidad; admite una versión para cada modo:

```python
from PIL import Image
logo = ctk.CTkImage(light_image=Image.open("logo_oscuro.png"),
                    dark_image=Image.open("logo_claro.png"), size=(32, 32))
ctk.CTkLabel(app, image=logo, text="  Mi empresa", compound="left").pack()
ctk.CTkButton(app, image=logo, text="", width=40).pack()
```

> Utilice siempre `CTkImage` en *widgets* de CustomTkinter (no `PhotoImage`): de lo contrario aparece un aviso y la imagen no se escala.

---

## 12. Diseño con `grid`

`grid` es el gestor más práctico para interfaces con barra lateral y contenido:

```python
app.grid_columnconfigure(1, weight=1)             # La columna de contenido se expande
app.grid_rowconfigure(0, weight=1)

lateral = ctk.CTkFrame(app, width=180, corner_radius=0)
lateral.grid(row=0, column=0, sticky="nsw")
lateral.grid_rowconfigure(4, weight=1)            # Empuja los últimos controles hacia abajo

contenido = ctk.CTkFrame(app, fg_color="transparent")
contenido.grid(row=0, column=1, sticky="nsew", padx=20, pady=20)
```

| Opción | Efecto |
|---|---|
| `sticky="nsew"` | Ocupa toda la celda. |
| `padx=(izq, der)`, `pady=(arr, abj)` | Márgenes asimétricos. |
| `columnspan`, `rowspan` | Ocupa varias celdas. |
| `columnconfigure(i, weight=1, uniform="g")` | Reparte el espacio sobrante; `uniform` iguala anchos. |
| `grid_remove()` / `grid()` | Ocultar y volver a mostrar conservando la configuración. |

Para cambiar de «página», cree todos los marcos en la misma celda y muestre el activo con `tkraise()`, o use `grid_remove()` en los demás.

---

## 13. Variables, eventos y atajos de teclado

```python
nombre = ctk.StringVar(value="")
ctk.CTkEntry(app, textvariable=nombre)
nombre.trace_add("write", lambda *_: validar())   # Reaccionar a cada cambio

app.bind("<Control-s>", lambda e: guardar())
app.bind("<Escape>", lambda e: app.destroy())
boton.bind("<Enter>", lambda e: estado.configure(text="Guarda el registro"))
app.protocol("WM_DELETE_WINDOW", al_cerrar)       # Confirmar o guardar antes de salir
app.after(1000, refrescar)                        # Ejecutar dentro de 1 s (temporizadores)
```

Variables disponibles: `StringVar`, `IntVar`, `DoubleVar`, `BooleanVar` (reexportadas por `customtkinter`).

---

## 14. Arquitectura orientada a objetos

Para aplicaciones de más de una pantalla, separe:

- **Modelo**: datos y reglas de negocio, sin importar `customtkinter` (testeable).
- **Vistas**: una clase `CTkFrame` por pantalla o componente reutilizable, con un método `refrescar()`.
- **Aplicación**: la ventana `CTk` que crea las vistas, gestiona la navegación y comparte el modelo.

```python
class Tarjeta(ctk.CTkFrame):
    """Componente reutilizable: indicador con título y valor."""
    def __init__(self, master, titulo: str):
        super().__init__(master, corner_radius=12)
        ctk.CTkLabel(self, text=titulo, text_color="gray").pack(anchor="w", padx=14, pady=(10, 0))
        self.valor = ctk.CTkLabel(self, text="—", font=ctk.CTkFont(size=22, weight="bold"))
        self.valor.pack(anchor="w", padx=14, pady=(0, 10))

    def mostrar(self, texto: str) -> None:
        self.valor.configure(text=texto)
```

---

## 15. Tareas largas sin bloquear la interfaz

Tkinter procesa la interfaz en un solo hilo: una operación larga en un `command` congela la ventana. Ejecútela en un **hilo** y comunique el progreso con una **cola** leída mediante `after` (los *widgets* solo deben modificarse desde el hilo principal):

```python
import queue
import threading

class Exportador:
    def __init__(self, app, barra, boton):
        self.app, self.barra, self.boton = app, barra, boton
        self.cola: queue.Queue = queue.Queue()

    def iniciar(self):
        self.boton.configure(state="disabled")
        threading.Thread(target=self._trabajo, daemon=True).start()
        self.app.after(50, self._revisar)

    def _trabajo(self):                           # Hilo secundario: no toca widgets
        for i in range(101):
            time.sleep(0.02)
            self.cola.put(("progreso", i / 100))
        self.cola.put(("fin", "Completado"))

    def _revisar(self):                           # Hilo principal
        try:
            while True:
                tipo, dato = self.cola.get_nowait()
                if tipo == "progreso":
                    self.barra.set(dato)
                else:
                    self.boton.configure(state="normal")
                    return
        except queue.Empty:
            pass
        self.app.after(50, self._revisar)
```

---

## 16. Temas personalizados

Copie un tema incluido (`customtkinter/assets/themes/blue.json`), modifique los colores y cárguelo por ruta:

```python
ctk.set_default_color_theme("temas/corporativo.json")
```

```json
{
  "CTkButton": {
    "corner_radius": 8, "border_width": 0,
    "fg_color": ["#1F4E78", "#2E75B6"], "hover_color": ["#163A5A", "#245C8F"],
    "border_color": ["#3E454A", "#949A9F"], "text_color": ["#FFFFFF", "#FFFFFF"],
    "text_color_disabled": ["gray74", "gray60"]
  }
}
```

> Un tema personalizado debe contener **todas** las claves del original (un `KeyError` al crear un *widget* indica una clave ausente). Lo más seguro es copiar el JSON completo y cambiar solo los valores.

---

## 17. Gráficos con Matplotlib

```python
from matplotlib.figure import Figure
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

class Grafico(ctk.CTkFrame):
    def __init__(self, master):
        super().__init__(master)
        self.figura = Figure(figsize=(5, 3), dpi=100, layout="constrained")
        self.ejes = self.figura.add_subplot()
        self.lienzo = FigureCanvasTkAgg(self.figura, master=self)
        self.lienzo.get_tk_widget().pack(fill="both", expand=True)

    def dibujar(self, etiquetas, valores):
        oscuro = ctk.get_appearance_mode() == "Dark"
        fondo, texto = ("#2B2B2B", "white") if oscuro else ("#FFFFFF", "black")
        self.figura.set_facecolor(fondo)
        self.ejes.clear()
        self.ejes.set_facecolor(fondo)
        self.ejes.barh(etiquetas, valores, color="#2E75B6")
        self.ejes.tick_params(colors=texto)
        self.lienzo.draw_idle()
```

Use `matplotlib.figure.Figure` (no `pyplot`) dentro de aplicaciones gráficas para evitar ventanas adicionales y fugas de memoria.

---

## 18. Combinación con Tkinter y ttk

Los *widgets* estándar pueden colocarse en contenedores de CustomTkinter. El caso más habitual es la tabla `ttk.Treeview`:

```python
from tkinter import ttk

estilo = ttk.Style()
estilo.theme_use("default")
estilo.configure("Treeview", background="#2B2B2B", fieldbackground="#2B2B2B", foreground="white",
                 rowheight=26, borderwidth=0)
estilo.map("Treeview", background=[("selected", "#1F6AA5")])

tabla = ttk.Treeview(marco, columns=("fecha", "concepto", "importe"), show="headings")
for col, ancho in [("fecha", 100), ("concepto", 220), ("importe", 90)]:
    tabla.heading(col, text=col.capitalize())
    tabla.column(col, width=ancho, anchor="e" if col == "importe" else "w")
tabla.insert("", "end", values=("2026-03-01", "Supermercado", "54,20 €"))
```

Los *widgets* de Tkinter no siguen el modo de apariencia automáticamente: actualice su estilo al cambiar de modo.

---

## 19. Distribución como ejecutable

CustomTkinter incluye archivos de datos (temas JSON, fuentes) que deben empaquetarse:

```bash
pip install pyinstaller
pyinstaller --noconfirm --windowed --name ControlGastos --collect-data customtkinter app.py
```

- `--windowed` evita la consola en Windows y macOS.
- `--collect-data customtkinter` (o `--add-data "<ruta>/customtkinter;customtkinter/"`) incluye los temas.
- Recursos propios (iconos, temas): `--add-data "assets:assets"` y localícelos con una ruta relativa al ejecutable (`sys._MEIPASS` cuando está empaquetado).

---

## 20. Ejemplo práctico completo

**Control de gastos personal** con barra lateral de navegación, cuatro pantallas y persistencia en JSON:

- **Panel**: tarjetas de ingresos, gastos y saldo del mes; barra de consumo del presupuesto; gasto por categoría con barras de progreso.
- **Movimientos**: lista desplazable filtrable por texto y tipo, con eliminación y confirmación.
- **Nuevo movimiento**: formulario validado (importe, fecha, categoría con texto libre) y atajo `Ctrl+S`.
- **Ajustes**: presupuesto mensual con deslizador, modo de apariencia, escalado y exportación a CSV en segundo plano con barra de progreso.
- Logotipo generado con Pillow (`CTkImage`), datos guardados en `gastos.json` al cerrar.

```python
"""
control_gastos.py — Aplicación de escritorio de control de gastos con CustomTkinter.
Requiere: pip install customtkinter pillow
Uso: python control_gastos.py
"""
from __future__ import annotations

import csv
import json
import queue
import threading
import time
import uuid
from dataclasses import asdict, dataclass, field
from datetime import date, datetime
from pathlib import Path
from tkinter import messagebox

import customtkinter as ctk
from PIL import Image, ImageDraw

ARCHIVO_DATOS = Path("gastos.json")
CATEGORIAS = ["Alimentación", "Vivienda", "Transporte", "Ocio", "Salud", "Nómina", "Otros"]
VERDE, ROJO, AZUL = ("#067647", "#47CD89"), ("#B42318", "#F97066"), ("#1F4E78", "#5B9BD5")


# ============================================================ Modelo (sin interfaz)
@dataclass
class Movimiento:
    concepto: str
    importe: float
    categoria: str
    tipo: str                      # "Gasto" o "Ingreso"
    fecha: str                     # ISO AAAA-MM-DD
    id: str = field(default_factory=lambda: uuid.uuid4().hex[:8])


class Libro:
    def __init__(self, ruta: Path):
        self.ruta = ruta
        self.presupuesto = 1500.0
        self.movimientos: list[Movimiento] = []
        self.cargar()

    def cargar(self) -> None:
        if not self.ruta.exists():
            self._datos_iniciales()
            return
        datos = json.loads(self.ruta.read_text(encoding="utf-8"))
        self.presupuesto = datos.get("presupuesto", 1500.0)
        self.movimientos = [Movimiento(**m) for m in datos.get("movimientos", [])]

    def guardar(self) -> None:
        datos = {"presupuesto": self.presupuesto, "movimientos": [asdict(m) for m in self.movimientos]}
        self.ruta.write_text(json.dumps(datos, ensure_ascii=False, indent=2), encoding="utf-8")

    def _datos_iniciales(self) -> None:
        hoy = date.today()
        ejemplos = [("Nómina", 2350, "Nómina", "Ingreso", 1), ("Alquiler", 780, "Vivienda", "Gasto", 2),
                    ("Supermercado", 96.4, "Alimentación", "Gasto", 4), ("Abono transporte", 54.6, "Transporte", "Gasto", 5),
                    ("Cine", 18, "Ocio", "Gasto", 7), ("Farmacia", 23.15, "Salud", "Gasto", 9),
                    ("Supermercado", 71.3, "Alimentación", "Gasto", 11), ("Cena con amigos", 42, "Ocio", "Gasto", 12)]
        self.movimientos = [Movimiento(c, i, cat, t, hoy.replace(day=min(d, hoy.day)).isoformat())
                            for c, i, cat, t, d in ejemplos]

    def agregar(self, m: Movimiento) -> None:
        self.movimientos.append(m)

    def eliminar(self, id_: str) -> None:
        self.movimientos = [m for m in self.movimientos if m.id != id_]

    def del_mes(self, referencia: date | None = None) -> list[Movimiento]:
        ref = (referencia or date.today()).isoformat()[:7]
        return [m for m in self.movimientos if m.fecha.startswith(ref)]

    def resumen_mes(self) -> dict:
        mes = self.del_mes()
        ingresos = sum(m.importe for m in mes if m.tipo == "Ingreso")
        gastos = sum(m.importe for m in mes if m.tipo == "Gasto")
        por_categoria: dict[str, float] = {}
        for m in mes:
            if m.tipo == "Gasto":
                por_categoria[m.categoria] = por_categoria.get(m.categoria, 0) + m.importe
        return {"ingresos": ingresos, "gastos": gastos, "saldo": ingresos - gastos,
                "por_categoria": dict(sorted(por_categoria.items(), key=lambda kv: -kv[1]))}

    def filtrar(self, texto: str = "", tipo: str = "Todos") -> list[Movimiento]:
        texto = texto.lower().strip()
        res = [m for m in self.movimientos
               if (tipo == "Todos" or m.tipo == tipo[:-1])                        # "Gastos" → "Gasto"
               and (texto in m.concepto.lower() or texto in m.categoria.lower())]
        return sorted(res, key=lambda m: m.fecha, reverse=True)


def euros(valor: float) -> str:
    return f"{valor:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def crear_logo(tamano: int = 64) -> Image.Image:
    img = Image.new("RGBA", (tamano, tamano), (0, 0, 0, 0))
    d = ImageDraw.Draw(img)
    d.rounded_rectangle((0, 0, tamano - 1, tamano - 1), radius=tamano // 4, fill="#2E75B6")
    for i, alto in enumerate((0.35, 0.6, 0.45, 0.8)):
        x0 = tamano * (0.17 + i * 0.18)
        d.rectangle((x0, tamano * (0.85 - alto * 0.7), x0 + tamano * 0.12, tamano * 0.85), fill="white")
    return img


# ============================================================ Componentes
class Tarjeta(ctk.CTkFrame):
    def __init__(self, master, titulo: str):
        super().__init__(master, corner_radius=12)
        ctk.CTkLabel(self, text=titulo, text_color=("gray40", "gray65")).pack(anchor="w", padx=16, pady=(12, 0))
        self.valor = ctk.CTkLabel(self, text="—", font=ctk.CTkFont(size=24, weight="bold"))
        self.valor.pack(anchor="w", padx=16, pady=(0, 12))

    def mostrar(self, texto: str, color=None) -> None:
        self.valor.configure(text=texto, text_color=color or ("gray10", "gray95"))


class Pagina(ctk.CTkFrame):
    def __init__(self, master, app: "App", titulo: str):
        super().__init__(master, fg_color="transparent")
        self.app = app
        ctk.CTkLabel(self, text=titulo, font=ctk.CTkFont(size=24, weight="bold")).pack(anchor="w", pady=(0, 14))

    def refrescar(self) -> None:
        pass


# ============================================================ Páginas
class PaginaPanel(Pagina):
    def __init__(self, master, app):
        super().__init__(master, app, "Resumen del mes")
        fila = ctk.CTkFrame(self, fg_color="transparent")
        fila.pack(fill="x")
        self.tarjetas = {}
        for i, (clave, titulo) in enumerate([("ingresos", "Ingresos"), ("gastos", "Gastos"), ("saldo", "Saldo")]):
            fila.grid_columnconfigure(i, weight=1, uniform="t")
            self.tarjetas[clave] = Tarjeta(fila, titulo)
            self.tarjetas[clave].grid(row=0, column=i, sticky="ew", padx=(0 if i == 0 else 8, 0))

        caja = ctk.CTkFrame(self, corner_radius=12)
        caja.pack(fill="x", pady=14)
        self.texto_presupuesto = ctk.CTkLabel(caja, text="", anchor="w")
        self.texto_presupuesto.pack(fill="x", padx=16, pady=(12, 4))
        self.barra_presupuesto = ctk.CTkProgressBar(caja, height=14)
        self.barra_presupuesto.pack(fill="x", padx=16, pady=(0, 14))

        self.categorias = ctk.CTkScrollableFrame(self, label_text="Gasto por categoría", corner_radius=12)
        self.categorias.pack(fill="both", expand=True)
        self.categorias.grid_columnconfigure(1, weight=1)

    def refrescar(self) -> None:
        r = self.app.libro.resumen_mes()
        self.tarjetas["ingresos"].mostrar(euros(r["ingresos"]), VERDE)
        self.tarjetas["gastos"].mostrar(euros(r["gastos"]), ROJO)
        self.tarjetas["saldo"].mostrar(euros(r["saldo"]), VERDE if r["saldo"] >= 0 else ROJO)

        presupuesto = self.app.libro.presupuesto
        proporcion = r["gastos"] / presupuesto if presupuesto else 0
        self.texto_presupuesto.configure(
            text=f"Presupuesto: {euros(r['gastos'])} de {euros(presupuesto)} ({proporcion:.0%})")
        self.barra_presupuesto.configure(progress_color=ROJO if proporcion > 0.9 else ("#F79009" if proporcion > 0.7 else VERDE))
        self.barra_presupuesto.set(min(proporcion, 1))

        for w in self.categorias.winfo_children():
            w.destroy()
        maximo = max(r["por_categoria"].values(), default=1)
        for fila, (cat, valor) in enumerate(r["por_categoria"].items()):
            ctk.CTkLabel(self.categorias, text=cat, width=110, anchor="w").grid(row=fila, column=0, padx=8, pady=4, sticky="w")
            barra = ctk.CTkProgressBar(self.categorias, progress_color=AZUL)
            barra.set(valor / maximo)
            barra.grid(row=fila, column=1, sticky="ew", padx=8)
            ctk.CTkLabel(self.categorias, text=euros(valor), width=90, anchor="e").grid(row=fila, column=2, padx=8)
        if not r["por_categoria"]:
            ctk.CTkLabel(self.categorias, text="Sin gastos este mes").grid(row=0, column=0, pady=20)


class PaginaMovimientos(Pagina):
    def __init__(self, master, app):
        super().__init__(master, app, "Movimientos")
        barra = ctk.CTkFrame(self, fg_color="transparent")
        barra.pack(fill="x", pady=(0, 10))
        self.busqueda = ctk.StringVar()
        self.busqueda.trace_add("write", lambda *_: self.refrescar())
        ctk.CTkLabel(barra, text="Buscar:").pack(side="left", padx=(0, 6))       # Sin placeholder: usa textvariable
        ctk.CTkEntry(barra, textvariable=self.busqueda, width=240).pack(side="left")
        self.tipo = ctk.CTkSegmentedButton(barra, values=["Todos", "Gastos", "Ingresos"],
                                           command=lambda _: self.refrescar())
        self.tipo.set("Todos")
        self.tipo.pack(side="left", padx=10)
        self.total = ctk.CTkLabel(barra, text="")
        self.total.pack(side="right")
        self.lista = ctk.CTkScrollableFrame(self, corner_radius=12)
        self.lista.pack(fill="both", expand=True)
        self.lista.grid_columnconfigure(1, weight=1)

    def refrescar(self) -> None:
        for w in self.lista.winfo_children():
            w.destroy()
        movimientos = self.app.libro.filtrar(self.busqueda.get(), self.tipo.get())
        for fila, m in enumerate(movimientos):
            signo, color = ("-", ROJO) if m.tipo == "Gasto" else ("+", VERDE)
            ctk.CTkLabel(self.lista, text=datetime.fromisoformat(m.fecha).strftime("%d/%m/%Y"),
                         width=90, anchor="w").grid(row=fila, column=0, padx=8, pady=3)
            ctk.CTkLabel(self.lista, text=f"{m.concepto}  ·  {m.categoria}", anchor="w").grid(row=fila, column=1, sticky="ew")
            ctk.CTkLabel(self.lista, text=f"{signo}{euros(m.importe)}", text_color=color, width=110,
                         anchor="e").grid(row=fila, column=2, padx=8)
            ctk.CTkButton(self.lista, text="✕", width=30, height=26, fg_color="transparent",
                          text_color=ROJO, hover_color=("gray85", "gray25"),
                          command=lambda id_=m.id, c=m.concepto: self.eliminar(id_, c)).grid(row=fila, column=3, padx=4)
        neto = sum(m.importe if m.tipo == "Ingreso" else -m.importe for m in movimientos)
        self.total.configure(text=f"{len(movimientos)} movimientos · neto {euros(neto)}")

    def eliminar(self, id_: str, concepto: str) -> None:
        if messagebox.askyesno("Eliminar", f"¿Eliminar «{concepto}»?", parent=self):
            self.app.libro.eliminar(id_)
            self.app.datos_cambiados()


class PaginaNuevo(Pagina):
    def __init__(self, master, app):
        super().__init__(master, app, "Nuevo movimiento")
        form = ctk.CTkFrame(self, corner_radius=12)
        form.pack(fill="x")
        form.grid_columnconfigure(1, weight=1)

        self.tipo = ctk.CTkSegmentedButton(form, values=["Gasto", "Ingreso"])
        self.tipo.set("Gasto")
        self.concepto = ctk.CTkEntry(form, placeholder_text="Ej.: Supermercado")
        self.importe = ctk.CTkEntry(form, placeholder_text="0,00")
        self.fecha = ctk.CTkEntry(form, placeholder_text="dd/mm/aaaa")
        self.categoria = ctk.CTkComboBox(form, values=CATEGORIAS)
        self.categoria.set("Alimentación")
        campos = [("Tipo", self.tipo), ("Concepto", self.concepto), ("Importe (€)", self.importe),
                  ("Fecha", self.fecha), ("Categoría", self.categoria)]
        for fila, (texto, widget) in enumerate(campos):
            ctk.CTkLabel(form, text=texto, anchor="w", width=100).grid(row=fila, column=0, padx=16, pady=8, sticky="w")
            widget.grid(row=fila, column=1, padx=16, pady=8, sticky="ew")

        self.mensaje = ctk.CTkLabel(self, text="", anchor="w")
        self.mensaje.pack(fill="x", pady=10)
        botones = ctk.CTkFrame(self, fg_color="transparent")
        botones.pack(fill="x")
        self.boton_guardar = ctk.CTkButton(botones, text="Guardar  (Ctrl+S)", command=self.guardar)
        self.boton_guardar.pack(side="left")
        ctk.CTkButton(botones, text="Limpiar", fg_color="transparent", border_width=1,
                      text_color=("gray10", "gray90"), command=self.limpiar).pack(side="left", padx=10)
        self.limpiar()

    def _error(self, widget, texto: str) -> None:
        widget.configure(border_color=ROJO)
        self.mensaje.configure(text=f"⚠ {texto}", text_color=ROJO)
        widget.focus_set()

    def guardar(self) -> bool:
        for w in (self.concepto, self.importe, self.fecha):
            w.configure(border_color=("#979DA2", "#565B5E"))
        concepto = self.concepto.get().strip()
        if len(concepto) < 2:
            self._error(self.concepto, "Indique un concepto (mínimo 2 caracteres).")
            return False
        try:
            texto = self.importe.get().strip()
            if "," in texto:                                   # Formato español: 1.234,50
                texto = texto.replace(".", "").replace(",", ".")
            importe = float(texto)
            if importe <= 0:
                raise ValueError
        except ValueError:
            self._error(self.importe, "El importe debe ser un número positivo (ej.: 12,50).")
            return False
        try:
            fecha = datetime.strptime(self.fecha.get().strip(), "%d/%m/%Y").date()
        except ValueError:
            self._error(self.fecha, "Fecha no válida. Formato: dd/mm/aaaa.")
            return False
        categoria = self.categoria.get().strip().capitalize() or "Otros"

        self.app.libro.agregar(Movimiento(concepto, round(importe, 2), categoria, self.tipo.get(), fecha.isoformat()))
        if categoria not in self.categoria.cget("values"):
            self.categoria.configure(values=[*self.categoria.cget("values"), categoria])
        self.app.datos_cambiados()
        self.limpiar()
        self.mensaje.configure(text=f"✔ Guardado: {concepto} · {euros(importe)}", text_color=VERDE)
        return True

    def limpiar(self) -> None:
        self.concepto.delete(0, "end")
        self.importe.delete(0, "end")
        self.fecha.delete(0, "end")
        self.fecha.insert(0, date.today().strftime("%d/%m/%Y"))
        self.mensaje.configure(text="")
        self.concepto.focus_set()


class PaginaAjustes(Pagina):
    def __init__(self, master, app):
        super().__init__(master, app, "Ajustes")
        caja = ctk.CTkFrame(self, corner_radius=12)
        caja.pack(fill="x")
        caja.grid_columnconfigure(1, weight=1)

        ctk.CTkLabel(caja, text="Presupuesto mensual").grid(row=0, column=0, padx=16, pady=12, sticky="w")
        self.presupuesto = ctk.CTkSlider(caja, from_=500, to=5000, number_of_steps=90, command=self._presupuesto)
        self.presupuesto.grid(row=0, column=1, sticky="ew", padx=8)
        self.texto_presupuesto = ctk.CTkLabel(caja, text="", width=100)
        self.texto_presupuesto.grid(row=0, column=2, padx=16)

        ctk.CTkLabel(caja, text="Apariencia").grid(row=1, column=0, padx=16, pady=12, sticky="w")
        self.modo = ctk.CTkOptionMenu(caja, values=["Sistema", "Claro", "Oscuro"], command=self._modo)
        self.modo.grid(row=1, column=1, sticky="w", padx=8)

        ctk.CTkLabel(caja, text="Escala").grid(row=2, column=0, padx=16, pady=12, sticky="w")
        escala = ctk.CTkSegmentedButton(caja, values=["90 %", "100 %", "120 %"],
                                        command=lambda v: ctk.set_widget_scaling(int(v[:-2]) / 100))
        escala.set("100 %")
        escala.grid(row=2, column=1, sticky="w", padx=8)

        exportar = ctk.CTkFrame(self, corner_radius=12)
        exportar.pack(fill="x", pady=14)
        ctk.CTkLabel(exportar, text="Exportar todos los movimientos a CSV").pack(anchor="w", padx=16, pady=(12, 6))
        self.boton_exportar = ctk.CTkButton(exportar, text="Exportar", command=self.exportar)
        self.boton_exportar.pack(anchor="w", padx=16)
        self.progreso = ctk.CTkProgressBar(exportar)
        self.progreso.set(0)
        self.progreso.pack(fill="x", padx=16, pady=10)
        self.estado = ctk.CTkLabel(exportar, text="", anchor="w")
        self.estado.pack(fill="x", padx=16, pady=(0, 12))
        self.cola: queue.Queue = queue.Queue()

    def refrescar(self) -> None:
        self.presupuesto.set(self.app.libro.presupuesto)
        self.texto_presupuesto.configure(text=euros(self.app.libro.presupuesto))

    def _presupuesto(self, valor: float) -> None:
        self.app.libro.presupuesto = round(valor)
        self.texto_presupuesto.configure(text=euros(round(valor)))
        self.app.pendiente_guardar = True

    def _modo(self, valor: str) -> None:
        ctk.set_appearance_mode({"Sistema": "system", "Claro": "light", "Oscuro": "dark"}[valor])

    # --- Exportación en segundo plano
    def exportar(self, destino: Path = Path("movimientos.csv")) -> None:
        self.boton_exportar.configure(state="disabled")
        self.estado.configure(text="Exportando…", text_color=("gray10", "gray90"))
        filas = [asdict(m) for m in self.app.libro.movimientos]          # Copia en el hilo principal
        threading.Thread(target=self._hilo_exportar, args=(filas, destino), daemon=True).start()
        self.after(50, self._revisar_cola)

    def _hilo_exportar(self, filas: list[dict], destino: Path) -> None:
        try:
            with destino.open("w", newline="", encoding="utf-8-sig") as f:
                escritor = csv.DictWriter(f, fieldnames=["fecha", "tipo", "concepto", "categoria", "importe", "id"],
                                          delimiter=";")
                escritor.writeheader()
                for i, fila in enumerate(filas, start=1):
                    escritor.writerow(fila)
                    time.sleep(0.03)                                     # Simula un proceso lento
                    self.cola.put(("progreso", i / len(filas)))
            self.cola.put(("fin", f"✔ {len(filas)} movimientos exportados a {destino.resolve()}"))
        except OSError as e:
            self.cola.put(("error", f"⚠ No se pudo exportar: {e}"))

    def _revisar_cola(self) -> None:
        try:
            while True:
                tipo, dato = self.cola.get_nowait()
                if tipo == "progreso":
                    self.progreso.set(dato)
                else:
                    self.estado.configure(text=dato, text_color=VERDE if tipo == "fin" else ROJO)
                    self.boton_exportar.configure(state="normal")
                    return
        except queue.Empty:
            pass
        self.after(50, self._revisar_cola)


# ============================================================ Aplicación
class App(ctk.CTk):
    PAGINAS = [("panel", "Panel", PaginaPanel), ("movimientos", "Movimientos", PaginaMovimientos),
               ("nuevo", "Nuevo", PaginaNuevo), ("ajustes", "Ajustes", PaginaAjustes)]

    def __init__(self, ruta_datos: Path = ARCHIVO_DATOS):
        super().__init__()
        self.title("Control de gastos")
        self.geometry("980x640")
        self.minsize(820, 560)
        self.libro = Libro(ruta_datos)
        self.pendiente_guardar = False

        self.grid_columnconfigure(1, weight=1)
        self.grid_rowconfigure(0, weight=1)

        lateral = ctk.CTkFrame(self, width=190, corner_radius=0)
        lateral.grid(row=0, column=0, sticky="nsw")
        lateral.grid_rowconfigure(len(self.PAGINAS) + 1, weight=1)
        self.logo = ctk.CTkImage(crear_logo(), size=(32, 32))
        ctk.CTkLabel(lateral, image=self.logo, text="  Mis gastos", compound="left",
                     font=ctk.CTkFont(size=18, weight="bold")).grid(row=0, column=0, padx=20, pady=(22, 26), sticky="w")

        contenedor = ctk.CTkFrame(self, fg_color="transparent")
        contenedor.grid(row=0, column=1, sticky="nsew", padx=26, pady=22)
        contenedor.grid_columnconfigure(0, weight=1)
        contenedor.grid_rowconfigure(0, weight=1)

        self.paginas: dict[str, Pagina] = {}
        self.botones: dict[str, ctk.CTkButton] = {}
        for fila, (clave, texto, clase) in enumerate(self.PAGINAS, start=1):
            pagina = clase(contenedor, self)
            pagina.grid(row=0, column=0, sticky="nsew")
            self.paginas[clave] = pagina
            boton = ctk.CTkButton(lateral, text=texto, anchor="w", height=38, corner_radius=8,
                                  fg_color="transparent", text_color=("gray10", "gray90"),
                                  hover_color=("gray80", "gray28"), command=lambda c=clave: self.mostrar(c))
            boton.grid(row=fila, column=0, padx=12, pady=3, sticky="ew")
            self.botones[clave] = boton
        ctk.CTkLabel(lateral, text="Ctrl+1…4 para navegar", text_color="gray55",
                     font=ctk.CTkFont(size=11)).grid(row=len(self.PAGINAS) + 2, column=0, pady=16)

        for i, (clave, *_rest) in enumerate(self.PAGINAS, start=1):
            self.bind(f"<Control-Key-{i}>", lambda e, c=clave: self.mostrar(c))
        self.bind("<Control-s>", lambda e: self.actual == "nuevo" and self.paginas["nuevo"].guardar())
        self.protocol("WM_DELETE_WINDOW", self.cerrar)
        self.actual = ""
        self.mostrar("panel")

    def mostrar(self, clave: str) -> None:
        for c, b in self.botones.items():
            b.configure(fg_color=("gray75", "gray25") if c == clave else "transparent")
        self.paginas[clave].refrescar()
        self.paginas[clave].tkraise()
        self.actual = clave

    def datos_cambiados(self) -> None:
        self.pendiente_guardar = True
        self.paginas[self.actual].refrescar()

    def cerrar(self) -> None:
        if self.pendiente_guardar:
            self.libro.guardar()
        self.destroy()


if __name__ == "__main__":
    ctk.set_appearance_mode("system")
    ctk.set_default_color_theme("blue")
    App().mainloop()
```

Puntos destacados del ejemplo:

- El **modelo** (`Libro`, `Movimiento`) no depende de la interfaz y puede probarse por separado.
- Cada pantalla es una subclase de `Pagina` con `refrescar()`; la navegación usa `tkraise()` sobre marcos superpuestos en la misma celda.
- La exportación se ejecuta en un hilo y comunica el progreso mediante una cola revisada con `after`.
- Los colores son tuplas `(claro, oscuro)`, de modo que todo se adapta al cambiar de apariencia.

---

## 21. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'tkinter'` | Python sin Tk (Linux). | `sudo apt install python3-tk` o un Python con Tk. |
| `ValueError: 'fg' is not a supported argument` / `bg` | Parámetros de Tkinter. | `fg_color`, `text_color`, `bg_color`. |
| `CTkLabel Warning: Given image is not CTkImage` | Uso de `PhotoImage`. | Use `ctk.CTkImage(Image.open(...), size=...)`. |
| El *placeholder* no aparece | Hay `textvariable` asignado. | Elija uno de los dos o gestione la variable. |
| La ventana se congela | Tarea larga en el hilo de la interfaz. | Hilo + cola + `after`. |
| `RuntimeError: main thread is not in main loop` | Se modifican *widgets* desde otro hilo. | Solo el hilo principal toca la interfaz. |
| `RuntimeError: Too early to create font` | `CTkFont` creada antes que la ventana. | Cree la ventana `CTk` primero. |
| `KeyError` al cargar un tema personalizado | Faltan claves en el JSON. | Parta de una copia completa de un tema incluido. |
| La ventana secundaria aparece detrás o `grab_set` falla | La ventana aún no está visible. | `transient(padre)` y `after(100, self.grab_set)`. |
| Se abren varias ventanas iguales | Se crea una nueva en cada clic. | Guarde la referencia y compruebe `winfo_exists()`. |
| En un ejecutable falta el tema | No se empaquetaron los datos. | `--collect-data customtkinter`. |
| `lambda` en bucle ejecuta siempre el último elemento | Captura tardía de variables. | `lambda x=x: funcion(x)`. |

---

## 22. Buenas prácticas

- Establezca apariencia y tema antes de crear la ventana principal.
- Estructure la aplicación en clases: una ventana, vistas como `CTkFrame` y un modelo independiente de la interfaz.
- Use tuplas `(claro, oscuro)` para todo color personalizado.
- Prefiera `grid` con `weight` para diseños que se adapten al tamaño de la ventana.
- No bloquee el bucle principal: hilos para trabajo pesado y `after` para actualizar.
- Valide los formularios y muestre los errores junto al campo (borde y mensaje), no solo en ventanas emergentes.
- Ofrezca atajos de teclado para las acciones frecuentes y gestione el cierre con `WM_DELETE_WINDOW`.
- Para tablas o menús complejos, combine con ttk (`Treeview`) o valore PySide6 si la aplicación crece mucho.

---

## 23. Referencia rápida

```python
ctk.set_appearance_mode("system"|"light"|"dark") · ctk.set_default_color_theme("blue"|ruta.json) · ctk.set_widget_scaling(1.2)
app = ctk.CTk(); app.title(); app.geometry("800x600"); app.minsize(); app.mainloop()
CTkButton(m, text=, command=, image=, fg_color=, hover_color=) · CTkLabel(m, text=, font=, text_color=, anchor=)
CTkEntry(m, placeholder_text=, show=, textvariable=) .get() .delete(0,"end") .insert(0, t)
CTkTextbox(m) .insert("end", t) .get("1.0","end-1c") · CTkCheckBox · CTkSwitch · CTkRadioButton(variable=, value=)
CTkOptionMenu(values=, command=) · CTkComboBox(values=) · CTkSegmentedButton(values=) .set() .get()
CTkSlider(from_=, to=, number_of_steps=, command=) · CTkProgressBar() .set(0..1) mode="indeterminate" .start()
CTkFrame(corner_radius=, fg_color="transparent") · CTkScrollableFrame(label_text=) · CTkTabview() .add() .tab() .set()
CTkToplevel(padre) · CTkInputDialog(text=, title=).get_input() · CTkImage(light_image=, dark_image=, size=) · CTkFont(size=, weight=)
widget.configure(...) · widget.cget("text") · app.after(ms, f) · app.bind("<Control-s>", f) · app.protocol("WM_DELETE_WINDOW", f)
```

---

## 24. Ejercicios propuestos

1. Añada a la pantalla de movimientos un filtro por mes con `CTkOptionMenu` y un botón para editar un movimiento en una `CTkToplevel` modal.
2. Incorpore al panel un gráfico de barras de Matplotlib que respete el modo claro/oscuro.
3. Cree un tema corporativo en JSON y permita elegirlo desde Ajustes (requiere reiniciar las vistas).
4. Sustituya la lista de movimientos por un `ttk.Treeview` con ordenación al pulsar los encabezados.
5. Empaquete la aplicación con PyInstaller y guarde los datos en la carpeta de usuario (`Path.home()`).

---

## 25. Referencias

- Documentación oficial: https://customtkinter.tomschimansky.com/documentation/
- Tutoriales oficiales: https://customtkinter.tomschimansky.com/tutorial/
- Repositorio y ejemplos: https://github.com/TomSchimansky/CustomTkinter
- Documentación de Tkinter (base de CustomTkinter): https://docs.python.org/3/library/tkinter.html
