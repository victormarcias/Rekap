# Agents vs Workflows

The distinction that matters (comes from how Anthropic formalized it in their own guide to building systems with LLMs): it's not a difference in how "smart" the system is, it's a difference in **who controls the flow of steps**.

- **Workflow**: the developer defines the sequence of steps ahead of time, in code or fixed config. The LLM solves specific tasks *within* an already-drawn path — it doesn't decide the order or how many steps there are.
- **Agent**: the LLM decides the control flow in real time — how many steps are needed, which tool to use at each one, when to consider the task done.

## Workflow Patterns (the path is fixed)

**Prompt chaining**: splitting a task into sequential LLM steps, where one's output is the next one's input — each step is simpler and more reliable than asking for everything at once.

```python
# the developer defines the order — the LLM just executes each step
summary = llm.call(f"Summarize this text: {text}")
translation = llm.call(f"Translate this to English: {summary}")
title = llm.call(f"Generate a short title for: {translation}")
```

**Routing**: classifying the input and sending it down a specialized path based on type — instead of one generic prompt trying to cover every case.

```python
category = llm.call(f"Classify this ticket: {ticket}")  # "billing" | "technical" | "general"

if category == "billing":
    response = llm.call(billing_specialized_prompt, ticket)
elif category == "technical":
    response = llm.call(technical_specialized_prompt, ticket)
```

**Orchestrator-workers**: a central LLM splits a large task into subtasks and delegates them to "worker" LLMs that solve them in parallel or in sequence, without each worker knowing anything about the rest.

## Agent Pattern (the LLM controls the path)

A ReAct-style loop (*Reason + Act*): the LLM reasons about what to do, executes an action (calls a tool), observes the result, and decides the next step — with nobody having predefined how many rounds it's going to take.

```python
while not task_done:
    step = llm.call(conversation_history)  # the LLM decides WHAT to do now
    if step.needs_tool:
        result = execute_tool(step.tool_name, step.args)
        conversation_history.append(result)  # observes, and continues the loop
    else:
        task_done = True  # the LLM itself decided it's done
```

## When to use each one

| | Workflow | Agent |
|---|---|---|
| Steps are known ahead of time | ✅ | Not necessarily |
| Predictability | High — same input, same path | Low — the path can vary between runs |
| Cost | Lower (fixed steps, no extra exploration) | Higher (can iterate, retry, explore paths that don't work out) |
| Deterministically testable | ✅ | Hard — the same input can take different paths |
| Open-ended / ambiguous tasks | Breaks easily (doesn't account for the unexpected) | Exactly what it's for |

**Why the cost is higher in an agent, concretely**: besides the number of calls not being fixed, in an agent loop each new call usually includes **the entire prior history** (which tools it called, what they returned) so the model has memory of what it already tried — step 10 of the loop is much more expensive in tokens than step 1, because it drags along everything before it. A workflow doesn't have this problem: each step can have a prompt scoped to just what that step needs, without accumulating the full history.

**Practical rule**: if you can write the flowchart before starting, it's a workflow — cheaper, more reliable, easier to debug. If the path genuinely depends on what's discovered at each step (you don't know how many searches will be needed, or in what order), that's where an agent brings something a fixed workflow can't.

## It's not a binary choice

In practice, many real systems combine the two, **in both directions**:

- **A workflow with an agent inside**: the overall structure is a workflow (predictable steps, guardrails, validations), which delegates to an agent at a specific step when that step needs open-ended reasoning. This is literally what n8n's [AI Agent node](../stacks/n8n/n8n-and-agentic-ai.md#the-ai-agent-node) enables: the workflow stays the fixed structure of connected nodes, but one of those nodes internally runs an agentic loop.
- **An agent with a workflow inside**: the agent decides to call a tool, but that tool isn't an atomic action — internally it runs a fixed multi-step pipeline (e.g. "process order" = validate → charge → send email → update stock). The agent doesn't know or care that there's a deterministic workflow in there; from its perspective it's "one tool call, one result." It's the most common pattern in production systems: you don't give the agent fine-grained control over every low-level step (that would be more expensive and less reliable), you give it a high-level tool that already encapsulates a proven workflow, and the agent orchestrates at a higher level.

---
Related: [From Classical ML to Agentic AI](from-ml-to-agentic-ai.md), [Function Calling](function-calling.md), [n8n](../stacks/n8n/n8n-and-agentic-ai.md).
