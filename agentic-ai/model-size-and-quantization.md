# Model Size: Parameters, Quantization, and Memory

## The base unit: parameters

A model is measured in **parameters** — the "weights" it learned during training (7B = 7 billion). More parameters generally means more capacity, but also heavier and slower to run.

## Dense vs MoE (Mixture of Experts)

So far we've assumed a model uses **all** its parameters to process every token — that's a **dense** model. A **MoE** model splits the network into several "experts" (sub-networks), and a *routing* mechanism picks, per token, only a small subset of those experts to process it — not all of them.

```
70B dense model:  every token passes through all 70B full parameters

397B MoE model (activates ~35B per token):
  → 397B = TOTAL parameters (all experts combined)
  → ~35B = ACTIVE parameters (the ones actually processing that specific token)
```

That's why you'll see notation like "397B MoE (~35B active)" — those are two different numbers, not one.

**The common mistake**: thinking a MoE model "weighs" or uses memory as if it were the size of its active parameters. That's not how it works — **the memory needed to load it depends on the TOTAL parameters**, not the active ones, because you don't know ahead of time which experts each token will need (it can vary token to token) — so they all have to be loaded in memory, even if only some get used at a time. What MoE saves is **compute per token** (faster, cheaper to run), not memory.

The advantage of this approach: large capacity (many total parameters = the model "knows" more) without paying the full compute cost on every token — which is why MoE models like [Ornith](model-comparison.md) or [Grok](model-comparison.md) perform close to much larger dense models, at a fraction of the per-token cost.

## Precision: how much each parameter weighs

Parameters aren't always stored the same way — the **numeric precision** they're stored at determines how much a model takes up on disk, independent of how many parameters it has.

```
disk size (GB) ≈ (parameters × bytes per parameter) / 1,000,000,000
```

| Precision | Bytes/parameter | A 7B model weighs approx. | Quality |
|---|---|---|---|
| FP32 (full precision) | 4 bytes | ~28 GB | Maximum — rarely distributed this way |
| FP16 / BF16 (half precision) | 2 bytes | ~14 GB | The one used for training and serving in the cloud |
| INT8 | 1 byte | ~7 GB | Minimal quality loss |
| **Q4 (4-bit)** — Ollama's default | ~0.5–0.6 bytes | ~4–5 GB | Noticeable loss, but acceptable for most uses |
| Q2 (2-bit) | ~0.25–0.3 bytes | ~2–3 GB | Significant degradation — almost never worth it |

## Quantization: the trade-off

**Quantizing** is reducing the numeric precision of an already-trained model's weights, so it takes up less space and runs faster — in exchange for losing some response quality. It's why the same model can weigh very different amounts depending on which version you download: Llama 3.3 70B weighs ~140GB in FP16, but ~40GB in Q4 — the same model, different compression.

## Real RAM/VRAM when running ≠ just the disk size

When loading the model, the memory used starts out roughly equal to the weights' disk size — but during generation, the **KV cache** adds on: extra memory that grows with the length of the context being processed in that specific conversation. With short contexts it's a small margin over the base size; with long contexts (or several conversations running in parallel) it can add up to quite a bit more.

```
Total RAM/VRAM ≈ disk size of the (quantized) weights + KV cache (grows with context)
```

## Comparison table — typical hardware by model scale (Q4 quantized)

| Model (parameters) | Disk size (Q4) | Minimum recommended RAM/VRAM | Typical hardware |
|---|---|---|---|
| 3B – 8B | ~2–5 GB | 8 GB | Laptop or home PC, no dedicated GPU |
| 13B – 14B | ~7–9 GB | 16 GB | PC with a mid-range dedicated GPU |
| 30B – 34B | ~18–20 GB | 24–32 GB | High-end dedicated GPU (e.g. RTX 4090) |
| 70B | ~38–40 GB | 48–64 GB | Multiple GPUs or a dedicated server |
| 400B+ | ~200 GB+ | Multiple datacenter GPUs | Beyond the reach of personal hardware |

These are orders of magnitude, not exact numbers — they vary depending on the specific quantization implementation (Q4_0, Q4_K_M, etc.) and the context length used. For a **MoE** model, this table should be read by its **total** parameters, not the active ones (see above) — a "397B MoE" takes up memory like a 397B, even though it processes each token with much less compute.

## Groq and LPUs — specialized hardware for speed

Everything above (RAM/VRAM, disk size) assumes the standard hardware for running LLMs: the **GPU**. **Groq** (with a Q, not to be confused with [Grok](model-comparison.md) from xAI — completely different things that just share a similar name) is a company that makes a different chip: the **LPU** (Language Processing Unit), designed specifically for LLM **inference** — not for training, only for serving responses from an already-trained model, as fast as possible.

The core technical difference: a GPU uses HBM memory (large capacity, but slower); an LPU uses **on-chip SRAM** (much faster, but smaller capacity per chip) — that trade-off is what lets an LPU generate tokens noticeably faster than an equivalent GPU, at the cost of needing more chips in parallel for large models (because each one has less memory).

It's not something you install on a laptop — it's datacenter infrastructure (GroqCloud, or the same design licensed by NVIDIA starting in 2026). We mention it here because it's the other variable, besides the model's size and its quantization, that determines how fast an LLM responds in production: not just "does it fit in memory?" but also "what chip does it run on?"

## Why it matters

This is the fine print behind "self-hosting a model for free" (see [self-hosted / open source](llm-costs.md#beyond-the-api-self-hosted--open-source)): the cost isn't zero, it translates directly into what hardware you need — and that depends on two concrete decisions: what model size you choose, and at what quantization level you run it.

---
Related: [LLM Costs](llm-costs.md), [Model Comparison](model-comparison.md), [What is a token](what-is-a-token.md), [From Classical ML to Agentic AI](from-ml-to-agentic-ai.md#3-transformers--the-base-architecture-for-everything-that-came-after-2017).
