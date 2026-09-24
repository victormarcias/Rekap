# Authentication and Security — General Concepts

Fundamentals you need to have clear before writing a single line of login code — they apply in any language or framework. The concrete implementation in Python/FastAPI is in [Authentication in FastAPI](../stacks/fastapi/autenticacion.md).

## 1. Hashing vs Encryption vs Encoding

Three things that sound similar and get confused often, but solve different problems:

- **Encoding** (Base64, URL encoding): reversible, with no key at all — it's just a format transformation, not security. Anyone can decode it.
- **Encryption** (AES, etc.): reversible **with a key**. Used when at some point you're going to need to recover the original data (e.g. a card number that has to be partially displayed, or an entire session's TLS traffic).
- **Hashing** (SHA-256, bcrypt, Argon2): **irreversible**, a one-way function. Used when you're never going to need the original data — only to verify that two values match. For passwords, it's the only correct option.

```python
import base64, hashlib

# Encoding: reversible with no key at all — not security, just a format
encoded = base64.b64encode(b"password123")
base64.b64decode(encoded)  # returns "password123" as-is, no secret needed

# Hashing: irreversible — there's no "unhash()" function
hashed = hashlib.sha256(b"password123").hexdigest()
# there's no way to recover "password123" from hashed —
# you can only hash an attempt again and compare the hashes
```

## 2. Why a password is never "decrypted"

If a system can recover the original password from what it stored (because it "encrypted" it instead of hashing it), that's already a design flaw: if the server gets compromised, **all** passwords are exposed in plain text. With hashing, not even the server itself can recover the original password after registration — at login, the attempt is hashed again and the hashes are compared.

```python
# ❌ wrong design: if it's reversible ("encrypted"), whoever has the key recovers the original
saved_password = encrypt(password, server_secret_key)

# ✅ correct design: irreversible hash — not even the server can recover the original password
password_hash = hash(password)  # at login: hash(attempt) == password_hash ?
```

## 3. Salt

Without a salt, the same password always produces the same hash — which enables **rainbow tables**: precomputed hash → password tables for millions of common passwords. If an attacker gets the hash database, they look up each one in the table instead of having to compute anything. **Salt** is a random value unique per user that's combined with the password before hashing, and stored alongside the hash (it's not secret) — it makes precomputing a universal table useless, because each user's hash space is different even if two of them share the same password.

```python
# without salt: two users with the same password → same hash, visible in the DB
hash("1234") == hash("1234")  # True — an attacker sees both users share a password

# with salt: the same password produces different hashes per user
hash("1234" + salt_user_a) != hash("1234" + salt_user_b)  # True
```

## 4. Why password hashing algorithms are slow on purpose

`SHA-256`/`MD5` are designed to be **fast** — billions of hashes per second on a GPU. Excellent for file checksums, terrible for passwords: if an attacker steals the hash database, they can try billions of passwords per second. `bcrypt`, `scrypt`, and `Argon2` are deliberately **slow** (with a configurable "work factor"), and some (`scrypt`, `Argon2`) are also *memory-hard* — they require a lot of RAM per attempt, which makes attacking with parallel GPUs/ASICs much more expensive. This is the concrete reason why `hashlib.sha256(password)` alone, with nothing else, is **never** acceptable for passwords.

## 5. JWT — structure and "stateless"

A JWT has three parts separated by dots: `header.payload.signature`, each Base64URL-encoded — **not encrypted**. Anyone with the token can read the entire payload (a common beginner mistake is assuming it's secret) — the signature only guarantees it **wasn't modified**, not that it's confidential. That's why a password or secret never goes in the payload.

"Stateless" means the server doesn't store any session: it just verifies the signature with its key (secret or public, depending on the algorithm) and trusts the content if the signature is valid. This is what allows scaling horizontally without hitting a shared store on every request — see the Redis session vs in-memory session example in [Scalability](../system-design/quality-attributes.md).

## 6. Access token vs Refresh token

- **Access token**: short-lived (minutes), sent on every request, is what validates each protected endpoint.
- **Refresh token**: long-lived (days/weeks), stored more carefully, and used only to request a new access token when the old one expires — without forcing the user to log in again.

Separating them limits the damage window: if an access token leaks, it expires in minutes; the refresh token (more sensitive, longer-lived) travels much less often, reducing its exposure.

## 7. Authentication vs Authorization

- **Authentication**: who are you? (verifying identity — login).
- **Authorization**: what can you do? (permissions/roles — happens after authenticating).

The names of the HTTP status codes confuse this often: `401 Unauthorized` actually means "not authenticated" (the token is missing or invalid), and `403 Forbidden` means "authenticated, but not authorized" (the user is who they say they are, but doesn't have permission for that action). It's a common mistake to return `403` when there's actually no token — that case is `401`.

## 8. Where to store the token on the client

| Storage | Accessible by JS | Main risk |
|---|---|---|
| `localStorage` | ✅ | [**XSS**](../security/xss.md): any injected script can read the token and steal it |
| `httpOnly` cookie | ❌ | [**CSRF**](../security/csrf.md): it's sent automatically on every request to that domain, has to be mitigated with `SameSite` and/or a CSRF token |

There's no "secure by default" option — it's a trade-off: `localStorage` exposes the token to XSS, the `httpOnly` cookie protects it from XSS but opens the door to CSRF if not configured well (`SameSite=Strict/Lax` reduces that risk a lot in practice).

---
Related: [Authentication in FastAPI](../stacks/fastapi/autenticacion.md) (concrete implementation), [System quality attributes](../system-design/quality-attributes.md) (stateless scalability), [Idempotency and fault tolerance](../system-design/quality-attributes.md), [XSS](../security/xss.md), [CSRF](../security/csrf.md), [Zero Trust](../security/zero-trust.md).
