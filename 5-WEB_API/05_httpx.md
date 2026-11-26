# Manual de uso: httpx

> **Categoría:** Desarrollo web · **Utilidad:** Peticiones HTTP síncronas y asíncronas
> **Tipo:** Librería de terceros (`pip install httpx`) · **Versión de referencia:** 0.28

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Peticiones básicas](#4-peticiones-básicas)
5. [Diferencias con Requests](#5-diferencias-con-requests)
6. [El cliente (`Client`)](#6-el-cliente-client)
7. [Tiempos de espera y límites de conexiones](#7-tiempos-de-espera-y-límites-de-conexiones)
8. [Respuestas y errores](#8-respuestas-y-errores)
9. [Cliente asíncrono (`AsyncClient`)](#9-cliente-asíncrono-asyncclient)
10. [Concurrencia controlada](#10-concurrencia-controlada)
11. [Reintentos](#11-reintentos)
12. [Autenticación](#12-autenticación)
13. [*Streaming*: descargas y respuestas largas](#13-streaming-descargas-y-respuestas-largas)
14. [Subir archivos y formularios](#14-subir-archivos-y-formularios)
15. [HTTP/2](#15-http2)
16. [*Event hooks*](#16-event-hooks)
17. [SSL, *proxies* y redirecciones](#17-ssl-proxies-y-redirecciones)
18. [Transportes: pruebas y llamadas a aplicaciones ASGI/WSGI](#18-transportes-pruebas-y-llamadas-a-aplicaciones-asgiwsgi)
19. [Línea de comandos](#19-línea-de-comandos)
20. [Ejemplo práctico completo](#20-ejemplo-práctico-completo)
21. [Errores frecuentes](#21-errores-frecuentes)
22. [Buenas prácticas](#22-buenas-prácticas)
23. [Referencia rápida](#23-referencia-rápida)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Referencias](#25-referencias)

---

## 1. Descripción

**httpx** es un cliente HTTP moderno para Python con una API muy parecida a la de **Requests**, pero que añade **soporte asíncrono** (`async`/`await`), **HTTP/2**, *timeouts* activos por defecto, tipado completo y transportes intercambiables que permiten probar o llamar directamente a aplicaciones ASGI/WSGI (FastAPI, Flask…) sin red.

**Ventajas:**
- La misma API en versión síncrona (`Client`) y asíncrona (`AsyncClient`).
- Ideal para cientos o miles de peticiones concurrentes con `asyncio`.
- *Timeout* de 5 segundos por defecto (evita bloqueos indefinidos).
- HTTP/1.1 y HTTP/2.
- Transportes simulados para pruebas (`MockTransport`) y llamadas a aplicaciones en proceso.
- Anotaciones de tipo completas.

**Limitaciones:**
- Aún en versión 0.x: puede haber cambios entre versiones.
- Los reintentos integrados solo cubren errores de conexión (los reintentos por código HTTP se implementan aparte).
- Algunos detalles difieren de Requests (redirecciones, *timeouts*, codificación), lo que exige atención al migrar.

| Si necesita… | Opción |
|---|---|
| Peticiones asíncronas, HTTP/2 o API compatible con Requests | **httpx** |
| Peticiones síncronas sencillas en scripts | Requests |
| Cliente y servidor asíncronos de bajo nivel, WebSockets cliente | aiohttp |
| Probar una API FastAPI | httpx (`TestClient` de FastAPI se basa en httpx) |

---

## 2. Instalación e importación

```bash
pip install httpx
pip install "httpx[http2]"        # Soporte HTTP/2
pip install "httpx[cli]"          # Comando 'httpx' en la terminal
pip install "httpx[brotli,zstd]"  # Descompresión adicional
```

```python
import httpx
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `httpx.get()`, `post()`… | Funciones rápidas: crean un cliente para una sola petición. |
| `httpx.Client` | Cliente síncrono con conexiones reutilizables y configuración común. |
| `httpx.AsyncClient` | Cliente asíncrono (se usa con `await`). |
| `httpx.Timeout` | Configuración detallada de tiempos de espera. |
| `httpx.Limits` | Número máximo de conexiones simultáneas y en reposo. |
| Transporte | Capa que envía la petición: red real, simulada (`MockTransport`) o una app ASGI/WSGI. |
| *Event hooks* | Funciones que se ejecutan antes de enviar y al recibir cada petición. |
| `base_url` | URL base del cliente; las rutas se resuelven respecto a ella. |

---

## 4. Peticiones básicas

```python
import httpx

r = httpx.get("https://api.github.com/repos/encode/httpx")
print(r.status_code, r.json()["stargazers_count"])

httpx.post(url, json={"nombre": "Ana"})
httpx.post(url, data={"usuario": "ana"})                       # Formulario
httpx.put(url, json={...})
httpx.patch(url, json={"activo": False})
httpx.delete(url)
httpx.get(url, params={"q": "python", "page": 2}, headers={"Accept": "application/json"})
```

---

## 5. Diferencias con Requests

| Aspecto | Requests | httpx |
|---|---|---|
| Asíncrono | No | `AsyncClient` |
| *Timeout* por defecto | Ninguno | 5 s |
| Redirecciones | Se siguen por defecto | **No** se siguen por defecto (`follow_redirects=True`) |
| Objeto sesión | `requests.Session()` | `httpx.Client()` |
| HTTP/2 | No | Sí (`http2=True`) |
| Contenido sin procesar | `data="texto"` / `data=bytes` | `content="texto"` / `content=bytes` |
| `r.ok` | Sí | `r.is_success` (y `r.is_error`, `r.is_redirect`…) |
| `raise_for_status()` | Devuelve `None` | Devuelve la propia respuesta (encadenable) |
| *Proxies* | `proxies={...}` | `proxy="http://…"` o `mounts={...}` |
| Reintentos | Adaptador urllib3 | `HTTPTransport(retries=n)` (solo conexión) |
| Excepciones | `requests.exceptions.*` | `httpx.HTTPError`, `RequestError`, `HTTPStatusError`, `TimeoutException`… |

---

## 6. El cliente (`Client`)

Igual que la `Session` de Requests, el cliente **reutiliza conexiones** y centraliza la configuración. Úselo siempre que haga más de una petición.

```python
with httpx.Client(
    base_url="https://api.ejemplo.com/v1",
    headers={"User-Agent": "MiApp/1.0", "Accept": "application/json"},
    params={"lang": "es"},
    timeout=10.0,
    follow_redirects=True,
) as cliente:
    r = cliente.get("/clientes", params={"page": 1})             # https://api.ejemplo.com/v1/clientes?lang=es&page=1
    r = cliente.post("/clientes", json={"nombre": "Ana"})
```

| Parámetro de `Client` | Uso |
|---|---|
| `base_url` | URL base para rutas relativas. |
| `headers`, `params`, `cookies` | Valores comunes (se combinan con los de cada petición). |
| `auth` | Autenticación común. |
| `timeout` | Número o `httpx.Timeout`. |
| `limits` | `httpx.Limits(max_connections=, max_keepalive_connections=)`. |
| `follow_redirects` | Seguir redirecciones. |
| `http2` | Activar HTTP/2. |
| `verify`, `cert` | SSL. |
| `proxy`, `mounts` | *Proxies* y transportes por patrón de URL. |
| `transport` | Transporte personalizado (reintentos, simulación, app ASGI/WSGI). |
| `event_hooks` | `{"request": [...], "response": [...]}`. |

> Cree el cliente **una vez** y reutilícelo (por ejemplo, durante toda la vida de la aplicación). Crear un cliente por petición anula la ventaja del *pool* de conexiones.

---

## 7. Tiempos de espera y límites de conexiones

```python
httpx.get(url, timeout=10.0)                      # 10 s para todo
httpx.get(url, timeout=None)                      # Sin límite (desaconsejado)

tiempos = httpx.Timeout(10.0, connect=3.0, read=20.0, write=10.0, pool=5.0)
cliente = httpx.Client(timeout=tiempos)
cliente.get(url, timeout=httpx.Timeout(60.0))     # Sobrescribir en una petición
```

| Tipo | Significado |
|---|---|
| `connect` | Establecer la conexión. |
| `read` | Esperar datos (entre fragmentos recibidos). |
| `write` | Enviar datos. |
| `pool` | Esperar una conexión libre del *pool*. |

```python
limites = httpx.Limits(max_connections=50, max_keepalive_connections=20, keepalive_expiry=30)
cliente = httpx.AsyncClient(limits=limites)
```

> Si lanza más peticiones simultáneas que `max_connections`, el resto espera; si esperan más de `pool` segundos, se produce `PoolTimeout`.

---

## 8. Respuestas y errores

### 8.1 La respuesta

| Atributo / método | Contenido |
|---|---|
| `status_code`, `reason_phrase` | Código y texto. |
| `is_success`, `is_redirect`, `is_client_error`, `is_server_error`, `is_error` | Comprobaciones rápidas. |
| `headers`, `cookies` | Cabeceras y cookies. |
| `text`, `content`, `json()` | Contenido como texto, bytes o JSON. |
| `encoding` | Codificación (modificable). |
| `url`, `history` | URL final e historial de redirecciones. |
| `elapsed` | Tiempo transcurrido. |
| `http_version` | `"HTTP/1.1"` o `"HTTP/2"`. |
| `request` | Petición enviada. |
| `raise_for_status()` | Lanza `HTTPStatusError` (4xx/5xx) o devuelve la respuesta. |

```python
datos = cliente.get("/usuarios/1").raise_for_status().json()       # Encadenado
```

### 8.2 Jerarquía de excepciones

```
httpx.HTTPError
├── httpx.RequestError                 # Problemas al enviar o recibir
│   ├── httpx.TransportError
│   │   ├── httpx.TimeoutException     # ConnectTimeout, ReadTimeout, WriteTimeout, PoolTimeout
│   │   ├── httpx.NetworkError         # ConnectError, ReadError, WriteError, CloseError
│   │   ├── httpx.ProtocolError
│   │   └── httpx.ProxyError
│   ├── httpx.DecodingError
│   └── httpx.TooManyRedirects
└── httpx.HTTPStatusError              # Lanzada por raise_for_status()
httpx.InvalidURL · httpx.StreamError
```

```python
try:
    r = cliente.get("/pedidos/42").raise_for_status()
except httpx.HTTPStatusError as e:
    print(f"Error {e.response.status_code} en {e.request.url}: {e.response.text[:200]}")
except httpx.TimeoutException:
    print("El servidor tardó demasiado")
except httpx.RequestError as e:
    print(f"Error de red al acceder a {e.request.url}: {e!r}")
```

---

## 9. Cliente asíncrono (`AsyncClient`)

```python
import asyncio
import httpx

async def main():
    async with httpx.AsyncClient(base_url="https://api.ejemplo.com", timeout=10) as cliente:
        r = await cliente.get("/clientes")
        r.raise_for_status()
        print(r.json())

asyncio.run(main())
```

Varias peticiones en paralelo:

```python
async def obtener_todos(ids: list[int]) -> list[dict]:
    async with httpx.AsyncClient(base_url="https://api.ejemplo.com", timeout=10) as cliente:
        respuestas = await asyncio.gather(*(cliente.get(f"/productos/{i}") for i in ids))
        return [r.json() for r in respuestas if r.is_success]
```

Con gestión de errores individual:

```python
resultados = await asyncio.gather(*tareas, return_exceptions=True)
for r in resultados:
    if isinstance(r, Exception):
        log.warning("Falló: %r", r)
```

> En `AsyncClient`, **todas** las operaciones de red llevan `await` (`get`, `post`, `aread`, `aclose`, iteración con `async for`).

---

## 10. Concurrencia controlada

Lanzar miles de peticiones a la vez puede saturar el servidor o provocar errores 429. Limite la concurrencia con un semáforo o un `TaskGroup`:

```python
async def descargar_lote(urls: list[str], maximo: int = 10) -> dict[str, int | str]:
    semaforo = asyncio.Semaphore(maximo)
    resultados: dict[str, int | str] = {}

    async with httpx.AsyncClient(timeout=10, follow_redirects=True,
                                 limits=httpx.Limits(max_connections=maximo)) as cliente:
        async def una(url: str) -> None:
            async with semaforo:
                try:
                    r = await cliente.get(url)
                    resultados[url] = r.status_code
                except httpx.RequestError as e:
                    resultados[url] = type(e).__name__

        async with asyncio.TaskGroup() as grupo:                 # Python 3.11+
            for u in urls:
                grupo.create_task(una(u))
    return resultados
```

Procesar según van llegando:

```python
for futuro in asyncio.as_completed([cliente.get(u) for u in urls]):
    r = await futuro
    procesar(r)
```

---

## 11. Reintentos

### 11.1 Errores de conexión (integrado)

```python
transporte = httpx.HTTPTransport(retries=3)                 # Síncrono
cliente = httpx.Client(transport=transporte)

transporte = httpx.AsyncHTTPTransport(retries=3)            # Asíncrono
cliente = httpx.AsyncClient(transport=transporte)
```

> `retries` solo reintenta **fallos al conectar** (`ConnectError`, `ConnectTimeout`), no códigos 5xx ni 429.

### 11.2 Reintentos por código de estado con espera exponencial

```python
import random

REINTENTABLES = {429, 500, 502, 503, 504}

async def get_con_reintentos(cliente: httpx.AsyncClient, url: str, intentos: int = 4, **kwargs) -> httpx.Response:
    for intento in range(intentos):
        try:
            r = await cliente.get(url, **kwargs)
            if r.status_code not in REINTENTABLES:
                return r
            espera = float(r.headers.get("Retry-After", 0)) or (0.5 * 2 ** intento)
        except (httpx.TimeoutException, httpx.NetworkError):
            if intento == intentos - 1:
                raise
            espera = 0.5 * 2 ** intento
        await asyncio.sleep(espera + random.uniform(0, 0.25))      # Espera + variación aleatoria
    return r
```

> Alternativa declarativa: la librería **tenacity** (`@retry(stop=stop_after_attempt(4), wait=wait_exponential())`).

---

## 12. Autenticación

```python
httpx.get(url, auth=("usuario", "clave"))                         # Basic
httpx.get(url, auth=httpx.DigestAuth("usuario", "clave"))         # Digest
httpx.get(url, headers={"Authorization": f"Bearer {token}"})      # Bearer
```

Flujo de autenticación personalizado (renovación automática de *token*):

```python
class TokenRenovable(httpx.Auth):
    requires_response_body = True

    def __init__(self, url_token: str, client_id: str, client_secret: str):
        self.url_token, self.client_id, self.client_secret = url_token, client_id, client_secret
        self.token: str | None = None

    def auth_flow(self, peticion: httpx.Request):
        if self.token is None:
            yield from self._renovar()
        peticion.headers["Authorization"] = f"Bearer {self.token}"
        respuesta = yield peticion
        if respuesta.status_code == 401:                 # Token caducado: renovar y repetir una vez
            yield from self._renovar()
            peticion.headers["Authorization"] = f"Bearer {self.token}"
            yield peticion

    def _renovar(self):
        r = yield httpx.Request("POST", self.url_token, data={
            "grant_type": "client_credentials", "client_id": self.client_id, "client_secret": self.client_secret})
        self.token = r.json()["access_token"]

cliente = httpx.Client(auth=TokenRenovable("https://auth.ejemplo.com/token", ID, SECRETO))
```

> El mismo objeto `httpx.Auth` funciona con `Client` y `AsyncClient`.

---

## 13. *Streaming*: descargas y respuestas largas

```python
from pathlib import Path

def descargar(cliente: httpx.Client, url: str, destino: Path) -> None:
    with cliente.stream("GET", url) as r:
        r.raise_for_status()
        total = int(r.headers.get("Content-Length", 0))
        with destino.open("wb") as f:
            for trozo in r.iter_bytes(chunk_size=65536):
                f.write(trozo)
                print(f"\r{r.num_bytes_downloaded / 1_048_576:.1f} MB"
                      + (f" de {total / 1_048_576:.1f} MB" if total else ""), end="")
    print()
```

Asíncrono:

```python
async with cliente.stream("GET", url) as r:
    async for linea in r.aiter_lines():             # aiter_bytes / aiter_text / aiter_raw
        procesar(linea)
```

| Método (síncrono / asíncrono) | Devuelve |
|---|---|
| `iter_bytes()` / `aiter_bytes()` | Bytes decodificados (descomprimidos). |
| `iter_text()` / `aiter_text()` | Texto. |
| `iter_lines()` / `aiter_lines()` | Líneas. |
| `iter_raw()` / `aiter_raw()` | Bytes tal cual (sin descomprimir). |
| `read()` / `aread()` | Leer todo el cuerpo dentro del bloque `stream`. |

---

## 14. Subir archivos y formularios

```python
with open("informe.pdf", "rb") as f:
    r = cliente.post("/documentos", files={"archivo": ("informe.pdf", f, "application/pdf")},
                     data={"descripcion": "Informe Q4"})

cliente.post("/raw", content=b"\x00\x01binario", headers={"Content-Type": "application/octet-stream"})
cliente.post("/texto", content="texto plano", headers={"Content-Type": "text/plain; charset=utf-8"})
```

Subida de un archivo grande sin cargarlo en memoria:

```python
def leer_por_bloques(ruta, tamano=65536):
    with open(ruta, "rb") as f:
        while bloque := f.read(tamano):
            yield bloque

cliente.put("/subidas/copia.iso", content=leer_por_bloques("copia.iso"))
```

---

## 15. HTTP/2

```python
# pip install "httpx[http2]"
async with httpx.AsyncClient(http2=True) as cliente:
    r = await cliente.get("https://www.google.com")
    print(r.http_version)        # "HTTP/2"
```

HTTP/2 permite **multiplexar** muchas peticiones sobre una sola conexión TCP, lo que mejora el rendimiento en peticiones concurrentes al mismo servidor. Si el servidor no lo admite, httpx usa HTTP/1.1 automáticamente.

---

## 16. *Event hooks*

```python
import logging
log = logging.getLogger("http")

def registrar_peticion(peticion: httpx.Request) -> None:
    log.info("→ %s %s", peticion.method, peticion.url)

def registrar_respuesta(respuesta: httpx.Response) -> None:
    peticion = respuesta.request
    log.info("← %s %s %s", peticion.method, peticion.url, respuesta.status_code)

def lanzar_si_error(respuesta: httpx.Response) -> None:
    respuesta.raise_for_status()

cliente = httpx.Client(event_hooks={"request": [registrar_peticion],
                                    "response": [registrar_respuesta, lanzar_si_error]})
```

> En `AsyncClient`, los *hooks* deben ser funciones `async`. Dentro del *hook* de respuesta, el cuerpo aún no se ha leído: use `respuesta.read()` (o `await respuesta.aread()`) si necesita el contenido.

---

## 17. SSL, *proxies* y redirecciones

```python
import ssl

httpx.get(url, verify=True)                                      # Por defecto (certifi)
ctx = ssl.create_default_context(cafile="ca-corporativa.pem")
httpx.Client(verify=ctx)                                         # CA propia (0.28+: contexto SSL)
httpx.Client(verify=False)                                       # ⚠ Solo para pruebas
httpx.Client(cert=("cliente.crt", "cliente.key"))                # Certificado de cliente

httpx.Client(proxy="http://proxy.empresa:8080")
httpx.Client(mounts={"http://": httpx.HTTPTransport(proxy="http://p:8080"),
                     "https://interno.empresa": httpx.HTTPTransport()})     # Sin proxy para un dominio
# También: variables de entorno HTTP_PROXY, HTTPS_PROXY, NO_PROXY, SSL_CERT_FILE (trust_env=True por defecto)

httpx.get("http://github.com")                                   # 301, no sigue la redirección
httpx.get("http://github.com", follow_redirects=True)            # 200 tras redirigir
httpx.Client(follow_redirects=True, max_redirects=5)
```

---

## 18. Transportes: pruebas y llamadas a aplicaciones ASGI/WSGI

### 18.1 `MockTransport` para pruebas

```python
def manejador(peticion: httpx.Request) -> httpx.Response:
    if peticion.url.path == "/usuarios/1":
        return httpx.Response(200, json={"id": 1, "nombre": "Ana"})
    if peticion.method == "POST":
        return httpx.Response(201, json={"id": 2, **json.loads(peticion.content)})
    return httpx.Response(404)

cliente = httpx.Client(base_url="https://api.prueba", transport=httpx.MockTransport(manejador))
assert cliente.get("/usuarios/1").json()["nombre"] == "Ana"
assert cliente.get("/usuarios/9").status_code == 404
```

> `MockTransport` funciona también con `AsyncClient` (el manejador puede ser síncrono o `async`). Para un enfoque al estilo de `responses`, existen **respx** y **pytest-httpx**.

### 18.2 Llamar a una aplicación en el mismo proceso

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/salud")
def salud():
    return {"ok": True}

async def probar():
    transporte = httpx.ASGITransport(app=app)
    async with httpx.AsyncClient(transport=transporte, base_url="http://test") as cliente:
        assert (await cliente.get("/salud")).json() == {"ok": True}

# Flask u otras apps WSGI (síncrono):
cliente = httpx.Client(transport=httpx.WSGITransport(app=flask_app), base_url="http://test")
```

---

## 19. Línea de comandos

```bash
pip install "httpx[cli]"
httpx https://api.github.com/repos/encode/httpx
httpx https://httpbin.org/post -m POST -j '{"nombre": "Ana"}'
httpx https://ejemplo.com --follow-redirects --verbose
httpx https://ejemplo.com/archivo.zip --download archivo.zip
```

---

## 20. Ejemplo práctico completo

**Monitor asíncrono de servicios web**: lee una lista de URLs (archivo o argumentos), las comprueba en paralelo con concurrencia limitada, reintentos con espera exponencial para errores transitorios, medición de latencia, detección de redirecciones y versión HTTP, comprobación opcional de un texto esperado en la respuesta, resumen ordenado y exportación a JSON. Incluye pruebas con `MockTransport`, sin acceso a la red.

```python
"""
monitor.py — Monitor asíncrono de servicios web con httpx.
Requiere: pip install httpx
Uso:
    python monitor.py https://www.python.org https://httpbin.org/status/503
    python monitor.py -f urls.txt --concurrencia 20 --json resultado.json
    Formato de urls.txt:  URL [texto_esperado]   (líneas con # se ignoran)
"""
import argparse
import asyncio
import json
import random
import sys
import time
from dataclasses import asdict, dataclass, field
from pathlib import Path

import httpx

REINTENTABLES = {429, 500, 502, 503, 504}


@dataclass
class Comprobacion:
    url: str
    esperado: str | None = None


@dataclass
class Resultado:
    url: str
    estado: str                              # "ok" | "degradado" | "caido"
    codigo: int | None = None
    latencia_ms: float | None = None
    intentos: int = 0
    http_version: str | None = None
    url_final: str | None = None
    error: str | None = None
    avisos: list[str] = field(default_factory=list)


class Monitor:
    def __init__(self, concurrencia: int = 10, timeout: float = 8.0, intentos: int = 3,
                 lenta_ms: float = 1500, transporte: httpx.AsyncBaseTransport | None = None,
                 espera_base: float = 0.5):
        self.semaforo = asyncio.Semaphore(concurrencia)
        self.intentos = intentos
        self.lenta_ms = lenta_ms
        self.espera_base = espera_base
        self.cliente = httpx.AsyncClient(
            timeout=httpx.Timeout(timeout, connect=min(timeout, 5.0)),
            follow_redirects=True,
            limits=httpx.Limits(max_connections=concurrencia, max_keepalive_connections=concurrencia),
            headers={"User-Agent": "MonitorServicios/1.0"},
            transport=transporte or httpx.AsyncHTTPTransport(retries=1),
        )

    async def __aenter__(self) -> "Monitor":
        return self

    async def __aexit__(self, *exc) -> None:
        await self.cliente.aclose()

    async def comprobar(self, c: Comprobacion) -> Resultado:
        async with self.semaforo:
            resultado = Resultado(url=c.url, estado="caido")
            for intento in range(1, self.intentos + 1):
                resultado.intentos = intento
                inicio = time.perf_counter()
                try:
                    r = await self.cliente.get(c.url)
                except httpx.TimeoutException:
                    resultado.error = "Tiempo de espera agotado"
                except httpx.RequestError as e:
                    resultado.error = f"{type(e).__name__}: {e}"
                else:
                    resultado.latencia_ms = round((time.perf_counter() - inicio) * 1000, 1)
                    resultado.codigo = r.status_code
                    resultado.http_version = r.http_version
                    resultado.url_final = str(r.url) if r.history else None
                    resultado.error = None
                    if r.status_code not in REINTENTABLES:
                        self._evaluar(resultado, r, c)
                        return resultado
                    resultado.error = f"HTTP {r.status_code}"
                    retry_after = r.headers.get("Retry-After", "")
                    if retry_after.isdigit() and intento < self.intentos:
                        await asyncio.sleep(min(int(retry_after), 10))
                        continue
                if intento < self.intentos:
                    await asyncio.sleep(self.espera_base * 2 ** (intento - 1) + random.uniform(0, 0.1))
            return resultado

    def _evaluar(self, resultado: Resultado, r: httpx.Response, c: Comprobacion) -> None:
        if r.is_success:
            resultado.estado = "ok"
        elif r.is_client_error:
            resultado.estado = "caido"
            resultado.error = f"HTTP {r.status_code}"
            return
        else:
            resultado.estado = "degradado"
        if r.history:
            resultado.avisos.append(f"{len(r.history)} redirección(es)")
        if resultado.latencia_ms and resultado.latencia_ms > self.lenta_ms:
            resultado.estado = "degradado"
            resultado.avisos.append(f"respuesta lenta (> {self.lenta_ms:.0f} ms)")
        if c.esperado and c.esperado not in r.text:
            resultado.estado = "degradado"
            resultado.avisos.append(f"no contiene «{c.esperado}»")
        if r.url.scheme == "http":
            resultado.avisos.append("sin HTTPS")

    async def comprobar_todas(self, comprobaciones: list[Comprobacion]) -> list[Resultado]:
        return await asyncio.gather(*(self.comprobar(c) for c in comprobaciones))


# ------------------------------------------------------------------ Entrada y salida
def leer_comprobaciones(urls: list[str], archivo: Path | None) -> list[Comprobacion]:
    comprobaciones = [Comprobacion(u) for u in urls]
    if archivo:
        for linea in archivo.read_text(encoding="utf-8").splitlines():
            linea = linea.strip()
            if not linea or linea.startswith("#"):
                continue
            url, _, esperado = linea.partition(" ")
            comprobaciones.append(Comprobacion(url, esperado.strip() or None))
    validas = []
    for c in comprobaciones:
        if not c.url.startswith(("http://", "https://")):
            print(f"⚠ URL ignorada (falta http/https): {c.url}", file=sys.stderr)
            continue
        validas.append(c)
    return validas


ICONOS = {"ok": "✔", "degradado": "⚠", "caido": "✖"}


def mostrar(resultados: list[Resultado], segundos: float) -> None:
    orden = {"caido": 0, "degradado": 1, "ok": 2}
    resultados = sorted(resultados, key=lambda r: (orden[r.estado], -(r.latencia_ms or 0)))
    ancho = min(60, max(len(r.url) for r in resultados))
    print(f"\n{'':2}{'URL':<{ancho}}  {'Código':>6}  {'Latencia':>9}  {'HTTP':<8}  Detalle")
    print("-" * (ancho + 50))
    for r in resultados:
        latencia = f"{r.latencia_ms:.0f} ms" if r.latencia_ms is not None else "—"
        detalle = "; ".join(filter(None, [r.error, *r.avisos]))
        if r.intentos > 1:
            detalle += f" ({r.intentos} intentos)"
        print(f"{ICONOS[r.estado]} {r.url[:ancho]:<{ancho}}  {r.codigo or '—':>6}  {latencia:>9}  "
              f"{r.http_version or '—':<8}  {detalle}")
    cuenta = {e: sum(r.estado == e for r in resultados) for e in ICONOS}
    print(f"\n{len(resultados)} servicios en {segundos:.2f} s · "
          f"{cuenta['ok']} correctos · {cuenta['degradado']} degradados · {cuenta['caido']} caídos")


async def ejecutar(args) -> int:
    comprobaciones = leer_comprobaciones(args.urls, args.archivo)
    if not comprobaciones:
        print("No hay URLs que comprobar.", file=sys.stderr)
        return 2
    inicio = time.perf_counter()
    async with Monitor(concurrencia=args.concurrencia, timeout=args.timeout, intentos=args.intentos,
                       lenta_ms=args.lenta) as monitor:
        resultados = await monitor.comprobar_todas(comprobaciones)
    mostrar(resultados, time.perf_counter() - inicio)
    if args.json:
        args.json.write_text(json.dumps([asdict(r) for r in resultados], ensure_ascii=False, indent=2),
                             encoding="utf-8")
        print(f"Resultados guardados en {args.json}")
    return 1 if any(r.estado == "caido" for r in resultados) else 0


def main() -> int:
    p = argparse.ArgumentParser(description="Monitor asíncrono de servicios web.")
    p.add_argument("urls", nargs="*", help="URLs a comprobar")
    p.add_argument("-f", "--archivo", type=Path, help="archivo con una URL por línea")
    p.add_argument("-c", "--concurrencia", type=int, default=10)
    p.add_argument("-t", "--timeout", type=float, default=8.0)
    p.add_argument("-i", "--intentos", type=int, default=3)
    p.add_argument("--lenta", type=float, default=1500, help="umbral de respuesta lenta en ms")
    p.add_argument("--json", type=Path, help="guardar los resultados en JSON")
    try:
        return asyncio.run(ejecutar(p.parse_args()))
    except KeyboardInterrupt:
        return 130


if __name__ == "__main__":
    sys.exit(main())
```

Pruebas sin red (`test_monitor.py`):

```python
import asyncio
import httpx
from monitor import Comprobacion, Monitor

llamadas: dict[str, int] = {}


def servidor_simulado(peticion: httpx.Request) -> httpx.Response:
    ruta = peticion.url.path
    llamadas[ruta] = llamadas.get(ruta, 0) + 1
    if ruta == "/ok":
        return httpx.Response(200, text="<h1>Bienvenido</h1>")
    if ruta == "/intermitente":                              # Falla la primera vez
        return httpx.Response(503) if llamadas[ruta] == 1 else httpx.Response(200, text="ok")
    if ruta == "/viejo":
        return httpx.Response(301, headers={"Location": "https://servicio.test/ok"})
    if ruta == "/caido":
        raise httpx.ConnectError("Conexión rechazada", request=peticion)
    return httpx.Response(404)


def comprobar(*comprobaciones: Comprobacion):
    async def _ejecutar():
        async with Monitor(transporte=httpx.MockTransport(servidor_simulado), espera_base=0) as m:
            return await m.comprobar_todas(list(comprobaciones))
    return asyncio.run(_ejecutar())


def test_estados():
    ok, texto, inter, redir, caido, no_existe = comprobar(
        Comprobacion("https://servicio.test/ok", "Bienvenido"),
        Comprobacion("https://servicio.test/ok", "Inexistente"),
        Comprobacion("https://servicio.test/intermitente"),
        Comprobacion("https://servicio.test/viejo"),
        Comprobacion("https://servicio.test/caido"),
        Comprobacion("https://servicio.test/no-existe"),
    )
    assert ok.estado == "ok" and ok.codigo == 200
    assert texto.estado == "degradado" and "no contiene" in texto.avisos[0]
    assert inter.estado == "ok" and inter.intentos == 2
    assert redir.estado == "ok" and redir.url_final == "https://servicio.test/ok"
    assert caido.estado == "caido" and "ConnectError" in caido.error and caido.intentos == 3
    assert no_existe.estado == "caido" and no_existe.codigo == 404
```

```bash
python monitor.py https://www.python.org https://pypi.org http://example.com --json estado.json
pytest -q test_monitor.py
```

---

## 21. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `httpx.ReadTimeout` donde Requests funcionaba | *Timeout* por defecto de 5 s. | Ajuste `timeout=` según el servicio. |
| La respuesta es 301/302 y no el contenido | httpx no sigue redirecciones por defecto. | `follow_redirects=True`. |
| `RuntimeWarning: coroutine … was never awaited` | Falta `await` con `AsyncClient`. | `await cliente.get(...)`. |
| `RuntimeError: Event loop is closed` / cliente cerrado | El `AsyncClient` se usó fuera de su bloque o en otro bucle. | Cree y use el cliente dentro del mismo `asyncio.run`. |
| `PoolTimeout` | Más peticiones simultáneas que conexiones disponibles. | Aumente `Limits` o reduzca la concurrencia. |
| `TypeError: … unexpected keyword argument 'proxies'` / `'app'` | Parámetros eliminados en 0.28. | `proxy=`/`mounts=` y `transport=ASGITransport(app=…)`. |
| `ResponseNotRead` | Acceso a `.text`/`.json()` en una respuesta `stream` sin leer. | `r.read()` / `await r.aread()` dentro del bloque. |
| `ImportError: Using http2=True, but the 'h2' package is not installed` | Falta la dependencia. | `pip install "httpx[http2]"`. |
| Los reintentos no actúan con errores 503 | `retries` solo cubre errores de conexión. | Implemente reintentos por código (sección 11.2) o use tenacity. |
| Error 429 masivo | Concurrencia excesiva. | Semáforo, `Limits` y respeto de `Retry-After`. |

---

## 22. Buenas prácticas

- Reutilice un único `Client`/`AsyncClient` por servicio y ciérrelo (`with` / `async with`).
- Ajuste los *timeouts* de forma explícita según cada servicio.
- Limite la concurrencia con semáforos y `httpx.Limits`; respete los límites de las APIs.
- Implemente reintentos con espera exponencial para 429/5xx y errores transitorios.
- Use `raise_for_status()` (o compruebe `is_success`) antes de procesar el contenido.
- Use `stream()` para descargas grandes.
- En aplicaciones asíncronas (FastAPI, bots), use siempre `AsyncClient`; nunca Requests dentro de `async def`.
- Pruebe con `MockTransport`, respx o pytest-httpx en lugar de llamar a servicios reales.
- Al migrar desde Requests, revise redirecciones, *timeouts*, `data`/`content` y excepciones.

---

## 23. Referencia rápida

```python
httpx.get/post/put/patch/delete(url, params=, json=, data=, content=, files=, headers=, auth=, timeout=, follow_redirects=)
with httpx.Client(base_url=, headers=, timeout=httpx.Timeout(10, connect=3), limits=httpx.Limits(max_connections=20),
                  follow_redirects=True, http2=True, transport=httpx.HTTPTransport(retries=3)) as c: c.get("/ruta")
async with httpx.AsyncClient(...) as c: r = await c.get(url) · await asyncio.gather(*(c.get(u) for u in urls))
r.status_code · r.is_success · r.raise_for_status().json() · r.text · r.content · r.headers · r.http_version · r.elapsed
with c.stream("GET", url) as r: for b in r.iter_bytes() · async with c.stream(...) as r: async for l in r.aiter_lines()
HTTPError > RequestError (TimeoutException, NetworkError…) · HTTPStatusError
auth=("u", "c") · httpx.DigestAuth · class X(httpx.Auth): def auth_flow(self, req): yield req
event_hooks={"request": [f], "response": [g]} · verify=ssl_ctx · proxy="http://…" · mounts={...}
httpx.MockTransport(manejador) · httpx.ASGITransport(app=app) · httpx.WSGITransport(app=app)
```

---

## 24. Ejercicios propuestos

1. Reescriba con `AsyncClient` un script que consulte 200 URLs de una API y compare el tiempo con la versión síncrona.
2. Implemente un descargador asíncrono de imágenes con concurrencia limitada, que guarde cada archivo y muestre el total descargado.
3. Cree una clase `httpx.Auth` que obtenga y renueve automáticamente un *token* OAuth2 y pruébela con `MockTransport`.
4. Amplíe el monitor con un modo `--vigilar N` que repita las comprobaciones cada N segundos y avise solo cuando cambie el estado de un servicio.
5. Pruebe una API FastAPI propia con `ASGITransport` sin arrancar ningún servidor.

---

## 25. Referencias

- Documentación oficial: https://www.python-httpx.org/
- Repositorio: https://github.com/encode/httpx
- Guía de compatibilidad con Requests: https://www.python-httpx.org/compatibility/
- respx (simulación): https://lundberg.github.io/respx/
- tenacity (reintentos): https://tenacity.readthedocs.io/
