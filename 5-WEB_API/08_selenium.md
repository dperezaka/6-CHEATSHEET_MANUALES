# Manual de uso: Selenium

> **Categoría:** Desarrollo web · **Utilidad:** Automatizar navegadores
> **Tipo:** Librería de terceros (`pip install selenium`) · **Versión de referencia:** 4.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y controladores](#2-instalación-y-controladores)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer script](#4-primer-script)
5. [Opciones del navegador](#5-opciones-del-navegador)
6. [Localizar elementos](#6-localizar-elementos)
7. [Interactuar con elementos](#7-interactuar-con-elementos)
8. [Esperas](#8-esperas)
9. [Formularios: listas, casillas y archivos](#9-formularios-listas-casillas-y-archivos)
10. [Acciones avanzadas: ratón y teclado](#10-acciones-avanzadas-ratón-y-teclado)
11. [Alertas, marcos (*iframes*), ventanas y pestañas](#11-alertas-marcos-iframes-ventanas-y-pestañas)
12. [JavaScript, desplazamiento y capturas](#12-javascript-desplazamiento-y-capturas)
13. [Cookies, almacenamiento y sesiones](#13-cookies-almacenamiento-y-sesiones)
14. [Descargas](#14-descargas)
15. [Patrón *Page Object*](#15-patrón-page-object)
16. [Pruebas con pytest](#16-pruebas-con-pytest)
17. [Selenium Grid y ejecución remota](#17-selenium-grid-y-ejecución-remota)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**Selenium** es el estándar histórico para **automatizar navegadores web** reales (Chrome, Firefox, Edge, Safari) mediante el protocolo **WebDriver**, una especificación del W3C. Se usa principalmente para **pruebas de extremo a extremo** (E2E) de aplicaciones web, y también para automatizar tareas repetitivas en sitios web y extraer datos de páginas que dependen de JavaScript.

**Ventajas:**
- Estándar W3C: compatible con todos los navegadores principales.
- Disponible en muchos lenguajes (Python, Java, C#, JavaScript, Ruby).
- **Selenium Manager** descarga automáticamente el controlador adecuado.
- **Selenium Grid** para ejecutar pruebas en paralelo y en varias máquinas o navegadores.
- Enorme comunidad, documentación y soporte en servicios en la nube (BrowserStack, Sauce Labs…).

**Limitaciones:**
- Sin esperas automáticas: hay que gestionar explícitamente la sincronización (fuente habitual de pruebas inestables).
- Más lento y verboso que Playwright.
- Interceptar o simular peticiones de red requiere funciones adicionales (BiDi/CDP).
- Depende de controladores externos (aunque Selenium Manager lo simplifica).

| Si necesita… | Opción |
|---|---|
| Pruebas E2E con el estándar WebDriver, Grid o varios lenguajes | **Selenium** |
| Automatización moderna con esperas automáticas, red y trazas | Playwright |
| Extraer datos de HTML estático | Requests/httpx + BeautifulSoup |
| Rastreo masivo de sitios | Scrapy |

---

## 2. Instalación y controladores

```bash
pip install selenium
```

Desde la versión 4.6, **Selenium Manager** (incluido) localiza el navegador instalado y descarga automáticamente el controlador compatible (`chromedriver`, `geckodriver`, `msedgedriver`) la primera vez:

```python
from selenium import webdriver
driver = webdriver.Chrome()        # Sin configurar rutas
```

Controlador concreto (redes sin Internet, versiones fijadas):

```python
from selenium.webdriver.chrome.service import Service

servicio = Service(executable_path="/opt/drivers/chromedriver", log_output="chromedriver.log")
driver = webdriver.Chrome(service=servicio)
```

> La versión **principal** del controlador debe coincidir con la del navegador (por ejemplo, Chrome 153 ↔ ChromeDriver 153). Si no coinciden, aparece `SessionNotCreatedException`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| WebDriver | Protocolo y API para controlar el navegador. |
| `driver` | Objeto que representa la sesión del navegador. |
| Controlador (*driver* ejecutable) | Programa intermedio (`chromedriver`…) entre Selenium y el navegador. |
| `WebElement` | Elemento de la página (botón, campo…). |
| Localizador (`By`) | Estrategia para encontrar elementos: ID, CSS, XPath… |
| Espera explícita | Esperar a que se cumpla una condición (`WebDriverWait` + `expected_conditions`). |
| Espera implícita | Tiempo global de reintento al buscar elementos. |
| `ActionChains` | Secuencias de acciones de ratón y teclado. |
| *Headless* | Navegador sin interfaz gráfica. |
| *Page Object* | Patrón que encapsula cada página en una clase. |
| Grid | Infraestructura para ejecutar sesiones remotas y en paralelo. |

---

## 4. Primer script

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
try:
    driver.get("https://www.python.org")
    print(driver.title)

    buscador = driver.find_element(By.NAME, "q")
    buscador.send_keys("asyncio", Keys.ENTER)

    WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.CSS_SELECTOR, "ul.list-recent-events li")))
    for resultado in driver.find_elements(By.CSS_SELECTOR, "ul.list-recent-events li h3 a")[:5]:
        print("-", resultado.text, resultado.get_attribute("href"))
finally:
    driver.quit()                     # Cierra el navegador y el controlador
```

> Llame siempre a `driver.quit()` (en un `finally` o con `with`): si no, quedan procesos del navegador abiertos.

```python
with webdriver.Chrome() as driver:    # quit() automático al salir del bloque
    driver.get("https://example.com")
```

---

## 5. Opciones del navegador

### 5.1 Chrome / Edge

```python
opciones = webdriver.ChromeOptions()
opciones.add_argument("--headless=new")             # Sin ventana
opciones.add_argument("--window-size=1366,900")
opciones.add_argument("--lang=es-ES")
opciones.add_argument("--disable-gpu")
opciones.add_argument("--no-sandbox")                # Necesario en algunos contenedores Docker
opciones.add_argument("--incognito")
opciones.add_argument("--user-data-dir=/tmp/perfil-selenium")   # Perfil persistente
opciones.add_experimental_option("prefs", {
    "download.default_directory": "/tmp/descargas",
    "download.prompt_for_download": False,
    "profile.default_content_setting_values.notifications": 2,
})
opciones.page_load_strategy = "eager"                # normal | eager | none
opciones.binary_location = "/ruta/a/chrome"          # Navegador concreto

driver = webdriver.Chrome(options=opciones)          # Edge: webdriver.Edge(options=webdriver.EdgeOptions())
```

### 5.2 Firefox

```python
opciones = webdriver.FirefoxOptions()
opciones.add_argument("-headless")
opciones.set_preference("intl.accept_languages", "es-ES")
driver = webdriver.Firefox(options=opciones)
```

### 5.3 Tiempos y ventana

```python
driver.set_page_load_timeout(30)
driver.set_script_timeout(10)
driver.set_window_size(1280, 800)
driver.maximize_window()
```

| `page_load_strategy` | Espera hasta… |
|---|---|
| `normal` (defecto) | Carga completa (`load`). |
| `eager` | DOM listo (`DOMContentLoaded`). |
| `none` | Nada; solo la respuesta inicial. |

---

## 6. Localizar elementos

```python
from selenium.webdriver.common.by import By

driver.find_element(By.ID, "usuario")
driver.find_element(By.NAME, "q")
driver.find_element(By.CSS_SELECTOR, "form#login button[type='submit']")
driver.find_element(By.XPATH, "//button[normalize-space()='Entrar']")
driver.find_element(By.LINK_TEXT, "Contacto")
driver.find_element(By.PARTIAL_LINK_TEXT, "Conta")
driver.find_element(By.CLASS_NAME, "precio")             # Una sola clase
driver.find_element(By.TAG_NAME, "h1")

driver.find_elements(By.CSS_SELECTOR, ".producto")        # Lista (vacía si no hay)
```

| Método | Si no encuentra nada |
|---|---|
| `find_element` | Lanza `NoSuchElementException`. |
| `find_elements` | Devuelve `[]`. |

Búsquedas dentro de un elemento:

```python
tarjeta = driver.find_element(By.CSS_SELECTOR, ".producto[data-id='3']")
tarjeta.find_element(By.CSS_SELECTOR, ".precio").text
```

Localizadores relativos (Selenium 4):

```python
from selenium.webdriver.support.relative_locator import locate_with

etiqueta = driver.find_element(By.XPATH, "//label[text()='Email']")
campo = driver.find_element(locate_with(By.TAG_NAME, "input").to_right_of(etiqueta))
```

### 6.1 Elegir buenos localizadores

| Prioridad | Localizador | Motivo |
|---|---|---|
| 1 | `data-testid`, `id`, `name` | Estables y pensados para ello. |
| 2 | CSS con clases semánticas | Legibles y rápidos. |
| 3 | XPath por texto visible | Útil cuando no hay otra opción (`//button[text()='Guardar']`). |
| ✖ | XPath absolutos (`/html/body/div[3]/…`) | Se rompen con cualquier cambio de diseño. |

> Pida al equipo de desarrollo atributos `data-testid` en los elementos que se prueban: es la forma más robusta.

---

## 7. Interactuar con elementos

```python
campo = driver.find_element(By.ID, "email")
campo.clear()
campo.send_keys("ana@ejemplo.com")
campo.send_keys(Keys.TAB)
boton.click()
formulario.submit()                         # Envía el formulario que contiene al elemento

elemento.text                               # Texto visible
elemento.get_attribute("value")             # Valor actual de un input
elemento.get_attribute("href")
elemento.get_dom_attribute("data-id")       # Atributo tal como está en el HTML
elemento.get_property("checked")            # Propiedad JavaScript
elemento.value_of_css_property("color")
elemento.is_displayed(), elemento.is_enabled(), elemento.is_selected()
elemento.tag_name, elemento.location, elemento.size, elemento.rect
elemento.screenshot("elemento.png")
```

Teclas especiales: `Keys.ENTER`, `Keys.TAB`, `Keys.ESCAPE`, `Keys.BACKSPACE`, `Keys.DELETE`, `Keys.ARROW_DOWN`, `Keys.PAGE_DOWN`, `Keys.CONTROL`, `Keys.SHIFT`, `Keys.COMMAND`.

```python
campo.send_keys(Keys.CONTROL, "a")          # Seleccionar todo (Windows/Linux)
campo.send_keys(Keys.BACKSPACE)
```

---

## 8. Esperas

Las páginas modernas cargan contenido de forma asíncrona. **La causa número uno de pruebas inestables es no esperar correctamente.**

### 8.1 Espera explícita (recomendada)

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

espera = WebDriverWait(driver, timeout=10, poll_frequency=0.25)

boton = espera.until(EC.element_to_be_clickable((By.ID, "guardar")))
boton.click()

espera.until(EC.visibility_of_element_located((By.CSS_SELECTOR, ".aviso")))
espera.until(EC.invisibility_of_element_located((By.ID, "cargando")))
espera.until(EC.text_to_be_present_in_element((By.ID, "estado"), "Completado"))
espera.until(EC.url_contains("/panel"))
espera.until(EC.title_is("Panel de control"))
espera.until(EC.number_of_windows_to_be(2))
espera.until(EC.alert_is_present())
espera.until(EC.staleness_of(elemento_antiguo))        # El elemento se ha eliminado o recargado
espera.until(lambda d: len(d.find_elements(By.CSS_SELECTOR, ".fila")) >= 10)   # Condición propia
```

| Condición (`EC.`) | Espera a que… |
|---|---|
| `presence_of_element_located` | Exista en el DOM (aunque no sea visible). |
| `visibility_of_element_located` | Exista y sea visible. |
| `element_to_be_clickable` | Sea visible y esté habilitado. |
| `invisibility_of_element_located` | Desaparezca o se oculte. |
| `text_to_be_present_in_element` | Contenga un texto. |
| `text_to_be_present_in_element_value` | El `value` contenga un texto. |
| `element_to_be_selected` | Esté seleccionado. |
| `frame_to_be_available_and_switch_to_it` | El *iframe* esté disponible (y cambia a él). |
| `url_to_be`, `url_contains`, `url_matches` | La URL cumpla la condición. |
| `any_of(...)`, `all_of(...)`, `none_of(...)` | Combinaciones de condiciones. |

Ignorar excepciones mientras se espera:

```python
from selenium.common.exceptions import StaleElementReferenceException
WebDriverWait(driver, 10, ignored_exceptions=[StaleElementReferenceException]).until(...)
```

### 8.2 Espera implícita

```python
driver.implicitly_wait(5)       # find_element reintenta hasta 5 s
```

> No mezcle esperas implícitas y explícitas: los tiempos se pueden sumar de forma impredecible. Prefiera **solo explícitas**.

### 8.3 `time.sleep`

Evítelo: hace las pruebas lentas (si espera de más) o inestables (si espera de menos). Úselo solo para depurar.

---

## 9. Formularios: listas, casillas y archivos

```python
from selenium.webdriver.support.ui import Select

lista = Select(driver.find_element(By.ID, "pais"))
lista.select_by_visible_text("España")
lista.select_by_value("es")
lista.select_by_index(2)
lista.first_selected_option.text
[o.text for o in lista.options]
lista.deselect_all()                                   # Solo en <select multiple>

casilla = driver.find_element(By.ID, "acepto")
if not casilla.is_selected():
    casilla.click()

driver.find_element(By.CSS_SELECTOR, "input[name='tipo'][value='empresa']").click()   # Botón de opción

driver.find_element(By.CSS_SELECTOR, "input[type='file']").send_keys("/ruta/absoluta/documento.pdf")  # Subir archivo
```

> Para subir archivos, envíe la **ruta absoluta** al `<input type="file">`, aunque esté oculto tras un botón personalizado.

---

## 10. Acciones avanzadas: ratón y teclado

```python
from selenium.webdriver import ActionChains

acciones = ActionChains(driver)
acciones.move_to_element(menu).click(submenu).perform()              # Menú desplegable al pasar el ratón
acciones.double_click(celda).perform()
acciones.context_click(fila).perform()                                # Clic derecho
acciones.drag_and_drop(origen, destino).perform()
acciones.click_and_hold(control).move_by_offset(100, 0).release().perform()   # Deslizador
acciones.key_down(Keys.CONTROL).click(fila1).click(fila3).key_up(Keys.CONTROL).perform()  # Selección múltiple
acciones.scroll_to_element(pie).perform()
acciones.scroll_by_amount(0, 600).perform()
acciones.send_keys(Keys.ESCAPE).perform()
```

> Recuerde terminar la cadena con `.perform()`: sin él, no se ejecuta nada.

---

## 11. Alertas, marcos (*iframes*), ventanas y pestañas

### 11.1 Alertas (`alert`, `confirm`, `prompt`)

```python
alerta = WebDriverWait(driver, 5).until(EC.alert_is_present())
print(alerta.text)
alerta.accept()                 # Aceptar
# alerta.dismiss()              # Cancelar
# alerta.send_keys("texto")     # En un prompt
```

### 11.2 *iframes*

```python
WebDriverWait(driver, 10).until(EC.frame_to_be_available_and_switch_to_it((By.CSS_SELECTOR, "iframe#pago")))
driver.find_element(By.ID, "numero-tarjeta").send_keys("4242…")
driver.switch_to.default_content()          # Volver al documento principal
driver.switch_to.parent_frame()             # Subir un nivel
```

### 11.3 Ventanas y pestañas

```python
original = driver.current_window_handle
driver.find_element(By.LINK_TEXT, "Abrir informe").click()
WebDriverWait(driver, 10).until(EC.number_of_windows_to_be(2))
nueva = next(h for h in driver.window_handles if h != original)
driver.switch_to.window(nueva)
print(driver.title)
driver.close()                               # Cierra la pestaña actual
driver.switch_to.window(original)

driver.switch_to.new_window("tab")           # Abrir una pestaña nueva ('window' para ventana)
```

---

## 12. JavaScript, desplazamiento y capturas

```python
driver.execute_script("return document.readyState")                          # 'complete'
driver.execute_script("arguments[0].scrollIntoView({block: 'center'});", elemento)
driver.execute_script("arguments[0].click();", elemento)                      # Último recurso si algo tapa el botón
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
total = driver.execute_script("return document.querySelectorAll('.producto').length;")
driver.execute_async_script("const fin = arguments[arguments.length - 1]; setTimeout(() => fin('ok'), 500);")

driver.save_screenshot("pagina.png")                 # Parte visible
elemento.screenshot("tabla.png")                     # Un elemento
base64 = driver.get_screenshot_as_base64()
html = driver.page_source                            # HTML actual (tras ejecutar JavaScript)
```

Página completa en Firefox: `driver.save_full_page_screenshot("completa.png")`. Imprimir a PDF (Chrome/Firefox):

```python
from selenium.webdriver.common.print_page_options import PrintOptions
import base64
pdf = driver.print_page(PrintOptions())
open("pagina.pdf", "wb").write(base64.b64decode(pdf))
```

Desplazamiento infinito (cargar más contenido al bajar):

```python
altura = driver.execute_script("return document.body.scrollHeight")
while True:
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    try:
        WebDriverWait(driver, 5).until(lambda d: d.execute_script("return document.body.scrollHeight") > altura)
    except TimeoutException:
        break
    altura = driver.execute_script("return document.body.scrollHeight")
```

---

## 13. Cookies, almacenamiento y sesiones

```python
driver.get_cookies()
driver.get_cookie("sesion")
driver.add_cookie({"name": "consentimiento", "value": "si", "path": "/"})   # Tras visitar el dominio
driver.delete_cookie("sesion")
driver.delete_all_cookies()

driver.execute_script("return window.localStorage.getItem('token');")
driver.execute_script("window.localStorage.setItem('tema', 'oscuro');")
```

Reutilizar una sesión iniciada (evita repetir el *login* en cada prueba):

```python
import json
json.dump(driver.get_cookies(), open("cookies.json", "w"))
# En otra ejecución:
driver.get("https://app.ejemplo.com")
for c in json.load(open("cookies.json")):
    driver.add_cookie(c)
driver.refresh()
```

---

## 14. Descargas

```python
from pathlib import Path

carpeta = Path("descargas").resolve()
carpeta.mkdir(exist_ok=True)
opciones = webdriver.ChromeOptions()
opciones.add_experimental_option("prefs", {"download.default_directory": str(carpeta),
                                           "download.prompt_for_download": False})
opciones.enable_downloads = True                      # Necesario en Grid / remoto
driver = webdriver.Chrome(options=opciones)

driver.find_element(By.ID, "exportar-csv").click()
WebDriverWait(driver, 30).until(
    lambda d: any(p.suffix == ".csv" for p in carpeta.iterdir()) and not any(p.suffix == ".crdownload" for p in carpeta.iterdir()))
```

> A menudo es más sencillo y fiable descargar el archivo con Requests reutilizando las cookies de la sesión de Selenium.

---

## 15. Patrón *Page Object*

Encapsule cada página (o componente) en una clase con sus localizadores y acciones. Las pruebas quedan legibles y, si cambia la página, solo se modifica la clase.

```python
class PaginaLogin:
    USUARIO = (By.ID, "usuario")
    CLAVE = (By.ID, "clave")
    ENTRAR = (By.ID, "entrar")
    ERROR = (By.ID, "error-login")

    def __init__(self, driver, url_base: str):
        self.driver, self.url_base = driver, url_base
        self.espera = WebDriverWait(driver, 10)

    def abrir(self) -> "PaginaLogin":
        self.driver.get(self.url_base)
        return self

    def entrar(self, usuario: str, clave: str) -> "PaginaCatalogo":
        self.driver.find_element(*self.USUARIO).send_keys(usuario)
        self.driver.find_element(*self.CLAVE).send_keys(clave)
        self.driver.find_element(*self.ENTRAR).click()
        return PaginaCatalogo(self.driver)

    def mensaje_error(self) -> str:
        return self.espera.until(EC.visibility_of_element_located(self.ERROR)).text
```

```python
def test_login(driver):
    catalogo = PaginaLogin(driver, URL).abrir().entrar("demo", "demo1234")
    assert catalogo.esta_cargado()
```

---

## 16. Pruebas con pytest

```python
# conftest.py
import pytest
from selenium import webdriver

@pytest.fixture
def driver(request):
    opciones = webdriver.ChromeOptions()
    opciones.add_argument("--headless=new")
    opciones.add_argument("--window-size=1366,900")
    d = webdriver.Chrome(options=opciones)
    yield d
    d.quit()

@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    """Guarda una captura de pantalla cuando una prueba falla."""
    resultado = (yield).get_result()
    if resultado.when == "call" and resultado.failed and "driver" in item.fixturenames:
        item.funcargs["driver"].save_screenshot(f"fallo_{item.name}.png")
```

Ejecución en paralelo: `pip install pytest-xdist` y `pytest -n 4` (cada proceso abre su propio navegador).

---

## 17. Selenium Grid y ejecución remota

```bash
# Grid local con Docker
docker run -d -p 4444:4444 --shm-size=2g selenium/standalone-chrome
# Interfaz: http://localhost:4444
```

```python
opciones = webdriver.ChromeOptions()
driver = webdriver.Remote(command_executor="http://localhost:4444", options=opciones)
```

El mismo código funciona con servicios en la nube (BrowserStack, Sauce Labs, LambdaTest), indicando su URL y credenciales en las opciones.

---

## 18. Ejemplo práctico completo

**Pruebas E2E de una tienda web** con el patrón *Page Object*: inicio de sesión correcto e incorrecto, carga asíncrona del catálogo, búsqueda, filtros y ordenación, carrito, validación del formulario de pago, confirmación en un diálogo modal y cierre de sesión con `confirm()`. Incluye una pequeña aplicación de demostración (Flask + JavaScript) para poder ejecutar las pruebas en local, *fixtures* de pytest, capturas en caso de fallo y modo visible o *headless*.

### 18.1 Aplicación de demostración (`tienda_demo.py`)

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

### 18.2 *Page Objects* (`paginas.py`)

```python
"""paginas.py — Page Objects de la tienda de demostración."""
from __future__ import annotations

from decimal import Decimal

from selenium.common.exceptions import TimeoutException
from selenium.webdriver.common.by import By
from selenium.webdriver.remote.webdriver import WebDriver
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import Select, WebDriverWait


def euros(texto: str) -> Decimal:
    return Decimal(texto.replace("€", "").replace(".", "").replace(",", ".").strip())


class PaginaBase:
    CONTADOR = (By.CSS_SELECTOR, "[data-testid='contador']")
    SALIR = (By.ID, "salir")

    def __init__(self, driver: WebDriver, timeout: float = 10):
        self.driver = driver
        self.espera = WebDriverWait(driver, timeout)

    def visible(self, localizador):
        return self.espera.until(EC.visibility_of_element_located(localizador))

    def clicable(self, localizador):
        return self.espera.until(EC.element_to_be_clickable(localizador))

    def escribir(self, localizador, texto: str) -> None:
        campo = self.visible(localizador)
        campo.clear()
        campo.send_keys(texto)

    @property
    def unidades_carrito(self) -> int:
        return int(self.driver.find_element(*self.CONTADOR).text)

    def salir(self, confirmar: bool = True) -> None:
        self.clicable(self.SALIR).click()
        alerta = self.espera.until(EC.alert_is_present())
        alerta.accept() if confirmar else alerta.dismiss()


class PaginaLogin(PaginaBase):
    USUARIO = (By.ID, "usuario")
    CLAVE = (By.ID, "clave")
    ENTRAR = (By.ID, "entrar")
    ERROR = (By.ID, "error-login")

    def abrir(self, url: str) -> PaginaLogin:
        self.driver.get(url)
        self.visible(self.USUARIO)
        return self

    def entrar(self, usuario: str, clave: str) -> PaginaCatalogo:
        self.escribir(self.USUARIO, usuario)
        self.escribir(self.CLAVE, clave)
        self.clicable(self.ENTRAR).click()
        return PaginaCatalogo(self.driver)

    def error(self) -> str:
        self.espera.until(EC.text_to_be_present_in_element(self.ERROR, "incorrectos"))
        return self.driver.find_element(*self.ERROR).text


class PaginaCatalogo(PaginaBase):
    VISTA = (By.ID, "vista-catalogo")
    CARGANDO = (By.ID, "cargando")
    PRODUCTOS = (By.CSS_SELECTOR, "#lista-productos .producto")
    BUSQUEDA = (By.ID, "busqueda")
    CATEGORIA = (By.ID, "categoria")
    ORDEN = (By.ID, "orden")
    SIN_RESULTADOS = (By.ID, "sin-resultados")
    IR_PAGO = (By.ID, "ir-pago")

    def esperar_carga(self) -> PaginaCatalogo:
        self.visible(self.VISTA)
        self.espera.until(EC.invisibility_of_element_located(self.CARGANDO))
        self.espera.until(lambda d: d.find_elements(*self.PRODUCTOS))
        return self

    def productos(self) -> list[dict]:
        return [{"id": int(p.get_attribute("data-id")),
                 "nombre": p.find_element(By.CLASS_NAME, "nombre").text,
                 "precio": euros(p.find_element(By.CLASS_NAME, "precio").text)}
                for p in self.driver.find_elements(*self.PRODUCTOS)]

    def buscar(self, texto: str) -> PaginaCatalogo:
        antes = self.driver.find_element(By.ID, "lista-productos").get_attribute("innerHTML")
        self.escribir(self.BUSQUEDA, texto)
        self.espera.until(lambda d: d.find_element(By.ID, "lista-productos").get_attribute("innerHTML") != antes)
        return self

    def filtrar_categoria(self, categoria: str) -> PaginaCatalogo:
        Select(self.visible(self.CATEGORIA)).select_by_visible_text(categoria)
        return self

    def ordenar(self, texto_opcion: str) -> PaginaCatalogo:
        Select(self.visible(self.ORDEN)).select_by_visible_text(texto_opcion)
        return self

    def hay_resultados(self) -> bool:
        return not self.driver.find_elements(*self.SIN_RESULTADOS)

    def anadir(self, nombre: str) -> PaginaCatalogo:
        antes = self.unidades_carrito
        boton = self.clicable((By.XPATH, f"//div[@class='producto'][span[@class='nombre' and text()={nombre!r}]]"
                                         "//button[contains(@class,'anadir')]"))
        boton.click()
        self.espera.until(lambda d: self.unidades_carrito == antes + 1)
        return self

    def tramitar(self) -> PaginaPago:
        self.clicable(self.IR_PAGO).click()
        return PaginaPago(self.driver).esperar()


class PaginaPago(PaginaBase):
    VISTA = (By.ID, "vista-pago")
    TOTAL = (By.ID, "total")
    NOMBRE, EMAIL, CP = (By.ID, "nombre"), (By.ID, "email"), (By.ID, "cp")
    ACEPTO = (By.ID, "acepto")
    CONFIRMAR = (By.ID, "confirmar")
    ERROR = (By.ID, "error-pago")
    MODAL = (By.ID, "modal")
    NUM_PEDIDO = (By.ID, "num-pedido")
    CERRAR = (By.ID, "cerrar-modal")

    def esperar(self) -> PaginaPago:
        self.visible(self.VISTA)
        return self

    @property
    def total(self) -> Decimal:
        return euros(self.visible(self.TOTAL).text)

    def rellenar(self, nombre="", email="", cp="", aceptar=False) -> PaginaPago:
        self.escribir(self.NOMBRE, nombre)
        self.escribir(self.EMAIL, email)
        self.escribir(self.CP, cp)
        casilla = self.driver.find_element(*self.ACEPTO)
        if casilla.is_selected() != aceptar:
            casilla.click()
        return self

    def confirmar(self) -> None:
        self.clicable(self.CONFIRMAR).click()

    def muestra_error(self, texto: str) -> bool:
        """Espera a que el mensaje de error sea exactamente `texto` (evita leer un mensaje anterior)."""
        try:
            self.espera.until(lambda d: d.find_element(*self.ERROR).text == texto)
            return True
        except TimeoutException:
            return False

    def numero_pedido(self) -> str:
        self.visible(self.MODAL)
        self.espera.until(lambda d: d.find_element(*self.NUM_PEDIDO).text.startswith("PED-"))
        return self.driver.find_element(*self.NUM_PEDIDO).text

    def cerrar_confirmacion(self) -> PaginaCatalogo:
        self.clicable(self.CERRAR).click()
        self.espera.until(EC.invisibility_of_element_located(self.MODAL))
        return PaginaCatalogo(self.driver)
```

### 18.3 Configuración de pytest (`conftest.py`)

```python
"""conftest.py — Arranca la tienda de demostración y crea un navegador por prueba."""
import os
import threading
from pathlib import Path

import pytest
from selenium import webdriver
from werkzeug.serving import make_server

from tienda_demo import app

CAPTURAS = Path("capturas")


@pytest.fixture(scope="session")
def url_base():
    servidor = make_server("127.0.0.1", 5011, app)           # Servidor en un hilo para las pruebas
    hilo = threading.Thread(target=servidor.serve_forever, daemon=True)
    hilo.start()
    yield "http://127.0.0.1:5011"
    servidor.shutdown()


@pytest.fixture
def driver():
    opciones = webdriver.ChromeOptions()
    if os.environ.get("VISIBLE") != "1":                      # VISIBLE=1 pytest → ver el navegador
        opciones.add_argument("--headless=new")
    opciones.add_argument("--window-size=1366,900")
    opciones.add_argument("--lang=es-ES")
    opciones.add_argument("--no-sandbox")
    d = webdriver.Chrome(options=opciones)
    yield d
    d.quit()


@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    informe = (yield).get_result()
    if informe.when == "call" and informe.failed and "driver" in item.fixturenames:
        CAPTURAS.mkdir(exist_ok=True)
        item.funcargs["driver"].save_screenshot(str(CAPTURAS / f"{item.name}.png"))
```

### 18.4 Pruebas (`test_tienda.py`)

```python
"""test_tienda.py — Pruebas de extremo a extremo de la tienda de demostración."""
from decimal import Decimal

import pytest

from paginas import PaginaLogin


@pytest.fixture
def catalogo(driver, url_base):
    return PaginaLogin(driver).abrir(url_base).entrar("demo", "demo1234").esperar_carga()


def test_login_incorrecto(driver, url_base):
    pagina = PaginaLogin(driver).abrir(url_base)
    pagina.entrar("demo", "mala")
    assert pagina.error() == "Usuario o contraseña incorrectos"


def test_catalogo_carga_de_forma_asincrona(catalogo):
    productos = catalogo.productos()
    assert len(productos) == 6
    assert all(p["precio"] > 0 for p in productos)


@pytest.mark.parametrize("texto, esperados", [("usb", ["Hub USB-C"]), ("mon", ['Monitor 27" 4K']), ("zzz", [])])
def test_busqueda(catalogo, texto, esperados):
    catalogo.buscar(texto)
    assert [p["nombre"] for p in catalogo.productos()] == esperados
    assert catalogo.hay_resultados() == bool(esperados)


def test_filtro_y_orden(catalogo):
    catalogo.filtrar_categoria("Accesorios").ordenar("Precio descendente")
    precios = [p["precio"] for p in catalogo.productos()]
    assert precios == sorted(precios, reverse=True) and len(precios) == 2


def test_compra_completa(catalogo):
    catalogo.anadir("Teclado mecánico").anadir("Hub USB-C")
    assert catalogo.unidades_carrito == 2

    pago = catalogo.tramitar()
    assert pago.total == Decimal("129.80")

    pago.rellenar(nombre="Ana Pérez", email="ana.ejemplo.com", cp="28001", aceptar=True).confirmar()
    assert pago.muestra_error("Email no válido")

    pago.rellenar(nombre="Ana Pérez", email="ana@ejemplo.com", cp="2800", aceptar=True).confirmar()
    assert pago.muestra_error("Código postal no válido")

    pago.rellenar(nombre="Ana Pérez", email="ana@ejemplo.com", cp="28001", aceptar=False).confirmar()
    assert pago.muestra_error("Debe aceptar las condiciones")

    pago.rellenar(nombre="Ana Pérez", email="ana@ejemplo.com", cp="28001", aceptar=True).confirmar()
    assert pago.numero_pedido().startswith("PED-")

    catalogo = pago.cerrar_confirmacion()
    assert catalogo.unidades_carrito == 0


def test_cerrar_sesion_con_confirm(catalogo, driver):
    catalogo.salir(confirmar=False)                    # Cancelar el confirm(): sigue dentro
    assert "Hola, demo" in driver.page_source
    catalogo.salir(confirmar=True)
    PaginaLogin(driver).visible(PaginaLogin.USUARIO)   # Vuelve a la pantalla de acceso
```

### 18.5 Ejecución

```bash
pip install selenium flask pytest
python tienda_demo.py            # (opcional) probar la tienda manualmente en http://127.0.0.1:5000
pytest -v                        # Ejecuta las pruebas en modo headless
VISIBLE=1 pytest -v -k compra    # Ver el navegador durante una prueba (Windows: set VISIBLE=1)
```

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `SessionNotCreatedException: This version of ChromeDriver only supports Chrome version N` | Controlador y navegador con versiones distintas. | Deje que Selenium Manager lo gestione o instale el controlador de la misma versión. |
| `NoSuchElementException` | El elemento aún no existe, el localizador es incorrecto o está en un *iframe*. | Espera explícita; revise el selector; `switch_to.frame`. |
| `ElementNotInteractableException` | Elemento oculto o deshabilitado. | Espere a `element_to_be_clickable`; desplácese hasta él. |
| `ElementClickInterceptedException` | Otro elemento (aviso de cookies, modal) lo tapa. | Cierre el elemento superpuesto o espere a que desaparezca. |
| `StaleElementReferenceException` | La página se ha vuelto a dibujar y la referencia ya no es válida. | Vuelva a localizar el elemento justo antes de usarlo. |
| `TimeoutException` | La condición no se cumplió a tiempo. | Revise la condición, aumente el tiempo o investigue un error de la aplicación. |
| `UnexpectedAlertPresentException` | Hay una alerta abierta. | `switch_to.alert.accept()` o `dismiss()`. |
| `InvalidSelectorException` | Selector CSS/XPath con sintaxis incorrecta. | Pruébelo en la consola del navegador (`$$("…")`, `$x("…")`). |
| Procesos de Chrome acumulados | No se llamó a `quit()`. | `try/finally` o `with webdriver.Chrome() as driver`. |
| Pruebas que fallan «a veces» | Uso de `sleep` o esperas insuficientes. | Esperas explícitas basadas en condiciones. |
| Chrome se cierra en Docker | Memoria compartida insuficiente. | `--shm-size=2g` o `--disable-dev-shm-usage`. |

---

## 20. Buenas prácticas

- Use **esperas explícitas** basadas en condiciones; evite `time.sleep` y no mezcle esperas implícitas y explícitas.
- Prefiera localizadores estables (`data-testid`, `id`, `name`) y evite XPath absolutos.
- Organice el código con el patrón *Page Object* y mantenga las aserciones en las pruebas.
- Haga cada prueba independiente (sin depender del orden) y con datos propios.
- Ejecute en modo *headless* en integración continua y guarde capturas en los fallos.
- Cierre siempre el navegador con `quit()`.
- Reduzca el tiempo de las pruebas evitando repetir el *login* (cookies) y paralelizando con pytest-xdist o Grid.
- Para extraer datos de forma masiva, valore antes una API, Requests/BeautifulSoup o Scrapy; use el navegador solo cuando sea imprescindible.
- Respete las condiciones de uso de los sitios que automatice.

---

## 21. Referencia rápida

```python
from selenium import webdriver; from selenium.webdriver.common.by import By; from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait, Select; from selenium.webdriver.support import expected_conditions as EC
o = webdriver.ChromeOptions(); o.add_argument("--headless=new"); driver = webdriver.Chrome(options=o)
driver.get(url) · driver.title · driver.current_url · driver.page_source · driver.back() / forward() / refresh() · driver.quit()
driver.find_element(By.ID|NAME|CSS_SELECTOR|XPATH|LINK_TEXT|CLASS_NAME|TAG_NAME, "…") · find_elements(...)
el.click() · el.send_keys("x", Keys.ENTER) · el.clear() · el.text · el.get_attribute("v") · el.is_displayed()
WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.ID, "x"))) · visibility_of_element_located · invisibility_of_element_located
Select(el).select_by_visible_text("…") · ActionChains(driver).move_to_element(a).click(b).perform()
driver.switch_to.alert.accept() · switch_to.frame(el) · switch_to.default_content() · switch_to.window(h) · window_handles
driver.execute_script("return …", el) · save_screenshot("x.png") · el.screenshot() · get_cookies() · add_cookie({})
webdriver.Remote(command_executor="http://grid:4444", options=o)
```

---

## 22. Ejercicios propuestos

1. Automatice la búsqueda de un término en un sitio público y guarde en CSV los títulos y enlaces de los resultados de las tres primeras páginas.
2. Añada a las pruebas de la tienda un caso que verifique que la ordenación por nombre es alfabética y que el botón «Tramitar pedido» está deshabilitado con el carrito vacío.
3. Implemente una función que espere a que termine una descarga en una carpeta y compruebe el contenido del archivo descargado.
4. Ejecute las pruebas de la tienda en paralelo con `pytest-xdist` y compare los tiempos.
5. Levante Selenium Grid con Docker y ejecute las mismas pruebas en Chrome y Firefox mediante `webdriver.Remote`.

---

## 23. Referencias

- Documentación oficial: https://www.selenium.dev/documentation/
- API de Python: https://www.selenium.dev/selenium/docs/api/py/
- Selenium Manager: https://www.selenium.dev/documentation/selenium_manager/
- Esperas: https://www.selenium.dev/documentation/webdriver/waits/
- Selenium Grid: https://www.selenium.dev/documentation/grid/
