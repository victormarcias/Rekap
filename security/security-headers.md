# Security Headers

The server doesn't add security headers on its own — they have to be declared explicitly on every response (see the [example middleware in Deploy to Cloud Run](../devops/deploy-cloud-run.md#5-security-headers-via-middleware) for a concrete implementation). Here's the detail of what each one prevents.

## Content-Security-Policy (CSP)

Restricts which origins the page can load and execute scripts, styles, images, etc. from. It's the defense that mitigates the impact of an [XSS](xss.md) even if escaping fails somewhere specific — even if a malicious script manages to get injected, the browser won't execute it if it comes from a disallowed origin.

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com
```

## X-Frame-Options / `frame-ancestors` (CSP)

Prevents the site from being loaded inside another domain's `<iframe>` — mitigates **clickjacking**: the attacker overlays your site (invisible, with opacity 0) on top of their own page, so the user thinks they're clicking on something harmless when they're actually clicking one of your buttons (e.g. "Confirm transfer").

```
X-Frame-Options: DENY
```

## X-Content-Type-Options: nosniff

Prevents the browser from trying to "guess" a response's content type and ignoring the declared `Content-Type` — without this, a file uploaded as an image but that actually contains JavaScript could end up executing as a script instead of displaying as an image.

```
X-Content-Type-Options: nosniff
```

## Strict-Transport-Security (HSTS)

Tells the browser to force HTTPS on **every** future request to that domain, even if the user types `http://` by hand or clicks an old link with `http`. Without this, every visit is exposed to a downgrade to HTTP on the first connection (and to a man-in-the-middle attack during that window).

```
Strict-Transport-Security: max-age=63072000; includeSubDomains
```

---
Related: [XSS](xss.md), [Deploy to Cloud Run](../devops/deploy-cloud-run.md#5-security-headers-via-middleware).
