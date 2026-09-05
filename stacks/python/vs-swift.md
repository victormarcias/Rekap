# Python vs Swift

Sintaxis y OOP de ambos lenguajes lado a lado — muchos conceptos son el mismo mecanismo con otro nombre (ver el caso de los *dunder methods* más abajo, que en Swift se resuelven con protocolos y operadores custom).

## Sintaxis general

```python
x = 5                          # variable — dinámico, sin declarar tipo
MAX = 100                      # constante — no existe una real, es solo convención (mayúsculas)
x: int = 5                     # tipado — opcional, con type hints

saludo = f"Hola {nombre}"      # string interpolation
texto = """texto
multi"""                       # string multilínea

# esto es un comentario

valor = None                   # ausencia de valor

from typing import Optional
edad: Optional[int] = None     # optional — o int | None (3.10+)

if x is not None:              # optional binding — no hay sintaxis dedicada, se chequea a mano
    usar(x)

resultado = x if x is not None else default   # nil-coalescing — no hay operador dedicado
```

```swift
var x = 5                      // variable
let MAX = 100                  // constante — real, enforced por el compilador
let x = 5                      // tipado — estático y obligatorio (con inferencia)

let saludo = "Hola \(nombre)"  // string interpolation
let texto = """
texto
multi
"""                             // string multilínea

// esto es un comentario

let valor: Int? = nil          // ausencia de valor

var edad: Int? = nil           // optional

if let x = x {                 // optional binding
    usar(x)
}

let resultado = x ?? default   // nil-coalescing

let forzado = x!                // forced unwrap — no existe en Python, siempre hay que chequear a mano
```

## Colecciones

```python
lista = [1, 2, 3]                          # array / lista
d = {"a": 1}                               # diccionario
s = {1, 2, 3}                              # set
t = (1, "a")                               # tupla

cuadrados = [x**2 for x in range(10)]      # comprehension
positivos = [x for x in lista if x > 0]    # filter
```

```swift
let lista = [1, 2, 3]                      // array / lista
let d = ["a": 1]                           // diccionario
let s: Set = [1, 2, 3]                     // set
let t = (1, "a")                           // tupla

let cuadrados = (0..<10).map { $0 * $0 }   // map — Swift no tiene comprehension nativa
let positivos = lista.filter { $0 > 0 }    // filter
```

## Funciones y Closures

```python
def sumar(a, b):                # función
    return a + b

def f(x=10):                    # parámetro con default
    ...

f(x=1, y=2)                     # argumentos con nombre — opcional en la llamada

def f(*args):                   # variádica
    ...

duplicar = lambda x: x * 2      # closure / lambda
```

```swift
func sumar(_ a: Int, _ b: Int) -> Int {   // función
    a + b
}

func f(x: Int = 10) {           // parámetro con default
    // ...
}

f(x: 1, y: 2)                   // argumentos con nombre — obligatorio salvo "_" en la firma

func f(_ args: Int...) {        // variádica
    // ...
}

let duplicar = { (x: Int) in x * 2 }   // closure / lambda

numeros.map { $0 * 2 }          // trailing closure — no existe el concepto en Python
```

## Control de flujo

```python
if x > 0:                       # if / else
    hacer_algo()
else:
    hacer_otra_cosa()

if not cond:                    # guard (salida temprana) — no existe, se hace manual
    return

for x in lista:                 # for-in
    usar(x)

while cond:                     # while
    ...

match valor:                    # switch / match (3.10+)
    case 1:
        print("uno")
    case _:
        print("otro")

range(0, 10)                    # range — exclusivo
```

```swift
if x > 0 {                      // if / else
    hacerAlgo()
} else {
    hacerOtraCosa()
}

guard cond else { return }      // guard (salida temprana)

for x in lista {                // for-in
    usar(x)
}

while cond {                    // while
    // ...
}

switch valor {                  // switch
case 1:
    print("uno")
default:
    print("otro")
}

0..<10                          // range exclusivo
0...10                          // range inclusivo
```

## Manejo de errores

```python
try:                             # try / catch
    algo()
except ValueError as e:
    manejar(e)

raise ValueError("mensaje")      # lanzar un error propio

def f():                         # declarar que puede fallar — no hace falta declarar nada
    ...
```

```swift
do {                              // try / catch
    try algo()
} catch {
    manejar(error)
}

enum MiError: Error { case mensaje }
throw MiError.mensaje             // lanzar un error propio

func f() throws {                 // declarar que puede fallar — obligatorio marcar la función
    // ...
}
```

## Clases y OOP

```python
class Animal:
    def __init__(self, nombre):       # constructor
        self.nombre = nombre

class Perro(Animal):                  # herencia
    def __init__(self, nombre):
        super().__init__(nombre)      # llamar al padre

    def ladrar(self):                 # método de instancia
        print("Guau")

    @property                         # propiedad computada
    def descripcion(self):
        return f"Perro: {self.nombre}"

    @staticmethod                     # método estático — no se puede overridear
    def crear_generico():
        return Perro("Genérico")

    @classmethod                      # método de clase — sí se puede overridear
    def desde_nombre(cls, nombre):
        return cls(nombre)

    def __del__(self):                # destructor
        print("adiós")

    def _protegido(self): ...         # protegido — convención, no enforced
    def __privado(self): ...          # privado — name mangling, no enforced

class Contador:
    vistos = []                       # atributo de clase — se crea UNA sola vez, compartido por todas las instancias

    def __init__(self):
        self.total = 0                # atributo de instancia — recién existe cuando corre __init__, uno por objeto
```

