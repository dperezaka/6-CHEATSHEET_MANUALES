# Manual de uso: PyOpenGL

> **Categoría:** Videojuegos y gráficos · **Utilidad:** Acceso a OpenGL para gráficos 3D
> **Tipo:** Librería externa (requiere instalación con `pip`). Enlace de Python a la API OpenGL.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Crear una ventana y un contexto OpenGL](#4-crear-una-ventana-y-un-contexto-opengl)
5. [El *pipeline* moderno: búferes, VAO y *shaders*](#5-el-pipeline-moderno-búferes-vao-y-shaders)
6. [Primer triángulo](#6-primer-triángulo)
7. [Matemáticas 3D: matrices y transformaciones](#7-matemáticas-3d-matrices-y-transformaciones)
8. [Geometría indexada, profundidad y caras ocultas](#8-geometría-indexada-profundidad-y-caras-ocultas)
9. [Texturas](#9-texturas)
10. [Iluminación](#10-iluminación)
11. [Cámara e interacción](#11-cámara-e-interacción)
12. [Técnicas avanzadas](#12-técnicas-avanzadas)
13. [OpenGL clásico (modo inmediato)](#13-opengl-clásico-modo-inmediato)
14. [Depuración y errores de OpenGL](#14-depuración-y-errores-de-opengl)
15. [Rendimiento e integración](#15-rendimiento-e-integración)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**PyOpenGL** es el enlace (*binding*) estándar de Python para **OpenGL**, la API multiplataforma de gráficos acelerados por la tarjeta gráfica (GPU). Da acceso directo a todas las funciones de OpenGL (desde la versión 1.1 hasta la 4.6), GLU y GLUT. No es un motor de juegos: es la **capa de bajo nivel** sobre la que se construyen motores, visores científicos, herramientas CAD o simuladores.

**Ventajas:**
- Acceso completo a la GPU: *shaders* GLSL, búferes, texturas, *framebuffers*, instanciado, *compute shaders*.
- Conocimiento transferible: la misma API se usa en C, C++, Java, JavaScript (WebGL) y otros lenguajes.
- Combinable con cualquier gestor de ventanas: GLFW, Pygame, pyglet, Qt (PySide6), wxPython, SDL.
- Ideal para aprender cómo funcionan los gráficos 3D por dentro y para visualizaciones a medida.

**Limitaciones:**
- Muy bajo nivel: dibujar un cubo iluminado requiere decenas de líneas.
- No incluye ventana, entrada, carga de modelos, sonido ni matemáticas 3D (se complementa con otras librerías).
- Cada llamada pasa de Python a C: con miles de llamadas por fotograma el rendimiento cae (hay que agrupar el trabajo en búferes).
- OpenGL está en modo de mantenimiento en algunas plataformas (macOS lo limita a la versión 4.1).

| Si necesita… | Opción |
|---|---|
| Aprender gráficos 3D a bajo nivel o crear un visor/motor propio | **PyOpenGL** |
| API OpenGL más «pythónica» y con menos sobrecarga | ModernGL |
| Motor 3D completo | Panda3D |
| Juego 3D sencillo con poco código | Ursina |
| Juego 2D | Pygame / Arcade |

---

## 2. Instalación e importación

```bash
pip install PyOpenGL PyOpenGL_accelerate      # PyOpenGL_accelerate: extensiones en C que lo aceleran
pip install glfw numpy                        # Ventana (GLFW) y matrices/búferes (NumPy)
pip install pillow                            # Opcional: cargar y guardar imágenes (texturas, capturas)
```

> GLFW se instala con su biblioteca nativa incluida en Windows, macOS y Linux. En Linux también puede necesitar los controladores Mesa (`sudo apt install libgl1 libglfw3`).

Importaciones habituales:

```python
import glfw
import numpy as np
from OpenGL.GL import *                    # Funciones gl* y constantes GL_*
from OpenGL.GL.shaders import compileProgram, compileShader
```

> `from OpenGL.GL import *` es la convención habitual: las funciones ya llevan el prefijo `gl` y las constantes `GL_`, así que no hay conflicto de nombres.

Comprobar la versión de OpenGL disponible (con un contexto creado):

```python
print(glGetString(GL_VERSION).decode())           # p. ej. "4.6.0 NVIDIA 550.54"
print(glGetString(GL_RENDERER).decode())          # Tarjeta gráfica
print(glGetString(GL_SHADING_LANGUAGE_VERSION).decode())
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Contexto OpenGL | Estado de OpenGL asociado a una ventana. **Debe existir antes de llamar a cualquier función `gl*`.** Lo crea la librería de ventanas. |
| Máquina de estados | OpenGL guarda un estado global (objeto activo, color de borrado, pruebas activadas…). Las funciones actúan sobre lo que esté «enlazado» (*bound*). |
| Vértice | Punto con atributos: posición, color, normal, coordenadas de textura. |
| VBO (*Vertex Buffer Object*) | Búfer en la memoria de la GPU con los datos de los vértices. |
| EBO / IBO (*Element Buffer*) | Búfer de índices para reutilizar vértices. |
| VAO (*Vertex Array Object*) | Guarda la configuración de cómo leer los atributos desde los VBO. |
| *Shader* | Programa en **GLSL** que se ejecuta en la GPU. Mínimo: *vertex shader* (posición de cada vértice) y *fragment shader* (color de cada píxel). |
| Programa | *Shaders* compilados y enlazados. |
| *Uniform* | Variable global del *shader* que se envía desde Python (matrices, color de luz, tiempo). |
| Textura | Imagen en la GPU que los *shaders* muestrean. |
| *Framebuffer* | Destino del dibujo (la ventana u otra textura). |
| Doble búfer | Se dibuja en un búfer oculto y se intercambia con el visible (`swap_buffers`) para evitar parpadeo. |

**El *pipeline* gráfico moderno:**

```
Datos de vértices (VBO) ──► Vertex shader ──► Ensamblado de primitivas ──► Rasterización
                               (posición)          (triángulos)              (fragmentos)
                                                                                   │
            Pantalla ◄── Pruebas (profundidad, mezcla) ◄── Fragment shader ◄───────┘
                                                             (color)
```

**Dos estilos de OpenGL:**

| Estilo | Versión | Características | Uso |
|---|---|---|---|
| Clásico / modo inmediato | 1.x–2.x | `glBegin`/`glEnd`, `glVertex`, luces fijas, matrices de OpenGL | Obsoleto; útil para prototipos rapidísimos (sección 13). |
| Moderno / *core profile* | 3.3+ | Búferes, VAO, *shaders* GLSL, matemáticas propias | **Recomendado.** |

**Coordenadas:** tras el *vertex shader*, los puntos visibles están en el cubo **NDC** de -1 a 1 en X, Y y Z. X hacia la derecha, Y hacia arriba; el (0, 0) es el centro de la ventana.

---

## 4. Crear una ventana y un contexto OpenGL

### 4.1 Con GLFW (recomendado)

```python
import glfw
from OpenGL.GL import *

if not glfw.init():
    raise RuntimeError("No se pudo iniciar GLFW")

glfw.window_hint(glfw.CONTEXT_VERSION_MAJOR, 3)
glfw.window_hint(glfw.CONTEXT_VERSION_MINOR, 3)
glfw.window_hint(glfw.OPENGL_PROFILE, glfw.OPENGL_CORE_PROFILE)
glfw.window_hint(glfw.OPENGL_FORWARD_COMPAT, True)         # Obligatorio en macOS
glfw.window_hint(glfw.SAMPLES, 4)                           # Antialiasing (MSAA)

ventana = glfw.create_window(1280, 720, "OpenGL con Python", None, None)
if not ventana:
    glfw.terminate()
    raise RuntimeError("No se pudo crear la ventana OpenGL 3.3")
glfw.make_context_current(ventana)
glfw.swap_interval(1)                                       # Sincronización vertical

glClearColor(0.1, 0.12, 0.16, 1.0)
while not glfw.window_should_close(ventana):
    glfw.poll_events()
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
    # ... dibujar ...
    glfw.swap_buffers(ventana)

glfw.terminate()
```

Eventos y estado en GLFW:

```python
def al_redimensionar(ventana, ancho, alto):
    glViewport(0, 0, ancho, alto)

def al_pulsar(ventana, tecla, codigo, accion, mods):
    if tecla == glfw.KEY_ESCAPE and accion == glfw.PRESS:
        glfw.set_window_should_close(ventana, True)

glfw.set_framebuffer_size_callback(ventana, al_redimensionar)
glfw.set_key_callback(ventana, al_pulsar)
glfw.set_cursor_pos_callback(ventana, lambda v, x, y: ...)
glfw.set_mouse_button_callback(ventana, lambda v, boton, accion, mods: ...)
glfw.set_scroll_callback(ventana, lambda v, dx, dy: ...)

glfw.get_key(ventana, glfw.KEY_W) == glfw.PRESS              # Estado actual de una tecla
glfw.get_cursor_pos(ventana)
glfw.get_time()                                               # Segundos desde glfw.init()
glfw.set_input_mode(ventana, glfw.CURSOR, glfw.CURSOR_DISABLED)    # Capturar el ratón (cámara FPS)
glfw.get_framebuffer_size(ventana)                            # Tamaño real en píxeles (pantallas HiDPI)
```

> En pantallas de alta densidad (Retina, escalado del 150 %), el tamaño del *framebuffer* puede ser mayor que el de la ventana. Use siempre `get_framebuffer_size` para `glViewport`.

### 4.2 Con Pygame

```python
import pygame
from pygame.locals import DOUBLEBUF, OPENGL

pygame.init()
pygame.display.gl_set_attribute(pygame.GL_CONTEXT_MAJOR_VERSION, 3)
pygame.display.gl_set_attribute(pygame.GL_CONTEXT_MINOR_VERSION, 3)
pygame.display.gl_set_attribute(pygame.GL_CONTEXT_PROFILE_MASK, pygame.GL_CONTEXT_PROFILE_CORE)
pygame.display.set_mode((1280, 720), DOUBLEBUF | OPENGL)
...
pygame.display.flip()                      # Equivale a swap_buffers
```

### 4.3 Otras opciones

| Librería | Uso |
|---|---|
| `glfw` | Ventana y entrada, ligera; la más habitual con OpenGL moderno. |
| `pygame` | Si ya usa Pygame para sonido o entrada. |
| `pyglet` | Ventanas, texto y audio en Python puro. |
| PySide6 / PyQt6 (`QOpenGLWidget`) | Visor 3D dentro de una aplicación de escritorio. |
| GLUT (`OpenGL.GLUT`) | Clásica; requiere tener *freeglut* instalado en el sistema. |

---

## 5. El *pipeline* moderno: búferes, VAO y *shaders*

### 5.1 Datos de vértices con NumPy

Los datos se envían a la GPU como arreglos contiguos de `float32`:

```python
import numpy as np

#                    posición (x, y, z)    color (r, g, b)
vertices = np.array([-0.5, -0.5, 0.0,      1.0, 0.0, 0.0,
                      0.5, -0.5, 0.0,      0.0, 1.0, 0.0,
                      0.0,  0.5, 0.0,      0.0, 0.0, 1.0], dtype=np.float32)
```

> Use **siempre** `dtype=np.float32` (o `np.uint32` para índices). Los `float64` por defecto de NumPy producen geometría corrupta.

### 5.2 VBO y VAO

```python
vao = glGenVertexArrays(1)
glBindVertexArray(vao)                                     # A partir de aquí se graba la configuración en el VAO

vbo = glGenBuffers(1)
glBindBuffer(GL_ARRAY_BUFFER, vbo)
glBufferData(GL_ARRAY_BUFFER, vertices.nbytes, vertices, GL_STATIC_DRAW)

paso = 6 * 4                                               # Bytes por vértice: 6 floats × 4 bytes
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, paso, ctypes.c_void_p(0))        # location 0: posición
glEnableVertexAttribArray(0)
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, paso, ctypes.c_void_p(3 * 4))    # location 1: color
glEnableVertexAttribArray(1)

glBindVertexArray(0)                                       # Desenlazar
```

| Parámetro de `glVertexAttribPointer` | Significado |
|---|---|
| `index` | `location` del atributo en el *shader*. |
| `size` | Componentes (1–4). |
| `type` | `GL_FLOAT`, `GL_INT`… |
| `normalized` | Normalizar enteros a 0–1. |
| `stride` | Bytes entre un vértice y el siguiente. |
| `pointer` | Desplazamiento en bytes (`ctypes.c_void_p(n)`). |

| Uso del búfer | Cuándo |
|---|---|
| `GL_STATIC_DRAW` | Datos que no cambian (modelos). |
| `GL_DYNAMIC_DRAW` | Datos que cambian a menudo (partículas). |
| `GL_STREAM_DRAW` | Datos que cambian cada fotograma y se usan una vez. |

Actualizar parte de un búfer: `glBufferSubData(GL_ARRAY_BUFFER, desplazamiento, datos.nbytes, datos)`.

### 5.3 *Shaders* GLSL

```python
VERTEX_SHADER = """
#version 330 core
layout (location = 0) in vec3 a_posicion;
layout (location = 1) in vec3 a_color;
out vec3 v_color;                       // Se interpola y pasa al fragment shader
uniform mat4 u_mvp;

void main() {
    v_color = a_color;
    gl_Position = u_mvp * vec4(a_posicion, 1.0);
}
"""

FRAGMENT_SHADER = """
#version 330 core
in vec3 v_color;
out vec4 color_final;

void main() {
    color_final = vec4(v_color, 1.0);
}
"""
```

Compilar y enlazar:

```python
from OpenGL.GL.shaders import compileProgram, compileShader

programa = compileProgram(
    compileShader(VERTEX_SHADER, GL_VERTEX_SHADER),
    compileShader(FRAGMENT_SHADER, GL_FRAGMENT_SHADER),
)
glUseProgram(programa)
```

> `compileShader` lanza una excepción con el mensaje del compilador GLSL si hay errores: léalo, indica la línea.

> Cree el VAO **antes** de compilar en perfil *core*: `compileProgram` valida el programa y algunos controladores fallan si no hay ningún VAO enlazado.

### 5.4 *Uniforms*

```python
ubicacion = glGetUniformLocation(programa, "u_mvp")            # Búsquela una vez y guárdela
glUniformMatrix4fv(ubicacion, 1, GL_TRUE, matriz)              # GL_TRUE: la matriz NumPy está por filas
glUniform3f(glGetUniformLocation(programa, "u_luz"), 1.0, 0.9, 0.8)
glUniform1f(glGetUniformLocation(programa, "u_tiempo"), glfw.get_time())
glUniform1i(glGetUniformLocation(programa, "u_textura"), 0)    # Unidad de textura 0
glUniform3fv(ubicacion, 1, np.array([1, 2, 3], dtype=np.float32))
```

| Tipo GLSL | Función |
|---|---|
| `float` / `int` / `bool` | `glUniform1f` / `glUniform1i` |
| `vec2` / `vec3` / `vec4` | `glUniform2f` / `glUniform3f` / `glUniform4f` (o `…fv` con arreglos) |
| `mat3` / `mat4` | `glUniformMatrix3fv` / `glUniformMatrix4fv` |
| `sampler2D` | `glUniform1i` con la unidad de textura |

> El programa debe estar activo (`glUseProgram`) antes de asignar sus *uniforms*. Si el *shader* no usa una variable, el compilador la elimina y `glGetUniformLocation` devuelve -1.

---

## 6. Primer triángulo

```python
import ctypes

import glfw
import numpy as np
from OpenGL.GL import *
from OpenGL.GL.shaders import compileProgram, compileShader

VS = """#version 330 core
layout (location = 0) in vec2 a_pos;
layout (location = 1) in vec3 a_color;
out vec3 v_color;
uniform float u_angulo;
void main() {
    mat2 rot = mat2(cos(u_angulo), sin(u_angulo), -sin(u_angulo), cos(u_angulo));
    gl_Position = vec4(rot * a_pos, 0.0, 1.0);
    v_color = a_color;
}"""
FS = """#version 330 core
in vec3 v_color;
out vec4 f_color;
void main() { f_color = vec4(v_color, 1.0); }"""

glfw.init()
for clave, valor in [(glfw.CONTEXT_VERSION_MAJOR, 3), (glfw.CONTEXT_VERSION_MINOR, 3),
                     (glfw.OPENGL_PROFILE, glfw.OPENGL_CORE_PROFILE), (glfw.OPENGL_FORWARD_COMPAT, True)]:
    glfw.window_hint(clave, valor)
ventana = glfw.create_window(800, 600, "Triángulo", None, None)
glfw.make_context_current(ventana)

datos = np.array([-0.6, -0.5, 1, 0, 0,
                   0.6, -0.5, 0, 1, 0,
                   0.0,  0.6, 0, 0, 1], dtype=np.float32)
vao = glGenVertexArrays(1); glBindVertexArray(vao)
vbo = glGenBuffers(1); glBindBuffer(GL_ARRAY_BUFFER, vbo)
glBufferData(GL_ARRAY_BUFFER, datos.nbytes, datos, GL_STATIC_DRAW)
glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, 20, ctypes.c_void_p(0)); glEnableVertexAttribArray(0)
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 20, ctypes.c_void_p(8)); glEnableVertexAttribArray(1)

programa = compileProgram(compileShader(VS, GL_VERTEX_SHADER), compileShader(FS, GL_FRAGMENT_SHADER))
u_angulo = glGetUniformLocation(programa, "u_angulo")

while not glfw.window_should_close(ventana):
    glfw.poll_events()
    glClearColor(0.1, 0.1, 0.15, 1); glClear(GL_COLOR_BUFFER_BIT)
    glUseProgram(programa)
    glUniform1f(u_angulo, glfw.get_time())
    glBindVertexArray(vao)
    glDrawArrays(GL_TRIANGLES, 0, 3)
    glfw.swap_buffers(ventana)

glDeleteBuffers(1, [vbo]); glDeleteVertexArrays(1, [vao]); glDeleteProgram(programa)
glfw.terminate()
```

Primitivas de `glDrawArrays` / `glDrawElements`:

| Primitiva | Dibuja |
|---|---|
| `GL_TRIANGLES` | Un triángulo por cada 3 vértices (la más usada). |
| `GL_TRIANGLE_STRIP` / `GL_TRIANGLE_FAN` | Triángulos encadenados / en abanico. |
| `GL_LINES` / `GL_LINE_STRIP` / `GL_LINE_LOOP` | Líneas. |
| `GL_POINTS` | Puntos (tamaño con `gl_PointSize` y `glEnable(GL_PROGRAM_POINT_SIZE)`). |

---

## 7. Matemáticas 3D: matrices y transformaciones

Para pasar de coordenadas del objeto a la pantalla se multiplican tres matrices 4×4:

```
gl_Position = Proyección × Vista × Modelo × vértice
                 │           │       └─ coloca el objeto en el mundo (mover, girar, escalar)
                 │           └───────── coloca el mundo frente a la cámara
                 └───────────────────── perspectiva (lo lejano se ve más pequeño)
```

Funciones con NumPy (matrices por filas; se envían con `GL_TRUE` para transponer):

```python
import math
import numpy as np

def perspectiva(fov_grados, aspecto, cerca, lejos):
    f = 1 / math.tan(math.radians(fov_grados) / 2)
    return np.array([[f / aspecto, 0, 0, 0],
                     [0, f, 0, 0],
                     [0, 0, (lejos + cerca) / (cerca - lejos), 2 * lejos * cerca / (cerca - lejos)],
                     [0, 0, -1, 0]], dtype=np.float32)

def mirar_a(ojo, objetivo, arriba=(0, 1, 0)):
    ojo, objetivo, arriba = (np.asarray(v, dtype=np.float32) for v in (ojo, objetivo, arriba))
    f = objetivo - ojo; f /= np.linalg.norm(f)
    s = np.cross(f, arriba); s /= np.linalg.norm(s)
    u = np.cross(s, f)
    m = np.identity(4, dtype=np.float32)
    m[0, :3], m[1, :3], m[2, :3] = s, u, -f
    m[:3, 3] = -m[:3, :3] @ ojo
    return m

def traslacion(x, y, z):
    m = np.identity(4, dtype=np.float32); m[:3, 3] = (x, y, z); return m

def escala(sx, sy=None, sz=None):
    return np.diag([sx, sy or sx, sz or sx, 1]).astype(np.float32)

def rotacion_y(grados):
    c, s = math.cos(math.radians(grados)), math.sin(math.radians(grados))
    return np.array([[c, 0, s, 0], [0, 1, 0, 0], [-s, 0, c, 0], [0, 0, 0, 1]], dtype=np.float32)

modelo = traslacion(2, 0, 0) @ rotacion_y(45) @ escala(0.5)       # Se aplica de derecha a izquierda
mvp = perspectiva(60, 16 / 9, 0.1, 100) @ mirar_a((0, 3, 8), (0, 0, 0)) @ modelo
glUniformMatrix4fv(u_mvp, 1, GL_TRUE, mvp)
```

> **Orden de las transformaciones:** `T @ R @ S` escala primero, luego gira y por último traslada. Cambiar el orden cambia el resultado.

> Alternativas a escribir las matrices a mano: `pyrr` (`pyrr.matrix44.create_perspective_projection`) y `PyGLM` (`glm.perspective`, `glm.lookAt`), con la misma convención que la librería GLM de C++.

**Convenciones de OpenGL:** sistema de mano derecha, cámara mirando hacia **-Z**, Y hacia arriba. El rango de profundidad visible va del plano cercano al lejano: elija `cerca` lo más grande posible (p. ej. 0,1) para no perder precisión.

---

## 8. Geometría indexada, profundidad y caras ocultas

### 8.1 Búfer de índices (EBO)

Un cubo tiene 8 esquinas pero 12 triángulos (36 vértices). Con índices se reutilizan vértices:

```python
indices = np.array([0, 1, 2,  2, 3, 0], dtype=np.uint32)        # Dos triángulos = un cuadrado

glBindVertexArray(vao)
ebo = glGenBuffers(1)
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo)                     # El VAO recuerda este enlace
glBufferData(GL_ELEMENT_ARRAY_BUFFER, indices.nbytes, indices, GL_STATIC_DRAW)

glDrawElements(GL_TRIANGLES, len(indices), GL_UNSIGNED_INT, None)
```

> Si cada cara necesita su propia normal o coordenadas de textura (como en un cubo con iluminación), los vértices de las esquinas **no** se pueden compartir: se usan 24 vértices (4 por cara).

### 8.2 Prueba de profundidad

```python
glEnable(GL_DEPTH_TEST)                                       # Lo cercano tapa lo lejano
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)            # Borrar también la profundidad en cada fotograma
```

### 8.3 Ocultar caras traseras

```python
glEnable(GL_CULL_FACE)
glCullFace(GL_BACK)
glFrontFace(GL_CCW)            # Caras frontales: vértices en sentido antihorario vistos de frente
```

### 8.4 Modo alámbrico y transparencia

```python
glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)      # Alámbrico (GL_FILL para volver)
glEnable(GL_BLEND)
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA)
```

> Dibuje primero los objetos opacos y después los transparentes, ordenados de lejos a cerca.

---

## 9. Texturas

```python
from PIL import Image

def cargar_textura(ruta: str) -> int:
    imagen = Image.open(ruta).convert("RGBA").transpose(Image.Transpose.FLIP_TOP_BOTTOM)  # OpenGL: origen abajo
    datos = np.asarray(imagen, dtype=np.uint8)
    textura = glGenTextures(1)
    glBindTexture(GL_TEXTURE_2D, textura)
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT)
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT)
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR)
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, imagen.width, imagen.height, 0, GL_RGBA, GL_UNSIGNED_BYTE, datos)
    glGenerateMipmap(GL_TEXTURE_2D)
    return textura
