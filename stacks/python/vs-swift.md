# Python vs Swift: Sintaxis y OOP lado a lado

Sintaxis y OOP de ambos lenguajes lado a lado — muchos conceptos son el mismo mecanismo con otro nombre (ver el caso de los *dunder methods* más abajo, que en Swift se resuelven con protocolos y operadores custom).

## Sintaxis general

| Nombre | Ejemplo en Python | Ejemplo en Swift |
|---|---|---|
| Variable | `x = 5` (dinámico, sin declarar tipo) | `var x = 5` |
| Constante | No existe `const` real — convención `MAX = 100` (mayúsculas) | `let MAX = 100` (constante real, enforced por el compilador) |
| Tipado | Opcional, con type hints: `x: int = 5` | Estático y obligatorio (con inferencia): `let x = 5` |
| String interpolation | `f"Hola {nombre}"` | `"Hola \(nombre)"` |
| String multilínea | `"""texto\nmulti"""` | `"""texto\nmulti"""` (mismo símbolo, triple comillas) |
| Comentario | `# comentario` | `// comentario` |
| Ausencia de valor | `None` | `nil` |
| Optional | `Optional[int]` / `int \| None` (3.10+) | `Int?` |
| Forced unwrap | No existe — siempre hay que chequear a mano | `valor!` (fuerza el unwrap, crashea si es `nil`) |
| Optional binding | `if x is not None: usar(x)` | `if let x = x { usar(x) }` |
| Nil-coalescing | `x if x is not None else default` | `x ?? default` |

## Colecciones

| Nombre | Ejemplo en Python | Ejemplo en Swift |
|---|---|---|
| Array / Lista | `lista = [1, 2, 3]` | `let lista = [1, 2, 3]` |
| Diccionario | `d = {"a": 1}` | `let d = ["a": 1]` |
| Set | `s = {1, 2, 3}` | `let s: Set = [1, 2, 3]` |
| Tupla | `t = (1, "a")` | `let t = (1, "a")` |
| Comprehension / map | `[x**2 for x in range(10)]` | `(0..<10).map { $0 * $0 }` (Swift no tiene comprehension nativa) |
| Filter | `[x for x in lista if x > 0]` | `lista.filter { $0 > 0 }` |

## Funciones y Closures

| Nombre | Ejemplo en Python | Ejemplo en Swift |
|---|---|---|
| Función | `def sumar(a, b): return a + b` | `func sumar(_ a: Int, _ b: Int) -> Int { a + b }` |
| Parámetro con default | `def f(x=10): ...` | `func f(x: Int = 10) { ... }` |
| Argumentos con nombre | `f(x=1, y=2)` (opcional en la llamada) | `f(x: 1, y: 2)` (obligatorio salvo `_` en la firma) |
| Variádica | `def f(*args): ...` | `func f(_ args: Int...) { ... }` |
| Closure / Lambda | `lambda x: x * 2` | `{ x in x * 2 }` |
| Trailing closure | No existe el concepto | `numeros.map { $0 * 2 }` (closure final sin paréntesis) |

## Control de flujo

| Nombre | Ejemplo en Python | Ejemplo en Swift |
|---|---|---|
| Guard (salida temprana) | No existe — `if not cond: return` | `guard cond else { return }` |
| For-in | `for x in lista: usar(x)` | `for x in lista { usar(x) }` |
| While | `while cond: ...` | `while cond { ... }` |
| Range | `range(0, 10)` (exclusivo) | `0..<10` (exclusivo) / `0...10` (inclusivo) |

### If / else

```python
if x > 0:
    hacer_algo()
else:
    hacer_otra_cosa()
```

```swift
if x > 0 {
    hacerAlgo()
} else {
    hacerOtraCosa()
}
```

### Switch / Match

```python
match valor:
    case 1:
        print("uno")
    case _:
        print("otro")
```

```swift
switch valor {
case 1:
    print("uno")
default:
    print("otro")
}
```

## Manejo de errores

| Nombre | Ejemplo en Python | Ejemplo en Swift |
|---|---|---|
| Declarar que puede fallar | No hace falta declarar nada | `func f() throws { ... }` (obligatorio marcar la función) |

### Try / catch

```python
try:
    algo()
except ValueError as e:
    manejar(e)
```

```swift
do {
    try algo()
} catch {
    manejar(error)
}
```

### Lanzar un error propio

```python
raise ValueError("mensaje")
```

```swift
enum MiError: Error { case mensaje }
throw MiError.mensaje
```

## Clases y OOP

