# Overview: layers of a stack (runtime, framework, ORM, web/mobile frontend)

## The layers

| Layer | What it does | Python | JS/TypeScript (Node) | Java | Go | C# | iOS (Swift/Obj-C) | Android (Kotlin/Java) |
|---|---|---|---|---|---|---|---|---|
| **Runtime** | Executes the code, period — decides nothing about architecture | CPython | Node.js (V8 + libuv) / Bun / Deno | JVM | Its own runtime, compiled to a native binary (no VM) | .NET (CLR) | Compiled natively (LLVM) + Objective-C runtime (dynamic messaging, underlies even 100% Swift apps) | ART (Android Runtime) — an on-device VM, Dalvik's successor, different from the server-side JVM |
| **Backend framework** | Receives HTTP, runs business logic, returns data | FastAPI, Flask, Django | Express, NestJS | Spring Boot | Gin, Echo, Fiber, `net/http` (stdlib) | ASP.NET Core | — (consumes APIs, doesn't serve them; minor exception: Vapor) | — (consumes APIs, doesn't serve them) |
| **ORM** | Translates code objects ↔ DB rows | SQLAlchemy, Django ORM | Prisma, TypeORM, Sequelize | Hibernate / JPA | GORM, `sqlc` | Entity Framework Core | Core Data, SwiftData (local persistence) | Room (local persistence) |
| **Frontend framework** | Renders UI | — (web) | React, Vue, Svelte, Angular (web) | — | — | — | SwiftUI, UIKit (mobile) | Jetpack Compose, Views/XML (mobile) |
| **Full-stack meta-framework** | Frontend + a slice of backend in one project | Django | Next.js, Nuxt, Remix, SvelteKit | — | — | Blazor | — | — |

## Language vs Runtime (not the same thing)

