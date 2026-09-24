# Type system: TypeScript vs Python vs Swift vs Kotlin vs Java

| Concept | TypeScript | Python | Swift | Kotlin | Java |
|---|---|---|---|---|---|
| Type alias | `type X = ...` | `type X = ...` (3.12+) | `typealias X = ...` | `typealias X = ...` | Doesn't exist — a class is used |
| Union | `A \| B` | `A \| B` (3.10+) | Not native — resolved with an `enum` of associated values | Not native — resolved with a `sealed class` | Doesn't exist — inheritance is used |
| Intersection | `A & B` | No native syntax | `A & B` (protocol composition — same symbol) | Not native — implements several interfaces | Implements several interfaces |
| Nullable | `T \| null` | `T \| None` (3.10+) / `Optional[T]` | `T?` | `T?` | `Optional<T>` (wrapper, no native syntax) |
| Optional Chaining | `?.` | No native syntax — `getattr(obj, 'attr', None)` or a manual check | `?.` (safe navigation, same symbol) | `?.` (safe call, same symbol) | No native syntax — chained `Optional.map()` or a manual check |
| Generics | `<T>` | `TypeVar` / `list[T]` | `<T>` | `<T>` | `<T>` |
| `unknown` vs `any` | `unknown` (safe) / `any` (turns off checking) | No distinction — with no type hint, everything is dynamic | No direct equivalent | `Any` (forces a cast, like `unknown`) | `Object` (forces a cast, like `unknown`) |
| `never` | `never` | `NoReturn` (from `typing`) | `Never` | `Nothing` | No direct equivalent exists |

**A pattern that repeats**: Swift, Kotlin, and TypeScript are the most similar to each other — all three have native `?` for nullable and first-class `typealias`/`type`. Java is the most different: it has no native unions or intersections (resolved with inheritance/interfaces), and nullable requires the `Optional<T>` wrapper instead of language syntax. Python, despite being dynamic, adopted the same `|` symbol for unions and nullable starting in 3.10 — but, unlike the other four, that typing remains optional and isn't enforced at runtime (see [General Syntax](python/syntax.md)).

---
Related: [Type System (TypeScript)](typescript/types.md), [React Fundamentals](../frontend-react/react-fundamentals.md#the-same-pattern-on-mobile) (the same kind of comparison, for Virtual DOM instead of types).
