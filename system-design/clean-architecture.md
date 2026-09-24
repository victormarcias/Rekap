# Clean Architecture

Proposed by Robert C. Martin ("Uncle Bob"). Solves a concrete problem: an app's business logic normally lives mixed together with infrastructure details (the web framework, the database, the UI) — and those details change far more often than the business rules themselves. Clean Architecture separates the two so an infrastructure change (migrating from Postgres to Mongo, from Flask to FastAPI) doesn't force you to touch or even understand the business logic.

## The Dependency Rule

Source code can only depend **inward** — outer layers (framework, DB, UI) know about and depend on the inner ones (business rules); the inner ones **have no idea the outer ones exist**. The business logic imports nothing from FastAPI, SQLAlchemy, or React — it has no way of knowing what it's running on top of.

## The layers, from inside out

1. **Entities**: the most general business rules, independent of this particular application — concepts that would exist even if everything else changed (e.g. "a shipped order can't be cancelled," a domain rule, not one specific to this app).
2. **Use Cases**: business rules specific to this application — orchestrate the entities to accomplish something concrete (e.g. "cancel an order": find it, validate it can be cancelled, save it).
3. **Interface Adapters**: translate data between the format the use cases work with and the format the outer layers need — controllers (receive the HTTP request and call the use case), presenters, gateways.
4. **Frameworks & Drivers**: the outermost, most replaceable layer — the web framework, the database driver, the UI. This is where it's "known" that Postgres or FastAPI exists.

## Example — Dependency Inversion in practice

```python
# --- Entities: imports nothing from outside, doesn't even know a DB exists ---
class Order:
    def __init__(self, id, total, status):
        self.id = id
        self.total = total
        self.status = status

    def cancel(self):
        if self.status == "shipped":
            raise ValueError("Can't cancel an order that's already shipped")
        self.status = "cancelled"

# --- Use Case: defines WHAT it needs (an interface), not HOW it's implemented ---
from typing import Protocol

class OrderRepository(Protocol):
    def get(self, order_id: str) -> Order: ...
    def save(self, order: Order) -> None: ...

class CancelOrderUseCase:
    def __init__(self, repo: OrderRepository):
        self.repo = repo  # depends on the ABSTRACTION, not Postgres/Mongo/whatever

    def execute(self, order_id: str):
        order = self.repo.get(order_id)
        order.cancel()
        self.repo.save(order)

# --- Frameworks & Drivers: this is where it's KNOWN that Postgres exists ---
class PostgresOrderRepository:  # implements the OrderRepository "contract"
    def get(self, order_id):
        ...  # real query to Postgres
    def save(self, order):
        ...  # real UPDATE to Postgres
```

`CancelOrderUseCase` never imports `psycopg2` or `sqlalchemy` — it only knows the `OrderRepository` contract. Two direct consequences: it can be tested with an in-memory `FakeOrderRepository` without spinning up a real database, and you can migrate from Postgres to Mongo by writing a new adapter, without touching a single line of the business rule.

## Why it matters

- **Testable without real infrastructure**: use case tests run in milliseconds, with no DB or network — the equivalent of the [Fake from test doubles](../system-design/testing.md#2-test-doubles--mock-vs-stub-vs-fake-vs-spy).
- **Independent of framework and DB**: switching an external tool shouldn't force you to rewrite the business logic.
- **The domain reads on its own**: someone new to the team can understand the business rules by reading `Order`/`CancelOrderUseCase`, without having to understand FastAPI or the ORM first.

## Relationship with Hexagonal Architecture (Ports & Adapters)

Same spirit, different terminology — in practice they're used almost as synonyms. Hexagonal talks about **ports** (the interfaces, like `OrderRepository` above) and **adapters** (the concrete implementations, like `PostgresOrderRepository`); Clean Architecture talks about concentric layers. The underlying mechanism is the same: the domain defines the contract, the infrastructure implements it, never the other way around.

**Plugin Architecture (Microkernel Architecture)** is a sibling pattern — same mechanism (a core that defines a contract, and external modules that implement it to "plug in"), but with a different emphasis: Hexagonal isolates the domain from **technical infrastructure** ("I can switch from Postgres to Mongo without touching the business"); Plugin/Microkernel extends a minimal core with **optional features** ("I can add or remove a module without touching the core") — the typical example is an IDE's extensions or a CMS's plugins.

The **Dependency Inversion Principle** (the D in SOLID) is literally the mechanism that makes the Dependency Rule possible — Clean Architecture is, to a large extent, DIP applied systematically to the whole app. [Controller / Service / Repository](../backend/controller-service-repository.md) is a simpler, more pragmatic version of the same spirit — many teams use it without fully implementing all 4 layers. It works as a middle ground: less ceremony, most of the benefit.

## Trade-off

It's not free: it adds indirection (interfaces, data mapping between layers) that can be over-engineering for a simple CRUD — more files, more jumps to follow the flow of a single operation. It's worth it when the business logic is complex and will live for a long time, or when you genuinely expect to change infrastructure in the future. For a prototype, a script, or a very simple service, the cost of the indirection usually outweighs the benefit.

---
Related: [SOLID principles](solid.md), [Controller / Service / Repository](../backend/controller-service-repository.md), [Structural patterns](structural-patterns.md#adapter).
