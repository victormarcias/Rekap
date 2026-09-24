# Python — Concurrency and Memory

## GIL (Global Interpreter Lock)

An internal lock in the CPython interpreter (Python's standard implementation) that guarantees **only one thread executes Python bytecode at a time**, even on a machine with many cores. It's the reason Python's `threading` module doesn't speed up CPU-bound work — see [CPU Scalability](../../backend/scaling-cpu.md).

```python
# ❌ 4 threads, but the GIL prevents them from running Python bytecode in real parallel —
# for CPU-bound work, this isn't faster than a single thread
import threading
threads = [threading.Thread(target=cpu_heavy_function) for _ in range(4)]
```

**Where `threading` does help despite the GIL**: in I/O-bound work (waiting on a network response, reading a file) — while a thread waits on an I/O operation, the GIL is released and another thread can run. That's why `threading` in Python is useful for concurrent I/O, but not for real CPU parallelism — for that you need `multiprocessing`, which uses separate processes, each with its own interpreter and its own GIL.

## Garbage Collection

CPython uses two combined mechanisms:

- **Reference counting**: every object carries a counter of how many references point to it. When the counter hits 0, it's freed immediately — it's the main, deterministic mechanism.
- **Generational garbage collector**: solves the case reference counting alone can't — **circular references** (`a` references `b`, `b` references `a`, neither ever hits 0 even though nothing external uses them anymore). The generational GC runs periodically and detects these cycles.

```python
a = []
b = [a]
a.append(b)     # circular reference: a → b → a
# the reference count of a and b never drops to 0 on its own, even though nothing external uses them —
# the generational garbage collector is what eventually frees this cycle
```

---
Related: [CPU Scalability](../../backend/scaling-cpu.md), [Process Scalability](../../devops/scaling-processes.md), [Backend Diagnostics](../../diagnostics/backend.md#memory) (memory leaks).
