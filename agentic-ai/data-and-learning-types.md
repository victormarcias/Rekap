# Data and Learning Types

The kind of data available decides which kind of learning is possible — and, with it, which kind of model makes sense.

## Structured vs unstructured

| | Structured | Semi-structured | Unstructured |
|---|---|---|---|
| Shape | Fixed schema: rows and columns | Keys/tags give structure, but fields can vary per record | No predefined schema |
| Examples | SQL tables, spreadsheets, CSV | JSON, XML, logs, emails (headers + body) | Free text, PDFs, images, audio, video |
| Where it usually lives | [RDBMS](../database/rdbms.md) | [NoSQL](../database/nosql.md) document stores | Object storage, data lakes, vector DBs |
| Typical model | Classical ML (regression, decision trees) | Parsed into structured, or handled like text | Deep learning, LLMs, embeddings |

The same information, in the three shapes:

```python
# structured — fixed columns, every row has the same shape
row = ("A-102", "Ana", 3, 45.90)  # order_id, customer, items, total

# semi-structured — keys describe the data, but each record can have different fields
doc = {"order_id": "A-102", "customer": {"name": "Ana"}, "tags": ["gift"]}

# unstructured — no schema at all, the meaning is only in the content
text = "Ana ordered 3 items for $45.90, wrapped as a gift."
```

Most of the data a company has is unstructured (documents, tickets, chats, recordings). Before LLMs, using it meant extracting features by hand; with embeddings and LLMs it can be searched and processed directly — which is what makes [RAG](rag.md) possible.

## Labeled vs unlabeled

- **Labeled**: each example comes with the correct answer attached (email → `spam`). Needed for supervised learning; expensive, because a human usually has to annotate each example.
- **Unlabeled**: raw data with no answer attached. Cheap and abundant — the internet is mostly unlabeled data.

```python
labeled = [
    ("You won a prize, click here!", "spam"),
    ("Meeting moved to 3pm", "not_spam"),
]

unlabeled = ["You won a prize, click here!", "Meeting moved to 3pm"]  # same inputs, no answer
```

## Learning types

| Type | Data it needs | What it learns | Examples |
|---|---|---|---|
| **Supervised** | Labeled | Map an input to a known output — a category (*classification*) or a number (*regression*) | Spam filter, price prediction |
| **Unsupervised** | Unlabeled | Find structure on its own, no "correct answer" | Customer clustering, anomaly detection |
| **Semi-supervised** | A few labeled + many unlabeled | Uses the unlabeled data to get more out of the few labels | Image classification with 1% of images labeled |
| **Self-supervised** | Unlabeled, but the labels come from the data itself | Predict a hidden part of the input (next word, masked word) | LLM pretraining |
| **Reinforcement learning (RL)** | No dataset — rewards from an environment | Which actions maximize the reward over time | Games, robotics, RLHF in LLMs |

Self-supervised learning is what made LLMs possible: any text becomes training data, because each next word is a free label.

```python
words = "the cat sat on the mat".split()
pairs = [(words[:i], words[i]) for i in range(1, len(words))]
# (['the'], 'cat'), (['the', 'cat'], 'sat'), ... — input and label, with no human annotating anything
```

---
Related: [From ML to Agentic AI](from-ml-to-agentic-ai.md), [Foundation Models](foundation-models.md), [RDBMS](../database/rdbms.md), [NoSQL](../database/nosql.md), [RAG](rag.md).
