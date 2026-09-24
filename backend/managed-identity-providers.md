# Managed Identity Providers (Cognito, Auth0, Firebase Auth)

Instead of building your own auth flow (registration, login, hashing, JWT — see [Authentication and Security](authentication.md)), delegate it to a specialized external service that already solved, tested, and hardened it.

## What they solve

- Registration/login (including social login — Google, GitHub, etc. — without implementing OAuth against each provider by hand).
- MFA (multi-factor authentication).
- Password recovery, email verification.
- Token issuance and validation (JWT, sessions).
- Compliance/certifications a small team can hardly put together on its own (SOC 2, etc.).

```python
# conceptual — the backend no longer hashes passwords or issues its own JWTs,
# it just validates the token the external provider issued
def get_current_user(token: str):
    claims = cognito_client.verify_token(token)  # verification is done by the provider's SDK
    return claims["sub"]
```

## The trade-off

**Pros**: less code of your own to maintain and less attack surface — every line of auth you don't write is a line that can't have a security bug in it. Large providers invest in security more than most teams can.

**Cons**: *vendor lock-in* — migrating from one provider to another later means migrating users, tokens, and sometimes re-verifying identities. Less fine-grained control over the exact flow (customizing the login UX can be limited). Cost that grows with the number of active users.

## Examples

- **AWS Cognito**: integrates natively with the rest of AWS services (API Gateway, Lambda).
- **Auth0** (now part of Okta): more flexible/cloud-agnostic, strong at customizing flows.
- **Firebase Auth**: the simplest option for apps already using the rest of the Firebase ecosystem.

## The concepts still apply

Using a managed provider doesn't make [Authentication and Security](authentication.md) irrelevant — the backend still needs to understand what a JWT is, how a signature is validated, and the difference between [Authentication vs Authorization](authentication.md#7-authentication-vs-authorization) (the provider handles authentication — who you are — but the specific permissions of your domain — what you can do — remain your backend's responsibility).

---
Related: [Authentication and Security](authentication.md), [API Gateway](api-gateway.md) (authentication is usually validated there, centrally).
