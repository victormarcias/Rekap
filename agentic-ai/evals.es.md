# Evals

Los evals miden si un LLM o un agente hace lo que se espera — el equivalente a los tests para un sistema cuyo output no es determinístico. Sin ellos, cambiar un prompt o de modelo es una apuesta: algo puede mejorar mientras otra cosa se rompe sin que nadie lo note.

Los unit tests clásicos no alcanzan porque el mismo input puede dar outputs distintos, y "correcto" muchas veces es una cuestión de grado (una respuesta puede ser correcta pero incompleta, o correcta con el tono equivocado).

## Tipos de evals

| Tipo | Cómo califica | Costo | Sirve para |
|---|---|---|---|
| **Benchmarks públicos** | Datasets estándar (MMLU, SWE-bench, GPQA) | Gratis, ya publicados | Comparar modelos en general — no tu caso de uso |
| **Checks por código** | Match exacto, regex, validación de JSON schema, ejecutar el código generado | Muy bajo | Todo lo que tenga respuesta verificable: clasificación, extracción, elección de tool |
| **LLM-as-a-judge** | Otro LLM califica la respuesta contra una rúbrica | Bajo | Respuestas abiertas: resúmenes, tono, fidelidad |
| **Evaluación humana** | Personas califican o comparan respuestas | Alto | Calibrar los otros métodos, calidad subjetiva |

Los benchmarks públicos tienen dos problemas conocidos: *contaminación* (las preguntas del test se filtraron a los datos de entrenamiento) y *saturación* (todos los modelos top sacan casi 100%). Un buen puntaje ahí dice poco de cómo se comporta un modelo con tus datos.

## Golden dataset

Un set de inputs reales con su output esperado (o criterio de evaluación), versionado en el repo y corrido en cada cambio de prompt o de modelo — una suite de tests de regresión:

```python
golden = [
    {"input": "Cancel my order A-102", "expected_tool": "cancel_order"},
    {"input": "Where is my package?", "expected_tool": "track_shipment"},
    {"input": "I want a refund for the broken lamp", "expected_tool": "create_refund"},
]

def run_evals(agent) -> float:
    passed = sum(agent.pick_tool(case["input"]) == case["expected_tool"] for case in golden)
    return passed / len(golden)  # comparar este score entre versiones de prompt/modelo
```

Las fallas reales de producción son la mejor fuente de casos nuevos: cada bug encontrado se convierte en una fila más del dataset.

## LLM-as-a-judge

```python
JUDGE_PROMPT = """Calificá de 1 a 5 qué tan fiel es la respuesta al contexto.
5 = todo lo que dice la respuesta está respaldado por el contexto. 1 = inventa datos.

Contexto: {context}
Respuesta: {answer}

Respondé solo con el número."""
```

Los jueces tienen sesgos: tienden a preferir respuestas más largas, la primera opción que ven, y respuestas de su misma familia de modelos. Ayuda usar una rúbrica concreta, comparar pares en vez de puntajes absolutos, y chequear una muestra contra calificaciones humanas.

## Qué medir

| Sistema | Métricas |
|---|---|
| [RAG](rag.es.md) | Relevancia del retrieval (¿se trajeron los chunks correctos?), fidelidad (¿la respuesta está respaldada por ellos?), relevancia de la respuesta |
| Agentes | Éxito de la tarea, elección correcta de tool, cantidad de pasos, costo y latencia por tarea |
| Cualquier feature con LLM | Cumplimiento de formato, rechazos donde no corresponden, resistencia a [prompt injection](risks-and-mitigations.es.md#riesgos-de-seguridad-y-técnicos) |

## Offline vs online

- **Offline**: el golden dataset corre antes de deployar, idealmente en CI — atrapa regresiones antes de que las vean los usuarios.
- **Online**: en producción — feedback de usuarios (👍/👎), A/B tests entre prompts o modelos, y trazas muestreadas calificadas por un juez. Herramientas como Langfuse, LangSmith o promptfoo cubren ambos.

---
Relacionado: [Foundation Models](foundation-models.es.md#4-evaluación), [RAG](rag.es.md#dónde-suele-fallar), [Riesgos y Mitigaciones](risks-and-mitigations.es.md), [Comparación de Modelos](model-comparison.es.md), [Diseño de Agentes](agent-design.es.md).
