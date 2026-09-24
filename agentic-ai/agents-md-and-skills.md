# AGENTS.md and Skills

Two pieces of the agentic "harness" (everything around the LLM that makes it useful in a real repo, beyond the model itself) that solve the same underlying problem: **an LLM has no memory between conversations** — every new chat starts from zero, knowing nothing about your project or what was already decided yesterday.

## AGENTS.md — the "blank slate" problem

Every time you open a new conversation with a coding agent, the model doesn't know: your stack, your conventions, your build/test commands, or anything discussed in previous sessions. Without that context, the agent **guesses** — and guesses wrong, in ways that end up costing more time than they save.

**AGENTS.md** is a text file at the repo root (or in any subfolder) that gives the agent that entry context: tech stack, commands for running tests/builds, team conventions, things to avoid. The agent reads it automatically on startup, so you don't have to repeat the same things in every conversation.

```markdown
# AGENTS.md (minimal example)

## Stack
FastAPI + PostgreSQL + React. Python 3.12, Node 20.

## Commands
- Tests: `pytest`
- Build frontend: `npm run build`

## Conventions
- Never use `git push --force` without confirming first.
- New endpoints go in `api/routes/`, with their corresponding test.
```

It's an **open** format, adopted by multiple agentic coding tools (not specific to just one) — the idea is that the same `AGENTS.md` works regardless of which agent you're using that day.

## Skills — modular capabilities

A **Skill** packages instructions + metadata + resources into a self-contained unit the agent can invoke when the task calls for it — the equivalent of a plugin: installed once, and available in any future conversation without re-explaining how to do that task.

**Anatomy of a Skill**: a `SKILL.md` file with two parts.

```markdown
---
name: deploy-preview
description: Deploy a preview environment for the branch
---

## Instructions

1. Run the build pipeline
2. Push to staging CDN
3. Return the preview URL
```

1. **YAML frontmatter**: metadata — the `name` becomes the command (`/deploy-preview`).
2. **Markdown body**: step-by-step instructions for how to execute the task.
3. The folder containing the `SKILL.md` is, by convention, the skill's name.

**Invocation**: explicit (the user types `/deploy-preview`) or automatic (the agent detects the current task matches the skill's description and fires it on its own) — either way, the skill stays "dormant" until needed, without occupying conversation context ahead of time.

**Where they live**: at the personal level (available in any project you open) or at the project level (committed to the repo, shared with the team) — the difference is the same as between a preference of yours and a team convention.

## The difference from MCP

An [MCP server](mcp.md) gives the agent **access to an external system** (an API, a database). A Skill gives the agent **a recipe for how to do something** — it can use MCP tools in the process, but the Skill itself is just instructions, not a new technical integration. AGENTS.md, in turn, is passive context (the agent reads it, doesn't "execute" it) — Skills are active capabilities the agent decides to invoke.

---
Related: [MCP](mcp.md), [PRD and Spec-Driven Development](prd-and-spec-driven-development.md), [Agent Design](agent-design.md).
