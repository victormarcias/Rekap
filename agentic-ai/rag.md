# RAG (Retrieval-Augmented Generation)

Un LLM solo "sabe" lo que vio en su entrenamiento — no conoce datos privados de una empresa, documentación interna, ni nada posterior a su fecha de corte. RAG resuelve esto: **buscar información relevante antes de responder, y agregarla al prompt como contexto**, para que el modelo responda basado en datos reales en vez de inventar (alucinar).

## El pipeline, paso a paso

```
Documentos → Chunking → Embeddings → Vector DB
                                          ↓
Pregunta del usuario → Embedding → Búsqueda por similitud → Top-K chunks relevantes
                                          ↓
                    Prompt final = pregunta + chunks recuperados → LLM → Respuesta
```

### 1. Chunking — partir los documentos

Los documentos fuente (PDFs, docs internos, artículos) se parten en pedazos manejables antes de indexarlos — no se busca sobre el documento entero, se busca sobre chunks chicos.

```python
# chunking simple por tamaño fijo, con overlap para no cortar una idea a la mitad
def chunk_texto(texto, chunk_size=500, overlap=50):
    chunks = []
    for i in range(0, len(texto), chunk_size - overlap):
        chunks.append(texto[i:i + chunk_size])
    return chunks
```

El tamaño del chunk es un trade-off: chunks muy chicos pierden contexto (una oración sola puede no significar nada aislada); chunks muy grandes traen información irrelevante junto con la relevante, y ocupan más [tokens](que-es-un-token.md) en el prompt final.

### 2. Embeddings — texto a vector semántico

Un **embedding** es un vector (una lista de números) que representa el *significado* de un texto — textos con significado parecido quedan cerca en ese espacio de muchas dimensiones, sin importar si comparten las mismas palabras literalmente.

```python
# ejemplo conceptual — un modelo de embeddings convierte texto en un vector
embedding_1 = modelo_embeddings.encode("cómo cancelar mi suscripción")
embedding_2 = modelo_embeddings.encode("quiero dar de baja mi plan")
# estos dos vectores van a quedar MUY cerca entre sí en el espacio vectorial,
# aunque no comparten ninguna palabra — el embedding capturó que significan lo mismo
```

Esto es lo que hace posible la búsqueda semántica: buscar por *significado*, no por coincidencia exacta de palabras (a diferencia de un `LIKE '%texto%'` en SQL, que solo encuentra coincidencias literales).

### 3. Vector DB — guardar y buscar por similitud

Los embeddings de todos los chunks se guardan en una base de datos vectorial (Pinecone, Weaviate, pgvector como extensión de Postgres, entre otras) — optimizada específicamente para responder "¿cuáles de estos millones de vectores están más cerca de este vector de la pregunta?" de forma rápida.

```python
# pseudocódigo del flujo de búsqueda
embedding_pregunta = modelo_embeddings.encode(pregunta_usuario)
chunks_relevantes = vector_db.search(embedding_pregunta, top_k=5)  # los 5 más parecidos
```

### 4. Prompt aumentado

Los chunks recuperados se agregan al prompt como contexto, junto con la pregunta original:

```python
prompt = f"""
Contexto relevante:
{chr(10).join(chunks_relevantes)}

Pregunta: {pregunta_usuario}

Respondé usando solo la información del contexto de arriba.
"""
respuesta = llm.call(prompt)
```

## Por qué no alcanza con meter todo en el prompt

Si los documentos fuente entran enteros en el [context window](que-es-un-token.md#por-qué-importa) del modelo, RAG no haría falta — pero en la práctica, la documentación de una empresa real son miles de páginas, muy por encima de cualquier context window, y aunque entrara, cada request pagaría por procesar todo eso de nuevo (ver [Costos de LLMs](costos-llms.md)) en vez de solo los chunks realmente relevantes a esa pregunta puntual.

## Dónde suele fallar

- **Chunking mal hecho**: si un chunk corta una idea a la mitad, la búsqueda puede no encontrarlo o traerlo sin sentido.
- **Top-K mal calibrado**: muy pocos chunks pierden información relevante; demasiados diluyen el contexto con ruido y suben el costo.
- **La pregunta no se parece semánticamente a la respuesta**: embeddings buscan por similitud de significado, no siempre alineado con qué información responde la pregunta (un problema conocido, mitigado con técnicas como *re-ranking* o *hypothetical document embeddings*).

---
Relacionado: [Qué es un token](que-es-un-token.md), [De ML clásico a Agentic AI](historia-de-ml-a-agentic.md#6-rag--darle-al-llm-información-que-no-tiene-2023), [Costos de LLMs](costos-llms.md).
