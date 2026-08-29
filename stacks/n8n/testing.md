# n8n — Cómo se testea

n8n no tiene un framework de tests tipo [pytest](../../system-design/testing.md) ni impone una arquitectura de código sobre la que escribir unit tests. La **unidad testeable es el workflow completo**, no una función aislada — y lo que hay para probarlo es manual/integración, apoyado en herramientas que trae la propia UI, no un test runner separado.

## Ejecución manual — paso a paso o completa

Desde el editor se puede correr el workflow entero (**Execute Workflow**) o un solo nodo (**Execute Step**), viendo el JSON de entrada y salida de cada nodo en el momento. Es el equivalente más cercano a "correr el código y mirar qué devuelve" — pero manual, no algo que se automatice y corra solo en cada cambio.

## Pinned data — fijar el output de un nodo

Se puede "pinnear" el resultado de un nodo (ej. la respuesta de una API externa) para que las próximas ejecuciones usen ese dato guardado en vez de volver a llamar al servicio real.

```
Nodo "HTTP Request" (llama a una API externa)
  → primera ejecución: llama de verdad, trae la respuesta real
  → pineás esa respuesta
  → ejecuciones siguientes: usan el dato pinneado, no vuelven a pegarle a la API

Sirve para iterar rápido en los nodos SIGUIENTES sin gastar cuota,
esperar latencia real, ni depender de que el servicio externo esté arriba
```

Es conceptualmente parecido a un [mock](../../system-design/testing.md#2-test-doubles--mock-vs-stub-vs-fake-vs-spy) — reemplazar una dependencia externa por un valor fijo, conocido, para poder probar el resto de la lógica de forma aislada y repetible.

## Execution history — el log de qué pasó

Cada ejecución (manual o disparada por un trigger real) queda guardada con el input/output de cada nodo, y si falló, en qué nodo exactamente y con qué error. Es la herramienta principal para debuggear después de que algo salió mal en producción — no previene el error, pero da visibilidad completa de la causa.

## Error workflow — la forma más cercana a manejo automático de fallos

Se puede configurar un workflow separado que se dispara automáticamente cuando otro workflow falla — típicamente para alertar (Slack, email) o loggear el fallo en algún lado. No es un test, es manejo de errores en producción, pero es lo más parecido a una red de seguridad automatizada que ofrece n8n de fábrica.

## Si hace falta algo más cercano a CI real

n8n no lo da de fábrica, pero se puede armar: exportar el workflow a JSON, y correrlo desde la **n8n CLI** contra datos de prueba dentro de un pipeline de CI, comparando el output contra lo esperado a mano.

```bash
n8n execute --id <workflow_id>   # corre un workflow desde la terminal, sin la UI
```

Es un approach casero (arma el equipo, no viene integrado) — para casos donde el workflow es lo bastante crítico como para justificar ese esfuerzo extra.

## Por qué importa

Esto es, en concreto, lo que está detrás del trade-off que ya mencionamos en [n8n y Agentic AI](n8n-y-agentic.md#trade-off-frente-a-escribir-el-agente-en-código): "testing automatizado real" no es algo que n8n ofrezca nativamente — lo que hay es un conjunto de herramientas manuales/de inspección, útiles para desarrollar e iterar, pero lejos del [test pyramid](../../system-design/testing.md#1-test-pyramid) (unit → integration → e2e) que se arma con código.

---
Relacionado: [Fundamentos de n8n](basico.md), [n8n y Agentic AI](n8n-y-agentic.md), [Testing — conceptos generales](../../system-design/testing.md).
