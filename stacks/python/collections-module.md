# Python — Módulo `collections`

Estructuras de datos especializadas de la stdlib — resuelven en una línea patrones que con `list`/`dict` puros terminan en más código y peor performance.

## `Counter` — contar ocurrencias

```python
from collections import Counter

palabras = ["a", "b", "a", "c", "a", "b"]
conteo = Counter(palabras)
conteo               # Counter({'a': 3, 'b': 2, 'c': 1})
conteo.most_common(2)  # [('a', 3), ('b', 2)] — los 2 más frecuentes

# ❌ a mano: más código, mismo resultado
conteo_manual = {}
for p in palabras:
    conteo_manual[p] = conteo_manual.get(p, 0) + 1
```

## `defaultdict` — evita el `KeyError` al agrupar

```python
from collections import defaultdict

grupos = defaultdict(list)   # cada key nueva arranca con una lista vacía, sin chequear antes
for nombre, categoria in [("Ana", "A"), ("Beto", "B"), ("Caro", "A")]:
    grupos[categoria].append(nombre)

grupos   # {'A': ['Ana', 'Caro'], 'B': ['Beto']}

# ❌ con dict normal, hay que chequear/crear la lista a mano cada vez
grupos_manual = {}
for nombre, categoria in [("Ana", "A"), ("Beto", "B")]:
    if categoria not in grupos_manual:
        grupos_manual[categoria] = []
    grupos_manual[categoria].append(nombre)
```

## `namedtuple` — tupla con nombres de campo

```python
from collections import namedtuple

Punto = namedtuple("Punto", ["x", "y"])
p = Punto(1, 2)
p.x, p.y        # 1, 2 — acceso por nombre, no solo por índice
p[0], p[1]      # 1, 2 — sigue siendo una tupla, también funciona por índice
```

Más liviano que una clase completa cuando solo hace falta agrupar datos inmutables con nombre — sin escribir `__init__` a mano. Para casos más ricos (métodos, valores default, comparación por tipo) conviene `@dataclass` en su lugar.

## `deque` — cola de doble punta

```python
from collections import deque

d = deque(maxlen=3)          # tamaño fijo: al llenarse, descarta automáticamente el más viejo
d.append(1); d.append(2); d.append(3)
d.append(4)                    # descarta el 1 solo — útil para "últimos N eventos"
d                                # deque([2, 3, 4], maxlen=3)

d.appendleft(0)                # O(1) insertar al principio — una list normal es O(n) acá
```

Preferir `deque` sobre `list` cuando se inserta/saca seguido de **ambos** extremos — `list.insert(0, x)` y `list.pop(0)` son O(n) (mueven todos los elementos), `deque.appendleft()`/`popleft()` son O(1).

---
Relacionado: [Sintaxis general](sintaxis.md), [OOP](oop.md) (`dataclass` vs `namedtuple`).
