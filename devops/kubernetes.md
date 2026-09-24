# Kubernetes — readiness probes, HPA, resource limits

Three basic K8s pieces that determine whether a cluster scales well or poorly.

## Readiness probe

A periodic check that tells K8s whether a pod is ready to receive traffic. Without this, a pod that just started (still initializing connections, loading config) receives requests anyway and fails them — the readiness probe keeps it "out of rotation" until it responds OK.

## HPA (Horizontal Pod Autoscaler)

Adds or removes pod replicas automatically based on actual resource usage (CPU, memory, or a custom metric). Without HPA, scaling means someone has to notice the load and change the replica count by hand.

## Resource limits

`requests` is what the pod asks to have reserved (K8s won't schedule it on a node that can't guarantee it); `limits` is the ceiling — if the pod exceeds it, it gets throttled (CPU) or killed by OOM (memory). Without limits, a pod with a memory leak can consume all of a node's memory and take down its neighbors.

```yaml
readinessProbe:
  httpGet: { path: /health, port: 3000 }
  periodSeconds: 10
resources:
  requests: { cpu: "250m", memory: "256Mi" }
  limits: { cpu: "500m", memory: "512Mi" }
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 70 } }
```

The three pieces work together: HPA decides how many pods are needed, resource limits define how much each one can consume, and the readiness probe keeps a pod the HPA just created from receiving traffic before it's ready.

---
Related: [Availability](../system-design/atributos-de-calidad.md#disponibilidad), [Elasticity](../system-design/atributos-de-calidad.md#elasticidad), [DevOps Diagnostics](../diagnostics/devops.md).
