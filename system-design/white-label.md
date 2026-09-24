# White-Label: Considerations for Building a White-Label App

**What it is**: a single product sold/licensed to multiple clients, and each one presents it as if it were their own brand (logo, colors, domain) — the client's end user doesn't know, and doesn't care, that underneath it runs the same platform other clients of the same provider use. The difference from "a configurable app" is fundamental: here **a single codebase and infrastructure** has to serve N different clients, without them mixing with each other.

## Multi-tenancy: the central architectural problem

Each client is a **tenant**. The underlying design question: how to separate each tenant's data and config, all running on the same infrastructure.

### Data isolation strategies

| Strategy | Isolation | Operational cost | Query complexity |
|---|---|---|---|
| **Separate DB per tenant** | High — one tenant can't touch another's data even by accident | High — N databases to maintain, migrate, back up | Low — every query is scoped by nature |
| **Separate schema, shared DB** | Medium | Medium | Low-medium |
| **Shared row (`tenant_id` on every table)** | Low — depends 100% on the code filtering correctly | Low — a single DB for everyone | High — every query needs the right filter |

```sql
-- most common pattern (shared DB, shared schema): tenant_id on every table
SELECT * FROM orders WHERE tenant_id = 'client-123' AND id = 42;

-- forgetting the WHERE tenant_id is the most expensive bug that exists in a white-label system:
-- a client ends up seeing (or worse, editing) another client's data
```

It's the same underlying problem as [Sharding vs Partitioning](../database/sharding-vs-partitioning.md) (splitting data by some key), applied at the client level instead of the data-volume level.

## Dynamic branding: theming without touching code

Every tenant needs its logo, color palette, typography — without that meaning a deploy or a code branch per client. The standard pattern: **CSS Variables** (see [CSS](../frontend-react/css.md#css-variables-custom-properties)) loaded at runtime based on the active tenant, plus a config object with its asset URLs.

```json
// tenant config, resolved based on the request's domain/subdomain
{
  "tenantId": "client-123",
  "branding": {
    "primaryColor": "#1a73e8",
    "logoUrl": "https://cdn.myapp.com/tenants/client-123/logo.svg",
    "appName": "Acme Dashboard"
  }
}
```

## Domains: subdomain vs custom domain

- **Subdomain** (`client1.myapp.com`): simple — a single wildcard TLS certificate serves everyone, the code reads the request's subdomain to know which tenant it is.
- **Client's own domain** (`app.clientweb.com`, with a CNAME pointing to your infra): more professional (the client doesn't see your brand in the URL), but every domain needs its own certificate ([TLS handshake](what-happens-when-you-type-a-url.md#4-tls-handshake-if-https)) — Let's Encrypt automates issuance/renewal, but it's extra infrastructure to maintain and monitor.

## Security: tenant isolation isn't optional

In a white-label product, a bug that leaks one tenant's data to another isn't just any bug — it's the worst possible scenario (one client seeing another client's data, sometimes their direct competitor). This pushes toward:

- **Middleware that automatically injects `tenant_id`** into every query, instead of trusting every developer to remember to add it by hand.
- **Automated isolation-specific tests**: create 2 test tenants and verify that no query from one returns the other's data.
- The tenant as part of the [JWT](../backend/authentication.md#5-jwt--structure-and-stateless) claim — something validated on every request, not something inferred after authenticating.

## Feature flags and per-tenant plans

Not every tenant needs the same features — a white-label product typically sells different plans (basic/pro/enterprise), each with a set of enabled features. Handled with feature flags evaluated against the active tenant's config — **never with different code branches per client**, maintaining N forks of the same product doesn't scale.

## Onboarding: creating a new tenant is a process, not a deploy

If adding a new client means someone on the team has to touch code or infrastructure by hand, the model doesn't scale. The goal is for "creating a tenant" to be an automated, repeatable operation: running a migration that creates its record, generating its branding config, provisioning its subdomain — with no manual intervention on each signup.

## Why it matters

It's the same problem any multi-client system solves at scale (B2B SaaS, agency platforms) — getting the architecture right from day one avoids a painful migration later: going from "everything shared with no `tenant_id`" to a model with real isolation, with data already mixed together in production, costs far more than designing it well from the start.

---
Related: [Sharding vs Partitioning](../database/sharding-vs-partitioning.md), [CSS Variables](../frontend-react/css.md#css-variables-custom-properties), [Authentication and Security](../backend/authentication.md), [System quality attributes](quality-attributes.md), [What happens when you type a URL](what-happens-when-you-type-a-url.md).
