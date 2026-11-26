# Manual de uso: Datetime

> **Categoría:** Utilidades generales · **Utilidad:** Gestionar fechas y horas
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Formateo y conversión de texto](#5-formateo-y-conversión-de-texto)
6. [Aritmética de fechas](#6-aritmética-de-fechas)
7. [Zonas horarias](#7-zonas-horarias)
8. [Operaciones avanzadas](#8-operaciones-avanzadas)
9. [Ejemplo práctico completo](#9-ejemplo-práctico-completo)
10. [Errores frecuentes](#10-errores-frecuentes)
11. [Buenas prácticas](#11-buenas-prácticas)
12. [Referencia rápida](#12-referencia-rápida)
13. [Ejercicios propuestos](#13-ejercicios-propuestos)
14. [Referencias](#14-referencias)

---

## 1. Descripción

El módulo `datetime` permite crear, manipular, comparar y formatear fechas y horas. Junto con `zoneinfo` (zonas horarias) y `calendar`, cubre la mayoría de necesidades: calcular plazos, edades, diferencias de tiempo, convertir formatos de texto y trabajar con distintas zonas horarias.

---

## 2. Instalación e importación

No requiere instalación.

```python
from datetime import date, time, datetime, timedelta, timezone
from zoneinfo import ZoneInfo        # Python 3.9+
```

> En Windows, `zoneinfo` puede necesitar el paquete de datos: `pip install tzdata`.

---

## 3. Conceptos clave

| Clase | Representa | Ejemplo |
|---|---|---|
| `date` | Fecha (año, mes, día) | `2026-11-24` |
| `time` | Hora del día | `14:30:00` |
| `datetime` | Fecha y hora | `2026-11-24 14:30:00` |
| `timedelta` | Duración / diferencia | `3 días, 4:00:00` |
| `timezone` / `ZoneInfo` | Zona horaria | `Europe/Madrid` |

**Fechas *naive* y *aware*:**
- *Naive*: sin zona horaria. Ambiguas si el sistema trabaja con varias zonas.
- *Aware*: con zona horaria (`tzinfo`). Recomendadas en aplicaciones reales.

---

## 4. Operaciones básicas

### 4.1 Fecha y hora actuales

```python
hoy = date.today()                                   # 2026-11-24
ahora = datetime.now()                               # Hora local (naive)
ahora_madrid = datetime.now(ZoneInfo("Europe/Madrid"))   # Con zona horaria
ahora_utc = datetime.now(timezone.utc)
```

### 4.2 Crear fechas concretas

```python
d = date(2026, 12, 25)
t = time(9, 30)
dt = datetime(2026, 12, 25, 9, 30, 0)
combinada = datetime.combine(d, t)
```

### 4.3 Acceder a sus componentes

```python
dt.year, dt.month, dt.day
dt.hour, dt.minute, dt.second, dt.microsecond
dt.weekday()          # 0 = lunes … 6 = domingo
dt.isoweekday()       # 1 = lunes … 7 = domingo
dt.isocalendar()      # (año, semana ISO, día)
dt.date()             # Solo la fecha
dt.time()             # Solo la hora
```

### 4.4 Reemplazar componentes

```python
inicio_mes = dt.replace(day=1, hour=0, minute=0, second=0, microsecond=0)
```

### 4.5 Comparar fechas

```python
date(2026, 1, 1) < date(2026, 6, 1)          # True
max([date(2026, 3, 1), date(2025, 12, 1)])
sorted(lista_de_fechas)
```

---

## 5. Formateo y conversión de texto

### 5.1 De fecha a texto: `strftime`

```python
dt = datetime(2026, 11, 24, 18, 5)

dt.strftime("%d/%m/%Y")              # 24/11/2026
dt.strftime("%d/%m/%Y %H:%M")        # 24/11/2026 18:05
dt.strftime("%Y-%m-%d")              # 2026-11-24
dt.strftime("%I:%M %p")              # 06:05 PM
dt.isoformat()                       # 2026-11-24T18:05:00
```

### 5.2 De texto a fecha: `strptime`

```python
datetime.strptime("24/11/2026", "%d/%m/%Y")
datetime.strptime("2026-11-24 18:05", "%Y-%m-%d %H:%M")
datetime.fromisoformat("2026-11-24T18:05:00")     # Formato ISO
date.fromisoformat("2026-11-24")
```

### 5.3 Códigos de formato más usados

| Código | Significado | Ejemplo |
|---|---|---|
| `%d` | Día (2 dígitos) | `24` |
| `%m` | Mes (2 dígitos) | `11` |
| `%Y` | Año (4 dígitos) | `2026` |
| `%y` | Año (2 dígitos) | `26` |
| `%H` | Hora 24 h | `18` |
| `%I` | Hora 12 h | `06` |
| `%M` | Minutos | `05` |
| `%S` | Segundos | `00` |
| `%p` | AM/PM | `PM` |
| `%A` / `%a` | Día de la semana (largo / corto) | `Tuesday` / `Tue` |
| `%B` / `%b` | Mes (largo / corto) | `November` / `Nov` |
| `%j` | Día del año | `328` |
| `%U` / `%W` | Semana del año | `47` |
| `%z` | Desfase UTC | `+0100` |
| `%Z` | Nombre de zona | `CET` |

### 5.4 Nombres de días y meses en español

Los nombres dependen de la configuración regional. Una solución portable es usar listas propias:

```python
DIAS = ["lunes", "martes", "miércoles", "jueves", "viernes", "sábado", "domingo"]
MESES = ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio",
         "agosto", "septiembre", "octubre", "noviembre", "diciembre"]

def fecha_larga(d: date) -> str:
    return f"{DIAS[d.weekday()]}, {d.day} de {MESES[d.month - 1]} de {d.year}"

print(fecha_larga(date(2026, 11, 24)))   # martes, 24 de noviembre de 2026
```

Alternativa con `locale` (depende del sistema operativo):

```python
import locale
locale.setlocale(locale.LC_TIME, "es_ES.UTF-8")     # En Windows: "Spanish_Spain"
datetime.now().strftime("%A, %d de %B de %Y")
```

---

## 6. Aritmética de fechas

### 6.1 Sumar y restar tiempo con `timedelta`

```python
hoy = date.today()

hoy + timedelta(days=30)
hoy - timedelta(weeks=2)
datetime.now() + timedelta(hours=3, minutes=15)
```

### 6.2 Diferencia entre fechas

```python
inicio = datetime(2026, 11, 24, 9, 0)
fin = datetime(2026, 11, 26, 17, 30)

duracion = fin - inicio              # timedelta
duracion.days                        # 2
duracion.seconds                     # 30600 (resto, sin contar días)
duracion.total_seconds()             # 203400.0
duracion.total_seconds() / 3600      # 56.5 horas
```

### 6.3 Calcular la edad

```python
def edad(nacimiento: date, referencia: date | None = None) -> int:
    referencia = referencia or date.today()
    return referencia.year - nacimiento.year - (
        (referencia.month, referencia.day) < (nacimiento.month, nacimiento.day)
    )

edad(date(1990, 12, 31))
```

### 6.4 Sumar meses (no existe `timedelta(months=...)`)

```python
import calendar

def sumar_meses(d: date, meses: int) -> date:
    mes_total = d.month - 1 + meses
    anio = d.year + mes_total // 12
    mes = mes_total % 12 + 1
    dia = min(d.day, calendar.monthrange(anio, mes)[1])
    return date(anio, mes, dia)

sumar_meses(date(2026, 1, 31), 1)     # 2026-02-28
```

> Alternativa: `pip install python-dateutil` y `from dateutil.relativedelta import relativedelta`.

### 6.5 Primer y último día del mes

```python
import calendar

def limites_mes(anio: int, mes: int) -> tuple[date, date]:
    ultimo = calendar.monthrange(anio, mes)[1]
    return date(anio, mes, 1), date(anio, mes, ultimo)
```

### 6.6 Generar rangos de fechas

```python
def rango_fechas(inicio: date, fin: date, paso: int = 1):
    actual = inicio
    while actual <= fin:
        yield actual
        actual += timedelta(days=paso)

for d in rango_fechas(date(2026, 12, 1), date(2026, 12, 7)):
    print(d)
```

### 6.7 Días laborables entre dos fechas

```python
def dias_laborables(inicio: date, fin: date, festivos: set[date] = frozenset()) -> int:
    return sum(
        1 for d in rango_fechas(inicio, fin)
        if d.weekday() < 5 and d not in festivos
    )

festivos = {date(2026, 12, 8), date(2026, 12, 25)}
dias_laborables(date(2026, 12, 1), date(2026, 12, 31), festivos)
```

---

## 7. Zonas horarias

### 7.1 Crear fechas con zona horaria

```python
madrid = ZoneInfo("Europe/Madrid")
nueva_york = ZoneInfo("America/New_York")

reunion = datetime(2026, 11, 24, 16, 0, tzinfo=madrid)
```

### 7.2 Convertir entre zonas

```python
reunion.astimezone(nueva_york)          # 2026-11-24 10:00:00-05:00
reunion.astimezone(timezone.utc)        # 2026-11-24 15:00:00+00:00
```

### 7.3 Asignar zona a una fecha naive

```python
naive = datetime(2026, 11, 24, 16, 0)
aware = naive.replace(tzinfo=madrid)
```

### 7.4 Marcas de tiempo Unix (*timestamps*)

```python
ts = datetime.now(timezone.utc).timestamp()          # Segundos desde 1970-01-01 UTC
datetime.fromtimestamp(ts, tz=timezone.utc)
datetime.fromtimestamp(1764000000, tz=madrid)
```

### 7.5 Recomendación general

> Guarde y procese las fechas en **UTC** y conviértalas a la zona local solo al mostrarlas al usuario.

---

## 8. Operaciones avanzadas

### 8.1 Medir tiempos de ejecución

```python
import time

inicio = time.perf_counter()
# ... código a medir ...
print(f"Duración: {time.perf_counter() - inicio:.3f} s")
```

### 8.2 Formatear una duración legible

```python
def formatear_duracion(td: timedelta) -> str:
    total = int(td.total_seconds())
    dias, resto = divmod(total, 86400)
    horas, resto = divmod(resto, 3600)
    minutos, segundos = divmod(resto, 60)
    partes = []
    if dias: partes.append(f"{dias} d")
    if horas: partes.append(f"{horas} h")
    if minutos: partes.append(f"{minutos} min")
    if segundos or not partes: partes.append(f"{segundos} s")
    return " ".join(partes)

formatear_duracion(timedelta(days=1, hours=2, minutes=5))   # 1 d 2 h 5 min
```

### 8.3 Módulo `calendar`

```python
import calendar

calendar.isleap(2028)                 # Año bisiesto
calendar.monthrange(2026, 2)          # (día de la semana del día 1, días del mes)
print(calendar.month(2026, 12))       # Calendario en texto
```

### 8.4 Fechas en Pandas

```python
import pandas as pd

df["fecha"] = pd.to_datetime(df["fecha"], format="%d/%m/%Y")
df["mes"] = df["fecha"].dt.month
df["dia_semana"] = df["fecha"].dt.dayofweek
pd.date_range("2026-01-01", "2026-12-31", freq="MS")   # Primer día de cada mes
```

---

## 9. Ejemplo práctico completo

Calculadora de vencimientos de facturas con días laborables y recordatorios:

```python
from datetime import date, timedelta

FESTIVOS_2026 = {
    date(2026, 1, 1), date(2026, 1, 6), date(2026, 5, 1),
    date(2026, 8, 15), date(2026, 10, 12), date(2026, 11, 1),
    date(2026, 12, 6), date(2026, 12, 8), date(2026, 12, 25),
}


def es_laborable(d: date) -> bool:
    return d.weekday() < 5 and d not in FESTIVOS_2026


def sumar_dias_laborables(inicio: date, dias: int) -> date:
    actual = inicio
    while dias > 0:
        actual += timedelta(days=1)
        if es_laborable(actual):
            dias -= 1
    return actual


def estado_factura(vencimiento: date, hoy: date) -> str:
    diferencia = (vencimiento - hoy).days
    if diferencia < 0:
        return f"VENCIDA hace {-diferencia} días"
    if diferencia <= 5:
        return f"Vence en {diferencia} días ⚠"
    return f"Vence en {diferencia} días"


facturas = [
    ("F-2026-101", date(2026, 10, 20), 30),
    ("F-2026-118", date(2026, 11, 10), 15),
    ("F-2026-125", date(2026, 11, 20), 30),
]

hoy = date(2026, 11, 24)
print(f"{'Factura':<12}{'Emisión':<12}{'Vencimiento':<14}Estado")
for numero, emision, plazo in facturas:
    vence = sumar_dias_laborables(emision, plazo)
    print(f"{numero:<12}{emision:%d/%m/%Y}  {vence:%d/%m/%Y}    {estado_factura(vence, hoy)}")
```

---

## 10. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ValueError: time data ... does not match format` | El formato de `strptime` no coincide con el texto. | Revise separadores y códigos (`%d/%m/%Y` frente a `%Y-%m-%d`). |
| `TypeError: can't compare offset-naive and offset-aware datetimes` | Mezcla de fechas con y sin zona. | Use siempre fechas *aware* o siempre *naive*. |
| `TypeError: unsupported operand type(s) for -: 'datetime.datetime' and 'datetime.date'` | Operar `date` con `datetime`. | Convierta con `.date()` o `datetime.combine()`. |
| `ValueError: day is out of range for month` | Fecha imposible (31 de febrero). | Use `calendar.monthrange` para validar. |
| `ZoneInfoNotFoundError` | Falta la base de datos de zonas. | `pip install tzdata`. |
| Confundir `td.seconds` con el total | `seconds` excluye los días. | Use `total_seconds()`. |
| Nombres de mes en inglés | `strftime` usa la configuración regional. | Use listas propias o `locale`. |

---

## 11. Buenas prácticas

- Use fechas *aware* en cualquier aplicación que intercambie datos entre sistemas.
- Almacene en **UTC** y convierta solo para mostrar.
- Use el formato **ISO 8601** (`isoformat()`) para guardar e intercambiar fechas.
- No calcule meses o años sumando días fijos.
- Para mediciones de rendimiento, use `time.perf_counter()`, no `datetime.now()`.
- Centralice los formatos de fecha en constantes para mantener la coherencia.

---

## 12. Referencia rápida

```python
date.today()                         datetime.now(tz)
date(a, m, d)                        datetime(a, m, d, h, mi, s, tzinfo=tz)
dt.strftime("%d/%m/%Y")              datetime.strptime(txt, "%d/%m/%Y")
dt.isoformat()                       datetime.fromisoformat(txt)
timedelta(days, hours, minutes)      td.total_seconds()
dt.astimezone(ZoneInfo("..."))       dt.replace(tzinfo=...)
dt.timestamp()                       datetime.fromtimestamp(ts, tz)
dt.weekday()                         calendar.monthrange(a, m)
```

---

## 13. Ejercicios propuestos

1. Escriba una función que, dada una fecha de nacimiento, indique cuántos días faltan para el próximo cumpleaños.
2. Cree un generador de calendario de turnos rotativos de 4 personas para un mes completo, excluyendo fines de semana.
3. Convierta una lista de horas de reunión en Madrid a las zonas de Nueva York, Londres y Tokio.
4. Lea un CSV con columnas `entrada` y `salida` (fecha y hora) y calcule las horas trabajadas por empleado y semana.

---

## 14. Referencias

- `datetime`: https://docs.python.org/es/3/library/datetime.html
- `zoneinfo`: https://docs.python.org/es/3/library/zoneinfo.html
- `calendar`: https://docs.python.org/es/3/library/calendar.html
