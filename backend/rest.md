# REST

An architectural style for designing APIs — not the same thing as "using HTTP methods" (that's just one of its six constraints).

## The 6 constraints

1. **Client-server**: separation of concerns — the client doesn't know how the server is implemented (language, DB), the server doesn't know how the UI is rendered. Each side evolves independently.
2. **Stateless**: every request carries all the information needed to process it — the server doesn't store context from the same client's previous requests between calls. See [Scalability](../system-design/atributos-de-calidad.md#escalabilidad) (shared session vs in-memory session — statelessness is what enables it).
3. **Cacheable**: responses explicitly indicate whether they're cacheable (`Cache-Control`), so the client or intermediaries can reuse them without re-requesting. See [CDN](../devops/cdn.md).
4. **Uniform interface**: resources identified by URLs, manipulated with standard HTTP verbs (see [HTTP Methods](http-methods.md)) and self-descriptive representations (JSON with Content-Type) — plus HATEOAS, see below.
5. **Layered system**: the client can't (and doesn't need to) know whether it's talking directly to the origin server or to a proxy/gateway/load balancer in between. See [API Gateway](api-gateway.md), [Load balancers](load-balancers.md).
6. **Code-on-demand** (optional): the server can extend the client's functionality by sending executable code — the only non-mandatory constraint of the six.

## Resource conventions (part of the uniform interface)

```
❌ POST /createOrder
✅ POST /orders

❌ GET /getOrderItems?orderId=1
✅ GET /orders/1/items
```

Plural nouns instead of verbs in the URL (the HTTP method already says the verb), and nesting to express relationships between resources.

## HATEOAS — the constraint almost nobody implements

**Hypermedia As The Engine Of Application State**: every response should include links to the actions/resources available from that state, so the client navigates the API dynamically instead of having hardcoded URLs ahead of time — like a browser following links on a website, instead of having the entire structure memorized in advance.

```json
{
  "id": 1,
  "status": "pending",
  "total": 100,
  "_links": {
    "self": { "href": "/orders/1" },
    "cancel": { "href": "/orders/1/cancel", "method": "POST" },
    "pay": { "href": "/orders/1/pay", "method": "POST" }
  }
}
```

Without this, the client needs to know **all** possible URLs in advance, typically through external documentation — with HATEOAS, the API is self-discoverable.

## "RESTful" vs real REST

In practice, almost no API that calls itself "REST" implements HATEOAS — most only satisfy "resources + HTTP verbs + JSON," which is barely half of the uniform interface constraint. Worth keeping in mind: if someone asks "is your API REST?", the honest answer is almost always "it's HTTP with REST conventions, not full REST."

---
Related: [HTTP Methods](http-methods.md), [Scalability](../system-design/atributos-de-calidad.md#escalabilidad), [API Gateway](api-gateway.md), [CDN](../devops/cdn.md), [GraphQL](graphql.md) (the alternative).