```

Uso en el *shader* y al dibujar:

```glsl
in vec2 v_uv;
uniform sampler2D u_textura;
out vec4 f_color;
void main() { f_color = texture(u_textura, v_uv); }
```

```python
glActiveTexture(GL_TEXTURE0)
glBindTexture(GL_TEXTURE_2D, textura)
glUniform1i(u_textura, 0)
```

| Parámetro | Valores |
|---|---|
| `GL_TEXTURE_WRAP_S/T` | `GL_REPEAT`, `GL_MIRRORED_REPEAT`, `GL_CLAMP_TO_EDGE`. |
| `GL_TEXTURE_MIN_FILTER` | `GL_NEAREST`, `GL_LINEAR`, `GL_LINEAR_MIPMAP_LINEAR` (mejor calidad al alejarse). |
| `GL_TEXTURE_MAG_FILTER` | `GL_NEAREST` (*pixel art*), `GL_LINEAR` (suave). |

Texturas generadas por código (sin archivos):

```python
tablero = np.indices((256, 256)).sum(axis=0) // 32 % 2
pixeles = np.where(tablero[..., None], [230, 230, 230], [60, 60, 60]).astype(np.uint8)
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB8, 256, 256, 0, GL_RGB, GL_UNSIGNED_BYTE, pixeles)
```

> Si el ancho en bytes de cada fila no es múltiplo de 4 (texturas RGB de tamaño impar), llame antes a `glPixelStorei(GL_UNPACK_ALIGNMENT, 1)`.

---

## 10. Iluminación

El modelo **Phong** (o su variante Blinn-Phong) combina tres términos:

| Término | Simula |
|---|---|
| Ambiente | Luz indirecta que llega a todo. |
| Difusa | Superficies orientadas hacia la luz (depende de la **normal**). |
| Especular | Brillos según la posición de la cámara. |

Los vértices necesitan **normales** (vector perpendicular a la superficie).

```glsl
// Vertex shader
#version 330 core
layout (location = 0) in vec3 a_pos;
layout (location = 1) in vec3 a_normal;
uniform mat4 u_modelo, u_vista, u_proyeccion;
uniform mat3 u_normal;                      // inversa transpuesta de la parte 3×3 del modelo
out vec3 v_pos_mundo, v_normal;
void main() {
    vec4 mundo = u_modelo * vec4(a_pos, 1.0);
    v_pos_mundo = mundo.xyz;
    v_normal = u_normal * a_normal;
    gl_Position = u_proyeccion * u_vista * mundo;
}
```

```glsl
// Fragment shader (Blinn-Phong con luz direccional)
#version 330 core
in vec3 v_pos_mundo, v_normal;
uniform vec3 u_dir_luz, u_color_luz, u_color_objeto, u_camara;
out vec4 f_color;
void main() {
    vec3 n = normalize(v_normal);
    vec3 l = normalize(-u_dir_luz);
    vec3 v = normalize(u_camara - v_pos_mundo);
    vec3 h = normalize(l + v);
    float difusa = max(dot(n, l), 0.0);
    float especular = pow(max(dot(n, h), 0.0), 64.0);
    vec3 color = (0.15 + difusa) * u_color_objeto * u_color_luz + 0.4 * especular * u_color_luz;
    f_color = vec4(color, 1.0);
}
```

```python
matriz_normal = np.linalg.inv(modelo[:3, :3]).T.astype(np.float32)
glUniformMatrix3fv(u_normal, 1, GL_TRUE, matriz_normal)
```

> Si el modelo solo se traslada, gira o escala de forma uniforme, la matriz normal es simplemente `modelo[:3, :3]`.

Niebla lineal o exponencial en el *fragment shader*:

```glsl
float distancia = length(u_camara - v_pos_mundo);
float niebla = clamp(exp(-pow(distancia * 0.02, 2.0)), 0.0, 1.0);
f_color = vec4(mix(u_color_niebla, color, niebla), 1.0);
```

---

## 11. Cámara e interacción

### 11.1 Cámara orbital (rotar alrededor de un punto)

```python
class CamaraOrbital:
    def __init__(self, objetivo=(0, 0, 0), distancia=10, yaw=45, pitch=25):
        self.objetivo = np.array(objetivo, dtype=np.float32)
        self.distancia, self.yaw, self.pitch = distancia, yaw, pitch

    @property
    def posicion(self):
        y, p = math.radians(self.yaw), math.radians(self.pitch)
        return self.objetivo + self.distancia * np.array(
            [math.cos(p) * math.sin(y), math.sin(p), math.cos(p) * math.cos(y)], dtype=np.float32)

    def vista(self):
        return mirar_a(self.posicion, self.objetivo)

    def arrastrar(self, dx, dy):
        self.yaw -= dx * 0.3
        self.pitch = max(-85, min(85, self.pitch + dy * 0.3))

    def zoom(self, cantidad):
        self.distancia = max(2, min(80, self.distancia * (0.9 ** cantidad)))
