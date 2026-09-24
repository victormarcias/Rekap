# Python — `collections` Module

Specialized data structures from the stdlib — they solve in one line patterns that with plain `list`/`dict` end up as more code and worse performance.

## `Counter` — counting occurrences

```python
from collections import Counter

words = ["a", "b", "a", "c", "a", "b"]
count = Counter(words)
count               # Counter({'a': 3, 'b': 2, 'c': 1})
count.most_common(2)  # [('a', 3), ('b', 2)] — the 2 most frequent

# ❌ by hand: more code, same result
manual_count = {}
for w in words:
    manual_count[w] = manual_count.get(w, 0) + 1
```

## `defaultdict` — avoids `KeyError` when grouping

```python
from collections import defaultdict

groups = defaultdict(list)   # every new key starts with an empty list, no checking beforehand
for name, category in [("Ana", "A"), ("Beto", "B"), ("Caro", "A")]:
    groups[category].append(name)

groups   # {'A': ['Ana', 'Caro'], 'B': ['Beto']}

# ❌ with a normal dict, you have to check/create the list by hand every time
manual_groups = {}
for name, category in [("Ana", "A"), ("Beto", "B")]:
    if category not in manual_groups:
        manual_groups[category] = []
    manual_groups[category].append(name)
```

## `namedtuple` — a tuple with named fields

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2)
p.x, p.y        # 1, 2 — access by name, not just by index
p[0], p[1]      # 1, 2 — still a tuple, index access also works
```

Lighter than a full class when you just need to group immutable named data — without writing `__init__` by hand. For richer cases (methods, default values, type-based comparison), `@dataclass` is the better choice.

## `deque` — double-ended queue

```python
from collections import deque

d = deque(maxlen=3)          # fixed size: once full, automatically discards the oldest
d.append(1); d.append(2); d.append(3)
d.append(4)                    # discards just the 1 — useful for "last N events"
d                                # deque([2, 3, 4], maxlen=3)

d.appendleft(0)                # O(1) insert at the beginning — a normal list is O(n) here
```

Prefer `deque` over `list` when you insert/remove often from **both** ends — `list.insert(0, x)` and `list.pop(0)` are O(n) (they move every element), `deque.appendleft()`/`popleft()` are O(1).

## `ChainMap` — chaining several dicts without merging them

```python
from collections import ChainMap

defaults = {"theme": "light", "language": "en"}
project_config = {"language": "es"}
user_config = {"theme": "dark"}

config = ChainMap(user_config, project_config, defaults)
config["theme"]      # "dark" — searches in order: user → project → defaults
config["language"]    # "es" — not in user_config, finds it in project_config
```

Searches in priority order without physically copying or merging the dicts — useful for config layers (user > project > defaults), where each level can override the previous one without duplicating data.

## `OrderedDict` — no longer exclusive today

Since Python 3.7 normal `dict`s already keep insertion order — `OrderedDict` lost most of its reason to exist. What it still has exclusively:

```python
from collections import OrderedDict

od = OrderedDict(a=1, b=2, c=3)
od.move_to_end("a")      # moves "a" to the end — a normal dict has no such method
list(od)                   # ['b', 'c', 'a']

OrderedDict(a=1, b=2) == OrderedDict(b=2, a=1)   # False — also compares order
{"a": 1, "b": 2} == {"b": 2, "a": 1}               # True — a normal dict ignores order when comparing
```

---
Related: [General Syntax](syntax.md), [OOP](oop.md) (`dataclass` vs `namedtuple`).
