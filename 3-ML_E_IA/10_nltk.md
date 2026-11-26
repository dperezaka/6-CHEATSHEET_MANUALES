# Manual de uso: NLTK

> **Categoría:** Inteligencia artificial · **Utilidad:** Procesamiento del lenguaje natural
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** Apache 2.0 · **Nombre:** *Natural Language Toolkit*

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación, importación y descarga de recursos](#2-instalación-importación-y-descarga-de-recursos)
3. [Conceptos clave](#3-conceptos-clave)
4. [Tokenización](#4-tokenización)
5. [Normalización: minúsculas, tildes y palabras vacías](#5-normalización-minúsculas-tildes-y-palabras-vacías)
6. [Raíces (*stemming*) y lemas](#6-raíces-stemming-y-lemas)
7. [Frecuencias, n-gramas y colocaciones](#7-frecuencias-n-gramas-y-colocaciones)
8. [Corpus incluidos](#8-corpus-incluidos)
9. [Etiquetado gramatical](#9-etiquetado-gramatical)
10. [Análisis sintáctico y entidades](#10-análisis-sintáctico-y-entidades)
11. [WordNet: significados y sinónimos](#11-wordnet-significados-y-sinónimos)
12. [Análisis de sentimiento](#12-análisis-de-sentimiento)
13. [Clasificación de textos](#13-clasificación-de-textos)
14. [Métricas y distancias entre textos](#14-métricas-y-distancias-entre-textos)
15. [NLTK y otras herramientas](#15-nltk-y-otras-herramientas)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**NLTK** (*Natural Language Toolkit*) es una de las librerías más veteranas y completas para el **procesamiento del lenguaje natural** (PLN) en Python. Ofrece herramientas para tokenizar, normalizar, obtener raíces y lemas, etiquetar gramaticalmente, analizar la sintaxis, calcular estadísticas de texto y entrenar clasificadores, junto con más de 100 corpus y recursos léxicos (WordNet, listas de palabras vacías, corpus anotados).

**Ventajas:**
- Muy completa para aprender y enseñar PLN: cada técnica clásica tiene su implementación.
- Gran colección de corpus y recursos léxicos, incluidos varios en español.
- Herramientas estadísticas (frecuencias, colocaciones, n-gramas) y clasificadores sencillos.
- Código transparente y documentado, ideal para entender los algoritmos.

**Limitaciones:**
- Más lenta y menos orientada a producción que spaCy.
- Varios componentes (etiquetador gramatical, entidades, sentimiento VADER) solo vienen preentrenados para **inglés**; para español hay que entrenarlos o usar otras librerías.
- No incluye modelos neuronales modernos.
- Requiere descargar los recursos por separado.

| Si necesita… | Opción |
|---|---|
| Aprender PLN, estadísticas de texto, recursos léxicos, prototipos | **NLTK** |
| PLN en producción, análisis lingüístico de español con modelos entrenados | spaCy |
| Clasificación y análisis con modelos de lenguaje modernos | Hugging Face Transformers |
| Vectorización y clasificación de textos con ML clásico | scikit-learn (`TfidfVectorizer`) |

---

## 2. Instalación, importación y descarga de recursos

```bash
pip install nltk
```

```python
import nltk
print(nltk.__version__)
```

### 2.1 Descargar recursos

Los corpus, modelos y listas se descargan aparte, una sola vez:

```python
nltk.download("punkt_tab")        # Tokenizador de frases (incluye español)
nltk.download("stopwords")        # Palabras vacías (incluye español)
nltk.download("wordnet")          # WordNet (inglés)
nltk.download("omw-2.0")          # Open Multilingual WordNet (incluye español)
nltk.download("cess_esp")         # Corpus anotado en español
nltk.download("averaged_perceptron_tagger_eng")   # Etiquetador gramatical (inglés)
nltk.download("vader_lexicon")    # Sentimiento (inglés)
nltk.download("popular")          # Paquete con los recursos más usados
```

Desde la terminal: `python -m nltk.downloader punkt_tab stopwords`.

Los recursos se guardan en `~/nltk_data` (o en la ruta de la variable de entorno `NLTK_DATA`). Consulte las rutas de búsqueda con `nltk.data.path`.

> **Descargas a través de un *proxy*:** desde NLTK 3.10, por seguridad, `nltk.download` se niega a descargar si la conexión pasa por un *proxy* (aparece un aviso sobre protección SSRF). En redes corporativas con un *proxy* de confianza, actívelo con la variable de entorno `NLTK_ALLOW_PROXIED_URLOPEN=1`. En servidores sin conexión, copie la carpeta `nltk_data` desde otro equipo.

> Los nombres de algunos recursos cambiaron en versiones recientes: `punkt` → `punkt_tab`, `averaged_perceptron_tagger` → `averaged_perceptron_tagger_eng`, y WordNet multilingüe usa `omw-2.0`. Los mensajes de `LookupError` indican el nombre exacto que hay que descargar.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Corpus | Colección de textos, a menudo anotados (categorías, etiquetas gramaticales). |
| Token | Unidad mínima de análisis: palabra, signo de puntuación, número. |
| Tokenización | Dividir un texto en frases o tokens. |
| Palabras vacías (*stopwords*) | Palabras muy frecuentes con poco contenido (artículos, preposiciones). |
| Raíz (*stem*) | Forma truncada de una palabra (`facturación` → `factur`). |
| Lema | Forma canónica de diccionario (`cobraron` → `cobrar`). |
| Etiqueta gramatical (POS) | Categoría de una palabra: nombre, verbo, adjetivo… |
| N-grama | Secuencia de *n* tokens consecutivos. |
| Colocación | Combinación de palabras que aparece junta más de lo que cabría esperar por azar. |
| Rasgo (*feature*) | Característica de un texto usada por un clasificador. |

**Flujo típico de procesamiento:**

```
Texto ─► Tokenización ─► Normalización (minúsculas, tildes, palabras vacías) ─► Raíces / lemas
      ─► Análisis (frecuencias, n-gramas, etiquetas) o rasgos ─► Clasificador
```

---

## 4. Tokenización

```python
from nltk.tokenize import sent_tokenize, word_tokenize, RegexpTokenizer, TweetTokenizer

texto = "Hola, Sr. García. ¿Cómo está? Bien, gracias."
sent_tokenize(texto, language="spanish")
# ['Hola, Sr. García.', '¿Cómo está?', 'Bien, gracias.']   ← reconoce la abreviatura «Sr.»

word_tokenize("Me han cobrado dos veces la factura.", language="spanish")
# ['Me', 'han', 'cobrado', 'dos', 'veces', 'la', 'factura', '.']
```

| Tokenizador | Uso |
|---|---|
| `word_tokenize(texto, language=...)` | Uso general (divide también en frases) |
| `sent_tokenize(texto, language=...)` | Frases, con abreviaturas del idioma |
| `RegexpTokenizer(r"\w+")` | Solo palabras, según una expresión regular |
| `TweetTokenizer()` | Redes sociales: conserva `@usuarios`, `#etiquetas` y emoticonos |
| `WhitespaceTokenizer()` | Divide por espacios |
| `ToktokTokenizer()` | Rápido, multilingüe |

---

## 5. Normalización: minúsculas, tildes y palabras vacías

```python
import unicodedata
from nltk.corpus import stopwords

vacias = set(stopwords.words("spanish"))            # 313 palabras: «de», «la», «que», «el»…
vacias |= {"gracias", "favor"}                      # Ampliar según el dominio

def quitar_tildes(texto: str) -> str:
    return "".join(c for c in unicodedata.normalize("NFD", texto) if unicodedata.category(c) != "Mn")

def normalizar(texto: str) -> list[str]:
    tokens = word_tokenize(texto.lower(), language="spanish")
    return [t for t in tokens if t.isalpha() and t not in vacias]
```

> Revise la lista de palabras vacías para su caso: incluye palabras como «no» o «sin», que son importantes en análisis de sentimiento («no funciona»). Quitar tildes ayuda a agrupar variantes escritas sin ellas, pero puede confundir palabras distintas («esta»/«está»).

---

## 6. Raíces (*stemming*) y lemas

### 6.1 Raíces en español

```python
from nltk.stem import SnowballStemmer

raiz = SnowballStemmer("spanish")
[raiz.stem(p) for p in ["factura", "facturación", "facturar", "facturas"]]
# ['factur', 'factur', 'factur', 'factur']
SnowballStemmer.languages        # Idiomas disponibles
```

### 6.2 Lemas

`WordNetLemmatizer` solo lematiza en **inglés**:

```python
from nltk.stem import WordNetLemmatizer
WordNetLemmatizer().lemmatize("routers")              # 'router'
WordNetLemmatizer().lemmatize("running", pos="v")     # 'run'
```

| Técnica | Resultado | Ventaja | Inconveniente |
|---|---|---|---|
| Raíz | `cobraron` → `cobr` | Rápido, sin diccionario | Raíces no legibles; agrupa de más o de menos |
| Lema | `cobraron` → `cobrar` | Formas reales | Requiere modelo del idioma (en español: spaCy) |

---

## 7. Frecuencias, n-gramas y colocaciones

### 7.1 Distribución de frecuencias

```python
from nltk.probability import FreqDist

frecuencias = FreqDist(tokens)
frecuencias.most_common(10)
frecuencias["factura"]
frecuencias.plot(20)                     # Requiere matplotlib

from nltk.probability import ConditionalFreqDist
por_categoria = ConditionalFreqDist((categoria, palabra) for categoria, palabras in datos for palabra in palabras)
por_categoria["facturación"].most_common(5)
```

### 7.2 N-gramas

```python
from nltk import ngrams, bigrams
list(bigrams(["no", "tengo", "línea"]))            # [('no', 'tengo'), ('tengo', 'línea')]
list(ngrams(tokens, 3))
```

### 7.3 Colocaciones

```python
from nltk.collocations import BigramCollocationFinder, BigramAssocMeasures

buscador = BigramCollocationFinder.from_words(tokens)
buscador.apply_freq_filter(5)                       # Ignorar pares poco frecuentes
buscador.nbest(BigramAssocMeasures.pmi, 10)         # Información mutua puntual
buscador.nbest(BigramAssocMeasures.likelihood_ratio, 10)
```

Las colocaciones revelan expresiones del dominio («cuenta bancaria», «dar de baja», «teléfono fijo»), útiles para crear vocabularios o rasgos.

### 7.4 Exploración de un texto

```python
texto = nltk.Text(tokens)
texto.concordance("factura", width=60)     # Contextos en que aparece una palabra
texto.similar("router")                    # Palabras que aparecen en contextos parecidos
texto.collocations()
texto.dispersion_plot(["factura", "router"])
```

---

## 8. Corpus incluidos

| Corpus | Contenido | Idioma |
|---|---|---|
| `cess_esp` | ~6 000 frases con etiquetas gramaticales y árboles sintácticos | Español |
| `conll2002` | Entidades nombradas | Español y neerlandés |
| `europarl_raw` | Actas del Parlamento Europeo | Varios (incluido español) |
| `udhr` | Declaración Universal de los Derechos Humanos | +300 idiomas |
| `stopwords` | Palabras vacías | 30+ idiomas |
| `gutenberg`, `brown`, `reuters`, `movie_reviews` | Literatura, prensa, críticas | Inglés |
| `wordnet`, `omw-2.0` | Red léxica de significados | Inglés / multilingüe |

```python
from nltk.corpus import cess_esp
cess_esp.words()[:10]
cess_esp.tagged_sents()[0][:5]     # [('El', 'da0ms0'), ('grupo', 'ncms000'), ...]
```

**Corpus propio** a partir de archivos de texto:

```python
from nltk.corpus.reader import PlaintextCorpusReader
corpus = PlaintextCorpusReader("textos/", r".*\.txt", encoding="utf-8")
corpus.fileids(); corpus.words("informe1.txt"); corpus.sents()
```

---

## 9. Etiquetado gramatical

### 9.1 Inglés (modelo incluido)

```python
nltk.pos_tag(word_tokenize("The router does not work"))
# [('The', 'DT'), ('router', 'NN'), ('does', 'VBZ'), ('not', 'RB'), ('work', 'VB')]
```

### 9.2 Español: entrenar un etiquetador con CESS-ESP

NLTK no incluye un etiquetador preentrenado para español, pero se puede entrenar uno en segundos:

```python
from nltk.tag import DefaultTagger, UnigramTagger, BigramTagger

frases = [[(p.lower(), e[:2]) for p, e in frase] for frase in cess_esp.tagged_sents()]   # Etiquetas simplificadas
entrenamiento, prueba = frases[:5500], frases[5500:]
etiquetador = BigramTagger(entrenamiento,
                           backoff=UnigramTagger(entrenamiento, backoff=DefaultTagger("nc")))
etiquetador.accuracy(prueba)
etiquetador.tag(["el", "router", "no", "funciona"])
```

El *backoff* encadena etiquetadores: si el de bigramas no conoce el contexto, recurre al de unigramas, y si tampoco conoce la palabra, asigna «nc» (nombre común).

| Etiqueta EAGLES (2 letras) | Categoría |
|---|---|
| `nc` / `np` | Nombre común / propio |
| `vm` / `va` / `vs` | Verbo principal / auxiliar / *ser* |
| `aq` | Adjetivo calificativo |
| `da` / `di` | Artículo / determinante indefinido |
| `sp` | Preposición |
| `rg` / `rn` | Adverbio / negación |

> Para un etiquetado y una lematización de calidad en español, use spaCy (manual 11).

---

## 10. Análisis sintáctico y entidades

### 10.1 Gramáticas y análisis sintáctico

```python
gramatica = nltk.CFG.fromstring("""
    O -> SN SV
    SN -> Det N
    SV -> V SN
    Det -> 'el' | 'la'
    N -> 'técnico' | 'avería'
    V -> 'reparó'
""")
for arbol in nltk.ChartParser(gramatica).parse("el técnico reparó la avería".split()):
    arbol.pretty_print()
```

### 10.2 Fragmentación (*chunking*) con expresiones regulares

```python
fragmentador = nltk.RegexpParser("SN: {<da>?<nc><aq>*}")     # Sintagmas nominales con etiquetas EAGLES
fragmentador.parse(etiquetador.tag(tokens))
```

### 10.3 Entidades nombradas

`nltk.ne_chunk` funciona solo en **inglés** (recursos `maxent_ne_chunker_tab` y `words`). Para español, use spaCy o Transformers, o entrene un modelo con el corpus `conll2002`.

---

## 11. WordNet: significados y sinónimos

```python
from nltk.corpus import wordnet as wn

significados = wn.synsets("perro", lang="spa")        # Requiere omw-2.0
significados[0].definition()                          # Definición (en inglés)
significados[0].lemma_names("spa")                    # ['can', 'canino', 'perro', ...]
significados[0].hypernyms()                           # Conceptos más generales
wn.synset("dog.n.01").path_similarity(wn.synset("cat.n.01"))    # Similitud semántica
wn.langs()                                            # Idiomas disponibles
```

Usos: ampliar consultas con sinónimos, agrupar términos, medir similitud entre conceptos.

---

## 12. Análisis de sentimiento

**VADER** está diseñado para textos cortos en **inglés** (redes sociales, reseñas):

```python
from nltk.sentiment import SentimentIntensityAnalyzer
SentimentIntensityAnalyzer().polarity_scores("The delivery was great!")
# {'neg': 0.0, 'neu': 0.406, 'pos': 0.594, 'compound': 0.6588}
```

Para español:
- Entrene un clasificador con textos etiquetados (sección 13).
- Use un modelo de Transformers en español o multilingüe (manual 08).

> No aplique VADER a textos en español: su léxico es inglés y los resultados no son fiables.

---

## 13. Clasificación de textos

NLTK incluye clasificadores sencillos que trabajan con **diccionarios de rasgos**:

```python
from nltk.classify import NaiveBayesClassifier, accuracy

def rasgos(texto):
    return {f"contiene({raiz.stem(t)})": True for t in normalizar(texto)}

conjunto = [(rasgos(texto), categoria) for texto, categoria in datos]
entrenamiento, prueba = conjunto[:800], conjunto[800:]

clasificador = NaiveBayesClassifier.train(entrenamiento)
accuracy(clasificador, prueba)
clasificador.show_most_informative_features(10)
clasificador.classify(rasgos("La factura no es correcta"))
clasificador.prob_classify(rasgos("...")).prob("facturación")
```

| Clasificador | Notas |
|---|---|
| `NaiveBayesClassifier` | Rápido, interpretable, buena referencia |
| `DecisionTreeClassifier` | Reglas legibles; lento con muchos rasgos |
| `MaxentClassifier` | Regresión logística; lento |
| `SklearnClassifier(LinearSVC())` | Usa cualquier modelo de scikit-learn con rasgos de NLTK |

```python
from nltk.classify.scikitlearn import SklearnClassifier
from sklearn.svm import LinearSVC
SklearnClassifier(LinearSVC()).train(entrenamiento)
```

> Para volúmenes grandes o máximo rendimiento, use directamente `TfidfVectorizer` + un clasificador de scikit-learn, o un modelo de Transformers ajustado.

---

## 14. Métricas y distancias entre textos

```python
nltk.edit_distance("dirección", "direccion")              # 1 (distancia de Levenshtein)
nltk.jaccard_distance(set(tokens_a), set(tokens_b))       # 0 = iguales, 1 = disjuntos
from nltk.metrics import ConfusionMatrix, precision, recall, f_measure
print(ConfusionMatrix(reales, predichos))
from nltk.translate.bleu_score import sentence_bleu        # Calidad de traducción
```

**Corrección ortográfica sencilla** con distancia de edición:

```python
vocabulario = {"factura", "router", "fibra", "portabilidad"}
corregir = lambda palabra: min(vocabulario, key=lambda v: nltk.edit_distance(palabra, v))
corregir("rooter")              # 'router'
```

---

## 15. NLTK y otras herramientas

| Tarea | NLTK | Alternativa recomendada en producción |
|---|---|---|
| Tokenización | `word_tokenize` | spaCy |
| Palabras vacías | `stopwords` | spaCy (`token.is_stop`) |
| Raíces | `SnowballStemmer` | — |
| Lemas en español | — | spaCy |
| Etiquetado gramatical en español | Entrenar con `cess_esp` | spaCy |
| Entidades en español | — | spaCy, Transformers |
| Sentimiento en español | Entrenar clasificador | Transformers |
| Estadística de corpus, colocaciones | **NLTK** | — |
| WordNet multilingüe | **NLTK** | — |

Una combinación habitual: spaCy o Transformers para el análisis lingüístico y NLTK para estadísticas de corpus, colocaciones, métricas y recursos léxicos.

---

## 16. Ejemplo práctico completo

**Análisis y clasificación de incidencias de soporte en español**: genera 1 200 incidencias de una operadora de telecomunicaciones en cuatro categorías; muestra la tokenización, normalización y obtención de raíces de un texto; calcula las palabras más frecuentes por categoría y las **colocaciones** del dominio; **entrena un etiquetador gramatical para español** con el corpus CESS-ESP; y entrena y evalúa un **clasificador Naive Bayes** con rasgos de raíces y bigramas, mostrando los rasgos más informativos y las probabilidades para incidencias nuevas.

Requisitos: `pip install nltk` (el programa descarga `punkt_tab`, `stopwords` y `cess_esp` si faltan)

```python
"""
Análisis y clasificación de incidencias de soporte en español con NLTK.

1. Normaliza y tokeniza los textos, elimina palabras vacías y obtiene raíces.
2. Analiza frecuencias de términos y colocaciones (bigramas significativos).
3. Entrena un etiquetador gramatical para español con el corpus CESS-ESP.
4. Entrena y evalúa un clasificador Naive Bayes de categorías de incidencia.
"""
import random
import unicodedata
from collections import Counter

import nltk
from nltk.classify import NaiveBayesClassifier, accuracy
from nltk.collocations import BigramAssocMeasures, BigramCollocationFinder
from nltk.corpus import cess_esp, stopwords
from nltk.probability import FreqDist
from nltk.stem import SnowballStemmer
from nltk.tag import BigramTagger, DefaultTagger, UnigramTagger
from nltk.tokenize import word_tokenize

RECURSOS = ["punkt_tab", "stopwords", "cess_esp"]
random.seed(42)

PLANTILLAS = {
    "facturación": ["me han cobrado dos veces la factura de {mes}", "el importe de la factura no coincide con mi tarifa",
                    "no encuentro la factura de {mes} en el área de cliente", "quiero cambiar la cuenta bancaria de domiciliación",
                    "me cobraron una penalización que no corresponde", "solicito una factura rectificativa del recibo de {mes}"],
    "técnica": ["la fibra se corta varias veces al día desde {mes}", "el router no enciende y la luz parpadea en rojo",
                "el técnico no vino a revisar la avería del router", "después del cambio de tarifa no tengo internet",
                "la velocidad de internet es muy lenta por las tardes", "no tengo línea en el teléfono fijo",
                "la televisión se queda congelada al cambiar de canal", "el wifi no llega a las habitaciones del fondo"],
    "contratación": ["quiero contratar la tarifa de fibra con televisión", "me gustaría ampliar la velocidad de mi fibra",
                     "deseo dar de baja la línea móvil adicional", "quiero portar mi número desde otro operador",
                     "me interesa la oferta de {mes} para clientes nuevos", "cómo puedo añadir otra línea móvil a mi contrato",
                     "quiero saber cuánto costaría la tarifa con más gigas"],
    "envíos": ["el router nuevo no ha llegado todavía", "el repartidor dejó el paquete en otra dirección",
               "el pedido de la tarjeta SIM lleva una semana de retraso", "quiero cambiar la dirección de entrega del pedido",
               "me llegó el decodificador roto", "el seguimiento del envío no se actualiza desde {mes}",
               "el técnico tenía que traer el router y no ha llegado"],
}
COLETILLAS = ["", " por favor", " es urgente", " ya llamé ayer", " gracias", " llevo días esperando",
              " necesito una solución", " estoy muy molesto"]
MESES = ["enero", "febrero", "marzo", "abril", "mayo", "junio"]


def generar_incidencias(n: int = 1200) -> list[tuple[str, str]]:
    datos = []
    for _ in range(n):
        categoria = random.choice(list(PLANTILLAS))
        texto = random.choice(PLANTILLAS[categoria]).format(mes=random.choice(MESES)) + random.choice(COLETILLAS)
        datos.append((texto[0].upper() + texto[1:] + ".", categoria))
    return datos


# ------------------------------------------------------------------ Preprocesado
MESES_TODOS = ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio", "agosto", "septiembre",
               "octubre", "noviembre", "diciembre"]
VACIAS = (set(stopwords.words("spanish")) | {"quiero", "gustaría", "deseo", "favor", "gracias"}
          | set(MESES_TODOS))                                  # Los meses no indican la categoría
RAIZ = SnowballStemmer("spanish")


def quitar_tildes(texto: str) -> str:
    return "".join(c for c in unicodedata.normalize("NFD", texto) if unicodedata.category(c) != "Mn")


def normalizar(texto: str) -> list[str]:
    """Tokeniza, pasa a minúsculas, elimina signos y palabras vacías."""
    tokens = word_tokenize(texto.lower(), language="spanish")
    return [t for t in tokens if t.isalpha() and t not in VACIAS and len(t) > 2]


def caracteristicas(texto: str) -> dict[str, bool]:
    """Bolsa de raíces (sin tildes) + bigramas de raíces, como diccionario de rasgos."""
    raices = [quitar_tildes(RAIZ.stem(t)) for t in normalizar(texto)]
    rasgos = {f"raiz({r})": True for r in raices}
    rasgos |= {f"bigrama({a}_{b})": True for a, b in nltk.bigrams(raices)}
    return rasgos


def main() -> None:
    for recurso in RECURSOS:                                  # Descarga solo si falta
        nltk.download(recurso, quiet=True)
    print(f"NLTK {nltk.__version__}\n")

    incidencias = generar_incidencias()
    ejemplo = incidencias[0][0]
    print(f"Texto:        {ejemplo}")
    print(f"Tokens:       {word_tokenize(ejemplo, language='spanish')}")
    print(f"Normalizado:  {normalizar(ejemplo)}")
    print(f"Raíces:       {[RAIZ.stem(t) for t in normalizar(ejemplo)]}\n")

    # 1. Frecuencias de términos por categoría
    for categoria in PLANTILLAS:
        palabras = [p for t, c in incidencias if c == categoria for p in normalizar(t)]
        frecuentes = ", ".join(f"{p} ({n})" for p, n in FreqDist(palabras).most_common(5))
        print(f"{categoria:<13} {frecuentes}")

    # 2. Colocaciones: pares de palabras que aparecen juntos más de lo esperable
    todas = [p for t, _ in incidencias for p in normalizar(t)]
    buscador = BigramCollocationFinder.from_words(todas)
    buscador.apply_freq_filter(10)
    colocaciones = buscador.nbest(BigramAssocMeasures.pmi, 8)
    print(f"\nColocaciones (PMI): {', '.join(' '.join(c) for c in colocaciones)}")

    # 3. Etiquetador gramatical entrenado con el corpus CESS-ESP (≈ 6 000 frases en español)
    frases = [[(palabra.lower(), etiqueta[:2]) for palabra, etiqueta in frase] for frase in cess_esp.tagged_sents()]
    random.shuffle(frases)
    corte = int(len(frases) * 0.9)
    respaldo = UnigramTagger(frases[:corte], backoff=DefaultTagger("nc"))   # nc = nombre común
    etiquetador = BigramTagger(frases[:corte], backoff=respaldo)
    print(f"\nEtiquetador gramatical · exactitud en prueba: {etiquetador.accuracy(frases[corte:]):.1%}")
    frase = word_tokenize("El repartidor dejó el paquete roto en otra dirección", language="spanish")
    print("  " + " ".join(f"{p}/{e}" for p, e in etiquetador.tag([p.lower() for p in frase])))

    # 4. Clasificador de categorías
    conjunto = [(caracteristicas(t), c) for t, c in incidencias]
    random.shuffle(conjunto)
    corte = int(len(conjunto) * 0.8)
    clasificador = NaiveBayesClassifier.train(conjunto[:corte])
    print(f"\nClasificador Naive Bayes · exactitud en prueba: {accuracy(clasificador, conjunto[corte:]):.1%}")
    print("Rasgos más informativos:")
    for rasgo, _ in clasificador.most_informative_features(6):
        distribucion = Counter({c: clasificador.prob_classify({rasgo: True}).prob(c) for c in PLANTILLAS})
        categoria, prob = distribucion.most_common(1)[0]
        print(f"  {rasgo:<28} → {categoria} ({prob:.0%})")

    print("\nIncidencias nuevas:")
    for texto in ["La factura de julio tiene un cargo que no reconozco.",
                  "Desde esta mañana no funciona internet y el router hace ruido.",
                  "¿Puedo pasar mi número de móvil a vuestra compañía?",
                  "El técnico no vino y el paquete con el router tampoco ha llegado."]:
        probabilidades = clasificador.prob_classify(caracteristicas(texto))
        orden = sorted(PLANTILLAS, key=probabilidades.prob, reverse=True)
        print(f"  {orden[0]:<13} ({probabilidades.prob(orden[0]):.0%}, 2.ª: {orden[1]} "
              f"{probabilidades.prob(orden[1]):.0%})  {texto}")


if __name__ == "__main__":
    main()
```

Resultado:

```
NLTK 3.10.4

Texto:        Me han cobrado dos veces la factura de junio gracias.
Tokens:       ['Me', 'han', 'cobrado', 'dos', 'veces', 'la', 'factura', 'de', 'junio', 'gracias', '.']
Normalizado:  ['cobrado', 'dos', 'veces', 'factura']
Raíces:       ['cobr', 'dos', 'vec', 'factur']

facturación   factura (204), cobrado (55), dos (55), veces (55), cobraron (53)
técnica       router (76), internet (64), wifi (45), llega (45), habitaciones (45)
contratación  línea (97), móvil (97), tarifa (93), fibra (84), cómo (55)
envíos        router (88), llegado (88), pedido (78), dirección (77), llamé (49)

Colocaciones (PMI): lenta tardes, teléfono fijo, corta varias, enciende luz, luz parpadea, parpadea rojo, después cambio, queda congelada

Etiquetador gramatical · exactitud en prueba: 91.5%
  el/da repartidor/nc dejó/vm el/da paquete/nc roto/vm en/sp otra/di dirección/nc

Clasificador Naive Bayes · exactitud en prueba: 100.0%
Rasgos más informativos:
  raiz(line)                   → contratación (75%)
  raiz(factur)                 → facturación (99%)
  raiz(lleg)                   → envíos (72%)
  raiz(tarif)                  → contratación (49%)
  raiz(llev)                   → envíos (38%)
  raiz(fibr)                   → contratación (69%)

Incidencias nuevas:
  facturación   (99%, 2.ª: contratación 0%)  La factura de julio tiene un cargo que no reconozco.
  técnica       (99%, 2.ª: envíos 1%)  Desde esta mañana no funciona internet y el router hace ruido.
  contratación  (100%, 2.ª: envíos 0%)  ¿Puedo pasar mi número de móvil a vuestra compañía?
  técnica       (96%, 2.ª: envíos 4%)  El técnico no vino y el paquete con el router tampoco ha llegado.
```

> La exactitud del 100 % se debe a que las incidencias se generan a partir de plantillas: con textos reales, escritos libremente por clientes, cabe esperar valores más bajos y es imprescindible evaluar con datos reales. El etiquetador alcanza un 91,5 % de exactitud entrenado solo con CESS-ESP; nótese que etiqueta «roto» como verbo (participio), una ambigüedad típica.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `LookupError: Resource punkt_tab not found` | Recurso no descargado. | `nltk.download("punkt_tab")` (el mensaje indica el nombre exacto). |
| `LookupError: Resource 'omw-2.0' not found` al usar `lang="spa"` | WordNet multilingüe no descargado. | `nltk.download("omw-2.0")`. |
| La descarga falla con un aviso sobre *proxy* y SSRF | Protección de NLTK ≥ 3.10 ante *proxies*. | `NLTK_ALLOW_PROXIED_URLOPEN=1` si el *proxy* es de confianza, o copie `nltk_data` manualmente. |
| La tokenización de frases corta en abreviaturas («Sr.», «Dña.») | Se usó el idioma por defecto (inglés). | `sent_tokenize(texto, language="spanish")`. |
| `pos_tag` da etiquetas absurdas en español | El modelo incluido es inglés. | Entrene con `cess_esp` o use spaCy. |
| Resultados de sentimiento sin sentido en español | VADER es solo para inglés. | Clasificador propio o modelo de Transformers. |
| `UnicodeDecodeError` al leer corpus propios | Codificación distinta de UTF-8. | Indique `encoding=` en el lector. |
| Se pierden negaciones al eliminar palabras vacías | «no», «sin» están en la lista. | Quítelas de la lista: `vacias -= {"no", "sin"}`. |
| Clasificador con exactitud alta en pruebas y baja en producción | Datos de prueba poco representativos o repetidos. | Evalúe con datos reales y separe por cliente o fecha. |

---

## 18. Buenas prácticas

- Descargue los recursos al instalar la aplicación (o en la imagen Docker), no en cada ejecución.
- Indique siempre `language="spanish"` en los tokenizadores.
- Adapte la lista de palabras vacías al dominio y a la tarea (conserve las negaciones si analiza opiniones).
- Use raíces para agrupar variantes rápidamente, y spaCy cuando necesite lemas de calidad.
- Analice frecuencias y colocaciones antes de modelar: revelan el vocabulario y los problemas de los datos.
- Empiece con un clasificador sencillo (Naive Bayes) como referencia y compárelo con modelos más avanzados.
- Separe la normalización en funciones reutilizables y aplíquela igual en entrenamiento y predicción.
- Evalúe siempre con textos reales del dominio.

---

## 19. Referencia rápida

```python
import nltk; nltk.download("punkt_tab"); nltk.download("stopwords")
from nltk.tokenize import word_tokenize, sent_tokenize, RegexpTokenizer, TweetTokenizer
word_tokenize(t, language="spanish") · sent_tokenize(t, language="spanish")
from nltk.corpus import stopwords, cess_esp, wordnet as wn
stopwords.words("spanish") · cess_esp.tagged_sents() · wn.synsets("perro", lang="spa")
from nltk.stem import SnowballStemmer; SnowballStemmer("spanish").stem("facturación")
from nltk.probability import FreqDist, ConditionalFreqDist; FreqDist(tokens).most_common(10)
nltk.bigrams(t) · nltk.ngrams(t, 3) · BigramCollocationFinder.from_words(t).nbest(BigramAssocMeasures.pmi, 10)
nltk.Text(tokens).concordance("palabra")
UnigramTagger(frases, backoff=DefaultTagger("nc")) · BigramTagger(...) · etiquetador.accuracy(prueba)
NaiveBayesClassifier.train(conjunto) · nltk.classify.accuracy(c, prueba) · c.show_most_informative_features(10)
nltk.edit_distance(a, b) · nltk.jaccard_distance(set_a, set_b) · ConfusionMatrix(reales, predichos)
```

---

## 20. Ejercicios propuestos

1. Analice un texto literario en español del Proyecto Gutenberg: frases, palabras más frecuentes (sin palabras vacías), colocaciones y riqueza léxica (palabras distintas / total).
2. Construya un corrector ortográfico para nombres de productos de un catálogo usando `edit_distance`.
3. Entrene un etiquetador con `cess_esp` usando `TrigramTagger` y compare su exactitud con el de bigramas del ejemplo.
4. Entrene un clasificador de sentimiento en español con opiniones etiquetadas y compare Naive Bayes con `SklearnClassifier(LinearSVC())`.
5. Amplíe el ejemplo completo con incidencias escritas a mano (con faltas de ortografía y expresiones coloquiales) y mida cuánto baja la exactitud.

---

## 21. Referencias

- Documentación oficial: https://www.nltk.org/
- Libro *Natural Language Processing with Python* (Bird, Klein y Loper): https://www.nltk.org/book/
- Datos y recursos de NLTK: https://www.nltk.org/data.html
- Lista de corpus disponibles: https://www.nltk.org/nltk_data/
- Repositorio: https://github.com/nltk/nltk
