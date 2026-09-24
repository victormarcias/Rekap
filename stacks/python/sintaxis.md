# Python — Sintaxis General

## Indentación en vez de llaves

Python usa indentación (4 espacios, por convención de PEP 8) para definir bloques, no `{ }`. Mezclar tabs y espacios en el mismo archivo rompe el código con un `IndentationError`.

```python
if age >= 18:
    print("Mayor de edad")
else:
    print("Menor de edad")
```

## Variables y tipado dinámico

No se declara el tipo — se infiere en runtime, y **puede cambiar** (a diferencia de TypeScript, donde `let x = 5; x = "texto"` es error de compilación; acá es válido, el error solo aparece si después algo intenta operar con el tipo equivocado).

```python
x = 5
x = "ahora soy un string"  # válido — no hay error de tipo hasta que algo lo use mal
```

**Type hints** son opcionales y no se enforcean en runtime por defecto — le sirven al linter/IDE para avisar antes de correr el código, no al intérprete para bloquear la ejecución.

```python
def greet(name: str, age: int) -> str:
    return f"{name} tiene {age} años"

greet("Ana", "treinta")  # el intérprete lo corre igual; solo el linter se queja del tipo
```

## Tipos básicos

```python
entero = 42
flotante = 3.14
texto = "hola"
booleano = True          # con mayúscula, no "true"/"false"
nada = None               # equivalente a null/nil
```

## Estructuras de datos

```python
nums = [1, 2, 3]                # list
punto = (10, 20)                 # tuple
persona = {"name": "Ana", "age": 30}   # dict
unicos = {1, 2, 3}                # set
```

