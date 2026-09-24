# GraphQL

Alternativa a [REST](rest.es.md) para diseñar APIs: el cliente pide exactamente los datos que necesita en una query, en vez de que el servidor decida de antemano qué shape devuelve cada endpoint.

## El problema que resuelve: over-fetching y under-fetching

- **Over-fetching**: un endpoint REST devuelve el objeto completo aunque el cliente solo necesite 2 campos de 20 (una lista que solo muestra nombre y precio, pero el endpoint manda también descripción, stock, categoría...).
- **Under-fetching**: lo opuesto — el cliente necesita datos de varios recursos relacionados y termina encadenando múltiples requests (ver [HTTP chaining](../diagnostics/backend.es.md#http-chaining)).

GraphQL resuelve los dos a la vez: un solo request, el cliente especifica el shape exacto que necesita, incluso cruzando relaciones.

```graphql
query {
  order(id: 1) {
    total
    items {
      name
      price
    }
  }
}
```

En REST, lo mismo requeriría `GET /orders/1` + `GET /orders/1/items` (o un endpoint a medida armado específicamente para esa combinación).

## Un solo endpoint, schema fuerte

Toda la API vive detrás de un único endpoint (normalmente `POST /graphql`) — no hay una URL por recurso. El servidor expone un schema tipado que define exactamente qué se puede pedir, y el cliente puede introspectarlo (herramientas como GraphiQL/Apollo Studio lo usan para autocompletar queries).

```graphql
type Order {
  id: ID!
  total: Float!
  items: [OrderItem!]!
}

type Query {
  order(id: ID!): Order
}
```

## Resolvers y el riesgo de N+1

Cada campo del schema tiene un **resolver** — la función que sabe cómo obtener ese dato. Sin cuidado, resolver `items` dentro de cada `order` de una lista dispara una query separada por order — el mismo [problema N+1](../diagnostics/backend.es.md#problema-n1) de siempre, ahora escondido detrás de la conveniencia de la query. La solución típica es un **dataloader** que batchea y cachea esas resoluciones dentro del mismo request.

## Trade-offs contra REST

- **Cache**: REST se apoya en la cache HTTP estándar — URLs distintas son cacheables independientemente (ver [CDN](../devops/cdn.es.md)). GraphQL usa un solo endpoint por `POST`, no cacheable por HTTP nativo — hace falta cache a nivel aplicación (ej. cache normalizada del lado del cliente, como hace Apollo Client).
- **Complejidad del servidor**: hay que diseñar resolvers, cuidar N+1, y limitar la profundidad de queries anidadas (un cliente malicioso podría pedir una query gigante y carísima de resolver).
- **Simplicidad del cliente**: el cliente pide justo lo que necesita para cada pantalla, sin depender de que el backend agregue un endpoint custom cada vez que cambia un requerimiento de UI.

## Cuándo conviene cada uno

**REST**: APIs públicas simples, cuando la cacheabilidad HTTP nativa importa, equipos que ya conocen bien HTTP. **GraphQL**: apps con muchas pantallas distintas consumiendo datos relacionados de formas variadas (mobile + web con necesidades distintas del mismo backend), cuando minimizar la cantidad de requests importa más que la simplicidad de cache.

---
Relacionado: [REST](rest.es.md), [Diagnóstico Backend](../diagnostics/backend.es.md#problema-n1) (N+1), [HTTP chaining](../diagnostics/backend.es.md#http-chaining).
