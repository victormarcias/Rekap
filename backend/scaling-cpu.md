# CPU Scalability

When the bottleneck is compute, not waiting — the [cause and diagnosis](../diagnostics/backend.md#cpu-bound) are already in `diagnostics/backend.md`. Here the focus is the scalability question: once it's identified that the problem is CPU, what do you do?

## Vertical: more cores, same process

Works if the work is already well parallelized (uses multiple threads/workers within the process) — more available cores means more real parallelism. If the work runs on a single thread with no parallelization, adding cores doesn't help: that thread stays limited to one core no matter how many are free next to it.

## Horizontal: more processes

Once it no longer makes sense to keep adding cores to a single machine (or the work doesn't parallelize well within a process), the alternative is running **more processes/instances** in parallel, each using its own set of cores — see [Clustering](../diagnostics/backend.md#clustering) for the pattern within a single machine, and [Process Scalability](../devops/scaling-processes.md) for taking it to multiple machines.

## Before scaling: is it really insufficient CPU?

Scaling (vertical or horizontal) makes sense once the algorithm is already reasonably optimized — adding cores to an `O(n²)` algorithm that should be `O(n log n)` is paying for infrastructure to paper over a code problem (see [Unoptimized algorithms](../diagnostics/backend.md#unoptimized-algorithms)). Confirm with a profiler which function is consuming the time before deciding to scale.

---
Related: [Backend Diagnostics](../diagnostics/backend.md#cpu-bound), [Vertical vs Horizontal Scalability](../devops/scaling-vertical-vs-horizontal.md), [Process Scalability](../devops/scaling-processes.md).
