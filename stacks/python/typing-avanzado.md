# Python — Typing Avanzado

Type hints más allá de lo básico (`str`, `int`, `-> bool`) — ver [Sintaxis general](sintaxis.md) para la base.

## `Optional` / `Union`

```python
from typing import Optional, Union

def buscar(id: int) -> Optional[str]:      # equivalente a str | None (sintaxis 3.10+)
    ...

def procesar(valor: Union[int, str]):        # equivalente a int | str (sintaxis 3.10+)
    ...
```

`Optional[X]` es azúcar para `Union[X, None]` — no significa "parámetro opcional" (eso lo da un default `= None`), significa "puede ser `X` o puede ser `None`".

## `TypedDict`

Le da forma a un `dict` — qué keys existen y de qué tipo es cada una — sin envolverlo en una clase real como haría un `dataclass`/`BaseModel`. Sigue siendo un `dict` en runtime, el chequeo es solo estático (linter/IDE).

```python
from typing import TypedDict

class UserDict(TypedDict):
    name: str
    age: int

def crear_user() -> UserDict:
    return {"name": "Ana", "age": 30}   # el linter marca error si falta una key o sobra otra
```

## `Protocol` — structural typing

Define una interfaz por **forma** (qué métodos/atributos tiene un objeto), no por herencia — el chequeo es estructural, como el duck typing de siempre, pero ahora el linter lo puede verificar sin correr el código.

```python
from typing import Protocol

class TienePerimetro(Protocol):
    def perimetro(self) -> float: ...

class Cuadrado:
    def __init__(self, lado: float):
        self.lado = lado
    def perimetro(self) -> float:
        return self.lado * 4

def imprimir_perimetro(figura: TienePerimetro):
    print(figura.perimetro())

imprimir_perimetro(Cuadrado(3))   # ✅ el linter lo acepta — Cuadrado no hereda de TienePerimetro,
                                    # pero tiene el método perimetro() con la firma correcta
```

**`Protocol` vs heredar de una clase abstracta (`ABC`)**: con `ABC`, una clase tiene que declarar explícitamente que implementa la interfaz (`class Cuadrado(TienePerimetro)`) — es *nominal typing*. Con `Protocol`, cualquier clase que "tenga la forma" correcta califica automáticamente, sin heredar de nada ni saber que el `Protocol` existe — es duck typing formalizado y chequeado estáticamente, no en runtime.

**`Protocol` no es lo mismo que `Depends` de FastAPI** — son de dominios distintos y no se comparan de igual a igual: `Protocol` es una herramienta del **sistema de tipos** (define una interfaz, la verifica el linter, no existe en runtime); `Depends` es un mecanismo de **Dependency Injection en runtime** (FastAPI resuelve y ejecuta algo antes de correr el handler). Uno describe forma; el otro inyecta comportamiento — ver `Depends` en [Endpoints para microservicios](../../backend/fastapi/endpoints-microservicios.md#4-dependency-injection-con-depends).

---
Relacionado: [Sintaxis general](sintaxis.md), [OOP](oop.md) (duck typing), [Backend — Endpoints para microservicios](../../backend/fastapi/endpoints-microservicios.md) (`Depends`).
