# Monolith vs Microservices

Two ways of organizing a backend system's code and deploy — it's not "microservices always wins," it's a real trade-off.

## Monolith

The entire application is **one codebase, one process, one deploy**. The different parts (orders, users, payments) talk to each other with direct, in-memory function calls — no network involved.

**Pros**: simple to develop, test, and deploy at first; a DB transaction that crosses "modules" is a normal transaction (see [ACID](../database/acid.md)); no network latency between internal components.

**Cons**: scaling means scaling the **entire** process even if only one part has heavy load; a bug in one module can take down the whole app; as the team grows, everyone stepping on the same repo/deploy becomes an organizational bottleneck, not a technical one.

## Microservices

Each part of the system is an **independent service**, with its own process, its own deploy, and (ideally) its own database. They communicate over the network — HTTP, or asynchronously via queues/events (see [Kafka Architecture](kafka.md), [Message Queues](message-queues.md)).

**Pros**: each service scales independently (see [Scalability](../system-design/atributos-de-calidad.md#escalabilidad)); a team can deploy its service without coordinating with others; a bug in one service doesn't necessarily take down the others.

**Cons**: what used to be a function call is now a network call — slower, and it can fail (see [Fault tolerance](../system-design/atributos-de-calidad.md#tolerancia-a-fallos)); a transaction that crosses two services is no longer a simple DB transaction — it has to be solved with more complex patterns (sagas, eventual consistency); a lot more operational complexity (monitoring, deploys, contract versioning between services).

## The anti-pattern: distributed monolith

The worst of both worlds: services separated at deploy time, but so tightly coupled to each other (chained synchronous calls, shared DB schemas, deploys that have to be coordinated in a specific order) that in practice **they all have to be deployed together** for anything to work. You pay every cost of microservices (network latency, operational complexity) with none of the benefits (real independence).

## Practical rule

"Monolith first" is common advice: start with a well-layered monolith (see [Controller / Service / Repository](controller-service-repository.md)) and **extract** services only once a specific part genuinely needs to scale or deploy independently — don't split into microservices from day one without yet knowing where the domain's natural boundaries are.

---
Related: [Scalability](../system-design/atributos-de-calidad.md#escalabilidad), [Controller / Service / Repository](controller-service-repository.md), [API Gateway](api-gateway.md).
