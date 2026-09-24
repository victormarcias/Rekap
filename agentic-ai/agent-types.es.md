# Tipos de Agentes de IA

## Definición general

Un agente es un sistema que **percibe** su entorno, **procesa** esa información, y **actúa** de forma autónoma para lograr un objetivo — sin que un humano tenga que decidir cada paso. El ciclo se repite continuamente:

```
Percepción (recolecta datos: APIs, DB, sensores)
    → Razonamiento (analiza, evalúa opciones, genera un plan)
    → Acción (ejecuta: API calls, escritura, robótica)
    → Aprendizaje (ajusta comportamiento futuro según el resultado)
    → vuelve a Percepción
```

No todos los "agentes" de la taxonomía de abajo hacen las cuatro etapas con la misma sofisticación — es una escala, de reglas fijas a razonamiento abierto.

## La escala completa

### Agentes Reactivos Simples

Los más básicos — reglas condición-acción (`if-then`), **sin memoria** de estados pasados, no planifican a futuro. Reaccionan solo al estado actual.

```python
# ejemplo: termostato — reacciona al dato actual, no recuerda nada de antes
def termostato(temperatura_actual):
    if temperatura_actual < 18:
        return "encender_calefaccion"
    return "apagar_calefaccion"
```

### Agentes Basados en Modelos

Mantienen un **modelo interno del mundo** con memoria de corto plazo — pueden manejar entornos parcialmente observables porque rastrean cómo evolucionó el entorno, no solo el estado presente.

*Ejemplo*: un robot aspirador que recuerda qué zonas ya limpió y dónde están los obstáculos, para no repetir ni chocar.

### Agentes Basados en Objetivos

Además del modelo del mundo, tienen una **meta explícita** y usan algoritmos de búsqueda/planificación para encontrar el camino hacia ella — más flexibles que los reactivos, adaptan la acción según qué tan lejos están del objetivo.

*Ejemplo*: una app de navegación que calcula la ruta óptima a un destino específico.

### Agentes Basados en Utilidad

Cuando hay **múltiples objetivos, a veces en conflicto**, no alcanza con "¿llegué a la meta sí o no?" — hace falta una función de utilidad que mida qué tan buena es cada opción, y elegir la que maximiza la utilidad esperada. Manejan incertidumbre mejor que los basados en objetivos puros.

*Ejemplo*: un auto autónomo balanceando velocidad, seguridad y consumo de combustible a la vez — no hay una sola meta, hay un trade-off entre varias.

### Agentes de Aprendizaje (RL)

Mejoran con la experiencia — combinan un elemento de rendimiento (que actúa) con uno de aprendizaje (que ajusta el comportamiento según el resultado de acciones pasadas). Típicamente vía **reinforcement learning**: el agente prueba acciones, recibe una recompensa o penalización, y ajusta su política para maximizar recompensa futura.

*Ejemplo*: un sistema de recomendación que refina sus sugerencias según cómo interactuás con lo que te mostró antes.

### Agentes Basados en LLM

El **LLM es el motor de razonamiento** — comprenden lenguaje natural complejo, generan planes sofisticados, y hacen razonamiento contextual avanzado sin reglas hardcodeadas para cada situación. Es el tipo de agente que ya desarrollamos en detalle en [Agentes vs Workflows](agents-vs-workflows.es.md) (el loop ReAct) y [n8n](../stacks/n8n/n8n-y-agentic.md) (el nodo AI Agent).

*Ejemplo*: un agente que orquesta un flujo de trabajo completo interpretando instrucciones complejas en lenguaje natural.

### Sistemas Multiagente (MAS)

Múltiples agentes autónomos **interactuando entre sí** — cooperando o compitiendo, resolviendo problemas distribuidos que un solo agente no podría, con necesidad de negociación y coordinación entre ellos.

*Ejemplo*: vehículos autónomos coordinándose en una intersección, o varios agentes especializados (uno busca datos, otro escribe, otro revisa) trabajando en cadena sobre la misma tarea.

---
Relacionado: [Agentes vs Workflows](agents-vs-workflows.es.md), [Diseño de Agentes](agent-design.es.md).
