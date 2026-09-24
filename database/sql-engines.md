# SQL Engines

Quick overview of what's used today to build a backend — databases, ORMs, and frameworks are three separate decisions (often mixed together in conversation) that don't necessarily go together: FastAPI doesn't come with an ORM included, Django does.

| Name | Type | Language/Platform | Typical use |
|---|---|---|---|
| **PostgreSQL** | DB — Relational | — | Reasonable default for almost any app — supports JSON, full-text search, extensions |
| **MySQL** | DB — Relational | — | Widely installed in legacy web (WordPress, PHP), simple to administer |
| **SQLite** | DB — Relational | — | A single file, no server — small apps, mobile, prototypes, Django's dev default |
| **Mongo** | DB — NoSQL (Document) | — | Flexible schema (JSON/BSON), fast iteration without formal migrations |
| **Redis** | DB — NoSQL (Key-Value / in-memory) | — | Cache, rate limiting, simple queues, Pub/Sub — doesn't replace a primary DB |
| **Dynamo** | DB — NoSQL (Key-Value / Document) | Managed by AWS | Access patterns known in advance, high scale without managing servers |
| **Pinecone** | DB — NoSQL (Vector) | Managed | RAG, semantic search — embeddings + similarity search (ANN) |
| **SQLAlchemy** | ORM (Data Mapper) | Python | Separates the object from how it's persisted — flexible but more verbose |
| **Django ORM** | ORM (Active Record) | Python | The model knows how to persist itself (`model.save()`), integrated into Django, not used standalone |
| **Prisma** | ORM | Node/TypeScript | Generates a typed client from a declarative schema — strong end-to-end typing |
| **TypeORM** | ORM (Active Record + Data Mapper) | Node/TypeScript | Supports both styles, similar in spirit to SQLAlchemy |
| **Mongoose** | ORM (ODM, Mongo-specific) | Node/JavaScript | Adds schemas/validation to a schemaless DB (Mongo) |
| **Django** | Framework | Python | "Batteries included" — ORM, admin panel, auth, all integrated |
| **FastAPI** | Framework | Python | Async-first, minimalist — ORM/auth chosen separately |
| **Flask** | Framework | Python | Micro-framework, everything added separately |
| **Express** | Framework | Node.js | Minimalist, assembled by hand |
| **NestJS** | Framework | Node/TypeScript | Opinionated architecture (Angular-inspired), strong TypeORM/Prisma integration |
| **Spring Boot** | Framework | Java/Kotlin | Its own ecosystem (Spring Data), heavily used in large companies |
| **Ruby on Rails** | Framework | Ruby | "Convention over configuration," ORM (ActiveRecord) included |

**FastAPI vs Django**, the most common comparison in the Python world: Django is "all included" (ORM, admin, auth, forms) meant to get started fast with strong conventions; FastAPI is minimalist and async-first, meant for APIs — assembled by picking each piece (ORM, auth, etc.) separately. Neither fully replaces the other: Django is still common in apps with a heavy admin panel, FastAPI dominates in pure APIs and microservices where async performance matters.

---
See [NoSQL](nosql.md) and [ORM (Object-Relational Mapping)](../backend/controller-service-repository.md#orm-object-relational-mapping) for the details of each. To separate runtime from framework from ORM from frontend (why "Node vs FastAPI" isn't an even comparison), see [Overview: layers of a stack](../stacks/mapa-de-stacks.md).
