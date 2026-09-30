# Foundation Models

Un modelo grande preentrenado una sola vez sobre datos amplios (texto, código, imágenes) que después sirve de base para muchas tareas distintas — GPT, Claude, Gemini y Llama son foundation models. El término (Stanford, 2021) le pone nombre al cambio que ya se describe en [Modelos preentrenados](from-ml-to-agentic-ai.es.md#4-modelos-preentrenados--un-modelo-base-muchos-usos-2018-2020): de "un modelo por tarea" a "un modelo base, adaptado a cada tarea".

| | Modelo específico | Foundation model |
|---|---|---|
| Datos de entrenamiento | Pocos, etiquetados, para una tarea | Masivos, mayormente sin etiquetar, generales |
| Tareas | Una | Muchas, sin reentrenar |
| Costo de entrenamiento | Bajo | Muy alto (miles de GPUs, semanas a meses) |
| Adaptarlo a una tarea nueva | Entrenar otro modelo | Prompt, RAG o un fine-tuning liviano |

## Ciclo de vida

```
Datos → Pretraining → Post-training (SFT → alignment) → Evaluación → Deploy → Monitoreo
                                          ↑                                      │
                                          └──────────── feedback ────────────────┘
```

### 1. Datos

Juntar y limpiar el corpus de entrenamiento: deduplicar, filtrar contenido de baja calidad o tóxico, sacar datos personales. La calidad de los datos pesa tanto en el modelo final como su tamaño.

### 2. Pretraining

Entrenamiento [self-supervised](data-and-learning-types.es.md#tipos-de-aprendizaje): predecir el próximo token sobre billones de tokens. Es por lejos la fase más cara. El resultado es un **modelo base** — sabe mucho, pero solo continúa texto, no sigue instrucciones:

```
Prompt: "¿Cuál es la capital de Francia?"

Modelo base      → "¿Cuál es la capital de Alemania? ¿Cuál es la capital de Italia?"   (continúa el patrón)
Modelo instruct  → "París."
```

### 3. Post-training

Convierte el modelo base en un asistente:

- **SFT (Supervised Fine-Tuning)**: entrenar con ejemplos etiquetados de *prompt → respuesta ideal*, para que aprenda a seguir instrucciones y un formato de conversación.
- **Alignment**: ajustar el modelo a las preferencias humanas (útil, honesto, inofensivo).
  - **RLHF** (Reinforcement Learning from Human Feedback): personas rankean varias respuestas, un *reward model* aprende esas preferencias, y el LLM se entrena con RL para maximizar esa recompensa.
  - **DPO** (Direct Preference Optimization): aprende directo de pares de respuestas *preferida vs rechazada*, sin reward model aparte — más simple y barato.
  - **RLAIF / Constitutional AI**: el feedback lo da otro modelo guiado por principios escritos, en vez de personas.

### 4. Evaluación

Benchmarks, red teaming y evaluación humana antes de publicarlo — ver [Evals](evals.es.md).

### 5. Deploy

Se expone por API (modelos propietarios) o se publican los pesos (*open weights*) para correrlo self-hosted. Para que sea más barato de servir, se suele [cuantizar](model-size-and-quantization.es.md#cuantización-el-trade-off) o *destilar* (ver abajo).

### 6. Monitoreo y actualizaciones

En producción: uso, costo, errores, abuso y degradación de calidad. El conocimiento del modelo queda congelado en su *fecha de corte*, así que periódicamente se entrenan versiones nuevas — el feedback de producción alimenta la próxima ronda de post-training.

## Adaptar un foundation model

De más barato a más caro:

| Técnica | Qué cambia | Cuándo |
|---|---|---|
| [Prompt engineering](prompt-engineering.es.md) | Solo el input | Siempre primero |
| [RAG](rag.es.md) | El input, con datos recuperados | Al modelo le falta información privada o actualizada |
| **PEFT / LoRA** | Un set chico de pesos extra (<1% del modelo) | Ajustar estilo, formato o un dominio acotado |
| **Fine-tuning completo** | Todos los pesos | Dominio muy específico, con muchos datos y presupuesto |
| **Distillation** | Un modelo nuevo, más chico, entrenado para imitar al grande | Bajar costo/latencia conservando la mayor parte de la calidad |

**LoRA** (Low-Rank Adaptation) congela los pesos originales y entrena solo matrices chicas agregadas encima — fine-tuning con una fracción de la memoria y el costo:

```python
from peft import LoraConfig, get_peft_model

config = LoraConfig(r=8, lora_alpha=16, target_modules=["q_proj", "v_proj"])
model = get_peft_model(base_model, config)  # pesos base congelados, solo entrenan los adapters
model.print_trainable_parameters()          # típicamente bastante menos del 1% del total
```

---
Relacionado: [De ML clásico a Agentic AI](from-ml-to-agentic-ai.es.md), [Tipos de Datos y de Aprendizaje](data-and-learning-types.es.md), [Modelos Generativos](generative-models.es.md), [Tamaño y Cuantización de Modelos](model-size-and-quantization.es.md), [Evals](evals.es.md).
