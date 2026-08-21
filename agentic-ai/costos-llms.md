# Costos de LLMs: Anthropic vs OpenAI vs Google

**Los precios de este archivo pueden estar desactualizados** — los proveedores sacan modelos nuevos y ajustan precios seguido. Antes de tomar una decisión de presupuesto real, verificar en la fuente oficial: [Anthropic](https://claude.com/pricing), [OpenAI](https://platform.openai.com/docs/pricing), [Google](https://ai.google.dev/gemini-api/docs/pricing). Lo que sí no cambia es la lógica de cómo se cobra — eso es lo que vale la pena entender.

## Cómo se cobra, en general

- **Precio por millón de tokens (MTok)**, separado en **input** (lo que le mandás) y **output** (lo que genera) — el output siempre cuesta varias veces más que el input, porque generar texto token por token es más caro computacionalmente que leerlo.
- Cada proveedor ofrece varios **tiers de modelo**: uno "flagship" (el más capaz, más caro), uno intermedio, y uno rápido/barato para tareas simples — mismo patrón en los tres proveedores, distintos nombres.
- **Prompt caching**: reutilizar contexto ya procesado (un system prompt largo, un documento) cuesta una fracción del precio normal — en vez de pagar el precio completo de input cada vez que mandás el mismo contexto de nuevo.
- **Batch API**: mandar requests que no necesitan respuesta inmediata (se procesan en background, en minutos u horas) a mitad de precio en la mayoría de los proveedores.

## Comparación — precios oficiales en USD por millón de tokens (verificado agosto 2026)

| Proveedor | Modelo (tier) | Input | Output |
|---|---|---|---|
| **Anthropic** | Claude Opus 5 (flagship) | $5 | $25 |
| **Anthropic** | Claude Sonnet 5 (balanceado, el más usado en producción) | $2 | $10 |
| **Anthropic** | Claude Haiku 4.5 (rápido/barato) | $1 | $5 |
| **OpenAI** | GPT-5.5 (flagship) | $5 | $30 |
| **OpenAI** | GPT-5 (balanceado) | $1.25 | $10 |
| **OpenAI** | GPT-5-mini (rápido/barato) | $0.25 | $2 |
| **OpenAI** | GPT-4o-mini (legacy, todavía muy usado por precio) | $0.15 | $0.60 |
| **Google** | Gemini 2.5 Flash (rápido/barato) | $0.30 | $2.50 |

Google no publicó en la fuente consultada un tier "flagship" comparable a Opus/GPT-5.5 con el mismo detalle — si eso importa para una decisión concreta, confirmar en la página oficial de Gemini.

## Prompt caching, con números reales (Anthropic)

```
Cache write (5 min):  1.25x el precio normal de input
Cache write (1 hora): 2x el precio normal de input
Cache read (hit):     0.1x el precio normal de input  → 90% más barato
```

Ejemplo con Claude Opus 5 (input normal $5/MTok): si cacheás un system prompt grande y lo reusás en cada request, la primera vez pagás $6.25/MTok (cache write de 5 min), pero cada request siguiente que lo reusa paga solo $0.50/MTok (cache read) en vez de $5 — se amortiza a partir del segundo uso.

## Batch API (Anthropic, mismo patrón en OpenAI)

50% de descuento en input **y** output, a cambio de que la respuesta no es inmediata (se procesa async). Ejemplo con Claude Sonnet 5: de $2/$10 (input/output normal) a $1/$5 en batch.

## Ejemplo de cálculo real

Caso real documentado por Anthropic: procesar 10.000 tickets de soporte (~3.700 tokens por conversación en promedio) con Claude Haiku 4.5 ($1/MTok input, $5/MTok output) sale **~$37 total** — menos de medio centavo por ticket.

```python
# fórmula genérica para estimar costo de un request
def costo_estimado(input_tokens, output_tokens, precio_input_mtok, precio_output_mtok):
    return (input_tokens / 1_000_000 * precio_input_mtok) + (output_tokens / 1_000_000 * precio_output_mtok)

costo_estimado(50_000, 15_000, precio_input_mtok=5, precio_output_mtok=25)  # Claude Opus 5 → $0.625
```

## Evitar gasto por loops que no cortan solos

Todo lo de arriba asume requests normales — pero en un [agent](agentes-vs-workflows.md#patrón-de-agent-el-llm-controla-el-camino), donde el número de pasos no está fijo, un loop que queda reintentando contra una tool o API caída no solo es un problema de resiliencia, es plata: cada intento fallido consume tokens (el LLM razona, decide reintentar, arma el llamado) sin producir ningún resultado útil. Un [Circuit Breaker](../system-design/atributos-de-calidad.md#tolerancia-a-fallos) corta esos reintentos después de N fallos — ahí el mismo patrón que sirve para no tumbar un sistema en cascada, también sirve como límite de gasto concreto en un agent.

## Más allá de la API: self-hosted / open source

Modelos open-source (Llama, Mistral, entre otros) se pueden self-hostear — ahí el costo deja de ser "por token" y pasa a ser el costo del hardware/GPU que corre el modelo. Tiene sentido a volumen muy alto y sostenido, donde el costo fijo de la infraestructura termina siendo más barato que pagar por token indefinidamente — el mismo trade-off que ya vimos entre [VPS y Cloud Run](../devops/vps-vs-cloud-run.md): pagás infraestructura fija vs pagás por uso real.

---
Relacionado: [De ML clásico a Agentic AI](historia-de-ml-a-agentic.md), [Agentes vs Workflows](agentes-vs-workflows.md), [Circuit Breaker](../system-design/atributos-de-calidad.md#tolerancia-a-fallos), [VPS vs Cloud Run](../devops/vps-vs-cloud-run.md).
