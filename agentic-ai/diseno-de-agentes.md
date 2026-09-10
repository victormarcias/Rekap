# Diseño de Agentes de IA

## Componentes centrales

Cualquier agente, sin importar su tipo (ver [Tipos de Agentes](tipos-de-agentes.md)), se arma con estas piezas:

- **Percepción**: procesamiento de los datos de entrada (texto, imágenes, eventos) — multimodal si hace falta.
- **Razonamiento**: el motor de decisiones — puede ser un LLM, un modelo de ML clásico, o reglas fijas.
- **Memoria**: estado de corto plazo (la conversación actual) y de largo plazo (qué pasó en interacciones anteriores) — sin esto, el agente "olvida" todo entre pasos.
- **Acción**: la ejecución real — conectar con APIs, sistemas externos, bases de datos.
- **Retroalimentación**: monitoreo del resultado, para aprender y ajustar el comportamiento futuro.

## Arquitectura en capas

```
Capa de Entrada       → APIs | Sensores | UI | Eventos
      ↓
Capa de Procesamiento → LLM | Motor de reglas | Modelos de ML | Memoria
      ↓
Capa de Decisión      → Planificador | Evaluador | Selector de acciones
      ↓
Capa de Salida        → Ejecutor | Integraciones | Actuadores
      ↑
      └── Ciclo de retroalimentación (vuelve a alimentar la Capa de Entrada)
```

**Principios clave** del diseño: **modularidad** (cada capa se puede reemplazar sin tocar las demás — cambiar de LLM no debería romper la capa de entrada), **escalabilidad**, y **observabilidad** (poder ver qué decidió el agente y por qué, no una caja negra — ver [Transparencia en Riesgos y Mitigaciones](riesgos-y-mitigaciones.md#mitigaciones-técnicas)).

## Patrones de ejecución

### Plan-and-Execute

Planificación y ejecución **separadas** en dos fases: primero se arma un plan de alto nivel completo, después se ejecuta cada paso del plan en orden.

```python
plan = llm.call(f"Armá un plan de pasos para: {tarea}")  # fase 1: planificar todo de una
for paso in plan.pasos:
    ejecutar(paso)  # fase 2: ejecutar en orden, sin volver a replanificar en el medio
```

Ideal para tareas **estructuradas** donde los pasos se pueden prever de antemano — predecible y fácil de debuggear, porque el plan completo existe antes de tocar nada.

### ReAct (Reason + Act)

Ciclo iterativo de razonar y actuar, un paso a la vez — ya desarrollado en detalle en [Agentes vs Workflows](agentes-vs-workflows.md#patrón-de-agent-el-llm-controla-el-camino). Ideal para entornos dinámicos/exploratorios, donde no se puede planificar todo de antemano porque cada paso depende del resultado del anterior.

**Plan-and-Execute vs ReAct**: el primero decide todo el camino antes de dar el primer paso; el segundo decide un paso, ve qué pasó, y recién ahí decide el siguiente. Plan-and-Execute es más predecible pero más rígido — si algo sale distinto a lo planeado a mitad de camino, no se auto-corrige tan bien como ReAct.

### Human-in-the-Loop (HITL)

Integra intervención humana explícita dentro del flujo — no reemplaza al agente, lo complementa en los puntos donde el riesgo de una decisión autónoma es demasiado alto.

- **Supervisor**: valida decisiones críticas antes de que se ejecuten.
- **Validador**: da feedback sobre un resultado ya generado.
- **Experto**: interviene en excepciones o ambigüedades que el agente no puede resolver solo.

```python
def ejecutar_con_hitl(accion_propuesta):
    if accion_propuesta.es_critica:  # ej. borrar datos, mandar plata, publicar algo
        aprobacion = pedir_aprobacion_humana(accion_propuesta)
        if not aprobacion:
            return "cancelado"
    return ejecutar(accion_propuesta)
```

Crítico en dominios de alto riesgo (salud, finanzas) — aumenta la confiabilidad a costa de perder algo de la autonomía completa que promete un agent puro.

---
Relacionado: [Tipos de Agentes](tipos-de-agentes.md), [Agentes vs Workflows](agentes-vs-workflows.md), [Prompt Engineering](prompt-engineering.md#estructura-de-un-system-prompt), [Riesgos y Mitigaciones](riesgos-y-mitigaciones.md).