```

### 11.2 Cámara libre (primera persona)

```python
def frente(yaw, pitch):
    y, p = math.radians(yaw), math.radians(pitch)
    return np.array([math.cos(p) * math.sin(y), math.sin(p), -math.cos(p) * math.cos(y)], dtype=np.float32)

def mover(dt):
    f = frente(yaw, pitch); derecha = np.cross(f, (0, 1, 0)); derecha /= np.linalg.norm(derecha)
    if glfw.get_key(ventana, glfw.KEY_W) == glfw.PRESS: posicion += f * velocidad * dt
    if glfw.get_key(ventana, glfw.KEY_S) == glfw.PRESS: posicion -= f * velocidad * dt
    if glfw.get_key(ventana, glfw.KEY_D) == glfw.PRESS: posicion += derecha * velocidad * dt
    if glfw.get_key(ventana, glfw.KEY_A) == glfw.PRESS: posicion -= derecha * velocidad * dt
vista = mirar_a(posicion, posicion + frente(yaw, pitch))
```

### 11.3 Medir el tiempo

```python
anterior = glfw.get_time()
while not glfw.window_should_close(ventana):
    ahora = glfw.get_time(); dt = ahora - anterior; anterior = ahora
```

---

## 12. Técnicas avanzadas

### 12.1 Instanciado (miles de copias en una llamada)

```python
desplazamientos = np.random.uniform(-50, 50, (10_000, 3)).astype(np.float32)
vbo_inst = glGenBuffers(1)
glBindVertexArray(vao)
glBindBuffer(GL_ARRAY_BUFFER, vbo_inst)
glBufferData(GL_ARRAY_BUFFER, desplazamientos.nbytes, desplazamientos, GL_STATIC_DRAW)
glVertexAttribPointer(2, 3, GL_FLOAT, GL_FALSE, 12, ctypes.c_void_p(0))
glEnableVertexAttribArray(2)
glVertexAttribDivisor(2, 1)                                  # Avanza una vez por instancia, no por vértice

