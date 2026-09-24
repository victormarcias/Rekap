# AI Agent Design

## Core components

Any agent, regardless of its type (see [Types of Agents](agent-types.md)), is built from these pieces:

- **Perception**: processing input data (text, images, events) — multimodal if needed.
- **Reasoning**: the decision engine — can be an LLM, a classical ML model, or fixed rules.
- **Memory**: short-term state (the current conversation) and long-term (what happened in previous interactions) — without this, the agent "forgets" everything between steps. See [Conversational Memory](conversational-memory.md) for the concrete strategies for managing it.
- **Action**: the actual execution — connecting to APIs, external systems, databases.
- **Feedback**: monitoring the result, to learn and adjust future behavior.

## In practice: Model + Memory + Tools

The components above are the conceptual framework — in practice, when building an agent with an LLM, they simplify to three concrete pieces:

- **Model**: the LLM that reasons and decides — the **Reasoning** from above.
- **Memory**: the conversation history, and optionally long-term memory — see [Conversational Memory](conversational-memory.md).
- **Tools**: the functions/APIs the agent can invoke — the **Action** from above, see [Function Calling](function-calling.md).

**Perception** and **Feedback** are almost never a separate module in this setup: perception is whatever goes into the prompt on each call (text, a tool's result), and feedback happens outside the agent's runtime — it gets evaluated afterward, on logs and results, not as a piece that runs at every step.

## Layered architecture

```
Input Layer          → APIs | Sensors | UI | Events
      ↓
Processing Layer     → LLM | Rules engine | ML models | Memory
      ↓
Decision Layer       → Planner | Evaluator | Action selector
      ↓
Output Layer         → Executor | Integrations | Actuators
      ↑
      └── Feedback loop (feeds back into the Input Layer)
```

**Key design principles**: **modularity** (each layer can be swapped without touching the others — switching LLMs shouldn't break the input layer), **scalability**, and **observability** (being able to see what the agent decided and why, not a black box — see [Transparency in Risks and Mitigations](risks-and-mitigations.md#technical-mitigations)).

## Execution patterns

### Plan-and-Execute

Planning and execution **separated** into two phases: first a complete high-level plan is built, then each step of the plan gets executed in order.

```python
plan = llm.call(f"Build a step-by-step plan for: {task}")  # phase 1: plan everything at once
for step in plan.steps:
    execute(step)  # phase 2: execute in order, without replanning in between
```

Ideal for **structured** tasks where the steps can be foreseen ahead of time — predictable and easy to debug, because the complete plan exists before touching anything.

### ReAct (Reason + Act)

An iterative cycle of reasoning and acting, one step at a time — already covered in detail in [Agents vs Workflows](agents-vs-workflows.md#agent-pattern-the-llm-controls-the-path). Ideal for dynamic/exploratory environments, where you can't plan everything ahead of time because each step depends on the previous one's result.

**Plan-and-Execute vs ReAct**: the former decides the whole path before taking the first step; the latter decides one step, sees what happened, and only then decides the next one. Plan-and-Execute is more predictable but more rigid — if something turns out differently than planned halfway through, it doesn't self-correct as well as ReAct.

### Human-in-the-Loop (HITL)

Integrates explicit human intervention into the flow — doesn't replace the agent, complements it at the points where the risk of an autonomous decision is too high.

- **Supervisor**: validates critical decisions before they execute.
- **Validator**: gives feedback on an already-generated result.
- **Expert**: steps in for exceptions or ambiguities the agent can't resolve on its own.

```python
def execute_with_hitl(proposed_action):
    if proposed_action.is_critical:  # e.g. deleting data, sending money, publishing something
        approval = request_human_approval(proposed_action)
        if not approval:
            return "cancelled"
    return execute(proposed_action)
```

Critical in high-risk domains (healthcare, finance) — increases reliability at the cost of losing some of the full autonomy a pure agent promises.

---
Related: [Types of Agents](agent-types.md), [Agents vs Workflows](agents-vs-workflows.md), [Prompt Engineering](prompt-engineering.md#structure-of-a-system-prompt), [Risks and Mitigations](risks-and-mitigations.md).
