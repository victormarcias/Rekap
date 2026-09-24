# API Gateway

A single entry point that receives all external traffic and routes it to the right internal service — often confused with a Load Balancer, but they solve different problems.

## Gateway vs Load Balancer

A [Load Balancer](load-balancers.md) distributes traffic across **replicas of the same service** (round robin, least connections). An API Gateway routes between **different services** (`/orders` goes to the orders service, `/users` goes to the users service) and usually also handles cross-cutting concerns that no individual service should have to implement on its own:

- **Centralized authentication**: validates the token once, at the edge, before the request reaches any internal service.
- **Rate limiting**: applies usage limits per client/API key in one place (see [429 Too Many Requests](../system-design/http-status-codes.md#4xx--client-error)).
- **Request/response transformation**: adapts formats between what the external client sees and what each internal service expects.
- **Aggregation**: a single client request can translate into several calls to different internal services, combining the responses into one — the **BFF (Backend for Frontend)** pattern is a variant of this, a custom gateway for each type of client (web, mobile).

```
Client → API Gateway → /orders/*  → Order Service
                      → /users/*   → User Service
                      → /payments/* → Payment Service
```

## Why it matters in microservices

Without a gateway, every [microservice](monolith-vs-microservices.md) would have to implement its own authentication, its own rate limiting, its own CORS handling — repeated N times, with N chances to do it differently or to have a security bug in just one of them. The gateway centralizes that once, at the edge of the system.

**The cost**: it's a single point of failure if it's not properly redundant, and it adds an extra network hop (latency) to every request.

## Common tools

Kong, AWS API Gateway, Traefik (already mentioned as L7 in [Load balancers](load-balancers.md) — many tools act as both a load balancer and a lightweight gateway, the line between the two roles isn't always sharp in practice).

---
Related: [Load balancers](load-balancers.md), [Monolith vs Microservices](monolith-vs-microservices.md), [Authentication and Security](authentication.md).
