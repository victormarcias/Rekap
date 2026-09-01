# Panorama: capas de un stack (runtime, framework, ORM, frontend web/mobile)

## Las capas

| Capa | Qué hace | Python | JS/TypeScript (Node) | Java | Go | C# | iOS (Swift/Obj-C) | Android (Kotlin/Java) |
|---|---|---|---|---|---|---|---|---|
| **Runtime** | Ejecuta el código, punto — no decide nada de arquitectura | CPython | Node.js (V8 + libuv) / Bun / Deno | JVM | Runtime propio, compilado a binario nativo (sin VM) | .NET (CLR) | Compilado nativo (LLVM) + Objective-C runtime (mensajería dinámica, subyace hasta en apps 100% Swift) | ART (Android Runtime) — VM en el dispositivo, sucesora de Dalvik, distinta de la JVM de servidor |
| **Backend framework** | Recibe HTTP, corre lógica de negocio, devuelve datos | FastAPI, Flask, Django | Express, NestJS | Spring Boot | Gin, Echo, Fiber, `net/http` (stdlib) | ASP.NET Core | — (consume APIs, no las sirve; excepción marginal: Vapor) | — (consume APIs, no las sirve) |
| **ORM** | Traduce objetos del código ↔ filas de una DB | SQLAlchemy, Django ORM | Prisma, TypeORM, Sequelize | Hibernate / JPA | GORM, `sqlc` | Entity Framework Core | Core Data, SwiftData (persistencia local) | Room (persistencia local) |
| **Frontend framework** | Renderiza UI | — (web) | React, Vue, Svelte, Angular (web) | — | — | — | SwiftUI, UIKit (mobile) | Jetpack Compose, Views/XML (mobile) |
| **Full-stack meta-framework** | Frontend + una porción de backend en un solo proyecto | Django | Next.js, Nuxt, Remix, SvelteKit | — | — | Blazor | — | — |

## Lenguaje vs Runtime (no es lo mismo)

| Lenguaje | Runtime / cómo se ejecuta | Ámbito típico |
|---|---|---|
| C | Compilado directo a binario nativo, sin runtime/VM | Sistemas, embebidos |
| C# | **.NET (CLR)** — compilado a bytecode intermedio (IL) y luego JIT, análogo a la JVM. No es un lenguaje "script" | Web backend (ASP.NET), desktop Windows, juegos (Unity) |
| C++ | Compilado directo a binario nativo, sin runtime/VM (salvo stdlib mínima) | Sistemas, juegos, alto rendimiento |
| Go | Runtime propio embebido en el binario compilado (GC + scheduler de goroutines) — no es una VM externa | Web backend, CLIs, binarios standalone |
| Java | JVM | Web backend, Android (vía ART) |
| JavaScript | Motor del browser (V8/SpiderMonkey/JSC) o Node.js en el servidor | Web backend + frontend |
| Kotlin | JVM (Kotlin/JVM) / Kotlin Native (compilado) / Kotlin/JS | Android nativo, web backend (JVM), multiplataforma |
| Objective-C | Compilado a nativo — el "Objective-C runtime" es una librería en C para mensajería dinámica, no una VM | iOS/macOS (legacy) |
| Python | CPython (intérprete) | Web backend, scripting, data |
| Swift | Compilado a nativo (LLVM) con runtime liviano embebido (ARC) — no es una VM completa | iOS/macOS nativo (web backend existe — Vapor — pero es marginal) |
| TypeScript | Se transpila a JavaScript — corre en el mismo runtime que JS, no tiene uno propio | Web backend + frontend |

## Qué capa es cada tecnología

| Tecnología | Lenguaje | Runtime | Backend framework | ORM | Frontend framework | Full-stack meta-framework |
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

## Estilo de API (independiente del lenguaje)

| | REST | GraphQL |
|---|---|---|
| Python | FastAPI/Flask/Django nativo | Strawberry, Graphene |
| Node | Express/NestJS nativo | Apollo Server |
| Java | Spring MVC nativo | `graphql-java` |

Ver [REST](../backend/rest.md) y [GraphQL](../backend/graphql.md).

## Filosofía: "todo incluido" vs "armá tu stack"

| Ecosistema | Todo incluido | Elegís cada pieza |
|---|---|---|
| Python | Django | FastAPI, Flask |
| JS/Node | NestJS | Express |
| Java | Spring Boot | — |
| Go | — | Gin, Echo, stdlib |

---
Relacionado: [Motores de SQL](../database/motores-de-sql.md), [Node.js — runtime](node/runtime.md), [Concurrencia y Memoria (Python)](python/concurrencia-y-memoria.md), [Sync vs Async en FastAPI](../backend/fastapi/sync-vs-async.md), [Sistema de tipos comparado](tipos-comparativa.md).
