# Load Balancers

Distribute incoming traffic across several instances of a service — the piece that makes [horizontal scalability](../system-design/quality-attributes.md#scalability) possible. In the cloud it's usually a managed service from the provider (AWS, GCP, Azure); the software you run yourself in front of an app is a [reverse proxy](reverse-proxy.md).

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
- **Sticky sessions (IP hash)**: the same client IP always goes to the same instance — useful if there's server-side in-memory state that can't be shared (ideally, see [Scalability](../system-design/quality-attributes.md#scalability), you don't need this at all).

## Managed load balancers (cloud)

In the cloud the load balancer is a **managed service**: the provider runs it, scales it, and keeps it highly available — nothing to install or patch. Each provider offers one L4 and one L7 variant:

| Provider | L4 | L7 | Notes |
|---|---|---|---|
| **AWS** | **NLB** (Network Load Balancer) | **ALB** (Application Load Balancer) | Both under *Elastic Load Balancing (ELB)*. NLB: maximum throughput, minimum latency. ALB: path/header routing, integrates with auth |
| **GCP** | **Network Load Balancer** | **Application Load Balancer** | Both under *Cloud Load Balancing*. Can be global: one IP in front of backends in several regions |
| **Azure** | **Azure Load Balancer** | **Application Gateway** | Application Gateway also adds a WAF (web application firewall) |

In Kubernetes, a `Service` of type `LoadBalancer` makes the cloud provision one of these automatically:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: LoadBalancer   # the cloud creates the load balancer and gives it a public IP
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8000
```

Software like Nginx, HAProxy, Traefik, or Envoy can also balance, but they are **reverse proxies** you run yourself — see [Reverse Proxy](reverse-proxy.md).
---
Related: [Reverse Proxy](reverse-proxy.md), [API Gateway](api-gateway.md), [Kubernetes](../devops/kubernetes.md), [Scalability](../system-design/quality-attributes.md#scalability), [AWS](../cloud/aws/core-services.md), [GCP](../cloud/gcp/core-services.md), [Azure](../cloud/azure/core-services.md).
