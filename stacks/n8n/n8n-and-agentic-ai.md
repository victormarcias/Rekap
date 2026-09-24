# n8n and Agentic AI

## The AI Agent node

n8n has an **AI Agent** node (with LangChain integrated underneath) that gives an LLM access to the rest of the workflow's nodes as **tools** — it's the visual/low-code version of the same [tool use and agentic](../../agentic-ai/from-ml-to-agentic-ai.md#7-tool-use--function-calling--the-llm-can-act-not-just-talk-2023) pattern built with code: the LLM decides which node/tool to use, with what data, and chains steps until the task is solved. It's configured by connecting three pieces — Model, Memory, Tools — the same [Model + Memory + Tools](../../agentic-ai/agent-design.md#in-practice-model--memory--tools) architecture built by hand in code.

It's also a concrete example of the "[workflow with an agent inside](../../agentic-ai/agents-vs-workflows.md#its-not-a-binary-choice)" pattern: the workflow itself stays the fixed structure of connected nodes (predictable, with guardrails), but the AI Agent node internally runs an agentic loop when that specific step needs open-ended reasoning.

## Trade-off against writing the agent in code

n8n is much faster to prototype with and doesn't require the whole team to know how to code — but complex logic, real automated testing, fine-grained version control (a visual workflow is harder to diff in a PR than code), and critical performance are handled better by writing the agent directly.

## When n8n makes sense vs code

- **n8n**: fast prototypes, automations and integrations between SaaS with no heavy logic, teams where non-dev people need to be able to maintain the workflow.
- **Code**: complex business logic, need for automated tests, fine-grained version control, critical performance, or an engineering team that already has its own stack and prefers not to depend on an external tool for its main product.

---
Related: [n8n Basics](basics.md), [How it's tested](testing.md), [Agents vs Workflows](../../agentic-ai/agents-vs-workflows.md), [Types of AI Agents](../../agentic-ai/agent-types.md), [From Classical ML to Agentic AI](../../agentic-ai/from-ml-to-agentic-ai.md).
