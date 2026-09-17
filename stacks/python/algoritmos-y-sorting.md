# Python — Algoritmos, Sorting y Estructuras de Datos

Aplicación práctica de [Big-O](../../system-design/big-o.md) a las estructuras y funciones built-in de Python — qué estructura/función elegir para que el código no se vuelva el cuello de botella con datasets grandes.

## Big-O de las estructuras built-in — lo que hay que saber de memoria

| Operación | `list` | `dict` / `set` |
|---|---|---|
| Acceso por índice/key | O(1) | O(1) |
| Buscar un valor (`x in coleccion`) | O(n) | O(1) |
| Insertar al final | O(1) amortizado | O(1) |
| Insertar al principio | O(n) | — |
| `sorted()` / `.sort()` | O(n log n) | — |

El caso que más se pregunta: buscar si un elemento existe con `in` — en una `list` recorre todo (O(n)); en un `set`/`dict` es hash lookup (O(1)). Con datasets grandes, cambiar `if x in lista` por `if x in set(lista)` es la optimización más barata que existe.

```python
# ❌ O(n) por cada búsqueda — con m búsquedas sobre n elementos, O(n*m)
ids_validos = [1, 5, 20, 35, ...]  # lista de 10.000 elementos
if user_id in ids_validos:
    ...

# ✅ O(1) por búsqueda — el set se arma una vez, O(n), y cada lookup es instantáneo
ids_validos_set = set(ids_validos)
if user_id in ids_validos_set:
    ...
```

## Operaciones de conjunto (`set`)

```python
a = {1, 2, 3}
b = {2, 3, 4}

a & b     # {2, 3} — intersección
a | b     # {1, 2, 3, 4} — unión
a - b     # {1} — diferencia (está en a pero no en b)
a ^ b     # {1, 4} — diferencia simétrica (lo que no está en ambos)
```

Resolver esto a mano con `list` es O(n*m) (comparar cada elemento de una contra todos los de la otra); con `set`, cada operación es O(min(len(a), len(b))) — otra razón, además del lookup O(1) de arriba, para preferir `set` cuando la tarea es **comparar** colecciones, no solo guardar valores.

## `sorted()` vs `.sort()`

```python
lista = [3, 1, 2]
lista.sort()               # ordena in-place, devuelve None
nueva = sorted(lista)      # devuelve una lista nueva, no toca la original — funciona con cualquier iterable
```

`sorted()` es la opción segura cuando no se quiere mutar la colección original o cuando el input no es una `list` (una `tuple`, un `dict.items()`, un generator).

## `key=` — ordenar por un criterio custom

```python
personas = [{"nombre": "Ana", "edad": 30}, {"nombre": "Beto", "edad": 25}]

sorted(personas, key=lambda p: p["edad"])              # ordena por edad, ascendente
sorted(personas, key=lambda p: p["edad"], reverse=True)  # descendente

# múltiples criterios: primero por edad, después por nombre (tupla como key)
sorted(personas, key=lambda p: (p["edad"], p["nombre"]))
```

`key=` evalúa la función **una vez por elemento** y ordena según ese resultado — más eficiente que un comparador tradicional (`cmp`, que ya ni existe en Python 3) porque no recalcula la clave en cada comparación par a par.

## Ordenamiento estable (Timsort)

El algoritmo de `sorted()`/`.sort()` en CPython es **Timsort**, y es **estable**: elementos con la misma key mantienen su orden relativo original. Importa para ordenamientos en varios pasos — ordenar primero por nombre y después por edad deja, dentro de cada edad, el orden alfabético ya aplicado.

```python
# ordenar en dos pasos aprovechando estabilidad, en vez de una key compuesta
personas.sort(key=lambda p: p["nombre"])   # 1er paso: por nombre
personas.sort(key=lambda p: p["edad"])     # 2do paso: por edad — el orden por nombre sobrevive dentro de cada edad
```

## Datos time-based: ordenar y filtrar por fecha/hora

```python
from datetime import datetime

eventos = [{"ts": datetime(2026, 1, 5)}, {"ts": datetime(2026, 1, 1)}]
eventos_ordenados = sorted(eventos, key=lambda e: e["ts"])   # datetime es comparable directamente, sin parsear a mano

# ventana deslizante de "últimos N eventos" — ver deque en collections-module.md
```

`datetime` es comparable de forma nativa (`<`, `>`, `sorted()`) porque implementa los dunder methods de comparación — no hace falta convertir a timestamp para ordenar por fecha.

## `heapq` — top-N sin ordenar todo

```python
import heapq

numeros = [5, 1, 9, 3, 7, 2]
heapq.nlargest(3, numeros)    # [9, 7, 5] — los 3 más grandes
heapq.nsmallest(3, numeros)    # [1, 2, 3] — los 3 más chicos
```

Para encontrar los top-N de una colección grande, `heapq.nlargest`/`nsmallest` es O(n log k) — más eficiente que `sorted(numeros)[-3:]`, que ordena la colección **entera** (O(n log n)) solo para descartar casi todo después.

## `bisect` — búsqueda binaria sobre una lista ya ordenada

```python
import bisect

nums = [1, 3, 4, 4, 6, 8]

bisect.bisect_left(nums, 4)     # 2 — primer índice donde insertar 4 sin romper el orden (antes de los 4 existentes)
bisect.bisect_right(nums, 4)    # 4 — después de los 4 existentes

bisect.insort(nums, 5)          # inserta 5 manteniendo el orden, sin ordenar todo de nuevo
nums                              # [1, 3, 4, 4, 5, 6, 8]
```

O(log n) para encontrar el punto de inserción — mucho más rápido que agregar un elemento y volver a ordenar todo (O(n log n)) cada vez que se necesita insertar manteniendo el orden. Requiere que la lista ya esté ordenada de antemano — `bisect` no ordena, solo busca dónde insertar.

---
Relacionado: [Big-O](../../system-design/big-o.md), [Módulo `collections`](collections-module.md) (`deque` para ventanas de eventos), [Índices](../../database/indices.md) (mismo espíritu de Big-O, a nivel de DB).
