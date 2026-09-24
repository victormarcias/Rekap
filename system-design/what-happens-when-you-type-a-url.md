# What happens when you type a URL

The full journey from pressing Enter to the page being rendered on screen — covers every layer involved: network, server, and client.

## 1. Parsing the URL

The browser splits the URL into its parts: protocol (`https`), host (`www.example.com`), port (default 443 for https, 80 for http), path (`/products`), query string (`?id=1`).

```
https://www.example.com:443/products?id=1
└─┬──┘   └───────┬───────┘└┬┘└───┬───┘└──┬──┘
protocol       host      port   path   query
```

## 2. DNS Resolution — from domain to IP

The host (`www.example.com`) doesn't work for connecting over the network, an IP is needed. The browser searches in cascade, stopping at the first place it finds the answer:

1. **Browser cache** — did I resolve this domain recently?
2. **Operating system cache**.
3. **Local router**.
4. **ISP's DNS resolver** (or a public one like `8.8.8.8`, `1.1.1.1`) — if none of the above has the answer, this one does the full recursive resolution:
   - Asks a **root server** (`.`) → tells it where the `.com` TLD's servers are.
   - Asks the **TLD server** (`.com`) → tells it where `example.com`'s **authoritative nameserver** is.
   - Asks the **authoritative nameserver** → returns the real IP (`A`/`AAAA` record).
5. The IP gets cached at every level according to the DNS record's **TTL**, to avoid repeating this whole trip on the next visit.

This step can add tens to hundreds of ms if nothing is cached — it's one of the first things to look at when an app "starts slow." See [Cold start](../diagnostics/devops.md) and [CDN](../diagnostics/devops.md) for related latency.

## 3. TCP Connection — three-way handshake

With the IP in hand, the browser opens a TCP connection with the server (port 443/80):

```
Client → SYN      → Server
Client ← SYN-ACK  ← Server
Client → ACK      → Server
```

Three network trips (or 1.5 *round trips*) before a single byte of data can be sent. With **HTTP/2** or **HTTP/3** (QUIC, over UDP) this cost is reduced by reusing connections or avoiding the traditional TCP handshake.

## 4. TLS handshake (if HTTPS)

On top of the already-open TCP connection, the encryption layer gets negotiated:

1. Client sends `ClientHello` (TLS versions and ciphers it supports).
2. Server responds with its **certificate** (includes the public key) and the chosen cipher.
3. Client validates the certificate against a trusted **certificate authority (CA)**, verifies the domain matches and it isn't expired/revoked.
4. A symmetric session key (faster than asymmetric) gets negotiated, used to encrypt the rest of the communication.

TLS 1.3 reduced this to 1 round trip (vs 2 for TLS 1.2). This step is what guarantees no one in the middle (public Wi-Fi, ISP) can read or modify the data.

## 5. HTTP Request

With the encrypted connection ready, the browser sends the request:

```http
GET /products?id=1 HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 ...
Accept: text/html
Cookie: session=abc123
```

## 6. The server processes the request

On the server side there can be several layers before reaching a response:

- **Load balancer** (L4/L7) receives the connection and decides which application server to send it to.
- **Application server** runs the logic: authentication, database queries, calls to other services.
- Possibly hits a **cache** (Redis) before going to the database.
- Builds the response (server-rendered HTML, JSON, etc.).

See [Load balancers](README.md) and [Backend Diagnostics](../diagnostics/backend.md)/[Database Diagnostics](../diagnostics/database.md) for what can go wrong at this step.

## 7. HTTP Response

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Cache-Control: public, max-age=3600
Content-Length: 5324

<!DOCTYPE html>...
```

The `status code` tells the browser how to interpret the response (`200` OK, `301`/`302` redirect, `404` not found, `500` server error).

## 8. The browser renders

With the HTML in hand, the **critical rendering path** kicks off:

1. **Parsing HTML → DOM tree**: the browser builds the document's node tree. On finding a `<link rel="stylesheet">` or a `<script>`, it fires additional requests (reusing the connection if possible — *keep-alive*, or multiplexed if it's HTTP/2).
2. **Parsing CSS → CSSOM**: tree of computed styles.
3. **DOM + CSSOM → Render Tree**: combines both, discarding nodes that aren't displayed (`display: none`).
4. **Layout (reflow)**: calculates each element's exact position and size on screen.
5. **Paint**: converts the render tree into actual pixels.
6. **Composite**: combines the different layers (e.g. elements with `position: fixed`, GPU animations) into the final image.

A `<script>` with no `defer`/`async` **blocks** HTML parsing until it's downloaded and executed — that's why the classic recommendation is putting scripts at the end of `<body>`, or using `defer` (downloads in parallel, executes after HTML parsing) / `async` (downloads in parallel, executes as soon as it's ready, no guaranteed order).

```html
<script src="app.js" defer></script>
```

## Summary of the journey

```
URL → parsing → DNS (cache → resolver → root → TLD → authoritative)
    → TCP handshake → TLS handshake → HTTP request
    → [load balancer → app server → cache/DB] → HTTP response
    → HTML parse (DOM) + CSS parse (CSSOM) → Render Tree → Layout → Paint → Composite
```

Each leg is a different diagnostic point — see [diagnostics by layer](../diagnostics/README.md) to dig into each one when something's slow.