glDrawElementsInstanced(GL_TRIANGLES, n_indices, GL_UNSIGNED_INT, None, 10_000)
```

```glsl
layout (location = 2) in vec3 a_desplazamiento;
// gl_Position = u_vp * vec4(a_pos + a_desplazamiento, 1.0);   gl_InstanceID identifica la copia
```

### 12.2 Renderizar a una textura (*framebuffer*)

Base del posprocesado (desenfoque, *bloom*), espejos, minimapas y sombras:

```python
fbo = glGenFramebuffers(1); glBindFramebuffer(GL_FRAMEBUFFER, fbo)
color = glGenTextures(1); glBindTexture(GL_TEXTURE_2D, color)
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, ancho, alto, 0, GL_RGBA, GL_UNSIGNED_BYTE, None)
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, color, 0)
profundidad = glGenRenderbuffers(1); glBindRenderbuffer(GL_RENDERBUFFER, profundidad)
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, ancho, alto)
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, profundidad)
assert glCheckFramebufferStatus(GL_FRAMEBUFFER) == GL_FRAMEBUFFER_COMPLETE
glBindFramebuffer(GL_FRAMEBUFFER, 0)                       # Volver a la ventana
```

### 12.3 Sombras (*shadow mapping*)

1. Renderizar la escena desde la luz a un *framebuffer* de solo profundidad.
2. Renderizar la escena normal y, en el *fragment shader*, comparar la profundidad del punto vista desde la luz con la almacenada: si es mayor, está en sombra.

### 12.4 Cargar modelos

PyOpenGL no carga modelos. Opciones: `trimesh` (OBJ, STL, PLY, glTF) o `pywavefront` (OBJ):

```python
import trimesh
malla = trimesh.load("modelo.obj", force="mesh")
vertices = np.hstack([malla.vertices, malla.vertex_normals]).astype(np.float32)
indices = malla.faces.astype(np.uint32).ravel()
```

### 12.5 Capturar la imagen

```python
ancho, alto = glfw.get_framebuffer_size(ventana)
glPixelStorei(GL_PACK_ALIGNMENT, 1)
datos = glReadPixels(0, 0, ancho, alto, GL_RGB, GL_UNSIGNED_BYTE)
Image.frombytes("RGB", (ancho, alto), datos).transpose(Image.Transpose.FLIP_TOP_BOTTOM).save("captura.png")
```

---

## 13. OpenGL clásico (modo inmediato)

Útil para prototipos de pocas líneas o código antiguo. Requiere un **perfil de compatibilidad** (no *core*) y está obsoleto:

```python
import math

import glfw
from OpenGL.GL import *

glfw.init()
ventana = glfw.create_window(800, 600, "Clásico", None, None)       # Sin hints de perfil core
glfw.make_context_current(ventana)
glEnable(GL_DEPTH_TEST)

while not glfw.window_should_close(ventana):
    glfw.poll_events()
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
    cerca, alto = 0.1, 0.1 * math.tan(math.radians(60) / 2)              # Perspectiva de 60°
    glMatrixMode(GL_PROJECTION); glLoadIdentity(); glFrustum(-alto * 800 / 600, alto * 800 / 600, -alto, alto, cerca, 100)
    glMatrixMode(GL_MODELVIEW); glLoadIdentity(); glTranslatef(0, 0, -5); glRotatef(glfw.get_time() * 50, 1, 1, 0)
    glBegin(GL_TRIANGLES)
    glColor3f(1, 0, 0); glVertex3f(-1, -1, 0)
    glColor3f(0, 1, 0); glVertex3f(1, -1, 0)
    glColor3f(0, 0, 1); glVertex3f(0, 1, 0)
    glEnd()
    glfw.swap_buffers(ventana)
