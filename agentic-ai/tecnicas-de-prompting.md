# Técnicas de Prompting

Formas de mejorar la salida de un LLM **sin reentrenar el modelo** — todas actúan sobre el prompt, no sobre los pesos del modelo (a diferencia del fine-tuning, que sí los ajusta — ver el final de este archivo).

## Prompt Engineering

Diseñar el prompt con intención: instrucciones claras, contexto suficiente, formato de salida especificado — la diferencia entre una respuesta útil y una genérica suele estar más en cómo se pide que en qué modelo se usa.

```python
# ❌ vago: el modelo tiene que adivinar qué formato/nivel de detalle esperás
prompt = "Explicame qué es un índice de base de datos"

# ✅ específico: contexto, audiencia, formato esperado
prompt = """
Explicá qué es un índice de base de datos para alguien que ya sabe SQL básico
pero nunca vio índices. Usá un ejemplo concreto con una tabla de usuarios.
Respondé en 3 párrafos cortos, sin código.
"""
```

## Chain of Thought (CoT)

Pedirle al modelo que **razone paso a paso antes de dar la respuesta final**, en vez de saltar directo a una conclusión — mejora notablemente la precisión en tareas que requieren varios pasos lógicos (matemática, debugging, decisiones con múltiples condiciones).

```python
# sin CoT: el modelo puede saltar a una respuesta sin haber "pensado" los pasos intermedios
prompt = "¿Cuánto sale el envío si el pedido pesa 12kg y cuesta $50 con envío gratis solo arriba de $100?"

# con CoT: se le pide explícitamente que muestre el razonamiento antes de concluir
prompt = """
Resolvé esto paso a paso, mostrando cada cálculo antes de dar la respuesta final:
¿Cuánto sale el envío si el pedido pesa 12kg y cuesta $50 con envío gratis solo arriba de $100?
"""
```

Es, en esencia, la misma lógica del [loop ReAct](agentes-vs-workflows.md#patrón-de-agent-el-llm-controla-el-camino) — "razonar antes de actuar" — pero aplicada dentro de una sola respuesta, sin necesidad de un loop de tools.

## Zero-shot vs Few-shot Learning

- **Zero-shot**: pedirle al modelo que resuelva una tarea **sin ningún ejemplo previo** en el prompt — confía en lo que ya aprendió en su entrenamiento general.
- **Few-shot**: incluir unos pocos ejemplos de input/output deseado dentro del mismo prompt, para que el modelo infiera el patrón exacto que se espera.

```python
# Zero-shot: solo la instrucción
prompt = "Clasificá el sentimiento de este review: 'Llegó roto y tarde'"

# Few-shot: se muestran ejemplos del formato exacto de salida esperado
prompt = """
Clasificá el sentimiento como POSITIVO, NEGATIVO o NEUTRO.

Review: "Excelente calidad, lo recomiendo" → POSITIVO
Review: "Nunca llegó" → NEGATIVO
Review: "Es como cualquier otro" → NEUTRO

Review: "Llegó roto y tarde" →
"""
```

Few-shot suele mejorar la consistencia del formato de salida (útil cuando el output tiene que ser parseable, ej. JSON con una estructura exacta) — el costo es que cada ejemplo agrega [tokens](que-es-un-token.md) al prompt, y por ende al costo de cada llamado (ver [Costos de LLMs](costos-llms.md)).

## Fine-tuning (para comparar)

A diferencia de todo lo anterior, el fine-tuning sí **ajusta los pesos del modelo** entrenándolo con datos propios — ya lo vimos en la [historia de la evolución hacia LLMs](historia-de-ml-a-agentic.md#4-modelos-preentrenados--un-modelo-base-muchos-usos-2018-2020). Es más caro y lento de iterar que ajustar un prompt, pero sirve cuando el comportamiento que necesitás no se logra con ninguna técnica de prompting — ej. un estilo/formato muy específico que hay que repetir consistentemente miles de veces, o conocimiento de dominio que no entra razonablemente en un prompt.

**Regla práctica**: probar primero con prompt engineering + few-shot (rápido, barato, iterable) — recién considerar fine-tuning si eso no alcanza.

---
Relacionado: [Qué es un token](que-es-un-token.md), [Agentes vs Workflows](agentes-vs-workflows.md), [De ML clásico a Agentic AI](historia-de-ml-a-agentic.md).
