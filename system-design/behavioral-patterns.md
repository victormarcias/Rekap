# Design Patterns — Behavioral

Patterns that define how objects communicate and split responsibilities among themselves.

## Strategy

A family of interchangeable algorithms, each encapsulated in its own class, selectable at runtime. Avoids giant `if/elif` chains that grow with every new case and violate Open/Closed.

```python
class VipDiscount:
    def apply(self, total: float) -> float: return total * 0.2

class RegularDiscount:
    def apply(self, total: float) -> float: return total * 0.1

def checkout(total: float, strategy):
    return total - strategy.apply(total)

checkout(100, VipDiscount())  # the discount algorithm is chosen from outside, without touching checkout()
```

See the full example (in TypeScript) in [Open/Closed Principle](solid.md) — Strategy is the concrete implementation of that principle.

## Observer

An object (subject) keeps a list of dependents (observers) and automatically notifies them of any state change. It's the basis of event emitters, pub/sub, and how React decides to re-render when state changes.

```python
class EventEmitter:
    def __init__(self):
        self._listeners = {}

    def on(self, event: str, callback):
        self._listeners.setdefault(event, []).append(callback)

    def emit(self, event: str, *args):
        for cb in self._listeners.get(event, []):
            cb(*args)

emitter = EventEmitter()
emitter.on("order_created", lambda order: send_confirmation_email(order))
emitter.on("order_created", lambda order: update_inventory(order))
emitter.emit("order_created", order)  # both listeners fire, decoupled from each other
```

```ts
class EventEmitter {
  private listeners: Record<string, ((...args: any[]) => void)[]> = {};

  on(event: string, callback: (...args: any[]) => void) {
    (this.listeners[event] ??= []).push(callback);
  }

  emit(event: string, ...args: any[]) {
    this.listeners[event]?.forEach((cb) => cb(...args));
  }
}

const emitter = new EventEmitter();
emitter.on('order_created', (order) => sendConfirmationEmail(order));
emitter.emit('order_created', order);
```

## Command

Encapsulates an action (and its parameters) as an object, instead of executing it directly. Allows queuing, logging, retrying, or undoing that action without coupling whoever triggers it to whoever executes it.

```ts
interface Command { execute(): Promise<void>; }

class SendEmailCommand implements Command {
  constructor(private to: string, private body: string) {}
  async execute() { await emailService.send(this.to, this.body); }
}

// ✅ the queue only needs to know how to execute Commands, not know about each task type
class JobQueue {
  private queue: Command[] = [];
  push(cmd: Command) { this.queue.push(cmd); }
  async processAll() { for (const cmd of this.queue) await cmd.execute(); }
}

const queue = new JobQueue();
queue.push(new SendEmailCommand('ana@mail.com', 'Welcome'));
```

## Chain of Responsibility

Passes a request through a chain of handlers, where each one decides whether to process it, modify it, or pass it to the next one. Middlewares (Express, Django, any HTTP framework) are the most common everyday example of this pattern.

```python
class Handler:
    def __init__(self, next_handler=None):
        self.next = next_handler

    def handle(self, request):
        if self.next:
            return self.next.handle(request)

class AuthHandler(Handler):
    def handle(self, request):
        if not request.get("token"):
            raise PermissionError("Not authenticated")
        return super().handle(request)

class RateLimitHandler(Handler):
    def handle(self, request):
        if is_rate_limited(request["user_id"]):
            raise Exception("Rate limit exceeded")
        return super().handle(request)

chain = AuthHandler(RateLimitHandler())
chain.handle(request)  # goes through auth, then rate limit, then the rest of the logic
```

---
Related: [SOLID principles](solid.md) · full catalog at [refactoring.guru](https://refactoring.guru/design-patterns/behavioral-patterns).
