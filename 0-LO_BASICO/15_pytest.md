# Manual de uso: pytest

> **Categoría:** Pruebas · **Utilidad:** Marco estándar de pruebas
> **Tipo:** Paquete externo (requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera prueba](#4-primera-prueba)
5. [Ejecutar pruebas](#5-ejecutar-pruebas)
6. [Aserciones](#6-aserciones)
7. [Excepciones y avisos](#7-excepciones-y-avisos)
8. [Fixtures](#8-fixtures)
9. [Fixtures integradas más útiles](#9-fixtures-integradas-más-útiles)
10. [Parametrización](#10-parametrización)
11. [Marcas (*markers*)](#11-marcas-markers)
12. [Dobles de prueba: `monkeypatch` y `unittest.mock`](#12-dobles-de-prueba-monkeypatch-y-unittestmock)
13. [Organización del proyecto y `conftest.py`](#13-organización-del-proyecto-y-conftestpy)
14. [Configuración](#14-configuración)
15. [Código asíncrono](#15-código-asíncrono)
16. [*Plugins* recomendados](#16-plugins-recomendados)
17. [Depuración de pruebas](#17-depuración-de-pruebas)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**pytest** es el marco de pruebas de referencia en Python. Permite escribir pruebas como **funciones simples** con `assert`, y ofrece un sistema de **fixtures** para preparar datos y recursos, **parametrización** para probar muchos casos con el mismo código y un ecosistema de más de 1.500 *plugins*.

```python
def sumar(a: int, b: int) -> int:
    return a + b

def test_sumar():
    assert sumar(2, 3) == 5
```

**Ventajas:**
- Sintaxis mínima: funciones y `assert`, sin clases obligatorias.
- Mensajes de error detallados que muestran los valores comparados.
- Fixtures reutilizables con inyección por nombre y control del ámbito.
- Ejecuta también pruebas escritas con `unittest`.
- Ecosistema de *plugins*: cobertura, ejecución en paralelo, asincronía, Django, *benchmarks*…

**Limitaciones:**
- La «magia» de las fixtures (inyección por nombre) requiere conocer las convenciones.
- Las fixtures muy encadenadas pueden dificultar la lectura si no se organizan bien.

| Si necesita… | Opción |
|---|---|
| Pruebas unitarias y de integración en general | **pytest** |
| Sin dependencias externas | `unittest` (librería estándar) |
| Generar casos automáticamente | Hypothesis (se integra con pytest) |
| Varias versiones de Python | nox / tox (ejecutan pytest) |

---

## 2. Instalación

```bash
uv add --dev pytest
poetry add --group dev pytest
python -m pip install pytest
```

Comprobar:

```bash
pytest --version
```

> Ejecute pytest como `python -m pytest` o `uv run pytest` para usar el del entorno del proyecto. `python -m pytest` además añade el directorio actual a `sys.path`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Prueba | Función cuyo nombre empieza por `test_` (o método en una clase `Test*`). |
| Descubrimiento | pytest busca archivos `test_*.py` o `*_test.py` y dentro, las pruebas. |
| Aserción | `assert` con una condición que debe cumplirse. |
| Fixture | Función que prepara (y limpia) un recurso; se inyecta como parámetro. |
| Ámbito | Duración de una fixture: `function`, `class`, `module`, `package`, `session`. |
| Parametrización | Ejecutar la misma prueba con distintos datos. |
| Marca | Etiqueta para clasificar o modificar pruebas (`skip`, `xfail`, propias). |
| `conftest.py` | Archivo con fixtures y configuración compartidas por un directorio. |
| ID de nodo | Identificador de una prueba: `tests/test_x.py::TestClase::test_metodo[param]`. |

---

## 4. Primera prueba

```python
# src/calculadora/precios.py
def precio_final(base: float, iva: float = 0.21, descuento: float = 0.0) -> float:
    if base < 0:
        raise ValueError("El precio base no puede ser negativo")
    if not 0 <= descuento < 1:
        raise ValueError("El descuento debe estar entre 0 y 1")
    return round(base * (1 - descuento) * (1 + iva), 2)
```

```python
# tests/test_precios.py
import pytest

from calculadora.precios import precio_final


def test_precio_con_iva():
    assert precio_final(100) == 121.0


def test_precio_con_descuento():
    assert precio_final(100, descuento=0.1) == 108.9


def test_base_negativa():
    with pytest.raises(ValueError, match="negativo"):
        precio_final(-1)
```

```text
$ pytest
============================= test session starts ==============================
platform linux -- Python 3.13.1, pytest-9.2.0
rootdir: /proyecto
configfile: pyproject.toml
collected 3 items

tests/test_precios.py ...                                                [100%]

============================== 3 passed in 0.02s ===============================
```

---

## 5. Ejecutar pruebas

```bash
pytest                                   # Todas
pytest tests/test_precios.py             # Un archivo
pytest tests/test_precios.py::test_base_negativa      # Una prueba
pytest tests/test_api.py::TestClientes   # Una clase
pytest -k "descuento and not negativo"   # Por nombre (expresión)
pytest -m lento                          # Por marca
pytest -m "not red"
```

Opciones frecuentes:

| Opción | Efecto |
|---|---|
| `-v` / `-vv` | Más detalle (nombre de cada prueba, diferencias completas). |
| `-q` | Salida mínima. |
| `-x` | Detenerse en el primer fallo. |
| `--maxfail=3` | Detenerse tras 3 fallos. |
| `--lf` | Solo las que fallaron la última vez (*last failed*). |
| `--ff` | Primero las que fallaron, luego el resto. |
| `--sw` | Paso a paso: continúa desde el último fallo (*stepwise*). |
| `-s` | Muestra `print()` (no captura la salida). |
| `-l` | Muestra las variables locales en los fallos. |
| `--tb=short` / `line` / `no` | Formato de las trazas de error. |
| `-ra` | Resumen final de todo lo que no pasó (fallos, omitidas, xfail). |
| `--durations=10` | Las 10 pruebas más lentas. |
| `--collect-only` / `--co` | Lista las pruebas sin ejecutarlas. |
| `-p no:cacheprovider` | Desactiva un *plugin*. |

Resultados posibles:

| Símbolo | Estado |
|---|---|
| `.` | Superada |
| `F` | Fallida (la aserción no se cumple) |
| `E` | Error (fallo en una fixture o fuera de la aserción) |
| `s` | Omitida (`skip`) |
| `x` | Fallo esperado (`xfail`) |
| `X` | Pasó inesperadamente (`xpass`) |

Códigos de salida: `0` todo correcto, `1` fallos, `2` interrumpido, `3` error interno, `4` error de uso, `5` no se encontraron pruebas.

---

## 6. Aserciones

pytest reescribe los `assert` para mostrar los valores implicados:

```python
def test_cliente():
    cliente = {"nombre": "Ana", "ciudad": "Madrid", "vip": False}
    assert cliente == {"nombre": "Ana", "ciudad": "Sevilla", "vip": False}
```

```text
E       AssertionError: assert {'ciudad': 'Madrid', ...} == {'ciudad': 'Sevilla', ...}
E         Omitting 2 identical items, use -vv to show
E         Differing items:
E         {'ciudad': 'Madrid'} != {'ciudad': 'Sevilla'}
```

Comparaciones útiles:

```python
import math

import pytest

assert resultado == pytest.approx(0.3)                    # Decimales: 0.1 + 0.2
assert [0.1 + 0.2, 0.2 + 0.4] == pytest.approx([0.3, 0.6])
assert valor == pytest.approx(100, rel=0.01)              # ±1 %
assert valor == pytest.approx(100, abs=0.5)               # ±0,5
assert "error" in mensaje
assert set(resultado) == {"a", "b"}                       # Sin importar el orden
assert sorted(lista) == [1, 2, 3]
assert isinstance(obj, Cliente)
assert math.isclose(a, b)
assert x is None
```

Mensaje personalizado:

```python
assert total > 0, f"El total debería ser positivo; se obtuvo {total}"
```

---

## 7. Excepciones y avisos

```python
import warnings

import pytest


def test_excepcion():
    with pytest.raises(ValueError):
        int("abc")


def test_mensaje_y_detalle():
    with pytest.raises(KeyError, match=r"cliente_\d+") as info:
        raise KeyError("cliente_42")
    assert info.type is KeyError
    assert "42" in str(info.value)


def test_grupo_de_excepciones():
    with pytest.RaisesGroup(ValueError, TypeError):
        raise ExceptionGroup("varios", [ValueError(), TypeError()])


def test_aviso():
    with pytest.warns(DeprecationWarning, match="obsoleta"):
        warnings.warn("función obsoleta", DeprecationWarning)


def test_sin_avisos():
    with warnings.catch_warnings():
        warnings.simplefilter("error")       # Cualquier aviso será un error
        funcion_moderna()
```

> Use siempre `match=` para comprobar que la excepción es la esperada y no otra del mismo tipo.

---

## 8. Fixtures

### 8.1 Definir y usar

```python
import pytest

from tienda.carrito import Carrito


@pytest.fixture
def carrito() -> Carrito:
    return Carrito()


@pytest.fixture
def carrito_con_productos(carrito: Carrito) -> Carrito:      # Una fixture puede usar otras
    carrito.anadir("TEC-01", precio=59.90, cantidad=2)
    carrito.anadir("RAT-02", precio=24.50)
    return carrito


def test_total(carrito_con_productos: Carrito):
    assert carrito_con_productos.total() == pytest.approx(144.30)


def test_vacio(carrito: Carrito):
    assert carrito.total() == 0
```

Cada prueba recibe una **instancia nueva** de la fixture (ámbito `function` por defecto).

### 8.2 Preparación y limpieza con `yield`

```python
import sqlite3

import pytest


@pytest.fixture
def conexion(tmp_path):
    conn = sqlite3.connect(tmp_path / "prueba.db")
    conn.execute("CREATE TABLE clientes (id INTEGER PRIMARY KEY, nombre TEXT)")
    yield conn                          # La prueba se ejecuta aquí
    conn.close()                        # Limpieza, aunque la prueba falle
```

### 8.3 Ámbitos

```python
@pytest.fixture(scope="session")        # Una vez para toda la ejecución
def motor_bd():
    motor = crear_motor_postgres()
    yield motor
    motor.dispose()


@pytest.fixture(scope="module")         # Una vez por archivo
def datos_referencia():
    return cargar_json("tests/datos/referencia.json")
```

| Ámbito | Se crea… | Uso típico |
|---|---|---|
| `function` | En cada prueba (por defecto) | Objetos con estado, datos modificables. |
| `class` | Una vez por clase `Test*` | Agrupaciones. |
| `module` | Una vez por archivo | Datos de solo lectura costosos de cargar. |
| `package` | Una vez por paquete | Recursos compartidos de un subdirectorio. |
| `session` | Una vez en toda la ejecución | Servidor de pruebas, contenedor, motor de BD. |

> Una fixture de ámbito amplio **no** puede depender de otra de ámbito más estrecho.

### 8.4 Fixtures automáticas y factorías

```python
@pytest.fixture(autouse=True)
def sin_red(monkeypatch):
    """Impide conexiones de red accidentales en todas las pruebas."""
    import socket

    def bloquear(*args, **kwargs):
        raise RuntimeError("Acceso a red no permitido en las pruebas")

    monkeypatch.setattr(socket.socket, "connect", bloquear)


@pytest.fixture
def crear_cliente():
    """Factoría: permite crear varios objetos con distintos datos."""
    creados = []

    def _crear(nombre: str = "Cliente", vip: bool = False) -> dict:
        cliente = {"id": len(creados) + 1, "nombre": nombre, "vip": vip}
        creados.append(cliente)
        return cliente

    yield _crear
    creados.clear()


def test_varios_clientes(crear_cliente):
    a = crear_cliente("Ana", vip=True)
    b = crear_cliente("Luis")
    assert a["id"] != b["id"]
```

### 8.5 Fixtures parametrizadas

```python
@pytest.fixture(params=["sqlite", "memoria"])
def repositorio(request, tmp_path):
    if request.param == "sqlite":
        return RepositorioSQLite(tmp_path / "bd.sqlite")
    return RepositorioMemoria()


def test_guardar(repositorio):          # Se ejecuta una vez por cada implementación
    repositorio.guardar({"id": 1})
    assert repositorio.obtener(1) is not None
```

### 8.6 Usar fixtures sin recibirlas como parámetro

```python
@pytest.mark.usefixtures("bd_limpia")
def test_algo():
    ...
```

Ver las fixtures disponibles:

```bash
pytest --fixtures                  # Todas, con su descripción
pytest --fixtures-per-test         # Las que usa cada prueba
pytest --setup-show                # Traza de creación y limpieza
```

---

## 9. Fixtures integradas más útiles

| Fixture | Uso |
|---|---|
| `tmp_path` | Directorio temporal (`pathlib.Path`) único por prueba. |
| `tmp_path_factory` | Directorios temporales en fixtures de ámbito amplio. |
| `monkeypatch` | Modificar atributos, variables de entorno, diccionarios o `sys.path` temporalmente. |
| `capsys` / `capfd` | Capturar `stdout` y `stderr`. |
| `caplog` | Capturar registros de `logging`. |
| `request` | Información de la prueba actual y parámetros de la fixture. |
| `recwarn` | Registrar avisos emitidos. |
| `pytestconfig` | Acceso a la configuración y opciones de la línea de comandos. |
| `cache` | Guardar valores entre ejecuciones. |

```python
import logging


def test_archivo(tmp_path):
    archivo = tmp_path / "salida.csv"
    exportar(archivo)
    assert archivo.read_text(encoding="utf-8").startswith("id;nombre")


def test_salida(capsys):
    print("Procesados: 3")
    salida, errores = capsys.readouterr()
    assert "3" in salida


def test_log(caplog):
    with caplog.at_level(logging.WARNING, logger="tienda"):
        procesar_pedido_sin_stock()
    assert "sin stock" in caplog.text
    assert caplog.records[0].levelname == "WARNING"


def test_entorno(monkeypatch):
    monkeypatch.setenv("APP_ENTORNO", "pruebas")
    monkeypatch.delenv("APP_TOKEN", raising=False)
    monkeypatch.setattr("tienda.config.LIMITE", 5)
    monkeypatch.chdir(tmp_path)
```

---

## 10. Parametrización

```python
import pytest


@pytest.mark.parametrize(
    ("base", "descuento", "esperado"),
    [
        (100, 0.0, 121.0),
        (100, 0.1, 108.9),
        (0, 0.5, 0.0),
        (19.99, 0.0, 24.19),
    ],
)
def test_precio_final(base, descuento, esperado):
    assert precio_final(base, descuento=descuento) == esperado
```

Identificadores legibles y casos especiales:

```python
@pytest.mark.parametrize(
    ("entrada", "esperado"),
    [
        pytest.param("12345678Z", True, id="nif-valido"),
        pytest.param("12345678A", False, id="letra-incorrecta"),
        pytest.param("", False, id="vacio"),
        pytest.param("X1234567L", True, id="nie", marks=pytest.mark.xfail(reason="NIE aún no soportado")),
    ],
)
def test_validar_nif(entrada, esperado):
    assert validar_nif(entrada) is esperado
```

Combinaciones (producto cartesiano):

```python
@pytest.mark.parametrize("moneda", ["EUR", "USD"])
@pytest.mark.parametrize("cantidad", [1, 10, 100])
def test_conversion(moneda, cantidad):      # 6 ejecuciones
    ...
```

Parametrizar una prueba de errores:

```python
@pytest.mark.parametrize(
    ("base", "descuento", "mensaje"),
    [(-1, 0, "negativo"), (100, 1.5, "entre 0 y 1"), (100, -0.1, "entre 0 y 1")],
)
def test_errores(base, descuento, mensaje):
    with pytest.raises(ValueError, match=mensaje):
        precio_final(base, descuento=descuento)
```

---

## 11. Marcas (*markers*)

```python
import sys

import pytest


@pytest.mark.skip(reason="Pendiente de la API v2")
def test_futuro(): ...


@pytest.mark.skipif(sys.platform == "win32", reason="Usa señales POSIX")
def test_senales(): ...


@pytest.mark.xfail(reason="Error conocido #123", strict=True)
def test_error_conocido(): ...


@pytest.mark.lento
def test_informe_anual(): ...


@pytest.mark.red
def test_api_externa(): ...


def test_dependencia_opcional():
    pd = pytest.importorskip("pandas")      # Omite si pandas no está instalado
    ...
```

Registrar las marcas propias (obligatorio con `--strict-markers`):

```toml
[tool.pytest]
markers = [
    "lento: pruebas que tardan más de 1 segundo",
    "red: requieren acceso a servicios externos",
    "integracion: requieren base de datos",
]
```

```bash
pytest -m "not lento and not red"        # Ejecución rápida local
pytest -m integracion                    # Solo integración
```

Aplicar una marca a todo un archivo:

```python
pytestmark = [pytest.mark.integracion, pytest.mark.lento]
```

---

## 12. Dobles de prueba: `monkeypatch` y `unittest.mock`

### 12.1 `monkeypatch` para sustituir funciones

```python
def test_tipo_cambio(monkeypatch):
    monkeypatch.setattr("tienda.divisas.obtener_tipo", lambda origen, destino: 1.10)
    assert convertir(100, "EUR", "USD") == 110.0
```

### 12.2 `unittest.mock`

```python
from unittest.mock import MagicMock, call, patch


def test_envia_correo_al_confirmar():
    correo = MagicMock()
    servicio = ServicioPedidos(correo=correo)

    servicio.confirmar(pedido_id=7)

    correo.enviar.assert_called_once_with(destinatario="ana@ejemplo.com", asunto="Pedido 7 confirmado")


def test_reintentos():
    with patch("tienda.pagos.pasarela.cobrar", side_effect=[TimeoutError, TimeoutError, "OK"]) as cobrar:
        assert cobrar_con_reintentos(importe=50) == "OK"
    assert cobrar.call_count == 3
    cobrar.assert_has_calls([call(50)] * 3)


def test_fecha_fija():
    with patch("tienda.facturas.datetime") as dt:
        dt.now.return_value = datetime(2026, 12, 31, 23, 59)
        assert numero_factura() == "2026-0001"
```

| Atributo / método | Uso |
|---|---|
| `return_value` | Valor que devuelve el doble. |
| `side_effect` | Excepción a lanzar, función a ejecutar o lista de valores sucesivos. |
| `assert_called_once_with(...)` | Verifica una única llamada con esos argumentos. |
| `assert_not_called()` | Verifica que no se llamó. |
| `call_args`, `call_args_list`, `call_count` | Inspección de las llamadas. |
| `spec=Clase` / `autospec=True` | El doble respeta la interfaz real (detecta métodos mal escritos). |

> **Parchee donde se usa, no donde se define**: si `tienda/pagos.py` hace `from pasarela import cobrar`, el destino es `"tienda.pagos.cobrar"`.

> Prefiera **inyectar dependencias** (pasar el cliente de correo al constructor) a parchear módulos: las pruebas son más claras y robustas.

---

## 13. Organización del proyecto y `conftest.py`

```
proyecto/
├── pyproject.toml
├── src/
│   └── tienda/
│       ├── __init__.py
│       ├── carrito.py
│       └── pagos.py
└── tests/
    ├── conftest.py                 # Fixtures comunes a todas las pruebas
    ├── datos/                      # Archivos de prueba (JSON, CSV)
    ├── unitarias/
    │   ├── test_carrito.py
    │   └── test_pagos.py
    └── integracion/
        ├── conftest.py             # Fixtures solo para integración (BD, contenedores)
        └── test_repositorio.py
```

- `conftest.py` **no se importa**: pytest lo carga automáticamente y sus fixtures están disponibles en su directorio y subdirectorios.
- Use la estructura `src/` e instale el proyecto en modo editable (`uv sync`, `pip install -e .`) para que las pruebas usen el paquete instalado.
- Con `--import-mode=importlib` no necesita `__init__.py` en `tests/`, pero los nombres de archivo de prueba deben ser únicos o estar en paquetes distintos.

Clases para agrupar pruebas relacionadas:

```python
class TestCarrito:
    def test_anadir(self, carrito):
        ...

    def test_eliminar(self, carrito):
        ...
```

---

## 14. Configuración

Desde **pytest 9**, la tabla `[tool.pytest]` admite tipos TOML nativos. En versiones anteriores, use `[tool.pytest.ini_options]` (sigue siendo válida).

```toml
[tool.pytest]
minversion = "9.0"
testpaths = ["tests"]
addopts = ["-ra", "--import-mode=importlib"]
strict = true                       # strict_config, strict_markers, strict_xfail…
xfail_strict = true
markers = [
    "lento: pruebas que tardan más de 1 s",
    "red: requieren acceso a red",
]
filterwarnings = [
    "error",                                          # Los avisos fallan la prueba
    "ignore::DeprecationWarning:libreria_antigua.*",
]
log_cli = false
log_cli_level = "INFO"
console_output_style = "progress"
```

| Opción | Uso |
|---|---|
| `testpaths` | Dónde buscar pruebas si no se indica ruta. |
| `addopts` | Opciones añadidas siempre a la línea de comandos. |
| `strict` / `--strict-markers` | Error si se usa una marca no registrada (evita erratas). |
| `filterwarnings` | Convertir avisos en errores o ignorarlos. |
| `python_files`, `python_classes`, `python_functions` | Cambiar los patrones de descubrimiento. |
| `norecursedirs` | Directorios que no se exploran. |
| `log_cli` | Mostrar registros en vivo. |
| `timeout` | Con `pytest-timeout`: límite por prueba. |

---

## 15. Código asíncrono

Con el *plugin* `pytest-asyncio`:

```bash
uv add --dev pytest-asyncio
```

```toml
[tool.pytest]
asyncio_mode = "auto"                     # No hace falta marcar cada prueba
asyncio_default_fixture_loop_scope = "function"
```

```python
import httpx
import pytest


@pytest.fixture
async def cliente():
    async with httpx.AsyncClient(base_url="http://prueba") as c:
        yield c


async def test_consulta(cliente):
    resultado = await obtener_cotizacion("EUR", cliente=cliente)
    assert resultado > 0
```

> Alternativa: el *plugin* de **AnyIO** (`@pytest.mark.anyio`), útil si su código usa AnyIO o Trio.

---

## 16. *Plugins* recomendados

| *Plugin* | Uso |
|---|---|
| `pytest-cov` | Cobertura de código. |
| `pytest-xdist` | Ejecución en paralelo: `pytest -n auto`. |
| `pytest-asyncio` / `anyio` | Pruebas asíncronas. |
| `pytest-mock` | Fixture `mocker` (envoltorio de `unittest.mock`). |
| `pytest-randomly` | Orden aleatorio (detecta dependencias entre pruebas). |
| `pytest-timeout` | Límite de tiempo por prueba. |
| `pytest-httpx` / `respx` | Simular respuestas HTTP de httpx. |
| `responses` | Simular respuestas HTTP de requests. |
| `pytest-django` | Integración con Django. |
| `pytest-benchmark` | Medición de rendimiento. |
| `syrupy` | Pruebas de instantánea (*snapshot*). |
| `freezegun` / `time-machine` | Congelar la fecha y hora. |
| `testcontainers` | Bases de datos reales en contenedores Docker. |
| `hypothesis` | Pruebas basadas en propiedades. |

```bash
pytest -n auto                       # pytest-xdist: un proceso por núcleo
pytest -p randomly -p no:randomly    # Activar/desactivar un plugin
```

---

## 17. Depuración de pruebas

```bash
pytest --pdb                     # Abre el depurador en el primer fallo
pytest --trace                   # Abre el depurador al inicio de cada prueba
pytest -x --pdb --lf             # Depurar la última que falló
pytest -l --tb=long              # Variables locales y traza completa
pytest -s                        # Ver print()
pytest --log-cli-level=DEBUG     # Registros en vivo
```

Punto de ruptura en el código:

```python
def test_complejo():
    datos = preparar()
    breakpoint()                 # pytest gestiona la captura de salida automáticamente
    assert procesar(datos)
```

En VS Code y PyCharm, las pruebas se descubren y depuran desde el panel de pruebas usando la configuración de `pyproject.toml`.

---

## 18. Ejemplo práctico completo

**Batería de pruebas de un servicio de pedidos** con: fixtures en `conftest.py` (repositorio SQLite temporal, factorías, reloj fijo), parametrización con IDs, dobles con `MagicMock(spec=...)`, comprobación de registros con `caplog`, excepciones con `match`, archivos con `tmp_path` y marcas registradas.

### 18.1 Código de producción (`src/tienda/pedidos.py`)

```python
from __future__ import annotations

import csv
import logging
import sqlite3
from dataclasses import dataclass
from datetime import datetime
from pathlib import Path
from typing import Callable, Protocol

log = logging.getLogger("tienda.pedidos")


class StockInsuficiente(Exception):
    pass


@dataclass(frozen=True)
class Linea:
    sku: str
    cantidad: int
    precio: float


class Notificador(Protocol):
    def enviar(self, destinatario: str, asunto: str) -> None: ...


class Repositorio:
    def __init__(self, ruta: Path | str) -> None:
        self.conn = sqlite3.connect(ruta)
        self.conn.executescript("""
            CREATE TABLE IF NOT EXISTS stock (sku TEXT PRIMARY KEY, unidades INTEGER NOT NULL);
            CREATE TABLE IF NOT EXISTS pedidos (id INTEGER PRIMARY KEY, cliente TEXT, total REAL, fecha TEXT);
        """)

    def stock(self, sku: str) -> int:
        fila = self.conn.execute("SELECT unidades FROM stock WHERE sku = ?", (sku,)).fetchone()
        return fila[0] if fila else 0

    def fijar_stock(self, sku: str, unidades: int) -> None:
        with self.conn:
            self.conn.execute("INSERT OR REPLACE INTO stock VALUES (?, ?)", (sku, unidades))

    def registrar(self, cliente: str, lineas: list[Linea], total: float, fecha: datetime) -> int:
        with self.conn:
            for l in lineas:
                self.conn.execute("UPDATE stock SET unidades = unidades - ? WHERE sku = ?", (l.cantidad, l.sku))
            cur = self.conn.execute("INSERT INTO pedidos (cliente, total, fecha) VALUES (?, ?, ?)",
                                    (cliente, total, fecha.isoformat()))
        return int(cur.lastrowid)

    def pedidos(self) -> list[tuple]:
        return self.conn.execute("SELECT id, cliente, total, fecha FROM pedidos ORDER BY id").fetchall()


def calcular_total(lineas: list[Linea], cupon: str | None = None) -> float:
    if not lineas:
        raise ValueError("El pedido no tiene líneas")
    if any(l.cantidad <= 0 for l in lineas):
        raise ValueError("Las cantidades deben ser positivas")
    subtotal = sum(l.cantidad * l.precio for l in lineas)
    descuento = {"BIENVENIDA": 0.10, "VIP": 0.20}.get((cupon or "").upper(), 0.0)
    envio = 0.0 if subtotal >= 50 else 4.95
    return round(subtotal * (1 - descuento) + envio, 2)


class ServicioPedidos:
    def __init__(self, repo: Repositorio, notificador: Notificador,
                 reloj: Callable[[], datetime] = datetime.now) -> None:
        self.repo, self.notificador, self.reloj = repo, notificador, reloj

    def crear(self, cliente: str, lineas: list[Linea], cupon: str | None = None) -> int:
        for l in lineas:
            disponible = self.repo.stock(l.sku)
            if disponible < l.cantidad:
                log.warning("Stock insuficiente de %s: %s < %s", l.sku, disponible, l.cantidad)
                raise StockInsuficiente(f"{l.sku}: disponibles {disponible}, solicitadas {l.cantidad}")
        total = calcular_total(lineas, cupon)
        id_ = self.repo.registrar(cliente, lineas, total, self.reloj())
        self.notificador.enviar(cliente, f"Pedido {id_} confirmado: {total:.2f} €")
        log.info("Pedido %s creado para %s", id_, cliente)
        return id_

    def exportar_csv(self, destino: Path) -> int:
        filas = self.repo.pedidos()
        with destino.open("w", newline="", encoding="utf-8") as f:
            escritor = csv.writer(f, delimiter=";")
            escritor.writerow(["id", "cliente", "total", "fecha"])
            escritor.writerows(filas)
        return len(filas)
```

### 18.2 Fixtures compartidas (`tests/conftest.py`)

```python
from datetime import datetime
from unittest.mock import MagicMock

import pytest

from tienda.pedidos import Linea, Notificador, Repositorio, ServicioPedidos

FECHA_FIJA = datetime(2026, 11, 26, 10, 30)


@pytest.fixture
def repo(tmp_path) -> Repositorio:
    repositorio = Repositorio(tmp_path / "tienda.db")
    repositorio.fijar_stock("TEC-01", 10)
    repositorio.fijar_stock("RAT-02", 3)
    yield repositorio
    repositorio.conn.close()


@pytest.fixture
def notificador() -> MagicMock:
    return MagicMock(spec=Notificador)       # spec: detecta métodos que no existen


@pytest.fixture
def servicio(repo, notificador) -> ServicioPedidos:
    return ServicioPedidos(repo, notificador, reloj=lambda: FECHA_FIJA)


@pytest.fixture
def linea():
    """Factoría de líneas con valores por defecto razonables."""
    def _linea(sku: str = "TEC-01", cantidad: int = 1, precio: float = 59.90) -> Linea:
        return Linea(sku, cantidad, precio)
    return _linea
```

### 18.3 Pruebas unitarias (`tests/test_total.py`)

```python
import pytest

from tienda.pedidos import Linea, calcular_total


@pytest.mark.parametrize(
    ("lineas", "cupon", "esperado"),
    [
        pytest.param([Linea("A", 1, 10.0)], None, 14.95, id="con-gastos-envio"),
        pytest.param([Linea("A", 5, 10.0)], None, 50.00, id="envio-gratis-desde-50"),
        pytest.param([Linea("A", 10, 10.0)], "bienvenida", 90.00, id="cupon-10-minusculas"),
        pytest.param([Linea("A", 10, 10.0)], "VIP", 80.00, id="cupon-vip"),
        pytest.param([Linea("A", 10, 10.0)], "FALSO", 100.00, id="cupon-desconocido"),
        pytest.param([Linea("A", 2, 19.99), Linea("B", 1, 5.0)], None, 49.93, id="varias-lineas"),
    ],
)
def test_calcular_total(lineas, cupon, esperado):
    assert calcular_total(lineas, cupon) == pytest.approx(esperado)


@pytest.mark.parametrize(
    ("lineas", "mensaje"),
    [
        pytest.param([], "no tiene líneas", id="vacio"),
        pytest.param([Linea("A", 0, 10.0)], "positivas", id="cantidad-cero"),
        pytest.param([Linea("A", -2, 10.0)], "positivas", id="cantidad-negativa"),
    ],
)
def test_calcular_total_errores(lineas, mensaje):
    with pytest.raises(ValueError, match=mensaje):
        calcular_total(lineas)
```

### 18.4 Pruebas del servicio (`tests/test_servicio.py`)

```python
import logging

import pytest

from tienda.pedidos import StockInsuficiente


class TestCrearPedido:
    def test_registra_y_descuenta_stock(self, servicio, repo, linea):
        id_ = servicio.crear("ana@ejemplo.com", [linea(cantidad=2)])
        assert id_ == 1
        assert repo.stock("TEC-01") == 8
        assert repo.pedidos() == [(1, "ana@ejemplo.com", 119.8, "2026-11-26T10:30:00")]

    def test_notifica_al_cliente(self, servicio, notificador, linea):
        servicio.crear("ana@ejemplo.com", [linea(cantidad=1)])
        notificador.enviar.assert_called_once_with("ana@ejemplo.com", "Pedido 1 confirmado: 59.90 €")

    def test_sin_stock_no_registra_ni_notifica(self, servicio, repo, notificador, linea, caplog):
        with caplog.at_level(logging.WARNING, logger="tienda.pedidos"):
            with pytest.raises(StockInsuficiente, match="RAT-02: disponibles 3, solicitadas 5"):
                servicio.crear("luis@ejemplo.com", [linea(), linea("RAT-02", 5, 24.5)])
        assert repo.pedidos() == []
        assert repo.stock("TEC-01") == 10
        notificador.enviar.assert_not_called()
        assert "Stock insuficiente de RAT-02" in caplog.text

    def test_registra_en_el_log(self, servicio, linea, caplog):
        with caplog.at_level(logging.INFO, logger="tienda.pedidos"):
            servicio.crear("ana@ejemplo.com", [linea()])
        assert [r.getMessage() for r in caplog.records] == ["Pedido 1 creado para ana@ejemplo.com"]


def test_exportar_csv(servicio, linea, tmp_path):
    servicio.crear("ana@ejemplo.com", [linea()])
    servicio.crear("luis@ejemplo.com", [linea(cantidad=3)], cupon="VIP")
    destino = tmp_path / "pedidos.csv"

    assert servicio.exportar_csv(destino) == 2
    lineas = destino.read_text(encoding="utf-8").splitlines()
    assert lineas[0] == "id;cliente;total;fecha"
    assert lineas[2].startswith("2;luis@ejemplo.com;143.76;")


@pytest.mark.lento
def test_muchos_pedidos(servicio, repo, linea):
    repo.fijar_stock("TEC-01", 10_000)
    for _ in range(500):
        servicio.crear("carga@ejemplo.com", [linea()])
    assert len(repo.pedidos()) == 500
```

### 18.5 Configuración y ejecución

```toml
[tool.pytest]
testpaths = ["tests"]
addopts = ["-ra", "--import-mode=importlib"]
strict = true
markers = ["lento: pruebas que tardan más de 1 s"]
filterwarnings = ["error"]
```

```text
$ uv run pytest -v -m "not lento"
tests/test_servicio.py::TestCrearPedido::test_registra_y_descuenta_stock PASSED
tests/test_servicio.py::TestCrearPedido::test_notifica_al_cliente PASSED
tests/test_servicio.py::TestCrearPedido::test_sin_stock_no_registra_ni_notifica PASSED
tests/test_servicio.py::TestCrearPedido::test_registra_en_el_log PASSED
tests/test_servicio.py::test_exportar_csv PASSED
tests/test_total.py::test_calcular_total[con-gastos-envio] PASSED
tests/test_total.py::test_calcular_total[envio-gratis-desde-50] PASSED
tests/test_total.py::test_calcular_total[cupon-10-minusculas] PASSED
tests/test_total.py::test_calcular_total[cupon-vip] PASSED
tests/test_total.py::test_calcular_total[cupon-desconocido] PASSED
tests/test_total.py::test_calcular_total[varias-lineas] PASSED
tests/test_total.py::test_calcular_total_errores[vacio] PASSED
tests/test_total.py::test_calcular_total_errores[cantidad-cero] PASSED
tests/test_total.py::test_calcular_total_errores[cantidad-negativa] PASSED

======================= 14 passed, 1 deselected in 0.12s =======================
```

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'mi_paquete'` | El paquete no está instalado o no está en `sys.path`. | Instálelo en editable (`uv sync`, `pip install -e .`) o use `pythonpath = ["src"]`. |
| `fixture 'x' not found` | Errata, o la fixture está en un `conftest.py` de otro directorio. | `pytest --fixtures`; mueva la fixture a un `conftest.py` común. |
| `no tests ran` (código 5) | Nombres que no siguen el patrón `test_*`. | Renombre archivos y funciones o ajuste `python_files`. |
| `import file mismatch` | Dos archivos de prueba con el mismo nombre en carpetas distintas. | `--import-mode=importlib` o añada `__init__.py`. |
| `'lento' not found in markers configuration` | Marca no registrada con `strict`. | Regístrela en `markers`. |
| `ScopeMismatch` | Fixture de ámbito amplio que usa otra de ámbito menor. | Ajuste los ámbitos. |
| El *mock* no tiene efecto | Se parcheó donde se define en lugar de donde se usa. | Parchee la ruta del módulo que lo importa. |
| Las pruebas pasan solas y fallan juntas | Estado compartido (globales, BD, archivos). | Use fixtures con limpieza; detecte con `pytest-randomly`. |
| Comparación de decimales falla | Precisión de coma flotante. | `pytest.approx`. |
| `PytestUnraisableExceptionWarning` / avisos convertidos en error | `filterwarnings = ["error"]` y un aviso real. | Corrija la causa o ignore ese aviso concreto. |

---

## 20. Buenas prácticas

- Una prueba comprueba **un comportamiento**; nombre descriptivo (`test_sin_stock_no_registra_ni_notifica`).
- Estructura **Preparar – Actuar – Comprobar** (*Arrange–Act–Assert*).
- Pruebas **independientes** y deterministas: sin orden implícito, sin red, con fecha y aleatoriedad controladas.
- Use fixtures para la preparación y `yield` para la limpieza; `tmp_path` para archivos.
- Parametrice en lugar de copiar pruebas; dé **IDs** legibles a los casos.
- Inyecte dependencias (notificadores, relojes, clientes HTTP) para poder sustituirlas sin parchear.
- Use `spec`/`autospec` en los dobles para detectar desajustes con la interfaz real.
- Active `strict = true` y `filterwarnings = ["error"]` para no ocultar problemas.
- Separe pruebas rápidas y lentas con marcas; ejecute las rápidas al guardar y todas en CI.
- Mida la cobertura con **pytest-cov**, pero valore la calidad de las aserciones, no solo el porcentaje.

---

## 21. Referencia rápida

```bash
pytest [ruta[::prueba]] [-k expr] [-m marca] [-x] [--lf] [--ff] [-v|-q] [-s] [-ra]
pytest --pdb · --durations=10 · --co · --fixtures · --setup-show · -n auto (xdist)
```

```python
assert x == pytest.approx(y)
with pytest.raises(Exc, match="texto") as info: ...
with pytest.warns(Aviso): ...
@pytest.fixture(scope="function|module|session", autouse=False, params=[...])
def f(): ...; yield recurso; limpieza
@pytest.mark.parametrize(("a", "b"), [(1, 2), pytest.param(3, 4, id="x")])
@pytest.mark.skip / skipif(cond, reason=) / xfail(reason=, strict=True) / usefixtures("f")
pytest.importorskip("modulo")
tmp_path · monkeypatch · capsys · caplog · request
MagicMock(spec=C) · patch("modulo.donde.se.usa", return_value=, side_effect=)
```

---

## 22. Ejercicios propuestos

1. Escriba pruebas parametrizadas para una función de validación de IBAN, con IDs descriptivos.
2. Cree una fixture de ámbito `session` que levante un servidor HTTP local y pruebas que lo consulten.
3. Sustituya un `patch` de un módulo por inyección de dependencias y compare la legibilidad de ambas pruebas.
4. Active `pytest-randomly` en un proyecto propio y corrija las dependencias de orden que aparezcan.
5. Amplíe el ejemplo con una prueba de integración marcada como `integracion` que use PostgreSQL mediante `testcontainers`.

---

## 23. Referencias

- Documentación oficial: https://docs.pytest.org/
- Fixtures: https://docs.pytest.org/en/stable/how-to/fixtures.html
- Parametrización: https://docs.pytest.org/en/stable/how-to/parametrize.html
- Configuración: https://docs.pytest.org/en/stable/reference/customize.html
- Buenas prácticas de organización: https://docs.pytest.org/en/stable/explanation/goodpractices.html
- `unittest.mock`: https://docs.python.org/es/3/library/unittest.mock.html
- Lista de *plugins*: https://docs.pytest.org/en/stable/reference/plugin_list.html
