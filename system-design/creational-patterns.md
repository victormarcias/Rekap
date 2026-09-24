# Design Patterns — Creational

Patterns that abstract away object creation logic, so client code isn't coupled to specific concrete classes.

## Factory Method

Defines a way to create an object, but lets a function (or subclass) decide which concrete class to instantiate. Useful when the exact type to create depends on something only known at runtime (config, data type, environment).

```python
class MySQLConnection: ...
class PostgresConnection: ...

def create_connection(driver: str):
    if driver == "mysql":
        return MySQLConnection()
    elif driver == "postgres":
        return PostgresConnection()
    raise ValueError(f"Unknown driver: {driver}")

conn = create_connection(config.db_driver)  # the rest of the code doesn't know the concrete class
```

```ts
interface Connection { query(sql: string): Promise<unknown>; }
class MySQLConnection implements Connection { async query(sql: string) { /* ... */ } }
class PostgresConnection implements Connection { async query(sql: string) { /* ... */ } }

function createConnection(driver: 'mysql' | 'postgres'): Connection {
  return driver === 'mysql' ? new MySQLConnection() : new PostgresConnection();
}
```

## Abstract Factory

A level above the Factory Method: a factory that creates entire families of related objects, guaranteeing they're compatible with each other (e.g. not accidentally mixing one cloud provider's storage with another's queue).

```ts
interface CloudFactory {
  createStorage(): Storage;
  createQueue(): Queue;
}

class AWSFactory implements CloudFactory {
  createStorage() { return new S3Storage(); }
  createQueue() { return new SQSQueue(); }
}

class GCPFactory implements CloudFactory {
  createStorage() { return new GCSStorage(); }
  createQueue() { return new PubSubQueue(); }
}

// setupInfra can never end up mixing S3 with PubSub — the factory guarantees the right family
function setupInfra(factory: CloudFactory) {
  const storage = factory.createStorage();
  const queue = factory.createQueue();
}
```

## Builder

Build a complex object step by step, separating construction from the final representation. Useful when the direct constructor would have too many optional parameters and becomes unreadable at the call site.

```python
# ❌ constructor with many optional parameters, hard to read at the call site
query = Query(table="orders", where="status='paid'", order_by="created_at", limit=10, select=["id", "total"])

# ✅ Builder: each method adds a piece, reads almost like a sentence
query = (
    QueryBuilder("orders")
    .select("id", "total")
    .where("status='paid'")
    .order_by("created_at")
    .limit(10)
    .build()
)
```

```ts
class QueryBuilder {
  private parts: string[] = [];
  constructor(private table: string) {}
  select(...cols: string[]) { this.parts.push(`SELECT ${cols.join(', ')} FROM ${this.table}`); return this; }
  where(cond: string) { this.parts.push(`WHERE ${cond}`); return this; }
  limit(n: number) { this.parts.push(`LIMIT ${n}`); return this; }
  build() { return this.parts.join(' '); }
}

const sql = new QueryBuilder('orders').select('id', 'total').where("status='paid'").limit(10).build();
```

## Singleton

Guarantees a class has a single shared instance across the whole app, with a global access point (e.g. a DB connection pool, a logger, config loaded once).

```python
class Logger:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

logger1, logger2 = Logger(), Logger()
assert logger1 is logger2  # same instance
```

```ts
class Logger {
  private static instance: Logger;
  private constructor() {}
  static getInstance(): Logger {
    if (!Logger.instance) Logger.instance = new Logger();
    return Logger.instance;
  }
}

const logger = Logger.getInstance();
```

**Careful**: it's the most criticized pattern on the list — it introduces shared global state, makes testing harder (you can't easily inject a mock in its place) and hides a dependency that should be explicit in the signature of the class using it. In practice, many frameworks solve the same problem with **Dependency Injection** instead of a manual Singleton (see [DIP in SOLID](solid.md)).

---
Related: [SOLID principles](solid.md) · full catalog at [refactoring.guru](https://refactoring.guru/design-patterns/creational-patterns).
