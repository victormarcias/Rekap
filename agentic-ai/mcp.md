# MCP (Model Context Protocol)

An open protocol (created by Anthropic, adopted by the rest of the industry) for connecting an LLM to external tools and data sources in a standardized way — it's literally what I myself use in this conversation to touch the browser, read files, or run commands.

## The problem it solves: N×M integrations

Before MCP, every combination of agent + external tool (GitHub, Slack, a database) needed its own custom integration — if you had 3 agents and 3 tools, you'd end up writing 9 different integrations, each fragile and its own to maintain.

```
Without a standard protocol:      With MCP:
Agent A ─┬─ GitHub                Agent A ─┐
Agent B ─┼─ Slack                 Agent B ─┼─ MCP ─┬─ GitHub
Agent C ─┴─ Database              Agent C ─┘        ├─ Slack
                                                      └─ Database
N x M custom integrations         N + M integrations (one per agent, one per tool)
```

## Architecture: Host, Client, Server

- **Host**: the application using the LLM (Claude Code, Claude Desktop, Cursor, etc.) — decides which MCP servers to connect.
- **Client**: lives inside the host, keeps a **1:1** connection with a server and speaks the protocol (JSON-RPC) with it.
- **Server**: exposes the actual capabilities — it's not the LLM, it's the program that knows how to talk to GitHub, to a database, to the filesystem, etc.

An MCP server can expose three types of capabilities:

- **Tools**: functions the LLM can invoke (e.g. `create_issue`, `read_file`) — the equivalent of [tool use / function calling](function-calling.md).
- **Resources**: data the host can read and give to the LLM as context (e.g. a file's contents).
- **Prompts**: reusable prompt templates the server exposes for common tasks.

The transport between client and server is **JSON-RPC** over `stdio` (local process) or HTTP/SSE (remote server).

## Why it matters: one server, every agent

The central advantage is that **the same MCP server works for any compatible host** — whoever builds the GitHub integration writes it once, and it can be used by Claude Code, Cursor, or any other agent that speaks the protocol. That's why the MCP server ecosystem grew so fast: it's not "one integration per product," it's "one integration, N products."

---
Related: [Function Calling](function-calling.md), [Agents vs Workflows](agents-vs-workflows.md#agent-pattern-the-llm-controls-the-path), [Agent Design](agent-design.md), [AGENTS.md and Skills](agents-md-and-skills.md).
