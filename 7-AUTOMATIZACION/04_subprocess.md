# Manual de uso: Subprocess

> **Categoría:** Automatización y sistema · **Utilidad:** Ejecutar programas y comandos del sistema
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas con run](#4-operaciones-básicas-con-run)
5. [Capturar la salida](#5-capturar-la-salida)
6. [Gestión de errores](#6-gestión-de-errores)
7. [Entrada, entorno y directorio](#7-entrada-entorno-y-directorio)
8. [Comandos de shell](#8-comandos-de-shell)
9. [Procesos en segundo plano con Popen](#9-procesos-en-segundo-plano-con-popen)
10. [Diferencias entre Windows y Linux/macOS](#10-diferencias-entre-windows-y-linuxmacos)
11. [Casos de uso habituales](#11-casos-de-uso-habituales)
12. [Ejemplo práctico completo](#12-ejemplo-práctico-completo)
13. [Seguridad](#13-seguridad)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

`subprocess` permite ejecutar programas externos y comandos del sistema desde Python, comunicarse con ellos (enviar datos, leer su salida) y obtener su código de resultado.

**Casos de uso habituales:**
- Ejecutar herramientas de línea de comandos (`git`, `ffmpeg`, `pandoc`, LibreOffice…).
- Automatizar tareas del sistema (copias, red, servicios).
- Lanzar otros scripts de Python.
- Abrir aplicaciones.
- Integrar programas que no tienen librería de Python.

Sustituye a funciones antiguas como `os.system`, `os.popen` y `commands`.

---

## 2. Instalación e importación

No requiere instalación.

```python
import subprocess
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `subprocess.run()` | Ejecuta un comando y **espera** a que termine. Uso recomendado. |
| `subprocess.Popen` | Inicia un proceso y permite interactuar con él mientras se ejecuta. |
| Argumentos como lista | `["git", "status"]`: cada elemento es un argumento. Forma segura. |
| `shell=True` | Ejecuta el comando a través del intérprete del sistema (cmd, bash). |
| `stdout` / `stderr` | Salida estándar / salida de errores. |
| `stdin` | Entrada estándar. |
| `returncode` | Código de salida: `0` = éxito; otro valor = error. |
| `CompletedProcess` | Objeto devuelto por `run()` con el resultado. |

---

## 4. Operaciones básicas con run

### 4.1 Ejecutar un comando

```python
resultado = subprocess.run(["python", "--version"])
print(resultado.returncode)      # 0 si todo fue bien
```

La salida aparece directamente en la consola.

### 4.2 Comando con argumentos

```python
subprocess.run(["git", "log", "--oneline", "-n", "5"])
subprocess.run(["ping", "-c", "3", "google.com"])          # Linux/macOS
subprocess.run(["ping", "-n", "3", "google.com"])          # Windows
```

> Cada argumento es un elemento independiente de la lista. **No** separe por espacios dentro de un mismo elemento: `["git", "log --oneline"]` es incorrecto.

### 4.3 Convertir una cadena de comando en lista

```python
import shlex

shlex.split('ffmpeg -i "mi video.mp4" -vf scale=1280:-1 salida.mp4')
# ['ffmpeg', '-i', 'mi video.mp4', '-vf', 'scale=1280:-1', 'salida.mp4']
```

### 4.4 Ejecutar otro script de Python

```python
import sys

subprocess.run([sys.executable, "otro_script.py", "--modo", "rapido"])
```

> Use `sys.executable` para asegurar que se usa el mismo intérprete (y entorno virtual) que el script actual.

### 4.5 Parámetros principales de `run`

| Parámetro | Efecto |
|---|---|
| `capture_output=True` | Captura `stdout` y `stderr`. |
| `text=True` | Devuelve texto (`str`) en lugar de bytes. |
| `encoding="utf-8"` | Codificación del texto (implica `text=True`). |
| `check=True` | Lanza excepción si el código de salida no es 0. |
| `timeout=30` | Tiempo máximo en segundos. |
| `cwd="ruta"` | Directorio de trabajo del proceso. |
| `env={...}` | Variables de entorno. |
| `input="texto"` | Datos enviados a la entrada estándar. |
| `shell=True` | Ejecutar a través del shell. |
| `stdout=`, `stderr=` | Redirigir salidas (`PIPE`, `DEVNULL`, archivo, `STDOUT`). |

---

## 5. Capturar la salida

### 5.1 Capturar como texto

```python
r = subprocess.run(["git", "status", "--short"], capture_output=True, text=True)

print(r.returncode)
print(r.stdout)        # Salida normal
print(r.stderr)        # Mensajes de error
```

### 5.2 Procesar la salida línea a línea

```python
r = subprocess.run(["git", "branch", "--format=%(refname:short)"],
                   capture_output=True, text=True, check=True)
ramas = r.stdout.strip().splitlines()
```

### 5.3 Salida en JSON

Muchas herramientas modernas ofrecen salida JSON:

```python
import json

r = subprocess.run(["pip", "list", "--format=json"], capture_output=True, text=True, check=True)
paquetes = {p["name"]: p["version"] for p in json.loads(r.stdout)}
```

### 5.4 Combinar `stdout` y `stderr`

```python
r = subprocess.run(["comando"], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, text=True)
print(r.stdout)        # Contiene ambas salidas en orden
```

### 5.5 Descartar la salida

```python
subprocess.run(["comando_ruidoso"], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
```

### 5.6 Guardar la salida en un archivo

```python
with open("salida.log", "w", encoding="utf-8") as f:
    subprocess.run(["pip", "freeze"], stdout=f, stderr=subprocess.STDOUT, text=True)
```

### 5.7 Atajo: `check_output`

```python
version = subprocess.check_output(["git", "--version"], text=True).strip()
```

### 5.8 Codificación en Windows

Algunos programas de Windows escriben en la codificación de la consola (`cp850`, `cp1252`):

```python
r = subprocess.run(["ipconfig"], capture_output=True, encoding="cp850", errors="replace")
```

> `errors="replace"` evita que un carácter no reconocido detenga el programa.

---

## 6. Gestión de errores

### 6.1 Comprobar el código de salida

```python
r = subprocess.run(["git", "pull"], capture_output=True, text=True)
if r.returncode != 0:
    print("Error:", r.stderr)
```

### 6.2 `check=True` (recomendado)

```python
try:
    subprocess.run(["git", "pull"], capture_output=True, text=True, check=True)
except subprocess.CalledProcessError as e:
    print(f"El comando falló con código {e.returncode}")
    print("Salida:", e.stdout)
    print("Error:", e.stderr)
```

### 6.3 Tiempo máximo

```python
try:
    subprocess.run(["ping", "-c", "100", "ejemplo.com"], timeout=10, check=True)
except subprocess.TimeoutExpired:
    print("El comando tardó demasiado y se ha detenido")
```

### 6.4 Programa no encontrado

```python
try:
    subprocess.run(["ffmpeg", "-version"], capture_output=True, check=True)
except FileNotFoundError:
    print("ffmpeg no está instalado o no está en el PATH")
```

Comprobación previa:

```python
import shutil
if shutil.which("ffmpeg") is None:
    raise SystemExit("Instale ffmpeg antes de continuar")
```

### 6.5 Función envoltorio reutilizable

```python
import logging
log = logging.getLogger(__name__)


def ejecutar(cmd: list[str], timeout: int = 300, **kwargs) -> str:
    log.info("Ejecutando: %s", " ".join(cmd))
    try:
        r = subprocess.run(cmd, capture_output=True, text=True, check=True, timeout=timeout, **kwargs)
    except FileNotFoundError:
        raise RuntimeError(f"Programa no encontrado: {cmd[0]}") from None
    except subprocess.TimeoutExpired:
        raise RuntimeError(f"Tiempo agotado ({timeout} s): {cmd[0]}") from None
    except subprocess.CalledProcessError as e:
        raise RuntimeError(f"{cmd[0]} terminó con código {e.returncode}:\n{e.stderr.strip()}") from None
    return r.stdout
```

---

## 7. Entrada, entorno y directorio

### 7.1 Enviar datos a la entrada estándar

```python
r = subprocess.run(["sort"], input="pera\nmanzana\nkiwi\n", capture_output=True, text=True)
print(r.stdout)

r = subprocess.run(["python", "-c", "import sys; print(sys.stdin.read().upper())"],
                   input="hola", capture_output=True, text=True)
```

### 7.2 Directorio de trabajo

```python
subprocess.run(["git", "status"], cwd="C:/Proyectos/mi_app")
```

### 7.3 Variables de entorno

```python
import os

entorno = os.environ.copy()          # Partir del entorno actual
entorno["APP_ENV"] = "pruebas"
entorno["PYTHONIOENCODING"] = "utf-8"
subprocess.run([sys.executable, "app.py"], env=entorno)
```

> Si pasa un `env` sin copiar `os.environ`, el proceso no tendrá `PATH` ni el resto de variables.

### 7.4 Encadenar comandos (tubería)

Equivalente a `dir | findstr .py` o `ls | grep .py`, sin usar el shell:

```python
p1 = subprocess.Popen(["ls", "-l"], stdout=subprocess.PIPE)
p2 = subprocess.Popen(["grep", ".py"], stdin=p1.stdout, stdout=subprocess.PIPE, text=True)
p1.stdout.close()
salida, _ = p2.communicate()
```

> En la mayoría de casos es más sencillo capturar la salida del primer comando y filtrarla con Python.

---

## 8. Comandos de shell

### 8.1 `shell=True`

Ejecuta el comando como una cadena a través de `cmd.exe` (Windows) o `/bin/sh` (Linux/macOS). Necesario para:
- Comandos internos del shell (`dir`, `echo`, `cd`, `type` en Windows).
- Tuberías, redirecciones y comodines escritos en la cadena (`|`, `>`, `*`).

```python
subprocess.run("dir /b *.txt", shell=True)                  # Windows
subprocess.run("ls *.py | wc -l", shell=True)               # Linux/macOS
subprocess.run("echo %USERNAME%", shell=True)               # Windows
```

### 8.2 Riesgo de inyección

```python
# ❌ PELIGROSO: si nombre = "x & del /q C:\\*" se ejecutaría otro comando
nombre = input("Archivo: ")
subprocess.run(f"type {nombre}", shell=True)

# ✅ SEGURO: lista de argumentos sin shell
subprocess.run(["cat", nombre])
```

Si necesita `shell=True` con datos variables, escápelos:

```python
import shlex
subprocess.run(f"grep -r {shlex.quote(texto)} .", shell=True)      # Solo Linux/macOS
```

### 8.3 PowerShell

```python
r = subprocess.run(
    ["powershell", "-NoProfile", "-Command", "Get-Process | Sort-Object CPU -Descending | Select-Object -First 5"],
    capture_output=True, text=True,
)
print(r.stdout)
```

---

## 9. Procesos en segundo plano con Popen

### 9.1 Iniciar sin esperar

```python
proceso = subprocess.Popen(["python", "servidor.py"])
print("Servidor iniciado con PID", proceso.pid)

# ... otras tareas ...

proceso.terminate()          # Solicitar finalización
proceso.wait(timeout=10)     # Esperar a que termine
```

### 9.2 Métodos de `Popen`

| Método / atributo | Función |
|---|---|
| `poll()` | `None` si sigue en ejecución; si no, el código de salida. |
| `wait(timeout)` | Espera a que termine. |
| `communicate(input, timeout)` | Envía datos, lee toda la salida y espera. |
| `terminate()` | Solicita la finalización (SIGTERM). |
| `kill()` | Fuerza la finalización (SIGKILL). |
| `pid` | Identificador del proceso. |
| `returncode` | Código de salida (cuando ha terminado). |

### 9.3 Leer la salida en tiempo real

```python
with subprocess.Popen(
    ["ping", "-c", "5", "python.org"],
    stdout=subprocess.PIPE, stderr=subprocess.STDOUT, text=True, bufsize=1,
) as p:
    for linea in p.stdout:
        print("›", linea, end="")

print("Código de salida:", p.returncode)
```

Para scripts de Python hijos, añada `-u` (salida sin búfer):

```python
subprocess.Popen([sys.executable, "-u", "tarea_larga.py"], stdout=subprocess.PIPE, text=True)
```

### 9.4 Comunicarse y esperar con tiempo máximo

```python
p = subprocess.Popen(["programa"], stdin=subprocess.PIPE, stdout=subprocess.PIPE, text=True)
try:
    salida, errores = p.communicate(input="datos\n", timeout=30)
except subprocess.TimeoutExpired:
    p.kill()
    salida, errores = p.communicate()
```

### 9.5 Lanzar varios procesos en paralelo

```python
archivos = ["video1.mp4", "video2.mp4", "video3.mp4"]
procesos = [
    subprocess.Popen(["ffmpeg", "-y", "-i", a, "-vf", "scale=1280:-1", f"hd_{a}"],
                     stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    for a in archivos
]
for a, p in zip(archivos, procesos):
    print(a, "OK" if p.wait() == 0 else "ERROR")
```

### 9.6 Proceso independiente (que sobreviva al script)

```python
if os.name == "nt":
    subprocess.Popen(["notepad.exe"],
                     creationflags=subprocess.DETACHED_PROCESS | subprocess.CREATE_NEW_PROCESS_GROUP)
else:
    subprocess.Popen(["gedit"], start_new_session=True)
```

---

## 10. Diferencias entre Windows y Linux/macOS

| Aspecto | Windows | Linux / macOS |
|---|---|---|
| Shell con `shell=True` | `cmd.exe` | `/bin/sh` |
| Comandos internos | `dir`, `copy`, `del`, `type`, `echo` (requieren `shell=True`) | `ls`, `cp`, `rm`, `cat` (son programas) |
| Separador de rutas | `\` | `/` |
| Ping | `ping -n 3 host` | `ping -c 3 host` |
| Codificación de consola | `cp850` / `cp1252` (o UTF-8 en terminales modernas) | UTF-8 |
| Ocultar ventana | `creationflags=subprocess.CREATE_NO_WINDOW` | No aplica |
| Abrir archivo | `os.startfile(r)` / `["cmd", "/c", "start", "", r]` | `["open", r]` (macOS) · `["xdg-open", r]` (Linux) |

Código portable:

```python
import sys

def comando_ping(host: str, veces: int = 3) -> list[str]:
    opcion = "-n" if sys.platform == "win32" else "-c"
    return ["ping", opcion, str(veces), host]
```

Ocultar la ventana de consola en Windows (por ejemplo, desde una aplicación gráfica):

```python
flags = subprocess.CREATE_NO_WINDOW if sys.platform == "win32" else 0
subprocess.run(["programa.exe"], creationflags=flags)
```

---

## 11. Casos de uso habituales

### 11.1 Git

```python
def git(*args: str, repo: str = ".") -> str:
    return subprocess.run(["git", *args], cwd=repo, capture_output=True, text=True, check=True).stdout.strip()

rama = git("rev-parse", "--abbrev-ref", "HEAD")
cambios = git("status", "--porcelain")
if cambios:
    git("add", "-A")
    git("commit", "-m", "Actualización automática")
```

### 11.2 Convertir documentos con LibreOffice

```python
def a_pdf(documento: str, carpeta_salida: str = ".") -> None:
    soffice = shutil.which("soffice") or shutil.which("libreoffice")
    if not soffice:
        raise RuntimeError("LibreOffice no está instalado")
    subprocess.run([soffice, "--headless", "--convert-to", "pdf", "--outdir", carpeta_salida, documento],
                   check=True, timeout=120, capture_output=True)
```

### 11.3 Procesar vídeo o audio con ffmpeg

```python
subprocess.run(["ffmpeg", "-y", "-i", "entrada.mov", "-c:v", "libx264", "-crf", "23",
                "-preset", "veryfast", "salida.mp4"], check=True)
subprocess.run(["ffmpeg", "-y", "-i", "video.mp4", "-vn", "-q:a", "2", "audio.mp3"], check=True)
```

### 11.4 Instalar paquetes desde un script

```python
subprocess.run([sys.executable, "-m", "pip", "install", "requests"], check=True)
```

### 11.5 Comprobar conectividad

```python
def hay_conexion(host: str = "8.8.8.8") -> bool:
    r = subprocess.run(comando_ping(host, 1), stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    return r.returncode == 0
```

### 11.6 Información del sistema (Windows)

```python
r = subprocess.run(["systeminfo"], capture_output=True, encoding="cp850", errors="replace")
r = subprocess.run(["wmic", "logicaldisk", "get", "caption,freespace,size"], capture_output=True, text=True)
```

### 11.7 Tareas programadas y servicios

```python
# Windows: crear una tarea programada diaria
subprocess.run(["schtasks", "/Create", "/SC", "DAILY", "/TN", "BackupDiario",
                "/TR", rf'"{sys.executable}" "C:\scripts\backup.py"', "/ST", "08:30", "/F"], check=True)

# Linux: reiniciar un servicio
subprocess.run(["sudo", "systemctl", "restart", "nginx"], check=True)
```

---

## 12. Ejemplo práctico completo

Herramienta de diagnóstico que ejecuta varios comandos (con tiempo máximo, captura y gestión de errores), comprueba la conectividad y el estado de repositorios Git, y genera un informe.

```python
import json
import platform
import shutil
import subprocess
import sys
from dataclasses import dataclass, asdict
from datetime import datetime
from pathlib import Path

ES_WINDOWS = sys.platform == "win32"


@dataclass
class Resultado:
    nombre: str
    comando: str
    ok: bool
    codigo: int | None
    duracion_s: float
    salida: str
    error: str = ""


def ejecutar(nombre: str, cmd: list[str], timeout: int = 20, cwd: str | None = None) -> Resultado:
    inicio = datetime.now()
    texto_cmd = " ".join(cmd)
    if shutil.which(cmd[0]) is None:
        return Resultado(nombre, texto_cmd, False, None, 0.0, "", f"No encontrado: {cmd[0]}")
    try:
        r = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout, cwd=cwd,
                           encoding="utf-8", errors="replace",
                           creationflags=subprocess.CREATE_NO_WINDOW if ES_WINDOWS else 0)
        duracion = (datetime.now() - inicio).total_seconds()
        return Resultado(nombre, texto_cmd, r.returncode == 0, r.returncode, round(duracion, 2),
                         r.stdout.strip()[:2000], r.stderr.strip()[:1000])
    except subprocess.TimeoutExpired:
        return Resultado(nombre, texto_cmd, False, None, float(timeout), "", f"Tiempo agotado ({timeout} s)")


def diagnostico(repos: list[Path]) -> list[Resultado]:
    ping = ["ping", "-n" if ES_WINDOWS else "-c", "2", "8.8.8.8"]
    dns = ["nslookup", "python.org"]

    pruebas = [
        ("Versión de Python", [sys.executable, "--version"]),
        ("Pip actualizado", [sys.executable, "-m", "pip", "--version"]),
        ("Conectividad (ping)", ping),
        ("Resolución DNS", dns),
        ("Git instalado", ["git", "--version"]),
    ]
    resultados = [ejecutar(n, c) for n, c in pruebas]

    for repo in repos:
        if (repo / ".git").exists():
            r = ejecutar(f"Git: {repo.name}", ["git", "status", "--porcelain", "--branch"], cwd=str(repo))
            if r.ok:
                lineas = r.salida.splitlines()
                pendientes = [l for l in lineas if not l.startswith("##")]
                r.salida = f"{lineas[0] if lineas else ''} · {len(pendientes)} cambios sin confirmar"
            resultados.append(r)
    return resultados


def informe(resultados: list[Resultado]) -> None:
    print(f"\n{'=' * 70}")
    print(f" DIAGNÓSTICO · {platform.node()} · {platform.system()} {platform.release()}")
    print(f" {datetime.now():%d/%m/%Y %H:%M:%S}")
    print("=" * 70)
    for r in resultados:
        icono = "✅" if r.ok else "❌"
        print(f"{icono} {r.nombre:<28} {r.duracion_s:>6.2f} s")
        detalle = (r.salida.splitlines() or [""])[0] if r.ok else r.error
        if detalle:
            print(f"     {detalle[:80]}")
    fallos = sum(not r.ok for r in resultados)
    print("-" * 70)
    print(f" {len(resultados) - fallos} correctos · {fallos} con incidencias")

    ruta = Path(f"diagnostico_{datetime.now():%Y%m%d_%H%M%S}.json")
    ruta.write_text(json.dumps([asdict(r) for r in resultados], indent=2, ensure_ascii=False), encoding="utf-8")
    print(f" Informe detallado: {ruta}")


if __name__ == "__main__":
    repos = [p for p in Path.cwd().iterdir() if p.is_dir()]
    informe(diagnostico(repos))
```

---

## 13. Seguridad

- **Evite `shell=True`** con datos procedentes del usuario, archivos o red: permite inyectar comandos.
- Pase siempre los argumentos como **lista**.
- No construya comandos concatenando cadenas con datos externos.
- Valide rutas y parámetros antes de ejecutarlos.
- No incluya contraseñas en los argumentos (son visibles en la lista de procesos): use variables de entorno o la entrada estándar.
- Use rutas absolutas a los ejecutables en scripts críticos, o verifíquelas con `shutil.which`.
- Ejecute con los mínimos privilegios necesarios.

---

## 14. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `FileNotFoundError: [WinError 2]` | Programa no encontrado o comando interno de cmd (`dir`, `echo`). | Compruebe el PATH; para comandos internos use `shell=True`. |
| La salida es `b'...'` | Se capturó en bytes. | Añada `text=True` o `encoding="utf-8"`. |
| `UnicodeDecodeError` | Codificación distinta (Windows). | Use `encoding="cp850"` o `errors="replace"`. |
| El script se queda bloqueado | Proceso que espera entrada o salida muy grande con `Popen` + `wait()`. | Use `communicate()` o `run()` con `timeout`. |
| `CalledProcessError` | El comando devolvió un código distinto de 0 con `check=True`. | Revise `e.stderr`. |
| Variables de entorno perdidas | Se pasó `env` sin copiar `os.environ`. | Use `env={**os.environ, "CLAVE": "valor"}`. |
| Argumentos mal interpretados | Lista con argumentos combinados: `["git", "log -n 5"]`. | Separe cada argumento o use `shlex.split`. |
| Se usa otro Python | Se ejecutó `"python"` en lugar del intérprete actual. | Use `sys.executable`. |
| `cd` no tiene efecto | Cada `run` es un proceso nuevo. | Use el parámetro `cwd`. |

---

## 15. Buenas prácticas

- Use `subprocess.run()` para casi todo; `Popen` solo para procesos de larga duración o interacción en tiempo real.
- Pase los comandos como **listas** y evite `shell=True`.
- Use `check=True` y capture `CalledProcessError`.
- Indique siempre un `timeout` en comandos que puedan bloquearse.
- Use `capture_output=True, text=True` cuando necesite procesar la salida.
- Use `sys.executable` para lanzar scripts de Python.
- Verifique que los programas existen con `shutil.which`.
- Si existe una librería de Python equivalente (por ejemplo, `shutil` en lugar de `cp`), prefiérala.
- Registre los comandos ejecutados y sus errores con `logging`.

---

## 16. Referencia rápida

```python
r = subprocess.run(["cmd", "arg"], capture_output=True, text=True, check=True,
                   timeout=30, cwd="ruta", env={**os.environ, "X": "1"}, input="datos")
r.returncode / r.stdout / r.stderr

subprocess.check_output(["cmd"], text=True)
subprocess.DEVNULL / subprocess.PIPE / subprocess.STDOUT
subprocess.CalledProcessError / subprocess.TimeoutExpired / FileNotFoundError

p = subprocess.Popen(["cmd"], stdout=subprocess.PIPE, text=True)
p.poll() / p.wait(timeout) / p.communicate(input, timeout) / p.terminate() / p.kill()
for linea in p.stdout: ...

shlex.split("cadena de comando") / shlex.quote(arg) / shutil.which("prog")
subprocess.CREATE_NO_WINDOW   # Windows
```

---

## 17. Ejercicios propuestos

1. Cree una función que devuelva un diccionario con los paquetes instalados y sus versiones usando `pip list --format=json`.
2. Escriba un script que recorra todos los repositorios Git de una carpeta y muestre cuáles tienen cambios pendientes.
3. Convierta por lotes todos los `.docx` de una carpeta a PDF con LibreOffice, con un tiempo máximo por archivo.
4. Lance un servidor local en segundo plano, compruebe que responde y deténgalo al terminar.
5. Muestre en tiempo real la salida de un proceso largo añadiendo la hora delante de cada línea.

---

## 18. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/subprocess.html
- `shlex`: https://docs.python.org/es/3/library/shlex.html
