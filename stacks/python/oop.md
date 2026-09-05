# Python — OOP

## `__init__` vs `__new__`

`__new__` crea la instancia (asigna la memoria); `__init__` la inicializa (setea los atributos) sobre una instancia que ya existe. En la enorme mayoría del código de aplicación normal, solo se sobreescribe `__init__` — `__new__` se toca en casos avanzados (ej. implementar un Singleton, o subclasear un tipo inmutable como `str`/`tuple`).

```python
class Punto:
    def __new__(cls, *args, **kwargs):
        print("1. Creando la instancia")
        return super().__new__(cls)

    def __init__(self, x, y):
        print("2. Inicializando la instancia")
        self.x, self.y = x, y

Punto(1, 2)
# 1. Creando la instancia
# 2. Inicializando la instancia
```

```python
class Contador:
    vistos = []           # atributo de clase — compartido por TODAS las instancias

    def __init__(self):
        self.total = 0     # atributo de instancia — uno nuevo por cada objeto

a, b = Contador(), Contador()
a.vistos.append(1)
b.vistos    # [1] — mismo objeto compartido
a.total = 5
b.total     # 0 — independiente entre instancias
```

## `staticmethod` vs `classmethod` vs método de instancia

```python
class Order:
    def total_con_descuento(self, pct):      # instancia: recibe self, opera sobre ESA instancia
        return self.total * (1 - pct / 100)

    @classmethod
    def from_dict(cls, data):                  # classmethod: recibe cls, típico constructor alternativo
        return cls(data["total"])

    @staticmethod
    def es_descuento_valido(pct):               # staticmethod: no recibe ni self ni cls
        return 0 <= pct <= 100
```

- **Instancia**: necesita `self` — opera sobre los datos de una instancia particular.
- **`classmethod`**: recibe la clase (`cls`) en vez de una instancia — típico para constructores alternativos (`from_dict`, `from_json`).
- **`staticmethod`**: no recibe ni `self` ni `cls` — una función de utilidad que conceptualmente pertenece a la clase, pero no necesita nada de ella.

## Dunder (magic) methods

Métodos con nombre `__algo__` que el intérprete llama automáticamente en ciertas situaciones — el mecanismo por el que tus propias clases se pueden comportar como los tipos built-in.

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):                        # cómo se ve en el debugger/repr()
        return f"Vector({self.x}, {self.y})"

    def __eq__(self, other):                   # qué significa == entre dos Vector
        return self.x == other.x and self.y == other.y

    def __add__(self, other):                  # qué hace el operador +
        return Vector(self.x + other.x, self.y + other.y)

    def __len__(self):                         # qué devuelve len(vector)
        return 2

Vector(1, 2) + Vector(3, 4)   # Vector(4, 6) — gracias a __add__
```

`__str__` vs `__repr__`: `__str__` es para el usuario final (`print(obj)`); `__repr__` es para debugging/desarrollo. Si solo definís uno, definí `__repr__` — Python lo usa como fallback de `__str__` si este último no existe.

## Duck typing

"Si camina como un pato y grazna como un pato, es un pato" — Python no chequea el **tipo** de un objeto antes de usarlo, chequea si tiene el **método/atributo** que se le va a pedir. No hace falta heredar de una interfaz formal para que algo "cuente" como compatible.

```python
class Pato:
    def hacer_sonido(self): return "Cuac"

class Persona:
    def hacer_sonido(self): return "Digo cuac"  # no hereda de Pato, no le importa

def hacer_ruido(cosa):
    print(cosa.hacer_sonido())  # no le importa el tipo, solo que tenga hacer_sonido()

hacer_ruido(Pato())      # funciona
hacer_ruido(Persona())    # también funciona
```

## Herencia múltiple y MRO (Method Resolution Order)

Python permite heredar de más de una clase a la vez — a diferencia de TypeScript/Java. Cuando dos clases padre tienen un método con el mismo nombre, el **MRO** define en qué orden Python busca cuál usar (algoritmo C3 linearization — en la práctica: izquierda a derecha, profundidad después).

```python
class A:
    def saludo(self): return "A"

class B:
    def saludo(self): return "B"

class C(A, B):    # hereda de A primero, B después
    pass

C().saludo()       # "A" — Python busca primero en A por el orden declarado en C(A, B)
C.__mro__           # muestra el orden exacto de búsqueda
```

---
Relacionado: [Patrones creacionales](../../system-design/patrones-creacionales.md) (Singleton usa `__new__`), [Patrones estructurales](../../system-design/patrones-estructurales.md) (Class Adapter necesita herencia múltiple), [Tipos y Mutabilidad](tipos-y-mutabilidad.md) (mutable default arguments, mismo mecanismo de fondo), [Python vs Swift](vs-swift.md#clases-y-oop) (atributo de clase = `static var`).
