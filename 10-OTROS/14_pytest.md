# Manual de uso: pytest

> **Categoría:** Utilidades generales · **Utilidad:** Realizar pruebas de código
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Estructura de un proyecto con pruebas](#4-estructura-de-un-proyecto-con-pruebas)
5. [Operaciones básicas](#5-operaciones-básicas)
6. [Ejecutar las pruebas](#6-ejecutar-las-pruebas)
7. [Fixtures](#7-fixtures)
8. [Parametrización](#8-parametrización)
9. [Marcadores](#9-marcadores)
10. [Simulación (mocking)](#10-simulación-mocking)
11. [Operaciones avanzadas](#11-operaciones-avanzadas)
12. [Cobertura de código](#12-cobertura-de-código)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

**pytest** es el framework de pruebas más utilizado en Python. Permite escribir pruebas con funciones sencillas y la instrucción `assert` estándar, y ofrece descubrimiento automático de pruebas, mensajes de error detallados, *fixtures* reutilizables, parametrización y un amplio ecosistema de complementos.

**Tipos de prueba que se pueden escribir:**
- **Unitarias:** comprueban una función o clase de forma aislada.
- **De integración:** comprueban la colaboración entre componentes (base de datos, API…).
- **Funcionales / extremo a extremo:** comprueban el comportamiento completo de la aplicación.

**Ventajas de probar el código:**
- Detecta errores antes de que lleguen a producción.
- Permite refactorizar con seguridad.
- Documenta el comportamiento esperado.

---

## 2. Instalación e importación

```bash
pip install pytest
```

Complementos recomendados:

```bash
pip install pytest-cov          # Cobertura de código
pip install pytest-mock         # Fixture "mocker" para simulaciones
pip install pytest-xdist        # Ejecución en paralelo
pip install pytest-asyncio      # Pruebas de código asíncrono
```

En los archivos de prueba:

```python
import pytest
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Prueba (*test*) | Función cuyo nombre empieza por `test_`. |
| `assert` | Comprueba una condición; si es falsa, la prueba falla. |
| Descubrimiento | pytest busca automáticamente archivos `test_*.py` o `*_test.py`. |
| *Fixture* | Función que prepara datos o recursos para las pruebas. |
| Parametrización | Ejecutar la misma prueba con distintos datos. |
| Marcador | Etiqueta para clasificar, omitir o configurar pruebas. |
| *Mock* | Objeto simulado que sustituye una dependencia real. |
| `conftest.py` | Archivo para compartir fixtures y configuración. |

### 3.1 Patrón AAA (Arrange – Act – Assert)

```python
def test_aplicar_descuento():
    # Preparar (Arrange)
    precio = 100

    # Actuar (Act)
    resultado = aplicar_descuento(precio, 0.2)

    # Comprobar (Assert)
    assert resultado == 80
```

---

## 4. Estructura de un proyecto con pruebas

```
mi_proyecto/
├── pyproject.toml
├── src/
│   └── tienda/
│       ├── __init__.py
│       ├── carrito.py
│       └── precios.py
└── tests/
    ├── conftest.py           # Fixtures compartidas
    ├── test_carrito.py
    └── test_precios.py
```

Convenciones de nombres que pytest reconoce por defecto:

| Elemento | Convención |
|---|---|
| Archivos | `test_*.py` o `*_test.py` |
| Funciones | `test_*` |
| Clases | `Test*` (sin método `__init__`) |
| Métodos | `test_*` |

---

## 5. Operaciones básicas

### 5.1 Primera prueba

**`precios.py`**

```python
def aplicar_iva(base: float, tipo: float = 0.21) -> float:
    if base < 0:
        raise ValueError("La base no puede ser negativa")
    return round(base * (1 + tipo), 2)
```

**`test_precios.py`**

```python
from precios import aplicar_iva


def test_iva_general():
    assert aplicar_iva(100) == 121.0


def test_iva_reducido():
    assert aplicar_iva(100, 0.10) == 110.0


def test_base_cero():
    assert aplicar_iva(0) == 0
```

Ejecutar:

```bash
pytest
```

Resultado:

```
============================= test session starts ==============================
collected 3 items

test_precios.py ...                                                      [100%]

============================== 3 passed in 0.01s ===============================
```

### 5.2 Tipos de comprobaciones

```python
assert resultado == 10
assert resultado != 0
assert valor is None
assert valor is not None
assert "error" in mensaje
assert elemento in lista
assert len(lista) == 3
assert isinstance(obj, Producto)
assert total > 0
assert not lista                         # Lista vacía
assert dict_a == {"a": 1, "b": 2}
assert lista == sorted(lista)
assert resultado == 10, "Mensaje personalizado si falla"
```

pytest muestra automáticamente el detalle de la diferencia cuando una comprobación falla:

```
>       assert {"a": 1, "b": 2} == {"a": 1, "b": 3}
E       AssertionError: assert {'a': 1, 'b': 2} == {'a': 1, 'b': 3}
E         Differing items:
E         {'b': 2} != {'b': 3}
```

### 5.3 Comparar números decimales

```python
from pytest import approx

assert 0.1 + 0.2 == approx(0.3)
assert 2.0001 == approx(2, rel=1e-3)
assert [0.1 + 0.2, 0.3] == approx([0.3, 0.3])
```

### 5.4 Comprobar excepciones

```python
import pytest


def test_base_negativa():
    with pytest.raises(ValueError):
        aplicar_iva(-10)


def test_mensaje_de_error():
    with pytest.raises(ValueError, match="no puede ser negativa"):
        aplicar_iva(-10)


def test_detalle_excepcion():
    with pytest.raises(ValueError) as info:
        aplicar_iva(-10)
    assert "negativa" in str(info.value)
```

### 5.5 Comprobar avisos

```python
import warnings

def test_aviso():
    with pytest.warns(DeprecationWarning):
        warnings.warn("obsoleto", DeprecationWarning)
```

### 5.6 Agrupar pruebas en clases

```python
class TestCarrito:
    def test_vacio_al_crear(self):
        assert Carrito().total() == 0

    def test_agregar_producto(self):
        c = Carrito()
        c.agregar("Teclado", 29.9)
        assert c.total() == 29.9
```

---

## 6. Ejecutar las pruebas

| Comando | Efecto |
|---|---|
| `pytest` | Ejecuta todas las pruebas. |
| `pytest -v` | Modo detallado (muestra cada prueba). |
| `pytest -q` | Modo silencioso. |
| `pytest tests/test_precios.py` | Un archivo concreto. |
| `pytest tests/test_precios.py::test_iva_general` | Una prueba concreta. |
| `pytest tests/test_carrito.py::TestCarrito` | Una clase. |
| `pytest -k "iva and not reducido"` | Pruebas cuyo nombre coincide con la expresión. |
| `pytest -m lento` | Pruebas con un marcador. |
| `pytest -x` | Detenerse en el primer fallo. |
| `pytest --maxfail=3` | Detenerse tras 3 fallos. |
| `pytest --lf` | Solo las que fallaron la última vez. |
| `pytest --ff` | Primero las que fallaron, luego el resto. |
| `pytest -s` | Mostrar la salida de `print()`. |
| `pytest -l` | Mostrar variables locales en los fallos. |
| `pytest --pdb` | Abrir el depurador al fallar. |
| `pytest --durations=5` | Mostrar las 5 pruebas más lentas. |
| `pytest -n auto` | En paralelo (requiere pytest-xdist). |
| `pytest --collect-only` | Listar pruebas sin ejecutarlas. |

### 6.1 Símbolos del resultado

| Símbolo | Significado |
|---|---|
| `.` | Prueba superada |
| `F` | Fallo (assert falso) |
| `E` | Error (excepción inesperada o error en fixture) |
| `s` | Omitida (*skipped*) |
| `x` | Fallo esperado (*xfail*) |
| `X` | Superada aunque se esperaba un fallo (*xpass*) |

---

## 7. Fixtures

Las *fixtures* preparan datos, objetos o recursos que necesitan las pruebas. Se inyectan por **nombre de parámetro**.

### 7.1 Fixture básica

```python
import pytest
from tienda.carrito import Carrito


@pytest.fixture
def carrito():
    return Carrito()


@pytest.fixture
def carrito_con_productos(carrito):          # Una fixture puede usar otra
    carrito.agregar("Teclado", 29.9, 2)
    carrito.agregar("Ratón", 14.5, 1)
    return carrito


def test_total(carrito_con_productos):
    assert carrito_con_productos.total() == approx(74.3)


def test_vaciar(carrito_con_productos):
    carrito_con_productos.vaciar()
    assert carrito_con_productos.total() == 0
```

> Cada prueba recibe una **instancia nueva** de la fixture: las pruebas no se afectan entre sí.

### 7.2 Preparación y limpieza con `yield`

```python
@pytest.fixture
def conexion_bd():
    conexion = crear_conexion("sqlite:///:memory:")    # Preparación
    crear_tablas(conexion)
    yield conexion                                      # Se entrega a la prueba
    conexion.close()                                    # Limpieza (siempre se ejecuta)
```

### 7.3 Ámbito (*scope*)

```python
@pytest.fixture(scope="session")      # Una vez por ejecución completa
def configuracion():
    return cargar_config("config_test.yaml")
```

| Ámbito | Se crea… |
|---|---|
| `function` (por defecto) | Para cada prueba. |
| `class` | Una vez por clase. |
| `module` | Una vez por archivo. |
| `package` | Una vez por paquete. |
| `session` | Una vez por ejecución. |

### 7.4 Fixtures automáticas

```python
@pytest.fixture(autouse=True)
def entorno_de_pruebas(monkeypatch):
    monkeypatch.setenv("APP_ENV", "test")
```

### 7.5 Fixtures que devuelven funciones (factorías)

```python
@pytest.fixture
def crear_producto():
    def _crear(nombre="Producto", precio=10.0, stock=5):
        return Producto(nombre=nombre, precio=precio, stock=stock)
    return _crear


def test_sin_stock(crear_producto):
    p = crear_producto(stock=0)
    assert not p.disponible()
```

### 7.6 Compartir fixtures con `conftest.py`

Las fixtures definidas en `tests/conftest.py` están disponibles en todas las pruebas de esa carpeta y subcarpetas, **sin necesidad de importarlas**.

### 7.7 Fixtures integradas más útiles

| Fixture | Uso |
|---|---|
| `tmp_path` | Carpeta temporal única (`pathlib.Path`). |
| `tmp_path_factory` | Carpetas temporales con ámbito de sesión. |
| `monkeypatch` | Modificar temporalmente atributos, variables de entorno o diccionarios. |
| `capsys` | Capturar la salida de `print()`. |
| `caplog` | Capturar mensajes de `logging`. |
| `request` | Información sobre la prueba en curso. |

```python
def test_guardar_archivo(tmp_path):
    ruta = tmp_path / "salida.txt"
    guardar_informe(ruta, "Hola")
    assert ruta.read_text(encoding="utf-8") == "Hola"


def test_salida_consola(capsys):
    print("Proceso completado")
    salida = capsys.readouterr()
    assert "completado" in salida.out


def test_log(caplog):
    import logging
    with caplog.at_level(logging.WARNING):
        logging.getLogger("app").warning("Stock bajo")
    assert "Stock bajo" in caplog.text


def test_variable_entorno(monkeypatch):
    monkeypatch.setenv("API_TOKEN", "token-de-prueba")
    monkeypatch.delenv("DEBUG", raising=False)
    assert obtener_token() == "token-de-prueba"
```

---

## 8. Parametrización

Ejecuta la misma prueba con diferentes conjuntos de datos.

### 8.1 Parametrización simple

```python
@pytest.mark.parametrize("base, tipo, esperado", [
    (100, 0.21, 121.0),
    (100, 0.10, 110.0),
    (100, 0.04, 104.0),
    (0, 0.21, 0.0),
    (19.99, 0.21, 24.19),
])
def test_aplicar_iva(base, tipo, esperado):
    assert aplicar_iva(base, tipo) == esperado
```

Cada combinación aparece como una prueba independiente:

```
test_precios.py::test_aplicar_iva[100-0.21-121.0] PASSED
test_precios.py::test_aplicar_iva[100-0.1-110.0] PASSED
...
```

### 8.2 Identificadores legibles

```python
@pytest.mark.parametrize("email, valido", [
    pytest.param("ana@ejemplo.com", True, id="email-correcto"),
    pytest.param("ana@", False, id="sin-dominio"),
    pytest.param("", False, id="vacio"),
    pytest.param("ana ejemplo.com", False, id="sin-arroba"),
])
def test_validar_email(email, valido):
    assert validar_email(email) is valido
```

### 8.3 Combinaciones (producto cartesiano)

```python
@pytest.mark.parametrize("cantidad", [1, 10, 100])
@pytest.mark.parametrize("tipo_cliente", ["particular", "empresa"])
def test_precio(cantidad, tipo_cliente):      # 3 × 2 = 6 pruebas
    assert calcular_precio(cantidad, tipo_cliente) > 0
```

### 8.4 Fixtures parametrizadas

```python
@pytest.fixture(params=["sqlite", "postgresql"])
def motor_bd(request):
    return crear_motor(request.param)
```

---

## 9. Marcadores

### 9.1 Omitir pruebas

```python
@pytest.mark.skip(reason="Pendiente de implementar")
def test_exportar_pdf(): ...


import sys

@pytest.mark.skipif(sys.platform == "win32", reason="No disponible en Windows")
def test_permisos_unix(): ...


def test_requiere_libreria():
    pandas = pytest.importorskip("pandas")    # Omite si no está instalada
```

### 9.2 Fallos esperados

```python
@pytest.mark.xfail(reason="Error conocido #123, pendiente de corregir")
def test_redondeo_especial():
    assert redondear(2.675, 2) == 2.68
```

### 9.3 Marcadores propios

```python
@pytest.mark.lento
def test_proceso_completo(): ...

@pytest.mark.integracion
def test_conexion_api(): ...
```

Regístrelos en `pyproject.toml` para evitar avisos:

```toml
[tool.pytest.ini_options]
markers = [
    "lento: pruebas que tardan más de unos segundos",
    "integracion: pruebas que usan servicios externos",
]
```

```bash
pytest -m "not lento"             # Excluir las lentas
pytest -m integracion
```

### 9.4 Usar fixtures con marcadores

```python
@pytest.mark.usefixtures("entorno_limpio")
class TestImportacion: ...
```

---

## 10. Simulación (mocking)

Sustituye dependencias externas (APIs, bases de datos, fecha actual, correo) por objetos controlados.

### 10.1 Con `monkeypatch`

```python
# servicio.py
import requests

def obtener_cambio(moneda: str) -> float:
    r = requests.get(f"https://api.ejemplo.com/cambio/{moneda}", timeout=10)
    r.raise_for_status()
    return r.json()["valor"]
```

```python
# test_servicio.py
import servicio


class RespuestaFalsa:
    def __init__(self, datos):
        self._datos = datos
    def raise_for_status(self):
        pass
    def json(self):
        return self._datos


def test_obtener_cambio(monkeypatch):
    def get_falso(url, timeout):
        assert url.endswith("/USD")
        return RespuestaFalsa({"valor": 1.08})

    monkeypatch.setattr(servicio.requests, "get", get_falso)
    assert servicio.obtener_cambio("USD") == 1.08
```

### 10.2 Con `unittest.mock`

```python
from unittest.mock import Mock, MagicMock, patch


def test_envio_correo():
    with patch("notificaciones.smtplib.SMTP") as smtp_falso:
        enviar_aviso("ana@ejemplo.com", "Pedido enviado")
        instancia = smtp_falso.return_value.__enter__.return_value
        instancia.send_message.assert_called_once()
```

### 10.3 Con `pytest-mock` (fixture `mocker`)

```python
def test_obtener_cambio(mocker):
    respuesta = mocker.Mock()
    respuesta.json.return_value = {"valor": 1.08}
    get = mocker.patch("servicio.requests.get", return_value=respuesta)

    assert servicio.obtener_cambio("USD") == 1.08
    get.assert_called_once_with("https://api.ejemplo.com/cambio/USD", timeout=10)
```

### 10.4 Simular errores

```python
def test_api_no_disponible(mocker):
    mocker.patch("servicio.requests.get", side_effect=requests.ConnectionError)
    with pytest.raises(requests.ConnectionError):
        servicio.obtener_cambio("USD")
```

### 10.5 Comprobaciones sobre los mocks

```python
mock.assert_called()
mock.assert_called_once()
mock.assert_called_with(arg1, clave=valor)
mock.assert_not_called()
mock.call_count
mock.call_args
```

> **Regla clave:** parchee el objeto **donde se usa**, no donde se define. Si `servicio.py` hace `import requests`, parchee `servicio.requests.get`.

### 10.6 Controlar la fecha actual

```python
from datetime import date

def test_dias_hasta_vencimiento(monkeypatch):
    class FechaFija(date):
        @classmethod
        def today(cls):
            return cls(2026, 11, 24)

    monkeypatch.setattr("facturas.date", FechaFija)
    assert dias_hasta(date(2026, 12, 1)) == 7
```

---

## 11. Operaciones avanzadas

### 11.1 Configuración en `pyproject.toml`

```toml
[tool.pytest.ini_options]
minversion = "8.0"
testpaths = ["tests"]
pythonpath = ["src"]
addopts = "-ra -q --strict-markers"
markers = [
    "lento: pruebas lentas",
    "integracion: requieren servicios externos",
]
filterwarnings = ["error::DeprecationWarning"]
```

| Opción | Efecto |
|---|---|
| `testpaths` | Carpetas donde buscar pruebas. |
| `pythonpath` | Añade rutas al `sys.path` (útil con estructura `src/`). |
| `addopts` | Opciones aplicadas siempre. |
| `-ra` | Resumen final de omitidas, fallidas, etc. |
| `--strict-markers` | Error si se usa un marcador no registrado. |

### 11.2 Pruebas asíncronas

```python
import pytest

@pytest.mark.asyncio
async def test_descarga_asincrona():
    resultado = await descargar_datos()
    assert resultado["estado"] == "ok"
```

### 11.3 Probar una API FastAPI

```python
from fastapi.testclient import TestClient
from main import app

cliente = TestClient(app)

def test_listar_tareas():
    r = cliente.get("/tareas")
    assert r.status_code == 200
    assert isinstance(r.json(), list)
```

### 11.4 Base de datos temporal para cada prueba

```python
@pytest.fixture
def sesion():
    from sqlalchemy import create_engine
    from sqlalchemy.orm import Session
    motor = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(motor)
    with Session(motor) as s:
        yield s
```

### 11.5 Pruebas basadas en propiedades (Hypothesis)

```python
# pip install hypothesis
from hypothesis import given, strategies as st

@given(st.floats(min_value=0, max_value=1e6, allow_nan=False))
def test_iva_nunca_menor_que_base(base):
    assert aplicar_iva(base) >= base
```

### 11.6 Integración continua (GitHub Actions)

```yaml
# .github/workflows/pruebas.yml
name: Pruebas
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -e . pytest pytest-cov
      - run: pytest --cov=src --cov-report=term-missing
```

---

## 12. Cobertura de código

La cobertura indica qué porcentaje del código se ejecuta durante las pruebas.

```bash
pytest --cov=src                                  # Resumen
pytest --cov=src --cov-report=term-missing        # Líneas no cubiertas
pytest --cov=src --cov-report=html                # Informe HTML en htmlcov/index.html
pytest --cov=src --cov-fail-under=80              # Falla si la cobertura es < 80 %
```

```
Name                    Stmts   Miss  Cover   Missing
-----------------------------------------------------
src/tienda/carrito.py      42      3    93%   55-57
src/tienda/precios.py      18      0   100%
-----------------------------------------------------
TOTAL                      60      3    95%
```

> Una cobertura alta no garantiza la ausencia de errores, pero una cobertura baja indica código sin verificar.

---

## 13. Ejemplo práctico completo

Módulo de carrito de compra y su batería de pruebas.

**`tienda/carrito.py`**

```python
from __future__ import annotations

from dataclasses import dataclass, field
from decimal import Decimal, ROUND_HALF_UP


class ErrorCarrito(Exception):
    pass


@dataclass
class Linea:
    producto: str
    precio: Decimal
    cantidad: int

    @property
    def subtotal(self) -> Decimal:
        return self.precio * self.cantidad


@dataclass
class Carrito:
    lineas: dict[str, Linea] = field(default_factory=dict)
    cupon: str | None = None

    CUPONES = {"BIENVENIDA10": Decimal("0.10"), "VIP20": Decimal("0.20")}
    ENVIO_GRATIS_DESDE = Decimal("50")
    COSTE_ENVIO = Decimal("4.95")

    def agregar(self, producto: str, precio: float | str, cantidad: int = 1) -> None:
        if cantidad <= 0:
            raise ErrorCarrito("La cantidad debe ser positiva")
        precio_dec = Decimal(str(precio))
        if precio_dec < 0:
            raise ErrorCarrito("El precio no puede ser negativo")
        if producto in self.lineas:
            self.lineas[producto].cantidad += cantidad
        else:
            self.lineas[producto] = Linea(producto, precio_dec, cantidad)

    def eliminar(self, producto: str) -> None:
        if producto not in self.lineas:
            raise ErrorCarrito(f"{producto} no está en el carrito")
        del self.lineas[producto]

    def aplicar_cupon(self, codigo: str) -> None:
        codigo = codigo.strip().upper()
        if codigo not in self.CUPONES:
            raise ErrorCarrito("Cupón no válido")
        self.cupon = codigo

    @property
    def unidades(self) -> int:
        return sum(l.cantidad for l in self.lineas.values())

    @property
    def subtotal(self) -> Decimal:
        return sum((l.subtotal for l in self.lineas.values()), Decimal("0"))

    @property
    def descuento(self) -> Decimal:
        return self.subtotal * self.CUPONES.get(self.cupon, Decimal("0"))

    @property
    def envio(self) -> Decimal:
        if not self.lineas:
            return Decimal("0")
        neto = self.subtotal - self.descuento
        return Decimal("0") if neto >= self.ENVIO_GRATIS_DESDE else self.COSTE_ENVIO

    @property
    def total(self) -> Decimal:
        bruto = self.subtotal - self.descuento + self.envio
        return bruto.quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)
```

**`tests/conftest.py`**

```python
import pytest
from tienda.carrito import Carrito


@pytest.fixture
def carrito() -> Carrito:
    return Carrito()


@pytest.fixture
def carrito_lleno(carrito: Carrito) -> Carrito:
    carrito.agregar("Teclado", "29.90", 1)
    carrito.agregar("Ratón", "14.50", 2)
    return carrito
```

**`tests/test_carrito.py`**

```python
from decimal import Decimal

import pytest
from tienda.carrito import Carrito, ErrorCarrito


class TestAgregar:
    def test_carrito_nuevo_vacio(self, carrito):
        assert carrito.unidades == 0
        assert carrito.total == Decimal("0.00")

    def test_agregar_un_producto(self, carrito):
        carrito.agregar("Teclado", "29.90")
        assert carrito.unidades == 1
        assert carrito.subtotal == Decimal("29.90")

    def test_agregar_mismo_producto_acumula(self, carrito):
        carrito.agregar("Ratón", "14.50", 1)
        carrito.agregar("Ratón", "14.50", 2)
        assert len(carrito.lineas) == 1
        assert carrito.lineas["Ratón"].cantidad == 3

    @pytest.mark.parametrize("cantidad", [0, -1, -10])
    def test_cantidad_no_valida(self, carrito, cantidad):
        with pytest.raises(ErrorCarrito, match="positiva"):
            carrito.agregar("Teclado", "29.90", cantidad)

    def test_precio_negativo(self, carrito):
        with pytest.raises(ErrorCarrito, match="negativo"):
            carrito.agregar("Teclado", "-1")


class TestEliminar:
    def test_eliminar_existente(self, carrito_lleno):
        carrito_lleno.eliminar("Ratón")
        assert "Ratón" not in carrito_lleno.lineas

    def test_eliminar_inexistente(self, carrito):
        with pytest.raises(ErrorCarrito):
            carrito.eliminar("Monitor")


class TestTotales:
    def test_subtotal(self, carrito_lleno):
        assert carrito_lleno.subtotal == Decimal("58.90")

    def test_envio_gratis_desde_50(self, carrito_lleno):
        assert carrito_lleno.envio == Decimal("0")
        assert carrito_lleno.total == Decimal("58.90")

    def test_envio_con_importe_bajo(self, carrito):
        carrito.agregar("Cable", "5.00")
        assert carrito.envio == Carrito.COSTE_ENVIO
        assert carrito.total == Decimal("9.95")

    def test_carrito_vacio_sin_envio(self, carrito):
        assert carrito.envio == Decimal("0")


class TestCupones:
    @pytest.mark.parametrize("codigo, descuento_esperado", [
        pytest.param("BIENVENIDA10", Decimal("5.89"), id="10%"),
        pytest.param("vip20", Decimal("11.78"), id="20%-minusculas"),
        pytest.param("  VIP20 ", Decimal("11.78"), id="20%-espacios"),
    ])
    def test_cupones_validos(self, carrito_lleno, codigo, descuento_esperado):
        carrito_lleno.aplicar_cupon(codigo)
        assert carrito_lleno.descuento == descuento_esperado

    def test_cupon_no_valido(self, carrito_lleno):
        with pytest.raises(ErrorCarrito, match="no válido"):
            carrito_lleno.aplicar_cupon("FALSO")
        assert carrito_lleno.cupon is None

    def test_cupon_puede_anular_envio_gratis(self, carrito):
        carrito.agregar("Auriculares", "55.00")
        assert carrito.envio == Decimal("0")
        carrito.aplicar_cupon("VIP20")                 # 55 - 11 = 44 < 50
        assert carrito.envio == Carrito.COSTE_ENVIO
        assert carrito.total == Decimal("48.95")
```

Ejecución:

```bash
pytest -v --cov=tienda --cov-report=term-missing
```

---

## 14. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| `collected 0 items` | Nombres sin el prefijo `test_`. | Respete las convenciones de nombres. |
| `ModuleNotFoundError` al importar el código | El paquete no está en el `sys.path`. | Configure `pythonpath = ["src"]` o instale con `pip install -e .`. |
| `fixture 'x' not found` | Fixture no definida o `conftest.py` mal ubicado. | Defínala en `conftest.py` de la carpeta correcta. |
| Las pruebas pasan solas pero fallan juntas | Estado compartido entre pruebas. | Use fixtures de ámbito `function` y evite variables globales. |
| `0.1 + 0.2 == 0.3` falla | Precisión de los decimales en coma flotante. | Use `pytest.approx` o `Decimal`. |
| El mock no tiene efecto | Se parcheó en el módulo equivocado. | Parchee donde se **usa** el objeto. |
| `PytestUnknownMarkWarning` | Marcador no registrado. | Regístrelo en `pyproject.toml`. |
| No se ven los `print()` | pytest captura la salida. | Ejecute con `-s`. |
| Una clase de prueba no se ejecuta | Tiene método `__init__`. | Elimine `__init__`; use fixtures. |

---

## 15. Buenas prácticas

- Una prueba debe comprobar **un único comportamiento** y tener un nombre descriptivo (`test_cupon_no_valido_lanza_error`).
- Siga el patrón **Preparar – Actuar – Comprobar**.
- Las pruebas deben ser **independientes** entre sí y poder ejecutarse en cualquier orden.
- Las pruebas unitarias deben ser **rápidas**: simule red, disco y bases de datos externas.
- Pruebe los casos límite: valores vacíos, cero, negativos, máximos, errores.
- Use parametrización en lugar de duplicar pruebas.
- Mantenga las fixtures comunes en `conftest.py`.
- Ejecute las pruebas antes de cada *commit* y en la integración continua.
- Cuando corrija un error, escriba primero una prueba que lo reproduzca.

---

## 16. Referencia rápida

```python
def test_x(): assert valor == esperado
pytest.approx(0.3)
with pytest.raises(ValueError, match="texto"): ...
with pytest.warns(DeprecationWarning): ...

@pytest.fixture(scope="function", autouse=False, params=[...])
def recurso(): ...; yield obj; limpieza

@pytest.mark.parametrize("a, b", [(1, 2), pytest.param(3, 4, id="caso")])
@pytest.mark.skip(reason=...) / skipif(cond, reason=...) / xfail(reason=...)
pytest.importorskip("modulo")

tmp_path / monkeypatch / capsys / caplog / mocker (pytest-mock)
monkeypatch.setattr(obj, "attr", valor) / setenv / delenv
```

```bash
pytest -v -x -k "expr" -m "marca" --lf -s --pdb --cov=src --cov-report=html -n auto
```

---

## 17. Ejercicios propuestos

1. Escriba pruebas para una función que valide DNI españoles, incluyendo casos correctos, letra incorrecta, longitud inválida y entradas vacías.
2. Cree una fixture que genere un CSV temporal con `tmp_path` y pruebe una función que lo procese.
3. Pruebe una función que consulta una API externa simulando respuestas correctas, errores 404 y excepciones de conexión.
4. Alcance un 90 % de cobertura en un módulo propio e identifique las ramas sin probar.
5. Configure GitHub Actions para ejecutar las pruebas en cada *push*.

---

## 18. Referencias

- Documentación oficial: https://docs.pytest.org/
- pytest-cov: https://pytest-cov.readthedocs.io/
- pytest-mock: https://pytest-mock.readthedocs.io/
- unittest.mock: https://docs.python.org/es/3/library/unittest.mock.html
