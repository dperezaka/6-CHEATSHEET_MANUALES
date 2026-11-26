# Manual de uso: CSV

> **Categoría:** Utilidades generales · **Utilidad:** Trabajar con archivos CSV
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Operaciones intermedias](#5-operaciones-intermedias)
6. [Operaciones avanzadas](#6-operaciones-avanzadas)
7. [CSV y Excel en español](#7-csv-y-excel-en-español)
8. [Ejemplo práctico completo](#8-ejemplo-práctico-completo)
9. [Errores frecuentes](#9-errores-frecuentes)
10. [Buenas prácticas](#10-buenas-prácticas)
11. [Referencia rápida](#11-referencia-rápida)
12. [Ejercicios propuestos](#12-ejercicios-propuestos)
13. [Referencias](#13-referencias)

---

## 1. Descripción

**CSV** (*Comma-Separated Values*) es un formato de texto plano para datos tabulares: cada línea es una fila y los valores se separan por un delimitador (coma, punto y coma, tabulador…). Es el formato de intercambio más común entre hojas de cálculo, bases de datos y sistemas.

El módulo `csv` de Python lee y escribe estos archivos gestionando correctamente comillas, delimitadores y saltos de línea dentro de los campos.

---

## 2. Instalación e importación

No requiere instalación.

```python
import csv
```

---

## 3. Conceptos clave

| Elemento | Descripción |
|---|---|
| `csv.reader` | Lee cada fila como una **lista**. |
| `csv.writer` | Escribe filas a partir de listas. |
| `csv.DictReader` | Lee cada fila como un **diccionario** (claves = cabeceras). |
| `csv.DictWriter` | Escribe filas a partir de diccionarios. |
| `delimiter` | Separador de campos (`,` por defecto). |
| `quotechar` | Carácter de entrecomillado (`"` por defecto). |
| Dialecto | Conjunto de reglas de formato (`excel`, `excel-tab`, `unix`). |

> **Importante:** abra siempre los archivos CSV con `newline=""` para evitar líneas en blanco duplicadas en Windows.

---

## 4. Operaciones básicas

### 4.1 Leer un CSV como listas

```python
with open("empleados.csv", newline="", encoding="utf-8") as f:
    lector = csv.reader(f)
    cabecera = next(lector)          # Primera fila
    for fila in lector:
        print(fila)                  # ['Ana', '28', 'Madrid']
```

### 4.2 Leer un CSV como diccionarios (recomendado)

```python
with open("empleados.csv", newline="", encoding="utf-8") as f:
    for fila in csv.DictReader(f):
        print(fila["nombre"], fila["edad"])
```

### 4.3 Escribir un CSV desde listas

```python
filas = [
    ["nombre", "edad", "ciudad"],
    ["Ana", 28, "Madrid"],
    ["Luis", 35, "Sevilla"],
]

with open("salida.csv", "w", newline="", encoding="utf-8") as f:
    escritor = csv.writer(f)
    escritor.writerows(filas)        # Varias filas
    escritor.writerow(["Marta", 42, "Valencia"])   # Una fila
```

### 4.4 Escribir un CSV desde diccionarios

```python
empleados = [
    {"nombre": "Ana", "edad": 28, "ciudad": "Madrid"},
    {"nombre": "Luis", "edad": 35, "ciudad": "Sevilla"},
]

with open("salida.csv", "w", newline="", encoding="utf-8") as f:
    escritor = csv.DictWriter(f, fieldnames=["nombre", "edad", "ciudad"])
    escritor.writeheader()
    escritor.writerows(empleados)
```

### 4.5 Añadir filas a un archivo existente

```python
with open("salida.csv", "a", newline="", encoding="utf-8") as f:
    csv.writer(f).writerow(["Pedro", 23, "Bilbao"])
```

---

## 5. Operaciones intermedias

### 5.1 Otros delimitadores

```python
csv.reader(f, delimiter=";")         # Punto y coma (Excel en español)
csv.reader(f, delimiter="\t")        # Tabulador (TSV)
csv.reader(f, delimiter="|")
```

### 5.2 Conversión de tipos

El módulo `csv` devuelve **siempre texto**. Convierta los valores explícitamente:

```python
with open("ventas.csv", newline="", encoding="utf-8") as f:
    ventas = [
        {
            "producto": fila["producto"],
            "unidades": int(fila["unidades"]),
            "precio": float(fila["precio"]),
        }
        for fila in csv.DictReader(f)
    ]

total = sum(v["unidades"] * v["precio"] for v in ventas)
```

### 5.3 Filtrar y transformar

```python
with open("clientes.csv", newline="", encoding="utf-8") as entrada, \
     open("clientes_madrid.csv", "w", newline="", encoding="utf-8") as salida:

    lector = csv.DictReader(entrada)
    escritor = csv.DictWriter(salida, fieldnames=lector.fieldnames)
    escritor.writeheader()

    for fila in lector:
        if fila["ciudad"].strip().lower() == "madrid":
            fila["email"] = fila["email"].lower()
            escritor.writerow(fila)
```

### 5.4 Agrupar y resumir

```python
from collections import defaultdict

totales = defaultdict(float)
with open("ventas.csv", newline="", encoding="utf-8") as f:
    for fila in csv.DictReader(f):
        totales[fila["region"]] += float(fila["importe"])

for region, total in sorted(totales.items(), key=lambda x: x[1], reverse=True):
    print(f"{region:<10} {total:>12,.2f} €")
```

### 5.5 Control del entrecomillado

```python
csv.writer(f, quoting=csv.QUOTE_MINIMAL)      # Solo cuando sea necesario (por defecto)
csv.writer(f, quoting=csv.QUOTE_ALL)          # Todos los campos
csv.writer(f, quoting=csv.QUOTE_NONNUMERIC)   # Todos salvo números
```

Los campos con comas, comillas o saltos de línea se entrecomillan automáticamente:

```python
csv.writer(f).writerow(['Calle Mayor, 5', 'Dijo "hola"'])
# "Calle Mayor, 5","Dijo ""hola"""
```

### 5.6 Campos ausentes o adicionales

```python
# Valores por defecto si faltan columnas
csv.DictReader(f, restval="")

# Ignorar claves extra al escribir
csv.DictWriter(f, fieldnames=campos, extrasaction="ignore")
```

---

## 6. Operaciones avanzadas

### 6.1 Detectar automáticamente el formato

```python
with open("desconocido.csv", newline="", encoding="utf-8") as f:
    muestra = f.read(2048)
    f.seek(0)
    dialecto = csv.Sniffer().sniff(muestra, delimiters=",;\t|")
    tiene_cabecera = csv.Sniffer().has_header(muestra)
    lector = csv.reader(f, dialecto)
    print("Delimitador detectado:", repr(dialecto.delimiter))
```

### 6.2 Registrar un dialecto propio

```python
csv.register_dialect("es", delimiter=";", quotechar='"', quoting=csv.QUOTE_MINIMAL)

with open("datos.csv", newline="", encoding="utf-8") as f:
    for fila in csv.reader(f, dialect="es"):
        ...
```

### 6.3 Procesar archivos muy grandes

El lector es un iterador: procesa fila a fila sin cargar todo en memoria.

```python
contador = 0
with open("enorme.csv", newline="", encoding="utf-8") as f:
    for fila in csv.DictReader(f):
        if float(fila["importe"]) > 1000:
            contador += 1
```

### 6.4 Escribir en memoria (por ejemplo, para una API)

```python
import io

buffer = io.StringIO()
escritor = csv.writer(buffer)
escritor.writerows([["a", 1], ["b", 2]])
contenido = buffer.getvalue()
```

### 6.5 Validar filas y registrar errores

```python
validas, errores = [], []
with open("importacion.csv", newline="", encoding="utf-8") as f:
    for num, fila in enumerate(csv.DictReader(f), start=2):   # Línea 1 = cabecera
        try:
            fila["edad"] = int(fila["edad"])
            if "@" not in fila["email"]:
                raise ValueError("email no válido")
            validas.append(fila)
        except (ValueError, KeyError) as e:
            errores.append((num, str(e)))

print(f"{len(validas)} válidas, {len(errores)} con errores")
```

### 6.6 Cuándo usar Pandas en lugar de `csv`

| Use `csv` si… | Use Pandas si… |
|---|---|
| Procesa fila a fila o archivos enormes. | Necesita filtrar, agrupar o calcular estadísticas. |
| No quiere dependencias externas. | Quiere tipos automáticos y operaciones vectorizadas. |
| Necesita control fino del formato. | Va a combinar varias tablas. |

```python
import pandas as pd
df = pd.read_csv("ventas.csv", sep=";", decimal=",", encoding="utf-8-sig")
```

---

## 7. CSV y Excel en español

Excel con configuración regional española:
- Usa **punto y coma** (`;`) como separador.
- Usa **coma** como separador decimal.
- Necesita **BOM** (`utf-8-sig`) para mostrar bien tildes y ñ.

```python
with open("para_excel.csv", "w", newline="", encoding="utf-8-sig") as f:
    escritor = csv.writer(f, delimiter=";")
    escritor.writerow(["Producto", "Precio"])
    escritor.writerow(["Señalizador", f"{12.5:.2f}".replace(".", ",")])
```

Para leer un CSV exportado desde Excel:

```python
with open("desde_excel.csv", newline="", encoding="utf-8-sig") as f:
    for fila in csv.DictReader(f, delimiter=";"):
        precio = float(fila["Precio"].replace(".", "").replace(",", "."))
```

---

## 8. Ejemplo práctico completo

Informe de ventas por región a partir de un CSV, con validación y exportación compatible con Excel:

```python
import csv
from collections import defaultdict
from pathlib import Path

ENTRADA = Path("ventas.csv")
SALIDA = Path("informe_regiones.csv")


def crear_datos_ejemplo():
    datos = [
        ["fecha", "region", "producto", "unidades", "precio"],
        ["2026-11-01", "Norte", "Teclado", "10", "29.90"],
        ["2026-11-02", "Sur", "Ratón", "25", "14.50"],
        ["2026-11-03", "Norte", "Monitor", "3", "189.00"],
        ["2026-11-04", "Este", "Teclado", "x", "29.90"],     # Fila con error
        ["2026-11-05", "Sur", "Monitor", "2", "189.00"],
    ]
    with ENTRADA.open("w", newline="", encoding="utf-8") as f:
        csv.writer(f).writerows(datos)


def procesar():
    resumen = defaultdict(lambda: {"unidades": 0, "importe": 0.0, "pedidos": 0})
    errores = []

    with ENTRADA.open(newline="", encoding="utf-8") as f:
        for linea, fila in enumerate(csv.DictReader(f), start=2):
            try:
                uds = int(fila["unidades"])
                precio = float(fila["precio"])
            except ValueError:
                errores.append(f"Línea {linea}: valores numéricos no válidos")
                continue
            r = resumen[fila["region"]]
            r["unidades"] += uds
            r["importe"] += uds * precio
            r["pedidos"] += 1

    with SALIDA.open("w", newline="", encoding="utf-8-sig") as f:
        escritor = csv.writer(f, delimiter=";")
        escritor.writerow(["Región", "Pedidos", "Unidades", "Importe (€)"])
        for region, r in sorted(resumen.items()):
            importe = f"{r['importe']:.2f}".replace(".", ",")
            escritor.writerow([region, r["pedidos"], r["unidades"], importe])

    print(f"Informe generado en {SALIDA}")
    for e in errores:
        print("AVISO:", e)


if __name__ == "__main__":
    crear_datos_ejemplo()
    procesar()
```

---

## 9. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| Líneas en blanco entre filas (Windows) | Falta `newline=""`. | Abra siempre con `newline=""`. |
| Tildes ilegibles en Excel | Falta BOM. | Escriba con `encoding="utf-8-sig"`. |
| Todo aparece en una sola columna en Excel | Delimitador incorrecto. | Use `delimiter=";"` para Excel en español. |
| `KeyError` en `DictReader` | Cabecera con espacios o BOM. | Use `utf-8-sig` y limpie: `fila = {k.strip(): v for k, v in fila.items()}`. |
| `ValueError` al convertir números | Valores vacíos o con coma decimal. | Valide y convierta (`replace(",", ".")`). |
| `UnicodeDecodeError` | Archivo en otra codificación (p. ej. `latin-1`). | Pruebe `encoding="latin-1"` o `cp1252`. |
| `_csv.Error: field larger than field limit` | Campos muy grandes. | `csv.field_size_limit(10_000_000)`. |

---

## 10. Buenas prácticas

- Use `DictReader` / `DictWriter` para un código más legible y resistente a cambios de orden de columnas.
- Abra siempre con `newline=""` y especifique `encoding`.
- Convierta y valide los tipos inmediatamente después de leer.
- No construya CSV concatenando cadenas con comas: use `csv.writer`, que gestiona comillas y caracteres especiales.
- Documente el delimitador y la codificación esperados en archivos compartidos.
- Para análisis complejos, pase a Pandas.

---

## 11. Referencia rápida

```python
csv.reader(f, delimiter=",", quotechar='"')
csv.writer(f, delimiter=",", quoting=csv.QUOTE_MINIMAL)
csv.DictReader(f, fieldnames=None, restval=None)
csv.DictWriter(f, fieldnames=[...], extrasaction="raise")
escritor.writeheader() / writerow(fila) / writerows(filas)
csv.Sniffer().sniff(muestra) / has_header(muestra)
csv.register_dialect("nombre", delimiter=";")
csv.QUOTE_MINIMAL | QUOTE_ALL | QUOTE_NONNUMERIC | QUOTE_NONE
```

---

## 12. Ejercicios propuestos

1. Lea un CSV de alumnos con sus notas y genere otro con la media de cada alumno y si aprueba o no.
2. Una dos CSV (clientes y pedidos) por un campo `id_cliente` sin usar Pandas.
3. Escriba un script que detecte automáticamente el delimitador de cualquier CSV y lo convierta al formato de Excel en español.
4. Elimine filas duplicadas de un CSV conservando el orden original.

---

## 13. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/csv.html
