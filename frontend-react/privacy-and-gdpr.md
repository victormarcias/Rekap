# Privacy and GDPR

**GDPR** (General Data Protection Regulation) is the European Union's data protection regulation — but in practice it matters for almost any app with international users: if a single user in the EU uses your product, GDPR applies, no matter where the company is based. It became the de facto standard other privacy laws (CCPA in California, LGPD in Brazil) largely copied.

## Data Security

### Data Minimization

Collect only the data that's **actually needed**, not everything that "might be useful later." A registration form asking for phone, address, and birthdate when only email is needed is more attack surface in case of a breach, and more compliance work for no benefit in return.

```jsx
// ❌ asks for more than the current feature needs
<Form fields={['email', 'password', 'phone', 'address', 'birthdate']} />

// ✅ only what's essential for registration — the rest gets asked later, if needed
<Form fields={['email', 'password']} />
```

### Data Encryption

Sensitive data encrypted both **in transit** (HTTPS/TLS — see [TLS handshake](../system-design/what-happens-when-you-type-a-url.md#4-tls-handshake-if-https)) and **at rest** (sensitive fields encrypted in the database, not just the connection). Encrypting isn't the same as hashing — a password gets hashed (can't be reversed), data you do need to recover later (e.g. a card number) gets encrypted (see [Hashing vs Encryption vs Encoding](../backend/authentication.md#1-hashing-vs-encryption-vs-encoding)).

### Correct Settings (secure by default configuration)

The most common mistake isn't a code vulnerability, it's an insecure default configuration: a public storage bucket when it should've been private, an API key with admin permissions when it only needed read access, missing security headers (see [Security headers](../devops/deploy-cloud-run.md#5-security-headers-via-middleware)). The general principle is *least privilege*: grant the minimum access necessary, never "everything just in case."

## User Consent and Privacy

### User Notice

An accessible privacy policy in understandable language (not a 20-page legal document nobody reads) explaining what data gets collected, for what, and for how long it's kept. GDPR requires this information to be available **before** collecting the data, not hidden after registration.

### Cookie Policy

Documenting which cookies the site uses and why, split by category:

- **Essential**: needed for the site to work (e.g. the session cookie) — don't require consent.
- **Non-essential**: analytics, marketing, third-party tracking — do require explicit consent before loading.

### User Consent

GDPR requires **opt-in**, not opt-out — the user has to actively accept before non-essential cookies get activated, not see them active by default with the option to reject them afterward. A cookie banner that comes with everything pre-checked, or that keeps loading analytics scripts even though the user hasn't responded yet, doesn't comply with this (a pattern known as a *dark pattern*).

```js
// non-essential cookies/scripts only load if explicit stored consent exists
function loadAnalytics() {
  const consent = localStorage.getItem('cookie-consent');
  if (consent !== 'accepted') return; // no consent, nothing loads

  const script = document.createElement('script');
  script.src = 'https://analytics.example.com/script.js';
  document.body.appendChild(script);
}
```

Consent also has to be just as easy to **withdraw** as to give — a "reject" button at the same visual level as "accept," not hidden in a settings submenu.

---
Related: [Client-side storage](client-side-storage.md#cookies), [Accessibility](accessibility.md).
