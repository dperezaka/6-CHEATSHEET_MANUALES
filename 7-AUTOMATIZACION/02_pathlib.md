# Manual de uso: Pathlib

> **Categoría:** Automatización y sistema · **Utilidad:** Manipulación de rutas y archivos
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Crear rutas](#4-crear-rutas)
5. [Componentes de una ruta](#5-componentes-de-una-ruta)
6. [Transformar rutas](#6-transformar-rutas)
7. [Comprobaciones e información](#7-comprobaciones-e-información)
8. [Leer y escribir archivos](#8-leer-y-escribir-archivos)
9. [Crear, mover y eliminar](#9-crear-mover-y-eliminar)
10. [Buscar archivos](#10-buscar-archivos)
11. [Equivalencias con os y os.path](#11-equivalencias-con-os-y-ospath)
12. [Ejemplo práctico completo](#12-ejemplo-práctico-completo)
13. [Errores frecuentes](#13-errores-frecuentes)
14. [Buenas prácticas](#14-buenas-prácticas)
15. [Referencia rápida](#15-referencia-rápida)
16. [Ejercicios propuestos](#16-ejercicios-propuestos)
17. [Referencias](#17-referencias)

---

## 1. Descripción

`pathlib` representa las rutas del sistema de archivos como **objetos** en lugar de cadenas de texto. Esto permite manipularlas con operadores y métodos claros, de forma portable entre Windows, macOS y Linux.

```python
# Con os.path
os.path.join(os.path.dirname(os.path.abspath(__file__)), "datos", "ventas.csv")

# Con pathlib
Path(__file__).resolve().parent / "datos" / "ventas.csv"
```

**Ventajas frente a `os.path`:**
- Código más legible (operador `/` para unir rutas).
- Métodos agrupados en el propio objeto (`ruta.exists()`, `ruta.read_text()`).
- Búsqueda de archivos integrada (`glob`, `rglob`).
- Compatible con casi todas las librerías modernas (Pandas, open, shutil, Pillow…).

---

## 2. Instalación e importación

No requiere instalación (Python 3.4+).

```python
from pathlib import Path
```

---

## 3. Conceptos clave

| Clase | Descripción |
|---|---|
| `Path` | Ruta concreta del sistema actual. **La que se usa habitualmente.** |
| `PurePath` | Ruta "pura": solo manipulación de texto, sin acceso al disco. |
| `PureWindowsPath` / `PurePosixPath` | Manipular rutas de otro sistema operativo. |
| `WindowsPath` / `PosixPath` | Implementaciones concretas (se eligen automáticamente). |

Los objetos `Path` son **inmutables**: los métodos de transformación devuelven una ruta nueva.

---

## 4. Crear rutas

```python
Path("datos/ventas.csv")                  # Ruta relativa
Path("C:/Usuarios/Ana/Documentos")        # Windows admite "/"
Path(r"C:\Usuarios\Ana\Documentos")       # Cadena raw
Path("datos", "2026", "ventas.csv")       # Varias partes

Path.cwd()                                # Directorio de trabajo actual
Path.home()                               # Carpeta del usuario
Path(__file__)                            # Archivo del script actual
Path(__file__).resolve().parent           # Carpeta del script (ruta absoluta)
```

### 4.1 Unir rutas con `/`

```python
base = Path.home() / "Documentos"
informe = base / "informes" / "2026" / "noviembre.xlsx"

carpeta = Path("salida")
archivo = carpeta / f"informe_{2026}.pdf"
```

`joinpath` es equivalente:

```python
base.joinpath("informes", "2026", "noviembre.xlsx")
```

### 4.2 Rutas a partir de la ubicación del script

```python
RAIZ = Path(__file__).resolve().parent
DATOS = RAIZ / "datos"
SALIDA = RAIZ / "salida"
CONFIG = RAIZ / "config.json"
```

> Este patrón hace que el script funcione igual se ejecute desde donde se ejecute.

---

## 5. Componentes de una ruta

```python
p = Path("/home/ana/informes/ventas_2026.tar.gz")

p.name          # 'ventas_2026.tar.gz'      Nombre completo
p.stem          # 'ventas_2026.tar'         Nombre sin la última extensión
p.suffix        # '.gz'                     Última extensión
p.suffixes      # ['.tar', '.gz']           Todas las extensiones
p.parent        # Path('/home/ana/informes')
p.parents[1]    # Path('/home/ana')
p.parts         # ('/', 'home', 'ana', 'informes', 'ventas_2026.tar.gz')
p.anchor        # '/'  (en Windows: 'C:\\')
p.drive         # ''   (en Windows: 'C:')
```

Recorrer las carpetas superiores:

```python
for carpeta in Path("/home/ana/informes/2026").parents:
    print(carpeta)
# /home/ana/informes
# /home/ana
# /home
# /
```

---

## 6. Transformar rutas

### 6.1 Cambiar nombre, extensión o raíz

```python
p = Path("informes/ventas.xlsx")

p.with_name("compras.xlsx")      # informes/compras.xlsx
p.with_suffix(".csv")            # informes/ventas.csv
p.with_suffix("")                # informes/ventas
p.with_stem("ventas_final")      # informes/ventas_final.xlsx   (Python 3.9+)
```

### 6.2 Rutas absolutas y relativas

```python
Path("datos").resolve()                     # Ruta absoluta, resolviendo enlaces y ".."
Path("datos").absolute()                    # Absoluta sin resolver enlaces
Path("~/Documentos").expanduser()           # Expande "~"

Path("/home/ana/a/b/c.txt").relative_to("/home/ana")     # a/b/c.txt
Path("/home/ana/a").is_relative_to("/home/ana")          # True  (3.9+)
```

### 6.3 Convertir a texto

```python
str(p)              # Texto con el separador del sistema
p.as_posix()        # Siempre con "/" (útil para URLs y configuraciones)
p.as_uri()          # 'file:///home/ana/...' (requiere ruta absoluta)
```

### 6.4 Comparar rutas

```python
Path("a/b") == Path("a/b")                 # True
Path("datos/Ventas.csv").match("*.csv")    # Coincidencia con patrón
Path("a/b/c.txt").full_match("**/*.txt")   # Python 3.13+
```

---

## 7. Comprobaciones e información

### 7.1 Existencia y tipo

```python
p.exists()          # ¿Existe?
p.is_file()         # ¿Es un archivo?
p.is_dir()          # ¿Es una carpeta?
p.is_symlink()      # ¿Es un enlace simbólico?
p.is_absolute()     # ¿Es una ruta absoluta?
p.samefile(otra)    # ¿Mismo archivo?
```

### 7.2 Tamaño, fechas y permisos

```python
from datetime import datetime

info = p.stat()
info.st_size                                   # Tamaño en bytes
datetime.fromtimestamp(info.st_mtime)          # Fecha de modificación
datetime.fromtimestamp(info.st_ctime)          # Creación (Windows) / cambio (Unix)

p.owner()           # Propietario (Unix)
p.chmod(0o644)      # Cambiar permisos
p.touch()           # Crear vacío o actualizar fecha de modificación
```

---

## 8. Leer y escribir archivos

### 8.1 Texto

```python
ruta = Path("notas.txt")

ruta.write_text("Primera línea\nSegunda línea\n", encoding="utf-8")
contenido = ruta.read_text(encoding="utf-8")
lineas = ruta.read_text(encoding="utf-8").splitlines()
```

### 8.2 Binario

```python
datos = Path("imagen.png").read_bytes()
Path("copia.png").write_bytes(datos)
```

### 8.3 Con `open` (archivos grandes, añadir contenido)

```python
with ruta.open("a", encoding="utf-8") as f:
    f.write("Nueva línea\n")

with Path("grande.log").open(encoding="utf-8") as f:
    for linea in f:                            # Línea a línea, sin cargar todo en memoria
        if "ERROR" in linea:
            print(linea.strip())
```

### 8.4 Integración con otras librerías

```python
import json, csv
import pandas as pd

config = json.loads(Path("config.json").read_text(encoding="utf-8"))
Path("salida.json").write_text(json.dumps(config, indent=2, ensure_ascii=False), encoding="utf-8")

df = pd.read_csv(Path("datos") / "ventas.csv")
df.to_excel(Path("salida") / "ventas.xlsx", index=False)
```

> `write_text` y `write_bytes` **sobrescriben** el archivo completo.

---

## 9. Crear, mover y eliminar

### 9.1 Crear carpetas

```python
Path("salida/2026/noviembre").mkdir(parents=True, exist_ok=True)
```

| Parámetro | Efecto |
|---|---|
| `parents=True` | Crea las carpetas intermedias necesarias. |
| `exist_ok=True` | No da error si la carpeta ya existe. |

### 9.2 Crear archivos vacíos

```python
Path("registro.log").touch(exist_ok=True)
```

### 9.3 Renombrar y mover

```python
origen = Path("informe.txt")

nuevo = origen.rename("informe_final.txt")        # Devuelve la nueva ruta
origen.replace("archivo/informe.txt")             # Sobrescribe el destino si existe

# Cambiar solo la extensión de un archivo existente
p = Path("datos.txt")
p.rename(p.with_suffix(".csv"))
```

> `rename` no funciona entre discos distintos: use `shutil.move(origen, destino)` (acepta objetos `Path`).

### 9.4 Eliminar

```python
Path("temporal.txt").unlink()                 # Archivo (error si no existe)
Path("temporal.txt").unlink(missing_ok=True)  # Sin error si no existe (3.8+)
Path("carpeta_vacia").rmdir()                 # Solo carpetas vacías
```

Eliminar una carpeta con todo su contenido:

```python
import shutil
shutil.rmtree(Path("carpeta"))
```

### 9.5 Copiar

`pathlib` incluye `copy()` a partir de Python 3.14. En versiones anteriores, use `shutil`:

```python
import shutil
shutil.copy2(Path("origen.xlsx"), Path("copias") / "origen.xlsx")
```

### 9.6 Enlaces simbólicos

```python
Path("enlace").symlink_to(Path("datos/actual"))
Path("enlace").readlink()
```

---

## 10. Buscar archivos

### 10.1 Listar el contenido de una carpeta

```python
carpeta = Path("datos")

for elemento in carpeta.iterdir():
    tipo = "📁" if elemento.is_dir() else "📄"
    print(tipo, elemento.name)

archivos = [p for p in carpeta.iterdir() if p.is_file()]
subcarpetas = [p for p in carpeta.iterdir() if p.is_dir()]
```

### 10.2 `glob`: buscar por patrón

```python
carpeta.glob("*.csv")              # CSV en la carpeta (no recursivo)
carpeta.glob("ventas_2026_*.xlsx")
carpeta.glob("*.[jp][pn]g")        # .jpg y .png
carpeta.glob("*/*.pdf")            # PDF un nivel por debajo
carpeta.glob("**/*.py")            # Recursivo
```

### 10.3 `rglob`: búsqueda recursiva

```python
for pdf in Path("Documentos").rglob("*.pdf"):
    print(pdf)
```

`rglob("*.pdf")` equivale a `glob("**/*.pdf")`.

| Patrón | Significado |
|---|---|
| `*` | Cualquier secuencia de caracteres (sin cruzar carpetas). |
| `?` | Un carácter cualquiera. |
| `[abc]` | Uno de los caracteres indicados. |
| `**` | Cualquier número de carpetas (búsqueda recursiva). |

> `glob` devuelve un **generador**. Conviértalo en lista si necesita reutilizar el resultado o contarlo: `list(carpeta.glob("*.csv"))`.

### 10.4 Varias extensiones

```python
EXTENSIONES = {".jpg", ".jpeg", ".png", ".webp"}
imagenes = [p for p in Path("fotos").rglob("*") if p.suffix.lower() in EXTENSIONES]
```

### 10.5 Ordenar resultados

```python
archivos = sorted(Path("informes").glob("*.pdf"))                              # Por nombre
recientes = sorted(Path("informes").glob("*.pdf"), key=lambda p: p.stat().st_mtime, reverse=True)
mas_grandes = sorted(Path(".").rglob("*"), key=lambda p: p.stat().st_size if p.is_file() else 0,
                     reverse=True)[:10]

ultimo = max(Path("backups").glob("*.zip"), key=lambda p: p.stat().st_mtime, default=None)
```

### 10.6 Excluir carpetas en búsquedas recursivas

```python
EXCLUIR = {".git", "__pycache__", ".venv", "node_modules"}

archivos_py = [
    p for p in Path("proyecto").rglob("*.py")
    if not any(parte in EXCLUIR for parte in p.parts)
]
```

### 10.7 `Path.walk` (Python 3.12+)

```python
for carpeta, subcarpetas, archivos in Path("proyecto").walk():
    subcarpetas[:] = [d for d in subcarpetas if d not in EXCLUIR]
    for nombre in archivos:
        print(carpeta / nombre)
```

---

## 11. Equivalencias con os y os.path

| `os` / `os.path` | `pathlib` |
|---|---|
| `os.getcwd()` | `Path.cwd()` |
| `os.path.expanduser("~")` | `Path.home()` |
| `os.path.join(a, b)` | `Path(a) / b` |
| `os.path.abspath(p)` | `Path(p).resolve()` |
| `os.path.basename(p)` | `p.name` |
| `os.path.dirname(p)` | `p.parent` |
| `os.path.splitext(p)` | `p.stem`, `p.suffix` |
| `os.path.exists(p)` | `p.exists()` |
| `os.path.isfile(p)` / `isdir(p)` | `p.is_file()` / `p.is_dir()` |
| `os.path.getsize(p)` | `p.stat().st_size` |
| `os.listdir(p)` | `p.iterdir()` |
| `os.makedirs(p, exist_ok=True)` | `p.mkdir(parents=True, exist_ok=True)` |
| `os.rename(a, b)` | `Path(a).rename(b)` |
| `os.remove(p)` | `p.unlink()` |
| `os.rmdir(p)` | `p.rmdir()` |
| `glob.glob("*.csv")` | `Path().glob("*.csv")` |
| `os.walk(p)` | `p.rglob("*")` / `p.walk()` (3.12+) |
| `open(p).read()` | `p.read_text()` |

---

## 12. Ejemplo práctico completo

Herramienta de mantenimiento de una carpeta de informes: renombra archivos con un formato estándar, archiva los antiguos por año/mes, elimina temporales y genera un inventario CSV.

```python
import csv
import os
import re
import shutil
import unicodedata
from datetime import datetime, timedelta
from pathlib import Path

RAIZ = Path(__file__).resolve().parent
CARPETA = RAIZ / "informes"
ARCHIVO = CARPETA / "_archivo"
INVENTARIO = CARPETA / "_inventario.csv"
DIAS_PARA_ARCHIVAR = 90
TEMPORALES = {".tmp", ".bak", ".~lock"}


def normalizar_nombre(nombre: str) -> str:
    """'Informe Ventas (Final) Ñ.xlsx' → 'informe_ventas_final_n.xlsx'"""
    p = Path(nombre)
    base = unicodedata.normalize("NFKD", p.stem).encode("ascii", "ignore").decode()
    base = re.sub(r"[^\w]+", "_", base.lower()).strip("_")
    return f"{base or 'sin_nombre'}{p.suffix.lower()}"


def destino_libre(ruta: Path) -> Path:
    contador = 1
    candidato = ruta
    while candidato.exists():
        candidato = ruta.with_stem(f"{ruta.stem}_{contador}")
        contador += 1
    return candidato


def crear_demo() -> None:
    CARPETA.mkdir(exist_ok=True)
    ejemplos = {
        "Informe Ventas (Final).xlsx": 10,
        "Resumen Q3 Ñandú.PDF": 120,
        "presupuesto 2026.docx": 200,
        "borrador.tmp": 1,
        "Acta reunión.txt": 5,
    }
    ahora = datetime.now().timestamp()
    for nombre, dias in ejemplos.items():
        p = CARPETA / nombre
        p.write_text(f"Contenido de {nombre}", encoding="utf-8")
        marca = ahora - dias * 86400
        os.utime(p, (marca, marca))             # Simula fechas antiguas


def mantenimiento(simulacion: bool = True) -> None:
    limite = datetime.now() - timedelta(days=DIAS_PARA_ARCHIVAR)
    acciones: list[str] = []

    for archivo in sorted(p for p in CARPETA.iterdir() if p.is_file()):
        if archivo.name.startswith("_"):
            continue

        # 1. Eliminar temporales
        if archivo.suffix.lower() in TEMPORALES:
            acciones.append(f"ELIMINAR   {archivo.name}")
            if not simulacion:
                archivo.unlink(missing_ok=True)
            continue

        modificado = datetime.fromtimestamp(archivo.stat().st_mtime)

        # 2. Normalizar nombre
        nombre_ok = normalizar_nombre(archivo.name)
        if nombre_ok != archivo.name:
            nuevo = destino_libre(archivo.with_name(nombre_ok))
            acciones.append(f"RENOMBRAR  {archivo.name}  →  {nuevo.name}")
            if not simulacion:
                archivo = archivo.rename(nuevo)
            nombre_actual = nuevo.name
        else:
            nombre_actual = archivo.name

        # 3. Archivar antiguos en _archivo/AAAA/MM
        if modificado < limite:
            carpeta_destino = ARCHIVO / f"{modificado:%Y}" / f"{modificado:%m}"
            destino = destino_libre(carpeta_destino / nombre_actual)
            acciones.append(f"ARCHIVAR   {nombre_actual}  →  {destino.relative_to(CARPETA).as_posix()}")
            if not simulacion:
                carpeta_destino.mkdir(parents=True, exist_ok=True)
                shutil.move(archivo, destino)

    print(f"\n{'SIMULACIÓN' if simulacion else 'EJECUCIÓN'} · {len(acciones)} acciones")
    for a in acciones:
        print("  ", a)


def generar_inventario() -> None:
    filas = []
    for p in sorted(CARPETA.rglob("*")):
        if p.is_file() and p != INVENTARIO:
            info = p.stat()
            filas.append({
                "ruta": p.relative_to(CARPETA).as_posix(),
                "nombre": p.name,
                "extension": p.suffix.lower(),
                "tamano_kb": round(info.st_size / 1024, 1),
                "modificado": datetime.fromtimestamp(info.st_mtime).strftime("%Y-%m-%d %H:%M"),
            })
    with INVENTARIO.open("w", newline="", encoding="utf-8-sig") as f:
        escritor = csv.DictWriter(f, fieldnames=list(filas[0]) if filas else ["ruta"], delimiter=";")
        escritor.writeheader()
        escritor.writerows(filas)
    print(f"\nInventario: {len(filas)} archivos → {INVENTARIO.name}")


if __name__ == "__main__":
    if not CARPETA.exists():
        crear_demo()
    mantenimiento(simulacion=True)
    mantenimiento(simulacion=False)
    generar_inventario()
```

---

## 13. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TypeError: unsupported operand type(s) for /: 'str' and 'str'` | Ninguno de los operandos es `Path`. | El primer elemento debe ser `Path`: `Path("a") / "b"`. |
| `FileNotFoundError` con rutas relativas | Se resuelven desde el directorio de trabajo, no desde el script. | Use `Path(__file__).resolve().parent`. |
| `FileExistsError` en `mkdir` | La carpeta ya existe. | Use `exist_ok=True`. |
| `FileNotFoundError` en `mkdir` | Falta la carpeta padre. | Use `parents=True`. |
| `OSError: [Errno 18] Invalid cross-device link` | `rename` entre discos distintos. | Use `shutil.move`. |
| `OSError: Directory not empty` | `rmdir` en carpeta con contenido. | Use `shutil.rmtree`. |
| `ValueError: ... is not in the subpath of ...` | `relative_to` con rutas no relacionadas. | Compruebe con `is_relative_to` o use `os.path.relpath`. |
| `glob` devuelve vacío la segunda vez | El generador ya se consumió. | Conviértalo en lista: `list(p.glob(...))`. |
| Caracteres extraños al leer | Codificación no indicada. | Use siempre `encoding="utf-8"`. |
| `AttributeError: 'PosixPath' object has no attribute 'with_stem'` | Python < 3.9. | Use `p.with_name(nuevo + p.suffix)`. |

---

## 14. Buenas prácticas

- Use `pathlib` como opción por defecto para trabajar con rutas en código nuevo.
- Defina las rutas base del proyecto como constantes a partir de `__file__`.
- Indique siempre `encoding="utf-8"` al leer y escribir texto.
- Use `mkdir(parents=True, exist_ok=True)` para crear estructuras de carpetas.
- Use `unlink(missing_ok=True)` para eliminar archivos que pueden no existir.
- Convierta los resultados de `glob` en lista cuando vaya a reutilizarlos.
- Use `as_posix()` al guardar rutas en configuraciones o informes compartidos.
- Combine `pathlib` con `shutil` para copiar, mover entre discos y eliminar carpetas completas.

---

## 15. Referencia rápida

```python
Path("a") / "b" / "c.txt"          Path.cwd()          Path.home()
Path(__file__).resolve().parent
p.name  p.stem  p.suffix  p.suffixes  p.parent  p.parents  p.parts
p.with_name()  p.with_suffix()  p.with_stem()  p.resolve()  p.expanduser()  p.relative_to()
p.exists()  p.is_file()  p.is_dir()  p.stat().st_size  p.stat().st_mtime
p.read_text(encoding="utf-8")  p.write_text(t, encoding="utf-8")  p.read_bytes()  p.open()
p.mkdir(parents=True, exist_ok=True)  p.touch()  p.rename()  p.replace()  p.unlink(missing_ok=True)  p.rmdir()
p.iterdir()  p.glob("*.csv")  p.rglob("*.pdf")  p.walk()  # 3.12+
str(p)  p.as_posix()
```

---

## 16. Ejercicios propuestos

1. Reescriba con `pathlib` un script propio que use `os.path`.
2. Escriba una función que devuelva el archivo modificado más recientemente de una carpeta con una extensión dada.
3. Organice una carpeta de fotos en subcarpetas `AAAA/MM` según su fecha de modificación.
4. Busque recursivamente todos los archivos de más de 100 MB y muéstrelos ordenados por tamaño.
5. Cambie la extensión de todos los `.txt` de una carpeta a `.md` conservando los nombres.

---

## 17. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/pathlib.html
- Correspondencia con `os`: https://docs.python.org/3/library/pathlib.html#corresponding-tools
