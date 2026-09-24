# SSRF (Server-Side Request Forgery)

Tricking the **server** into making an HTTP request to a URL the attacker chooses — typical when an app accepts a user-provided URL and the backend "visits" it (downloading an avatar from a URL, generating a thumbnail, validating a webhook URL).

```python
# ❌ vulnerable: the server fetches whatever URL the user passes it, no restriction
@app.post("/avatar-from-url")
def avatar_from_url(url: str):
    image = requests.get(url)
    return save_image(image.content)

# the attacker sends url="http://169.254.169.254/latest/meta-data/iam/security-credentials/"
# (AWS's internal metadata endpoint) — the server DOES have network access there,
# and ends up fetching credentials that should never leave the infrastructure
```

## Why it's especially dangerous in the cloud

The server has network access to internal resources (cloud provider metadata endpoints, internal databases, other microservices on the same private network) that an attacker, standing outside, couldn't reach directly. SSRF uses the server as a **proxy into** the network — the request "leaves" the server, but points inward.

## How it's prevented

- **Whitelist of allowed domains/IPs**, instead of accepting whatever URL the user sends.
- **Block private/internal IP ranges** (`169.254.0.0/16`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `127.0.0.1`) on any request the server builds from external input.
- **Disable automatic redirects** when fetching — a URL that looks "valid" at a glance can redirect to an internal one, and if the HTTP client blindly follows redirects, validating the original URL is pointless.

---
Related: [SQL Injection](sql-injection.md), [AWS — Core Services](../cloud/aws/core-services.md) (metadata endpoint), [Dockerization](../devops/docker.md).
