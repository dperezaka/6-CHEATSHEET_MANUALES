# Manual de uso: spaCy

> **Categoría:** Inteligencia artificial · **Utilidad:** Procesamiento avanzado del lenguaje natural
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** MIT · **Desarrollo:** Explosion

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación, importación y modelos](#2-instalación-importación-y-modelos)
3. [Conceptos clave](#3-conceptos-clave)
4. [Procesar textos: `Doc`, `Token` y `Span`](#4-procesar-textos-doc-token-y-span)
5. [Atributos lingüísticos: lemas, categorías y morfología](#5-atributos-lingüísticos-lemas-categorías-y-morfología)
6. [Análisis sintáctico de dependencias](#6-análisis-sintáctico-de-dependencias)
7. [Entidades nombradas](#7-entidades-nombradas)
8. [Reglas: `Matcher`, `PhraseMatcher` y `EntityRuler`](#8-reglas-matcher-phrasematcher-y-entityruler)
9. [La canalización de procesamiento (*pipeline*)](#9-la-canalización-de-procesamiento-pipeline)
10. [Componentes y atributos propios](#10-componentes-y-atributos-propios)
11. [Vectores y similitud](#11-vectores-y-similitud)
12. [Procesar grandes volúmenes](#12-procesar-grandes-volúmenes)
13. [Entrenar modelos propios](#13-entrenar-modelos-propios)
14. [Visualización con displaCy](#14-visualización-con-displacy)
15. [Guardar y reutilizar: `DocBin` y modelos](#15-guardar-y-reutilizar-docbin-y-modelos)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**spaCy** es una librería de **procesamiento del lenguaje natural** diseñada para producción: rápida, robusta y con modelos entrenados para más de 20 idiomas, **incluido el español**. Con una sola llamada analiza un texto y devuelve tokens, frases, lemas, categorías gramaticales, morfología, dependencias sintácticas y entidades nombradas, y permite combinar estos modelos estadísticos con **reglas** y componentes propios.

**Ventajas:**
- Modelos preentrenados para español de distintos tamaños, listos para usar.
- Muy rápida y eficiente en memoria; procesamiento por lotes y en paralelo.
- Canalización (*pipeline*) modular y ampliable con reglas y componentes propios.
- Sistema de entrenamiento reproducible basado en archivos de configuración.
- Integración con Transformers (`spacy-transformers`) y con LLM (`spacy-llm`).
- Herramientas de visualización (displaCy) y anotación (Prodigy).

**Limitaciones:**
- Los modelos generales cometen errores en textos de dominios muy específicos: suele ser necesario completarlos con reglas o reentrenarlos.
- Las entidades de los modelos en español se limitan a PER, LOC, ORG y MISC (sin fechas ni importes, que se añaden con reglas).
- No incluye modelos generativos ni análisis de sentimiento de serie.

| Si necesita… | Opción |
|---|---|
| Análisis lingüístico en producción (lemas, sintaxis, entidades) en español | **spaCy** |
| Estadísticas de corpus, recursos léxicos, enseñanza | NLTK |
| Clasificación, sentimiento y generación con modelos de lenguaje | Hugging Face Transformers |
| Extracción con reglas sobre texto sin análisis lingüístico | Expresiones regulares (`re`) |

---

## 2. Instalación, importación y modelos

```bash
pip install spacy
python -m spacy download es_core_news_sm      # Modelo pequeño en español
```

### 2.1 Modelos para español

| Modelo | Tamaño | Vectores | Uso |
|---|---|---|---|
| `es_core_news_sm` | ~15 MB | No | Rápido; prototipos y volúmenes grandes |
| `es_core_news_md` | ~40 MB | Sí (reducidos) | Equilibrio precisión / velocidad; similitud |
| `es_core_news_lg` | ~550 MB | Sí (completos) | Mayor precisión y vectores |
| `es_dep_news_trf` | ~400 MB | — | Basado en *transformers*: máxima precisión sintáctica (requiere `spacy-transformers`, mejor con GPU) |

```python
import spacy
nlp = spacy.load("es_core_news_md")
print(nlp.pipe_names)
# ['tok2vec', 'morphologizer', 'parser', 'attribute_ruler', 'lemmatizer', 'ner']
```

```bash
python -m spacy info                   # Versión y modelos instalados
python -m spacy validate               # Comprueba la compatibilidad de los modelos con la versión de spaCy
```

> La versión del modelo debe ser compatible con la de spaCy (mismo número *mayor.menor*, p. ej. spaCy 3.8 con modelos 3.8). Tras actualizar spaCy, vuelva a descargar los modelos.

Para incluir un modelo en `requirements.txt`, use la URL de su paquete en GitHub (`https://github.com/explosion/spacy-models/releases/...`) o instálelo como dependencia del proyecto.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `nlp` (`Language`) | Objeto que contiene el vocabulario y la canalización de procesamiento. |
| `Doc` | Texto procesado: secuencia de tokens con todas sus anotaciones. |
| `Token` | Palabra, signo o símbolo con sus atributos (lema, categoría, dependencia…). |
| `Span` | Fragmento de un `Doc` (una entidad, una frase, un sintagma). |
| `Vocab` / `StringStore` | Vocabulario compartido; las cadenas se almacenan como identificadores (*hash*). |
| Canalización (*pipeline*) | Secuencia de componentes que anotan el `Doc` (etiquetador, analizador, NER…). |
| Componente | Función o clase que recibe un `Doc` y lo devuelve anotado. |
| Atributo propio | Extensión de `Doc`, `Token` o `Span` accesible con `._.nombre`. |
| `DocBin` | Formato binario eficiente para guardar muchos `Doc`. |

**Flujo:**

```
Texto ─► Tokenizador ─► tok2vec ─► morphologizer ─► parser ─► lemmatizer ─► ner ─► (componentes propios) ─► Doc
```

---

## 4. Procesar textos: `Doc`, `Token` y `Span`

```python
doc = nlp("Marta Gómez reclamó el pedido PED-48213 en Zaragoza. Solicita el reembolso.")

for token in doc:
    print(token.i, token.text, token.lemma_, token.pos_, token.dep_, token.head.text)

doc[0]                  # Token
doc[0:2]                # Span «Marta Gómez»
doc.text
len(doc)                # Número de tokens
list(doc.sents)         # Frases (según el analizador sintáctico)
doc.ents                # Entidades
doc.noun_chunks         # Sintagmas nominales
```

> Los atributos terminados en guion bajo devuelven texto (`token.pos_` → `"NOUN"`); sin guion bajo, el identificador numérico interno (`token.pos`).

### 4.1 Atributos léxicos de un token

| Atributo | Ejemplo |
|---|---|
| `text`, `lower_`, `shape_` | `"PED-48213"`, `"ped-48213"`, `"XXX-dddd"` |
| `is_alpha`, `is_digit`, `is_punct`, `is_space` | Tipo de carácter |
| `like_num`, `like_email`, `like_url` | Parece un número, correo o URL |
| `is_stop` | Palabra vacía |
| `idx` | Posición en caracteres dentro del texto |
| `whitespace_` | Espacio que sigue al token |

---

## 5. Atributos lingüísticos: lemas, categorías y morfología

```python
for token in nlp("Los técnicos instalaron las antenas nuevas"):
    print(f"{token.text:<12}{token.lemma_:<12}{token.pos_:<7}{token.morph}")
# Los         el          DET    Definite=Def|Gender=Masc|Number=Plur|PronType=Art
# técnicos    técnico     NOUN   Gender=Masc|Number=Plur
# instalaron  instalar    VERB   Mood=Ind|Number=Plur|Person=3|Tense=Past|VerbForm=Fin
# ...
token.morph.get("Number")      # ['Plur']
spacy.explain("PROPN")         # 'proper noun'
```

| Etiqueta (`pos_`, Universal Dependencies) | Categoría |
|---|---|
| `NOUN` / `PROPN` | Nombre común / propio |
| `VERB` / `AUX` | Verbo / auxiliar |
| `ADJ` / `ADV` | Adjetivo / adverbio |
| `DET`, `PRON`, `ADP` | Determinante, pronombre, preposición |
| `NUM`, `PUNCT`, `SYM` | Número, puntuación, símbolo |

**Lematización** para normalizar textos (agrupar «cobró», «cobraron», «cobrar»):

```python
lemas = [t.lemma_.lower() for t in doc if t.is_alpha and not t.is_stop]
```

---

## 6. Análisis sintáctico de dependencias

Cada token tiene un **núcleo** (`head`) y una **relación** (`dep_`) con él, formando un árbol:

```python
doc = nlp("El cliente solicita el reembolso del pedido")
for token in doc:
    print(f"{token.text:<10}{token.dep_:<8}→ {token.head.text}")
# El        det     → cliente
# cliente   nsubj   → solicita
# solicita  ROOT    → solicita
# el        det     → reembolso
# reembolso obj     → solicita
# ...
```

| Relación | Significado |
|---|---|
| `ROOT` | Núcleo de la oración |
| `nsubj` | Sujeto |
| `obj` / `iobj` | Objeto directo / indirecto |
| `obl` | Complemento circunstancial |
| `amod` / `nmod` | Modificador adjetival / nominal |
| `xcomp` / `ccomp` | Complemento oracional |
| `det`, `case`, `punct` | Determinante, preposición, puntuación |

Navegación por el árbol:

```python
token.children; token.lefts; token.rights; token.ancestors; token.subtree
token.left_edge, token.right_edge          # Límites del sintagma que encabeza el token
doc[token.left_edge.i : token.right_edge.i + 1]    # Sintagma completo
```

**Aplicación:** extraer «quién hace qué» (sujeto–verbo–objeto) o la acción solicitada en un correo (sección 16).

---

## 7. Entidades nombradas

```python
doc = nlp("Ana Torres, de Cafés del Sur, trabaja en Sevilla.")
[(ent.text, ent.label_, ent.start_char, ent.end_char) for ent in doc.ents]
```

| Etiqueta (modelos en español) | Significado |
|---|---|
| `PER` | Persona |
| `LOC` | Lugar |
| `ORG` | Organización |
| `MISC` | Otras (eventos, obras, nacionalidades…) |

> Los modelos en español no reconocen fechas, importes, cantidades ni identificadores. Añádalos con reglas (`EntityRuler`). Los modelos pequeños también confunden a veces palabras en mayúscula al inicio de frase («Exijo», «Buenos días») con entidades: revise los resultados y corríjalos con reglas o con un componente de filtrado.

Modificar entidades:

```python
from spacy.tokens import Span
doc.ents = list(doc.ents) + [Span(doc, 3, 5, label="PRODUCTO")]
doc.set_ents([Span(doc, 0, 2, label="PER")], default="unmodified")
```

---

## 8. Reglas: `Matcher`, `PhraseMatcher` y `EntityRuler`

### 8.1 `Matcher`: patrones sobre atributos de los tokens

```python
from spacy.matcher import Matcher

matcher = Matcher(nlp.vocab)
matcher.add("NEGACION_VERBO", [[{"LOWER": "no"}, {"POS": "VERB"}]])
matcher.add("IMPORTE", [[{"LIKE_NUM": True}, {"LOWER": {"IN": ["euros", "€"]}}]])
matcher.add("PEDIDO", [[{"TEXT": {"REGEX": r"^PED-\d{5}$"}}]])

for id_patron, inicio, fin in matcher(doc):
    print(nlp.vocab.strings[id_patron], doc[inicio:fin].text)
```

| Clave del patrón | Ejemplo |
|---|---|
| `TEXT`, `LOWER`, `LEMMA` | `{"LEMMA": "cobrar"}` |
| `POS`, `TAG`, `DEP`, `MORPH` | `{"POS": "PROPN"}` |
| `IS_DIGIT`, `LIKE_NUM`, `IS_PUNCT`, `SHAPE` | `{"SHAPE": "dddd"}` (cuatro dígitos) |
| `ENT_TYPE` | `{"ENT_TYPE": "LOC"}` |
| `IN`, `NOT_IN`, `REGEX`, `>=`… | `{"LOWER": {"IN": ["euro", "euros"]}}` |
| `OP` | `"?"` opcional, `"+"` uno o más, `"*"` cero o más, `"!"` negación |

### 8.2 `PhraseMatcher`: listas de términos

```python
from spacy.matcher import PhraseMatcher

terminos = PhraseMatcher(nlp.vocab, attr="LOWER")           # Ignora mayúsculas
terminos.add("PRODUCTO", [nlp.make_doc(t) for t in ["fibra óptica", "router", "tarjeta SIM"]])
```

Muy eficiente con miles de términos (catálogos de productos, nombres de clientes, glosarios).

### 8.3 `EntityRuler`: reglas que generan entidades

```python
regla = nlp.add_pipe("entity_ruler", before="ner", config={"overwrite_ents": True})
regla.add_patterns([
    {"label": "ORG", "pattern": "Cafés del Sur"},
    {"label": "FECHA", "pattern": [{"IS_DIGIT": True}, {"LOWER": "de"},
                                   {"LOWER": {"IN": ["enero", "febrero", "marzo"]}}]},
])
regla.to_disk("patrones.jsonl")          # Guardar los patrones
```

> Colocado **antes** de `ner`, el modelo estadístico respeta las entidades de las reglas; **después**, las reglas completan lo que el modelo no detectó.

### 8.4 Otros componentes basados en reglas

`span_ruler` (fragmentos que pueden solaparse, en `doc.spans`), `attribute_ruler` (corrige atributos como POS o lemas) y `sentencizer` (frases por puntuación, sin analizador).

---

## 9. La canalización de procesamiento (*pipeline*)

```python
nlp.pipe_names
nlp.add_pipe("sentencizer", first=True)                 # Añadir (first, last, before, after)
nlp.remove_pipe("sentencizer")
nlp = spacy.load("es_core_news_md", disable=["parser"])          # Cargado pero desactivado
nlp = spacy.load("es_core_news_md", exclude=["ner"])             # No se carga
with nlp.select_pipes(enable=["tok2vec", "ner"]):                # Desactivar temporalmente
    doc = nlp(texto)
nlp_vacio = spacy.blank("es")                           # Solo tokenizador (reglas del idioma)
```

> Desactive los componentes que no necesite: el analizador sintáctico es el más costoso. Si solo necesita entidades, procese con `tok2vec` y `ner`.

---

## 10. Componentes y atributos propios

### 10.1 Atributos propios (extensiones)

```python
from spacy.tokens import Doc, Token, Span

Doc.set_extension("urgente", default=False)
Token.set_extension("es_producto", getter=lambda t: t.lower_ in catalogo)     # Calculado al acceder
Span.set_extension("normalizado", getter=lambda s: s.text.upper())

doc._.urgente = True
doc[3]._.es_producto
```

### 10.2 Componente propio

```python
from spacy.language import Language

@Language.component("detector_urgencia")
def detector_urgencia(doc):
    doc._.urgente = any(t.lemma_ in {"exigir", "denuncia", "urgente"} for t in doc)
    return doc

nlp.add_pipe("detector_urgencia", last=True)
```

Para componentes con configuración o estado, use `@Language.factory`:

```python
@Language.factory("filtro_entidades", default_config={"etiquetas": ["PER", "ORG"]})
def crear_filtro(nlp, name, etiquetas):
    def filtro(doc):
        doc.ents = [e for e in doc.ents if e.label_ in etiquetas]
        return doc
    return filtro
```

---

## 11. Vectores y similitud

Los modelos `md` y `lg` incluyen vectores de palabras:

```python
nlp = spacy.load("es_core_news_md")
nlp("perro").similarity(nlp("gato"))          # 0.85
nlp("factura").similarity(nlp("recibo"))      # 0.52
doc.vector                                     # Media de los vectores de los tokens
token.has_vector, token.is_oov                 # ¿Tiene vector? ¿Fuera del vocabulario?
```

> La similitud con vectores de palabras es aproximada: mide temas parecidos, no significado preciso de frases. Para búsqueda semántica de calidad, use *embeddings* de frases (manual de Transformers o `sentence-transformers`).

---

## 12. Procesar grandes volúmenes

```python
docs = nlp.pipe(textos, batch_size=256)                      # Mucho más rápido que un bucle con nlp(t)
docs = nlp.pipe(textos, batch_size=256, n_process=4)         # Varios procesos (CPU)

for doc, contexto in nlp.pipe(((t, {"id": i}) for i, t in enumerate(textos)), as_tuples=True):
    guardar(contexto["id"], doc.ents)                         # Conservar metadatos
```

| Consejo | Efecto |
|---|---|
| `nlp.pipe` en lugar de `nlp()` en bucle | 2–10× más rápido |
| Desactivar componentes innecesarios | Hasta 2–3× más rápido |
| Modelo `sm` | Más rápido que `lg` o `trf` |
| `n_process` | Paraleliza en CPU (no use con modelos *transformer* en GPU) |
| `spacy.prefer_gpu()` | Usa la GPU para modelos `trf` |

---

## 13. Entrenar modelos propios

### 13.1 Flujo recomendado (configuración y línea de órdenes)

```bash
# 1. Generar una configuración base
python -m spacy init config config.cfg --lang es --pipeline ner,textcat --optimize accuracy

# 2. Convertir los datos anotados a DocBin (.spacy): entrenamiento y validación
# 3. Entrenar
python -m spacy train config.cfg --output ./modelo --paths.train ./train.spacy --paths.dev ./dev.spacy

# 4. Evaluar
python -m spacy evaluate ./modelo/model-best ./test.spacy
```

Preparar los datos para NER:

```python
from spacy.tokens import DocBin

db = DocBin()
for texto, entidades in datos:                        # entidades: [(inicio, fin, etiqueta), ...]
    doc = nlp.make_doc(texto)
    doc.ents = [doc.char_span(i, f, label=e, alignment_mode="contract") for i, f, e in entidades]
    db.add(doc)
db.to_disk("train.spacy")
```

El sistema de configuración hace el entrenamiento **reproducible** (todos los hiperparámetros en un archivo) y permite partir de un modelo existente o de un *transformer* (`spacy-transformers`).

### 13.2 Entrenamiento en código (prototipos)

```python
from spacy.training import Example
from spacy.util import minibatch

nlp = spacy.blank("es")
textcat = nlp.add_pipe("textcat")                  # "textcat_multilabel" si un texto puede tener varias
for etiqueta in ["facturación", "técnica"]:
    textcat.add_label(etiqueta)
ejemplos = [Example.from_dict(nlp.make_doc(t), {"cats": cats}) for t, cats in datos]
optimizador = nlp.initialize(lambda: ejemplos)
for epoca in range(20):
    perdidas = {}
    for lote in minibatch(ejemplos, size=8):
        nlp.update(lote, sgd=optimizador, drop=0.2, losses=perdidas)
nlp("La factura es incorrecta").cats
```

> Para anotar datos con eficiencia, existen herramientas como **Prodigy** (de los autores de spaCy), Label Studio o Doccano.

---

## 14. Visualización con displaCy

```python
from spacy import displacy

displacy.render(doc, style="ent", jupyter=True)           # Entidades en Jupyter
displacy.render(doc, style="dep", options={"compact": True})   # Árbol de dependencias
html = displacy.render(docs, style="ent", page=True, options={"colors": {"PEDIDO": "#f6c26b"}})
displacy.serve(doc, style="dep", port=5000)               # Servidor web local
```

---

## 15. Guardar y reutilizar: `DocBin` y modelos

```python
from spacy.tokens import DocBin

DocBin(docs=docs, store_user_data=True).to_disk("procesados.spacy")      # Guardar documentos
docs = list(DocBin().from_disk("procesados.spacy").get_docs(nlp.vocab))  # Recuperar

nlp.to_disk("mi_modelo")                       # Guardar la canalización completa (incluidas reglas)
nlp = spacy.load("mi_modelo")
doc.to_json()                                  # Exportar a JSON
```

> Al guardar con `nlp.to_disk`, los componentes propios definidos con `@Language.component` deben estar **importados** antes de cargar el modelo; si no, aparece un error de fábrica desconocida.

---

## 16. Ejemplo práctico completo

**Procesamiento de correos de reclamación**: amplía el modelo `es_core_news_md` con un **EntityRuler** que reconoce números de pedido, facturas, importes, fechas y nombres de empresas; añade un **componente propio** que corrige errores típicos del reconocedor de entidades y otro que marca los correos urgentes a partir de los **lemas**; extrae de cada correo un registro estructurado usando entidades y el **árbol de dependencias** (acción solicitada y su complemento); entrena un **clasificador de textos** para el motivo de la reclamación sobre textos lematizados; y guarda los documentos en `DocBin` y una visualización HTML con displaCy.

Requisitos: `pip install spacy` y `python -m spacy download es_core_news_md`

```python
"""
Procesamiento de correos de reclamación con spaCy.

1. Amplía el modelo en español con reglas (EntityRuler) para reconocer números
   de pedido, importes, fechas y empresas, y con un componente propio que
   marca los correos urgentes a partir de los lemas.
2. Extrae de cada correo un registro estructurado (cliente, empresa, lugar,
   pedido, importe, fecha, acción solicitada) usando entidades y dependencias.
3. Entrena un clasificador de textos (textcat) para el motivo de la reclamación,
   usando los lemas para que generalice mejor con pocos ejemplos.
4. Guarda los documentos procesados (DocBin) y una visualización HTML (displaCy).
"""
import random

import spacy
from spacy import displacy
from spacy.language import Language
from spacy.tokens import Doc, DocBin, Span
from spacy.training import Example
from spacy.util import minibatch

random.seed(0)
spacy.util.fix_random_seed(0)
Doc.set_extension("urgente", default=False, force=True)

CORREOS = [
    "Buenos días. Soy Marta Gómez y el pedido PED-48213 llegó roto a Zaragoza el 3 de marzo de 2026. "
    "Solicito el reembolso de 249,90 euros cuanto antes.",
    "Me llamo Jorge Ruiz. Seguur Logística me cobró 35 € por un envío a Valencia que nunca recibí. "
    "Exijo la devolución inmediata o pondré una denuncia.",
    "Hola, el técnico de Fibranet no vino a mi casa de Sevilla el 12 de mayo de 2026. "
    "Quiero cambiar la cita para la semana próxima.",
    "Le escribe Ana Torres, abogada de la empresa Cafés del Sur. La factura FAC-2026-0091 de 1.200 euros "
    "contiene un error en el IVA. Rogamos que la rectifiquen.",
    "El paquete del pedido PED-50077 sigue en Madrid desde el 20 de abril de 2026. ¿Cuándo lo entregarán?",
]

EMPRESAS = ["Seguur Logística", "Fibranet", "Cafés del Sur"]
MESES = ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio", "agosto", "septiembre",
         "octubre", "noviembre", "diciembre"]
PATRONES = (
    [{"label": "ORG", "pattern": empresa} for empresa in EMPRESAS] +
    [{"label": "PEDIDO", "pattern": [{"TEXT": {"REGEX": r"^PED-\d{5}$"}}]},
     {"label": "FACTURA", "pattern": [{"TEXT": {"REGEX": r"^FAC-\d{4}-\d{4}$"}}]},
     {"label": "IMPORTE", "pattern": [{"LIKE_NUM": True}, {"LOWER": {"IN": ["euros", "euro", "€"]}}]},
     {"label": "FECHA", "pattern": [{"IS_DIGIT": True}, {"LOWER": "de"}, {"LOWER": {"IN": MESES}},
                                    {"LOWER": "de"}, {"SHAPE": "dddd"}]}]
)
VERBOS_PETICION = {"solicitar", "querer", "exigir", "rogar", "pedir"}
FORMAS_PETICION = {"solicito": "solicitar", "quiero": "querer", "exijo": "exigir",   # Por si el modelo
                   "rogamos": "rogar", "pido": "pedir"}                             # falla al inicio de frase
SALUDOS = {"buenos", "buenas", "hola", "saludos", "estimados"}
LEMAS_URGENCIA = {"exigir", "inmediato", "denuncia", "abogado", "abogada", "urgente", "cuanto"}

# Datos de entrenamiento del clasificador de motivos (textos breves etiquetados)
MOTIVOS = {
    "producto_dañado": ["llegó roto", "el producto venía dañado", "la caja estaba aplastada y el contenido roto",
                        "me enviaron el artículo defectuoso", "la pantalla vino rayada", "el mueble llegó con golpes",
                        "faltan piezas y una está rota", "quiero el reembolso porque llegó estropeado"],
    "cobro_indebido": ["me cobraron dos veces", "el importe de la factura es incorrecto", "error en el IVA de la factura",
                       "me cobraron un envío que no recibí", "cargo no autorizado en mi tarjeta",
                       "rectifiquen la factura por favor", "la factura tiene un importe equivocado",
                       "exijo la devolución del cobro"],
    "retraso_entrega": ["el pedido no ha llegado", "el paquete sigue en el almacén", "¿cuándo entregarán mi pedido?",
                        "llevo dos semanas esperando el envío", "el envío está retrasado", "el paquete no se mueve",
                        "el seguimiento no avanza desde hace días", "todavía no he recibido el pedido"],
    "cita_técnica": ["el técnico no vino", "quiero cambiar la cita de instalación", "nadie vino a reparar la avería",
                     "necesito una visita del técnico", "anularon la cita sin avisar", "el instalador no se presentó",
                     "quiero otra fecha para la visita técnica", "esperé al técnico toda la mañana en casa"],
}


# ------------------------------------------------------------------ Componentes de la canalización
@Language.component("detector_urgencia")
def detector_urgencia(doc: Doc) -> Doc:
    doc._.urgente = any(token.lemma_.lower() in LEMAS_URGENCIA for token in doc)
    return doc


@Language.component("limpiar_entidades")
def limpiar_entidades(doc: Doc) -> Doc:
    """Corrige errores típicos del NER estadístico: palabras en mayúscula al inicio de frase
    («Exijo», «Buenos días») y verbos pegados a nombres («Soy Marta Gómez» → «Marta Gómez»)."""
    entidades = []
    for ent in doc.ents:
        if ent.label_ not in {"PER", "LOC", "ORG", "MISC"} or ent.text in EMPRESAS:   # Reglas: se conservan
            entidades.append(ent)
            continue
        propios = [t.i for t in ent if t.pos_ == "PROPN" and t.lower_ not in SALUDOS]
        if propios:                                                   # Recortar a los nombres propios
            entidades.append(Span(doc, propios[0], propios[-1] + 1, label=ent.label_))
    doc.ents = entidades
    return doc


def crear_nlp() -> Language:
    nlp = spacy.load("es_core_news_md")
    regla = nlp.add_pipe("entity_ruler", before="ner", config={"overwrite_ents": True})
    regla.add_patterns(PATRONES)
    nlp.add_pipe("limpiar_entidades", after="ner")
    nlp.add_pipe("detector_urgencia", last=True)
    return nlp


# ------------------------------------------------------------------ Extracción de información
def accion_solicitada(doc: Doc) -> str:
    """Verbo de petición (solicitar, querer, exigir…) y su complemento, a partir de las dependencias."""
    for token in doc:
        if token.lemma_.lower() in VERBOS_PETICION or token.lower_ in FORMAS_PETICION:
            complemento = [h for h in token.children if h.dep_ in {"obj", "xcomp", "ccomp"}]
            if complemento:
                inicio, fin = complemento[0].left_edge.i, complemento[0].right_edge.i + 1
                verbo = FORMAS_PETICION.get(token.lower_, token.lemma_.lower())
                return f"{verbo}: {doc[inicio:fin].text}"
    return "—"


def extraer(doc: Doc) -> dict:
    primera = lambda etiqueta: next((e.text for e in doc.ents if e.label_ == etiqueta), "—")
    return {"cliente": primera("PER"), "empresa": primera("ORG"), "lugar": primera("LOC"),
            "referencia": next((e.text for e in doc.ents if e.label_ in {"PEDIDO", "FACTURA"}), "—"),
            "importe": primera("IMPORTE"), "fecha": primera("FECHA"),
            "accion": accion_solicitada(doc), "urgente": doc._.urgente}


# ------------------------------------------------------------------ Clasificador de textos
def lematizar(nlp_base: Language, texto: str) -> str:
    """Texto en lemas y minúsculas: «me cobró» y «me cobraron» se convierten en «cobrar»."""
    return " ".join(t.lemma_.lower() for t in nlp_base(texto) if t.is_alpha and not t.is_stop)


def entrenar_clasificador(nlp_base: Language, epocas: int = 25) -> Language:
    nlp = spacy.blank("es")
    textcat = nlp.add_pipe("textcat")
    for motivo in MOTIVOS:
        textcat.add_label(motivo)
    ejemplos = [Example.from_dict(nlp.make_doc(lematizar(nlp_base, texto)),
                                  {"cats": {m: float(m == motivo) for m in MOTIVOS}})
                for motivo, textos in MOTIVOS.items() for texto in textos]
    optimizador = nlp.initialize(lambda: ejemplos)
    for epoca in range(epocas):
        random.shuffle(ejemplos)
        perdidas = {}
        for lote in minibatch(ejemplos, size=8):
            nlp.update(lote, sgd=optimizador, drop=0.2, losses=perdidas)
    print(f"Clasificador entrenado · {len(ejemplos)} ejemplos · pérdida final {perdidas['textcat']:.3f}")
    return nlp


def main() -> None:
    nlp = crear_nlp()
    print(f"spaCy {spacy.__version__} · modelo {nlp.meta['name']} {nlp.meta['version']}")
    print(f"Canalización: {nlp.pipe_names}\n")

    docs = list(nlp.pipe(CORREOS, batch_size=50))              # Procesamiento por lotes
    clasificador = entrenar_clasificador(nlp)

    print()
    for n, doc in enumerate(docs, 1):
        datos = extraer(doc)
        motivos = clasificador(lematizar(nlp, doc.text)).cats
        motivo = max(motivos, key=motivos.get)
        print(f"Correo {n}{' · ⚠ URGENTE' if datos['urgente'] else ''}")
        print(f"  Entidades: {', '.join(f'{e.text} [{e.label_}]' for e in doc.ents)}")
        print(f"  Cliente: {datos['cliente']} · Empresa: {datos['empresa']} · Lugar: {datos['lugar']}")
        print(f"  Referencia: {datos['referencia']} · Importe: {datos['importe']} · Fecha: {datos['fecha']}")
        print(f"  Acción: {datos['accion']} · Motivo: {motivo} ({motivos[motivo]:.0%})\n")

    # Lemas más frecuentes (sin palabras vacías ni signos)
    lemas = [t.lemma_.lower() for d in docs for t in d if t.is_alpha and not t.is_stop and t.pos_ in {"NOUN", "VERB"}]
    frecuentes = sorted(set(lemas), key=lambda l: (-lemas.count(l), l))[:8]
    print("Lemas más frecuentes: " + ", ".join(f"{l} ({lemas.count(l)})" for l in frecuentes))

    # Guardar documentos procesados y visualización
    DocBin(docs=docs, store_user_data=True).to_disk("correos.spacy")
    with open("correos_entidades.html", "w", encoding="utf-8") as f:
        f.write(displacy.render(docs, style="ent", page=True,
                                options={"colors": {"PEDIDO": "#f6c26b", "IMPORTE": "#9be09b", "FECHA": "#a6c8ff",
                                                    "FACTURA": "#f6c26b"}}))
    recuperados = list(DocBin().from_disk("correos.spacy").get_docs(nlp.vocab))
    print(f"Guardados {len(recuperados)} documentos (correos.spacy) y visualización (correos_entidades.html)")


if __name__ == "__main__":
    main()
```

Resultado:

```
spaCy 3.8.18 · modelo core_news_md 3.8.0
Canalización: ['tok2vec', 'morphologizer', 'parser', 'attribute_ruler', 'lemmatizer', 'entity_ruler', 'ner', 'limpiar_entidades', 'detector_urgencia']

Clasificador entrenado · 32 ejemplos · pérdida final 0.000

Correo 1 · ⚠ URGENTE
  Entidades: Marta Gómez [PER], PED-48213 [PEDIDO], Zaragoza [LOC], 3 de marzo de 2026 [FECHA], 249,90 euros [IMPORTE]
  Cliente: Marta Gómez · Empresa: — · Lugar: Zaragoza
  Referencia: PED-48213 · Importe: 249,90 euros · Fecha: 3 de marzo de 2026
  Acción: solicitar: el reembolso de 249,90 euros · Motivo: producto_dañado (98%)

Correo 2 · ⚠ URGENTE
  Entidades: Jorge Ruiz [PER], Seguur Logística [ORG], 35 € [IMPORTE], Valencia [LOC]
  Cliente: Jorge Ruiz · Empresa: Seguur Logística · Lugar: Valencia
  Referencia: — · Importe: 35 € · Fecha: —
  Acción: exigir: la devolución inmediata · Motivo: cobro_indebido (100%)

Correo 3
  Entidades: Fibranet [ORG], Sevilla [LOC], 12 de mayo de 2026 [FECHA]
  Cliente: — · Empresa: Fibranet · Lugar: Sevilla
  Referencia: — · Importe: — · Fecha: 12 de mayo de 2026
  Acción: querer: cambiar la cita para la semana próxima · Motivo: cita_técnica (100%)

Correo 4 · ⚠ URGENTE
  Entidades: Ana Torres [PER], Cafés del Sur [ORG], FAC-2026-0091 [FACTURA], 1.200 euros [IMPORTE]
  Cliente: Ana Torres · Empresa: Cafés del Sur · Lugar: —
  Referencia: FAC-2026-0091 · Importe: 1.200 euros · Fecha: —
  Acción: rogar: que la rectifiquen · Motivo: cobro_indebido (99%)

Correo 5
  Entidades: PED-50077 [PEDIDO], Madrid [LOC], 20 de abril de 2026 [FECHA]
  Cliente: — · Empresa: — · Lugar: Madrid
  Referencia: PED-50077 · Importe: — · Fecha: 20 de abril de 2026
  Acción: — · Motivo: retraso_entrega (97%)

Lemas más frecuentes: euros (2), pedido (2), abogada (1), abril (1), cambiar (1), casa (1), cita (1), cobrar (1)
Guardados 5 documentos (correos.spacy) y visualización (correos_entidades.html)
```

> El ejemplo muestra el enfoque habitual en producción: el modelo estadístico aporta el análisis lingüístico general y las **reglas** cubren lo que el modelo no sabe (identificadores, importes, fechas, empresas propias) o corrigen sus errores típicos. Entrenar el clasificador sobre lemas permite que «me cobró» y «me cobraron» se traten igual, lo que mejora mucho los resultados con pocos ejemplos de entrenamiento.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `OSError: [E050] Can't find model 'es_core_news_sm'` | Modelo no descargado (o en otro entorno virtual). | `python -m spacy download es_core_news_sm` en el mismo entorno. |
| Aviso `[W095] Model ... was trained with spaCy v3.x and may not be compatible` | Versión del modelo y de spaCy distintas. | `python -m spacy validate` y vuelva a descargar el modelo. |
| `[E002] Can't find factory for 'mi_componente'` | Componente propio no registrado al cargar. | Importe el módulo que define el componente antes de `spacy.load`. |
| `[E103] Trying to set conflicting doc.ents` | Entidades solapadas. | Use `spacy.util.filter_spans` o `doc.set_ents`. |
| `[E030] Sentence boundaries unset` al usar `doc.sents` | Canalización sin analizador ni `sentencizer`. | Añada `nlp.add_pipe("sentencizer")`. |
| `[W007] ... Doc.similarity based on empty vectors` | Modelo `sm` sin vectores. | Use `es_core_news_md` o `lg`. |
| Entidades incorrectas al inicio de frase («Exijo» como lugar) | Limitación de los modelos pequeños. | Modelo mayor, reglas, o un componente de filtrado. |
| `char_span` devuelve `None` | Los caracteres no coinciden con límites de tokens. | `alignment_mode="contract"` o `"expand"`. |
| Procesamiento muy lento | Bucle con `nlp(texto)` y todos los componentes activos. | `nlp.pipe(..., batch_size=...)` y desactive componentes. |
| Memoria creciente en procesos largos | El vocabulario crece con textos muy variados. | Reinicie el objeto `nlp` periódicamente o procese en lotes separados. |

---

## 18. Buenas prácticas

- Elija el **modelo según el caso**: `sm` para velocidad, `md`/`lg` para precisión y vectores, `trf` para máxima precisión con GPU.
- Combine **modelo estadístico + reglas**: el modelo generaliza y las reglas garantizan identificadores, formatos y términos propios.
- Procese con **`nlp.pipe`** y desactive los componentes que no use.
- Use **lemas** para normalizar textos antes de contar, buscar o clasificar.
- Encapsule reglas y lógica en **componentes** de la canalización y guarde la canalización completa con `nlp.to_disk`.
- Evalúe con textos reales del dominio y reentrene (`spacy train`) cuando las reglas no basten.
- Use el sistema de **configuración** para entrenamientos reproducibles y versionados.
- Fije las versiones de spaCy y de los modelos en el entorno de producción.

---

## 19. Referencia rápida

```python
import spacy; nlp = spacy.load("es_core_news_md"); doc = nlp(texto); docs = nlp.pipe(textos, batch_size=256)
token.text · lemma_ · pos_ · tag_ · dep_ · head · morph · is_stop · is_alpha · like_num · ent_type_
doc.ents (ent.text, ent.label_) · doc.sents · doc.noun_chunks · doc[i:j] · doc.char_span(a, b, label=...)
token.children · token.subtree · token.left_edge · token.right_edge · spacy.explain("nsubj")
Matcher(nlp.vocab).add("N", [[{"LOWER": "no"}, {"POS": "VERB"}]]) · PhraseMatcher(nlp.vocab, attr="LOWER")
nlp.add_pipe("entity_ruler", before="ner").add_patterns([{"label": "ORG", "pattern": "Empresa"}])
@Language.component("nombre") · nlp.add_pipe("nombre", last=True) · Doc.set_extension("x", default=None)
nlp.select_pipes(enable=[...]) · spacy.load(m, disable=[...], exclude=[...]) · spacy.blank("es")
nlp("a").similarity(nlp("b")) · doc.vector
python -m spacy init config · train · evaluate · download · validate
displacy.render(doc, style="ent" | "dep") · DocBin(docs=docs).to_disk(r) · nlp.to_disk(dir)
```

---

## 20. Ejercicios propuestos

1. Extraiga de un conjunto de noticias en español las personas y organizaciones más mencionadas y cuántas veces aparecen juntas en la misma frase.
2. Cree un `EntityRuler` que reconozca DNI/NIE, IBAN y números de teléfono españoles, y anonimice esos datos en los textos (sustitución por `[DNI]`, `[IBAN]`…).
3. Use el árbol de dependencias para extraer tripletas sujeto–verbo–objeto de frases de contratos.
4. Anote 200 frases con una entidad nueva (por ejemplo, `PRODUCTO`) y entrene un modelo con `spacy train` partiendo de `es_core_news_md`.
5. Amplíe el ejemplo completo con un componente que detecte el tono de cada correo (cortés, neutro, agresivo) mediante reglas sobre lemas y signos de exclamación.

---

## 21. Referencias

- Documentación oficial: https://spacy.io/
- Curso gratuito de spaCy (en español): https://course.spacy.io/es/
- Modelos para español: https://spacy.io/models/es
- Reglas y `Matcher`: https://spacy.io/usage/rule-based-matching
- Entrenamiento: https://spacy.io/usage/training
