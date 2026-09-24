# Function Calling (Tool Use)

An LLM's ability to, instead of just returning text, decide "for this I need to execute something" and return a structured instruction (function name + arguments) for the application to execute — it's the concrete mechanism behind [tool use and agentic behavior](from-ml-to-agentic-ai.md#7-tool-use--function-calling--the-llm-can-act-not-just-talk-2023) and what an [MCP server](mcp.md) exposes as **tools**.

**"Tool use" vs "function calling"**: used as synonyms in practice, but *function calling* is technically a subset — the case where the tool is a custom function with a JSON schema (a term OpenAI coined in 2023). *Tool use* (Anthropic's term) is broader: it also includes platform built-in tools that aren't a function of yours (computer use, bash, web search).

## How a tool is defined

For the LLM to know what tools it has available, the application sends it a schema (JSON Schema) describing each function: name, description, and the parameters it expects.

```python
tools = [
    {
        "name": "get_weather",
        "description": "Gets the current weather for a city",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "City name"},
            },
            "required": ["city"],
        },
    }
]
```

The **description** matters as much as the name — it's the only thing the LLM has to decide whether this tool is relevant to the current task and how to fill in its parameters. A vague description leads the model to use it wrong, or not use it when it should.

## The flow: back and forth, not a single call

Function calling isn't "the LLM executes code" — **the LLM never executes anything**. It returns a structured instruction, the application actually executes it, and sends the result back to the LLM in the next message so it can continue.

```
1. App → LLM: user message + list of available tools
2. LLM → App: "I want to call get_weather with city='Buenos Aires'"
                (the LLM did NOT execute anything, it just decided what to call)
3. App: actually executes get_weather("Buenos Aires"), against a real API
4. App → LLM: here's that tool's result
5. LLM → App/User: final response, now with the real data incorporated
```

```python
# simplified loop, Anthropic API format
response = client.messages.create(model=MODEL, tools=tools, messages=messages)

if response.stop_reason == "tool_use":
    tool_call = response.content[-1]  # the block that requested executing the tool
    result = execute_real_tool(tool_call.name, tool_call.input)  # the app runs it, not the LLM

    messages.append({"role": "assistant", "content": response.content})
    messages.append({
        "role": "user",
        "content": [{"type": "tool_result", "tool_use_id": tool_call.id, "content": result}],
    })

    response = client.messages.create(model=MODEL, tools=tools, messages=messages)  # the LLM continues, now with the real data
```

This is exactly the low-level mechanism behind the [ReAct loop](agents-vs-workflows.md#agent-pattern-the-llm-controls-the-path) we already saw in pseudocode — here it's the real message format going back and forth.

## Multi-turn and parallel tool calls

A task can need several rounds of this loop (call a tool, see the result, decide to call another) before giving the final answer — there's no fixed step limit, the LLM decides when it already has what it needs (which is why a [Circuit Breaker](../system-design/quality-attributes.md#fault-tolerance) is worth having if something gets stuck retrying, see [LLM Costs](llm-costs.md#avoiding-spend-from-loops-that-dont-cut-themselves-off)). Some models can also request **several tool calls in the same response** (e.g. "I need the weather for 3 cities") to execute them in parallel instead of one by one.

## Function calling vs MCP

Function calling is the **mechanism**: how the LLM asks to execute something and receives the result, part of the model API's contract. [MCP](mcp.md) is a **higher-level protocol** that standardizes how those tools get exposed between different applications, so you don't reinvent the integration with every external service — an MCP server, underneath, ends up generating exactly the kind of tool schema function calling needs.

## Why it matters

Function calling is what separates an LLM that "only chats" from one that can **act** on the real world (query a DB, send an email, run code) — without this mechanism, the rest of agentic behavior (ReAct, MCP, orchestrator-workers) would have no way to touch anything outside the conversation.

---
Related: [From Classical ML to Agentic AI](from-ml-to-agentic-ai.md#7-tool-use--function-calling--the-llm-can-act-not-just-talk-2023), [Agents vs Workflows](agents-vs-workflows.md), [MCP](mcp.md), [Agent Design](agent-design.md), [LLM Costs](llm-costs.md).
