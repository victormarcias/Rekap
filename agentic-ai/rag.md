# RAG (Retrieval-Augmented Generation)

An LLM only "knows" what it saw during training — it doesn't know a company's private data, internal documentation, or anything after its cutoff date. RAG solves this: **look up relevant information before answering, and add it to the prompt as context**, so the model answers based on real data instead of making things up (hallucinating).

## The pipeline, step by step

```
Documents → Chunking → Embeddings → Vector DB
                                          ↓
User question → Embedding → Similarity search → Candidates (top-20) → Reranking → Final Top-K
                                          ↓
                    Final prompt = question + retrieved chunks → LLM → Answer
```

### 1. Chunking — splitting the documents

Source documents (PDFs, internal docs, articles) get split into manageable pieces before indexing — search doesn't happen over the whole document, it happens over small chunks.

```python
# simple fixed-size chunking, with overlap to avoid cutting an idea in half
def chunk_text(text, chunk_size=500, overlap=50):
    chunks = []
    for i in range(0, len(text), chunk_size - overlap):
        chunks.append(text[i:i + chunk_size])
    return chunks
```

Chunk size is a trade-off: chunks too small lose context (a single sentence might mean nothing in isolation); chunks too large bring in irrelevant information along with the relevant, and take up more [tokens](what-is-a-token.md) in the final prompt.

### 2. Embeddings — text to semantic vector

An **embedding** is a vector (a list of numbers) representing a text's *meaning* — texts with similar meaning end up close together in that many-dimensional space, regardless of whether they literally share the same words.

```python
# conceptual example — an embeddings model converts text into a vector
embedding_1 = embeddings_model.encode("how to cancel my subscription")
embedding_2 = embeddings_model.encode("I want to drop my plan")
# these two vectors will end up VERY close to each other in the vector space,
# even though they share no words — the embedding captured that they mean the same thing
```

This is what makes semantic search possible: searching by *meaning*, not by exact word match (unlike a `LIKE '%text%'` in SQL, which only finds literal matches).

### 3. Vector DB — storing and searching by similarity

The embeddings for all the chunks get stored in a vector database (Pinecone, Weaviate, pgvector as a Postgres extension, among others) — specifically optimized to answer "which of these millions of vectors are closest to this question's vector?" quickly.

```python
# search flow pseudocode
question_embedding = embeddings_model.encode(user_question)
relevant_chunks = vector_db.search(question_embedding, top_k=5)  # the 5 most similar
```

### 4. Reranking (optional)

The vector DB search is fast but less precise — an embedding compares the question and each document **separately**, without seeing them together. A **reranker** (a *cross-encoder* model) evaluates the question and one candidate **together, at once**, giving a much finer relevance score — but it's slower, so it isn't used to search across millions of vectors, only to reorder a handful of candidates the vector DB already filtered.

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

# MORE candidates than needed get retrieved (top 20, not directly top 5)
candidates = vector_db.search(question_embedding, top_k=20)

# the reranker evaluates question + document together, pair by pair — more precise than the embedding alone
pairs = [(user_question, chunk) for chunk in candidates]
scores = reranker.predict(pairs)

# sorted by the reranker's score, and only then are the best taken for the final prompt
relevant_chunks = [c for _, c in sorted(zip(scores, candidates), reverse=True)][:5]
```

The general pattern (fast, approximate search first, more expensive and precise filtering after, only on what already survived the first filter) shows up often in large systems — here applied to making a cross-encoder's precision viable without paying its cost across the entire corpus.

### 5. Augmented prompt

The retrieved chunks get added to the prompt as context, along with the original question:

```python
prompt = f"""
Relevant context:
{chr(10).join(relevant_chunks)}

Question: {user_question}

Answer using only the information from the context above.
"""
response = llm.call(prompt)
```

## Why just stuffing everything into the prompt isn't enough

If the source documents fit whole into the model's [context window](what-is-a-token.md#why-it-matters), RAG wouldn't be needed — but in practice, a real company's documentation is thousands of pages, well beyond any context window, and even if it fit, every request would pay to process all of that again (see [LLM Costs](llm-costs.md)) instead of just the chunks actually relevant to that specific question.

## Where it tends to fail

- **Bad chunking**: if a chunk cuts an idea in half, the search might not find it or might bring it back meaningless.
- **Poorly calibrated Top-K**: too few chunks lose relevant information; too many dilute the context with noise and raise the cost.
- **The question doesn't semantically resemble the answer**: embeddings search by similarity of meaning, not always aligned with what information actually answers the question — a known problem, mitigated with [Reranking](#4-reranking-optional) or *hypothetical document embeddings*.

---
Related: [What is a token](what-is-a-token.md), [From Classical ML to Agentic AI](from-ml-to-agentic-ai.md#6-rag--giving-the-llm-information-it-doesnt-have-2023), [LLM Costs](llm-costs.md), [Context Engineering](context-engineering.md).
