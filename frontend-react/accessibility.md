# Accessibility

## What accessibility is

That a person can use the app no matter how they interact with it — with a mouse, keyboard only, a screen reader, high zoom, color blindness, reduced mobility. It's not a separate feature bolted on at the end; it's a property of how the HTML/CSS/JS was built from the start (see [HTML](html.md) — correct semantics is already half the work).

## Universal Design vs Accessible Design

- **Accessible Design**: adapting something existing so people with disabilities can also use it — often a parallel solution or a patch (e.g. a "text-only" version of a site).
- **Universal Design**: designing from the start so it works for everyone, with no need for a parallel version — a subtitle helps someone deaf, but also someone watching the video without sound on the bus. Accessibility principles end up being good design for everyone, not a concession.

## Types of accessibility problems

- **Visual**: low color contrast, text that doesn't scale with zoom, information conveyed only through color (e.g. "fields in red are required" with no other indicator).
- **Motor**: interactive elements too small or too close together (hard to tap precisely), functionality that only responds to hover or drag with no keyboard alternative.
- **Auditory**: video/audio with no subtitles or transcript.
- **Cognitive**: unnecessarily complex language, inconsistent layouts between pages, time limits too short to complete an action (e.g. a session that expires with no warning).

## Screen readers

Software that reads the screen's content out loud (or in braille), navigating through the HTML's semantic structure — not by how it looks, but by what each element **is** (`<button>` announces "button," `<nav>` announces "navigation"). That's why semantic HTML and ARIA attributes (`aria-label`, `aria-expanded`, `role`) aren't decoration: they're literally the only thing a screen reader has to describe the interface.

```html
<!-- without aria-label, the screen reader only announces "button" — without saying what for -->
<button aria-label="Close modal">✕</button>

<!-- aria-expanded tells the screen reader the state of a collapsible element,
     something a sighted user sees from the arrow icon -->
<button aria-expanded="false" aria-controls="menu">Menu</button>
```

## Alt texts

An image's `alt` attribute is what a screen reader announces instead of the image, and what's shown if the image fails to load. An empty `alt` (`alt=""`) is a valid, explicit choice for purely decorative images — it tells the screen reader "skip me, I add no information" — very different from not setting `alt` at all, which makes some screen readers read the filename (`IMG_4821.jpg`).

```html
<!-- ✅ describes the information the image provides -->
<img src="sales-chart.png" alt="Sales growing 30% in the last quarter" />

<!-- ✅ decorative, adds no information — explicitly skipped -->
<img src="decorative-line.svg" alt="" />

<!-- ❌ doesn't describe anything useful -->
<img src="sales-chart.png" alt="image" />
```

---
Related: [HTML](html.md).
