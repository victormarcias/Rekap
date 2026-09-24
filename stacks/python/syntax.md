# Python — General Syntax

## Indentation instead of braces

Python uses indentation (4 spaces, by PEP 8 convention) to define blocks, not `{ }`. Mixing tabs and spaces in the same file breaks the code with an `IndentationError`.

```python
if age >= 18:
    print("Adult")
else:
    print("Minor")
```

## Variables and dynamic typing

The type isn't declared — it's inferred at runtime, and **can change** (unlike TypeScript, where `let x = 5; x = "text"` is a compile error; here it's valid, the error only shows up if something later tries to operate with the wrong type).

```python
x = 5
x = "now I'm a string"  # valid — no type error until something misuses it
```

**Type hints** are optional and not enforced at runtime by default — they help the linter/IDE warn before running the code, not the interpreter to block execution.

```python
def greet(name: str, age: int) -> str:
    return f"{name} is {age} years old"

greet("Ana", "thirty")  # the interpreter runs it anyway; only the linter complains about the type
```

## Basic types

```python
integer = 42
floating = 3.14
text = "hello"
boolean = True          # capitalized, not "true"/"false"
nothing = None               # equivalent to null/nil
```

## Data structures

```python
nums = [1, 2, 3]                # list
point = (10, 20)                 # tuple
person = {"name": "Ana", "age": 30}   # dict
unique = {1, 2, 3}                # set
```

Which one is mutable/ordered is in [Types and Mutability](types-and-mutability.md#mutable-vs-immutable).

## Slicing

Access to ranges of a sequence (`list`, `tuple`, `str`) with `[start:end:step]`, no manual loop needed.

```python
nums = [0, 1, 2, 3, 4, 5]
nums[1:4]     # [1, 2, 3] — from index 1 up to 4 (exclusive)
nums[:3]      # [0, 1, 2] — from the start
nums[-2:]     # [4, 5] — the last 2
nums[::2]     # [0, 2, 4] — every 2 elements
nums[::-1]    # [5, 4, 3, 2, 1, 0] — reversed, no function call needed
```

## Control flow

```python
for item in [1, 2, 3]:
    print(item)

for i, item in enumerate(["a", "b", "c"]):  # with index, no manual counter
    print(i, item)

i = 0
while i < 3:
    i += 1   # there's no i++ or ++i in Python
```

## Comprehensions

A compact way to build a list/dict/set from an iterable — the Python equivalent of chaining `.map()`/`.filter()` in JS, but in a single expression evaluated at once (not two separate passes).

```python
# ❌ verbose: two lines of intent mixed with control flow
squares = []
for x in range(10):
    if x % 2 == 0:
        squares.append(x ** 2)

# ✅ list comprehension: same result, one line
squares = [x ** 2 for x in range(10) if x % 2 == 0]

# the same pattern exists for dict and set
squares_dict = {x: x ** 2 for x in range(5)}
evens_set = {x for x in range(10) if x % 2 == 0}

# filtering by type — typical with tuples/lists of mixed content
data = (1, "hello", 2.5, 3, None, True, 4)
only_ints = [x for x in data if type(x) is int]   # [1, 3, 4] — type() (not isinstance) excludes bool,
                                                     # because bool is a subclass of int but type(True) is bool
total = sum(x for x in data if type(x) is int)     # generator directly in sum(), no intermediate list
```

## Functions

```python
def add(a, b):
    return a + b

def greet(name, greeting="Hello"):    # default arg
    return f"{greeting}, {name}"

def sum_all(*args):                   # *args: any number of positionals, arrives as a tuple
    return sum(args)

def config(**kwargs):                 # **kwargs: any number of named args, arrives as a dict
    return kwargs

square = lambda x: x ** 2             # one-line anonymous function (equivalent to an arrow function)
```

## Classes

```python
class User:
    def __init__(self, name: str, age: int):    # constructor
        self.name = name
        self.age = age

    def greet(self) -> str:                      # self = explicit "this", always the first parameter
        return f"Hi, I'm {self.name}"

class Admin(User):                 # inheritance
    def __init__(self, name, age, permissions):
        super().__init__(name, age)              # calls the parent's constructor
        self.permissions = permissions

user = User("Ana", 30)
print(user.greet())
```

`self` is the most notable difference from JS/Swift: in Python "this" is an explicit parameter you have to declare by hand on every instance method, not something the language provides implicitly.

## Exception handling

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
except (TypeError, ValueError):     # catch several exception types at once
    print("Invalid type or value")
else:
    print("No exception occurred")       # only runs if the try didn't throw anything
finally:
    print("This always runs")       # cleanup, whether there was an error or not

raise ValueError("Error message")  # manually throw an exception
```

**When `finally` runs**: always — with a caught exception, without one, with `return`/`break`/`continue` inside the `try`, or even with an exception that **doesn't** match any `except` (the `finally` runs and only then does the exception continue propagating outward).

```python
def f():
    try:
        return 1
    finally:
        print("this runs anyway, before the function returns")
# prints the message and THEN returns 1
```

**The gotcha**: a `return` (or `raise`) inside `finally` **overrides** any `return`/exception coming from the `try` — the original exception gets lost silently, with no trace it ever existed.

```python
def f():
    try:
        raise ValueError("real error")
    finally:
        return "all good"   # ❌ this wins — ValueError never propagates, it's lost

f()   # "all good" — the ValueError never reached anyone
```

## Context managers — `with`

Guarantees a resource gets released when leaving the block, even if an exception occurs in the middle — avoids the classic bug of a `close()` that never runs because something failed before it. See [Backend Diagnostics](../../diagnostics/backend.md) (the "Not releasing system resources" section).

```python
with open("data.csv") as f:
    content = f.read()
# the file closes automatically when leaving the block, even if read() throws an exception
```

## Modules and imports

```python
import os                        # imports the whole module, accessed as os.getcwd()
from datetime import datetime    # imports a specific name from the module
from math import pi as PI        # imports with an alias
import numpy as np               # conventional alias (community standard, not the language's)
```

## f-strings — string interpolation

```python
name = "Ana"
age = 30
print(f"{name} is {age} years old")   # direct interpolation, no + concatenation
print(f"{age + 1=}")                 # debug shorthand (3.8+): prints "age + 1=31"
```

## Naming conventions (PEP 8)

| Element | Convention | Example |
|---|---|---|
| Variables and functions | `snake_case` | `user_name`, `calculate_total()` |
| Classes | `PascalCase` | `UserRepository` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_RETRIES` |
| "Private" (convention, not enforced by the language) | `_` prefix | `self._internal_state` |

Unlike JS/TS (`camelCase` for almost everything) or Swift, Python separates `snake_case` for variables/functions from `PascalCase` for classes — the linter (`ruff`) flags deviations, but the interpreter doesn't block them.

---
Related: [Types and Mutability](types-and-mutability.md), [Basics](basics.md), [OOP](oop.md).
