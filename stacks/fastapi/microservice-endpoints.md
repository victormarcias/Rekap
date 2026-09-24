# Microservice Endpoints

How building an endpoint meant to live in a microservice looks in Python/FastAPI code. This isn't REST theory (that's already covered in other cheat sheets) — it's the framework's concrete syntax and patterns.

## 1. Anatomy of a FastAPI endpoint

An endpoint is an async function decorated with the HTTP verb and the path. `response_model` validates and serializes the response according to that schema, **filtering out any field of the internal object that shouldn't be exposed** (e.g. `password_hash`) even if the real object has it.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class OrderOut(BaseModel):
    id: int
    total: float
    status: str

@app.get("/orders/{order_id}", response_model=OrderOut)
async def get_order(order_id: int):
    order = await order_service.get_by_id(order_id)
    return order  # even if order has more internal fields, only OrderOut's get serialized
```

## 2. Path params vs Query params vs Body

FastAPI infers where each value comes from based on the function's signature: if the name matches the path, it's a path param; a simple type not in the path is a query param; a full Pydantic model is the body. Setting limits (`le`, `ge`, `gt`) on validation prevents someone from requesting `limit=1000000` and taking down the database.

```python
from fastapi import Query, Path

@app.get("/orders")
async def list_orders(
    status: str | None = Query(None, description="Filter by status"),
    limit: int = Query(20, le=100),   # ✅ hard cap, don't trust the client to be reasonable
    offset: int = Query(0, ge=0),
):
    ...

@app.get("/orders/{order_id}")
async def get_order(order_id: int = Path(..., gt=0)):  # ✅ rejects invalid IDs before touching the DB
    ...

class CreateOrderBody(BaseModel):
    items: list[str]
    user_id: int

@app.post("/orders")
async def create_order(body: CreateOrderBody):  # a Pydantic model as a parameter = comes from the body
    ...
```

## 3. Input validation with Pydantic

Pydantic validates types, ranges, and formats automatically and returns a `422` with the exact detail of which field failed, without writing an `if` by hand for every rule. `field_validator`s cover business rules a type alone can't express.

```python
from pydantic import BaseModel, field_validator

class CreateOrderBody(BaseModel):
    items: list[str]
    user_id: int

    @field_validator("items")
    @classmethod
    def items_not_empty(cls, v):
        if not v:
            raise ValueError("The order needs at least one item")
        return v

# a request with items=[] returns 422 with the validator's message —
# the endpoint's code doesn't even get to run
```

## 4. Dependency Injection with `Depends`

`Depends` resolves a shared dependency (DB session, user authenticated from the token) before running the handler, avoiding repeating that logic in every endpoint — it's Dependency Inversion (see [SOLID](../../system-design/solid.md)) applied at the framework level: the endpoint receives an already-ready session, without knowing how it was built.

```python
from fastapi import Depends, HTTPException, Header

async def get_db_session():
    session = SessionLocal()
    try:
        yield session       # injected into the endpoint
    finally:
        session.close()      # closes only when the request finishes, regardless of the outcome

async def get_current_user(authorization: str = Header(...)):
    user = decode_token(authorization)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

@app.get("/orders/{order_id}")
async def get_order(
    order_id: int,
    db=Depends(get_db_session),
    current_user=Depends(get_current_user),
):
    return await order_repository.find(db, order_id, current_user.id)
```

## 5. Exception handling with `@app.exception_handler`

Centralizes error handling in one place instead of repeating `try/except` in every endpoint, and avoids the security risk of an unhandled error returning a `500` with the full stack trace leaked to the client. The endpoint just raises the domain exception — it knows nothing about HTTP codes.

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class OrderNotFoundError(Exception):
    def __init__(self, order_id: int):
        self.order_id = order_id

@app.exception_handler(OrderNotFoundError)
async def order_not_found_handler(request: Request, exc: OrderNotFoundError):
    return JSONResponse(status_code=404, content={"error": "order_not_found", "order_id": exc.order_id})

@app.get("/orders/{order_id}")
async def get_order(order_id: int):
    order = await order_service.get_by_id(order_id)
    if not order:
        raise OrderNotFoundError(order_id)
    return order
```

## 6. Automatic documentation (OpenAPI)

FastAPI automatically generates Swagger UI (`/docs`) and ReDoc (`/redoc`) from type hints and Pydantic models — there's no separate spec file that can drift out of sync with the real code. In microservices this matters because another team needs to know your service's contract without reading the code or asking you for a meeting.

```python
@app.get("/orders/{order_id}", response_model=OrderOut, summary="Get an order by ID")
async def get_order(order_id: int):
    ...
# with this, /docs already shows the full schema, types, and request/response examples
```

## 7. Idempotency in writes

If the client retries the same `POST` after a timeout (without knowing whether the first attempt was processed), without an idempotency key that retry duplicates the order. By storing the result per key, the second attempt returns the same response instead of creating a new resource.

```python
from fastapi import Header

processed_keys: dict[str, dict] = {}  # in production: Redis with TTL, not a dict in process memory

@app.post("/orders")
async def create_order(body: CreateOrderBody, idempotency_key: str = Header(...)):
    if idempotency_key in processed_keys:
        return processed_keys[idempotency_key]  # already processed, don't create a new order

    order = await order_service.create(body)
    processed_keys[idempotency_key] = order
    return order
```

## 8. Testing endpoints with `TestClient`

`TestClient` (based on `httpx`) lets you test endpoints without spinning up a real server — it runs in-process, so tests are fast and can run in CI with no extra infrastructure.

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_get_order_not_found():
    response = client.get("/orders/999")
    assert response.status_code == 404
    assert response.json()["error"] == "order_not_found"

def test_create_order_validates_empty_items():
    response = client.post("/orders", json={"items": [], "user_id": 1})
    assert response.status_code == 422  # fails in Pydantic, doesn't even reach the handler
```

---
Related: [SOLID principles](../../system-design/solid.md), [System quality attributes](../../system-design/quality-attributes.md) (idempotency, fault tolerance), [General Syntax](../python/syntax.md), [Sync vs Async in FastAPI](sync-vs-async.md) (when the `async def` in these examples actually helps).
