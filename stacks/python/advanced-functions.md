# Python — Advanced Functions

## Decorators — the language's syntax

We already used decorators as an example of the [Decorator pattern](../../system-design/structural-patterns.md#decorator) — this is the mechanics of the `@` syntax itself.

`@decorator` above a function is syntactic sugar — `func = decorator(func)`, applied automatically.

```python
def with_logging(fn):
    def wrapper(*args, **kwargs):
        print(f"Calling {fn.__name__}")
        return fn(*args, **kwargs)
    return wrapper

@with_logging
def greet(name):
    return f"Hello, {name}"

# equivalent, without the @ syntax:
# greet = with_logging(greet)
```

**`functools.wraps`**: without this, the decorated function "loses" its identity — `greet.__name__` would become `"wrapper"` instead of `"greet"`, breaking introspection/debugging.

```python
from functools import wraps

def with_logging(fn):
    @wraps(fn)   # ✅ preserves __name__, __doc__, etc. from the original function
    def wrapper(*args, **kwargs):
        return fn(*args, **kwargs)
    return wrapper
```

## Closures

An inner function that "remembers" a variable from the scope of the function wrapping it, even after that outer function has already finished executing. It's the mechanism behind the decorators above: `wrapper` still has access to `fn` long after `with_logging` returned.

```python
def counter():
    n = 0
    def increment():
        nonlocal n   # without this, it would be a new n local to increment(), not counter()'s
        n += 1
        return n
    return increment

add = counter()
add()   # 1
add()   # 2 — "remembers" the n from the previous call, doesn't start over
```

**LEGB**: the order Python searches for a name in — **L**ocal (the current function) → **E**nclosing (the function wrapping it, the closures case) → **G**lobal (the module) → **B**uilt-in (`len`, `print`, etc.). `nonlocal` enables modifying a variable from the *enclosing* scope; `global`, one from the *global* scope — without either, an assignment always creates a new local variable instead of touching the outer one.

**The classic gotcha: late binding in loops**

```python
# ❌ the three lambdas share the SAME variable i — when called,
# the loop has already finished and i is 2 for all three
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]   # [2, 2, 2] — not [0, 1, 2] as you'd expect

# ✅ a default argument forces i to be evaluated at the moment each lambda is created,
# not at the moment it's called
funcs = [lambda i=i: i for i in range(3)]
[f() for f in funcs]   # [0, 1, 2]
```

A closure doesn't capture the *value* of the variable at the moment it's defined — it captures a reference to the *name*, and resolves it only when called. If that variable kept changing (a loop's `i`), every closure created in that loop sees the final value, not the one it had during its own iteration.

## Generators — lazy evaluation

A function with `yield` instead of `return` is a **generator**: it doesn't compute all the values at once — it yields one at a time, pausing its execution between each `yield`, and only keeps computing when asked for the next one.

```python
def squares_up_to(n):
    for i in range(n):
        yield i ** 2   # pauses here, returns a value, waits to be asked for the next

gen = squares_up_to(1_000_000)   # hasn't computed ANY square yet
next(gen)                            # 0 — only now does it compute the first one
next(gen)                            # 1
```

**Why it matters**: a list comprehension (`[x**2 for x in range(1_000_000)]`) computes and stores **everything** in memory at once — an equivalent generator (`(x**2 for x in range(1_000_000))`, with parentheses instead of brackets) doesn't use that memory, because it never has more than one value "alive" at a time. Same spirit as [processing DB results in chunks](../../database/scaling-database.md#processing-large-results-in-chunks): don't fetch/compute everything at once if you can go piece by piece.

## Iterators vs Iterables

- **Iterable**: any object that can be traversed with a `for` — implements `__iter__`, which returns an iterator.
- **Iterator**: the object that actually hands out the values one by one — implements `__next__` (and raises `StopIteration` when done).

```python
class Counter:
    def __init__(self, up_to):
        self.up_to = up_to

    def __iter__(self):          # makes it Iterable
        self.current = 0
        return self

    def __next__(self):          # makes it Iterator
        if self.current >= self.up_to:
            raise StopIteration
        self.current += 1
        return self.current

for n in Counter(3):   # 1, 2, 3
    print(n)
```

A generator (above) is, underneath, an Iterator automatically built by Python — no need to write `__iter__`/`__next__` by hand to achieve the same result.

---
Related: [Structural patterns](../../system-design/structural-patterns.md#decorator), [Database Scalability](../../database/scaling-database.md#processing-large-results-in-chunks).
