# Load Balancers

Reparten el tráfico entrante entre varias instancias de un servicio — la pieza que hace posible la [escalabilidad horizontal](../system-design/quality-attributes.es.md#escalabilidad). En el cloud suele ser un servicio gestionado del proveedor (AWS, GCP, Azure); el software que corrés vos mismo delante de una app es un [reverse proxy](reverse-proxy.es.md).

## L4 vs L7

- **L4 (transport layer)**: decide a dónde mandar el tráfico mirando solo IP y puerto — no abre el contenido del paquete. Más rápido (menos trabajo por request), pero no puede rutear según el contenido del request.
- **L7 (application layer)**: entiende el protocolo de aplicación (HTTP) — puede rutear según path (`/api` a un servicio, `/admin` a otro), header, o cookie. Más lento que L4 (tiene que parsear el request), pero mucho más flexible.

```
# L4: decide solo por IP:puerto — no sabe qué path está pidiendo el cliente
443 → instancia A o B (round robin, sin mirar el request)

# L7: puede rutear por contenido
/api/*    → cluster de backend
/static/* → cluster de assets estáticos
```

## Algoritmos de balanceo

- **Round robin**: reparte en orden circular, uno por uno. Simple, funciona bien si todas las instancias tienen capacidad similar.
- **Least connections**: manda la siguiente request a la instancia con menos conexiones activas en este momento — mejor cuando las requests tienen duración muy variable (una instancia con requests lentas no sigue recibiendo más tráfico solo porque "le tocaba" por turno).
- **Sticky sessions (IP hash)**: la misma IP de cliente siempre va a la misma instancia — útil si hay estado en memoria del lado del servidor que no se puede compartir (lo ideal, ver [Escalabilidad](../system-design/quality-attributes.es.md#escalabilidad), es no necesitar esto).

## Load balancers gestionados (cloud)

En el cloud el load balancer es un **servicio gestionado**: el proveedor lo corre, lo escala y lo mantiene con alta disponibilidad — no hay nada que instalar ni parchear. Cada proveedor ofrece una variante L4 y una L7:

| Proveedor | L4 | L7 | Notas |
|---|---|---|---|
| **AWS** | **NLB** (Network Load Balancer) | **ALB** (Application Load Balancer) | Ambos bajo *Elastic Load Balancing (ELB)*. NLB: máximo throughput, mínima latencia. ALB: ruteo por path/header, integra con auth |
| **GCP** | **Network Load Balancer** | **Application Load Balancer** | Ambos bajo *Cloud Load Balancing*. Puede ser global: una sola IP delante de backends en varias regiones |
| **Azure** | **Azure Load Balancer** | **Application Gateway** | Application Gateway suma además un WAF (web application firewall) |

En Kubernetes, un `Service` de tipo `LoadBalancer` hace que el cloud aprovisione uno de estos automáticamente:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: LoadBalancer   # el cloud crea el load balancer y le da una IP pública
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8000
```

Software como Nginx, HAProxy, Traefik o Envoy también puede balancear, pero son **reverse proxies** que corrés vos mismo — ver [Reverse Proxy](reverse-proxy.es.md).
---
Relacionado: [Reverse Proxy](reverse-proxy.es.md), [API Gateway](api-gateway.es.md), [Kubernetes](../devops/kubernetes.es.md), [Escalabilidad](../system-design/quality-attributes.es.md#escalabilidad), [AWS](../cloud/aws/core-services.es.md), [GCP](../cloud/gcp/core-services.es.md), [Azure](../cloud/azure/core-services.es.md).
