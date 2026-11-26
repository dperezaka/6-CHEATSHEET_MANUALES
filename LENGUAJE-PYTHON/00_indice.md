# Manual de programación en Python

> **Categoría:** Programación · **Nivel:** De inicial a profesional
> **Requisitos:** Python 3.12 o superior

---

## Presentación

Este manual recoge, en seis capítulos independientes, los conocimientos necesarios para programar en Python de forma profesional: desde la sintaxis básica hasta la organización, prueba y mantenimiento de proyectos reales.

Todos los capítulos siguen la misma estructura: descripción, conceptos clave, explicación por secciones con ejemplos, **ejemplo práctico completo**, tabla de errores frecuentes, buenas prácticas, referencia rápida, ejercicios y referencias oficiales. Todos los ejemplos completos usan solo la librería estándar (salvo `pytest` en el capítulo 6) y se han ejecutado para comprobar que funcionan.

---

## Contenido

| N.º | Archivo | Contenido | Ejemplo completo |
|---|---|---|---|
| 1 | `01_fundamentos.md` | Instalación, variables, tipos, operadores, cadenas, entrada/salida, condicionales, `match` y bucles. | Calculadora de factura por consola. |
| 2 | `02_estructuras_de_datos.md` | Listas, tuplas, diccionarios, conjuntos, comprensiones, desempaquetado, ordenación, `collections` y copias. | Análisis de ventas trimestrales. |
| 3 | `03_funciones_y_modulos.md` | Parámetros, retorno, ámbito, anotaciones de tipo, `lambda`, decoradores, generadores, módulos y paquetes. | Calculadora de préstamos con `argparse`. |
| 4 | `04_programacion_orientada_a_objetos.md` | Clases, propiedades, métodos especiales, herencia, clases abstractas, composición, `dataclass`, `Enum` y `Protocol`. | Sistema de pedidos con estados y descuentos. |
| 5 | `05_archivos_y_excepciones.md` | Excepciones, excepciones propias, `pathlib`, texto, CSV, JSON, gestores de contexto y `logging`. | Importador de pedidos desde CSV con validación. |
| 6 | `06_entornos_pruebas_y_calidad.md` | Entornos virtuales, `pip`, `uv`, `pyproject.toml`, `pytest`, depuración, `ruff`, `mypy`, secretos, rendimiento y concurrencia. | Validadores de DNI, IBAN, email y CP con pruebas. |

**Capítulos complementarios:** `08_tkinter.md` (interfaces gráficas).

---

## Itinerario recomendado

1. **Iniciación:** capítulos 1 y 2. Realice todos los ejercicios antes de continuar.
2. **Nivel intermedio:** capítulos 3 y 5. A partir de aquí puede automatizar tareas reales.
3. **Nivel avanzado:** capítulos 4 y 6. Necesarios para aplicaciones mantenibles y trabajo en equipo.
4. **Especialización:** interfaces gráficas (Tkinter), análisis de datos (pandas), web (FastAPI, Django)…

---

## Convenciones del manual

| Símbolo | Significado |
|---|---|
| `> Nota` | Aclaración o recomendación importante. |
| ⚠ | Advertencia: error habitual o riesgo. |
| ✔ / ❌ | Forma correcta / incorrecta. |
| `…` | Código omitido por brevedad. |

---

## Referencias generales

- Documentación oficial en español: https://docs.python.org/es/3/
- Tutorial oficial: https://docs.python.org/es/3/tutorial/
- PEP 8 – Guía de estilo: https://peps.python.org/pep-0008/
- Índice de paquetes (PyPI): https://pypi.org/
