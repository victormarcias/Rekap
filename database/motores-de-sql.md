# Motores de SQL

Panorama rápido de qué se usa hoy para armar un backend — bases de datos, ORMs y frameworks son tres decisiones separadas (a veces se mezclan al hablar) que no van necesariamente juntas: FastAPI no trae ORM incluido, Django sí.

| Nombre | Tipo | Lenguaje/Plataforma | Uso típico |
|---|---|---|---|
| **PostgreSQL** | DB — Relacional | — | Default razonable para casi cualquier app — soporta JSON, full-text search, extensiones |
| **MySQL** | DB — Relacional | — | Muy instalado en web legacy (WordPress, PHP), simple de administrar |
| **SQLite** | DB — Relacional | — | Un solo archivo, sin servidor — apps chicas, mobile, prototipos, default de Django en dev |
| **Mongo** | DB — NoSQL (Document) | — | Esquema flexible (JSON/BSON), iteración rápida sin migraciones formales |
| **Redis** | DB — NoSQL (Key-Value / in-memory) | — | Cache, rate limiting, colas simples, Pub/Sub — no reemplaza a una DB principal |
| **Dynamo** | DB — NoSQL (Key-Value / Document) | Gestionado por AWS | Patrones de acceso conocidos de antemano, alta escala sin administrar servidores |
| **Pinecone** | DB — NoSQL (Vector) | Gestionado | RAG, búsqueda semántica — embeddings + búsqueda por similitud (ANN) |
| **SQLAlchemy** | ORM (Data Mapper) | Python | Separa el objeto de cómo se persiste — flexible pero más verboso |
| **Django ORM** | ORM (Active Record) | Python | El modelo se sabe persistir a sí mismo (`modelo.save()`), integrado a Django, no se usa suelto |
| **Prisma** | ORM | Node/TypeScript | Genera un cliente tipado a partir de un schema declarativo — fuerte tipado end-to-end |
| **TypeORM** | ORM (Active Record + Data Mapper) | Node/TypeScript | Soporta ambos estilos, similar en espíritu a SQLAlchemy |
| **Mongoose** | ORM (ODM, específico de Mongo) | Node/JavaScript | Le agrega schemas/validación a una DB sin schema (Mongo) |
| **Django** | Framework | Python | "Batteries included" — ORM, admin panel, auth, todo integrado |
| **FastAPI** | Framework | Python | Async-first, minimalista — se elige ORM/auth aparte |
| **Flask** | Framework | Python | Micro-framework, todo se agrega por separado |
| **Express** | Framework | Node.js | Minimalista, se arma a mano |
| **NestJS** | Framework | Node/TypeScript | Arquitectura opinionada (inspirada en Angular), fuerte integración con TypeORM/Prisma |
| **Spring Boot** | Framework | Java/Kotlin | Ecosistema propio (Spring Data), muy usado en empresas grandes |
| **Ruby on Rails** | Framework | Ruby | "Convention over configuration", ORM (ActiveRecord) incluido |

**FastAPI vs Django**, la comparación más común en el mundo Python: Django es "todo incluido" (ORM, admin, auth, forms) pensado para arrancar rápido con convenciones fuertes; FastAPI es minimalista y async-first, pensado para APIs — se arma eligiendo cada pieza (ORM, auth, etc.) por separado. Ninguno reemplaza al otro por completo: Django sigue siendo común en apps con panel de administración pesado, FastAPI domina en APIs puras y microservicios donde importa el rendimiento async.

---
Ver [NoSQL](nosql.md) y [ORM (Object-Relational Mapping)](../backend/controller-service-repository.md#orm-object-relational-mapping) para el detalle de cada uno. Para separar runtime de framework de ORM de frontend (por qué "Node vs FastAPI" no es una comparación pareja), ver [Panorama: capas de un stack](../stacks/mapa-de-stacks.md).
