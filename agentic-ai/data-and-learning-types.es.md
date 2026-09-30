# Tipos de Datos y de Aprendizaje

El tipo de datos disponible decide qué tipo de aprendizaje es posible — y, con eso, qué tipo de modelo tiene sentido.

## Estructurados vs no estructurados

| | Estructurados | Semiestructurados | No estructurados |
|---|---|---|---|
| Forma | Esquema fijo: filas y columnas | Claves/tags dan estructura, pero los campos pueden variar por registro | Sin esquema predefinido |
| Ejemplos | Tablas SQL, planillas, CSV | JSON, XML, logs, emails (headers + cuerpo) | Texto libre, PDFs, imágenes, audio, video |
| Dónde suelen vivir | [RDBMS](../database/rdbms.es.md) | Document stores [NoSQL](../database/nosql.es.md) | Object storage, data lakes, vector DBs |
| Modelo típico | ML clásico (regresión, árboles de decisión) | Se parsean a estructurados, o se tratan como texto | Deep learning, LLMs, embeddings |

La misma información, en las tres formas:

```python
# estructurado — columnas fijas, todas las filas tienen la misma forma
row = ("A-102", "Ana", 3, 45.90)  # order_id, customer, items, total

# semiestructurado — las claves describen el dato, pero cada registro puede tener campos distintos
doc = {"order_id": "A-102", "customer": {"name": "Ana"}, "tags": ["gift"]}

# no estructurado — sin esquema, el significado está solo en el contenido
text = "Ana pidió 3 artículos por $45.90, envueltos para regalo."
```

La mayoría de los datos de una empresa son no estructurados (documentos, tickets, chats, grabaciones). Antes de los LLMs, usarlos implicaba extraer features a mano; con embeddings y LLMs se pueden buscar y procesar directamente — que es lo que hace posible [RAG](rag.es.md).

## Etiquetados vs no etiquetados

- **Etiquetados** (*labeled*): cada ejemplo viene con la respuesta correcta (email → `spam`). Necesarios para aprendizaje supervisado; caros, porque normalmente una persona tiene que anotar cada ejemplo.
- **No etiquetados** (*unlabeled*): datos crudos sin respuesta. Baratos y abundantes — internet es casi todo datos no etiquetados.

```python
labeled = [
    ("You won a prize, click here!", "spam"),
    ("Meeting moved to 3pm", "not_spam"),
]

unlabeled = ["You won a prize, click here!", "Meeting moved to 3pm"]  # mismos inputs, sin respuesta
```

## Tipos de aprendizaje

| Tipo | Datos que necesita | Qué aprende | Ejemplos |
|---|---|---|---|
| **Supervisado** | Etiquetados | Mapear un input a un output conocido — una categoría (*clasificación*) o un número (*regresión*) | Filtro de spam, predicción de precios |
| **No supervisado** | No etiquetados | Encontrar estructura por su cuenta, sin "respuesta correcta" | Clustering de clientes, detección de anomalías |
| **Semisupervisado** | Pocos etiquetados + muchos sin etiquetar | Usa los datos sin etiquetar para sacarle más jugo a las pocas etiquetas | Clasificar imágenes con 1% etiquetadas |
| **Self-supervised** | No etiquetados, pero las etiquetas salen del propio dato | Predecir una parte oculta del input (próxima palabra, palabra tapada) | Pretraining de LLMs |
| **Reinforcement learning (RL)** | Sin dataset — recompensas de un entorno | Qué acciones maximizan la recompensa en el tiempo | Juegos, robótica, RLHF en LLMs |

El aprendizaje self-supervised es lo que hizo posibles a los LLMs: cualquier texto se vuelve dato de entrenamiento, porque cada próxima palabra es una etiqueta gratis.

```python
words = "the cat sat on the mat".split()
pairs = [(words[:i], words[i]) for i in range(1, len(words))]
# (['the'], 'cat'), (['the', 'cat'], 'sat'), ... — input y etiqueta, sin que nadie anote nada
```

---
Relacionado: [De ML clásico a Agentic AI](from-ml-to-agentic-ai.es.md), [Foundation Models](foundation-models.es.md), [RDBMS](../database/rdbms.es.md), [NoSQL](../database/nosql.es.md), [RAG](rag.es.md).
