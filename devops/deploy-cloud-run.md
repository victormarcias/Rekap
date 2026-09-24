# Deploy to Cloud Run (serverless containers)

How to take the image from [Dockerization](docker.md) to production without managing your own server. An alternative to [Deploy to a VPS](deploy-vps.md) when you don't want to be responsible for patching the OS, renewing certificates by hand, or paying for a machine running 24/7 that nobody's using.

## 1. What a serverless container is and what "scale to zero" means

On a VPS, the machine is on and billing all the time, whether it has traffic or not. Cloud Run runs your container **only when a request comes in** — if there's no traffic, it shuts the instance down completely (scales to **zero** replicas) and you pay nothing for that time. The first request after being at zero pays the cost of starting the container back up — see [Cold start](../diagnostics/devops.md).

```bash
# the same build command as always — Cloud Run needs nothing special in the Dockerfile
docker build -t gcr.io/my-project/myapp .
```

## 2. Serverless Postgres with Neon

A traditional DB (RDS, Cloud SQL) follows the same "always on" model as a VPS — pairing it with compute that scales to zero doesn't make sense, because the DB would keep billing 24/7 even while the compute is off most of the time. **Neon** offers Postgres that also scales to zero (and wakes the connection back up on the first query), completing the serverless model end to end.

```python
# the app connects the same way as to any Postgres — Neon is compatible, no special driver needed
DATABASE_URL = "postgresql+asyncpg://user:pass@ep-xxx.neon.tech/myapp"
```

## 3. Deploying the container

```bash
docker push gcr.io/my-project/myapp

gcloud run deploy myapp \
  --image gcr.io/my-project/myapp \
  --platform managed \
  --region us-central1 \
  --set-env-vars DATABASE_URL=$DATABASE_URL \
  --allow-unauthenticated
```

`--set-env-vars` injects config the same way the local `--env-file` does for the [Dockerfile](docker.md) — the image still has no secrets baked in. `--allow-unauthenticated` exposes the service publicly (without it, Cloud Run requires an IAM token on every request, useful for internal services between microservices but not for a public API).

## 4. Custom domain + HTTPS

Unlike a VPS (where you install Nginx and run `certbot` by hand — see [Let's Encrypt](deploy-vps.md)), Cloud Run manages the TLS certificate automatically when you map a domain: you just point the DNS and Google handles issuing and renewing the certificate.

```bash
gcloud run domain-mappings create --service myapp --domain api.myapp.com --region us-central1
# returns the DNS records (CNAME/A) you need to add at your domain provider
```

## 5. Security headers via middleware

The server doesn't add security headers on its own — they have to be declared explicitly. Without them, the app is exposed to attacks that those headers specifically mitigate (clickjacking without `X-Frame-Options`, MIME sniffing without `X-Content-Type-Options`, downgrade to HTTP without `Strict-Transport-Security`).

```python
from starlette.middleware.base import BaseHTTPMiddleware

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["Strict-Transport-Security"] = "max-age=63072000; includeSubDomains"
        return response

app.add_middleware(SecurityHeadersMiddleware)
```

---
Related: [Dockerization](docker.md), [Deploy to a VPS](deploy-vps.md), [VPS vs Cloud Run](vps-vs-cloud-run.md), [Cold start](../diagnostics/devops.md), [Elasticity](../system-design/atributos-de-calidad.md#elasticidad), [Security Headers](../security/security-headers.md) (what each one prevents).
