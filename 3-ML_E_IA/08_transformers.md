# Manual de uso: Hugging Face Transformers

> **Categoría:** Inteligencia artificial · **Utilidad:** Modelos de lenguaje, NLP e IA generativa
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** Apache 2.0 · **Versión de referencia:** Transformers 5 (backend PyTorch)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [El Hub de modelos](#4-el-hub-de-modelos)
5. [Pipelines: IA en tres líneas](#5-pipelines-ia-en-tres-líneas)
6. [Tokenizadores](#6-tokenizadores)
7. [Modelos: clases `Auto*`](#7-modelos-clases-auto)
8. [Embeddings y búsqueda semántica](#8-embeddings-y-búsqueda-semántica)
9. [Generación de texto con modelos de lenguaje (LLM)](#9-generación-de-texto-con-modelos-de-lenguaje-llm)
10. [Ajuste fino (*fine-tuning*) con `Trainer`](#10-ajuste-fino-fine-tuning-con-trainer)
11. [Ajuste eficiente: LoRA y PEFT](#11-ajuste-eficiente-lora-y-peft)
12. [Visión, audio y modelos multimodales](#12-visión-audio-y-modelos-multimodales)
13. [Rendimiento y memoria: GPU, precisión y cuantización](#13-rendimiento-y-memoria-gpu-precisión-y-cuantización)
14. [Guardar, compartir y desplegar](#14-guardar-compartir-y-desplegar)
15. [Licencias, privacidad y uso responsable](#15-licencias-privacidad-y-uso-responsable)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Transformers** es la librería de Hugging Face que da acceso a **cientos de miles de modelos preentrenados** de lenguaje, visión, audio y multimodales con una API unificada. Permite usar en pocas líneas modelos de clasificación de texto, extracción de entidades, búsqueda semántica, reconocimiento de voz o generación de texto (LLM), y adaptarlos a datos propios mediante ajuste fino.

**Ventajas:**
- Acceso directo al **Hub** de Hugging Face: modelos abiertos de Google, Meta, Mistral, Qwen, Microsoft y miles de organizaciones e investigadores.
- API homogénea (`AutoTokenizer`, `AutoModel…`, `pipeline`) para cualquier arquitectura.
- `Trainer` para ajuste fino con pocas líneas, e integración con PEFT (LoRA), Accelerate y bitsandbytes.
- Modelos multilingües y específicos para el español.
- Documentación extensa y comunidad muy activa.

**Limitaciones:**
- Los modelos grandes requieren GPU con mucha memoria (o cuantización).
- La descarga inicial de modelos puede ocupar desde cientos de MB a decenas de GB.
- Calidad, sesgos y licencias varían mucho entre modelos: hay que evaluarlos.
- La API evoluciona rápido (la versión 5 eliminó varias tareas y el soporte de TensorFlow/JAX).

| Si necesita… | Opción |
|---|---|
| Modelos preentrenados de texto, visión, audio y LLM | **Transformers** |
| Embeddings de frases y búsqueda semántica con una API sencilla | `sentence-transformers` (basada en Transformers) |
| Servir LLM con alto rendimiento | vLLM, Text Generation Inference, llama.cpp, Ollama |
| PLN clásico (tokenización, lematización, entidades) rápido y ligero | spaCy, NLTK |
| Modelos propios desde cero | PyTorch |

---

## 2. Instalación e importación

```bash
pip install transformers torch              # Librería + backend PyTorch
pip install "transformers[torch]" datasets evaluate accelerate    # Entorno completo para ajuste fino
pip install sentencepiece                   # Necesario para algunos tokenizadores
```

> Transformers 5 solo admite **PyTorch** como backend (se eliminó TensorFlow y JAX/Flax). Para usar GPU, instale PyTorch con soporte CUDA (véase el manual de PyTorch).

Comprobar la instalación:

```bash
python -c "import transformers; print(transformers.__version__)"
```

Importaciones habituales:

```python
from transformers import pipeline, AutoTokenizer, AutoModel, AutoModelForSequenceClassification, AutoModelForCausalLM
```

**Caché de modelos:** los modelos descargados se guardan en `~/.cache/huggingface/hub`. Se puede cambiar con la variable de entorno `HF_HOME`. Para trabajar sin conexión, una vez descargados: `HF_HUB_OFFLINE=1`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| *Transformer* | Arquitectura de red neuronal basada en **atención**, base de los modelos de lenguaje modernos. |
| Modelo preentrenado | Modelo entrenado con grandes volúmenes de datos que se reutiliza para otras tareas. |
| *Checkpoint* | Pesos concretos de un modelo en el Hub (p. ej. `bert-base-multilingual-cased`). |
| Tokenizador | Convierte texto en identificadores numéricos (tokens) y viceversa. |
| Token | Unidad de texto del modelo: palabra, fragmento de palabra o carácter. |
| *Pipeline* | Objeto que encapsula preprocesado, modelo y posprocesado para una tarea. |
| Codificador (*encoder*) | Modelos tipo BERT: comprenden texto (clasificación, entidades, embeddings). |
| Decodificador (*decoder*) | Modelos tipo GPT/Llama/Qwen: generan texto (LLM). |
| Codificador-decodificador | Modelos tipo T5/BART: transforman texto (traducción, resumen). |
| *Embedding* | Vector numérico que representa el significado de un texto. |
| Ajuste fino | Entrenar un modelo preentrenado con datos propios para una tarea concreta. |
| Ventana de contexto | Número máximo de tokens que el modelo procesa de una vez. |

---

## 4. El Hub de modelos

El Hub (https://huggingface.co/models) aloja modelos, conjuntos de datos y demostraciones. Cada modelo tiene una **ficha** (*model card*) con su descripción, idiomas, licencia, métricas y limitaciones.

**Cómo elegir un modelo:**

| Criterio | Qué revisar |
|---|---|
| Tarea | Filtro de tarea del Hub (clasificación, NER, generación…) |
| Idioma | Modelos en español o multilingües |
| Tamaño | Número de parámetros: condiciona memoria y velocidad |
| Licencia | Apache 2.0 / MIT permiten uso comercial; otras tienen restricciones |
| Calidad | Métricas en la ficha, descargas, fecha, organización que lo publica |

Modelos de referencia para español:

| Tarea | Ejemplos de modelos |
|---|---|
| Sentimiento | `nlptown/bert-base-multilingual-uncased-sentiment`, `pysentimiento/robertuito-sentiment-analysis` |
| Clasificación sin entrenamiento | `MoritzLaurer/mDeBERTa-v3-base-mnli-xnli` |
| Entidades | `Babelscape/wikineural-multilingual-ner` |
| Embeddings | `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`, `intfloat/multilingual-e5-base` |
| Base para ajuste fino | `dccuchile/bert-base-spanish-wwm-cased`, `xlm-roberta-base` |
| LLM pequeños multilingües | `Qwen/Qwen2.5-0.5B-Instruct`, `Qwen/Qwen2.5-1.5B-Instruct` |

> Los modelos con acceso restringido (*gated*) requieren aceptar la licencia en la web e iniciar sesión: `hf auth login` (o la variable de entorno `HF_TOKEN`).

---

## 5. Pipelines: IA en tres líneas

### 5.1 Uso básico

```python
from transformers import pipeline

clasificador = pipeline("text-classification", model="nlptown/bert-base-multilingual-uncased-sentiment")
clasificador("El producto es excelente y llegó muy rápido.")
# [{'label': '5 stars', 'score': ...}]
```

> Indique siempre el modelo con `model=`: sin él, la *pipeline* usa un modelo por defecto (normalmente en inglés) que puede cambiar entre versiones.

### 5.2 Tareas disponibles (texto)

| Tarea | Uso |
|---|---|
| `text-classification` | Sentimiento, temas, intención, toxicidad |
| `zero-shot-classification` | Clasificar en categorías definidas al momento, sin entrenamiento |
| `token-classification` | Entidades (personas, lugares, organizaciones), etiquetado gramatical |
| `fill-mask` | Completar palabras ocultas |
| `feature-extraction` | Obtener embeddings |
| `text-generation` | Generación de texto y chat con LLM |
| `table-question-answering` | Preguntas sobre tablas |

> En **Transformers 5** se eliminaron las *pipelines* `summarization`, `translation`, `text2text-generation` y `question-answering`. Estas tareas se resuelven ahora con `text-generation` y un modelo de instrucciones (pidiendo «resume», «traduce» o «responde a partir de este texto»), o cargando directamente el modelo con las clases `Auto*`.

### 5.3 Opciones útiles

```python
ner = pipeline("token-classification", model="Babelscape/wikineural-multilingual-ner",
               aggregation_strategy="simple",       # Une los fragmentos de una misma entidad
               device=0)                            # GPU 0 ("cpu", "cuda", "mps" también válidos)

zero_shot = pipeline("zero-shot-classification", model="MoritzLaurer/mDeBERTa-v3-base-mnli-xnli")
zero_shot("Quiero cambiar la dirección de entrega",
          candidate_labels=["envío", "facturación", "devolución"],
          hypothesis_template="Este mensaje trata sobre {}.",    # Plantilla en el idioma del texto
          multi_label=False)

clasificador(lista_de_textos, batch_size=16, truncation=True)   # Procesar muchos textos
```

---

## 6. Tokenizadores

```python
tokenizador = AutoTokenizer.from_pretrained("dccuchile/bert-base-spanish-wwm-cased")

tokenizador.tokenize("Los transformadores revolucionaron el PLN")
# ['Los', 'transform', '##adores', 'revolucion', '##aron', 'el', 'P', '##L', '##N']

lote = tokenizador(["Primera frase.", "Una segunda frase algo más larga."],
                   padding=True,          # Rellenar hasta la más larga del lote
                   truncation=True,       # Cortar a la longitud máxima
                   max_length=128,
                   return_tensors="pt")   # Tensores de PyTorch
lote["input_ids"], lote["attention_mask"]
tokenizador.decode(lote["input_ids"][0], skip_special_tokens=True)
len(tokenizador)                           # Tamaño del vocabulario
```

| Salida | Significado |
|---|---|
| `input_ids` | Identificadores de los tokens |
| `attention_mask` | 1 = token real, 0 = relleno |
| `token_type_ids` | Segmento (en pares de frases, modelos tipo BERT) |

> Use **siempre el tokenizador del mismo modelo**: cada modelo tiene su propio vocabulario.

---

## 7. Modelos: clases `Auto*`

Las clases `Auto*` detectan la arquitectura a partir del nombre del modelo:

| Clase | Salida / tarea |
|---|---|
| `AutoModel` | Representaciones ocultas (embeddings) |
| `AutoModelForSequenceClassification` | Clasificación de textos |
| `AutoModelForTokenClassification` | Clasificación por token (entidades) |
| `AutoModelForCausalLM` | Generación de texto (LLM) |
| `AutoModelForSeq2SeqLM` | Traducción, resumen (T5, BART, mT5) |
| `AutoModelForMaskedLM` | Completar palabras |
| `AutoModelForImageClassification`, `AutoModelForSpeechSeq2Seq`… | Visión, audio |

```python
import torch

nombre = "nlptown/bert-base-multilingual-uncased-sentiment"
tokenizador = AutoTokenizer.from_pretrained(nombre)
modelo = AutoModelForSequenceClassification.from_pretrained(nombre).eval()

with torch.inference_mode():
    entradas = tokenizador(["Me encanta", "No funciona"], padding=True, return_tensors="pt")
    logits = modelo(**entradas).logits
    probabilidades = logits.softmax(dim=-1)
etiquetas = [modelo.config.id2label[i] for i in probabilidades.argmax(dim=-1).tolist()]
```

`modelo.config` contiene la configuración (número de capas, etiquetas `id2label`, longitud máxima…).

---

## 8. Embeddings y búsqueda semántica

Los *embeddings* representan textos como vectores: textos con significado parecido tienen vectores cercanos. Son la base de la búsqueda semántica, la agrupación de textos, la detección de duplicados y los sistemas **RAG** (generación aumentada con recuperación).

```python
import torch.nn.functional as F

nombre = "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
tokenizador = AutoTokenizer.from_pretrained(nombre)
modelo = AutoModel.from_pretrained(nombre).eval()

def codificar(textos):
    lote = tokenizador(textos, padding=True, truncation=True, return_tensors="pt")
    with torch.inference_mode():
        salida = modelo(**lote).last_hidden_state
    mascara = lote["attention_mask"].unsqueeze(-1).float()
    return F.normalize((salida * mascara).sum(1) / mascara.sum(1), dim=1)   # Media + normalización

similitud = codificar(["¿Dónde está mi pedido?"]) @ codificar(["Mi paquete no ha llegado", "Quiero la factura"]).T
```

Con la librería `sentence-transformers` el mismo cálculo es más directo:

```python
from sentence_transformers import SentenceTransformer
modelo = SentenceTransformer("sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2")
vectores = modelo.encode(textos, normalize_embeddings=True)
```

> Para millones de documentos, almacene los vectores en una base de datos vectorial (FAISS, Qdrant, Chroma, pgvector).

---

## 9. Generación de texto con modelos de lenguaje (LLM)

### 9.1 Con *pipeline*

```python
generador = pipeline("text-generation", model="Qwen/Qwen2.5-0.5B-Instruct", dtype="auto", device_map="auto")
mensajes = [
    {"role": "system", "content": "Eres un asistente que responde en español de forma breve."},
    {"role": "user", "content": "Resume en una frase qué es el aprendizaje automático."},
]
respuesta = generador(mensajes, max_new_tokens=100)
print(respuesta[0]["generated_text"][-1]["content"])
```

### 9.2 Con el modelo directamente

```python
nombre = "Qwen/Qwen2.5-0.5B-Instruct"
tokenizador = AutoTokenizer.from_pretrained(nombre)
modelo = AutoModelForCausalLM.from_pretrained(nombre, dtype="auto", device_map="auto")

entrada = tokenizador.apply_chat_template(mensajes, add_generation_prompt=True,
                                          return_tensors="pt", return_dict=True).to(modelo.device)
salida = modelo.generate(**entrada, max_new_tokens=200, do_sample=True, temperature=0.7, top_p=0.9)
texto = tokenizador.decode(salida[0, entrada["input_ids"].shape[1]:], skip_special_tokens=True)
```

> Los modelos de instrucciones (*Instruct*, *Chat*) esperan el formato de conversación de su **plantilla de chat**: use siempre `apply_chat_template` en lugar de construir el texto a mano.

### 9.3 Parámetros de generación

| Parámetro | Efecto |
|---|---|
| `max_new_tokens` | Longitud máxima de la respuesta |
| `do_sample=False` | Decodificación voraz: respuesta determinista |
| `do_sample=True` + `temperature` | Creatividad: 0,2 conservador, 1,0 variado |
| `top_p`, `top_k` | Limitan los tokens candidatos |
| `repetition_penalty` | Penaliza repeticiones (1,1–1,3) |
| `num_beams` | Búsqueda en haz (más coherente, más lenta) |
| `stop_strings` | Cadenas en las que detener la generación (requiere pasar `tokenizer=`) |

### 9.4 Usos habituales

Resumir, traducir, extraer datos estructurados en JSON, clasificar con instrucciones, redactar respuestas, responder preguntas sobre documentos (RAG). Para producción con muchos usuarios, sirva el modelo con **vLLM** o **Text Generation Inference**.

---

## 10. Ajuste fino (*fine-tuning*) con `Trainer`

Adaptar un modelo preentrenado a una tarea propia (por ejemplo, clasificar incidencias en categorías de la empresa) con unos cientos o miles de ejemplos etiquetados:

```python
from datasets import Dataset
from transformers import (AutoTokenizer, AutoModelForSequenceClassification, TrainingArguments, Trainer,
                          DataCollatorWithPadding, EarlyStoppingCallback)

nombre = "dccuchile/bert-base-spanish-wwm-cased"
etiquetas = ["facturación", "envío", "técnico"]
datos = Dataset.from_pandas(df[["texto", "label"]]).train_test_split(test_size=0.2, seed=42)

tokenizador = AutoTokenizer.from_pretrained(nombre)
datos = datos.map(lambda lote: tokenizador(lote["texto"], truncation=True, max_length=256), batched=True)

modelo = AutoModelForSequenceClassification.from_pretrained(
    nombre, num_labels=len(etiquetas),
    id2label=dict(enumerate(etiquetas)), label2id={e: i for i, e in enumerate(etiquetas)})

def metricas(prediccion):
    logits, reales = prediccion
    return {"accuracy": (logits.argmax(-1) == reales).mean()}

argumentos = TrainingArguments(
    output_dir="modelo_incidencias",
    num_train_epochs=4,
    per_device_train_batch_size=16,
    learning_rate=2e-5,
    weight_decay=0.01,
    eval_strategy="epoch",                 # Antes «evaluation_strategy»
    save_strategy="epoch",
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
    bf16=True,                             # GPU compatibles; omitir en CPU
    report_to="none",
)
entrenador = Trainer(model=modelo, args=argumentos, train_dataset=datos["train"], eval_dataset=datos["test"],
                     processing_class=tokenizador,                 # Antes «tokenizer=»
                     data_collator=DataCollatorWithPadding(tokenizador),
                     compute_metrics=metricas,
                     callbacks=[EarlyStoppingCallback(early_stopping_patience=2)])
entrenador.train()
entrenador.evaluate()
entrenador.save_model("modelo_incidencias/final")

pipeline("text-classification", model="modelo_incidencias/final")("No me ha llegado la factura de marzo")
```

> Las columnas deben llamarse `label` (etiqueta) y las que produce el tokenizador. Una tasa de aprendizaje de 2e-5 a 5e-5 y 2–5 épocas suelen bastar para modelos tipo BERT.

---

## 11. Ajuste eficiente: LoRA y PEFT

Ajustar todos los parámetros de un LLM requiere mucha memoria. **LoRA** entrena solo pequeñas matrices añadidas, típicamente menos del 1 % de los parámetros:

```bash
pip install peft
```

```python
from peft import LoraConfig, get_peft_model

configuracion = LoraConfig(r=16, lora_alpha=32, lora_dropout=0.05, target_modules="all-linear",
                           task_type="CAUSAL_LM")
modelo = get_peft_model(AutoModelForCausalLM.from_pretrained(nombre, dtype="auto"), configuracion)
modelo.print_trainable_parameters()      # p. ej. «trainable params: 0.9 % of total»
```

Combinado con cuantización en 4 bits (**QLoRA**), permite ajustar modelos de miles de millones de parámetros en una sola GPU. Para el ajuste de modelos de chat, la librería **TRL** (`SFTTrainer`) simplifica el proceso.

---

## 12. Visión, audio y modelos multimodales

```python
pipeline("image-classification", model="google/vit-base-patch16-224")("foto.jpg")
pipeline("object-detection", model="facebook/detr-resnet-50")("calle.jpg")
pipeline("automatic-speech-recognition", model="openai/whisper-small")("reunion.mp3")
pipeline("zero-shot-image-classification", model="openai/clip-vit-base-patch32")(
    "producto.jpg", candidate_labels=["zapato", "bolso", "camiseta"])
pipeline("image-text-to-text", model="Qwen/Qwen2.5-VL-3B-Instruct")       # Preguntas sobre imágenes
```

> Las tareas de audio requieren `ffmpeg` instalado en el sistema; las de imagen, `Pillow`.

---

## 13. Rendimiento y memoria: GPU, precisión y cuantización

| Técnica | Cómo | Efecto |
|---|---|---|
| GPU | `device=0` / `.to("cuda")` / `device_map="auto"` | 10–100× más rápido |
| Media precisión | `dtype=torch.bfloat16` o `dtype="auto"` | Mitad de memoria |
| Cuantización 8/4 bits | `quantization_config=BitsAndBytesConfig(load_in_4bit=True)` (`pip install bitsandbytes`) | 4–8× menos memoria |
| Procesar por lotes | `batch_size` en *pipelines* | Mayor rendimiento |
| Inferencia sin gradientes | `torch.inference_mode()` | Menos memoria |
| Atención optimizada | `attn_implementation="sdpa"` (por defecto) | Más velocidad |

**Memoria aproximada para cargar un modelo:** parámetros × bytes por parámetro (2 en `bfloat16`, 0,5 en 4 bits). Un modelo de 7 000 millones de parámetros ocupa unos 14 GB en `bfloat16` y unos 4 GB en 4 bits, más la memoria de la generación.

> En Transformers 5, el argumento de precisión es `dtype` (el antiguo `torch_dtype` está en desuso).

---

## 14. Guardar, compartir y desplegar

```python
modelo.save_pretrained("mi_modelo")          # Pesos en formato safetensors + configuración
tokenizador.save_pretrained("mi_modelo")
modelo = AutoModelForSequenceClassification.from_pretrained("mi_modelo")   # Carga local

modelo.push_to_hub("mi-organizacion/clasificador-incidencias", private=True)   # Requiere iniciar sesión
```

Opciones de despliegue: API propia con FastAPI, **Inference Endpoints** de Hugging Face, **Text Generation Inference** o **vLLM** para LLM, exportación a ONNX con **Optimum** para inferencia rápida en CPU.

---

## 15. Licencias, privacidad y uso responsable

- **Licencias:** revise la licencia de cada modelo antes de usarlo en un producto. Algunas prohíben el uso comercial o imponen condiciones (número de usuarios, usos prohibidos).
- **Privacidad:** ejecutar modelos en local evita enviar datos a terceros, un punto relevante para cumplir el RGPD con datos personales.
- **Sesgos:** los modelos reproducen sesgos de sus datos de entrenamiento; evalúelos con datos representativos de su caso de uso.
- **Alucinaciones:** los LLM pueden generar información falsa con apariencia convincente. No los use como fuente de hechos sin verificación, y revise sus respuestas antes de enviarlas a clientes.
- **Seguridad:** cargue modelos de fuentes fiables. Evite `trust_remote_code=True` salvo que haya revisado el código, y prefiera pesos en formato **safetensors**.

---

## 16. Ejemplo práctico completo

**Análisis automático de opiniones de clientes**: a partir de ocho reseñas en español, el programa (1) puntúa el **sentimiento** de 1 a 5 estrellas, (2) asigna un **tema** sin entrenamiento previo mediante clasificación *zero-shot* con una plantilla de hipótesis en español, (3) **extrae entidades** (personas, lugares, organizaciones), (4) construye un **buscador semántico** con *embeddings* para encontrar reseñas parecidas a una consulta, y (5) redacta una **propuesta de respuesta** a la peor reseña con un modelo de lenguaje pequeño usando su plantilla de chat.

Requisitos: `pip install transformers torch sentencepiece`. La primera ejecución descarga unos 3 GB de modelos.

```python
"""
Análisis automático de opiniones de clientes con Hugging Face Transformers.

Para un conjunto de reseñas en español:
  1. Puntúa el sentimiento (1 a 5 estrellas) con un modelo multilingüe.
  2. Clasifica el tema sin entrenamiento previo (zero-shot).
  3. Extrae entidades (personas, lugares, organizaciones).
  4. Busca reseñas similares por significado con embeddings.
  5. Redacta una respuesta al cliente con un modelo de lenguaje pequeño.
Todos los modelos se descargan del Hub la primera vez y quedan en caché.
"""
import torch
import torch.nn.functional as F
from transformers import AutoModel, AutoModelForCausalLM, AutoTokenizer, pipeline

MODELOS = {
    "sentimiento": "nlptown/bert-base-multilingual-uncased-sentiment",
    "zero_shot": "MoritzLaurer/mDeBERTa-v3-base-mnli-xnli",
    "entidades": "Babelscape/wikineural-multilingual-ner",
    "embeddings": "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2",
    "generacion": "Qwen/Qwen2.5-0.5B-Instruct",
}
DISPOSITIVO = "cuda" if torch.cuda.is_available() else "cpu"
TEMAS = ["entrega y envío", "calidad del producto", "atención al cliente", "precio", "devoluciones"]

RESENAS = [
    "El pedido llegó a Sevilla con tres días de retraso y la caja estaba rota.",
    "Laura, del servicio de atención al cliente, me resolvió el problema en cinco minutos. ¡Genial!",
    "La cafetera funciona perfectamente, buena relación calidad-precio.",
    "Pedí la devolución hace dos semanas y Amazon todavía no me ha reembolsado el dinero.",
    "Demasiado caro para lo que ofrece; en Carrefour lo tienen más barato.",
    "El paquete no llegó nunca y nadie contesta al teléfono de la oficina de Madrid.",
    "Producto de mala calidad: se rompió a la semana de uso.",
    "Envío rapidísimo, lo recibí al día siguiente en Valencia.",
]


# ------------------------------------------------------------------ 1-3. Pipelines
def analizar_resenas(resenas: list[str]) -> list[dict]:
    sentimiento = pipeline("text-classification", model=MODELOS["sentimiento"], device=DISPOSITIVO)
    clasificador = pipeline("zero-shot-classification", model=MODELOS["zero_shot"], device=DISPOSITIVO)
    ner = pipeline("token-classification", model=MODELOS["entidades"],
                   aggregation_strategy="simple", device=DISPOSITIVO)

    resultados = []
    for texto, sent, tema, entidades in zip(
            resenas,
            sentimiento(resenas, batch_size=8, truncation=True),
            clasificador(resenas, candidate_labels=TEMAS,
                         hypothesis_template="Esta opinión trata sobre {}."),
            ner(resenas)):
        resultados.append({
            "texto": texto,
            "estrellas": int(sent["label"].split()[0]),        # «4 stars» → 4
            "confianza": sent["score"],
            "tema": tema["labels"][0],
            "confianza_tema": tema["scores"][0],
            "entidades": [(e["word"], e["entity_group"]) for e in entidades if e["score"] > 0.6],
        })
    return resultados


# ------------------------------------------------------------------ 4. Embeddings y búsqueda semántica
class BuscadorSemantico:
    def __init__(self, nombre_modelo: str):
        self.tokenizador = AutoTokenizer.from_pretrained(nombre_modelo)
        self.modelo = AutoModel.from_pretrained(nombre_modelo).to(DISPOSITIVO).eval()

    @torch.inference_mode()
    def codificar(self, textos: list[str]) -> torch.Tensor:
        lote = self.tokenizador(textos, padding=True, truncation=True, max_length=256,
                                return_tensors="pt").to(DISPOSITIVO)
        salida = self.modelo(**lote).last_hidden_state                # (lote, tokens, dimensión)
        mascara = lote["attention_mask"].unsqueeze(-1).float()
        media = (salida * mascara).sum(1) / mascara.sum(1)            # Media de los tokens reales
        return F.normalize(media, dim=1)                              # Norma 1: producto = coseno

    def buscar(self, consulta: str, documentos: list[str], k: int = 3) -> list[tuple[float, str]]:
        similitudes = self.codificar([consulta]) @ self.codificar(documentos).T
        valores, indices = similitudes[0].topk(min(k, len(documentos)))
        return [(v.item(), documentos[i]) for v, i in zip(valores, indices)]


# ------------------------------------------------------------------ 5. Generación de respuestas
class Redactor:
    def __init__(self, nombre_modelo: str):
        self.tokenizador = AutoTokenizer.from_pretrained(nombre_modelo)
        self.modelo = AutoModelForCausalLM.from_pretrained(nombre_modelo, dtype="auto").to(DISPOSITIVO).eval()

    @torch.inference_mode()
    def responder(self, resena: str, tema: str, max_tokens: int = 120) -> str:
        mensajes = [
            {"role": "system", "content": "Eres el equipo de atención al cliente de una tienda online. "
                                          "Responde en español, con tono cordial y en un máximo de tres frases."},
            {"role": "user", "content": f"Tema: {tema}.\nOpinión del cliente: «{resena}»\nRedacta la respuesta."},
        ]
        entrada = self.tokenizador.apply_chat_template(mensajes, add_generation_prompt=True,
                                                       return_tensors="pt", return_dict=True).to(DISPOSITIVO)
        salida = self.modelo.generate(**entrada, max_new_tokens=max_tokens, do_sample=False,
                                      repetition_penalty=1.1,
                                      pad_token_id=self.tokenizador.eos_token_id)
        nuevos = salida[0, entrada["input_ids"].shape[1]:]                # Solo los tokens generados
        return self.tokenizador.decode(nuevos, skip_special_tokens=True).strip()


def main() -> None:
    print(f"Dispositivo: {DISPOSITIVO}\n")
    resultados = analizar_resenas(RESENAS)
    print(f"{'★':<3}{'Tema':<22}{'Entidades':<34}Reseña")
    for r in resultados:
        entidades = ", ".join(f"{p} ({t})" for p, t in r["entidades"]) or "—"
        print(f"{r['estrellas']:<3}{r['tema']:<22}{entidades[:32]:<34}{r['texto'][:60]}")

    media = sum(r["estrellas"] for r in resultados) / len(resultados)
    print(f"\nPuntuación media: {media:.1f} ★ · reseñas negativas (≤ 2 ★): "
          f"{sum(r['estrellas'] <= 2 for r in resultados)} de {len(resultados)}")

    buscador = BuscadorSemantico(MODELOS["embeddings"])
    consulta = "no me han devuelto el dinero"
    print(f"\nReseñas más parecidas a «{consulta}»:")
    for similitud, texto in buscador.buscar(consulta, RESENAS):
        print(f"  {similitud:.2f}  {texto}")

    peor = min(resultados, key=lambda r: (r["estrellas"], -r["confianza"]))
    redactor = Redactor(MODELOS["generacion"])
    print(f"\nRespuesta propuesta a: «{peor['texto']}»\n")
    print(redactor.responder(peor["texto"], peor["tema"]))


if __name__ == "__main__":
    main()
```

**Salida del programa:**

1. Una tabla con, para cada reseña, las estrellas estimadas, el tema asignado y las entidades encontradas (por ejemplo, «Sevilla (LOC)», «Laura (PER)», «Amazon (ORG)»).
2. La puntuación media y el número de reseñas negativas.
3. Las tres reseñas más parecidas a «no me han devuelto el dinero», con su similitud de coseno (entre 0 y 1). La reseña sobre el reembolso pendiente debería aparecer en primer lugar aunque no comparta casi palabras con la consulta.
4. El texto de una respuesta cordial a la reseña con peor puntuación.

> Las puntuaciones exactas dependen de la versión de cada modelo. El modelo de sentimiento se entrenó con reseñas de productos en seis idiomas; el de generación es deliberadamente pequeño (0,5 B parámetros) para funcionar en CPU, por lo que sus respuestas son correctas pero sencillas: para producción, use un modelo mayor y revise siempre el texto antes de enviarlo.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `OSError: ... is not a local folder and is not a valid model identifier` | Nombre de modelo mal escrito, sin conexión o modelo privado. | Revise el nombre en el Hub, la conexión o inicie sesión (`hf auth login`). |
| `KeyError: 'The task summarization has been removed from transformers'` (también `translation`, `question-answering`…) | Tarea eliminada en Transformers 5. | Use `text-generation` con un modelo de instrucciones o las clases `Auto*`. |
| `RuntimeError: CUDA out of memory` | Modelo o lote demasiado grandes. | `dtype=torch.bfloat16`, cuantización en 4 bits, lotes más pequeños. |
| `Token indices sequence length is longer than the specified maximum` | Texto más largo que la ventana del modelo. | `truncation=True` o divida el texto en fragmentos. |
| Resultados absurdos o en inglés | *Pipeline* sin `model=` (modelo por defecto en inglés). | Indique un modelo multilingüe o en español. |
| El LLM no sigue instrucciones o repite el *prompt* | No se usó la plantilla de chat o se decodificó todo el texto. | `apply_chat_template(..., add_generation_prompt=True)` y decodifique solo los tokens nuevos. |
| `TypeError: __init__() got an unexpected keyword argument 'evaluation_strategy'` | Nombre antiguo del argumento. | Use `eval_strategy`. |
| Aviso sobre `torch_dtype` | Argumento en desuso en Transformers 5. | Use `dtype=`. |
| `ValueError: Asking to pad but the tokenizer does not have a padding token` | Algunos LLM no definen *pad*. | `tokenizador.pad_token = tokenizador.eos_token`. |
| Aviso «Some weights ... were not initialized» al ajustar | Se crea una nueva capa de clasificación. | Normal al hacer ajuste fino: esa capa se entrena con sus datos. |
| Descarga lenta o repetida | Caché en otra ubicación o borrada. | Configure `HF_HOME`; use `HF_HUB_OFFLINE=1` una vez descargado. |

---

## 18. Buenas prácticas

- Indique **siempre el modelo** explícitamente y fije su versión (`revision=`) en producción.
- Elija modelos **en español o multilingües** y compruebe su licencia.
- Empiece con *pipelines* y modelos preentrenados; recurra al ajuste fino solo si el rendimiento no es suficiente.
- **Evalúe** con un conjunto de ejemplos propios etiquetados antes de usar un modelo en un proceso real.
- Use el **modelo más pequeño** que cumpla los requisitos: es más rápido, barato y fácil de desplegar.
- Procese **por lotes** y use GPU, media precisión o cuantización para volúmenes grandes.
- Con LLM: plantilla de chat, instrucciones claras, temperatura baja para tareas objetivas y **revisión humana** de lo que se envía a clientes.
- Mantenga los datos sensibles en infraestructura propia cuando la normativa lo exija.
- Guarde los modelos en **safetensors** y documente versión, datos de ajuste y métricas.

---

## 19. Referencia rápida

```python
from transformers import pipeline, AutoTokenizer, AutoModel, AutoModelForSequenceClassification, AutoModelForCausalLM

pipeline("text-classification" | "zero-shot-classification" | "token-classification" | "feature-extraction"
         | "fill-mask" | "text-generation" | "automatic-speech-recognition" | "image-classification",
         model="org/modelo", device=0, dtype="auto")
p(textos, batch_size=16, truncation=True) · zero_shot(t, candidate_labels=[...], hypothesis_template="... {}.")

tok = AutoTokenizer.from_pretrained(n); tok(textos, padding=True, truncation=True, return_tensors="pt"); tok.decode(ids)
m = AutoModelForSequenceClassification.from_pretrained(n); m(**lote).logits.softmax(-1); m.config.id2label
m = AutoModelForCausalLM.from_pretrained(n, dtype="auto", device_map="auto")
tok.apply_chat_template(mensajes, add_generation_prompt=True, return_tensors="pt", return_dict=True)
m.generate(**entrada, max_new_tokens=200, do_sample=False | True, temperature=0.7, top_p=0.9)

TrainingArguments(output_dir, num_train_epochs, learning_rate=2e-5, eval_strategy="epoch", load_best_model_at_end=True)
Trainer(model, args, train_dataset, eval_dataset, processing_class=tok, compute_metrics=f).train()
m.save_pretrained(dir) · tok.save_pretrained(dir) · m.push_to_hub("org/nombre") · HF_HOME · HF_HUB_OFFLINE=1
```

---

## 20. Ejercicios propuestos

1. Compare dos modelos de sentimiento en español con 50 reseñas etiquetadas a mano y calcule su exactitud.
2. Construya un buscador semántico sobre las preguntas frecuentes de una web y devuelva la respuesta más parecida a la consulta del usuario.
3. Ajuste un modelo BERT en español para clasificar incidencias de soporte en tres categorías con `Trainer` y compare con la clasificación *zero-shot*.
4. Use un LLM pequeño para extraer de facturas en texto los campos «proveedor», «fecha» e «importe» en formato JSON, y valide el resultado con `json.loads`.
5. Implemente un sistema RAG mínimo: recupere con *embeddings* los tres fragmentos más relevantes de un documento y páselos al LLM para responder preguntas sobre él.

---

## 21. Referencias

- Documentación oficial: https://huggingface.co/docs/transformers
- Hub de modelos: https://huggingface.co/models
- Curso de Hugging Face (LLM y PLN): https://huggingface.co/learn
- Guía de migración a Transformers 5: https://github.com/huggingface/transformers/releases
- PEFT: https://huggingface.co/docs/peft · TRL: https://huggingface.co/docs/trl
