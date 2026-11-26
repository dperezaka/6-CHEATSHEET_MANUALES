# Manual de uso: Scrapy

> **Categoría:** Desarrollo web · **Utilidad:** *Web scraping* y rastreo (*crawling*) avanzado
> **Tipo:** Librería de terceros (`pip install scrapy`) · **Versión de referencia:** 2.13 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y creación del proyecto](#2-instalación-y-creación-del-proyecto)
3. [Conceptos clave](#3-conceptos-clave)
4. [Arquitectura y flujo de datos](#4-arquitectura-y-flujo-de-datos)
5. [Primera araña (*spider*)](#5-primera-araña-spider)
6. [Selectores CSS y XPath](#6-selectores-css-y-xpath)
7. [Seguir enlaces y paginación](#7-seguir-enlaces-y-paginación)
8. [Peticiones: `Request`, `start()` y paso de datos](#8-peticiones-request-start-y-paso-de-datos)
9. [*Items* e *ItemLoaders*](#9-items-e-itemloaders)
10. [*Pipelines*](#10-pipelines)
11. [Exportación de resultados (*feeds*)](#11-exportación-de-resultados-feeds)
12. [Configuración (`settings.py`)](#12-configuración-settingspy)
13. [`CrawlSpider` y `LinkExtractor`](#13-crawlspider-y-linkextractor)
14. [Formularios, inicio de sesión y APIs JSON](#14-formularios-inicio-de-sesión-y-apis-json)
15. [*Middlewares*](#15-middlewares)
16. [La consola `scrapy shell`](#16-la-consola-scrapy-shell)
17. [Ejecutar desde un script](#17-ejecutar-desde-un-script)
18. [Páginas con JavaScript](#18-páginas-con-javascript)
19. [Pruebas y contratos](#19-pruebas-y-contratos)
20. [Despliegue](#20-despliegue)
21. [Ejemplo práctico completo](#21-ejemplo-práctico-completo)
22. [Errores frecuentes](#22-errores-frecuentes)
23. [Buenas prácticas](#23-buenas-prácticas)
24. [Referencia rápida](#24-referencia-rápida)
25. [Ejercicios propuestos](#25-ejercicios-propuestos)
26. [Referencias](#26-referencias)

---

## 1. Descripción

**Scrapy** es un *framework* completo para **rastrear sitios web y extraer datos estructurados** a gran escala. A diferencia de combinar Requests y BeautifulSoup a mano, Scrapy gestiona por sí mismo la cola de peticiones, la concurrencia asíncrona, los reintentos, la eliminación de duplicados, el respeto de `robots.txt`, los límites de velocidad, la caché, las cookies y la exportación a JSON, CSV o XML. El código se organiza en **arañas** (*spiders*), **items**, ***pipelines*** y ***middlewares***.

**Ventajas:**
- Muy rápido: decenas de peticiones simultáneas sobre un motor asíncrono (Twisted/asyncio).
- Gestión integrada de colas, duplicados, reintentos, redirecciones, cookies y compresión.
- Selectores CSS y XPath potentes.
- Exportación directa a JSON, JSON Lines, CSV y XML, en local o en la nube.
- Muy extensible: *pipelines*, *middlewares*, extensiones, señales.
- Control de cortesía: `robots.txt`, retardos, *AutoThrottle*, caché HTTP.

**Limitaciones:**
- Curva de aprendizaje mayor que Requests + BeautifulSoup.
- No ejecuta JavaScript por sí mismo (se integra con Playwright mediante `scrapy-playwright`).
- Modelo asíncrono basado en *callbacks*: requiere adaptar la forma de programar.
- Excesivo para extraer datos de una o dos páginas.

| Si necesita… | Opción |
|---|---|
| Rastrear muchos sitios o miles de páginas de forma robusta | **Scrapy** |
| Extraer datos de pocas páginas estáticas | Requests/httpx + BeautifulSoup4 |
| Páginas que dependen de JavaScript | Playwright (o `scrapy-playwright`) |
| Automatizar interacciones en un navegador | Playwright / Selenium |

---

## 2. Instalación y creación del proyecto

```bash
python -m venv .venv
source .venv/bin/activate                # Windows: .venv\Scripts\activate
pip install scrapy

scrapy startproject librerias            # Crea el proyecto
cd librerias
scrapy genspider libros books.toscrape.com   # Crea una araña a partir de una plantilla
scrapy list                              # Arañas disponibles
scrapy crawl libros -O libros.json       # Ejecutar y exportar (sobrescribe)
```

Estructura generada:

```
librerias/
├── scrapy.cfg              # Configuración de despliegue
└── librerias/
    ├── __init__.py
    ├── items.py            # Definición de los datos
    ├── middlewares.py      # Middlewares de descarga y de araña
    ├── pipelines.py        # Procesamiento de items
    ├── settings.py         # Configuración
    └── spiders/
        └── libros.py       # Arañas
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Araña (*Spider*) | Clase que define dónde empezar, cómo seguir enlaces y cómo extraer datos. |
| `Request` | Petición a una URL con un *callback* que procesará la respuesta. |
| `Response` | Respuesta descargada, con selectores `css()` y `xpath()`. |
| *Callback* | Método que recibe la respuesta y produce (`yield`) items y nuevas peticiones. |
| *Item* | Registro de datos extraído (diccionario, `dataclass` o `scrapy.Item`). |
| *ItemLoader* | Ayuda a rellenar items aplicando procesadores de limpieza. |
| *Pipeline* | Etapa que procesa cada item: limpiar, validar, eliminar duplicados, guardar. |
| *Feed* | Exportación automática de los items (JSON, CSV…). |
| *Middleware* | Componente que intercepta peticiones/respuestas (cabeceras, *proxies*, reintentos). |
| *Scheduler* | Cola de peticiones pendientes (con eliminación de duplicados). |
| *Engine* | Motor que coordina todos los componentes. |

---

## 4. Arquitectura y flujo de datos

```
            ┌──────────── Engine ────────────┐
 Spider ──► │ Request → Scheduler → Downloader│ ──► Internet
   ▲        │      (middlewares de descarga)  │
   │        └─────────────────────────────────┘
   │ Response                       │
   └────────── (middlewares de araña)┘
 Spider ──► Items ──► Item Pipelines ──► Feeds / base de datos
```

1. La araña genera las peticiones iniciales.
2. El *scheduler* las encola (descartando duplicadas).
3. El *downloader* las descarga respetando concurrencia, retardos y `robots.txt`.
4. La respuesta llega al *callback* de la araña, que produce items y nuevas peticiones.
5. Los items pasan por los *pipelines* y se exportan.

---

## 5. Primera araña (*spider*)

```python
# librerias/spiders/citas.py
import scrapy


class CitasSpider(scrapy.Spider):
    name = "citas"                                        # Identificador único
    allowed_domains = ["quotes.toscrape.com"]             # No sale de estos dominios
    start_urls = ["https://quotes.toscrape.com/"]         # Páginas iniciales

    def parse(self, response):                            # Callback por defecto
        for cita in response.css("div.quote"):
            yield {
                "texto": cita.css("span.text::text").get(),
                "autor": cita.css("small.author::text").get(),
                "etiquetas": cita.css("div.tags a.tag::text").getall(),
            }
        siguiente = response.css("li.next a::attr(href)").get()
        if siguiente:
            yield response.follow(siguiente, callback=self.parse)
```

```bash
scrapy crawl citas -O citas.json          # -O sobrescribe, -o añade
scrapy crawl citas -O citas.csv
scrapy crawl citas -O citas.jsonl -s LOG_LEVEL=INFO
scrapy runspider citas.py -O citas.json   # Ejecutar un archivo suelto sin proyecto
```

---

## 6. Selectores CSS y XPath

### 6.1 CSS

```python
response.css("h1::text").get()                         # Primer resultado (o None)
response.css("h1::text").get(default="")               # Valor por defecto
response.css("ul li::text").getall()                   # Lista de todos
response.css("a::attr(href)").getall()                 # Atributos
response.css("a").attrib["href"]                       # Atributo del primer elemento
response.css("div.precio *::text").getall()            # Texto de todos los descendientes
response.css("p.star-rating").xpath("@class").get()    # Combinar CSS y XPath
response.css("span.precio::text").re_first(r"[\d.,]+") # Expresión regular
```

### 6.2 XPath

```python
response.xpath("//h1/text()").get()
response.xpath("//a/@href").getall()
response.xpath("//div[@class='producto']").getall()
response.xpath("//th[text()='UPC']/following-sibling::td/text()").get()   # Celda junto a un texto
response.xpath("//p[contains(@class, 'star-rating')]/@class").get()
response.xpath("normalize-space(//div[@id='descripcion'])").get()          # Texto sin espacios sobrantes
response.xpath("//li[position() <= 3]/text()").getall()
```

| Necesidad | CSS | XPath |
|---|---|---|
| Texto del elemento | `h1::text` | `//h1/text()` |
| Atributo | `a::attr(href)` | `//a/@href` |
| Por clase | `div.producto` | `//div[contains(@class,'producto')]` |
| Por texto contenido | — | `//a[contains(text(),'Siguiente')]` |
| Hermano siguiente | `h2 + p` | `//h2/following-sibling::p[1]` |
| Padre | — | `//span/..` |

### 6.3 Selectores relativos

Dentro de un bucle, use selectores **relativos** al elemento:

```python
for producto in response.css("article.product_pod"):
    titulo = producto.css("h3 a::attr(title)").get()        # CSS: relativo automáticamente
    precio = producto.xpath(".//p[@class='price_color']/text()").get()   # XPath: empiece por './/'
```

> En XPath, `//p` busca en **todo el documento** aunque se aplique sobre un subelemento. Use `.//p` para buscar dentro del elemento actual.

---

## 7. Seguir enlaces y paginación

```python
def parse(self, response):
    # Un enlace (acepta URLs relativas o selectores <a>)
    yield response.follow("pagina-2.html", callback=self.parse)

    # Todos los enlaces que coincidan con un selector
    yield from response.follow_all(css="article.product_pod h3 a", callback=self.parse_libro)

    # Paginación
    if siguiente := response.css("li.next a::attr(href)").get():
        yield response.follow(siguiente, callback=self.parse)

def parse_libro(self, response):
    yield {"titulo": response.css("h1::text").get(), "url": response.url}
```

> Scrapy **descarta automáticamente las URLs repetidas** (filtro de duplicados). Para forzar una petición repetida: `dont_filter=True`.

---

## 8. Peticiones: `Request`, `start()` y paso de datos

### 8.1 Peticiones iniciales con `start()`

En las versiones actuales de Scrapy (2.13+), las peticiones iniciales personalizadas se generan con el método **asíncrono** `start()`:

```python
class ProductosSpider(scrapy.Spider):
    name = "productos"

    async def start(self):
        for categoria in ["teclados", "ratones", "monitores"]:
            yield scrapy.Request(f"https://tienda.ejemplo.com/{categoria}",
                                 callback=self.parse, cb_kwargs={"categoria": categoria})

    def parse(self, response, categoria):
        ...
```

> En versiones anteriores a la 2.13 se usaba `def start_requests(self)`. En las versiones recientes ese método **ya no se invoca**: si mantiene código antiguo, renómbrelo a `async def start(self)`. Si solo necesita una lista fija de URLs, basta con `start_urls`.

### 8.2 Parámetros de `Request`

```python
scrapy.Request(
    url,
    callback=self.parse_detalle,
    method="GET",
    headers={"Accept-Language": "es-ES"},
    cookies={"moneda": "EUR"},
    cb_kwargs={"categoria": "teclados"},       # Argumentos para el callback
    meta={"profundidad_manual": 2},            # Datos para middlewares
    priority=10,                               # Mayor prioridad = antes
    dont_filter=False,
    errback=self.error,                        # Función ante errores
)
```

### 8.3 Pasar datos entre *callbacks*

```python
def parse(self, response):
    for p in response.css("article.producto"):
        datos = {"nombre": p.css("h2::text").get(), "precio": p.css(".precio::text").get()}
        yield response.follow(p.css("a::attr(href)").get(), callback=self.parse_ficha,
                              cb_kwargs={"datos": datos})

def parse_ficha(self, response, datos):
    datos["descripcion"] = response.css("#descripcion::text").get()
    yield datos
```

### 8.4 Gestionar errores

```python
from scrapy.spidermiddlewares.httperror import HttpError
from twisted.internet.error import DNSLookupError, TimeoutError

def error(self, failure):
    if failure.check(HttpError):
        self.logger.warning("HTTP %s en %s", failure.value.response.status, failure.value.response.url)
    elif failure.check(DNSLookupError, TimeoutError):
        self.logger.error("Error de red en %s", failure.request.url)
```

---

## 9. *Items* e *ItemLoaders*

### 9.1 *Items* con `dataclass` (recomendado)

```python
# items.py
from dataclasses import dataclass, field

@dataclass
class LibroItem:
    titulo: str
    precio: float
    disponible: int = 0
    valoracion: int = 0
    categoria: str = ""
    url: str = ""
    etiquetas: list[str] = field(default_factory=list)
```

También se admiten diccionarios, `attrs` o la clase clásica `scrapy.Item`:

```python
class LibroItem(scrapy.Item):
    titulo = scrapy.Field()
    precio = scrapy.Field()
```

### 9.2 *ItemLoader*: extracción con procesadores

```python
from itemloaders.processors import TakeFirst, MapCompose, Join
from scrapy.loader import ItemLoader

def a_precio(texto: str) -> float:
    return float(texto.replace("£", "").replace("€", "").replace(",", ".").strip())

class LibroLoader(ItemLoader):
    default_output_processor = TakeFirst()           # Un único valor por campo
    titulo_in = MapCompose(str.strip)
    precio_in = MapCompose(a_precio)
    etiquetas_out = lambda self, valores: valores    # Mantener lista

def parse_libro(self, response):
    l = LibroLoader(item={}, response=response)
    l.add_css("titulo", "h1::text")
    l.add_css("precio", "p.price_color::text")
    l.add_xpath("categoria", "//ul[@class='breadcrumb']/li[3]/a/text()")
    l.add_value("url", response.url)
    yield l.load_item()
```

| Procesador | Efecto |
|---|---|
| `TakeFirst()` | Primer valor no vacío. |
| `MapCompose(f1, f2)` | Aplica funciones a cada valor. |
| `Join(" ")` | Une los valores. |
| `Identity()` | Deja la lista tal cual. |

---

## 10. *Pipelines*

Cada *item* producido pasa por los *pipelines* activos, en orden de prioridad (número menor primero).

> En las versiones recientes de Scrapy, los métodos de los *pipelines* ya **no reciben** el argumento `spider` (`process_item(self, item)`, `open_spider(self)`, `close_spider(self)`). Si necesita la araña, la configuración o las estadísticas, guarde el `crawler` en `from_crawler()` y use `self.crawler.spider`, `self.crawler.settings` o `self.crawler.stats`. En versiones anteriores la firma era `process_item(self, item, spider)`; Scrapy sigue aceptándola con un aviso de obsolescencia.

```python
# pipelines.py
import sqlite3
from itemadapter import ItemAdapter
from scrapy.exceptions import DropItem


class ValidacionPipeline:
    def process_item(self, item):
        a = ItemAdapter(item)                         # Interfaz común para dict, dataclass, Item…
        if not a.get("titulo"):
            raise DropItem("Item sin título")
        if a.get("precio") is None or a["precio"] <= 0:
            raise DropItem(f"Precio no válido en {a.get('url')}")
        a["titulo"] = a["titulo"].strip()
        return item


class DuplicadosPipeline:
    def __init__(self):
        self.vistos: set[str] = set()

    def process_item(self, item):
        clave = ItemAdapter(item).get("upc") or ItemAdapter(item).get("url")
        if clave in self.vistos:
            raise DropItem(f"Duplicado: {clave}")
        self.vistos.add(clave)
        return item


class SQLitePipeline:
    def __init__(self, ruta: str):
        self.ruta = ruta

    @classmethod
    def from_crawler(cls, crawler):                    # Acceso a la configuración
        return cls(crawler.settings.get("SQLITE_RUTA", "datos.db"))

    def open_spider(self):
        self.conn = sqlite3.connect(self.ruta)
        self.conn.execute("CREATE TABLE IF NOT EXISTS libros (upc TEXT PRIMARY KEY, titulo TEXT, precio REAL)")

    def close_spider(self):
        self.conn.commit()
        self.conn.close()

    def process_item(self, item):
        a = ItemAdapter(item)
        self.conn.execute("INSERT OR REPLACE INTO libros VALUES (?, ?, ?)", (a["upc"], a["titulo"], a["precio"]))
        return item
```

```python
# settings.py
ITEM_PIPELINES = {
    "librerias.pipelines.ValidacionPipeline": 100,
    "librerias.pipelines.DuplicadosPipeline": 200,
    "librerias.pipelines.SQLitePipeline": 300,
}
SQLITE_RUTA = "libros.db"
```

Descarga de imágenes y archivos: `ImagesPipeline` y `FilesPipeline` integrados (`IMAGES_STORE`, campo `image_urls`).

---

## 11. Exportación de resultados (*feeds*)

Desde la línea de comandos:

```bash
scrapy crawl libros -O libros.json        # Sobrescribe
scrapy crawl libros -o libros.jsonl       # Añade (JSON Lines admite añadir sin romper el formato)
scrapy crawl libros -O libros.csv
```

En la configuración (más control):

```python
FEEDS = {
    "salida/libros_%(time)s.jsonl": {"format": "jsonlines", "encoding": "utf8"},
    "salida/libros.csv": {
        "format": "csv",
        "fields": ["titulo", "precio", "categoria", "valoracion"],
        "mode": "overwrite",                 # "append" para añadir
        "item_export_kwargs": {"delimiter": ";"},
    },
    "s3://mi-bucket/libros/%(name)s.json": {"format": "json"},   # Requiere botocore
}
FEED_EXPORT_ENCODING = "utf-8"
```

| Formato | Uso |
|---|---|
| `json` | Una lista JSON (no admite añadir). |
| `jsonlines` | Un objeto por línea: ideal para grandes volúmenes y añadir. |
| `csv` | Hojas de cálculo. |
| `xml` | Integraciones heredadas. |

---

## 12. Configuración (`settings.py`)

```python
BOT_NAME = "librerias"
USER_AGENT = "LibreriasBot/1.0 (+https://ejemplo.com/contacto)"

# Cortesía
ROBOTSTXT_OBEY = True                       # El proyecto generado lo activa por defecto
DOWNLOAD_DELAY = 0.5                        # Segundos entre peticiones al mismo dominio
RANDOMIZE_DOWNLOAD_DELAY = True             # Variación aleatoria (0.5×–1.5×)
CONCURRENT_REQUESTS = 16
CONCURRENT_REQUESTS_PER_DOMAIN = 4
AUTOTHROTTLE_ENABLED = True                 # Ajusta la velocidad según la latencia del servidor
AUTOTHROTTLE_TARGET_CONCURRENCY = 2.0

# Robustez
RETRY_ENABLED = True
RETRY_TIMES = 3
RETRY_HTTP_CODES = [500, 502, 503, 504, 522, 524, 408, 429]
DOWNLOAD_TIMEOUT = 30

# Desarrollo: caché HTTP (no vuelve a descargar páginas ya obtenidas)
HTTPCACHE_ENABLED = True
HTTPCACHE_EXPIRATION_SECS = 86400
HTTPCACHE_DIR = "httpcache"

# Límites
DEPTH_LIMIT = 5
CLOSESPIDER_ITEMCOUNT = 1000                # Detener tras N items
CLOSESPIDER_TIMEOUT = 3600

LOG_LEVEL = "INFO"
FEED_EXPORT_ENCODING = "utf-8"
```

Configuración por araña o en la línea de comandos:

```python
class LibrosSpider(scrapy.Spider):
    custom_settings = {"DOWNLOAD_DELAY": 1, "CLOSESPIDER_PAGECOUNT": 50}
```

```bash
scrapy crawl libros -s DOWNLOAD_DELAY=2 -s LOG_LEVEL=DEBUG
scrapy crawl libros -a categoria=poesia      # Argumento → self.categoria
```

---

## 13. `CrawlSpider` y `LinkExtractor`

Para rastrear un sitio siguiendo enlaces según **reglas**:

```python
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule


class CatalogoSpider(CrawlSpider):
    name = "catalogo"
    allowed_domains = ["books.toscrape.com"]
    start_urls = ["https://books.toscrape.com/"]

    rules = (
        Rule(LinkExtractor(restrict_css="ul.nav-list")),                       # Categorías: solo seguir
        Rule(LinkExtractor(restrict_css="li.next")),                           # Paginación: seguir
        Rule(LinkExtractor(restrict_css="article.product_pod h3"), callback="parse_libro"),  # Fichas: extraer
    )

    def parse_libro(self, response):
        yield {"titulo": response.css("h1::text").get(), "url": response.url}
```

| Parámetro de `LinkExtractor` | Uso |
|---|---|
| `allow` / `deny` | Expresiones regulares sobre la URL. |
| `restrict_css` / `restrict_xpaths` | Zonas de la página donde buscar enlaces. |
| `allow_domains` / `deny_domains` | Dominios. |
| `deny_extensions` | Extensiones ignoradas (por defecto, imágenes, PDF…). |

> En un `CrawlSpider` **no** sobrescriba el método `parse`: lo usa internamente.

---

## 14. Formularios, inicio de sesión y APIs JSON

### 14.1 Enviar formularios

```python
from scrapy import FormRequest

class AccesoSpider(scrapy.Spider):
    name = "acceso"
    start_urls = ["https://quotes.toscrape.com/login"]

    def parse(self, response):
        return FormRequest.from_response(              # Toma los campos ocultos (token CSRF) del formulario
            response,
            formdata={"username": self.usuario, "password": self.clave},
            callback=self.tras_login,
        )

    def tras_login(self, response):
        if "Logout" not in response.text:
            self.logger.error("Inicio de sesión fallido")
            return
        yield response.follow("/", callback=self.parse_privado)   # Las cookies se mantienen
```

### 14.2 APIs JSON

Muchas páginas dinámicas obtienen sus datos de una API JSON (visible en la pestaña **Red** del navegador). Consumirla directamente es más rápido y estable que analizar HTML:

```python
class ApiSpider(scrapy.Spider):
    name = "api"

    async def start(self):
        yield scrapy.Request("https://api.ejemplo.com/productos?page=1", headers={"Accept": "application/json"})

    def parse(self, response):
        datos = response.json()
        yield from datos["resultados"]
        if datos.get("siguiente"):
            yield response.follow(datos["siguiente"], callback=self.parse)

# Petición con cuerpo JSON
scrapy.http.JsonRequest(url, data={"q": "teclado", "page": 1}, callback=self.parse)
```

---

## 15. *Middlewares*

```python
# middlewares.py
import random

class AgenteRotatorioMiddleware:
    AGENTES = ["MiBot/1.0 (+contacto)", "MiBot/1.0 (+contacto; nodo-2)"]

    def process_request(self, request):
        request.headers["User-Agent"] = random.choice(self.AGENTES)
        return None                                   # Continuar con la petición


class EstadisticasMiddleware:
    @classmethod
    def from_crawler(cls, crawler):
        m = cls()
        m.stats = crawler.stats
        return m

    def process_response(self, request, response):
        self.stats.inc_value(f"mis_respuestas/{response.status}")
        return response
```

```python
DOWNLOADER_MIDDLEWARES = {
    "librerias.middlewares.AgenteRotatorioMiddleware": 400,
    "librerias.middlewares.EstadisticasMiddleware": 950,
}
```

| Método (*downloader middleware*) | Momento |
|---|---|
| `process_request(request)` | Antes de descargar (devolver `None`, una `Response` o un `Request`). |
| `process_response(request, response)` | Tras descargar. |
| `process_exception(request, exception)` | Ante errores de descarga. |

> Como en los *pipelines*, las versiones anteriores incluían un último argumento `spider` en estos métodos; en las versiones recientes está obsoleto.

---

## 16. La consola `scrapy shell`

Herramienta imprescindible para probar selectores antes de escribir la araña:

```bash
scrapy shell "https://books.toscrape.com/"
```

```python
>>> response.status
200
>>> response.css("article.product_pod h3 a::attr(title)").getall()[:3]
['A Light in the Attic', 'Tipping the Velvet', 'Soumission']
>>> fetch("https://books.toscrape.com/catalogue/page-2.html")     # Descargar otra página
>>> view(response)                                                 # Abrirla en el navegador
>>> from scrapy.shell import inspect_response                      # Desde una araña: abrir la consola en un punto
```

Otros comandos útiles:

```bash
scrapy fetch --nolog https://ejemplo.com > pagina.html     # Descargar como lo haría Scrapy
scrapy view https://ejemplo.com                            # Ver lo que «ve» Scrapy (sin JavaScript)
scrapy parse --spider=libros -c parse_libro URL            # Probar un callback con una URL
scrapy check                                               # Ejecutar contratos
scrapy bench                                               # Prueba de rendimiento
```

---

## 17. Ejecutar desde un script

```python
from scrapy.crawler import CrawlerProcess
from scrapy.utils.project import get_project_settings

proceso = CrawlerProcess(get_project_settings())   # O un diccionario de ajustes
proceso.crawl("libros", categoria="poesia")         # Por nombre (dentro de un proyecto) o por clase
proceso.start()                                     # Bloquea hasta terminar
```

Sin proyecto:

```python
proceso = CrawlerProcess({"FEEDS": {"citas.json": {"format": "json", "mode": "overwrite"}},
                          "LOG_LEVEL": "WARNING"})
proceso.crawl(CitasSpider)
proceso.start()
```

> `CrawlerProcess.start()` solo puede llamarse **una vez** por proceso de Python (el *reactor* de Twisted no se puede reiniciar). Para lanzar varias arañas, añádalas todas con `crawl()` antes de `start()`.

---

## 18. Páginas con JavaScript

Scrapy descarga el HTML tal como lo envía el servidor. Si los datos se cargan con JavaScript:

1. **Busque la API** que usa la página (pestaña Red → XHR/Fetch) y consúmala directamente (sección 14.2).
2. Busque los datos incrustados en el HTML: `<script type="application/ld+json">`, `window.__INITIAL_STATE__ = {...}`.
3. Use **scrapy-playwright** para renderizar las páginas en un navegador real:

```bash
pip install scrapy-playwright
playwright install chromium
```

```python
DOWNLOAD_HANDLERS = {
    "http": "scrapy_playwright.handler.ScrapyPlaywrightDownloadHandler",
    "https": "scrapy_playwright.handler.ScrapyPlaywrightDownloadHandler",
}

yield scrapy.Request(url, meta={"playwright": True})        # Solo las peticiones que lo necesiten
```

---

## 19. Pruebas y contratos

### 19.1 Probar *callbacks* con respuestas locales

```python
from pathlib import Path
from scrapy.http import HtmlResponse, Request
from librerias.spiders.libros import LibrosSpider

def respuesta_falsa(ruta: str, url: str) -> HtmlResponse:
    return HtmlResponse(url=url, body=Path(ruta).read_bytes(), encoding="utf-8", request=Request(url))

def test_parse_libro():
    araña = LibrosSpider()
    items = list(araña.parse_libro(respuesta_falsa("tests/ficha.html", "https://books.toscrape.com/x")))
    assert items[0]["titulo"] == "A Light in the Attic"
```

### 19.2 Contratos

```python
def parse_libro(self, response):
    """Extrae la ficha de un libro.

    @url https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html
    @returns items 1 1
    @scrapes titulo precio upc
    """
```

```bash
scrapy check libros
```

---

## 20. Despliegue

- **Scrapyd**: servidor para desplegar y programar arañas mediante una API HTTP (`pip install scrapyd scrapyd-client`, `scrapyd-deploy`).
- **Zyte Scrapy Cloud**: plataforma gestionada.
- **Cron / tareas programadas** o contenedores Docker que ejecuten `scrapy crawl`.
- **Persistencia entre ejecuciones** (reanudar un rastreo largo):

```bash
scrapy crawl libros -s JOBDIR=trabajos/libros-1     # Ctrl+C para pausar; repetir el comando para reanudar
```

---

## 21. Ejemplo práctico completo

**Rastreador de catálogo de libros** en un único archivo ejecutable: araña con `start()` asíncrono, paginación, entrada en cada ficha con paso de datos entre *callbacks*, *item* `dataclass`, *pipelines* de normalización, validación, duplicados y almacenamiento en SQLite, exportación simultánea a JSON Lines y CSV, cortesía (`robots.txt`, retardos, *AutoThrottle*), caché HTTP opcional, gestión de errores con `errback`, estadísticas propias y resumen final. Está preparado para **books.toscrape.com**, un sitio creado para practicar *scraping*.

```python
"""
rastreador_libros.py — Rastreador de catálogo con Scrapy (sin proyecto).
Requiere: pip install scrapy
Uso:
    python rastreador_libros.py --paginas 3
    python rastreador_libros.py --url https://books.toscrape.com/catalogue/page-1.html --cache
"""
import argparse
import re
import sqlite3
from dataclasses import dataclass, field
from decimal import Decimal
from pathlib import Path

import scrapy
from itemadapter import ItemAdapter
from scrapy.crawler import CrawlerProcess
from scrapy.exceptions import DropItem, IgnoreRequest
from scrapy.spidermiddlewares.httperror import HttpError

VALORACIONES = {"One": 1, "Two": 2, "Three": 3, "Four": 4, "Five": 5}


# ------------------------------------------------------------------ Item
@dataclass
class Libro:
    titulo: str = ""
    precio: str = ""                    # Texto bruto; se normaliza en un pipeline
    moneda: str = ""
    disponible: int = 0
    valoracion: int = 0
    categoria: str = ""
    upc: str = ""
    url: str = ""
    pagina_listado: int = 0
    etiquetas: list[str] = field(default_factory=list)


# ------------------------------------------------------------------ Araña
class LibrosSpider(scrapy.Spider):
    name = "libros"

    def __init__(self, url_inicial: str, max_paginas: int = 0, **kwargs):
        super().__init__(**kwargs)
        self.url_inicial = url_inicial
        self.max_paginas = int(max_paginas)

    async def start(self):
        yield scrapy.Request(self.url_inicial, callback=self.parse_listado, errback=self.error,
                             cb_kwargs={"pagina": 1})

    def parse_listado(self, response, pagina: int):
        fichas = response.css("article.product_pod")
        self.logger.info("Listado página %d: %d libros", pagina, len(fichas))
        self.crawler.stats.inc_value("libros/paginas_listado")
        for ficha in fichas:
            datos = {"titulo": ficha.css("h3 a::attr(title)").get(default="").strip(),
                     "pagina_listado": pagina}
            yield response.follow(ficha.css("h3 a::attr(href)").get(), callback=self.parse_libro,
                                  errback=self.error, cb_kwargs={"datos": datos})

        siguiente = response.css("li.next a::attr(href)").get()
        if siguiente and (not self.max_paginas or pagina < self.max_paginas):
            yield response.follow(siguiente, callback=self.parse_listado, errback=self.error,
                                  cb_kwargs={"pagina": pagina + 1})

    def parse_libro(self, response, datos: dict):
        principal = response.css("div.product_main")
        tabla = {fila.css("th::text").get(): fila.css("td::text").get()
                 for fila in response.css("table.table-striped tr")}
        clases_estrellas = principal.css("p.star-rating::attr(class)").get(default="").split()
        disponibilidad = " ".join(principal.css("p.availability ::text").getall())
        stock = re.search(r"(\d+) available", disponibilidad)

        yield Libro(
            titulo=principal.css("h1::text").get(default=datos["titulo"]).strip(),
            precio=principal.css("p.price_color::text").get(default=""),
            disponible=int(stock.group(1)) if stock else 0,
            valoracion=next((VALORACIONES[c] for c in clases_estrellas if c in VALORACIONES), 0),
            categoria=response.xpath("//ul[@class='breadcrumb']/li[3]/a/text()").get(default="").strip(),
            upc=tabla.get("UPC", ""),
            url=response.url,
            pagina_listado=datos["pagina_listado"],
        )

    def error(self, failure):
        if failure.check(IgnoreRequest):                   # p. ej., prohibido por robots.txt
            self.logger.info("Omitida %s: %s", failure.request.url, failure.value)
            return
        self.crawler.stats.inc_value("libros/errores")
        if failure.check(HttpError):
            r = failure.value.response
            self.logger.warning("HTTP %s en %s", r.status, r.url)
        else:
            self.logger.error("Error en %s: %r", failure.request.url, failure.value)


# ------------------------------------------------------------------ Pipelines
class NormalizarPipeline:
    """Convierte el precio a Decimal y separa la moneda; añade etiquetas derivadas."""

    def process_item(self, item):
        a = ItemAdapter(item)
        m = re.search(r"([£$€])\s*([\d.,]+)", a["precio"] or "")
        if not m:
            raise DropItem(f"Precio no reconocido en {a['url']}: {a['precio']!r}")
        a["moneda"], a["precio"] = m.group(1), str(Decimal(m.group(2).replace(",", "")))
        etiquetas = []
        if a["valoracion"] >= 4:
            etiquetas.append("recomendado")
        if a["disponible"] == 0:
            etiquetas.append("agotado")
        elif a["disponible"] < 5:
            etiquetas.append("ultimas-unidades")
        if Decimal(a["precio"]) < 20:
            etiquetas.append("economico")
        a["etiquetas"] = etiquetas
        return item


class ValidarPipeline:
    def __init__(self, stats):
        self.stats = stats

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.stats)

    def process_item(self, item):
        a = ItemAdapter(item)
        faltan = [c for c in ("titulo", "upc", "categoria") if not a.get(c)]
        if faltan:
            self.stats.inc_value("libros/descartados")
            raise DropItem(f"Faltan campos {faltan} en {a['url']}")
        return item


class DuplicadosPipeline:
    def __init__(self):
        self.vistos: set[str] = set()

    def process_item(self, item):
        upc = ItemAdapter(item)["upc"]
        if upc in self.vistos:
            raise DropItem(f"Libro duplicado: {upc}")
        self.vistos.add(upc)
        return item


class SQLitePipeline:
    def __init__(self, ruta: str, stats):
        self.ruta, self.stats = ruta, stats

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.settings.get("SQLITE_RUTA", "libros.db"), crawler.stats)

    def open_spider(self):
        self.conn = sqlite3.connect(self.ruta)
        self.conn.execute("""CREATE TABLE IF NOT EXISTS libros (
            upc TEXT PRIMARY KEY, titulo TEXT, precio REAL, moneda TEXT, disponible INTEGER,
            valoracion INTEGER, categoria TEXT, url TEXT, actualizado TEXT DEFAULT CURRENT_TIMESTAMP)""")

    def process_item(self, item):
        a = ItemAdapter(item)
        self.conn.execute(
            "INSERT OR REPLACE INTO libros (upc, titulo, precio, moneda, disponible, valoracion, categoria, url) "
            "VALUES (?, ?, ?, ?, ?, ?, ?, ?)",
            (a["upc"], a["titulo"], float(a["precio"]), a["moneda"], a["disponible"], a["valoracion"],
             a["categoria"], a["url"]))
        self.stats.inc_value("libros/guardados")
        return item

    def close_spider(self):
        self.conn.commit()
        filas = self.conn.execute(
            "SELECT categoria, COUNT(*), ROUND(AVG(precio), 2), ROUND(AVG(valoracion), 1) "
            "FROM libros GROUP BY categoria ORDER BY COUNT(*) DESC LIMIT 10").fetchall()
        self.conn.close()
        if filas:
            print(f"\n{'Categoría':<22}{'Libros':>7}{'Precio medio':>14}{'Valoración':>12}")
            print("-" * 55)
            for categoria, n, precio, valoracion in filas:
                print(f"{categoria[:21]:<22}{n:>7}{precio:>14.2f}{valoracion:>11.1f}★")


# ------------------------------------------------------------------ Ejecución
def ejecutar(url: str, paginas: int, salida: Path, cache: bool, retardo: float) -> dict:
    salida.mkdir(parents=True, exist_ok=True)
    ajustes = {
        "USER_AGENT": "LibrosBot/1.0 (+formacion)",
        "ROBOTSTXT_OBEY": True,
        "DOWNLOAD_DELAY": retardo,
        "CONCURRENT_REQUESTS_PER_DOMAIN": 4,
        "AUTOTHROTTLE_ENABLED": True,
        "AUTOTHROTTLE_START_DELAY": retardo,
        "RETRY_TIMES": 2,
        "DOWNLOAD_TIMEOUT": 20,
        "HTTPCACHE_ENABLED": cache,
        "HTTPCACHE_DIR": str(salida / "httpcache"),
        "LOG_LEVEL": "INFO",
        "ITEM_PIPELINES": {
            NormalizarPipeline: 100,
            ValidarPipeline: 200,
            DuplicadosPipeline: 300,
            SQLitePipeline: 400,
        },
        "SQLITE_RUTA": str(salida / "libros.db"),
        "FEEDS": {
            str(salida / "libros.jsonl"): {"format": "jsonlines", "encoding": "utf8", "mode": "overwrite"},
            str(salida / "libros.csv"): {
                "format": "csv", "encoding": "utf-8-sig", "mode": "overwrite",
                "fields": ["titulo", "precio", "moneda", "disponible", "valoracion", "categoria", "upc", "url"],
                "item_export_kwargs": {"delimiter": ";"},
            },
        },
    }
    proceso = CrawlerProcess(ajustes)
    crawler = proceso.create_crawler(LibrosSpider)
    proceso.crawl(crawler, url_inicial=url, max_paginas=paginas)
    proceso.start()
    return crawler.stats.get_stats()


def main() -> int:
    p = argparse.ArgumentParser(description="Rastreador de catálogo de libros con Scrapy.")
    p.add_argument("--url", default="https://books.toscrape.com/catalogue/page-1.html")
    p.add_argument("--paginas", type=int, default=2, help="páginas del listado (0 = todas)")
    p.add_argument("--salida", type=Path, default=Path("salida_scrapy"))
    p.add_argument("--retardo", type=float, default=0.5)
    p.add_argument("--cache", action="store_true", help="activar caché HTTP (útil en desarrollo)")
    args = p.parse_args()

    estadisticas = ejecutar(args.url, args.paginas, args.salida, args.cache, args.retardo)
    print(f"\nPeticiones: {estadisticas.get('downloader/request_count', 0)} · "
          f"Items: {estadisticas.get('item_scraped_count', 0)} · "
          f"Descartados: {estadisticas.get('item_dropped_count', 0)} · "
          f"Errores: {estadisticas.get('libros/errores', 0)} · "
          f"Motivo de cierre: {estadisticas.get('finish_reason')}")
    print(f"Resultados en {args.salida}/ (libros.jsonl, libros.csv, libros.db)")
    return 0 if estadisticas.get("item_scraped_count") else 1


if __name__ == "__main__":
    raise SystemExit(main())
```

Pruebas del análisis sin red (`test_rastreador.py`):

```python
from scrapy.http import HtmlResponse, Request
from rastreador_libros import Libro, LibrosSpider, NormalizarPipeline

FICHA = b"""<html><body><ul class="breadcrumb"><li><a href="/">Home</a></li><li><a href="/b">Books</a></li>
<li><a href="/c">Poetry</a></li></ul><div class="product_main"><h1>Libro A</h1>
<p class="price_color">\xc2\xa351.77</p><p class="instock availability">In stock (3 available)</p>
<p class="star-rating Four"></p></div><table class="table table-striped"><tr><th>UPC</th><td>abc123</td></tr></table>
</body></html>"""


def test_parse_libro_y_normalizacion():
    url = "https://books.test/catalogue/a_1/index.html"
    respuesta = HtmlResponse(url=url, body=FICHA, encoding="utf-8", request=Request(url))
    arana = LibrosSpider(url_inicial=url)
    libro = next(arana.parse_libro(respuesta, datos={"titulo": "", "pagina_listado": 1}))
    assert isinstance(libro, Libro)
    assert (libro.titulo, libro.categoria, libro.upc, libro.disponible, libro.valoracion) == \
        ("Libro A", "Poetry", "abc123", 3, 4)

    libro = NormalizarPipeline().process_item(libro)
    assert (libro.precio, libro.moneda) == ("51.77", "£")
    assert libro.etiquetas == ["recomendado", "ultimas-unidades"]
```

```bash
python rastreador_libros.py --paginas 3 --cache
pytest -q test_rastreador.py
```

---

## 22. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| La araña termina sin datos y el log dice `Forbidden by robots.txt` | El sitio lo prohíbe en `robots.txt`. | Respételo; busque una API o permiso. |
| `start_requests()` no se ejecuta | En versiones recientes las peticiones iniciales se generan con `async def start()`. | Renombre el método a `async def start(self)`. |
| `Filtered offsite request` | El dominio no está en `allowed_domains`. | Añádalo (sin `https://` ni rutas). |
| `Filtered duplicate request` | URL ya visitada. | Es lo esperado; use `dont_filter=True` solo si es necesario. |
| Selectores que funcionan en el navegador devuelven `None` | El contenido se genera con JavaScript o el HTML difiere. | `scrapy view URL`; use la API o scrapy-playwright. |
| `.get()` devuelve un texto con saltos de línea y espacios | Texto con formato del HTML. | `.get(default="").strip()` o `normalize-space()` en XPath. |
| XPath dentro de un bucle devuelve datos de otros elementos | Se usó `//` en lugar de `.//`. | Use rutas relativas `.//`. |
| `ReactorNotRestartable` | Se llamó dos veces a `CrawlerProcess.start()`. | Lance todas las arañas antes de un único `start()`. |
| El archivo JSON queda corrupto al usar `-o` | Se añadió a un JSON existente. | Use `-O` (sobrescribir) o el formato `jsonlines`. |
| Aviso de opción `overwrite` obsoleta en `FEEDS` | Nombre antiguo. | Use `"mode": "overwrite"` o `"mode": "append"`. |
| Muchos errores 429/503 | Demasiada velocidad. | `DOWNLOAD_DELAY`, `AUTOTHROTTLE_ENABLED`, menos concurrencia. |
| `ScrapyDeprecationWarning: … process_item() requires a spider argument` | Firma antigua de los *pipelines*. | Use `process_item(self, item)` y obtenga el `crawler` en `from_crawler()`. |
| `KeyError` al añadir un campo en un `scrapy.Item` | Campo no declarado. | Declárelo con `scrapy.Field()` o use `dataclass`/dict. |

---

## 23. Buenas prácticas

- Pruebe los selectores en `scrapy shell` antes de escribir la araña.
- Use selectores estables (clases semánticas, atributos `data-*`, textos de cabecera) y rutas relativas en bucles.
- Defina *items* con `dataclass` y mueva la limpieza y validación a *pipelines* o *ItemLoaders*.
- Mantenga `ROBOTSTXT_OBEY = True`, un `USER_AGENT` identificable, retardos y *AutoThrottle*.
- Active `HTTPCACHE_ENABLED` durante el desarrollo para no repetir descargas.
- Prefiera las APIs JSON de los sitios frente al análisis de HTML cuando existan.
- Exporte a JSON Lines para volúmenes grandes o ejecuciones incrementales.
- Revise las estadísticas finales (errores, descartes, códigos HTTP) en cada ejecución.
- Use `JOBDIR` para rastreos largos que deban poder pausarse y reanudarse.
- Escriba pruebas de los *callbacks* con respuestas HTML guardadas.

---

## 24. Referencia rápida

```bash
scrapy startproject p · scrapy genspider nombre dominio · scrapy crawl nombre -O salida.json -a arg=v -s AJUSTE=v
scrapy shell URL · scrapy view URL · scrapy fetch URL · scrapy parse --spider=s -c callback URL · scrapy check · scrapy list
```

```python
class S(scrapy.Spider): name; allowed_domains; start_urls; custom_settings
    async def start(self): yield scrapy.Request(url, callback=, cb_kwargs=, errback=)
    def parse(self, response): yield {...} / yield response.follow(href, callback=) / yield from response.follow_all(css=...)
response.css("sel::text").get(default="") · .getall() · ::attr(x) · .re_first(r"") · response.xpath(".//x/text()")
response.json() · response.url · response.status · FormRequest.from_response(response, formdata={})
CrawlSpider + rules = (Rule(LinkExtractor(restrict_css=..., allow=r""), callback="parse_x", follow=True),)
ITEM_PIPELINES = {"mod.Pipeline": 300} · process_item(self, item) · from_crawler(cls, crawler) · raise DropItem · ItemAdapter(item)
FEEDS = {"x.jsonl": {"format": "jsonlines", "mode": "overwrite"}} · ROBOTSTXT_OBEY · DOWNLOAD_DELAY · AUTOTHROTTLE_ENABLED
HTTPCACHE_ENABLED · RETRY_TIMES · CONCURRENT_REQUESTS_PER_DOMAIN · DEPTH_LIMIT · CLOSESPIDER_ITEMCOUNT · JOBDIR
CrawlerProcess(ajustes).crawl(Spider, **args); .start()
```

---

## 25. Ejercicios propuestos

1. Cree una araña para `quotes.toscrape.com` que extraiga citas, autores y etiquetas, y que entre en la página de cada autor para obtener su fecha de nacimiento.
2. Convierta el ejemplo completo en un proyecto Scrapy estándar (`startproject`) con `items.py`, `pipelines.py` y `settings.py` separados.
3. Implemente un `CrawlSpider` que recorra todas las categorías de `books.toscrape.com` y cuente los libros de cada una.
4. Añada un *pipeline* que descargue las portadas con `ImagesPipeline` y genere miniaturas.
5. Use `JOBDIR` para pausar y reanudar un rastreo completo, y compare dos ejecuciones para detectar cambios de precio.

---

## 26. Referencias

- Documentación oficial: https://docs.scrapy.org/
- Tutorial oficial: https://docs.scrapy.org/en/latest/intro/tutorial.html
- Selectores: https://docs.scrapy.org/en/latest/topics/selectors.html
- scrapy-playwright: https://github.com/scrapy-plugins/scrapy-playwright
- Sitios de práctica: https://books.toscrape.com/ · https://quotes.toscrape.com/
