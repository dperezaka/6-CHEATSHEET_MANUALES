# Manual de uso: SimPy

> **Categoría:** Simulación · **Utilidad:** Simulación de eventos discretos
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** MIT · **Dependencias:** ninguna

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [El entorno y el reloj de simulación](#4-el-entorno-y-el-reloj-de-simulación)
5. [Procesos](#5-procesos)
6. [Eventos](#6-eventos)
7. [Recursos compartidos (`Resource`)](#7-recursos-compartidos-resource)
8. [Contenedores (`Container`)](#8-contenedores-container)
9. [Almacenes (`Store`)](#9-almacenes-store)
10. [Interrupciones](#10-interrupciones)
11. [Aleatoriedad y distribuciones](#11-aleatoriedad-y-distribuciones)
12. [Recogida de estadísticas](#12-recogida-de-estadísticas)
13. [Diseño de experimentos y réplicas](#13-diseño-de-experimentos-y-réplicas)
14. [Simulación en tiempo real](#14-simulación-en-tiempo-real)
15. [Estructura de un modelo](#15-estructura-de-un-modelo)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**SimPy** es una librería de **simulación de eventos discretos** basada en procesos. Permite modelar sistemas en los que entidades (clientes, pedidos, vehículos, piezas) compiten por recursos limitados (empleados, máquinas, muelles, servidores) a lo largo del tiempo, para analizar colas, tiempos de espera, ocupación y cuellos de botella.

El tiempo de simulación avanza **de evento en evento**: no se simula cada segundo, sino solo los instantes en que algo cambia (una llegada, el fin de un servicio, una avería). Por eso se pueden simular meses de operación en segundos.

**Aplicaciones habituales:**
- Centros de atención telefónica, oficinas, hospitales y servicios públicos (dimensionado de plantillas).
- Logística: almacenes, muelles de carga, transporte, cadenas de suministro.
- Fabricación: líneas de producción, averías, mantenimiento, inventarios.
- Redes y sistemas informáticos: colas de peticiones, servidores, capacidad.
- Tráfico, aeropuertos, puertos y gestión de eventos.

**Ventajas:**
- Código Python puro, ligero y sin dependencias.
- Modelado natural con funciones generadoras: cada entidad se describe como una secuencia de pasos.
- Recursos predefinidos para los patrones más comunes (servidores, depósitos, almacenes).
- Flexible: cualquier lógica de Python dentro de los procesos.

**Limitaciones:**
- No incluye interfaz gráfica, animación ni recogida automática de estadísticas: hay que programarlas.
- No incluye herramientas de análisis estadístico de resultados (se usan `statistics`, pandas, SciPy).
- Rendimiento limitado para modelos con decenas de millones de eventos.
- Requiere comprender los generadores de Python (`yield`).

| Si necesita… | Opción |
|---|---|
| Simulación de eventos discretos con código Python | **SimPy** |
| Simulación con interfaz gráfica y animación (no Python) | AnyLogic, Arena, FlexSim |
| Simulación de Monte Carlo sin dimensión temporal | NumPy / Numba |
| Modelos basados en agentes en una rejilla espacial | Mesa |
| Ecuaciones diferenciales (sistemas continuos) | SciPy (`solve_ivp`) |

---

## 2. Instalación e importación

```bash
pip install simpy
```

Comprobar la instalación:

```bash
python -c "import simpy; print(simpy.__version__)"
```

Importaciones habituales:

```python
import random
import statistics
import simpy
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Entorno (`Environment`) | Motor de la simulación: gestiona el reloj y la lista de eventos pendientes. |
| Reloj (`env.now`) | Tiempo de simulación actual. Las unidades (minutos, horas…) las decide el modelador. |
| Proceso | Función generadora que describe el comportamiento de una entidad. Se registra con `env.process()`. |
| Evento | Algo que ocurre en un instante: un plazo cumplido, un recurso disponible, una señal. |
| `yield` | Suspende el proceso hasta que ocurra el evento indicado. |
| `timeout` | Evento que ocurre tras un retardo: representa el paso del tiempo (servicio, desplazamiento…). |
| Recurso (`Resource`) | Elemento con capacidad limitada que las entidades solicitan y liberan (empleados, máquinas). |
| Contenedor (`Container`) | Cantidad continua o homogénea (litros, unidades de stock). |
| Almacén (`Store`) | Colección de objetos concretos (pedidos, piezas, mensajes). |
| Interrupción | Señal que un proceso envía a otro para detener lo que está haciendo (averías, prioridades). |
| Réplica | Ejecución independiente del modelo con otra semilla aleatoria. |

**Cómo funciona:**

```
env.process(p) ──► El proceso se ejecuta hasta su primer yield
                   └─► yield evento ──► se suspende
El entorno toma el siguiente evento por orden de tiempo, avanza el reloj
y reanuda los procesos que esperaban ese evento ──► hasta el siguiente yield
```

---

## 4. El entorno y el reloj de simulación

```python
env = simpy.Environment()                # Reloj en 0
env = simpy.Environment(initial_time=480)   # Reloj inicial (p. ej. 8:00 en minutos)

env.now                  # Tiempo actual
env.peek()               # Instante del siguiente evento (inf si no quedan)
```

### 4.1 Ejecutar la simulación

```python
env.run()                         # Hasta que no queden eventos
env.run(until=480)                # Hasta el instante 480
env.run(until=evento)             # Hasta que ocurra un evento (o termine un proceso)
env.step()                        # Procesar un único evento (depuración)
```

> Si existen procesos con bucles infinitos (`while True`), `env.run()` sin `until` no termina nunca. Indique siempre un límite o diseñe los procesos para que finalicen.

### 4.2 Unidades de tiempo

SimPy no impone unidades. Elija una al empezar y úsela en todo el modelo:

```python
MINUTO = 1
HORA = 60 * MINUTO
JORNADA = 8 * HORA
env.run(until=JORNADA)
```

---

## 5. Procesos

### 5.1 Definir y registrar un proceso

Un proceso es una **función generadora**: contiene al menos un `yield`.

```python
def reloj(env, nombre, intervalo):
    while True:
        print(f"{env.now:5.1f} {nombre}")
        yield env.timeout(intervalo)           # Espera «intervalo» unidades de tiempo

env = simpy.Environment()
env.process(reloj(env, "rápido", 0.5))
env.process(reloj(env, "lento", 1))
env.run(until=2)
```

```
  0.0 rápido
  0.0 lento
  0.5 rápido
  1.0 lento
  1.0 rápido
  1.5 rápido
```

### 5.2 Generador de llegadas

Patrón básico de casi todos los modelos: un proceso que crea entidades a intervalos aleatorios.

```python
def llegadas(env, tasa_por_minuto):
    n = 0
    while True:
        yield env.timeout(random.expovariate(tasa_por_minuto))   # Tiempo entre llegadas
        n += 1
        env.process(cliente(env, f"Cliente {n}"))
```

### 5.3 Esperar a otro proceso y valores de retorno

`env.process()` devuelve un evento que se dispara cuando el proceso termina; su valor es el `return` del generador:

```python
def inspeccion(env):
    yield env.timeout(3)
    return "apta"

def pieza(env):
    resultado = yield env.process(inspeccion(env))     # Espera a que termine
    print(f"{env.now}: inspección {resultado}")
```

### 5.4 Procesos como métodos de una clase

En modelos de cierto tamaño, los procesos se definen como métodos (ver sección 15):

```python
class Taller:
    def __init__(self, env):
        self.env = env
        self.env.process(self.produccion())

    def produccion(self):
        while True:
            yield self.env.timeout(5)
```

---

## 6. Eventos

### 6.1 Tipos de eventos

| Evento | Se dispara cuando… |
|---|---|
| `env.timeout(retardo, value=None)` | Transcurre el retardo. |
| `env.process(generador)` | El proceso termina. |
| `recurso.request()`, `contenedor.get()`, `almacen.put()`… | La operación sobre el recurso puede completarse. |
| `env.event()` | Otro proceso lo activa manualmente con `succeed()` o `fail()`. |
| `env.all_of([...])` / `a & b` | Han ocurrido **todos** los eventos. |
| `env.any_of([...])` / `a \| b` | Ha ocurrido **al menos uno**. |

### 6.2 Esperar varios eventos

```python
def proceso(env):
    plazo_corto = env.timeout(3, value="corto")
    plazo_largo = env.timeout(5, value="largo")

    resultado = yield plazo_corto | plazo_largo       # El primero que ocurra
    print(env.now, list(resultado.values()))           # 3 ['corto']
    print(plazo_corto in resultado)                    # True

    yield env.all_of([env.timeout(1), env.timeout(2)])  # Ambos: espera 2
```

> El resultado de `|` y `&` es un diccionario evento → valor. Use `evento in resultado` para saber cuál se produjo. Es la base para modelar **abandonos** (servicio frente a paciencia) y **plazos máximos**.

### 6.3 Eventos manuales (señales)

```python
def semaforo(env, verde):
    yield env.timeout(10)
    verde.succeed("abierto")                 # Activa el evento con un valor

def vehiculo(env, verde):
    estado = yield verde                     # Espera la señal
    print(f"{env.now}: paso ({estado})")

verde = env.event()
env.process(semaforo(env, verde))
env.process(vehiculo(env, verde))
```

Un evento solo se puede activar **una vez**. Para señales repetidas, cree un evento nuevo cada vez.

### 6.4 Eventos fallidos

```python
evento.fail(ValueError("Pedido rechazado"))

def proceso(env):
    try:
        yield evento
    except ValueError as error:
        print("Capturado:", error)
```

---

## 7. Recursos compartidos (`Resource`)

### 7.1 Recurso básico

Representa un número limitado de servidores idénticos (cajeros, operarios, máquinas). Las peticiones que no pueden atenderse esperan en cola por orden de llegada (FIFO).

```python
mostrador = simpy.Resource(env, capacity=2)

def cliente(env, nombre, mostrador):
    llegada = env.now
    with mostrador.request() as peticion:      # Pide un puesto
        yield peticion                         # Espera a que haya uno libre
        espera = env.now - llegada
        yield env.timeout(random.uniform(3, 7))   # Servicio
    # Al salir del bloque with, el puesto se libera automáticamente
```

> **Use siempre `with`**: garantiza que el recurso se libera aunque el proceso termine de forma inesperada (abandono, interrupción). Sin `with`, debe llamar a `mostrador.release(peticion)`.

### 7.2 Información del recurso

```python
mostrador.capacity       # Capacidad total
mostrador.count          # Puestos ocupados
len(mostrador.queue)     # Peticiones en espera
mostrador.users          # Peticiones en servicio
```

### 7.3 Recurso con prioridad (`PriorityResource`)

Las peticiones con **menor** valor de prioridad se atienden antes; a igual prioridad, por orden de llegada.

```python
agentes = simpy.PriorityResource(env, capacity=1)

with agentes.request(priority=0) as peticion:    # Preferente
    yield peticion
with agentes.request(priority=1) as peticion:    # Normal
    yield peticion
```

### 7.4 Recurso con expulsión (`PreemptiveResource`)

Una petición de mayor prioridad **expulsa** a quien está siendo atendido, que recibe una interrupción:

```python
quirofano = simpy.PreemptiveResource(env, capacity=1)

def paciente(env, nombre, prioridad, duracion):
    with quirofano.request(priority=prioridad) as peticion:
        yield peticion
        try:
            yield env.timeout(duracion)
        except simpy.Interrupt as interrupcion:
            expulsado_por = interrupcion.cause.by        # Petición que provocó la expulsión
            print(f"{env.now}: {nombre} interrumpido")
```

Con `request(priority=p, preempt=False)` una petición prioritaria espera sin expulsar.

---

## 8. Contenedores (`Container`)

Modelan cantidades homogéneas: combustible, materia prima, stock de un producto, saldo.

```python
deposito = simpy.Container(env, capacity=10_000, init=4_000)

def surtidor(env, deposito):
    yield deposito.get(50)              # Espera si no hay 50 disponibles
    yield deposito.put(...)             # Espera si no cabe

deposito.level                          # Nivel actual
deposito.capacity
```

**Reposición automática de inventario:**

```python
def control_stock(env, almacen, punto_pedido, cantidad, plazo_entrega):
    while True:
        if almacen.level <= punto_pedido:
            yield env.timeout(plazo_entrega)
            yield almacen.put(cantidad)
        yield env.timeout(1)                  # Revisión diaria
```

---

## 9. Almacenes (`Store`)

Modelan colecciones de **objetos distintos**: pedidos, piezas, mensajes, vehículos aparcados.

### 9.1 `Store`: cola FIFO de objetos

```python
cinta = simpy.Store(env, capacity=2)

def productor(env):
    for i in range(4):
        yield cinta.put(f"pieza {i}")      # Espera si la cinta está llena
        yield env.timeout(1)

def consumidor(env):
    while True:
        pieza = yield cinta.get()          # Espera si está vacía
        yield env.timeout(3)
```

`cinta.items` contiene la lista de objetos almacenados.

### 9.2 `FilterStore`: obtener objetos que cumplen una condición

```python
muelle = simpy.FilterStore(env)
pedido = yield muelle.get(lambda p: p["destino"] == "Sevilla")
```

### 9.3 `PriorityStore`: obtener primero el de mayor prioridad

```python
pendientes = simpy.PriorityStore(env)
yield pendientes.put(simpy.PriorityItem(priority=1, item="pedido normal"))
yield pendientes.put(simpy.PriorityItem(priority=0, item="pedido urgente"))
siguiente = yield pendientes.get()           # PriorityItem(priority=0, item='pedido urgente')
```

| Tipo | Se obtiene… | Ejemplo |
|---|---|---|
| `Store` | El más antiguo | Cinta transportadora |
| `FilterStore` | El primero que cumple una condición | Muelle con pedidos para varios destinos |
| `PriorityStore` | El de menor valor de prioridad | Cola de pedidos urgentes |

---

## 10. Interrupciones

Un proceso puede interrumpir a otro: el interrumpido recibe una excepción `simpy.Interrupt` en su `yield` actual.

```python
def maquina(env):
    while True:
        try:
            yield env.timeout(10)                         # Fabricando
            print(f"{env.now}: pieza terminada")
        except simpy.Interrupt as interrupcion:
            print(f"{env.now}: parada por {interrupcion.cause}")
            yield env.timeout(5)                          # Reparación

def averias(env, proceso_maquina):
    while True:
        yield env.timeout(random.expovariate(1 / 50))
        proceso_maquina.interrupt("avería")

p = env.process(maquina(env))
env.process(averias(env, p))
```

> Si el proceso interrumpido no captura `simpy.Interrupt`, la simulación se detiene con la excepción. Al interrumpirse, el tiempo restante del `timeout` se pierde: guárdelo si debe reanudarse (`restante = fin_previsto - env.now`).

---

## 11. Aleatoriedad y distribuciones

### 11.1 Distribuciones habituales

| Fenómeno | Distribución | Python |
|---|---|---|
| Tiempo entre llegadas aleatorias independientes | Exponencial | `rng.expovariate(tasa)` (media = 1/tasa) |
| Duración de servicios (asimétrica, positiva) | Lognormal | `rng.lognormvariate(mu, sigma)` |
| Duración con mínimo, moda y máximo estimados | Triangular | `rng.triangular(minimo, maximo, moda)` |
| Duración con poca información (entre a y b) | Uniforme | `rng.uniform(a, b)` |
| Tiempos de reparación, vida útil | Weibull / Gamma | `rng.weibullvariate`, `rng.gammavariate` |
| Decisión sí/no | Bernoulli | `rng.random() < p` |
| Elección entre categorías | Discreta | `rng.choices(opciones, weights=pesos)` |

> **Cuidado con la exponencial:** `expovariate` recibe la **tasa** (llegadas por unidad de tiempo), no la media. Para una media de 4 minutos: `expovariate(1 / 4)`.

### 11.2 Generador propio y semillas

Use un generador `random.Random(semilla)` por réplica en lugar del módulo global:

```python
rng = random.Random(42)                 # Reproducible
tiempo = rng.expovariate(1 / 4)
```

Esto permite repetir exactamente cualquier réplica y comparar escenarios con los mismos números aleatorios (*números aleatorios comunes*), lo que reduce la variabilidad de las comparaciones.

### 11.3 Llegadas con intensidad variable

Para demanda que cambia a lo largo del día (horas punta), se usa el método de *thinning*: generar con la tasa máxima y aceptar cada llegada con probabilidad `tasa(t) / tasa_max` (véase el ejemplo completo).

---

## 12. Recogida de estadísticas

SimPy no recoge estadísticas automáticamente. Los patrones habituales son:

### 12.1 Registrar en el propio proceso

```python
esperas = []

def cliente(env, mostrador):
    llegada = env.now
    with mostrador.request() as peticion:
        yield peticion
        esperas.append(env.now - llegada)
        yield env.timeout(5)
```

### 12.2 Monitor periódico

```python
def monitor(env, recurso, registro, intervalo=1):
    while True:
        registro.append((env.now, recurso.count, len(recurso.queue)))
        yield env.timeout(intervalo)
```

### 12.3 Métricas de colas habituales

| Métrica | Cálculo |
|---|---|
| Espera media | Media de `inicio_servicio − llegada` |
| Nivel de servicio | % de clientes atendidos antes de un umbral |
| Tasa de abandono | Abandonos / llegadas |
| Ocupación (utilización) | Tiempo de servicio total / (capacidad × tiempo) |
| Longitud media de la cola | Media ponderada por el tiempo de `len(recurso.queue)` |
| Tiempo en el sistema | `salida − llegada` |

### 12.4 Análisis con pandas

```python
import pandas as pd

registros.append({"id": n, "llegada": llegada, "inicio": inicio, "fin": env.now})
df = pd.DataFrame(registros)
df["espera"] = df["inicio"] - df["llegada"]
df.groupby(pd.cut(df["llegada"], bins=range(0, 481, 60)))["espera"].mean()   # Por hora
```

---

## 13. Diseño de experimentos y réplicas

### 13.1 Por qué hacer réplicas

Una única ejecución es **un resultado aleatorio**, no una conclusión. Repita la simulación con semillas distintas y estime la media y su intervalo de confianza:

```python
import statistics as st

def intervalo_confianza_95(valores):
    media = st.fmean(valores)
    semiamplitud = 1.96 * st.stdev(valores) / len(valores) ** 0.5
    return media, semiamplitud

resultados = [simular(agentes=10, semilla=s) for s in range(30)]
media, error = intervalo_confianza_95([r["espera"] for r in resultados])
print(f"Espera media: {media:.1f} ± {error:.1f} min")
```

> Con menos de 30 réplicas, sustituya 1,96 por el valor de la t de Student (`scipy.stats.t.ppf(0.975, n - 1)`).

### 13.2 Sistemas terminantes y estacionarios

| Tipo | Ejemplo | Cómo simular |
|---|---|---|
| Terminante | Jornada de una oficina, un evento | Simular el periodo completo, varias réplicas |
| Estacionario | Planta que funciona 24/7 | Descartar un periodo inicial de calentamiento (*warm-up*) y simular periodos largos |

```python
def simular_estacionario(env, calentamiento, duracion):
    env.run(until=calentamiento)
    estadisticas.reiniciar()              # Descartar los datos del periodo inicial
    env.run(until=calentamiento + duracion)
```

### 13.3 Comparar escenarios

```python
for n_agentes in range(6, 13):
    resultados = [simular(n_agentes, semilla) for semilla in range(30)]
    ...
```

Use las **mismas semillas** en todos los escenarios para que las diferencias se deban a los cambios del sistema y no al azar. Para muchos escenarios, paralelice las réplicas con `concurrent.futures.ProcessPoolExecutor`.

---

## 14. Simulación en tiempo real

`RealtimeEnvironment` sincroniza el reloj de simulación con el reloj real. Útil para demostraciones, pruebas de sistemas o *hardware-in-the-loop*:

```python
import simpy.rt

env = simpy.rt.RealtimeEnvironment(factor=0.1, strict=False)   # 1 unidad = 0,1 s reales
env.process(proceso(env))
env.run(until=100)                                             # Tarda unos 10 s
```

Con `strict=True` (por defecto) se lanza un error si el cálculo de un paso tarda más que el tiempo real disponible.

---

## 15. Estructura de un modelo

Para modelos de más de unas decenas de líneas, organice el código en una clase que contenga el entorno, los recursos, los procesos y las estadísticas, y una función que ejecute una réplica y devuelva resultados:

```python
from dataclasses import dataclass, field

@dataclass
class Parametros:
    n_operarios: int = 3
    tiempo_entre_llegadas: float = 4.0
    duracion_turno: float = 480

class Modelo:
    def __init__(self, parametros: Parametros, semilla: int):
        self.p = parametros
        self.env = simpy.Environment()
        self.rng = random.Random(semilla)
        self.operarios = simpy.Resource(self.env, self.p.n_operarios)
        self.esperas: list[float] = []

    def llegadas(self): ...
    def pedido(self, id_pedido): ...

    def ejecutar(self) -> dict:
        self.env.process(self.llegadas())
        self.env.run(until=self.p.duracion_turno)
        return {"espera_media": statistics.fmean(self.esperas)}

resultado = Modelo(Parametros(n_operarios=4), semilla=1).ejecutar()
```

**Proceso de trabajo recomendado:**

1. Definir el objetivo y las métricas (¿qué decisión se quiere tomar?).
2. Recoger datos y ajustar distribuciones (tiempos de llegada, servicio, averías).
3. Construir el modelo en pasos pequeños, comprobando cada uno.
4. **Verificar** (el código hace lo previsto) y **validar** (el modelo reproduce el sistema real).
5. Diseñar los escenarios y ejecutar réplicas.
6. Analizar los resultados con intervalos de confianza.

---

## 16. Ejemplo práctico completo

**Dimensionado de un centro de atención telefónica**: simula una jornada de 8 horas con llamadas que llegan de forma aleatoria (con un pico de 10:00 a 12:00), clientes preferentes atendidos con prioridad, abandono por impaciencia y pausas escalonadas de los agentes. Ejecuta 30 réplicas para cada tamaño de plantilla (6 a 12 agentes) y recomienda el mínimo que cumple el objetivo de nivel de servicio (80 % de llamadas atendidas en 20 segundos) y de abandono (≤ 5 %) con un 95 % de confianza. Usa `PriorityResource`, eventos compuestos (`|`), generadores aleatorios con semilla, *thinning* para la demanda variable e intervalos de confianza.

Requisitos: `pip install simpy`

```python
"""
Simulación de un centro de atención telefónica con SimPy.

Modela llamadas que llegan de forma aleatoria (con más volumen a media mañana),
agentes generalistas, clientes preferentes con prioridad, abandono por
impaciencia y pausas de los agentes. Ejecuta varias réplicas por cada tamaño de
plantilla y recomienda el mínimo que cumple el nivel de servicio objetivo.
"""
import random
import statistics as st
from dataclasses import dataclass, field

import simpy

# ------------------------------------------------------------------ Parámetros (minutos)
DURACION_JORNADA = 8 * 60                 # 9:00 a 17:00
LLAMADAS_HORA_BASE = 90                   # Media de llegadas por hora
PICO = (60, 180, 1.6)                     # De 10:00 a 12:00, 1,6 veces más llamadas
DURACION_MEDIA = 4.5                      # Duración media de una llamada
PACIENCIA_MEDIA = 3.0                     # Espera media antes de colgar
PROB_PREFERENTE = 0.15
PAUSA_CADA, PAUSA_DURACION = 120, 10      # Cada agente descansa 10 min cada 2 h
OBJETIVO_SEGUNDOS, OBJETIVO_PORCENTAJE = 20, 0.80   # 80 % atendidas en menos de 20 s
MAX_ABANDONO = 0.05
REPLICAS = 30
PLANTILLAS = range(6, 13)


@dataclass
class Estadisticas:
    esperas: list[float] = field(default_factory=list)
    esperas_preferentes: list[float] = field(default_factory=list)
    abandonos: int = 0
    llamadas: int = 0
    tiempo_ocupado: float = 0.0
    cola_maxima: int = 0
    fin_ultima_llamada: float = 0.0


class CentroLlamadas:
    def __init__(self, env: simpy.Environment, n_agentes: int, rng: random.Random):
        self.env = env
        self.rng = rng
        self.n_agentes = n_agentes
        self.agentes = simpy.PriorityResource(env, capacity=n_agentes)
        self.stats = Estadisticas()

    # ---------- Procesos
    def generar_llamadas(self):
        """Llegadas de Poisson con intensidad variable (método de *thinning*)."""
        tasa_max = LLAMADAS_HORA_BASE * PICO[2] / 60
        id_llamada = 0
        while True:
            yield self.env.timeout(self.rng.expovariate(tasa_max))
            if self.env.now >= DURACION_JORNADA:
                return
            if self.rng.random() <= self._tasa(self.env.now) / tasa_max:
                id_llamada += 1
                preferente = self.rng.random() < PROB_PREFERENTE
                self.env.process(self.llamada(id_llamada, preferente))

    def llamada(self, id_llamada: int, preferente: bool):
        self.stats.llamadas += 1
        llegada = self.env.now
        prioridad = 0 if preferente else 1                     # Menor valor = más prioridad
        with self.agentes.request(priority=prioridad) as peticion:
            self.stats.cola_maxima = max(self.stats.cola_maxima, len(self.agentes.queue))
            paciencia = self.env.timeout(self.rng.expovariate(1 / PACIENCIA_MEDIA))
            resultado = yield peticion | paciencia             # Lo que ocurra primero
            if peticion not in resultado:
                self.stats.abandonos += 1                      # Cuelga antes de ser atendido
                return
            espera = self.env.now - llegada
            (self.stats.esperas_preferentes if preferente else self.stats.esperas).append(espera)
            duracion = self.rng.lognormvariate(1.3, 0.55)      # Media ≈ 4,3 min, asimétrica
            yield self.env.timeout(duracion)
            self.stats.tiempo_ocupado += duracion
            self.stats.fin_ultima_llamada = max(self.stats.fin_ultima_llamada, self.env.now)

    def pausas(self, agente: int):
        """Cada agente ocupa un puesto con máxima prioridad durante su descanso."""
        yield self.env.timeout(PAUSA_CADA * (agente + 1) / self.n_agentes)   # Pausas escalonadas
        while self.env.now < DURACION_JORNADA:
            with self.agentes.request(priority=-1) as peticion:
                yield peticion
                yield self.env.timeout(PAUSA_DURACION)
            yield self.env.timeout(PAUSA_CADA)

    # ---------- Utilidades
    @staticmethod
    def _tasa(minuto: float) -> float:
        inicio, fin, factor = PICO
        return LLAMADAS_HORA_BASE / 60 * (factor if inicio <= minuto < fin else 1.0)


def simular(n_agentes: int, semilla: int) -> dict[str, float]:
    env = simpy.Environment()
    centro = CentroLlamadas(env, n_agentes, random.Random(semilla))
    env.process(centro.generar_llamadas())
    for agente in range(n_agentes):
        env.process(centro.pausas(agente))
    env.run()                                      # Hasta que no queden eventos (se vacía la cola)

    s = centro.stats
    todas = s.esperas + s.esperas_preferentes
    atendidas = len(todas)
    en_objetivo = sum(e <= OBJETIVO_SEGUNDOS / 60 for e in todas)
    return {
        "nivel_servicio": en_objetivo / s.llamadas,
        "abandono": s.abandonos / s.llamadas,
        "espera_media_s": st.fmean(todas) * 60 if todas else 0.0,
        "espera_pref_s": st.fmean(s.esperas_preferentes) * 60 if s.esperas_preferentes else 0.0,
        "ocupacion": s.tiempo_ocupado / (n_agentes * max(DURACION_JORNADA, s.fin_ultima_llamada)),
        "cola_maxima": s.cola_maxima,
        "llamadas": s.llamadas,
        "atendidas": atendidas,
    }


def intervalo_95(valores: list[float]) -> tuple[float, float]:
    media = st.fmean(valores)
    semiamplitud = 1.96 * st.stdev(valores) / len(valores) ** 0.5
    return media, semiamplitud


def main() -> None:
    print(f"Centro de llamadas · {REPLICAS} réplicas por escenario · objetivo "
          f"{OBJETIVO_PORCENTAJE:.0%} en {OBJETIVO_SEGUNDOS} s y abandono ≤ {MAX_ABANDONO:.0%}\n")
    print(f"{'Agentes':>7}{'Nivel servicio':>20}{'Abandono':>10}{'Espera':>9}"
          f"{'Esp. pref.':>11}{'Ocupación':>11}{'Cola máx.':>10}")

    recomendado = None
    for n in PLANTILLAS:
        resultados = [simular(n, semilla) for semilla in range(REPLICAS)]
        ns, ns_ic = intervalo_95([r["nivel_servicio"] for r in resultados])
        ab, _ = intervalo_95([r["abandono"] for r in resultados])
        espera = st.fmean(r["espera_media_s"] for r in resultados)
        espera_pref = st.fmean(r["espera_pref_s"] for r in resultados)
        ocupacion = st.fmean(r["ocupacion"] for r in resultados)
        cola = max(r["cola_maxima"] for r in resultados)
        cumple = ns - ns_ic >= OBJETIVO_PORCENTAJE and ab <= MAX_ABANDONO
        marca = " ✔" if cumple else ""
        print(f"{n:>7}{ns:>13.1%} ±{ns_ic:5.1%}{ab:>10.1%}{espera:>7.0f} s{espera_pref:>9.0f} s"
              f"{ocupacion:>11.0%}{cola:>10}{marca}")
        if cumple and recomendado is None:
            recomendado = n
        llamadas = st.fmean(r["llamadas"] for r in resultados)   # Similar en todos los escenarios

    print(f"\nLlamadas medias por jornada: {llamadas:.0f}")
    if recomendado:
        print(f"Plantilla recomendada: {recomendado} agentes (mínimo que cumple el objetivo "
              "con el 95 % de confianza)")
    else:
        print("Ninguna plantilla analizada cumple el objetivo: amplíe el rango.")


if __name__ == "__main__":
    main()
```

Resultado:

```
Centro de llamadas · 30 réplicas por escenario · objetivo 80% en 20 s y abandono ≤ 5%

Agentes      Nivel servicio  Abandono   Espera Esp. pref.  Ocupación Cola máx.
      6        32.4% ± 1.1%     31.3%     52 s       27 s        83%        15
      7        44.0% ± 1.4%     23.1%     37 s       20 s        80%        14
      8        56.0% ± 1.5%     16.8%     26 s       15 s        76%        13
      9        65.8% ± 1.7%     12.0%     18 s       11 s        71%        13
     10        75.4% ± 1.4%      7.9%     13 s        8 s        67%        12
     11        81.0% ± 1.5%      5.9%      9 s        6 s        63%        11
     12        87.7% ± 1.2%      3.7%      6 s        3 s        58%        13 ✔

Llamadas medias por jornada: 828
Plantilla recomendada: 12 agentes (mínimo que cumple el objetivo con el 95 % de confianza)
```

> El nivel de servicio se calcula sobre todas las llamadas recibidas (las abandonadas cuentan como no atendidas en plazo). La espera media solo incluye las llamadas atendidas. La ocupación no incluye el tiempo de pausa. Con 11 agentes la media supera el 80 %, pero el límite inferior del intervalo de confianza no lo hace y el abandono supera el 5 %: por eso se recomiendan 12.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| El proceso no hace nada | Se llamó a la función generadora sin registrarla. | `env.process(mi_proceso(env))`. |
| `ValueError: <...> is not a generator` | Se pasó a `env.process()` una función sin `yield` o la función sin llamar. | Añada al menos un `yield` y pase `funcion(env)`, no `funcion`. |
| El tiempo no avanza / todo ocurre en el instante 0 | Se usó `time.sleep()` o falta el `yield` delante de `env.timeout()`. | Use `yield env.timeout(t)`. |
| `env.run()` no termina | Procesos con bucles infinitos y sin `until`. | Indique `env.run(until=...)` o finalice los bucles. |
| El recurso se queda ocupado para siempre | Petición sin `with` ni `release()`, o abandono sin cancelar la petición. | Use siempre `with recurso.request() as peticion:`. |
| `simpy.Interrupt` detiene la simulación | El proceso interrumpido no captura la excepción. | Envuelva el `yield` en `try/except simpy.Interrupt`. |
| `RuntimeError: ... has already been triggered` | Se activó dos veces el mismo `env.event()`. | Cree un evento nuevo para cada señal. |
| Esperas y colas muy distintas a lo esperado | `expovariate` recibió la media en lugar de la tasa. | Use `expovariate(1 / media)`. |
| Resultados distintos en cada ejecución | No se fijó la semilla. | Use `random.Random(semilla)` en cada réplica. |
| Conclusiones que cambian al repetir | Se analizó una sola réplica. | Ejecute réplicas y calcule intervalos de confianza. |
| Ocupación o métricas por tiempo incorrectas | Se divide por `env.now` cuando la simulación siguió avanzando por procesos auxiliares. | Divida por la duración del periodo analizado. |
| Simulación muy lenta | Monitores con intervalos muy pequeños o demasiados procesos auxiliares. | Registre en los eventos en lugar de muestrear; aumente el intervalo del monitor. |

---

## 18. Buenas prácticas

- Defina **una unidad de tiempo** y declare todos los parámetros como constantes o en una clase de parámetros.
- Use **`with recurso.request()`** para garantizar la liberación de los recursos.
- Use un **generador aleatorio por réplica** con semilla (`random.Random(semilla)`) y las mismas semillas en todos los escenarios.
- **Nunca saque conclusiones de una sola réplica**: informe medias con intervalos de confianza.
- Separe el **modelo** (procesos y recursos), la **ejecución de experimentos** (réplicas, escenarios) y el **análisis** (estadísticas, gráficos).
- Empiece con un modelo sencillo, verifíquelo con casos de resultado conocido (p. ej. fórmulas de colas M/M/c) y añada detalle de forma gradual.
- Ajuste las distribuciones con **datos reales** siempre que sea posible.
- Documente los supuestos del modelo: qué se simplifica y por qué.
- En sistemas estacionarios, elimine el periodo de calentamiento antes de medir.
- Guarde los resultados de cada réplica (no solo las medias) para poder analizarlos después.

---

## 19. Referencia rápida

```python
import simpy

env = simpy.Environment(initial_time=0); env.now; env.peek()
env.process(generador(env)) · env.run() · env.run(until=t) · env.run(until=evento)

yield env.timeout(t, value=v)                     # Esperar un tiempo
valor = yield env.process(otro(env))              # Esperar a otro proceso (valor de return)
r = yield a | b  ·  yield a & b  ·  env.any_of([...])  ·  env.all_of([...])
ev = env.event(); ev.succeed(valor); ev.fail(excepcion); valor = yield ev

simpy.Resource(env, capacity=n) · PriorityResource · PreemptiveResource
with recurso.request(priority=p) as peticion: yield peticion
recurso.count · recurso.capacity · len(recurso.queue)

simpy.Container(env, capacity=c, init=i); yield cont.get(x); yield cont.put(x); cont.level
simpy.Store(env, capacity=c); yield store.put(obj); obj = yield store.get(); store.items
simpy.FilterStore(env); yield fs.get(lambda o: condicion)
simpy.PriorityStore(env); yield ps.put(simpy.PriorityItem(prioridad, obj))

proceso.interrupt(causa) · except simpy.Interrupt as i: i.cause
simpy.rt.RealtimeEnvironment(factor=1.0, strict=True)
```

---

## 20. Ejercicios propuestos

1. Simule una oficina bancaria con 3 cajeros y llegadas exponenciales de media 2 minutos. Compare la espera media con el resultado teórico del modelo de colas M/M/3.
2. Modele una gasolinera con 4 surtidores y un depósito de combustible (`Container`) que se repone cuando baja del 20 %. Calcule cuántos clientes se quedan sin servicio por falta de combustible.
3. Simule una línea de producción de tres máquinas en serie con almacenes intermedios de capacidad limitada (`Store`) y averías aleatorias (interrupciones). Identifique el cuello de botella.
4. Modele las urgencias de un hospital con tres niveles de gravedad usando `PreemptiveResource` y analice el tiempo de espera por nivel.
5. Amplíe el ejemplo completo con turnos de agentes que cambian a lo largo del día y busque la combinación de turnos de menor coste que cumple el nivel de servicio.

---

## 21. Referencias

- Documentación oficial: https://simpy.readthedocs.io/
- Guía de conceptos (*SimPy in 10 minutes*): https://simpy.readthedocs.io/en/latest/simpy_intro/index.html
- Ejemplos oficiales: https://simpy.readthedocs.io/en/latest/examples/index.html
- Referencia de la API: https://simpy.readthedocs.io/en/latest/api_reference/index.html
- Repositorio: https://gitlab.com/team-simpy/simpy