| Language | Runtime / how it executes | Typical domain |
|---|---|---|
| C | Compiled directly to a native binary, no runtime/VM | Systems, embedded |
| C# | **.NET (CLR)** — compiled to intermediate bytecode (IL) and then JIT'd, analogous to the JVM. Not a "script" language | Web backend (ASP.NET), Windows desktop, games (Unity) |
| C++ | Compiled directly to a native binary, no runtime/VM (except a minimal stdlib) | Systems, games, high performance |
| Go | Its own runtime embedded in the compiled binary (GC + goroutine scheduler) — not an external VM | Web backend, CLIs, standalone binaries |
| Java | JVM | Web backend, Android (via ART) |
| JavaScript | The browser's engine (V8/SpiderMonkey/JSC) or Node.js on the server | Web backend + frontend |
| Kotlin | JVM (Kotlin/JVM) / Kotlin Native (compiled) / Kotlin/JS | Native Android, web backend (JVM), multiplatform |
| Objective-C | Compiled to native — the "Objective-C runtime" is a C library for dynamic messaging, not a VM | iOS/macOS (legacy) |
| Python | CPython (interpreter) | Web backend, scripting, data |
| Swift | Compiled to native (LLVM) with a lightweight embedded runtime (ARC) — not a full VM | Native iOS/macOS (a web backend exists — Vapor — but it's marginal) |
| TypeScript | Transpiles to JavaScript — runs on the same runtime as JS, has no runtime of its own | Web backend + frontend |

## Which layer each technology is

| Technology | Language | Runtime | Backend framework | ORM | Frontend framework | Full-stack meta-framework |
|---|---|---|---|---|---|---|
| Angular | TS | ❌ | ❌ | ❌ | ✅ | ❌ |
| ART (Android Runtime) | Kotlin / Java | ✅ | ❌ | ❌ | ❌ | ❌ |
| ASP.NET Core | C# | ❌ | ✅ | ❌ | ❌ | ❌ |
| Blazor | C# | ❌ | ❌ | ❌ | ✅ | ✅ |
| Bun | JS / TS | ✅ | ❌ | ❌ | ❌ | ❌ |
| Core Data / SwiftData | Swift | ❌ | ❌ | ✅ | ❌ | ❌ |
| CPython | Python | ✅ | ❌ | ❌ | ❌ | ❌ |
| Deno | JS / TS | ✅ | ❌ | ❌ | ❌ | ❌ |
| Django | Python | ❌ | ✅ | ❌ | ❌ | ✅ |
| Django ORM | Python | ❌ | ❌ | ✅ | ❌ | ❌ |
| Entity Framework Core | C# | ❌ | ❌ | ✅ | ❌ | ❌ |
| Express | JS / TS | ❌ | ✅ | ❌ | ❌ | ❌ |
| FastAPI | Python | ❌ | ✅ | ❌ | ❌ | ❌ |
| Flask | Python | ❌ | ✅ | ❌ | ❌ | ❌ |
| Gin / Echo / Fiber | Go | ❌ | ✅ | ❌ | ❌ | ❌ |
| Go (runtime) | Go | ✅ | ❌ | ❌ | ❌ | ❌ |
| GORM | Go | ❌ | ❌ | ✅ | ❌ | ❌ |
| Hibernate / JPA | Java / Kotlin | ❌ | ❌ | ✅ | ❌ | ❌ |
| Jetpack Compose | Kotlin | ❌ | ❌ | ❌ | ✅ | ❌ |
| JVM | Java / Kotlin | ✅ | ❌ | ❌ | ❌ | ❌ |
| NestJS | TS | ❌ | ✅ | ❌ | ❌ | ❌ |
| .NET (CLR) | C# | ✅ | ❌ | ❌ | ❌ | ❌ |
| Next.js | JS / TS | ❌ | ❌ | ❌ | ✅ | ✅ |
| Node.js | JS / TS | ✅ | ❌ | ❌ | ❌ | ❌ |
| Nuxt | JS / TS | ❌ | ❌ | ❌ | ✅ | ✅ |
| Objective-C runtime | Objective-C / Swift | ✅ | ❌ | ❌ | ❌ | ❌ |
| Prisma | TS / JS | ❌ | ❌ | ✅ | ❌ | ❌ |
| React | JS / TS | ❌ | ❌ | ❌ | ✅ | ❌ |
| Remix | JS / TS | ❌ | ❌ | ❌ | ✅ | ✅ |
| Room | Kotlin / Java | ❌ | ❌ | ✅ | ❌ | ❌ |
| Sequelize | JS | ❌ | ❌ | ✅ | ❌ | ❌ |
| Spring Boot | Java / Kotlin | ❌ | ✅ | ❌ | ❌ | ❌ |
| SQLAlchemy | Python | ❌ | ❌ | ✅ | ❌ | ❌ |
| Svelte | JS / TS | ❌ | ❌ | ❌ | ✅ | ❌ |
| SvelteKit | JS / TS | ❌ | ❌ | ❌ | ✅ | ✅ |
| SwiftUI | Swift | ❌ | ❌ | ❌ | ✅ | ❌ |
| TypeORM | TS | ❌ | ❌ | ✅ | ❌ | ❌ |
| UIKit | Swift / Objective-C | ❌ | ❌ | ❌ | ✅ | ❌ |
| Views / XML (Android) | Kotlin / Java | ❌ | ❌ | ❌ | ✅ | ❌ |
| Vue | JS / TS | ❌ | ❌ | ❌ | ✅ | ❌ |

## API style (language-independent)

| | REST | GraphQL |
|---|---|---|
| Python | Native FastAPI/Flask/Django | Strawberry, Graphene |
| Node | Native Express/NestJS | Apollo Server |
| Java | Native Spring MVC | `graphql-java` |

See [REST](../backend/rest.md) and [GraphQL](../backend/graphql.md).

## Philosophy: "batteries included" vs "build your own stack"

| Ecosystem | Batteries included | You pick each piece |
|---|---|---|
| Python | Django | FastAPI, Flask |
| JS/Node | NestJS | Express |
| Java | Spring Boot | — |
| Go | — | Gin, Echo, stdlib |

---
Related: [SQL Engines](../database/sql-engines.md), [Node.js — runtime](node/runtime.md), [Concurrency and Memory (Python)](python/concurrency-and-memory.md), [Sync vs Async in FastAPI](fastapi/sync-vs-async.md), [Compared type system](type-system-comparison.md).
