# HTML

## General semantics and containers

Use the tag that describes the content's **role**, not one that just makes it look right. A `<div>` says nothing about what that block is; `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>` do — and that meaning gets used by screen readers, search crawlers, and even the browser itself (e.g. landmark navigation).

```html
<!-- ❌ everything is a div, no information about the structure -->
<div class="header">...</div>
<div class="main-content">...</div>

<!-- ✅ the structure reads without needing the classes -->
<header>...</header>
<main>
  <article>
    <h1>Post title</h1>
    <section>...</section>
  </article>
  <aside>Related content</aside>
</main>
<footer>...</footer>
```

`<article>` vs `<section>`: `<article>` is content that makes sense on its own outside the page (a post, a product); `<section>` is a thematic grouping within something bigger, with no meaning isolated on its own.

## Forms

Form elements (`<input>`, `<select>`, `<textarea>`, `<label>`) come with basic browser validation for free, accessibility (a properly associated `<label>` makes the screen reader announce the field, and touching the text also focuses the input), and semantics the browser uses for autocomplete.

```html
<!-- the for=id connects the label to the input — needed for accessibility -->
<label for="email">Email</label>
<input id="email" type="email" required autocomplete="email" />

<!-- correct type = correct mobile keyboard + native validation -->
<input type="email" />   <!-- validates email format -->
<input type="tel" />     <!-- numeric keyboard on mobile -->
<input type="number" min="0" max="100" />
```

A `<div onClick>` simulating a button loses all of this — it's not focusable with Tab, doesn't respond to Enter/Space, and a screen reader doesn't know it's interactive. Use `<button>` (or `<input type="submit">`) whenever something triggers an action.

## SEO

What a search crawler can index depends on the HTML having the information where it expects it, not just on it "looking good":

```html
<head>
  <title>Unique page title (shows up in the search result)</title>
  <meta name="description" content="1-2 line summary, appears below the title in search" />
  <link rel="canonical" href="https://myapp.com/product/123" />
</head>
```

- **A single `<h1>` per page**, heading hierarchy with no gaps (`h1` → `h2` → `h3`, not `h1` straight to `h3`) — the crawler builds a content outline from that hierarchy.
- Content that only appears after running JS might not be indexed by every crawler — see [CSR vs SSR](rendering.md#csr-vs-ssr), because CSR sends an almost-empty HTML.

## Tab navigation

The order Tab traverses the page follows the **DOM order**, not the visual order — if CSS repositions an element (`order` in flex, `position: absolute`), focus can "jump" in an order that doesn't match what's seen, confusing anyone navigating with keyboard only.

```html
<!-- tabindex="0": adds the element to the natural tab order (useful for elements not interactive by default, like a div acting as a button) -->
<div role="button" tabindex="0">Custom action</div>

<!-- tabindex="-1": removable from tab order, but focusable via JS (e.g. moving focus to a newly opened modal) -->
<div tabindex="-1" id="modal">...</div>

<!-- positive tabindex (2, 3...): ❌ avoid — forces a manual order that breaks
     as soon as an element is added/moved in between, and overrides the DOM's natural order -->
```

## Anchors

`<a href>` navigates (changes the URL, works with "open in new tab," the browser indexes it as a real link) — an `onClick` on a `<span>` or `<div>` does none of that. Simple rule: if it navigates, it's an `<a>`; if it triggers an action without changing pages, it's a `<button>`.

```html
<!-- ✅ navigates to another URL -->
<a href="/product/123">View product</a>

<!-- ❌ common but wrong: using <a> with no real href to trigger JS -->
<a href="#" onClick={...}>Save</a>  <!-- confuses navigation with action -->

<!-- ✅ if it's an action, it's a button -->
<button onClick={...}>Save</button>
```

---
Related: [Accessibility](accessibility.md), [Rendering: SSR vs CSR vs SSG vs SPA](rendering.md).
