# Manual de uso: Pint

> **Categoría:** Ciencia e ingeniería · **Utilidad:** Trabajar con magnitudes y unidades físicas
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** BSD

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [El registro de unidades](#4-el-registro-de-unidades)
5. [Crear cantidades](#5-crear-cantidades)
6. [Conversión de unidades](#6-conversión-de-unidades)
7. [Operaciones y dimensiones](#7-operaciones-y-dimensiones)
8. [Temperaturas y unidades con desplazamiento](#8-temperaturas-y-unidades-con-desplazamiento)
9. [Formato y presentación](#9-formato-y-presentación)
10. [Unidades personalizadas y monedas](#10-unidades-personalizadas-y-monedas)
11. [Validar unidades en funciones](#11-validar-unidades-en-funciones)
12. [Integración con NumPy, pandas y otras librerías](#12-integración-con-numpy-pandas-y-otras-librerías)
13. [Contextos y sistemas de unidades](#13-contextos-y-sistemas-de-unidades)
14. [Incertidumbres de medida](#14-incertidumbres-de-medida)
15. [Rendimiento y organización del código](#15-rendimiento-y-organización-del-código)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Pint** es una librería para definir, operar y convertir **magnitudes físicas**: valores numéricos acompañados de sus unidades. Comprueba automáticamente la coherencia dimensional de los cálculos y convierte entre cualquier par de unidades compatibles.

```python
distancia = 120 * ureg.km
tiempo = 90 * ureg.minute
(distancia / tiempo).to("m/s")       # 22.22 m/s
```

**Ventajas:**
- Evita errores de unidades (sumar metros y segundos, confundir kW y kWh) detectándolos en tiempo de ejecución.
- Más de 1000 unidades predefinidas: SI, anglosajonas, CGS, técnicas, atómicas…
- Conversión automática, incluidas unidades compuestas (`kWh/m²/día`, `€/kWh`, `L/100 km`).
- Interpreta cadenas de texto con unidades (`"3.5 m"`, `"25 degC"`), útil para leer formularios y archivos.
- Integración con NumPy, pandas (*pint-pandas*), xarray (*pint-xarray*) y Matplotlib.
- Permite definir unidades propias (monedas, unidades de negocio).

**Limitaciones:**
- Añade un coste de rendimiento en cálculos masivos con escalares.
- Las temperaturas con desplazamiento (°C, °F) requieren un tratamiento específico.
- No todas las librerías aceptan objetos `Quantity`: a veces hay que extraer la magnitud.
- Las cantidades de **registros distintos** no se pueden combinar.

| Si necesita… | Opción |
|---|---|
| Unidades físicas en cálculos de ingeniería y ciencia | **Pint** |
| Unidades en astronomía y física con arrays grandes | `astropy.units` |
| Unidades en expresiones simbólicas | `sympy.physics.units` |
| Solo convertir unos pocos valores | Factores de conversión manuales |

---

## 2. Instalación e importación

```bash
pip install pint
pip install pint[uncertainties]       # Con soporte de incertidumbres de medida
pip install pint-pandas               # Columnas con unidades en pandas
```

Comprobar la instalación:

```bash
python -c "import pint; print(pint.__version__)"
```

Importación habitual:

```python
import pint

ureg = pint.UnitRegistry()            # Registro de unidades (uno por aplicación)
Q_ = ureg.Quantity                    # Abreviatura para crear cantidades
```

> El nombre `ureg` (*unit registry*) y el alias `Q_` son convenciones de la documentación oficial y de la comunidad.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Registro (`UnitRegistry`) | Contiene la definición de todas las unidades y sus relaciones. |
| Cantidad (`Quantity`) | Magnitud + unidades: `3.5 meter`. |
| Magnitud | Parte numérica (`float`, `int`, array de NumPy…): `.magnitude` o `.m`. |
| Unidades | Parte de unidades: `.units` o `.u`. |
| Dimensión | Naturaleza física: `[length]`, `[mass]`, `[time]`, `[temperature]`… |
| Dimensionalidad | Combinación de dimensiones de una cantidad: velocidad = `[length] / [time]`. |
| Unidad base | Unidad de referencia de cada dimensión en el sistema activo (SI por defecto: m, kg, s, K, A…). |
| Unidad con desplazamiento | Unidad cuyo cero no coincide con el de la base (°C, °F). |
| Contexto | Conjunto de reglas para convertir entre dimensiones distintas (p. ej. longitud de onda ↔ frecuencia). |

---

## 4. El registro de unidades

### 4.1 Crear el registro

```python
ureg = pint.UnitRegistry()                              # Definiciones por defecto
ureg = pint.UnitRegistry(system="mks")                  # Sistema de unidades base
ureg = pint.UnitRegistry("mis_unidades.txt")            # Definiciones propias desde archivo
ureg = pint.UnitRegistry(autoconvert_offset_to_baseunit=True)   # Ver sección 8
```

> **Use un único registro en toda la aplicación.** Las cantidades de registros distintos no son compatibles entre sí y producen `ValueError: Cannot operate with Quantity and Quantity of different registries`.

### 4.2 Registro compartido en una aplicación

```python
# unidades.py
import pint

ureg = pint.UnitRegistry()
ureg.formatter.default_format = "~P"
Q_ = ureg.Quantity
pint.set_application_registry(ureg)       # Necesario para serializar (pickle) cantidades
```

```python
# Resto de módulos
from unidades import ureg, Q_
```

### 4.3 Consultar unidades

```python
ureg.km                                     # Unidad
ureg.Unit("kWh")                            # Unidad desde cadena
"furlong" in ureg                           # ¿Existe la unidad?
ureg.get_dimensionality("N")                # [mass] * [length] / [time] ** 2
ureg.get_compatible_units("[pressure]")     # Unidades de presión disponibles
ureg.km.is_compatible_with(ureg.mile)       # True
```

---

## 5. Crear cantidades

### 5.1 Formas de crear una cantidad

```python
d = 3.5 * ureg.meter                     # Multiplicando por una unidad
d = 3.5 * ureg.m                         # Con símbolo
d = Q_(3.5, "m")                         # Magnitud y unidades por separado
d = Q_(3.5, ureg.m)
d = Q_("3.5 m")                          # Desde cadena
v = Q_("80 km/h")
e = ureg("2 kW * 3 h").to("kWh")         # Expresión completa: 6.0 kWh
```

### 5.2 Nombres de unidades admitidos

Pint reconoce nombres completos, símbolos, prefijos y plurales:

| Escritura | Interpretación |
|---|---|
| `"meter"`, `"metre"`, `"m"`, `"meters"` | Metro |
| `"km"`, `"kilometer"` | Kilómetro |
| `"m**2"`, `"m^2"`, `"m²"` | Metro cuadrado |
| `"kWh"`, `"kilowatt_hour"`, `"kW*h"` | Kilovatio hora |
| `"km/h"`, `"kph"` | Kilómetro por hora |
| `"degC"`, `"celsius"`, `"°C"` | Grado Celsius |
| `"%"`, `"percent"` | Porcentaje |
| `"h"`, `"hour"`, `"min"`, `"s"`, `"day"`, `"year"` | Tiempo |

> Los nombres de unidades están en **inglés** (`day`, `year`, `hour`). Para interpretar textos en español, sustituya las palabras antes de crear la cantidad o defina alias (sección 10).

### 5.3 Acceder a las partes

```python
v = Q_(80, "km/h")
v.magnitude          # 80   (o v.m)
v.units              # kilometer / hour   (o v.u)
v.dimensionality     # [length] / [time]
v.m_as("m/s")        # 22.22  → magnitud ya convertida (sin unidades)
```

---

## 6. Conversión de unidades

### 6.1 Convertir

```python
v = Q_(80, "km/h")
v.to("m/s")                  # 22.22 meter / second   → devuelve una nueva cantidad
v.to(ureg.mph)               # 49.71 mile_per_hour

d = Q_(5, "m")
d.ito("cm")                  # Convierte en el sitio (modifica d): 500.0 centimeter
```

### 6.2 Conversiones automáticas útiles

```python
Q_(3, "km").to_base_units()            # 3000 meter  (unidades base del SI)
Q_(1500, "m").to_compact()             # 1.5 kilometer  (prefijo más legible)
Q_(1_500_000, "W").to_compact()        # 1.5 megawatt
(Q_(2, "m") / Q_(50, "cm")).to_reduced_units()   # 4.0 (simplifica unidades)
Q_(1, "N*m").to("J")                   # 1.0 joule
```

### 6.3 Unidades compuestas

```python
Q_(1, "km/L").to("mi/gal")             # 2.35 mile / gallon
Q_(4.9, "kWh/m**2/day").to("MJ/m**2/year")
Q_(0.145, "EUR/kWh")                   # Requiere definir la moneda (sección 10)
```

### 6.4 Magnitudes adimensionales y porcentajes

```python
ratio = Q_(2, "m") / Q_(50, "cm")      # 0.04 meter / centimeter
ratio.to("")                           # 4.0 dimensionless
ratio.to("percent")                    # 400.0 percent
(Q_(60, "%") * Q_(185, "MWh")).to("MWh")   # 111.0 megawatt_hour (sin .to: 11100 MWh·%)
float(ratio.to(""))                    # 4.0 → número de Python
```

---

## 7. Operaciones y dimensiones

### 7.1 Aritmética con unidades

```python
distancia = Q_(120, "km")
tiempo = Q_(1.5, "h")
velocidad = distancia / tiempo                     # 80.0 kilometer / hour
area = Q_(3, "m") * Q_(4, "m")                     # 12 meter ** 2
energia = Q_(2, "kW") * Q_(3, "h")                 # 6 kilowatt * hour
total = Q_(3.5, "m") + Q_(20, "cm")                # 3.7 meter (convierte la segunda)
Q_(9, "m**2") ** 0.5                               # 3.0 meter
```

- **Suma y resta:** solo entre dimensiones iguales; el resultado toma las unidades del primer operando.
- **Producto y división:** combinan las unidades sin convertir; use `.to()` o `.to_reduced_units()` para simplificar.

### 7.2 Errores dimensionales

```python
Q_(1, "m") + Q_(1, "s")
# DimensionalityError: Cannot convert from 'meter' ([length]) to 'second' ([time])

Q_(10, "kWh").to("kW")
# DimensionalityError: energía y potencia no son convertibles (falta el tiempo)
```

> Esta es la principal utilidad de Pint: un `DimensionalityError` indica casi siempre un **error real** en la fórmula o en los datos.

### 7.3 Comparaciones

```python
Q_(1, "km") > Q_(900, "m")                         # True
Q_(1, "km") == Q_(1000, "m")                       # True
Q_(1, "km") > Q_(1, "s")                           # DimensionalityError
(5 * ureg.m).check("[length]")                     # True: comprobación de dimensión
```

### 7.4 Funciones matemáticas

```python
import numpy as np

np.sqrt(Q_(16, "m**2"))                 # 4.0 meter
np.sin(Q_(30, "degree"))                # ≈ 0.5 dimensionless (convierte a radianes)
np.exp(Q_(2, "m"))                      # DimensionalityError: el exponente debe ser adimensional
round(Q_(2.3456, "m"), 2)               # 2.35 meter
```

> Las funciones del módulo `math` **no** entienden de unidades: `math.sqrt(Q_(16, "m**2"))` descarta las unidades o falla. Use las funciones de NumPy.

---

## 8. Temperaturas y unidades con desplazamiento

### 8.1 El problema

Las escalas Celsius y Fahrenheit tienen el cero desplazado respecto al kelvin. Por eso, sumar dos temperaturas es **ambiguo**:

```python
Q_(10, "degC") + Q_(5, "degC")
# OffsetUnitCalculusError: Ambiguous operation with offset unit
```

¿Significa 15 °C, o 10 °C más un incremento de 5 °C? Pint obliga a especificarlo.

### 8.2 Temperaturas absolutas e incrementos

| Unidad | Significado |
|---|---|
| `degC`, `degF` | Temperatura **absoluta** (un punto en la escala) |
| `delta_degC`, `delta_degF` | **Diferencia** de temperatura |
| `kelvin`, `degR` | Escalas absolutas (sin desplazamiento): se operan sin restricciones |

```python
T = Q_(25, "degC")
T.to("degF")                                  # 77.0 degree_Fahrenheit
T.to("K")                                     # 298.15 kelvin

Q_(10, "degC") + Q_(5, "delta_degC")          # 15 degree_Celsius
Q_(30, "degC") - Q_(20, "degC")               # 10 delta_degree_Celsius
Q_(10, "delta_degC").to("delta_degF")         # 18.0 delta_degree_Fahrenheit
```

### 8.3 Temperaturas en fórmulas

En fórmulas físicas (calor, potencia térmica), use siempre una **diferencia** o una escala absoluta:

```python
masa = Q_(200, "kg")
calor_especifico = Q_(4.18, "kJ/(kg*delta_degC)")
salto = Q_(60, "degC") - Q_(15, "degC")        # delta_degC
energia = (masa * calor_especifico * salto).to("kWh")     # 10.45 kWh
```

> Con `UnitRegistry(autoconvert_offset_to_baseunit=True)`, Pint convierte automáticamente a kelvin en multiplicaciones. Es cómodo, pero puede ocultar errores: úselo solo si entiende sus implicaciones.

---

## 9. Formato y presentación

### 9.1 Especificadores de formato

| Especificador | Resultado para `80 km/h` | Uso |
|---|---|---|
| (por defecto) | `80.0 kilometer / hour` | Depuración |
| `~` | `80.0 km / h` | Símbolos abreviados |
| `P` / `~P` | `80.0 kilometer/hour` / `80.0 km/h` | Texto legible (*pretty*), con `²` y `·` |
| `~C` | `80.0 km/h` | Compacto |
| `~L` | `80\ \frac{\mathrm{km}}{\mathrm{h}}` | LaTeX |
| `~H` | HTML | Informes web y Jupyter |
| `~D` | `80.0 km / h` | Formato por defecto con símbolos |

```python
v = Q_(80, "km/h")
f"{v:~P}"                     # '80.0 km/h'
f"{v:.2f~P}"                  # '80.00 km/h'   → formato numérico + formato de unidades
f"{Q_(3.6, 'MJ').to('kWh'):~P}"    # '1.0 kWh'
f"{v.magnitude:,.2f} {v.units:~P}"  # Control total por separado
```

### 9.2 Formato por defecto del registro

```python
ureg.formatter.default_format = "~P"       # Pint 0.24+
print(Q_(12, "m**2"))                       # 12 m²
```

> En versiones anteriores a la 0.24 se usaba `ureg.default_format = "~P"`.

### 9.3 Alinear cantidades en tablas

El ancho de campo no se aplica directamente a una cantidad. Convierta primero a texto:

```python
texto = f"{valor:.1f~P}"
print(f"{'Potencia':<20}{texto:>15}")
```

---

## 10. Unidades personalizadas y monedas

### 10.1 Definir unidades

```python
ureg.define("palet = 1.2 * m**2 = pal")                 # Unidad derivada con símbolo
ureg.define("turno = 8 * hour")
ureg.define("caja = [embalaje]")                        # Nueva dimensión base
ureg.define("palet_cajas = 40 * caja")
ureg.define("@alias hour = hora")                       # Alias de una unidad existente
```

Formato general: `nombre = definición = símbolo = alias1 = alias2`.

### 10.2 Monedas

Pint no incluye monedas: se definen como una dimensión propia.

```python
ureg.define("euro = [moneda] = € = EUR")
ureg.define("centimo = 0.01 euro")

precio = Q_(0.15, "€/kWh")
consumo = Q_(300, "kWh")
(precio * consumo)                         # 45.0 €
```

> Para convertir entre monedas con tipos de cambio variables, defina cada moneda como dimensión distinta y aplique el tipo de cambio de forma explícita: los tipos de cambio no son constantes físicas.

### 10.3 Definiciones en un archivo

```text
# mis_unidades.txt
@import default_en.txt          # Mantener las definiciones estándar
euro = [moneda] = € = EUR
palet = 1.2 * m**2 = pal
turno = 8 * hour
```

```python
ureg = pint.UnitRegistry("mis_unidades.txt")
# o bien: ureg.load_definitions("mis_unidades.txt")
```

---

## 11. Validar unidades en funciones

### 11.1 `@ureg.check`: comprobar dimensiones

```python
@ureg.check("[length]", "[time]")
def velocidad_media(distancia, tiempo):
    return distancia / tiempo

velocidad_media(Q_(10, "km"), Q_(30, "min"))       # ✓
velocidad_media(Q_(10, "kg"), Q_(30, "min"))       # ✗ DimensionalityError
```

Use `None` o `""` para argumentos sin comprobación.

### 11.2 `@ureg.wraps`: convertir a la entrada y a la salida

Útil para envolver funciones que trabajan con números «desnudos» en unidades fijas (librerías antiguas, código numérico optimizado):

```python
@ureg.wraps(ureg.W, (ureg.V, ureg.A))      # Salida en W; entradas convertidas a V y A
def potencia(tension, corriente):
    return tension * corriente             # Aquí son floats

potencia(Q_(230, "V"), Q_(2000, "mA"))     # 460.0 watt
```

> `wraps` es muy útil para combinar Pint con Numba, SciPy u otras librerías que no admiten cantidades: la función interna recibe floats y el exterior mantiene la seguridad de unidades.

### 11.3 Validación manual

```python
def validar(q, dimension: str) -> None:
    if not isinstance(q, ureg.Quantity) or not q.check(dimension):
        raise ValueError(f"Se esperaba una cantidad de dimensión {dimension}, se recibió {q!r}")
```

---

## 12. Integración con NumPy, pandas y otras librerías

### 12.1 NumPy

```python
distancias = Q_(np.array([1.2, 3.5, 0.8]), "km")
# o: np.array([1.2, 3.5, 0.8]) * ureg.km

distancias.to("m")             # [1200. 3500. 800.] meter
distancias.mean()              # 1.833 kilometer
distancias.max()
np.sum(distancias)
distancias[distancias > Q_(1, "km")]
distancias.magnitude           # Array de NumPy sin unidades
```

### 12.2 pandas (pint-pandas)

```python
import pandas as pd
import pint_pandas

df = pd.DataFrame({
    "distancia": pd.Series([12.0, 25.5, 7.2], dtype="pint[km]"),
    "tiempo": pd.Series([15, 30, 10], dtype="pint[min]"),
})
df["velocidad"] = (df["distancia"] / df["tiempo"]).pint.to("km/h")
df.pint.dequantify()            # Columnas con unidades en la cabecera (para exportar)
```

### 12.3 Matplotlib

```python
ureg.setup_matplotlib()
plt.plot(tiempos, distancias)             # Los ejes muestran las unidades
plt.gca().yaxis.set_units(ureg.mile)      # Cambiar las unidades del eje
```

### 12.4 Serialización

```python
# A texto (JSON, CSV, base de datos)
registro = {"valor": q.magnitude, "unidad": f"{q.units:~}"}
q = Q_(registro["valor"], registro["unidad"])

# Con pickle: requiere pint.set_application_registry(ureg)
```

> Guarde siempre la magnitud junto con su unidad. Un número suelto en una base de datos pierde su significado.

---

## 13. Contextos y sistemas de unidades

### 13.1 Contextos

Permiten conversiones entre dimensiones diferentes cuando existe una relación física:

```python
with ureg.context("sp"):                  # Espectroscopia: longitud de onda ↔ frecuencia ↔ energía
    Q_(500, "nm").to("THz")               # 599.58 terahertz

Q_(500, "nm").to("THz", "sp")             # Forma abreviada
```

Contextos incluidos: `sp` (espectroscopia), `boltzmann`, `chemistry` (requiere masa molar), `energy`, entre otros.

### 13.2 Sistemas de unidades

```python
ureg.default_system = "imperial"          # Unidades base anglosajonas
Q_(1, "km").to_base_units()               # En yardas
ureg.default_system = "mks"               # Volver al SI
```

---

## 14. Incertidumbres de medida

Requiere `pip install pint[uncertainties]`:

```python
longitud = Q_(10.0, "m").plus_minus(0.1)       # (10.00 +/- 0.10) meter
area = longitud * longitud                     # (100.0 +/- 2.0) meter ** 2
longitud.plus_minus(0.01, relative=True)       # Incertidumbre relativa
```

La incertidumbre se propaga automáticamente en las operaciones.

---

## 15. Rendimiento y organización del código

### 15.1 Coste de Pint

Cada operación con cantidades comprueba y combina unidades: en bucles con millones de escalares, el coste es apreciable.

| Situación | Recomendación |
|---|---|
| Muchos valores con las mismas unidades | Use **arrays de NumPy** dentro de una única cantidad. |
| Bucle numérico intensivo | Convierta a unidades fijas, extraiga `.magnitude`, calcule y vuelva a añadir las unidades (o use `@ureg.wraps`). |
| Cálculo de configuración, informes, validaciones | Use cantidades directamente. |

```python
# Lento
total = sum(Q_(x, "kWh") for x in lecturas)

# Rápido
total = Q_(np.sum(lecturas), "kWh")
```

### 15.2 Arquitectura recomendada

```
Entrada (texto, formularios, CSV) ─► Cantidades Pint (validación de unidades)
        ─► Núcleo de cálculo (cantidades o floats en unidades fijas vía @wraps)
        ─► Salida (formato con unidades, o magnitud + unidad por separado)
```

Use las unidades en las **fronteras** del sistema (entrada, salida, interfaces entre módulos), donde se producen la mayoría de errores.

---

## 16. Ejemplo práctico completo

**Dimensionado de una instalación fotovoltaica de autoconsumo** para una nave industrial: a partir de datos de entrada en texto con unidades (consumo, irradiación, potencia y superficie de los paneles, precios), calcula la potencia pico necesaria, el número de paneles limitado por la cubierta, la producción, la cobertura, la inversión, el ahorro, el periodo de retorno y la sección de cable por caída de tensión. Define el euro como unidad, valida dimensiones con `@ureg.check`, usa porcentajes y unidades compuestas, y exporta un informe con magnitud y unidad por separado.

Requisitos: `pip install pint`

```python
"""
Dimensionado de una instalación fotovoltaica de autoconsumo con Pint.

Calcula la potencia de paneles necesaria a partir del consumo de una nave
industrial, la superficie de cubierta, la producción, el ahorro económico,
la sección de cable y el periodo de retorno, controlando las unidades en
todos los pasos.
"""
from pathlib import Path

import pint

ureg = pint.UnitRegistry()
ureg.define("euro = [moneda] = € = EUR")
ureg.formatter.default_format = "~P"             # Símbolos abreviados: kWh, m², €
Q_ = ureg.Quantity

# ------------------------------------------------------------------ Datos de entrada
# Pueden venir de un formulario o un archivo: las cadenas se interpretan con sus unidades
ENTRADA = {
    "consumo_anual": "185 MWh/year",            # Energía POR AÑO (no solo MWh)
    "cobertura_objetivo": "60 %",
    "irradiacion_diaria": "4.9 kWh/m²/día",       # Media anual en el plano de los paneles
    "potencia_panel": "550 W",
    "superficie_panel": "2.58 m²",
    "rendimiento_sistema": "0.80",                # Pérdidas: inversor, cableado, temperatura
    "superficie_cubierta": "1400 m²",
    "factor_ocupacion": "0.55",                   # Pasillos, sombras, separaciones
    "coste_instalacion": "0.95 €/W",
    "precio_energia": "0.145 €/kWh",
    "tension_cadena": "800 V",
    "distancia_cable": "65 m",
    "caida_tension_max": "1.5 %",
}

RESISTIVIDAD_COBRE = Q_(0.0225, "ohm * mm² / m")   # A 70 °C
SECCIONES_NORMALIZADAS = [Q_(s, "mm²") for s in (4, 6, 10, 16, 25, 35, 50, 70, 95)]


def leer(datos: dict[str, str]) -> dict[str, pint.Quantity]:
    return {clave: Q_(valor.replace("día", "day")) for clave, valor in datos.items()}


# ------------------------------------------------------------------ Cálculos
@ureg.check("[energy] / [time]", "[energy] / [length] ** 2 / [time]", "")
def potencia_pico_necesaria(energia_anual, irradiacion, rendimiento):
    # Si energia_anual viniera en MWh (sin «/year»), @check lanzaría DimensionalityError
    """Potencia pico: energía / (horas solares pico × rendimiento)."""
    horas_sol_pico = (irradiacion / Q_(1, "kW/m²")).to("h/year")   # Irradiancia estándar: 1 kW/m²
    return (energia_anual / (horas_sol_pico * rendimiento)).to("kW")


def seccion_cable(potencia, tension, longitud, caida_max):
    """Sección mínima en corriente continua (ida y vuelta) por caída de tensión."""
    corriente = (potencia / tension).to("A")
    seccion = (2 * longitud * corriente * RESISTIVIDAD_COBRE / (caida_max * tension)).to("mm²")
    normalizada = next((s for s in SECCIONES_NORMALIZADAS if s >= seccion), None)
    return corriente, seccion, normalizada


def main() -> None:
    d = leer(ENTRADA)

    energia_objetivo = (d["consumo_anual"] * d["cobertura_objetivo"]).to("MWh/year")
    p_necesaria = potencia_pico_necesaria(energia_objetivo, d["irradiacion_diaria"],
                                          d["rendimiento_sistema"])

    n_paneles = int((p_necesaria / d["potencia_panel"]).to("").magnitude) + 1
    superficie_util = d["superficie_cubierta"] * d["factor_ocupacion"]
    max_paneles = int((superficie_util / d["superficie_panel"]).to("").magnitude)
    limitado = n_paneles > max_paneles
    n_paneles = min(n_paneles, max_paneles)

    p_instalada = (n_paneles * d["potencia_panel"]).to("kW")
    produccion = (p_instalada * d["irradiacion_diaria"] / Q_(1, "kW/m²")
                  * d["rendimiento_sistema"]).to("MWh/year")
    cobertura = (produccion / d["consumo_anual"]).to("percent")

    inversion = (p_instalada * d["coste_instalacion"]).to("€")
    ahorro = (produccion * d["precio_energia"]).to("€/year")
    retorno = (inversion / ahorro).to("year")

    corriente, seccion, normalizada = seccion_cable(
        p_instalada / 4, d["tension_cadena"], d["distancia_cable"], d["caida_tension_max"])  # 4 inversores

    # ------------------------------------------------------------------ Informe
    filas = [
        ("Energía objetivo", energia_objetivo, ".1f"),
        ("Potencia pico necesaria", p_necesaria, ".1f"),
        ("Superficie ocupada", (n_paneles * d["superficie_panel"]).to("m²"), ".0f"),
        ("Potencia instalada", p_instalada, ".2f"),
        ("Producción anual", produccion, ".1f"),
        ("Cobertura real", cobertura, ".1f"),
        ("Inversión", inversion, ",.0f"),
        ("Ahorro anual", ahorro, ",.0f"),
        ("Periodo de retorno", retorno, ".1f"),
        ("Corriente por inversor", corriente, ".1f"),
        ("Sección mínima de cable", seccion, ".2f"),
    ]
    print("DIMENSIONADO DE INSTALACIÓN FOTOVOLTAICA\n" + "-" * 48)
    for nombre, valor, fmt in filas:
        texto = f"{valor:{fmt}~P}"                # Formato numérico + unidades abreviadas
        print(f"{nombre:<28}{texto:>20}")
    print(f"{'Sección normalizada':<28}{str(normalizada) if normalizada else 'Revisar diseño':>20}")
    print(f"{'Paneles instalados':<28}{n_paneles:>20}")
    if limitado:
        print("\n⚠ La cubierta limita el número de paneles: no se alcanza la cobertura objetivo.")

    # Equivalencias útiles
    print(f"\nProducción diaria media: {produccion.to('kWh/day'):.0f~P}")
    print(f"Energía anual en julios: {(produccion * Q_(1, 'year')).to('GJ'):.0f~P}")

    Path("informe_fv.txt").write_text(
        "\n".join(f"{n};{v.magnitude:.4f};{v.units:~P}" for n, v, _ in filas), encoding="utf-8")


if __name__ == "__main__":
    main()
```

Resultado:

```
DIMENSIONADO DE INSTALACIÓN FOTOVOLTAICA
------------------------------------------------
Energía objetivo                     111.0 MWh/a
Potencia pico necesaria                  77.5 kW
Superficie ocupada                        364 m²
Potencia instalada                      77.55 kW
Producción anual                     111.0 MWh/a
Cobertura real                            60.0 %
Inversión                               73,672 €
Ahorro anual                          16,100 €/a
Periodo de retorno                         4.6 a
Corriente por inversor                    24.2 A
Sección mínima de cable                 5.91 mm²
Sección normalizada                        6 mm²
Paneles instalados                           141

Producción diaria media: 304 kWh/d
Energía anual en julios: 400 GJ
```

> **Pint detecta errores reales.** En una primera versión de este ejemplo, el consumo se indicó como `"185 MWh"` en lugar de `"185 MWh/year"`. Al convertir el resultado a kW, Pint lanzó `DimensionalityError: Cannot convert from 'megawatt_hour * year / hour' to 'kilowatt'`, revelando que faltaba la referencia temporal. Sin control de unidades, el cálculo habría producido un número incorrecto sin ningún aviso. El símbolo `a` (*annum*) es la abreviatura estándar de año.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `DimensionalityError: Cannot convert from 'X' to 'Y'` | Dimensiones incompatibles: suma de magnitudes distintas o conversión imposible. | Revise la fórmula: suele faltar o sobrar un factor (tiempo, superficie…). |
| `OffsetUnitCalculusError: Ambiguous operation with offset unit` | Suma o multiplicación con `degC`/`degF`. | Use `delta_degC` para incrementos o convierta a kelvin. |
| `UndefinedUnitError: 'xxx' is not defined in the unit registry` | Unidad inexistente o escrita en español (`"día"`, `"hora"`). | Use el nombre en inglés o defina un alias. |
| `ValueError: Cannot operate with Quantity and Quantity of different registries` | Se crearon varios `UnitRegistry`. | Use un único registro compartido. |
| Las unidades no se simplifican (`meter / centimeter`) | Pint no convierte automáticamente tras multiplicar o dividir. | Use `.to(...)`, `.to_reduced_units()` o `.to("")`. |
| `TypeError` al pasar una cantidad a otra librería | La librería no admite `Quantity`. | Pase `q.m_as("unidad")` o use `@ureg.wraps`. |
| `math.sqrt(q)` pierde las unidades | `math` no conoce Pint. | Use `np.sqrt(q)` o `q ** 0.5`. |
| El formato `f"{q:>20.1f}"` no alinea | El ancho no se aplica a la cantidad. | Formatee a texto primero: `f"{q:.1f~P}"` y alinee el texto. |
| `RuntimeError: Pint requires the 'uncertainties' package` | Se usó `plus_minus` sin el paquete opcional. | `pip install pint[uncertainties]`. |
| `AttributeError: 'UnitRegistry' object has no attribute 'default_format'` (aviso) | API antigua de formato. | Use `ureg.formatter.default_format` (Pint 0.24+). |
| Resultados extraños con porcentajes | Se mezclan `percent` y números normales. | Convierta con `.to("")` o `.to("percent")` explícitamente. |
| Error al deserializar con `pickle` | No se registró el registro de la aplicación. | `pint.set_application_registry(ureg)`. |

---

## 18. Buenas prácticas

- Cree **un único registro** (`ureg`) en un módulo propio e impórtelo desde el resto de la aplicación.
- Asigne unidades **lo antes posible** (al leer los datos) y retírelas **lo más tarde posible** (al presentar o exportar).
- Indique siempre la referencia temporal en las magnitudes periódicas: `MWh/year`, `€/month`, `L/day`.
- Use `delta_degC` para incrementos de temperatura y kelvin en fórmulas termodinámicas.
- Valide las funciones públicas con `@ureg.check` y envuelva el código numérico intensivo con `@ureg.wraps`.
- Use **arrays de NumPy** dentro de una cantidad en lugar de listas de cantidades.
- Defina monedas y unidades de negocio en un archivo de definiciones versionado.
- Almacene y transmita los datos como **magnitud + unidad** (nunca números sueltos).
- Configure `ureg.formatter.default_format = "~P"` para informes legibles.
- Escriba pruebas con valores conocidos, comparando con tolerancia: `math.isclose(Q_(1, "kWh").m_as("MJ"), 3.6)` (con `==` fallaría por redondeo).

---

## 19. Referencia rápida

```python
import pint
ureg = pint.UnitRegistry(); Q_ = ureg.Quantity
ureg.formatter.default_format = "~P"

q = 3.5 * ureg.m  |  Q_(3.5, "m")  |  Q_("3.5 m")  |  ureg("2 kW * 3 h")
q.magnitude / q.m · q.units / q.u · q.dimensionality · q.m_as("cm")
q.to("cm") · q.ito("cm") · q.to_base_units() · q.to_compact() · q.to_reduced_units() · q.to("")
q.check("[length]") · ureg.km.is_compatible_with(ureg.mile) · ureg.get_compatible_units("[energy]")

Q_(25, "degC").to("K") · Q_(10, "degC") + Q_(5, "delta_degC")
f"{q:.2f~P}" · f"{q:~L}" · f"{q.units:~P}"

ureg.define("euro = [moneda] = € = EUR") · ureg.load_definitions("archivo.txt")
@ureg.check("[length]", "[time]") · @ureg.wraps(ureg.W, (ureg.V, ureg.A))
with ureg.context("sp"): ... · Q_(10, "m").plus_minus(0.1)
pint.set_application_registry(ureg) · ureg.setup_matplotlib()

pint.DimensionalityError · pint.OffsetUnitCalculusError · pint.UndefinedUnitError
```

---

## 20. Ejercicios propuestos

1. Cree un conversor de recetas que escale cantidades (g, kg, mL, L, cucharadas, tazas) según el número de comensales y muestre el resultado en unidades métricas.
2. Calcule el consumo y el coste de un viaje en coche a partir de la distancia en millas, el consumo en L/100 km y el precio en €/L, y compárelo con un vehículo eléctrico en kWh/100 km.
3. Lea un CSV de lecturas de sensores con una columna de unidad por fila (`°C`, `°F`, `K`), normalice todas las temperaturas a °C y calcule las variaciones con `delta_degC`.
4. Escriba una función validada con `@ureg.check` que calcule la carga térmica para calentar el agua de un depósito y la potencia necesaria para hacerlo en un tiempo dado.
5. Amplíe el ejemplo completo para comparar tres modelos de panel (potencia, superficie y precio diferentes) y elegir el que minimiza el periodo de retorno.

---

## 21. Referencias

- Documentación oficial: https://pint.readthedocs.io/
- Tutorial: https://pint.readthedocs.io/en/stable/getting/tutorial.html
- Temperaturas y unidades con desplazamiento: https://pint.readthedocs.io/en/stable/user/nonmult.html
- pint-pandas: https://github.com/hgrecco/pint-pandas
- Repositorio: https://github.com/hgrecco/pint
