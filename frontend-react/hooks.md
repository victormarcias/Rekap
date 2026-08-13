# Hooks

A diferencia del resto de los temas de esta carpeta, esto no tiene versión "genérica" — los hooks son una API específica de React (y de los frameworks que copiaron el modelo, como Preact). `useState`/`useEffect`/`memo`/`useMemo`/`useCallback` ya se cubrieron en detalle, con el foco puesto en performance, en [Diagnóstico Frontend](../diagnostico/frontend.md). Acá el foco es el mecanismo por detrás: las reglas y cómo armar hooks propios.

## Reglas de hooks

1. **Solo llamar hooks en el nivel superior** de un componente — nunca dentro de un `if`, un loop, o una función anidada.
2. **Solo llamar hooks desde componentes de React o desde otros hooks** — nunca desde una función JS común.

```jsx
// ❌ rompe la regla 1: si la condición cambia entre renders, el orden de los hooks cambia
function Component({ show }) {
  if (show) {
    const [value, setValue] = useState(0); // a veces se llama, a veces no
  }
}

// ✅ el hook siempre se llama, la condición va adentro
function Component({ show }) {
  const [value, setValue] = useState(0);
  if (show) { /* usar value acá */ }
}
```

**Por qué existe esta regla**: React no identifica cada hook por nombre, sino por el **orden** en que se llaman durante el render — internamente los guarda en una lista y los asocia por posición. Si un hook a veces se llama y a veces no, el orden se desalinea entre renders y React le asigna a un hook el estado que le correspondía a otro.

## Custom hooks

Una función que empieza con `use` y llama a otros hooks adentro — la forma de extraer lógica con estado reutilizable entre componentes, sin duplicar código ni recurrir a patrones más pesados (HOCs, render props).

```jsx
// ✅ custom hook: encapsula fetch + loading + error, reutilizable en cualquier componente
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url).then(r => r.json()).then(setData).finally(() => setLoading(false));
  }, [url]);

  return { data, loading };
}

// uso: cualquier componente pide solo lo que necesita, sin repetir la lógica de fetch
function UserProfile({ userId }) {
  const { data, loading } = useFetch(`/api/users/${userId}`);
  if (loading) return <Spinner />;
  return <div>{data.name}</div>;
}
```

Un custom hook no comparte estado entre los componentes que lo usan — cada llamada tiene su propia instancia de `useState`/`useEffect`, como si el código estuviera copiado y pegado (pero sin estarlo).

## `useRef`

Guarda un valor mutable que **persiste entre renders sin causar un re-render** cuando cambia — a diferencia de `useState`, escribir en `ref.current` no le avisa a React que algo cambió. Dos usos típicos:

```jsx
// 1. Referencia a un nodo del DOM real (ej. para hacer foco manualmente)
function SearchInput() {
  const inputRef = useRef(null);
  useEffect(() => { inputRef.current.focus(); }, []);
  return <input ref={inputRef} />;
}

// 2. Guardar un valor que necesita sobrevivir renders pero no debe disparar un re-render
function Timer() {
  const intervalId = useRef(null);
  const start = () => { intervalId.current = setInterval(() => {}, 1000); };
  const stop = () => clearInterval(intervalId.current);
  return <button onClick={start}>Start</button>;
}
```

Si el valor debe reflejarse en la UI, es `useState`; si es "bookkeeping" interno que la UI no necesita mostrar, es `useRef`.

## `useContext`

Lee un valor provisto más arriba en el árbol por un `Context.Provider`, sin tener que pasarlo manualmente prop por prop a través de cada componente intermedio (ver [prop drilling](estado-global.md#prop-drilling-el-problema)).

```jsx
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  // Toolbar no usa el theme, pero antes tenía que recibirlo igual
  // para poder pasárselo a ThemedButton — con Context ya no
  return <ThemedButton />;
}

function ThemedButton() {
  const theme = useContext(ThemeContext); // 'dark' — lee directo, sin pasar por Toolbar
  return <button className={theme}>Click</button>;
}
```

Cualquier componente que use `useContext` se re-renderiza cuando el `value` del Provider cambia, sin importar cuán abajo esté en el árbol — ver [Estado global: Context API vs Redux](estado-global.md) para cuándo esto se vuelve un problema de performance y qué alternativas hay.

---
Relacionado: [Diagnóstico Frontend](../diagnostico/frontend.md) (`useEffect`, `memo`/`useMemo`/`useCallback`), [Estado global](estado-global.md), [React Fundamentos](react-fundamentos.md).
