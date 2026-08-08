# Escalabilidad de CPU

Cuando el cuello de botella es cómputo, no espera — la [causa y el diagnóstico](../diagnostico/backend.md#cpu-bound) ya están en `diagnostico/backend.md`. Acá el foco es la pregunta de escalabilidad: una vez identificado que el problema es CPU, ¿qué hacer?

## Vertical: más cores, mismo proceso

Sirve si el trabajo ya está bien paralelizado (usa múltiples threads/workers dentro del proceso) — más cores disponibles significa más paralelismo real. Si el trabajo corre en un solo hilo sin paralelizar, agregar cores no ayuda: ese hilo sigue limitado a un solo core sin importar cuántos haya libres al lado.

## Horizontal: más procesos

Cuando ya no tiene sentido seguir agregando cores a una sola máquina (o el trabajo no paraleliza bien dentro de un proceso), la alternativa es correr **más procesos/instancias** en paralelo, cada uno usando su propio set de cores — ver [Clustering](../diagnostico/backend.md#clustering) para el patrón dentro de una sola máquina, y [Escalabilidad de Procesos](../devops/escalabilidad-procesos.md) para llevarlo a múltiples máquinas.

## Antes de escalar: ¿es realmente CPU insuficiente?

Escalar (vertical u horizontal) tiene sentido cuando el algoritmo ya está razonablemente optimizado — agregar cores a un algoritmo `O(n²)` que debería ser `O(n log n)` es pagar infraestructura para tapar un problema de código (ver [Algoritmos no optimizados](../diagnostico/backend.md#algoritmos-no-optimizados)). Confirmar con un profiler qué función consume el tiempo antes de decidir escalar.

---
Relacionado: [Diagnóstico Backend](../diagnostico/backend.md#cpu-bound), [Escalabilidad vertical vs horizontal](../devops/escalabilidad-vertical-horizontal.md), [Escalabilidad de Procesos](../devops/escalabilidad-procesos.md).