```swift
class Animal {
    let nombre: String

    init(nombre: String) {            // constructor
        self.nombre = nombre
    }
}

class Perro: Animal {                 // herencia
    override init(nombre: String) {
        super.init(nombre: nombre)    // llamar al padre
    }

    func ladrar() {                   // método de instancia
        print("Guau")
    }

    var descripcion: String {         // propiedad computada
        "Perro: \(nombre)"
    }

    static func crearGenerico() -> Perro {          // método estático — no se puede overridear
        Perro(nombre: "Genérico")
    }

    class func desdeNombre(_ nombre: String) -> Perro {   // método de clase — sí se puede overridear
        Perro(nombre: nombre)
    }

    deinit {                          // destructor
        print("adiós")
    }

    fileprivate func protegido() { }  // protegido
    private func privado() { }        // privado — enforced por el compilador
}

class Contador {
    static var vistos: [Int] = []     // atributo de clase — un solo array, compartido por todas las instancias

    var total = 0                     // atributo de instancia — inicializado fresco en cada init
}
```

## Protocolos, interfaces y "duck typing"

```python
from typing import Protocol

class MiProtocolo(Protocol):          # interfaz / protocolo
    def hacer(self) -> None: ...

class X:                              # implementarlo — duck typing, alcanza con tener el método
    def hacer(self) -> None:
        print("hecho")

# extender un tipo existente — monkey-patching, poco idiomático en Python

# value type — no existe la distinción, todo objeto es referencia
# reference type — class (todo en Python es así)
```

```swift
protocol MiProtocolo {                // interfaz / protocolo
    func hacer()
}

class X: MiProtocolo {                // implementarlo — conformancia explícita
    func hacer() {
        print("hecho")
    }
}

extension Int {                       // extender un tipo existente — idiomático en Swift
    func esPar() -> Bool { self % 2 == 0 }
}

struct Punto { var x: Int; var y: Int }   // value type — se copia al pasar

class Contador { var valor = 0 }      // reference type — se comparte (mismo comportamiento que Python)
```

## Dunder methods vs protocolos/operadores de Swift

Los `__metodo__` de Python no son "magia" — son el mismo mecanismo que en Swift se resuelve con **conformancia a protocolos** y **operadores custom**: una forma estandarizada de que un tipo propio se comporte como los tipos nativos del lenguaje (se imprima, se compare, se sume, se itere).

```python
class Punto:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):                # representación en texto — str(obj) / print(obj)
        return f"Punto({self.x}, {self.y})"

    def __eq__(self, other):          # igualdad (==)
        return self.x == other.x and self.y == other.y

    def __lt__(self, other):          # comparación (<)
        return (self.x, self.y) < (other.x, other.y)

    def __add__(self, other):         # suma (+)
        return Punto(self.x + other.x, self.y + other.y)

    def __len__(self):                # longitud (len(obj))
        return 2

    def __getitem__(self, i):         # acceso por índice (obj[i])
        return (self.x, self.y)[i]

    def __call__(self):               # llamar como función (obj())
        return f"{self.x},{self.y}"

    def __iter__(self):               # iterar (for x in obj)
        return iter((self.x, self.y))

    def __enter__(self):              # cleanup garantizado — with obj:
        return self

    def __exit__(self, *args):
        self.cerrar()
```

```swift
struct Punto {
    var x: Int
    var y: Int
}

extension Punto: CustomStringConvertible {   // representación en texto
    var description: String { "Punto(\(x), \(y))" }
}

extension Punto: Equatable {                 // igualdad (==)
    static func == (l: Punto, r: Punto) -> Bool {
        l.x == r.x && l.y == r.y
    }
}

extension Punto: Comparable {                // comparación (<)
    static func < (l: Punto, r: Punto) -> Bool {
        (l.x, l.y) < (r.x, r.y)
    }
}

extension Punto {
    static func + (l: Punto, r: Punto) -> Punto {   // suma (+) — operador custom
        Punto(x: l.x + r.x, y: l.y + r.y)
    }

    var count: Int { 2 }              // "longitud" — convención, o conformar Collection

    subscript(i: Int) -> Int {        // acceso por índice (obj[i])
        i == 0 ? x : y
    }

    // llamar como función (obj()) — no hay equivalente directo,
    // los closures ya son de primera clase, no hace falta este patrón
}

extension Punto: Sequence {           // iterar (for x in obj)
    func makeIterator() -> some IteratorProtocol {
        [x, y].makeIterator()
    }
}

func usarPunto() {
    let p = Punto(x: 1, y: 2)
    defer { p.cerrar() }              // cleanup garantizado — mismo propósito que __exit__, mecanismo distinto
    // usar p acá
}
```

`with` envuelve un bloque completo desde afuera; `defer` se declara adentro de la función y corre al salir de su scope, sin necesidad de envolver nada — mismo propósito (garantizar que algo se limpie), mecanismo distinto.

---
Relacionado: [Sistema de tipos comparado](../tipos-comparativa.md) (mismo estilo de tabla, enfocado en tipos), [Sintaxis general de Python](sintaxis.md), [OOP en Python](oop.md), [Mutable default arguments](tipos-y-mutabilidad.md#el-gotcha-clásico-mutable-default-arguments) (mismo criterio: se evalúa/crea una sola vez, no por instancia/llamada).
