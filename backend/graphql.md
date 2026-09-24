# GraphQL

An alternative to [REST](rest.md) for designing APIs: the client requests exactly the data it needs in a query, instead of the server deciding in advance what shape each endpoint returns.

## The problem it solves: over-fetching and under-fetching

- **Over-fetching**: a REST endpoint returns the full object even when the client only needs 2 fields out of 20 (a list that only shows name and price, but the endpoint also sends description, stock, category...).
- **Under-fetching**: the opposite — the client needs data from several related resources and ends up chaining multiple requests (see [HTTP chaining](../diagnostics/backend.md#http-chaining)).

GraphQL solves both at once: a single request, the client specifies the exact shape it needs, even across relationships.

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

In REST, the same thing would require `GET /orders/1` + `GET /orders/1/items` (or a custom endpoint built specifically for that combination).

## A single endpoint, strong schema

The entire API lives behind a single endpoint (usually `POST /graphql`) — there's no one URL per resource. The server exposes a typed schema that defines exactly what can be requested, and the client can introspect it (tools like GraphiQL/Apollo Studio use this to autocomplete queries).

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

## Resolvers and the N+1 risk

Every field in the schema has a **resolver** — the function that knows how to fetch that data. Without care, resolving `items` inside each `order` in a list fires a separate query per order — the same old [N+1 problem](../diagnostics/backend.md#n1-problem), now hidden behind the query's convenience. The typical fix is a **dataloader** that batches and caches those resolutions within the same request.

## Trade-offs against REST

- **Cache**: REST relies on standard HTTP caching — different URLs are independently cacheable (see [CDN](../devops/cdn.md)). GraphQL uses a single `POST` endpoint, not cacheable by native HTTP — application-level caching is needed (e.g. normalized client-side cache, like Apollo Client does).
- **Server complexity**: resolvers have to be designed, N+1 has to be watched for, and nested query depth has to be limited (a malicious client could request a huge, expensive-to-resolve query).
- **Client simplicity**: the client requests exactly what it needs for each screen, without depending on the backend adding a custom endpoint every time a UI requirement changes.

## When each one makes sense

**REST**: simple public APIs, when native HTTP cacheability matters, teams that already know HTTP well. **GraphQL**: apps with many different screens consuming related data in varied ways (mobile + web with different needs from the same backend), when minimizing the number of requests matters more than cache simplicity.

---
Related: [REST](rest.md), [Backend Diagnostics](../diagnostics/backend.md#n1-problem) (N+1), [HTTP chaining](../diagnostics/backend.md#http-chaining).
