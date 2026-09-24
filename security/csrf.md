# CSRF (Cross-Site Request Forgery)

Tricking an **already-authenticated** victim's browser into sending an unwanted request to a site where it has an active session. The browser attaches that domain's cookies automatically to any request, no matter where it comes from — so the server, if it does nothing else, can't tell a legitimate request from a forged one at a glance.

```html
<!-- Malicious page the victim visits while they have an active session on bank.com -->
<form action="https://bank.com/transfer" method="POST" id="attack">
  <input type="hidden" name="amount" value="10000">
  <input type="hidden" name="destination" value="attackers-account">
</form>
<script>document.getElementById("attack").submit()</script>
<!-- The browser sends bank.com's session cookie automatically with this POST -->
```

## Why a GET shouldn't have side effects

If `GET /transfer?amount=10000&destination=...` changed state, the attack wouldn't even need a form — a single `<img src="https://bank.com/transfer?...">` on any page would do it. It's one of the underlying reasons behind [HTTP Safe Methods](../backend/http-methods.md#safe-methods--no-side-effects).

## How it's prevented

- **`SameSite` on the session cookie**: `Strict`/`Lax` tell the browser not to send that cookie on requests coming from another site — cuts the attack off at the source, without touching the backend (see [Cookies](../frontend-react/almacenamiento-cliente.md#cookies)).
- **CSRF token**: a unique value per session (or per form) that the server requires on every state-changing request, and that an external attacker has no way to know or replicate.

```python
# The server generates a unique token when creating the session and requires it
# on any request that modifies state
@app.post("/transfer")
def transfer(amount: float, csrf_token: str = Form(...)):
    if csrf_token != session["csrf_token"]:
        raise HTTPException(403, "Invalid CSRF token")
    ...
```

`SameSite` and a CSRF token aren't mutually exclusive — `SameSite=Lax` (the default in modern browsers) already covers most cases, but a CSRF token is still the explicit defense for APIs that need to accept cross-site cookies on purpose.

---
Related: [XSS](xss.md), [Authentication and Security](../backend/authentication.md#8-where-to-store-the-token-on-the-client), [HTTP Methods](../backend/http-methods.md#safe-methods--no-side-effects), [Client-side storage](../frontend-react/almacenamiento-cliente.md#cookies).
