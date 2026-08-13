# React — Fundamentos

Cómo funciona React por dentro, antes de entrar en hooks (ver [Hooks](hooks.md)) o performance (ver [Diagnóstico Frontend](../diagnostico/frontend.md)).

## JSX

Sintaxis que mezcla HTML-like markup con JS, pero **no es HTML** — es azúcar sintáctica que un compilador (Babel/SWC) transforma en llamadas a función antes de que el browser vea una sola línea.

```jsx
// esto...
const element = <h1 className="title">Hola {nombre}</h1>;

// ...se transpila a esto (React 17+, "nuevo" JSX transform):
import { jsx as _jsx } from 'react/jsx-runtime';
const element = _jsx('h1', { className: 'title', children: `Hola ${nombre}` });
```

Por eso JSX puede usar `{}` para meter cualquier expresión JS válida (variables, funciones, ternarios) — en tiempo de compilación termina siendo un argumento más de una llamada a función normal. Y por eso un componente de React **tiene** que devolver JSX válido (o `null`) — no es HTML libre, tiene las reglas de una expresión JS (ej. `class` no existe, es `className`, porque `class` es palabra reservada en JS).

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

**El rol de `key`**: cuando React reconcilia una lista, usa `key` para identificar qué elemento es cuál entre un render y el siguiente — sin una `key` estable, React puede confundir "se reordenó un item" con "se borró uno y se creó otro nuevo", perdiendo estado interno de esos componentes innecesariamente (ver [Falta de `key` en listas](../diagnostico/frontend.md#falta-de-key-en-listas)).

---
Relacionado: [Hooks](hooks.md), [Diagnóstico Frontend](../diagnostico/frontend.md).
