# Foundation Models

A large model pretrained once on broad data (text, code, images) that then serves as the base for many different tasks — GPT, Claude, Gemini, and Llama are foundation models. The term (Stanford, 2021) names the shift already described in [Pretrained models](from-ml-to-agentic-ai.md#4-pretrained-models--one-base-model-many-uses-2018-2020): from "one model per task" to "one base model, adapted to each task".

| | Task-specific model | Foundation model |
|---|---|---|
| Training data | Small, labeled, for one task | Massive, mostly unlabeled, general |
| Tasks | One | Many, without retraining |
| Training cost | Low | Very high (thousands of GPUs, weeks to months) |
| Adapting to a new task | Train a new model | Prompt, RAG, or light fine-tuning |

## Lifecycle

```
Data → Pretraining → Post-training (SFT → alignment) → Evaluation → Deployment → Monitoring
                                          ↑                                          │
                                          └────────────── feedback ──────────────────┘
```

### 1. Data

Collecting and cleaning the training corpus: deduplicating, filtering low quality or toxic content, removing personal data. Data quality has as much impact on the final model as its size.

### 2. Pretraining

[Self-supervised](data-and-learning-types.md#learning-types) training: predict the next token over trillions of tokens. It's the most expensive phase by far. The result is a **base model** — it knows a lot, but it only continues text, it doesn't follow instructions:

```
Prompt: "What is the capital of France?"

Base model      → "What is the capital of Germany? What is the capital of Italy?"   (continues the pattern)
Instruct model  → "Paris."
```

### 3. Post-training

Turns the base model into an assistant:

- **SFT (Supervised Fine-Tuning)**: training on labeled examples of *prompt → ideal answer*, so the model learns to follow instructions and a conversation format.
- **Alignment**: adjusting the model to human preferences (helpful, honest, harmless).
  - **RLHF** (Reinforcement Learning from Human Feedback): humans rank several answers, a *reward model* learns those preferences, and the LLM is trained with RL to maximize that reward.
  - **DPO** (Direct Preference Optimization): learns directly from pairs of *preferred vs rejected* answers, without a separate reward model — simpler and cheaper.
  - **RLAIF / Constitutional AI**: the feedback comes from another model guided by written principles, instead of from humans.

### 4. Evaluation

Benchmarks, red teaming, and human evaluation before release — see [Evals](evals.md).

### 5. Deployment

Exposed through an API (proprietary models) or published as open weights to run self-hosted. To make it cheaper to serve, it's usually [quantized](model-size-and-quantization.md#quantization-the-trade-off) or *distilled* (see below).

### 6. Monitoring and updates

In production: usage, cost, errors, abuse, and quality drift. The model's knowledge stays frozen at its *cutoff date*, so new versions get trained periodically — production feedback feeds the next round of post-training.

## Adapting a foundation model

From cheapest to most expensive:

| Technique | What changes | When |
|---|---|---|
| [Prompt engineering](prompt-engineering.md) | Only the input | Always first |
| [RAG](rag.md) | The input, with retrieved data | The model lacks private or up-to-date information |
| **PEFT / LoRA** | A small set of extra weights (<1% of the model) | Adjusting style, format, or a narrow domain |
| **Full fine-tuning** | All the weights | A very specific domain, with lots of data and budget |
| **Distillation** | A new, smaller model trained to imitate the large one | Cutting cost/latency while keeping most of the quality |

**LoRA** (Low-Rank Adaptation) freezes the original weights and trains only small matrices added on top — fine-tuning at a fraction of the memory and cost:

```python
from peft import LoraConfig, get_peft_model

config = LoraConfig(r=8, lora_alpha=16, target_modules=["q_proj", "v_proj"])
model = get_peft_model(base_model, config)  # base weights frozen, only the adapters train
model.print_trainable_parameters()          # typically well under 1% of the total
```

---
Related: [From ML to Agentic AI](from-ml-to-agentic-ai.md), [Data and Learning Types](data-and-learning-types.md), [Generative Models](generative-models.md), [Model Size and Quantization](model-size-and-quantization.md), [Evals](evals.md).
