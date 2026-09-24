# SOLID Principles

Five object-oriented design principles for writing maintainable code that's easy to extend without breaking what already exists.

## S — Single Responsibility Principle

A class/module should have a single reason to change. If you mix business logic, persistence, and notifications in the same class, a change in how emails get sent forces you to touch (and potentially break) the class that calculates totals.

```python
# ❌ Order mixes calculation, persistence, and notification — 3 reasons to change
class Order:
    def calculate_total(self): ...
    def save_to_db(self): ...
    def send_confirmation_email(self): ...

# ✅ each class has a single responsibility
class Order:
    def calculate_total(self): ...

class OrderRepository:
    def save(self, order: Order): ...

class OrderNotifier:
    def send_confirmation(self, order: Order): ...
```

```ts
// ❌
class Order {
  calculateTotal() { /* ... */ }
  saveToDb() { /* ... */ }
  sendConfirmationEmail() { /* ... */ }
}

// ✅
class Order { calculateTotal() { /* ... */ } }
class OrderRepository { save(order: Order) { /* ... */ } }
class OrderNotifier { sendConfirmation(order: Order) { /* ... */ } }
```

## O — Open/Closed Principle

Code should be open to extension but closed to modification: adding a new case shouldn't require touching code that already works and is already tested. An `if/elif` that grows with every new type violates this — every addition is a chance to break the previous cases.

```python
# ❌ adding a new discount type forces you to modify this function
def calculate_discount(order, type):
    if type == "vip": return order.total * 0.2
    elif type == "regular": return order.total * 0.1

# ✅ a new discount = a new class, without touching the existing ones
class VipDiscount:
    def apply(self, total: float) -> float: return total * 0.2

class RegularDiscount:
    def apply(self, total: float) -> float: return total * 0.1
```

```ts
// ❌
function calculateDiscount(order: Order, type: string) {
  if (type === 'vip') return order.total * 0.2;
  if (type === 'regular') return order.total * 0.1;
}

// ✅
interface DiscountStrategy { apply(total: number): number; }
class VipDiscount implements DiscountStrategy { apply(total: number) { return total * 0.2; } }
class RegularDiscount implements DiscountStrategy { apply(total: number) { return total * 0.1; } }
```

## L — Liskov Substitution Principle

A subtype has to be able to replace its base type without breaking the behavior code using it expects. The classic example: `Square extends Rectangle` seems geometrically reasonable, but if `Rectangle` exposes `setWidth`/`setHeight` independently, `Square` breaks that guarantee (changing the width also changes the height) — any function that assumes `Rectangle`'s contract fails when given a `Square`.

```python
# ❌ Square breaks Rectangle's contract: setWidth also changes height
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h

class Square(Rectangle):
    def set_width(self, w): self.width = self.height = w
    def set_height(self, h): self.width = self.height = h
    # a test that assumes Rectangle: set_width(5); set_height(4) → area should be 20, Square gives 16

# ✅ don't force inheritance where the subtype doesn't fulfill the parent's contract
class Rectangle:
    def __init__(self, w, h): self.width, self.height = w, h
    def area(self): return self.width * self.height

class Square:
    def __init__(self, side): self.side = side
    def area(self): return self.side ** 2
```

```ts
// ❌
class Rectangle {
  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
}
class Square extends Rectangle {
  setWidth(w: number) { this.width = this.height = w; }
  setHeight(h: number) { this.width = this.height = h; }
}

// ✅
interface Shape { area(): number; }
class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  area() { return this.width * this.height; }
}
class Square implements Shape {
  constructor(private side: number) {}
  area() { return this.side ** 2; }
}
```

## I — Interface Segregation Principle

Don't force a client to depend on methods it doesn't use. A "fat" interface with many methods forces partial implementations to throw errors or leave methods empty — better to have several small, specific interfaces.

```python
# ❌ fat interface: a simple printer is forced to "implement" scan/fax
class MultiFunctionDevice(Protocol):
    def print(self, doc): ...
    def scan(self, doc): ...
    def fax(self, doc): ...

class SimplePrinter(MultiFunctionDevice):
    def print(self, doc): ...
    def scan(self, doc): raise NotImplementedError
    def fax(self, doc): raise NotImplementedError

# ✅ small interfaces, each client implements only what it needs
class Printer(Protocol):
    def print(self, doc): ...

class Scanner(Protocol):
    def scan(self, doc): ...

class SimplePrinter(Printer):
    def print(self, doc): ...
```

```ts
// ❌
interface MultiFunctionDevice {
  print(doc: Doc): void;
  scan(doc: Doc): void;
  fax(doc: Doc): void;
}
class SimplePrinter implements MultiFunctionDevice {
  print(doc: Doc) { /* ... */ }
  scan(doc: Doc): never { throw new Error('not supported'); }
  fax(doc: Doc): never { throw new Error('not supported'); }
}

// ✅
interface Printer { print(doc: Doc): void; }
interface Scanner { scan(doc: Doc): void; }
class SimplePrinter implements Printer { print(doc: Doc) { /* ... */ } }
```

## D — Dependency Inversion Principle

High-level modules (business logic) shouldn't depend on low-level modules (implementation details like a specific DB) — both should depend on an abstraction. This is what makes it possible to test `OrderService` with a fake repository, or switch from Postgres to Mongo without touching the business logic.

```python
# ❌ OrderService depends directly on a concrete implementation (Postgres)
class PostgresOrderRepository:
    def save(self, order): ...

class OrderService:
    def __init__(self):
        self.repo = PostgresOrderRepository()  # coupled to Postgres

# ✅ OrderService depends on an abstraction, injected from outside
class OrderRepository(Protocol):
    def save(self, order): ...

class OrderService:
    def __init__(self, repo: OrderRepository):
        self.repo = repo  # any implementation works (Postgres, Mongo, in-memory for tests)
```

```ts
// ❌
class PostgresOrderRepository { save(order: Order) { /* ... */ } }
class OrderService {
  private repo = new PostgresOrderRepository(); // coupled
}

// ✅
interface OrderRepository { save(order: Order): void; }
class OrderService {
  constructor(private repo: OrderRepository) {} // injected
}
```

## Summary

| Letter | Principle | In one sentence |
|---|---|---|
| S | Single Responsibility | One class, one reason to change |
| O | Open/Closed | Extend without modifying existing code |
| L | Liskov Substitution | A subtype shouldn't break the base type's contract |
| I | Interface Segregation | Small, specific interfaces, not one fat one |
| D | Dependency Inversion | Depend on abstractions, not concrete implementations |

Related: [Clean Architecture](clean-architecture.md) (DIP applied systematically to the whole app), [Controller / Service / Repository](../backend/controller-service-repository.md) (DIP is the basis of that separation), [Adapter pattern](structural-patterns.md#adapter).
