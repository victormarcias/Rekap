# Estados de un Request

Cualquier componente que muestra datos que vienen de una API tiene que contemplar **4 estados**, no solo "los datos ya llegaron":

1. **Loading / Pending** — el request está en vuelo, todavía no hay respuesta.
2. **Success / Data** — resolvió bien, ya tenés los datos para mostrar.
3. **Error** — falló (error de red, o un status code que no es 2xx — ver [HTTP Status Codes](../system-design/http-status-codes.es.md)).
4. **Retry** — qué hacer cuando falla: ¿reintentar automático?, ¿cuántas veces?, ¿con qué espera entre intentos?

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);       // estado 3: error
  const [retryCount, setRetryCount] = useState(0); // estado 4: cuántas veces ya reintentó

  useEffect(() => {
    setLoading(true);
    setError(null);

    fetch(url)
      .then(r => {
        if (!r.ok) throw new Error(`Status ${r.status}`);
        return r.json();
      })
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url, retryCount]);   // retryCount en las deps: cambiarlo dispara un nuevo intento

  const retry = () => setRetryCount(c => c + 1);

  return { data, loading, error, retry };
}
```

```jsx
function UserProfile({ userId }) {
  const { data, loading, error, retry } = useFetch(`/api/users/${userId}`);

  if (loading) return <Spinner />;
  if (error) return <button onClick={retry}>Error al cargar — reintentar</button>;
  return <div>{data.name}</div>;
}
```

**Por qué falta tan seguido**: es fácil cubrir Loading y Success (son los dos casos "felices"), y quedarse corto en Error y Retry — el resultado es una UI que se cuelga silenciosa o muestra un error sin salida cuando la red falla, en vez de darle al usuario una forma de reintentar.

**Reintentos automáticos vs manuales**: reintentar automático (sin límite) puede convertirse en un loop que nunca corta — mismo problema, mismo mecanismo de solución que ya vimos con [Circuit Breaker](../system-design/quality-attributes.es.md#tolerancia-a-fallos): cortar después de N intentos fallidos, en vez de reintentar para siempre.

Este es, en el fondo, el problema que resuelven librerías como React Query o SWR — modelan estos 4 estados (más cache, revalidación, reintentos con backoff) para no tener que reimplementarlos a mano en cada `useFetch` propio.

---
Relacionado: [Hooks](hooks.es.md) (`useEffect`, el `useFetch` de ejemplo), [HTTP Status Codes](../system-design/http-status-codes.es.md), [Circuit Breaker](../system-design/quality-attributes.es.md#tolerancia-a-fallos), [Error Boundaries](error-boundaries.es.md) (errores de render, no de red — concepto distinto).
