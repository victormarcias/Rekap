# React — Fundamentos

Cómo funciona React por dentro, antes de entrar en hooks (ver [Hooks](hooks.md)) o performance (ver [Diagnóstico Frontend](../diagnostics/frontend.es.md)).

## Virtual DOM

Un **Virtual DOM** es un árbol de objetos JS liviano que representa la UI deseada — no toca el DOM real. Cuando el estado cambia, React arma un Virtual DOM nuevo, lo compara (*diffing*) contra el anterior, y solo aplica al DOM real los cambios mínimos necesarios (*reconciliation*).

```jsx
// cambiar solo el texto de un <li> no recrea la lista entera —
// React compara el árbol anterior contra el nuevo y actualiza
// solamente el nodo de texto que cambió
function List({ items }) {
  return <ul>{items.map(item => <li key={item.id}>{item.text}</li>)}</ul>;
}
```

**Por qué existe**: manipular el DOM real es caro (cada cambio puede disparar layout/paint del browser). Comparar objetos JS en memoria es barato. El Virtual DOM le permite a React calcular *qué* cambió sin tocar el DOM en cada paso intermedio, y aplicar todos los cambios reales de una sola vez, en el mínimo número de operaciones posible.

**El rol de `key`**: cuando React reconcilia una lista, usa `key` para identificar qué elemento es cuál entre un render y el siguiente — sin una `key` estable, React puede confundir "se reordenó un item" con "se borró uno y se creó otro nuevo", perdiendo estado interno de esos componentes innecesariamente (ver [Falta de `key` en listas](../diagnostics/frontend.es.md#falta-de-key-en-listas)).

## JSX

Sintaxis que mezcla HTML-like markup con JS, pero **no es HTML** — es azúcar sintáctica que un compilador (Babel/SWC) transforma en llamadas a función antes de que el browser vea una sola línea. Es la forma que tiene **React** (web y React Native por igual) de describir el árbol del Virtual DOM de arriba — no es exclusiva de web, es exclusiva de React como librería.

```jsx
// esto...
const element = <h1 className="title">Hola {nombre}</h1>;

// ...se transpila a esto (React 17+, "nuevo" JSX transform):
import { jsx as _jsx } from 'react/jsx-runtime';
const element = _jsx('h1', { className: 'title', children: `Hola ${nombre}` });
```

Por eso JSX puede usar `{}` para meter cualquier expresión JS válida (variables, funciones, ternarios) — en tiempo de compilación termina siendo un argumento más de una llamada a función normal. Y por eso un componente de React **tiene** que devolver JSX válido (o `null`) — no es HTML libre, tiene las reglas de una expresión JS (ej. `class` no existe, es `className`, porque `class` es palabra reservada en JS).

## El mismo patrón en Mobile

"Describir declarativamente cómo se ve la UI, comparar contra la versión anterior, aplicar solo el mínimo cambio real" no es una idea exclusiva de React — es la solución convergente de casi todo framework de UI declarativa moderno. Cada ecosistema llegó a esto por separado porque el problema de fondo es el mismo: recalcular *todo* el árbol de UI real en cada cambio de estado es carísimo, sin importar si esa UI real es el DOM del browser o una vista nativa.

### Swift (SwiftUI, iOS)

Las `View` son structs inmutables que describen la UI deseada; SwiftUI las compara contra el árbol anterior y actualiza solo lo que cambió en el render real.

```swift
struct CounterView: View {
    @State private var count = 0
    var body: some View {
        Button("Count: \(count)") { count += 1 }
    }
}
// al cambiar `count`, SwiftUI recalcula `body`, compara contra el árbol
// anterior y actualiza solo el texto del botón — mismo mecanismo que React
```

### Kotlin (Jetpack Compose, Android)

Mismo mecanismo — funciones `@Composable` describen la UI, Compose hace su propio diffing ("recomposition") y actualiza solo las partes afectadas.

```kotlin
@Composable
fun CounterView() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) { Text("Count: $count") }
}
// al cambiar `count`, Compose "recompone" solo lo afectado — mismo mecanismo, otro nombre
```

### React Native

Literalmente el mismo React/Virtual DOM que la versión web, pero con un renderer distinto al final — en vez de aplicar los cambios a nodos del DOM del browser, los aplica a vistas nativas reales (`UIView` en iOS, `View` de Android). Es **React DOM** (el renderer web) el que es específico de web, no React ni el Virtual DOM en sí — React Native reusa exactamente el mismo core y el mismo JSX de arriba. Para la práctica (componentes, styling, navegación, el bridge con código nativo) ver [React Native — Fundamentos](../stacks/react-native/fundamentos.md).

---
Relacionado: [Hooks](hooks.md), [Diagnóstico Frontend](../diagnostics/frontend.es.md).