Cuál es mutable/ordenada de cada una está en [Tipos y Mutabilidad](tipos-y-mutabilidad.md#mutable-vs-inmutable).

## Slicing

Acceso a rangos de una secuencia (`list`, `tuple`, `str`) con `[inicio:fin:paso]`, sin necesidad de un loop manual.

```python
nums = [0, 1, 2, 3, 4, 5]
nums[1:4]     # [1, 2, 3] — desde índice 1 hasta 4 (exclusivo)
nums[:3]      # [0, 1, 2] — desde el principio
nums[-2:]     # [4, 5] — los últimos 2
nums[::2]     # [0, 2, 4] — cada 2 elementos
nums[::-1]    # [5, 4, 3, 2, 1, 0] — invertido, sin llamar a ninguna función
```

## Control de flujo

```python
for item in [1, 2, 3]:
    print(item)

for i, item in enumerate(["a", "b", "c"]):  # con índice, sin llevar un contador manual
    print(i, item)

i = 0
while i < 3:
    i += 1   # no existe i++ ni ++i en Python
```

## Comprehensions

Forma compacta de construir una lista/dict/set a partir de un iterable — el equivalente Python de encadenar `.map()`/`.filter()` en JS, pero en una sola expresión evaluada de una vez (no dos pasadas separadas).

```python
# ❌ verboso: dos líneas de intención mezcladas con el control de flujo
squares = []
for x in range(10):
    if x % 2 == 0:
        squares.append(x ** 2)

# ✅ list comprehension: mismo resultado, una línea
squares = [x ** 2 for x in range(10) if x % 2 == 0]

# el mismo patrón existe para dict y set
squares_dict = {x: x ** 2 for x in range(5)}
evens_set = {x for x in range(10) if x % 2 == 0}

# filtrar por tipo — típico con tuplas/listas de contenido mixto
datos = (1, "hola", 2.5, 3, None, True, 4)
solo_ints = [x for x in datos if type(x) is int]   # [1, 3, 4] — type() (no isinstance) excluye bool,
                                                     # porque bool es subclase de int pero type(True) is bool
total = sum(x for x in datos if type(x) is int)     # generator directo en sum(), sin lista intermedia
```

## Funciones

```python
def add(a, b):
    return a + b

def greet(name, greeting="Hola"):    # default arg
    return f"{greeting}, {name}"

def sum_all(*args):                   # *args: cualquier cantidad de posicionales, llega como tuple
    return sum(args)

def config(**kwargs):                 # **kwargs: cualquier cantidad de nombrados, llega como dict
    return kwargs

square = lambda x: x ** 2             # función anónima de una línea (equivalente a arrow function)
```

## Clases

```python
class User:
    def __init__(self, name: str, age: int):    # constructor
        self.name = name
        self.age = age

    def greet(self) -> str:                      # self = "this" explícito, siempre primer parámetro
        return f"Hola, soy {self.name}"

class Admin(User):                 # herencia
    def __init__(self, name, age, permissions):
        super().__init__(name, age)              # llama al constructor del padre
        self.permissions = permissions

user = User("Ana", 30)
print(user.greet())
```

`self` es la diferencia más notable frente a JS/Swift: en Python el "this" es un parámetro explícito que hay que declarar a mano en cada método de instancia, no algo implícito del lenguaje.

## Manejo de excepciones

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
except (TypeError, ValueError):     # capturar varios tipos de excepción a la vez
    print("Tipo o valor inválido")
else:
    print("No hubo excepción")       # corre solo si el try no lanzó nada
finally:
    print("Esto corre siempre")       # cleanup, haya habido error o no

raise ValueError("Mensaje de error")  # lanzar una excepción manualmente
```

**Cuándo corre `finally`**: siempre — con excepción capturada, sin excepción, con `return`/`break`/`continue` adentro del `try`, o incluso con una excepción que **no** matchea ningún `except` (corre el `finally` y recién después la excepción se sigue propagando hacia afuera).

```python
def f():
    try:
        return 1
    finally:
        print("esto corre igual, antes de que la función retorne")
# imprime el mensaje y DESPUÉS devuelve 1
```

**El gotcha**: un `return` (o `raise`) dentro del `finally` **pisa** cualquier `return`/excepción que venía del `try` — la excepción original se pierde en silencio, sin ningún rastro de que existió.

```python
def f():
    try:
        raise ValueError("error real")
    finally:
        return "todo bien"   # ❌ esto gana — ValueError nunca se propaga, se pierde

f()   # "todo bien" — el ValueError jamás llegó a nadie
```

## Context managers — `with`

Garantiza que un recurso se libere al salir del bloque, incluso si ocurre una excepción en el medio — evita el error clásico de un `close()` que nunca se ejecuta porque algo falló antes. Ver [Diagnóstico Backend](../../diagnostics/backend.es.md) (sección "No liberar recursos del sistema").

```python
with open("data.csv") as f:
    contenido = f.read()
# el archivo se cierra automáticamente al salir del bloque, incluso si read() lanza una excepción
```

## Módulos e imports

```python
import os                        # importa el módulo entero, se accede como os.getcwd()
from datetime import datetime    # importa un nombre específico del módulo
from math import pi as PI        # importa con alias
import numpy as np               # alias convencional (estándar de la comunidad, no del lenguaje)
```

## f-strings — interpolación de strings

```python
name = "Ana"
age = 30
print(f"{name} tiene {age} años")   # interpolación directa, sin concatenar con +
print(f"{age + 1=}")                 # debug shorthand (3.8+): imprime "age + 1=31"
```

## Convenciones de nombres (PEP 8)

| Elemento | Convención | Ejemplo |
|---|---|---|
| Variables y funciones | `snake_case` | `user_name`, `calculate_total()` |
| Clases | `PascalCase` | `UserRepository` |
| Constantes | `UPPER_SNAKE_CASE` | `MAX_RETRIES` |
| "Privado" (convención, no lo enforcea el lenguaje) | prefijo `_` | `self._internal_state` |

A diferencia de JS/TS (`camelCase` para casi todo) o Swift, Python separa `snake_case` para variables/funciones de `PascalCase` para clases — el linter (`ruff`) marca las desviaciones, pero el intérprete no las bloquea.

---
Relacionado: [Tipos y Mutabilidad](tipos-y-mutabilidad.md), [Básico](basico.md), [OOP](oop.md).
