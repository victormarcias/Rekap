# Reverse Proxy

A server that sits in front of one or more application servers and receives the clients' requests on their behalf — the client only talks to the proxy, never directly to the app. **Nginx** is the canonical example: software you install, configure, and run yourself.

## Forward vs reverse proxy

| | Forward proxy | Reverse proxy |
|---|---|---|
| Sits in front of | The **clients** | The **servers** |
| Hides | Who the client is (from the server) | Which server answers (from the client) |
| Configured by | The client's side (company network, VPN) | The server's side (whoever runs the app) |
| Example | Corporate proxy filtering outbound traffic | Nginx in front of a FastAPI app |

## What it does

Everything the app would otherwise have to implement itself, handled once at the edge:

- **TLS termination**: handles HTTPS and talks plain HTTP to the app behind it.
- **Static files and compression**: serves assets directly and compresses responses.
- **Caching**: answers repeated requests without bothering the app.
- **Rate limiting**: rejects abusive clients before they reach the app.
- **Hiding the origin**: the app listens on an internal port, never exposed to the internet.
- **Several apps on one machine**: routes by domain or path.

```nginx
# one Nginx in front of the app: TLS, static files, compression, rate limit
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

server {
    listen 443 ssl;
    server_name myapp.com;
    ssl_certificate     /etc/letsencrypt/live/myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.com/privkey.pem;
    gzip on;

    location /static/ { root /var/www/myapp; }          # served by Nginx, never reaches the app

    location / {
        limit_req zone=api burst=20;                    # rate limiting at the edge
        proxy_pass http://127.0.0.1:8000;               # the app, on an internal port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;        # without this the app only sees 127.0.0.1
    }
}
```

The step-by-step setup on a real server is in [Nginx as reverse proxy](../devops/deploy-vps.md#4-nginx-as-reverse-proxy).

## Reverse proxy vs load balancer vs API gateway

The three forward requests to something behind them, so they overlap — the difference is what each one is *for*:

| | Reverse proxy | [Load balancer](load-balancers.md) | [API gateway](api-gateway.md) |
|---|---|---|---|
| Main job | Be the public face of one or more apps | Spread traffic across **replicas** of the same service | Route between **different services** + auth, quotas |
| Typical form | Software you run (Nginx, Caddy) | Managed cloud service (AWS ELB, GCP Cloud Load Balancing) | Managed service or software (Kong, AWS API Gateway) |
| Knows about | Domains, paths, TLS | Which instances are healthy | Clients, tokens, API versions |

A load balancer is a specialized reverse proxy, which is why Nginx can do both: with several servers in an `upstream`, it balances between them.

```nginx
upstream app_servers {
    least_conn;                  # same algorithms as a load balancer
    server 10.0.0.11:8000;
    server 10.0.0.12:8000;
}

server {
    location / { proxy_pass http://app_servers; }
}
```

In practice, on a single VPS Nginx is the reverse proxy; once the app runs on several instances in the cloud, the provider's managed load balancer goes in front.

## Common tools

| Tool | Notes |
|---|---|
| **Nginx** | The most widespread. Reverse proxy + web server, static config files |
| **Caddy** | Like Nginx, with automatic HTTPS out of the box |
| **HAProxy** | Specialized in proxying and balancing TCP/HTTP, very high performance |
| **Traefik** | Detects Docker/K8s services automatically and configures itself |
| **Envoy** | Proxy used as the data plane of service meshes (Istio) |
| **K8s Ingress controller** | The reverse proxy (usually Nginx or Traefik) that exposes a cluster's HTTP services |

---
Related: [Load balancers](load-balancers.md), [API Gateway](api-gateway.md), [Deploy to a VPS](../devops/deploy-vps.md#4-nginx-as-reverse-proxy), [CDN](../devops/cdn.md), [Kubernetes](../devops/kubernetes.md).
