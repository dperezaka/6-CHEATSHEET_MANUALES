# Manual de uso: Panda3D

> **Categoría:** Videojuegos y gráficos · **Utilidad:** Desarrollo de videojuegos y gráficos 3D
> **Tipo:** Librería externa (requiere instalación con `pip`). Motor 3D completo con núcleo en C++.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera aplicación](#4-primera-aplicación)
5. [El grafo de escena y los `NodePath`](#5-el-grafo-de-escena-y-los-nodepath)
6. [Modelos, texturas y geometría por código](#6-modelos-texturas-y-geometría-por-código)
7. [Cámara](#7-cámara)
8. [Tareas: el bucle del juego](#8-tareas-el-bucle-del-juego)
9. [Entrada: teclado, ratón y mando](#9-entrada-teclado-ratón-y-mando)
10. [Personajes animados (`Actor`)](#10-personajes-animados-actor)
11. [Intervalos y secuencias](#11-intervalos-y-secuencias)
12. [Iluminación, sombras, materiales y niebla](#12-iluminación-sombras-materiales-y-niebla)
13. [Colisiones y físicas](#13-colisiones-y-físicas)
14. [Interfaz gráfica (DirectGUI) y sonido](#14-interfaz-gráfica-directgui-y-sonido)
15. [Configuración, rendimiento y distribución](#15-configuración-rendimiento-y-distribución)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Panda3D** es un **motor de videojuegos 3D** de código abierto, creado por Disney para sus juegos en línea y mantenido hoy por la Universidad Carnegie Mellon y su comunidad. El núcleo está escrito en C++ (rendimiento) y se programa en **Python** (productividad). Incluye todo lo necesario para un juego 3D: grafo de escena, carga de modelos, animación de personajes, iluminación y *shaders*, colisiones, físicas, sonido, interfaz gráfica, red y herramientas de empaquetado.

**Ventajas:**
- Motor completo y maduro, usado en productos comerciales.
- Rendimiento elevado: el trabajo pesado se hace en C++.
- Flexible: no impone editor ni estructura; todo se controla desde código.
- *Pipeline* de renderizado moderno: *shaders* GLSL, sombras, materiales PBR (con `simplepbr`), posprocesado.
- Integración con Bullet (físicas), OpenAL/FMOD (sonido) y formatos estándar (glTF, EGG, BAM).
- Herramienta oficial para crear ejecutables e instaladores (`build_apps`).
- Licencia BSD modificada: uso comercial libre.

**Limitaciones:**
- No tiene editor visual de escenas: los niveles se construyen por código o en Blender.
- Curva de aprendizaje mayor que Ursina o Pygame.
- La documentación es extensa, pero parte de los ejemplos de la comunidad están desactualizados.

| Si necesita… | Opción |
|---|---|
| Juego o simulación 3D completa, con control y rendimiento | **Panda3D** |
| Prototipo 3D rápido con muy poco código (sobre Panda3D) | Ursina |
| Juego 2D | Pygame / Arcade |
| Programar la GPU a bajo nivel | PyOpenGL |

---

## 2. Instalación e importación

```bash
pip install panda3d
pip install panda3d-gltf panda3d-simplepbr       # Opcional: modelos glTF y materiales PBR
```

Comprobar la instalación (muestra una escena de ejemplo si se ejecuta en un escritorio):

```bash
python -c "import panda3d; print(panda3d.__version__)"
```

Importaciones habituales:

```python
from direct.showbase.ShowBase import ShowBase
from direct.task import Task
from direct.actor.Actor import Actor
from direct.interval.IntervalGlobal import Sequence, Parallel, LerpPosInterval, LerpHprInterval, Wait, Func
from direct.gui.OnscreenText import OnscreenText
from direct.gui.DirectGui import DirectButton, DirectFrame, DirectLabel
from panda3d.core import (Point3, Vec3, Vec4, NodePath, AmbientLight, DirectionalLight, PointLight,
                          CollisionTraverser, CollisionNode, CollisionSphere, CollisionHandlerQueue,
                          CollisionHandlerPusher, BitMask32, TextNode, loadPrcFileData)
```

> Panda3D incluye modelos de ejemplo (`models/box`, `models/teapot`, `models/smiley`, `models/frowney`, `models/environment`, `models/panda` con animación de caminar), muy útiles para aprender y para prototipos.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `ShowBase` | Clase base de la aplicación: crea la ventana, la cámara, el grafo de escena y el gestor de tareas. |
| Grafo de escena | Árbol de nodos que representa todo lo que se dibuja. Su raíz 3D es `render`; la 2D, `aspect2d`. |
| `NodePath` | «Asa» para manipular un nodo: posición, rotación, escala, color, padre… |
| Modelo | Geometría cargada de un archivo (`.egg`, `.bam`, `.gltf`). |
| `Actor` | Modelo con esqueleto y animaciones. |
| Tarea (`Task`) | Función que se ejecuta en cada fotograma (equivale al bucle del juego). |
| Evento | Mensaje con nombre (`"space"`, `"mouse1"`, `"colision-jugador"`) al que se responde con `accept()`. |
| Intervalo | Animación de una propiedad durante un tiempo (mover, rotar, cambiar color). |
| HPR | Rotación en grados: *Heading* (giro horizontal, eje Z), *Pitch* (cabeceo, eje X), *Roll* (alabeo, eje Y). |
| PRC | Sistema de configuración del motor (resolución, pantalla completa, sincronización…). |

**Sistema de coordenadas:** Panda3D usa **Z hacia arriba** e **Y hacia delante** (sistema de mano derecha):

```
        Z (arriba)
        │   Y (delante)
        │  ╱
        │ ╱
        └──────── X (derecha)
```

> Muchos programas 3D (Blender exporta en Z arriba; OpenGL y Unity usan Y arriba) emplean convenciones distintas. Los exportadores para Panda3D hacen la conversión automáticamente.

**Raíces del grafo de escena:**

| Nodo | Contenido |
|---|---|
| `render` | Mundo 3D. |
| `render2d` | Capa 2D de -1 a 1 en ambos ejes (se estira con la ventana). |
| `aspect2d` | Capa 2D que respeta la proporción de la ventana (la habitual para la interfaz). |
| `base.a2dTopLeft`, `a2dBottomRight`… | Anclajes en las esquinas de la pantalla. |

---

## 4. Primera aplicación

```python
from direct.showbase.ShowBase import ShowBase
from direct.task import Task


class MiJuego(ShowBase):
    def __init__(self):
        super().__init__()
        self.disableMouse()                                   # Desactiva el control de cámara por ratón por defecto

        self.tetera = self.loader.loadModel("models/teapot")
        self.tetera.reparentTo(self.render)                   # Sin padre en render, no se dibuja
        self.tetera.setPos(0, 0, 0)

        self.camera.setPos(0, -20, 6)
        self.camera.lookAt(0, 0, 1)

        self.taskMgr.add(self.girar, "girar-tetera")
        self.accept("escape", self.userExit)

    def girar(self, task):
        self.tetera.setH(task.time * 60)                      # 60 grados por segundo
        return Task.cont                                      # Seguir ejecutándose


app = MiJuego()
app.run()
```

Tras crear `ShowBase`, quedan disponibles variables globales (`base`, `render`, `loader`, `taskMgr`, `camera`, `globalClock`). Dentro de la clase, use siempre `self.` para mayor claridad.

> `self.disableMouse()` es casi siempre necesario: el control de cámara por defecto (arrastrar con el ratón) anula los cambios de posición que haga por código.

---

## 5. El grafo de escena y los `NodePath`

### 5.1 Transformaciones

```python
modelo.setPos(2, 10, 0)                        # x, y, z
modelo.setX(5); modelo.setZ(modelo.getZ() + 1)
modelo.setHpr(45, 0, 0)                        # Heading, Pitch, Roll (grados)
modelo.setH(90)
modelo.setScale(0.5)                           # Uniforme
modelo.setScale(1, 2, 1)                       # Por ejes
modelo.setPosHprScale(0, 5, 0, 45, 0, 0, 1, 1, 1)
modelo.lookAt(objetivo)                        # Orientarse hacia otro nodo o punto
modelo.headsUp(objetivo)                       # Igual, manteniendo la vertical
```

### 5.2 Jerarquía: posiciones relativas

Las transformaciones de un nodo se aplican a todos sus hijos:

```python
coche = self.render.attachNewNode("coche")       # Nodo vacío (agrupador)
carroceria = self.loader.loadModel("models/box"); carroceria.reparentTo(coche)
rueda = self.loader.loadModel("models/smiley"); rueda.reparentTo(coche); rueda.setPos(1, 1, -0.5)

coche.setPos(10, 0, 0)                           # Toda la jerarquía se mueve
rueda.getPos()                                   # Relativa al padre
rueda.getPos(self.render)                        # Relativa al mundo
rueda.setPos(self.render, 0, 0, 0)               # Colocarla en coordenadas del mundo
rueda.wrtReparentTo(otro)                        # Cambiar de padre sin moverla visualmente
```

### 5.3 Vectores y puntos

```python
from panda3d.core import Vec3, Point3

direccion = Vec3(1, 1, 0).normalized()
distancia = (enemigo.getPos() - jugador.getPos()).length()
adelante = jugador.getQuat(self.render).getForward()        # Vector «delante» del jugador
jugador.setPos(jugador, 0, velocidad * dt, 0)               # Avanzar en su propia dirección
```

### 5.4 Visibilidad, color y otras propiedades

```python
modelo.hide(); modelo.show()
modelo.setColor(1, 0.5, 0, 1)                    # RGBA de 0 a 1
modelo.setColorScale(1, 1, 1, 0.5)               # Multiplica el color (útil para transparencias)
modelo.setTransparency(True)
modelo.setTwoSided(True)
modelo.setBin("fixed", 10); modelo.setDepthTest(False)     # Dibujar siempre encima (marcadores)
modelo.setTag("tipo", "enemigo"); modelo.getTag("tipo")
modelo.removeNode()                              # Eliminarlo de la escena
self.render.ls()                                 # Imprimir el árbol de la escena
self.render.find("**/coche")                     # Buscar por nombre
modelo.findAllMatches("**/rueda*")
```

---

## 6. Modelos, texturas y geometría por código

### 6.1 Formatos

| Formato | Descripción |
|---|---|
| `.egg` | Texto, propio de Panda3D (exportadores para Blender y Maya). |
| `.bam` | Binario, propio de Panda3D: **carga más rápida**, recomendado para distribuir. |
| `.gltf` / `.glb` | Estándar abierto (requiere `panda3d-gltf`). Recomendado al exportar desde Blender. |
| `.obj`, `.fbx`, `.dae` | Soportados mediante Assimp (calidad variable). |

```python
modelo = self.loader.loadModel("modelos/casa.glb")
modelo.reparentTo(self.render)
modelo.writeBamFile("modelos/casa.bam")          # Convertir para cargar más rápido
```

Conversión por línea de comandos:

```bash
gltf2bam casa.glb casa.bam          # Con panda3d-gltf
egg2bam casa.egg -o casa.bam
pview casa.bam                       # Visor de modelos incluido
```

### 6.2 Texturas

```python
textura = self.loader.loadTexture("texturas/hierba.png")
suelo.setTexture(textura, 1)                          # Prioridad 1: sustituye a la del modelo
suelo.setTexScale(TextureStage.getDefault(), 20, 20)  # Repetir la textura 20 veces
```

### 6.3 Geometría por código

`CardMaker` crea rectángulos (suelos, carteles, *sprites* 3D):

```python
from panda3d.core import CardMaker

cm = CardMaker("suelo")
cm.setFrame(-50, 50, -50, 50)                         # Izquierda, derecha, abajo, arriba
suelo = self.render.attachNewNode(cm.generate())
suelo.setP(-90)                                       # Tumbarlo (por defecto está vertical)
suelo.setColor(0.25, 0.5, 0.3, 1)
```

Geometría arbitraria con `GeomVertexData`:

```python
from panda3d.core import (Geom, GeomNode, GeomTriangles, GeomVertexData, GeomVertexFormat,
                          GeomVertexWriter)

def crear_triangulo(nombre="triangulo"):
    datos = GeomVertexData(nombre, GeomVertexFormat.getV3n3c4(), Geom.UHStatic)
    vertice, normal, color = (GeomVertexWriter(datos, c) for c in ("vertex", "normal", "color"))
    for (x, y, z), c in [((-1, 0, 0), (1, 0, 0, 1)), ((1, 0, 0), (0, 1, 0, 1)), ((0, 0, 1.5), (0, 0, 1, 1))]:
        vertice.addData3(x, y, z); normal.addData3(0, -1, 0); color.addData4(*c)
    triangulos = GeomTriangles(Geom.UHStatic)
    triangulos.addVertices(0, 1, 2)
    geom = Geom(datos); geom.addPrimitive(triangulos)
    nodo = GeomNode(nombre); nodo.addGeom(geom)
    return NodePath(nodo)
```

### 6.4 Texto en 3D

```python
from panda3d.core import TextNode
texto = TextNode("cartel"); texto.setText("¡Hola!"); texto.setAlign(TextNode.ACenter)
cartel = self.render.attachNewNode(texto); cartel.setPos(0, 0, 3); cartel.setBillboardPointEye()   # Siempre de cara a la cámara
```

---

## 7. Cámara

```python
self.disableMouse()
self.camera.setPos(0, -30, 10)
self.camera.lookAt(0, 0, 0)
self.camLens.setFov(70)                                # Campo de visión (grados)
self.camLens.setNearFar(0.1, 1000)                     # Planos de recorte
self.setBackgroundColor(0.5, 0.7, 0.9)                 # Color del cielo
```

### 7.1 Cámara en tercera persona

```python
def seguir_camara(self, dt):
    objetivo = self.jugador.getPos(self.render) + self.jugador.getQuat(self.render).xform(Vec3(0, -12, 6))
    actual = self.camera.getPos()
    self.camera.setPos(actual + (objetivo - actual) * min(1, 5 * dt))     # Suavizado
    self.camera.lookAt(self.jugador.getPos() + Vec3(0, 0, 1.5))
```

Más sencillo (sin suavizado): hacer la cámara hija del jugador.

```python
self.camera.reparentTo(self.jugador)
self.camera.setPos(0, -12, 6); self.camera.lookAt(self.jugador, 0, 0, 1.5)
```

### 7.2 Cámara en primera persona con ratón

```python
from panda3d.core import WindowProperties

props = WindowProperties(); props.setCursorHidden(True); props.setMouseMode(WindowProperties.M_relative)
self.win.requestProperties(props)

def mirar(self, task):
    if self.mouseWatcherNode.hasMouse():
        centro_x, centro_y = self.win.getXSize() // 2, self.win.getYSize() // 2
        puntero = self.win.getPointer(0)
        dx, dy = puntero.getX() - centro_x, puntero.getY() - centro_y
        self.camera.setH(self.camera.getH() - dx * 0.15)
        self.camera.setP(max(-85, min(85, self.camera.getP() - dy * 0.15)))
        self.win.movePointer(0, centro_x, centro_y)
    return Task.cont
```

### 7.3 Proyección ortográfica (juegos isométricos o 2,5D)

```python
from panda3d.core import OrthographicLens
lente = OrthographicLens(); lente.setFilmSize(40, 30); self.cam.node().setLens(lente)
```

---

## 8. Tareas: el bucle del juego

Una tarea es una función que el gestor de tareas llama en cada fotograma.

```python
self.taskMgr.add(self.actualizar, "actualizar")

def actualizar(self, task):
    dt = globalClock.getDt()                  # Segundos desde el fotograma anterior
    self.jugador.setY(self.jugador, 5 * dt)   # 5 unidades/s hacia delante
    return task.cont                          # task.cont: continuar · task.done: terminar · task.again: repetir con retardo
```

| Elemento | Uso |
|---|---|
| `globalClock.getDt()` | *Delta time* del fotograma. |
| `globalClock.getFrameTime()` | Tiempo total desde el inicio. |
| `task.time` | Tiempo desde que se añadió la tarea. |
| `self.taskMgr.doMethodLater(2.0, f, "nombre")` | Ejecutar `f` dentro de 2 s (si devuelve `task.again`, se repite). |
| `self.taskMgr.remove("nombre")` | Eliminar una tarea. |
| `extraArgs=[...]`, `appendTask=True` | Pasar argumentos a la tarea. |
| `sort=` | Orden de ejecución entre tareas. |

```python
from direct.showbase.ShowBaseGlobal import globalClock      # Importación explícita (opcional)

def aparecer_enemigo(self, task):
    self.crear_enemigo()
    return task.again                                       # Repetir cada 3 s

self.taskMgr.doMethodLater(3, self.aparecer_enemigo, "aparecer")
```

> Las tareas asíncronas (`async def`) también se admiten: `await Task.pause(1.0)` pausa sin bloquear el juego.

---

## 9. Entrada: teclado, ratón y mando

### 9.1 Eventos de teclado

```python
self.accept("space", self.saltar)                     # Al pulsar
self.accept("space-up", self.fin_salto)               # Al soltar
self.accept("arrow_left-repeat", self.girar)          # Repetición mientras se mantiene
self.accept("control-s", self.guardar)
self.accept("escape", self.userExit)
self.ignore("space")                                   # Dejar de escuchar
```

Nombres habituales: letras (`"a"`, `"w"`), `"space"`, `"enter"`, `"escape"`, `"tab"`, `"shift"`, `"control"`, `"arrow_up"`, `"arrow_down"`, `"arrow_left"`, `"arrow_right"`, `"f1"`…, `"mouse1"` (izq.), `"mouse2"` (central), `"mouse3"` (der.), `"wheel_up"`, `"wheel_down"`.

### 9.2 Estado de las teclas (movimiento continuo)

Patrón con diccionario:

```python
self.teclas = {"adelante": False, "atras": False, "izquierda": False, "derecha": False}
for tecla, accion in [("w", "adelante"), ("s", "atras"), ("a", "izquierda"), ("d", "derecha"),
                      ("arrow_up", "adelante"), ("arrow_down", "atras"),
                      ("arrow_left", "izquierda"), ("arrow_right", "derecha")]:
    self.accept(tecla, self.teclas.__setitem__, [accion, True])
    self.accept(f"{tecla}-up", self.teclas.__setitem__, [accion, False])
```

Alternativa: consultar el teclado físico (independiente de la distribución del teclado):

```python
from panda3d.core import KeyboardButton
pulsada = self.mouseWatcherNode.isButtonDown
if pulsada(KeyboardButton.asciiKey("w")): ...
if pulsada(KeyboardButton.up()): ...
```

### 9.3 Ratón

```python
if self.mouseWatcherNode.hasMouse():
    x, y = self.mouseWatcherNode.getMouse()             # -1 … 1
```

Seleccionar objetos 3D con el ratón (rayo desde la cámara):

```python
from panda3d.core import CollisionRay, GeomNode

self.rayo = CollisionRay()
nodo = CollisionNode("rayo-raton"); nodo.addSolid(self.rayo)
nodo.setFromCollideMask(GeomNode.getDefaultCollideMask())
self.rayo_np = self.camera.attachNewNode(nodo)
self.cola_raton = CollisionHandlerQueue()
self.trav_raton = CollisionTraverser(); self.trav_raton.addCollider(self.rayo_np, self.cola_raton)

def al_clic(self):
    if not self.mouseWatcherNode.hasMouse():
        return
    self.rayo.setFromLens(self.camNode, self.mouseWatcherNode.getMouse())
    self.trav_raton.traverse(self.render)
    if self.cola_raton.getNumEntries():
        self.cola_raton.sortEntries()
        objeto = self.cola_raton.getEntry(0).getIntoNodePath()
        print("Seleccionado:", objeto.findNetTag("seleccionable"))
```

### 9.4 Mandos

```python
from panda3d.core import InputDevice

mandos = self.devices.getDevices(InputDevice.DeviceClass.gamepad)
if mandos:
    self.mando = mandos[0]
    self.attachInputDevice(self.mando, prefix="mando")
    self.accept("mando-face_a", self.saltar)
    eje = self.mando.findAxis(InputDevice.Axis.left_x).value       # -1 … 1
```

---

## 10. Personajes animados (`Actor`)

```python
from direct.actor.Actor import Actor

self.panda = Actor("models/panda-model", {"caminar": "models/panda-walk4"})
self.panda.reparentTo(self.render)
self.panda.setScale(0.005)

self.panda.loop("caminar")                     # Reproducir en bucle
self.panda.play("caminar")                     # Una vez
self.panda.stop()
self.panda.pose("caminar", 5)                  # Quieto en el fotograma 5
self.panda.setPlayRate(1.5, "caminar")         # Velocidad
self.panda.getCurrentAnim()                    # Animación en curso
self.panda.getNumFrames("caminar")
```

Mezcla de animaciones (transición suave):

```python
self.panda.enableBlend()
self.panda.setControlEffect("caminar", 0.7)
self.panda.setControlEffect("correr", 0.3)
```

Unir objetos a un hueso (por ejemplo, un arma en la mano):

```python
mano = self.personaje.exposeJoint(None, "modelRoot", "mano_derecha")
espada.reparentTo(mano)
```

> En glTF, las animaciones se incluyen en el mismo archivo: `Actor("personaje.glb")` y `actor.getAnimNames()` para listarlas.

---

## 11. Intervalos y secuencias

Los intervalos animan propiedades en el tiempo sin escribir tareas:

```python
from direct.interval.IntervalGlobal import (Sequence, Parallel, LerpPosInterval, LerpHprInterval,
                                            LerpScaleInterval, LerpColorScaleInterval, Wait, Func)

subir = LerpPosInterval(puerta, 1.5, Point3(0, 0, 4), blendType="easeInOut")
girar = LerpHprInterval(moneda, 2.0, Vec3(360, 0, 0))
latido = Sequence(LerpScaleInterval(icono, 0.3, 1.2), LerpScaleInterval(icono, 0.3, 1.0))
desvanecer = LerpColorScaleInterval(objeto, 1.0, (1, 1, 1, 0))

girar.loop()                          # En bucle
subir.start()                         # Una vez
Sequence(Wait(1.0), subir, Func(print, "Puerta abierta")).start()
Parallel(subir, girar).start()
intervalo.pause() / resume() / finish()
```

Atajos sobre `NodePath`:

```python
modelo.posInterval(2, Point3(10, 0, 0)).start()
modelo.hprInterval(4, Vec3(360, 0, 0)).loop()
modelo.scaleInterval(0.5, 2).start()
```

| `blendType` | Efecto |
|---|---|
| `"noBlend"` | Velocidad constante. |
| `"easeIn"` / `"easeOut"` / `"easeInOut"` | Acelera al principio / frena al final / ambos. |

---

## 12. Iluminación, sombras, materiales y niebla

### 12.1 Luces

```python
from panda3d.core import AmbientLight, DirectionalLight, PointLight, Spotlight

ambiente = AmbientLight("ambiente"); ambiente.setColor((0.3, 0.3, 0.35, 1))
self.render.setLight(self.render.attachNewNode(ambiente))

sol = DirectionalLight("sol"); sol.setColor((0.9, 0.85, 0.75, 1))
sol_np = self.render.attachNewNode(sol); sol_np.setHpr(45, -60, 0)
self.render.setLight(sol_np)

bombilla = PointLight("bombilla"); bombilla.setColor((1, 0.6, 0.3, 1)); bombilla.setAttenuation((1, 0, 0.02))
bombilla_np = farola.attachNewNode(bombilla); bombilla_np.setZ(3)
self.render.setLight(bombilla_np)
```

> Una luz solo ilumina los nodos sobre los que se activa con `setLight()` (y a sus hijos). `self.render.setLight(...)` ilumina toda la escena.

### 12.2 Sombras e iluminación por píxel

```python
sol.setShadowCaster(True, 2048, 2048)              # Resolución del mapa de sombras
sol.getLens().setFilmSize(60, 60)                  # Zona cubierta por la sombra
sol.getLens().setNearFar(1, 100)
self.render.setShaderAuto()                        # Shaders automáticos: iluminación por píxel, sombras, normal maps
```

### 12.3 Materiales

```python
from panda3d.core import Material

material = Material(); material.setBaseColor((0.8, 0.1, 0.1, 1)); material.setRoughness(0.4); material.setMetallic(0.2)
modelo.setMaterial(material, 1)
```

Para materiales PBR realistas (modelos glTF), use `simplepbr`:

```python
import simplepbr
simplepbr.init(enable_shadows=True, use_normal_maps=True)
```

### 12.4 Niebla y cielo

```python
from panda3d.core import Fog
niebla = Fog("niebla"); niebla.setColor(0.6, 0.7, 0.8); niebla.setExpDensity(0.015)
self.render.setFog(niebla)
self.setBackgroundColor(0.6, 0.7, 0.8)           # Mismo color que la niebla
```

### 12.5 Posprocesado

```python
from direct.filter.CommonFilters import CommonFilters
filtros = CommonFilters(self.win, self.cam)
filtros.setBloom(intensity=0.6)
filtros.setAmbientOcclusion()
filtros.setCartoonInk()
```

---

## 13. Colisiones y físicas

### 13.1 Sistema de colisiones

| Elemento | Función |
|---|---|
| Sólido de colisión | Forma simple: `CollisionSphere`, `CollisionCapsule`, `CollisionBox`, `CollisionRay`, `CollisionPlane`, `CollisionPolygon`. |
| `CollisionNode` | Nodo que contiene sólidos de colisión. |
| Objeto «desde» (*from*) | El que se mueve y comprueba (jugador, proyectil). Se registra en el *traverser*. |
| Objeto «hacia» (*into*) | El que recibe colisiones (paredes, enemigos). |
| `CollisionTraverser` | Comprueba las colisiones en cada fotograma. |
| Manejador (*handler*) | Decide qué hacer: `CollisionHandlerQueue` (lista), `CollisionHandlerEvent` (lanza eventos), `CollisionHandlerPusher` (empuja para no atravesar), `CollisionHandlerFloor` (pegarse al suelo). |
| Máscaras (`BitMask32`) | Filtran qué choca con qué. |

```python
from panda3d.core import CollisionTraverser, CollisionNode, CollisionSphere, CollisionHandlerPusher, BitMask32

self.cTrav = CollisionTraverser()                    # Con este nombre, ShowBase lo ejecuta automáticamente

# Jugador (objeto «desde»): esfera que empuja contra las paredes
nodo = CollisionNode("jugador")
nodo.addSolid(CollisionSphere(0, 0, 1, 1))
nodo.setFromCollideMask(BitMask32.bit(1))
nodo.setIntoCollideMask(BitMask32.allOff())
self.col_jugador = self.jugador.attachNewNode(nodo)
self.empujador = CollisionHandlerPusher()
self.empujador.addCollider(self.col_jugador, self.jugador)
self.cTrav.addCollider(self.col_jugador, self.empujador)

# Pared (objeto «hacia»)
from panda3d.core import CollisionBox, Point3
nodo_pared = CollisionNode("pared")
nodo_pared.addSolid(CollisionBox(Point3(0, 0, 0), 5, 0.5, 2))      # Centro y semiejes
nodo_pared.setIntoCollideMask(BitMask32.bit(1))
pared.attachNewNode(nodo_pared)
```

### 13.2 Eventos de colisión

```python
from panda3d.core import CollisionHandlerEvent

self.eventos_col = CollisionHandlerEvent()
self.eventos_col.addInPattern("%fn-toca-%in")               # Nombre del evento: jugador-toca-moneda
self.cTrav.addCollider(self.col_jugador, self.eventos_col)
self.accept("jugador-toca-moneda", self.recoger)

def recoger(self, entrada):
    moneda = entrada.getIntoNodePath().getParent()
    moneda.removeNode()
```

| Patrón | Significado |
|---|---|
| `%fn` / `%in` | Nombre del nodo «desde» / «hacia». |
| `addInPattern` | Al empezar a tocar. |
| `addAgainPattern` | Mientras se toca. |
| `addOutPattern` | Al dejar de tocar. |

### 13.3 Depuración

```python
self.col_jugador.show()                    # Ver el sólido de colisión
self.cTrav.showCollisions(self.render)     # Resaltar las colisiones detectadas
```

### 13.4 Físicas con Bullet

Para gravedad, rebotes, objetos que caen o vehículos, use el módulo **Bullet** incluido:

```python
from panda3d.bullet import BulletWorld, BulletRigidBodyNode, BulletBoxShape, BulletPlaneShape

self.mundo = BulletWorld(); self.mundo.setGravity(Vec3(0, 0, -9.81))

suelo = BulletRigidBodyNode("suelo"); suelo.addShape(BulletPlaneShape(Vec3(0, 0, 1), 0))
self.render.attachNewNode(suelo); self.mundo.attachRigidBody(suelo)

caja = BulletRigidBodyNode("caja"); caja.setMass(1.0); caja.addShape(BulletBoxShape(Vec3(0.5, 0.5, 0.5)))
caja_np = self.render.attachNewNode(caja); caja_np.setPos(0, 0, 10); self.mundo.attachRigidBody(caja)
self.loader.loadModel("models/box").reparentTo(caja_np)

def fisicas(self, task):
    self.mundo.doPhysics(globalClock.getDt())
    return task.cont
```

---

## 14. Interfaz gráfica (DirectGUI) y sonido

### 14.1 Texto en pantalla

```python
from direct.gui.OnscreenText import OnscreenText
from panda3d.core import TextNode

self.marcador = OnscreenText(text="Puntos: 0", parent=self.a2dTopLeft, pos=(0.08, -0.1), scale=0.07,
                             fg=(1, 1, 1, 1), align=TextNode.ALeft, shadow=(0, 0, 0, 0.6), mayChange=True)
self.marcador.setText("Puntos: 10")
```

### 14.2 Botones, paneles y controles

```python
from direct.gui.DirectGui import DirectButton, DirectFrame, DirectLabel, DirectSlider, DirectEntry, DGG

panel = DirectFrame(frameColor=(0, 0, 0, 0.6), frameSize=(-0.6, 0.6, -0.4, 0.4))
DirectLabel(parent=panel, text="Pausa", scale=0.1, pos=(0, 0, 0.2), frameColor=(0, 0, 0, 0), text_fg=(1, 1, 1, 1))
DirectButton(parent=panel, text="Continuar", scale=0.07, pos=(0, 0, 0), command=self.reanudar,
             frameColor=(0.12, 0.31, 0.47, 1), text_fg=(1, 1, 1, 1), pad=(0.4, 0.2), relief=DGG.FLAT)
volumen = DirectSlider(parent=panel, range=(0, 1), value=0.7, scale=0.4, pos=(0, 0, -0.2),
                       command=lambda: self.sfxManagerList[0].setVolume(volumen["value"]))
nombre = DirectEntry(scale=0.06, width=12, initialText="Jugador", command=self.guardar_nombre)

panel.hide(); panel.show(); panel.destroy()
```

### 14.3 Sonido

```python
self.musica = self.loader.loadMusic("sonidos/musica.ogg")
self.musica.setLoop(True); self.musica.setVolume(0.5); self.musica.play()

self.salto = self.loader.loadSfx("sonidos/salto.wav")
self.salto.play()
```

Sonido posicional 3D:

```python
from direct.showbase.Audio3DManager import Audio3DManager
audio3d = Audio3DManager(self.sfxManagerList[0], self.camera)
motor = audio3d.loadSfx("sonidos/motor.ogg"); audio3d.attachSoundToObject(motor, coche); motor.setLoop(True); motor.play()
```

---

## 15. Configuración, rendimiento y distribución

### 15.1 Configuración (PRC)

Antes de crear `ShowBase`:

```python
from panda3d.core import loadPrcFileData

loadPrcFileData("", """
window-title Mi juego 3D
win-size 1280 720
fullscreen false
sync-video true
show-frame-rate-meter true
framebuffer-multisample 1
multisamples 4
""")
```

| Variable | Efecto |
|---|---|
| `win-size 1280 720` | Tamaño de la ventana. |
| `fullscreen true` | Pantalla completa. |
| `sync-video true` | Sincronización vertical. |
| `show-frame-rate-meter true` | Contador de FPS. |
| `framebuffer-multisample 1` + `multisamples 4` | Antialiasing. |
| `audio-library-name null` | Desactiva el sonido (equipos sin audio). |
| `model-path $MAIN_DIR/recursos` | Carpeta adicional para buscar modelos. |
| `window-type offscreen` | Renderizar sin ventana (servidores, capturas). |

Cambiar propiedades de la ventana en ejecución:

```python
from panda3d.core import WindowProperties
props = WindowProperties(); props.setFullscreen(True); props.setSize(1920, 1080)
self.win.requestProperties(props)
```

### 15.2 Rendimiento

| Técnica | Cómo |
|---|---|
| Ver dónde se va el tiempo | `PStats`: `loadPrcFileData("", "want-pstats 1")` y ejecutar `pstats`. |
| Reducir llamadas de dibujo | `nodo.flattenStrong()` en geometría estática; `RigidBodyCombiner` en objetos que se mueven poco. |
| Muchas copias del mismo objeto | `modelo.instanceTo(padre)` o *instancing* por hardware. |
| Objetos lejanos | Niveles de detalle (`LODNode`), niebla y plano lejano de la cámara. |
| Carga rápida | Convertir los modelos a `.bam`. |
| Analizar la escena | `self.render.analyze()`. |
| Ver geometría | `self.toggleWireframe()`, `self.toggleTexture()`. |

### 15.3 Distribución (`build_apps`)

```python
# setup.py
from setuptools import setup

setup(
    name="mi-juego",
    options={
        "build_apps": {
            "gui_apps": {"MiJuego": "main.py"},
            "include_patterns": ["recursos/**/*"],
            "plugins": ["pandagl", "p3openal_audio"],
            "platforms": ["win_amd64", "manylinux2014_x86_64", "macosx_10_13_x86_64"],
            "log_filename": "$USER_APPDATA/MiJuego/salida.log",
        }
    },
)
```

```bash
python setup.py build_apps        # Ejecutables por plataforma en build/
python setup.py bdist_apps        # Archivos comprimidos / instaladores
```

---

## 16. Ejemplo práctico completo

**«El panda recolector»**: juego 3D en tercera persona con los modelos incluidos en Panda3D. El jugador controla al panda animado (WASD/flechas) en un recinto con obstáculos; debe recoger esferas sonrientes antes de que termine el tiempo mientras esquiva a los «enfadados», que le persiguen cada vez más rápido. Incluye cámara que sigue al jugador con suavizado, luces con sombras, niebla, colisiones con paredes (`CollisionHandlerPusher`) y con objetos (`CollisionHandlerEvent`), intervalos, interfaz con DirectGUI (menú, marcador, pausa, fin), máquina de estados y récord guardado. No necesita archivos externos.

```python
"""
El panda recolector: juego 3D con Panda3D (solo modelos incluidos en la librería).
Requiere: pip install panda3d
Controles: WASD/flechas mover · P/Esc pausa · Enter empezar
"""
import json
import math
import random
from pathlib import Path

from direct.actor.Actor import Actor
from direct.gui.DirectGui import DGG, DirectButton, DirectFrame
from direct.gui.OnscreenText import OnscreenText
from direct.interval.IntervalGlobal import Func, LerpScaleInterval, Parallel, Sequence
from direct.showbase.ShowBase import ShowBase
from panda3d.core import (AmbientLight, BitMask32, CardMaker, CollisionBox, CollisionHandlerEvent,
                          CollisionHandlerPusher, CollisionNode, CollisionSphere, CollisionTraverser,
                          DirectionalLight, Fog, Point3, TextNode, Vec3, loadPrcFileData)

loadPrcFileData("", """
window-title El panda recolector
win-size 1280 720
sync-video true
framebuffer-multisample 1
multisamples 4
""")

RUTA_RECORD = Path.home() / ".panda_recolector.json"
TAM_RECINTO = 30                       # Semilado del recinto
DURACION = 90                          # Segundos de partida
MASCARA_PAREDES = BitMask32.bit(1)
MASCARA_OBJETOS = BitMask32.bit(2)
AZUL = (0.12, 0.31, 0.47, 1)


def cargar_record() -> int:
    try:
        return json.loads(RUTA_RECORD.read_text(encoding="utf-8"))["record"]
    except (OSError, ValueError, KeyError):
        return 0


def guardar_record(valor: int) -> None:
    try:
        RUTA_RECORD.write_text(json.dumps({"record": valor}), encoding="utf-8")
    except OSError:
        pass


class PandaRecolector(ShowBase):
    def __init__(self):
        super().__init__()
        self.disableMouse()
        self.setBackgroundColor(0.62, 0.74, 0.86, 1)
        self.record = cargar_record()
        self.estado = "menu"
        self.teclas = {"adelante": False, "atras": False, "izquierda": False, "derecha": False}

        self.crear_escenario()
        self.crear_iluminacion()
        self.crear_jugador()
        self.crear_colisiones()
        self.crear_interfaz()
        self.configurar_controles()

        self.recogibles = []
        self.enemigos = []
        self.taskMgr.add(self.actualizar, "actualizar")
        self.mostrar_menu()

    # ---------------------------------------------------------------- Construcción de la escena
    def crear_escenario(self):
        cm = CardMaker("suelo")
        cm.setFrame(-TAM_RECINTO, TAM_RECINTO, -TAM_RECINTO, TAM_RECINTO)
        suelo = self.render.attachNewNode(cm.generate())
        suelo.setP(-90)
        suelo.setColor(0.36, 0.58, 0.33, 1)

        # Cuadrícula decorativa
        for i in range(-TAM_RECINTO, TAM_RECINTO + 1, 5):
            for horizontal in (True, False):
                linea = CardMaker("linea")
                linea.setFrame(-TAM_RECINTO, TAM_RECINTO, -0.04, 0.04) if horizontal \
                    else linea.setFrame(-0.04, 0.04, -TAM_RECINTO, TAM_RECINTO)
                nodo = self.render.attachNewNode(linea.generate())
                nodo.setP(-90)
                nodo.setPos((0, i, 0.01) if horizontal else (i, 0, 0.01))
                nodo.setColor(0.31, 0.52, 0.29, 1)

        # Muros perimetrales y obstáculos interiores (centro_x, centro_y, semiancho_x, semiancho_y, altura)
        self.paredes = self.render.attachNewNode("paredes")
        bloques = [(0, TAM_RECINTO, TAM_RECINTO, 0.5, 1.5), (0, -TAM_RECINTO, TAM_RECINTO, 0.5, 1.5),
                   (TAM_RECINTO, 0, 0.5, TAM_RECINTO, 1.5), (-TAM_RECINTO, 0, 0.5, TAM_RECINTO, 1.5)]
        random.seed(3)
        for _ in range(9):
            x, y = random.uniform(-22, 22), random.uniform(-22, 22)
            if abs(x) < 6 and abs(y) < 6:
                continue                                  # Deja libre la zona de salida
            bloques.append((x, y, random.uniform(1, 3), random.uniform(1, 3), random.uniform(1, 2.5)))
        for x, y, sx, sy, alto in bloques:
            caja = self.loader.loadModel("models/box")     # Cubo de 1×1×1 con origen en una esquina
            caja.reparentTo(self.paredes)
            caja.setScale(sx * 2, sy * 2, alto)
            caja.setPos(x - sx, y - sy, 0)
            caja.setTextureOff(1)                          # Quita la textura de ejemplo del modelo
            caja.setColor(0.55, 0.47, 0.40, 1)
            colision = CollisionNode("pared")
            colision.addSolid(CollisionBox(Point3(x, y, alto / 2), sx, sy, alto / 2))
            colision.setIntoCollideMask(MASCARA_PAREDES)
            self.paredes.attachNewNode(colision)

        niebla = Fog("niebla")
        niebla.setColor(0.62, 0.74, 0.86)
        niebla.setExpDensity(0.012)
        self.render.setFog(niebla)

    def crear_iluminacion(self):
        ambiente = AmbientLight("ambiente")
        ambiente.setColor((0.45, 0.45, 0.5, 1))
        self.render.setLight(self.render.attachNewNode(ambiente))

        sol = DirectionalLight("sol")
        sol.setColor((0.9, 0.85, 0.75, 1))
        sol.setShadowCaster(True, 2048, 2048)
        sol.getLens().setFilmSize(70, 70)
        sol.getLens().setNearFar(10, 120)
        sol_np = self.render.attachNewNode(sol)
        sol_np.setPos(30, -30, 50)
        sol_np.lookAt(0, 0, 0)
        self.render.setLight(sol_np)
        self.render.setShaderAuto()                       # Iluminación por píxel y sombras

    def crear_jugador(self):
        self.jugador = self.render.attachNewNode("jugador")      # Nodo raíz: posición y orientación
        self.panda = Actor("models/panda-model", {"caminar": "models/panda-walk4"})
        self.panda.reparentTo(self.jugador)
        self.panda.setScale(0.004)
        self.panda.setH(180)                                     # El modelo mira hacia -Y
        self.velocidad_jugador = 9.0
        self.camera.setPos(0, -16, 9)

    def crear_colisiones(self):
        self.cTrav = CollisionTraverser()                        # ShowBase lo recorre en cada fotograma

        # Esfera del jugador contra las paredes: el empujador impide atravesarlas
        nodo = CollisionNode("cuerpo")
        nodo.addSolid(CollisionSphere(0, 0, 1.2, 1.1))
        nodo.setFromCollideMask(MASCARA_PAREDES)
        nodo.setIntoCollideMask(BitMask32.allOff())
        col_cuerpo = self.jugador.attachNewNode(nodo)
        self.empujador = CollisionHandlerPusher()
        self.empujador.setHorizontal(True)                       # Solo empuja en el plano XY
        self.empujador.addCollider(col_cuerpo, self.jugador)
        self.cTrav.addCollider(col_cuerpo, self.empujador)

        # Segunda esfera contra objetos: solo genera eventos (se pueden atravesar)
        nodo = CollisionNode("jugador")
        nodo.addSolid(CollisionSphere(0, 0, 1.2, 1.1))
        nodo.setFromCollideMask(MASCARA_OBJETOS)
        nodo.setIntoCollideMask(BitMask32.allOff())
        col_jugador = self.jugador.attachNewNode(nodo)
        self.eventos = CollisionHandlerEvent()
        self.eventos.addInPattern("%fn-toca-%in")
        self.cTrav.addCollider(col_jugador, self.eventos)
        self.accept("jugador-toca-recogible", self.recoger)
        self.accept("jugador-toca-enemigo", self.tocar_enemigo)

    def crear_interfaz(self):
        estilo = {"fg": (1, 1, 1, 1), "shadow": (0, 0, 0, 0.7), "mayChange": True}
        self.txt_puntos = OnscreenText(parent=self.a2dTopLeft, pos=(0.08, -0.12), scale=0.08,
                                       align=TextNode.ALeft, **estilo)
        self.txt_tiempo = OnscreenText(parent=self.a2dTopRight, pos=(-0.08, -0.12), scale=0.08,
                                       align=TextNode.ARight, **estilo)
        self.txt_vidas = OnscreenText(parent=self.a2dTopLeft, pos=(0.08, -0.22), scale=0.055,
                                      align=TextNode.ALeft, **estilo)
        self.txt_aviso = OnscreenText(pos=(0, 0.55), scale=0.07, **estilo)

        self.panel = DirectFrame(frameColor=(0, 0, 0, 0.65), frameSize=(-0.85, 0.85, -0.55, 0.55))
        self.txt_panel = OnscreenText(parent=self.panel, pos=(0, 0.3), scale=0.11, fg=(1, 1, 1, 1),
                                      mayChange=True)
        self.txt_panel_sub = OnscreenText(parent=self.panel, pos=(0, 0.12), scale=0.055,
                                          fg=(0.85, 0.85, 0.85, 1), mayChange=True)
        self.boton = DirectButton(parent=self.panel, text="Jugar", scale=0.08, pos=(0, 0, -0.2),
                                  frameColor=AZUL, text_fg=(1, 1, 1, 1), pad=(0.6, 0.3),
                                  relief=DGG.FLAT, command=self.al_pulsar_boton)
        self.mostrar_hud(False)

    def configurar_controles(self):
        for tecla, accion in [("w", "adelante"), ("s", "atras"), ("a", "izquierda"), ("d", "derecha"),
                              ("arrow_up", "adelante"), ("arrow_down", "atras"),
                              ("arrow_left", "izquierda"), ("arrow_right", "derecha")]:
            self.accept(tecla, self.teclas.__setitem__, [accion, True])
            self.accept(f"{tecla}-up", self.teclas.__setitem__, [accion, False])
        self.accept("p", self.alternar_pausa)
        self.accept("escape", self.alternar_pausa)
        self.accept("enter", self.al_pulsar_boton)

    # ---------------------------------------------------------------- Estados
    def mostrar_hud(self, visible: bool):
        for texto in (self.txt_puntos, self.txt_tiempo, self.txt_vidas):
            texto.show() if visible else texto.hide()

    def mostrar_panel(self, titulo: str, subtitulo: str, boton: str):
        self.txt_panel.setText(titulo)
        self.txt_panel_sub.setText(subtitulo)
        self.boton["text"] = boton
        self.panel.show()

    def mostrar_menu(self):
        self.estado = "menu"
        self.mostrar_hud(False)
        self.mostrar_panel("El panda recolector",
                           f"Recoge las caras sonrientes y huye de las enfadadas\n"
                           f"WASD/flechas para moverte - P para pausar\nRécord: {self.record}", "Jugar")

    def al_pulsar_boton(self):
        if self.estado in ("menu", "fin"):
            self.empezar_partida()
        elif self.estado == "pausa":
            self.alternar_pausa()

    def empezar_partida(self):
        for nodo in self.recogibles + self.enemigos:
            nodo.removeNode()
        self.recogibles, self.enemigos = [], []
        self.puntos, self.vidas, self.tiempo_restante = 0, 3, float(DURACION)
        self.invulnerable = 0.0
        self.jugador.setPos(0, 0, 0)
        self.jugador.setH(0)
        for _ in range(12):
            self.crear_recogible()
        for _ in range(2):
            self.crear_enemigo()
        self.taskMgr.doMethodLater(10, self.refuerzo_enemigos, "refuerzos")
        self.panel.hide()
        self.mostrar_hud(True)
        self.estado = "jugando"
        self.avisar("¡A recoger!")

    def alternar_pausa(self):
        if self.estado == "jugando":
            self.estado = "pausa"
            self.panda.stop()
            self.mostrar_panel("Pausa", "Pulse P o el botón para continuar", "Continuar")
        elif self.estado == "pausa":
            self.estado = "jugando"
            self.panel.hide()

    def terminar_partida(self, motivo: str):
        self.estado = "fin"
        self.taskMgr.remove("refuerzos")
        self.panda.stop()
        nuevo = self.puntos > self.record
        if nuevo:
            self.record = self.puntos
            guardar_record(self.record)
        self.mostrar_panel(motivo, f"Puntuación: {self.puntos}" + ("  -  ¡NUEVO RÉCORD!" if nuevo else
                                                                     f"  -  Récord: {self.record}"),
                           "Jugar otra vez")

    # ---------------------------------------------------------------- Objetos
    def posicion_libre(self, distancia_minima=8.0) -> Point3:
        while True:
            p = Point3(random.uniform(-26, 26), random.uniform(-26, 26), 0)
            if (p - self.jugador.getPos()).length() > distancia_minima:
                return p

    def crear_recogible(self):
        nodo = self.render.attachNewNode("recogible")
        modelo = self.loader.loadModel("models/smiley")
        modelo.reparentTo(nodo)
        modelo.setScale(0.6)
        modelo.setZ(1.0)
        modelo.setColorScale(1.0, 0.85, 0.2, 1)               # Tono dorado
        modelo.hprInterval(2.5, Vec3(360, 0, 0)).loop()
        Sequence(modelo.posInterval(0.9, Point3(0, 0, 1.4), blendType="easeInOut"),
                 modelo.posInterval(0.9, Point3(0, 0, 1.0), blendType="easeInOut")).loop()
        colision = CollisionNode("recogible")
        colision.addSolid(CollisionSphere(0, 0, 1.2, 0.8))
        colision.setIntoCollideMask(MASCARA_OBJETOS)
        nodo.attachNewNode(colision)
        nodo.setPos(self.posicion_libre())
        self.recogibles.append(nodo)

    def crear_enemigo(self):
        nodo = self.render.attachNewNode("enemigo")
        modelo = self.loader.loadModel("models/frowney")
        modelo.reparentTo(nodo)
        modelo.setScale(0.9)
        modelo.setZ(1.2)
        colision = CollisionNode("enemigo")
        colision.addSolid(CollisionSphere(0, 0, 1.2, 0.9))
        colision.setIntoCollideMask(MASCARA_OBJETOS)
        nodo.attachNewNode(colision)
        nodo.setPos(self.posicion_libre(15))
        nodo.setPythonTag("velocidad", random.uniform(3.0, 4.5))
        self.enemigos.append(nodo)

    def refuerzo_enemigos(self, task):
        if self.estado != "jugando":
            return task.again
        self.crear_enemigo()
        for enemigo in self.enemigos:
            enemigo.setPythonTag("velocidad", enemigo.getPythonTag("velocidad") * 1.08)
        self.avisar("¡Llegan refuerzos!")
        return task.again

    # ---------------------------------------------------------------- Eventos de colisión
    def recoger(self, entrada):
        if self.estado != "jugando":
            return
        nodo = entrada.getIntoNodePath().getParent()
        if nodo not in self.recogibles:
            return
        self.recogibles.remove(nodo)
        nodo.find("**/+CollisionNode").removeNode()          # Evita recogerlo dos veces
        self.puntos += 10
        Sequence(Parallel(LerpScaleInterval(nodo, 0.25, 1.8), nodo.posInterval(0.25, nodo.getPos() + Vec3(0, 0, 2))),
                 Func(nodo.removeNode)).start()
        self.crear_recogible()

    def tocar_enemigo(self, entrada):
        if self.estado != "jugando" or self.invulnerable > 0:
            return
        self.vidas -= 1
        self.invulnerable = 2.0
        empuje = self.jugador.getPos() - entrada.getIntoNodePath().getParent().getPos()
        empuje.z = 0
        if empuje.length() > 0:
            empuje.normalize()
            self.jugador.setPos(self.jugador.getPos() + empuje * 3)
        if self.vidas <= 0:
            self.terminar_partida("¡Te han atrapado!")
        else:
            self.avisar("¡Ay! Cuidado")

    def avisar(self, texto: str):
        self.txt_aviso.setText(texto)
        self.txt_aviso.setScale(0.07)
        self.taskMgr.remove("ocultar-aviso")
        self.taskMgr.doMethodLater(1.5, lambda task: self.txt_aviso.setText(""), "ocultar-aviso")

    # ---------------------------------------------------------------- Bucle principal
    def actualizar(self, task):
        dt = min(globalClock.getDt(), 0.05)
        if self.estado == "jugando":
            self.mover_jugador(dt)
            self.mover_enemigos(dt)
            self.tiempo_restante -= dt
            self.invulnerable = max(0.0, self.invulnerable - dt)
            visible = self.invulnerable == 0 or int(self.invulnerable * 10) % 2 == 0
            self.panda.show() if visible else self.panda.hide()
            self.txt_puntos.setText(f"Puntos: {self.puntos}")
            self.txt_vidas.setText(f"Vidas: {self.vidas}")
            self.txt_tiempo.setText(f"{max(0, self.tiempo_restante):.0f} s")
            if self.tiempo_restante <= 0:
                self.terminar_partida("¡Tiempo!")
        elif self.estado == "menu":
            self.jugador.setH(self.jugador.getH() + 20 * dt)          # El panda gira en el menú
        self.seguir_camara(dt)
        return task.cont

    def mover_jugador(self, dt):
        giro = self.teclas["izquierda"] - self.teclas["derecha"]
        avance = self.teclas["adelante"] - self.teclas["atras"]
        self.jugador.setH(self.jugador.getH() + giro * 160 * dt)
        self.jugador.setY(self.jugador, avance * self.velocidad_jugador * dt)    # En su propia dirección
        self.jugador.setZ(0)
        if avance or giro:
            if self.panda.getCurrentAnim() != "caminar":
                self.panda.loop("caminar")
            self.panda.setPlayRate(1.6 if avance >= 0 else -1.2, "caminar")
        else:
            self.panda.stop()
            self.panda.pose("caminar", 5)

    def mover_enemigos(self, dt):
        objetivo = self.jugador.getPos()
        for enemigo in self.enemigos:
            direccion = objetivo - enemigo.getPos()
            direccion.z = 0
            if direccion.length() > 0.1:
                direccion.normalize()
                # Desviación sinusoidal para que no avancen en línea recta
                lateral = Vec3(-direccion.y, direccion.x, 0) * math.sin(globalClock.getFrameTime() * 2 + id(enemigo) % 7)
                enemigo.setPos(enemigo.getPos() + (direccion + lateral * 0.4) * enemigo.getPythonTag("velocidad") * dt)
                enemigo.lookAt(objetivo.x, objetivo.y, enemigo.getZ())

    def seguir_camara(self, dt):
        detras = self.jugador.getQuat(self.render).xform(Vec3(0, -16, 0))
        objetivo = self.jugador.getPos() + detras + Vec3(0, 0, 9)
        actual = self.camera.getPos()
        self.camera.setPos(actual + (objetivo - actual) * min(1.0, 4 * dt))
        self.camera.lookAt(self.jugador.getPos() + Vec3(0, 0, 1.5))


if __name__ == "__main__":
    PandaRecolector().run()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| El modelo no aparece | No se añadió a la escena. | `modelo.reparentTo(self.render)`. |
| El modelo no aparece aunque está en `render` | Está fuera de la vista de la cámara o tiene escala enorme/diminuta. | Revise posición y escala; `self.render.ls()`; acerque la cámara. |
| La cámara no obedece a `setPos` | Control de ratón por defecto activo. | `self.disableMouse()`. |
| `IOError: Could not load model file(s)` | Ruta o formato incorrectos. | Ruta relativa al script o en `model-path`; para glTF instale `panda3d-gltf`. |
| Modelo totalmente negro | No hay luces que lo iluminen. | Añada `AmbientLight` y `DirectionalLight` y actívelas con `setLight`. |
| Las sombras no se ven | Falta el generador de *shaders*. | `self.render.setShaderAuto()` y `setShadowCaster(True)`. |
| Velocidad distinta según el ordenador | Movimiento por fotograma. | Multiplique por `globalClock.getDt()`. |
| La tarea se ejecuta una sola vez | La función no devuelve `task.cont`. | `return task.cont`. |
| Las colisiones no se detectan | Máscaras que no coinciden, falta `cTrav` o el colisionador no se registró. | Revise `setFromCollideMask` / `setIntoCollideMask` y `addCollider`. |
| El evento de colisión se dispara muchas veces | Se usa `addAgainPattern` o el objeto sigue tocando. | Use `addInPattern` y elimine o desactive el sólido tras procesarlo. |
| La ventana no abre en un servidor | No hay pantalla. | `loadPrcFileData("", "window-type offscreen")`. |
| Errores de audio en consola | No hay dispositivo de sonido. | `loadPrcFileData("", "audio-library-name null")`. |
| El personaje mira hacia atrás | El modelo se exportó mirando a -Y. | Gire el modelo hijo (`setH(180)`) y mueva el nodo padre. |
| `No definition in for character U+00b7` y el carácter no aparece | La fuente por defecto no incluye ese símbolo. | Use una fuente propia: `self.loader.loadFont("fuente.ttf")` y el parámetro `font=`. |
| Rotaciones extrañas al combinar HPR | Bloqueo de cardán. | Use cuaterniones (`setQuat`) o `lookAt`/`headsUp`. |

---

## 18. Buenas prácticas

- Llame a `disableMouse()` y gestione la cámara por código.
- Separe el **nodo de lógica** (posición, colisiones) del **modelo visual** (escala, orientación del archivo).
- Mueva con *delta time* y limite su valor máximo.
- Use **intervalos** para animaciones de presentación y **tareas** para la lógica del juego.
- Defina máscaras de colisión con nombres (constantes) y use sólidos simples (esferas, cápsulas, cajas).
- Use `.bam` para distribuir y glTF como formato de intercambio con Blender.
- Agrupe la geometría estática y aplique `flattenStrong()` para reducir llamadas de dibujo.
- Organice el juego en estados (menú, partida, pausa, fin) y elimine nodos y tareas al cambiar de estado.
- Configure la aplicación con PRC (resolución, sincronización, antialiasing) en lugar de valores dispersos.
- Mida con PStats antes de optimizar.

---

## 19. Referencia rápida

```python
from direct.showbase.ShowBase import ShowBase
class App(ShowBase):
    def __init__(self): super().__init__(); self.disableMouse(); self.taskMgr.add(self.bucle, "bucle")
    def bucle(self, task): dt = globalClock.getDt(); return task.cont
App().run()

m = self.loader.loadModel("models/box"); m.reparentTo(self.render)
m.setPos(x, y, z) / setHpr(h, p, r) / setScale(s) / lookAt(obj) / setColor(r, g, b, a) / hide() / removeNode()
m.setY(m, distancia)  (avanzar en su dirección) / getPos(self.render) / wrtReparentTo(nuevo)
self.camera.setPos / lookAt / reparentTo;  self.camLens.setFov(70);  self.setBackgroundColor(r, g, b)
self.accept("tecla"/"tecla-up"/"mouse1", f, [args]) / self.ignore(...) / self.taskMgr.doMethodLater(s, f, "n")
Actor("modelo", {"anim": "archivo"}).loop("anim") / play / stop / pose / setPlayRate
modelo.posInterval(t, Point3) / hprInterval / scaleInterval / Sequence(...).start() / Parallel / Wait / Func / .loop()
AmbientLight / DirectionalLight(setShadowCaster) / PointLight → self.render.setLight(np); self.render.setShaderAuto()
self.cTrav = CollisionTraverser(); CollisionNode + CollisionSphere/Box/Ray; setFrom/IntoCollideMask(BitMask32.bit(n))
CollisionHandlerPusher (paredes) / CollisionHandlerEvent.addInPattern("%fn-toca-%in") / CollisionHandlerQueue (rayos)
OnscreenText(text=, pos=, scale=, parent=self.a2dTopLeft) / DirectButton(text=, command=) / DirectFrame
loadPrcFileData("", "win-size 1280 720\nsync-video true") / python setup.py build_apps
```

---

## 20. Ejercicios propuestos

1. Cree un sistema solar con el Sol, tres planetas y una luna usando la jerarquía de nodos y tareas de rotación.
2. Construya un laberinto a partir de una matriz de texto con paredes (`models/box`), colisiones y cámara en primera persona.
3. Programe una torre de cajas que se derrumba con físicas Bullet al lanzar una esfera con el ratón.
4. Diseñe un menú con DirectGUI que permita elegir dificultad, volumen y pantalla completa y guarde la configuración en JSON.
5. Amplíe el ejemplo completo con potenciadores (velocidad, congelar enemigos), un minimapa con una segunda cámara y un ejecutable con `build_apps`.

---

## 21. Referencias

- Documentación oficial (manual): https://docs.panda3d.org/1.10/python/index
- Referencia de la API: https://docs.panda3d.org/1.10/python/reference/index
- Ejemplos incluidos en el SDK: https://github.com/panda3d/panda3d/tree/master/samples
- Foro y Discord de la comunidad: https://discourse.panda3d.org/
- panda3d-gltf y simplepbr: https://github.com/Moguri/panda3d-gltf · https://github.com/Moguri/panda3d-simplepbr
- Exportador de Blender (blend2bam): https://github.com/Moguri/blend2bam
