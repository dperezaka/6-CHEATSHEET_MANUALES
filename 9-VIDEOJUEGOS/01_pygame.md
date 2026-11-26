# Manual de uso: Pygame

> **Categoría:** Videojuegos y gráficos · **Utilidad:** Crear videojuegos 2D
> **Tipo:** Librería externa (requiere instalación con `pip`). Basada en SDL2.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [La ventana y el bucle principal](#4-la-ventana-y-el-bucle-principal)
5. [Dibujo: superficies, colores y formas](#5-dibujo-superficies-colores-y-formas)
6. [Imágenes y texto](#6-imágenes-y-texto)
7. [Eventos: teclado, ratón y mando](#7-eventos-teclado-ratón-y-mando)
8. [Movimiento y tiempo (*delta time*)](#8-movimiento-y-tiempo-delta-time)
9. [Sprites y grupos](#9-sprites-y-grupos)
10. [Colisiones](#10-colisiones)
11. [Animación](#11-animación)
12. [Sonido y música](#12-sonido-y-música)
13. [Cámara, escenarios y mapas de casillas](#13-cámara-escenarios-y-mapas-de-casillas)
14. [Estados del juego y estructura del proyecto](#14-estados-del-juego-y-estructura-del-proyecto)
15. [Rendimiento y distribución](#15-rendimiento-y-distribución)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Pygame** es la librería más conocida para crear **videojuegos 2D** en Python. Es una capa sobre **SDL2** (Simple DirectMedia Layer) que proporciona acceso a la ventana, el dibujo de gráficos, el teclado, el ratón, los mandos, el sonido y el temporizador. No impone una estructura: el programador escribe su propio bucle de juego, lo que la convierte en una herramienta excelente para aprender cómo funciona un videojuego por dentro.

**Ventajas:**
- Sencilla de instalar y de aprender; enorme cantidad de tutoriales y ejemplos.
- Control total del bucle del juego, el dibujo y la lógica.
- Funciona en Windows, macOS y Linux (y en navegador mediante *pygbag*).
- Incluye sprites, grupos, detección de colisiones, sonido, música, fuentes y transformaciones de imagen.
- Ideal para prototipos, juegos educativos, *game jams* y herramientas de visualización interactiva.

**Limitaciones:**
- El dibujo se realiza por software (CPU): con miles de objetos en pantalla o resoluciones muy altas el rendimiento cae.
- Sin motor de físicas, editor de escenas ni sistema de entidades: todo se programa a mano.
- Sin soporte 3D.

| Si necesita… | Opción |
|---|---|
| Aprender programación de juegos, prototipos 2D, control total | **Pygame** |
| Juegos 2D con aceleración GPU, cámaras y físicas integradas | Arcade |
| Juegos 3D sencillos con muy poco código | Ursina |
| Motor 3D completo | Panda3D |
| Gráficos 3D a bajo nivel | PyOpenGL |

### Pygame y Pygame CE

Existen dos distribuciones que se importan igual (`import pygame`):

| Paquete | Descripción |
|---|---|
| `pygame` | Distribución original. |
| `pygame-ce` | *Community Edition*: bifurcación mantenida por gran parte de los antiguos desarrolladores, con versiones más frecuentes y funciones adicionales. |

> Instale **solo una** de las dos en el mismo entorno. Todo el código de este manual funciona con ambas.

---

## 2. Instalación e importación

```bash
pip install pygame            # o bien: pip install pygame-ce
```

Comprobar la instalación (abre un juego de demostración):

```bash
python -m pygame.examples.aliens
```

Importación habitual:

```python
import pygame
from pygame.math import Vector2
```

Inicialización y cierre:

```python
pygame.init()          # Inicializa todos los módulos (pantalla, fuentes, sonido…)
...
pygame.quit()          # Libera los recursos al terminar
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Superficie (`Surface`) | Imagen en memoria sobre la que se dibuja. La ventana también es una superficie. |
| Pantalla (*display*) | Superficie principal, la que se ve en la ventana. |
| Rectángulo (`Rect`) | Posición y tamaño de un elemento; se usa para colocar, mover y detectar colisiones. |
| Bucle del juego (*game loop*) | Bucle infinito que repite: **leer eventos → actualizar → dibujar**. |
| Fotograma (*frame*) | Cada repetición del bucle. Se suelen buscar 60 fotogramas por segundo (FPS). |
| Reloj (`Clock`) | Limita los FPS y mide el tiempo transcurrido entre fotogramas (*delta time*). |
| Evento | Acción del usuario o del sistema: tecla pulsada, clic, cierre de ventana… |
| Sprite | Objeto del juego con imagen (`image`) y rectángulo (`rect`). |
| Grupo (`Group`) | Colección de sprites que se actualizan, dibujan y comprueban colisiones juntos. |
| *Blit* | Copiar (dibujar) una superficie sobre otra. |

**Sistema de coordenadas:** el origen `(0, 0)` está en la **esquina superior izquierda**; X crece hacia la derecha e **Y hacia abajo**.

```
(0,0) ──────────► x
  │
  │      (200, 150)
  │          ●
  ▼
  y
```

**El bucle del juego:**

```
┌──────────────────────────────────────────────┐
│ 1. Eventos     → ¿qué ha hecho el jugador?    │
│ 2. Actualizar  → mover, colisiones, reglas    │
│ 3. Dibujar     → borrar, pintar, mostrar      │
│ 4. Esperar     → clock.tick(60)               │
└───────────────────────── repetir ────────────┘
```

---

## 4. La ventana y el bucle principal

### 4.1 Programa mínimo

```python
import pygame

pygame.init()
pantalla = pygame.display.set_mode((800, 600))
pygame.display.set_caption("Mi juego")
reloj = pygame.time.Clock()

ejecutando = True
while ejecutando:
    # 1. Eventos
    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            ejecutando = False

    # 2. Actualizar (lógica del juego)

    # 3. Dibujar
    pantalla.fill((30, 30, 40))
    pygame.draw.circle(pantalla, (255, 200, 0), (400, 300), 40)
    pygame.display.flip()           # Muestra lo dibujado

    # 4. Limitar a 60 FPS
    reloj.tick(60)

pygame.quit()
```

> Sin procesar `pygame.event.get()` en cada fotograma, el sistema operativo considera que la ventana «no responde».

### 4.2 Opciones de la ventana

```python
pantalla = pygame.display.set_mode((1280, 720), pygame.RESIZABLE)     # Redimensionable
pantalla = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)          # Pantalla completa (resolución nativa)
pantalla = pygame.display.set_mode((640, 360), pygame.SCALED)          # Escala automática (estilo pixel art)
pantalla = pygame.display.set_mode((800, 600), vsync=1)                # Sincronización vertical (con SCALED u OPENGL)
pygame.display.set_icon(pygame.image.load("icono.png"))
pygame.display.toggle_fullscreen()                                      # Alternar pantalla completa
ancho, alto = pantalla.get_size()
```

| Opción | Efecto |
|---|---|
| `pygame.RESIZABLE` | El usuario puede cambiar el tamaño (evento `VIDEORESIZE`). |
| `pygame.FULLSCREEN` | Pantalla completa. |
| `pygame.SCALED` | La resolución lógica se escala a la ventana: ideal para *pixel art*. |
| `pygame.NOFRAME` | Ventana sin bordes. |

### 4.3 `flip()` frente a `update()`

```python
pygame.display.flip()                       # Actualiza toda la pantalla
pygame.display.update(lista_de_rects)       # Solo las zonas indicadas (optimización)
```

---

## 5. Dibujo: superficies, colores y formas

### 5.1 Colores

```python
NEGRO = (0, 0, 0)
BLANCO = (255, 255, 255)
AZUL = (31, 78, 120)
SEMITRANSPARENTE = (255, 0, 0, 128)      # RGBA (solo en superficies con alfa)
color = pygame.Color("tomato")            # Nombres CSS
color = pygame.Color("#1F4E78")
mezcla = pygame.Color("red").lerp("blue", 0.5)
```

### 5.2 Formas (`pygame.draw`)

```python
pygame.draw.rect(pantalla, AZUL, (50, 50, 200, 100))                      # Relleno
pygame.draw.rect(pantalla, BLANCO, (50, 50, 200, 100), width=3, border_radius=12)   # Borde redondeado
pygame.draw.circle(pantalla, (255, 200, 0), (400, 300), 40)
pygame.draw.ellipse(pantalla, (0, 200, 100), (300, 200, 160, 80))
pygame.draw.line(pantalla, BLANCO, (0, 0), (800, 600), 2)
pygame.draw.lines(pantalla, BLANCO, closed=False, points=[(10, 10), (100, 50), (200, 20)])
pygame.draw.aaline(pantalla, BLANCO, (0, 0), (800, 600))                  # Con suavizado
pygame.draw.polygon(pantalla, (200, 50, 50), [(400, 100), (450, 200), (350, 200)])
pygame.draw.arc(pantalla, BLANCO, (100, 100, 100, 100), 0, 3.14, 3)
```

### 5.3 Superficies

```python
capa = pygame.Surface((200, 100))                      # Opaca
capa.fill((40, 40, 60))
transparente = pygame.Surface((200, 100), pygame.SRCALPHA)   # Con canal alfa por píxel
transparente.fill((0, 0, 0, 150))                      # Negro al 60 %
capa.set_alpha(180)                                    # Transparencia global de la superficie
capa.set_colorkey((255, 0, 255))                       # Un color se vuelve transparente

pantalla.blit(capa, (100, 100))                        # Dibujar en la posición (x, y)
pantalla.blit(capa, rect)                              # O en la posición de un Rect
pantalla.blit(hoja, (0, 0), area=pygame.Rect(32, 0, 32, 32))   # Solo una porción
```

### 5.4 `Rect`: posición y tamaño

```python
rect = pygame.Rect(100, 50, 64, 64)     # x, y, ancho, alto
rect.center = (400, 300)                # Colocar por el centro
rect.midbottom = (400, 580)             # Centro del borde inferior
rect.x += 5                             # Mover
rect.move_ip(5, 0)                      # Mover en el sitio
rect.clamp_ip(pantalla.get_rect())      # Mantener dentro de la pantalla
rect.inflate(-10, -10)                  # Rect reducido (útil para colisiones más justas)
rect.collidepoint(pygame.mouse.get_pos())
```

Puntos de anclaje: `topleft`, `topright`, `bottomleft`, `bottomright`, `center`, `midtop`, `midbottom`, `midleft`, `midright`, `centerx`, `centery`, `left`, `right`, `top`, `bottom`.

> `Rect` usa enteros. Para movimientos suaves, guarde la posición en un `Vector2` (decimales) y copie el valor al `rect` en cada fotograma.

### 5.5 Transformaciones

```python
grande = pygame.transform.scale(imagen, (128, 128))
doble = pygame.transform.scale_by(imagen, 2)
girada = pygame.transform.rotate(imagen, 45)                 # Grados, sentido antihorario
espejo = pygame.transform.flip(imagen, True, False)          # Horizontal
suave = pygame.transform.smoothscale(imagen, (300, 200))
rotozoom = pygame.transform.rotozoom(imagen, 30, 1.5)        # Rotar y escalar con suavizado
```

> Rote siempre a partir de la **imagen original**, no de la ya rotada: rotar repetidamente degrada la imagen y la hace crecer.

---

## 6. Imágenes y texto

### 6.1 Cargar imágenes

```python
from pathlib import Path
RECURSOS = Path(__file__).parent / "recursos"

fondo = pygame.image.load(RECURSOS / "fondo.png").convert()            # Sin transparencia
nave = pygame.image.load(RECURSOS / "nave.png").convert_alpha()        # Con transparencia
pygame.image.save(pantalla, "captura.png")                             # Captura de pantalla
```

> Llame a `convert()` / `convert_alpha()` tras crear la ventana: adapta el formato de la imagen al de la pantalla y multiplica la velocidad de dibujo.

### 6.2 Hojas de sprites

```python
def cortar_hoja(hoja: pygame.Surface, ancho: int, alto: int) -> list[pygame.Surface]:
    fotogramas = []
    for y in range(0, hoja.get_height(), alto):
        for x in range(0, hoja.get_width(), ancho):
            fotogramas.append(hoja.subsurface((x, y, ancho, alto)).copy())
    return fotogramas
```

### 6.3 Texto

```python
fuente = pygame.font.Font(None, 36)                          # Fuente por defecto
fuente = pygame.font.Font(RECURSOS / "fuente.ttf", 28)       # Fuente propia (recomendado)
fuente = pygame.font.SysFont("arial", 24, bold=True)         # Fuente del sistema

texto = fuente.render("Puntos: 120", True, BLANCO)           # True = suavizado
rect_texto = texto.get_rect(topright=(780, 20))
pantalla.blit(texto, rect_texto)

ancho, alto = fuente.size("Texto")                           # Medir sin dibujar
```

> `render()` es costoso: si el texto no cambia, renderícelo una vez y guarde la superficie.

---

## 7. Eventos: teclado, ratón y mando

### 7.1 Cola de eventos

```python
for evento in pygame.event.get():
    if evento.type == pygame.QUIT:
        ejecutando = False
    elif evento.type == pygame.KEYDOWN:
        if evento.key == pygame.K_ESCAPE:
            ejecutando = False
        elif evento.key == pygame.K_SPACE:
            disparar()
    elif evento.type == pygame.MOUSEBUTTONDOWN and evento.button == 1:
        print("Clic en", evento.pos)
    elif evento.type == pygame.MOUSEWHEEL:
        zoom += evento.y
    elif evento.type == pygame.VIDEORESIZE:
        print("Nuevo tamaño:", evento.w, evento.h)
```

| Evento | Atributos |
|---|---|
| `QUIT` | Cierre de la ventana. |
| `KEYDOWN`, `KEYUP` | `key`, `mod`, `unicode`. |
| `MOUSEBUTTONDOWN`, `MOUSEBUTTONUP` | `pos`, `button` (1 izq., 2 central, 3 der.). |
| `MOUSEMOTION` | `pos`, `rel`, `buttons`. |
| `MOUSEWHEEL` | `x`, `y`. |
| `TEXTINPUT` | `text` (escritura con acentos y teclados internacionales). |
| `JOYBUTTONDOWN`, `JOYAXISMOTION`, `JOYHATMOTION` | Mandos. |
| `WINDOWFOCUSLOST`, `WINDOWFOCUSGAINED` | La ventana pierde o recupera el foco. |

### 7.2 Eventos frente a estado del teclado

| Usar eventos (`KEYDOWN`) | Usar estado (`get_pressed`) |
|---|---|
| Acciones puntuales: saltar, disparar, pausar, abrir menú. | Acciones continuas: moverse mientras se mantiene la tecla. |

```python
teclas = pygame.key.get_pressed()
direccion = Vector2(0, 0)
if teclas[pygame.K_LEFT] or teclas[pygame.K_a]:
    direccion.x -= 1
if teclas[pygame.K_RIGHT] or teclas[pygame.K_d]:
    direccion.x += 1
if teclas[pygame.K_UP] or teclas[pygame.K_w]:
    direccion.y -= 1
if teclas[pygame.K_DOWN] or teclas[pygame.K_s]:
    direccion.y += 1
if direccion.length_squared() > 0:
    direccion = direccion.normalize()          # Misma velocidad en diagonal
```

Modificadores:

```python
if evento.type == pygame.KEYDOWN and evento.key == pygame.K_s and evento.mod & pygame.KMOD_CTRL:
    guardar_partida()
```

### 7.3 Ratón

```python
x, y = pygame.mouse.get_pos()
izquierdo, central, derecho = pygame.mouse.get_pressed()
pygame.mouse.set_visible(False)
pygame.mouse.set_cursor(pygame.SYSTEM_CURSOR_HAND)
```

### 7.4 Mandos (*joysticks*)

```python
pygame.joystick.init()
mandos = [pygame.joystick.Joystick(i) for i in range(pygame.joystick.get_count())]

for evento in pygame.event.get():
    if evento.type == pygame.JOYDEVICEADDED:
        mandos.append(pygame.joystick.Joystick(evento.device_index))
    elif evento.type == pygame.JOYBUTTONDOWN and evento.button == 0:
        saltar()

if mandos:
    eje_x = mandos[0].get_axis(0)               # -1.0 … 1.0
    if abs(eje_x) < 0.15:
        eje_x = 0                                # Zona muerta
```

### 7.5 Eventos propios y temporizadores

```python
APARECER_ENEMIGO = pygame.event.custom_type()
pygame.time.set_timer(APARECER_ENEMIGO, 1500)             # Cada 1,5 s

for evento in pygame.event.get():
    if evento.type == APARECER_ENEMIGO:
        crear_enemigo()

pygame.time.set_timer(APARECER_ENEMIGO, 0)                # Detener
```

---

## 8. Movimiento y tiempo (*delta time*)

Si el movimiento se expresa en «píxeles por fotograma», el juego va más rápido en ordenadores rápidos. La solución es expresarlo en **píxeles por segundo** y multiplicar por el tiempo transcurrido (`dt`):

```python
reloj = pygame.time.Clock()
VELOCIDAD = 300                            # píxeles por segundo
posicion = Vector2(400, 300)

while ejecutando:
    dt = reloj.tick(60) / 1000             # Segundos desde el fotograma anterior
    dt = min(dt, 0.05)                     # Evita saltos enormes si la ventana se congela
    posicion += direccion * VELOCIDAD * dt
    rect.center = round(posicion.x), round(posicion.y)
```

### 8.1 Vectores (`pygame.math.Vector2`)

```python
v = Vector2(3, 4)
v.length()                       # 5.0
v.normalize()                    # Vector unitario
v.angle_to(Vector2(1, 0))
v.rotate(90)                     # Grados
v.distance_to(Vector2(0, 0))
v.lerp(objetivo, 0.1)            # Interpolación (seguir suavemente)
Vector2.from_polar((1, 45))      # Desde (longitud, ángulo)
```

Apuntar hacia el ratón:

```python
hacia_raton = Vector2(pygame.mouse.get_pos()) - posicion
angulo = hacia_raton.angle_to(Vector2(1, 0))
imagen_rotada = pygame.transform.rotate(imagen_original, angulo)
rect = imagen_rotada.get_rect(center=posicion)
```

### 8.2 Gravedad y salto

```python
GRAVEDAD = 1800                  # píxeles/s²
FUERZA_SALTO = 650

velocidad.y += GRAVEDAD * dt
posicion += velocidad * dt
if posicion.y >= SUELO:
    posicion.y, velocidad.y, en_suelo = SUELO, 0, True
if salto_pulsado and en_suelo:
    velocidad.y, en_suelo = -FUERZA_SALTO, False
```

### 8.3 FPS y tiempo

```python
reloj.get_fps()                  # FPS reales (promedio)
pygame.time.get_ticks()          # Milisegundos desde pygame.init()
```

---

## 9. Sprites y grupos

### 9.1 Definir un sprite

```python
class Jugador(pygame.sprite.Sprite):
    def __init__(self, posicion, *grupos):
        super().__init__(*grupos)
        self.image = pygame.Surface((40, 40), pygame.SRCALPHA)
        pygame.draw.polygon(self.image, (80, 200, 255), [(20, 0), (40, 40), (0, 40)])
        self.rect = self.image.get_rect(center=posicion)
        self.posicion = Vector2(posicion)
        self.velocidad = 300

    def update(self, dt):
        teclas = pygame.key.get_pressed()
        direccion = Vector2(teclas[pygame.K_RIGHT] - teclas[pygame.K_LEFT],
                            teclas[pygame.K_DOWN] - teclas[pygame.K_UP])
        if direccion.length_squared():
            self.posicion += direccion.normalize() * self.velocidad * dt
        self.rect.center = self.posicion
```

> Un sprite necesita, como mínimo, los atributos `image` y `rect`; los grupos los usan para dibujarlo.

### 9.2 Grupos

```python
todos = pygame.sprite.Group()
enemigos = pygame.sprite.Group()

jugador = Jugador((400, 500), todos)                 # Se añade al crearlo
enemigo = Enemigo((100, 50)); todos.add(enemigo); enemigos.add(enemigo)

todos.update(dt)                                      # Llama a update(dt) de cada sprite
todos.draw(pantalla)                                  # Dibuja image en rect de cada uno
enemigo.kill()                                        # Lo elimina de todos sus grupos
len(enemigos)                                         # Número de sprites
for e in enemigos: ...                                # Recorrer
```

| Clase | Uso |
|---|---|
| `Group` | Grupo básico. |
| `GroupSingle` | Contiene un único sprite (jugador). |
| `LayeredUpdates` | Dibuja por capas (`layer`): fondo, objetos, primer plano. |
| `RenderUpdates` | `draw()` devuelve los rects modificados (para `display.update`). |

---

## 10. Colisiones

### 10.1 Entre rectángulos

```python
if jugador.rect.colliderect(enemigo.rect): ...
if rect.collidepoint(pos_raton): ...
indice = rect.collidelist(lista_de_rects)             # Índice del primero que choca, o -1
```

### 10.2 Entre sprites y grupos

```python
# Un sprite contra un grupo → lista de sprites del grupo con los que choca
golpes = pygame.sprite.spritecollide(jugador, enemigos, dokill=False)

# Grupo contra grupo → {sprite_de_A: [sprites_de_B]}
impactos = pygame.sprite.groupcollide(disparos, enemigos, True, True)   # Elimina ambos
for disparo, alcanzados in impactos.items():
    puntos += 10 * len(alcanzados)

# ¿Choca con alguno? (más rápido: se detiene en el primero)
if pygame.sprite.spritecollideany(jugador, enemigos): ...
```

### 10.3 Colisiones más precisas

```python
pygame.sprite.spritecollide(jugador, enemigos, False, pygame.sprite.collide_circle)   # Usa sprite.radius
pygame.sprite.spritecollide(jugador, enemigos, False, pygame.sprite.collide_mask)     # Píxel a píxel
pygame.sprite.spritecollide(jugador, enemigos, False, pygame.sprite.collide_rect_ratio(0.8))

self.mask = pygame.mask.from_surface(self.image)     # Crear la máscara (en __init__)
```

| Método | Precisión | Coste |
|---|---|---|
| `collide_rect` (por defecto) | Rectángulo | Muy bajo |
| `collide_rect_ratio(0.8)` | Rectángulo reducido | Muy bajo |
| `collide_circle` | Círculo | Bajo |
| `collide_mask` | Píxel a píxel | Alto (úselo tras un filtro por rectángulo) |

### 10.4 Resolver colisiones con paredes (plataformas)

Mueva y resuelva **un eje cada vez**:

```python
def mover(self, dt, paredes):
    self.rect.x += round(self.velocidad.x * dt)
    for pared in paredes:
        if self.rect.colliderect(pared):
            if self.velocidad.x > 0: self.rect.right = pared.left
            elif self.velocidad.x < 0: self.rect.left = pared.right
    self.rect.y += round(self.velocidad.y * dt)
    self.en_suelo = False
    for pared in paredes:
        if self.rect.colliderect(pared):
            if self.velocidad.y > 0:
                self.rect.bottom, self.en_suelo = pared.top, True
            elif self.velocidad.y < 0:
                self.rect.top = pared.bottom
            self.velocidad.y = 0
```

---

## 11. Animación

### 11.1 Fotogramas por tiempo

```python
class Animacion:
    def __init__(self, fotogramas, duracion_fotograma=0.1, bucle=True):
        self.fotogramas, self.duracion, self.bucle = fotogramas, duracion_fotograma, bucle
        self.tiempo = 0.0

    def actualizar(self, dt):
        self.tiempo += dt

    @property
    def imagen(self):
        indice = int(self.tiempo / self.duracion)
        if self.bucle:
            return self.fotogramas[indice % len(self.fotogramas)]
        return self.fotogramas[min(indice, len(self.fotogramas) - 1)]
```

Uso con estados del personaje:

```python
self.animaciones = {"quieto": Animacion(quieto, 0.2), "correr": Animacion(correr, 0.08)}
self.estado = "correr" if abs(self.velocidad.x) > 0 else "quieto"
self.animaciones[self.estado].actualizar(dt)
self.image = self.animaciones[self.estado].imagen
if self.mirando_izquierda:
    self.image = pygame.transform.flip(self.image, True, False)
```

### 11.2 Interpolación (*tweening*)

```python
def suavizar(t: float) -> float:                     # ease-out cúbico, t entre 0 y 1
    return 1 - (1 - t) ** 3

progreso = min(1, tiempo / duracion)
posicion = inicio.lerp(destino, suavizar(progreso))
```

### 11.3 Partículas

```python
class Particula:
    def __init__(self, pos):
        self.pos = Vector2(pos)
        self.vel = Vector2(random.uniform(-1, 1), random.uniform(-1, 1)) * random.uniform(50, 250)
        self.vida = random.uniform(0.3, 0.8)

    def actualizar(self, dt):
        self.pos += self.vel * dt
        self.vel *= 0.95
        self.vida -= dt

particulas = [p for p in particulas if p.vida > 0]   # Eliminar las muertas
```

---

## 12. Sonido y música

```python
pygame.mixer.init()                                     # Lo hace pygame.init(); llámelo si necesita opciones

disparo = pygame.mixer.Sound("recursos/disparo.wav")    # Efectos: WAV u OGG
disparo.set_volume(0.5)
disparo.play()
canal = disparo.play(loops=-1)                           # En bucle (devuelve el canal)
canal.stop()

pygame.mixer.music.load("recursos/musica.ogg")          # Música: se transmite desde el archivo
pygame.mixer.music.set_volume(0.4)
pygame.mixer.music.play(loops=-1, fade_ms=2000)
pygame.mixer.music.pause() / unpause() / fadeout(1500) / stop()

pygame.mixer.set_num_channels(16)                        # Sonidos simultáneos
```

| Tipo | Uso | Formato recomendado |
|---|---|---|
| `mixer.Sound` | Efectos cortos (se cargan en memoria). | WAV / OGG |
| `mixer.music` | Música de fondo (una pista a la vez, en *streaming*). | OGG / MP3 |

> Si el ordenador no tiene dispositivo de audio, `pygame.mixer.init()` lanza `pygame.error`. Capture la excepción y continúe sin sonido.

Sonidos generados por código (sin archivos):

```python
import array, math

def tono(frecuencia=440, duracion=0.15, volumen=0.3, muestreo=44100):
    n = int(muestreo * duracion)
    muestras = array.array("h", (int(volumen * 32767 * math.sin(2 * math.pi * frecuencia * i / muestreo)
                                     * (1 - i / n)) for i in range(n)))
    return pygame.mixer.Sound(buffer=muestras)          # Requiere mixer en mono de 16 bits
```

---

## 13. Cámara, escenarios y mapas de casillas

### 13.1 Cámara que sigue al jugador

El mundo es más grande que la pantalla: se dibuja cada objeto desplazado por la posición de la cámara.

```python
class Camara:
    def __init__(self, ancho_pantalla, alto_pantalla, ancho_mundo, alto_mundo):
        self.desplazamiento = Vector2()
        self.pantalla = Vector2(ancho_pantalla, alto_pantalla)
        self.mundo = Vector2(ancho_mundo, alto_mundo)

    def seguir(self, objetivo: pygame.Rect, dt: float):
        destino = Vector2(objetivo.center) - self.pantalla / 2
        self.desplazamiento += (destino - self.desplazamiento) * min(1, 5 * dt)    # Suavizado
        self.desplazamiento.x = max(0, min(self.desplazamiento.x, self.mundo.x - self.pantalla.x))
        self.desplazamiento.y = max(0, min(self.desplazamiento.y, self.mundo.y - self.pantalla.y))

    def aplicar(self, rect: pygame.Rect) -> pygame.Rect:
        return rect.move(-round(self.desplazamiento.x), -round(self.desplazamiento.y))

for sprite in todos:
    pantalla.blit(sprite.image, camara.aplicar(sprite.rect))
```

### 13.2 Mapa de casillas (*tilemap*) desde texto

```python
NIVEL = [
    "####################",
    "#..................#",
    "#....###......P....#",
    "#..........####....#",
    "####################",
]
TAM = 32
paredes = [pygame.Rect(x * TAM, y * TAM, TAM, TAM)
           for y, fila in enumerate(NIVEL) for x, c in enumerate(fila) if c == "#"]
inicio = next((x * TAM, y * TAM) for y, fila in enumerate(NIVEL) for x, c in enumerate(fila) if c == "P")
```

> Para niveles diseñados con el editor **Tiled** (.tmx), use la librería `pytmx`.

### 13.3 Fondo con paralaje

```python
for capa, factor in [(fondo_lejano, 0.2), (fondo_medio, 0.5)]:
    x = -(camara.desplazamiento.x * factor) % capa.get_width()
    pantalla.blit(capa, (x - capa.get_width(), 0))
    pantalla.blit(capa, (x, 0))
```

---

## 14. Estados del juego y estructura del proyecto

### 14.1 Máquina de estados

Un juego tiene varias «pantallas» (menú, juego, pausa, fin) con lógica distinta. Cada una se implementa como un estado:

```python
class Estado:
    def __init__(self, juego): self.juego = juego
    def manejar_evento(self, evento): ...
    def actualizar(self, dt): ...
    def dibujar(self, pantalla): ...

class Menu(Estado):
    def manejar_evento(self, evento):
        if evento.type == pygame.KEYDOWN and evento.key == pygame.K_RETURN:
            self.juego.cambiar_estado(Partida(self.juego))

class Juego:
    def __init__(self):
        self.estado = Menu(self)
    def cambiar_estado(self, nuevo): self.estado = nuevo
    def ejecutar(self):
        while self.ejecutando:
            dt = self.reloj.tick(60) / 1000
            for evento in pygame.event.get():
                self.estado.manejar_evento(evento)
            self.estado.actualizar(dt)
            self.estado.dibujar(self.pantalla)
            pygame.display.flip()
```

### 14.2 Organización de archivos

```
mi_juego/
├── main.py                # Punto de entrada
├── configuracion.py       # Constantes: tamaño, colores, velocidades
├── estados/               # menu.py, partida.py, fin.py
├── entidades/             # jugador.py, enemigo.py, proyectil.py
├── recursos/
│   ├── imagenes/
│   ├── sonidos/
│   └── fuentes/
└── datos/                 # Niveles, partidas guardadas, récords
```

### 14.3 Guardar datos (récords, opciones)

```python
import json
from pathlib import Path

RUTA = Path.home() / ".mi_juego" / "datos.json"

def cargar() -> dict:
    try:
        return json.loads(RUTA.read_text(encoding="utf-8"))
    except (FileNotFoundError, json.JSONDecodeError):
        return {"record": 0, "volumen": 0.7}

def guardar(datos: dict) -> None:
    RUTA.parent.mkdir(parents=True, exist_ok=True)
    RUTA.write_text(json.dumps(datos, indent=2), encoding="utf-8")
```

---

## 15. Rendimiento y distribución

### 15.1 Rendimiento

| Problema | Solución |
|---|---|
| Imágenes lentas de dibujar | `convert()` / `convert_alpha()` tras cargarlas. |
| Texto que se renderiza en cada fotograma | Renderice solo cuando cambie. |
| Rotar o escalar en cada fotograma | Precalcule las versiones o almacene en caché por ángulo. |
| Muchas colisiones | Filtre por distancia o use una rejilla espacial; `collide_mask` solo tras `colliderect`. |
| Objetos fuera de pantalla | No los dibuje ni actualice (o hágalo con menor frecuencia). |
| Fondo estático complejo | Dibújelo una vez en una superficie y haga un único `blit`. |
| Saber qué es lento | `python -m cProfile -s cumtime main.py`. |

### 15.2 Distribuir el juego

**Ejecutable de escritorio (PyInstaller):**

```bash
pip install pyinstaller
pyinstaller --onefile --windowed --name "MiJuego" --add-data "recursos;recursos" main.py   # Windows (en Linux/macOS: "recursos:recursos")
```

```python
import sys
from pathlib import Path

def recurso(ruta: str) -> Path:
    base = Path(getattr(sys, "_MEIPASS", Path(__file__).resolve().parent))
    return base / ruta
```

**Navegador (pygbag):** convierte el juego a WebAssembly. Requiere que el bucle principal sea asíncrono:

```python
import asyncio

async def main():
    while True:
        ...
        pygame.display.flip()
        await asyncio.sleep(0)       # Cede el control al navegador en cada fotograma

asyncio.run(main())
```

```bash
pip install pygbag
pygbag mi_juego          # Carpeta con main.py → sirve el juego en http://localhost:8000
```

---

## 16. Ejemplo práctico completo

**«Lluvia de meteoritos»**: juego de acción completo con menú, partida, pausa y pantalla de fin. El jugador controla una nave (flechas/WASD o mando), dispara (espacio) y debe destruir meteoritos que caen cada vez más rápido. Incluye *delta time*, sprites y grupos, colisiones por círculo, partículas, potenciadores (escudo y disparo triple), vidas con invulnerabilidad temporal, sacudida de cámara, fondo de estrellas con paralaje, sonidos generados por código, récord guardado en disco y pantalla completa con F11. No necesita archivos de recursos: todos los gráficos se dibujan con código.

```python
"""
Lluvia de meteoritos: juego completo con Pygame (sin recursos externos).
Requiere: pip install pygame      (o pygame-ce)
Controles: flechas/WASD mover · Espacio disparar · P/Esc pausa · F11 pantalla completa
"""
import array
import json
import math
import random
from pathlib import Path

import pygame
from pygame.math import Vector2

# ------------------------------------------------------------------ Configuración
ANCHO, ALTO = 960, 640
FPS = 60
TITULO = "Lluvia de meteoritos"
RUTA_DATOS = Path.home() / ".lluvia_meteoritos.json"

FONDO = pygame.Color("#0B1020")
BLANCO = pygame.Color("#F2F4F8")
GRIS = pygame.Color("#8892A6")
CIAN = pygame.Color("#4FC3F7")
NARANJA = pygame.Color("#FF8A3D")
AMARILLO = pygame.Color("#FFD54F")
ROJO = pygame.Color("#EF5350")
VERDE = pygame.Color("#66BB6A")


# ------------------------------------------------------------------ Utilidades
def cargar_datos() -> dict:
    try:
        return json.loads(RUTA_DATOS.read_text(encoding="utf-8"))
    except (OSError, json.JSONDecodeError):
        return {"record": 0}


def guardar_datos(datos: dict) -> None:
    try:
        RUTA_DATOS.write_text(json.dumps(datos), encoding="utf-8")
    except OSError:
        pass                                            # Sin permisos: el récord no se conserva


class Sonidos:
    """Efectos sintetizados. Si no hay tarjeta de sonido, el juego continúa en silencio."""

    def __init__(self):
        self.activo = False
        try:
            pygame.mixer.init(frequency=22050, size=-16, channels=1)
            self.activo = True
        except pygame.error:
            return
        self.disparo = self._tono(880, 0.06, 0.15, barrido=-400)
        self.explosion = self._ruido(0.35, 0.35)
        self.impacto = self._tono(160, 0.3, 0.4, barrido=-100)
        self.potenciador = self._tono(520, 0.25, 0.25, barrido=600)

    @staticmethod
    def _tono(frecuencia, duracion, volumen, barrido=0.0, muestreo=22050):
        n = int(muestreo * duracion)
        fase, datos = 0.0, array.array("h")
        for i in range(n):
            fase += 2 * math.pi * (frecuencia + barrido * i / n) / muestreo
            datos.append(int(volumen * 32767 * math.copysign(1, math.sin(fase)) * (1 - i / n)))
        return pygame.mixer.Sound(buffer=datos)

    @staticmethod
    def _ruido(duracion, volumen, muestreo=22050):
        n = int(muestreo * duracion)
        datos = array.array("h", (int(volumen * 32767 * random.uniform(-1, 1) * (1 - i / n) ** 2) for i in range(n)))
        return pygame.mixer.Sound(buffer=datos)

    def reproducir(self, nombre: str) -> None:
        if self.activo:
            getattr(self, nombre).play()


# ------------------------------------------------------------------ Entidades
class Nave(pygame.sprite.Sprite):
    VELOCIDAD = 380
    CADENCIA = 0.18                                    # Segundos entre disparos

    def __init__(self, *grupos):
        super().__init__(*grupos)
        self.imagen_base = pygame.Surface((48, 52), pygame.SRCALPHA)
        pygame.draw.polygon(self.imagen_base, CIAN, [(24, 0), (46, 48), (24, 38), (2, 48)])
        pygame.draw.polygon(self.imagen_base, BLANCO, [(24, 10), (32, 34), (24, 30), (16, 34)])
        self.image = self.imagen_base
        self.rect = self.image.get_rect(midbottom=(ANCHO / 2, ALTO - 30))
        self.posicion = Vector2(self.rect.center)
        self.radius = 18
        self.vidas = 3
        self.recarga = 0.0
        self.invulnerable = 0.0
        self.escudo = 0.0
        self.triple = 0.0

    def update(self, dt: float, direccion: Vector2) -> None:
        if direccion.length_squared():
            self.posicion += direccion.normalize() * self.VELOCIDAD * dt
        self.posicion.x = max(28, min(ANCHO - 28, self.posicion.x))
        self.posicion.y = max(ALTO * 0.45, min(ALTO - 30, self.posicion.y))
        self.rect.center = round(self.posicion.x), round(self.posicion.y)
        self.recarga = max(0.0, self.recarga - dt)
        self.invulnerable = max(0.0, self.invulnerable - dt)
        self.escudo = max(0.0, self.escudo - dt)
        self.triple = max(0.0, self.triple - dt)
        # Parpadeo durante la invulnerabilidad
        visible = self.invulnerable == 0 or int(self.invulnerable * 12) % 2 == 0
        self.image = self.imagen_base if visible else pygame.Surface((1, 1), pygame.SRCALPHA)

    def puede_disparar(self) -> bool:
        return self.recarga == 0

    def disparar(self, *grupos) -> None:
        self.recarga = self.CADENCIA
        angulos = (-12, 0, 12) if self.triple else (0,)
        for angulo in angulos:
            Disparo(self.rect.midtop, angulo, *grupos)

    def recibir_impacto(self) -> bool:
        """Devuelve True si el impacto quita una vida."""
        if self.invulnerable or self.escudo:
            return False
        self.vidas -= 1
        self.invulnerable = 2.0
        return True


class Disparo(pygame.sprite.Sprite):
    VELOCIDAD = 720

    def __init__(self, posicion, angulo, *grupos):
        super().__init__(*grupos)
        self.image = pygame.Surface((6, 18), pygame.SRCALPHA)
        pygame.draw.rect(self.image, AMARILLO, (0, 0, 6, 18), border_radius=3)
        self.image = pygame.transform.rotate(self.image, -angulo)
        self.rect = self.image.get_rect(center=posicion)
        self.posicion = Vector2(posicion)
        self.velocidad = Vector2(0, -self.VELOCIDAD).rotate(angulo)
        self.radius = 4

    def update(self, dt: float) -> None:
        self.posicion += self.velocidad * dt
        self.rect.center = round(self.posicion.x), round(self.posicion.y)
        if self.rect.bottom < 0:
            self.kill()


class Meteorito(pygame.sprite.Sprite):
    def __init__(self, nivel: int, *grupos):
        super().__init__(*grupos)
        self.tamano = random.choice((1, 2, 3))
        radio = 14 * self.tamano + random.randint(0, 6)
        self.radius = int(radio * 0.85)
        self.vida = self.tamano
        self.puntos = 40 - 10 * self.tamano
        self.imagen_base = self._dibujar(radio)
        self.image = self.imagen_base
        self.posicion = Vector2(random.uniform(radio, ANCHO - radio), -radio)
        velocidad = random.uniform(110, 190) * (1 + 0.12 * nivel) / (0.6 + 0.2 * self.tamano)
        self.velocidad = Vector2(random.uniform(-60, 60), velocidad)
        self.angulo, self.giro = 0.0, random.uniform(-90, 90)
        self.rect = self.image.get_rect(center=self.posicion)

    @staticmethod
    def _dibujar(radio: int) -> pygame.Surface:
        superficie = pygame.Surface((radio * 2 + 4, radio * 2 + 4), pygame.SRCALPHA)
        centro = Vector2(radio + 2, radio + 2)
        puntos = [centro + Vector2(radio * random.uniform(0.75, 1.0), 0).rotate(a) for a in range(0, 360, 30)]
        pygame.draw.polygon(superficie, pygame.Color("#7A6A5E"), puntos)
        pygame.draw.polygon(superficie, pygame.Color("#A08E7E"), puntos, 3)
        for _ in range(3):
            c = centro + Vector2(random.uniform(0, radio * 0.5), 0).rotate(random.uniform(0, 360))
            pygame.draw.circle(superficie, pygame.Color("#5E5048"), c, max(2, radio // 5))
        return superficie

    def update(self, dt: float) -> None:
        self.posicion += self.velocidad * dt
        self.angulo = (self.angulo + self.giro * dt) % 360
        self.image = pygame.transform.rotate(self.imagen_base, self.angulo)     # Siempre desde la original
        self.rect = self.image.get_rect(center=(round(self.posicion.x), round(self.posicion.y)))
        if self.rect.top > ALTO or not -100 < self.posicion.x < ANCHO + 100:
            self.kill()


class Potenciador(pygame.sprite.Sprite):
    TIPOS = {"escudo": VERDE, "triple": NARANJA}

    def __init__(self, posicion, *grupos):
        super().__init__(*grupos)
        self.tipo = random.choice(list(self.TIPOS))
        self.image = pygame.Surface((26, 26), pygame.SRCALPHA)
        pygame.draw.circle(self.image, self.TIPOS[self.tipo], (13, 13), 12)
        pygame.draw.circle(self.image, BLANCO, (13, 13), 12, 2)
        letra = pygame.font.Font(None, 22).render("E" if self.tipo == "escudo" else "T", True, BLANCO)
        self.image.blit(letra, letra.get_rect(center=(13, 14)))
        self.rect = self.image.get_rect(center=posicion)
        self.posicion = Vector2(posicion)
        self.radius = 13

    def update(self, dt: float) -> None:
        self.posicion.y += 120 * dt
        self.rect.center = round(self.posicion.x), round(self.posicion.y)
        if self.rect.top > ALTO:
            self.kill()


class Particulas:
    def __init__(self):
        self.lista: list[list] = []                  # [posición, velocidad, vida, vida_inicial, color, tamaño]

    def explosion(self, posicion, color, cantidad=24, fuerza=260):
        for _ in range(cantidad):
            velocidad = Vector2(random.uniform(0.2, 1) * fuerza, 0).rotate(random.uniform(0, 360))
            vida = random.uniform(0.3, 0.8)
            self.lista.append([Vector2(posicion), velocidad, vida, vida, pygame.Color(color), random.randint(2, 4)])

    def actualizar(self, dt):
        for p in self.lista:
            p[0] += p[1] * dt
            p[1] *= 0.92
            p[2] -= dt
        self.lista = [p for p in self.lista if p[2] > 0]

    def dibujar(self, superficie, desplazamiento):
        for posicion, _, vida, inicial, color, tamano in self.lista:
            c = color.lerp(FONDO, 1 - vida / inicial)
            pygame.draw.circle(superficie, c, posicion + desplazamiento, tamano)


class CampoEstelar:
    def __init__(self, cantidad=120):
        self.estrellas = [[random.uniform(0, ANCHO), random.uniform(0, ALTO), random.choice((1, 2, 3))]
                          for _ in range(cantidad)]

    def actualizar(self, dt, velocidad=60):
        for e in self.estrellas:
            e[1] += velocidad * e[2] * dt
            if e[1] > ALTO:
                e[0], e[1] = random.uniform(0, ANCHO), 0

    def dibujar(self, superficie):
        for x, y, capa in self.estrellas:
            superficie.fill(GRIS.lerp(BLANCO, capa / 3), (x, y, capa, capa))


# ------------------------------------------------------------------ Estados
class Estado:
    def __init__(self, juego: "Juego"):
        self.juego = juego

    def manejar_evento(self, evento): ...
    def actualizar(self, dt): ...
    def dibujar(self, pantalla): ...


class Menu(Estado):
    def __init__(self, juego):
        super().__init__(juego)
        self.tiempo = 0.0

    def manejar_evento(self, evento):
        if evento.type == pygame.KEYDOWN and evento.key in (pygame.K_RETURN, pygame.K_SPACE):
            self.juego.cambiar_estado(Partida(self.juego))
        elif evento.type == pygame.JOYBUTTONDOWN:
            self.juego.cambiar_estado(Partida(self.juego))

    def actualizar(self, dt):
        self.tiempo += dt
        self.juego.estrellas.actualizar(dt)

    def dibujar(self, pantalla):
        pantalla.fill(FONDO)
        self.juego.estrellas.dibujar(pantalla)
        j = self.juego
        j.texto(pantalla, TITULO.upper(), j.fuente_titulo, CIAN, (ANCHO / 2, ALTO * 0.32))
        if int(self.tiempo * 2) % 2 == 0:
            j.texto(pantalla, "Pulse ENTER para empezar", j.fuente, BLANCO, (ANCHO / 2, ALTO * 0.52))
        j.texto(pantalla, "Flechas/WASD: mover · Espacio: disparar · P: pausa · F11: pantalla completa",
                j.fuente_pequena, GRIS, (ANCHO / 2, ALTO * 0.62))
        j.texto(pantalla, f"Récord: {j.datos['record']:,}".replace(",", "."), j.fuente, AMARILLO,
                (ANCHO / 2, ALTO * 0.75))


class Partida(Estado):
    def __init__(self, juego):
        super().__init__(juego)
        self.todos = pygame.sprite.Group()
        self.disparos = pygame.sprite.Group()
        self.meteoritos = pygame.sprite.Group()
        self.potenciadores = pygame.sprite.Group()
        self.nave = Nave(self.todos)
        self.particulas = Particulas()
        self.puntos = 0
        self.tiempo = 0.0
        self.proximo_meteorito = 0.5
        self.sacudida = 0.0
        self.mando = pygame.joystick.Joystick(0) if pygame.joystick.get_count() else None

    @property
    def nivel(self) -> int:
        return 1 + int(self.tiempo // 20)

    def manejar_evento(self, evento):
        if evento.type == pygame.KEYDOWN and evento.key in (pygame.K_p, pygame.K_ESCAPE):
            self.juego.cambiar_estado(Pausa(self.juego, self))

    def leer_controles(self) -> tuple[Vector2, bool]:
        teclas = pygame.key.get_pressed()
        direccion = Vector2((teclas[pygame.K_RIGHT] or teclas[pygame.K_d]) - (teclas[pygame.K_LEFT] or teclas[pygame.K_a]),
                            (teclas[pygame.K_DOWN] or teclas[pygame.K_s]) - (teclas[pygame.K_UP] or teclas[pygame.K_w]))
        disparar = teclas[pygame.K_SPACE]
        if self.mando:
            ejes = Vector2(self.mando.get_axis(0), self.mando.get_axis(1))
            if ejes.length() > 0.2:
                direccion += ejes
            disparar = disparar or self.mando.get_button(0)
        return direccion, bool(disparar)

    def actualizar(self, dt):
        self.tiempo += dt
        direccion, disparar = self.leer_controles()
        self.nave.update(dt, direccion)
        if disparar and self.nave.puede_disparar():
            self.nave.disparar(self.todos, self.disparos)
            self.juego.sonidos.reproducir("disparo")

        self.proximo_meteorito -= dt
        if self.proximo_meteorito <= 0:
            Meteorito(self.nivel, self.todos, self.meteoritos)
            self.proximo_meteorito = max(0.18, 0.9 - 0.08 * self.nivel) * random.uniform(0.6, 1.3)

        self.disparos.update(dt)
        self.meteoritos.update(dt)
        self.potenciadores.update(dt)
        self.particulas.actualizar(dt)
        self.juego.estrellas.actualizar(dt, 60 + 10 * self.nivel)
        self.sacudida = max(0.0, self.sacudida - dt)
        self.comprobar_colisiones()

    def comprobar_colisiones(self):
        impactos = pygame.sprite.groupcollide(self.meteoritos, self.disparos, False, True,
                                              pygame.sprite.collide_circle)
        for meteorito in impactos:
            meteorito.vida -= len(impactos[meteorito])
            if meteorito.vida <= 0:
                self.puntos += meteorito.puntos * self.nivel
                self.particulas.explosion(meteorito.posicion, NARANJA, 10 + 8 * meteorito.tamano)
                self.juego.sonidos.reproducir("explosion")
                if random.random() < 0.08:
                    Potenciador(meteorito.posicion, self.todos, self.potenciadores)
                meteorito.kill()
            else:
                self.particulas.explosion(meteorito.posicion, GRIS, 6, 120)

        for meteorito in pygame.sprite.spritecollide(self.nave, self.meteoritos, False, pygame.sprite.collide_circle):
            if self.nave.escudo:
                self.particulas.explosion(meteorito.posicion, VERDE, 16)
                meteorito.kill()
            elif self.nave.recibir_impacto():
                self.particulas.explosion(self.nave.posicion, CIAN, 40, 320)
                self.juego.sonidos.reproducir("impacto")
                self.sacudida = 0.35
                meteorito.kill()
                if self.nave.vidas <= 0:
                    self.juego.cambiar_estado(FinPartida(self.juego, self.puntos, self.tiempo))
                    return

        for potenciador in pygame.sprite.spritecollide(self.nave, self.potenciadores, True, pygame.sprite.collide_circle):
            setattr(self.nave, potenciador.tipo, 8.0)
            self.juego.sonidos.reproducir("potenciador")

    def dibujar(self, pantalla):
        lienzo = self.juego.lienzo
        lienzo.fill(FONDO)
        self.juego.estrellas.dibujar(lienzo)
        self.potenciadores.draw(lienzo)
        self.meteoritos.draw(lienzo)
        self.disparos.draw(lienzo)
        lienzo.blit(self.nave.image, self.nave.rect)
        if self.nave.escudo:
            radio = 34 + 3 * math.sin(self.tiempo * 10)
            pygame.draw.circle(lienzo, VERDE, self.nave.rect.center, radio, 2)
        self.particulas.dibujar(lienzo, Vector2())
        self.dibujar_marcador(lienzo)

        desplazamiento = (0, 0)
        if self.sacudida:
            intensidad = 10 * self.sacudida
            desplazamiento = (random.uniform(-intensidad, intensidad), random.uniform(-intensidad, intensidad))
        pantalla.fill(FONDO)
        pantalla.blit(lienzo, desplazamiento)

    def dibujar_marcador(self, superficie):
        j = self.juego
        superficie.blit(j.fuente.render(f"{self.puntos:,}".replace(",", "."), True, BLANCO), (20, 14))
        superficie.blit(j.fuente_pequena.render(f"Nivel {self.nivel}", True, GRIS), (20, 48))
        for i in range(self.nave.vidas):
            icono = pygame.transform.scale(self.nave.imagen_base, (22, 24))
            superficie.blit(icono, (ANCHO - 34 - i * 30, 16))
        for i, (nombre, restante, color) in enumerate([("Escudo", self.nave.escudo, VERDE),
                                                         ("Triple", self.nave.triple, NARANJA)]):
            if restante:
                barra = pygame.Rect(ANCHO - 170, 52 + i * 18, int(120 * restante / 8), 8)
                pygame.draw.rect(superficie, color, barra, border_radius=4)
                superficie.blit(j.fuente_pequena.render(nombre, True, color), (ANCHO - 240, 46 + i * 18))
        superficie.blit(j.fuente_pequena.render(f"{j.reloj.get_fps():.0f} FPS", True, GRIS), (20, ALTO - 26))


class Pausa(Estado):
    def __init__(self, juego, partida: Partida):
        super().__init__(juego)
        self.partida = partida
        self.velo = pygame.Surface((ANCHO, ALTO), pygame.SRCALPHA)
        self.velo.fill((0, 0, 0, 150))

    def manejar_evento(self, evento):
        if evento.type == pygame.KEYDOWN:
            if evento.key in (pygame.K_p, pygame.K_ESCAPE, pygame.K_RETURN):
                self.juego.cambiar_estado(self.partida)
            elif evento.key == pygame.K_q:
                self.juego.cambiar_estado(Menu(self.juego))

    def dibujar(self, pantalla):
        self.partida.dibujar(pantalla)
        pantalla.blit(self.velo, (0, 0))
        self.juego.texto(pantalla, "PAUSA", self.juego.fuente_titulo, BLANCO, (ANCHO / 2, ALTO * 0.42))
        self.juego.texto(pantalla, "P: continuar · Q: volver al menú", self.juego.fuente, GRIS, (ANCHO / 2, ALTO * 0.56))


class FinPartida(Estado):
    def __init__(self, juego, puntos: int, tiempo: float):
        super().__init__(juego)
        self.puntos, self.tiempo = puntos, tiempo
        self.nuevo_record = puntos > juego.datos["record"]
        if self.nuevo_record:
            juego.datos["record"] = puntos
            guardar_datos(juego.datos)
        self.espera = 1.0                                 # Evita saltarse la pantalla sin querer

    def manejar_evento(self, evento):
        if self.espera == 0 and evento.type == pygame.KEYDOWN and evento.key == pygame.K_RETURN:
            self.juego.cambiar_estado(Partida(self.juego))
        elif evento.type == pygame.KEYDOWN and evento.key == pygame.K_ESCAPE:
            self.juego.cambiar_estado(Menu(self.juego))

    def actualizar(self, dt):
        self.espera = max(0.0, self.espera - dt)
        self.juego.estrellas.actualizar(dt)

    def dibujar(self, pantalla):
        j = self.juego
        pantalla.fill(FONDO)
        j.estrellas.dibujar(pantalla)
        j.texto(pantalla, "FIN DE LA PARTIDA", j.fuente_titulo, ROJO, (ANCHO / 2, ALTO * 0.3))
        j.texto(pantalla, f"Puntuación: {self.puntos:,}".replace(",", "."), j.fuente, BLANCO, (ANCHO / 2, ALTO * 0.45))
        j.texto(pantalla, f"Tiempo: {self.tiempo:.0f} s", j.fuente_pequena, GRIS, (ANCHO / 2, ALTO * 0.52))
        if self.nuevo_record:
            j.texto(pantalla, "¡NUEVO RÉCORD!", j.fuente, AMARILLO, (ANCHO / 2, ALTO * 0.62))
        if self.espera == 0:
            j.texto(pantalla, "ENTER: jugar de nuevo · Esc: menú", j.fuente_pequena, GRIS, (ANCHO / 2, ALTO * 0.75))


# ------------------------------------------------------------------ Juego
class Juego:
    def __init__(self):
        pygame.init()
        pygame.display.set_caption(TITULO)
        self.pantalla = pygame.display.set_mode((ANCHO, ALTO), pygame.SCALED)
        self.lienzo = pygame.Surface((ANCHO, ALTO))         # Se dibuja aquí para poder sacudir la cámara
        self.reloj = pygame.time.Clock()
        self.fuente_titulo = pygame.font.Font(None, 72)
        self.fuente = pygame.font.Font(None, 40)
        self.fuente_pequena = pygame.font.Font(None, 26)
        self.sonidos = Sonidos()
        self.datos = cargar_datos()
        self.estrellas = CampoEstelar()
        pygame.joystick.init()
        self.estado: Estado = Menu(self)
        self.ejecutando = True

    def cambiar_estado(self, nuevo: Estado) -> None:
        self.estado = nuevo

    @staticmethod
    def texto(superficie, contenido, fuente, color, centro):
        imagen = fuente.render(contenido, True, color)
        superficie.blit(imagen, imagen.get_rect(center=centro))

    def paso(self, dt: float) -> None:
        """Un fotograma completo: eventos, actualización y dibujo."""
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                self.ejecutando = False
            elif evento.type == pygame.KEYDOWN and evento.key == pygame.K_F11:
                pygame.display.toggle_fullscreen()
            elif evento.type == pygame.WINDOWFOCUSLOST and isinstance(self.estado, Partida):
                self.cambiar_estado(Pausa(self, self.estado))       # Pausa automática
            else:
                self.estado.manejar_evento(evento)
        self.estado.actualizar(dt)
        self.estado.dibujar(self.pantalla)
        pygame.display.flip()

    def ejecutar(self) -> None:
        while self.ejecutando:
            dt = min(self.reloj.tick(FPS) / 1000, 0.05)
            self.paso(dt)
        pygame.quit()


if __name__ == "__main__":
    Juego().ejecutar()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| La ventana aparece y se cierra al instante | No hay bucle principal. | Añada el bucle `while` con la lectura de eventos. |
| La ventana «no responde» | No se procesan los eventos. | Llame a `pygame.event.get()` en cada fotograma. |
| `pygame.error: video system not initialized` | Se usó la pantalla antes de `pygame.init()` o después de `pygame.quit()`. | Inicialice al principio y salga del bucle antes de `quit()`. |
| No se ve nada de lo dibujado | Falta `pygame.display.flip()`. | Llámelo al final de cada fotograma. |
| Rastros de los objetos al moverse | No se borra la pantalla. | `pantalla.fill(color)` al principio del dibujo. |
| `FileNotFoundError` al cargar imágenes | Ruta relativa al directorio de ejecución. | Use rutas basadas en `Path(__file__).parent`. |
| El juego va lento | Imágenes sin `convert()`, texto renderizado en cada fotograma, rotaciones repetidas. | Véase la sección 15.1. |
| Velocidad distinta en cada ordenador | Movimiento por fotograma. | Multiplique por `dt`. |
| Movimiento «a saltos» o que no avanza a poca velocidad | `Rect` solo guarda enteros. | Guarde la posición en `Vector2` y copie al `rect`. |
| La imagen se deforma o crece al rotarla | Se rota la imagen ya rotada. | Rote siempre la imagen original. |
| `pygame.error: mixer not initialized` / sin audio | No hay dispositivo de sonido. | Capture la excepción y desactive el sonido. |
| `TypeError: Invalid position for blit` | Posición con tipo no válido. | Use una tupla `(x, y)`, un `Vector2` o un `Rect`. |
| Las teclas «se pierden» | Se llamó a `pygame.event.get()` en varios sitios. | Lea los eventos una sola vez por fotograma y repártalos. |
| Conflicto entre `pygame` y `pygame-ce` | Ambos instalados. | Desinstale uno: `pip uninstall pygame pygame-ce` y reinstale el elegido. |

---

## 18. Buenas prácticas

- Respete el orden **eventos → actualizar → dibujar** y no mezcle lógica con dibujo.
- Mueva todo con ***delta time*** (unidades por segundo) y limite `dt` para evitar saltos.
- Use `Vector2` para posiciones y velocidades; `Rect` para dibujar y colisionar.
- Organice el juego en **estados** (menú, partida, pausa, fin) y en clases de entidades.
- Agrupe los objetos en `sprite.Group` para actualizar, dibujar y colisionar en bloque.
- Cargue los recursos una sola vez, con `convert()`/`convert_alpha()` y rutas absolutas.
- Centralice las constantes (tamaño, colores, velocidades) en un módulo de configuración.
- Diseñe colisiones «justas» para el jugador (círculos o rects reducidos).
- Dé respuesta visual y sonora a cada acción (partículas, sacudida, sonido).
- Pruebe en distintas resoluciones y añada pausa automática al perder el foco.

---

## 19. Referencia rápida

```python
pygame.init(); pantalla = pygame.display.set_mode((800, 600), pygame.SCALED); reloj = pygame.time.Clock()
while ejecutando:
    dt = reloj.tick(60) / 1000
    for e in pygame.event.get(): QUIT / KEYDOWN(e.key) / MOUSEBUTTONDOWN(e.pos, e.button) / custom_type()
    teclas = pygame.key.get_pressed(); pygame.mouse.get_pos()
    pantalla.fill(color); pantalla.blit(superficie, pos); pygame.display.flip()
pygame.quit()

pygame.draw.rect / circle / ellipse / line / polygon / arc       (superficie, color, geometría, width)
pygame.Surface((w, h), pygame.SRCALPHA) / .fill / .set_alpha / .subsurface / .get_rect(center=)
pygame.image.load(ruta).convert_alpha() / pygame.image.save(sup, "f.png")
pygame.transform.scale / scale_by / rotate / rotozoom / flip / smoothscale
pygame.font.Font(ruta|None, tamaño).render(texto, True, color)
pygame.Rect(x, y, w, h).center / colliderect / collidepoint / clamp_ip / inflate / move_ip
Vector2(x, y).normalize() / length() / rotate(g) / lerp(v, t) / distance_to(v)

class S(pygame.sprite.Sprite): image, rect, update(dt)     Group().add / update / draw / kill()
pygame.sprite.spritecollide(s, grupo, dokill, collide_circle) / groupcollide(a, b, True, True) / spritecollideany
pygame.mixer.Sound("f.wav").play() / pygame.mixer.music.load("m.ogg"); music.play(-1)
pygame.time.set_timer(EVENTO, ms) / pygame.time.get_ticks() / reloj.get_fps()
```

---

## 20. Ejercicios propuestos

1. Cree una pelota que rebote en los bordes de la ventana y cambie de color en cada rebote.
2. Programe el juego *Pong* para dos jugadores con marcador, sonido y aumento progresivo de la velocidad.
3. Desarrolle un juego de plataformas con un nivel definido en texto, gravedad, salto y cámara que sigue al jugador.
4. Implemente un *Snake* con rejilla, crecimiento, récord guardado en JSON y menú de dificultad.
5. Amplíe el ejemplo completo con un jefe final cada 5 niveles, una tabla de las 10 mejores puntuaciones con nombre y la versión web con *pygbag*.

---

## 21. Referencias

- Documentación oficial: https://www.pygame.org/docs/
- Pygame CE: https://pyga.me/docs/
- Tutoriales y proyectos: https://www.pygame.org/wiki/tutorials
- *Program Arcade Games with Python and Pygame*: https://programarcadegames.com/
- pygbag (juegos en el navegador): https://pypi.org/project/pygbag/
- Recursos gráficos y sonidos libres: https://kenney.nl/assets · https://opengameart.org/
