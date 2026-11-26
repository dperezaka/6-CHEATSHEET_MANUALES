# Manual de uso: Requests

> **Categoría:** Desarrollo web · **Utilidad:** Realizar peticiones HTTP
> **Tipo:** Librería de terceros (`pip install requests`) · **Versión de referencia:** 2.32 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Peticiones básicas](#4-peticiones-básicas)
5. [Parámetros, cabeceras y cuerpo](#5-parámetros-cabeceras-y-cuerpo)
6. [La respuesta (`Response`)](#6-la-respuesta-response)
7. [Gestión de errores y códigos de estado](#7-gestión-de-errores-y-códigos-de-estado)
8. [Tiempos de espera (*timeouts*)](#8-tiempos-de-espera-timeouts)
9. [Sesiones (`Session`)](#9-sesiones-session)
10. [Reintentos automáticos](#10-reintentos-automáticos)
11. [Autenticación](#11-autenticación)
12. [Cookies](#12-cookies)
13. [Subir archivos](#13-subir-archivos)
14. [Descargar archivos y *streaming*](#14-descargar-archivos-y-streaming)
15. [Paginación de APIs](#15-paginación-de-apis)
16. [Certificados SSL y *proxies*](#16-certificados-ssl-y-proxies)
17. [*Hooks*, adaptadores y depuración](#17-hooks-adaptadores-y-depuración)
18. [Concurrencia](#18-concurrencia)
19. [Pruebas: simular respuestas](#19-pruebas-simular-respuestas)
20. [Ejemplo práctico completo](#20-ejemplo-práctico-completo)
21. [Errores frecuentes](#21-errores-frecuentes)
22. [Buenas prácticas](#22-buenas-prácticas)
23. [Referencia rápida](#23-referencia-rápida)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Referencias](#25-referencias)

---

## 1. Descripción

**Requests** es la librería más popular de Python para realizar peticiones **HTTP**: consultar APIs REST, enviar formularios, descargar archivos o interactuar con servicios web. Su lema es «HTTP para humanos»: ofrece una API sencilla y legible sobre **urllib3**, con gestión automática de codificaciones, JSON, cookies, redirecciones, conexiones persistentes y autenticación.

**Ventajas:**
- API muy intuitiva: `requests.get(url).json()`.
- Gestión automática de JSON, formularios, archivos, cookies y redirecciones.
- Sesiones con conexiones reutilizables y cabeceras comunes.
- Reintentos configurables mediante adaptadores de urllib3.
- Estable, muy documentada y presente en innumerables proyectos.

**Limitaciones:**
- Solo síncrona: no admite `async/await`.
- No soporta HTTP/2.
- Sin *timeout* por defecto (hay que indicarlo siempre).
- Para miles de peticiones concurrentes, httpx (asíncrono) o aiohttp son más adecuados.

| Si necesita… | Opción |
|---|---|
| Peticiones HTTP síncronas sencillas y robustas | **Requests** |
| Peticiones asíncronas, HTTP/2 o API compatible con Requests | httpx |
| Muchísimas peticiones asíncronas en paralelo | aiohttp / httpx |
| Extraer datos del HTML descargado | BeautifulSoup4 |
| Páginas que se generan con JavaScript | Playwright / Selenium |

---

## 2. Instalación e importación

```bash
pip install requests
```

```python
import requests
from requests.adapters import HTTPAdapter
from requests.exceptions import HTTPError, ConnectionError, Timeout, RequestException
from urllib3.util.retry import Retry
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Método HTTP | Acción: `GET` (leer), `POST` (crear), `PUT`/`PATCH` (actualizar), `DELETE` (borrar). |
| URL | Dirección del recurso; puede incluir parámetros de consulta (`?q=python`). |
| Cabeceras | Metadatos: `Authorization`, `Content-Type`, `Accept`, `User-Agent`… |
| Cuerpo | Datos enviados: JSON, formulario, archivos. |
| Código de estado | Resultado: 2xx éxito, 3xx redirección, 4xx error del cliente, 5xx error del servidor. |
| `Response` | Objeto con el estado, las cabeceras y el contenido de la respuesta. |
| `Session` | Objeto que reutiliza conexiones, cookies y configuración entre peticiones. |
| *Timeout* | Tiempo máximo de espera de conexión y de lectura. |
| Adaptador | Componente que gestiona el transporte (reintentos, tamaño del *pool*). |

---

## 4. Peticiones básicas

```python
import requests

r = requests.get("https://api.github.com/repos/psf/requests", timeout=10)
print(r.status_code)            # 200
print(r.json()["stargazers_count"])

requests.post(url, json={"nombre": "Ana"}, timeout=10)
requests.put(url, json={...}, timeout=10)
requests.patch(url, json={"activo": False}, timeout=10)
requests.delete(url, timeout=10)
requests.head(url, timeout=10)              # Solo cabeceras
requests.options(url, timeout=10)
requests.request("GET", url, timeout=10)    # Método genérico
```

---

## 5. Parámetros, cabeceras y cuerpo

### 5.1 Parámetros de consulta

```python
params = {"q": "python requests", "page": 2, "tags": ["web", "http"]}
r = requests.get("https://api.ejemplo.com/buscar", params=params, timeout=10)
print(r.url)     # https://api.ejemplo.com/buscar?q=python+requests&page=2&tags=web&tags=http
```

> Use siempre `params=` en lugar de construir la URL a mano: Requests codifica correctamente espacios, tildes y caracteres especiales.

### 5.2 Cabeceras

```python
cabeceras = {
    "User-Agent": "MiAplicacion/1.0 (contacto@ejemplo.com)",
    "Accept": "application/json",
    "Accept-Language": "es-ES",
}
r = requests.get(url, headers=cabeceras, timeout=10)
```

### 5.3 Tipos de cuerpo

```python
# JSON (Content-Type: application/json automáticamente)
requests.post(url, json={"titulo": "Informe", "etiquetas": ["q4"]}, timeout=10)

# Formulario (application/x-www-form-urlencoded)
requests.post(url, data={"usuario": "ana", "clave": "secreta"}, timeout=10)

# Texto o bytes sin procesar
requests.post(url, data="<xml>…</xml>".encode(), headers={"Content-Type": "application/xml"}, timeout=10)

# Archivos (multipart/form-data) → sección 13
requests.post(url, files={"archivo": open("informe.pdf", "rb")}, timeout=30)
```

---

## 6. La respuesta (`Response`)

| Atributo / método | Contenido |
|---|---|
| `r.status_code` | Código HTTP (200, 404…). |
| `r.ok` | `True` si el código es menor que 400. |
| `r.reason` | Texto del estado («OK», «Not Found»). |
| `r.headers` | Cabeceras (diccionario que no distingue mayúsculas). |
| `r.text` | Contenido decodificado como texto. |
| `r.content` | Contenido en bytes (imágenes, PDF…). |
| `r.json()` | Contenido JSON como objeto Python. |
| `r.encoding` | Codificación usada para `text` (modificable). |
| `r.url` | URL final (tras redirecciones). |
| `r.history` | Lista de respuestas de redirección. |
| `r.elapsed` | Tiempo transcurrido (`timedelta`). |
| `r.cookies` | Cookies recibidas. |
| `r.links` | Enlaces de la cabecera `Link` (paginación). |
| `r.request` | Petición enviada (método, URL, cabeceras, cuerpo). |
| `r.raise_for_status()` | Lanza `HTTPError` si el código es 4xx o 5xx. |

```python
r = requests.get(url, timeout=10)
if r.headers.get("Content-Type", "").startswith("application/json"):
    datos = r.json()
else:
    texto = r.text
```

> Si una página muestra caracteres extraños, la codificación detectada es incorrecta: `r.encoding = "utf-8"` (o `r.apparent_encoding`) antes de leer `r.text`.

---

## 7. Gestión de errores y códigos de estado

### 7.1 Jerarquía de excepciones

| Excepción | Cuándo |
|---|---|
| `RequestException` | Base de todas las excepciones de Requests. |
| `ConnectionError` | No se pudo conectar (DNS, servidor caído, sin red). |
| `Timeout` (`ConnectTimeout`, `ReadTimeout`) | Se superó el tiempo de espera. |
| `HTTPError` | Lanzada por `raise_for_status()` con códigos 4xx/5xx. |
| `TooManyRedirects` | Bucle de redirecciones. |
| `JSONDecodeError` (`requests.exceptions.JSONDecodeError`) | La respuesta no es JSON válido. |
| `MissingSchema`, `InvalidURL` | URL mal formada. |

### 7.2 Patrón recomendado

```python
import requests

def obtener_usuario(id_: int) -> dict | None:
    try:
        r = requests.get(f"https://api.ejemplo.com/usuarios/{id_}", timeout=(3.05, 10))
        if r.status_code == 404:
            return None
        r.raise_for_status()
        return r.json()
    except requests.Timeout:
        raise RuntimeError("El servidor no responde a tiempo")
    except requests.HTTPError as e:
        detalle = e.response.text[:200]
        raise RuntimeError(f"Error {e.response.status_code}: {detalle}") from e
    except requests.ConnectionError as e:
        raise RuntimeError("No hay conexión con el servidor") from e
    except requests.exceptions.JSONDecodeError as e:
        raise RuntimeError("La respuesta no es JSON válido") from e
```

### 7.3 Códigos de estado con nombre

```python
from http import HTTPStatus

if r.status_code == HTTPStatus.TOO_MANY_REQUESTS:          # 429
    espera = int(r.headers.get("Retry-After", 60))
```

---

## 8. Tiempos de espera (*timeouts*)

**Requests no tiene *timeout* por defecto**: sin él, una petición puede quedarse bloqueada indefinidamente.

```python
requests.get(url, timeout=10)            # 10 s para conectar y 10 s para cada lectura
requests.get(url, timeout=(3.05, 27))    # (conexión, lectura)
requests.get(url, timeout=None)          # Esperar indefinidamente (desaconsejado)
```

> El *timeout* de lectura es el tiempo máximo **entre bytes recibidos**, no la duración total de la descarga.

*Timeout* por defecto en una sesión:

```python
class SesionConTimeout(requests.Session):
    def __init__(self, timeout=(3.05, 15)):
        super().__init__()
        self.timeout = timeout

    def request(self, *args, **kwargs):
        kwargs.setdefault("timeout", self.timeout)
        return super().request(*args, **kwargs)
```

---

## 9. Sesiones (`Session`)

Una sesión **reutiliza la conexión TCP** (mucho más rápida en peticiones repetidas al mismo servidor), mantiene **cookies** y permite definir cabeceras, autenticación y parámetros comunes.

```python
with requests.Session() as s:
    s.headers.update({"User-Agent": "MiApp/1.0", "Accept": "application/json"})
    s.params = {"lang": "es"}                        # Parámetros en todas las peticiones
    s.auth = ("usuario", "clave")

    r1 = s.get("https://api.ejemplo.com/clientes", timeout=10)
    r2 = s.get("https://api.ejemplo.com/pedidos", timeout=10, headers={"X-Extra": "1"})   # Se combinan
```

| Atributo de `Session` | Uso |
|---|---|
| `headers` | Cabeceras comunes. |
| `params` | Parámetros comunes. |
| `auth` | Autenticación común. |
| `cookies` | Almacén de cookies. |
| `verify`, `cert` | Certificados SSL. |
| `proxies` | *Proxies*. |
| `hooks` | Funciones ejecutadas en cada respuesta. |
| `mount(prefijo, adaptador)` | Adaptadores (reintentos, *pool*). |

---

## 10. Reintentos automáticos

```python
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

reintentos = Retry(
    total=5,                                         # Intentos máximos
    connect=3, read=3,                               # Por tipo de fallo
    backoff_factor=0.5,                              # Esperas: 0.5, 1, 2, 4… segundos
    status_forcelist=[429, 500, 502, 503, 504],      # Códigos que se reintentan
    allowed_methods=["GET", "HEAD", "OPTIONS", "PUT", "DELETE"],   # Métodos idempotentes
    respect_retry_after_header=True,
    raise_on_status=False,                           # Devuelve la última respuesta en lugar de lanzar
)
adaptador = HTTPAdapter(max_retries=reintentos, pool_connections=10, pool_maxsize=20)

s = requests.Session()
s.mount("https://", adaptador)
s.mount("http://", adaptador)
```

> No reintente automáticamente `POST` salvo que la API sea idempotente (por ejemplo, con cabecera `Idempotency-Key`): podría crear registros duplicados.

---

## 11. Autenticación

```python
from requests.auth import HTTPBasicAuth, HTTPDigestAuth

requests.get(url, auth=("usuario", "clave"), timeout=10)                    # Basic
requests.get(url, auth=HTTPDigestAuth("usuario", "clave"), timeout=10)     # Digest

# Token Bearer (OAuth2, JWT)
requests.get(url, headers={"Authorization": f"Bearer {token}"}, timeout=10)

# Clave de API en cabecera o parámetro
requests.get(url, headers={"X-API-Key": clave}, timeout=10)
requests.get(url, params={"api_key": clave}, timeout=10)
```

Autenticación personalizada reutilizable:

```python
from requests.auth import AuthBase

class TokenAuth(AuthBase):
    def __init__(self, token: str):
        self.token = token

    def __call__(self, peticion):
        peticion.headers["Authorization"] = f"Bearer {self.token}"
        return peticion

s.auth = TokenAuth(os.environ["API_TOKEN"])
```

Obtener un *token* OAuth2 (credenciales de cliente):

```python
r = requests.post("https://auth.ejemplo.com/oauth/token", timeout=10, data={
    "grant_type": "client_credentials", "client_id": CLIENT_ID, "client_secret": CLIENT_SECRET})
token = r.json()["access_token"]
```

> Nunca escriba credenciales en el código: léalas de variables de entorno o de un gestor de secretos. Para flujos OAuth2 completos: `requests-oauthlib`.

---

## 12. Cookies

```python
r = requests.get(url, timeout=10)
r.cookies.get("sesion")

requests.get(url, cookies={"preferencia": "oscuro"}, timeout=10)

with requests.Session() as s:
    s.post("https://ejemplo.com/login", data={"usuario": "ana", "clave": "…"}, timeout=10)
    s.get("https://ejemplo.com/area-privada", timeout=10)       # La cookie de sesión se envía sola
    print(s.cookies.get_dict())
```

---

## 13. Subir archivos

```python
from pathlib import Path

ruta = Path("informe.pdf")
with ruta.open("rb") as f:
    r = requests.post(
        "https://api.ejemplo.com/documentos",
        files={"archivo": (ruta.name, f, "application/pdf")},     # (nombre, contenido, tipo)
        data={"descripcion": "Informe trimestral"},               # Campos adicionales del formulario
        timeout=60,
    )

# Varios archivos con el mismo nombre de campo
archivos = [("fotos", ("a.jpg", open("a.jpg", "rb"), "image/jpeg")),
            ("fotos", ("b.jpg", open("b.jpg", "rb"), "image/jpeg"))]
```

> Para archivos muy grandes sin cargarlos en memoria: `requests-toolbelt` (`MultipartEncoder`) o enviar el archivo directamente con `data=f`.

---

## 14. Descargar archivos y *streaming*

```python
from pathlib import Path

def descargar(url: str, destino: Path, bloque: int = 1024 * 64) -> Path:
    with requests.get(url, stream=True, timeout=(5, 30)) as r:
        r.raise_for_status()
        temporal = destino.with_suffix(destino.suffix + ".part")
        with temporal.open("wb") as f:
            for trozo in r.iter_content(chunk_size=bloque):
                f.write(trozo)
        temporal.replace(destino)                  # Solo se renombra si la descarga terminó
    return destino
```

Con barra de progreso (tqdm):

```python
from tqdm import tqdm

with requests.get(url, stream=True, timeout=(5, 30)) as r:
    r.raise_for_status()
    total = int(r.headers.get("Content-Length", 0)) or None
    with open("archivo.zip", "wb") as f, tqdm(total=total, unit="B", unit_scale=True, unit_divisor=1024) as barra:
        for trozo in r.iter_content(65536):
            barra.update(f.write(trozo))
```

Reanudar una descarga interrumpida (si el servidor admite rangos):

```python
ya = destino.stat().st_size if destino.exists() else 0
r = requests.get(url, headers={"Range": f"bytes={ya}-"}, stream=True, timeout=30)
modo = "ab" if r.status_code == 206 else "wb"          # 206 Partial Content
```

Leer líneas de respuestas largas (logs, NDJSON, *Server-Sent Events*):

```python
with requests.get(url, stream=True, timeout=(5, 60)) as r:
    for linea in r.iter_lines(decode_unicode=True):
        if linea:
            evento = json.loads(linea)
```

---

## 15. Paginación de APIs

### 15.1 Por número de página

```python
def todos(s: requests.Session, url: str, tamano: int = 100):
    pagina = 1
    while True:
        r = s.get(url, params={"page": pagina, "per_page": tamano}, timeout=10)
        r.raise_for_status()
        lote = r.json()
        if not lote:
            break
        yield from lote
        pagina += 1
```

### 15.2 Por cabecera `Link` (GitHub y otras)

```python
def todos_por_link(s: requests.Session, url: str, **params):
    while url:
        r = s.get(url, params=params, timeout=10)
        r.raise_for_status()
        yield from r.json()
        url = r.links.get("next", {}).get("url")
        params = None                               # La URL 'next' ya incluye los parámetros
```

### 15.3 Por cursor

```python
cursor = None
while True:
    datos = s.get(url, params={"cursor": cursor, "limit": 200}, timeout=10).json()
    procesar(datos["items"])
    cursor = datos.get("next_cursor")
    if not cursor:
        break
```

---

## 16. Certificados SSL y *proxies*

```python
requests.get(url, verify=True, timeout=10)                       # Por defecto: verifica con certifi
requests.get(url, verify="/ruta/ca-corporativa.pem", timeout=10) # CA propia (redes corporativas)
requests.get(url, cert=("cliente.crt", "cliente.key"), timeout=10)   # Certificado de cliente (mTLS)
requests.get(url, verify=False, timeout=10)                      # ⚠ Desactiva la verificación: solo pruebas
```

```python
proxies = {"http": "http://proxy.empresa:8080", "https": "http://usuario:clave@proxy.empresa:8080"}
requests.get(url, proxies=proxies, timeout=10)
# También desde el entorno: HTTP_PROXY, HTTPS_PROXY, NO_PROXY, REQUESTS_CA_BUNDLE
```

> `verify=False` expone la comunicación a ataques de intermediario. En entornos corporativos, configure el certificado de la CA interna con `verify=` o `REQUESTS_CA_BUNDLE`.

---

## 17. *Hooks*, adaptadores y depuración

### 17.1 *Hooks*

```python
def registrar(respuesta, *args, **kwargs):
    log.info("%s %s → %s (%.0f ms)", respuesta.request.method, respuesta.url,
             respuesta.status_code, respuesta.elapsed.total_seconds() * 1000)

def comprobar(respuesta, *args, **kwargs):
    respuesta.raise_for_status()

s.hooks["response"] = [registrar, comprobar]
```

### 17.2 Inspeccionar la petición enviada

```python
r = requests.post(url, json={"a": 1}, timeout=10)
print(r.request.method, r.request.url)
print(r.request.headers)
print(r.request.body)
```

Preparar una petición sin enviarla:

```python
peticion = requests.Request("POST", url, json={"a": 1}, headers={"X": "1"}).prepare()
print(peticion.body)
r = requests.Session().send(peticion, timeout=10)
```

### 17.3 Registro detallado de urllib3

```python
import logging
logging.basicConfig(level=logging.DEBUG)          # Muestra conexiones, reintentos y redirecciones
```

---

## 18. Concurrencia

Requests es síncrona, pero puede paralelizar peticiones con hilos (las peticiones esperan E/S y liberan el GIL):

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def estado(url: str) -> tuple[str, int | str]:
    try:
        return url, sesion.head(url, timeout=5, allow_redirects=True).status_code
    except requests.RequestException as e:
        return url, type(e).__name__

sesion = requests.Session()
sesion.mount("https://", HTTPAdapter(pool_maxsize=16))
with ThreadPoolExecutor(max_workers=16) as ex:
    for futuro in as_completed(ex.submit(estado, u) for u in urls):
        print(*futuro.result())
```

> Una `Session` puede compartirse entre hilos para peticiones simples, pero no modifique su configuración (cabeceras, cookies) desde varios hilos a la vez. Para cientos o miles de peticiones concurrentes, use **httpx** asíncrono.

---

## 19. Pruebas: simular respuestas

No haga peticiones reales en las pruebas unitarias. Opciones:

### 19.1 `unittest.mock`

```python
from unittest.mock import patch, Mock

def test_obtener_usuario():
    respuesta = Mock(status_code=200)
    respuesta.json.return_value = {"id": 1, "nombre": "Ana"}
    with patch("miapp.cliente.requests.get", return_value=respuesta):
        assert obtener_usuario(1)["nombre"] == "Ana"
```

### 19.2 `responses` (recomendado)

```bash
pip install responses
```

```python
import responses

@responses.activate
def test_api():
    responses.get("https://api.ejemplo.com/usuarios/1", json={"id": 1, "nombre": "Ana"}, status=200)
    responses.get("https://api.ejemplo.com/usuarios/2", status=404)
    assert obtener_usuario(1)["nombre"] == "Ana"
    assert obtener_usuario(2) is None
    assert len(responses.calls) == 2
```

Otras opciones: `requests-mock`, o grabar respuestas reales con `vcrpy`.

---

## 20. Ejemplo práctico completo

**Cliente reutilizable para una API REST** con sesión, cabeceras comunes, *timeout* por defecto, reintentos con espera exponencial, autenticación por *token*, gestión de errores con excepciones propias, paginación, respeto de `Retry-After`, registro de peticiones y una pequeña CLI que sincroniza usuarios y publicaciones y los exporta a CSV y JSON. Está preparado para la API pública de pruebas **JSONPlaceholder** (`https://jsonplaceholder.typicode.com`), pero sirve para cualquier API similar cambiando la URL base.

```python
"""
cliente_api.py — Cliente HTTP robusto con Requests.
Requiere: pip install requests
Uso:
    python cliente_api.py resumen
    python cliente_api.py exportar --salida datos/ --usuario 1
    python cliente_api.py crear --usuario 1 --titulo "Hola" --cuerpo "Texto"
Variables de entorno opcionales: API_URL, API_TOKEN
"""
import argparse
import csv
import json
import logging
import os
import sys
import time
from collections import Counter
from pathlib import Path
from typing import Any, Iterator

import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

log = logging.getLogger("cliente_api")


# ------------------------------------------------------------------ Excepciones propias
class ErrorAPI(Exception):
    """Error genérico de la API."""

    def __init__(self, mensaje: str, estado: int | None = None, detalle: Any = None):
        super().__init__(mensaje)
        self.estado, self.detalle = estado, detalle


class NoEncontrado(ErrorAPI):
    pass


class NoAutorizado(ErrorAPI):
    pass


class DatosNoValidos(ErrorAPI):
    pass


# ------------------------------------------------------------------ Cliente
class ClienteAPI:
    def __init__(self, url_base: str, token: str | None = None, timeout: tuple[float, float] = (3.05, 15),
                 reintentos: int = 4, agente: str = "ClienteAPI/1.0"):
        self.url_base = url_base.rstrip("/")
        self.timeout = timeout
        self.sesion = requests.Session()
        self.sesion.headers.update({"User-Agent": agente, "Accept": "application/json"})
        if token:
            self.sesion.headers["Authorization"] = f"Bearer {token}"

        politica = Retry(total=reintentos, backoff_factor=0.5, status_forcelist=[429, 500, 502, 503, 504],
                         allowed_methods=["GET", "HEAD", "OPTIONS", "PUT", "DELETE"],
                         respect_retry_after_header=True, raise_on_status=False)
        adaptador = HTTPAdapter(max_retries=politica, pool_connections=4, pool_maxsize=10)
        self.sesion.mount("https://", adaptador)
        self.sesion.mount("http://", adaptador)
        self.sesion.hooks["response"].append(self._registrar)

    # ---------- Infraestructura
    @staticmethod
    def _registrar(r: requests.Response, *args, **kwargs) -> None:
        log.debug("%s %s → %s en %.0f ms", r.request.method, r.url, r.status_code,
                  r.elapsed.total_seconds() * 1000)

    def __enter__(self) -> "ClienteAPI":
        return self

    def __exit__(self, *exc) -> None:
        self.sesion.close()

    def _peticion(self, metodo: str, ruta: str, **kwargs) -> Any:
        url = f"{self.url_base}/{ruta.lstrip('/')}"
        kwargs.setdefault("timeout", self.timeout)
        try:
            r = self.sesion.request(metodo, url, **kwargs)
        except requests.Timeout as e:
            raise ErrorAPI(f"Tiempo de espera agotado en {metodo} {url}") from e
        except requests.ConnectionError as e:
            raise ErrorAPI(f"No se pudo conectar con {self.url_base}") from e

        if r.status_code == 404:
            raise NoEncontrado(f"Recurso no encontrado: {ruta}", 404)
        if r.status_code in (401, 403):
            raise NoAutorizado("Credenciales no válidas o sin permisos", r.status_code)
        if r.status_code in (400, 422):
            raise DatosNoValidos("Datos rechazados por la API", r.status_code, self._cuerpo(r))
        if r.status_code >= 400:
            raise ErrorAPI(f"Error {r.status_code} en {metodo} {url}", r.status_code, self._cuerpo(r))
        if r.status_code == 204 or not r.content:
            return None
        try:
            return r.json()
        except requests.exceptions.JSONDecodeError as e:
            raise ErrorAPI("La respuesta no es JSON válido", r.status_code, r.text[:200]) from e

    @staticmethod
    def _cuerpo(r: requests.Response) -> Any:
        try:
            return r.json()
        except ValueError:
            return r.text[:500]

    def get(self, ruta: str, **params) -> Any:
        return self._peticion("GET", ruta, params=params or None)

    def post(self, ruta: str, datos: dict) -> Any:
        return self._peticion("POST", ruta, json=datos)

    def patch(self, ruta: str, datos: dict) -> Any:
        return self._peticion("PATCH", ruta, json=datos)

    def delete(self, ruta: str) -> None:
        self._peticion("DELETE", ruta)

    def paginar(self, ruta: str, tamano: int = 50, **params) -> Iterator[dict]:
        """Recorre una colección paginada con _page/_limit (convención de json-server)."""
        pagina = 1
        while True:
            lote = self.get(ruta, _page=pagina, _limit=tamano, **params)
            if not lote:
                return
            yield from lote
            if len(lote) < tamano:
                return
            pagina += 1

    # ---------- Operaciones de dominio
    def usuarios(self) -> list[dict]:
        return self.get("/users")

    def usuario(self, id_: int) -> dict:
        return self.get(f"/users/{id_}")

    def publicaciones(self, usuario_id: int | None = None) -> list[dict]:
        filtros = {"userId": usuario_id} if usuario_id else {}
        return list(self.paginar("/posts", **filtros))

    def crear_publicacion(self, usuario_id: int, titulo: str, cuerpo: str) -> dict:
        titulo = titulo.strip()
        if len(titulo) < 3:
            raise DatosNoValidos("El título debe tener al menos 3 caracteres")
        return self.post("/posts", {"userId": usuario_id, "title": titulo, "body": cuerpo})


# ------------------------------------------------------------------ Comandos
def cmd_resumen(cliente: ClienteAPI, args) -> int:
    inicio = time.perf_counter()
    usuarios = {u["id"]: u for u in cliente.usuarios()}
    publicaciones = cliente.publicaciones()
    por_usuario = Counter(p["userId"] for p in publicaciones)
    print(f"{len(usuarios)} usuarios · {len(publicaciones)} publicaciones "
          f"({time.perf_counter() - inicio:.2f} s)\n")
    print(f"{'Usuario':<28}{'Email':<32}{'Publicaciones':>14}")
    print("-" * 74)
    for uid, n in por_usuario.most_common(args.top):
        u = usuarios.get(uid, {"name": f"#{uid}", "email": "—"})
        print(f"{u['name']:<28}{u['email']:<32}{n:>14}")
    return 0


def cmd_exportar(cliente: ClienteAPI, args) -> int:
    salida = Path(args.salida)
    salida.mkdir(parents=True, exist_ok=True)
    publicaciones = cliente.publicaciones(args.usuario)
    if not publicaciones:
        print("No hay publicaciones que exportar.", file=sys.stderr)
        return 1
    ruta_json = salida / "publicaciones.json"
    ruta_json.write_text(json.dumps(publicaciones, ensure_ascii=False, indent=2), encoding="utf-8")
    ruta_csv = salida / "publicaciones.csv"
    with ruta_csv.open("w", newline="", encoding="utf-8-sig") as f:
        escritor = csv.DictWriter(f, fieldnames=["id", "userId", "title", "body"], delimiter=";")
        escritor.writeheader()
        escritor.writerows({k: p.get(k, "") for k in escritor.fieldnames} for p in publicaciones)
    print(f"✔ {len(publicaciones)} publicaciones exportadas a {ruta_json} y {ruta_csv}")
    return 0


def cmd_crear(cliente: ClienteAPI, args) -> int:
    cliente.usuario(args.usuario)                                  # Lanza NoEncontrado si no existe
    nueva = cliente.crear_publicacion(args.usuario, args.titulo, args.cuerpo)
    print(f"✔ Publicación creada con id {nueva.get('id')}: {nueva.get('title')}")
    return 0


def main(argv: list[str] | None = None) -> int:
    parser = argparse.ArgumentParser(description="Cliente de ejemplo para una API REST.")
    parser.add_argument("--url", default=os.environ.get("API_URL", "https://jsonplaceholder.typicode.com"))
    parser.add_argument("-v", "--verbose", action="store_true")
    sub = parser.add_subparsers(dest="comando", required=True)

    p = sub.add_parser("resumen", help="publicaciones por usuario")
    p.add_argument("--top", type=int, default=10)
    p.set_defaults(func=cmd_resumen)

    p = sub.add_parser("exportar", help="exporta publicaciones a JSON y CSV")
    p.add_argument("--salida", default="exportacion")
    p.add_argument("--usuario", type=int)
    p.set_defaults(func=cmd_exportar)

    p = sub.add_parser("crear", help="crea una publicación")
    p.add_argument("--usuario", type=int, required=True)
    p.add_argument("--titulo", required=True)
    p.add_argument("--cuerpo", default="")
    p.set_defaults(func=cmd_crear)

    args = parser.parse_args(argv)
    logging.basicConfig(level=logging.DEBUG if args.verbose else logging.WARNING,
                        format="%(levelname)s %(name)s: %(message)s")
    try:
        with ClienteAPI(args.url, token=os.environ.get("API_TOKEN")) as cliente:
            return args.func(cliente, args)
    except NoEncontrado as e:
        print(f"✖ {e}", file=sys.stderr)
        return 2
    except DatosNoValidos as e:
        print(f"✖ {e}: {e.detalle or ''}", file=sys.stderr)
        return 3
    except ErrorAPI as e:
        print(f"✖ {e}", file=sys.stderr)
        return 1


if __name__ == "__main__":
    sys.exit(main())
```

Pruebas con `responses` (`test_cliente_api.py`):

```python
import pytest
import responses

from cliente_api import ClienteAPI, DatosNoValidos, NoEncontrado

URL = "https://api.prueba"


@responses.activate
def test_paginacion_y_filtro():
    responses.get(f"{URL}/posts", json=[{"id": i, "userId": 1} for i in range(1, 51)],
                  match=[responses.matchers.query_param_matcher({"_page": "1", "_limit": "50", "userId": "1"})])
    responses.get(f"{URL}/posts", json=[{"id": 51, "userId": 1}],
                  match=[responses.matchers.query_param_matcher({"_page": "2", "_limit": "50", "userId": "1"})])
    with ClienteAPI(URL) as c:
        assert len(c.publicaciones(1)) == 51


@responses.activate
def test_errores():
    responses.get(f"{URL}/users/99", status=404)
    responses.post(f"{URL}/posts", status=422, json={"error": "title requerido"})
    with ClienteAPI(URL, reintentos=0) as c:
        with pytest.raises(NoEncontrado):
            c.usuario(99)
        with pytest.raises(DatosNoValidos) as exc:
            c.post("/posts", {})
        assert exc.value.detalle == {"error": "title requerido"}
        with pytest.raises(DatosNoValidos):
            c.crear_publicacion(1, "x", "")                    # Validación local, sin petición


@responses.activate
def test_reintento_tras_503():
    responses.get(f"{URL}/users", status=503)
    responses.get(f"{URL}/users", json=[{"id": 1}])
    with ClienteAPI(URL, reintentos=2) as c:
        c.sesion.adapters["https://"].max_retries.backoff_factor = 0      # Sin esperas en la prueba
        assert c.usuarios() == [{"id": 1}]
```

```bash
python cliente_api.py resumen --top 5
python cliente_api.py -v exportar --usuario 1
pip install pytest responses && pytest -q test_cliente_api.py
```

---

## 21. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| El programa se queda colgado | Petición sin `timeout`. | Indique siempre `timeout=`. |
| `requests.exceptions.SSLError: CERTIFICATE_VERIFY_FAILED` | CA desconocida (red corporativa, certificado propio). | `verify="ca.pem"` o `REQUESTS_CA_BUNDLE`; actualice `certifi`. |
| `JSONDecodeError: Expecting value` | La respuesta no es JSON (HTML de error, vacía). | Compruebe `status_code` y `Content-Type` antes de `.json()`. |
| Caracteres extraños en `r.text` | Codificación detectada incorrectamente. | `r.encoding = "utf-8"` o `r.apparent_encoding`. |
| `MissingSchema: Invalid URL` | Falta `http://` o `https://`. | Use la URL completa. |
| `ConnectionError: Max retries exceeded` | Servidor inaccesible, DNS o *proxy*. | Revise la URL, la red y `HTTPS_PROXY`. |
| `429 Too Many Requests` | Límite de peticiones de la API. | Respete `Retry-After`, reduzca la concurrencia, use reintentos con espera. |
| Se envía `data=dict` y la API esperaba JSON | `data=` envía formulario. | Use `json=`. |
| Lentitud en muchas peticiones al mismo servidor | Una conexión nueva por petición. | Use una `Session`. |
| Archivos grandes agotan la memoria | Descarga sin `stream=True`. | `stream=True` + `iter_content()`. |
| `ResourceWarning: unclosed socket` | Respuestas con `stream=True` sin cerrar. | Use `with requests.get(..., stream=True) as r:`. |

---

## 22. Buenas prácticas

- Indique **siempre** un `timeout`.
- Use una `Session` para varias peticiones al mismo servicio y ciérrela al terminar (`with`).
- Llame a `raise_for_status()` o compruebe `status_code` antes de usar el contenido.
- Configure reintentos con espera exponencial solo para métodos idempotentes.
- Envíe un `User-Agent` descriptivo y respete los límites y condiciones de uso de cada API.
- Guarde credenciales en variables de entorno o gestores de secretos.
- Use `params=` y `json=` en lugar de construir URLs y cuerpos a mano.
- Descargue archivos grandes con `stream=True` y escriba primero en un archivo temporal.
- Encapsule cada API en una clase cliente con excepciones propias.
- Simule las respuestas en las pruebas (`responses`, `requests-mock`).

---

## 23. Referencia rápida

```python
requests.get/post/put/patch/delete/head(url, params=, json=, data=, files=, headers=, auth=, cookies=,
                                        timeout=(3.05, 15), allow_redirects=, stream=, verify=, proxies=)
r.status_code · r.ok · r.raise_for_status() · r.json() · r.text · r.content · r.headers · r.url · r.links · r.elapsed
with requests.Session() as s: s.headers.update({...}); s.auth = ...; s.get(...)
s.mount("https://", HTTPAdapter(max_retries=Retry(total=5, backoff_factor=0.5, status_forcelist=[429, 500, 502, 503, 504])))
auth=("u", "c") · HTTPDigestAuth · headers={"Authorization": f"Bearer {t}"} · class X(AuthBase)
files={"f": ("nombre.pdf", open(...,"rb"), "application/pdf")} · stream=True + r.iter_content(65536) / r.iter_lines()
RequestException > ConnectionError · Timeout · HTTPError · TooManyRedirects · JSONDecodeError
s.hooks["response"].append(f) · requests.Request(...).prepare() · responses (pruebas)
```

---

## 24. Ejercicios propuestos

1. Escriba una función que compruebe una lista de URLs y muestre el código de estado y el tiempo de respuesta de cada una, en paralelo con hilos.
2. Consulte la API pública de GitHub para listar los repositorios de un usuario recorriendo todas las páginas con la cabecera `Link`.
3. Cree un descargador que reanude descargas interrumpidas con la cabecera `Range` y muestre el progreso con tqdm.
4. Amplíe el ejemplo completo con un comando `comentarios` que obtenga los comentarios de cada publicación y calcule el usuario con más interacción.
5. Escriba pruebas con `responses` que cubran *timeouts*, errores 500 con reintentos y respuestas no JSON.

---

## 25. Referencias

- Documentación oficial: https://requests.readthedocs.io/
- Repositorio: https://github.com/psf/requests
- urllib3 (reintentos): https://urllib3.readthedocs.io/en/stable/reference/urllib3.util.html
- responses (pruebas): https://github.com/getsentry/responses
- JSONPlaceholder: https://jsonplaceholder.typicode.com/
