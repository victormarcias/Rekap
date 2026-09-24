# NoSQL

Databases that step away from the classic relational model (tables + SQL + strict ACID) to prioritize horizontal scalability, flexible schemas, or specific data models.

## Main categories

| Type | Model | Examples | Typical use case |
|---|---|---|---|
| **Key-Value** | Key → opaque value | Redis, DynamoDB | Cache, sessions, counters, feature flags |
| **Document** | Nested JSON/BSON documents | MongoDB, Couchbase | Semi-structured data, schemas that change often |
| **Column-family** | Rows with dynamic columns grouped by family | Cassandra, HBase | Mass writes, time series, multi-region high availability |
| **Graph** | Nodes + relationships (edges) | Neo4j, Amazon Neptune | Highly relational data (social networks, recommendations, fraud) |
| **Vector** | Embedding (numeric vector) → similarity search | Pinecone, Weaviate, pgvector | RAG, semantic search, similarity-based recommendations |

## Why NoSQL: the central trade-off

Relational prioritizes **consistency and integrity** (constraints, foreign keys, multi-table ACID transactions). NoSQL generally prioritizes **availability and horizontal scalability**, sacrificing some consistency or query expressiveness.

## CAP Theorem

In a distributed system, facing a network partition (**P**, inevitable in practice), you have to choose between:

- **C**onsistency: every node sees the same data at the same time.
- **A**vailability: the system keeps responding even if some nodes can't communicate with each other.

### The three combinations

- **CP** (Consistency + Partition tolerance): facing a partition, the system prioritizes correctness — nodes that can't confirm consistency with the rest **stop responding** (or reject the operation) instead of risking returning something stale. Example: MongoDB in its default configuration — if a node can't reach a majority of the replica set, it rejects the write instead of blindly applying it.
- **AP** (Availability + Partition tolerance): facing a partition, the system **always keeps responding**, even if that means returning data that hasn't synced with the rest of the nodes yet — eventual consistency (see below). Example: Cassandra, DynamoDB.
- **CA** (Consistency + Availability): only possible if **there's never a partition** — in practice, that means a single node, because there are no "other nodes" to desync from. A single-server Postgres/MySQL falls here; as soon as multi-node replication is added, P is back in play and you have to choose between C and A like any distributed system.

| System | Prioritizes |
|---|---|
| Postgres/MySQL (single-node) | CA (P doesn't apply on a single node) |
| MongoDB (default config) | CP |
| Cassandra, DynamoDB | AP (eventual consistency) |

The theorem says that, facing a network partition, a distributed system can only guarantee **one** of the two — Consistency or Availability, never both at the same time — hence the name **CAP** (Consistency, Availability, Partition tolerance). P isn't something you "choose": in a truly distributed system, network partitions will happen sooner or later — the real choice is between C and A, and CA only exists as the special case of "not actually distributed."

## Eventual consistency

In AP systems (e.g. Cassandra), a write may not be visible on every node immediately — it eventually converges, but a read right after a write can return stale data. Acceptable for things like like counters; not acceptable for a bank account balance.

## Document — example (MongoDB)

```js
db.users.insertOne({
  name: "Ana",
  emails: ["ana@mail.com", "ana@work.com"], // array, no need for a separate table
  address: { city: "Buenos Aires", zip: "1000" } // nested, no JOIN
});
```

Advantage: no need to define the schema ahead of time, and nested/related data goes in a single document (fewer joins). Disadvantage: data duplication, harder to guarantee referential integrity.

## Vector — fundamental characteristics

Unlike the other categories, here you don't search by exact key or structured filter, but by **similarity**: each piece of data is represented as an embedding (a numeric vector capturing its meaning), and the query is "which of these vectors are closest to my query's vector?". Solving this by brute force against millions of vectors doesn't scale — these databases use **ANN** (Approximate Nearest Neighbor, e.g. HNSW) indexes that trade some exact precision for millisecond responses. Most also support **metadata filtering**: combining the similarity search with exact filters (e.g. "only documents from this user").

It's the infrastructure piece behind RAG — see [RAG (Retrieval-Augmented Generation)](../agentic-ai/rag.md#3-vector-db--storing-and-searching-by-similarity) for the full pipeline (chunking → embeddings → vector DB → retrieval).

## When to choose NoSQL vs SQL

**NoSQL when:**
- The schema changes frequently or is very heterogeneous across records.
- You need to scale writes horizontally across many nodes/regions.
- Data access is simple (by key) and you don't need complex joins or multi-entity transactions.

**SQL when:**
- You need strong referential integrity and ACID transactions across multiple entities (e.g. financial systems).
- Relationships between entities are the heart of the domain and are queried with complex queries (joins, aggregations).
- The team already has expertise and mature tooling in the SQL ecosystem.

In practice, many systems use **both** (polyglot persistence): Postgres for the transactional domain, Redis for cache/sessions, Elasticsearch for full-text search.

Related: [Sharding vs partitioning](sharding-vs-partitioning.md), [ACID / transactions / isolation levels](acid.md).
