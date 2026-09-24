# n8n y Agentic AI

## El nodo AI Agent

n8n tiene un nodo **AI Agent** (con LangChain integrado por debajo) que le da a un LLM acceso al resto de los nodos del workflow como **tools** — es la versión visual/low-code del mismo patrón de [tool use y agentic](../../agentic-ai/from-ml-to-agentic-ai.es.md#7-tool-use--function-calling--el-llm-puede-hacer-no-solo-hablar-2023) que se arma con código: el LLM decide qué nodo/tool usar, con qué datos, y encadena pasos hasta resolver la tarea. Se configura conectándole tres piezas — Model, Memory, Tools — la misma arquitectura de [Model + Memory + Tools](../../agentic-ai/agent-design.es.md#en-la-práctica-model--memory--tools) que se arma a mano en código.

Es también un ejemplo concreto del patrón "[workflow con un agent adentro](../../agentic-ai/agents-vs-workflows.es.md#no-es-una-elección-binaria)": el workflow en sí sigue siendo la estructura fija de nodos conectados (predecible, con guardrails), pero el nodo AI Agent corre internamente un loop agentic cuando ese paso puntual necesita razonamiento abierto.

## Trade-off frente a escribir el agente en código

n8n es mucho más rápido para prototipar y no requiere que todo el equipo sepa programar — pero lógica compleja, testing automatizado real, control de versiones granular (un workflow visual es más difícil de diffear en un PR que código) y performance crítica se manejan mejor escribiendo el agente directamente.

## Cuándo conviene n8n vs código

- **n8n**: prototipos rápidos, automatizaciones e integraciones entre SaaS sin lógica pesada, equipos donde gente no-dev necesita poder mantener el workflow.
- **Código**: lógica de negocio compleja, necesidad de tests automatizados, control de versiones fino, performance crítica, o un equipo de ingeniería que ya tiene su propio stack y prefiere no depender de una herramienta externa para su producto principal.

---
Relacionado: [Fundamentos de n8n](basico.md), [Cómo se testea](testing.md), [Agentes vs Workflows](../../agentic-ai/agents-vs-workflows.es.md), [Tipos de Agentes de IA](../../agentic-ai/agent-types.es.md), [De ML clásico a Agentic AI](../../agentic-ai/from-ml-to-agentic-ai.es.md).
