# Estado global: Context API vs Redux

## Prop drilling — el problema

Pasar una prop a través de varios componentes intermedios que no la usan, solo para que llegue a un componente hijo que sí la necesita. Cuanto más profundo el árbol, más componentes intermedios quedan acoplados a una prop que les es irrelevante — y cambiar el shape de ese dato obliga a tocar todos los intermedios.

```jsx
// user viaja por Layout y Sidebar sin que ninguno de los dos lo use
function App({ user }) {
  return <Layout user={user} />;
}
function Layout({ user }) {
  return <Sidebar user={user} />;
}
function Sidebar({ user }) {
  return <UserBadge user={user} />; // el único que realmente lo necesita
}
```

## Context API — la solución nativa

Provee un valor en un punto del árbol y lo hace disponible directo a cualquier descendiente vía `useContext`, sin pasar por los componentes intermedios (ver [`useContext`](hooks.es.md#usecontext)).

```jsx
const UserContext = createContext(null);

function App({ user }) {
  return (
    <UserContext.Provider value={user}>
      <Layout /> {/* ya no necesita recibir ni reenviar user */}
    </UserContext.Provider>
  );
}
function UserBadge() {
  const user = useContext(UserContext); // lee directo, sin importar la profundidad
  return <span>{user.name}</span>;
}
```

**Dónde se queda corto**: Context no es un manejador de estado optimizado — cualquier componente que consuma ese Context se re-renderiza cada vez que el `value` cambia, aunque el componente solo use una parte de ese valor. Un Context con un objeto grande que cambia seguido (ej. el estado completo de un carrito de compras que se actualiza en cada click) puede re-renderizar de más a componentes que no les importaba ese cambio puntual.

## Redux (u otras librerías externas)

Cuando el estado global es grande, cambia seguido, o necesita lógica de actualización compleja (varias acciones que lo modifican desde distintos lugares), una librería dedicada (Redux, Zustand, Jotai) agrega lo que Context no tiene: selectors que suscriben a un componente solo a la porción de estado que le importa (evitando re-renders de más), DevTools para inspeccionar cada cambio de estado paso a paso, y un patrón explícito de cómo se actualiza el estado (acciones + reducers en Redux).

```jsx
// Redux Toolkit — ejemplo mínimo
const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: {
    addItem: (state, action) => { state.items.push(action.payload); },
  },
});

function CartBadge() {
  // solo se re-renderiza si items.length cambia — no si cambia
  // cualquier otra parte del estado global de la app
  const count = useSelector(state => state.cart.items.length);
  return <span>{count}</span>;
}
```

## Cuándo usar cada uno

- **Prop drilling liso**: si son 1-2 niveles, a veces es más simple que agregar Context — no toda cadena de props merece una abstracción.
- **Context**: estado que cambia poco (tema, idioma, usuario logueado) y no tiene actualizaciones de alta frecuencia.
- **Redux/Zustand**: estado que cambia seguido, con múltiples fuentes de actualización, o donde la performance de re-renders ya es un problema medido (no una suposición — ver [Performance Diagnostics](performance-diagnostics.es.md)).

---
Relacionado: [Hooks](hooks.es.md), [Diagnóstico Frontend](../diagnostics/frontend.es.md).
