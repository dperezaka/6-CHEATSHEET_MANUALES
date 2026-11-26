# Manual de uso: Tkinter

> **Categoría:** Interfaces gráficas (GUI) · **Utilidad:** Crear interfaces gráficas sencillas
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [La ventana principal](#4-la-ventana-principal)
5. [Widgets básicos](#5-widgets-básicos)
6. [Variables de control](#6-variables-de-control)
7. [Gestores de geometría](#7-gestores-de-geometría)
8. [Eventos y atajos de teclado](#8-eventos-y-atajos-de-teclado)
9. [Widgets avanzados](#9-widgets-avanzados)
10. [Menús, diálogos y ventanas secundarias](#10-menús-diálogos-y-ventanas-secundarias)
11. [Estilos y temas (ttk)](#11-estilos-y-temas-ttk)
12. [Canvas, imágenes y gráficos](#12-canvas-imágenes-y-gráficos)
13. [Tareas largas sin bloquear la interfaz](#13-tareas-largas-sin-bloquear-la-interfaz)
14. [Estructura de una aplicación](#14-estructura-de-una-aplicación)
15. [Distribuir la aplicación](#15-distribuir-la-aplicación)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Tkinter** es la librería estándar de Python para crear interfaces gráficas de escritorio. Es una capa sobre **Tk**, un conjunto de herramientas gráficas multiplataforma, y viene incluida con Python en Windows y macOS.

**Ventajas:**
- No requiere instalación.
- Funciona en Windows, macOS y Linux.
- Curva de aprendizaje suave.
- Suficiente para herramientas internas, utilidades, formularios y prototipos.
- El submódulo `ttk` ofrece widgets con aspecto nativo.

**Limitaciones:**
- El aspecto por defecto es menos moderno que el de Qt o las aplicaciones web (se mejora con `ttk`, temas o **CustomTkinter**).
- Menos widgets avanzados que PyQt/PySide (no hay tablas editables, gráficos o navegador integrados).

| Si necesita… | Opción |
|---|---|
| Herramienta interna sencilla, sin dependencias | **Tkinter** |
| Aspecto moderno con la misma lógica | CustomTkinter |
| Aplicación profesional compleja | PySide6 / PyQt6 |
| Aplicación táctil o móvil | Kivy |
| Paneles de datos y herramientas en tiempo real | Dear PyGui |

---

## 2. Instalación e importación

No requiere instalación en Windows ni macOS (instaladores de python.org).

En Linux puede ser necesario instalar el paquete del sistema:

```bash
sudo apt install python3-tk        # Debian / Ubuntu
sudo dnf install python3-tkinter   # Fedora
```

Comprobar que funciona:

```bash
python -m tkinter                  # Abre una ventana de prueba
```

Importaciones habituales:

```python
import tkinter as tk
from tkinter import ttk, messagebox, filedialog, simpledialog, colorchooser, font
```

> `tk` contiene los widgets clásicos; `ttk` (*themed Tk*) los widgets con estilo nativo. **Use `ttk` siempre que exista el widget equivalente.**

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Ventana raíz | `tk.Tk()`: ventana principal. Solo debe existir una por aplicación. |
| Widget | Elemento visual: etiqueta, botón, campo de texto, lista… |
| Contenedor | Widget que agrupa a otros (`Frame`, `LabelFrame`, ventana). |
| Gestor de geometría | Coloca los widgets: `pack`, `grid` o `place`. |
| Variable de control | `StringVar`, `IntVar`, `DoubleVar`, `BooleanVar` enlazadas a widgets. |
| Evento | Acción del usuario (clic, tecla, cambio de tamaño) que dispara una función. |
| Bucle principal | `mainloop()`: mantiene la ventana abierta y procesa los eventos. |
| Opciones | Propiedades de un widget (`text`, `width`, `command`…), modificables con `config()`. |

**Jerarquía:** todo widget tiene un **padre** (la ventana o un contenedor) que se indica como primer argumento:

```python
marco = ttk.Frame(root)               # Padre: la ventana
boton = ttk.Button(marco, text="OK")  # Padre: el marco
```

---

## 4. La ventana principal

### 4.1 Ventana mínima

```python
import tkinter as tk

root = tk.Tk()
root.title("Mi aplicación")
root.geometry("640x420")          # ancho x alto
root.mainloop()
```

### 4.2 Configuración de la ventana

```python
root.geometry("800x600+100+50")   # Tamaño y posición (x=100, y=50)
root.minsize(500, 300)
root.maxsize(1600, 1000)
root.resizable(True, False)       # Redimensionable en ancho, no en alto
root.attributes("-topmost", True) # Siempre encima
root.attributes("-alpha", 0.95)   # Transparencia (Windows/macOS)
root.state("zoomed")              # Maximizada (Windows)
root.iconbitmap("icono.ico")      # Icono (Windows)
root.iconphoto(True, tk.PhotoImage(file="icono.png"))   # Icono multiplataforma
root.configure(bg="#F5F5F5")
```

### 4.3 Centrar la ventana

```python
def centrar(ventana: tk.Misc, ancho: int, alto: int) -> None:
    ventana.update_idletasks()
    x = (ventana.winfo_screenwidth() - ancho) // 2
    y = (ventana.winfo_screenheight() - alto) // 3
    ventana.geometry(f"{ancho}x{alto}+{x}+{y}")
```

### 4.4 Confirmar el cierre

```python
def al_cerrar():
    if messagebox.askokcancel("Salir", "¿Desea cerrar la aplicación?"):
        root.destroy()

root.protocol("WM_DELETE_WINDOW", al_cerrar)
```

### 4.5 Nitidez en pantallas de alta resolución (Windows)

Sin esto, la ventana puede verse borrosa con escalado del 125 % o 150 %:

```python
import sys
if sys.platform == "win32":
    import ctypes
    try:
        ctypes.windll.shcore.SetProcessDpiAwareness(1)
    except (AttributeError, OSError):
        pass
```

> Ejecute este código **antes** de crear `tk.Tk()`.

---

## 5. Widgets básicos

### 5.1 Etiqueta (`Label`)

```python
ttk.Label(root, text="Nombre del cliente:")
ttk.Label(root, text="Título", font=("Segoe UI", 16, "bold"), foreground="#1F4E78")
ttk.Label(root, text="Texto largo que se ajusta...", wraplength=300, justify="left")
ttk.Label(root, image=imagen)                         # Imagen (ver sección 12)
```

### 5.2 Botón (`Button`)

```python
def guardar():
    print("Guardando…")

ttk.Button(root, text="Guardar", command=guardar)
ttk.Button(root, text="Eliminar", command=lambda: eliminar(registro_id))   # Con argumentos
boton.config(state="disabled")      # Desactivar
boton.config(state="normal")        # Activar
boton.state(["disabled"]); boton.state(["!disabled"])   # Alternativa ttk
```

### 5.3 Campo de texto (`Entry`)

```python
entrada = ttk.Entry(root, width=30)
entrada.insert(0, "Texto inicial")
texto = entrada.get()
entrada.delete(0, tk.END)            # Vaciar
entrada.focus()                      # Poner el cursor
ttk.Entry(root, show="•")            # Contraseña
```

### 5.4 Texto multilínea (`Text`)

```python
texto = tk.Text(root, height=8, width=50, wrap="word", font=("Consolas", 10), undo=True)
texto.insert("1.0", "Primera línea\nSegunda línea")
contenido = texto.get("1.0", "end-1c")      # Todo el texto (sin el salto final)
texto.delete("1.0", tk.END)
texto.config(state="disabled")              # Solo lectura
```

> Los índices de `Text` tienen formato `"línea.columna"` (la primera línea es 1, la primera columna 0).

Formato con etiquetas:

```python
texto.tag_configure("error", foreground="red", font=("Segoe UI", 10, "bold"))
texto.insert(tk.END, "Error: archivo no encontrado\n", "error")
```

### 5.5 Casilla de verificación (`Checkbutton`)

```python
acepta = tk.BooleanVar(value=False)
ttk.Checkbutton(root, text="Acepto las condiciones", variable=acepta,
                command=lambda: print(acepta.get()))
```

### 5.6 Botones de opción (`Radiobutton`)

```python
tipo = tk.StringVar(value="particular")
ttk.Radiobutton(root, text="Particular", variable=tipo, value="particular")
ttk.Radiobutton(root, text="Empresa", variable=tipo, value="empresa")
```

### 5.7 Lista desplegable (`Combobox`)

```python
ciudad = ttk.Combobox(root, values=["Madrid", "Barcelona", "Sevilla"], state="readonly")
ciudad.current(0)                                 # Seleccionar el primero
ciudad.get()
ciudad.bind("<<ComboboxSelected>>", lambda e: print(ciudad.get()))
```

### 5.8 Selector numérico (`Spinbox`) y deslizador (`Scale`)

```python
ttk.Spinbox(root, from_=1, to=100, increment=1, width=6)
ttk.Spinbox(root, values=("Baja", "Media", "Alta"), state="readonly")
ttk.Scale(root, from_=0, to=100, orient="horizontal", length=200,
          command=lambda v: print(round(float(v))))
```

### 5.9 Barra de progreso

```python
progreso = ttk.Progressbar(root, length=300, mode="determinate", maximum=100)
progreso["value"] = 45
ttk.Progressbar(root, mode="indeterminate").start(10)     # Animación continua
```

### 5.10 Contenedores

```python
marco = ttk.Frame(root, padding=10)
grupo = ttk.LabelFrame(root, text="Datos del cliente", padding=10)
separador = ttk.Separator(root, orient="horizontal")
```

### 5.11 Opciones comunes y `config()`

```python
etiqueta = ttk.Label(root, text="Inicial")
etiqueta.config(text="Actualizado")
etiqueta["text"] = "Otra forma"
etiqueta.cget("text")
etiqueta.configure(cursor="hand2")              # Cursor de mano
```

---

## 6. Variables de control

Enlazan un valor con uno o varios widgets: al cambiar la variable, el widget se actualiza, y viceversa.

| Clase | Tipo |
|---|---|
| `tk.StringVar` | Texto |
| `tk.IntVar` | Entero |
| `tk.DoubleVar` | Decimal |
| `tk.BooleanVar` | Booleano |

```python
nombre = tk.StringVar(value="Ana")
ttk.Entry(root, textvariable=nombre)
ttk.Label(root, textvariable=nombre)         # Se actualiza automáticamente

nombre.get()
nombre.set("Luis")
```

### 6.1 Reaccionar a cambios

```python
def al_cambiar(*_):
    boton_guardar.config(state="normal" if nombre.get().strip() else "disabled")

nombre.trace_add("write", al_cambiar)
```

### 6.2 Validar la entrada

```python
def solo_numeros(propuesto: str) -> bool:
    return propuesto == "" or propuesto.replace(",", "", 1).isdigit()

vcmd = (root.register(solo_numeros), "%P")      # %P = valor que tendría el campo
ttk.Entry(root, validate="key", validatecommand=vcmd)
```

| Código | Significado |
|---|---|
| `%P` | Valor si se acepta el cambio |
| `%s` | Valor actual |
| `%S` | Texto insertado o eliminado |
| `%d` | Acción (1 insertar, 0 borrar) |

> `IntVar.get()` lanza `TclError` si el campo contiene texto no numérico: valide o capture la excepción.

---

## 7. Gestores de geometría

Hay tres gestores. **No mezcle `pack` y `grid` en el mismo contenedor.**

### 7.1 `pack`: apilar

```python
ttk.Label(root, text="Arriba").pack(side="top", pady=5)
ttk.Button(root, text="Izquierda").pack(side="left", padx=5)
ttk.Button(root, text="Derecha").pack(side="right", padx=5)

contenido = ttk.Frame(root)
contenido.pack(fill="both", expand=True, padx=10, pady=10)     # Ocupa el espacio disponible
```

| Opción | Efecto |
|---|---|
| `side` | `top`, `bottom`, `left`, `right` |
| `fill` | `x`, `y`, `both`: se estira para ocupar el espacio |
| `expand` | Reclama el espacio sobrante |
| `padx`, `pady` | Margen exterior (número o tupla `(izq, der)`) |
| `ipadx`, `ipady` | Margen interior |
| `anchor` | Alineación: `n`, `s`, `e`, `w`, `center`… |

### 7.2 `grid`: rejilla (ideal para formularios)

```python
ttk.Label(form, text="Nombre:").grid(row=0, column=0, sticky="w", padx=5, pady=5)
ttk.Entry(form).grid(row=0, column=1, sticky="ew", padx=5, pady=5)
ttk.Label(form, text="Email:").grid(row=1, column=0, sticky="w", padx=5, pady=5)
ttk.Entry(form).grid(row=1, column=1, sticky="ew", padx=5, pady=5)
ttk.Button(form, text="Guardar").grid(row=2, column=0, columnspan=2, pady=10)

form.columnconfigure(1, weight=1)          # La columna 1 se estira al redimensionar
```

| Opción | Efecto |
|---|---|
| `row`, `column` | Posición |
| `rowspan`, `columnspan` | Ocupar varias filas o columnas |
| `sticky` | Pegar a los bordes: `n`, `s`, `e`, `w`, `ew`, `nsew` |
| `padx`, `pady` | Márgenes |

`columnconfigure` / `rowconfigure` con `weight` determinan qué filas y columnas crecen al ampliar la ventana.

### 7.3 `place`: posición absoluta

```python
ttk.Label(root, text="Fijo").place(x=20, y=40)
ttk.Label(root, text="Centrado").place(relx=0.5, rely=0.5, anchor="center")
```

> Úselo solo en casos concretos (superposiciones, pantallas de bienvenida): no se adapta bien al tamaño de la ventana.

### 7.4 Combinar gestores con marcos

```python
cabecera = ttk.Frame(root); cabecera.pack(fill="x")
cuerpo = ttk.Frame(root); cuerpo.pack(fill="both", expand=True)
pie = ttk.Frame(root); pie.pack(fill="x")

# Dentro de "cuerpo" se puede usar grid
ttk.Label(cuerpo, text="Campo").grid(row=0, column=0)
```

### 7.5 Paneles redimensionables

```python
paneles = ttk.PanedWindow(root, orient="horizontal")
paneles.pack(fill="both", expand=True)
paneles.add(ttk.Frame(paneles, width=200), weight=1)
paneles.add(ttk.Frame(paneles, width=500), weight=3)
```

---

## 8. Eventos y atajos de teclado

### 8.1 `bind`

```python
def al_pulsar(evento):
    print(evento.widget, evento.x, evento.y, evento.keysym)

widget.bind("<Button-1>", al_pulsar)          # Clic izquierdo
widget.bind("<Double-Button-1>", al_pulsar)   # Doble clic
widget.bind("<Button-3>", al_pulsar)          # Clic derecho (Windows/Linux)
widget.bind("<Enter>", al_pulsar)             # El ratón entra
widget.bind("<Leave>", al_pulsar)
widget.bind("<KeyRelease>", al_pulsar)        # Al soltar una tecla
widget.bind("<FocusIn>", al_pulsar); widget.bind("<FocusOut>", al_pulsar)
root.bind("<Configure>", al_pulsar)           # Cambio de tamaño o posición
root.bind("<MouseWheel>", al_pulsar)          # Rueda (Windows/macOS)
```

### 8.2 Atajos de teclado

```python
root.bind("<Return>", lambda e: guardar())
root.bind("<Escape>", lambda e: root.destroy())
root.bind("<Control-s>", lambda e: guardar())
root.bind("<Control-n>", lambda e: nuevo())
root.bind("<F5>", lambda e: recargar())
root.bind_all("<Control-q>", lambda e: root.quit())     # En toda la aplicación
```

> En macOS use `<Command-s>`. Una solución portable: `MOD = "Command" if sys.platform == "darwin" else "Control"` y `f"<{MOD}-s>"`.

### 8.3 Eventos virtuales

```python
combo.bind("<<ComboboxSelected>>", ...)
arbol.bind("<<TreeviewSelect>>", ...)
cuaderno.bind("<<NotebookTabChanged>>", ...)
lista.bind("<<ListboxSelect>>", ...)
```

### 8.4 Tareas programadas con `after`

```python
def actualizar_reloj():
    reloj.config(text=time.strftime("%H:%M:%S"))
    root.after(1000, actualizar_reloj)       # Repetir cada segundo

actualizar_reloj()

id_tarea = root.after(5000, mostrar_aviso)   # Una vez, a los 5 s
root.after_cancel(id_tarea)                  # Cancelar
```

---

## 9. Widgets avanzados

### 9.1 Tabla (`Treeview`)

```python
columnas = ("codigo", "nombre", "precio", "stock")
tabla = ttk.Treeview(root, columns=columnas, show="headings", height=12, selectmode="browse")

for col, titulo, ancho, alin in [("codigo", "Código", 80, "w"), ("nombre", "Nombre", 220, "w"),
                                  ("precio", "Precio", 90, "e"), ("stock", "Stock", 70, "e")]:
    tabla.heading(col, text=titulo)
    tabla.column(col, width=ancho, anchor=alin)

tabla.insert("", tk.END, iid="TEC-01", values=("TEC-01", "Teclado", "59,90 €", 120))

barra = ttk.Scrollbar(root, orient="vertical", command=tabla.yview)
tabla.configure(yscrollcommand=barra.set)
tabla.grid(row=0, column=0, sticky="nsew"); barra.grid(row=0, column=1, sticky="ns")
```

Operaciones frecuentes:

```python
seleccion = tabla.selection()                 # iids seleccionados
valores = tabla.item(seleccion[0], "values")
tabla.item("TEC-01", values=("TEC-01", "Teclado RGB", "64,90 €", 110))
tabla.delete("TEC-01")
tabla.delete(*tabla.get_children())           # Vaciar
tabla.bind("<<TreeviewSelect>>", al_seleccionar)
tabla.bind("<Double-1>", al_editar)

# Colores por fila
tabla.tag_configure("sin_stock", foreground="#C00000")
tabla.tag_configure("par", background="#F2F2F2")
tabla.insert("", tk.END, values=(...), tags=("sin_stock",))
```

Ordenar al hacer clic en la cabecera:

```python
def ordenar(tabla, col, descendente=False):
    datos = [(tabla.set(i, col), i) for i in tabla.get_children("")]
    def clave(x):
        v = x[0].replace("€", "").replace(".", "").replace(",", ".").strip()
        try:
            return (0, float(v))
        except ValueError:
            return (1, x[0].lower())
    datos.sort(key=clave, reverse=descendente)
    for pos, (_, iid) in enumerate(datos):
        tabla.move(iid, "", pos)
    tabla.heading(col, command=lambda: ordenar(tabla, col, not descendente))

for col in columnas:
    tabla.heading(col, command=lambda c=col: ordenar(tabla, c))
```

Árbol jerárquico (`show="tree headings"`):

```python
carpeta = arbol.insert("", tk.END, text="Periféricos", open=True)
arbol.insert(carpeta, tk.END, text="Teclado", values=(...))
```

### 9.2 Pestañas (`Notebook`)

```python
cuaderno = ttk.Notebook(root)
pestana_clientes = ttk.Frame(cuaderno, padding=10)
pestana_productos = ttk.Frame(cuaderno, padding=10)
cuaderno.add(pestana_clientes, text="Clientes")
cuaderno.add(pestana_productos, text="Productos")
cuaderno.pack(fill="both", expand=True)

cuaderno.select(1)                    # Ir a la segunda pestaña
cuaderno.index("current")
```

### 9.3 Lista (`Listbox`)

```python
lista = tk.Listbox(root, height=8, selectmode="extended")
for elemento in ["Madrid", "Sevilla", "Valencia"]:
    lista.insert(tk.END, elemento)
seleccion = [lista.get(i) for i in lista.curselection()]
```

### 9.4 Barras de desplazamiento

```python
texto = tk.Text(marco, wrap="none")
sb_y = ttk.Scrollbar(marco, orient="vertical", command=texto.yview)
sb_x = ttk.Scrollbar(marco, orient="horizontal", command=texto.xview)
texto.configure(yscrollcommand=sb_y.set, xscrollcommand=sb_x.set)
texto.grid(row=0, column=0, sticky="nsew"); sb_y.grid(row=0, column=1, sticky="ns"); sb_x.grid(row=1, column=0, sticky="ew")
marco.rowconfigure(0, weight=1); marco.columnconfigure(0, weight=1)
```

`tkinter.scrolledtext.ScrolledText` incluye ya la barra vertical.

### 9.5 Marco desplazable (contenido largo)

```python
class MarcoDesplazable(ttk.Frame):
    def __init__(self, padre, **kw):
        super().__init__(padre, **kw)
        self.canvas = tk.Canvas(self, highlightthickness=0)
        barra = ttk.Scrollbar(self, orient="vertical", command=self.canvas.yview)
        self.interior = ttk.Frame(self.canvas)
        self.interior.bind("<Configure>", lambda e: self.canvas.configure(scrollregion=self.canvas.bbox("all")))
        ventana = self.canvas.create_window((0, 0), window=self.interior, anchor="nw")
        self.canvas.bind("<Configure>", lambda e: self.canvas.itemconfigure(ventana, width=e.width))
        self.canvas.configure(yscrollcommand=barra.set)
        self.canvas.pack(side="left", fill="both", expand=True)
        barra.pack(side="right", fill="y")
```

---

## 10. Menús, diálogos y ventanas secundarias

### 10.1 Barra de menús

```python
barra = tk.Menu(root)

archivo = tk.Menu(barra, tearoff=False)
archivo.add_command(label="Nuevo", accelerator="Ctrl+N", command=nuevo)
archivo.add_command(label="Abrir…", accelerator="Ctrl+O", command=abrir)
archivo.add_separator()
archivo.add_command(label="Salir", command=root.destroy)
barra.add_cascade(label="Archivo", menu=archivo, underline=0)

ver = tk.Menu(barra, tearoff=False)
modo_oscuro = tk.BooleanVar()
ver.add_checkbutton(label="Modo oscuro", variable=modo_oscuro, command=cambiar_tema)
barra.add_cascade(label="Ver", menu=ver)

root.config(menu=barra)
```

> `accelerator` solo muestra el texto del atajo: el atajo se activa con `root.bind(...)`.

### 10.2 Menú contextual (clic derecho)

```python
contextual = tk.Menu(root, tearoff=False)
contextual.add_command(label="Copiar", command=copiar)
contextual.add_command(label="Eliminar", command=eliminar)

def mostrar_menu(evento):
    contextual.tk_popup(evento.x_root, evento.y_root)

tabla.bind("<Button-3>", mostrar_menu)            # En macOS: <Button-2>
```

### 10.3 Cuadros de mensaje

```python
messagebox.showinfo("Información", "Datos guardados correctamente.")
messagebox.showwarning("Aviso", "Hay campos vacíos.")
messagebox.showerror("Error", "No se pudo conectar con la base de datos.")
messagebox.askyesno("Confirmar", "¿Eliminar el registro?")            # True / False
messagebox.askyesnocancel("Guardar", "¿Guardar los cambios?")         # True / False / None
messagebox.askokcancel("Salir", "¿Desea salir?")
```

### 10.4 Diálogos de archivo, entrada y color

```python
ruta = filedialog.askopenfilename(title="Abrir CSV", filetypes=[("CSV", "*.csv"), ("Todos", "*.*")])
rutas = filedialog.askopenfilenames()
ruta = filedialog.asksaveasfilename(defaultextension=".xlsx", filetypes=[("Excel", "*.xlsx")],
                                    initialfile="informe.xlsx")
carpeta = filedialog.askdirectory()

nombre = simpledialog.askstring("Nuevo cliente", "Nombre:", parent=root)
edad = simpledialog.askinteger("Edad", "Introduzca la edad:", minvalue=0, maxvalue=120)
color = colorchooser.askcolor(title="Elegir color")[1]       # '#rrggbb' o None
```

> Todos devuelven `""` o `None` si el usuario cancela: compruébelo antes de continuar.

### 10.5 Ventanas secundarias (`Toplevel`)

```python
class DialogoCliente(tk.Toplevel):
    def __init__(self, padre, datos=None):
        super().__init__(padre)
        self.title("Cliente")
        self.resizable(False, False)
        self.transient(padre)            # Ligada a la ventana principal
        self.grab_set()                  # Modal: bloquea la ventana principal
        self.resultado = None

        self.nombre = tk.StringVar(value=(datos or {}).get("nombre", ""))
        ttk.Label(self, text="Nombre:").grid(row=0, column=0, padx=10, pady=10)
        e = ttk.Entry(self, textvariable=self.nombre, width=30)
        e.grid(row=0, column=1, padx=10, pady=10)
        e.focus()
        ttk.Button(self, text="Aceptar", command=self.aceptar).grid(row=1, column=1, sticky="e", padx=10, pady=10)
        self.bind("<Return>", lambda _: self.aceptar())
        self.bind("<Escape>", lambda _: self.destroy())

    def aceptar(self):
        self.resultado = {"nombre": self.nombre.get().strip()}
        self.destroy()


dialogo = DialogoCliente(root)
root.wait_window(dialogo)               # Espera a que se cierre
print(dialogo.resultado)
```

---

## 11. Estilos y temas (ttk)

### 11.1 Temas incluidos

```python
estilo = ttk.Style()
print(estilo.theme_names())             # ('winnative', 'clam', 'alt', 'default', 'classic', 'vista', 'xpnative')
estilo.theme_use("clam")                # 'clam' es el más personalizable
```

### 11.2 Personalizar estilos

```python
estilo.configure("TButton", font=("Segoe UI", 10), padding=(12, 6))
estilo.configure("TLabel", font=("Segoe UI", 10))
estilo.configure("Titulo.TLabel", font=("Segoe UI", 18, "bold"), foreground="#1F4E78")

estilo.configure("Primario.TButton", background="#1F4E78", foreground="white")
estilo.map("Primario.TButton",
           background=[("active", "#163A5A"), ("disabled", "#A0A0A0")])

estilo.configure("Treeview", rowheight=26, font=("Segoe UI", 10))
estilo.configure("Treeview.Heading", font=("Segoe UI", 10, "bold"))

ttk.Label(root, text="Clientes", style="Titulo.TLabel")
ttk.Button(root, text="Guardar", style="Primario.TButton")
```

> El nombre de un estilo propio termina con el nombre de la clase base: `"Algo.TButton"`, `"Algo.TLabel"`, `"Algo.TFrame"`.

### 11.3 Fuente por defecto de toda la aplicación

```python
from tkinter import font
font.nametofont("TkDefaultFont").configure(family="Segoe UI", size=10)
font.nametofont("TkTextFont").configure(family="Segoe UI", size=10)
```

### 11.4 Temas de terceros

| Paquete | Aspecto |
|---|---|
| `sv-ttk` | Sun Valley (estilo Windows 11), claro y oscuro. |
| `ttkbootstrap` | Temas inspirados en Bootstrap. |
| `azure-ttk-theme` | Moderno, claro y oscuro. |

```python
# pip install sv-ttk
import sv_ttk
sv_ttk.set_theme("dark")
```

---

## 12. Canvas, imágenes y gráficos

### 12.1 Canvas (dibujo)

```python
canvas = tk.Canvas(root, width=400, height=250, bg="white", highlightthickness=0)
canvas.pack()

canvas.create_line(10, 10, 390, 10, fill="#999", width=2)
canvas.create_rectangle(20, 30, 120, 100, fill="#1F4E78", outline="")
canvas.create_oval(150, 30, 220, 100, fill="orange")
canvas.create_text(200, 150, text="Texto en el canvas", font=("Segoe UI", 12))
id_barra = canvas.create_rectangle(20, 200, 220, 220, fill="seagreen")

canvas.coords(id_barra, 20, 200, 300, 220)      # Mover / redimensionar
canvas.itemconfig(id_barra, fill="red")
canvas.delete(id_barra)
canvas.delete("all")
```

### 12.2 Imágenes

`PhotoImage` admite PNG y GIF. Para JPEG y redimensionar, use **Pillow**:

```python
from PIL import Image, ImageTk

imagen = Image.open("logo.jpg")
imagen.thumbnail((200, 200))
foto = ImageTk.PhotoImage(imagen)

etiqueta = ttk.Label(root, image=foto)
etiqueta.image = foto           # ⚠ Mantener una referencia: si no, la imagen desaparece
etiqueta.pack()
```

### 12.3 Gráficos de Matplotlib

```python
from matplotlib.figure import Figure
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg, NavigationToolbar2Tk

figura = Figure(figsize=(6, 3.5), dpi=100)
ax = figura.add_subplot()
ax.bar(["Norte", "Sur", "Este"], [120, 95, 140], color="#1F4E78")
ax.set_title("Ventas por región")

lienzo = FigureCanvasTkAgg(figura, master=marco)
lienzo.draw()
lienzo.get_tk_widget().pack(fill="both", expand=True)
NavigationToolbar2Tk(lienzo, marco)                 # Barra de zoom y guardado

# Actualizar
ax.clear(); ax.plot(...); lienzo.draw_idle()
```

> Use `matplotlib.figure.Figure`, no `pyplot`, para evitar ventanas adicionales y fugas de memoria.

---

## 13. Tareas largas sin bloquear la interfaz

Tkinter procesa los eventos en un único hilo. Si una función tarda (descarga, consulta, cálculo), **la ventana se congela** («No responde»).

### 13.1 Ejecutar en un hilo y comunicarse con una cola

> ⚠ Tkinter **no es seguro entre hilos**: los widgets solo deben modificarse desde el hilo principal.

```python
import queue
import threading

cola = queue.Queue()

def tarea_larga(n):                                    # Se ejecuta en otro hilo
    for i in range(1, n + 1):
        time.sleep(0.05)                               # Trabajo simulado
        cola.put(("progreso", i * 100 / n))
    cola.put(("fin", "Proceso completado"))

def revisar_cola():                                    # Se ejecuta en el hilo principal
    try:
        while True:
            tipo, valor = cola.get_nowait()
            if tipo == "progreso":
                barra["value"] = valor
            elif tipo == "fin":
                boton.config(state="normal")
                messagebox.showinfo("Listo", valor)
                return
    except queue.Empty:
        pass
    root.after(100, revisar_cola)

def iniciar():
    boton.config(state="disabled")
    threading.Thread(target=tarea_larga, args=(100,), daemon=True).start()
    revisar_cola()
```

### 13.2 Procesos cortos en bucle

Para bucles que deben actualizar la interfaz, divida el trabajo en partes con `after`:

```python
def procesar_lote(elementos, indice=0, tamano=50):
    for elemento in elementos[indice:indice + tamano]:
        procesar(elemento)
    barra["value"] = min(100, (indice + tamano) * 100 / len(elementos))
    if indice + tamano < len(elementos):
        root.after(1, procesar_lote, elementos, indice + tamano, tamano)
```

---

## 14. Estructura de una aplicación

Para aplicaciones de más de unas pocas decenas de líneas, organice el código en **clases** y separe la interfaz de la lógica de negocio.

```python
import tkinter as tk
from tkinter import ttk


class PanelClientes(ttk.Frame):
    def __init__(self, padre, servicio):
        super().__init__(padre, padding=10)
        self.servicio = servicio
        self._crear_widgets()

    def _crear_widgets(self):
        ...


class Aplicacion(tk.Tk):
    def __init__(self, servicio):
        super().__init__()
        self.title("Gestión")
        self.geometry("900x600")
        self._configurar_estilos()
        PanelClientes(self, servicio).pack(fill="both", expand=True)

    def _configurar_estilos(self):
        ttk.Style(self).theme_use("clam")


if __name__ == "__main__":
    Aplicacion(servicio=ServicioClientes("clientes.db")).mainloop()
```

Organización recomendada de archivos:

```
mi_app/
├── main.py              # Punto de entrada
├── interfaz/            # Ventanas y paneles (solo interfaz)
├── servicios/           # Lógica de negocio y acceso a datos
├── recursos/            # Iconos e imágenes
└── config.py
```

---

## 15. Distribuir la aplicación

### 15.1 Ejecutar sin consola (Windows)

Guarde el archivo con extensión **`.pyw`** o ejecútelo con `pythonw.exe`.

### 15.2 Crear un ejecutable con PyInstaller

```bash
pip install pyinstaller
pyinstaller --onefile --windowed --name "GestorClientes" --icon recursos/icono.ico main.py
```

| Opción | Efecto |
|---|---|
| `--onefile` | Un único ejecutable. |
| `--windowed` (`-w`) | Sin ventana de consola. |
| `--icon` | Icono del ejecutable. |
| `--add-data "recursos;recursos"` | Incluir archivos (en Linux/macOS, separador `:`). |

Para localizar recursos dentro del ejecutable:

```python
import sys
from pathlib import Path

def recurso(ruta: str) -> Path:
    base = Path(getattr(sys, "_MEIPASS", Path(__file__).resolve().parent))
    return base / ruta
```

---

## 16. Ejemplo práctico completo

**Gestor de clientes** con base de datos SQLite: tabla con búsqueda en vivo y ordenación, formulario con validación, alta/edición/baja, menú y atajos, menú contextual, exportación a CSV en segundo plano con barra de progreso, barra de estado y estilos personalizados.

```python
"""
Gestor de clientes con Tkinter + SQLite.
Solo usa la librería estándar.
"""
import csv
import queue
import re
import sqlite3
import sys
import threading
import time
import tkinter as tk
from pathlib import Path
from tkinter import filedialog, messagebox, ttk

RUTA_BD = Path(__file__).resolve().parent / "clientes.db"
CIUDADES = ["Madrid", "Barcelona", "Sevilla", "Valencia", "Bilbao", "Zaragoza", "Málaga"]
PATRON_EMAIL = re.compile(r"^[\w.+-]+@[\w-]+\.[\w.-]+$")
MOD = "Command" if sys.platform == "darwin" else "Control"


# ------------------------------------------------------------------ Datos
class Repositorio:
    def __init__(self, ruta: Path):
        self.conn = sqlite3.connect(ruta)
        self.conn.row_factory = sqlite3.Row
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS clientes (
                id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, email TEXT NOT NULL UNIQUE,
                telefono TEXT, ciudad TEXT, vip INTEGER NOT NULL DEFAULT 0)
        """)
        if self.conn.execute("SELECT COUNT(*) FROM clientes").fetchone()[0] == 0:
            ejemplos = [(f"Cliente {i:03d}", f"cliente{i}@ejemplo.com", f"6{i:08d}",
                         CIUDADES[i % len(CIUDADES)], int(i % 7 == 0)) for i in range(1, 61)]
            with self.conn:
                self.conn.executemany(
                    "INSERT INTO clientes (nombre, email, telefono, ciudad, vip) VALUES (?, ?, ?, ?, ?)", ejemplos)

    def buscar(self, texto: str = "") -> list[sqlite3.Row]:
        patron = f"%{texto.strip()}%"
        return self.conn.execute(
            "SELECT * FROM clientes WHERE nombre LIKE ? OR email LIKE ? OR ciudad LIKE ? ORDER BY nombre",
            (patron, patron, patron)).fetchall()

    def guardar(self, datos: dict, id_: int | None = None) -> None:
        with self.conn:
            if id_ is None:
                self.conn.execute("INSERT INTO clientes (nombre, email, telefono, ciudad, vip) "
                                  "VALUES (:nombre, :email, :telefono, :ciudad, :vip)", datos)
            else:
                self.conn.execute("UPDATE clientes SET nombre=:nombre, email=:email, telefono=:telefono, "
                                  "ciudad=:ciudad, vip=:vip WHERE id=:id", {**datos, "id": id_})

    def eliminar(self, id_: int) -> None:
        with self.conn:
            self.conn.execute("DELETE FROM clientes WHERE id = ?", (id_,))


# ------------------------------------------------------------------ Diálogo de edición
class DialogoCliente(tk.Toplevel):
    def __init__(self, padre: tk.Misc, titulo: str, datos: dict | None = None):
        super().__init__(padre)
        self.title(titulo)
        self.resizable(False, False)
        self.transient(padre)
        self.resultado: dict | None = None
        datos = datos or {}

        self.vars = {
            "nombre": tk.StringVar(value=datos.get("nombre", "")),
            "email": tk.StringVar(value=datos.get("email", "")),
            "telefono": tk.StringVar(value=datos.get("telefono", "") or ""),
            "ciudad": tk.StringVar(value=datos.get("ciudad", "") or CIUDADES[0]),
            "vip": tk.BooleanVar(value=bool(datos.get("vip", 0))),
        }

        marco = ttk.Frame(self, padding=15)
        marco.pack(fill="both", expand=True)
        vcmd_tel = (self.register(lambda p: p == "" or (p.isdigit() and len(p) <= 9)), "%P")

        campos = [("Nombre *", ttk.Entry(marco, textvariable=self.vars["nombre"], width=34)),
                  ("Email *", ttk.Entry(marco, textvariable=self.vars["email"], width=34)),
                  ("Teléfono", ttk.Entry(marco, textvariable=self.vars["telefono"], width=34,
                                         validate="key", validatecommand=vcmd_tel)),
                  ("Ciudad", ttk.Combobox(marco, textvariable=self.vars["ciudad"], values=CIUDADES, width=31))]
        for fila, (etiqueta, widget) in enumerate(campos):
            ttk.Label(marco, text=etiqueta).grid(row=fila, column=0, sticky="w", pady=4)
            widget.grid(row=fila, column=1, sticky="ew", pady=4, padx=(10, 0))
        ttk.Checkbutton(marco, text="Cliente VIP", variable=self.vars["vip"]).grid(
            row=len(campos), column=1, sticky="w", pady=4, padx=(10, 0))

        self.error = ttk.Label(marco, foreground="#C00000")
        self.error.grid(row=len(campos) + 1, column=0, columnspan=2, sticky="w")

        botones = ttk.Frame(marco)
        botones.grid(row=len(campos) + 2, column=0, columnspan=2, sticky="e", pady=(10, 0))
        ttk.Button(botones, text="Cancelar", command=self.destroy).pack(side="right")
        ttk.Button(botones, text="Guardar", style="Primario.TButton", command=self.aceptar).pack(side="right", padx=6)

        self.bind("<Return>", lambda _: self.aceptar())
        self.bind("<Escape>", lambda _: self.destroy())
        campos[0][1].focus()
        self.grab_set()

    def aceptar(self) -> None:
        datos = {k: v.get() for k, v in self.vars.items()}
        datos["nombre"] = datos["nombre"].strip()
        datos["email"] = datos["email"].strip().lower()
        datos["vip"] = int(datos["vip"])
        if not datos["nombre"]:
            self.error.config(text="El nombre es obligatorio.")
        elif not PATRON_EMAIL.match(datos["email"]):
            self.error.config(text="El email no es válido.")
        else:
            self.resultado = datos
            self.destroy()


# ------------------------------------------------------------------ Ventana principal
class Aplicacion(tk.Tk):
    COLUMNAS = (("nombre", "Nombre", 200, "w"), ("email", "Email", 230, "w"),
                ("telefono", "Teléfono", 110, "center"), ("ciudad", "Ciudad", 110, "w"), ("vip", "VIP", 50, "center"))

    def __init__(self, repo: Repositorio):
        super().__init__()
        self.repo = repo
        self.cola: queue.Queue = queue.Queue()
        self.title("Gestor de clientes")
        self.geometry("820x540")
        self.minsize(650, 400)
        self._estilos()
        self._menu()
        self._interfaz()
        self._atajos()
        self.refrescar()
        self.protocol("WM_DELETE_WINDOW", self.salir)

    # ---------- Construcción
    def _estilos(self) -> None:
        estilo = ttk.Style(self)
        estilo.theme_use("clam")
        estilo.configure("Titulo.TLabel", font=("Segoe UI", 16, "bold"), foreground="#1F4E78")
        estilo.configure("Primario.TButton", background="#1F4E78", foreground="white", padding=(12, 5))
        estilo.map("Primario.TButton", background=[("active", "#163A5A"), ("disabled", "#9AA9B8")])
        estilo.configure("Treeview", rowheight=24)
        estilo.configure("Treeview.Heading", font=("Segoe UI", 9, "bold"))
        estilo.configure("Estado.TLabel", background="#E9EEF3", padding=(8, 3))

    def _menu(self) -> None:
        barra = tk.Menu(self)
        archivo = tk.Menu(barra, tearoff=False)
        archivo.add_command(label="Nuevo cliente", accelerator="Ctrl+N", command=self.nuevo)
        archivo.add_command(label="Exportar CSV…", accelerator="Ctrl+E", command=self.exportar)
        archivo.add_separator()
        archivo.add_command(label="Salir", command=self.salir)
        barra.add_cascade(label="Archivo", menu=archivo)
        self.config(menu=barra)

        self.contextual = tk.Menu(self, tearoff=False)
        self.contextual.add_command(label="Editar", command=self.editar)
        self.contextual.add_command(label="Eliminar", command=self.eliminar)

    def _interfaz(self) -> None:
        cabecera = ttk.Frame(self, padding=(12, 10))
        cabecera.pack(fill="x")
        ttk.Label(cabecera, text="Clientes", style="Titulo.TLabel").pack(side="left")
        self.busqueda = tk.StringVar()
        self.busqueda.trace_add("write", lambda *_: self.refrescar())
        buscador = ttk.Entry(cabecera, textvariable=self.busqueda, width=30)
        buscador.pack(side="right")
        ttk.Label(cabecera, text="Buscar:").pack(side="right", padx=6)

        cuerpo = ttk.Frame(self, padding=(12, 0))
        cuerpo.pack(fill="both", expand=True)
        cuerpo.rowconfigure(0, weight=1)
        cuerpo.columnconfigure(0, weight=1)

        self.tabla = ttk.Treeview(cuerpo, columns=[c[0] for c in self.COLUMNAS], show="headings",
                                  selectmode="browse")
        for clave, titulo, ancho, alin in self.COLUMNAS:
            self.tabla.heading(clave, text=titulo, command=lambda c=clave: self.ordenar(c, False))
            self.tabla.column(clave, width=ancho, anchor=alin, stretch=clave in ("nombre", "email"))
        self.tabla.tag_configure("vip", foreground="#8B5A00")
        self.tabla.tag_configure("par", background="#F5F7FA")
        barra = ttk.Scrollbar(cuerpo, orient="vertical", command=self.tabla.yview)
        self.tabla.configure(yscrollcommand=barra.set)
        self.tabla.grid(row=0, column=0, sticky="nsew")
        barra.grid(row=0, column=1, sticky="ns")
        self.tabla.bind("<Double-1>", lambda _: self.editar())
        self.tabla.bind("<Button-3>", self._menu_contextual)
        self.tabla.bind("<<TreeviewSelect>>", lambda _: self._actualizar_botones())

        botones = ttk.Frame(self, padding=12)
        botones.pack(fill="x")
        ttk.Button(botones, text="Nuevo", style="Primario.TButton", command=self.nuevo).pack(side="left")
        self.btn_editar = ttk.Button(botones, text="Editar", command=self.editar)
        self.btn_editar.pack(side="left", padx=6)
        self.btn_eliminar = ttk.Button(botones, text="Eliminar", command=self.eliminar)
        self.btn_eliminar.pack(side="left")
        self.btn_exportar = ttk.Button(botones, text="Exportar CSV", command=self.exportar)
        self.btn_exportar.pack(side="right")
        self.progreso = ttk.Progressbar(botones, length=160, mode="determinate")

        self.estado = ttk.Label(self, style="Estado.TLabel", anchor="w")
        self.estado.pack(fill="x", side="bottom")
        buscador.focus()

    def _atajos(self) -> None:
        self.bind(f"<{MOD}-n>", lambda _: self.nuevo())
        self.bind(f"<{MOD}-e>", lambda _: self.exportar())
        self.bind("<Delete>", lambda _: self.eliminar())
        self.bind("<F5>", lambda _: self.refrescar())

    # ---------- Acciones
    def refrescar(self) -> None:
        self.tabla.delete(*self.tabla.get_children())
        filas = self.repo.buscar(self.busqueda.get())
        for i, c in enumerate(filas):
            etiquetas = ("par",) if i % 2 else ()
            if c["vip"]:
                etiquetas += ("vip",)
            self.tabla.insert("", tk.END, iid=str(c["id"]), tags=etiquetas,
                              values=(c["nombre"], c["email"], c["telefono"] or "", c["ciudad"] or "",
                                      "★" if c["vip"] else ""))
        self.estado.config(text=f"{len(filas)} clientes" + (f" · filtro: «{self.busqueda.get()}»"
                                                              if self.busqueda.get() else ""))
        self._actualizar_botones()

    def _seleccionado(self) -> dict | None:
        sel = self.tabla.selection()
        if not sel:
            return None
        valores = self.tabla.item(sel[0], "values")
        return {"id": int(sel[0]), "nombre": valores[0], "email": valores[1], "telefono": valores[2],
                "ciudad": valores[3], "vip": valores[4] == "★"}

    def _actualizar_botones(self) -> None:
        estado = "normal" if self.tabla.selection() else "disabled"
        self.btn_editar.config(state=estado)
        self.btn_eliminar.config(state=estado)

    def _menu_contextual(self, evento) -> None:
        fila = self.tabla.identify_row(evento.y)
        if fila:
            self.tabla.selection_set(fila)
            self.contextual.tk_popup(evento.x_root, evento.y_root)

    def ordenar(self, columna: str, desc: bool) -> None:
        datos = [(self.tabla.set(i, columna).lower(), i) for i in self.tabla.get_children("")]
        datos.sort(reverse=desc)
        for pos, (_, iid) in enumerate(datos):
            self.tabla.move(iid, "", pos)
        self.tabla.heading(columna, command=lambda: self.ordenar(columna, not desc))

    def _dialogo(self, titulo: str, datos: dict | None = None) -> dict | None:
        dlg = DialogoCliente(self, titulo, datos)
        self.wait_window(dlg)
        return dlg.resultado

    def nuevo(self) -> None:
        if (datos := self._dialogo("Nuevo cliente")) is not None:
            self._guardar(datos)

    def editar(self) -> None:
        if (actual := self._seleccionado()) and (datos := self._dialogo("Editar cliente", actual)) is not None:
            self._guardar(datos, actual["id"])

    def _guardar(self, datos: dict, id_: int | None = None) -> None:
        try:
            self.repo.guardar(datos, id_)
        except sqlite3.IntegrityError:
            messagebox.showerror("Email duplicado", f"Ya existe un cliente con el email {datos['email']}.")
            return
        self.refrescar()
        self.estado.config(text=f"✔ Cliente «{datos['nombre']}» guardado")

    def eliminar(self) -> None:
        actual = self._seleccionado()
        if actual and messagebox.askyesno("Eliminar", f"¿Eliminar a «{actual['nombre']}»?", icon="warning"):
            self.repo.eliminar(actual["id"])
            self.refrescar()

    def exportar(self) -> None:
        ruta = filedialog.asksaveasfilename(defaultextension=".csv", initialfile="clientes.csv",
                                            filetypes=[("CSV", "*.csv")])
        if not ruta:
            return
        filas = [dict(f) for f in self.repo.buscar(self.busqueda.get())]
        self.btn_exportar.config(state="disabled")
        self.progreso.pack(side="right", padx=10)
        self.progreso["value"] = 0
        threading.Thread(target=self._exportar_hilo, args=(ruta, filas), daemon=True).start()
        self.after(100, self._revisar_cola)

    def _exportar_hilo(self, ruta: str, filas: list[dict]) -> None:
        try:
            with open(ruta, "w", newline="", encoding="utf-8-sig") as f:
                w = csv.DictWriter(f, fieldnames=["id", "nombre", "email", "telefono", "ciudad", "vip"], delimiter=";")
                w.writeheader()
                for i, fila in enumerate(filas, 1):
                    w.writerow(fila)
                    time.sleep(0.01)                            # Simula un proceso más lento
                    self.cola.put(("progreso", i * 100 / len(filas)))
            self.cola.put(("ok", f"{len(filas)} clientes exportados a {Path(ruta).name}"))
        except OSError as e:
            self.cola.put(("error", str(e)))

    def _revisar_cola(self) -> None:
        try:
            while True:
                tipo, valor = self.cola.get_nowait()
                if tipo == "progreso":
                    self.progreso["value"] = valor
                    continue
                self.progreso.pack_forget()
                self.btn_exportar.config(state="normal")
                (messagebox.showinfo if tipo == "ok" else messagebox.showerror)("Exportación", valor)
                return
        except queue.Empty:
            self.after(100, self._revisar_cola)

    def salir(self) -> None:
        self.repo.conn.close()
        self.destroy()


if __name__ == "__main__":
    if sys.platform == "win32":
        try:
            import ctypes
            ctypes.windll.shcore.SetProcessDpiAwareness(1)
        except (AttributeError, OSError):
            pass
    Aplicacion(Repositorio(RUTA_BD)).mainloop()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'tkinter'` | Linux sin el paquete del sistema. | `sudo apt install python3-tk`. |
| `_tkinter.TclError: no display name and no $DISPLAY` | Se ejecuta en un servidor sin entorno gráfico. | Ejecútelo en un equipo con escritorio. |
| La ventana no aparece / el programa termina enseguida | Falta `mainloop()`. | Llame a `root.mainloop()` al final. |
| `TclError: cannot use geometry manager grid inside . which already has slaves managed by pack` | Mezcla de `pack` y `grid` en el mismo contenedor. | Use un solo gestor por contenedor. |
| La función del botón se ejecuta al arrancar | `command=funcion()` (con paréntesis). | `command=funcion` o `command=lambda: funcion(arg)`. |
| La imagen no se muestra | El objeto `PhotoImage` fue eliminado por el recolector de basura. | Guarde una referencia (`etiqueta.image = foto`). |
| La ventana se congela («No responde») | Tarea larga en el hilo principal. | Use hilos + `queue` + `after`. |
| `RuntimeError: main thread is not in main loop` | Se modificó un widget desde otro hilo. | Comunique los cambios mediante una cola. |
| Todos los botones creados en un bucle hacen lo mismo | La `lambda` captura la variable del bucle. | Use un argumento por defecto: `lambda c=col: ...`. |
| `TclError: expected floating-point number but got ""` | `IntVar`/`DoubleVar` con campo vacío o texto. | Valide la entrada o capture la excepción. |
| Aparece una ventana vacía adicional | Se crearon dos `tk.Tk()`. | Use `tk.Toplevel` para ventanas secundarias. |
| El estilo de un botón `ttk` no cambia el color | El tema nativo ignora algunas opciones. | Use el tema `"clam"`. |
| Ventana borrosa en Windows | Escalado DPI. | `SetProcessDpiAwareness(1)` antes de crear la ventana. |

---

## 18. Buenas prácticas

- Use widgets **`ttk`** y el tema `clam` (o un tema de terceros) para un aspecto más moderno.
- Use `grid` para formularios y `pack` para la disposición general; no los mezcle en el mismo contenedor.
- Configure `weight` en filas/columnas para que la interfaz se adapte al tamaño de la ventana.
- Organice la aplicación en **clases** y separe la interfaz de la lógica y los datos.
- Use variables de control y `trace_add` para formularios reactivos.
- Valide los datos antes de guardarlos y muestre los errores de forma clara.
- No bloquee el hilo principal: tareas largas en hilos y actualización de la interfaz con `after`.
- Añada atajos de teclado y permita trabajar sin ratón (`Tab`, `Enter`, `Esc`).
- Confirme las acciones destructivas.
- Si necesita un diseño moderno sin cambiar de enfoque, valore **CustomTkinter**; para aplicaciones complejas, **PySide6**.

---

## 19. Referencia rápida

```python
root = tk.Tk(); root.title(t); root.geometry("800x600"); root.mainloop()
ttk.Label / Button(command=f) / Entry(textvariable=v) / Combobox(values=[...]) / Checkbutton / Radiobutton
ttk.Spinbox / Scale / Progressbar / Frame / LabelFrame / Notebook / Treeview / Scrollbar / Separator
tk.Text / Listbox / Canvas / Menu / Toplevel
tk.StringVar / IntVar / DoubleVar / BooleanVar → .get() .set() .trace_add("write", f)

w.pack(side, fill, expand, padx, pady) / w.grid(row, column, sticky, columnspan, padx) / w.place(relx, rely, anchor)
contenedor.columnconfigure(i, weight=1) / rowconfigure(i, weight=1)
w.config(opción=valor) / w.bind("<Evento>", f) / root.after(ms, f) / root.protocol("WM_DELETE_WINDOW", f)

messagebox.showinfo / showerror / askyesno · filedialog.askopenfilename / asksaveasfilename / askdirectory
ttk.Style().theme_use("clam"); style.configure("Mio.TButton", ...); style.map(...)
```

---

## 20. Ejercicios propuestos

1. Cree un conversor de unidades (temperatura, longitud, peso) con pestañas y actualización en vivo.
2. Desarrolle un editor de notas con menú Archivo (nuevo, abrir, guardar), atajos y aviso de cambios sin guardar.
3. Construya un visor de CSV que cargue el archivo en un `Treeview` con búsqueda y ordenación por columnas.
4. Diseñe un panel de ventas con un gráfico de Matplotlib que se actualice al cambiar un filtro.
5. Añada al ejemplo completo una pestaña de estadísticas (clientes por ciudad, porcentaje VIP) y genere un ejecutable con PyInstaller.

---

## 21. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/tkinter.html
- `tkinter.ttk`: https://docs.python.org/es/3/library/tkinter.ttk.html
- TkDocs (tutorial de referencia): https://tkdocs.com/tutorial/
