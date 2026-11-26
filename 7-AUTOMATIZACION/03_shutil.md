# Manual de uso: Shutil

> **Categoría:** Automatización y sistema · **Utilidad:** Copiar, mover y eliminar archivos
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Copiar archivos](#4-copiar-archivos)
5. [Copiar carpetas completas](#5-copiar-carpetas-completas)
6. [Mover y renombrar](#6-mover-y-renombrar)
7. [Eliminar carpetas](#7-eliminar-carpetas)
8. [Comprimir y descomprimir](#8-comprimir-y-descomprimir)
9. [Utilidades del sistema](#9-utilidades-del-sistema)
10. [Operaciones avanzadas](#10-operaciones-avanzadas)
11. [Ejemplo práctico completo](#11-ejemplo-práctico-completo)
12. [Errores frecuentes](#12-errores-frecuentes)
13. [Buenas prácticas](#13-buenas-prácticas)
14. [Referencia rápida](#14-referencia-rápida)
15. [Ejercicios propuestos](#15-ejercicios-propuestos)
16. [Referencias](#16-referencias)

---

## 1. Descripción

`shutil` (*shell utilities*) proporciona operaciones de alto nivel sobre archivos y carpetas que en la terminal se harían con `cp`, `mv`, `rm -r`, `zip` o `tar`:

- Copiar archivos conservando o no sus metadatos.
- Copiar árboles completos de carpetas.
- Mover archivos y carpetas, incluso entre discos.
- Eliminar carpetas con todo su contenido.
- Crear y extraer archivos comprimidos (ZIP, TAR).
- Consultar el espacio en disco y localizar ejecutables.

Complementa a `os` y `pathlib`, y acepta tanto cadenas como objetos `Path`.

---

## 2. Instalación e importación

No requiere instalación.

```python
import shutil
from pathlib import Path
```

---

## 3. Conceptos clave

| Función | Contenido copiado | Metadatos | Destino puede ser carpeta |
|---|---|---|---|
| `copyfile(src, dst)` | Solo datos | No | No |
| `copy(src, dst)` | Datos + permisos | Permisos | Sí |
| `copy2(src, dst)` | Datos + permisos + fechas | Todos los posibles | Sí |
| `copytree(src, dst)` | Carpeta completa | Usa `copy2` | Debe no existir (salvo `dirs_exist_ok`) |
| `move(src, dst)` | Mueve archivo o carpeta | Se conservan | Sí |
| `rmtree(ruta)` | Elimina carpeta y contenido | — | — |

> **Metadatos** = fechas de modificación/acceso, permisos y, cuando el sistema lo permite, atributos extendidos. En general, use **`copy2`** para copias de seguridad.

---

## 4. Copiar archivos

### 4.1 `copy2` (recomendado)

```python
shutil.copy2("informe.xlsx", "copias/informe.xlsx")    # Copia con fechas originales
shutil.copy2("informe.xlsx", "copias/")                # Si el destino es una carpeta, conserva el nombre
```

### 4.2 `copy` y `copyfile`

```python
shutil.copy("plantilla.docx", "contratos/contrato_001.docx")   # Sin conservar fechas
shutil.copyfile("datos.csv", "datos_backup.csv")              # Solo contenido; el destino debe ser un archivo
```

### 4.3 Valor de retorno

Todas devuelven la ruta del archivo de destino:

```python
destino = shutil.copy2("informe.pdf", "archivo/")
print(destino)    # 'archivo/informe.pdf'
```

### 4.4 Copiar varios archivos

```python
from pathlib import Path

destino = Path("copias")
destino.mkdir(exist_ok=True)

for archivo in Path("datos").glob("*.csv"):
    shutil.copy2(archivo, destino / archivo.name)
```

### 4.5 Copiar sin sobrescribir

```python
def copiar_si_no_existe(origen: Path, destino: Path) -> bool:
    if destino.exists():
        return False
    destino.parent.mkdir(parents=True, exist_ok=True)
    shutil.copy2(origen, destino)
    return True
```

### 4.6 Copiar solo si el origen es más reciente

```python
def copiar_si_cambia(origen: Path, destino: Path) -> bool:
    if destino.exists():
        o, d = origen.stat(), destino.stat()
        if o.st_size == d.st_size and o.st_mtime <= d.st_mtime:
            return False
    destino.parent.mkdir(parents=True, exist_ok=True)
    shutil.copy2(origen, destino)
    return True
```

### 4.7 Copiar metadatos únicamente

```python
shutil.copymode("origen.sh", "destino.sh")    # Solo permisos
shutil.copystat("origen.txt", "destino.txt")  # Permisos y fechas
```

---

## 5. Copiar carpetas completas

### 5.1 `copytree`

```python
shutil.copytree("proyecto", "proyecto_backup")
```

> Por defecto, el destino **no debe existir**.

### 5.2 Fusionar con una carpeta existente

```python
shutil.copytree("plantillas", "salida/plantillas", dirs_exist_ok=True)   # Python 3.8+
```

Los archivos existentes se sobrescriben.

### 5.3 Excluir archivos o carpetas

```python
shutil.copytree(
    "proyecto",
    "entrega",
    ignore=shutil.ignore_patterns("*.pyc", "__pycache__", ".git", ".venv", "*.log", ".env"),
)
```

### 5.4 Filtro personalizado

La función de filtro recibe la carpeta actual y la lista de nombres, y devuelve los nombres a **ignorar**:

```python
def ignorar_grandes(carpeta: str, nombres: list[str]) -> set[str]:
    limite = 50 * 1024 * 1024        # 50 MB
    ignorar = set()
    for nombre in nombres:
        ruta = Path(carpeta) / nombre
        if ruta.is_file() and ruta.stat().st_size > limite:
            ignorar.add(nombre)
    return ignorar

shutil.copytree("fotos", "fotos_ligeras", ignore=ignorar_grandes)
```

Combinar filtros:

```python
patrones = shutil.ignore_patterns("*.tmp", ".git")

def filtro(carpeta, nombres):
    return set(patrones(carpeta, nombres)) | ignorar_grandes(carpeta, nombres)
```

### 5.5 Función de copia personalizada (con registro)

```python
def copiar_y_registrar(origen, destino):
    print(f"Copiando {origen}")
    return shutil.copy2(origen, destino)

shutil.copytree("datos", "backup", copy_function=copiar_y_registrar)
```

### 5.6 Enlaces simbólicos

```python
shutil.copytree("origen", "destino", symlinks=True)    # Copia los enlaces como enlaces
```

---

## 6. Mover y renombrar

### 6.1 `move`

```python
shutil.move("informe.pdf", "archivo/")                  # Mover a una carpeta
shutil.move("informe.pdf", "archivo/informe_2026.pdf")  # Mover y renombrar
shutil.move("proyecto_antiguo", "archivo/")             # Mover carpeta completa
```

- Funciona entre **discos distintos** (copia y elimina el original), a diferencia de `os.rename`.
- Si el destino es una carpeta existente, el elemento se mueve **dentro** de ella.
- Si el destino es un archivo existente, puede sobrescribirse (según el sistema).

### 6.2 Mover sin sobrescribir

```python
def mover_seguro(origen: Path, carpeta_destino: Path) -> Path:
    carpeta_destino.mkdir(parents=True, exist_ok=True)
    destino = carpeta_destino / origen.name
    n = 1
    while destino.exists():
        destino = carpeta_destino / f"{origen.stem} ({n}){origen.suffix}"
        n += 1
    return Path(shutil.move(origen, destino))
```

### 6.3 Mover archivos según un criterio

```python
for pdf in Path("Descargas").glob("*.pdf"):
    if "factura" in pdf.name.lower():
        mover_seguro(pdf, Path("Contabilidad/Facturas"))
```

---

## 7. Eliminar carpetas

### 7.1 `rmtree`

```python
shutil.rmtree("carpeta_temporal")
```

> ⚠ **Elimina de forma permanente** la carpeta y todo su contenido, sin pasar por la papelera.

### 7.2 Ignorar errores

```python
shutil.rmtree("cache", ignore_errors=True)
```

### 7.3 Archivos de solo lectura (Windows)

En Windows, `rmtree` falla con archivos de solo lectura (habitual en carpetas `.git`):

```python
import os
import stat


def quitar_solo_lectura(funcion, ruta, info_excepcion):
    os.chmod(ruta, stat.S_IWRITE)
    funcion(ruta)

shutil.rmtree("repositorio", onexc=quitar_solo_lectura)     # Python 3.12+
# En versiones anteriores: shutil.rmtree("repositorio", onerror=quitar_solo_lectura)
```

### 7.4 Eliminación segura con confirmación

```python
def eliminar_carpeta(ruta: Path, raiz_permitida: Path) -> None:
    ruta = ruta.resolve()
    raiz_permitida = raiz_permitida.resolve()
    if not ruta.is_relative_to(raiz_permitida) or ruta == raiz_permitida:
        raise PermissionError(f"Ruta fuera de la zona permitida: {ruta}")
    if not ruta.is_dir():
        return
    shutil.rmtree(ruta)
```

> Antes de usar `rmtree`, verifique la ruta: un error en una variable puede borrar carpetas importantes.

### 7.5 Enviar a la papelera (alternativa)

`shutil` no usa la papelera. Para ello existe la librería externa `send2trash`:

```python
# pip install send2trash
from send2trash import send2trash
send2trash("carpeta_antigua")
```

---

## 8. Comprimir y descomprimir

### 8.1 Crear un archivo comprimido

```python
shutil.make_archive(
    base_name="backups/proyecto_2026-11-24",   # Ruta sin extensión
    format="zip",                              # zip, tar, gztar, bztar, xztar
    root_dir="proyecto",                       # Carpeta cuyo contenido se comprime
)
# Crea backups/proyecto_2026-11-24.zip
```

| Formato | Extensión | Compresión |
|---|---|---|
| `zip` | `.zip` | Sí (compatible con Windows) |
| `tar` | `.tar` | No |
| `gztar` | `.tar.gz` | gzip |
| `bztar` | `.tar.bz2` | bzip2 |
| `xztar` | `.tar.xz` | xz (mayor compresión) |

### 8.2 Incluir la carpeta raíz dentro del ZIP

```python
# El ZIP contendrá "proyecto/..." en lugar de los archivos sueltos
shutil.make_archive("backups/proyecto", "zip", root_dir=".", base_dir="proyecto")
```

### 8.3 Extraer

```python
shutil.unpack_archive("backups/proyecto.zip", "restaurado/")
shutil.unpack_archive("datos.tar.gz", "datos/")
shutil.unpack_archive("archivo.descarga", "destino/", format="zip")   # Forzar formato
```

> En Python 3.12+ puede indicar `filter="data"` al extraer TAR para bloquear rutas peligrosas. Extraiga solo archivos de origen fiable.

### 8.4 Formatos disponibles

```python
shutil.get_archive_formats()
shutil.get_unpack_formats()
```

### 8.5 Control detallado con `zipfile`

Para excluir archivos o elegir el nivel de compresión, use `zipfile`:

```python
import zipfile

with zipfile.ZipFile("entrega.zip", "w", compression=zipfile.ZIP_DEFLATED, compresslevel=9) as z:
    for archivo in Path("proyecto").rglob("*"):
        if archivo.is_file() and ".git" not in archivo.parts and archivo.suffix != ".pyc":
            z.write(archivo, archivo.relative_to("proyecto"))
```

---

## 9. Utilidades del sistema

### 9.1 Espacio en disco

```python
total, usado, libre = shutil.disk_usage("/")       # En Windows: "C:\\"
print(f"Total: {total / 1024**3:.1f} GB")
print(f"Usado: {usado / 1024**3:.1f} GB ({usado / total:.0%})")
print(f"Libre: {libre / 1024**3:.1f} GB")
```

Comprobar espacio antes de copiar:

```python
def hay_espacio(origen: Path, destino: Path, margen: float = 1.1) -> bool:
    necesario = sum(f.stat().st_size for f in origen.rglob("*") if f.is_file())
    libre = shutil.disk_usage(destino).free
    return libre > necesario * margen
```

### 9.2 Localizar ejecutables

```python
shutil.which("python")      # '/usr/bin/python' o None
shutil.which("git")
shutil.which("soffice")     # LibreOffice

if shutil.which("ffmpeg") is None:
    raise SystemExit("Es necesario instalar ffmpeg")
```

### 9.3 Tamaño de la terminal

```python
columnas, filas = shutil.get_terminal_size()
print("-" * columnas)
```

### 9.4 Cambiar propietario (Unix)

```python
shutil.chown("archivo.txt", user="www-data", group="www-data")
```

---

## 10. Operaciones avanzadas

### 10.1 Copiar contenido entre objetos de archivo

```python
with open("origen.bin", "rb") as fo, open("destino.bin", "wb") as fd:
    shutil.copyfileobj(fo, fd, length=1024 * 1024)       # Bloques de 1 MB
```

Útil con descargas, archivos comprimidos o flujos en memoria:

```python
import gzip

with gzip.open("datos.csv.gz", "rb") as fo, open("datos.csv", "wb") as fd:
    shutil.copyfileobj(fo, fd)
```

### 10.2 Copia con barra de progreso

```python
from tqdm import tqdm

def copiar_con_progreso(origen: Path, destino: Path) -> None:
    tamano = origen.stat().st_size
    with origen.open("rb") as fo, destino.open("wb") as fd, \
         tqdm(total=tamano, unit="B", unit_scale=True, desc=origen.name) as barra:
        while bloque := fo.read(1024 * 1024):
            fd.write(bloque)
            barra.update(len(bloque))
    shutil.copystat(origen, destino)
```

### 10.3 Verificar una copia con hash

```python
import hashlib

def hash_archivo(ruta: Path, algoritmo: str = "sha256") -> str:
    h = hashlib.new(algoritmo)
    with ruta.open("rb") as f:
        while bloque := f.read(1024 * 1024):
            h.update(bloque)
    return h.hexdigest()

assert hash_archivo(Path("original.iso")) == hash_archivo(Path("copia.iso"))
```

### 10.4 Rotación de copias de seguridad

```python
def rotar_backups(carpeta: Path, patron: str = "*.zip", conservar: int = 7) -> list[Path]:
    copias = sorted(carpeta.glob(patron), key=lambda p: p.stat().st_mtime, reverse=True)
    eliminadas = []
    for antigua in copias[conservar:]:
        antigua.unlink()
        eliminadas.append(antigua)
    return eliminadas
```

---

## 11. Ejemplo práctico completo

Sistema de copias de seguridad: comprueba el espacio disponible, excluye archivos innecesarios, crea un ZIP con fecha, verifica su integridad, conserva solo las últimas N copias y registra todo en un log.

```python
import logging
import shutil
import zipfile
from datetime import datetime
from pathlib import Path

# ---------------- Configuración ----------------
ORIGEN = Path.home() / "Documentos" / "Proyectos"
DESTINO = Path.home() / "Backups"
CONSERVAR = 7
EXCLUIR = ("*.tmp", "*.log", "*.pyc", "__pycache__", ".git", ".venv", "node_modules", "~$*")
# ------------------------------------------------

DESTINO.mkdir(parents=True, exist_ok=True)
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-7s | %(message)s",
    handlers=[logging.FileHandler(DESTINO / "backup.log", encoding="utf-8"), logging.StreamHandler()],
)
log = logging.getLogger("backup")


def tamano_legible(n: float) -> str:
    for unidad in ("B", "KB", "MB", "GB"):
        if n < 1024:
            return f"{n:.1f} {unidad}"
        n /= 1024
    return f"{n:.1f} TB"


def calcular_tamano(carpeta: Path) -> int:
    return sum(f.stat().st_size for f in carpeta.rglob("*") if f.is_file())


def crear_backup() -> Path:
    if not ORIGEN.is_dir():
        raise FileNotFoundError(f"No existe la carpeta de origen: {ORIGEN}")

    necesario = calcular_tamano(ORIGEN)
    libre = shutil.disk_usage(DESTINO).free
    log.info("Origen: %s (%s) · Libre en destino: %s", ORIGEN, tamano_legible(necesario), tamano_legible(libre))
    if libre < necesario * 1.2:
        raise OSError("Espacio insuficiente en el destino")

    marca = datetime.now().strftime("%Y-%m-%d_%H%M%S")
    temporal = DESTINO / f"_tmp_{marca}"
    try:
        # 1. Copia filtrada a una carpeta temporal
        shutil.copytree(ORIGEN, temporal / ORIGEN.name, ignore=shutil.ignore_patterns(*EXCLUIR))

        # 2. Compresión
        archivo = shutil.make_archive(str(DESTINO / f"backup_{ORIGEN.name}_{marca}"), "zip",
                                      root_dir=temporal, base_dir=ORIGEN.name)
        archivo = Path(archivo)
    finally:
        shutil.rmtree(temporal, ignore_errors=True)          # Limpiar siempre

    # 3. Verificación de integridad
    with zipfile.ZipFile(archivo) as z:
        corrupto = z.testzip()
        if corrupto:
            raise RuntimeError(f"Archivo dañado dentro del ZIP: {corrupto}")
        n_archivos = len(z.namelist())

    log.info("Copia creada: %s · %d archivos · %s", archivo.name, n_archivos,
             tamano_legible(archivo.stat().st_size))
    return archivo


def rotar() -> None:
    copias = sorted(DESTINO.glob(f"backup_{ORIGEN.name}_*.zip"), key=lambda p: p.stat().st_mtime, reverse=True)
    for antigua in copias[CONSERVAR:]:
        antigua.unlink()
        log.info("Copia antigua eliminada: %s", antigua.name)


def restaurar(archivo: Path, destino: Path) -> None:
    if destino.exists() and any(destino.iterdir()):
        raise FileExistsError(f"El destino no está vacío: {destino}")
    shutil.unpack_archive(archivo, destino)
    log.info("Restaurado %s en %s", archivo.name, destino)


if __name__ == "__main__":
    try:
        crear_backup()
        rotar()
    except Exception:
        log.exception("La copia de seguridad ha fallado")
        raise SystemExit(1)
```

> Programe este script con **Schedule**, el Programador de tareas de Windows o `cron` para ejecutarlo automáticamente.

---

## 12. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `FileExistsError` en `copytree` | La carpeta de destino ya existe. | Use `dirs_exist_ok=True` o elimine el destino. |
| `SameFileError` | Origen y destino son el mismo archivo. | Revise las rutas. |
| `IsADirectoryError` con `copyfile` | El destino es una carpeta. | Use `copy2` o indique el nombre de archivo. |
| `PermissionError` en `rmtree` (Windows) | Archivos de solo lectura o abiertos. | Use `onexc`/`onerror` o cierre los programas. |
| `shutil.Error` en `move` | El destino ya contiene un elemento con ese nombre. | Compruebe antes o genere un nombre único. |
| ZIP con rutas completas o vacío | `root_dir` / `base_dir` mal definidos. | `root_dir` = carpeta desde la que se calculan las rutas; `base_dir` = qué incluir. |
| `make_archive` crea `.zip.zip` | Se incluyó la extensión en `base_name`. | Indique el nombre **sin** extensión. |
| Fechas modificadas en la copia | Uso de `copy` o `copyfile`. | Use `copy2`. |
| El ZIP se incluye a sí mismo | Destino dentro de la carpeta de origen. | Guarde el archivo fuera de `root_dir`. |

---

## 13. Buenas prácticas

- Use `copy2` para copias de seguridad (conserva fechas).
- Use `pathlib` para construir las rutas y `shutil` para las operaciones.
- Verifique las rutas antes de `rmtree`; limite las eliminaciones a carpetas controladas.
- Incluya un **modo simulación** en scripts que mueven o eliminan.
- Excluya archivos innecesarios con `ignore_patterns`.
- Compruebe el espacio disponible antes de copias grandes.
- Verifique los archivos comprimidos (`testzip`) y, si es crítico, compare hashes.
- Registre las operaciones con `logging`.
- Limpie las carpetas temporales en un bloque `finally`.

---

## 14. Referencia rápida

```python
shutil.copy2(src, dst)                         # Copiar archivo con metadatos
shutil.copy(src, dst) / copyfile(src, dst)
shutil.copytree(src, dst, ignore=shutil.ignore_patterns("*.tmp"), dirs_exist_ok=True)
shutil.move(src, dst)                          # Mover (también entre discos)
shutil.rmtree(ruta, ignore_errors=False, onexc=func)
shutil.make_archive(base, "zip", root_dir, base_dir)
shutil.unpack_archive(archivo, destino)
shutil.disk_usage(ruta)                        # (total, usado, libre)
shutil.which("programa")
shutil.copyfileobj(fo, fd, length)
shutil.copystat(src, dst) / copymode(src, dst)
shutil.get_terminal_size()
```

---

## 15. Ejercicios propuestos

1. Cree un script que copie todos los archivos modificados en las últimas 24 horas a una carpeta de copia.
2. Sincronice dos carpetas (copiar solo archivos nuevos o modificados) mostrando un resumen.
3. Genere un ZIP de un proyecto excluyendo `.git`, `.venv`, `__pycache__` y archivos `.env`.
4. Mueva las facturas en PDF de *Descargas* a carpetas `Facturas/AAAA/MM` según la fecha del archivo.
5. Programe copias de seguridad diarias que conserven solo las últimas 10.

---

## 16. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/shutil.html
- `zipfile`: https://docs.python.org/es/3/library/zipfile.html
