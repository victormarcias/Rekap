# Reverse Proxy

Un servidor que se pone delante de uno o más servidores de aplicación y recibe los requests de los clientes en su nombre — el cliente solo habla con el proxy, nunca directo con la app. **Nginx** es el ejemplo canónico: software que instalás, configurás y corrés vos mismo.

## Forward vs reverse proxy

| | Forward proxy | Reverse proxy |
|---|---|---|
| Se pone delante de | Los **clientes** | Los **servidores** |
| Oculta | Quién es el cliente (frente al servidor) | Qué servidor responde (frente al cliente) |
| Lo configura | El lado del cliente (red corporativa, VPN) | El lado del servidor (quien corre la app) |
| Ejemplo | Proxy corporativo que filtra el tráfico saliente | Nginx delante de una app FastAPI |

## Qué hace

Todo lo que la app tendría que implementar por su cuenta, resuelto una sola vez en el borde:

- **Terminación TLS**: maneja el HTTPS y le habla HTTP plano a la app que tiene detrás.
- **Archivos estáticos y compresión**: sirve los assets directo y comprime las respuestas.
- **Caching**: responde requests repetidos sin molestar a la app.
- **Rate limiting**: rechaza clientes abusivos antes de que lleguen a la app.
- **Ocultar el origen**: la app escucha en un puerto interno, nunca expuesto a internet.
- **Varias apps en una máquina**: rutea por dominio o por path.

```nginx
# un Nginx delante de la app: TLS, estáticos, compresión, rate limit
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

server {
    listen 443 ssl;
    server_name myapp.com;
    ssl_certificate     /etc/letsencrypt/live/myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.com/privkey.pem;
    gzip on;

    location /static/ { root /var/www/myapp; }          # lo sirve Nginx, nunca llega a la app

    location / {
        limit_req zone=api burst=20;                    # rate limiting en el borde
        proxy_pass http://127.0.0.1:8000;               # la app, en un puerto interno
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;        # sin esto la app solo ve 127.0.0.1
    }
}
```

El paso a paso en un servidor real está en [Nginx como reverse proxy](../devops/deploy-vps.es.md#4-nginx-como-reverse-proxy).

## Reverse proxy vs load balancer vs API gateway

Los tres reenvían requests hacia algo que tienen detrás, así que se superponen — la diferencia es para qué sirve cada uno:

| | Reverse proxy | [Load balancer](load-balancers.es.md) | [API gateway](api-gateway.es.md) |
|---|---|---|---|
| Trabajo principal | Ser la cara pública de una o más apps | Repartir tráfico entre **réplicas** del mismo servicio | Rutear entre **servicios distintos** + auth, cuotas |
| Forma típica | Software que corrés vos (Nginx, Caddy) | Servicio gestionado del cloud (AWS ELB, GCP Cloud Load Balancing) | Servicio gestionado o software (Kong, AWS API Gateway) |
| Sabe de | Dominios, paths, TLS | Qué instancias están sanas | Clientes, tokens, versiones de API |

Un load balancer es un reverse proxy especializado, por eso Nginx puede hacer las dos cosas: con varios servidores en un `upstream`, balancea entre ellos.

```nginx
upstream app_servers {
    least_conn;                  # los mismos algoritmos que un load balancer
    server 10.0.0.11:8000;
    server 10.0.0.12:8000;
}

server {
    location / { proxy_pass http://app_servers; }
}
```

En la práctica, en un solo VPS Nginx es el reverse proxy; cuando la app corre en varias instancias en el cloud, el load balancer gestionado del proveedor se pone adelante.

## Herramientas comunes

| Herramienta | Notas |
|---|---|
| **Nginx** | La más difundida. Reverse proxy + web server, config en archivos estáticos |
| **Caddy** | Como Nginx, con HTTPS automático de fábrica |
| **HAProxy** | Especializado en proxy y balanceo TCP/HTTP, rendimiento muy alto |
| **Traefik** | Detecta servicios de Docker/K8s automáticamente y se configura solo |
| **Envoy** | Proxy usado como data plane de service meshes (Istio) |
| **K8s Ingress controller** | El reverse proxy (normalmente Nginx o Traefik) que expone los servicios HTTP de un cluster |

---
Relacionado: [Load balancers](load-balancers.es.md), [API Gateway](api-gateway.es.md), [Deploy a un VPS](../devops/deploy-vps.es.md#4-nginx-como-reverse-proxy), [CDN](../devops/cdn.es.md), [Kubernetes](../devops/kubernetes.es.md).
