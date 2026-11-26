# Manual de uso: JSON

> **Categoría:** Utilidades generales · **Utilidad:** Leer y escribir datos JSON
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Equivalencia de tipos](#4-equivalencia-de-tipos)
5. [Operaciones básicas](#5-operaciones-básicas)
6. [Operaciones intermedias](#6-operaciones-intermedias)
7. [Operaciones avanzadas](#7-operaciones-avanzadas)
8. [Ejemplo práctico completo](#8-ejemplo-práctico-completo)
9. [Errores frecuentes](#9-errores-frecuentes)
10. [Buenas prácticas](#10-buenas-prácticas)
11. [Referencia rápida](#11-referencia-rápida)
12. [Ejercicios propuestos](#12-ejercicios-propuestos)
13. [Referencias](#13-referencias)

---

## 1. Descripción

**JSON** (*JavaScript Object Notation*) es el formato de intercambio de datos más utilizado en la web: APIs REST, archivos de configuración, almacenamiento ligero y comunicación entre servicios.

El módulo `json` de Python permite:
- **Serializar**: convertir objetos de Python (diccionarios, listas…) en texto JSON.
- **Deserializar**: convertir texto JSON en objetos de Python.

---

## 2. Instalación e importación

No requiere instalación: forma parte de la librería estándar.

```python
import json
```

---

## 3. Conceptos clave

Las cuatro funciones principales se distinguen por la letra **s** (*string*):

| Función | Origen | Destino |
|---|---|---|
| `json.dumps(obj)` | Objeto Python | Cadena JSON |
| `json.loads(texto)` | Cadena JSON | Objeto Python |
| `json.dump(obj, archivo)` | Objeto Python | Archivo `.json` |
| `json.load(archivo)` | Archivo `.json` | Objeto Python |

> Regla mnemotécnica: con **s** trabaja con *strings*; sin **s**, con archivos.

---

## 4. Equivalencia de tipos

| Python | JSON |
|---|---|
| `dict` | objeto `{}` |
| `list`, `tuple` | array `[]` |
| `str` | string `""` |
| `int`, `float` | number |
| `True` / `False` | `true` / `false` |
| `None` | `null` |

**Limitaciones:**
- Las claves de un objeto JSON siempre son cadenas (`{1: "a"}` se convierte en `{"1": "a"}`).
- Las tuplas se convierten en listas y no se recuperan como tuplas.
- `datetime`, `Decimal`, `set` o clases propias **no** son serializables por defecto (ver sección 7).

---

## 5. Operaciones básicas

### 5.1 Convertir un objeto Python en texto JSON

```python
cliente = {
    "id": 101,
    "nombre": "Ana López",
    "activo": True,
    "saldo": 1250.75,
    "etiquetas": ["vip", "newsletter"],
    "direccion": None,
}

texto = json.dumps(cliente)
print(texto)
# {"id": 101, "nombre": "Ana López", "activo": true, ...}
```

### 5.2 Formato legible y caracteres en español

```python
texto = json.dumps(cliente, indent=4, ensure_ascii=False)
print(texto)
```

- `indent=4` añade sangría y saltos de línea.
- `ensure_ascii=False` conserva tildes, ñ y otros caracteres sin escaparlos.

### 5.3 Convertir texto JSON en objeto Python

```python
texto = '{"producto": "Teclado", "precio": 29.9, "stock": 15}'
datos = json.loads(texto)

print(datos["producto"])     # Teclado
print(type(datos))           # <class 'dict'>
```

### 5.4 Guardar en un archivo

```python
with open("cliente.json", "w", encoding="utf-8") as f:
    json.dump(cliente, f, indent=4, ensure_ascii=False)
```

### 5.5 Leer desde un archivo

```python
with open("cliente.json", "r", encoding="utf-8") as f:
    cliente = json.load(f)

print(cliente["nombre"])
```

---

## 6. Operaciones intermedias

### 6.1 Ordenar claves y compactar

```python
json.dumps(cliente, sort_keys=True)                     # Claves ordenadas alfabéticamente
json.dumps(cliente, separators=(",", ":"))              # Formato compacto (menor tamaño)
```

### 6.2 Acceso seguro a datos anidados

```python
config = json.loads('{"bd": {"host": "localhost", "puerto": 5432}}')

host = config.get("bd", {}).get("host", "127.0.0.1")
usuario = config.get("bd", {}).get("usuario", "admin")  # Valor por defecto si no existe
```

### 6.3 Recorrer estructuras complejas

```python
pedidos = json.loads("""
[
  {"id": 1, "cliente": "Ana", "lineas": [{"art": "A", "uds": 2}, {"art": "B", "uds": 1}]},
  {"id": 2, "cliente": "Luis", "lineas": [{"art": "C", "uds": 5}]}
]
""")

for pedido in pedidos:
    total = sum(l["uds"] for l in pedido["lineas"])
    print(f"Pedido {pedido['id']} ({pedido['cliente']}): {total} unidades")
```

### 6.4 Modificar y volver a guardar

```python
from pathlib import Path

ruta = Path("config.json")
config = json.loads(ruta.read_text(encoding="utf-8"))

config["version"] = "2.0"
config.setdefault("opciones", {})["modo_oscuro"] = True

ruta.write_text(json.dumps(config, indent=2, ensure_ascii=False), encoding="utf-8")
```

### 6.5 JSON Lines (un objeto por línea)

Formato habitual en logs y grandes volúmenes de datos (`.jsonl`):

```python
registros = [{"evento": "login", "usuario": "ana"}, {"evento": "logout", "usuario": "ana"}]

# Escribir
with open("eventos.jsonl", "w", encoding="utf-8") as f:
    for r in registros:
        f.write(json.dumps(r, ensure_ascii=False) + "\n")

# Leer
with open("eventos.jsonl", encoding="utf-8") as f:
    eventos = [json.loads(linea) for linea in f if linea.strip()]
```

### 6.6 Integración con Pandas

```python
import pandas as pd

df = pd.read_json("datos.json")                      # JSON tabular → DataFrame
df = pd.json_normalize(pedidos, record_path="lineas", meta=["id", "cliente"])  # Aplanar anidados
df.to_json("salida.json", orient="records", force_ascii=False, indent=2)
```

---

## 7. Operaciones avanzadas

### 7.1 Serializar tipos no compatibles con `default`

```python
from datetime import datetime, date
from decimal import Decimal

def convertir(obj):
    if isinstance(obj, (datetime, date)):
        return obj.isoformat()
    if isinstance(obj, Decimal):
        return float(obj)
    if isinstance(obj, set):
        return sorted(obj)
    raise TypeError(f"Tipo no serializable: {type(obj).__name__}")

factura = {"fecha": datetime.now(), "importe": Decimal("199.95"), "tags": {"b", "a"}}
print(json.dumps(factura, default=convertir, indent=2))
```

### 7.2 Codificador personalizado (`JSONEncoder`)

```python
class CodificadorEmpresa(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, (datetime, date)):
            return obj.isoformat()
        if isinstance(obj, Decimal):
            return str(obj)
        return super().default(obj)

json.dumps(factura, cls=CodificadorEmpresa)
```

### 7.3 Serializar clases propias

```python
from dataclasses import dataclass, asdict

@dataclass
class Producto:
    codigo: str
    nombre: str
    precio: float

p = Producto("A01", "Teclado", 29.9)
texto = json.dumps(asdict(p), ensure_ascii=False)

# Y de vuelta:
p2 = Producto(**json.loads(texto))
```

### 7.4 Personalizar la lectura con `object_hook`

```python
def a_fecha(d):
    for clave, valor in d.items():
        if clave.startswith("fecha") and isinstance(valor, str):
            try:
                d[clave] = datetime.fromisoformat(valor)
            except ValueError:
                pass
    return d

datos = json.loads('{"fecha_alta": "2026-11-24T10:30:00"}', object_hook=a_fecha)
print(type(datos["fecha_alta"]))   # <class 'datetime.datetime'>
```

### 7.5 Precisión decimal al leer

```python
datos = json.loads('{"importe": 0.1}', parse_float=Decimal)
print(datos["importe"])   # Decimal('0.1')
```

### 7.6 Validar JSON desde la terminal

```bash
python -m json.tool datos.json                 # Valida y formatea
python -m json.tool --sort-keys datos.json
```

---

## 8. Ejemplo práctico completo

Gestor de configuración con valores por defecto, validación y guardado seguro:

```python
import json
from pathlib import Path

CONFIG_POR_DEFECTO = {
    "aplicacion": {"nombre": "MiApp", "idioma": "es"},
    "base_datos": {"host": "localhost", "puerto": 5432},
    "registro": {"nivel": "INFO", "archivo": "app.log"},
}


def fusionar(base: dict, nuevos: dict) -> dict:
    """Combina diccionarios anidados respetando los valores por defecto."""
    resultado = base.copy()
    for clave, valor in nuevos.items():
        if isinstance(valor, dict) and isinstance(resultado.get(clave), dict):
            resultado[clave] = fusionar(resultado[clave], valor)
        else:
            resultado[clave] = valor
    return resultado


def cargar_config(ruta: Path) -> dict:
    if not ruta.exists():
        guardar_config(ruta, CONFIG_POR_DEFECTO)
        return CONFIG_POR_DEFECTO.copy()
    try:
        usuario = json.loads(ruta.read_text(encoding="utf-8"))
    except json.JSONDecodeError as e:
        raise ValueError(f"Configuración corrupta en línea {e.lineno}, columna {e.colno}") from e
    return fusionar(CONFIG_POR_DEFECTO, usuario)


def guardar_config(ruta: Path, config: dict) -> None:
    temporal = ruta.with_suffix(".tmp")
    temporal.write_text(json.dumps(config, indent=2, ensure_ascii=False), encoding="utf-8")
    temporal.replace(ruta)          # Escritura atómica: evita archivos a medio escribir


if __name__ == "__main__":
    ruta = Path("config.json")
    config = cargar_config(ruta)
    print("Puerto de la BD:", config["base_datos"]["puerto"])

    config["registro"]["nivel"] = "DEBUG"
    guardar_config(ruta, config)
```

---

## 9. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `JSONDecodeError: Expecting property name enclosed in double quotes` | Comillas simples o coma final. | JSON solo admite comillas dobles y no permite comas finales. |
| `JSONDecodeError: Expecting value: line 1 column 1` | Texto vacío o no es JSON (p. ej. HTML de error). | Compruebe el contenido antes de decodificar. |
| `TypeError: Object of type datetime is not JSON serializable` | Tipo no compatible. | Use `default=` o un `JSONEncoder` propio. |
| Caracteres como `ñ` en el archivo | `ensure_ascii=True` por defecto. | Use `ensure_ascii=False`. |
| `UnicodeDecodeError` al leer | Codificación incorrecta. | Abra siempre con `encoding="utf-8"`. |
| Las claves numéricas se vuelven texto | Limitación del formato JSON. | Convierta las claves al leer: `{int(k): v for k, v in d.items()}`. |

Gestión de errores recomendada:

```python
try:
    datos = json.loads(texto)
except json.JSONDecodeError as e:
    print(f"JSON no válido: {e.msg} (línea {e.lineno}, columna {e.colno})")
```

---

## 10. Buenas prácticas

- Abra siempre los archivos con `encoding="utf-8"`.
- Use `indent` y `ensure_ascii=False` en archivos que vayan a leer personas.
- Use formato compacto en comunicaciones entre servicios.
- Valide la estructura de los datos recibidos (por ejemplo, con **Pydantic**).
- No use JSON para guardar contraseñas o datos sensibles sin cifrar.
- Para grandes volúmenes, prefiera JSON Lines y procese línea a línea.
- Use escritura atómica (archivo temporal + `replace`) para archivos críticos.

---

## 11. Referencia rápida

```python
json.dumps(obj, indent=2, ensure_ascii=False, sort_keys=True, default=func)
json.loads(texto, object_hook=func, parse_float=Decimal)
json.dump(obj, f, indent=2, ensure_ascii=False)
json.load(f)
json.JSONDecodeError       # Excepción de lectura
json.JSONEncoder           # Clase base para codificadores propios
```

---

## 12. Ejercicios propuestos

1. Cree un archivo `agenda.json` con una lista de contactos (nombre, teléfono, email) y escriba funciones para añadir, buscar y eliminar contactos.
2. Lea un JSON con pedidos anidados y genere un resumen con el importe total por cliente.
3. Serialice una lista de objetos `dataclass` que contengan fechas y recupérelos con sus tipos originales.
4. Convierta un archivo CSV en JSON y viceversa.

---

## 13. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/json.html
- Especificación JSON: https://www.json.org/json-es.html
