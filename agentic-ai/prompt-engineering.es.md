# Prompt Engineering

Formas de mejorar la salida de un LLM **sin reentrenar el modelo** — todas actúan sobre el prompt, no sobre los pesos del modelo (a diferencia del fine-tuning, que sí los ajusta — ver el final de este archivo).

## Instrucciones claras y contexto

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

## Estructura de un system prompt

Un system prompt de agente conviene ordenarlo de lo más general a lo más específico, para que el modelo tenga el marco antes que el detalle:

1. **Rol y objetivo** (*role prompting*): quién es el agente y cuál es su tarea (`"Sos un asistente de soporte; tu tarea es resolver tickets de nivel 1"`). Acota el espacio de respuestas más que ninguna otra parte del prompt.
2. **Contexto**: lo que el agente necesita saber y no puede inferir — reglas del negocio, formato de los datos que va a recibir, qué queda fuera de alcance.
3. **Instrucciones**: qué hacer y qué no, paso a paso. Acá van los ejemplos [few-shot](#zero-shot-vs-few-shot-learning) si el formato de salida tiene que ser exacto.
4. **Tools**: qué herramientas tiene y **cuándo** usar cada una — exponer la tool no alcanza, hay que decir en qué situación corresponde (ver [Function Calling](function-calling.es.md)).
5. **Variables**: los datos que cambian en cada request (fecha, usuario, historial) entran como placeholders que se rellenan en runtime — si se hardcodean, el prompt queda desactualizado apenas cambia el dato.

```text
# Rol
Sos un asistente de agendamiento para una clínica. Tu tarea es coordinar turnos.

# Contexto
- Horario: lunes a viernes, 9 a 18h. Un turno dura 30 minutos.

# Instrucciones
- Confirmá nombre y motivo de consulta antes de agendar.
- Si el horario pedido no está libre, ofrecé los dos más cercanos.

# Tools
- Calendar_Check → ver disponibilidad. Usala antes de confirmar cualquier turno.
- Calendar_Book → reservar una vez confirmado con el paciente.

# Variables
Fecha y hora actual: {now}
Paciente: {user_name}
```

**Formato**: usar headers Markdown (`#`, `##`) para separar las secciones — el modelo los lee como jerarquía y no mezcla, por ejemplo, contexto con instrucciones.

**Largo**: completo pero no verboso. Cada token del system prompt se paga en **cada** llamado (ver [Costos de LLMs](llm-costs.es.md)), y en un agente el system prompt viaja en todas las vueltas del loop — lo de más se multiplica por iteración.

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

Es, en esencia, la misma lógica del [loop ReAct](agents-vs-workflows.es.md#patrón-de-agent-el-llm-controla-el-camino) — "razonar antes de actuar" — pero aplicada dentro de una sola respuesta, sin necesidad de un loop de tools.

**Dónde va**: al final del prompt, después de las instrucciones. Con modelos que ya razonan de fábrica (OpenAI o1/o3, Claude con *extended thinking*, DeepSeek R1) pedir CoT explícito es redundante — generan una cadena de razonamiento interna antes de responder sin que se lo pidas (ver [test-time compute](what-is-a-token.es.md#test-time-compute--pensar-más-al-responder-no-al-entrenar)).

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

Few-shot suele mejorar la consistencia del formato de salida (útil cuando el output tiene que ser parseable, ej. JSON con una estructura exacta) — el costo es que cada ejemplo agrega [tokens](what-is-a-token.es.md) al prompt, y por ende al costo de cada llamado (ver [Costos de LLMs](llm-costs.es.md)).

## Fine-tuning (para comparar)

A diferencia de todo lo anterior, el fine-tuning sí **ajusta los pesos del modelo** entrenándolo con datos propios — ya lo vimos en la [historia de la evolución hacia LLMs](from-ml-to-agentic-ai.es.md#4-modelos-preentrenados--un-modelo-base-muchos-usos-2018-2020). Es más caro y lento de iterar que ajustar un prompt, pero sirve cuando el comportamiento que necesitás no se logra con ninguna técnica de prompting — ej. un estilo/formato muy específico que hay que repetir consistentemente miles de veces, o conocimiento de dominio que no entra razonablemente en un prompt.

**Regla práctica**: probar primero con prompt engineering + few-shot (rápido, barato, iterable) — recién considerar fine-tuning si eso no alcanza.

---
Relacionado: [Qué es un token](what-is-a-token.es.md), [Function Calling](function-calling.es.md), [Costos de LLMs](llm-costs.es.md), [Diseño de Agentes de IA](agent-design.es.md), [Agentes vs Workflows](agents-vs-workflows.es.md), [De ML clásico a Agentic AI](from-ml-to-agentic-ai.es.md), [Context Engineering](context-engineering.es.md).
