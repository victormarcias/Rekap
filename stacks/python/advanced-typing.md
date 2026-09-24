# Python — Advanced Typing

Type hints beyond the basics (`str`, `int`, `-> bool`) — see [General Syntax](syntax.md) for the foundation.

## `Optional` / `Union`

```python
from typing import Optional, Union

def find(id: int) -> Optional[str]:      # equivalent to str | None (3.10+ syntax)
    ...

def process(value: Union[int, str]):        # equivalent to int | str (3.10+ syntax)
    ...
```

`Optional[X]` is sugar for `Union[X, None]` — it doesn't mean "optional parameter" (that's given by a `= None` default), it means "can be `X` or can be `None`."

## `TypedDict`

Gives shape to a `dict` — what keys exist and what type each one is — without wrapping it in a real class the way a `dataclass`/`BaseModel` would. It's still a `dict` at runtime, the check is only static (linter/IDE).

```python
from typing import TypedDict

class UserDict(TypedDict):
    name: str
    age: int

def create_user() -> UserDict:
    return {"name": "Ana", "age": 30}   # the linter flags an error if a key is missing or extra
```

## `Protocol` — structural typing

Defines an interface by **shape** (what methods/attributes an object has), not by inheritance — the check is structural, like the usual duck typing, but now the linter can verify it without running the code.

```python
from typing import Protocol

class HasPerimeter(Protocol):
    def perimeter(self) -> float: ...

class Square:
    def __init__(self, side: float):
        self.side = side
    def perimeter(self) -> float:
        return self.side * 4

def print_perimeter(shape: HasPerimeter):
    print(shape.perimeter())

print_perimeter(Square(3))   # ✅ the linter accepts it — Square doesn't inherit from HasPerimeter,
                                    # but it has the perimeter() method with the right signature
```

**`Protocol` vs inheriting from an abstract class (`ABC`)**: with `ABC`, a class has to explicitly declare that it implements the interface (`class Square(HasPerimeter)`) — it's *nominal typing*. With `Protocol`, any class that "has the right shape" qualifies automatically, without inheriting from anything or even knowing the `Protocol` exists — it's duck typing formalized and checked statically, not at runtime.

**`Protocol` is not the same as FastAPI's `Depends`** — they're from different domains and aren't a fair comparison: `Protocol` is a **type system** tool (defines an interface, checked by the linter, doesn't exist at runtime); `Depends` is a **runtime Dependency Injection** mechanism (FastAPI resolves and runs something before running the handler). One describes shape; the other injects behavior — see `Depends` in [Microservice Endpoints](../fastapi/microservice-endpoints.md#4-dependency-injection-with-depends).

---
Related: [General Syntax](syntax.md), [OOP](oop.md) (duck typing), [Backend — Microservice Endpoints](../fastapi/microservice-endpoints.md) (`Depends`).
