# PRD y Spec-Driven Development

## Qué es un PRD

**Product Requirement Document**: documento que define qué hay que construir, para quién, y por qué — tradicionalmente escrito por un Product Manager para un equipo humano de ingeniería. Incluye el problema a resolver, el alcance, y criterios de aceptación (cómo se sabe que está terminado).

## Spec-Driven Development (aplicado a agentic coding)

La idea: en vez de pedirle a un agente "hacé una app de tareas" y dejar que interprete todo sobre la marcha, primero se escribe una **spec clara y estructurada** — y el agente trabaja *contra* esa spec, no contra la memoria de lo que se dijo en el chat.

```
Prompt suelto:
"Hacé un endpoint para crear usuarios"
  → el agente interpreta scope, validaciones, errores a su criterio —
    puede o no coincidir con lo que realmente hacía falta

Spec-driven:
spec.md define: campos requeridos, validaciones, códigos de error esperados,
casos de éxito y de falla — el agente implementa CONTRA ese documento,
y el resultado se puede verificar comparándolo con la spec
```

Esto da dos cosas que un prompt suelto no da:

1. El agente puede partir la tarea en pasos **verificables contra la spec** (ver [Plan-and-Execute](diseno-de-agentes.md#plan-and-execute)) — en vez de improvisar el criterio de éxito en cada paso.
2. Quien revisa el resultado puede chequearlo contra un documento fijo, no contra su propia memoria de qué había pedido en el chat.

## PRD vs Spec técnica — no son lo mismo

El PRD es el punto de partida (qué construir, para quién, por qué) — pero en agentic coding, la spec que realmente consume el agente suele ser **más técnica** que un PRD clásico de producto: criterios de aceptación concretos, a veces casos de test explícitos, estructura de datos esperada. En la práctica, muchos workflows arrancan con un PRD y lo van afinando hacia algo más técnico antes de dárselo al agente — el PRD es el insumo, no necesariamente la spec final que el agente ejecuta.

## Por qué importa para agentic

Sin una spec, verificar si un agente "hizo lo correcto" depende de la memoria/criterio de quien lo está usando en ese momento — con una spec escrita, la verificación es objetiva: ¿el resultado cumple lo que dice el documento? Es la misma lógica de fondo que un [PRD tradicional](#qué-es-un-prd) aporta a un equipo humano, aplicada a que un agente tenga algo estable contra qué trabajar en vez de un prompt que se puede reinterpretar cada vez.

---
Relacionado: [Diseño de Agentes](diseno-de-agentes.md), [Agentes vs Workflows](agentes-vs-workflows.md).
