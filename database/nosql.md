# NoSQL

Bases de datos que se apartan del modelo relacional clásico (tablas + SQL + ACID estricto) para priorizar escalabilidad horizontal, esquemas flexibles o modelos de datos específicos.

## Categorías principales

| Tipo | Modelo | Ejemplos | Caso de uso típico |
|---|---|---|---|
| **Key-Value** | Clave → valor opaco | Redis, DynamoDB | Cache, sesiones, contadores, feature flags |
| **Documento** | Documentos JSON/BSON anidados | MongoDB, Couchbase | Datos semi-estructurados, schemas que cambian seguido |
| **Column-family** | Filas con columnas dinámicas agrupadas por familia | Cassandra, HBase | Escritura masiva, series temporales, alta disponibilidad multi-región |
| **Grafo** | Nodos + relaciones (edges) | Neo4j, Amazon Neptune | Datos altamente relacionales (redes sociales, recomendaciones, fraude) |
| **Vector** | Embedding (vector numérico) → búsqueda por similitud | Pinecone, Weaviate, pgvector | RAG, búsqueda semántica, recomendaciones por similitud |

## Por qué NoSQL: el trade-off central

Relacional prioriza **consistencia e integridad** (constraints, foreign keys, transacciones ACID multi-tabla). NoSQL generalmente prioriza **disponibilidad y escalabilidad horizontal**, sacrificando algo de consistencia o de expresividad de queries.

## CAP Theorem

En un sistema distribuido, ante una partición de red (**P**, inevitable en la práctica), hay que elegir entre:

- **C**onsistency: todos los nodos ven el mismo dato al mismo tiempo.
- **A**vailability: el sistema sigue respondiendo aunque algunos nodos no puedan comunicarse entre sí.

### Las tres combinaciones

- **CP** (Consistency + Partition tolerance): ante una partición, el sistema prioriza que el dato sea correcto — los nodos que no pueden confirmar consistencia con el resto **dejan de responder** (o rechazan la operación) en vez de arriesgarse a devolver algo desactualizado. Ejemplo: MongoDB en su configuración default — si un nodo no puede alcanzar la mayoría del replica set, rechaza la escritura en vez de aplicarla a ciegas.
- **AP** (Availability + Partition tolerance): ante una partición, el sistema **sigue respondiendo siempre**, aunque eso signifique devolver un dato que todavía no se sincronizó con el resto de los nodos — consistencia eventual (ver abajo). Ejemplo: Cassandra, DynamoDB.
- **CA** (Consistency + Availability): solo es posible si **nunca hay una partición** — en la práctica, eso significa un solo nodo, porque no hay "otros nodos" con los que desincronizarse. Un Postgres/MySQL de un único servidor cae acá; apenas se agrega replicación multi-nodo, la P vuelve a estar en juego y hay que elegir entre C y A como cualquier sistema distribuido.

| Sistema | Prioriza |
|---|---|
| Postgres/MySQL (single-node) | CA (no aplica P en un solo nodo) |
| MongoDB (config default) | CP |
| Cassandra, DynamoDB | AP (consistencia eventual) |

El teorema dice que, ante una partición de red, un sistema distribuido solo puede garantizar **una** de las dos — Consistency o Availability, nunca ambas al mismo tiempo — de ahí el nombre **CAP** (Consistency, Availability, Partition tolerance). La P no es una opción que se "elige": en un sistema realmente distribuido, las particiones de red van a pasar tarde o temprano — la elección real es entre C y A, y CA solo existe como caso especial de "no soy realmente distribuido".

## Consistencia eventual

En sistemas AP (ej. Cassandra), un write puede no verse inmediatamente en todos los nodos — eventualmente converge, pero una lectura inmediata después de un write puede devolver el dato viejo. Aceptable para casos como contadores de likes; no aceptable para saldo de una cuenta bancaria.

## Documento — ejemplo (MongoDB)

```js
db.users.insertOne({
  name: "Ana",
  emails: ["ana@mail.com", "ana@work.com"], // array, sin necesidad de tabla aparte
  address: { city: "Buenos Aires", zip: "1000" } // anidado, sin JOIN
});
```

Ventaja: no hace falta definir el schema por adelantado, y datos anidados/relacionados van en un solo documento (menos joins). Desventaja: duplicación de datos, más difícil garantizar integridad referencial.

## Vector — características fundamentales

A diferencia de las demás categorías, acá no se busca por clave exacta ni por filtro estructurado, sino por **similitud**: cada dato se representa como un embedding (un vector numérico que captura su significado), y la consulta es "¿cuáles de estos vectores están más cerca del vector de mi pregunta?". Resolverlo por fuerza bruta contra millones de vectores no escala — estas bases usan índices **ANN** (Approximate Nearest Neighbor, ej. HNSW) que sacrifican algo de precisión exacta a cambio de responder en milisegundos. La mayoría también soporta **metadata filtering**: combinar la búsqueda por similitud con filtros exactos (ej. "solo documentos de este usuario").

Es la pieza de infraestructura detrás de RAG — ver [RAG (Retrieval-Augmented Generation)](../agentic-ai/rag.md#3-vector-db--guardar-y-buscar-por-similitud) para el pipeline completo (chunking → embeddings → vector DB → retrieval).

## Cuándo elegir NoSQL vs SQL

**NoSQL cuando:**
- El schema cambia frecuentemente o es muy heterogéneo entre registros.
- Necesitás escalar escritura horizontalmente a través de muchos nodos/regiones.
- El acceso a datos es simple (por key) y no necesitás joins complejos ni transacciones multi-entidad.

**SQL cuando:**
- Necesitás integridad referencial fuerte y transacciones ACID entre múltiples entidades (ej. sistemas financieros).
- Las relaciones entre entidades son el corazón del dominio y se consultan con queries complejas (joins, agregaciones).
- El equipo ya tiene expertise y tooling maduro en el ecosistema SQL.

En la práctica, muchos sistemas usan **ambos** (polyglot persistence): Postgres para el dominio transaccional, Redis para cache/sesiones, Elasticsearch para búsqueda full-text.

Relacionado: [Sharding vs partitioning](sharding-vs-partitioning.md), [ACID / transacciones / isolation levels](acid-transacciones-isolation.md).
