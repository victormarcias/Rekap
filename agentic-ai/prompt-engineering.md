# Prompt Engineering

Ways to improve an LLM's output **without retraining the model** — all of them act on the prompt, not on the model's weights (unlike fine-tuning, which does adjust them — see the end of this file).

## Clear instructions and context

Design the prompt with intent: clear instructions, sufficient context, specified output format — the difference between a useful answer and a generic one usually lies more in how it's asked than which model is used.

```python
# ❌ vague: the model has to guess what format/level of detail you expect
prompt = "Explain what a database index is"

# ✅ specific: context, audience, expected format
prompt = """
Explain what a database index is to someone who already knows basic SQL
but has never seen indexes. Use a concrete example with a users table.
Answer in 3 short paragraphs, no code.
"""
```

## Structure of a system prompt

An agent's system prompt is best ordered from most general to most specific, so the model has the framework before the detail:

1. **Role and goal** (*role prompting*): who the agent is and what its task is (`"You are a support assistant; your task is to resolve level 1 tickets"`). Constrains the response space more than any other part of the prompt.
2. **Context**: what the agent needs to know and can't infer — business rules, the format of data it'll receive, what's out of scope.
3. **Instructions**: what to do and what not to, step by step. This is where [few-shot](#zero-shot-vs-few-shot-learning) examples go if the output format needs to be exact.
4. **Tools**: what tools it has and **when** to use each one — exposing the tool isn't enough, you have to say in which situation it applies (see [Function Calling](function-calling.md)).
5. **Variables**: the data that changes on every request (date, user, history) goes in as placeholders filled in at runtime — if hardcoded, the prompt goes stale as soon as the data changes.

```text
# Role
You are a scheduling assistant for a clinic. Your task is to coordinate appointments.

# Context
- Hours: Monday to Friday, 9am to 6pm. An appointment lasts 30 minutes.

# Instructions
- Confirm the name and reason for the visit before scheduling.
- If the requested time isn't available, offer the two closest ones.

# Tools
- Calendar_Check → check availability. Use it before confirming any appointment.
- Calendar_Book → book once confirmed with the patient.

# Variables
Current date and time: {now}
Patient: {user_name}
```

**Format**: use Markdown headers (`#`, `##`) to separate sections — the model reads them as hierarchy and doesn't mix, for example, context with instructions.

**Length**: complete but not verbose. Every token of the system prompt gets paid for on **every** call (see [LLM Costs](llm-costs.md)), and in an agent the system prompt travels on every round of the loop — the extra gets multiplied per iteration.

## Chain of Thought (CoT)

Asking the model to **reason step by step before giving the final answer**, instead of jumping straight to a conclusion — noticeably improves accuracy on tasks requiring several logical steps (math, debugging, decisions with multiple conditions).

```python
# without CoT: the model can jump to an answer without having "thought through" the intermediate steps
prompt = "How much does shipping cost if the order weighs 12kg and costs $50, with free shipping only above $100?"

# with CoT: explicitly asked to show the reasoning before concluding
prompt = """
Solve this step by step, showing each calculation before giving the final answer:
How much does shipping cost if the order weighs 12kg and costs $50, with free shipping only above $100?
"""
```

It's, in essence, the same logic as the [ReAct loop](agents-vs-workflows.md#agent-pattern-the-llm-controls-the-path) — "reason before acting" — but applied within a single response, with no need for a tool loop.

**Where it goes**: at the end of the prompt, after the instructions. With models that already reason out of the box (OpenAI o1/o3, Claude with *extended thinking*, DeepSeek R1) asking for explicit CoT is redundant — they generate an internal reasoning chain before responding without being asked (see [test-time compute](what-is-a-token.md#test-time-compute--thinking-more-when-answering-not-when-training)).

## Zero-shot vs Few-shot Learning

- **Zero-shot**: asking the model to solve a task **with no prior example** in the prompt — relies on what it already learned during general training.
- **Few-shot**: including a few examples of the desired input/output within the same prompt, so the model infers the exact pattern expected.

```python
# Zero-shot: just the instruction
prompt = "Classify the sentiment of this review: 'Arrived broken and late'"

# Few-shot: examples of the exact expected output format are shown
prompt = """
Classify the sentiment as POSITIVE, NEGATIVE, or NEUTRAL.

Review: "Excellent quality, I recommend it" → POSITIVE
Review: "Never arrived" → NEGATIVE
Review: "It's like any other" → NEUTRAL

Review: "Arrived broken and late" →
"""
```

Few-shot tends to improve output format consistency (useful when the output has to be parseable, e.g. JSON with an exact structure) — the cost is that each example adds [tokens](what-is-a-token.md) to the prompt, and therefore to the cost of every call (see [LLM Costs](llm-costs.md)).

## Fine-tuning (for comparison)

Unlike everything above, fine-tuning does **adjust the model's weights** by training it on your own data — already covered in the [history of the evolution toward LLMs](from-ml-to-agentic-ai.md#4-pretrained-models--one-base-model-many-uses-2018-2020). It's more expensive and slower to iterate on than adjusting a prompt, but it's useful when the behavior you need can't be achieved with any prompting technique — e.g. a very specific style/format that has to be repeated consistently thousands of times, or domain knowledge that doesn't reasonably fit in a prompt.

**Practical rule**: try prompt engineering + few-shot first (fast, cheap, iterable) — only consider fine-tuning if that's not enough.

---
Related: [What is a token](what-is-a-token.md), [Function Calling](function-calling.md), [LLM Costs](llm-costs.md), [AI Agent Design](agent-design.md), [Agents vs Workflows](agents-vs-workflows.md), [From Classical ML to Agentic AI](from-ml-to-agentic-ai.md), [Context Engineering](context-engineering.md).
