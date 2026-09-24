# Risks and Mitigations in AI Agents

## Security and technical risks

- **Hallucinations**: the model generates false information with high confidence — it doesn't "know" it's wrong, it produces it with the same confident tone as a correct answer. Mitigated (not eliminated) with [RAG](rag.md) — giving it real data instead of letting it make things up.
- **Adversarial input**: input designed **on purpose** to exploit a weakness in the model — not an edge case that shows up on its own, it's a deliberate attack on how the model processes input. The classic example (computer vision) is an image with noise almost imperceptible to a human that makes a classifier get it wrong with high confidence. **Prompt Injection** below is the LLM/agent-specific version of this.
- **Prompt Injection**: manipulating the agent's instructions through the input — e.g. a user (or a document the agent reads via RAG) includes text like "ignore your previous instructions and do X." It's to an agent what SQL injection is to a query: input treated as if it were a trusted instruction.
- **Incorrect tool use**: the agent executes a tool with malformed arguments or in a context where it wasn't appropriate (e.g. deletes instead of archiving).
- **Infinite loops**: the agent gets stuck in a cycle without converging on an answer — not just a UX problem, it's real money spent on every round (see [Circuit Breaker as a spend limit](llm-costs.md#avoiding-spend-from-loops-that-dont-cut-themselves-off)).
- **Data poisoning**: the training data (or, in an agent with RAG, the knowledge base it queries) was maliciously manipulated to bias the responses.

## Ethical and operational risks

- **Algorithmic bias**: discriminatory decisions reflecting biases present in the training data — the model doesn't "decide" to be unfair, it reproduces patterns that were already in the data.
- **Lack of accountability**: when an autonomous agent fails, it's hard to assign blame — was it the prompt, the model, the tool it called, whoever designed the system?
- **Unpredictability**: in complex systems (multi-agent, long ReAct loops), the exact behavior is hard to predict ahead of time, even for whoever built it.
- **Excessive costs**: uncontrolled consumption of tokens/API calls — see [LLM Costs](llm-costs.md).
- **Employment impact**: automating tasks previously done by people causes job displacement — a real risk at the organizational/social level, not a technical one.

## Technical mitigations

- **Data validation**: diverse data, free of obvious bias, with encryption and access control.
- **Transparency**: explainability of the agent's decisions + logging to be able to audit afterward what happened and why.
- **Rigorous testing (Red Teaming)**: actively simulating attacks (trying to break the agent on purpose) to find vulnerabilities before someone else does.
- **Least privilege**: specialized sub-agents with limited, segmented permissions — an agent that only needs to read shouldn't have write permission.

## Process mitigations

- **Human oversight**: see [Human-in-the-Loop](agent-design.md#human-in-the-loop-hitl) — validation of critical decisions.
- **Continuous monitoring**: detecting anomalies and unexpected behavior in production, not just in testing.
- **Contingency**: rollback protocols and emergency shutdown — being able to "turn off" an agent quickly if it starts misbehaving.

## Governance and ethics

- **Ethical frameworks**: fairness, privacy, human rights as explicit design criteria, not an afterthought.
- **Audits**: periodic bias reviews, not just at launch.
- **Regulation**: adherence to frameworks like GDPR — see [Privacy and GDPR](../frontend-react/privacy-and-gdpr.md) for the detail on consent/data minimization, which applies equally to AI systems processing user data.
- **Education**: training the team on the risks and limits of the system they're building, not assuming "the AI already handles it."

## Data protection and PII

- **PII handling**: automatic detection and anonymization of personally identifiable information before it reaches the model or ends up in logs.
- **Content filtering**: blocking malicious prompts and outputs — both what comes in and what the agent generates.
- **Active moderation**: systems with rules and human feedback for edge cases.
- **Mitigating Prompt Injection in practice**: sandboxing (the tool the agent runs has limited permissions, see least privilege above), strict validation of what each tool can do, and never treating content from an external document (retrieved via RAG, for example) with the same trust level as the developer's instructions.

---
Related: [Agents vs Workflows](agents-vs-workflows.md), [LLM Costs](llm-costs.md), [RAG](rag.md), [Agent Design](agent-design.md), [Privacy and GDPR](../frontend-react/privacy-and-gdpr.md).
