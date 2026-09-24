# n8n — Basics

## What it is

A **node-based** (drag and connect visual blocks) and **open source** workflow automation tool — an alternative to Zapier/Make aimed at technical people: it can be self-hosted for free, and comes with a code node (JS/Python) for custom logic the purely no-code alternatives don't offer. A workflow connects a **trigger** (what fires the execution) with a chain of **actions** (calling an API, sending an email, writing to a DB, transforming data).

## Where it runs: Local vs Self-hosted (VPS) vs Cloud

- **Local**: running n8n on your own machine (Docker or `npm install n8n`) to develop and test — not accessible from outside your network without exposing it.
- **Self-hosted (VPS)**: your own instance on your own server (see [Deploy to a VPS](../../devops/deploy-vps.md)) — full control, no free-plan limits, but you manage uptime, backups, and updates.
- **n8n Cloud**: a version managed by n8n's own creators — no infrastructure of your own to maintain, in exchange for a usage-based paid plan.

The choice is the same trade-off we already saw between [VPS and Cloud Run](../../devops/vps-vs-cloud-run.md): how much control you want vs how much maintenance you're willing to take on.

## How it works: nodes, triggers, and JSON

Each **node** is a step in the workflow — an action, a condition, a data transformation. The **trigger node** is the one that fires everything (an incoming webhook, a cron every X minutes, an event from a connected app). Nodes are chained together, and what passes between them is always **JSON**: one node's output is the next one's input.

```json
// typical node output (e.g. after calling an API) —
// this is exactly what the next node receives as input
{
  "id": 123,
  "name": "Ana",
  "email": "ana@mail.com"
}
```

n8n ships with hundreds of prebuilt integrations (Slack, Gmail, Google Sheets, databases, etc.), but the most versatile node is **HTTP Request** — it calls any REST API that doesn't have a native integration, with the same verbs/headers/body you'd build by hand (see [REST](../../backend/rest.md), [HTTP Methods](../../backend/http-methods.md)).

## Configuration: three different things called "config"

**1. The workflow itself — JSON.** All its nodes, connections, and parameters are saved and exported as a single JSON file — this is what you version in git, share with the team, or migrate between instances. Don't confuse it with the JSON passed *between* nodes at runtime (above) — this is the definition of the entire workflow.

```json
{
  "name": "Process new order",
  "nodes": [
    { "id": "1", "type": "n8n-nodes-base.webhook", "parameters": { "path": "new-order" } },
    { "id": "2", "type": "n8n-nodes-base.httpRequest", "parameters": { "url": "https://api.myapp.com/orders" } }
  ],
  "connections": {
    "1": { "main": [[{ "node": "2", "type": "main", "index": 0 }]] }
  }
}
```

**2. The server instance — environment variables (`.env`).** How the n8n server itself is configured (not a specific workflow) doesn't use its own config file — they're standard environment variables.

```bash
# .env
N8N_PORT=5678
N8N_HOST=n8n.myapp.com
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
N8N_ENCRYPTION_KEY=a-secret-key
```

**3. If self-hosted with Docker — `docker-compose.yml`.** The YAML isn't n8n's, it's Docker Compose's — it orchestrates the container, its environment variables, and the volumes where data persists.

```yaml
services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_PORT=5678
      - DB_TYPE=postgresdb
    volumes:
      - ~/.n8n:/home/node/.n8n
```

A `package.json` only shows up if n8n is installed via `npm install n8n` instead of Docker — that's standard Node package management, nothing n8n-specific.

---
Related: [n8n and Agentic AI](n8n-and-agentic-ai.md), [How it's tested](testing.md), [Deploy to a VPS](../../devops/deploy-vps.md), [VPS vs Cloud Run](../../devops/vps-vs-cloud-run.md), [REST](../../backend/rest.md).