| Nombre | Ejemplo en Python | Ejemplo en Swift |
|---|---|---|
| Definir clase | `class Animal:` | `class Animal {` |
| Constructor | `def __init__(self, nombre): self.nombre = nombre` | `init(nombre: String) { self.nombre = nombre }` |
| Herencia | `class Perro(Animal):` | `class Perro: Animal {` |
| Llamar al padre | `super().__init__(nombre)` | `super.init(nombre: nombre)` |
| Método de instancia | `def ladrar(self): ...` | `func ladrar() { ... }` |
| Destructor | `def __del__(self): ...` | `deinit { ... }` |
| Privado | `self.__x` (name mangling, convención — no enforced) | `private var x` (enforced por el compilador) |
| Protegido | `self._x` (convención, no enforced) | `internal var x` (default) / `fileprivate var x` |

### Propiedad computada

```python
@property
def area(self):
    return self.ancho * self.alto
```

```swift
var area: Double {
    ancho * alto
}
```

### Método estático vs método de clase

```python
class Circulo:
    @staticmethod
    def crear_unitario():
        return Circulo(radio=1)

    @classmethod
    def desde_diametro(cls, d):
        return cls(radio=d / 2)
```

```swift
class Circulo {
    static func crearUnitario() -> Circulo {   // no se puede overridear
        Circulo(radio: 1)
    }

    class func desdeDiametro(_ d: Double) -> Circulo {  // sí se puede overridear
        Circulo(radio: d / 2)
    }
}
```

## Protocolos, interfaces y "duck typing"

| Nombre | Ejemplo en Python | Ejemplo en Swift |
|---|---|---|
| Extender un tipo existente | Monkey-patching (poco común, no idiomático) | `extension Int { func esPar() -> Bool { self % 2 == 0 } }` (idiomático) |
| Value type (se copia) | No existe la distinción — todo objeto es referencia | `struct` |
| Reference type (se comparte) | `class` (todo en Python es así) | `class` (mismo comportamiento que Python) |

### Interfaz / Protocolo

```python
class MiProtocolo(Protocol):
    def hacer(self) -> None: ...

class X(MiProtocolo):        # duck typing: alcanza con tener el método
    def hacer(self) -> None:
        print("hecho")
```

```swift
protocol MiProtocolo {
    func hacer()
}

class X: MiProtocolo {       // conformancia explícita, declarada
    func hacer() {
        print("hecho")
    }
}
```

## Dunder methods vs protocolos/operadores de Swift

Los `__metodo__` de Python no son "magia" — son el mismo mecanismo que en Swift se resuelve con **conformancia a protocolos** y **operadores custom**: una forma estandarizada de que un tipo propio se comporte como los tipos nativos del lenguaje (se imprima, se compare, se sume, se itere).

| Qué hace | Ejemplo en Python | Ejemplo en Swift |
|---|---|---|
| Suma (`+`) | `def __add__(self, other): ...` | `static func + (l: X, r: X) -> X { ... }` (operador custom, sin protocolo obligatorio) |
| Longitud (`len(obj)`) | `def __len__(self): ...` | `var count: Int { ... }` (convención, o conformar `Collection`) |
| Acceso por índice (`obj[i]`) | `def __getitem__(self, i): ...` | `subscript(i: Int) -> T { ... }` |
| Llamar como función (`obj()`) | `def __call__(self): ...` | No hay equivalente directo — los closures ya son de primera clase, no hace falta este patrón |

### Representación en texto (`str(obj)` / `print(obj)`)

```python
class Punto:
    def __str__(self):
        return f"Punto({self.x}, {self.y})"
```

```swift
extension Punto: CustomStringConvertible {
    var description: String {
        "Punto(\(x), \(y))"
    }
}
```

### Igualdad (`==`)

```python
class Punto:
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
```

```swift
extension Punto: Equatable {
    static func == (l: Punto, r: Punto) -> Bool {
        l.x == r.x && l.y == r.y
    }
}
```

### Comparación (`<`, `>`)

```python
class Version:
    def __lt__(self, other):
        return self.numero < other.numero
```

```swift
extension Version: Comparable {
    static func < (l: Version, r: Version) -> Bool {
        l.numero < r.numero
    }
}
```

### Iterar (`for x in obj`)

```python
class Coleccion:
    def __iter__(self):
        return iter(self.items)
```

```swift
extension Coleccion: Sequence {
    func makeIterator() -> some IteratorProtocol {
        items.makeIterator()
    }
}
```

### Cleanup garantizado (`with obj:` vs `defer`)

```python
class Recurso:
    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.cerrar()
```

```swift
func usarRecurso() {
    let recurso = Recurso()
    defer { recurso.cerrar() }   // se ejecuta siempre al salir de la función
    // usar recurso acá
}
```

Mismo propósito (garantizar que algo se limpie), mecanismo distinto: `with` envuelve un bloque completo desde afuera; `defer` se declara adentro de la función y corre al salir de su scope, sin necesidad de envolver nada.

---
Relacionado: [Sistema de tipos comparado](../tipos-comparativa.md) (mismo estilo de tabla, enfocado en tipos), [Sintaxis general de Python](sintaxis.md), [OOP en Python](oop.md).
