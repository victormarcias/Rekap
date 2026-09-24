# Sistema de tipos: TypeScript vs Python vs Swift vs Kotlin vs Java

| Concepto | TypeScript | Python | Swift | Kotlin | Java |
|---|---|---|---|---|---|
| Type alias | `type X = ...` | `type X = ...` (3.12+) | `typealias X = ...` | `typealias X = ...` | No existe — se usa una clase |
| Union | `A \| B` | `A \| B` (3.10+) | No nativo — se resuelve con `enum` de associated values | No nativo — se resuelve con `sealed class` | No existe — se usa herencia |
| Intersection | `A & B` | No tiene sintaxis nativa | `A & B` (protocol composition — mismo símbolo) | No nativo — se implementan varias interfaces | Se implementan varias interfaces |
| Nullable | `T \| null` | `T \| None` (3.10+) / `Optional[T]` | `T?` | `T?` | `Optional<T>` (wrapper, no sintaxis nativa) |
| Optional Chaining | `?.` | No tiene sintaxis nativa — `getattr(obj, 'attr', None)` o chequeo manual | `?.` (safe navigation, mismo símbolo) | `?.` (safe call, mismo símbolo) | No tiene sintaxis nativa — `Optional.map()` encadenado o chequeo manual |
| Generics | `<T>` | `TypeVar` / `list[T]` | `<T>` | `<T>` | `<T>` |
| `unknown` vs `any` | `unknown` (seguro) / `any` (apaga el chequeo) | No hay distinción — sin type hint, todo es dinámico | No tiene un equivalente directo | `Any` (obliga a castear, como `unknown`) | `Object` (obliga a castear, como `unknown`) |
| `never` | `never` | `NoReturn` (de `typing`) | `Never` | `Nothing` | No existe un equivalente directo |

**Patrón que se repite**: Swift, Kotlin y TypeScript son los más parecidos entre sí — los tres tienen `?` nativo para nullable y `typealias`/`type` de primera clase. Java es el más distinto: no tiene unions ni intersections nativas (se resuelven con herencia/interfaces), y el nullable requiere el wrapper `Optional<T>` en vez de sintaxis del lenguaje. Python, pese a ser dinámico, adoptó el mismo símbolo `|` para unions y nullable a partir de la 3.10 — pero, a diferencia de los otros cuatro, ese tipado sigue siendo opcional y no se enforcea en runtime (ver [Sintaxis general](python/sintaxis.md)).

---
Relacionado: [Sistema de tipos (TypeScript)](typescript/tipos.md), [React Fundamentos](../frontend-react/react-fundamentals.es.md#el-mismo-patrón-en-mobile) (mismo tipo de comparación, para Virtual DOM en vez de tipos).
