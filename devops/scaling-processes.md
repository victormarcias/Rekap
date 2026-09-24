# Process Scalability

Adding more processes/workers in parallel — the intermediate step between "a single process" and "scaling to multiple machines." We already saw the pattern within a single machine as [Clustering](../diagnostics/backend.md#clustering); this is the same idea, taken to the full infrastructure level.

## Within a machine, first

Before adding machines, use the cores you already have: a single-threaded runtime (Node) running one process leaves the rest of the machine's cores idle. Running one process per core (`cluster` module, PM2 in cluster mode, multiple gunicorn workers) is the cheapest way to scale — there's no network involved, just separate processes on the same machine.

## Between machines, next

Once a single machine can't keep up anymore (all its cores are already in use, and capacity is still short), the next step is [scaling horizontally](scaling-vertical-vs-horizontal.md) by adding more instances/machines — each running its own set of processes — distributed by a [load balancer](../backend/load-balancers.md).

## The requirement: processes with no state of their own

Whether it's multi-process on one machine or multi-instance across machines, each process has to be able to handle any request without depending on state that only it holds in memory — the same stateless condition from [Scalability](../system-design/quality-attributes.md#scalability) that enables horizontal scaling without breaking anything. A process that stores sessions in its own memory breaks as soon as there's more than one of it handling the same traffic.

---
Related: [Clustering](../diagnostics/backend.md#clustering), [Vertical vs Horizontal Scalability](scaling-vertical-vs-horizontal.md), [Load balancers](../backend/load-balancers.md).