glfw.terminate()
```

| Clásico | Moderno equivalente |
|---|---|
| `glBegin`/`glVertex`/`glEnd` | VBO + VAO + `glDrawArrays` |
| `glMatrixMode`, `glRotatef`, `glFrustum`/`gluPerspective` | Matrices propias + *uniforms* |
| `glLightfv`, `glMaterialfv` | Iluminación en el *fragment shader* |
| Listas de visualización (`glNewList`) | VBO estáticos |

> Las funciones `glu*` (`gluPerspective`, `gluLookAt`) están en la biblioteca GLU del sistema, que no siempre viene instalada (en Linux: `sudo apt install libglu1-mesa`). Si falta, se produce `NullFunctionError`; por eso el ejemplo usa `glFrustum`.

> El modo inmediato envía cada vértice desde Python en cada fotograma: es muy lento con geometría grande. No lo use para proyectos nuevos.

---

## 14. Depuración y errores de OpenGL

PyOpenGL comprueba `glGetError()` tras cada llamada y lanza `OpenGL.error.GLError` con la función culpable. Es útil durante el desarrollo, pero tiene coste:

```python
import OpenGL
OpenGL.ERROR_CHECKING = False          # Antes de importar OpenGL.GL; solo en la versión final
from OpenGL.GL import *
```

Otras opciones de configuración (antes de `from OpenGL.GL import *`):

| Opción | Efecto |
|---|---|
| `OpenGL.ERROR_CHECKING = False` | Desactiva la comprobación de errores (más rápido). |
| `OpenGL.ERROR_LOGGING = False` | Desactiva el registro de llamadas con error. |
| `OpenGL.FULL_LOGGING = True` | Registra todas las llamadas (depuración intensiva). |

Comprobar *shaders* manualmente:

```python
shader = glCreateShader(GL_FRAGMENT_SHADER)
glShaderSource(shader, codigo); glCompileShader(shader)
if not glGetShaderiv(shader, GL_COMPILE_STATUS):
    print(glGetShaderInfoLog(shader).decode())
```

Herramientas externas: **RenderDoc** (captura y analiza fotogramas, compatible con aplicaciones Python), NVIDIA Nsight, AMD Radeon GPU Profiler.

---

## 15. Rendimiento e integración

### 15.1 Rendimiento

| Técnica | Motivo |
|---|---|
| Subir los datos a la GPU una vez (VBO estáticos) | Evitar copiar datos en cada fotograma. |
| Agrupar geometría y usar instanciado | Cada llamada Python→C tiene coste; menos llamadas de dibujo. |
| Buscar las ubicaciones de *uniforms* una vez | `glGetUniformLocation` es lento. |
| Calcular en el *shader* | La GPU procesa millones de vértices y píxeles en paralelo. |
| Usar NumPy vectorizado para generar geometría | Bucles Python lentos con muchas mallas. |
| Instalar `PyOpenGL_accelerate` | Acelera conversiones de arreglos. |
| Desactivar `ERROR_CHECKING` en la versión final | Reduce la sobrecarga por llamada. |
| Valorar **ModernGL** | API orientada a objetos con muy poca sobrecarga. |

### 15.2 Integración en aplicaciones de escritorio (Qt)

```python
from PySide6.QtOpenGLWidgets import QOpenGLWidget

class Visor(QOpenGLWidget):
    def initializeGL(self):
        glClearColor(0.1, 0.1, 0.15, 1)       # Crear búferes, shaders, texturas
    def resizeGL(self, ancho, alto):
        glViewport(0, 0, ancho, alto)
    def paintGL(self):
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)   # Dibujar
```

### 15.3 Distribución

Con PyInstaller incluya los módulos de PyOpenGL que se cargan dinámicamente:

```bash
pyinstaller --onefile --windowed --collect-submodules OpenGL --collect-binaries glfw main.py
```

---

## 16. Ejemplo práctico completo

**Visor de terreno 3D**: aplicación con OpenGL 3.3 *core* que genera un terreno procedural (ruido de valor por octavas) con colores por altura, un plano de agua animado y semitransparente, un bosque de 1.500 árboles dibujados con **instanciado**, iluminación Blinn-Phong con luz direccional que recorre el ciclo día-noche, niebla y cielo degradado. Incluye cámara orbital con ratón (arrastrar y rueda), modo alámbrico (F), pausa del ciclo solar (Espacio), regenerar el terreno (R), captura de pantalla (F12) y FPS en el título.

```python
"""
Visor de terreno 3D con PyOpenGL (OpenGL 3.3 core) + GLFW + NumPy.
Requiere: pip install PyOpenGL glfw numpy pillow
Controles: arrastrar ratón = girar · rueda = zoom · F = alámbrico · R = nuevo terreno
           Espacio = pausar el sol · F12 = captura · Esc = salir
"""
import ctypes
import math
import time

import glfw
import numpy as np
from OpenGL.GL import *
from OpenGL.GL.shaders import compileProgram, compileShader

ANCHO, ALTO = 1280, 720
TAM_TERRENO = 160            # Vértices por lado
ESCALA_XZ = 0.5              # Distancia entre vértices
ALTURA_MAX = 14.0
NIVEL_AGUA = 3.2
NUM_ARBOLES = 1500

# ------------------------------------------------------------------ Shaders
VS_TERRENO = """#version 330 core
layout (location = 0) in vec3 a_pos;
layout (location = 1) in vec3 a_normal;
layout (location = 2) in vec3 a_color;
uniform mat4 u_vp;
out vec3 v_pos, v_normal, v_color;
void main() {
    v_pos = a_pos; v_normal = a_normal; v_color = a_color;
    gl_Position = u_vp * vec4(a_pos, 1.0);
}"""

VS_ARBOL = """#version 330 core
layout (location = 0) in vec3 a_pos;
layout (location = 1) in vec3 a_normal;
layout (location = 2) in vec3 a_color;
layout (location = 3) in vec4 a_instancia;          // xyz: posición, w: escala
uniform mat4 u_vp;
uniform float u_tiempo;
out vec3 v_pos, v_normal, v_color;
void main() {
    vec3 p = a_pos * a_instancia.w;
    p.x += sin(u_tiempo * 1.5 + a_instancia.x) * 0.06 * max(a_pos.y, 0.0);   // Balanceo con el viento
    v_pos = p + a_instancia.xyz;
    v_normal = a_normal;
    v_color = a_color * (0.85 + 0.3 * fract(sin(dot(a_instancia.xz, vec2(12.9, 78.2))) * 43758.5));
    gl_Position = u_vp * vec4(v_pos, 1.0);
}"""

VS_AGUA = """#version 330 core
layout (location = 0) in vec3 a_pos;
uniform mat4 u_vp;
uniform float u_tiempo;
out vec3 v_pos, v_normal, v_color;
void main() {
    vec3 p = a_pos;
    float ola = sin(p.x * 0.6 + u_tiempo * 1.2) * cos(p.z * 0.5 + u_tiempo) * 0.08;
    p.y += ola;
    v_pos = p;
    v_normal = normalize(vec3(-cos(p.x * 0.6 + u_tiempo * 1.2) * 0.05, 1.0, sin(p.z * 0.5 + u_tiempo) * 0.04));
    v_color = vec3(0.12, 0.38, 0.62);
    gl_Position = u_vp * vec4(p, 1.0);
}"""

FS_ILUMINADO = """#version 330 core
in vec3 v_pos, v_normal, v_color;
uniform vec3 u_dir_sol, u_color_sol, u_camara, u_color_niebla;
uniform float u_ambiente, u_brillo, u_alfa;
out vec4 f_color;
void main() {
    vec3 n = normalize(v_normal);
    vec3 l = normalize(-u_dir_sol);
    vec3 v = normalize(u_camara - v_pos);
    vec3 h = normalize(l + v);
    float difusa = max(dot(n, l), 0.0);
    float especular = u_brillo * pow(max(dot(n, h), 0.0), 48.0);
    vec3 color = v_color * (u_ambiente + difusa * u_color_sol) + especular * u_color_sol;
    float distancia = length(u_camara - v_pos);
    float niebla = clamp(exp(-pow(distancia * 0.012, 2.0)), 0.0, 1.0);
    f_color = vec4(mix(u_color_niebla, color, niebla), u_alfa);
}"""

VS_CIELO = """#version 330 core
const vec2 POS[3] = vec2[](vec2(-1.0, -1.0), vec2(3.0, -1.0), vec2(-1.0, 3.0));
out vec2 v_ndc;
void main() { v_ndc = POS[gl_VertexID]; gl_Position = vec4(POS[gl_VertexID], 0.9999, 1.0); }"""

FS_CIELO = """#version 330 core
in vec2 v_ndc;
uniform vec3 u_color_arriba, u_color_horizonte;
out vec4 f_color;
void main() { f_color = vec4(mix(u_color_horizonte, u_color_arriba, clamp(v_ndc.y * 0.6 + 0.4, 0.0, 1.0)), 1.0); }"""


# ------------------------------------------------------------------ Matemáticas
def perspectiva(fov, aspecto, cerca, lejos):
    f = 1 / math.tan(math.radians(fov) / 2)
    return np.array([[f / aspecto, 0, 0, 0], [0, f, 0, 0],
                     [0, 0, (lejos + cerca) / (cerca - lejos), 2 * lejos * cerca / (cerca - lejos)],
                     [0, 0, -1, 0]], dtype=np.float32)


