# Manual de uso: Redis-py

> **Categoría:** Bases de datos · **Utilidad:** Trabajar con Redis
> **Tipo:** Librería externa (cliente oficial de Redis para Python) · Paquete `redis`

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Conectar](#4-conectar)
5. [Cadenas y contadores](#5-cadenas-y-contadores)
6. [Caducidad de claves (TTL)](#6-caducidad-de-claves-ttl)
7. [Hashes](#7-hashes)
8. [Listas y colas](#8-listas-y-colas)
9. [Conjuntos](#9-conjuntos)
10. [Conjuntos ordenados (rankings)](#10-conjuntos-ordenados-rankings)
11. [Gestión de claves](#11-gestión-de-claves)
12. [Pipelines y transacciones](#12-pipelines-y-transacciones)
13. [Publicación/suscripción y Streams](#13-publicaciónsuscripción-y-streams)
14. [Patrones habituales](#14-patrones-habituales)
15. [Funcionalidades avanzadas](#15-funcionalidades-avanzadas)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Redis** es una base de datos **en memoria** de tipo clave-valor, extremadamente rápida (cientos de miles de operaciones por segundo). Además de cadenas, ofrece estructuras de datos como listas, conjuntos, hashes, conjuntos ordenados y streams.

**redis-py** (paquete `redis`) es el cliente oficial para Python.

**Casos de uso habituales:**

| Uso | Estructura |
|---|---|
| Caché de consultas, páginas o respuestas de API | Cadenas con caducidad |
| Sesiones de usuario | Hashes con caducidad |
| Contadores (visitas, *likes*, intentos de acceso) | `INCR` |
| Limitación de peticiones (*rate limiting*) | Contadores con caducidad |
| Colas de tareas | Listas / Streams |
| Rankings y clasificaciones | Conjuntos ordenados |
| Etiquetas, seguidores, elementos únicos | Conjuntos |
| Mensajería en tiempo real | Pub/Sub |
| Bloqueos distribuidos | `SET NX` con caducidad |

> Redis guarda los datos en memoria; puede persistirlos en disco (RDB/AOF), pero normalmente se usa **junto a** una base de datos principal (PostgreSQL, MySQL, MongoDB), no en su lugar.

---

## 2. Instalación e importación

```bash
pip install redis
pip install "redis[hiredis]"         # Analizador en C: mayor rendimiento
```

```python
import redis
```

Para practicar:

```bash
docker run --name redis -p 6379:6379 -d redis:7
```

Herramientas: `redis-cli` (consola), **RedisInsight** (interfaz gráfica).

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Clave | Texto que identifica un valor (`usuario:1001:perfil`). |
| Valor | Cadena, lista, hash, conjunto, conjunto ordenado, stream… |
| TTL | Tiempo de vida de una clave; al expirar se elimina sola. |
| Base de datos | Redis tiene 16 bases numeradas (0–15) por defecto. |
| Atomicidad | Cada comando es atómico. |
| *Pipeline* | Envía varios comandos de una vez (menos latencia). |
| `decode_responses` | Devuelve `str` en lugar de `bytes`. |

**Convención de nombres de claves:** `objeto:id:campo`, separadas por dos puntos (`producto:TEC-01`, `carrito:ana`, `cache:api:clientes:p1`).

---

## 4. Conectar

### 4.1 Conexión básica

```python
r = redis.Redis(host="localhost", port=6379, db=0, decode_responses=True)
r.ping()                # True
```

> Sin `decode_responses=True`, Redis devuelve `bytes` (`b"valor"`). Actívelo salvo que almacene datos binarios.

### 4.2 Desde URL y variables de entorno

```python
import os
r = redis.Redis.from_url(os.getenv("REDIS_URL", "redis://localhost:6379/0"), decode_responses=True)

# Con contraseña y TLS:
# rediss://:clave@servidor.ejemplo.com:6380/0
# redis://usuario:clave@host:6379/0      (ACL, Redis 6+)
```

### 4.3 Opciones útiles

```python
r = redis.Redis(
    host="localhost", port=6379, db=0, password=os.getenv("REDIS_PASSWORD"),
    decode_responses=True,
    socket_timeout=5,               # Tiempo máximo por operación
    socket_connect_timeout=5,
    health_check_interval=30,       # Comprueba conexiones inactivas
    retry_on_timeout=True,
)
```

### 4.4 Pool de conexiones

`redis.Redis` ya usa un pool internamente. Para compartirlo explícitamente:

```python
pool = redis.ConnectionPool.from_url("redis://localhost:6379/0", decode_responses=True, max_connections=20)
r = redis.Redis(connection_pool=pool)
```

> Cree **un único** cliente por aplicación y reutilícelo; es seguro entre hilos.

### 4.5 Información del servidor

```python
info = r.info()
info["redis_version"], info["used_memory_human"], info["connected_clients"]
r.dbsize()                      # Número de claves
```

---

## 5. Cadenas y contadores

### 5.1 Guardar y leer

```python
r.set("empresa:nombre", "Mi Empresa, S.L.")
r.get("empresa:nombre")                   # 'Mi Empresa, S.L.'
r.get("no_existe")                        # None

r.mset({"config:idioma": "es", "config:moneda": "EUR"})
r.mget("config:idioma", "config:moneda")  # ['es', 'EUR']

r.getdel("token:temporal")                # Leer y eliminar
r.append("log:hoy", "nueva línea\n")
r.strlen("empresa:nombre")
```

### 5.2 Opciones de `set`

```python
r.set("clave", "valor", ex=60)            # Caduca en 60 segundos
r.set("clave", "valor", px=500)           # Caduca en 500 milisegundos
r.set("clave", "valor", nx=True)          # Solo si NO existe (devuelve None si ya existía)
r.set("clave", "valor", xx=True)          # Solo si YA existe
r.set("clave", "valor", keepttl=True)     # Conserva la caducidad actual
r.set("clave", "nuevo", get=True)         # Devuelve el valor anterior
```

### 5.3 Contadores atómicos

```python
r.incr("visitas:home")                    # +1 → devuelve el nuevo valor
r.incrby("visitas:home", 10)
r.decr("stock:TEC-01")
r.decrby("stock:TEC-01", 3)
r.incrbyfloat("facturacion:hoy", 59.90)
```

> Los valores numéricos se guardan como texto; `incr` funciona de forma atómica aunque muchos procesos lo llamen a la vez.

### 5.4 Guardar objetos (JSON)

```python
import json

r.set("producto:TEC-01", json.dumps({"nombre": "Teclado", "precio": 59.9}, ensure_ascii=False), ex=3600)
producto = json.loads(r.get("producto:TEC-01") or "null")
```

---

## 6. Caducidad de claves (TTL)

```python
r.expire("sesion:abc", 1800)              # Caduca en 30 minutos
r.pexpire("sesion:abc", 1_800_000)        # En milisegundos
r.expireat("oferta:navidad", 1767225599)  # En un instante (timestamp)
r.ttl("sesion:abc")                       # Segundos restantes (-1 sin caducidad, -2 no existe)
r.persist("sesion:abc")                   # Quitar la caducidad

from datetime import datetime, timedelta
r.set("codigo:verificacion:ana", "482913", ex=timedelta(minutes=10))
r.expireat("informe:diario", datetime(2026, 11, 25, 0, 0))
```

> Cualquier clave de caché, sesión o dato temporal debe tener caducidad para evitar que la memoria crezca sin límite.

---

## 7. Hashes

Un hash almacena varios campos dentro de una clave: ideal para representar objetos.

```python
r.hset("usuario:1001", mapping={
    "nombre": "Ana López",
    "email": "ana@ejemplo.com",
    "ciudad": "Madrid",
    "puntos": 120,
})

r.hget("usuario:1001", "email")              # 'ana@ejemplo.com'
r.hmget("usuario:1001", ["nombre", "ciudad"])
r.hgetall("usuario:1001")                    # dict completo
r.hset("usuario:1001", "ciudad", "Barcelona")
r.hincrby("usuario:1001", "puntos", 50)
r.hincrbyfloat("usuario:1001", "saldo", 12.5)
r.hexists("usuario:1001", "telefono")
r.hdel("usuario:1001", "ciudad")
r.hkeys("usuario:1001"); r.hvals("usuario:1001"); r.hlen("usuario:1001")
r.hsetnx("usuario:1001", "alta", "2026-11-24")   # Solo si el campo no existe
```

> Todos los valores se devuelven como texto: convierta los números (`int(r.hget(..., "puntos"))`).

Carrito de la compra:

```python
clave = "carrito:ana"
r.hincrby(clave, "TEC-01", 1)
r.hincrby(clave, "RAT-01", 2)
r.expire(clave, timedelta(days=7))
carrito = {codigo: int(uds) for codigo, uds in r.hgetall(clave).items()}
```

---

## 8. Listas y colas

Listas ordenadas por inserción; permiten añadir y extraer por ambos extremos.

```python
r.rpush("cola:emails", "email1", "email2")    # Añadir al final
r.lpush("cola:emails", "urgente")             # Añadir al principio
r.lpop("cola:emails")                         # Extraer del principio
r.rpop("cola:emails")
r.lrange("cola:emails", 0, -1)                # Todos los elementos
r.lrange("ultimos:accesos", 0, 9)             # Los 10 primeros
r.llen("cola:emails")
r.lindex("cola:emails", 0)
r.lrem("cola:emails", 0, "email2")            # Eliminar todas las apariciones
r.ltrim("ultimos:accesos", 0, 99)             # Conservar solo los 100 primeros
```

### 8.1 Cola de tareas con espera bloqueante

```python
# Productor
r.rpush("cola:tareas", json.dumps({"tipo": "enviar_email", "destino": "ana@ejemplo.com"}))

# Trabajador: espera hasta 5 s a que llegue una tarea
while True:
    elemento = r.blpop("cola:tareas", timeout=5)     # (clave, valor) o None
    if elemento is None:
        continue
    _, datos = elemento
    procesar(json.loads(datos))
```

### 8.2 Cola fiable (no perder tareas si el trabajador falla)

```python
tarea = r.blmove("cola:tareas", "cola:en_proceso", timeout=5, src="LEFT", dest="RIGHT")
if tarea:
    try:
        procesar(json.loads(tarea))
        r.lrem("cola:en_proceso", 1, tarea)
    except Exception:
        r.lmove("cola:en_proceso", "cola:fallidas", "LEFT", "RIGHT")
```

### 8.3 Historial limitado

```python
def registrar_acceso(usuario: str, pagina: str) -> None:
    clave = f"historial:{usuario}"
    r.lpush(clave, pagina)
    r.ltrim(clave, 0, 19)              # Solo las 20 últimas
```

---

## 9. Conjuntos

Colecciones sin orden y **sin duplicados**.

```python
r.sadd("producto:TEC-01:etiquetas", "gaming", "oferta", "mecánico")
r.smembers("producto:TEC-01:etiquetas")       # set de valores
r.sismember("producto:TEC-01:etiquetas", "oferta")
r.smismember("producto:TEC-01:etiquetas", ["oferta", "nuevo"])   # [1, 0]
r.srem("producto:TEC-01:etiquetas", "oferta")
r.scard("producto:TEC-01:etiquetas")          # Número de elementos
r.srandmember("sorteo:participantes", 3)      # 3 al azar (sin extraer)
r.spop("sorteo:participantes")                # Extrae uno al azar

# Operaciones de conjuntos
r.sinter("usuario:ana:intereses", "usuario:luis:intereses")       # Comunes
r.sunion("etiquetas:a", "etiquetas:b")
r.sdiff("clientes:2025", "clientes:2026")                         # Clientes perdidos
r.sinterstore("comunes:ana:luis", ["usuario:ana:intereses", "usuario:luis:intereses"])
```

Visitantes únicos del día:

```python
from datetime import date
r.sadd(f"visitantes:{date.today()}", ip)
r.expire(f"visitantes:{date.today()}", timedelta(days=30))
unicos = r.scard(f"visitantes:{date.today()}")
```

> Para contar elementos únicos en grandes volúmenes con muy poca memoria (con un error de ~0,8 %), use **HyperLogLog**: `r.pfadd(clave, valor)` y `r.pfcount(clave)`.

---

## 10. Conjuntos ordenados (rankings)

Cada elemento tiene una **puntuación**; se mantienen ordenados automáticamente.

```python
r.zadd("ranking:ventas", {"Ana": 15200, "Luis": 9800, "Marta": 12750})
r.zincrby("ranking:ventas", 500, "Luis")                  # Suma puntos

r.zrange("ranking:ventas", 0, 2, desc=True, withscores=True)
# [('Ana', 15200.0), ('Marta', 12750.0), ('Luis', 10300.0)]

r.zrevrank("ranking:ventas", "Luis")                      # Posición (0 = primero)
r.zscore("ranking:ventas", "Marta")
r.zcard("ranking:ventas")
r.zcount("ranking:ventas", 10000, "+inf")                 # Cuántos superan 10 000
r.zrangebyscore("ranking:ventas", 10000, 20000, withscores=True)
r.zrem("ranking:ventas", "Luis")
r.zpopmax("ranking:ventas")                               # Extraer el mayor
r.zremrangebyrank("ranking:ventas", 0, -11)               # Conservar solo el top 10
```

Opciones de `zadd`:

```python
r.zadd("ranking", {"Ana": 100}, nx=True)        # Solo si no existe
r.zadd("ranking", {"Ana": 120}, gt=True)        # Solo si la nueva puntuación es mayor
```

Eventos ordenados por fecha (puntuación = timestamp):

```python
import time
r.zadd("pedidos:recientes", {"PED-1045": time.time()})
hace_una_hora = time.time() - 3600
r.zrangebyscore("pedidos:recientes", hace_una_hora, "+inf")
r.zremrangebyscore("pedidos:recientes", "-inf", time.time() - 86400)   # Limpiar los de hace más de 1 día
```

---

## 11. Gestión de claves

```python
r.exists("usuario:1001")                    # Número de claves existentes (0 o 1)
r.delete("clave1", "clave2")                # Elimina (devuelve cuántas)
r.unlink("clave_grande")                    # Elimina en segundo plano (no bloquea)
r.type("usuario:1001")                      # 'hash'
r.rename("antigua", "nueva")
r.copy("origen", "destino")
r.randomkey()
```

### 11.1 Buscar claves por patrón

```python
for clave in r.scan_iter(match="cache:api:*", count=500):
    r.unlink(clave)
```

> **No use `r.keys("*")` en producción**: recorre todas las claves de una vez y bloquea el servidor. Use `scan_iter`, que recorre por partes.

### 11.2 Vaciar

```python
r.flushdb()          # Vacía la base de datos actual
r.flushall()         # Vacía TODAS las bases de datos (¡precaución!)
```

---

## 12. Pipelines y transacciones

### 12.1 Pipeline: varios comandos en un solo viaje

Cada comando implica un viaje de ida y vuelta por la red. Un *pipeline* los agrupa:

```python
with r.pipeline() as pipe:
    for i in range(1000):
        pipe.set(f"producto:{i}:stock", 100)
    pipe.execute()                           # Envía todo de una vez
```

```python
with r.pipeline() as pipe:
    pipe.incr("visitas:home")
    pipe.expire("visitas:home", 86400)
    pipe.hgetall("usuario:1001")
    visitas, _, usuario = pipe.execute()     # Resultados en orden
```

> Por defecto, `r.pipeline()` es **transaccional** (`MULTI/EXEC`): los comandos se ejecutan de forma atómica, sin intercalarse con los de otros clientes. Con `r.pipeline(transaction=False)` solo se agrupan para ganar velocidad.

### 12.2 Transacciones optimistas con `WATCH`

Para operaciones «leer → decidir → escribir» que deben ser atómicas:

```python
def comprar(r, producto: str, cantidad: int) -> bool:
    clave = f"stock:{producto}"
    with r.pipeline() as pipe:
        while True:
            try:
                pipe.watch(clave)                       # Vigilar la clave
                stock = int(pipe.get(clave) or 0)
                if stock < cantidad:
                    pipe.unwatch()
                    return False
                pipe.multi()                            # Inicio de la transacción
                pipe.decrby(clave, cantidad)
                pipe.execute()                          # Falla si otro cliente cambió la clave
                return True
            except redis.WatchError:
                continue                                # Reintentar
```

Atajo equivalente:

```python
def operacion(pipe):
    stock = int(pipe.get("stock:TEC-01") or 0)
    pipe.multi()
    pipe.set("stock:TEC-01", stock - 1)

r.transaction(operacion, "stock:TEC-01")
```

### 12.3 Scripts Lua (atomicidad en el servidor)

```python
DESCONTAR = r.register_script("""
local stock = tonumber(redis.call('GET', KEYS[1]) or '0')
local cantidad = tonumber(ARGV[1])
if stock >= cantidad then
    return redis.call('DECRBY', KEYS[1], cantidad)
end
return -1
""")

resto = DESCONTAR(keys=["stock:TEC-01"], args=[2])
if resto == -1:
    print("Stock insuficiente")
```

> Los scripts Lua se ejecutan de forma atómica y en un solo viaje: suelen ser la opción más simple y eficiente para lógica condicional.

---

## 13. Publicación/suscripción y Streams

### 13.1 Pub/Sub (mensajes en tiempo real)

```python
# Publicador
r.publish("notificaciones", json.dumps({"tipo": "pedido", "id": 1045}))

# Suscriptor
p = r.pubsub(ignore_subscribe_messages=True)
p.subscribe("notificaciones")
p.psubscribe("alertas:*")                    # Por patrón
for mensaje in p.listen():
    print(mensaje["channel"], mensaje["data"])
```

En segundo plano:

```python
def manejar(mensaje):
    print("Recibido:", mensaje["data"])

p = r.pubsub()
p.subscribe(**{"notificaciones": manejar})
hilo = p.run_in_thread(sleep_time=0.1, daemon=True)
# ... hilo.stop()
```

> Pub/Sub **no almacena** los mensajes: si no hay suscriptores conectados, se pierden. Para mensajes persistentes use Streams.

### 13.2 Streams (registro persistente de eventos)

```python
# Añadir eventos
r.xadd("eventos:pedidos", {"pedido": "1045", "estado": "creado"}, maxlen=10_000, approximate=True)

# Leer
r.xrange("eventos:pedidos", "-", "+", count=10)
r.xrevrange("eventos:pedidos", "+", "-", count=5)
r.xlen("eventos:pedidos")
```

Grupos de consumidores (varios trabajadores repartiéndose los eventos):

```python
try:
    r.xgroup_create("eventos:pedidos", "facturacion", id="0", mkstream=True)
except redis.ResponseError:
    pass                                                  # El grupo ya existe

while True:
    respuesta = r.xreadgroup("facturacion", "trabajador-1", {"eventos:pedidos": ">"}, count=10, block=5000)
    for _, mensajes in respuesta or []:
        for id_mensaje, datos in mensajes:
            facturar(datos)
            r.xack("eventos:pedidos", "facturacion", id_mensaje)    # Confirmar
```

---

## 14. Patrones habituales

### 14.1 Caché de resultados (*cache-aside*)

```python
import functools
import hashlib


def cache_redis(prefijo: str, ttl: int = 300):
    def decorador(funcion):
        @functools.wraps(funcion)
        def envoltorio(*args, **kwargs):
            firma = hashlib.sha1(json.dumps([args, kwargs], sort_keys=True, default=str).encode()).hexdigest()
            clave = f"cache:{prefijo}:{firma}"
            if (guardado := r.get(clave)) is not None:
                return json.loads(guardado)
            resultado = funcion(*args, **kwargs)
            r.set(clave, json.dumps(resultado, default=str, ensure_ascii=False), ex=ttl)
            return resultado
        envoltorio.invalidar = lambda: [r.unlink(k) for k in r.scan_iter(f"cache:{prefijo}:*")]
        return envoltorio
    return decorador


@cache_redis("ventas_mes", ttl=600)
def ventas_del_mes(anio: int, mes: int) -> dict:
    return consulta_lenta_a_la_bd(anio, mes)
```

### 14.2 Limitación de peticiones (*rate limiting*)

Ventana fija:

```python
def permitido(usuario: str, limite: int = 100, ventana: int = 60) -> bool:
    clave = f"limite:{usuario}:{int(time.time() // ventana)}"
    with r.pipeline() as pipe:
        pipe.incr(clave)
        pipe.expire(clave, ventana)
        peticiones, _ = pipe.execute()
    return peticiones <= limite
```

Ventana deslizante (más precisa):

```python
def permitido_deslizante(usuario: str, limite: int = 100, ventana: int = 60) -> bool:
    ahora = time.time()
    clave = f"limite:{usuario}"
    with r.pipeline() as pipe:
        pipe.zremrangebyscore(clave, 0, ahora - ventana)
        pipe.zadd(clave, {f"{ahora}": ahora})
        pipe.zcard(clave)
        pipe.expire(clave, ventana)
        _, _, total, _ = pipe.execute()
    return total <= limite
```

### 14.3 Bloqueo distribuido

Evita que dos procesos (en distintos servidores) ejecuten la misma tarea a la vez:

```python
bloqueo = r.lock("bloqueo:cierre_diario", timeout=300, blocking_timeout=5)
if bloqueo.acquire():
    try:
        ejecutar_cierre_diario()
    finally:
        bloqueo.release()
else:
    print("Otro proceso está ejecutando el cierre")

# O con gestor de contexto
with r.lock("bloqueo:importacion", timeout=600):
    importar_datos()
```

### 14.4 Sesiones

```python
import secrets

def crear_sesion(usuario_id: int, datos: dict, duracion: int = 1800) -> str:
    token = secrets.token_urlsafe(32)
    clave = f"sesion:{token}"
    r.hset(clave, mapping={"usuario_id": usuario_id, **datos})
    r.expire(clave, duracion)
    return token

def obtener_sesion(token: str, renovar: int = 1800) -> dict | None:
    clave = f"sesion:{token}"
    datos = r.hgetall(clave)
    if datos:
        r.expire(clave, renovar)          # Caducidad deslizante
    return datos or None
```

### 14.5 Códigos de un solo uso (OTP)

```python
def generar_codigo(email: str) -> str:
    codigo = f"{secrets.randbelow(1_000_000):06d}"
    r.set(f"otp:{email}", codigo, ex=600)
    r.delete(f"otp_intentos:{email}")
    return codigo

def verificar_codigo(email: str, codigo: str) -> bool:
    intentos = r.incr(f"otp_intentos:{email}")
    r.expire(f"otp_intentos:{email}", 600)
    if intentos > 5:
        return False
    guardado = r.get(f"otp:{email}")
    if guardado and secrets.compare_digest(guardado, codigo):
        r.delete(f"otp:{email}", f"otp_intentos:{email}")
        return True
    return False
```

---

## 15. Funcionalidades avanzadas

### 15.1 Cliente asíncrono

```python
import asyncio
import redis.asyncio as aioredis

async def main():
    r = aioredis.from_url("redis://localhost:6379", decode_responses=True)
    await r.set("saludo", "hola", ex=60)
    print(await r.get("saludo"))
    async with r.pipeline() as pipe:
        await pipe.incr("contador").expire("contador", 60).execute()
    await r.aclose()

asyncio.run(main())
```

### 15.2 Datos binarios y serialización

```python
r_bin = redis.Redis(decode_responses=False)
r_bin.set("imagen:logo", open("logo.png", "rb").read(), ex=3600)
```

Para objetos Python complejos puede usar `pickle`, pero **solo con datos de confianza** (cargar datos `pickle` manipulados permite ejecutar código). JSON es la opción recomendada.

### 15.3 Módulos de Redis Stack / Redis 8

Redis Stack (y Redis 8) añade JSON nativo, búsqueda e índices secundarios, series temporales y filtros de Bloom:

```python
r.json().set("producto:TEC-01", "$", {"nombre": "Teclado", "precio": 59.9, "etiquetas": ["gaming"]})
r.json().get("producto:TEC-01", "$.precio")
```

### 15.4 Persistencia y memoria

| Opción | Descripción |
|---|---|
| RDB | Instantáneas periódicas del contenido. |
| AOF | Registro de cada escritura (más duradero). |
| `maxmemory` | Límite de memoria. |
| `maxmemory-policy` | Qué eliminar al llegar al límite (`allkeys-lru` para cachés). |

```python
r.config_get("maxmemory*")
r.memory_usage("usuario:1001")          # Bytes que ocupa una clave
```

### 15.5 Pruebas sin servidor

**fakeredis** imita Redis en memoria para pruebas unitarias:

```python
import fakeredis
r = fakeredis.FakeRedis(decode_responses=True)
```

---

## 16. Ejemplo práctico completo

Servicio de catálogo con: caché de consultas a una base de datos lenta (SQLite simulada), contador de visitas, ranking de productos más vistos, carrito con caducidad, control de stock atómico con script Lua, limitación de peticiones, cola de pedidos procesada por un trabajador y bloqueo distribuido para el cierre diario.

```python
"""
Servicio de catálogo con Redis.
Requisitos: pip install redis
Variable de entorno: REDIS_URL (por defecto redis://localhost:6379/0)
"""
import json
import os
import sqlite3
import threading
import time
from datetime import date, timedelta

import redis

r = redis.Redis.from_url(os.getenv("REDIS_URL", "redis://localhost:6379/0"), decode_responses=True)
PREFIJO = "demo"                         # Espacio de nombres para no mezclar con otras claves


def k(*partes) -> str:
    return ":".join([PREFIJO, *map(str, partes)])


# ------------------------------------------------------------ "Base de datos" lenta
BD = sqlite3.connect(":memory:", check_same_thread=False)
BD.executescript("""
CREATE TABLE productos (codigo TEXT PRIMARY KEY, nombre TEXT, categoria TEXT, precio REAL);
INSERT INTO productos VALUES
 ('TEC-01','Teclado mecánico','Periféricos',59.90), ('RAT-01','Ratón inalámbrico','Periféricos',24.50),
 ('MON-27','Monitor 27"','Pantallas',219.00), ('POR-14','Portátil 14"','Equipos',899.00),
 ('AUR-BT','Auriculares BT','Audio',79.00), ('CAB-HD','Cable HDMI 2 m','Accesorios',9.90);
""")
BD_LOCK = threading.Lock()


def consulta_lenta(codigo: str) -> dict | None:
    time.sleep(0.2)                                         # Simula una consulta costosa
    with BD_LOCK:
        fila = BD.execute("SELECT codigo, nombre, categoria, precio FROM productos WHERE codigo = ?",
                          (codigo,)).fetchone()
    return dict(zip(["codigo", "nombre", "categoria", "precio"], fila)) if fila else None


# ------------------------------------------------------------ Caché
def obtener_producto(codigo: str) -> dict | None:
    clave = k("cache", "producto", codigo)
    if (guardado := r.get(clave)) is not None:
        return json.loads(guardado)
    producto = consulta_lenta(codigo)
    r.set(clave, json.dumps(producto, ensure_ascii=False), ex=300)    # También se cachea "no encontrado"
    return producto


def invalidar_producto(codigo: str) -> None:
    r.unlink(k("cache", "producto", codigo))


# ------------------------------------------------------------ Visitas y ranking
def registrar_visita(codigo: str, usuario: str) -> None:
    hoy = date.today().isoformat()
    with r.pipeline(transaction=False) as pipe:
        pipe.incr(k("visitas", codigo))
        pipe.zincrby(k("ranking", "vistos", hoy), 1, codigo)
        pipe.expire(k("ranking", "vistos", hoy), timedelta(days=8))
        pipe.pfadd(k("unicos", hoy), usuario)
        pipe.expire(k("unicos", hoy), timedelta(days=8))
        pipe.lpush(k("historial", usuario), codigo)
        pipe.ltrim(k("historial", usuario), 0, 9)
        pipe.execute()


def mas_vistos(n: int = 3) -> list[tuple[str, int]]:
    clave = k("ranking", "vistos", date.today().isoformat())
    return [(c, int(p)) for c, p in r.zrange(clave, 0, n - 1, desc=True, withscores=True)]


# ------------------------------------------------------------ Limitación de peticiones
def permitido(usuario: str, limite: int = 20, ventana: int = 10) -> bool:
    clave = k("limite", usuario, int(time.time() // ventana))
    with r.pipeline() as pipe:
        pipe.incr(clave)
        pipe.expire(clave, ventana)
        total, _ = pipe.execute()
    return total <= limite


# ------------------------------------------------------------ Stock atómico (Lua)
DESCONTAR_VARIOS = r.register_script("""
for i, clave in ipairs(KEYS) do
    if tonumber(redis.call('GET', clave) or '0') < tonumber(ARGV[i]) then
        return 0
    end
end
for i, clave in ipairs(KEYS) do
    redis.call('DECRBY', clave, ARGV[i])
end
return 1
""")


def reservar_stock(articulos: dict[str, int]) -> bool:
    claves = [k("stock", c) for c in articulos]
    return DESCONTAR_VARIOS(keys=claves, args=list(articulos.values())) == 1


# ------------------------------------------------------------ Carrito
def anadir_carrito(usuario: str, codigo: str, unidades: int = 1) -> None:
    clave = k("carrito", usuario)
    r.hincrby(clave, codigo, unidades)
    r.expire(clave, timedelta(days=7))


def confirmar_carrito(usuario: str) -> int | None:
    clave = k("carrito", usuario)
    articulos = {c: int(u) for c, u in r.hgetall(clave).items()}
    if not articulos or not reservar_stock(articulos):
        return None
    pedido_id = r.incr(k("secuencia", "pedidos"))
    r.rpush(k("cola", "pedidos"), json.dumps({"id": pedido_id, "usuario": usuario, "articulos": articulos}))
    r.delete(clave)
    return pedido_id


# ------------------------------------------------------------ Trabajador de la cola
def trabajador(parar: threading.Event, procesados: list) -> None:
    while not parar.is_set():
        elemento = r.blmove(k("cola", "pedidos"), k("cola", "en_proceso"), timeout=1, src="LEFT", dest="RIGHT")
        if elemento is None:
            continue
        pedido = json.loads(elemento)
        total = sum(obtener_producto(c)["precio"] * u for c, u in pedido["articulos"].items())
        r.hset(k("pedido", pedido["id"]), mapping={"usuario": pedido["usuario"], "total": round(total, 2),
                                                   "estado": "procesado"})
        r.incrbyfloat(k("facturacion", date.today().isoformat()), total)
        r.lrem(k("cola", "en_proceso"), 1, elemento)
        procesados.append(pedido["id"])


# ------------------------------------------------------------ Cierre diario con bloqueo
def cierre_diario() -> str:
    try:
        with r.lock(k("bloqueo", "cierre"), timeout=60, blocking_timeout=0.1):
            time.sleep(0.5)
            return "cierre ejecutado"
    except redis.exceptions.LockError:
        return "otro proceso ya está ejecutando el cierre"


# ------------------------------------------------------------ Demostración
def main() -> None:
    r.ping()
    for clave in r.scan_iter(f"{PREFIJO}:*"):
        r.unlink(clave)
    with r.pipeline() as pipe:
        for codigo, stock in {"TEC-01": 5, "RAT-01": 10, "MON-27": 2, "POR-14": 1, "AUR-BT": 4, "CAB-HD": 50}.items():
            pipe.set(k("stock", codigo), stock)
        pipe.execute()

    # 1. Caché
    t = time.perf_counter(); obtener_producto("MON-27"); t1 = time.perf_counter() - t
    t = time.perf_counter(); obtener_producto("MON-27"); t2 = time.perf_counter() - t
    print(f"Caché: 1.ª consulta {t1 * 1000:.0f} ms · 2.ª consulta {t2 * 1000:.1f} ms (TTL {r.ttl(k('cache', 'producto', 'MON-27'))} s)")

    # 2. Visitas y ranking
    import random
    random.seed(2)
    codigos = ["TEC-01", "RAT-01", "MON-27", "POR-14", "AUR-BT", "CAB-HD"]
    for _ in range(300):
        registrar_visita(random.choices(codigos, weights=[5, 3, 4, 2, 3, 1])[0], f"u{random.randint(1, 80)}")
    print("Más vistos hoy:", mas_vistos())
    print("Visitantes únicos (HyperLogLog):", r.pfcount(k("unicos", date.today().isoformat())))

    # 3. Limitación de peticiones
    aceptadas = sum(permitido("bot") for _ in range(30))
    print(f"Limitación: {aceptadas} de 30 peticiones aceptadas en la ventana")

    # 4. Carritos, stock y cola de pedidos
    parar, procesados = threading.Event(), []
    hilo = threading.Thread(target=trabajador, args=(parar, procesados), daemon=True)
    hilo.start()

    resultados = []
    for usuario, articulos in [("ana", {"TEC-01": 2, "RAT-01": 1}), ("luis", {"MON-27": 2}),
                               ("marta", {"MON-27": 1}), ("pedro", {"POR-14": 1, "CAB-HD": 3})]:
        for codigo, uds in articulos.items():
            anadir_carrito(usuario, codigo, uds)
        resultados.append((usuario, confirmar_carrito(usuario)))
    for usuario, pedido_id in resultados:
        print(f"  {usuario:<6} → {'pedido ' + str(pedido_id) if pedido_id else 'rechazado (sin stock)'}")

    time.sleep(1.5)
    parar.set()
    hilo.join()
    print("Pedidos procesados por el trabajador:", procesados)
    print("Stock restante:", {c: int(r.get(k("stock", c))) for c in codigos})
    print("Facturación de hoy:", r.get(k("facturacion", date.today().isoformat())), "€")

    # 5. Bloqueo distribuido: dos procesos intentan el cierre a la vez
    salida = []
    hilos = [threading.Thread(target=lambda: salida.append(cierre_diario())) for _ in range(2)]
    for h in hilos: h.start()
    for h in hilos: h.join()
    print("Cierre diario:", salida)


if __name__ == "__main__":
    main()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ConnectionError: Error 111 connecting to localhost:6379. Connection refused` | Redis no está en ejecución o host/puerto incorrectos. | Inicie Redis y revise la configuración. |
| `AuthenticationError` / `NOAUTH Authentication required` | Falta la contraseña o es incorrecta. | Indique `password` o use la URL con credenciales. |
| Valores como `b'texto'` | `decode_responses=False`. | Active `decode_responses=True`. |
| `ResponseError: WRONGTYPE Operation against a key holding the wrong kind of value` | Comando de un tipo sobre una clave de otro tipo (p. ej. `hget` sobre una cadena). | Revise los nombres de claves; use prefijos por tipo. |
| `DataError: Invalid input of type: 'dict'` | Se intentó guardar un diccionario con `set`. | Serialice con `json.dumps` o use un hash. |
| `ResponseError: value is not an integer` | `incr` sobre un valor no numérico. | Compruebe el contenido de la clave. |
| La memoria crece sin parar | Claves sin caducidad. | Asigne TTL a cachés, sesiones y temporales; configure `maxmemory-policy`. |
| El servidor se bloquea brevemente | `KEYS *`, borrado de claves enormes o `FLUSHALL`. | Use `scan_iter`, `unlink` y evite operaciones masivas en horas punta. |
| Se pierden mensajes de Pub/Sub | No había suscriptores conectados. | Use Streams para mensajes persistentes. |
| `WatchError` repetido | Mucha concurrencia sobre la misma clave. | Use scripts Lua o reduzca la contención. |
| `LockError` al liberar | El bloqueo caducó antes de terminar la tarea. | Aumente `timeout` o renueve el bloqueo (`extend`). |

---

## 18. Buenas prácticas

- Use **un único** cliente (con su pool) por aplicación.
- Active `decode_responses=True` salvo que trabaje con datos binarios.
- Defina una convención de nombres de claves con prefijos (`app:objeto:id:campo`).
- Asigne **siempre** caducidad a cachés, sesiones, códigos y contadores temporales.
- Use el tipo de dato adecuado: hashes para objetos, conjuntos ordenados para rankings, listas/streams para colas.
- Agrupe operaciones con *pipelines* y use scripts Lua para lógica atómica condicional.
- No use `KEYS` en producción; use `SCAN`.
- No trate Redis como única fuente de verdad para datos críticos: combínelo con una base de datos persistente.
- Proteja el servidor con contraseña/ACL y no lo exponga a Internet.
- Use JSON para serializar objetos; evite `pickle` con datos no confiables.

---

## 19. Referencia rápida

```python
r = redis.Redis.from_url("redis://localhost:6379/0", decode_responses=True); r.ping()

# Cadenas / contadores
r.set(k, v, ex=60, nx=True) / r.get(k) / r.mset({...}) / r.mget(...) / r.incr(k) / r.incrby(k, n)
# Caducidad
r.expire(k, s) / r.ttl(k) / r.persist(k)
# Hashes
r.hset(k, mapping={...}) / r.hget(k, f) / r.hgetall(k) / r.hincrby(k, f, n) / r.hdel(k, f)
# Listas
r.rpush(k, *v) / r.lpop(k) / r.blpop(k, timeout) / r.lrange(k, 0, -1) / r.ltrim(k, 0, 99) / r.blmove(...)
# Conjuntos
r.sadd(k, *v) / r.smembers(k) / r.sismember(k, v) / r.scard(k) / r.sinter(a, b)
# Conjuntos ordenados
r.zadd(k, {m: p}) / r.zincrby(k, n, m) / r.zrange(k, 0, 9, desc=True, withscores=True) / r.zrevrank(k, m)
# Claves
r.exists(k) / r.delete(k) / r.unlink(k) / r.scan_iter(match="p:*") / r.type(k)
# Pipelines, transacciones, Lua, bloqueos
with r.pipeline() as p: p.incr(a); p.expire(a, 60); p.execute()
r.register_script(lua)(keys=[...], args=[...])
with r.lock("nombre", timeout=60): ...
# Pub/Sub y Streams
r.publish(canal, msg) / r.pubsub().subscribe(canal)
r.xadd(stream, {...}) / r.xreadgroup(grupo, consumidor, {stream: ">"}) / r.xack(stream, grupo, id)
```

---

## 20. Ejercicios propuestos

1. Añada una caché Redis con caducidad de 10 minutos a una función que consulte una API externa, y mida la mejora.
2. Implemente un ranking semanal de jugadores con puntuaciones que se reinicie cada lunes.
3. Cree un limitador de intentos de inicio de sesión que bloquee 15 minutos tras 5 fallos.
4. Desarrolle una cola de tareas con varios trabajadores usando Streams y grupos de consumidores.
5. Use un bloqueo distribuido para garantizar que una tarea programada no se ejecute dos veces a la vez en distintos servidores.

---

## 21. Referencias

- Documentación de redis-py: https://redis.readthedocs.io/
- Documentación de Redis (comandos y tipos de datos): https://redis.io/docs/latest/
- Lista de comandos: https://redis.io/docs/latest/commands/
