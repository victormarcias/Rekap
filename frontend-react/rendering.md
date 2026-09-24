# Rendering: SSR vs CSR vs SSG vs ISR vs SPA

Where and when a page's HTML gets generated — on the server, at build time, or in the browser — and what each choice implies. A generic frontend architecture concept; implemented by Next.js, Nuxt, SvelteKit, Remix, each with its own syntax.

## CSR vs SSR

- **CSR (Client-Side Rendering)**: the server sends an almost-empty HTML (`<div id="root"></div>`) plus a JS bundle. The browser has to download and run that JS before any content appears — a blank screen in the meantime, and nothing a search crawler can read without running JS.
- **SSR**: the server runs the app and returns the HTML **already rendered with the real content** for that request. The user sees content as soon as the response arrives, without waiting for the JS to load — better First Contentful Paint and SEO out of the box.

## SPA (Single Page Application)

An app that loads a single initial HTML and then handles all navigation between "pages" on the client, with JS, without requesting a new document from the server on every URL change (a client-side router like React Router intercepts navigation and only updates the DOM). CSR almost always implies an SPA; SSR/SSG can coexist with the same SPA pattern **after** the first render — the initial HTML comes pre-rendered, but subsequent navigation stays client-side, without reloading the whole page.

Contrasts with the traditional **MPA (Multi-Page Application)** model, where every link is a new request to the server that returns a complete HTML document — simpler, but loses UI state (scroll, unsubmitted inputs) on every navigation.

## Hydration

The HTML that arrives from the server is static content — it doesn't have React's event listeners attached yet. **Hydration** is the step where the client's JS "takes control" of that already-existing HTML and makes it interactive, without recreating the DOM nodes from scratch (it reuses what the server already sent).

```jsx
// E.g. with Next.js — the same component runs on the server (first render) and on the client (hydration)
export default function ProductPage({ product }) {
  return <h1>{product.name}</h1>; // on the server: real HTML. On the client: it hydrates over that HTML.
}

export async function getServerSideProps() {
  const product = await fetchProduct(); // runs on the server, on every request
  return { props: { product } };
}
```

If the HTML the server returns doesn't exactly match what the client would render (e.g. using `Date.now()` or `Math.random()` directly in the render), hydration fails with a *hydration mismatch* — a classic SSR bug.

## Isomorphism (Universal apps)

That the **same component code** runs unchanged on both the server and the client — which is exactly what the example above shows: `ProductPage` doesn't know or care whether Node is running it on the server or the browser during hydration. Before this was standard (frameworks like classic Angular.js or Backbone only ran on the client), you had to write and maintain two implementations of the same render — one on the server (e.g. in a template engine) and another in client JS — with the constant risk of them drifting apart. "Isomorphic" and "Universal" are used as synonyms in practice.

## SSG, the cousin variant

**Static Site Generation**: the same "render on the server" concept but at **build time**, not per request — the HTML gets generated once and served the same to everyone (with a CDN in front, see [CDN](../devops/cdn.md)). Useful when content doesn't depend on the user or change between requests (a blog, landing pages); SSR is needed when it does (a dashboard with the logged-in user's data).

**Pure SSG's limit**: if the content changes (e.g. a product's price gets updated), the only way to reflect it is rebuilding and redeploying **the entire site again** — even if only one page out of thousands changed.

## ISR (Incremental Static Regeneration)

Solves exactly that limit: instead of the entire site staying fixed until the next full deploy, each static page can **regenerate itself, in the background, every so often** — without rebuilding the rest of the site.

```jsx
// E.g. with Next.js (Pages Router)
export async function getStaticProps() {
  const product = await fetchProduct();
  return {
    props: { product },
    revalidate: 60, // this page can regenerate in the background, at most every 60 seconds
  };
}
```

How it works in practice: the first user requesting the page after the 60 seconds have passed **still gets the old cached version** (waits for nothing) — but it triggers, in parallel, a background regeneration. The next user already gets the updated version. This pattern is called *stale-while-revalidate*: there's never a user waiting for the server to render, in exchange for tolerating that slightly stale content sometimes gets served for a brief moment.

**SSG vs ISR, the real difference**: SSG generates once at build time and stays fixed until the next manual deploy. ISR also generates at build time, but can also regenerate itself later, without a deploy — it's SSG with a configurable expiration date per page, instead of a single frozen build for the whole site.

## Trade-off

SSR moves work from the client to the server: a better initial experience, but every request now costs server CPU (instead of being a static file served by a CDN) and adds hydration complexity. It's not free — it's choosing where to pay the cost of rendering.

A not-so-obvious advantage of SSR for troubleshooting: if something goes wrong in the HTML the user receives, it's much more likely the problem is on the server side (the `getServerSideProps`, the DB query, the fetch) because that's where that HTML was generated — it narrows the search space compared to CSR, where the bug could be anywhere in a long chain of JS running on the client.
