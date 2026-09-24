# AGENTS.md y Skills

Dos piezas del "harness" agentic (todo lo que rodea al LLM para que sea útil en un repo real, más allá del modelo en sí) que resuelven un mismo problema de fondo: **un LLM no tiene memoria entre conversaciones** — cada charla nueva arranca de cero, sin saber nada de tu proyecto ni de lo que ya se decidió ayer.

## AGENTS.md — el problema del "blank slate"

Cada vez que abrís una conversación nueva con un agente de código, el modelo no sabe: tu stack, tus convenciones, tus comandos de build, ni nada de lo que se discutió en sesiones anteriores. Sin ese contexto, el agente **adivina** — y adivina mal, de formas que terminan costando más tiempo del que ahorran.

**AGENTS.md** es un archivo de texto en la raíz del repo (o en cualquier subcarpeta) que le da al agente ese contexto de entrada: stack tecnológico, comandos para correr tests/build, convenciones del equipo, cosas a evitar. El agente lo lee automáticamente al arrancar, así no hay que repetir lo mismo en cada conversación.

```markdown
# AGENTS.md (ejemplo mínimo)

## Stack
FastAPI + PostgreSQL + React. Python 3.12, Node 20.

## Comandos
- Tests: `pytest`
- Build frontend: `npm run build`

## Convenciones
- Nunca usar `git push --force` sin confirmar antes.
- Los endpoints nuevos van en `api/routes/`, con su test correspondiente.
```

Es un formato **abierto**, adoptado por múltiples herramientas de código agentic (no es específico de una sola) — la idea es que un mismo `AGENTS.md` sirva sin importar qué agente estés usando ese día.

## Skills — capacidades modulares

Un **Skill** empaqueta instrucciones + metadata + recursos en una unidad autocontenida que el agente puede invocar cuando la tarea lo amerita — el equivalente a un plugin: se instala una vez, y queda disponible en cualquier conversación futura sin volver a explicar cómo hacer esa tarea.

**Anatomía de un Skill**: un archivo `SKILL.md` con dos partes.

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

1. **YAML frontmatter**: metadata — el `name` se vuelve el comando (`/deploy-preview`).
2. **Cuerpo en Markdown**: instrucciones paso a paso de cómo ejecutar la tarea.
3. La carpeta que contiene el `SKILL.md` es, por convención, el nombre del skill.

**Invocación**: explícita (el usuario escribe `/deploy-preview`) o automática (el agente detecta que la tarea actual matchea la descripción del skill y lo dispara solo) — en cualquier caso, el skill queda "dormido" hasta que hace falta, sin ocupar contexto de la conversación de antemano.

**Dónde viven**: a nivel personal (disponibles en cualquier proyecto que abras) o a nivel de proyecto (committeados al repo, compartidos con el equipo) — la diferencia es la misma que entre una preferencia tuya y una convención del equipo.

## La diferencia con MCP

Un [servidor MCP](mcp.es.md) le da al agente **acceso a un sistema externo** (una API, una base de datos). Un Skill le da al agente **una receta de cómo hacer algo** — puede usar herramientas MCP en el proceso, pero el Skill en sí es solo instrucciones, no una integración técnica nueva. AGENTS.md, a su vez, es contexto pasivo (el agente lo lee, no lo "ejecuta") — Skills son capacidades activas que el agente decide invocar.

---
Relacionado: [MCP](mcp.es.md), [PRD y Spec-Driven Development](prd-and-spec-driven-development.es.md), [Diseño de Agentes](agent-design.es.md).
