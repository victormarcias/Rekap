# Types of AI Agents

## General definition

An agent is a system that **perceives** its environment, **processes** that information, and **acts** autonomously to achieve a goal — without a human having to decide every step. The cycle repeats continuously:

```
Perception (gathers data: APIs, DB, sensors)
    → Reasoning (analyzes, evaluates options, generates a plan)
    → Action (executes: API calls, writes, robotics)
    → Learning (adjusts future behavior based on the outcome)
    → back to Perception
```

Not every "agent" in the taxonomy below does all four stages with the same sophistication — it's a spectrum, from fixed rules to open-ended reasoning.

## The full spectrum

### Simple Reactive Agents

The most basic — condition-action (`if-then`) rules, **no memory** of past states, no planning ahead. React only to the current state.

```python
# example: a thermostat — reacts to the current reading, remembers nothing from before
def thermostat(current_temperature):
    if current_temperature < 18:
        return "turn_on_heating"
    return "turn_off_heating"
```

### Model-Based Agents

Maintain an **internal model of the world** with short-term memory — can handle partially observable environments because they track how the environment evolved, not just the present state.

*Example*: a robot vacuum that remembers which zones it already cleaned and where the obstacles are, so it doesn't repeat or collide.

### Goal-Based Agents

Besides the world model, they have an **explicit goal** and use search/planning algorithms to find the path toward it — more flexible than reactive agents, adapting the action based on how far they are from the goal.

*Example*: a navigation app that calculates the optimal route to a specific destination.

### Utility-Based Agents

When there are **multiple goals, sometimes conflicting**, "did I reach the goal, yes or no?" isn't enough — a utility function is needed to measure how good each option is, and pick the one that maximizes expected utility. They handle uncertainty better than pure goal-based agents.

*Example*: a self-driving car balancing speed, safety, and fuel consumption at once — there's no single goal, there's a trade-off between several.

### Learning Agents (RL)

Improve with experience — combine a performance element (that acts) with a learning element (that adjusts behavior based on the outcome of past actions). Typically via **reinforcement learning**: the agent tries actions, receives a reward or penalty, and adjusts its policy to maximize future reward.

*Example*: a recommendation system that refines its suggestions based on how you interact with what it showed you before.

### LLM-Based Agents

The **LLM is the reasoning engine** — understands complex natural language, generates sophisticated plans, and does advanced contextual reasoning with no hardcoded rules for every situation. It's the type of agent already covered in detail in [Agents vs Workflows](agents-vs-workflows.md) (the ReAct loop) and [n8n](../stacks/n8n/n8n-and-agentic-ai.md) (the AI Agent node).

*Example*: an agent that orchestrates a complete workflow by interpreting complex natural-language instructions.

### Multi-Agent Systems (MAS)

Multiple autonomous agents **interacting with each other** — cooperating or competing, solving distributed problems a single agent couldn't, requiring negotiation and coordination between them.

*Example*: self-driving vehicles coordinating at an intersection, or several specialized agents (one searches for data, another writes, another reviews) working in a chain on the same task.

---
Related: [Agents vs Workflows](agents-vs-workflows.md), [Agent Design](agent-design.md).
