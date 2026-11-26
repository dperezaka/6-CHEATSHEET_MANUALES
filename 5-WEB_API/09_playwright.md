# Manual de uso: Playwright

> **Categoría:** Desarrollo web · **Utilidad:** Automatización moderna de navegadores
> **Tipo:** Librería de terceros (`pip install playwright`) · **Versión de referencia:** 1.4x o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer script (API síncrona)](#4-primer-script-api-síncrona)
5. [API asíncrona](#5-api-asíncrona)
6. [Navegador, contexto y página](#6-navegador-contexto-y-página)
7. [Localizadores](#7-localizadores)
8. [Acciones](#8-acciones)
9. [Esperas automáticas y aserciones (`expect`)](#9-esperas-automáticas-y-aserciones-expect)
10. [Red: esperar, interceptar y simular peticiones](#10-red-esperar-interceptar-y-simular-peticiones)
11. [Autenticación y reutilización de sesión](#11-autenticación-y-reutilización-de-sesión)
12. [Diálogos, *iframes*, pestañas y ventanas emergentes](#12-diálogos-iframes-pestañas-y-ventanas-emergentes)
13. [Archivos: subidas y descargas](#13-archivos-subidas-y-descargas)
14. [Capturas, PDF y vídeo](#14-capturas-pdf-y-vídeo)
15. [Emulación: dispositivos, idioma, geolocalización y reloj](#15-emulación-dispositivos-idioma-geolocalización-y-reloj)
16. [JavaScript en la página](#16-javascript-en-la-página)
17. [Pruebas con pytest-playwright](#17-pruebas-con-pytest-playwright)
18. [Pruebas de API (`APIRequestContext`)](#18-pruebas-de-api-apirequestcontext)
19. [Herramientas: *codegen*, inspector y *trace viewer*](#19-herramientas-codegen-inspector-y-trace-viewer)
20. [*Scraping* de páginas dinámicas](#20-scraping-de-páginas-dinámicas)
21. [Ejecución en CI y Docker](#21-ejecución-en-ci-y-docker)
22. [Ejemplo práctico completo](#22-ejemplo-práctico-completo)
23. [Errores frecuentes](#23-errores-frecuentes)
24. [Buenas prácticas](#24-buenas-prácticas)
25. [Referencia rápida](#25-referencia-rápida)
26. [Ejercicios propuestos](#26-ejercicios-propuestos)
27. [Referencias](#27-referencias)

---

## 1. Descripción

**Playwright** es una librería de Microsoft para **automatizar navegadores** Chromium (Chrome, Edge), Firefox y WebKit (motor de Safari) con una única API. Está diseñada para **pruebas de extremo a extremo** fiables y rápidas, y es igualmente útil para automatizar tareas web y extraer datos de páginas que dependen de JavaScript.

**Ventajas:**
- **Esperas automáticas**: antes de cada acción espera a que el elemento sea visible, estable y esté habilitado.
- **Aserciones con reintento** (`expect`): eliminan la mayoría de pruebas inestables.
- Localizadores orientados al usuario (`get_by_role`, `get_by_label`, `get_by_text`).
- Control total de la **red**: esperar respuestas, interceptar, modificar o simular peticiones.
- **Contextos** aislados y ligeros (como perfiles de incógnito) para paralelizar sin abrir varios navegadores.
- Herramientas: generador de código (*codegen*), inspector y visor de trazas.
- Navegadores gestionados por la propia librería (sin controladores externos).
- API síncrona y asíncrona.

**Limitaciones:**
- Usa sus propias versiones parcheadas de los navegadores (aunque admite Chrome y Edge instalados).
- No implementa el estándar WebDriver: no es compatible con Selenium Grid.
- Descarga varios cientos de MB de navegadores.
- Algunas funciones (PDF, ciertas emulaciones) solo están disponibles en Chromium.

| Si necesita… | Opción |
|---|---|
| Pruebas E2E modernas y fiables, control de red, trazas | **Playwright** |
| Estándar WebDriver, Grid o compatibilidad con infraestructura existente | Selenium |
| Extraer datos de HTML estático | Requests/httpx + BeautifulSoup |
| Rastreo masivo con páginas dinámicas | Scrapy + scrapy-playwright |

---

## 2. Instalación

```bash
pip install playwright
playwright install                     # Descarga Chromium, Firefox y WebKit
playwright install chromium            # Solo Chromium
playwright install --with-deps chromium   # Linux: también las dependencias del sistema
pip install pytest-playwright          # Integración con pytest (opcional)
```

Usar el Chrome o Edge instalado en el equipo:

```python
navegador = p.chromium.launch(channel="chrome")      # o "msedge"
```

> Los navegadores se guardan en una caché compartida (`~/.cache/ms-playwright` en Linux, `%USERPROFILE%\AppData\Local\ms-playwright` en Windows). Tras actualizar la librería, ejecute de nuevo `playwright install`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Playwright` | Punto de entrada (`sync_playwright()` / `async_playwright()`). |
| `Browser` | Instancia del navegador (Chromium, Firefox o WebKit). |
| `BrowserContext` | Sesión aislada (cookies, almacenamiento, permisos). Equivale a un perfil de incógnito. |
| `Page` | Pestaña dentro de un contexto. |
| `Locator` | Forma de encontrar elementos **de manera diferida**: se resuelve en el momento de actuar, con reintentos. |
| *Auto-waiting* | Espera automática a que el elemento sea accionable antes de cada acción. |
| `expect` | Aserciones que reintentan hasta cumplirse o agotar el tiempo. |
| `Route` | Intercepción de una petición de red para continuarla, modificarla, simularla o bloquearla. |
| `storage_state` | Cookies y `localStorage` guardados para reutilizar una sesión iniciada. |
| Traza (*trace*) | Grabación de una ejecución (acciones, DOM, red, consola) para depurar. |

```
Playwright
└── Browser (chromium)
    ├── BrowserContext A (usuario 1)  ── Page 1, Page 2
    └── BrowserContext B (usuario 2)  ── Page 1
```

---

## 4. Primer script (API síncrona)

```python
from playwright.sync_api import sync_playwright, expect

with sync_playwright() as p:
    navegador = p.chromium.launch(headless=True)           # headless=False para ver el navegador
    pagina = navegador.new_page()
    pagina.goto("https://playwright.dev/python/")

    print(pagina.title())
    pagina.get_by_role("link", name="Docs").click()
    expect(pagina).to_have_url("https://playwright.dev/python/docs/intro")
    expect(pagina.get_by_role("heading", level=1)).to_contain_text("Installation")

    pagina.screenshot(path="docs.png", full_page=True)
    navegador.close()
```

Opciones útiles de `launch`:

```python
p.chromium.launch(headless=False, slow_mo=300)          # Ver y ralentizar (depuración)
p.firefox.launch()
p.webkit.launch()
p.chromium.launch(args=["--start-maximized"], downloads_path="descargas")
```

---

## 5. API asíncrona

```python
import asyncio
from playwright.async_api import async_playwright, expect

async def main():
    async with async_playwright() as p:
        navegador = await p.chromium.launch()
        pagina = await navegador.new_page()
        await pagina.goto("https://example.com")
        await expect(pagina.get_by_role("heading")).to_have_text("Example Domain")
        print(await pagina.title())
        await navegador.close()

asyncio.run(main())
```

La API asíncrona permite procesar varias páginas en paralelo:

```python
async def titulo(contexto, url):
    pagina = await contexto.new_page()
    await pagina.goto(url)
    resultado = await pagina.title()
    await pagina.close()
    return resultado

titulos = await asyncio.gather(*(titulo(contexto, u) for u in urls))
```

> Use la API síncrona en scripts y pruebas sencillas, y la asíncrona en aplicaciones `asyncio` o para paralelizar. **No** use la API síncrona dentro de un bucle `asyncio` en ejecución (por ejemplo, en Jupyter o FastAPI).

---

## 6. Navegador, contexto y página

```python
contexto = navegador.new_context(
    viewport={"width": 1366, "height": 900},
    locale="es-ES",
    timezone_id="Europe/Madrid",
    user_agent="MiBot/1.0",
    base_url="http://127.0.0.1:5000",         # Permite page.goto("/ruta")
    storage_state="sesion.json",              # Sesión guardada (sección 11)
    ignore_https_errors=True,                 # Certificados no válidos (entornos de prueba)
    extra_http_headers={"X-Entorno": "pruebas"},
    accept_downloads=True,
    record_video_dir="videos/",
    color_scheme="dark",
)
contexto.set_default_timeout(10_000)          # Milisegundos para acciones y esperas
contexto.set_default_navigation_timeout(30_000)
pagina = contexto.new_page()

pagina.goto("/catalogo", wait_until="domcontentloaded")   # load | domcontentloaded | networkidle | commit
pagina.reload(); pagina.go_back(); pagina.go_forward()
pagina.url, pagina.title(), pagina.content()              # URL, título, HTML actual
pagina.close(); contexto.close(); navegador.close()
```

> Cree un **contexto nuevo por prueba o por usuario**: es barato (milisegundos) y garantiza el aislamiento de cookies y almacenamiento.

---

## 7. Localizadores

Los localizadores son **diferidos y estrictos**: no buscan el elemento hasta que se usan, reintentan hasta que aparece y fallan si coinciden con varios elementos en una acción que espera uno solo.

### 7.1 Localizadores recomendados (orientados al usuario)

```python
pagina.get_by_role("button", name="Guardar")                     # Rol ARIA + nombre accesible
pagina.get_by_role("link", name="Contacto", exact=True)
pagina.get_by_role("heading", name="Catálogo", level=2)
pagina.get_by_role("checkbox", name="Acepto las condiciones")
pagina.get_by_role("combobox", name="Categoría")                 # <select>
pagina.get_by_label("Contraseña")                                # Campo asociado a una etiqueta
pagina.get_by_placeholder("Nombre del producto")
pagina.get_by_text("Pedido confirmado")                          # Texto visible (subcadena)
pagina.get_by_text("Total", exact=True)
pagina.get_by_alt_text("Logotipo")
pagina.get_by_title("Cerrar")
pagina.get_by_test_id("contador")                                # data-testid="contador"
```

| Prioridad | Localizador | Motivo |
|---|---|---|
| 1 | `get_by_role` | Refleja cómo percibe la página un usuario o un lector de pantalla. |
| 2 | `get_by_label`, `get_by_placeholder`, `get_by_text` | Basados en textos visibles. |
| 3 | `get_by_test_id` | Atributo pensado para pruebas, muy estable. |
| 4 | `locator("css=…")` / `locator("xpath=…")` | Cuando no hay alternativa. |

### 7.2 CSS, XPath y combinaciones

```python
pagina.locator("#lista-productos .producto")
pagina.locator("xpath=//button[normalize-space()='Añadir']")
pagina.locator(".producto").filter(has_text="Teclado")                         # Filtrar por texto
pagina.locator(".producto").filter(has=pagina.get_by_role("button", name="Añadir"))
pagina.locator(".producto").filter(has_not_text="Agotado")
pagina.locator(".producto").first / .last / .nth(2)
pagina.locator(".producto", has_text="Hub").get_by_role("button")              # Encadenar
pagina.get_by_role("button").and_(pagina.get_by_title("Suscribirse"))          # Ambas condiciones
pagina.get_by_role("button", name="Aceptar").or_(pagina.get_by_role("button", name="Accept"))
```

### 7.3 Leer información

```python
fila = pagina.locator(".producto").first
fila.text_content()                  # Texto (incluido oculto)
fila.inner_text()                    # Texto visible
fila.inner_html()
fila.get_attribute("data-id")
pagina.get_by_label("Email").input_value()
fila.is_visible(), fila.is_enabled(), fila.is_checked(), fila.is_editable()
pagina.locator(".producto").count()
pagina.locator(".producto .nombre").all_inner_texts()      # Lista de textos
for producto in pagina.locator(".producto").all():          # Lista de localizadores
    print(producto.locator(".precio").inner_text())
```

> `all()`, `count()` y `all_inner_texts()` **no esperan**: devuelven lo que hay en ese instante. Antes, espere a que la lista esté cargada (por ejemplo, con `expect(locator).to_have_count(6)` o `locator.first.wait_for()`).

---

## 8. Acciones

```python
pagina.get_by_label("Usuario").fill("demo")                  # Vacía y escribe
pagina.get_by_label("Buscar").press_sequentially("usb", delay=50)   # Tecla a tecla (dispara eventos de teclado)
pagina.get_by_label("Buscar").press("Enter")
pagina.keyboard.press("Control+A")
pagina.get_by_role("button", name="Entrar").click()
pagina.get_by_text("Fila 3").dblclick()
pagina.get_by_text("Fila 3").click(button="right")
pagina.get_by_text("Fila 3").click(modifiers=["Shift"])
pagina.get_by_role("checkbox", name="Acepto").check()        # uncheck(), set_checked(True)
pagina.get_by_label("Categoría").select_option("Accesorios")  # Por valor o etiqueta; también label=, index=
pagina.get_by_role("menuitem", name="Archivo").hover()
pagina.locator("#origen").drag_to(pagina.locator("#destino"))
pagina.get_by_label("Comentario").clear()
pagina.locator("#pie").scroll_into_view_if_needed()
pagina.mouse.wheel(0, 800)
pagina.get_by_role("button", name="Guardar").click(force=True)   # Omite comprobaciones (último recurso)
```

Antes de cada acción, Playwright comprueba automáticamente que el elemento:

| Comprobación | `click` | `fill` | `check` | `hover` |
|---|---|---|---|---|
| Está adjunto al DOM | ✔ | ✔ | ✔ | ✔ |
| Es visible | ✔ | ✔ | ✔ | ✔ |
| Es estable (sin animación) | ✔ | — | ✔ | ✔ |
| Recibe eventos (no está tapado) | ✔ | — | ✔ | ✔ |
| Está habilitado | ✔ | ✔ | ✔ | — |
| Es editable | — | ✔ | — | — |

---

## 9. Esperas automáticas y aserciones (`expect`)

### 9.1 Aserciones con reintento

```python
from playwright.sync_api import expect

expect(pagina).to_have_title("Tienda demo")
expect(pagina).to_have_url(re.compile(r".*/catalogo"))
expect(pagina.get_by_role("alert")).to_have_text("Usuario o contraseña incorrectos")
expect(pagina.get_by_test_id("contador")).to_have_text("2")
expect(pagina.locator(".producto")).to_have_count(6)
expect(pagina.get_by_text("Cargando")).to_be_hidden()
expect(pagina.get_by_role("button", name="Tramitar")).to_be_enabled()
expect(pagina.get_by_role("checkbox")).to_be_checked()
expect(pagina.get_by_label("Email")).to_have_value("ana@ejemplo.com")
expect(pagina.locator(".producto .nombre")).to_have_text(["Hub USB-C", "Webcam"])   # Lista ordenada
expect(pagina.locator("#modal")).to_have_attribute("role", "dialog")
expect(pagina.locator(".aviso")).to_have_class(re.compile("exito"))
expect(pagina.get_by_role("button")).not_to_be_disabled()                         # Negación: not_to_…
expect(pagina.locator(".producto")).to_have_count(6, timeout=15_000)
```

| Aserción | Comprueba |
|---|---|
| `to_be_visible` / `to_be_hidden` | Visibilidad. |
| `to_be_enabled` / `to_be_disabled` / `to_be_editable` | Estado. |
| `to_be_checked` / `to_be_focused` / `to_be_empty` | Estado de formularios y foco. |
| `to_have_text` / `to_contain_text` | Texto exacto / contenido (admite listas y expresiones regulares). |
| `to_have_value` / `to_have_values` | Valor de campos. |
| `to_have_count` | Número de elementos. |
| `to_have_attribute` / `to_have_class` / `to_have_css` / `to_have_id` | Atributos y estilos. |
| `to_have_url` / `to_have_title` | Página. |
| `to_have_accessible_name` / `to_have_role` | Accesibilidad. |

> Use `expect` en lugar de `assert locator.inner_text() == …`: el `assert` lee el valor una sola vez y falla si la página aún no se ha actualizado.

### 9.2 Esperas explícitas (cuando no hay una acción o aserción)

```python
pagina.get_by_text("Cargando").wait_for(state="hidden")         # attached | detached | visible | hidden
pagina.wait_for_url("**/catalogo")
pagina.wait_for_load_state("networkidle")
pagina.wait_for_function("() => window.datosListos === true")
pagina.wait_for_timeout(500)                                    # Pausa fija: solo para depurar
```

---

## 10. Red: esperar, interceptar y simular peticiones

### 10.1 Esperar una respuesta concreta

```python
with pagina.expect_response("**/api/productos") as info:
    pagina.get_by_role("button", name="Entrar").click()
respuesta = info.value
print(respuesta.status, respuesta.json())                     # Datos que recibió la página

with pagina.expect_request(lambda r: r.url.endswith("/api/pedidos") and r.method == "POST") as info:
    pagina.get_by_role("button", name="Confirmar").click()
print(info.value.post_data_json)                              # Cuerpo enviado
```

### 10.2 Escuchar todo el tráfico

```python
pagina.on("request", lambda r: print(">>", r.method, r.url))
pagina.on("response", lambda r: print("<<", r.status, r.url))
pagina.on("requestfailed", lambda r: print("✖", r.url, r.failure))
pagina.on("console", lambda m: print("consola:", m.type, m.text))
pagina.on("pageerror", lambda e: print("error JS:", e))
```

### 10.3 Interceptar y simular (*mock*)

```python
# Simular la respuesta de una API
pagina.route("**/api/productos", lambda ruta: ruta.fulfill(
    status=200, json=[{"id": 99, "nombre": "Producto simulado", "precio": 1.0, "categoria": "Pruebas"}]))

# Simular un error del servidor
pagina.route("**/api/pedidos", lambda ruta: ruta.fulfill(status=503, json={"error": "mantenimiento"}))

# Modificar la respuesta real
def anadir_producto(ruta):
    respuesta = ruta.fetch()                              # Petición real
    datos = respuesta.json()
    datos.append({"id": 100, "nombre": "Extra", "precio": 5.0, "categoria": "Pruebas"})
    ruta.fulfill(response=respuesta, json=datos)
pagina.route("**/api/productos", anadir_producto)

# Modificar la petición (cabeceras, cuerpo)
pagina.route("**/api/**", lambda ruta: ruta.continue_(headers={**ruta.request.headers, "X-Prueba": "1"}))

# Bloquear recursos (acelera el scraping)
pagina.route(re.compile(r"\.(png|jpe?g|gif|webp|svg|woff2?)$"), lambda ruta: ruta.abort())
contexto.route("**/*", lambda r: r.abort() if r.request.resource_type in {"image", "font", "media"} else r.continue_())

pagina.unroute("**/api/productos")
```

### 10.4 Reproducir tráfico grabado (HAR)

```python
contexto = navegador.new_context(record_har_path="trafico.har")           # Grabar
...
contexto.close()
pagina.route_from_har("trafico.har", url="**/api/**")                     # Reproducir sin red
```

---

## 11. Autenticación y reutilización de sesión

Inicie sesión **una vez**, guarde el estado y reutilícelo en el resto de contextos y pruebas:

```python
# Iniciar sesión y guardar
contexto = navegador.new_context()
pagina = contexto.new_page()
pagina.goto("https://app.ejemplo.com/login")
pagina.get_by_label("Usuario").fill(os.environ["APP_USUARIO"])
pagina.get_by_label("Contraseña").fill(os.environ["APP_CLAVE"])
pagina.get_by_role("button", name="Entrar").click()
expect(pagina.get_by_text("Mi cuenta")).to_be_visible()
contexto.storage_state(path=".auth/usuario.json")       # Cookies + localStorage

# Reutilizar
contexto = navegador.new_context(storage_state=".auth/usuario.json")
```

> Añada la carpeta `.auth/` al `.gitignore`: contiene credenciales de sesión.

Perfil persistente (como un navegador normal, útil para automatizaciones personales):

```python
contexto = p.chromium.launch_persistent_context("perfil_playwright", headless=False)
```

Autenticación HTTP básica: `navegador.new_context(http_credentials={"username": "u", "password": "c"})`.

---

## 12. Diálogos, *iframes*, pestañas y ventanas emergentes

### 12.1 Diálogos (`alert`, `confirm`, `prompt`)

Por defecto, Playwright **descarta** automáticamente los diálogos. Para responderlos:

```python
pagina.once("dialog", lambda d: d.accept())                      # Aceptar el siguiente diálogo
pagina.once("dialog", lambda d: d.dismiss())                     # Cancelar
pagina.once("dialog", lambda d: d.accept("texto para el prompt"))

def registrar_dialogo(d):
    print(d.type, d.message)
    d.accept()
pagina.on("dialog", registrar_dialogo)
```

> Registre el manejador **antes** de la acción que abre el diálogo.

### 12.2 *iframes*

```python
marco = pagina.frame_locator("iframe#pago")
marco.get_by_label("Número de tarjeta").fill("4242 4242 4242 4242")
marco.get_by_role("button", name="Pagar").click()
```

### 12.3 Pestañas y ventanas emergentes

```python
with contexto.expect_page() as info:
    pagina.get_by_role("link", name="Abrir informe").click()     # target=_blank
nueva = info.value
nueva.wait_for_load_state()
print(nueva.title())

with pagina.expect_popup() as info:
    pagina.get_by_role("button", name="Ayuda").click()           # window.open
ayuda = info.value
```

---

## 13. Archivos: subidas y descargas

```python
pagina.get_by_label("Adjuntar").set_input_files("documentos/informe.pdf")
pagina.get_by_label("Adjuntar").set_input_files(["a.png", "b.png"])
pagina.get_by_label("Adjuntar").set_input_files([])                          # Quitar archivos
pagina.get_by_label("Adjuntar").set_input_files(
    files=[{"name": "datos.csv", "mimeType": "text/csv", "buffer": b"a;b\n1;2\n"}])   # Desde memoria

# Selector de archivos abierto por un botón personalizado
with pagina.expect_file_chooser() as info:
    pagina.get_by_role("button", name="Subir").click()
info.value.set_files("foto.jpg")

# Descargas
with pagina.expect_download() as info:
    pagina.get_by_role("button", name="Exportar CSV").click()
descarga = info.value
descarga.save_as(f"descargas/{descarga.suggested_filename}")
```

---

## 14. Capturas, PDF y vídeo

```python
pagina.screenshot(path="pagina.png")                         # Parte visible
pagina.screenshot(path="completa.png", full_page=True)       # Página completa
pagina.locator("#resumen").screenshot(path="resumen.png")    # Un elemento
pagina.screenshot(path="sin_datos.png", mask=[pagina.locator(".email")])   # Ocultar datos sensibles
imagen = pagina.screenshot()                                 # Bytes

pagina.pdf(path="informe.pdf", format="A4", print_background=True,
           margin={"top": "1cm", "bottom": "1cm"})           # Solo Chromium

contexto = navegador.new_context(record_video_dir="videos/", record_video_size={"width": 1280, "height": 720})
...
contexto.close()                                             # El vídeo se guarda al cerrar
print(pagina.video.path())
```

Comparación visual en pruebas: `expect(pagina).to_have_screenshot()` no está disponible en Python; use librerías como `pytest-playwright-visual` o compare imágenes con Pillow.

---

## 15. Emulación: dispositivos, idioma, geolocalización y reloj

```python
iphone = p.devices["iPhone 13"]
contexto = navegador.new_context(**iphone, locale="es-ES")    # Viewport, user agent, táctil…

contexto = navegador.new_context(
    geolocation={"latitude": 40.4168, "longitude": -3.7038}, permissions=["geolocation"],
    timezone_id="Europe/Madrid", locale="es-ES", color_scheme="dark", reduced_motion="reduce",
)
contexto.set_offline(True)                                    # Simular sin conexión
pagina.emulate_media(media="print")                           # Estilos de impresión

# Controlar el tiempo de la página
pagina.clock.install(time=datetime(2026, 12, 24, 23, 59, 50))
pagina.goto("/")
pagina.clock.fast_forward("00:20")                            # Avanzar 20 segundos
```

---

## 16. JavaScript en la página

```python
total = pagina.evaluate("() => document.querySelectorAll('.producto').length")
datos = pagina.evaluate("selector => [...document.querySelectorAll(selector)].map(e => e.textContent)", ".nombre")
pagina.locator("#lista").evaluate("el => el.scrollTop = el.scrollHeight")
precios = pagina.locator(".precio").evaluate_all("els => els.map(e => e.textContent)")
pagina.add_init_script("window.localStorage.setItem('cookies_aceptadas', 'si')")   # Antes de cargar cada página
pagina.expose_function("registrar", lambda texto: print("desde la página:", texto))
```

---

## 17. Pruebas con pytest-playwright

```bash
pip install pytest-playwright
pytest                                         # Chromium, headless
pytest --headed --slowmo 300                   # Ver el navegador
pytest --browser firefox --browser webkit      # Varios navegadores
pytest --tracing retain-on-failure             # Guardar trazas de las pruebas fallidas (test-results/)
pytest --video retain-on-failure --screenshot only-on-failure
pytest --base-url http://127.0.0.1:5000
pytest -n 4                                    # En paralelo (pytest-xdist)
```

El plugin proporciona *fixtures* listas: `page`, `context`, `browser`, `browser_name`, `playwright`.

```python
import re
from playwright.sync_api import Page, expect

def test_titulo(page: Page):
    page.goto("https://playwright.dev/python/")
    expect(page).to_have_title(re.compile("Playwright"))
```

Personalizar el contexto para todas las pruebas:

```python
# conftest.py
import pytest

@pytest.fixture(scope="session")
def browser_context_args(browser_context_args):
    return {**browser_context_args, "locale": "es-ES", "viewport": {"width": 1366, "height": 900}}
```

---

## 18. Pruebas de API (`APIRequestContext`)

Playwright incluye un cliente HTTP que comparte cookies con el navegador (útil para preparar datos o comprobar el *backend*):

```python
api = p.request.new_context(base_url="http://127.0.0.1:5000", extra_http_headers={"Accept": "application/json"})
r = api.post("/api/login", data={"usuario": "demo", "clave": "demo1234"})
assert r.ok
productos = api.get("/api/productos").json()
api.dispose()

contexto.request.get("/api/perfil")       # Mismo almacenamiento de cookies que las páginas del contexto
pagina.request.post("/api/pedidos", data={...})
```

---

## 19. Herramientas: *codegen*, inspector y *trace viewer*

```bash
playwright codegen https://demo.playwright.dev/todomvc          # Graba sus acciones y genera código Python
playwright codegen --target python-pytest --save-storage=auth.json https://app.ejemplo.com
PWDEBUG=1 pytest -k compra                                       # Inspector paso a paso (Windows: set PWDEBUG=1)
playwright show-trace traza.zip                                  # Visor de trazas
playwright screenshot --full-page https://example.com captura.png
playwright pdf https://example.com pagina.pdf
```

Grabar una traza desde el código:

```python
contexto.tracing.start(screenshots=True, snapshots=True, sources=True)
...
contexto.tracing.stop(path="traza.zip")
```

El visor muestra, para cada acción, una instantánea del DOM antes y después, la red, la consola y el código fuente: es la herramienta más útil para diagnosticar fallos en CI.

Pausar la ejecución e inspeccionar: `pagina.pause()`.

---

## 20. *Scraping* de páginas dinámicas

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    navegador = p.chromium.launch()
    contexto = navegador.new_context(locale="es-ES", user_agent="MiScraper/1.0 (+contacto@ejemplo.com)")
    contexto.route("**/*", lambda r: r.abort() if r.request.resource_type in {"image", "font", "media"} else r.continue_())
    pagina = contexto.new_page()

    # Opción 1: capturar directamente la API JSON que usa la página (lo más fiable)
    with pagina.expect_response(lambda r: "/api/productos" in r.url and r.ok) as info:
        pagina.goto("https://tienda.ejemplo.com/catalogo")
    productos = info.value.json()

    # Opción 2: leer el DOM ya renderizado
    pagina.locator(".producto").first.wait_for()
    filas = pagina.locator(".producto").evaluate_all(
        "els => els.map(e => ({nombre: e.querySelector('.nombre').textContent, precio: e.querySelector('.precio').textContent}))")

    # Desplazamiento infinito
    anterior = -1
    while (actual := pagina.locator(".producto").count()) != anterior:
        anterior = actual
        pagina.mouse.wheel(0, 5000)
        pagina.wait_for_timeout(800)

    navegador.close()
```

> Combine Playwright con BeautifulSoup si lo prefiere: `BeautifulSoup(pagina.content(), "lxml")`. Respete `robots.txt`, las condiciones de uso y una frecuencia de acceso razonable.

---

## 21. Ejecución en CI y Docker

```yaml
# .github/workflows/e2e.yml (GitHub Actions)
- uses: actions/setup-python@v5
  with: { python-version: "3.13" }
- run: pip install -r requirements.txt
- run: playwright install --with-deps chromium
- run: pytest --tracing retain-on-failure
- uses: actions/upload-artifact@v4
  if: failure()
  with: { name: trazas, path: test-results/ }
```

```dockerfile
FROM mcr.microsoft.com/playwright/python:v1.55.0-noble   # Use la etiqueta de su versión de Playwright
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["pytest"]
```

---

## 22. Ejemplo práctico completo

**Automatización y pruebas de una tienda web con Playwright**: incluye una aplicación de demostración (Flask + JavaScript con carga asíncrona, filtros, carrito, formulario validado, diálogo modal y `confirm()`), un **script** que inicia sesión una sola vez y reutiliza la sesión, extrae el catálogo capturando la respuesta de la API, realiza una compra completa registrando la petición enviada y genera capturas, PDF y una traza; y una batería de **pruebas pytest-playwright** con localizadores por rol, aserciones `expect`, simulación de errores de red y emulación móvil.

### 22.1 Aplicación de demostración (`tienda_demo.py`)

```python
"""
tienda_demo.py — Tienda web de demostración para practicar automatización de navegadores.
Requiere: pip install flask
Uso: python tienda_demo.py      → http://127.0.0.1:5000  (usuario: demo · contraseña: demo1234)
"""
from flask import Flask, jsonify, request

app = Flask(__name__)
PRODUCTOS = [
    {"id": 1, "nombre": "Teclado mecánico", "precio": 89.90, "categoria": "Periféricos"},
    {"id": 2, "nombre": "Ratón inalámbrico", "precio": 29.50, "categoria": "Periféricos"},
    {"id": 3, "nombre": "Monitor 27\" 4K", "precio": 329.00, "categoria": "Monitores"},
    {"id": 4, "nombre": "Hub USB-C", "precio": 39.90, "categoria": "Accesorios"},
    {"id": 5, "nombre": "Webcam Full HD", "precio": 59.00, "categoria": "Accesorios"},
    {"id": 6, "nombre": "Auriculares", "precio": 74.99, "categoria": "Audio"},
]

PAGINA = """<!DOCTYPE html>
<html lang="es"><head><meta charset="utf-8"><title>Tienda demo</title>
<style>
 body{font-family:system-ui,sans-serif;max-width:860px;margin:auto;padding:1rem}
 .oculto{display:none} .producto{border:1px solid #ddd;padding:.6rem;margin:.4rem 0;display:flex;justify-content:space-between}
 .error{color:#C00000} #aviso{background:#E8F4EA;padding:.5rem} #modal{position:fixed;inset:30% 30%;background:#fff;border:2px solid #1F4E78;padding:1rem}
</style></head>
<body>
<header><h1>Tienda demo</h1><span id="usuario-actual"></span>
  <button id="salir" class="oculto">Salir</button> · Carrito: <span id="contador-carrito" data-testid="contador">0</span></header>

<section id="vista-login">
  <h2>Iniciar sesión</h2>
  <form id="form-login">
    <label>Usuario <input id="usuario" name="usuario" autocomplete="username"></label>
    <label>Contraseña <input id="clave" name="clave" type="password"></label>
    <button type="submit" id="entrar">Entrar</button>
  </form>
  <p id="error-login" class="error" role="alert"></p>
</section>

<section id="vista-catalogo" class="oculto">
  <h2>Catálogo</h2>
  <label>Buscar <input id="busqueda" placeholder="Nombre del producto"></label>
  <label>Categoría <select id="categoria"><option value="">Todas</option></select></label>
  <label>Ordenar <select id="orden"><option value="nombre">Nombre</option><option value="precio-asc">Precio ascendente</option><option value="precio-desc">Precio descendente</option></select></label>
  <p id="cargando">Cargando productos…</p>
  <div id="lista-productos" aria-live="polite"></div>
  <button id="ir-pago" disabled>Tramitar pedido</button>
</section>

<section id="vista-pago" class="oculto">
  <h2>Datos de envío</h2>
  <ul id="resumen-carrito"></ul><p>Total: <strong id="total">0,00 €</strong></p>
  <form id="form-pago" novalidate>
    <label>Nombre <input id="nombre" required></label>
    <label>Email <input id="email" type="email" required></label>
    <label>Código postal <input id="cp" pattern="\\d{5}" required></label>
    <label><input type="checkbox" id="acepto"> Acepto las condiciones</label>
    <button type="submit" id="confirmar">Confirmar pedido</button>
  </form>
  <p id="error-pago" class="error" role="alert"></p>
</section>

<div id="modal" class="oculto" role="dialog"><h2>¡Pedido confirmado!</h2><p>Número de pedido: <strong id="num-pedido"></strong></p><button id="cerrar-modal">Aceptar</button></div>

<script>
const $ = s => document.querySelector(s);
let productos = [], carrito = [];
const euros = n => n.toFixed(2).replace(".", ",") + " €";
function mostrar(v){ for (const id of ["vista-login","vista-catalogo","vista-pago"]) $("#"+id).classList.toggle("oculto", id!==v); }
function pintar(){
  const q = $("#busqueda").value.toLowerCase(), cat = $("#categoria").value, orden = $("#orden").value;
  let lista = productos.filter(p => p.nombre.toLowerCase().includes(q) && (!cat || p.categoria === cat));
  lista.sort((a,b) => orden==="precio-asc" ? a.precio-b.precio : orden==="precio-desc" ? b.precio-a.precio : a.nombre.localeCompare(b.nombre));
  $("#lista-productos").innerHTML = lista.map(p => `<div class="producto" data-id="${p.id}">
     <span class="nombre">${p.nombre}</span> <span class="precio">${euros(p.precio)}</span>
     <button class="anadir" data-id="${p.id}">Añadir</button></div>`).join("") || "<p id='sin-resultados'>Sin resultados</p>";
}
async function cargar(){
  $("#cargando").classList.remove("oculto");
  const r = await fetch("/api/productos"); productos = await r.json();
  const cats = [...new Set(productos.map(p => p.categoria))];
  $("#categoria").innerHTML = '<option value="">Todas</option>' + cats.map(c => `<option>${c}</option>`).join("");
  $("#cargando").classList.add("oculto"); pintar();
}
$("#form-login").addEventListener("submit", async e => {
  e.preventDefault();
  const r = await fetch("/api/login", {method:"POST", headers:{"Content-Type":"application/json"},
    body: JSON.stringify({usuario: $("#usuario").value, clave: $("#clave").value})});
  if (!r.ok) { $("#error-login").textContent = "Usuario o contraseña incorrectos"; return; }
  $("#error-login").textContent = ""; $("#usuario-actual").textContent = "Hola, " + $("#usuario").value;
  $("#salir").classList.remove("oculto"); mostrar("vista-catalogo"); cargar();
});
$("#busqueda").addEventListener("input", pintar); $("#categoria").addEventListener("change", pintar); $("#orden").addEventListener("change", pintar);
$("#lista-productos").addEventListener("click", e => {
  if (!e.target.matches(".anadir")) return;
  carrito.push(productos.find(p => p.id == e.target.dataset.id));
  $("#contador-carrito").textContent = carrito.length; $("#ir-pago").disabled = false;
  e.target.textContent = "Añadido ✓";
});
$("#ir-pago").addEventListener("click", () => {
  $("#resumen-carrito").innerHTML = carrito.map(p => `<li>${p.nombre} — ${euros(p.precio)}</li>`).join("");
  $("#total").textContent = euros(carrito.reduce((s,p) => s+p.precio, 0)); mostrar("vista-pago");
});
$("#form-pago").addEventListener("submit", async e => {
  e.preventDefault();
  if (!$("#nombre").value.trim()) return $("#error-pago").textContent = "Indique su nombre";
  if (!$("#email").checkValidity()) return $("#error-pago").textContent = "Email no válido";
  if (!/^\\d{5}$/.test($("#cp").value)) return $("#error-pago").textContent = "Código postal no válido";
  if (!$("#acepto").checked) return $("#error-pago").textContent = "Debe aceptar las condiciones";
  $("#error-pago").textContent = ""; $("#confirmar").disabled = true;
  const r = await fetch("/api/pedidos", {method:"POST", headers:{"Content-Type":"application/json"},
     body: JSON.stringify({nombre: $("#nombre").value, email: $("#email").value, productos: carrito.map(p => p.id)})});
  const d = await r.json(); $("#num-pedido").textContent = d.numero; $("#modal").classList.remove("oculto");
});
$("#cerrar-modal").addEventListener("click", () => { $("#modal").classList.add("oculto"); carrito = [];
  $("#contador-carrito").textContent = 0; $("#ir-pago").disabled = true; $("#confirmar").disabled = false; mostrar("vista-catalogo"); pintar(); });
$("#salir").addEventListener("click", () => { if (confirm("¿Cerrar sesión?")) location.reload(); });
</script></body></html>"""

pedidos = []

@app.get("/")
def inicio():
    return PAGINA

@app.post("/api/login")
def login():
    datos = request.get_json(silent=True) or {}
    ok = datos.get("usuario") == "demo" and datos.get("clave") == "demo1234"
    return (jsonify(ok=True), 200) if ok else (jsonify(ok=False), 401)

@app.get("/api/productos")
def productos():
    import time
    time.sleep(0.8)                       # Simula una API lenta: obliga a usar esperas
    return jsonify(PRODUCTOS)

@app.post("/api/pedidos")
def crear_pedido():
    datos = request.get_json(silent=True) or {}
    numero = f"PED-{1000 + len(pedidos) + 1}"
    pedidos.append({**datos, "numero": numero})
    return jsonify(numero=numero), 201

if __name__ == "__main__":
    app.run(port=5000, debug=False)
```

### 22.2 Script de automatización (`automatizar_tienda.py`)

```python
"""
automatizar_tienda.py — Automatiza la tienda de demostración con Playwright.
Requiere: pip install playwright flask && playwright install chromium
Uso:
    python tienda_demo.py                       # En otra terminal
    python automatizar_tienda.py [--visible] [--url http://127.0.0.1:5000]
Genera en ./resultado: catalogo.json, catalogo.png, pedido.pdf, traza.zip
"""
import argparse
import json
import sys
from pathlib import Path

from playwright.sync_api import Browser, Error as PlaywrightError, expect, sync_playwright

SALIDA = Path("resultado")
SESION = SALIDA / ".sesion.json"


def iniciar_sesion(navegador: Browser, url: str, usuario: str, clave: str) -> None:
    """Inicia sesión una vez y guarda el estado (cookies y localStorage)."""
    contexto = navegador.new_context(base_url=url, locale="es-ES")
    pagina = contexto.new_page()
    pagina.goto("/")
    pagina.get_by_label("Usuario").fill(usuario)
    pagina.get_by_label("Contraseña").fill(clave)
    pagina.get_by_role("button", name="Entrar").click()
    error = pagina.get_by_role("alert").first
    catalogo = pagina.get_by_role("heading", name="Catálogo")
    expect(catalogo.or_(error)).to_be_visible()
    if error.is_visible() and error.inner_text():
        raise RuntimeError(f"No se pudo iniciar sesión: {error.inner_text()}")
    contexto.storage_state(path=SESION)
    contexto.close()


def extraer_catalogo(pagina) -> list[dict]:
    """Captura la respuesta JSON de la API y la contrasta con lo que muestra la página."""
    with pagina.expect_response(lambda r: r.url.endswith("/api/productos") and r.ok) as info:
        pagina.get_by_role("button", name="Entrar").click()
    productos = info.value.json()

    expect(pagina.get_by_text("Cargando productos…")).to_be_hidden()
    tarjetas = pagina.locator("#lista-productos .producto")
    expect(tarjetas).to_have_count(len(productos))
    visibles = tarjetas.evaluate_all("""els => els.map(e => ({
        id: Number(e.dataset.id),
        nombre: e.querySelector('.nombre').textContent,
        precio_texto: e.querySelector('.precio').textContent }))""")
    por_id = {p["id"]: p for p in productos}
    for v in visibles:
        por_id[v["id"]]["precio_texto"] = v["precio_texto"]
    return productos


def comprar(pagina, nombres: list[str], datos_envio: dict) -> dict:
    for nombre in nombres:
        tarjeta = pagina.locator(".producto").filter(has_text=nombre)
        tarjeta.get_by_role("button", name="Añadir").click()
        expect(tarjeta.get_by_role("button")).to_have_text("Añadido ✓")
    expect(pagina.get_by_test_id("contador")).to_have_text(str(len(nombres)))

    pagina.get_by_role("button", name="Tramitar pedido").click()
    expect(pagina.get_by_role("heading", name="Datos de envío")).to_be_visible()
    total = pagina.locator("#total").inner_text()

    pagina.get_by_label("Nombre", exact=True).fill(datos_envio["nombre"])   # exact: «Ordenar» también contiene «Nombre»
    pagina.get_by_label("Email").fill(datos_envio["email"])
    pagina.get_by_label("Código postal").fill(datos_envio["cp"])
    pagina.get_by_role("checkbox", name="Acepto las condiciones").check()

    with pagina.expect_request(lambda r: r.url.endswith("/api/pedidos") and r.method == "POST") as peticion, \
            pagina.expect_response("**/api/pedidos") as respuesta:
        pagina.get_by_role("button", name="Confirmar pedido").click()

    dialogo = pagina.get_by_role("dialog")
    expect(dialogo).to_be_visible()
    expect(dialogo).to_contain_text("PED-")
    return {"numero": respuesta.value.json()["numero"], "total": total,
            "enviado": peticion.value.post_data_json}


def main() -> int:
    parser = argparse.ArgumentParser(description="Automatización de la tienda de demostración.")
    parser.add_argument("--url", default="http://127.0.0.1:5000")
    parser.add_argument("--visible", action="store_true", help="mostrar el navegador")
    args = parser.parse_args()
    SALIDA.mkdir(exist_ok=True)

    with sync_playwright() as p:
        navegador = p.chromium.launch(headless=not args.visible, slow_mo=250 if args.visible else 0)
        try:
            iniciar_sesion(navegador, args.url, "demo", "demo1234")
            print("✔ Sesión iniciada y guardada")

            contexto = navegador.new_context(base_url=args.url, locale="es-ES", storage_state=SESION,
                                             viewport={"width": 1366, "height": 900})
            contexto.set_default_timeout(10_000)
            contexto.tracing.start(screenshots=True, snapshots=True)
            pagina = contexto.new_page()
            pagina.on("pageerror", lambda e: print("⚠ Error de JavaScript:", e))

            # La tienda de demostración guarda la sesión en memoria, así que se rellena el formulario;
            # en una aplicación real con cookies, storage_state permitiría saltar este paso.
            pagina.goto("/")
            pagina.get_by_label("Usuario").fill("demo")
            pagina.get_by_label("Contraseña").fill("demo1234")
            productos = extraer_catalogo(pagina)
            (SALIDA / "catalogo.json").write_text(json.dumps(productos, ensure_ascii=False, indent=2), encoding="utf-8")
            pagina.screenshot(path=SALIDA / "catalogo.png", full_page=True)
            print(f"✔ Catálogo: {len(productos)} productos → {SALIDA / 'catalogo.json'}")

            baratos = sorted(productos, key=lambda x: x["precio"])[:2]
            pedido = comprar(pagina, [b["nombre"] for b in baratos],
                             {"nombre": "Ana Pérez", "email": "ana@ejemplo.com", "cp": "28001"})
            print(f"✔ Pedido {pedido['numero']} por {pedido['total']} (productos {pedido['enviado']['productos']})")

            pagina.emulate_media(media="print")
            pagina.pdf(path=SALIDA / "pedido.pdf", format="A4", print_background=True)

            pagina.once("dialog", lambda d: d.accept())                # confirm() de «Salir»
            pagina.get_by_role("button", name="Aceptar").click()
            pagina.get_by_role("button", name="Salir").click()
            expect(pagina.get_by_role("heading", name="Iniciar sesión")).to_be_visible()
            print("✔ Sesión cerrada")

            contexto.tracing.stop(path=SALIDA / "traza.zip")
            contexto.close()
            print(f"Resultados en {SALIDA}/ · Ver la traza: playwright show-trace {SALIDA / 'traza.zip'}")
            return 0
        except (PlaywrightError, RuntimeError, AssertionError) as e:
            print(f"✖ {e}", file=sys.stderr)
            return 1
        finally:
            navegador.close()


if __name__ == "__main__":
    sys.exit(main())
```

### 22.3 Pruebas con pytest-playwright (`conftest.py` y `test_tienda_pw.py`)

```python
# conftest.py
import threading

import pytest
from werkzeug.serving import make_server

from tienda_demo import app


@pytest.fixture(scope="session")
def base_url():
    """Arranca la tienda en un hilo; pytest-playwright usa base_url en page.goto('/')."""
    servidor = make_server("127.0.0.1", 5012, app)
    threading.Thread(target=servidor.serve_forever, daemon=True).start()
    yield "http://127.0.0.1:5012"
    servidor.shutdown()


@pytest.fixture(scope="session")
def browser_context_args(browser_context_args):
    return {**browser_context_args, "locale": "es-ES", "viewport": {"width": 1366, "height": 900}}


@pytest.fixture
def catalogo(page):
    page.goto("/")
    page.get_by_label("Usuario").fill("demo")
    page.get_by_label("Contraseña").fill("demo1234")
    page.get_by_role("button", name="Entrar").click()
    page.get_by_text("Cargando productos…").wait_for(state="hidden")
    return page
```

```python
# test_tienda_pw.py
import re

import pytest
from playwright.sync_api import Page, expect


def test_login_incorrecto(page: Page):
    page.goto("/")
    page.get_by_label("Usuario").fill("demo")
    page.get_by_label("Contraseña").fill("incorrecta")
    page.get_by_role("button", name="Entrar").click()
    expect(page.get_by_role("alert").first).to_have_text("Usuario o contraseña incorrectos")


def test_catalogo_y_busqueda(catalogo: Page):
    productos = catalogo.locator("#lista-productos .producto")
    expect(productos).to_have_count(6)
    catalogo.get_by_label("Buscar").fill("usb")
    expect(productos).to_have_count(1)
    expect(productos.locator(".nombre")).to_have_text(["Hub USB-C"])
    catalogo.get_by_label("Buscar").fill("zzz")
    expect(catalogo.get_by_text("Sin resultados")).to_be_visible()


def test_orden_por_precio(catalogo: Page):
    catalogo.get_by_label("Ordenar").select_option(label="Precio ascendente")
    precios = catalogo.locator(".producto .precio").all_inner_texts()
    valores = [float(p.replace("€", "").replace(",", ".")) for p in precios]
    assert valores == sorted(valores)


def test_compra_con_validaciones(catalogo: Page):
    for nombre in ["Teclado mecánico", "Hub USB-C"]:
        catalogo.locator(".producto").filter(has_text=nombre).get_by_role("button", name="Añadir").click()
    expect(catalogo.get_by_test_id("contador")).to_have_text("2")
    catalogo.get_by_role("button", name="Tramitar pedido").click()
    expect(catalogo.locator("#total")).to_have_text("129,80 €")

    error = catalogo.locator("#error-pago")
    catalogo.get_by_role("button", name="Confirmar pedido").click()
    expect(error).to_have_text("Indique su nombre")
    catalogo.get_by_label("Nombre", exact=True).fill("Ana")
    catalogo.get_by_label("Email").fill("ana.ejemplo.com")
    catalogo.get_by_role("button", name="Confirmar pedido").click()
    expect(error).to_have_text("Email no válido")
    catalogo.get_by_label("Email").fill("ana@ejemplo.com")
    catalogo.get_by_label("Código postal").fill("28001")
    catalogo.get_by_role("button", name="Confirmar pedido").click()
    expect(error).to_have_text("Debe aceptar las condiciones")
    catalogo.get_by_role("checkbox", name="Acepto las condiciones").check()
    catalogo.get_by_role("button", name="Confirmar pedido").click()
    expect(catalogo.get_by_role("dialog")).to_contain_text(re.compile(r"PED-\d+"))


def test_api_simulada(page: Page):
    """La página muestra exactamente lo que devuelve la API (simulada con page.route)."""
    page.route("**/api/productos", lambda r: r.fulfill(json=[
        {"id": 1, "nombre": "Producto de prueba", "precio": 9.99, "categoria": "Pruebas"}]))
    page.goto("/")
    page.get_by_label("Usuario").fill("demo")
    page.get_by_label("Contraseña").fill("demo1234")
    page.get_by_role("button", name="Entrar").click()
    expect(page.locator(".producto")).to_have_count(1)
    expect(page.locator(".producto .precio")).to_have_text("9,99 €")
    expect(page.get_by_label("Categoría")).to_contain_text("Pruebas")


def test_cancelar_cierre_de_sesion(catalogo: Page):
    catalogo.once("dialog", lambda d: d.dismiss())
    catalogo.get_by_role("button", name="Salir").click()
    expect(catalogo.get_by_role("heading", name="Catálogo")).to_be_visible()


@pytest.mark.parametrize("dispositivo", ["iPhone 13", "Pixel 7"])
def test_movil(playwright, browser, base_url, dispositivo):
    contexto = browser.new_context(**playwright.devices[dispositivo], base_url=base_url, locale="es-ES")
    pagina = contexto.new_page()
    pagina.goto("/")
    expect(pagina.get_by_role("button", name="Entrar")).to_be_visible()
    contexto.close()
```

### 22.4 Ejecución

```bash
pip install playwright pytest-playwright flask
playwright install chromium
python tienda_demo.py &                               # Windows: ejecútelo en otra terminal
python automatizar_tienda.py                          # Script de automatización
pytest -v                                             # Pruebas (arrancan su propia instancia de la tienda)
pytest --headed --slowmo 400 -k compra                # Ver una prueba
pytest --tracing retain-on-failure && playwright show-trace test-results/*/trace.zip
```

---

## 23. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `Executable doesn't exist at …/ms-playwright/…` | No se han descargado los navegadores (o se actualizó la librería). | `playwright install`. |
| `Host system is missing dependencies` (Linux) | Faltan librerías del sistema. | `playwright install-deps` o `--with-deps`. |
| `strict mode violation: … resolved to N elements` | El localizador coincide con varios elementos (p. ej., `get_by_label("Nombre")` también encuentra una lista cuya etiqueta contiene «Nombre»). | Hágalo más específico: `exact=True`, `name=`, `filter`, `first`, `nth`. |
| `TimeoutError: … waiting for …` | El elemento no apareció, está oculto o tapado. | Revise el localizador con el inspector; espere al estado previo correcto. |
| `It looks like you are using Playwright Sync API inside the asyncio loop` | API síncrona en Jupyter, FastAPI u otro bucle activo. | Use la API asíncrona (`async_playwright`). |
| El diálogo `confirm()` se cierra solo | Playwright descarta los diálogos sin manejador. | `page.once("dialog", lambda d: d.accept())` antes de la acción. |
| `assert` falla de forma intermitente | Lectura única antes de que la página se actualice. | Use `expect(...)` con reintento. |
| `count()` o `all()` devuelven 0 | No esperan a que carguen los elementos. | Espere antes con `expect(...).to_have_count()` o `wait_for()`. |
| `page.pdf()` falla | Solo disponible en Chromium (sin interfaz). | Use Chromium en modo *headless*. |
| `Target page, context or browser has been closed` | Se usó un objeto después de cerrarlo. | Revise el orden de cierre y los bloques `with`. |
| Las pruebas pasan en local y fallan en CI | Tiempos, tamaño de ventana, idioma o datos distintos. | Fije `viewport`, `locale`, `timezone_id`; use trazas para diagnosticar. |

---

## 24. Buenas prácticas

- Use localizadores orientados al usuario (`get_by_role`, `get_by_label`) o `get_by_test_id`; evite CSS/XPath frágiles.
- Confíe en las esperas automáticas y en `expect`; evite `wait_for_timeout` salvo para depurar.
- Cree un contexto nuevo por prueba y reutilice la sesión con `storage_state`.
- Simule las APIs externas con `page.route` para obtener pruebas rápidas y deterministas.
- Grabe trazas en los fallos (`--tracing retain-on-failure`) y revíselas con `show-trace`.
- Fije `viewport`, `locale` y `timezone_id` para resultados reproducibles.
- Organice las pruebas con *fixtures* (y *Page Objects* si el proyecto es grande).
- Use *codegen* para descubrir localizadores, pero revise y simplifique el código generado.
- Para extraer datos, capture las respuestas JSON en lugar de leer el DOM cuando sea posible, y bloquee imágenes y fuentes.
- No guarde credenciales ni archivos de sesión en el repositorio.

---

## 25. Referencia rápida

```python
from playwright.sync_api import sync_playwright, expect
with sync_playwright() as p:
    b = p.chromium.launch(headless=True, slow_mo=0)        # firefox / webkit / channel="chrome"
    ctx = b.new_context(base_url=, locale="es-ES", viewport={}, storage_state=, **p.devices["iPhone 13"])
    page = ctx.new_page(); page.goto("/ruta")
page.get_by_role("button", name="X") · get_by_label · get_by_placeholder · get_by_text · get_by_test_id · locator("css")
loc.filter(has_text=) · .first · .nth(i) · .count() · .all() · .inner_text() · .get_attribute() · .evaluate_all()
loc.click() · fill() · press("Enter") · check() · select_option() · hover() · set_input_files() · drag_to()
expect(loc).to_be_visible() · to_have_text() · to_have_count() · to_have_value() · expect(page).to_have_url()
with page.expect_response("**/api/x") as r: ... · page.route("**/api", lambda r: r.fulfill(json=...)) · r.abort() · r.continue_()
page.once("dialog", lambda d: d.accept()) · page.frame_locator("iframe") · with ctx.expect_page() as nueva: ...
with page.expect_download() as d: ... · page.screenshot(path=, full_page=True) · page.pdf(path=) · ctx.tracing.start()/stop(path=)
ctx.storage_state(path="estado.json") · p.request.new_context(base_url=) · page.clock.install(time=)
playwright install · playwright codegen URL · playwright show-trace traza.zip · PWDEBUG=1 · pytest --headed --browser firefox
```

---

## 26. Ejercicios propuestos

1. Use `playwright codegen` para grabar una búsqueda en un sitio público y convierta el resultado en una prueba con `expect`.
2. Escriba un script asíncrono que abra 10 páginas en paralelo dentro de un mismo contexto y obtenga su título y el número de enlaces.
3. Añada a las pruebas de la tienda un caso que simule que `/api/pedidos` responde con error 503 y compruebe el comportamiento de la página.
4. Extraiga los datos de una página con desplazamiento infinito bloqueando imágenes y fuentes, y compare el tiempo con y sin bloqueo.
5. Configure un flujo de GitHub Actions que ejecute las pruebas en Chromium y Firefox y publique las trazas de las pruebas fallidas.

---

## 27. Referencias

- Documentación oficial (Python): https://playwright.dev/python/
- Localizadores: https://playwright.dev/python/docs/locators
- Aserciones: https://playwright.dev/python/docs/test-assertions
- Red: https://playwright.dev/python/docs/network
- pytest-playwright: https://playwright.dev/python/docs/test-runners
- Trace viewer: https://playwright.dev/python/docs/trace-viewer