def mirar_a(ojo, objetivo, arriba=(0.0, 1.0, 0.0)):
    ojo, objetivo, arriba = (np.asarray(v, dtype=np.float32) for v in (ojo, objetivo, arriba))
    f = objetivo - ojo
    f /= np.linalg.norm(f)
    s = np.cross(f, arriba)
    s /= np.linalg.norm(s)
    u = np.cross(s, f)
    m = np.identity(4, dtype=np.float32)
    m[0, :3], m[1, :3], m[2, :3] = s, u, -f
    m[:3, 3] = -m[:3, :3] @ ojo
    return m


# ------------------------------------------------------------------ Generación procedural
def ruido_valor(tam: int, semilla: int, octavas: int = 5) -> np.ndarray:
    """Ruido de valor por octavas (suma de rejillas aleatorias interpoladas), normalizado a 0-1."""
    rng = np.random.default_rng(semilla)
    resultado = np.zeros((tam, tam), dtype=np.float32)
    amplitud, total = 1.0, 0.0
    for octava in range(octavas):
        celdas = 3 * 2 ** octava
        rejilla = rng.random((celdas + 1, celdas + 1)).astype(np.float32)
        coords = np.linspace(0, celdas, tam, endpoint=False)
        i = coords.astype(int)
        t = coords - i
        t = t * t * (3 - 2 * t)                                     # Suavizado
        a, b = rejilla[i][:, i], rejilla[i][:, i + 1]
        c, d = rejilla[i + 1][:, i], rejilla[i + 1][:, i + 1]
        tx, ty = t[None, :], t[:, None]
        resultado += amplitud * ((a * (1 - tx) + b * tx) * (1 - ty) + (c * (1 - tx) + d * tx) * ty)
        total += amplitud
        amplitud *= 0.5
    resultado /= total
    return (resultado - resultado.min()) / (resultado.max() - resultado.min())


def generar_terreno(semilla: int):
    n = TAM_TERRENO
    alturas = ruido_valor(n, semilla) ** 1.6 * ALTURA_MAX
    centro = (n - 1) * ESCALA_XZ / 2
    xs = np.arange(n, dtype=np.float32) * ESCALA_XZ - centro
    x, z = np.meshgrid(xs, xs)
    posiciones = np.stack([x, alturas, z], axis=-1)

    # Normales por diferencias centrales
    dx = np.gradient(alturas, ESCALA_XZ, axis=1)
    dz = np.gradient(alturas, ESCALA_XZ, axis=0)
    normales = np.stack([-dx, np.ones_like(alturas), -dz], axis=-1)
    normales /= np.linalg.norm(normales, axis=-1, keepdims=True)

    # Colores por altura y pendiente: arena, hierba, roca, nieve
    h = alturas / ALTURA_MAX
    pendiente = 1 - normales[..., 1]
    arena, hierba = np.array([0.76, 0.70, 0.50]), np.array([0.28, 0.52, 0.22])
    roca, nieve = np.array([0.45, 0.42, 0.40]), np.array([0.95, 0.95, 0.97])
    colores = np.where((alturas < NIVEL_AGUA + 0.5)[..., None], arena, hierba)
    colores = np.where(((h > 0.55) | (pendiente > 0.35))[..., None], roca, colores)
    colores = np.where((h > 0.78)[..., None], nieve, colores)

    vertices = np.concatenate([posiciones, normales, colores], axis=-1).astype(np.float32).reshape(-1, 9)
    fila = np.arange(n - 1)
    a = (fila[:, None] * n + fila[None, :]).ravel()
    indices = np.stack([a, a + n, a + 1, a + 1, a + n, a + n + 1], axis=-1).astype(np.uint32).ravel()

    # Árboles: en zonas de hierba con poca pendiente
    rng = np.random.default_rng(semilla + 1)
    candidatos = np.argwhere((alturas > NIVEL_AGUA + 0.8) & (h < 0.5) & (pendiente < 0.25))
    elegidos = candidatos[rng.choice(len(candidatos), min(NUM_ARBOLES, len(candidatos)), replace=False)]
    instancias = np.column_stack([posiciones[elegidos[:, 0], elegidos[:, 1]],
                                  rng.uniform(0.6, 1.3, len(elegidos))]).astype(np.float32)
    return vertices, indices, instancias


def malla_arbol():
    """Tronco (prisma) y copa (cono) de 8 lados, con normales y colores."""
    lados, vertices, indices = 8, [], []

    def anillo(radio, y, normal_y, color):
        base = len(vertices)
        for i in range(lados):
            ang = 2 * math.pi * i / lados
            nx, nz = math.cos(ang), math.sin(ang)
            vertices.append([radio * nx, y, radio * nz, nx, normal_y, nz, *color])
        return base

    tronco, copa = (0.36, 0.24, 0.14), (0.16, 0.42, 0.18)
    b0, b1 = anillo(0.12, 0.0, 0.0, tronco), anillo(0.10, 0.8, 0.0, tronco)
    c0 = anillo(0.75, 0.6, 0.5, copa)
    punta = len(vertices)
    vertices.append([0, 2.6, 0, 0, 1, 0, *copa])
    for i in range(lados):
        j = (i + 1) % lados
        indices += [b0 + i, b1 + i, b0 + j, b0 + j, b1 + i, b1 + j]   # Tronco
        indices += [c0 + i, punta, c0 + j]                             # Copa
    return np.array(vertices, dtype=np.float32), np.array(indices, dtype=np.uint32)


