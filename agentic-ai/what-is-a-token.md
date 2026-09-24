# What is a token

The basic unit an LLM processes — **it's not a word, or a syllable, and it has nothing to do with meaning**. It's a piece of text (a subword) that the *tokenizer* decided to treat as a single unit, based on how often that piece appears in the data the model was trained on.

## It's not semantic — it's statistical

The process that splits text into tokens (tokenization, typically something like *Byte-Pair Encoding*) knows nothing about grammar or meaning — it repeatedly merges the pairs of characters that most often appear together in the training corpus, until it forms a vocabulary of frequent "pieces." The result: common words tend to be a single token, rare or long words get split into several pieces that don't necessarily match real morphemes.

```
"the" → 1 token (appears constantly, is its own token)
"unbelievable" → probably 3 tokens: "un" + "believ" + "able"
                  (roughly matches morphemes here, but that's a coincidence,
                   it's not that the tokenizer "understands" "un-" is a prefix)
```

## ~4 characters per token, as a quick rule

In English, the typical approximation is ~4 characters or ~0.75 words per token. **In Spanish and other languages it's usually less efficient** — tokenizers are trained with more English data, so a Spanish text can need more tokens to say the same thing as its English equivalent. It's the concrete reason behind something we've already mentioned: writing in English generally uses fewer tokens than the same content in Spanish.

## How the model picks the next token

Generating text is, at bottom, predicting **one token at a time**: given all the previous text, the model computes how likely each possible token in the vocabulary is as the "next" one, and picks one — then repeats the process with that token now included in the context, one by one, until it's done.

```
Prompt: "The sun is..."

The model doesn't "know" the answer — it computes a probability (logit) for
each candidate word in the vocabulary, and normalizes them with softmax
so they add up to 100%:

  "yellow"  → 0.5  (50% probability)
  "red"     → 0.3  (30%)
  "bright"  → 0.2  (20%)

With low temperature, it almost always picks the most likely one ("yellow").
With high temperature, there's more chance it picks a less obvious option.
```

**Softmax** is the function that converts those raw scores (*logits*) into probabilities that add up to 1 — so the model can "choose" based on that distribution instead of comparing arbitrary numbers with no common scale.

**Temperature** (typically 0 to 2) controls how deterministic or creative that choice is:
- **Low temperature (near 0)**: almost always picks the most likely token — more consistent, predictable responses, ideal for tasks where you don't want variation (classification, data extraction).
- **High temperature (near 2)**: more likely to pick less obvious tokens — more varied/creative responses, but also more erratic.

## Test-Time Compute — thinking more when answering, not when training

Up to this point, scaling an LLM meant training it with more parameters and more data — that compute gets spent **once**, during training, and afterward the model responds at the same speed regardless of whether the question is trivial or very hard. **Test-Time Compute** (or *inference-time scaling*) is the reverse idea: letting the model spend more compute **at the moment of answering**, generating longer reasoning before giving the final answer, when the task calls for it.

It's the technique behind "reasoning" models (OpenAI o1/o3, Claude with *extended thinking*): instead of going straight to the answer, the model generates an extensive internal reasoning chain before responding — the same underlying idea as asking for [Chain of Thought](prompt-engineering.md#chain-of-thought-cot) in the prompt, but automatic, much longer, and specifically trained for it (instead of depending on the user asking for it).

```
Simple question: "Capital of France?"
  → little test-time compute needed, answers almost directly

Complex question: "Prove that the sum of the first n odd numbers is n²"
  → the model "thinks" more before answering: explores several reasoning
    paths, discards some, converges on one, only then answers
```

**The trade-off**: more "thinking" tokens = more [cost](llm-costs.md) and more latency — no point spending it on trivial questions. The real advantage shows up in multi-step logical tasks (math, debugging, planning), where reasoning more before answering changes the outcome.

## Multimodal — tokens beyond text

Multimodal models tokenize and process more than text — audio and images also get converted into some kind of "token" the model can reason about in the same space as text (an image gets split into patches that function as tokens, for example). The underlying mechanism (converting input into discrete units, predicting the output token by token) is the same, only what each token represents changes.

## Why it matters

- **Price**: providers charge per token, not per character or word — see [LLM Costs](llm-costs.md).
- **Context window**: the limit on how much text a model can "see" at once is measured in tokens, not words — a 200K-token context window isn't 200K words, it's quite a bit fewer (depending on language and content).
- **Why an LLM sometimes "cuts weird" on a rare word or a proper name**: if that word never appeared often during training, the tokenizer splits it into several unintuitive pieces — more common with proper names, very specific technical jargon, or text in a language poorly represented in training.

---
Related: [LLM Costs](llm-costs.md), [From Classical ML to Agentic AI](from-ml-to-agentic-ai.md), [Prompt Engineering](prompt-engineering.md), [Context Engineering](context-engineering.md).
