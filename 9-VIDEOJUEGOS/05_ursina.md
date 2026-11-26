# Manual de uso: Ursina

> **Categoría:** Videojuegos y gráficos · **Utilidad:** Crear juegos y entornos 3D
> **Tipo:** Librería externa (requiere instalación con `pip`). Motor construido sobre Panda3D.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera aplicación](#4-primera-aplicación)
5. [Entidades: modelos, posición, color y textura](#5-entidades-modelos-posición-color-y-textura)
6. [Lógica: `update`, `input` y el tiempo](#6-lógica-update-input-y-el-tiempo)
7. [Cámara y controladores](#7-cámara-y-controladores)
8. [Colisiones y *raycasts*](#8-colisiones-y-raycasts)
9. [Animaciones, secuencias e `invoke`](#9-animaciones-secuencias-e-invoke)
10. [Interfaz de usuario](#10-interfaz-de-usuario)
11. [Luces, sombras, cielo y *shaders*](#11-luces-sombras-cielo-y-shaders)
12. [Modelos, terrenos y mallas por código](#12-modelos-terrenos-y-mallas-por-código)
13. [Sonido](#13-sonido)
14. [Organización del proyecto y escenas](#14-organización-del-proyecto-y-escenas)
15. [Rendimiento y distribución](#15-rendimiento-y-distribución)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Ursina** es un motor de juegos para Python que permite crear **juegos y entornos 3D (y 2D) con muy poco código**. Está construido sobre **Panda3D**, del que hereda el rendimiento y la robustez, pero ofrece una API mucho más sencilla: todo es una `Entity` (entidad) a la que se asigna un modelo, un color, una posición y un colisionador; la lógica se escribe en métodos `update()` e `input()`.

**Ventajas:**
- Productividad muy alta: un mundo 3D navegable en menos de 10 líneas.
- Incluye modelos básicos (cubo, esfera, plano…), texturas, cielo, *shaders* y fuentes.
- Prefabricados listos: controlador en primera persona, cámara de editor, botones, paneles, deslizadores, barras de vida, terreno por mapa de alturas.
- Recarga en caliente de código y recursos durante el desarrollo.
- Acceso completo a Panda3D cuando se necesita más control.
- Útil tanto para juegos como para visualizaciones 3D, prototipos y docencia.

**Limitaciones:**
- Menos documentación y comunidad que Pygame o Panda3D; cambia entre versiones.
- Las colisiones están pensadas para casos sencillos (no hay motor de físicas completo integrado).
- Para proyectos grandes conviene conocer Panda3D, porque los problemas avanzados se resuelven a ese nivel.

| Si necesita… | Opción |
|---|---|
| Juego o entorno 3D rápido, prototipos, docencia | **Ursina** |
| Motor 3D completo con control fino | Panda3D |
| Juego 2D clásico | Pygame / Arcade |
| Gráficos a bajo nivel | PyOpenGL |

---

## 2. Instalación e importación

```bash
pip install ursina
```

> Ursina instala Panda3D como dependencia. Requiere una tarjeta gráfica con OpenGL.

Comprobar la instalación:

```bash
python -c "import ursina; print(ursina.__file__)"
pip show ursina                 # Versión
```

Importación habitual:

```python
from ursina import *
```

> En Ursina es habitual `from ursina import *`: importa `Entity`, `color`, `held_keys`, `time`, `camera`, `mouse`, `Vec3`, `Text`, `Button`, etc. En proyectos grandes puede importar solo lo necesario: `from ursina import Ursina, Entity, color`.

Prefabricados (se importan aparte):

```python
from ursina.prefabs.first_person_controller import FirstPersonController
from ursina.prefabs.platformer_controller_2d import PlatformerController2d
from ursina.prefabs.health_bar import HealthBar
from ursina.shaders import lit_with_shadows_shader
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Ursina()` | Crea la aplicación y la ventana. Debe crearse **antes** que cualquier entidad. |
| `app.run()` | Inicia el bucle del juego. |
| `Entity` | Objeto básico: puede tener modelo, textura, color, colisionador, hijos y lógica. |
| `scene` | Raíz del mundo 3D (padre por defecto de las entidades). |
| `camera.ui` | Raíz de la interfaz 2D (padre por defecto de `Text` y `Button`). |
| `update()` | Método (de una entidad o global) que se ejecuta en cada fotograma. |
| `input(key)` | Método que recibe cada pulsación o liberación de tecla/botón. |
| `held_keys` | Diccionario con el estado de las teclas mantenidas (1 pulsada, 0 no). |
| `time.dt` | Segundos desde el fotograma anterior (*delta time*). |
| Colisionador | Forma invisible para detectar colisiones, clics y *raycasts* (`'box'`, `'sphere'`, `'mesh'`). |
| Prefabricado (*prefab*) | Entidad lista para usar: controladores, botones, paneles… |

**Sistema de coordenadas:** **Y hacia arriba**, **Z hacia delante** (alejándose de la cámara por defecto) y X hacia la derecha. Las rotaciones se expresan en grados.

```
        Y (arriba)
        │   Z (delante)
        │  ╱
        │ ╱
        └──────── X (derecha)
```

**Interfaz 2D:** en `camera.ui`, el centro de la pantalla es `(0, 0)`; el alto visible va de -0,5 a 0,5 y el ancho depende de la proporción de la ventana (`window.aspect_ratio`). Esquinas útiles: `window.top_left`, `window.top_right`, `window.bottom_left`, `window.bottom_right`.

---

## 4. Primera aplicación

```python
from ursina import *

app = Ursina(title="Mi juego", borderless=False)

cubo = Entity(model="cube", color=color.orange, texture="white_cube", scale=2, collider="box")

def update():                                   # Función global: se llama en cada fotograma
    cubo.rotation_y += 60 * time.dt
    cubo.x += (held_keys["d"] - held_keys["a"]) * 4 * time.dt

def input(key):
    if key == "space":
        cubo.color = color.random_color()
    if key == "escape":
        application.quit()

EditorCamera()                                  # Cámara: arrastrar con el botón derecho para girar
app.run()
```

Opciones de `Ursina()` y de la ventana:

```python
app = Ursina(title="Juego", fullscreen=False, borderless=False, development_mode=True, vsync=True,
             size=(1280, 720))
window.color = color.black                    # Color de fondo
window.fps_counter.enabled = True             # Contador de FPS
window.exit_button.visible = False            # Botón de cerrar (ventanas sin borde)
window.fullscreen = True                      # Cambiar en ejecución
window.borderless = False
```

> Con `development_mode=True` (por defecto) Ursina muestra contadores y permite recargar código con **F5**. Desactívelo en la versión final.

---

## 5. Entidades: modelos, posición, color y textura

### 5.1 Crear entidades

```python
suelo = Entity(model="plane", scale=(50, 1, 50), texture="grass", texture_scale=(25, 25), collider="box")
caja = Entity(model="cube", position=(2, 0.5, 5), color=color.azure, texture="white_cube", collider="box")
bola = Entity(model="sphere", x=-2, y=1, scale=1.5, color=color.red)
diamante = Entity(model="diamond", y=2, color=color.gold)
```

Modelos incluidos: `"cube"`, `"sphere"`, `"plane"`, `"quad"` (plano vertical), `"circle"`, `"diamond"`, `"icosphere"`, `"wireframe_cube"`, `"arrow"`, `"sky_dome"`. Procedurales: `Cylinder()`, `Cone()`, `Pipe()`, `Grid()`, `Circle()`, `Quad(radius=…)`, `Plane(subdivisions=…)`.

Texturas incluidas: `"white_cube"`, `"brick"`, `"grass"`, `"shore"`, `"noise"`, `"perlin_noise"`, `"sky_default"`, `"sky_sunset"`, `"radial_gradient"`, `"vignette"`, `"circle"`, `"heightmap_1"`.

### 5.2 Transformaciones

```python
caja.position = (0, 1, 3)              # o Vec3(0, 1, 3)
caja.x += 1; caja.y = 2; caja.z -= 0.5
caja.rotation = (0, 45, 0)             # Grados (x, y, z)
caja.rotation_y += 90
caja.scale = 2                         # Uniforme
caja.scale = (1, 3, 1)                 # Por ejes
caja.scale_y = 0.5
caja.look_at(jugador)                  # Orientar hacia otra entidad o punto
caja.look_at_2d(jugador.position, "y") # Girar solo en el eje Y
caja.forward, caja.right, caja.up      # Vectores de dirección de la entidad
caja.world_position                    # Posición absoluta (si tiene padre)
caja.origin_y = -0.5                   # Punto de pivote (apoya el cubo sobre el suelo)
```

### 5.3 Jerarquía

```python
coche = Entity()
carroceria = Entity(parent=coche, model="cube", scale=(2, 0.6, 4), color=color.red)
rueda = Entity(parent=coche, model=Cylinder(12), position=(1, -0.3, 1.2), rotation_z=90, scale=(0.6, 0.3, 0.6))
coche.position = (0, 0.5, 10)          # Mueve todo el conjunto
rueda.world_parent = scene             # Cambiar de padre manteniendo la posición en el mundo
```

### 5.4 Color

```python
color.red, color.azure, color.lime, color.orange, color.gold, color.violet, color.gray, color.black, color.white
color.rgb(0.12, 0.31, 0.47)            # Componentes 0–1
color.rgb32(31, 78, 120)               # Componentes 0–255
color.hex("#1F4E78")
color.hsv(210, 0.7, 0.5)               # Tono (0–360), saturación, valor
color.random_color()
color.red.tint(-0.3)                   # Oscurecer / aclarar
color.azure.tint(0.2)
caja.color = color.rgba(1, 1, 1, 0.5)  # Semitransparente
caja.alpha = 0.5
```

### 5.5 Visibilidad y eliminación

```python
caja.enabled = False                   # Desactiva: no se dibuja ni ejecuta update
caja.visible = False                   # Oculta, pero sigue activa
destroy(caja)                          # Elimina
destroy(caja, delay=2)                 # Elimina dentro de 2 segundos
copia = duplicate(caja, x=5)           # Clonar
```

---

## 6. Lógica: `update`, `input` y el tiempo

### 6.1 Clases de entidades con comportamiento

```python
class Enemigo(Entity):
    def __init__(self, objetivo, **kwargs):
        super().__init__(model="cube", color=color.red, collider="box", **kwargs)
        self.objetivo = objetivo
        self.velocidad = 3

    def update(self):                                     # Se llama solo, en cada fotograma
        self.look_at_2d(self.objetivo.position, "y")
        self.position += self.forward * self.velocidad * time.dt

    def input(self, key):                                 # Recibe todas las teclas
        if key == "p":
            self.velocidad = 0
```

> Las entidades con método `update` o `input` los ejecutan automáticamente mientras estén activas (`enabled = True`).

### 6.2 Teclado y ratón

```python
def input(key):
    if key == "space": saltar()                 # Al pulsar
    if key == "space up": fin_salto()           # Al soltar
    if key == "left mouse down": disparar()
    if key == "right mouse down": apuntar()
    if key == "scroll up": camera.fov -= 5
    if key == "f11": window.fullscreen = not window.fullscreen

def update():
    direccion = Vec3(held_keys["d"] - held_keys["a"], 0, held_keys["w"] - held_keys["s"]).normalized()
    jugador.position += direccion * 5 * time.dt
```

Nombres de teclas: letras (`"a"`), `"space"`, `"enter"`, `"escape"`, `"tab"`, `"shift"`, `"control"`, `"up arrow"`, `"down arrow"`, `"left arrow"`, `"right arrow"`, `"f1"`…; ratón: `"left mouse down"`, `"left mouse up"`, `"right mouse down"`, `"middle mouse down"`, `"scroll up"`, `"scroll down"`; mando: `"gamepad a"`, `"gamepad left stick x"` (en `held_keys`).

| Objeto `mouse` | Uso |
|---|---|
| `mouse.position` | Posición en coordenadas de la UI. |
| `mouse.hovered_entity` | Entidad (con colisionador) bajo el cursor. |
| `mouse.world_point` | Punto 3D bajo el cursor. |
| `mouse.velocity` | Movimiento del ratón en el fotograma. |
| `mouse.locked = True` | Captura el cursor (cámara en primera persona). |
| `mouse.visible = False` | Ocultar el cursor. |

Clic sobre entidades:

```python
caja = Entity(model="cube", collider="box")
caja.on_click = lambda: setattr(caja, "color", color.random_color())

def on_mouse_enter(): caja.scale = 1.1
caja.on_mouse_enter = on_mouse_enter
caja.on_mouse_exit = lambda: setattr(caja, "scale", 1)
```

### 6.3 Tiempo

| Elemento | Uso |
|---|---|
| `time.dt` | Segundos desde el último fotograma (úselo siempre al mover). |
| `time.time()` | Reloj (módulo `time` de Python; Ursina reexporta `time` con `dt`). |
| `application.time_scale = 0.5` | Cámara lenta global. |
| `application.pause()` / `application.resume()` | Pausar el juego (las entidades con `ignore_paused=True` siguen funcionando). |
| `application.paused` | ¿Está en pausa? |

---

## 7. Cámara y controladores

### 7.1 Cámara manual

```python
camera.position = (0, 15, -20)
camera.rotation_x = 35
camera.fov = 70
camera.orthographic = True; camera.fov = 20       # Proyección ortográfica (juegos 2D/isométricos)
camera.parent = jugador                            # Cámara que sigue a una entidad
```

### 7.2 Cámara de editor

```python
EditorCamera()            # Botón derecho: girar · rueda: zoom · botón central: desplazar · F: centrar
```

### 7.3 Controlador en primera persona

```python
from ursina.prefabs.first_person_controller import FirstPersonController

suelo = Entity(model="plane", scale=64, texture="grass", texture_scale=(16, 16), collider="box")
jugador = FirstPersonController(speed=6, jump_height=2, gravity=1, mouse_sensitivity=Vec2(40, 40))
jugador.cursor.color = color.white
```

- WASD para moverse, ratón para mirar, espacio para saltar.
- Necesita que el suelo y los obstáculos tengan **colisionador**.
- `jugador.enabled = False` libera el ratón (por ejemplo, en un menú) y `mouse.locked = False` muestra el cursor.

### 7.4 Cámara en tercera persona sencilla

```python
camera.parent = jugador
camera.position = (0, 6, -12)
camera.rotation_x = 20
```

Con suavizado:

```python
def update():
    objetivo = jugador.world_position + jugador.back * 10 + Vec3(0, 5, 0)
    camera.world_position = lerp(camera.world_position, objetivo, min(1, 5 * time.dt))
    camera.look_at(jugador.world_position + Vec3(0, 1, 0))
```

### 7.5 Plataformas 2D

```python
from ursina.prefabs.platformer_controller_2d import PlatformerController2d

camera.orthographic = True; camera.fov = 20
suelo = Entity(model="quad", scale=(40, 1), y=-3, collider="box", color=color.gray)
jugador = PlatformerController2d(y=0, color=color.orange, walk_speed=6, jump_height=3)
camera.add_script(SmoothFollow(target=jugador, offset=(0, 1, -30), speed=4))
```

---

## 8. Colisiones y *raycasts*

### 8.1 Colisionadores

```python
Entity(model="cube", collider="box")              # Caja alineada a la entidad (rápido)
Entity(model="sphere", collider="sphere")         # Esfera
Entity(model="mi_modelo", collider="mesh")        # Forma exacta (lento; solo estático)
Entity(model="cube", collider=BoxCollider(entity, center=Vec3(0, 0.5, 0), size=Vec3(1, 2, 1)))
entidad.collider = None                           # Quitar
entidad.collider.visible = True                   # Ver para depurar
```

### 8.2 Intersecciones

```python
golpe = jugador.intersects()                      # Con cualquier entidad con colisionador
if golpe.hit:
    print("Toca a", golpe.entity, "en", golpe.world_point)

golpe = jugador.intersects(ignore=(suelo,))
for entidad in golpe.entities: ...
```

> `intersects()` comprueba en ese momento; llámelo en `update`. Ambos objetos necesitan colisionador.

### 8.3 *Raycast* y *boxcast*

Lanzan un rayo (o una caja) en una dirección y devuelven el primer impacto:

```python
golpe = raycast(origin=jugador.world_position + Vec3(0, 1, 0), direction=jugador.forward,
                distance=20, ignore=(jugador,), debug=False)
if golpe.hit:
    print(golpe.entity, golpe.distance, golpe.world_point, golpe.world_normal)

# ¿Hay suelo bajo el jugador?
en_suelo = raycast(jugador.world_position + Vec3(0, 0.1, 0), Vec3(0, -1, 0), distance=0.2, ignore=(jugador,)).hit

# Movimiento que no atraviesa paredes
direccion = Vec3(held_keys["d"] - held_keys["a"], 0, held_keys["w"] - held_keys["s"]).normalized()
if not boxcast(jugador.position + Vec3(0, 0.5, 0), direction=direccion, distance=0.6,
               thickness=(0.8, 0.8), ignore=(jugador,)).hit:
    jugador.position += direccion * 5 * time.dt
```

### 8.4 Distancias

```python
distance(a, b)            # Distancia 3D entre entidades o puntos
distance_xz(a, b)         # Ignorando la altura
```

> Para muchos objetos (monedas, enemigos), comprobar `distance()` es más barato que usar colisionadores.

---

## 9. Animaciones, secuencias e `invoke`

### 9.1 Animar propiedades

```python
caja.animate_position((0, 3, 0), duration=1, curve=curve.out_bounce)
caja.animate_rotation((0, 360, 0), duration=2, curve=curve.linear, loop=True)
caja.animate_scale(2, duration=0.3, curve=curve.out_expo)
caja.animate_color(color.red, duration=0.5)
caja.animate("alpha", 0, duration=1)                 # Cualquier atributo numérico
caja.animate_y(caja.y + 1, duration=0.5, loop=True, curve=curve.in_out_sine)
caja.fade_out(duration=0.5); caja.fade_in()
caja.shake(duration=0.3, magnitude=0.2)
caja.blink(color.red)
```

Curvas: `curve.linear`, `in_sine`, `out_sine`, `in_out_sine`, `in_expo`, `out_expo`, `out_bounce`, `in_out_back`, `out_elastic`…

### 9.2 Secuencias

```python
s = Sequence(
    Func(setattr, puerta, "color", color.green),
    Wait(0.5),
    Func(puerta.animate_y, 4, duration=1),
    Wait(1),
    Func(print, "Puerta abierta"),
    loop=False,
)
s.start()
s.pause(); s.resume(); s.kill()
```

### 9.3 Ejecutar con retardo

```python
invoke(aparecer_enemigo, delay=3)                        # Una vez, dentro de 3 s
invoke(Func(print, "hola"), delay=1)
```

### 9.4 Animaciones de sprites y modelos

```python
Animation("explosion", position=(0, 2, 0), fps=24, loop=False)    # Carga explosion_0000.png, _0001…
FrameAnimation3d("personaje_correr_", fps=12)                      # Secuencia de modelos
# Modelos glTF con esqueleto: usar Actor de Panda3D
from direct.actor.Actor import Actor
actor = Actor("personaje.glb"); actor.reparentTo(Entity(scale=0.5)); actor.loop("caminar")
```

---

## 10. Interfaz de usuario

### 10.1 Texto

```python
marcador = Text("Puntos: 0", position=window.top_left + Vec2(0.03, -0.03), origin=(-0.5, 0.5), scale=1.5)
marcador.text = "Puntos: 10"
Text("<gold>¡Victoria!<default> Pulse Enter", origin=(0, 0), scale=2, background=True)  # Etiquetas de color
Text.default_font = "VeraMono.ttf"
print_on_screen("Guardado", position=(0, 0.3), duration=2)       # Mensaje temporal
```

### 10.2 Botones y paneles

```python
boton = Button(text="Jugar", color=color.azure, scale=(0.3, 0.08), y=0.1, text_size=1.2)
boton.on_click = empezar
boton.tooltip = Tooltip("Empieza una partida nueva")

from ursina.prefabs.window_panel import WindowPanel   # (también disponible directamente con from ursina import *)
panel = WindowPanel(title="Opciones", content=(
    Text("Volumen"),
    Slider(0, 1, default=0.7, dynamic=True, on_value_changed=lambda: None),
    Button(text="Cerrar", color=color.red, on_click=lambda: setattr(panel, "enabled", False)),
), popup=True)

campo = InputField(default_value="Jugador", max_lines=1, y=-0.2)
nombre = campo.text
```

Otros prefabricados de interfaz: `ButtonList`, `ButtonGroup`, `Draggable`, `HealthBar`, `Panel`, `RadialMenu`, `DropdownMenu`, `FileBrowser`.

### 10.3 Barra de vida

```python
from ursina.prefabs.health_bar import HealthBar
vida = HealthBar(max_value=100, value=100, position=window.top_left + Vec2(0.03, -0.08), bar_color=color.lime)
vida.value -= 10
```

### 10.4 Menús que pausan el juego

```python
menu = Entity(parent=camera.ui, enabled=False, ignore_paused=True)
Entity(parent=menu, model="quad", scale=(3, 2), color=color.black66)
Button(parent=menu, text="Continuar", scale=(0.3, 0.08), on_click=lambda: alternar_pausa())

def alternar_pausa():
    menu.enabled = not menu.enabled
    application.paused = menu.enabled
    mouse.locked = not menu.enabled
```

> Los elementos que deben funcionar durante la pausa necesitan `ignore_paused=True`.

---

## 11. Luces, sombras, cielo y *shaders*

### 11.1 Luces

```python
sol = DirectionalLight(shadows=True)
sol.look_at(Vec3(1, -1, -1))
AmbientLight(color=color.rgba(0.4, 0.4, 0.5, 1))
PointLight(parent=farola, y=3, color=color.orange)
```

Ajustar la zona que cubren las sombras del sol:

```python
sol.shadow_map_resolution = Vec2(2048, 2048)
sol.update_bounds(entity=scene)              # Ajustar a la escena (llamar tras crear el nivel)
```

### 11.2 *Shaders*

```python
from ursina.shaders import lit_with_shadows_shader, basic_lighting_shader, unlit_shader

Entity.default_shader = lit_with_shadows_shader       # Para todas las entidades creadas después
caja = Entity(model="cube", shader=lit_with_shadows_shader)
```

| *Shader* | Efecto |
|---|---|
| `unlit_shader` | Sin iluminación (color plano). Por defecto en la UI. |
| `basic_lighting_shader` | Iluminación sencilla sin luces de escena. |
| `lit_with_shadows_shader` | Responde a luces y recibe/proyecta sombras. |
| `normals_shader`, `matcap_shader`, `triplanar_shader` | Depuración y efectos. |

*Shader* propio:

```python
from ursina import Shader
mi_shader = Shader(language=Shader.GLSL, vertex="""…""", fragment="""…""")
entidad.shader = mi_shader
entidad.set_shader_input("tiempo", 0)
```

### 11.3 Cielo, niebla y fondo

```python
Sky()                                       # Cielo por defecto
Sky(texture="sky_sunset")
scene.fog_density = 0.02                    # Niebla exponencial
scene.fog_color = color.rgb(0.6, 0.7, 0.8)
window.color = color.rgb(0.6, 0.7, 0.8)     # Mismo color de fondo
```

---

## 12. Modelos, terrenos y mallas por código

### 12.1 Cargar modelos

```python
Entity(model="modelos/casa")                  # Busca .bam, .glb/.gltf, .obj, .blend… en la carpeta del proyecto
Entity(model="modelos/arbol.obj", texture="modelos/arbol.png", scale=0.5)
```

> Ursina puede importar archivos `.blend` si Blender está instalado (los convierte automáticamente). Para distribuir, exporte a `.glb` o `.bam`.

### 12.2 Terreno por mapa de alturas

```python
terreno = Entity(model=Terrain("heightmap_1", skip=8), scale=(80, 6, 80), texture="grass",
                 texture_scale=(10, 10), collider="mesh")
```

### 12.3 Mallas propias

```python
verts = [(0, 0, 0), (1, 0, 0), (0.5, 1, 0)]
tris = [(0, 1, 2)]
Entity(model=Mesh(vertices=verts, triangles=tris, colors=[color.red, color.green, color.blue]), double_sided=True)

Entity(model=Mesh(vertices=puntos, mode="line", thickness=3), color=color.yellow)    # Líneas
Entity(model=Mesh(vertices=puntos, mode="point", thickness=0.1))                     # Nube de puntos
```

### 12.4 Combinar entidades estáticas

Muchas entidades estáticas (paredes, árboles) se dibujan más rápido combinadas en una sola malla:

```python
nivel = Entity()
for x in range(50):
    Entity(parent=nivel, model="cube", x=x, texture="brick")
nivel.combine()                               # Una sola malla
nivel.texture = "brick"
```

---

## 13. Sonido

```python
salto = Audio("sonidos/salto.wav", autoplay=False, volume=0.6)
salto.play()
musica = Audio("sonidos/musica.ogg", loop=True, autoplay=True, volume=0.3)
musica.fade_out(duration=2)
musica.stop()
Audio("sine", pitch=1.5, volume=0.2)          # Sonidos incluidos: "sine", "square", "triangle", "noise"
```

| Parámetro | Efecto |
|---|---|
| `autoplay` | Reproducir al crear (por defecto `True`). |
| `loop` | Repetir. |
| `volume`, `pitch`, `balance` | Volumen, tono, panorama. |
| `auto_destroy=True` | Eliminar la entidad de audio al terminar. |

> Si no hay dispositivo de sonido, Panda3D lo indica en la consola y el juego sigue funcionando sin audio.

---

## 14. Organización del proyecto y escenas

```
mi_juego/
├── main.py                 # Crea Ursina(), carga el menú y llama a app.run()
├── configuracion.py
├── entidades/              # jugador.py, enemigo.py, moneda.py
├── escenas/                # menu.py, nivel_1.py
├── interfaz/               # hud.py, pausa.py
├── modelos/  texturas/  sonidos/
```

Cambiar de escena (nivel): agrupe las entidades del nivel bajo un padre y destrúyalo al salir.

```python
class Nivel(Entity):
    def __init__(self, numero):
        super().__init__()
        self.suelo = Entity(parent=self, model="plane", scale=50, collider="box")
        ...

nivel = Nivel(1)
destroy(nivel)              # Elimina también todos sus hijos
nivel = Nivel(2)
```

Guardar datos:

```python
import json
from pathlib import Path
RUTA = Path.home() / ".mi_juego.json"
datos = json.loads(RUTA.read_text()) if RUTA.exists() else {"record": 0}
RUTA.write_text(json.dumps(datos))
```

---

## 15. Rendimiento y distribución

### 15.1 Rendimiento

| Técnica | Cómo |
|---|---|
| Reducir entidades | Combine geometría estática con `combine()`. |
| Colisionadores simples | `"box"`/`"sphere"`; use `"mesh"` solo en terrenos estáticos. |
| Menos comprobaciones | Use `distance()` en lugar de colisionadores para objetos simples. |
| Desactivar lo lejano | `entidad.enabled = False` fuera de la vista o de la zona de juego. |
| Sombras | Limite la zona (`update_bounds`) y la resolución del mapa de sombras. |
| Medir | `window.fps_counter.enabled = True`; `window.entity_counter`, `window.collider_counter`. |

### 15.2 Distribución

Ursina incluye una herramienta de empaquetado:

```bash
python -m ursina.build                      # Crea la carpeta build/ con el juego y Python incluido (Windows)
```

Alternativas: el `build_apps` de Panda3D (multiplataforma) o PyInstaller:

```bash
pyinstaller --noconfirm --windowed --collect-all ursina --collect-all panda3d main.py
```

> Antes de distribuir: `Ursina(development_mode=False)` y convierta los modelos a `.bam`/`.glb` (no dependa de Blender en el equipo del usuario).

---

## 16. Ejemplo práctico completo

**«Torre de cristales»**: juego de plataformas 3D en primera persona. El jugador debe recoger los cristales repartidos por una torre de plataformas flotantes (algunas se mueven) antes de que acabe el tiempo, evitando caer a la lava. Incluye generación procedural de la torre, plataformas móviles que arrastran al jugador, cristales animados, potenciador de salto, lava con brillo pulsante, luz con sombras, cielo y niebla, marcador y temporizador, menú principal, pausa (Esc), pantallas de victoria y derrota, y récord del mejor tiempo guardado. No necesita archivos externos.

```python
"""
Torre de cristales: plataformas 3D en primera persona con Ursina (sin recursos externos).
Requiere: pip install ursina
Controles: WASD mover · ratón mirar · Espacio saltar · Esc pausa
"""
import json
import math
import random
from pathlib import Path

from ursina import *
from ursina.prefabs.first_person_controller import FirstPersonController
from ursina.shaders import basic_lighting_shader, lit_with_shadows_shader, unlit_shader

RUTA_RECORD = Path.home() / ".torre_cristales.json"
TIEMPO_LIMITE = 180
NUM_CRISTALES = 12
ALTURA_LAVA = -2
SOMBRAS = True              # Póngalo a False en equipos sin tarjeta gráfica o en máquinas virtuales

app = Ursina(title="Torre de cristales", development_mode=False, vsync=True)
window.color = color.rgb(0.55, 0.65, 0.8)
window.exit_button.visible = False
Entity.default_shader = lit_with_shadows_shader if SOMBRAS else basic_lighting_shader


def cargar_record():
    try:
        return json.loads(RUTA_RECORD.read_text(encoding="utf-8")).get("mejor_tiempo")
    except (OSError, ValueError):
        return None


def guardar_record(segundos: float) -> None:
    try:
        RUTA_RECORD.write_text(json.dumps({"mejor_tiempo": round(segundos, 1)}), encoding="utf-8")
    except OSError:
        pass


# ------------------------------------------------------------------ Entidades del mundo
class Plataforma(Entity):
    def __init__(self, posicion, tamano, movil=False, **kwargs):
        super().__init__(model="cube", position=posicion, scale=(tamano, 0.6, tamano), texture="white_cube",
                         collider="box", color=color.hsv(30 + posicion[1] * 6, 0.35, 0.75), **kwargs)
        self.movil = movil
        self.origen = Vec3(*posicion)
        self.fase = random.uniform(0, math.tau)
        self.amplitud = random.uniform(2.5, 4.5)
        self.eje = random.choice((Vec3(1, 0, 0), Vec3(0, 0, 1)))
        self.desplazamiento = Vec3(0, 0, 0)
        if movil:
            self.color = color.azure.tint(-0.1)

    def update(self):
        if not self.movil or application.paused:
            return
        anterior = self.position
        self.position = self.origen + self.eje * math.sin(time.time() * 0.8 + self.fase) * self.amplitud
        self.desplazamiento = self.position - anterior


class Cristal(Entity):
    def __init__(self, posicion, **kwargs):
        super().__init__(model="diamond", position=posicion, scale=0.7, color=color.cyan, **kwargs)
        self.base_y = posicion[1]
        self.animate_rotation_y(360, duration=2.5, curve=curve.linear, loop=True)

    def update(self):
        self.y = self.base_y + math.sin(time.time() * 2.5 + self.x) * 0.15


class Potenciador(Entity):
    def __init__(self, posicion, **kwargs):
        super().__init__(model="sphere", position=posicion, scale=0.5, color=color.lime, **kwargs)
        self.animate_scale(0.65, duration=0.6, curve=curve.in_out_sine, loop=True)


# ------------------------------------------------------------------ Juego
class Juego(Entity):
    def __init__(self):
        super().__init__(ignore_paused=True)
        self.mundo = None
        self.jugador = None
        self.estado = "menu"
        self.record = cargar_record()

        Sky(texture="sky_sunset")
        scene.fog_density = 0.012
        scene.fog_color = color.rgb(0.55, 0.65, 0.8)
        self.sol = DirectionalLight(shadows=SOMBRAS)
        self.sol.look_at(Vec3(1, -1.6, 0.8))
        AmbientLight(color=color.rgba(0.45, 0.45, 0.55, 1))

        self.lava = Entity(model="plane", scale=300, y=ALTURA_LAVA, color=color.orange, texture="perlin_noise",
                           texture_scale=(12, 12), shader=unlit_shader)   # La lava no recibe luz ni sombras
        self.crear_interfaz()
        self.construir_torre()                          # La torre sirve de fondo del menú
        self.mostrar_menu()

    # ---------------------------------------------------------------- Interfaz
    def crear_interfaz(self):
        self.hud = Entity(parent=camera.ui, enabled=False)
        self.txt_cristales = Text(parent=self.hud, position=window.top_left + Vec2(0.03, -0.03), scale=1.6)
        self.txt_tiempo = Text(parent=self.hud, position=window.top_right + Vec2(-0.03, -0.03),
                               origin=(0.5, 0.5), scale=1.6)
        self.txt_aviso = Text(parent=self.hud, y=0.3, origin=(0, 0), scale=1.4, color=color.yellow)

        self.panel = Entity(parent=camera.ui, ignore_paused=True)
        Entity(parent=self.panel, model="quad", scale=(0.9, 0.62), color=color.black66, z=1)
        self.txt_titulo = Text(parent=self.panel, y=0.2, origin=(0, 0), scale=2.6)
        self.txt_detalle = Text(parent=self.panel, y=0.07, origin=(0, 0), scale=1.1, color=color.light_gray)
        self.boton = Button(parent=self.panel, text="Jugar", y=-0.1, scale=(0.32, 0.08), color=color.azure.tint(-0.2),
                            ignore_paused=True, on_click=self.al_pulsar_boton)
        self.boton_salir = Button(parent=self.panel, text="Salir", y=-0.2, scale=(0.32, 0.07),
                                  color=color.dark_gray, ignore_paused=True, on_click=application.quit)

    def mostrar_panel(self, titulo, detalle, texto_boton):
        self.txt_titulo.text = titulo
        self.txt_detalle.text = detalle
        self.boton.text = texto_boton
        self.panel.enabled = True
        mouse.locked = False
        mouse.visible = True
        if self.jugador:
            self.jugador.enabled = False

    def mostrar_menu(self):
        self.estado = "menu"
        record = f"Mejor tiempo: {self.record:.1f} s" if self.record else "Sin récord todavía"
        self.mostrar_panel("Torre de cristales",
                           f"Recoge los {NUM_CRISTALES} cristales sin caer a la lava\n"
                           f"WASD mover · ratón mirar · Espacio saltar · Esc pausa\n{record}", "Jugar")
        camera.position, camera.rotation = (0, 22, -55), (12, 0, 0)

    def al_pulsar_boton(self):
        if self.estado in ("menu", "fin"):
            self.empezar()
        elif self.estado == "pausa":
            self.alternar_pausa()

    # ---------------------------------------------------------------- Partida
    def construir_torre(self):
        if self.mundo:
            destroy(self.mundo)
        self.mundo = Entity()
        self.plataformas, self.cristales, self.potenciadores = [], [], []
        random.seed()
        inicio = Plataforma((0, 0, 0), 8, parent=self.mundo)
        self.plataformas.append(inicio)
        posicion = Vec3(0, 0, 0)
        for i in range(1, 34):
            angulo = i * 0.62 + random.uniform(-0.3, 0.3)
            radio = 7 + 5 * math.sin(i * 0.4)
            posicion = Vec3(math.cos(angulo) * radio, i * 1.25, math.sin(angulo) * radio)
            movil = i % 5 == 0
            plataforma = Plataforma(tuple(posicion), random.uniform(2.6, 3.6), movil=movil, parent=self.mundo)
            self.plataformas.append(plataforma)
        # Cristales sobre plataformas fijas, repartidos en altura
        fijas = [p for p in self.plataformas[2:] if not p.movil]
        for plataforma in sorted(random.sample(fijas, NUM_CRISTALES), key=lambda p: p.y):
            self.cristales.append(Cristal(plataforma.position + Vec3(0, 1.3, 0), parent=self.mundo))
        for plataforma in random.sample(fijas, 3):
            self.potenciadores.append(Potenciador(plataforma.position + Vec3(0.8, 1, 0.8), parent=self.mundo))
        # Faro en la cima: indica el final de la torre
        cima = self.plataformas[-1]
        Entity(parent=self.mundo, model="cube", position=cima.position + Vec3(0, 2, 0), scale=(0.6, 3.4, 0.6),
               color=color.gold, texture="white_cube", collider="box")
        if SOMBRAS:
            self.sol.update_bounds(entity=self.mundo)

    def empezar(self):
        self.construir_torre()
        if self.jugador:
            destroy(self.jugador)
        self.jugador = FirstPersonController(position=(0, 2, 0), speed=7, jump_height=2.2, gravity=1,
                                             mouse_sensitivity=Vec2(50, 50))
        self.jugador.cursor.color = color.white
        camera.fov = 85
        self.recogidos, self.tiempo, self.salto_extra = 0, 0.0, 0.0
        self.ultima_plataforma = self.plataformas[0]
        self.estado = "jugando"
        self.panel.enabled = False
        self.hud.enabled = True
        mouse.locked = True
        application.paused = False
        self.avisar("¡Sube y recoge los cristales!")

    def alternar_pausa(self):
        if self.estado == "jugando":
            self.estado = "pausa"
            application.paused = True
            self.mostrar_panel("Pausa", "Pulse Esc o el botón para continuar", "Continuar")
        elif self.estado == "pausa":
            self.estado = "jugando"
            application.paused = False
            self.panel.enabled = False
            self.jugador.enabled = True
            mouse.locked = True

    def terminar(self, victoria: bool):
        self.estado = "fin"
        application.paused = False
        self.hud.enabled = False
        if victoria:
            nuevo = self.record is None or self.tiempo < self.record
            if nuevo:
                self.record = self.tiempo
                guardar_record(self.tiempo)
            detalle = f"Tiempo: {self.tiempo:.1f} s" + ("  ·  ¡Nuevo récord!" if nuevo else
                                                         f"  ·  Récord: {self.record:.1f} s")
            self.mostrar_panel("¡Torre conquistada!", detalle, "Jugar otra vez")
        else:
            self.mostrar_panel("Fin de la partida", f"Cristales: {self.recogidos}/{NUM_CRISTALES}", "Reintentar")

    def avisar(self, texto):
        self.txt_aviso.text = texto
        self.txt_aviso.alpha = 1
        self.txt_aviso.animate("alpha", 0, duration=2.5, delay=1)

    # ---------------------------------------------------------------- Bucle
    def update(self):
        self.lava.color = color.hsv(20, 1, 0.75 + 0.25 * math.sin(time.time() * 2))
        if self.estado != "jugando" or application.paused:
            return
        self.tiempo += time.dt
        restante = TIEMPO_LIMITE - self.tiempo
        jugador = self.jugador

        # Arrastre de plataformas móviles: el jugador se mueve con la plataforma que pisa
        suelo = raycast(jugador.world_position + Vec3(0, 0.1, 0), Vec3(0, -1, 0), distance=0.3, ignore=(jugador,))
        if suelo.hit and isinstance(suelo.entity, Plataforma):
            self.ultima_plataforma = suelo.entity
            if suelo.entity.movil:
                jugador.position += suelo.entity.desplazamiento

        for cristal in self.cristales[:]:
            if distance(cristal, jugador.position + Vec3(0, 1, 0)) < 1.4:
                self.cristales.remove(cristal)
                self.recogidos += 1
                cristal.animate_scale(0, duration=0.25, curve=curve.in_back)
                destroy(cristal, delay=0.3)
                self.avisar(f"¡Cristal {self.recogidos}/{NUM_CRISTALES}!")

        for potenciador in self.potenciadores[:]:
            if distance(potenciador, jugador.position + Vec3(0, 1, 0)) < 1.2:
                self.potenciadores.remove(potenciador)
                destroy(potenciador)
                self.salto_extra = 10.0
                jugador.jump_height = 4
                self.avisar("¡Supersalto durante 10 s!")
        if self.salto_extra:
            self.salto_extra = max(0.0, self.salto_extra - time.dt)
            if self.salto_extra == 0:
                jugador.jump_height = 2.2

        if jugador.y < ALTURA_LAVA + 0.5:                     # Cae a la lava: vuelve a la última plataforma
            jugador.position = self.ultima_plataforma.world_position + Vec3(0, 2, 0)
            self.tiempo += 10
            camera.shake(duration=0.3, magnitude=0.5)
            self.avisar("¡Lava! +10 s de penalización")

        self.txt_cristales.text = f"Cristales: {self.recogidos}/{NUM_CRISTALES}" + \
                                  ("   Supersalto" if self.salto_extra else "")
        self.txt_tiempo.text = f"{max(0, restante):.0f} s"
        self.txt_tiempo.color = color.red if restante < 20 else color.white
        if self.recogidos == NUM_CRISTALES:
            self.terminar(victoria=True)
        elif restante <= 0:
            self.terminar(victoria=False)

    def input(self, key):
        if key == "escape" and self.estado in ("jugando", "pausa"):
            self.alternar_pausa()


juego = Juego()

if __name__ == "__main__":
    app.run()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `AttributeError` / nada se muestra al crear entidades | Se crearon entidades antes de `Ursina()`. | Cree `app = Ursina()` en primer lugar. |
| La ventana se abre y se cierra | Falta `app.run()`. | Añádalo al final del script. |
| El jugador atraviesa el suelo o cae al vacío | El suelo no tiene colisionador. | `collider="box"` (o `"mesh"` en terrenos). |
| Los clics en entidades no funcionan | Falta colisionador. | Añada `collider="box"` a la entidad. |
| `update` de una entidad no se ejecuta | Entidad desactivada o el método tiene otro nombre/argumentos. | `enabled=True`; `def update(self):` sin parámetros. |
| Velocidad distinta según el ordenador | Movimiento sin `time.dt`. | Multiplique siempre por `time.dt`. |
| El ratón queda atrapado en un menú | `FirstPersonController` activo y `mouse.locked = True`. | `jugador.enabled = False; mouse.locked = False`. |
| Los botones no responden en pausa | La pausa detiene todas las entidades. | `ignore_paused=True` en la interfaz de pausa. |
| No se ven sombras | *Shader* sin iluminación o luz sin sombras. | `DirectionalLight(shadows=True)` y `lit_with_shadows_shader`. |
| El programa se cierra sin mensaje (`Segmentation fault`) al usar `lit_with_shadows_shader` | Controlador gráfico por software o antiguo (máquinas virtuales, escritorio remoto) que no soporta el *shader* de sombras. | Actualice el controlador o use `basic_lighting_shader` y `DirectionalLight(shadows=False)` (constante `SOMBRAS` del ejemplo). |
| Sombras cortadas o borrosas | Zona de sombra mal ajustada. | `sol.update_bounds(entity=...)` y resolución mayor. |
| Juego lento con muchos objetos | Demasiadas entidades o colisionadores `mesh`. | `combine()`, colisionadores simples, `distance()`. |
| Modelo `.blend` no carga en otro equipo | Necesita Blender instalado para convertirlo. | Exporte a `.glb`/`.bam` antes de distribuir. |
| La API de un tutorial no funciona | Cambios entre versiones de Ursina. | Consulte la documentación de la versión instalada (`pip show ursina`). |
| Mensajes de error de audio en consola | No hay dispositivo de sonido. | Son avisos de Panda3D; el juego funciona sin audio. |

---

## 18. Buenas prácticas

- Cree `Ursina()` lo primero y `app.run()` lo último.
- Encapsule el comportamiento en **clases que heredan de `Entity`** con sus propios `update` e `input`.
- Multiplique todo movimiento por `time.dt`.
- Agrupe cada nivel bajo una entidad padre para destruirlo y regenerarlo fácilmente.
- Use colisionadores simples y `distance()` para objetos recogibles.
- Desactive (`enabled = False`) en lugar de destruir lo que vaya a reutilizar (menús, HUD).
- Separe la interfaz (`camera.ui`) del mundo (`scene`) y use `ignore_paused=True` en los menús de pausa.
- Ajuste las sombras a la zona de juego y combine la geometría estática.
- Desactive `development_mode` y convierta los modelos antes de distribuir.
- Cuando necesite algo que Ursina no ofrece, recurra a Panda3D (la entidad es un `NodePath`).

---

## 19. Referencia rápida

```python
from ursina import *
app = Ursina(title="T", development_mode=False)
e = Entity(model="cube"|"sphere"|"plane"|"quad"|"diamond", color=color.azure, texture="white_cube",
           position=(x, y, z), rotation=(rx, ry, rz), scale=s, collider="box"|"sphere"|"mesh", parent=scene)
e.x / y / z / rotation_y / scale / forward / right / look_at(obj) / enabled / visible / color / alpha
destroy(e, delay=0) / duplicate(e) / invoke(f, delay=1) / Sequence(Func(f), Wait(1)).start()
def update(): e.x += held_keys["d"] * 5 * time.dt          def input(key): if key == "space": ...
class Mio(Entity): def update(self): ...   def input(self, key): ...
e.on_click = f;  mouse.hovered_entity / world_point / locked
e.animate_position(v, duration=1, curve=curve.out_bounce, loop=False) / animate_rotation / animate_scale / animate_color / shake / fade_out
e.intersects().hit / raycast(origen, direccion, distance=, ignore=(...)) / boxcast / distance(a, b)
camera.position / rotation / fov / orthographic / parent;  EditorCamera();  FirstPersonController()
Text("t", position=window.top_left, origin=(-0.5, 0.5), scale=2) / Button(text=, on_click=) / WindowPanel / Slider / HealthBar
DirectionalLight(shadows=True).look_at(Vec3(1, -1, 1)); AmbientLight(); Entity.default_shader = lit_with_shadows_shader
Sky(); scene.fog_density = 0.02; Audio("archivo.wav", loop=True); application.paused = True
app.run()
```

---

## 20. Ejercicios propuestos

1. Cree una escena con 100 cubos de colores aleatorios en una rejilla y haga que cada uno salte (`animate_y`) al hacer clic sobre él.
2. Construya un laberinto a partir de una lista de cadenas de texto y recórralo con `FirstPersonController`; muestre el tiempo empleado.
3. Programe un juego estilo *Minecraft* sencillo: colocar bloques con el botón derecho y quitarlos con el izquierdo usando `mouse.hovered_entity` y `mouse.normal`.
4. Diseñe un juego de plataformas 2D con `PlatformerController2d`, monedas y una cámara `SmoothFollow`.
5. Amplíe el ejemplo completo con enemigos que patrullen plataformas, una tabla de mejores tiempos con nombre (`InputField`) y una versión ejecutable con `python -m ursina.build`.

---

## 21. Referencias

- Sitio y documentación oficial: https://www.ursinaengine.org/
- Documentación de la API: https://www.ursinaengine.org/api_reference.html
- Ejemplos: https://www.ursinaengine.org/samples.html
- Repositorio (código, ejemplos y prefabricados): https://github.com/pokepetter/ursina
- Comunidad (Discord y Reddit `r/ursinaengine`): enlazados desde el sitio oficial
- Panda3D (motor subyacente): https://docs.panda3d.org/
