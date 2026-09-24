# Python — Types and Mutability

## Mutable vs immutable

| Type | Mutable | Ordered | Example |
|---|---|---|---|
| `list` | ✅ | ✅ | `[1, 2, 3]` |
| `tuple` | ❌ | ✅ | `(1, 2, 3)` |
| `dict` | ✅ | Yes (since 3.7) | `{"a": 1, "b": 2}` |
| `set` | ✅ | ❌ | `{1, 2, 3}` |
| `frozenset` | ❌ | ❌ | `frozenset({1, 2, 3})` |
| `str` | ❌ | ✅ | `"hello"` |
| `int` / `float` / `bool` | ❌ | — | `42`, `3.14`, `True` |

```python
a = [1, 2, 3]
a.append(4)       # ✅ modifies the same list, same reference (id(a) doesn't change)

b = "hello"
b += " world"      # ❌ doesn't modify the original string — creates a NEW string and b points to that new object

person = {"name": "Ana", "age": 30}
person["age"] = 31             # dicts do get mutated
person.get("email", "N/A")     # safe access: returns "N/A" instead of raising KeyError

unique = {1, 2, 2, 3}            # {1, 2, 3} — a set discards duplicates automatically, because it's mutable but unordered
```

Why it matters: a mutable object passed as an argument to a function can be modified by that function and the change "shows up" outside; an immutable one can't — see [Shallow copy vs Deep copy](../../system-design/quality-attributes.md#shallow-copy-vs-deep-copy) for the more subtle case (mutating something nested inside a "new" copy).

## `is` vs `==`

- **`==`**: compares **value** — do they have the same content?
- **`is`**: compares **identity** — are they the same object in memory (same `id()`)?

```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b   # True — same content
a is b   # False — two different objects in memory, even though they look the same

c = a
c is a   # True — c is literally the same object as a, not a copy
```

**The special case of `None`**: always compared with `is None`, not `== None` — by convention, and because `is` is explicit about what actually matters (identity), without depending on a custom `__eq__` someone might have overridden.

## The classic gotcha: mutable default arguments

```python
# ❌ the default {} is created ONCE, when the function is defined —
# not once per call — so every call with no argument shares the SAME dict
def add_item(item, cart={}):
    cart[item] = True
    return cart

add_item("apple")   # {'apple': True}
add_item("banana")    # {'apple': True, 'banana': True} — carries over state from the previous call!

# ✅ use None as the default, and create the mutable object inside the function
def add_item(item, cart=None):
    if cart is None:
        cart = {}
    cart[item] = True
    return cart
```

One of Python's best-known gotchas — the underlying reason is that a function's default values are evaluated **once**, when the function is defined, not on every call.

---
Related: [Shallow copy vs Deep copy](../../system-design/quality-attributes.md#shallow-copy-vs-deep-copy), [General Syntax](syntax.md).
