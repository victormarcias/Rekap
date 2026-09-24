# Memory Scalability

RAM capacity at the infrastructure level — different from a memory leak at the code level, already covered in [Backend Diagnostics](../diagnostics/backend.md#memory). Here the problem isn't that the process loses memory over time, it's that **legitimate** workload already needs more RAM than is available.

## OOMKilled

In Kubernetes, if a pod exceeds the memory `limit` set in its [resource limits](kubernetes.md#resource-limits), the kernel kills it immediately (`OOMKilled`) — there's no gradual degradation, it's an abrupt cutoff. Seeing that `OOMKilled` in the pod logs is a sign the limit is too small for the real load, not necessarily a leak.

```yaml
resources:
  limits: { memory: "512Mi" }  # if the process needs more than this, K8s kills it, doesn't throttle it
```

## When to scale memory

- **Vertical**: bump the instance size/the pod's `limit`, when memory usage is genuinely proportional to load (more concurrent users, larger datasets in memory).
- **In-memory caches that grow unbounded**: a cache with no eviction policy (no TTL, no size limit) eventually consumes all available memory — it's not a technical leak, but the symptom is the same. An external store (e.g. Redis) with eviction policies fixes this, instead of caching everything in the process's own memory.
- **Swap**: when physical RAM runs out, the operating system can use disk as virtual memory — it works, but disk access is orders of magnitude slower than RAM; a process actively "swapping" looks exactly like it's hung even though it's technically still alive.

---
Related: [Backend Diagnostics](../diagnostics/backend.md#memory) (memory leaks), [Kubernetes](kubernetes.md#resource-limits).
