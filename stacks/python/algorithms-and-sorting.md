# Python — Algorithms, Sorting, and Data Structures

Practical application of [Big-O](../../system-design/big-o.md) to Python's built-in structures and functions — which structure/function to choose so the code doesn't become the bottleneck with large datasets.

## Big-O of built-in structures — what to know by heart

| Operation | `list` | `dict` / `set` |
|---|---|---|
| Access by index/key | O(1) | O(1) |
| Search for a value (`x in collection`) | O(n) | O(1) |
| Insert at the end | O(1) amortized | O(1) |
| Insert at the beginning | O(n) | — |
| `sorted()` / `.sort()` | O(n log n) | — |

The most commonly asked case: checking if an element exists with `in` — in a `list` it scans everything (O(n)); in a `set`/`dict` it's a hash lookup (O(1)). With large datasets, changing `if x in list` to `if x in set(list)` is the cheapest optimization there is.

```python
# ❌ O(n) per lookup — with m lookups over n elements, O(n*m)
valid_ids = [1, 5, 20, 35, ...]  # a list of 10,000 elements
if user_id in valid_ids:
    ...

# ✅ O(1) per lookup — the set is built once, O(n), and each lookup is instant
valid_ids_set = set(valid_ids)
if user_id in valid_ids_set:
    ...
```

## Set operations (`set`)

```python
a = {1, 2, 3}
b = {2, 3, 4}

a & b     # {2, 3} — intersection
a | b     # {1, 2, 3, 4} — union
a - b     # {1} — difference (in a but not in b)
a ^ b     # {1, 4} — symmetric difference (what's not in both)
```

Solving this by hand with `list` is O(n*m) (comparing each element of one against all of the other); with `set`, each operation is O(min(len(a), len(b))) — another reason, besides the O(1) lookup above, to prefer `set` when the task is **comparing** collections, not just storing values.

## `sorted()` vs `.sort()`

```python
lst = [3, 1, 2]
lst.sort()               # sorts in-place, returns None
new = sorted(lst)      # returns a new list, doesn't touch the original — works with any iterable
```

`sorted()` is the safe choice when you don't want to mutate the original collection or when the input isn't a `list` (a `tuple`, a `dict.items()`, a generator).

## `key=` — sorting by a custom criterion

```python
people = [{"name": "Ana", "age": 30}, {"name": "Beto", "age": 25}]

sorted(people, key=lambda p: p["age"])              # sorts by age, ascending
sorted(people, key=lambda p: p["age"], reverse=True)  # descending

# multiple criteria: first by age, then by name (tuple as key)
sorted(people, key=lambda p: (p["age"], p["name"]))
```

`key=` evaluates the function **once per element** and sorts based on that result — more efficient than a traditional comparator (`cmp`, which doesn't even exist in Python 3 anymore) because it doesn't recompute the key on every pairwise comparison.

## Stable sorting (Timsort)

CPython's `sorted()`/`.sort()` algorithm is **Timsort**, and it's **stable**: elements with the same key keep their original relative order. Matters for multi-step sorts — sorting first by name and then by age leaves, within each age, the alphabetical order already applied.

```python
# sort in two steps taking advantage of stability, instead of a compound key
people.sort(key=lambda p: p["name"])   # step 1: by name
people.sort(key=lambda p: p["age"])     # step 2: by age — the name order survives within each age
```

## Time-based data: sorting and filtering by date/time

```python
from datetime import datetime

events = [{"ts": datetime(2026, 1, 5)}, {"ts": datetime(2026, 1, 1)}]
sorted_events = sorted(events, key=lambda e: e["ts"])   # datetime is directly comparable, no manual parsing needed

# sliding window of "last N events" — see deque in collections-module.md
```

`datetime` is natively comparable (`<`, `>`, `sorted()`) because it implements the comparison dunder methods — no need to convert to a timestamp to sort by date.

## `heapq` — top-N without sorting everything

```python
import heapq

numbers = [5, 1, 9, 3, 7, 2]
heapq.nlargest(3, numbers)    # [9, 7, 5] — the 3 largest
heapq.nsmallest(3, numbers)    # [1, 2, 3] — the 3 smallest
```

To find the top-N of a large collection, `heapq.nlargest`/`nsmallest` is O(n log k) — more efficient than `sorted(numbers)[-3:]`, which sorts the **entire** collection (O(n log n)) just to discard almost all of it afterward.

## `bisect` — binary search over an already-sorted list

```python
import bisect

nums = [1, 3, 4, 4, 6, 8]

bisect.bisect_left(nums, 4)     # 2 — first index where 4 can be inserted without breaking the order (before the existing 4s)
bisect.bisect_right(nums, 4)    # 4 — after the existing 4s

bisect.insort(nums, 5)          # inserts 5 keeping the order, without re-sorting everything
nums                              # [1, 3, 4, 4, 5, 6, 8]
```

O(log n) to find the insertion point — much faster than adding an element and re-sorting everything (O(n log n)) every time you need to insert while keeping the order. Requires the list to already be sorted beforehand — `bisect` doesn't sort, it only searches for where to insert.

---
Related: [Big-O](../../system-design/big-o.md), [`collections` Module](collections-module.md) (`deque` for event windows), [Indexes](../../database/indexes.md) (the same Big-O spirit, at the DB level).
