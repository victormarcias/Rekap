# HTTP Methods

HTTP verbs have semantics defined by the protocol — using them "just because" (everything as `POST`) throws that information away.

## The verbs and their purpose

- **GET**: read a resource. Shouldn't have side effects.
- **POST**: create a new resource, or execute an action that doesn't fit the other verbs.
- **PUT**: replace a resource **entirely** — the body sends the whole object, whatever isn't included gets lost/reset.
- **PATCH**: **partially** update a resource — the body only sends the fields that change.
- **DELETE**: delete a resource.

```
PUT /users/1    {"name": "Ana", "email": "ana@mail.com"}
  → replaces the entire user; if it had a "phone" field and you didn't send it, it's lost

PATCH /users/1  {"email": "new@mail.com"}
  → only updates the email, the rest of the user stays intact
```

## Safe methods — no side effects

**GET**, **HEAD**, **OPTIONS** are *safe*: they shouldn't change server state. This isn't just a convention — a browser, a crawler, or a proxy can freely retry a `GET` (for caching, prefetching, etc.) assuming it doesn't matter how many times it runs. A `GET` endpoint that deletes data breaks that guarantee and can cause accidental deletions from tools that don't expect a `GET` to have side effects.

## Idempotency by verb

Already covered in detail in [Idempotency](../system-design/atributos-de-calidad.md#idempotencia) — quick recap applied to the verbs:

| Verb | Idempotent | Why |
|---|---|---|
| GET | ✅ | Reading changes nothing, no matter how many times |
| PUT | ✅ | Replacing with the same value N times gives the same result as once |
| DELETE | ✅ | Deleting something that no longer exists still returns "doesn't exist" |
| PATCH | Depends | If the patch is `{"stock": 5}` yes; if it's `{"stock": stock - 1}` no — each application subtracts again |
| POST | ❌ | Every `POST` creates a new resource — retrying without an idempotency key duplicates it |

This is exactly why an automatic network retry is safe on a `PUT`/`DELETE` but risky on a `POST` without an idempotency key.

---
Related: [REST](rest.md) (the methods are one piece of its uniform interface, not the same thing as REST), [Idempotency](../system-design/atributos-de-calidad.md#idempotencia), [HTTP Status Codes](../system-design/http-status-codes.md) (`405 Method Not Allowed` when a resource doesn't support the requested verb).