# ------------------------------------------------------------------ Utilidades OpenGL
class Malla:
    """VAO con atributos posición(3) + normal(3) + color(3) e índices; instancias opcionales."""

    def __init__(self, vertices: np.ndarray, indices: np.ndarray, instancias: np.ndarray | None = None):
        self.n_indices, self.n_instancias = len(indices), 0
        self.vao = glGenVertexArrays(1)
        glBindVertexArray(self.vao)
        self.vbo, self.ebo = glGenBuffers(2)
        glBindBuffer(GL_ARRAY_BUFFER, self.vbo)
        glBufferData(GL_ARRAY_BUFFER, vertices.nbytes, vertices, GL_STATIC_DRAW)
        componentes = vertices.shape[1]
        for ubicacion, (tam, desplazamiento) in enumerate([(3, 0), (3, 3), (3, 6)][: componentes // 3]):
            glVertexAttribPointer(ubicacion, tam, GL_FLOAT, GL_FALSE, componentes * 4,
                                  ctypes.c_void_p(desplazamiento * 4))
            glEnableVertexAttribArray(ubicacion)
        glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, self.ebo)
        glBufferData(GL_ELEMENT_ARRAY_BUFFER, indices.nbytes, indices, GL_STATIC_DRAW)
        self.vbo_inst = None
        if instancias is not None:
            self.n_instancias = len(instancias)
            self.vbo_inst = glGenBuffers(1)
            glBindBuffer(GL_ARRAY_BUFFER, self.vbo_inst)
            glBufferData(GL_ARRAY_BUFFER, instancias.nbytes, instancias, GL_STATIC_DRAW)
            glVertexAttribPointer(3, 4, GL_FLOAT, GL_FALSE, 16, ctypes.c_void_p(0))
            glEnableVertexAttribArray(3)
            glVertexAttribDivisor(3, 1)
        glBindVertexArray(0)

    def dibujar(self):
        glBindVertexArray(self.vao)
        if self.n_instancias:
            glDrawElementsInstanced(GL_TRIANGLES, self.n_indices, GL_UNSIGNED_INT, None, self.n_instancias)
        else:
            glDrawElements(GL_TRIANGLES, self.n_indices, GL_UNSIGNED_INT, None)

    def liberar(self):
        buferes = [self.vbo, self.ebo] + ([self.vbo_inst] if self.vbo_inst else [])
        glDeleteBuffers(len(buferes), buferes)
        glDeleteVertexArrays(1, [self.vao])


class Programa:
    def __init__(self, vs: str, fs: str):
        self.id = compileProgram(compileShader(vs, GL_VERTEX_SHADER), compileShader(fs, GL_FRAGMENT_SHADER))
        self._ubicaciones: dict[str, int] = {}

    def usar(self):
        glUseProgram(self.id)
        return self

    def __setitem__(self, nombre: str, valor):
        if nombre not in self._ubicaciones:
            self._ubicaciones[nombre] = glGetUniformLocation(self.id, nombre)
        u = self._ubicaciones[nombre]
        if u == -1:
            return                                   # El shader no usa esta variable
        valor = np.asarray(valor, dtype=np.float32)
        if valor.shape == (4, 4):
            glUniformMatrix4fv(u, 1, GL_TRUE, valor)
        elif valor.shape == (3,):
            glUniform3fv(u, 1, valor)
        else:
            glUniform1f(u, float(valor))


# ------------------------------------------------------------------ Aplicación
class VisorTerreno:
    def __init__(self, ancho=ANCHO, alto=ALTO, visible=True):
        if not glfw.init():
            raise RuntimeError("No se pudo iniciar GLFW")
        for clave, valor in [(glfw.CONTEXT_VERSION_MAJOR, 3), (glfw.CONTEXT_VERSION_MINOR, 3),
                             (glfw.OPENGL_PROFILE, glfw.OPENGL_CORE_PROFILE),
                             (glfw.OPENGL_FORWARD_COMPAT, True), (glfw.SAMPLES, 4), (glfw.VISIBLE, visible)]:
            glfw.window_hint(clave, valor)
        self.ventana = glfw.create_window(ancho, alto, "Visor de terreno", None, None)
        if not self.ventana:
            glfw.terminate()
            raise RuntimeError("Se necesita una tarjeta gráfica compatible con OpenGL 3.3")
        glfw.make_context_current(self.ventana)
        glfw.swap_interval(1)
        for registrar, funcion in [(glfw.set_key_callback, self.al_pulsar),
                                   (glfw.set_mouse_button_callback, self.al_boton),
                                   (glfw.set_cursor_pos_callback, self.al_mover_raton),
                                   (glfw.set_scroll_callback, self.al_rueda)]:
            registrar(self.ventana, funcion)

        self.cielo_vao = glGenVertexArrays(1)       # El cielo genera sus vértices en el shader
        glBindVertexArray(self.cielo_vao)           # Un VAO enlazado evita errores al validar en perfil core
        self.prog_terreno = Programa(VS_TERRENO, FS_ILUMINADO)
        self.prog_arbol = Programa(VS_ARBOL, FS_ILUMINADO)
        self.prog_agua = Programa(VS_AGUA, FS_ILUMINADO)
        self.prog_cielo = Programa(VS_CIELO, FS_CIELO)
        self.arbol_vertices, self.arbol_indices = malla_arbol()
        self.terreno = self.arboles = None
        self.semilla = 7
        self.crear_mundo()
        self.agua = self.crear_agua()

        self.yaw, self.pitch, self.distancia = 35.0, 28.0, 70.0
        self.arrastrando, self.raton_anterior = False, (0.0, 0.0)
        self.alambrico, self.sol_pausado = False, False
        self.hora = 0.32                               # 0 = medianoche, 0.5 = mediodía
        glEnable(GL_DEPTH_TEST)
        glEnable(GL_CULL_FACE)
        glEnable(GL_MULTISAMPLE)
        glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA)

    # ---------------------------------------------------------------- Escena
    def crear_mundo(self):
        for malla in (self.terreno, self.arboles):
            if malla:
                malla.liberar()
        vertices, indices, instancias = generar_terreno(self.semilla)
        self.terreno = Malla(vertices, indices)
        self.arboles = Malla(self.arbol_vertices, self.arbol_indices, instancias)
        self.num_arboles = len(instancias)

    @staticmethod
    def crear_agua():
        mitad, n = (TAM_TERRENO - 1) * ESCALA_XZ / 2 + 20, 80
        xs = np.linspace(-mitad, mitad, n, dtype=np.float32)
        x, z = np.meshgrid(xs, xs)
        vertices = np.stack([x, np.full_like(x, NIVEL_AGUA), z], axis=-1).reshape(-1, 3)
        fila = np.arange(n - 1)
        a = (fila[:, None] * n + fila[None, :]).ravel()
        indices = np.stack([a, a + n, a + 1, a + 1, a + n, a + n + 1], axis=-1).astype(np.uint32).ravel()
        return Malla(vertices.astype(np.float32), indices)

    def iluminacion(self):
        angulo = (self.hora - 0.25) * 2 * math.pi            # Sale a las 6:00, se pone a las 18:00
        altura_sol = math.sin(angulo)
        direccion = -np.array([math.cos(angulo), max(altura_sol, -0.2), 0.35], dtype=np.float32)
        dia = float(np.clip(altura_sol * 3 + 0.3, 0, 1))
        atardecer = float(np.clip(1 - abs(altura_sol) * 4, 0, 1)) * (altura_sol > -0.1)
        color_sol = np.array([1.0, 0.95, 0.85]) * dia + np.array([0.6, 0.25, 0.0]) * atardecer * 0.6
        horizonte = np.array([0.70, 0.80, 0.92]) * dia + np.array([0.9, 0.45, 0.2]) * atardecer * 0.7 \
            + np.array([0.03, 0.04, 0.08]) * (1 - dia)
        arriba = np.array([0.28, 0.50, 0.85]) * dia + np.array([0.01, 0.01, 0.04]) * (1 - dia)
        ambiente = 0.12 + 0.25 * dia
        return direccion, color_sol.astype(np.float32), horizonte.astype(np.float32), arriba.astype(np.float32), ambiente

    # ---------------------------------------------------------------- Dibujo
    def dibujar(self, tiempo: float):
        ancho, alto = glfw.get_framebuffer_size(self.ventana)
        glViewport(0, 0, ancho, alto)
        y, p = math.radians(self.yaw), math.radians(self.pitch)
        camara = self.distancia * np.array([math.cos(p) * math.sin(y), math.sin(p), math.cos(p) * math.cos(y)],
                                           dtype=np.float32)
        vp = perspectiva(55, ancho / max(alto, 1), 0.5, 400) @ mirar_a(camara, (0, 2, 0))
        direccion, color_sol, horizonte, arriba, ambiente = self.iluminacion()

        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
        glPolygonMode(GL_FRONT_AND_BACK, GL_FILL)
        glDepthMask(GL_FALSE)                                         # El cielo no escribe profundidad
        self.prog_cielo.usar()
        self.prog_cielo["u_color_arriba"], self.prog_cielo["u_color_horizonte"] = arriba, horizonte
        glBindVertexArray(self.cielo_vao)
        glDrawArrays(GL_TRIANGLES, 0, 3)
        glDepthMask(GL_TRUE)

        glPolygonMode(GL_FRONT_AND_BACK, GL_LINE if self.alambrico else GL_FILL)
        for programa, malla, brillo, alfa in [(self.prog_terreno, self.terreno, 0.05, 1.0),
                                              (self.prog_arbol, self.arboles, 0.0, 1.0),
                                              (self.prog_agua, self.agua, 0.9, 0.78)]:
            programa.usar()
            programa["u_vp"], programa["u_tiempo"], programa["u_camara"] = vp, tiempo, camara
            programa["u_dir_sol"], programa["u_color_sol"], programa["u_ambiente"] = direccion, color_sol, ambiente
            programa["u_color_niebla"], programa["u_brillo"], programa["u_alfa"] = horizonte, brillo, alfa
            if alfa < 1:                                              # Transparente: al final y con mezcla
                glEnable(GL_BLEND)
                glDisable(GL_CULL_FACE)
            malla.dibujar()
            glDisable(GL_BLEND)
            glEnable(GL_CULL_FACE)

    # ---------------------------------------------------------------- Entrada
    def al_pulsar(self, ventana, tecla, codigo, accion, mods):
        if accion != glfw.PRESS:
            return
        if tecla == glfw.KEY_ESCAPE:
            glfw.set_window_should_close(ventana, True)
        elif tecla == glfw.KEY_F:
            self.alambrico = not self.alambrico
        elif tecla == glfw.KEY_SPACE:
            self.sol_pausado = not self.sol_pausado
        elif tecla == glfw.KEY_R:
            self.semilla += 1
            self.crear_mundo()
        elif tecla == glfw.KEY_F12:
            self.captura(f"terreno_{int(time.time())}.png")

    def al_boton(self, ventana, boton, accion, mods):
        if boton == glfw.MOUSE_BUTTON_LEFT:
            self.arrastrando = accion == glfw.PRESS
            self.raton_anterior = glfw.get_cursor_pos(ventana)

    def al_mover_raton(self, ventana, x, y):
        if self.arrastrando:
            dx, dy = x - self.raton_anterior[0], y - self.raton_anterior[1]
            self.yaw -= dx * 0.3
            self.pitch = float(np.clip(self.pitch + dy * 0.3, 5, 85))
        self.raton_anterior = (x, y)

    def al_rueda(self, ventana, dx, dy):
        self.distancia = float(np.clip(self.distancia * 0.9 ** dy, 15, 150))

    def captura(self, ruta: str):
        from PIL import Image                                         # Importación perezosa (opcional)
        ancho, alto = glfw.get_framebuffer_size(self.ventana)
        glPixelStorei(GL_PACK_ALIGNMENT, 1)
        datos = glReadPixels(0, 0, ancho, alto, GL_RGB, GL_UNSIGNED_BYTE)
        Image.frombytes("RGB", (ancho, alto), datos).transpose(Image.Transpose.FLIP_TOP_BOTTOM).save(ruta)
        print(f"Captura guardada: {ruta}")

    # ---------------------------------------------------------------- Bucle
    def ejecutar(self):
        anterior = glfw.get_time()
        fotogramas, ultimo_titulo = 0, anterior
        while not glfw.window_should_close(self.ventana):
            glfw.poll_events()
            ahora = glfw.get_time()
            dt, anterior = ahora - anterior, ahora
            if not self.sol_pausado:
                self.hora = (self.hora + dt / 60) % 1.0               # Un día dura 60 segundos
            self.dibujar(ahora)
            glfw.swap_buffers(self.ventana)
            fotogramas += 1
            if ahora - ultimo_titulo >= 0.5:
                horas = int(self.hora * 24)
                glfw.set_window_title(self.ventana, f"Visor de terreno · {fotogramas / (ahora - ultimo_titulo):.0f} FPS · "
                                                    f"{horas:02d}:{int(self.hora * 1440) % 60:02d} · {self.num_arboles} árboles")
                fotogramas, ultimo_titulo = 0, ahora
        self.cerrar()

    def cerrar(self):
        for malla in (self.terreno, self.arboles, self.agua):
            malla.liberar()
        glDeleteVertexArrays(1, [self.cielo_vao])
        for programa in (self.prog_terreno, self.prog_arbol, self.prog_agua, self.prog_cielo):
            glDeleteProgram(programa.id)
        glfw.terminate()


