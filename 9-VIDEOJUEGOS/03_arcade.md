# Manual de uso: Arcade

> **Categoría:** Videojuegos y gráficos · **Utilidad:** Desarrollo de videojuegos 2D
> **Tipo:** Librería externa (requiere instalación con `pip`). Basada en OpenGL (mediante pyglet).

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [La ventana y el ciclo del juego](#4-la-ventana-y-el-ciclo-del-juego)
5. [Dibujo de formas y texto](#5-dibujo-de-formas-y-texto)
6. [Sprites y listas de sprites](#6-sprites-y-listas-de-sprites)
7. [Entrada: teclado, ratón y mando](#7-entrada-teclado-ratón-y-mando)
8. [Colisiones](#8-colisiones)
9. [Motores de físicas](#9-motores-de-físicas)
10. [Animación de sprites](#10-animación-de-sprites)
11. [Cámaras](#11-cámaras)
12. [Mapas de casillas (Tiled) y escenas](#12-mapas-de-casillas-tiled-y-escenas)
13. [Vistas, interfaz gráfica (GUI) y sonido](#13-vistas-interfaz-gráfica-gui-y-sonido)
14. [Partículas y efectos](#14-partículas-y-efectos)
15. [Rendimiento y distribución](#15-rendimiento-y-distribución)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Arcade** es una librería moderna para crear **videojuegos 2D** en Python. A diferencia de Pygame, dibuja con la **tarjeta gráfica** (OpenGL), por lo que puede mostrar miles de sprites con buen rendimiento, rotarlos y escalarlos sin coste apreciable. Ofrece una API orientada a objetos y de alto nivel: ventanas y vistas, listas de sprites, cámaras, motores de físicas, carga de mapas de **Tiled**, interfaz gráfica, partículas y sonido.

**Ventajas:**
- Aceleración por GPU: miles de sprites a 60 FPS.
- API clara, con tipado completo y documentación extensa con decenas de ejemplos.
- Motores de físicas listos: movimiento con paredes, plataformas (gravedad, saltos, escaleras) y Pymunk.
- Cámaras con desplazamiento y zoom, y soporte nativo de mapas Tiled.
- Incluye recursos gratuitos (`:resources:`): personajes, casillas, objetos, sonidos y mapas de ejemplo.
- Módulo de interfaz gráfica (botones, etiquetas, deslizadores, diseños) y sistema de vistas.

**Limitaciones:**
- Requiere una tarjeta gráfica compatible con OpenGL 3.3 (prácticamente cualquier equipo moderno).
- Solo 2D (aunque permite *shaders* y efectos).
- Comunidad más pequeña que la de Pygame; algunos tutoriales en internet corresponden a la versión 2.x, con API distinta.

| Si necesita… | Opción |
|---|---|
| Juego 2D moderno con muchos sprites, cámaras, físicas y mapas Tiled | **Arcade** |
| Aprender los fundamentos con control total del bucle | Pygame |
| Juego 3D | Ursina / Panda3D |

> **Versión 3.x:** este manual corresponde a Arcade 3. Respecto a la 2.6 cambian, entre otros, las cámaras (`Camera2D`), el dibujo de rectángulos (`draw_rect_filled` con `XYWH`/`LBWH`) y la interfaz gráfica.

---

## 2. Instalación e importación

```bash
pip install arcade
```

Comprobar la instalación (lista y ejecuta ejemplos incluidos):

```bash
python -m arcade.examples.platform_tutorial.17_views
python -c "import arcade; print(arcade.__version__)"
```

Importaciones habituales:

```python
import arcade
import arcade.gui                    # Interfaz gráfica
import arcade.particles              # Partículas
from arcade.types import Color
```

Recursos incluidos: se cargan con el prefijo `:resources:`.

```python
textura = arcade.load_texture(":resources:images/animated_characters/female_adventurer/femaleAdventurer_idle.png")
sonido = arcade.load_sound(":resources:sounds/coin1.wav")
mapa = arcade.load_tilemap(":resources:tiled_maps/map2_level_1.json", scaling=0.5)
```

| Carpeta | Contenido |
|---|---|
| `:resources:images/animated_characters/` | Personajes con fotogramas de caminar, saltar, trepar. |
| `:resources:images/tiles/` | Casillas de terreno, cajas, ladrillos. |
| `:resources:images/items/` | Monedas, gemas, llaves, banderas. |
| `:resources:images/enemies/` | *Slimes*, abejas, ratones… |
| `:resources:images/space_shooter/` | Naves y láseres. |
| `:resources:sounds/` | Monedas, saltos, golpes, explosiones, *game over*. |
| `:resources:tiled_maps/` | Mapas de ejemplo de Tiled. |

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Window` | La ventana del juego. Recibe eventos y llama a `on_draw` y `on_update`. |
| `View` | Pantalla del juego (menú, partida, fin). La ventana muestra una vista cada vez. |
| `on_draw()` | Método donde se dibuja todo (se llama unas 60 veces por segundo). |
| `on_update(delta_time)` | Lógica del juego; `delta_time` son los segundos desde la última llamada. |
| `Sprite` | Objeto con textura, posición, tamaño, ángulo y velocidad. |
| `SpriteList` | Lista de sprites que se dibuja en la GPU de una vez. **Todo sprite debe estar en una lista para dibujarse.** |
| Textura | Imagen cargada en la tarjeta gráfica. |
| Motor de físicas | Objeto que mueve sprites y resuelve colisiones (paredes, gravedad). |
| `Camera2D` | Define qué parte del mundo se ve (desplazamiento, zoom). |
| `Scene` | Conjunto ordenado de `SpriteList` con nombre (capas), habitualmente creado desde un mapa Tiled. |

**Sistema de coordenadas:** el origen `(0, 0)` está en la **esquina inferior izquierda**; X crece hacia la derecha e **Y hacia arriba** (como en matemáticas). La posición de un sprite es la de su **centro**.

```
  y
  ▲
  │      (200, 150)
  │          ●
  │
(0,0) ──────────► x
```

**Ciclo del juego en Arcade:** no se escribe el bucle; Arcade lo gestiona y llama a los métodos de la ventana o vista:

```
arcade.run() ─┬─► on_update(delta_time)    → lógica
              ├─► on_draw()                → dibujo
              └─► on_key_press(), on_mouse_motion()… → eventos
```

---

## 4. La ventana y el ciclo del juego

### 4.1 Programa mínimo

```python
import arcade

ANCHO, ALTO, TITULO = 1280, 720, "Mi juego"


class MiJuego(arcade.Window):
    def __init__(self):
        super().__init__(ANCHO, ALTO, TITULO)
        self.background_color = arcade.color.DARK_SLATE_BLUE
        self.x = 100

    def on_update(self, delta_time: float):
        self.x += 200 * delta_time                 # 200 píxeles por segundo
        if self.x > ANCHO:
            self.x = 0

    def on_draw(self):
        self.clear()                               # Borrar con el color de fondo
        arcade.draw_circle_filled(self.x, ALTO / 2, 40, arcade.color.YELLOW)


def main():
    MiJuego()
    arcade.run()


if __name__ == "__main__":
    main()
```

### 4.2 Opciones de la ventana

```python
super().__init__(1280, 720, "Título", fullscreen=False, resizable=True, vsync=True,
                 antialiasing=True, update_rate=1 / 60, center_window=True)

self.set_fullscreen(not self.fullscreen)          # Alternar pantalla completa
self.set_mouse_visible(False)
self.width, self.height                            # Tamaño actual
arcade.get_window()                                # Ventana activa (desde cualquier sitio)
```

| Método de la ventana/vista | Cuándo se llama |
|---|---|
| `on_draw()` | En cada fotograma, para dibujar. |
| `on_update(delta_time)` | En cada fotograma, para la lógica (paso variable). |
| `on_fixed_update(delta_time)` | A intervalos fijos (físicas deterministas). |
| `on_key_press(key, modifiers)` / `on_key_release` | Teclas. |
| `on_mouse_press(x, y, button, modifiers)` / `on_mouse_release` / `on_mouse_motion(x, y, dx, dy)` / `on_mouse_scroll` | Ratón. |
| `on_resize(width, height)` | Cambio de tamaño de la ventana. |
| `on_show_view()` / `on_hide_view()` | Al mostrar/ocultar una vista. |

> En `on_draw()` llame siempre primero a `self.clear()`.

---

## 5. Dibujo de formas y texto

### 5.1 Formas primitivas

```python
arcade.draw_circle_filled(400, 300, 50, arcade.color.ORANGE)
arcade.draw_circle_outline(400, 300, 60, arcade.color.WHITE, border_width=3)
arcade.draw_ellipse_filled(600, 300, 120, 60, arcade.color.TEAL)
arcade.draw_rect_filled(arcade.XYWH(200, 200, 100, 60), arcade.color.STEEL_BLUE)      # Centro, ancho, alto
arcade.draw_rect_filled(arcade.LBWH(10, 10, 200, 40), (0, 0, 0, 150))                  # Esquina inferior izq.
arcade.draw_rect_outline(arcade.LRBT(100, 300, 100, 200), arcade.color.WHITE, 2)       # Izq., der., abajo, arriba
arcade.draw_line(0, 0, 400, 300, arcade.color.WHITE, 2)
arcade.draw_polygon_filled([(100, 100), (150, 200), (200, 100)], arcade.color.RED)
arcade.draw_triangle_filled(100, 100, 150, 200, 200, 100, arcade.color.GREEN)
arcade.draw_arc_outline(300, 300, 100, 100, arcade.color.WHITE, 0, 180, 3)
```

| Constructor de rectángulo | Parámetros |
|---|---|
| `arcade.XYWH(x, y, ancho, alto)` | Centro y tamaño. |
| `arcade.LBWH(izq, abajo, ancho, alto)` | Esquina inferior izquierda y tamaño. |
| `arcade.LRBT(izq, der, abajo, arriba)` | Bordes. |

> Las funciones `draw_*` son cómodas para prototipos, pero lentas con muchos elementos. Para escenas grandes use sprites (`SpriteSolidColor`, `SpriteCircle`) o `arcade.shape_list.ShapeElementList`.

### 5.2 Colores

```python
arcade.color.AMBER                        # Más de 1000 colores con nombre
arcade.csscolor.CORNFLOWER_BLUE           # Colores CSS
Color(31, 78, 120)                         # RGB
Color(255, 0, 0, 128)                      # RGBA
Color.from_hex_string("#1F4E78")
```

### 5.3 Texto

```python
# Rápido de escribir, lento si se usa mucho (crea el texto en cada fotograma)
arcade.draw_text("Puntos: 10", 20, ALTO - 40, arcade.color.WHITE, 20)

# Recomendado: objeto Text creado una vez
self.marcador = arcade.Text("Puntos: 0", 20, ALTO - 40, arcade.color.WHITE, 20, bold=True)
self.marcador.text = f"Puntos: {self.puntos}"          # Actualizar
self.marcador.draw()                                    # En on_draw

arcade.Text("Centrado", ANCHO / 2, ALTO / 2, font_size=40, anchor_x="center", anchor_y="center")
arcade.load_font("recursos/fuente.ttf")                 # Fuente propia; luego font_name="Nombre de la fuente"
```

---

## 6. Sprites y listas de sprites

### 6.1 Crear sprites

```python
jugador = arcade.Sprite(":resources:images/animated_characters/female_person/femalePerson_idle.png",
                        scale=0.5, center_x=200, center_y=150)
caja = arcade.SpriteSolidColor(64, 64, 400, 100, arcade.color.BROWN)        # Rectángulo sólido
bola = arcade.SpriteCircle(16, arcade.color.YELLOW)                          # Círculo
suave = arcade.SpriteCircle(32, arcade.color.WHITE, soft=True)               # Círculo difuminado (brillos)
```

### 6.2 Propiedades

```python
jugador.center_x, jugador.center_y = 300, 200
jugador.position = (300, 200)
jugador.left, jugador.right, jugador.top, jugador.bottom         # Bordes (se pueden asignar)
jugador.width, jugador.height
jugador.scale = 0.75
jugador.angle = 45                       # Grados, sentido horario
jugador.alpha = 128                      # Transparencia (0–255)
jugador.color = arcade.color.RED         # Tinte
jugador.visible = False
jugador.change_x, jugador.change_y = 3, 0     # Velocidad (la usan update() y los motores de físicas)
jugador.change_angle = 2
jugador.texture = otra_textura           # Cambiar la imagen
jugador.properties["vida"] = 3           # Datos propios (también se rellenan desde Tiled)
```

### 6.3 Listas de sprites

```python
self.monedas = arcade.SpriteList(use_spatial_hash=True)     # Hash espacial: colisiones rápidas con objetos estáticos
for x in range(100, 1200, 120):
    self.monedas.append(arcade.Sprite(":resources:images/items/coinGold.png", 0.5, x, 300))

self.monedas.draw()                       # En on_draw: una sola llamada a la GPU
self.monedas.update(delta_time)           # Llama a update() de cada sprite
moneda.remove_from_sprite_lists()         # Eliminar un sprite de todas sus listas
len(self.monedas)
```

| Parámetro de `SpriteList` | Uso |
|---|---|
| `use_spatial_hash=True` | Objetos que **no se mueven** (paredes, monedas): acelera las colisiones. |
| `use_spatial_hash=False` (por defecto) | Objetos que se mueven mucho (balas, enemigos). |

### 6.4 Sprites propios

```python
class Bala(arcade.Sprite):
    def __init__(self, x, y, angulo):
        super().__init__(":resources:images/space_shooter/laserBlue01.png", scale=0.8,
                         center_x=x, center_y=y, angle=angulo)
        self.velocidad = 800
        self.tiempo_vida = 1.5

    def update(self, delta_time: float = 1 / 60):
        self.center_x += math.sin(math.radians(self.angle)) * self.velocidad * delta_time
        self.center_y += math.cos(math.radians(self.angle)) * self.velocidad * delta_time
        self.tiempo_vida -= delta_time
        if self.tiempo_vida <= 0:
            self.remove_from_sprite_lists()
```

### 6.5 Texturas

```python
textura = arcade.load_texture(":resources:images/enemies/slimeBlue.png")
izquierda = textura.flip_left_right()             # Versión espejo
hoja = arcade.load_spritesheet("recursos/personaje.png")
fotogramas = hoja.get_texture_grid(size=(32, 32), columns=8, count=8)
```

---

## 7. Entrada: teclado, ratón y mando

### 7.1 Teclado

```python
def on_key_press(self, tecla, modificadores):
    if tecla in (arcade.key.LEFT, arcade.key.A):
        self.izquierda = True
    elif tecla == arcade.key.SPACE:
        self.saltar()
    elif tecla == arcade.key.F11:
        self.window.set_fullscreen(not self.window.fullscreen)
    elif tecla == arcade.key.S and modificadores & arcade.key.MOD_CTRL:
        self.guardar()

def on_key_release(self, tecla, modificadores):
    if tecla in (arcade.key.LEFT, arcade.key.A):
        self.izquierda = False
```

Constantes: `arcade.key.A`…`Z`, `UP`, `DOWN`, `LEFT`, `RIGHT`, `SPACE`, `ENTER`/`RETURN`, `ESCAPE`, `TAB`, `LSHIFT`, `F1`… Modificadores: `MOD_SHIFT`, `MOD_CTRL`, `MOD_ALT`.

> Para movimiento continuo, guarde el estado de cada tecla (pulsada/soltada) y calcule la velocidad en `on_update`. Evita que el personaje se detenga al soltar una tecla mientras se mantiene la contraria.

### 7.2 Ratón

```python
def on_mouse_motion(self, x, y, dx, dy):
    self.mira.position = (x, y)

def on_mouse_press(self, x, y, boton, modificadores):
    if boton == arcade.MOUSE_BUTTON_LEFT:
        mundo = self.camara.unproject((x, y))          # Pantalla → coordenadas del mundo (con cámara)
        tocados = arcade.get_sprites_at_point((mundo.x, mundo.y), self.enemigos)

def on_mouse_scroll(self, x, y, scroll_x, scroll_y):
    self.camara.zoom = max(0.5, min(3, self.camara.zoom + scroll_y * 0.1))
```

### 7.3 Mandos

```python
import pyglet

mandos = pyglet.input.get_controllers()
if mandos:
    self.mando = mandos[0]
    self.mando.open()
    self.mando.push_handlers(self)                     # Llama a on_button_press(controller, button)…

def on_update(self, delta_time):
    if self.mando:
        eje_x = self.mando.leftx                       # -1 … 1
        if abs(eje_x) < 0.2:
            eje_x = 0

def on_button_press(self, mando, boton):
    if boton == "a":
        self.saltar()
```

---

## 8. Colisiones

```python
golpeados = arcade.check_for_collision_with_list(self.jugador, self.monedas)   # Lista de sprites tocados
for moneda in golpeados:
    moneda.remove_from_sprite_lists()
    self.puntos += 1

if arcade.check_for_collision(self.jugador, self.jefe): ...                   # Dos sprites
tocados = arcade.check_for_collision_with_lists(self.jugador, [self.enemigos, self.trampas])
arcade.get_sprites_at_point((x, y), self.botones)                              # Sprites bajo un punto
cercano, distancia = arcade.get_closest_sprite(self.jugador, self.enemigos)
```

### 8.1 Cajas de colisión (*hit boxes*)

```python
from arcade.hitbox import algo_detailed, algo_simple, algo_bounding_box

sprite = arcade.Sprite("enemigo.png", hit_box_algorithm=algo_detailed)    # Contorno preciso
sprite.hit_box.points                                                       # Puntos del polígono
self.enemigos.draw_hit_boxes(arcade.color.RED)                              # Depuración (en on_draw)
```

| Algoritmo | Precisión | Coste |
|---|---|---|
| `algo_bounding_box` | Rectángulo | Mínimo |
| `algo_simple` (por defecto) | Polígono sencillo que ignora zonas transparentes | Bajo |
| `algo_detailed` | Contorno detallado | Mayor (al cargar) |

---

## 9. Motores de físicas

### 9.1 `PhysicsEngineSimple`: vista cenital con paredes

```python
self.fisica = arcade.PhysicsEngineSimple(self.jugador, walls=self.paredes)

def on_update(self, delta_time):
    self.jugador.change_x = (self.derecha - self.izquierda) * VELOCIDAD
    self.jugador.change_y = (self.arriba - self.abajo) * VELOCIDAD
    self.fisica.update()                          # Mueve y evita atravesar paredes
```

> Estos motores usan `change_x`/`change_y` como **píxeles por actualización** (a 60 actualizaciones por segundo). Para velocidad independiente de los FPS, multiplique por `delta_time * 60` o use `on_fixed_update`.

### 9.2 `PhysicsEnginePlatformer`: plataformas

```python
self.fisica = arcade.PhysicsEnginePlatformer(
    self.jugador,
    walls=self.escena["Plataformas"],           # Suelos y paredes (estáticos)
    platforms=self.plataformas_moviles,         # Plataformas que se mueven
    ladders=self.escaleras,                     # Escaleras (sin gravedad mientras se toca)
    gravity_constant=1.0,
)

def on_key_press(self, tecla, mod):
    if tecla == arcade.key.SPACE and self.fisica.can_jump():
        self.jugador.change_y = FUERZA_SALTO
    # o bien: self.fisica.jump(FUERZA_SALTO)

def on_update(self, delta_time):
    self.fisica.update()
```

| Método / propiedad | Uso |
|---|---|
| `can_jump(y_distance=5)` | ¿Está apoyado en el suelo? |
| `enable_multi_jump(2)` + `increment_jump_counter()` | Doble salto. |
| `is_on_ladder()` | ¿Está en una escalera? |
| `gravity_constant` | Intensidad de la gravedad. |

Plataformas móviles (desde código o Tiled): asigne `change_x`/`change_y` y límites `boundary_left`, `boundary_right`, `boundary_top`, `boundary_bottom`; el motor las mueve e invierte al llegar al límite.

### 9.3 Pymunk: físicas realistas

Para rebotes, fricción, objetos que ruedan y uniones:

```python
# pip install pymunk
self.fisica = arcade.PymunkPhysicsEngine(gravity=(0, -1500), damping=0.9)
self.fisica.add_sprite(self.jugador, mass=2, friction=0.9, moment_of_inertia=arcade.PymunkPhysicsEngine.MOMENT_INF,
                       collision_type="jugador", max_horizontal_velocity=400)
self.fisica.add_sprite_list(self.paredes, body_type=arcade.PymunkPhysicsEngine.STATIC, friction=0.7)
self.fisica.add_sprite_list(self.cajas, mass=1, friction=0.6)
self.fisica.apply_force(self.jugador, (800, 0))
self.fisica.step()                                   # En on_update
```

---

## 10. Animación de sprites

### 10.1 Animación manual por estados

```python
RUTA = ":resources:images/animated_characters/female_adventurer/femaleAdventurer"

class Personaje(arcade.Sprite):
    def __init__(self):
        quieto = arcade.load_texture(f"{RUTA}_idle.png")
        super().__init__(quieto, scale=0.5)
        caminar = [arcade.load_texture(f"{RUTA}_walk{i}.png") for i in range(8)]
        self.texturas = {
            "quieto": [quieto, quieto.flip_left_right()],
            "saltar": [arcade.load_texture(f"{RUTA}_jump.png")] * 2,
            "caminar": [(t, t.flip_left_right()) for t in caminar],
        }
        self.texturas["saltar"][1] = self.texturas["saltar"][0].flip_left_right()
        self.mirando = 0                      # 0 derecha, 1 izquierda
        self.tiempo = 0.0

    def update_animation(self, delta_time: float = 1 / 60):
        if self.change_x < 0: self.mirando = 1
        elif self.change_x > 0: self.mirando = 0
        self.tiempo += delta_time
        if self.change_y != 0:
            self.texture = self.texturas["saltar"][self.mirando]
        elif self.change_x == 0:
            self.texture = self.texturas["quieto"][self.mirando]
        else:
            fotograma = int(self.tiempo * 12) % len(self.texturas["caminar"])
            self.texture = self.texturas["caminar"][fotograma][self.mirando]

# En on_update:
self.jugador.update_animation(delta_time)
```

### 10.2 `TextureAnimationSprite`

```python
from arcade import TextureAnimation, TextureKeyframe, TextureAnimationSprite

claves = [TextureKeyframe(t, duration=100) for t in fotogramas]          # Milisegundos
sprite = TextureAnimationSprite(animation=TextureAnimation(keyframes=claves), scale=0.5)
sprite.update_animation(delta_time)
```

### 10.3 Interpolación y efectos simples

```python
sprite.alpha = int(255 * (1 - progreso))                          # Desvanecer
sprite.scale = 1 + 0.1 * math.sin(tiempo * 6)                    # Latido
sprite.position = arcade.math.lerp_2d(origen, destino, progreso)  # Desplazamiento suave
```

---

## 11. Cámaras

Una `Camera2D` define qué parte del mundo se ve. Lo habitual es usar **dos**: una que sigue al jugador (mundo) y otra fija para la interfaz (marcador).

```python
self.camara = arcade.Camera2D()
self.camara_gui = arcade.Camera2D()

def on_draw(self):
    self.clear()
    with self.camara.activate():          # Todo lo dibujado aquí se ve a través de la cámara del mundo
        self.escena.draw()
    with self.camara_gui.activate():      # Interfaz fija en pantalla
        self.marcador.draw()

def on_update(self, delta_time):
    objetivo = self.jugador.position
    self.camara.position = arcade.math.lerp_2d(self.camara.position, objetivo, min(1, 6 * delta_time))  # Suavizado

def on_resize(self, ancho, alto):
    self.camara.match_window()
    self.camara_gui.match_window()
```

Limitar la cámara a los bordes del mapa:

```python
medio_ancho, medio_alto = self.window.width / 2, self.window.height / 2
x = max(medio_ancho, min(x, ancho_mapa - medio_ancho))
y = max(medio_alto, min(y, alto_mapa - medio_alto))
self.camara.position = (x, y)
```

| Propiedad / método | Uso |
|---|---|
| `position` | Punto del mundo en el **centro** de la pantalla. |
| `zoom` | Acercamiento (1 = normal, 2 = el doble). |
| `angle` | Rotación de la cámara. |
| `unproject((x, y))` | Coordenadas de pantalla → mundo (clic del ratón). |
| `match_window()` | Ajustar al nuevo tamaño de ventana. |

Sacudida de cámara:

```python
from arcade.camera.grips import ScreenShake2D

self.sacudida = ScreenShake2D(self.camara.view_data, max_amplitude=12, acceleration_duration=0.05,
                              falloff_time=0.4, shake_frequency=15)
self.sacudida.start()                 # Al recibir un golpe
self.sacudida.update(delta_time)      # En on_update
with self.camara.activate():
    self.sacudida.update_camera(); self.escena.draw(); self.sacudida.readjust_camera()
```

---

## 12. Mapas de casillas (Tiled) y escenas

[Tiled](https://www.mapeditor.org/) es un editor gratuito de mapas por casillas. Arcade carga sus mapas en formato JSON (`.json`/`.tmj`).

```python
opciones = {
    "Platforms": {"use_spatial_hash": True},                 # Capas estáticas: colisiones rápidas
    "Coins": {"use_spatial_hash": True},
}
self.mapa = arcade.load_tilemap(":resources:tiled_maps/map2_level_1.json", scaling=0.5, layer_options=opciones)
self.escena = arcade.Scene.from_tilemap(self.mapa)            # Una SpriteList por capa
self.escena.add_sprite("Jugador", self.jugador)               # Añadir capas propias
self.escena.add_sprite_list_after("Enemigos", "Jugador")

ancho_mapa = self.mapa.width * self.mapa.tile_width * 0.5     # En píxeles (con el escalado)
if self.mapa.background_color:
    self.window.background_color = self.mapa.background_color

self.escena.draw()                                             # Dibuja todas las capas en orden
self.escena["Coins"]                                           # Acceso a una capa (SpriteList)
```

Capas de objetos (puntos de aparición, zonas):

```python
for objeto in self.mapa.object_lists.get("Enemigos", []):
    x, y = objeto.shape                       # Para objetos punto
    tipo = objeto.properties.get("tipo")
```

Propiedades personalizadas de Tiled en las casillas: `sprite.properties["dano"]`.

| Recomendación de Tiled | Motivo |
|---|---|
| Una capa por función (plataformas, monedas, peligros, fondo) | Cada capa se convierte en una `SpriteList`. |
| Incrustar los *tilesets* en el mapa o usar rutas relativas | Evita errores de carga. |
| Formato JSON | Es el que lee Arcade. |

---

## 13. Vistas, interfaz gráfica (GUI) y sonido

### 13.1 Vistas

Cada pantalla es una `arcade.View` con sus propios `on_draw`, `on_update` y eventos:

```python
class Menu(arcade.View):
    def on_show_view(self):
        self.window.background_color = arcade.color.DARK_BLUE_GRAY

    def on_draw(self):
        self.clear()
        arcade.draw_text("Pulse ENTER", self.window.width / 2, self.window.height / 2,
                         font_size=30, anchor_x="center")

    def on_key_press(self, tecla, mod):
        if tecla == arcade.key.ENTER:
            self.window.show_view(Partida())

ventana = arcade.Window(1280, 720, "Juego")
ventana.show_view(Menu())
arcade.run()
```

Pausa conservando la partida: la vista de pausa guarda una referencia a la partida, la dibuja de fondo y vuelve a ella con `show_view(self.partida)`.

### 13.2 Interfaz gráfica (`arcade.gui`)

```python
import arcade.gui as gui

class Menu(arcade.View):
    def __init__(self):
        super().__init__()
        self.ui = gui.UIManager()
        ancla = self.ui.add(gui.UIAnchorLayout())
        caja = gui.UIBoxLayout(space_between=16)
        caja.add(gui.UILabel(text="Mi juego", font_size=40))
        boton = gui.UIFlatButton(text="Jugar", width=240)
        caja.add(boton)
        ancla.add(caja, anchor_x="center", anchor_y="center")

        @boton.event("on_click")
        def al_pulsar(evento):
            self.window.show_view(Partida())

    def on_show_view(self):
        self.ui.enable()                  # Imprescindible para recibir clics

    def on_hide_view(self):
        self.ui.disable()

    def on_draw(self):
        self.clear()
        self.ui.draw()
```

Elementos: `UIFlatButton`, `UITextureButton`, `UILabel`, `UIInputText`, `UISlider`, `UIDropdown`, `UIMessageBox`; diseños `UIAnchorLayout`, `UIBoxLayout(vertical=True)`, `UIGridLayout`.

### 13.3 Sonido

```python
self.sonido_moneda = arcade.load_sound(":resources:sounds/coin1.wav")
arcade.play_sound(self.sonido_moneda, volume=0.6)

self.musica = arcade.load_sound("recursos/musica.ogg", streaming=True)
self.reproductor = arcade.play_sound(self.musica, volume=0.4, loop=True)
arcade.stop_sound(self.reproductor)
```

> Cargue los sonidos una vez (al iniciar la vista o el juego), no en cada reproducción.

---

## 14. Partículas y efectos

```python
import arcade.particles as part
from arcade.math import rand_in_circle

textura = arcade.make_soft_circle_texture(12, arcade.color.GOLD)

def explosion(x, y):
    return part.Emitter(
        center_xy=(x, y),
        emit_controller=part.EmitBurst(30),                      # 30 partículas de golpe
        particle_factory=lambda emisor: part.FadeParticle(
            filename_or_texture=textura,
            change_xy=rand_in_circle((0, 0), 4.0),
            lifetime=random.uniform(0.3, 0.8),
            scale=random.uniform(0.5, 1.2),
        ),
    )

self.emisores.append(explosion(x, y))
# En on_update:
for emisor in self.emisores[:]:
    emisor.update(delta_time)
    if emisor.can_reap():
        self.emisores.remove(emisor)
# En on_draw:
for emisor in self.emisores:
    emisor.draw()
```

| Controlador | Efecto |
|---|---|
| `EmitBurst(n)` | `n` partículas de una vez (explosiones). |
| `EmitMaintainCount(n)` | Mantiene `n` partículas vivas (fuego, humo). |
| `EmitInterval(s)` | Una partícula cada `s` segundos. |

*Shaders* y posprocesado: Arcade permite *shaders* GLSL (`self.window.ctx.program(...)`) y efectos como iluminación 2D (`arcade.experimental.lights`) para quien necesite gráficos avanzados.

---

## 15. Rendimiento y distribución

### 15.1 Rendimiento

| Técnica | Cómo |
|---|---|
| Dibujar en bloque | Agrupe los sprites en `SpriteList` y llame a `draw()` una vez por lista. |
| Objetos estáticos | `use_spatial_hash=True` en paredes, suelos y objetos fijos. |
| Texto | Use `arcade.Text` (creado una vez) en lugar de `draw_text`. |
| Formas | Use sprites o `ShapeElementList` en lugar de muchas llamadas `draw_*`. |
| Medir | `arcade.enable_timings()` y `arcade.get_fps()`; `arcade.print_timings()`. |
| Colisiones | Compruebe solo contra las listas necesarias; filtre por distancia. |

### 15.2 Distribución

```bash
pip install pyinstaller
pyinstaller --onefile --windowed --name "MiJuego" --collect-data arcade main.py
```

> `--collect-data arcade` incluye los recursos internos de Arcade (también los `:resources:`). Para recursos propios añada `--add-data "recursos;recursos"` (Windows) o `"recursos:recursos"` (Linux/macOS), y localícelos con `sys._MEIPASS` como en otros proyectos con PyInstaller.

También es posible empaquetar con **Nuitka** (`python -m nuitka --standalone --include-package-data=arcade main.py`), que genera ejecutables más rápidos.

---

## 16. Ejemplo práctico completo

**«Aventura en la pradera»**: juego de plataformas completo con los recursos incluidos en Arcade. Dos niveles cargados de mapas Tiled, personaje animado (quieto, caminar, saltar), físicas de plataformas con doble salto, monedas con sonido y partículas, enemigos que patrullan y se pueden aplastar saltando sobre ellos, zonas peligrosas, vidas, cámara que sigue al jugador limitada al mapa, marcador con cámara propia, menú y pausa con `arcade.gui`, pantallas de victoria y fin de partida, y récord guardado en disco.

```python
"""
Aventura en la pradera: plataformas 2D con Arcade 3 (solo recursos incluidos en la librería).
Requiere: pip install arcade
Controles: ←/→ o A/D mover · Espacio/↑/W saltar (doble salto) · Esc pausa · F11 pantalla completa
"""
import json
import random
from pathlib import Path

import arcade
import arcade.gui as gui
import arcade.particles as particulas
from arcade.math import lerp_2d, rand_in_circle

ANCHO, ALTO, TITULO = 1280, 720, "Aventura en la pradera"
ESCALA = 0.5
TAM_CASILLA = 128 * ESCALA
NIVELES = [":resources:tiled_maps/map2_level_1.json", ":resources:tiled_maps/map2_level_2.json"]
VELOCIDAD = 7
FUERZA_SALTO = 19
GRAVEDAD = 1.1
RUTA_RECORD = Path.home() / ".aventura_pradera.json"
RUTA_PERSONAJE = ":resources:images/animated_characters/female_adventurer/femaleAdventurer"


def cargar_record() -> int:
    try:
        return json.loads(RUTA_RECORD.read_text(encoding="utf-8")).get("record", 0)
    except (OSError, ValueError):
        return 0


def guardar_record(valor: int) -> None:
    try:
        RUTA_RECORD.write_text(json.dumps({"record": valor}), encoding="utf-8")
    except OSError:
        pass


class Recursos:
    """Texturas y sonidos cargados una sola vez."""

    def __init__(self):
        self.moneda = arcade.load_sound(":resources:sounds/coin1.wav")
        self.salto = arcade.load_sound(":resources:sounds/jump1.wav")
        self.golpe = arcade.load_sound(":resources:sounds/hurt2.wav")
        self.aplastar = arcade.load_sound(":resources:sounds/hit3.wav")
        self.fin = arcade.load_sound(":resources:sounds/gameover1.wav")
        self.nivel = arcade.load_sound(":resources:sounds/upgrade1.wav")
        self.chispa = arcade.make_soft_circle_texture(14, arcade.color.GOLD)
        self.polvo = arcade.make_soft_circle_texture(14, arcade.color.LIGHT_GRAY)


# ------------------------------------------------------------------ Entidades
class Jugador(arcade.Sprite):
    def __init__(self):
        quieto = arcade.load_texture(f"{RUTA_PERSONAJE}_idle.png")
        super().__init__(quieto, scale=ESCALA)
        caminar = [arcade.load_texture(f"{RUTA_PERSONAJE}_walk{i}.png") for i in range(8)]
        saltar = arcade.load_texture(f"{RUTA_PERSONAJE}_jump.png")
        caer = arcade.load_texture(f"{RUTA_PERSONAJE}_fall.png")
        self.texturas = {
            "quieto": (quieto, quieto.flip_left_right()),
            "saltar": (saltar, saltar.flip_left_right()),
            "caer": (caer, caer.flip_left_right()),
            "caminar": [(t, t.flip_left_right()) for t in caminar],
        }
        self.mirando = 0
        self.tiempo = 0.0
        self.invulnerable = 0.0

    def update_animation(self, delta_time: float = 1 / 60):
        self.tiempo += delta_time
        if self.change_x < 0:
            self.mirando = 1
        elif self.change_x > 0:
            self.mirando = 0
        if self.change_y > 0.5:
            self.texture = self.texturas["saltar"][self.mirando]
        elif self.change_y < -0.5:
            self.texture = self.texturas["caer"][self.mirando]
        elif self.change_x == 0:
            self.texture = self.texturas["quieto"][self.mirando]
        else:
            fotograma = int(self.tiempo * 14) % len(self.texturas["caminar"])
            self.texture = self.texturas["caminar"][fotograma][self.mirando]
        # Parpadeo tras recibir un golpe
        self.invulnerable = max(0.0, self.invulnerable - delta_time)
        self.alpha = 120 if self.invulnerable and int(self.invulnerable * 10) % 2 else 255


class Enemigo(arcade.Sprite):
    """Slime que patrulla y da la vuelta ante una pared o un precipicio."""

    def __init__(self, x: float, y: float, paredes: arcade.SpriteList):
        self.texturas = [arcade.load_texture(":resources:images/enemies/slimeBlue.png"),
                         arcade.load_texture(":resources:images/enemies/slimeBlue_move.png")]
        super().__init__(self.texturas[0], scale=ESCALA, center_x=x, center_y=y)
        self.paredes = paredes
        self.change_x = random.choice((-1.6, 1.6))
        self.fisica = arcade.PhysicsEnginePlatformer(self, walls=paredes, gravity_constant=GRAVEDAD)
        self.tiempo = 0.0

    def update(self, delta_time: float = 1 / 60):
        x_anterior = self.center_x
        self.fisica.update()
        delante = self.right + 4 if self.change_x > 0 else self.left - 4
        sin_suelo = not arcade.get_sprites_at_point((delante, self.bottom - 8), self.paredes)
        bloqueado = abs(self.center_x - x_anterior) < 0.1
        if self.fisica.can_jump() and (sin_suelo or bloqueado):
            self.change_x *= -1
        self.tiempo += delta_time
        textura = self.texturas[int(self.tiempo * 4) % 2]
        self.texture = textura if self.change_x < 0 else textura.flip_left_right()


# ------------------------------------------------------------------ Vistas
class VistaMenu(arcade.View):
    def __init__(self, recursos: Recursos):
        super().__init__()
        self.recursos = recursos
        self.ui = gui.UIManager()
        ancla = self.ui.add(gui.UIAnchorLayout())
        caja = gui.UIBoxLayout(space_between=18)
        caja.add(gui.UILabel(text=TITULO, font_size=44, bold=True, text_color=arcade.color.WHITE))
        caja.add(gui.UILabel(text=f"Récord: {cargar_record()}", font_size=20, text_color=arcade.color.GOLD))
        jugar = caja.add(gui.UIFlatButton(text="Jugar", width=260, height=56))
        salir = caja.add(gui.UIFlatButton(text="Salir", width=260, height=56))
        caja.add(gui.UILabel(text="←/→ mover · Espacio saltar (doble salto) · Esc pausa",
                             font_size=14, text_color=arcade.color.LIGHT_GRAY))
        ancla.add(caja, anchor_x="center", anchor_y="center")
        jugar.on_click = lambda evento: self.window.show_view(VistaPartida(self.recursos))
        salir.on_click = lambda evento: arcade.exit()
        self.fondo = arcade.SpriteList()
        for i in range(25):
            self.fondo.append(arcade.Sprite(":resources:images/items/coinGold.png", scale=0.3,
                                            center_x=random.uniform(0, ANCHO), center_y=random.uniform(0, ALTO)))

    def on_show_view(self):
        self.window.background_color = arcade.color.DARK_MIDNIGHT_BLUE
        self.ui.enable()

    def on_hide_view(self):
        self.ui.disable()

    def on_update(self, delta_time):
        for moneda in self.fondo:
            moneda.center_y -= 40 * delta_time
            if moneda.top < 0:
                moneda.center_y = ALTO + 20

    def on_draw(self):
        self.clear()
        self.fondo.draw()
        self.ui.draw()

    def on_key_press(self, tecla, modificadores):
        if tecla in (arcade.key.ENTER, arcade.key.RETURN):
            self.window.show_view(VistaPartida(self.recursos))


class VistaPartida(arcade.View):
    def __init__(self, recursos: Recursos, nivel: int = 0, puntos: int = 0, vidas: int = 3):
        super().__init__()
        self.recursos = recursos
        self.nivel, self.puntos, self.vidas = nivel, puntos, vidas
        self.teclas = {"izquierda": False, "derecha": False}
        self.emisores: list[particulas.Emitter] = []
        self.camara = arcade.Camera2D()
        self.camara_gui = arcade.Camera2D()
        self.marcador = arcade.Text("", 24, ALTO - 44, arcade.color.WHITE, 22, bold=True)
        self.info = arcade.Text("", 24, ALTO - 74, arcade.color.WHITE, 14)
        self.cargar_nivel()

    # ---------------------------------------------------------------- Nivel
    def cargar_nivel(self):
        opciones = {"Platforms": {"use_spatial_hash": True}, "Coins": {"use_spatial_hash": True},
                    "Don't Touch": {"use_spatial_hash": True}}
        self.mapa = arcade.load_tilemap(NIVELES[self.nivel], scaling=ESCALA, layer_options=opciones)
        self.escena = arcade.Scene.from_tilemap(self.mapa)
        self.ancho_mapa = self.mapa.width * TAM_CASILLA
        self.alto_mapa = self.mapa.height * TAM_CASILLA

        self.jugador = Jugador()
        self.inicio = (TAM_CASILLA * 2, TAM_CASILLA * 3)
        self.jugador.position = self.inicio
        self.escena.add_sprite("Jugador", self.jugador)

        self.fisica = arcade.PhysicsEnginePlatformer(self.jugador, walls=self.escena["Platforms"],
                                                     gravity_constant=GRAVEDAD)
        self.fisica.enable_multi_jump(2)

        # Enemigos sobre casillas de plataforma con espacio libre encima, lejos del inicio
        self.enemigos = arcade.SpriteList()
        candidatos = [p for p in self.escena["Platforms"]
                      if p.center_x > TAM_CASILLA * 8
                      and not arcade.get_sprites_at_point((p.center_x, p.top + TAM_CASILLA / 2), self.escena["Platforms"])]
        random.seed(self.nivel)
        for plataforma in random.sample(candidatos, min(4 + 2 * self.nivel, len(candidatos))):
            self.enemigos.append(Enemigo(plataforma.center_x, plataforma.top + 30, self.escena["Platforms"]))
        self.escena.add_sprite_list_before("Enemigos", "Jugador", sprite_list=self.enemigos)
        self.total_monedas = len(self.escena["Coins"])
        self.camara.position = self.limitar_camara(self.jugador.position)

    def on_show_view(self):
        self.window.background_color = arcade.color.CORNFLOWER_BLUE

    # ---------------------------------------------------------------- Entrada
    def on_key_press(self, tecla, modificadores):
        if tecla in (arcade.key.LEFT, arcade.key.A):
            self.teclas["izquierda"] = True
        elif tecla in (arcade.key.RIGHT, arcade.key.D):
            self.teclas["derecha"] = True
        elif tecla in (arcade.key.SPACE, arcade.key.UP, arcade.key.W):
            self.saltar()
        elif tecla == arcade.key.ESCAPE:
            self.window.show_view(VistaPausa(self))
        elif tecla == arcade.key.F11:
            self.window.set_fullscreen(not self.window.fullscreen)

    def on_key_release(self, tecla, modificadores):
        if tecla in (arcade.key.LEFT, arcade.key.A):
            self.teclas["izquierda"] = False
        elif tecla in (arcade.key.RIGHT, arcade.key.D):
            self.teclas["derecha"] = False

    def saltar(self):
        if self.fisica.can_jump():
            self.fisica.jump(FUERZA_SALTO)
            arcade.play_sound(self.recursos.salto, volume=0.4)
            self.emisores.append(self.rafaga(self.jugador.center_x, self.jugador.bottom, self.recursos.polvo, 10))

    # ---------------------------------------------------------------- Lógica
    def on_update(self, delta_time):
        self.jugador.change_x = (self.teclas["derecha"] - self.teclas["izquierda"]) * VELOCIDAD
        self.fisica.update()                            # can_jump() repone el doble salto al tocar el suelo
        self.jugador.update_animation(delta_time)
        self.enemigos.update(delta_time)
        self.jugador.left = max(0, self.jugador.left)

        for moneda in arcade.check_for_collision_with_list(self.jugador, self.escena["Coins"]):
            self.emisores.append(self.rafaga(moneda.center_x, moneda.center_y, self.recursos.chispa, 24))
            moneda.remove_from_sprite_lists()
            self.puntos += 10
            arcade.play_sound(self.recursos.moneda, volume=0.5)

        for enemigo in arcade.check_for_collision_with_list(self.jugador, self.enemigos):
            if self.jugador.change_y < 0 and self.jugador.bottom > enemigo.center_y:   # Cae sobre él
                self.emisores.append(self.rafaga(enemigo.center_x, enemigo.center_y, self.recursos.polvo, 20))
                enemigo.remove_from_sprite_lists()
                self.jugador.change_y = FUERZA_SALTO * 0.6
                self.puntos += 50
                arcade.play_sound(self.recursos.aplastar, volume=0.5)
            else:
                self.perder_vida()
                break

        if arcade.check_for_collision_with_list(self.jugador, self.escena["Don't Touch"]) or self.jugador.top < -200:
            self.perder_vida(volver_al_inicio=True)

        if self.jugador.center_x >= self.ancho_mapa - TAM_CASILLA:
            self.completar_nivel()
            return

        for emisor in self.emisores[:]:
            emisor.update(delta_time)
            if emisor.can_reap():
                self.emisores.remove(emisor)

        destino = self.limitar_camara(self.jugador.position)
        self.camara.position = lerp_2d(self.camara.position, destino, min(1.0, 8 * delta_time))
        self.marcador.text = f"Puntos: {self.puntos}"
        quedan = len(self.escena["Coins"])
        self.info.text = f"Nivel {self.nivel + 1}/{len(NIVELES)} · Vidas: {self.vidas} · " \
                         f"Monedas: {self.total_monedas - quedan}/{self.total_monedas}"

    def limitar_camara(self, posicion):
        medio_ancho, medio_alto = self.window.width / 2, self.window.height / 2
        x = max(medio_ancho, min(posicion[0], self.ancho_mapa - medio_ancho))
        y = max(medio_alto, min(posicion[1], max(medio_alto, self.alto_mapa - medio_alto)))
        return x, y

    def rafaga(self, x, y, textura, cantidad) -> particulas.Emitter:
        return particulas.Emitter(
            center_xy=(x, y),
            emit_controller=particulas.EmitBurst(cantidad),
            particle_factory=lambda emisor: particulas.FadeParticle(
                filename_or_texture=textura, change_xy=rand_in_circle((0.0, 0.0), 4.0),
                lifetime=random.uniform(0.3, 0.7), scale=random.uniform(0.4, 1.0)),
        )

    def perder_vida(self, volver_al_inicio: bool = False):
        if self.jugador.invulnerable and not volver_al_inicio:
            return
        self.vidas -= 1
        arcade.play_sound(self.recursos.golpe, volume=0.6)
        if self.vidas <= 0:
            arcade.play_sound(self.recursos.fin)
            self.window.show_view(VistaFinal(self.recursos, self.puntos, victoria=False))
            return
        self.jugador.invulnerable = 2.0
        if volver_al_inicio:
            self.jugador.position = self.inicio
            self.jugador.change_x = self.jugador.change_y = 0
        else:
            self.jugador.change_y = FUERZA_SALTO * 0.5          # Pequeño rebote

    def completar_nivel(self):
        arcade.play_sound(self.recursos.nivel)
        bonificacion = 100 * self.vidas
        if self.nivel + 1 < len(NIVELES):
            self.window.show_view(VistaPartida(self.recursos, self.nivel + 1, self.puntos + bonificacion, self.vidas))
        else:
            self.window.show_view(VistaFinal(self.recursos, self.puntos + bonificacion, victoria=True))

    # ---------------------------------------------------------------- Dibujo
    def on_draw(self):
        self.clear()
        with self.camara.activate():
            self.escena.draw()
            for emisor in self.emisores:
                emisor.draw()
        with self.camara_gui.activate():
            arcade.draw_rect_filled(arcade.LBWH(12, ALTO - 88, 430, 76), (0, 0, 0, 110))
            self.marcador.draw()
            self.info.draw()

    def on_resize(self, ancho, alto):
        self.camara.match_window()
        self.camara_gui.match_window()


class VistaPausa(arcade.View):
    def __init__(self, partida: VistaPartida):
        super().__init__()
        self.partida = partida
        self.ui = gui.UIManager()
        ancla = self.ui.add(gui.UIAnchorLayout())
        caja = gui.UIBoxLayout(space_between=14)
        caja.add(gui.UILabel(text="PAUSA", font_size=40, bold=True))
        continuar = caja.add(gui.UIFlatButton(text="Continuar", width=240))
        menu = caja.add(gui.UIFlatButton(text="Menú principal", width=240))
        ancla.add(caja, anchor_x="center", anchor_y="center")
        continuar.on_click = lambda evento: self.window.show_view(self.partida)
        menu.on_click = lambda evento: self.window.show_view(VistaMenu(self.partida.recursos))

    def on_show_view(self):
        self.ui.enable()
        self.partida.teclas = {"izquierda": False, "derecha": False}

    def on_hide_view(self):
        self.ui.disable()

    def on_draw(self):
        self.partida.on_draw()                                  # La partida congelada de fondo
        arcade.draw_rect_filled(arcade.LBWH(0, 0, self.window.width, self.window.height), (0, 0, 0, 150))
        self.ui.draw()

    def on_key_press(self, tecla, modificadores):
        if tecla == arcade.key.ESCAPE:
            self.window.show_view(self.partida)


class VistaFinal(arcade.View):
    def __init__(self, recursos: Recursos, puntos: int, victoria: bool):
        super().__init__()
        self.recursos, self.puntos, self.victoria = recursos, puntos, victoria
        self.record = cargar_record()
        self.nuevo_record = puntos > self.record
        if self.nuevo_record:
            guardar_record(puntos)
        titulo = "¡VICTORIA!" if victoria else "FIN DE LA PARTIDA"
        color = arcade.color.GOLD if victoria else arcade.color.SALMON
        self.textos = [
            arcade.Text(titulo, ANCHO / 2, ALTO * 0.62, color, 52, bold=True, anchor_x="center"),
            arcade.Text(f"Puntuación: {puntos}", ANCHO / 2, ALTO * 0.5, arcade.color.WHITE, 28, anchor_x="center"),
            arcade.Text("¡Nuevo récord!" if self.nuevo_record else f"Récord: {self.record}",
                        ANCHO / 2, ALTO * 0.43, arcade.color.GOLD, 20, anchor_x="center"),
            arcade.Text("ENTER: jugar de nuevo · Esc: menú", ANCHO / 2, ALTO * 0.3,
                        arcade.color.LIGHT_GRAY, 16, anchor_x="center"),
        ]

    def on_show_view(self):
        self.window.background_color = arcade.color.DARK_MIDNIGHT_BLUE

    def on_draw(self):
        self.clear()
        for texto in self.textos:
            texto.draw()

    def on_key_press(self, tecla, modificadores):
        if tecla in (arcade.key.ENTER, arcade.key.RETURN):
            self.window.show_view(VistaPartida(self.recursos))
        elif tecla == arcade.key.ESCAPE:
            self.window.show_view(VistaMenu(self.recursos))


def main():
    ventana = arcade.Window(ANCHO, ALTO, TITULO, vsync=True)
    ventana.show_view(VistaMenu(Recursos()))
    arcade.run()


if __name__ == "__main__":
    main()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| El sprite no se dibuja | No está en ninguna `SpriteList` o no se llama a `draw()`. | Añádalo a una lista y dibújela en `on_draw`. |
| Rastros o pantalla sin borrar | Falta `self.clear()`. | Llámelo al principio de `on_draw`. |
| `AttributeError: module 'arcade' has no attribute 'draw_xywh_rectangle_filled'` / `Camera` | Código de Arcade 2.x. | Use `draw_rect_filled(arcade.XYWH(...))` y `Camera2D`. |
| `NoOpenGLException` / no abre la ventana | Sin OpenGL 3.3 (máquina virtual, escritorio remoto, controlador antiguo). | Actualice los controladores gráficos o use un equipo con GPU. |
| `FileNotFoundError` con `:resources:` | Ruta mal escrita. | Revise mayúsculas y extensión; liste con `arcade.resources`. |
| Los botones de la GUI no responden | `UIManager` sin activar. | `self.ui.enable()` en `on_show_view` y `disable()` en `on_hide_view`. |
| El HUD se mueve con el mundo | Se dibuja con la cámara del mundo. | Use una segunda `Camera2D` para la interfaz. |
| El clic del ratón no coincide con los objetos | Coordenadas de pantalla con la cámara desplazada. | `camara.unproject((x, y))`. |
| El personaje atraviesa suelos a gran velocidad | Velocidades enormes por fotograma. | Limite la velocidad o use `on_fixed_update`. |
| Colisiones lentas con muchos objetos | Listas sin hash espacial. | `use_spatial_hash=True` en objetos estáticos. |
| Texto lento | Uso masivo de `draw_text`. | Cree objetos `arcade.Text` una vez. |
| El mapa Tiled no carga casillas | *Tileset* externo con ruta incorrecta. | Incruste los *tilesets* o use rutas relativas al mapa. |
| `KeyError` al acceder a `escena["Capa"]` | El nombre de la capa no coincide con Tiled. | Respete mayúsculas, espacios y apóstrofos. |

---

## 18. Buenas prácticas

- Use **vistas** (`View`) para separar menú, partida, pausa y fin.
- Cargue texturas y sonidos **una vez** y reutilícelos.
- Agrupe todos los sprites en `SpriteList` y active el hash espacial en los estáticos.
- Use dos cámaras: mundo y GUI; limite la cámara del mundo a los bordes del mapa.
- Diseñe los niveles en **Tiled** con capas por función (plataformas, monedas, peligros).
- Use objetos `arcade.Text` en lugar de `draw_text` en cada fotograma.
- Guarde el estado de las teclas y calcule el movimiento en `on_update`.
- Dé respuesta al jugador: sonido, partículas, parpadeo de invulnerabilidad.
- Mantenga las constantes (velocidades, gravedad, escalas) en un único lugar para ajustar la jugabilidad.
- Mida con `arcade.enable_timings()` antes de optimizar.

---

## 19. Referencia rápida

```python
import arcade
class Juego(arcade.View):                 # o arcade.Window
    def on_show_view(self): self.window.background_color = arcade.color.BLACK
    def on_update(self, dt): ...
    def on_draw(self): self.clear(); self.sprites.draw()
    def on_key_press(self, tecla, mod): ...   arcade.key.LEFT / SPACE / ESCAPE
ventana = arcade.Window(1280, 720, "T"); ventana.show_view(Juego()); arcade.run()

arcade.draw_circle_filled(x, y, r, c) / draw_rect_filled(arcade.XYWH(x, y, w, h), c) / draw_line / draw_polygon_filled
arcade.Text("t", x, y, color, tamaño, anchor_x="center").draw()
s = arcade.Sprite(":resources:images/...png", scale=0.5, center_x=, center_y=); s.change_x; s.angle; s.texture
lista = arcade.SpriteList(use_spatial_hash=True); lista.append(s); lista.draw(); lista.update(dt); s.remove_from_sprite_lists()
arcade.check_for_collision_with_list(s, lista) / check_for_collision(a, b) / get_sprites_at_point((x, y), lista)
arcade.PhysicsEngineSimple(s, walls=) / PhysicsEnginePlatformer(s, walls=, gravity_constant=); .update(); .can_jump(); .jump(v)
cam = arcade.Camera2D(); with cam.activate(): ...; cam.position = (x, y); cam.zoom; cam.unproject((x, y))
mapa = arcade.load_tilemap(":resources:tiled_maps/map.json", scaling=0.5); escena = arcade.Scene.from_tilemap(mapa)
ui = arcade.gui.UIManager(); UIAnchorLayout / UIBoxLayout / UIFlatButton(text=).on_click / UILabel; ui.enable(); ui.draw()
arcade.play_sound(arcade.load_sound(":resources:sounds/coin1.wav"), volume=0.5)
arcade.particles.Emitter(center_xy=, emit_controller=EmitBurst(n), particle_factory=lambda e: FadeParticle(...))
```

---

## 20. Ejercicios propuestos

1. Cree un juego de recoger monedas con vista cenital, `PhysicsEngineSimple` y paredes generadas por código.
2. Programe un *shoot 'em up* espacial con los recursos `:resources:images/space_shooter/`, oleadas de enemigos y partículas de explosión.
3. Diseñe en Tiled un nivel propio con escaleras y plataformas móviles y cárguelo en Arcade.
4. Añada al menú del ejemplo un control deslizante de volumen y una opción de pantalla completa con `arcade.gui`.
5. Amplíe el ejemplo completo con un tercer nivel, enemigos voladores que persiguen al jugador y una tabla de mejores puntuaciones con el nombre del jugador (`UIInputText`).

---

## 21. Referencias

- Documentación oficial: https://api.arcade.academy/
- Tutorial de plataformas paso a paso: https://api.arcade.academy/en/latest/tutorials/platform_tutorial/index.html
- Ejemplos: https://api.arcade.academy/en/latest/example_code/index.html
- Recursos incluidos (`:resources:`): https://api.arcade.academy/en/latest/api_docs/resources.html
- Editor de mapas Tiled: https://www.mapeditor.org/
- Recursos gráficos gratuitos (Kenney): https://kenney.nl/assets
