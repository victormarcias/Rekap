# LLM Costs: Anthropic vs OpenAI vs Google

**The prices in this file may be outdated** — providers release new models and adjust prices often. Before making a real budget decision, check the official source: [Anthropic](https://claude.com/pricing), [OpenAI](https://platform.openai.com/docs/pricing), [Google](https://ai.google.dev/gemini-api/docs/pricing). What doesn't change is the logic of how billing works — that's what's worth understanding.

## How billing works, in general

- **Price per million [tokens](what-is-a-token.md) (MTok)**, split into **input** (what you send it) and **output** (what it generates) — output always costs several times more than input, because generating text token by token is more computationally expensive than reading it.
- Each provider offers several **model tiers**: a "flagship" (the most capable, most expensive), a mid tier, and a fast/cheap one for simple tasks — same pattern across all three providers, different names.
- **Prompt caching**: reusing already-processed context (a long system prompt, a document) costs a fraction of the normal price — instead of paying the full input price every time you send the same context again.
- **Batch API**: sending requests that don't need an immediate response (processed in the background, in minutes or hours) at half price with most providers.

For a comparison between concrete models (proprietary and open-source, pros/cons of each) see [Model Comparison](model-comparison.md) — here the focus is the **mechanics** of billing, not exact prices for a specific model (those change too often to keep up to date in a reference file).

## Prompt caching, with real numbers (Anthropic)

```
Cache write (5 min):  1.25x the normal input price
Cache write (1 hour): 2x the normal input price
Cache read (hit):     0.1x the normal input price  → 90% cheaper
```

Example with Claude Opus 5 (normal input $5/MTok): if you cache a large system prompt and reuse it on every request, the first time you pay $6.25/MTok (5-min cache write), but every subsequent request reusing it pays only $0.50/MTok (cache read) instead of $5 — it pays for itself starting from the second use.

## Batch API (Anthropic, same pattern on OpenAI)

50% discount on input **and** output, in exchange for the response not being immediate (processed async). Example with Claude Sonnet 5: from $2/$10 (normal input/output) to $1/$5 in batch.

## Real calculation example

A real case documented by Anthropic: processing 10,000 support tickets (~3,700 tokens per conversation on average) with Claude Haiku 4.5 ($1/MTok input, $5/MTok output) costs **~$37 total** — less than half a cent per ticket.

```python
# generic formula for estimating a request's cost
def estimated_cost(input_tokens, output_tokens, input_price_mtok, output_price_mtok):
    return (input_tokens / 1_000_000 * input_price_mtok) + (output_tokens / 1_000_000 * output_price_mtok)

estimated_cost(50_000, 15_000, input_price_mtok=5, output_price_mtok=25)  # Claude Opus 5 → $0.625
```

## Avoiding spend from loops that don't cut themselves off

Everything above assumes normal requests — but in an [agent](agents-vs-workflows.md#agent-pattern-the-llm-controls-the-path), where the number of steps isn't fixed, a loop that keeps retrying against a down tool or API isn't just a resilience problem, it's money: every failed attempt consumes tokens (the LLM reasons, decides to retry, builds the call) without producing any useful result. A [Circuit Breaker](../system-design/quality-attributes.md#fault-tolerance) cuts off those retries after N failures — the same pattern that keeps a system from cascading down also works as a concrete spending limit in an agent.

## Beyond the API: self-hosted / open source

Open-source models (Llama, Mistral, among others) can be self-hosted — there the cost stops being "per token" and becomes the cost of the hardware/GPU running the model. Makes sense at very high, sustained volume, where the fixed infrastructure cost ends up cheaper than paying per token indefinitely — the same trade-off we already saw between [VPS and Cloud Run](../devops/vps-vs-cloud-run.md): you pay fixed infrastructure vs pay for actual use.

**Ollama** is the simplest way to do this on your own machine (laptop, local server, VPS): runs open-source models locally with a single command, no ML infrastructure to configure.

```bash
ollama run llama3.3      # downloads the model (if missing) and opens a terminal chat
ollama serve              # exposes a local API (OpenAI-format compatible) on localhost:11434
```

$0 cost per token — the limit becomes the hardware (available RAM/VRAM determines what model size fits, see [Model Size and Quantization](model-size-and-quantization.md)), and an open-source model's quality running locally usually falls below a flagship like Opus/GPT-5.5. Makes sense for prototyping without spending on the API, for data that can't leave the machine (privacy), or for high, sustained volume where amortizing the hardware ends up cheaper than paying per token.

### Open-source (self-hosted) vs proprietary LLM (API)

Beyond cost, it's a trade-off with concrete pros and cons on each side:

**Advantages of self-hosting an open-source model:**
- **Data security**: nothing leaves your machine/infrastructure — relevant when the data is sensitive and can't go through a third party's API (see [PII and data protection](risks-and-mitigations.md#data-protection-and-pii)).
- **Free**: no per-token cost, beyond hardware you already have or pay for once.
- **Uncensored models possible**: fine-tunes of open-source models exist explicitly designed to strip the base model's content restrictions (e.g. **Dolphin**, a Llama/Mistral fine-tune) — something a proprietary provider doesn't offer, because its models ship with those restrictions by design and they're not user-configurable.

**Disadvantages:**
- **Lower performance**: an open-source model running locally generally performs below a proprietary flagship (Opus, GPT-5.5) on complex tasks — the gap narrows on simple tasks, but doesn't disappear.
- **Own hardware needed**: requires sufficient GPU/RAM (your own, or rented from a cloud provider) — an infrastructure and maintenance cost the proprietary API avoids entirely.

---
Related: [What is a token](what-is-a-token.md), [From Classical ML to Agentic AI](from-ml-to-agentic-ai.md), [Agents vs Workflows](agents-vs-workflows.md), [Circuit Breaker](../system-design/quality-attributes.md#fault-tolerance), [VPS vs Cloud Run](../devops/vps-vs-cloud-run.md), [Risks and Mitigations](risks-and-mitigations.md), [Model Size and Quantization](model-size-and-quantization.md).
