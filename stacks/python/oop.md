# Python — OOP

## `__init__` vs `__new__`

`__new__` creates the instance (allocates memory); `__init__` initializes it (sets the attributes) on an instance that already exists. In the vast majority of normal application code, only `__init__` gets overridden — `__new__` gets touched in advanced cases (e.g. implementing a Singleton, or subclassing an immutable type like `str`/`tuple`).

```python
class Point:
    def __new__(cls, *args, **kwargs):
        print("1. Creating the instance")
        return super().__new__(cls)

    def __init__(self, x, y):
        print("2. Initializing the instance")
        self.x, self.y = x, y

Point(1, 2)
# 1. Creating the instance
# 2. Initializing the instance
```

```python
class Counter:
    totalGlobal = 0               # class attribute — shared, exists once

    def __init__(self):
        self.total = 0             # instance attribute — a new one per object
        Counter.totalGlobal += 1  # every new instance adds to the shared counter

c1 = Counter()
c2 = Counter()
c1.totalGlobal, c2.totalGlobal    # (2, 2) — same class attribute, both see the same value

c1.total += 5
c1.total, c2.total                # (5, 0) — no overlap: each instance has its own

Counter.totalGlobal += 10
c1.totalGlobal, c2.totalGlobal    # (12, 12) — this does overlap: it's the same attribute for both
```

## `staticmethod` vs `classmethod` vs instance method

```python
class Order:
    def total_with_discount(self, pct):      # instance: receives self, operates on THAT instance
        return self.total * (1 - pct / 100)

    @classmethod
    def from_dict(cls, data):                  # classmethod: receives cls, typical alternative constructor
        return cls(data["total"])

    @staticmethod
    def is_valid_discount(pct):               # staticmethod: receives neither self nor cls
        return 0 <= pct <= 100
```

- **Instance**: needs `self` — operates on a particular instance's data.
- **`classmethod`**: receives the class (`cls`) instead of an instance — typical for alternative constructors (`from_dict`, `from_json`).
- **`staticmethod`**: receives neither `self` nor `cls` — a utility function that conceptually belongs to the class, but doesn't need anything from it.

## Dunder (magic) methods

Methods named `__something__` that the interpreter calls automatically in certain situations — the mechanism by which your own classes can behave like the built-in types.

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):                        # how it looks in the debugger/repr()
        return f"Vector({self.x}, {self.y})"

    def __eq__(self, other):                   # what == means between two Vectors
        return self.x == other.x and self.y == other.y

    def __add__(self, other):                  # what the + operator does
        return Vector(self.x + other.x, self.y + other.y)

    def __len__(self):                         # what len(vector) returns
        return 2

Vector(1, 2) + Vector(3, 4)   # Vector(4, 6) — thanks to __add__
```

`__str__` vs `__repr__`: `__str__` is for the end user (`print(obj)`); `__repr__` is for debugging/development. If you only define one, define `__repr__` — Python uses it as `__str__`'s fallback if the latter doesn't exist.

## Duck typing

"If it walks like a duck and quacks like a duck, it's a duck" — Python doesn't check an object's **type** before using it, it checks whether it has the **method/attribute** that's about to be requested. You don't need to inherit from a formal interface for something to "count" as compatible.

```python
class Duck:
    def make_sound(self): return "Quack"

class Person:
    def make_sound(self): return "I say quack"  # doesn't inherit from Duck, doesn't care

def make_noise(thing):
    print(thing.make_sound())  # doesn't care about the type, only that it has make_sound()

make_noise(Duck())      # works
make_noise(Person())    # also works
```

## Multiple inheritance and MRO (Method Resolution Order)

Python allows inheriting from more than one class at once — unlike TypeScript/Java. When two parent classes have a method with the same name, the **MRO** defines the order Python searches for which one to use (C3 linearization algorithm — in practice: left to right, depth after).

```python
class A:
    def greeting(self): return "A"

class B:
    def greeting(self): return "B"

class C(A, B):    # inherits from A first, B second
    pass

C().greeting()       # "A" — Python searches A first, per the order declared in C(A, B)
C.__mro__           # shows the exact search order
```

---
Related: [Creational patterns](../../system-design/creational-patterns.md) (Singleton uses `__new__`), [Structural patterns](../../system-design/structural-patterns.md) (Class Adapter needs multiple inheritance), [Types and Mutability](types-and-mutability.md) (mutable default arguments, the same underlying mechanism).
