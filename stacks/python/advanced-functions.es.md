# Python — Funciones Avanzadas

## Decorators — la sintaxis del lenguaje

Ya usamos decorators como ejemplo del [patrón Decorator](../../system-design/structural-patterns.es.md#decorator) — esto es la mecánica de la sintaxis `@` en sí.

`@decorador` arriba de una función es azúcar sintáctico — `func = decorador(func)`, aplicado automáticamente.

```python
def with_logging(fn):
    def wrapper(*args, **kwargs):
        print(f"Llamando a {fn.__name__}")
        return fn(*args, **kwargs)
    return wrapper

@with_logging
def saludar(nombre):
    return f"Hola, {nombre}"

# equivalente, sin la sintaxis @:
# saludar = with_logging(saludar)
```

**`functools.wraps`**: sin esto, la función decorada "pierde" su identidad — `saludar.__name__` pasaría a ser `"wrapper"` en vez de `"saludar"`, rompiendo introspección/debugging.

```python
from functools import wraps

def with_logging(fn):
    @wraps(fn)   # ✅ preserva __name__, __doc__, etc. de la función original
    def wrapper(*args, **kwargs):
        return fn(*args, **kwargs)
    return wrapper
```

## Closures

Una función interna que "recuerda" una variable del scope de la función que la envuelve, incluso después de que esa función externa ya terminó de ejecutarse. Es el mecanismo detrás de los decorators de arriba: `wrapper` sigue teniendo acceso a `fn` mucho después de que `with_logging` retornó.

```python
def contador():
    n = 0
    def incrementar():
        nonlocal n   # sin esto, sería un nuevo n local a incrementar(), no el de contador()
        n += 1
        return n
    return incrementar

sumar = contador()
sumar()   # 1
sumar()   # 2 — "recuerda" el n de la llamada anterior, no arranca de nuevo
```

**LEGB**: el orden en que Python busca un nombre — **L**ocal (la función actual) → **E**nclosing (la función que la envuelve, el caso de closures) → **G**lobal (el módulo) → **B**uilt-in (`len`, `print`, etc.). `nonlocal` habilita modificar una variable del scope *enclosing*; `global`, una del scope *global* — sin ninguno de los dos, una asignación siempre crea una variable local nueva en vez de tocar la de afuera.

**El gotcha clásico: late binding en loops**

```python
# ❌ los tres lambdas comparten la MISMA variable i — cuando se llaman,
# el loop ya terminó y i vale 2 para los tres
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]   # [2, 2, 2] — no [0, 1, 2] como se esperaría

# ✅ default argument fuerza a evaluar i en el momento de crear cada lambda,
# no en el momento de llamarla
funcs = [lambda i=i: i for i in range(3)]
[f() for f in funcs]   # [0, 1, 2]
```

Un closure no captura el *valor* de la variable en el momento en que se define — captura una referencia al *nombre*, y lo resuelve recién cuando se llama. Si esa variable siguió cambiando (el `i` de un loop), todos los closures creados en ese loop ven el valor final, no el que tenía en su propia iteración.

## Generators — evaluación perezosa

Una función con `yield` en vez de `return` es un **generator**: no calcula todos los valores de una — cede uno por vez, pausando su ejecución entre cada `yield`, y solo sigue calculando cuando se le pide el próximo.

```python
def cuadrados_hasta(n):
    for i in range(n):
        yield i ** 2   # pausa acá, devuelve un valor, espera a que le pidan el siguiente

gen = cuadrados_hasta(1_000_000)   # no calculó NINGÚN cuadrado todavía
next(gen)                            # 0 — recién ahora calcula el primero
next(gen)                            # 1
```

**Por qué importa**: una list comprehension (`[x**2 for x in range(1_000_000)]`) calcula y guarda **todo** en memoria de una — un generator equivalente (`(x**2 for x in range(1_000_000))`, con paréntesis en vez de corchetes) no ocupa esa memoria, porque nunca tiene más de un valor "vivo" a la vez. Mismo espíritu que [procesar resultados de DB en chunks](../../database/scaling-database.es.md#procesar-resultados-grandes-en-chunks): no traer/calcular todo de una si se puede ir de a partes.

## Iterators vs Iterables

- **Iterable**: cualquier objeto que se puede recorrer con un `for` — implementa `__iter__`, que devuelve un iterator.
- **Iterator**: el objeto que efectivamente entrega los valores uno por uno — implementa `__next__` (y lanza `StopIteration` cuando se termina).

```python
class Contador:
    def __init__(self, hasta):
        self.hasta = hasta

    def __iter__(self):          # lo hace Iterable
        self.actual = 0
        return self

    def __next__(self):          # lo hace Iterator
        if self.actual >= self.hasta:
            raise StopIteration
        self.actual += 1
        return self.actual

for n in Contador(3):   # 1, 2, 3
    print(n)
```

Un generator (arriba) es, por detrás, un Iterator armado automáticamente por Python — no hace falta escribir `__iter__`/`__next__` a mano para lograr el mismo resultado.

---
Relacionado: [Patrones estructurales](../../system-design/structural-patterns.es.md#decorator), [Escalabilidad de Base de Datos](../../database/scaling-database.es.md#procesar-resultados-grandes-en-chunks).
