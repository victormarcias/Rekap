# Zero Trust

A security model: **never trust, always verify**. Every request is authenticated and authorized, no matter where it comes from — there's no automatically trusted "inside" zone.

## The model it replaces: perimeter (castle and moat)

The classic model concentrates security at the network edge — firewall, VPN — and once something gets past that edge, it moves with broad trust and little re-verification. It works as long as the edge doesn't break, but it fails exactly when it matters most: a stolen VPN credential, a compromised employee, or a breached internal service give an attacker free movement once inside, because the design never expected to have to distrust "the inside."

## Core principles

- **Verify explicitly**: every request is authenticated and authorized — regardless of whether it comes from the internet or the internal network. "Internal" stops being a synonym for "trustworthy."
- **Least privilege**: access limited to strictly what that specific identity needs, never more "just in case" — the same principle we already saw in [SQL Injection](sql-injection.md#how-its-prevented) at the DB level, or in an [AI agent's](../agentic-ai/risks-and-mitigations.md#technical-mitigations) permissions.
- **Assume breach**: design as if the attacker is already inside — segment the network, encrypt internal traffic as well as external, actively monitor instead of trusting that the perimeter held.

## Example: authenticating "internal" traffic too

```python
# ❌ perimeter model: trusts any request coming from the internal network
@app.get("/orders/{order_id}")
def get_order(order_id: str, request: Request):
    if is_internal_ip(request.client.host):   # "already past the firewall, nothing more needed"
        return find_order(order_id)
    raise HTTPException(403)

# ✅ Zero Trust: validates identity on every request, regardless of origin
@app.get("/orders/{order_id}")
def get_order(order_id: str, token: str = Depends(verify_token)):
    if not has_permission(token, "orders:read"):
        raise HTTPException(403)
    return find_order(order_id)
```

The difference isn't cosmetic: in the first case, anyone who manages to get inside the network (a stolen VPN, a compromised container in the same VPC) gets in with no further questions. In the second, they also need a valid token with the specific permission — the network alone isn't enough.

## mTLS between microservices

The same idea applied to service-to-service communication: with **mTLS** (mutual TLS) each service presents its own certificate, and the one receiving the connection verifies the caller's identity — not just the other way around (client verifying server, like in normal HTTPS). Two services on the same private network still authenticate each other on every call, instead of assuming "being in the same VPC" is already enough of a guarantee.

## When it matters

Microservices architectures, multi-tenant environments, remote teams where a single office perimeter to protect no longer exists. It's the conceptual foundation behind products like BeyondCorp (Google) or a service mesh doing mTLS by default between services — the idea that network location stopped being a valid trust signal.

---
Related: [Authentication and Security](../backend/authentication.md), [SQL Injection](sql-injection.md#how-its-prevented), [Risks and Mitigations in AI Agents](../agentic-ai/risks-and-mitigations.md#technical-mitigations).
