# Python — Tipos y Mutabilidad

## Mutable vs inmutable

- **Mutables**: `list`, `dict`, `set` — se pueden modificar in-place, la misma referencia sigue siendo válida después de cambiar el contenido.
- **Inmutables**: `tuple`, `str`, `int`, `float`, `bool`, `frozenset` — cualquier "modificación" en realidad crea un objeto nuevo.

```python
a = [1, 2, 3]
a.append(4)       # ✅ modifica la misma lista, misma referencia (id(a) no cambia)

b = "hola"
b += " mundo"      # ❌ no modifica el string original — crea un string NUEVO y b apunta a ese nuevo objeto
```

Por qué importa: un objeto mutable pasado como argumento a una función puede ser modificado por esa función y el cambio "se ve" afuera; uno inmutable, no — ver [Shallow copy vs Deep copy](../../system-design/atributos-de-calidad.md#shallow-copy-vs-deep-copy) para el caso más sutil (mutar algo anidado dentro de una copia "nueva").

## `is` vs `==`

- **`==`**: compara **valor** — ¿tienen el mismo contenido?
- **`is`**: compara **identidad** — ¿son el mismo objeto en memoria (mismo `id()`)?

```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b   # True — mismo contenido
a is b   # False — son dos objetos distintos en memoria, aunque luzcan iguales

c = a
c is a   # True — c es literalmente el mismo objeto que a, no una copia
```

**El caso especial de `None`**: siempre se compara con `is None`, no `== None` — por convención, y porque `is` es explícito sobre lo que realmente importa (identidad), sin depender de un `__eq__` custom que alguien pudo haber sobreescrito.

## El gotcha clásico: mutable default arguments

```python
# ❌ el default {} se crea UNA SOLA VEZ, cuando se define la función —
# no una vez por cada llamada — así que todas las llamadas sin argumento comparten el MISMO dict
def add_item(item, cart={}):
    cart[item] = True
    return cart

add_item("manzana")   # {'manzana': True}
add_item("banana")    # {'manzana': True, 'banana': True} — ¡arrastra el estado de la llamada anterior!

# ✅ usar None como default, y crear el objeto mutable adentro de la función
def add_item(item, cart=None):
    if cart is None:
        cart = {}
    cart[item] = True
    return cart
```

Uno de los gotchas más conocidos de Python — la razón de fondo es que los valores default de una función se evalúan **una sola vez**, al definir la función, no en cada llamada.

---
Relacionado: [Shallow copy vs Deep copy](../../system-design/atributos-de-calidad.md#shallow-copy-vs-deep-copy), [Sintaxis general](sintaxis.md).
