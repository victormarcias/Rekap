# Load Balancers

Distribute incoming traffic across several instances of a service — the piece that makes [horizontal scalability](../system-design/atributos-de-calidad.md#escalabilidad) possible.

## L4 vs L7

- **L4 (transport layer)**: decides where to send traffic by looking only at IP and port — doesn't open the packet's contents. Faster (less work per request), but can't route based on the request's content.
- **L7 (application layer)**: understands the application protocol (HTTP) — can route by path (`/api` to one service, `/admin` to another), header, or cookie. Slower than L4 (has to parse the request), but much more flexible.

```
# L4: decides only by IP:port — doesn't know what path the client is requesting
443 → instance A or B (round robin, without looking at the request)

# L7: can route by content
/api/*    → backend cluster
/static/* → static assets cluster
```

## Load balancing algorithms

- **Round robin**: distributes in circular order, one by one. Simple, works well if all instances have similar capacity.
- **Least connections**: sends the next request to the instance with the fewest active connections right now — better when requests have highly variable duration (an instance with slow requests doesn't keep getting more traffic just because "it was its turn").
- **Sticky sessions (IP hash)**: the same client IP always goes to the same instance — useful if there's server-side in-memory state that can't be shared (ideally, see [Scalability](../system-design/atributos-de-calidad.md#escalabilidad), you don't need this at all).

## Who's who

| Name | Layer | Notes |
|---|---|---|
| **NLB** (AWS Network Load Balancer) | L4 | Maximum throughput, minimum latency |
| **ALB** (AWS Application Load Balancer) | L7 | Path/header routing, integrates with auth |
| **Nginx** | L7 (also works as L4) | The most used as its own reverse proxy — see [Deploy to a VPS](../devops/deploy-vps.md) |
| **HAProxy** | L4 and L7 | Specialized in load balancing, widely used in front of databases too |
| **Traefik** | L7 | Integrates natively with Docker/K8s, auto-detects new services |
| **Envoy** | L7 | Data proxy used as the foundation of service meshes (Istio) |
| **K8s Ingress** | L7 | The standard way to expose an K8s cluster's HTTP services externally |

---
Related: [Kubernetes](../devops/kubernetes.md), [Scalability](../system-design/atributos-de-calidad.md#escalabilidad), [Deploy to a VPS](../devops/deploy-vps.md) (Nginx as reverse proxy).
