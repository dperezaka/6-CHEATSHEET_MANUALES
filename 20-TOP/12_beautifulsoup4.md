# Manual de uso: BeautifulSoup4

> **Categoría:** Desarrollo web · **Utilidad:** Extraer información de páginas web (HTML y XML)
> **Tipo:** Librería de terceros (`pip install beautifulsoup4`) · **Versión de referencia:** 4.12 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Crear el objeto `BeautifulSoup`](#4-crear-el-objeto-beautifulsoup)
5. [Analizadores (*parsers*)](#5-analizadores-parsers)
6. [Buscar elementos: `find` y `find_all`](#6-buscar-elementos-find-y-find_all)
7. [Selectores CSS: `select` y `select_one`](#7-selectores-css-select-y-select_one)
8. [Extraer texto y atributos](#8-extraer-texto-y-atributos)
9. [Navegar por el árbol](#9-navegar-por-el-árbol)
10. [Tablas, listas y formularios](#10-tablas-listas-y-formularios)
11. [Modificar el documento](#11-modificar-el-documento)
12. [Codificación de caracteres](#12-codificación-de-caracteres)
13. [XML](#13-xml)
14. [Rendimiento](#14-rendimiento)
15. [Descargar páginas: Requests/httpx + BeautifulSoup](#15-descargar-páginas-requestshttpx--beautifulsoup)
16. [*Scraping* responsable y aspectos legales](#16-scraping-responsable-y-aspectos-legales)
17. [Pruebas](#17-pruebas)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**Beautiful Soup** es una librería para **analizar documentos HTML y XML** y extraer de ellos la información que interesa: textos, enlaces, tablas, precios, atributos… Convierte el documento en un árbol de objetos Python que se puede recorrer, buscar (por etiqueta, clase, atributos o **selectores CSS**) y modificar. Tolera HTML mal formado, algo muy habitual en la web real.

Beautiful Soup **no descarga** páginas: se combina con **Requests** o **httpx** para obtener el HTML, y con un *parser* (`html.parser`, **lxml** o **html5lib**) para analizarlo.

**Ventajas:**
- API muy sencilla e intuitiva.
- Tolerante con HTML incorrecto o incompleto.
- Búsqueda flexible: etiquetas, atributos, expresiones regulares, funciones y selectores CSS.
- Detección automática de la codificación.
- Ideal para *scraping* puntual, limpieza de HTML y procesamiento de documentos.

**Limitaciones:**
- No ejecuta JavaScript: no ve el contenido generado dinámicamente en el navegador.
- No gestiona descargas, colas, reintentos ni concurrencia (para eso, Scrapy).
- Más lento que usar **lxml** o **selectolax** directamente en volúmenes muy grandes.

| Si necesita… | Opción |
|---|---|
| Extraer datos de HTML estático de forma sencilla | **BeautifulSoup4** |
| Rastrear muchos sitios/páginas con colas, reintentos y *pipelines* | Scrapy |
| Páginas que se construyen con JavaScript | Playwright / Selenium |
| Máximo rendimiento al analizar HTML | lxml / selectolax |
| Descargar el HTML | Requests / httpx |

---

## 2. Instalación e importación

```bash
pip install beautifulsoup4
pip install lxml               # Analizador rápido (recomendado)
pip install html5lib           # Analizador que imita a un navegador (opcional)
```

```python
from bs4 import BeautifulSoup
```

> El paquete se instala como `beautifulsoup4`, pero se importa como `bs4`. El antiguo paquete `BeautifulSoup` (versión 3) está obsoleto.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `BeautifulSoup` | Objeto que representa el documento completo. |
| `Tag` | Un elemento HTML (`<div>`, `<a>`…) con nombre, atributos e hijos. |
| `NavigableString` | Texto dentro de una etiqueta. |
| `Comment` | Comentario HTML (`<!-- … -->`). |
| Analizador (*parser*) | Componente que convierte el texto en árbol (`html.parser`, `lxml`, `html5lib`). |
| Árbol | Estructura jerárquica: padres, hijos, hermanos. |
| Selector CSS | Expresión como `div.producto > a[href]` para localizar elementos. |

```html
<div class="producto" data-id="17">
  <h2><a href="/p/17">Teclado</a></h2>
  <span class="precio">59,90 €</span>
</div>
```

```
Tag div (class=['producto'], data-id='17')
├── Tag h2
│   └── Tag a (href='/p/17')
│       └── NavigableString 'Teclado'
└── Tag span (class=['precio'])
    └── NavigableString '59,90 €'
```

---

## 4. Crear el objeto `BeautifulSoup`

```python
from bs4 import BeautifulSoup

html = """
<html><head><title>Tienda</title></head>
<body>
  <h1 id="titulo">Catálogo</h1>
  <ul class="productos">
    <li class="producto destacado" data-id="1"><a href="/p/1">Teclado</a> <span class="precio">59,90 €</span></li>
    <li class="producto" data-id="2"><a href="/p/2">Ratón</a> <span class="precio">19,50 €</span></li>
    <li class="producto agotado" data-id="3"><a href="/p/3">Monitor</a> <span class="precio">189,00 €</span></li>
  </ul>
  <!-- comentario interno -->
</body></html>
"""

sopa = BeautifulSoup(html, "lxml")                        # Desde texto
with open("pagina.html", encoding="utf-8") as f:
    sopa = BeautifulSoup(f, "lxml")                       # Desde archivo

print(sopa.title.string)          # 'Tienda'
print(sopa.h1["id"])              # 'titulo'
print(sopa.prettify()[:200])      # HTML indentado
```

> Indique **siempre** el analizador. Si lo omite, Beautiful Soup elige el mejor instalado y muestra un aviso: el resultado podría variar entre equipos.

---

## 5. Analizadores (*parsers*)

| Analizador | Uso | Velocidad | Tolerancia | Instalación |
|---|---|---|---|---|
| `"html.parser"` | Incluido en Python. | Media | Buena | Ninguna |
| `"lxml"` | Recomendado para HTML. | **Alta** | Buena | `pip install lxml` |
| `"html5lib"` | Analiza exactamente como un navegador. | Baja | **Máxima** | `pip install html5lib` |
| `"xml"` / `"lxml-xml"` | Documentos XML. | Alta | — | `pip install lxml` |

Los analizadores pueden producir árboles distintos con HTML incorrecto:

```python
BeautifulSoup("<a></p>", "html.parser")   # <a></a>
BeautifulSoup("<a></p>", "lxml")          # <html><body><a></a></body></html>
BeautifulSoup("<a></p>", "html5lib")      # <html><head></head><body><a><p></p></a></body></html>
```

---

## 6. Buscar elementos: `find` y `find_all`

### 6.1 Por nombre de etiqueta

```python
sopa.find("li")                     # Primer <li> (o None)
sopa.find_all("li")                 # Lista de todos los <li>
sopa.find_all(["h1", "h2", "h3"])   # Varias etiquetas
sopa.find_all("li", limit=2)        # Como máximo 2
sopa("li")                          # Atajo de find_all("li")
sopa.li                             # Atajo de find("li")
```

### 6.2 Por clase, id y atributos

```python
sopa.find_all("li", class_="producto")             # Contiene la clase 'producto'
sopa.find_all(class_="agotado")                    # Cualquier etiqueta con la clase
sopa.find(id="titulo")
sopa.find_all("li", attrs={"data-id": "2"})        # Atributos con guion: usar attrs
sopa.find_all("a", href=True)                      # Que tengan href
sopa.find_all("li", class_="producto destacado")   # Coincidencia EXACTA del atributo class completo
```

> `class` es palabra reservada en Python: use `class_`. Para atributos con guiones (`data-id`, `aria-label`), use `attrs={...}`.

### 6.3 Con expresiones regulares y funciones

```python
import re

sopa.find_all("a", href=re.compile(r"^/p/\d+$"))
sopa.find_all(re.compile(r"^h[1-6]$"))                         # Todas las cabeceras
sopa.find_all(string=re.compile(r"\d+,\d{2} €"))               # Textos que parecen precios

def es_producto_con_precio_alto(tag):
    if tag.name != "li" or "producto" not in tag.get("class", []):
        return False
    precio = tag.find(class_="precio")
    return precio and float(precio.text.replace("€", "").replace(",", ".")) > 50

sopa.find_all(es_producto_con_precio_alto)
```

### 6.4 Buscar por texto

```python
sopa.find("a", string="Ratón")                     # Texto exacto
sopa.find_all(string=lambda t: "€" in t)           # Nodos de texto
sopa.find(string="Ratón").parent                   # La etiqueta que lo contiene
```

### 6.5 Búsqueda solo en hijos directos

```python
lista = sopa.find("ul")
lista.find_all("li", recursive=False)              # Solo los <li> hijos directos
```

---

## 7. Selectores CSS: `select` y `select_one`

Beautiful Soup admite selectores CSS (mediante la librería `soupsieve`, que se instala automáticamente). Suelen ser la forma más concisa de localizar elementos.

```python
sopa.select("li.producto")                          # Lista
sopa.select_one("h1#titulo")                        # Primero (o None)
sopa.select("ul.productos > li")                    # Hijos directos
sopa.select("li.producto a[href^='/p/']")           # Atributo que empieza por
sopa.select("li[data-id='2'] .precio")
sopa.select("li.producto:not(.agotado)")            # Negación
sopa.select("li:nth-of-type(2)")                    # Posición
sopa.select("li:has(> span.precio)")                # Que contengan…
sopa.select("a:-soup-contains('Rat')")              # Que contengan un texto (extensión de soupsieve)
sopa.select("h1, h2, h3")                           # Varios selectores
```

| Selector | Significado |
|---|---|
| `etiqueta`, `.clase`, `#id` | Por nombre, clase o id. |
| `a b` / `a > b` | Descendiente / hijo directo. |
| `a + b` / `a ~ b` | Hermano siguiente inmediato / cualquier hermano siguiente. |
| `[attr]`, `[attr="v"]` | Atributo presente / con valor. |
| `[attr^="v"]`, `[attr$="v"]`, `[attr*="v"]` | Empieza, termina, contiene. |
| `:nth-child(n)`, `:first-child`, `:last-child`, `:nth-of-type(n)` | Posición. |
| `:not(sel)`, `:has(sel)`, `:is(a, b)` | Lógicos. |
| `:-soup-contains("texto")` | Contiene un texto. |

> Truco: en el navegador, **clic derecho → Inspeccionar → clic derecho sobre el elemento → Copiar → Copiar selector** da un selector de partida (aunque conviene simplificarlo y hacerlo más robusto).

---

## 8. Extraer texto y atributos

### 8.1 Texto

```python
li = sopa.select_one("li.producto")
li.text                                   # 'Teclado 59,90 €' (todo el texto, incluidos hijos)
li.get_text(" ", strip=True)              # Separador y limpieza de espacios
li.get_text(separator="|", strip=True)    # 'Teclado|59,90 €'
li.a.string                               # 'Teclado' (solo si hay un único hijo de texto; si no, None)
list(li.stripped_strings)                 # ['Teclado', '59,90 €']
```

| Propiedad / método | Devuelve |
|---|---|
| `.text` / `.get_text()` | Todo el texto concatenado. |
| `.get_text(" ", strip=True)` | Texto limpio con separador. |
| `.string` | El texto si la etiqueta tiene un único hijo de texto; `None` en otro caso. |
| `.strings` / `.stripped_strings` | Generador de fragmentos de texto. |

### 8.2 Atributos

```python
a = sopa.select_one("li.producto a")
a["href"]                     # '/p/1' (KeyError si no existe)
a.get("href")                 # '/p/1' (None si no existe)
a.get("title", "")            # Valor por defecto
a.attrs                       # {'href': '/p/1'}
li["class"]                   # ['producto', 'destacado']  ← los atributos multivalor son listas
li.get("data-id")             # '1'
li.has_attr("data-id")        # True
```

### 8.3 URLs absolutas

```python
from urllib.parse import urljoin

base = "https://tienda.ejemplo.com/catalogo/pagina-2.html"
enlaces = [urljoin(base, a["href"]) for a in sopa.select("a[href]")]
# '/p/1' → 'https://tienda.ejemplo.com/p/1'
```

### 8.4 Limpieza de valores

```python
import re
from decimal import Decimal

def precio(texto: str) -> Decimal | None:
    """'1.234,56 €' → Decimal('1234.56')"""
    m = re.search(r"(\d{1,3}(?:\.\d{3})*|\d+),(\d{2})", texto)
    if not m:
        return None
    return Decimal(m.group(1).replace(".", "") + "." + m.group(2))

def texto_o_vacio(nodo, selector: str) -> str:
    elemento = nodo.select_one(selector)
    return elemento.get_text(" ", strip=True) if elemento else ""
```

---

## 9. Navegar por el árbol

```python
li = sopa.select_one("li[data-id='2']")

li.parent                        # <ul class="productos">
li.parents                       # Generador hasta la raíz
li.find_parent("ul")             # Primer antepasado <ul>

li.contents                      # Lista de hijos (incluye textos con espacios)
li.children                      # Generador de hijos directos
li.descendants                   # Todos los descendientes

li.find_next_sibling("li")       # Siguiente <li> hermano
li.find_previous_sibling("li")
li.next_sibling                  # Siguiente nodo (puede ser un texto '\n')
li.find_next("span")             # Siguiente <span> en el documento (no solo hermanos)
li.find_all_next("a")
```

> `next_sibling`, `previous_sibling` y `contents` incluyen los **saltos de línea y espacios** entre etiquetas. Use `find_next_sibling("etiqueta")` para saltar directamente a la siguiente etiqueta.

Patrón muy frecuente: valor que sigue a una etiqueta de texto.

```html
<dl><dt>Marca</dt><dd>Logitech</dd><dt>Garantía</dt><dd>2 años</dd></dl>
```

```python
ficha = {dt.get_text(strip=True): dt.find_next_sibling("dd").get_text(strip=True)
         for dt in sopa.select("dl dt")}
# {'Marca': 'Logitech', 'Garantía': '2 años'}
```

---

## 10. Tablas, listas y formularios

### 10.1 Tabla HTML a lista de diccionarios

```python
def tabla_a_dicts(tabla) -> list[dict[str, str]]:
    cabeceras = [th.get_text(" ", strip=True) for th in tabla.select("thead th")] \
        or [c.get_text(" ", strip=True) for c in tabla.select_one("tr").find_all(["th", "td"])]
    filas = tabla.select("tbody tr") or tabla.select("tr")[1:]
    datos = []
    for fila in filas:
        celdas = [c.get_text(" ", strip=True) for c in fila.find_all(["td", "th"])]
        if celdas:
            datos.append(dict(zip(cabeceras, celdas)))
    return datos

datos = tabla_a_dicts(sopa.select_one("table#ventas"))
```

> Con **pandas**, `pandas.read_html(str(tabla))` convierte directamente tablas HTML en *DataFrames* (requiere lxml).

### 10.2 Listas

```python
elementos = [li.get_text(strip=True) for li in sopa.select("ul.caracteristicas > li")]
```

### 10.3 Formularios (por ejemplo, para obtener *tokens* ocultos)

```python
form = sopa.select_one("form#login")
datos = {i["name"]: i.get("value", "") for i in form.select("input[name]")}
accion = urljoin(url, form.get("action", url))
metodo = form.get("method", "get").lower()
# datos incluye campos ocultos como csrf_token; complete usuario/clave y envíe con Requests
```

### 10.4 Metadatos de la página

```python
titulo = sopa.title.get_text(strip=True) if sopa.title else ""
descripcion = (sopa.find("meta", attrs={"name": "description"}) or {}).get("content", "")
og_imagen = (sopa.find("meta", property="og:image") or {}).get("content")
canonica = (sopa.find("link", rel="canonical") or {}).get("href")
idioma = sopa.html.get("lang") if sopa.html else None

import json
datos_estructurados = [json.loads(s.string) for s in sopa.select('script[type="application/ld+json"]') if s.string]
```

---

## 11. Modificar el documento

```python
# Cambiar texto y atributos
sopa.h1.string = "Catálogo 2026"
a = sopa.select_one("a")
a["href"] = "https://tienda.ejemplo.com/p/1"
a["target"] = "_blank"
del a["target"]

# Añadir elementos
nuevo = sopa.new_tag("li", attrs={"class": "producto", "data-id": "4"})
nuevo.string = "Hub USB-C"
sopa.select_one("ul.productos").append(nuevo)
sopa.h1.insert_after(sopa.new_tag("p"))

# Eliminar
for s in sopa(["script", "style", "noscript"]):
    s.decompose()                         # Elimina y destruye
comentario.extract()                      # Elimina y devuelve el elemento

# Sustituir y envolver
sopa.select_one("span.precio").replace_with("consultar")
sopa.select_one("a").wrap(sopa.new_tag("strong"))
sopa.select_one("strong").unwrap()        # Quita la etiqueta y conserva el contenido

# Obtener el resultado
html = str(sopa)                          # HTML
html_bonito = sopa.prettify()
```

Eliminar comentarios:

```python
from bs4 import Comment
for c in sopa.find_all(string=lambda t: isinstance(t, Comment)):
    c.extract()
```

Limpiar HTML a texto legible:

```python
def html_a_texto(html: str) -> str:
    sopa = BeautifulSoup(html, "lxml")
    for t in sopa(["script", "style", "nav", "footer", "header", "aside"]):
        t.decompose()
    return "\n".join(linea for linea in sopa.get_text("\n", strip=True).splitlines() if linea)
```

---

## 12. Codificación de caracteres

Beautiful Soup convierte todo a Unicode y detecta la codificación automáticamente:

```python
sopa = BeautifulSoup(contenido_en_bytes, "lxml")
sopa.original_encoding                         # 'utf-8', 'windows-1252'…

sopa = BeautifulSoup(contenido_en_bytes, "lxml", from_encoding="iso-8859-1")   # Forzar
```

> Con Requests, pase **`r.content`** (bytes) en lugar de `r.text`: así Beautiful Soup usa la declaración `<meta charset>` del documento, que suele ser más fiable que la cabecera HTTP.

`UnicodeDammit` para detectar la codificación de cualquier texto:

```python
from bs4 import UnicodeDammit
dammit = UnicodeDammit(datos_bytes, ["utf-8", "windows-1252"])
texto, codificacion = dammit.unicode_markup, dammit.original_encoding
```

---

## 13. XML

```python
xml = """<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0"><channel>
  <title>Noticias</title>
  <item><title>Primera</title><link>https://ej.com/1</link><pubDate>Mon, 24 Nov 2026 10:00:00 GMT</pubDate></item>
  <item><title>Segunda</title><link>https://ej.com/2</link><pubDate>Tue, 25 Nov 2026 09:00:00 GMT</pubDate></item>
</channel></rss>"""

sopa = BeautifulSoup(xml, "xml")                   # Respeta mayúsculas y etiquetas como <link>
for item in sopa.find_all("item"):
    print(item.title.text, item.link.text, item.pubDate.text)
```

> Con el analizador `"xml"` se distinguen mayúsculas y minúsculas (`pubDate`) y las etiquetas que en HTML son vacías (como `<link>`) conservan su contenido.

Espacios de nombres:

```python
sopa = BeautifulSoup(sitemap_xml, "xml")
urls = [loc.text for loc in sopa.find_all("loc")]
```

---

## 14. Rendimiento

```python
from bs4 import SoupStrainer

# Analizar solo los enlaces (mucho más rápido y con menos memoria)
solo_enlaces = SoupStrainer("a", href=True)
sopa = BeautifulSoup(html, "lxml", parse_only=solo_enlaces)

solo_productos = SoupStrainer("li", class_="producto")
```

Recomendaciones:
- Use el analizador **lxml**.
- Limite el análisis con `SoupStrainer` cuando solo necesite una parte del documento.
- Use `select_one`/`find` en lugar de `select`/`find_all` cuando solo necesite el primer resultado.
- Para millones de documentos, considere `lxml.html` directamente o **selectolax**.
- Libere memoria en bucles largos con `sopa.decompose()`.

---

## 15. Descargar páginas: Requests/httpx + BeautifulSoup

```python
import requests
from bs4 import BeautifulSoup

sesion = requests.Session()
sesion.headers["User-Agent"] = "MiScraper/1.0 (+contacto@ejemplo.com)"

def obtener_sopa(url: str) -> BeautifulSoup:
    r = sesion.get(url, timeout=15)
    r.raise_for_status()
    return BeautifulSoup(r.content, "lxml")

sopa = obtener_sopa("https://books.toscrape.com/")
for libro in sopa.select("article.product_pod"):
    print(libro.h3.a["title"], libro.select_one(".price_color").text)
```

Recorrer la paginación siguiendo el enlace «siguiente»:

```python
from urllib.parse import urljoin

def todas_las_paginas(url_inicial: str):
    url = url_inicial
    while url:
        sopa = obtener_sopa(url)
        yield url, sopa
        siguiente = sopa.select_one("li.next > a")
        url = urljoin(url, siguiente["href"]) if siguiente else None
```

> Si el contenido no aparece en el HTML descargado pero sí en el navegador, se está generando con JavaScript. Busque en la pestaña **Red** de las herramientas de desarrollo una API JSON que lo proporcione, o use Playwright/Selenium.

---

## 16. *Scraping* responsable y aspectos legales

- Consulte y respete el archivo **`robots.txt`** (`urllib.robotparser`) y las condiciones de uso del sitio.
- Use la **API oficial** si existe.
- Identifíquese con un `User-Agent` descriptivo y un contacto.
- Limite la frecuencia de peticiones (pausas entre páginas) y evite horas punta.
- Guarde en caché las páginas ya descargadas para no repetir peticiones.
- No recopile datos personales sin base legal (RGPD) ni contenido protegido para redistribuirlo.
- No eluda mecanismos de acceso (inicios de sesión ajenos, CAPTCHA, bloqueos).

```python
from urllib.robotparser import RobotFileParser

robots = RobotFileParser("https://ejemplo.com/robots.txt")
robots.read()
if robots.can_fetch("MiScraper", "https://ejemplo.com/catalogo/"):
    ...
retraso = robots.crawl_delay("MiScraper") or 1.0
```

---

## 17. Pruebas

Guarde páginas reales como archivos de prueba (*fixtures*) y pruebe el análisis sin red:

```python
from pathlib import Path
from bs4 import BeautifulSoup
from mi_scraper import extraer_productos

def test_extraer_productos():
    html = Path("tests/fixtures/catalogo.html").read_text(encoding="utf-8")
    productos = extraer_productos(BeautifulSoup(html, "lxml"))
    assert len(productos) == 20
    assert productos[0]["titulo"] == "A Light in the Attic"
    assert productos[0]["precio"] > 0
```

> Separe siempre la **descarga** del **análisis**: las funciones que reciben un `BeautifulSoup` (o un `str`) son fáciles de probar y de reutilizar.

---

## 18. Ejemplo práctico completo

**Extractor de catálogo de libros**: recorre todas las páginas de un catálogo con paginación, entra en la ficha de cada libro, extrae título, precio, disponibilidad, valoración, categoría, UPC y descripción, normaliza los valores, respeta `robots.txt` y una pausa entre peticiones, guarda una caché local del HTML, y exporta a CSV y JSON con un resumen por categoría. Está preparado para **books.toscrape.com**, un sitio creado expresamente para practicar *scraping*.

```python
"""
libros.py — Extractor de catálogo con Requests + BeautifulSoup.
Requiere: pip install requests beautifulsoup4 lxml
Uso:
    python libros.py --paginas 2
    python libros.py --url https://books.toscrape.com/catalogue/category/books/travel_2/index.html
"""
import argparse
import csv
import hashlib
import json
import logging
import re
import sys
import time
from collections import defaultdict
from dataclasses import asdict, dataclass
from decimal import Decimal
from pathlib import Path
from statistics import mean
from urllib.parse import urljoin, urlparse
from urllib.robotparser import RobotFileParser

import requests
from bs4 import BeautifulSoup

log = logging.getLogger("libros")
VALORACIONES = {"One": 1, "Two": 2, "Three": 3, "Four": 4, "Five": 5}


@dataclass
class Libro:
    titulo: str
    precio: Decimal
    moneda: str
    disponible: int
    valoracion: int
    categoria: str
    upc: str
    descripcion: str
    url: str
    imagen: str


# ------------------------------------------------------------------ Descarga
class Descargador:
    def __init__(self, cache: Path | None = Path(".cache_html"), pausa: float = 0.5,
                 agente: str = "LibrosScraper/1.0 (+formacion)"):
        self.sesion = requests.Session()
        self.sesion.headers.update({"User-Agent": agente, "Accept-Language": "es,en;q=0.8"})
        self.cache, self.pausa, self.agente = cache, pausa, agente
        self.robots: dict[str, RobotFileParser] = {}
        self.descargas = 0
        if cache:
            cache.mkdir(exist_ok=True)

    def permitido(self, url: str) -> bool:
        origen = "{0.scheme}://{0.netloc}".format(urlparse(url))
        if origen not in self.robots:
            rp = RobotFileParser(origen + "/robots.txt")
            try:
                r = self.sesion.get(rp.url, timeout=10)
                rp.parse(r.text.splitlines() if r.status_code == 200 else [])
            except requests.RequestException:
                rp.parse([])                              # Sin robots.txt accesible: se permite
            self.robots[origen] = rp
        return self.robots[origen].can_fetch(self.agente, url)

    def sopa(self, url: str) -> BeautifulSoup:
        ruta_cache = self.cache / (hashlib.sha1(url.encode()).hexdigest() + ".html") if self.cache else None
        if ruta_cache and ruta_cache.exists():
            return BeautifulSoup(ruta_cache.read_bytes(), "lxml")
        if not self.permitido(url):
            raise PermissionError(f"robots.txt no permite acceder a {url}")
        if self.descargas:
            time.sleep(self.pausa)                        # Cortesía con el servidor
        r = self.sesion.get(url, timeout=15)
        r.raise_for_status()
        self.descargas += 1
        if ruta_cache:
            ruta_cache.write_bytes(r.content)
        return BeautifulSoup(r.content, "lxml")


# ------------------------------------------------------------------ Análisis (funciones puras, fáciles de probar)
def precio_y_moneda(texto: str) -> tuple[Decimal, str]:
    m = re.search(r"([£$€])\s*([\d.,]+)", texto)
    if not m:
        raise ValueError(f"Precio no reconocido: {texto!r}")
    return Decimal(m.group(2).replace(",", "")), m.group(1)


def enlaces_de_listado(sopa: BeautifulSoup, url_pagina: str) -> tuple[list[str], str | None]:
    fichas = [urljoin(url_pagina, a["href"]) for a in sopa.select("article.product_pod h3 a[href]")]
    siguiente = sopa.select_one("ul.pager li.next a[href]")
    return fichas, urljoin(url_pagina, siguiente["href"]) if siguiente else None


def analizar_ficha(sopa: BeautifulSoup, url: str) -> Libro:
    principal = sopa.select_one("div.product_main")
    if principal is None:
        raise ValueError(f"La página no parece una ficha de libro: {url}")
    precio, moneda = precio_y_moneda(principal.select_one("p.price_color").get_text())

    stock_texto = principal.select_one("p.availability").get_text(" ", strip=True)
    stock = re.search(r"\((\d+) available\)", stock_texto)

    clase_valoracion = principal.select_one("p.star-rating")
    valoracion = next((VALORACIONES[c] for c in clase_valoracion.get("class", []) if c in VALORACIONES), 0) \
        if clase_valoracion else 0

    tabla = {fila.th.get_text(strip=True): fila.td.get_text(strip=True)
             for fila in sopa.select("table.table-striped tr") if fila.th and fila.td}

    migas = [a.get_text(strip=True) for a in sopa.select("ul.breadcrumb li a")]
    descripcion = sopa.select_one("#product_description")
    descripcion_texto = descripcion.find_next_sibling("p").get_text(strip=True) if descripcion else ""
    imagen = sopa.select_one("#product_gallery img, div.item img")

    return Libro(
        titulo=principal.h1.get_text(strip=True),
        precio=precio,
        moneda=moneda,
        disponible=int(stock.group(1)) if stock else 0,
        valoracion=valoracion,
        categoria=migas[-1] if len(migas) >= 3 else "",
        upc=tabla.get("UPC", ""),
        descripcion=descripcion_texto,
        url=url,
        imagen=urljoin(url, imagen["src"]) if imagen and imagen.get("src") else "",
    )


# ------------------------------------------------------------------ Proceso
def extraer(url_inicial: str, descargador: Descargador, max_paginas: int | None = None) -> list[Libro]:
    libros: list[Libro] = []
    url, pagina = url_inicial, 0
    while url and (max_paginas is None or pagina < max_paginas):
        pagina += 1
        listado = descargador.sopa(url)
        fichas, url = enlaces_de_listado(listado, url)
        log.info("Página %d: %d libros", pagina, len(fichas))
        for enlace in fichas:
            try:
                libros.append(analizar_ficha(descargador.sopa(enlace), enlace))
            except (ValueError, requests.RequestException, PermissionError) as e:
                log.warning("Omitido %s: %s", enlace, e)
    return libros


def exportar(libros: list[Libro], carpeta: Path) -> None:
    carpeta.mkdir(parents=True, exist_ok=True)
    filas = [{**asdict(l), "precio": str(l.precio)} for l in libros]
    (carpeta / "libros.json").write_text(json.dumps(filas, ensure_ascii=False, indent=2), encoding="utf-8")
    with (carpeta / "libros.csv").open("w", newline="", encoding="utf-8-sig") as f:
        escritor = csv.DictWriter(f, fieldnames=[k for k in filas[0] if k != "descripcion"],
                                  delimiter=";", extrasaction="ignore")
        escritor.writeheader()
        escritor.writerows(filas)


def resumen(libros: list[Libro]) -> None:
    por_categoria: dict[str, list[Libro]] = defaultdict(list)
    for l in libros:
        por_categoria[l.categoria or "(sin categoría)"].append(l)
    print(f"\n{'Categoría':<24}{'Libros':>7}{'Precio medio':>14}{'Valoración':>12}{'Stock':>8}")
    print("-" * 65)
    for categoria, grupo in sorted(por_categoria.items(), key=lambda x: -len(x[1])):
        print(f"{categoria[:23]:<24}{len(grupo):>7}{mean(float(l.precio) for l in grupo):>13.2f}"
              f"{grupo[0].moneda}{mean(l.valoracion for l in grupo):>11.1f}★{sum(l.disponible for l in grupo):>8}")
    mejor = max(libros, key=lambda l: (l.valoracion, -l.precio))
    print(f"\nMejor valorado y más barato: «{mejor.titulo}» ({mejor.valoracion}★, {mejor.moneda}{mejor.precio})")


def main() -> int:
    p = argparse.ArgumentParser(description="Extractor de catálogo de libros.")
    p.add_argument("--url", default="https://books.toscrape.com/catalogue/page-1.html")
    p.add_argument("--paginas", type=int, default=1, help="número máximo de páginas del listado")
    p.add_argument("--salida", type=Path, default=Path("salida_libros"))
    p.add_argument("--pausa", type=float, default=0.5, help="segundos entre peticiones")
    p.add_argument("--sin-cache", action="store_true")
    p.add_argument("-v", "--verbose", action="store_true")
    args = p.parse_args()
    logging.basicConfig(level=logging.INFO if args.verbose else logging.WARNING, format="%(levelname)s %(message)s")

    descargador = Descargador(cache=None if args.sin_cache else Path(".cache_html"), pausa=args.pausa)
    inicio = time.perf_counter()
    try:
        libros = extraer(args.url, descargador, args.paginas)
    except requests.RequestException as e:
        print(f"✖ Error de red: {e}", file=sys.stderr)
        return 1
    if not libros:
        print("No se ha extraído ningún libro.", file=sys.stderr)
        return 1
    exportar(libros, args.salida)
    resumen(libros)
    print(f"\n✔ {len(libros)} libros en {time.perf_counter() - inicio:.1f} s "
          f"({descargador.descargas} descargas) → {args.salida}/libros.csv y libros.json")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

Pruebas sin red (`test_libros.py`):

```python
from decimal import Decimal
from bs4 import BeautifulSoup
from libros import analizar_ficha, enlaces_de_listado, precio_y_moneda

LISTADO = """<ol><li><article class="product_pod"><h3><a href="libro-a_1/index.html">A</a></h3></article></li>
<li><article class="product_pod"><h3><a href="libro-b_2/index.html">B</a></h3></article></li></ol>
<ul class="pager"><li class="next"><a href="page-2.html">next</a></li></ul>"""

FICHA = """<ul class="breadcrumb"><li><a href="/">Home</a></li><li><a href="/books">Books</a></li>
<li><a href="/travel">Travel</a></li><li class="active">Libro A</li></ul>
<div class="product_main"><h1>Libro A</h1><p class="price_color">£51.77</p>
<p class="instock availability"><i></i> In stock (22 available)</p><p class="star-rating Three"></p></div>
<div id="product_description"><h2>Product Description</h2></div><p>Una descripción.</p>
<table class="table table-striped"><tr><th>UPC</th><td>a897fe39b1053632</td></tr></table>"""


def test_listado():
    fichas, siguiente = enlaces_de_listado(BeautifulSoup(LISTADO, "lxml"), "https://x.test/catalogue/page-1.html")
    assert fichas == ["https://x.test/catalogue/libro-a_1/index.html", "https://x.test/catalogue/libro-b_2/index.html"]
    assert siguiente == "https://x.test/catalogue/page-2.html"


def test_ficha():
    libro = analizar_ficha(BeautifulSoup(FICHA, "lxml"), "https://x.test/a")
    assert (libro.titulo, libro.precio, libro.moneda) == ("Libro A", Decimal("51.77"), "£")
    assert (libro.disponible, libro.valoracion, libro.categoria) == (22, 3, "Travel")
    assert libro.upc == "a897fe39b1053632" and libro.descripcion == "Una descripción."


def test_precio():
    assert precio_y_moneda("Precio: € 1,234.50") == (Decimal("1234.50"), "€")
```

```bash
python libros.py --paginas 2 -v
pytest -q test_libros.py
```

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `AttributeError: 'NoneType' object has no attribute 'text'` | `find`/`select_one` no encontró nada. | Compruebe `None` antes de usar el resultado. |
| `find_all` devuelve una lista vacía, pero el elemento se ve en el navegador | El contenido se genera con JavaScript. | Busque la API JSON de la página o use Playwright/Selenium. |
| `GuessedAtParserWarning` | No se indicó el analizador. | `BeautifulSoup(html, "lxml")`. |
| `FeatureNotFound: Couldn't find a tree builder… lxml` | lxml no está instalado. | `pip install lxml` o use `"html.parser"`. |
| `.string` devuelve `None` | La etiqueta tiene varios hijos. | Use `.get_text(strip=True)`. |
| `class_="a b"` no encuentra elementos | Compara el atributo completo en ese orden. | Use `select(".a.b")`. |
| Caracteres extraños (`Ã©`) | Codificación mal detectada al pasar `r.text`. | Pase `r.content` o `from_encoding=`. |
| Saltos de línea inesperados en `next_sibling` | Los espacios entre etiquetas son nodos de texto. | `find_next_sibling("etiqueta")`. |
| Enlaces relativos que no funcionan | `href` relativos. | `urljoin(url_base, href)`. |
| `KeyError: 'href'` | La etiqueta no tiene ese atributo. | Use `.get("href")` o `select("a[href]")`. |
| Bloqueos o errores 403/429 | Demasiadas peticiones o *User-Agent* vacío. | Pausas, caché, `User-Agent` descriptivo y respeto de `robots.txt`. |

---

## 20. Buenas prácticas

- Indique siempre el analizador (preferiblemente `lxml`).
- Prefiera selectores CSS concisos y estables (clases semánticas, `id`, atributos `data-*`) frente a posiciones (`nth-child`).
- Compruebe siempre si un resultado es `None` antes de usarlo.
- Separe descarga, análisis y almacenamiento en funciones distintas.
- Normalice los datos al extraerlos (precios a `Decimal`, fechas a `date`, espacios con `strip`).
- Convierta los enlaces relativos con `urljoin`.
- Guarde páginas de ejemplo para pruebas y detecte cambios en la estructura del sitio.
- Respete `robots.txt`, las condiciones de uso y una frecuencia de peticiones razonable; use caché.
- Para rastreos grandes, pase a Scrapy; para páginas dinámicas, a Playwright.

---

## 21. Referencia rápida

```python
sopa = BeautifulSoup(html_o_bytes, "lxml" | "html.parser" | "html5lib" | "xml", parse_only=SoupStrainer(...))
sopa.find("tag", class_="c", id="i", attrs={"data-x": "1"}, string="t") · sopa.find_all(..., limit=, recursive=False)
sopa.select("div.c > a[href^='/p/']") · sopa.select_one("#id") · :not() :has() :nth-of-type() :-soup-contains()
tag.get_text(" ", strip=True) · tag.string · tag.stripped_strings · tag["attr"] · tag.get("attr", "") · tag.attrs
tag.parent · find_parent · children · descendants · find_next_sibling("x") · find_next("x")
new_tag · append · insert_after · replace_with · wrap · unwrap · decompose · extract · str(sopa) · prettify()
urljoin(base, href) · sopa.original_encoding · UnicodeDammit · RobotFileParser
```

---

## 22. Ejercicios propuestos

1. Extraiga de una página de Wikipedia todas las tablas de clase `wikitable` y guárdelas como CSV.
2. Escriba una función que, dado el HTML de un artículo, devuelva título, autor, fecha, texto limpio y enlaces externos.
3. Analice un *sitemap* XML y compruebe con Requests qué URLs devuelven un código distinto de 200.
4. Amplíe el ejemplo completo para descargar las portadas de los libros mejor valorados (5★) en una carpeta.
5. Cree un detector de cambios: guarde la versión anterior de los precios y muestre qué libros han subido o bajado.

---

## 23. Referencias

- Documentación oficial: https://www.crummy.com/software/BeautifulSoup/bs4/doc/
- Traducción al español: https://www.crummy.com/software/BeautifulSoup/bs4/doc.es/
- Soup Sieve (selectores CSS): https://facelessuser.github.io/soupsieve/
- Sitio de práctica: https://books.toscrape.com/
- `urllib.robotparser`: https://docs.python.org/es/3/library/urllib.robotparser.html
