# n8n — How it's tested

n8n doesn't have a test framework like [pytest](../../system-design/testing.md) or impose a code architecture to write unit tests against. The **testable unit is the entire workflow**, not an isolated function — and what's available to test it is manual/integration, backed by tools the UI itself provides, not a separate test runner.

## Manual execution — step by step or full run

From the editor you can run the entire workflow (**Execute Workflow**) or a single node (**Execute Step**), seeing each node's input and output JSON right there. It's the closest equivalent to "run the code and see what it returns" — but manual, not something automated that runs on its own with every change.

## Pinned data — locking a node's output

You can "pin" a node's result (e.g. an external API's response) so the next runs use that saved data instead of calling the real service again.

```
"HTTP Request" node (calls an external API)
  → first run: calls for real, brings back the real response
  → you pin that response
  → subsequent runs: use the pinned data, don't hit the API again

Useful for iterating quickly on the FOLLOWING nodes without spending quota,
waiting on real latency, or depending on the external service being up
```

It's conceptually similar to a [mock](../../system-design/testing.md#2-test-doubles--mock-vs-stub-vs-fake-vs-spy) — replacing an external dependency with a fixed, known value, to be able to test the rest of the logic in isolation and repeatably.

## Execution history — the log of what happened

Every run (manual or triggered by a real trigger) gets saved with each node's input/output, and if it failed, exactly which node and with what error. It's the main tool for debugging after something went wrong in production — it doesn't prevent the error, but it gives full visibility into the cause.

## Error workflow — the closest thing to automatic failure handling

You can configure a separate workflow that fires automatically when another workflow fails — typically to alert (Slack, email) or log the failure somewhere. It's not a test, it's production error handling, but it's the closest thing to an automated safety net n8n offers out of the box.

## If something closer to real CI is needed

n8n doesn't provide this out of the box, but it can be built: export the workflow to JSON, and run it from the **n8n CLI** against test data within a CI pipeline, comparing the output against what's expected by hand.

```bash
n8n execute --id <workflow_id>   # runs a workflow from the terminal, without the UI
```

It's a homegrown approach (the team builds it, it doesn't come integrated) — for cases where the workflow is critical enough to justify that extra effort.

## Why it matters

This is, concretely, what's behind the trade-off already mentioned in [n8n and Agentic AI](n8n-and-agentic-ai.md#trade-off-against-writing-the-agent-in-code): "real automated testing" isn't something n8n offers natively — what exists is a set of manual/inspection tools, useful for developing and iterating, but far from the [test pyramid](../../system-design/testing.md#1-test-pyramid) (unit → integration → e2e) built with code.

---
Related: [n8n Basics](basics.md), [n8n and Agentic AI](n8n-and-agentic-ai.md), [Testing — General Concepts](../../system-design/testing.md).
