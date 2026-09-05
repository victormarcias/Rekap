# Agentes vs Workflows

La distinción que importa (viene de cómo Anthropic la formalizó en su propia guía de arquitectura de sistemas con LLMs): no es una diferencia de qué tan "inteligente" es el sistema, es una diferencia de **quién controla el flujo de pasos**.

- **Workflow**: el desarrollador define la secuencia de pasos de antemano, en código o config fija. El LLM resuelve tareas puntuales *dentro* de un camino ya trazado — no decide el orden ni cuántos pasos hay.
- **Agent**: el LLM decide el control flow en tiempo real — cuántos pasos hacen falta, qué herramienta usar en cada uno, cuándo considerar la tarea terminada.

## Patrones de Workflow (el camino es fijo)

**Prompt chaining**: dividir una tarea en pasos secuenciales de LLM, donde el output de uno es el input del siguiente — cada paso es más simple y confiable que pedirle todo de una vez.

```python
# el desarrollador define el orden — el LLM solo ejecuta cada paso
resumen = llm.call(f"Resumí este texto: {texto}")
traduccion = llm.call(f"Traducí esto al inglés: {resumen}")
titulo = llm.call(f"Generá un título corto para: {traduccion}")
```

**Routing**: clasificar el input y mandarlo por un camino especializado según el tipo — en vez de un solo prompt genérico tratando de cubrir todos los casos.

```python
categoria = llm.call(f"Clasificá este ticket: {ticket}")  # "billing" | "technical" | "general"

if categoria == "billing":
    respuesta = llm.call(prompt_especializado_billing, ticket)
elif categoria == "technical":
    respuesta = llm.call(prompt_especializado_technical, ticket)
```

**Orchestrator-workers**: un LLM central parte una tarea grande en subtareas y las delega a LLMs "worker" que las resuelven en paralelo o en secuencia, sin que cada worker sepa nada del resto.

## Patrón de Agent (el LLM controla el camino)

Un loop tipo ReAct (*Reason + Act*): el LLM razona qué hacer, ejecuta una acción (llamar una tool), observa el resultado, y decide el próximo paso — sin que nadie haya predefinido cuántas vueltas va a dar.

```python
while not tarea_terminada:
    paso = llm.call(historial_de_la_conversacion)  # el LLM decide QUÉ hacer ahora
    if paso.necesita_tool:
        resultado = ejecutar_tool(paso.tool_name, paso.args)
        historial_de_la_conversacion.append(resultado)  # observa, y sigue el loop
    else:
        tarea_terminada = True  # el LLM mismo decidió que ya terminó
```

## Cuándo usar cada uno

| | Workflow | Agent |
|---|---|---|
| Los pasos se conocen de antemano | ✅ | No necesariamente |
| Predecibilidad | Alta — mismo input, mismo camino | Baja — el camino puede variar entre ejecuciones |
| Costo | Menor (pasos fijos, sin exploración de más) | Mayor (puede iterar, reintentar, explorar caminos que no sirven) |
| Testeable de forma determinística | ✅ | Difícil — el mismo input puede tomar caminos distintos |
| Tareas abiertas / ambiguas | Se rompe fácil (no contempla lo inesperado) | Es justo para lo que sirve |

**Por qué el costo es mayor en un agent, en concreto**: además de que la cantidad de llamados no es fija, en un loop de agent cada llamado nuevo suele incluir **todo el historial anterior** (qué tools llamó, qué le devolvieron) para que el modelo tenga memoria de qué ya probó — el paso 10 del loop es mucho más caro en tokens que el paso 1, porque arrastra todo lo previo. Un workflow no tiene ese problema: cada paso puede tener un prompt acotado solo a lo que ese paso necesita, sin acumular el historial completo.

**Regla práctica**: si podés escribir el diagrama de flujo antes de empezar, es un workflow — más barato, más confiable, más fácil de debuggear. Si el camino depende genuinamente de lo que se va descubriendo en cada paso (no sabés cuántas búsquedas va a hacer falta, ni en qué orden), ahí es donde un agent aporta algo que un workflow fijo no puede.

## No es una elección binaria

En la práctica, muchos sistemas reales combinan los dos, **en ambas direcciones**:

- **Workflow con un agent adentro**: la estructura general es un workflow (pasos predecibles, guardrails, validaciones), que en un paso puntual delega a un agent cuando ese paso específico necesita razonamiento abierto. Es literalmente lo que permite el nodo AI Agent de [n8n](../stacks/n8n/n8n-y-agentic.md#el-nodo-ai-agent): el workflow sigue siendo la estructura fija de nodos conectados, pero uno de esos nodos internamente corre un loop agentic.
- **Agent con un workflow adentro**: el agent decide llamar a una tool, pero esa tool no es una acción atómica — por dentro corre un pipeline fijo de varios pasos (ej. "procesar pedido" = validar → cobrar → mandar email → actualizar stock). El agent no sabe ni le importa que ahí adentro hay un workflow determinístico; desde su perspectiva es "un llamado a una tool, un resultado". Es el patrón más común en sistemas de producción: no le das al agent control fino de cada paso de bajo nivel (sería más caro y menos confiable), le das una tool de alto nivel que ya encapsula un workflow probado, y el agent orquesta a un nivel más alto.

---
Relacionado: [De ML clásico a Agentic AI](historia-de-ml-a-agentic.md), [Function Calling](function-calling.md), [n8n](../stacks/n8n/n8n-y-agentic.md).
