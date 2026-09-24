# Function Calling (Tool Use)

La capacidad de un LLM de, en vez de solo devolver texto, decidir "para esto necesito ejecutar algo" y devolver una instrucción estructurada (nombre de función + argumentos) para que la aplicación la ejecute — es el mecanismo concreto detrás de [tool use y agentic](historia-de-ml-a-agentic.md#7-tool-use--function-calling--el-llm-puede-hacer-no-solo-hablar-2023) y de lo que expone un [servidor MCP](mcp.md) como **tools**.

**"Tool use" vs "function calling"**: en la práctica se usan como sinónimos, pero *function calling* es técnicamente un subconjunto — el caso donde la tool es una función custom con JSON schema (término que acuñó OpenAI en 2023). *Tool use* (término de Anthropic) es más amplio: incluye también tools built-in de la plataforma que no son una función tuya (computer use, bash, web search).

## Cómo se define una tool

Para que el LLM sepa qué herramientas tiene disponibles, la aplicación le manda un schema (JSON Schema) describiendo cada función: nombre, descripción, y los parámetros que espera.

```python
tools = [
    {
        "name": "get_weather",
        "description": "Obtiene el clima actual de una ciudad",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "Nombre de la ciudad"},
            },
            "required": ["city"],
        },
    }
]
```

La **description** importa tanto como el nombre — es lo único que el LLM tiene para decidir si esta tool es relevante para la tarea actual y cómo llenar sus parámetros. Una descripción vaga produce que el modelo la use mal, o no la use cuando debería.

## El flujo: ida y vuelta, no una sola llamada

Function calling no es "el LLM ejecuta código" — **el LLM nunca ejecuta nada**. Devuelve una instrucción estructurada, la aplicación la ejecuta de verdad, y le manda el resultado de vuelta al LLM en el siguiente mensaje para que continúe.

```
1. App → LLM: mensaje del usuario + lista de tools disponibles
2. LLM → App: "quiero llamar a get_weather con city='Buenos Aires'"
                (el LLM NO ejecutó nada, solo decidió qué llamar)
3. App: ejecuta get_weather("Buenos Aires") de verdad, contra una API real
4. App → LLM: acá está el resultado de esa tool call
5. LLM → App/Usuario: respuesta final, ya con el dato real incorporado
```

```python
# loop simplificado, formato de la API de Anthropic
response = client.messages.create(model=MODEL, tools=tools, messages=messages)

if response.stop_reason == "tool_use":
    tool_call = response.content[-1]  # el bloque que pidió ejecutar la tool
    resultado = ejecutar_tool_real(tool_call.name, tool_call.input)  # la app la corre, no el LLM

    messages.append({"role": "assistant", "content": response.content})
    messages.append({
        "role": "user",
        "content": [{"type": "tool_result", "tool_use_id": tool_call.id, "content": resultado}],
    })

    response = client.messages.create(model=MODEL, tools=tools, messages=messages)  # el LLM sigue, ya con el dato real
```

Es exactamente el mecanismo de bajo nivel detrás del [loop de ReAct](agentes-vs-workflows.md#patrón-de-agent-el-llm-controla-el-camino) que ya vimos en pseudocódigo — acá está con el formato real de mensajes yendo y viniendo.

## Multi-turn y tool calls en paralelo

Una tarea puede necesitar varias vueltas de este loop (llamar una tool, ver el resultado, decidir llamar otra) antes de dar la respuesta final — no hay un límite fijo de pasos, el LLM decide cuándo ya tiene lo que necesita (por eso conviene un [Circuit Breaker](../system-design/quality-attributes.es.md#tolerancia-a-fallos) si algo se cuelga reintentando, ver [Costos de LLMs](costos-llms.md#evitar-gasto-por-loops-que-no-cortan-solos)). Algunos modelos también pueden pedir **varias tool calls en la misma respuesta** (ej. "necesito el clima de 3 ciudades") para ejecutarlas en paralelo en vez de una por una.

## Function calling vs MCP

Function calling es el **mecanismo**: cómo el LLM pide ejecutar algo y recibe el resultado, parte del contrato de la API del modelo. [MCP](mcp.md) es un **protocolo de más alto nivel** que estandariza cómo se exponen esas tools entre aplicaciones distintas, para no reinventar la integración con cada servicio externo — un servidor MCP, por debajo, termina generando exactamente el tipo de schema de tools que function calling necesita.

## Por qué importa

Function calling es lo que separa a un LLM que "solo conversa" de uno que puede **actuar** sobre el mundo real (consultar una DB, mandar un email, ejecutar código) — sin este mecanismo, el resto de agentic (ReAct, MCP, orchestrator-workers) no tendría forma de tocar nada fuera de la conversación.

---
Relacionado: [De ML clásico a Agentic AI](historia-de-ml-a-agentic.md#7-tool-use--function-calling--el-llm-puede-hacer-no-solo-hablar-2023), [Agentes vs Workflows](agentes-vs-workflows.md), [MCP](mcp.md), [Diseño de Agentes](diseno-de-agentes.md), [Costos de LLMs](costos-llms.md).
