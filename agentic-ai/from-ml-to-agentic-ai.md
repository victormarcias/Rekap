# From Classical ML to Agentic AI

How we got from "training a model for one specific task" to "an LLM that plans and executes multi-step tasks on its own" — the context needed to understand why agentic AI is what it is today, not an invention with no history behind it.

## Overview: Classical AI vs Generative AI vs LLMs

Before the timeline, the contrast between the three categories that will come up:

| | Classical AI | Generative AI | LLMs |
|---|---|---|---|
| Approach | Explicit rules and logic | Creative content generation | Text prediction based on large data |
| Examples | Expert systems, decision trees | GANs, VAEs | GPT, PaLM, LLaMA |
| Main use | Classification, structured prediction | Creating images/audio/text | Conversation, summarization, text generation |
| Training | Labeled data, manual rules | Deep neural networks | Pretraining on large corpora |
| Flexibility | Limited, task-specific | High, creative | High, adaptive across tasks without retraining |

LLMs are, technically, a specific case of Generative AI (they generate text) — they're split out in the table because their ability to reason with natural-language instructions is what made everything that follows in this timeline possible (RAG, tool use, agentic behavior).

## 1. Classical Machine Learning — one model, one task

A model is trained on labeled data to learn to solve **one specific task** (classify spam, predict a price, cluster customers). The model doesn't "understand" anything beyond that — a model that predicts house prices is useless for classifying emails, you have to train a new one from scratch.

```python
# example of the classical pattern: training ONE model for ONE specific task
from sklearn.linear_model import LogisticRegression

model = LogisticRegression()
model.fit(X_train, y_train)  # learns from labeled data, specific to this problem
model.predict(X_new)          # this is all it knows how to do — classify spam, in this example
```

## 2. Deep Learning — the model learns its own features (2012+)

Before this, someone had to manually design which characteristics of the data mattered (*feature engineering*). The turning point was **AlexNet** (2012, ImageNet) — a deep neural network that learned features on its own from raw images, far outperforming previous methods. It was still "one model, one task" (vision, text, etc. separately), but features no longer had to be designed by hand.

## 3. Transformers — the base architecture for everything that came after (2017)

The paper *"Attention Is All You Need"* (2017) introduced an architecture that processes entire sequences at once (instead of one word at a time, like previous RNNs), paying "attention" to which parts of the input matter most for each part of the output. It's the architecture behind virtually every current LLM.

## 4. Pretrained models — one base model, many uses (2018-2020)

BERT and GPT (original version) shifted the pattern from "one model per task" to **pretraining one large model once** on massive text (unlabeled, *self-supervised*), then adjusting it (*fine-tuning*) with little effort for specific tasks. General language knowledge stayed in the base model; fine-tuning just specialized it.

## 5. LLMs and the prompting era (2020, mainstream in 2022)

GPT-3 (2020) showed something new: with a large enough model, fine-tuning was no longer needed to solve a new task — **describing it in the prompt** was enough (*few-shot*/*zero-shot learning*). ChatGPT's launch (November 2022) was the moment this went mainstream outside the technical world.

```
Prompt: "Classify this email as spam or not spam: 'You won a prize, click here!'"
→ the same model that writes code, translates, and summarizes now also classifies —
  without having been specifically trained for this, just from the instruction in the prompt
```

## 6. RAG — giving the LLM information it doesn't have (2023+)

An LLM only "knows" what it saw during training — it doesn't know a company's private data, or events after its cutoff date. **Retrieval-Augmented Generation**: before answering, relevant information gets looked up (typically in a vector database) and added to the prompt as context, so the model answers based on that instead of making things up. See [RAG](rag.md) for the full pipeline detail (chunking, embeddings, vector DB).

## 7. Tool Use / Function Calling — the LLM can act, not just talk (2023+)

Up to this point, an LLM only returned text. With *function calling*, the model can decide "to answer this, I need to call this function" (search an API, query a DB, send an email) — the LLM chooses which tool to use and with what arguments, the application's code actually executes it and returns the result. The complete mechanism, with the real back-and-forth message format, is in [Function Calling](function-calling.md).

## 8. Agentic AI — plan, act, observe, repeat (2023-2024+)

The union of everything above into a **loop**: the LLM doesn't respond just once — it plans the necessary steps, executes an action (using tool use), observes the result, decides the next step, and repeats until completing the task or deciding it's done. It's the difference between "asking a chatbot something" and "asking it to solve a problem end to end." This exact pattern is, literally, how this conversation works: every time a repo file gets edited, the result gets reviewed, and the next step gets decided, that's an agent running that loop.

```
User: "Add a new file and link it from the index"
  → Agent plans: 1) create file, 2) edit index, 3) verify nothing broke
  → Agent executes step 1 (tool: write file)
  → Agent observes: did it work? yes → moves to step 2
  → Agent executes step 2 (tool: edit index)
  → Agent observes, verifies, reports it's done
```

---
Related: see the rest of `agentic-ai/` for the detail of each piece (RAG, tool use, agent architecture) as they get added.
