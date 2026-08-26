# MCP (Model Context Protocol)

Protocolo abierto (creado por Anthropic, adoptado por el resto de la industria) para conectar un LLM con herramientas y fuentes de datos externas de forma estandarizada — es literalmente lo que uso yo mismo en esta conversación para tocar el navegador, leer archivos, o correr comandos.

## El problema que resuelve: integraciones N×M

Antes de MCP, cada combinación de agente + herramienta externa (GitHub, Slack, una base de datos) necesitaba su propia integración custom — si tenías 3 agentes y 3 herramientas, terminabas escribiendo 9 integraciones distintas, cada una frágil y de mantenimiento propio.

```
Sin protocolo estándar:          Con MCP:
Agent A ─┬─ GitHub               Agent A ─┐
Agent B ─┼─ Slack                Agent B ─┼─ MCP ─┬─ GitHub
Agent C ─┴─ Database             Agent C ─┘        ├─ Slack
                                                      └─ Database
N x M integraciones custom       N + M integraciones (una por agente, una por tool)
```

## Arquitectura: Host, Client, Server

- **Host**: la aplicación que usa el LLM (Claude Code, Claude Desktop, Cursor, etc.) — es quien decide qué servidores MCP conectar.
- **Client**: vive dentro del host, mantiene una conexión **1:1** con un servidor y habla el protocolo (JSON-RPC) con él.
- **Server**: expone las capacidades reales — no es el LLM, es el programa que sabe hablar con GitHub, con una base de datos, con el sistema de archivos, etc.

Un servidor MCP puede exponer tres tipos de capacidades:

- **Tools**: funciones que el LLM puede invocar (ej. `create_issue`, `read_file`) — el equivalente a [tool use / function calling](historia-de-ml-a-agentic.md#7-tool-use--function-calling--el-llm-puede-hacer-no-solo-hablar-2023).
- **Resources**: datos que el host puede leer y darle de contexto al LLM (ej. el contenido de un archivo).
- **Prompts**: plantillas de prompt reutilizables que el servidor expone para tareas comunes.

El transporte entre client y server es **JSON-RPC** sobre `stdio` (proceso local) o HTTP/SSE (servidor remoto).

## Por qué importa: un servidor, todos los agentes

La ventaja central es que **un mismo servidor MCP sirve para cualquier host compatible** — quien construye la integración con GitHub la escribe una sola vez, y la puede usar tanto Claude Code como Cursor como cualquier otro agente que hable el protocolo. Es la razón por la que el ecosistema de servidores MCP creció tan rápido: no es "una integración por producto", es "una integración, N productos".

---
Relacionado: [Agentes vs Workflows](agentes-vs-workflows.md#patrón-de-agent-el-llm-controla-el-camino), [Diseño de Agentes](diseno-de-agentes.md), [AGENTS.md y Skills](agents-md-y-skills.md).
