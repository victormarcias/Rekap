# Evals

Evals measure whether an LLM or an agent does what's expected — the equivalent of tests for a system whose output isn't deterministic. Without them, changing a prompt or switching models is a guess: something may improve while something else silently breaks.

Classic unit tests fall short because the same input can produce different outputs, and "correct" is often a matter of degree (an answer can be right but incomplete, or right in the wrong tone).

## Types of evals

| Type | How it grades | Cost | Good for |
|---|---|---|---|
| **Public benchmarks** | Standard datasets (MMLU, SWE-bench, GPQA) | Free, already published | Comparing models in general — not your use case |
| **Code-based checks** | Exact match, regex, JSON schema validation, running the generated code | Very low | Anything with a verifiable answer: classification, extraction, tool choice |
| **LLM-as-a-judge** | Another LLM grades the answer against a rubric | Low | Open-ended answers: summaries, tone, faithfulness |
| **Human evaluation** | People rate or compare answers | High | Calibrating the other methods, subjective quality |

Public benchmarks have two known problems: *contamination* (test questions leaked into the training data) and *saturation* (every top model scores near 100%). A good score there says little about how a model behaves with your data.

## Golden dataset

A set of real inputs with their expected output (or grading criteria), versioned in the repo and run on every prompt or model change — a regression test suite:

```python
golden = [
    {"input": "Cancel my order A-102", "expected_tool": "cancel_order"},
    {"input": "Where is my package?", "expected_tool": "track_shipment"},
    {"input": "I want a refund for the broken lamp", "expected_tool": "create_refund"},
]

def run_evals(agent) -> float:
    passed = sum(agent.pick_tool(case["input"]) == case["expected_tool"] for case in golden)
    return passed / len(golden)  # compare this score across prompt/model versions
```

Real production failures are the best source of new cases: every bug found becomes a row in the dataset.

## LLM-as-a-judge

```python
JUDGE_PROMPT = """Rate from 1 to 5 how faithful the answer is to the context.
5 = everything in the answer is supported by the context. 1 = it invents facts.

Context: {context}
Answer: {answer}

Reply only with the number."""
```

Judges have biases: they tend to prefer longer answers, the first option shown, and answers from their own model family. It helps to use a concrete rubric, compare pairs instead of absolute scores, and check a sample against human ratings.

## What to measure

| System | Metrics |
|---|---|
| [RAG](rag.md) | Retrieval relevance (were the right chunks retrieved?), faithfulness (is the answer supported by them?), answer relevance |
| Agents | Task success, correct tool choice, number of steps, cost and latency per task |
| Any LLM feature | Format compliance, refusals where they shouldn't happen, [prompt injection](risks-and-mitigations.md#security-and-technical-risks) resistance |

## Offline vs online

- **Offline**: the golden dataset runs before deploying, ideally in CI — catches regressions before users see them.
- **Online**: in production — user feedback (👍/👎), A/B tests between prompts or models, and sampled traces graded by a judge. Tools like Langfuse, LangSmith, or promptfoo cover both.

---
Related: [Foundation Models](foundation-models.md#4-evaluation), [RAG](rag.md#where-it-tends-to-fail), [Risks and Mitigations](risks-and-mitigations.md), [Model Comparison](model-comparison.md), [Agent Design](agent-design.md).
