# HTTP Status Codes

The first digit defines the category; that digit alone already tells the client how to interpret the response without reading the body.

## 1xx — Informational

Provisional responses, before the final one — the client almost never handles these directly, the HTTP layer underneath resolves them.

- **100 Continue**: the server confirms it can receive the rest of a large request before the client sends it in full (avoids uploading a huge body only for the server to reject it afterward).
- **101 Switching Protocols**: confirms the protocol upgrade — it's the mechanism a WebSocket connection starts with (see [WebSocket / SSE / Streaming](../frontend-react/websocket-sse-streaming.md)).

## 2xx — Success

- **200 OK**: generic success, with a body in the response. The default for almost any successful `GET`/`PUT`/`PATCH`.
- **201 Created**: the request created a new resource (typically `POST`). The response should include where that resource lives (a `Location` header, or the id in the body).
- **202 Accepted**: the request was accepted but processing is asynchronous — it hasn't finished yet. Typical in work queues (see [Idempotency](quality-attributes.md#idempotency) for the retry case on this kind of request).
- **204 No Content**: success, but there's nothing to return in the body — common on `DELETE` or on a `PUT` where the client already has the updated data.

```
POST /orders          → 201 Created   (Location: /orders/42)
DELETE /orders/42      → 204 No Content
GET /orders/42          → 200 OK
```

## 3xx — Redirection

- **301 Moved Permanently**: the resource moved for good — browsers and crawlers cache this redirect aggressively, update their links.
- **302 Found**: temporary redirect — the resource is still there, it just points elsewhere this time, but nothing needs to be permanently updated.
- **304 Not Modified**: response to a conditional request (`If-None-Match`/`If-Modified-Since`) — tells the client "your cached copy is still valid, I'm not sending you the body again." The basis of HTTP caching (see [CDN](../devops/cdn.md)).

## 4xx — Client Error

The error is on the client's side — the request is malformed, unauthenticated, unauthorized, or asking for something that doesn't exist.

- **400 Bad Request**: the request is malformed or fails validation (missing a required field, an invalid type).
- **401 Unauthorized**: not authenticated — the token is missing, or invalid/expired.
- **403 Forbidden**: authenticated, but no permission for this action. See the full distinction in [Authentication vs Authorization](../backend/authentication.md#7-authentication-vs-authorization) — the most confused error on the whole list.
- **404 Not Found**: the resource doesn't exist. Also sometimes used **on purpose** instead of `403`, to avoid leaking to an attacker that a resource exists but they lack permission — depends on how much information you want to expose.
- **405 Method Not Allowed**: the resource exists, but doesn't support that HTTP verb (e.g. `DELETE /orders` when that route only accepts `GET`/`POST`).
- **409 Conflict**: the request is valid, but clashes with the resource's current state — the typical case is an update based on a stale version (see [Optimistic locking](../database/locks.md#pessimistic-vs-optimistic-locking)).
- **422 Unprocessable Entity**: the request has the correct format (valid JSON) but the data fails business rules/validation — the line with `400` is thin and varies by team; many frameworks (FastAPI included) use `422` specifically for schema validation errors.
- **429 Too Many Requests**: rate limit exceeded — usually comes with a `Retry-After` header indicating how long to wait before retrying.

## 5xx — Server Error

The error is on the server's side — the client did everything right, something broke on the other end.

- **500 Internal Server Error**: generic unhandled error — an exception that escaped without a specific handler (see [Exception handling](../stacks/fastapi/microservice-endpoints.md#5-exception-handling-with-appexception_handler)).
- **502 Bad Gateway**: a proxy/load balancer got an invalid response from the origin server (the server behind it is down or returned garbage).
- **503 Service Unavailable**: the server is temporarily unavailable (overloaded, under maintenance) — usually with `Retry-After`.
- **504 Gateway Timeout**: a proxy/load balancer waited too long for the origin server's response and gave up.

**502 vs 503 vs 504**: all three are usually returned by the load balancer, not the app — `502` is "the backend answered something I don't understand or didn't answer anything valid," `503` is "the backend isn't accepting connections right now," `504` is "the backend never answered in time." Distinguishing them helps you know where to look: `504` points to slowness (see [Backend Diagnostics](../diagnostics/backend.md)), `502`/`503` point to the process being down or never having started.

---
Related: [Authentication vs Authorization](../backend/authentication.md#7-authentication-vs-authorization), [Idempotency](quality-attributes.md#idempotency), [Endpoints for microservices](../stacks/fastapi/microservice-endpoints.md).
