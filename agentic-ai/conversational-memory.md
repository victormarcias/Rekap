# Conversational Memory

An LLM has no state between calls — every API request is independent, it doesn't "remember" anything from the previous one. For a chatbot to hold a conversation, the application has to resend the history (or a version of it) on **every** new message. The problem: that history grows with every turn, until it hits the model's [context window](what-is-a-token.md#why-it-matters) — and every resent token gets paid for again (see [LLM Costs](llm-costs.md)). The strategies below are the standard ways of managing that.

## Full buffer

Resending the entire conversation as-is, with nothing trimmed.

```python
history = []

def chat(user_message):
    history.append({"role": "user", "content": user_message})
    response = llm.call(messages=history)  # the WHOLE history gets sent again
    history.append({"role": "assistant", "content": response})
    return response
```

The simplest, but doesn't scale: every turn is more expensive than the last (the same cost problem as an [agent](agents-vs-workflows.md#when-to-use-each-one) loop), and eventually the history doesn't fit in the context window.

## Sliding window

Keeping only the last N messages, discarding the old ones.

```python
WINDOW = 10  # last 10 messages, regardless of how many came before

def chat(user_message, history):
    history.append({"role": "user", "content": user_message})
    context = history[-WINDOW:]  # discards the old stuff
    response = llm.call(messages=context)
    history.append({"role": "assistant", "content": response})
    return response
```

Bounded, predictable size, but the chatbot completely loses anything before the window — if the user mentions something in message 1 and asks about it in message 15, it's no longer in the context.

## Token-limited buffer

Same idea as a sliding window, but the cutoff is by token budget instead of message count — more precise when messages vary a lot in length (a one-line question weighs very differently in tokens than a message with a full stack trace).

```python
TOKEN_LIMIT = 3000

def trim_by_tokens(history, limit):
    context = []
    total = 0
    for m in reversed(history):  # starts from the most recent
        tokens = count_tokens(m["content"])
        if total + tokens > limit:
            break
        context.insert(0, m)
        total += tokens
    return context
```

## Summarization memory

Instead of discarding the old stuff, it gets summarized — the next call uses the summary, not the raw messages.

```python
def chat(user_message, history, summary):
    history.append({"role": "user", "content": user_message})

    if len(history) > THRESHOLD:
        summary = llm.call(f"Summarize this conversation in a few lines: {summary}\n{history[:-4]}")
        history = history[-4:]  # keeps the last few turns + the summary of everything before

    context = [{"role": "system", "content": f"Summary of the conversation so far: {summary}"}] + history
    response = llm.call(messages=context)
    history.append({"role": "assistant", "content": response})
    return response
```

Doesn't lose the memory of old stuff like the sliding window does — it compresses it. The cost is an extra LLM call to generate the summary, and some detail gets lost in the compression.

## Long-term memory (vector DB)

The strategies above only cover the current session. For the chatbot to remember something from a session a month ago without resending months of full history, fragments of past conversations get stored as embeddings in a [vector DB](../database/nosql.md#vector--fundamental-characteristics), and only what's relevant to the current question gets pulled in.

```python
def chat(user_message, recent_history):
    question_embedding = generate_embedding(user_message)
    relevant_memories = vector_db.search(question_embedding, top_k=3)  # searches past sessions, not just the current one

    context = [{"role": "system", "content": f"Relevant data from previous conversations: {relevant_memories}"}] + recent_history
    response = llm.call(messages=context)

    save_to_vector_db(user_message, response)  # this conversation also becomes available in the future
    return response
```

It's the same [RAG pipeline](rag.md#the-pipeline-step-by-step) (chunking, embeddings, vector DB), applied to past conversations instead of a document base.

## How to choose

Full buffer for prototypes or short conversations. Sliding window or token-limited buffer when volume grows but short-term memory is enough. Summarization when it matters to retain the full thread of a long conversation without paying the cost of the entire raw history. Vector DB when the chatbot needs to remember across different sessions, not just within the same conversation.

---
Related: [AI Agent Design](agent-design.md#core-components), [What is a token](what-is-a-token.md#why-it-matters), [RAG](rag.md), [LLM Costs](llm-costs.md), [Context Engineering](context-engineering.md).
