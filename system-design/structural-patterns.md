# Design Patterns — Structural

Patterns that define how to compose classes and objects into larger structures, without leaking internal complexity into client code.

## Adapter

Converts one class's interface into another interface the client expects, letting classes with incompatible interfaces work together without modifying either one.

**When to use it**: you're integrating a third-party library whose interface doesn't match what your code expects and you can't modify it; you're migrating from one provider to another (e.g. Stripe → MercadoPago) and want the rest of the app to keep talking to the same internal interface; or legacy code with an old interface needs to coexist with new code.

```python
# External SDK, with its own interface — we don't control it
class StripeSDK:
    def create_charge(self, amount_cents: int, currency: str, token: str) -> dict: ...

# Interface the rest of the app expects
class PaymentProcessor(Protocol):
    def pay(self, amount: float, card_token: str) -> bool: ...

# ✅ Adapter: translates Stripe's interface to the internal interface
class StripeAdapter:
    def __init__(self, stripe_sdk: StripeSDK):
        self._sdk = stripe_sdk

    def pay(self, amount: float, card_token: str) -> bool:
        result = self._sdk.create_charge(amount_cents=int(amount * 100), currency="usd", token=card_token)
        return result["status"] == "succeeded"

# the rest of the app only knows PaymentProcessor, never StripeSDK directly
def checkout(processor: PaymentProcessor, total: float, token: str):
    if processor.pay(total, token):
        print("Payment approved")
```

```ts
class StripeSDK {
  createCharge(amountCents: number, currency: string, token: string) {
    return { status: 'succeeded' }; // calls Stripe's real API
  }
}

interface PaymentProcessor { pay(amount: number, cardToken: string): boolean; }

// ✅ Adapter: translates Stripe's interface to the internal interface
class StripeAdapter implements PaymentProcessor {
  constructor(private sdk: StripeSDK) {}
  pay(amount: number, cardToken: string): boolean {
    const result = this.sdk.createCharge(Math.round(amount * 100), 'usd', cardToken);
    return result.status === 'succeeded';
  }
}
```

The benefit shows when you switch providers: tomorrow a `MercadoPagoAdapter implements PaymentProcessor` shows up with its own internal translation, and the code calling `checkout()` doesn't change a single line. **Object Adapter** (composes the adapted object, as above) is the most common in Python/TS/JS; **Class Adapter** (inherits from the adapted class) requires multiple inheritance, possible in Python but not in TypeScript.

## Decorator

Adds behavior to an object at runtime by wrapping it, without modifying its class or its siblings'. Express/Koa middlewares are, in essence, decorators chained over the request/response.

```python
import time

def with_logging(fn):
    def wrapper(*args, **kwargs):
        print(f"Calling {fn.__name__}")
        return fn(*args, **kwargs)
    return wrapper

def with_timing(fn):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = fn(*args, **kwargs)
        print(f"{fn.__name__} took {time.time() - start:.3f}s")
        return result
    return wrapper

@with_logging
@with_timing
def process_order(order_id): ...
# each decorator adds behavior without touching process_order's code
```

```ts
// Express: each middleware is a decorator wrapping the next link in the chain
app.use((req, res, next) => { console.log(`${req.method} ${req.url}`); next(); }); // logging
app.use((req, res, next) => { req.startTime = Date.now(); next(); }); // timing
app.get('/orders/:id', getOrderHandler); // the real handler doesn't know it's being wrapped
```

## Facade

A simple interface over a complex subsystem, so the client doesn't have to know or orchestrate all the internal pieces.

```python
# ❌ the client code knows and orchestrates 3 internal services
def checkout(cart, user, card):
    inventory_service.reserve(cart.items)
    payment_service.charge(user, card, cart.total)
    shipping_service.schedule(user.address, cart.items)

# ✅ Facade: the client only knows one simple interface
class CheckoutFacade:
    def complete(self, cart, user, card):
        inventory_service.reserve(cart.items)
        payment_service.charge(user, card, cart.total)
        shipping_service.schedule(user.address, cart.items)

CheckoutFacade().complete(cart, user, card)
```

```ts
class CheckoutFacade {
  complete(cart: Cart, user: User, card: Card) {
    inventoryService.reserve(cart.items);
    paymentService.charge(user, card, cart.total);
    shippingService.schedule(user.address, cart.items);
  }
}

new CheckoutFacade().complete(cart, user, card);
```

## Proxy

An object that controls access to another, intercepting calls to add extra logic (lazy loading, cache, permission checks) without the client noticing — implements the same interface as the real object.

```ts
interface UserRepository { findById(id: string): Promise<User>; }

class RealUserRepository implements UserRepository {
  async findById(id: string) { return db.users.findById(id); }
}

// ✅ Proxy: adds a cache in front of the real repository, transparent to the client
class CachedUserRepository implements UserRepository {
  private cache = new Map<string, User>();
  constructor(private real: UserRepository) {}

  async findById(id: string) {
    if (this.cache.has(id)) return this.cache.get(id)!;
    const user = await this.real.findById(id);
    this.cache.set(id, user);
    return user;
  }
}

const repo: UserRepository = new CachedUserRepository(new RealUserRepository());
```

## iOS Analogy

Adapter and Proxy play the same role a Swift `protocol` wrapping a third-party SDK does, or the old Objective-C ↔ Swift bridging: your code talks against the protocol you defined, and the wrapper translates to/from the external library.

---
Related: [SOLID principles](solid.md) (Adapter and Proxy are direct applications of Dependency Inversion) · full catalog at [refactoring.guru](https://refactoring.guru/design-patterns/structural-patterns).