if __name__ == "__main__":
    VisorTerreno().ejecutar()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `NullFunctionError: Attempt to call an undefined function glGenVertexArrays` | No hay contexto OpenGL activo o es demasiado antiguo. | Cree la ventana y llame a `make_context_current` **antes** de cualquier `gl*`; pida 3.3 *core*. |
| `glfw.create_window` devuelve `None` | El sistema no admite la versión pedida (máquina virtual, escritorio remoto, controlador antiguo). | Actualice controladores; en macOS añada `OPENGL_FORWARD_COMPAT`. |
| Pantalla negra sin errores | Matrices incorrectas, objeto detrás de la cámara o fuera del rango cerca/lejos. | Pruebe primero sin matrices (NDC); revise `GL_TRUE` al enviar matrices NumPy. |
| Geometría deformada o «explotada» | Datos en `float64` o `stride`/desplazamientos erróneos. | `dtype=np.float32`; revise bytes por vértice. |
| `ShaderCompilationError` | Error de GLSL. | Lea el mensaje (indica línea); revise `#version 330 core` en la **primera** línea. |
| `ShaderValidationError: Validation failure` | Ningún VAO enlazado al validar el programa (perfil *core*). | Cree y enlace un VAO antes de `compileProgram`. |
| `GLError: invalid operation` en `glUniform*` | Programa no activo o tipo de *uniform* distinto. | `glUseProgram` antes; función acorde al tipo GLSL. |
| `glGetUniformLocation` devuelve -1 | Nombre mal escrito o variable no usada (eliminada al compilar). | Revise el nombre; ignore -1 sin enviar el valor. |
| Caras desaparecen al activar `GL_CULL_FACE` | Orden de los vértices (horario/antihorario) incoherente. | Ordene los triángulos o use `glFrontFace(GL_CW)`. |
| Objetos lejanos «parpadean» entre sí (*z-fighting*) | Plano cercano muy pequeño. | Aumente `cerca` (0,1–1) y reduzca `lejos`. |
| Textura inclinada o con rayas | Alineación de filas. | `glPixelStorei(GL_UNPACK_ALIGNMENT, 1)`. |
| Textura boca abajo | Origen de la imagen arriba, de OpenGL abajo. | Voltee la imagen al cargarla. |
| Imagen desplazada en pantallas HiDPI | `glViewport` con el tamaño de la ventana. | Use `glfw.get_framebuffer_size`. |
| Muy lento | Modo inmediato, muchas llamadas por fotograma, `ERROR_CHECKING` activo. | VBO estáticos, instanciado, menos llamadas, desactive la comprobación al publicar. |

---

## 18. Buenas prácticas

- Use **OpenGL 3.3 *core*** o superior; evite el modo inmediato en proyectos nuevos.
- Cree el contexto antes de cualquier llamada `gl*` y libere los recursos (`glDelete*`) al cerrar.
- Prepare los datos con **NumPy** (`float32`/`uint32`) y súbalos a la GPU una sola vez.
- Encapsule mallas y programas en clases (como `Malla` y `Programa` del ejemplo).
- Guarde las ubicaciones de *uniforms* y atributos; use `layout (location = N)` en GLSL.
- Haga el trabajo pesado en los *shaders* y reduzca las llamadas de dibujo (instanciado, geometría agrupada).
- Active `GL_DEPTH_TEST` y `GL_CULL_FACE`; dibuje los transparentes al final.
- Use `get_framebuffer_size` para el *viewport* (pantallas HiDPI).
- Mantenga la comprobación de errores durante el desarrollo y desactívela en la versión final.
- Use RenderDoc para entender qué ocurre en cada fotograma.

---

## 19. Referencia rápida

```python
import glfw, numpy as np, ctypes; from OpenGL.GL import *; from OpenGL.GL.shaders import compileProgram, compileShader
glfw.init(); glfw.window_hint(glfw.CONTEXT_VERSION_MAJOR, 3) …(MINOR 3, OPENGL_PROFILE CORE, FORWARD_COMPAT)
v = glfw.create_window(w, h, "t", None, None); glfw.make_context_current(v); glfw.swap_interval(1)
while not glfw.window_should_close(v): glfw.poll_events(); glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT); …; glfw.swap_buffers(v)

vao = glGenVertexArrays(1); glBindVertexArray(vao)
vbo = glGenBuffers(1); glBindBuffer(GL_ARRAY_BUFFER, vbo); glBufferData(GL_ARRAY_BUFFER, a.nbytes, a, GL_STATIC_DRAW)
glVertexAttribPointer(loc, n, GL_FLOAT, GL_FALSE, stride_bytes, ctypes.c_void_p(offset_bytes)); glEnableVertexAttribArray(loc)
ebo → GL_ELEMENT_ARRAY_BUFFER;  glDrawArrays(GL_TRIANGLES, 0, n) / glDrawElements(GL_TRIANGLES, n, GL_UNSIGNED_INT, None)
glVertexAttribDivisor(loc, 1) + glDrawElementsInstanced(..., n_instancias)
prog = compileProgram(compileShader(vs, GL_VERTEX_SHADER), compileShader(fs, GL_FRAGMENT_SHADER)); glUseProgram(prog)
u = glGetUniformLocation(prog, "u_mvp"); glUniformMatrix4fv(u, 1, GL_TRUE, m) / glUniform3f / glUniform1f / glUniform1i
tex = glGenTextures(1); glBindTexture(GL_TEXTURE_2D, tex); glTexImage2D(...); glGenerateMipmap(GL_TEXTURE_2D)
glEnable(GL_DEPTH_TEST | GL_CULL_FACE | GL_BLEND); glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA); glPolygonMode(..., GL_LINE)
gl_Position = P × V × M × vértice   (matrices NumPy por filas → GL_TRUE)
glReadPixels(0, 0, w, h, GL_RGB, GL_UNSIGNED_BYTE)  → captura
```

---

## 20. Ejercicios propuestos

1. Dibuje un cuadrado con dos triángulos usando un búfer de índices y cambie su color con un *uniform* que dependa del tiempo.
2. Construya un cubo con 24 vértices (normales por cara), iluminación difusa y cámara orbital con el ratón.
3. Aplique una textura de tablero generada con NumPy a un plano y compare los filtros `GL_NEAREST` y `GL_LINEAR_MIPMAP_LINEAR`.
4. Dibuje 50.000 cubos con instanciado, cada uno con un color y una velocidad de giro distintos calculados en el *shader*.
5. Amplíe el ejemplo completo con sombras (*shadow mapping*), una cámara en primera persona que camine sobre el terreno y un efecto de posprocesado (viñeteado) con un *framebuffer*.

---

## 21. Referencias

- PyOpenGL: https://pyopengl.sourceforge.net/
- Documentación de PyOpenGL: https://pyopengl.sourceforge.net/documentation/
- Referencia de OpenGL 4.x: https://registry.khronos.org/OpenGL-Refpages/gl4/
- LearnOpenGL (tutorial de referencia, en C++ fácilmente trasladable): https://learnopengl.com/
- GLFW para Python: https://pypi.org/project/glfw/
- ModernGL (alternativa de alto rendimiento): https://moderngl.readthedocs.io/
- RenderDoc (depurador gráfico): https://renderdoc.org/
