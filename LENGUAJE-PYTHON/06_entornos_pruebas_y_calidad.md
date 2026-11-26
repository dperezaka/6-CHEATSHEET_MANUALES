# Manual de uso: Entornos, paquetes, pruebas y calidad del código en Python

> **Categoría:** Programación · **Utilidad:** Gestionar dependencias, probar, depurar y mantener proyectos profesionales
> **Tipo:** Herramientas estándar (`venv`, `pip`, `unittest`, `pdb`) y de terceros (`uv`, `pytest`, `ruff`, `mypy`)

---

## Índice

1. [Descripción](#1-descripción)
2. [Conceptos clave](#2-conceptos-clave)
3. [Entornos virtuales](#3-entornos-virtuales)
4. [Gestión de paquetes con `pip`](#4-gestión-de-paquetes-con-pip)
5. [Gestión de proyectos con `uv`](#5-gestión-de-proyectos-con-uv)
6. [Estructura de un proyecto profesional](#6-estructura-de-un-proyecto-profesional)
7. [Pruebas automatizadas con `pytest`](#7-pruebas-automatizadas-con-pytest)
8. [Depuración](#8-depuración)
9. [Estilo y análisis estático](#9-estilo-y-análisis-estático)
10. [Configuración y secretos](#10-configuración-y-secretos)
11. [Rendimiento](#11-rendimiento)
12. [Concurrencia](#12-concurrencia)
13. [Control de versiones](#13-control-de-versiones)
14. [Ejemplo práctico completo](#14-ejemplo-práctico-completo)
15. [Errores frecuentes](#15-errores-frecuentes)
16. [Buenas prácticas](#16-buenas-prácticas)
17. [Referencia rápida](#17-referencia-rápida)
18. [Ejercicios propuestos](#18-ejercicios-propuestos)
19. [Referencias](#19-referencias)

---

## 1. Descripción

Escribir código que funciona es solo el primer paso. Un proyecto profesional debe poder **instalarse** en otro equipo, **probarse** automáticamente, **mantener un estilo homogéneo** y **detectar errores** antes de llegar a producción.

| Necesidad | Herramienta |
|---|---|
| Aislar las dependencias de cada proyecto | `venv` / `uv` |
| Instalar librerías | `pip` / `uv` |
| Describir el proyecto y sus dependencias | `pyproject.toml` |
| Pruebas automatizadas | `pytest` (o `unittest`, estándar) |
| Formato y errores de estilo | `ruff` |
| Comprobación de tipos | `mypy` |
| Depuración | `breakpoint()` / `pdb` / depurador del editor |
| Medir rendimiento | `timeit`, `cProfile` |

---

## 2. Conceptos clave

| Concepto | Descripción |
|---|---|
| Entorno virtual | Carpeta con un intérprete y paquetes aislados para un proyecto. |
| Dependencia | Librería externa que necesita el proyecto. |
| PyPI | Repositorio público de paquetes de Python (https://pypi.org). |
| `pyproject.toml` | Archivo estándar de configuración del proyecto y sus herramientas. |
| Archivo de bloqueo | Registra las versiones exactas instaladas para reproducir el entorno (`uv.lock`). |
| Prueba unitaria | Verifica una función o clase de forma aislada. |
| *Fixture* | Datos o recursos preparados para las pruebas. |
| *Mock* | Objeto simulado que sustituye a una dependencia real (API, base de datos). |
| *Linter* | Herramienta que detecta errores y malas prácticas sin ejecutar el código. |
| Cobertura | Porcentaje del código ejecutado por las pruebas. |

---

## 3. Entornos virtuales

Cada proyecto debe tener **su propio entorno**: evita conflictos de versiones entre proyectos y no ensucia la instalación global de Python.

```bash
python -m venv .venv                     # Crear (en la carpeta del proyecto)
```

Activar:

```bash
.venv\Scripts\activate                   # Windows (CMD / PowerShell)
source .venv/bin/activate                # macOS / Linux
```

```bash
python -c "import sys; print(sys.prefix)"    # Comprobar qué entorno está activo
deactivate                                    # Desactivar
```

> - En PowerShell, si la activación está bloqueada: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.
> - **No** suba `.venv` al control de versiones: añádalo a `.gitignore`.
> - En VS Code: `Ctrl+Shift+P` → *Python: Select Interpreter* → elija `.venv`.

---

## 4. Gestión de paquetes con `pip`

```bash
pip install requests                     # Última versión
pip install "requests>=2.31,<3"          # Rango de versiones
pip install -U requests                  # Actualizar
pip uninstall requests
pip list                                 # Paquetes instalados
pip list --outdated
pip show requests                        # Información del paquete
```

### 4.1 `requirements.txt`

```bash
pip freeze > requirements.txt            # Guardar versiones exactas
pip install -r requirements.txt          # Reproducir el entorno
```

```text
# requirements.txt
requests==2.32.3
openpyxl==3.1.5
```

> Ejecute `python -m pip` en lugar de `pip` si hay varias versiones de Python instaladas: garantiza que se usa el `pip` del intérprete correcto.

---

## 5. Gestión de proyectos con `uv`

**uv** es un gestor moderno y muy rápido que sustituye a `pip`, `venv` y otras herramientas. Crea el entorno, instala dependencias y genera un archivo de bloqueo automáticamente.

```bash
pip install uv                           # O el instalador de https://docs.astral.sh/uv/

uv init mi_proyecto                      # Crea el proyecto con pyproject.toml
cd mi_proyecto
uv add requests openpyxl                 # Añade dependencias (y crea .venv)
uv add --dev pytest ruff mypy            # Dependencias solo de desarrollo
uv remove openpyxl
uv run python main.py                    # Ejecuta dentro del entorno (sin activarlo)
uv run pytest
uv sync                                  # Reproduce el entorno desde uv.lock
uv python install 3.13                   # Instala una versión de Python concreta
```

| Archivo | Contenido | ¿Al control de versiones? |
|---|---|---|
| `pyproject.toml` | Dependencias declaradas | Sí |
| `uv.lock` | Versiones exactas resueltas | Sí |
| `.venv/` | Entorno instalado | No |

---

## 6. Estructura de un proyecto profesional

```
facturador/
├── pyproject.toml
├── README.md
├── .gitignore
├── src/
│   └── facturador/
│       ├── __init__.py
│       ├── __main__.py          # Permite: python -m facturador
│       ├── modelos.py
│       ├── calculos.py
│       └── exportar.py
└── tests/
    ├── conftest.py              # Fixtures compartidas
    ├── test_calculos.py
    └── test_exportar.py
```

> La estructura `src/` evita que las pruebas importen por error el código de la carpeta actual en lugar del paquete instalado.

### 6.1 `pyproject.toml`

```toml
[project]
name = "facturador"
version = "1.0.0"
description = "Cálculo y exportación de facturas"
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["openpyxl>=3.1"]

[project.scripts]
facturador = "facturador.__main__:main"      # Crea el comando «facturador»

[dependency-groups]
dev = ["pytest>=8", "pytest-cov", "ruff", "mypy"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-ra --strict-markers"

[tool.ruff]
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM"]

[tool.mypy]
strict = true
```

### 6.2 `.gitignore` mínimo

```text
.venv/
__pycache__/
*.pyc
.pytest_cache/
.mypy_cache/
.ruff_cache/
.coverage
dist/
build/
.env
```

---

## 7. Pruebas automatizadas con `pytest`

```bash
uv add --dev pytest          # o: pip install pytest
```

### 7.1 Primera prueba

```python
# src/facturador/calculos.py
def aplicar_descuento(importe: float, porcentaje: float) -> float:
    if not 0 <= porcentaje <= 100:
        raise ValueError("Porcentaje fuera de rango")
    return round(importe * (1 - porcentaje / 100), 2)
```

```python
# tests/test_calculos.py
import pytest
from facturador.calculos import aplicar_descuento


def test_descuento_normal():
    assert aplicar_descuento(100, 15) == 85.0


def test_sin_descuento():
    assert aplicar_descuento(59.9, 0) == 59.9


def test_porcentaje_invalido():
    with pytest.raises(ValueError, match="fuera de rango"):
        aplicar_descuento(100, 120)
```

```bash
pytest                       # Ejecuta todas las pruebas
pytest -v                    # Detallado
pytest -k descuento          # Solo las que contienen «descuento»
pytest -x                    # Se detiene en el primer fallo
pytest --lf                  # Repite solo las que fallaron
pytest --cov=facturador --cov-report=term-missing   # Cobertura (pytest-cov)
```

> Las pruebas deben estar en archivos `test_*.py` y las funciones deben empezar por `test_`.

### 7.2 Pruebas parametrizadas

```python
@pytest.mark.parametrize(("importe", "porcentaje", "esperado"), [
    (100, 10, 90.0),
    (200, 50, 100.0),
    (80, 100, 0.0),
    (19.99, 0, 19.99),
])
def test_descuentos(importe, porcentaje, esperado):
    assert aplicar_descuento(importe, porcentaje) == esperado
```

### 7.3 *Fixtures*

```python
# tests/conftest.py
import pytest
from facturador.modelos import Factura, Linea


@pytest.fixture
def factura():
    f = Factura(cliente="Acme")
    f.anadir(Linea("Teclado", 2, 59.90))
    f.anadir(Linea("Monitor", 1, 249.00))
    return f
```

```python
def test_total(factura):                       # pytest inyecta la fixture por nombre
    assert factura.base == pytest.approx(368.80)
```

Fixtures integradas útiles:

| Fixture | Uso |
|---|---|
| `tmp_path` | Carpeta temporal (`Path`) única para la prueba. |
| `monkeypatch` | Sustituir atributos, variables de entorno o funciones. |
| `capsys` | Capturar lo escrito por `print`. |
| `caplog` | Capturar mensajes de `logging`. |

```python
def test_exportar_csv(tmp_path, factura):
    ruta = tmp_path / "factura.csv"
    exportar_csv(factura, ruta)
    assert ruta.read_text(encoding="utf-8-sig").startswith("producto;")
```

### 7.4 Simular dependencias (*mocks*)

```python
from unittest.mock import patch


def test_tipo_cambio_sin_red():
    with patch("facturador.divisas.consultar_api", return_value={"USD": 1.08}):
        assert convertir(100, "USD") == 108.0


def test_variable_entorno(monkeypatch):
    monkeypatch.setenv("IVA_POR_DEFECTO", "0.10")
    assert iva_configurado() == 0.10
```

> Compare decimales con `pytest.approx` o `math.isclose`, nunca con `==` sobre `float` calculados.

### 7.5 Qué probar

- Casos normales, **casos límite** (vacío, cero, máximos) y **casos de error**.
- Una idea por prueba, con un nombre que describa el comportamiento esperado.
- Patrón **AAA**: *Arrange* (preparar), *Act* (ejecutar), *Assert* (comprobar).

---

## 8. Depuración

### 8.1 `breakpoint()`

```python
def calcular(datos):
    resultado = transformar(datos)
    breakpoint()                     # Detiene la ejecución y abre el depurador pdb
    return resultado
```

| Comando `pdb` | Acción |
|---|---|
| `p variable` | Mostrar el valor. |
| `pp variable` | Mostrar con formato. |
| `n` | Siguiente línea. |
| `s` | Entrar en la función. |
| `c` | Continuar hasta el siguiente punto de parada. |
| `l` | Ver el código alrededor. |
| `w` | Ver la pila de llamadas. |
| `q` | Salir. |

```bash
python -m pdb script.py              # Depurar desde el inicio
pytest --pdb                         # Abrir el depurador cuando falla una prueba
```

> En VS Code o PyCharm, haga clic en el margen para poner un punto de interrupción y pulse **F5**: es más cómodo que `pdb`.

### 8.2 Leer un *traceback*

```text
Traceback (most recent call last):
  File "main.py", line 12, in <module>
    total = calcular_total(pedido)
  File "calculos.py", line 5, in calcular_total
    return sum(l["importe"] for l in pedido["lineas"])
KeyError: 'importe'
```

> Lea de **abajo hacia arriba**: la última línea indica el error y la anterior, dónde se produjo. Las líneas superiores muestran el camino de llamadas.

### 8.3 Aserciones

```python
assert 0 <= porcentaje <= 100, f"Porcentaje inválido: {porcentaje}"
```

> `assert` sirve para comprobar supuestos internos durante el desarrollo. Se desactiva con `python -O`: **no** lo use para validar datos de usuario.

---

## 9. Estilo y análisis estático

### 9.1 Ruff (formato y *linter*)

```bash
ruff format .                # Formatea el código (compatible con Black)
ruff check .                 # Detecta errores y malas prácticas
ruff check . --fix           # Corrige automáticamente lo que pueda
```

Detecta, entre otros: importaciones sin usar, variables no definidas, comparaciones con `None` mediante `==`, argumentos por defecto mutables y sintaxis obsoleta.

### 9.2 mypy (tipos)

```bash
mypy src/
```

```python
def total(precios: list[float]) -> float:
    return sum(precios)

total(["10", "20"])          # mypy: Argument 1 has incompatible type "list[str]"
```

### 9.3 Automatizar con *pre-commit*

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.0                   # Use la versión más reciente disponible
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```

```bash
pip install pre-commit
pre-commit install           # Se ejecuta automáticamente antes de cada commit
```

---

## 10. Configuración y secretos

**Nunca** escriba contraseñas, claves de API o cadenas de conexión en el código.

```python
import os

API_KEY = os.environ["API_KEY"]                         # Obligatoria: KeyError si falta
DEBUG = os.getenv("DEBUG", "false").lower() == "true"   # Opcional con valor por defecto
```

Archivo `.env` (excluido del control de versiones) con `python-dotenv`:

```text
# .env
API_KEY=sk-xxxxxxxx
DB_URL=postgresql://usuario:clave@localhost/app
```

```python
from dotenv import load_dotenv
load_dotenv()
```

> Incluya en el repositorio un `.env.example` con las variables necesarias **sin valores reales**.

---

## 11. Rendimiento

> Primero haga que funcione, después **mida** y solo entonces optimice lo que realmente es lento.

```python
import timeit
timeit.timeit("sum(range(1000))", number=10_000)
```

```bash
python -m timeit "'-'.join(str(n) for n in range(100))"
python -m cProfile -s cumtime script.py        # Qué funciones consumen más tiempo
```

**Mejoras habituales:**

| Lento | Rápido |
|---|---|
| `x in lista` (repetido) | `x in conjunto` |
| Concatenar cadenas en un bucle (`s += ...`) | `"".join(partes)` |
| Bucles numéricos sobre millones de datos | NumPy / pandas |
| Recalcular resultados iguales | `functools.lru_cache` |
| Cargar un archivo enorme en memoria | Procesarlo línea a línea / generadores |
| `lista.pop(0)` | `collections.deque.popleft()` |

---

## 12. Concurrencia

| Tipo de tarea | Herramienta |
|---|---|
| Esperas de red o disco (E/S): descargas, API, archivos | `concurrent.futures.ThreadPoolExecutor` o `asyncio` |
| Cálculo intensivo de CPU | `concurrent.futures.ProcessPoolExecutor` |

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import urllib.request

def descargar(url: str) -> tuple[str, int]:
    with urllib.request.urlopen(url, timeout=10) as r:
        return url, len(r.read())

with ThreadPoolExecutor(max_workers=8) as pool:
    futuros = [pool.submit(descargar, u) for u in urls]
    for futuro in as_completed(futuros):
        try:
            url, tamano = futuro.result()
            print(f"{url}: {tamano} bytes")
        except OSError as e:
            print(f"Error: {e}")
```

```python
import asyncio

async def tarea(nombre: str, segundos: float) -> str:
    await asyncio.sleep(segundos)            # Simula una espera de E/S
    return f"{nombre} terminada"

async def main():
    resultados = await asyncio.gather(tarea("A", 1), tarea("B", 2), tarea("C", 1.5))
    print(resultados)                        # Tarda ~2 s, no 4,5 s

asyncio.run(main())
```

> Para cálculo intensivo con `ProcessPoolExecutor`, el código que lanza los procesos debe estar dentro de `if __name__ == "__main__":` (obligatorio en Windows y macOS).

---

## 13. Control de versiones

Flujo mínimo con **Git**:

```bash
git init
git add .
git commit -m "Versión inicial del facturador"
git switch -c nueva-funcionalidad        # Rama de trabajo
git status / git diff / git log --oneline
```

> Haga *commits* pequeños y frecuentes, con mensajes que expliquen **qué** cambia y **por qué**. Ejecute las pruebas antes de cada *commit*.

---

## 14. Ejemplo práctico completo

**Proyecto con pruebas**: un módulo de validación de datos de clientes (NIF/DNI, IBAN, email, código postal) y su batería de pruebas con `pytest`, usando parametrización, *fixtures*, `tmp_path` y comprobación de excepciones.

**`src/validadores/__init__.py`**

```python
"""Validadores de datos de clientes españoles."""
import csv
import re
from dataclasses import dataclass, field
from pathlib import Path

LETRAS_DNI = "TRWAGMYFPDXBNJZSQVHLCKE"
PATRON_EMAIL = re.compile(r"^[\w.+-]+@[\w-]+(\.[\w-]+)+$")


class ErrorValidacion(ValueError):
    """El dato no supera la validación."""


def validar_dni(valor: str) -> str:
    """Valida un DNI o NIE y lo devuelve normalizado (mayúsculas, sin separadores)."""
    dni = re.sub(r"[\s.-]", "", valor).upper()
    if not re.fullmatch(r"[XYZ0-9]\d{7}[A-Z]", dni):
        raise ErrorValidacion(f"Formato de DNI/NIE no válido: {valor!r}")
    numero = int(dni[:8].translate(str.maketrans("XYZ", "012")))     # NIE: X→0, Y→1, Z→2
    if LETRAS_DNI[numero % 23] != dni[-1]:
        raise ErrorValidacion(f"Letra de control incorrecta: {valor!r}")
    return dni


def validar_iban(valor: str) -> str:
    """Valida un IBAN (algoritmo ISO 13616, módulo 97) y lo devuelve normalizado."""
    iban = re.sub(r"\s", "", valor).upper()
    if not re.fullmatch(r"[A-Z]{2}\d{2}[A-Z0-9]{10,30}", iban):
        raise ErrorValidacion(f"Formato de IBAN no válido: {valor!r}")
    if iban.startswith("ES") and len(iban) != 24:
        raise ErrorValidacion("Un IBAN español debe tener 24 caracteres")
    reordenado = iban[4:] + iban[:4]
    numerico = "".join(str(int(c, 36)) for c in reordenado)
    if int(numerico) % 97 != 1:
        raise ErrorValidacion(f"Dígitos de control del IBAN incorrectos: {valor!r}")
    return iban


def validar_email(valor: str) -> str:
    email = valor.strip().lower()
    if not PATRON_EMAIL.match(email):
        raise ErrorValidacion(f"Email no válido: {valor!r}")
    return email


def validar_codigo_postal(valor: str) -> str:
    cp = valor.strip()
    if not re.fullmatch(r"\d{5}", cp) or not 1 <= int(cp[:2]) <= 52:
        raise ErrorValidacion(f"Código postal no válido: {valor!r}")
    return cp


VALIDADORES = {"dni": validar_dni, "iban": validar_iban, "email": validar_email, "cp": validar_codigo_postal}


@dataclass
class Resultado:
    validos: list[dict[str, str]] = field(default_factory=list)
    errores: list[tuple[int, str, str]] = field(default_factory=list)   # (línea, campo, mensaje)


def validar_csv(ruta: Path) -> Resultado:
    """Valida un CSV (separador ';') con columnas dni, iban, email y cp."""
    resultado = Resultado()
    with ruta.open(newline="", encoding="utf-8-sig") as f:
        for linea, fila in enumerate(csv.DictReader(f, delimiter=";"), start=2):
            limpia, errores = {}, []
            for campo, funcion in VALIDADORES.items():
                try:
                    limpia[campo] = funcion(fila.get(campo) or "")
                except ErrorValidacion as e:
                    errores.append((linea, campo, str(e)))
            if errores:
                resultado.errores += errores
            else:
                resultado.validos.append(limpia)
    return resultado
```

**`tests/test_validadores.py`**

```python
import pytest

from validadores import (ErrorValidacion, validar_codigo_postal, validar_csv, validar_dni,
                         validar_email, validar_iban)


# ---------------------------------------------------------------- DNI / NIE
@pytest.mark.parametrize(("entrada", "esperado"), [
    ("12345678Z", "12345678Z"),
    ("12.345.678-z", "12345678Z"),
    ("X1234567L", "X1234567L"),
    (" 00000000t ", "00000000T"),
])
def test_dni_valido(entrada, esperado):
    assert validar_dni(entrada) == esperado


@pytest.mark.parametrize("entrada", ["12345678A", "1234567Z", "ABCDEFGHZ", ""])
def test_dni_invalido(entrada):
    with pytest.raises(ErrorValidacion):
        validar_dni(entrada)


# ---------------------------------------------------------------- IBAN
def test_iban_valido_con_espacios():
    assert validar_iban("es91 2100 0418 4502 0005 1332") == "ES9121000418450200051332"


@pytest.mark.parametrize(("entrada", "mensaje"), [
    ("ES9121000418450200051333", "control"),
    ("ES91210004184502", "24 caracteres"),
    ("9121000418450200051332", "Formato"),
])
def test_iban_invalido(entrada, mensaje):
    with pytest.raises(ErrorValidacion, match=mensaje):
        validar_iban(entrada)


# ---------------------------------------------------------------- Email y CP
def test_email_normalizado():
    assert validar_email("  Ana.Ruiz@Ejemplo.ES ") == "ana.ruiz@ejemplo.es"


@pytest.mark.parametrize("cp", ["28001", "08080", "52001"])
def test_cp_valido(cp):
    assert validar_codigo_postal(cp) == cp


@pytest.mark.parametrize("cp", ["00100", "53001", "2800", "28A01"])
def test_cp_invalido(cp):
    with pytest.raises(ErrorValidacion):
        validar_codigo_postal(cp)


# ---------------------------------------------------------------- CSV completo
@pytest.fixture
def csv_clientes(tmp_path):
    ruta = tmp_path / "clientes.csv"
    ruta.write_text(
        "dni;iban;email;cp\n"
        "12345678Z;ES9121000418450200051332;ana@ejemplo.com;28001\n"
        "12345678A;ES9121000418450200051332;luis@ejemplo;99999\n",
        encoding="utf-8-sig",
    )
    return ruta


def test_validar_csv(csv_clientes):
    resultado = validar_csv(csv_clientes)
    assert len(resultado.validos) == 1
    assert resultado.validos[0]["email"] == "ana@ejemplo.com"
    assert {campo for _, campo, _ in resultado.errores} == {"dni", "email", "cp"}
    assert all(linea == 3 for linea, _, _ in resultado.errores)
```

**Ejecución:**

```bash
uv init --lib validadores && cd validadores      # o cree la estructura manualmente
uv add --dev pytest pytest-cov
uv run pytest -v --cov=validadores --cov-report=term-missing
```

---

## 15. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError` aunque el paquete está instalado | Se instaló en otro intérprete o entorno. | Active `.venv` y use `python -m pip install`. |
| `error: externally-managed-environment` (Linux/macOS) | Intento de instalar en el Python del sistema. | Cree un entorno virtual. |
| No se puede activar `.venv` en PowerShell | Política de ejecución de scripts. | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`. |
| Funciona en su equipo y no en otro | Dependencias sin versiones fijadas. | Use `uv.lock` o `pip freeze > requirements.txt`. |
| `pytest` no encuentra pruebas | Archivos o funciones sin el prefijo `test_`. | Renómbrelos. |
| `ImportError` al ejecutar las pruebas | El paquete no está instalado en el entorno. | `uv sync` o `pip install -e .`. |
| Prueba con `float` falla por decimales | `0.1 + 0.2 == 0.3` es falso. | `pytest.approx()`. |
| Las pruebas pasan solas y fallan juntas | Estado compartido entre pruebas. | Use *fixtures* nuevas en cada prueba; evite variables globales. |
| Claves o contraseñas publicadas en el repositorio | Secretos en el código o `.env` versionado. | Variables de entorno y `.env` en `.gitignore`; revoque la clave expuesta. |
| `RuntimeError` al usar `ProcessPoolExecutor` | Falta `if __name__ == "__main__":`. | Proteja el punto de entrada. |

---

## 16. Buenas prácticas

- Un **entorno virtual por proyecto**; nunca instale librerías en el Python del sistema.
- Declare las dependencias en `pyproject.toml` y fije versiones con un archivo de bloqueo.
- Use la estructura `src/` + `tests/` en proyectos que vayan a crecer.
- Escriba pruebas para la lógica de negocio y ejecútelas antes de cada *commit*.
- Pruebe los casos límite y los de error, no solo el caso feliz.
- Formatee y revise el código automáticamente con `ruff`; compruebe tipos con `mypy`.
- Mantenga los secretos fuera del código y del control de versiones.
- Mida antes de optimizar.
- Documente en el `README.md` cómo instalar, ejecutar y probar el proyecto.
- Mantenga actualizadas las dependencias y revise sus avisos de seguridad (`pip-audit`).

---

## 17. Referencia rápida

```bash
python -m venv .venv  ·  .venv\Scripts\activate  ·  source .venv/bin/activate  ·  deactivate
python -m pip install pkg  ·  pip install -r requirements.txt  ·  pip freeze > requirements.txt
uv init  ·  uv add pkg  ·  uv add --dev pytest  ·  uv run script.py  ·  uv sync
pytest -v -k nombre -x --lf --pdb --cov=paquete
ruff format .  ·  ruff check . --fix  ·  mypy src/
python -m pdb script.py  ·  python -m cProfile -s cumtime script.py  ·  python -m timeit "expr"
```

```python
assert f(x) == esperado / pytest.raises(Exc, match="texto") / pytest.approx(v)
@pytest.mark.parametrize(("a", "b"), [(1, 2), (3, 4)])  /  @pytest.fixture  /  tmp_path, monkeypatch, capsys
breakpoint()  /  os.environ["CLAVE"]  /  os.getenv("CLAVE", "defecto")
ThreadPoolExecutor(max_workers=n)  /  asyncio.run(main())  /  asyncio.gather(...)
```

---

## 18. Ejercicios propuestos

1. Cree un proyecto con `uv`, añada `requests` como dependencia y escriba un script que consulte una API pública y guarde el resultado en JSON.
2. Escriba una batería de pruebas parametrizadas para las funciones de los manuales anteriores (cálculo de IVA, validación de entradas, préstamo).
3. Configure `ruff` y `mypy` en un proyecto existente y corrija todos los avisos.
4. Use `cProfile` para localizar la función más lenta de un script y optimícela, midiendo la mejora con `timeit`.
5. Amplíe el ejemplo completo con un validador de CIF de empresa y una interfaz de línea de comandos (`python -m validadores clientes.csv`) con sus pruebas.

---

## 19. Referencias

- `venv`: https://docs.python.org/es/3/library/venv.html
- Guía de empaquetado de Python: https://packaging.python.org/
- uv: https://docs.astral.sh/uv/
- pytest: https://docs.pytest.org/
- Ruff: https://docs.astral.sh/ruff/
- mypy: https://mypy.readthedocs.io/
- `pdb`: https://docs.python.org/es/3/library/pdb.html
- `concurrent.futures`: https://docs.python.org/es/3/library/concurrent.futures.html
- `asyncio`: https://docs.python.org/es/3/library/asyncio.html
