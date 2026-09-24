# Python — Concurrencia y Memoria

## GIL (Global Interpreter Lock)

Un lock interno del intérprete de CPython (la implementación estándar de Python) que garantiza que **solo un hilo ejecuta bytecode de Python a la vez**, incluso en una máquina con muchos cores. Es la razón por la que el módulo `threading` de Python no acelera trabajo CPU-bound — ver [Escalabilidad de CPU](../../backend/scaling-cpu.es.md).

```python
# ❌ 4 threads, pero el GIL impide que corran bytecode Python en paralelo real —
# para trabajo CPU-bound, esto no es más rápido que un solo thread
import threading
threads = [threading.Thread(target=cpu_heavy_function) for _ in range(4)]
```

**Dónde sí ayuda `threading` a pesar del GIL**: en trabajo I/O-bound (esperar una respuesta de red, leer un archivo) — mientras un thread espera una operación I/O, el GIL se libera y otro thread puede correr. Por eso `threading` en Python sirve para I/O concurrente, pero no para paralelismo real de CPU — para eso hace falta `multiprocessing`, que usa procesos separados, cada uno con su propio intérprete y su propio GIL.

## Garbage Collection

CPython usa dos mecanismos combinados:

- **Reference counting**: cada objeto lleva un contador de cuántas referencias apuntan a él. Cuando el contador llega a 0, se libera inmediatamente — es el mecanismo principal, determinístico.
- **Garbage collector generacional**: resuelve el caso que el reference counting solo no puede — **referencias circulares** (`a` referencia a `b`, `b` referencia a `a`, ninguno de los dos llega nunca a 0 aunque nada externo los use ya). El GC generacional corre periódicamente y detecta estos ciclos.

```python
a = []
b = [a]
a.append(b)     # referencia circular: a → b → a
# el reference count de a y b no baja a 0 nunca solo, aunque nada externo los use —
# el garbage collector generacional es el que eventualmente libera este ciclo
```

---
Relacionado: [Escalabilidad de CPU](../../backend/scaling-cpu.es.md), [Escalabilidad de Procesos](../../devops/scaling-processes.es.md), [Diagnóstico Backend](../../diagnostics/backend.es.md#memoria) (memory leaks).
