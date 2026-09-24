# Context Engineering

Decidir deliberadamente **qué entra en el context window del modelo en cada llamado** — no solo qué le preguntás, sino el system prompt, las tools disponibles, la memoria de la conversación, y los documentos recuperados vía RAG. Es más determinante para la calidad, el costo y la confiabilidad de un agente que el modelo que elijas: un modelo excelente con contexto desordenado o irrelevante rinde peor que uno más chico con contexto bien curado.

## Las piezas que ya arma esta disciplina

Context Engineering no es un concepto aislado — es el paraguas que junta varias piezas que ya están cubiertas por separado:

- **El límite**: [Context window](what-is-a-token.es.md#por-qué-importa) — cuánto contexto puede "ver" el modelo a la vez.
- **Qué mandar del historial**: [Memoria Conversacional](conversational-memory.es.md) — buffer completo, sliding window, summarization, vector DB.
- **Qué traer de fuera**: [RAG](rag.es.md) — retrieval + reranking, para no meter documentos enteros cuando alcanza con los fragmentos relevantes.
- **Qué mandar fijo**: [Estructura de un system prompt](prompt-engineering.es.md#estructura-de-un-system-prompt) — rol, contexto, instrucciones, tools, variables.

## Context Rot

La calidad de las respuestas se degrada a medida que el contexto crece — **incluso sin llegar al límite duro del context window**. No es que el modelo "se llene" y falle de golpe; es gradual: con más tokens en el contexto (sobre todo si hay información irrelevante, repetida o mal organizada), al modelo le cuesta más priorizar qué es relevante para la pregunta actual, y la precisión cae.

**Mitigación**: no es "meter más contexto por las dudas" — es lo opuesto. Mandar solo lo relevante (mismo espíritu que el retrieval de RAG) y limpiar activamente lo que ya no hace falta, en vez de dejar que el contexto crezca sin control.

## Compaction

Cuando una conversación o un agente lleva muchos pasos y el contexto se acerca al límite (o simplemente creció demasiado), en vez de cortar historial a lo bruto —perdiendo información— se lo **resume automáticamente**: la parte vieja se reemplaza por un resumen compacto generado por el propio modelo, preservando lo esencial con muchos menos tokens.

Es la misma idea que la [summarization memory](conversational-memory.es.md#summarization-memory), pero aplicada al contexto completo de un agente en ejecución, no solo al historial de chat — herramientas como Claude Code la corren automáticamente cuando una sesión larga se acerca al límite, sin que el usuario tenga que pedirlo.

## Regla práctica

Contexto relevante y bien organizado gana siempre a contexto grande. Antes de agregar algo al contexto (un documento entero, todo el historial, todas las tools disponibles) conviene preguntarse si hace falta para **ese paso puntual** — cada token de más no es gratis (se paga, ver [Costos de LLMs](llm-costs.es.md)) y puede activamente empeorar la respuesta por context rot, no solo encarecerla.

---
Relacionado: [Qué es un token](what-is-a-token.es.md), [Memoria Conversacional](conversational-memory.es.md), [RAG](rag.es.md), [Prompt Engineering](prompt-engineering.es.md), [Costos de LLMs](llm-costs.es.md).
