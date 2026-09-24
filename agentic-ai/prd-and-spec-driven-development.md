# PRD and Spec-Driven Development

## What a PRD is

**Product Requirement Document**: a document defining what to build, for whom, and why — traditionally written by a Product Manager for a human engineering team. Includes the problem to solve, the scope, and acceptance criteria (how you know it's done).

## Spec-Driven Development (applied to agentic coding)

The idea: instead of telling an agent "build a todo app" and letting it interpret everything on the fly, a **clear, structured spec** gets written first — and the agent works *against* that spec, not against its memory of what was said in the chat.

```
Loose prompt:
"Build an endpoint to create users"
  → the agent interprets scope, validations, errors on its own judgment —
    may or may not match what was actually needed

Spec-driven:
spec.md defines: required fields, validations, expected error codes,
success and failure cases — the agent implements AGAINST that document,
and the result can be verified by comparing it to the spec
```

This gives two things a loose prompt doesn't:

1. The agent can split the task into steps **verifiable against the spec** (see [Plan-and-Execute](agent-design.md#plan-and-execute)) — instead of improvising the success criteria at each step.
2. Whoever reviews the result can check it against a fixed document, not against their own memory of what they'd asked for in the chat.

## PRD vs Technical Spec — not the same thing

The PRD is the starting point (what to build, for whom, why) — but in agentic coding, the spec the agent actually consumes tends to be **more technical** than a classic product PRD: concrete acceptance criteria, sometimes explicit test cases, expected data structure. In practice, many workflows start with a PRD and refine it toward something more technical before handing it to the agent — the PRD is the input, not necessarily the final spec the agent executes against.

## Why it matters for agentic

Without a spec, verifying whether an agent "did the right thing" depends on the memory/judgment of whoever's using it at that moment — with a written spec, verification is objective: does the result satisfy what the document says? It's the same underlying logic a [traditional PRD](#what-a-prd-is) brings to a human team, applied so an agent has something stable to work against instead of a prompt that can be reinterpreted every time.

---
Related: [Agent Design](agent-design.md), [Agents vs Workflows](agents-vs-workflows.md).
