# Client-side Storage

Where an app's state lives and for how long — the answer changes depending on how sensitive the data is and whether it needs to survive a refresh, closing a tab, or a logout.

## In-memory state (JavaScript)

A regular variable or a `useState` — lives while the page is loaded, disappears on refresh. The default place for any UI state that doesn't need to persist (an open modal, an unsubmitted input).

## `localStorage`

Persists indefinitely in the browser, survives closing the tab and restarting the computer, scoped by origin (protocol + domain + port). Only stores strings — objects need `JSON.stringify`/`JSON.parse`.

```js
localStorage.setItem('theme', 'dark');
const theme = localStorage.getItem('theme'); // 'dark'
localStorage.setItem('user', JSON.stringify({ id: 1, name: 'Vic' }));
```

## `sessionStorage`

Same API as `localStorage`, but scoped to the **tab**: cleared when it closes, and not shared between tabs of the same site (each tab has its own `sessionStorage`, even if they open the same URL).

## Cookies

Contrary to popular belief, **cookies are the safest way to store information on the client** — not `localStorage`. The reason is a flag neither `localStorage` nor `sessionStorage` can have: `HttpOnly`.

```
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

- **`HttpOnly`**: the cookie is invisible to JavaScript (`document.cookie` doesn't show it) — if the app has an [XSS](../security/xss.md) vulnerability, the injected malicious script can read everything in `localStorage`, but not an `HttpOnly` cookie. This is the underlying reason: `localStorage` is 100% accessible to any JS running on the page, cookies might not be.
- **`Secure`**: only sent over HTTPS, never in plain text.
- **`SameSite`**: controls whether the cookie is sent on requests coming from another site — `Strict`/`Lax` mitigate [CSRF](../security/csrf.md).

The trade-off: cookies are sent automatically on **every** request to the same domain (adding weight to each request) and have a small size limit (~4KB) — that's why they don't work for storing large data blobs, only identifiers like a session token.

## Server-side state

The most sensitive or shared-across-devices data doesn't live on the client at all — the client only stores an identifier (e.g. a session cookie or a JWT), and the server resolves that identifier against its own storage (DB, Redis) to know who the user is and what belongs to them. It's the only option when state has to be consistent between the same user's phone and laptop at the same time.

## Which one to use

| | Survives refresh | Survives closing tab | Accessible by JS | Sent on every request | Size limit |
|---|---|---|---|---|---|
| Memory (`useState`) | ❌ | ❌ | ✅ | ❌ | Limited by available RAM — no fixed limit in practice |
| `sessionStorage` | ✅ | ❌ | ✅ | ❌ | ~5-10MB per origin (varies by browser) |
| `localStorage` | ✅ | ✅ | ✅ | ❌ | ~5-10MB per origin (varies by browser) |
| Cookie (`HttpOnly`) | ✅ | ✅ | ❌ | ✅ | ~4KB per cookie |

Practical rule: session/auth tokens → `HttpOnly` cookie. UI preferences with nothing sensitive (theme, language) → `localStorage`. State of a multi-step flow in the same visit (e.g. a wizard) → `sessionStorage`. Anything that depends on other users or must be the source of truth → server.
