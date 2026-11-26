# Manual de uso: Fundamentos de Python

> **Categoría:** Programación · **Utilidad:** Sintaxis básica, tipos de datos y control de flujo
> **Tipo:** Lenguaje de programación de propósito general (requiere instalar el intérprete)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y comprobación](#2-instalación-y-comprobación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer programa y ejecución](#4-primer-programa-y-ejecución)
5. [Variables y tipos de datos](#5-variables-y-tipos-de-datos)
6. [Operadores](#6-operadores)
7. [Cadenas de texto](#7-cadenas-de-texto)
8. [Entrada y salida por consola](#8-entrada-y-salida-por-consola)
9. [Control de flujo: condicionales](#9-control-de-flujo-condicionales)
10. [Bucles](#10-bucles)
11. [Ejemplo práctico completo](#11-ejemplo-práctico-completo)
12. [Errores frecuentes](#12-errores-frecuentes)
13. [Buenas prácticas](#13-buenas-prácticas)
14. [Referencia rápida](#14-referencia-rápida)
15. [Ejercicios propuestos](#15-ejercicios-propuestos)
16. [Referencias](#16-referencias)

---

## 1. Descripción

**Python** es un lenguaje de programación interpretado, de tipado dinámico y de propósito general. Destaca por una sintaxis legible, basada en la **indentación**, y por un ecosistema de librerías muy amplio.

**Ventajas:**
- Sintaxis clara y cercana al lenguaje natural.
- Multiplataforma: Windows, macOS y Linux.
- Librería estándar muy completa («baterías incluidas»).
- Ecosistema enorme: análisis de datos, web, automatización, IA, scripting, interfaces gráficas.
- Ideal tanto para el aprendizaje como para el uso profesional.

**Limitaciones:**
- Más lento que lenguajes compilados (C, Rust, Go) en cálculo intensivo puro (se compensa con librerías escritas en C, como NumPy).
- El tipado dinámico permite errores que solo aparecen en ejecución (se mitiga con anotaciones de tipo y herramientas como `mypy`).
- Poco habitual en desarrollo móvil nativo y en navegadores.

| Si necesita… | Herramienta en Python |
|---|---|
| Automatizar tareas y scripts | Librería estándar (`pathlib`, `shutil`, `subprocess`) |
| Análisis de datos | pandas, NumPy, Polars |
| Desarrollo web | Django, FastAPI, Flask |
| Interfaces gráficas | Tkinter, PySide6 |
| Aprendizaje automático | scikit-learn, PyTorch |

---

## 2. Instalación y comprobación

**Windows y macOS:** descargue el instalador desde https://www.python.org/downloads/.

> En Windows, marque la casilla **«Add python.exe to PATH»** durante la instalación.

**Linux:** Python suele venir preinstalado. Si no:

```bash
sudo apt install python3 python3-pip python3-venv     # Debian / Ubuntu
sudo dnf install python3 python3-pip                  # Fedora
```

Comprobar la instalación:

```bash
python --version          # Windows
python3 --version         # macOS / Linux
```

> Este manual asume **Python 3.12 o superior**. Python 2 está obsoleto desde 2020 y no debe usarse.

**Editor recomendado:** Visual Studio Code con la extensión oficial *Python*, o PyCharm.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Intérprete | Programa que lee y ejecuta el código Python línea a línea. |
| Script | Archivo de texto con extensión `.py` que contiene código. |
| REPL | Consola interactiva (`>>>`) para probar código al momento. |
| Indentación | Sangría (4 espacios) que define los bloques de código. Es obligatoria. |
| Variable | Nombre que hace referencia a un valor en memoria. |
| Tipo | Clase del valor: `int`, `str`, `list`… Python lo determina en ejecución. |
| Función | Bloque de código reutilizable con nombre (`print`, `len`…). |
| Módulo | Archivo `.py` que se puede importar desde otro. |
| Excepción | Error que se produce durante la ejecución y que puede capturarse. |

---

## 4. Primer programa y ejecución

### 4.1 Consola interactiva (REPL)

```bash
python
```

```python
>>> 2 + 3
5
>>> print("Hola, mundo")
Hola, mundo
>>> exit()
```

### 4.2 Script

Cree el archivo `hola.py`:

```python
print("Hola, mundo")
```

Ejecútelo:

```bash
python hola.py
```

### 4.3 Comentarios

```python
# Comentario de una línea

"""
Cadena de varias líneas.
Se usa como documentación (docstring) al inicio de módulos, funciones y clases.
"""
```

### 4.4 Punto de entrada

```python
def main() -> None:
    print("Programa principal")


if __name__ == "__main__":     # Solo se ejecuta si el archivo se lanza directamente
    main()
```

> Este patrón evita que el código se ejecute cuando el archivo se importa como módulo.

---

## 5. Variables y tipos de datos

### 5.1 Asignación

```python
nombre = "Ana"
edad = 34
altura = 1.68
activo = True
sin_valor = None

a, b, c = 1, 2, 3              # Asignación múltiple
x = y = 0                      # Mismo valor
a, b = b, a                    # Intercambio
```

> Las variables no se declaran: se crean al asignarles un valor. Use nombres en `snake_case`.

### 5.2 Tipos básicos

| Tipo | Ejemplo | Descripción |
|---|---|---|
| `int` | `42`, `-7`, `1_000_000` | Entero (precisión ilimitada) |
| `float` | `3.14`, `1e-3` | Decimal de coma flotante |
| `complex` | `2 + 3j` | Número complejo |
| `bool` | `True`, `False` | Booleano |
| `str` | `"texto"`, `'texto'` | Cadena de texto (inmutable) |
| `NoneType` | `None` | Ausencia de valor |

### 5.3 Consultar y convertir tipos

```python
type(42)                       # <class 'int'>
isinstance(42, int)            # True
isinstance(3.0, (int, float))  # True

int("25")                      # 25
float("3.5")                   # 3.5
str(100)                       # "100"
bool(0), bool(""), bool([])    # False, False, False (valores «falsos»)
int(3.99)                      # 3 (trunca, no redondea)
round(3.456, 2)                # 3.46
```

> Son **falsos** en un contexto booleano: `0`, `0.0`, `""`, `None`, `[]`, `{}`, `()`, `set()`. Todo lo demás es verdadero.

### 5.4 Precisión de los decimales

```python
0.1 + 0.2                      # 0.30000000000000004

from decimal import Decimal
Decimal("0.1") + Decimal("0.2")    # Decimal('0.3')  → use Decimal para importes
```

### 5.5 Constantes

Python no tiene constantes reales. Por convención se escriben en mayúsculas:

```python
IVA = 0.21
MAX_INTENTOS = 3
```

---

## 6. Operadores

### 6.1 Aritméticos

| Operador | Operación | Ejemplo | Resultado |
|---|---|---|---|
| `+` | Suma | `7 + 2` | `9` |
| `-` | Resta | `7 - 2` | `5` |
| `*` | Multiplicación | `7 * 2` | `14` |
| `/` | División (siempre `float`) | `7 / 2` | `3.5` |
| `//` | División entera | `7 // 2` | `3` |
| `%` | Resto (módulo) | `7 % 2` | `1` |
| `**` | Potencia | `7 ** 2` | `49` |

```python
total = 100
total += 10        # total = total + 10   (también -=, *=, /=, //=, %=, **=)
```

### 6.2 Comparación

| Operador | Significado |
|---|---|
| `==` / `!=` | Igual / distinto |
| `<` `<=` `>` `>=` | Menor, menor o igual, mayor, mayor o igual |
| `is` / `is not` | Mismo objeto (use solo con `None`, `True`, `False`) |
| `in` / `not in` | Pertenencia a una colección |

```python
18 <= edad < 65                # Comparaciones encadenadas
"a" in "casa"                  # True
valor is None                  # Forma correcta de comprobar None
```

### 6.3 Lógicos

```python
if edad >= 18 and tiene_dni:
    ...
if es_admin or es_propietario:
    ...
if not activo:
    ...
```

> `and` y `or` usan **evaluación en cortocircuito**: `nombre = entrada or "Anónimo"` asigna `"Anónimo"` si `entrada` está vacía.

### 6.4 Operador morsa (`:=`)

Asigna y evalúa en la misma expresión:

```python
if (n := len(datos)) > 100:
    print(f"Demasiados elementos: {n}")
```

---

## 7. Cadenas de texto

### 7.1 Creación

```python
simple = 'texto'
doble = "texto con 'comillas'"
multilinea = """Primera línea
Segunda línea"""
ruta = r"C:\carpeta\nuevo"      # Cadena «raw»: no interpreta \n
```

### 7.2 f-strings (formato recomendado)

```python
nombre, precio, cantidad = "Teclado", 59.9, 3

f"{nombre}: {precio} €"               # 'Teclado: 59.9 €'
f"Total: {precio * cantidad:.2f} €"   # 'Total: 179.70 €'
f"{1234567.891:,.2f}"                 # '1,234,567.89'
f"{0.256:.1%}"                        # '25.6%'
f"{42:05d}"                           # '00042'
f"|{nombre:<10}|{nombre:>10}|{nombre:^10}|"   # Alineación
f"{precio=}"                          # 'precio=59.9' (útil para depurar)
```

### 7.3 Índices y porciones (*slicing*)

```python
texto = "Python"
texto[0]          # 'P'
texto[-1]         # 'n'   (desde el final)
texto[0:3]        # 'Pyt' (el final no se incluye)
texto[2:]         # 'thon'
texto[::-1]       # 'nohtyP' (invertida)
len(texto)        # 6
```

### 7.4 Métodos más usados

```python
s = "  Hola Mundo  "
s.strip()                       # 'Hola Mundo'  (también lstrip, rstrip)
s.lower(), s.upper()            # minúsculas / mayúsculas
s.title()                       # 'Hola Mundo'
s.replace("Mundo", "Python")
s.startswith("  Ho"), s.endswith("o  ")
s.find("Mundo")                 # Posición o -1
s.count("o")

"a,b,c".split(",")              # ['a', 'b', 'c']
" - ".join(["a", "b", "c"])     # 'a - b - c'

"123".isdigit()                 # True
"abc".isalpha()                 # True
"factura_2026.pdf".removesuffix(".pdf")    # 'factura_2026'
```

> Las cadenas son **inmutables**: los métodos devuelven una cadena nueva; no modifican la original.

---

## 8. Entrada y salida por consola

### 8.1 `print`

```python
print("Hola", "mundo")                 # Hola mundo
print("a", "b", "c", sep=" | ")        # a | b | c
print("Sin salto", end="")
print("Error grave", file=sys.stderr)  # Salida de error
```

### 8.2 `input`

```python
nombre = input("Nombre: ")             # Siempre devuelve str
edad = int(input("Edad: "))            # Conversión necesaria
```

Lectura segura de un número:

```python
while True:
    try:
        edad = int(input("Edad: "))
        break
    except ValueError:
        print("Introduzca un número entero.")
```

---

## 9. Control de flujo: condicionales

### 9.1 `if` / `elif` / `else`

```python
if nota >= 9:
    calificacion = "Sobresaliente"
elif nota >= 7:
    calificacion = "Notable"
elif nota >= 5:
    calificacion = "Aprobado"
else:
    calificacion = "Suspenso"
```

### 9.2 Expresión condicional (ternaria)

```python
estado = "mayor de edad" if edad >= 18 else "menor de edad"
```

### 9.3 `match` (coincidencia de patrones)

```python
match comando.split():
    case ["salir"]:
        terminar()
    case ["abrir", archivo]:
        abrir(archivo)
    case ["copiar", origen, destino]:
        copiar(origen, destino)
    case _:
        print("Comando no reconocido")
```

```python
match codigo_http:
    case 200 | 201:
        print("Correcto")
    case 404:
        print("No encontrado")
    case n if 500 <= n < 600:
        print("Error del servidor")
```

> `match` no es un simple `switch`: también desestructura listas, diccionarios y objetos.

---

## 10. Bucles

### 10.1 `for`

```python
for fruta in ["manzana", "pera", "uva"]:
    print(fruta)

for i in range(5):            # 0, 1, 2, 3, 4
    ...
for i in range(1, 11, 2):     # 1, 3, 5, 7, 9  (inicio, fin excluido, paso)
    ...
```

### 10.2 `enumerate` y `zip`

```python
for posicion, nombre in enumerate(nombres, start=1):
    print(f"{posicion}. {nombre}")

for nombre, nota in zip(nombres, notas):
    print(f"{nombre}: {nota}")
```

### 10.3 `while`

```python
intentos = 0
while intentos < 3:
    intentos += 1
```

### 10.4 `break`, `continue` y `else`

```python
for linea in lineas:
    if not linea.strip():
        continue               # Salta a la siguiente iteración
    if linea == "FIN":
        break                  # Sale del bucle
    procesar(linea)
else:
    print("Se procesaron todas las líneas sin encontrar FIN")   # Solo si no hubo break
```

### 10.5 Bucles sobre diccionarios

```python
precios = {"pan": 1.2, "leche": 0.95}
for producto, precio in precios.items():
    print(f"{producto}: {precio:.2f} €")
```

---

## 11. Ejemplo práctico completo

**Calculadora de factura** por consola: solicita líneas de producto con validación, calcula subtotal, IVA y descuento por volumen, y muestra un ticket formateado.

```python
"""
Calculadora de factura por consola.
Solo usa la librería estándar.
"""
from decimal import Decimal, ROUND_HALF_UP

IVA = Decimal("0.21")
UMBRAL_DESCUENTO = Decimal("200")
DESCUENTO = Decimal("0.05")
CENTIMOS = Decimal("0.01")


def pedir_decimal(mensaje: str, minimo: Decimal = Decimal("0")) -> Decimal:
    while True:
        texto = input(mensaje).strip().replace(",", ".")
        try:
            valor = Decimal(texto)
        except ArithmeticError:
            print("  ⚠ Introduzca un número válido.")
            continue
        if valor < minimo:
            print(f"  ⚠ El valor debe ser mayor o igual que {minimo}.")
            continue
        return valor


def pedir_entero(mensaje: str, minimo: int = 1) -> int:
    while True:
        try:
            valor = int(input(mensaje))
        except ValueError:
            print("  ⚠ Introduzca un número entero.")
            continue
        if valor >= minimo:
            return valor
        print(f"  ⚠ El valor debe ser al menos {minimo}.")


def redondear(valor: Decimal) -> Decimal:
    return valor.quantize(CENTIMOS, rounding=ROUND_HALF_UP)


def main() -> None:
    print("=== CALCULADORA DE FACTURA ===")
    print("Deje el nombre del producto vacío para terminar.\n")

    lineas = []
    while True:
        producto = input("Producto: ").strip()
        if not producto:
            break
        precio = pedir_decimal("  Precio unitario (€): ")
        cantidad = pedir_entero("  Cantidad: ")
        lineas.append((producto, precio, cantidad))

    if not lineas:
        print("No se ha introducido ningún producto.")
        return

    subtotal = sum((precio * cantidad for _, precio, cantidad in lineas), Decimal("0"))
    descuento = subtotal * DESCUENTO if subtotal >= UMBRAL_DESCUENTO else Decimal("0")
    base = subtotal - descuento
    cuota_iva = base * IVA
    total = base + cuota_iva

    ancho = 52
    print("\n" + "=" * ancho)
    print(f"{'Producto':<24}{'Precio':>10}{'Cant.':>6}{'Importe':>12}")
    print("-" * ancho)
    for producto, precio, cantidad in lineas:
        print(f"{producto[:23]:<24}{precio:>10.2f}{cantidad:>6}{redondear(precio * cantidad):>12.2f}")
    print("-" * ancho)
    print(f"{'Subtotal':<40}{redondear(subtotal):>12.2f}")
    if descuento:
        print(f"{'Descuento ' + format(DESCUENTO, '.0%'):<40}{-redondear(descuento):>12.2f}")
    print(f"{'IVA ' + format(IVA, '.0%'):<40}{redondear(cuota_iva):>12.2f}")
    print("=" * ancho)
    print(f"{'TOTAL (€)':<40}{redondear(total):>12.2f}")


if __name__ == "__main__":
    main()
```

---

## 12. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `IndentationError: expected an indented block` | Falta la sangría tras `if`, `for`, `def`… | Indente el bloque con 4 espacios. |
| `TabError: inconsistent use of tabs and spaces` | Mezcla de tabuladores y espacios. | Configure el editor para insertar espacios. |
| `SyntaxError: expected ':'` | Falta `:` al final de `if`, `for`, `def`, `class`. | Añada los dos puntos. |
| `NameError: name 'x' is not defined` | Variable no creada o mal escrita. | Revise el nombre y el orden de asignación. |
| `TypeError: can only concatenate str (not "int") to str` | `"Edad: " + 30`. | Use f-strings: `f"Edad: {30}"`. |
| `ValueError: invalid literal for int()` | `int("abc")` o `int("3.5")`. | Valide la entrada o capture `ValueError`. |
| `IndexError: string index out of range` | Índice mayor que la longitud. | Compruebe `len()` antes de acceder. |
| `ZeroDivisionError` | División entre cero. | Compruebe el divisor antes de dividir. |
| `0.1 + 0.2 != 0.3` | Representación binaria de `float`. | Use `Decimal` o `math.isclose()`. |
| `'python' no se reconoce como un comando` | Python no está en el `PATH` (Windows). | Reinstale marcando «Add to PATH» o use `py`. |
| `if x = 5:` da `SyntaxError` | Uso de `=` en vez de `==`. | `=` asigna, `==` compara. |

---

## 13. Buenas prácticas

- Siga la guía de estilo **PEP 8**: 4 espacios, `snake_case` para variables y funciones, `MAYUSCULAS` para constantes.
- Use nombres descriptivos: `precio_total` mejor que `pt`.
- Use **f-strings** para formatear texto.
- Compare con `None` usando `is None`, nunca `== None`.
- Use `Decimal` para cantidades monetarias.
- Valide toda entrada del usuario antes de convertirla.
- Encapsule el código en funciones y use el patrón `if __name__ == "__main__":`.
- Prefiera `for` sobre colecciones a `for i in range(len(...))`; use `enumerate` si necesita el índice.
- Documente módulos y funciones con *docstrings*.
- Use un formateador automático (**Ruff** o **Black**) para mantener un estilo homogéneo.

---

## 14. Referencia rápida

```python
x = 10; s = "texto"; f = 3.14; b = True; n = None
type(x) / isinstance(x, int) / int() float() str() bool()
+ - * / // % **   == != < <= > >=   and or not   in   is None
f"{valor:.2f}" / s.strip() .lower() .upper() .split(",") ",".join(lista) .replace(a, b)
s[0] s[-1] s[1:4] s[::-1] len(s)
print(a, b, sep=", ", end="\n") / input("Mensaje: ")

if c: ... elif c: ... else: ...       /   v = a if c else b
match valor: case 1 | 2: ... case _: ...
for x in iterable: ...  / for i, x in enumerate(it): ... / for a, b in zip(x, y): ...
while c: ... / break / continue / for ... else
range(inicio, fin, paso)
```

---

## 15. Ejercicios propuestos

1. Escriba un programa que pida una temperatura en grados Celsius y la muestre en Fahrenheit y Kelvin con dos decimales.
2. Cree un juego de «adivina el número» (use `random.randint`) con un máximo de 7 intentos y pistas «mayor/menor».
3. Pida un año y determine si es bisiesto (divisible por 4 y no por 100, salvo que sea divisible por 400).
4. Muestre la tabla de multiplicar de un número con formato alineado.
5. Amplíe el ejemplo completo para que admita códigos de descuento introducidos por el usuario mediante `match`.

---

## 16. Referencias

- Documentación oficial: https://docs.python.org/es/3/
- Tutorial oficial de Python: https://docs.python.org/es/3/tutorial/
- PEP 8 – Guía de estilo: https://peps.python.org/pep-0008/
- Descargas: https://www.python.org/downloads/
