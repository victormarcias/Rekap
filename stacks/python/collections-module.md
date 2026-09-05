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

## `ChainMap` — encadenar varios dicts sin mergearlos

```python
from collections import ChainMap

defaults = {"tema": "claro", "idioma": "es"}
config_proyecto = {"idioma": "en"}
config_usuario = {"tema": "oscuro"}

config = ChainMap(config_usuario, config_proyecto, defaults)
config["tema"]      # "oscuro" — busca en orden: usuario → proyecto → defaults
config["idioma"]    # "en" — no está en config_usuario, lo encuentra en config_proyecto
```

Busca en orden de prioridad sin copiar ni mergear los dicts físicamente — útil para capas de configuración (usuario > proyecto > defaults), donde cada nivel puede sobreescribir al anterior sin duplicar datos.

## `OrderedDict` — hoy, ya no exclusivo

Desde Python 3.7 los `dict` normales ya mantienen el orden de inserción — `OrderedDict` perdió la mayor parte de su razón de ser. Lo que le queda de exclusivo:

```python
from collections import OrderedDict

od = OrderedDict(a=1, b=2, c=3)
od.move_to_end("a")      # mueve "a" al final — un dict normal no tiene este método
list(od)                   # ['b', 'c', 'a']

OrderedDict(a=1, b=2) == OrderedDict(b=2, a=1)   # False — compara también el orden
{"a": 1, "b": 2} == {"b": 2, "a": 1}               # True — un dict normal ignora el orden al comparar
```

---
Relacionado: [Sintaxis general](sintaxis.md), [OOP](oop.md) (`dataclass` vs `namedtuple`).
