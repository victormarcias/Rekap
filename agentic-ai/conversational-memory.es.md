# Memoria Conversacional

Un LLM no tiene estado entre llamados — cada request a la API es independiente, no "recuerda" nada del anterior. Para que un chatbot mantenga una conversación, la aplicación tiene que reenviar el historial (o una versión de él) en **cada** mensaje nuevo. El problema: ese historial crece con cada turno, hasta chocar contra el [context window](what-is-a-token.es.md#por-qué-importa) del modelo — y cada token reenviado se paga de nuevo (ver [Costos de LLMs](llm-costs.es.md)). Las estrategias de abajo son las formas estándar de administrar eso.

## Buffer completo

Reenviar toda la conversación tal cual, sin recortar nada.

```python
historial = []

def chat(mensaje_usuario):
    historial.append({"role": "user", "content": mensaje_usuario})
    respuesta = llm.call(mensajes=historial)  # se manda TODO el historial de nuevo
    historial.append({"role": "assistant", "content": respuesta})
    return respuesta
```

La más simple, pero no escala: cada turno es más caro que el anterior (mismo problema de costo que un loop de [agent](agents-vs-workflows.es.md#cuándo-usar-cada-uno)), y eventualmente el historial no entra en el context window.

## Sliding window

Quedarse solo con los últimos N mensajes, descartando lo viejo.

```python
VENTANA = 10  # últimos 10 mensajes, sin importar cuántos hubo antes

def chat(mensaje_usuario, historial):
    historial.append({"role": "user", "content": mensaje_usuario})
    contexto = historial[-VENTANA:]  # descarta lo viejo
    respuesta = llm.call(mensajes=contexto)
    historial.append({"role": "assistant", "content": respuesta})
    return respuesta
```

Tamaño acotado y predecible, pero el chatbot pierde por completo lo anterior a la ventana — si el usuario menciona algo en el mensaje 1 y pregunta por eso en el mensaje 15, ya no está en el contexto.

## Token-limited buffer

Misma idea que sliding window, pero el corte es por presupuesto de tokens en vez de cantidad de mensajes — más preciso cuando los mensajes varían mucho de largo (una pregunta de una línea pesa muy distinto en tokens que un mensaje con un stack trace completo).

```python
LIMITE_TOKENS = 3000

def recortar_por_tokens(historial, limite):
    contexto = []
    total = 0
    for m in reversed(historial):  # arranca por lo más reciente
        tokens = contar_tokens(m["content"])
        if total + tokens > limite:
            break
        contexto.insert(0, m)
        total += tokens
    return contexto
```

## Summarization memory

En vez de descartar lo viejo, se lo resume — la próxima llamada usa el resumen, no los mensajes crudos.

```python
def chat(mensaje_usuario, historial, resumen):
    historial.append({"role": "user", "content": mensaje_usuario})

    if len(historial) > UMBRAL:
        resumen = llm.call(f"Resumí esta conversación en pocas líneas: {resumen}\n{historial[:-4]}")
        historial = historial[-4:]  # se queda con los últimos turnos + el resumen de todo lo anterior

    contexto = [{"role": "system", "content": f"Resumen de la conversación hasta ahora: {resumen}"}] + historial
    respuesta = llm.call(mensajes=contexto)
    historial.append({"role": "assistant", "content": respuesta})
    return respuesta
```

No pierde la memoria de lo viejo como el sliding window — la comprime. El costo es un llamado extra al LLM para generar el resumen, y algo de detalle se pierde en la compresión.

## Memoria de largo plazo (vector DB)

Las estrategias anteriores solo cubren la sesión actual. Para que el chatbot recuerde algo de una sesión de hace un mes sin reenviar meses de historial completo, se guardan fragmentos de conversaciones pasadas como embeddings en una [vector DB](../database/nosql.es.md#vector--características-fundamentales), y se trae solo lo relevante a la pregunta actual.

```python
def chat(mensaje_usuario, historial_reciente):
    embedding_pregunta = generar_embedding(mensaje_usuario)
    recuerdos_relevantes = vector_db.search(embedding_pregunta, top_k=3)  # busca en sesiones pasadas, no solo la actual

    contexto = [{"role": "system", "content": f"Datos relevantes de conversaciones previas: {recuerdos_relevantes}"}] + historial_reciente
    respuesta = llm.call(mensajes=contexto)

    guardar_en_vector_db(mensaje_usuario, respuesta)  # esta conversación también queda disponible a futuro
    return respuesta
```

Es el mismo [pipeline de RAG](rag.es.md#el-pipeline-paso-a-paso) (chunking, embeddings, vector DB), aplicado a conversaciones pasadas en vez de una base de documentos.

## Cómo elegir

Buffer completo para prototipos o conversaciones cortas. Sliding window o token-limited buffer cuando el volumen crece pero alcanza con memoria de corto plazo. Summarization cuando importa retener el hilo completo de una conversación larga sin pagar el costo de todo el historial crudo. Vector DB cuando el chatbot necesita recordar entre sesiones distintas, no solo dentro de una misma conversación.

---
Relacionado: [Diseño de Agentes de IA](agent-design.es.md#componentes-centrales), [Qué es un token](what-is-a-token.es.md#por-qué-importa), [RAG](rag.es.md), [Costos de LLMs](llm-costs.es.md), [Context Engineering](context-engineering.es.md).
