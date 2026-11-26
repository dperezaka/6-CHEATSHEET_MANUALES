El ecosistema de Python para crear herramientas de línea de comandos (CLI) se divide según la responsabilidad de cada librería: **parsear argumentos** (flags y subcomandos), **interactuar con el usuario** (menús y preguntas), **dar formato visual** (colores, tablas, barras de progreso) o **construir aplicaciones completas en terminal (TUI)**.

---

### 1. Parseo de argumentos y subcomandos (la base del CLI)

Son las encargadas de leer `mi_app comando --flag valor` y generar automáticamente la ayuda (`--help`).

* **`Typer`:** La opción moderna más popular. Usa *type hints* de Python para definir argumentos y opciones con muy poco código. Se integra de forma nativa con `Rich`.
* **`Click`:** El estándar de facto de la industria (es la base sobre la que está construido `Typer`). Muy robusto, componible y ideal para CLIs complejos con múltiples niveles de comandos.
* **`argparse`:** Viene incluido en la librería estándar de Python. No requiere dependencias externas, aunque es más verboso que `Typer` o `Click`.
* **`Fire` (Google):** Convierte automáticamente cualquier función, clase o diccionario de Python en un CLI sin necesidad de escribir configuración extra.

---

### 2. Interacción y menús (alternativas y complementos a `questionary`)

Para pedir datos de forma guiada mediante flechas, selecciones múltiples o autocompletado.

* **`InquirerPy`:** Un *fork* moderno y muy activo de `PyInquirer` (también basado en `prompt_toolkit`). Tiene sintaxis estilo `questionary` y estilo diccionario, con excelente soporte para búsqueda difusa (*fuzzy search*).
* **`Rich` (`Prompt`, `Confirm`, `IntPrompt`):** Aunque `Rich` destaca por lo visual, incluye prompts sencillos para pedir texto, números o confirmaciones con estilos y validación básica.
* **`prompt_toolkit`:** La librería de bajo nivel sobre la que están construidos `questionary` e `InquirerPy`. Úsala si necesitas construir un REPL propio, historial de comandos complejo o atajos de teclado personalizados.

---

### 3. Salida visual, formato y progreso

Para que la salida en terminal sea legible, estructurada y atractiva.

* **`Rich`:** La referencia absoluta para formato en terminal. Permite renderizar tablas, sintaxis coloreada, Markdown, paneles, árboles de directorios, tracebacks legibles y barras de progreso animadas.
* **`tqdm`:** Ligera, rápida y centrada exclusivamente en añadir barras de progreso a cualquier bucle (`for x in tqdm(lista):`) con mínimo impacto en rendimiento.
* **`colorama`:** Utilidad ligera para asegurar que los códigos de color ANSI funcionen de forma uniforme en Windows, macOS y Linux.
* **`yaspin` / `halo`:** Especializadas en *spinners* animados mientras se ejecutan tareas en segundo plano (descargas, peticiones HTTP, compilaciones).

---

### 4. Interfaces de usuario en terminal completas (TUI)

Cuando un flujo lineal de preguntas no basta y necesitas una aplicación a pantalla completa dentro de la terminal (con paneles, pestañas, ratón y actualización en tiempo real).

* **`Textual`:** Creada por los mismos autores de `Rich`. Permite construir aplicaciones TUI modernas usando un sistema inspirado en CSS, widgets reactivos y soporte completo de ratón.
* **`urwid`:** Librería clásica y veterana para interfaces de consola en Linux/Unix.

---

### Combinaciones habituales según el proyecto

| Necesidad | Combinación recomendada |
| --- | --- |
| **CLI moderno, rápido y visual** | `Typer` + `Rich` + `questionary` |
| **Script rápido sin dependencias externas** | `argparse` (librería estándar) |
| **CLI empresarial muy extensible (plugins)** | `Click` + `Rich` |
| **Dashboard o app interactiva a pantalla completa** | `Textual` |


| Librería              | Uso                                                                                                                                                                                                                           |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`questionary`**     | Crear prompts interactivos en CLI (texto, contraseñas, confirmaciones, menús con flechas, selección múltiple, autocompletado, rutas y validación), con soporte para formularios (`prompt`), saltos condicionales y `asyncio`. |
| **`Typer`**           | Parsear argumentos, opciones y subcomandos de CLI usando *type hints* de Python con ayuda automática e integración nativa con `Rich`.                                                                                         |
| **`Click`**           | Parsear argumentos y construir CLIs robustos, componibles y extensibles con múltiples niveles de comandos.                                                                                                                    |
| **`argparse`**        | Parsear argumentos de línea de comandos desde la librería estándar de Python sin dependencias externas.                                                                                                                       |
| **`Fire`**            | Convertir automáticamente funciones, clases o diccionarios de Python en interfaces de línea de comandos.                                                                                                                      |
| **`InquirerPy`**      | Crear menús y preguntas interactivas en terminal (basado en `prompt_toolkit`), con soporte para búsqueda difusa (*fuzzy search*).                                                                                             |
| **`prompt_toolkit`**  | Construir interfaces interactivas de bajo nivel en terminal, como REPL propios, historial de comandos o atajos de teclado personalizados.                                                                                     |
| **`Rich`**            | Formatear salida en terminal (tablas, paneles, sintaxis coloreada, Markdown, árboles, *tracebacks*, barras de progreso) y pedir prompts básicos (`Prompt`, `Confirm`, `IntPrompt`).                                           |
| **`tqdm`**            | Añadir barras de progreso rápidas y ligeras a bucles o iterables con bajo impacto en rendimiento.                                                                                                                             |
| **`colorama`**        | Asegurar compatibilidad uniforme de códigos de color ANSI en consola entre Windows, macOS y Linux.                                                                                                                            |
| **`yaspin` / `halo**` | Mostrar *spinners* animados en terminal durante tareas en segundo plano o esperas.                                                                                                                                            |
| **`Textual`**         | Desarrollar aplicaciones TUI completas a pantalla completa en terminal con diseño tipo CSS, widgets reactivos y soporte de ratón.                                                                                             |
| **`urwid`**           | Crear interfaces de consola clásicas en entornos Linux/Unix.                                                                                                                                                                  |



