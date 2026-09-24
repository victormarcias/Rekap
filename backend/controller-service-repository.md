# Controller / Service / Repository

A layered pattern for separating responsibilities in the backend: each layer knows how to do one thing, and doesn't care how the others work.

## The three layers

- **Controller**: receives the HTTP request, validates/parses the input, calls the Service, builds the response. Has no business logic — it's the translation between "HTTP" and "the app's logic."
- **Service**: the business logic itself (calculating a discount, validating a rule, orchestrating several repositories). Knows nothing about HTTP or SQL.
- **Repository**: data access — abstracts away which DB engine it is, which ORM is used, how the query is built. The Service asks it for data, doesn't care how it gets it.

```python
# Repository: only knows how to talk to the DB
class OrderRepository:
    def find_by_id(self, order_id):
        return db.query(Order).filter(Order.id == order_id).first()
    def save(self, order):
        db.add(order); db.commit()

# Service: business logic — knows nothing about HTTP or SQL
class OrderService:
    def __init__(self, repo: OrderRepository):
        self.repo = repo

    def apply_discount(self, order_id, percentage):
        order = self.repo.find_by_id(order_id)
        if order.status != "pending":
            raise ValueError("Only a pending order can be discounted")
        order.total *= (1 - percentage / 100)
        self.repo.save(order)
        return order

# Controller: HTTP only — parses the request, calls the service, builds the response
@app.post("/orders/{order_id}/discount")
async def apply_discount(order_id: int, body: DiscountBody):
    order = order_service.apply_discount(order_id, body.percentage)
    return {"id": order.id, "total": order.total}
```

## ORM (Object-Relational Mapping)

Maps rows from a table to objects in the language — instead of writing raw SQL and parsing the result, the code works with class instances, and the ORM translates that to SQL behind the scenes. It's exactly what the `Repository` above is using (`db.query(Order)...`).

```python
# ❌ without an ORM: raw SQL + manual mapping of the result to objects
cursor.execute("SELECT * FROM orders WHERE status = %s", ("pending",))
rows = cursor.fetchall()
orders = [Order(id=r[0], status=r[1], total=r[2]) for r in rows]

# ✅ with an ORM: the library does the mapping, the result is already objects
orders = session.query(Order).filter_by(status="pending").all()
```

**ODBC (Open Database Connectivity) — the lower-level layer, and legacy**: an ORM connects to the DB through some driver — in Python, typically a native one (`psycopg2`, `asyncpg`), not ODBC. ODBC is an older, lower-level protocol, meant for **any** application (not just one written in a specific language) to connect to **any** DB with an ODBC driver installed. Today it shows up mostly in BI/reporting tools (Excel or Tableau connecting to a data warehouse — see [OLAP cubes](../diagnostics/database.md#olap-cubes)), not in a modern backend's typical stack.

## DTO (Data Transfer Object)

A simple object with no business logic, that only carries data between layers or between systems — no methods with behavior, just fields. In the Controller above, `DiscountBody` (what comes in) and the response `dict` (what goes out) are DTOs: **they decouple the API's external contract from the internal domain model** the Service uses.

Why it matters:
- Changing the DB schema (adding an internal column, renaming a field) shouldn't break the API contract if there's a DTO in the middle translating.
- The output DTO controls exactly what gets exposed — an internal field (`password_hash`, `internal_risk_score`) never leaks just because the domain model has it.
- The same domain can have different DTOs for different use cases (a lightweight summary for a list, a full one for the detail view) without duplicating the domain model itself.

```python
# Domain model: has everything the business needs, including internal fields
class Order:
    def __init__(self, id, total, status, internal_risk_score, user_id):
        ...

# Output DTO: only what the API client needs to see
class OrderDTO(BaseModel):
    id: int
    total: float
    status: str
    # internal_risk_score NEVER shows up here — the DTO decides what's exposed

def order_to_dto(order: Order) -> OrderDTO:
    return OrderDTO(id=order.id, total=order.total, status=order.status)
```

You were already using this without the name: the `response_model` and the body model in [Endpoints for microservices](../stacks/fastapi/endpoints-microservicios.md#1-anatomía-de-un-endpoint-en-fastapi) are output and input DTOs respectively — a Pydantic model that separates the API's shape from the internal domain model.

## Why separate them

Each layer has a single reason to change — it's [Single Responsibility](../system-design/solid.md#s--single-responsibility-principle) applied to a full request's architecture: a change in the API format only touches the Controller, a change in the business rule only touches the Service, a change from Postgres to Mongo only touches the Repository.

## The real benefit: testing without HTTP or a DB

The Service receives the Repository as a dependency instead of creating it itself — [Dependency Inversion](../system-design/solid.md#d--dependency-inversion-principle) — so in a test you can pass it a fake in-memory Repository instead of the real one, and test the business logic without spinning up an HTTP server or a database.

```python
class FakeOrderRepository:
    def __init__(self, orders): self.orders = orders
    def find_by_id(self, order_id): return self.orders[order_id]
    def save(self, order): pass  # nothing needs to persist for the test

def test_apply_discount_rejects_non_pending_order():
    fake_repo = FakeOrderRepository({1: Order(id=1, status="shipped", total=100)})
    service = OrderService(fake_repo)
    with pytest.raises(ValueError):
        service.apply_discount(1, 10)
```

---
Related: [Clean Architecture](../system-design/clean-architecture.md) (the full theory behind this layered separation), [SOLID principles](../system-design/solid.md), [Testing — general concepts](../system-design/testing.md#2-test-doubles--mock-vs-stub-vs-fake-vs-spy) (the `FakeOrderRepository` above is a Fake, not a Mock), [Endpoints for microservices](../stacks/fastapi/endpoints-microservicios.md) (`response_model` and Pydantic as DTOs in practice).
