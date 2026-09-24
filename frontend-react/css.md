# CSS

## CSS Reset / Normalize

Every browser ships its own default styles for HTML elements (margins on `<body>`, `<ul>` with bullets and padding, different `<h1>`...`<h6>` sizes, etc.) — and they aren't exactly the same across Chrome, Firefox, and Safari. A reset is a stylesheet loaded first, before any of your own CSS, to start from a predictable baseline.

```css
/* "Hard" reset (e.g. Eric Meyer style): wipes everything, even the useful parts —
   afterward you have to redefine heading sizes, lists, etc. by hand */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
ul, ol { list-style: none; }
a { text-decoration: none; color: inherit; }
```

**Normalize.css** is the more modern approach: instead of wiping everything, it **fixes inconsistencies between browsers** while keeping the defaults that are actually useful (an `<h1>` is still bigger than normal text, but the exact size gets equalized across browsers). Most current projects use this instead of a hard reset — or don't even add it by hand: frameworks like Tailwind already ship their own modern reset (the "preflight") included.

Why it matters: without this, the same HTML/CSS can look slightly different depending on the browser with no bug in the code at all — just different defaults competing with your own styles.

## Specificity and selectors

The browser resolves conflicts between CSS rules by **specificity**, not by whichever "looks more specific" at a glance. From lowest to highest weight: element selector (`div`) < class/attribute/pseudo-class (`.card`, `[type="text"]`, `:hover`) < id (`#header`) < inline styles (`style="..."`) < `!important`.

```css
div.card { color: blue; }      /* specificity: 0-1-1 */
#header .card { color: red; }  /* specificity: 1-1-0 → this one wins */
```

`!important` breaks the normal cascade and wins almost always — that's why it's a red flag in a codebase: it usually signals someone couldn't (or didn't know how to) beat another rule cleanly, and ends up starting a `!important`-vs-`!important` arms race.

### `@layer` — ordering the cascade without fighting specificity

`@layer` (CSS Cascade Layers) explicitly declares the priority order between groups of rules — a layer declared later beats one declared earlier, **regardless of each individual rule's specificity within that layer**. It's the modern way to avoid the `!important` war when several libraries coexist (Bootstrap, a design system, your own CSS) competing for the same elements.

```css
/* the order of this line defines priority — lowest to highest */
@layer reset, libraries, components, utilities;

@layer libraries {
  @import url("bootstrap.css"); /* whatever Bootstrap brings stays contained in this layer */
}

@layer components {
  /* beats ANY rule from .libraries, even if that rule has
     higher specificity (e.g. an id) — the layer wins over specificity */
  .button { color: blue; }
}
```

Within the same layer, normal specificity still applies — `@layer` doesn't replace those rules, it adds a priority level **above** them. Any CSS not inside any `@layer` is treated as the highest-priority layer of all, so old layer-less code keeps winning by default without having to migrate it.

## Flexbox vs Grid

- **Flexbox**: layout in **one dimension** — a row or a column. Built for distributing space among items in a list or aligning elements within a container.
- **Grid**: layout in **two dimensions** — rows and columns at once. Built for a page's overall structure or a complex component (dashboard, gallery).

```css
/* Flex: a navbar in a row, spread to the edges */
.navbar { display: flex; justify-content: space-between; align-items: center; }

/* Grid: page layout with a fixed sidebar and flexible content */
.layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
}
```

They don't compete — it's common to use Grid for the overall structure and Flex inside each grid cell.

## CSS Variables (Custom Properties)

Reusable values defined with `--name` and read with `var()`. Unlike a preprocessor's variables (SASS), they live in the DOM at runtime — they can be read/changed with JS and respect the normal cascade (can be overridden per selector, useful for light/dark themes).

```css
:root { --color-primary: #6b21a8; --spacing: 8px; }

.button { background: var(--color-primary); padding: calc(var(--spacing) * 2); }

/* dark theme: same variable name, different value in a different scope */
[data-theme="dark"] { --color-primary: #a78bfa; }
```

## Animations vs Transitions

- **Transition**: interpolates between two states when something changes (hover, focus, a class being added/removed). Needs a trigger — it doesn't start on its own.
- **Animation**: defines a sequence with `@keyframes`, can have multiple intermediate steps, loop, and start on its own when the element loads — doesn't depend on something changing.

```css
/* Transition: only interpolates between the normal state and :hover */
.button { transition: transform 0.2s ease; }
.button:hover { transform: scale(1.05); }

/* Animation: multi-step sequence, repeats on its own */
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}
.spinner { animation: pulse 1.5s infinite; }
```

## Stacking contexts (why my `z-index` doesn't work)

`z-index` doesn't compete globally against every `z-index` on the page — it only competes **within the same stacking context**. An element creates its own new stacking context (isolating its children from the rest of the page) when it has, among other things: a `position` other than `static` + an explicit `z-index`, an `opacity` below 1, a `transform`, or `will-change`.

```css
/* the modal has z-index: 9999, but if its parent container
   has opacity < 1 or a transform, the modal ends up "trapped"
   in the parent's stacking context and can't beat elements
   outside that parent, no matter how high its z-index is */
.parent { opacity: 0.99; } /* accidentally creates a stacking context */
.modal { position: fixed; z-index: 9999; } /* trapped inside the parent */
```

This is the classic bug: bumping `z-index` higher and higher fixes nothing if the problem is that the element is trapped in the wrong stacking context — it has to be fixed at the parent level (remove the unneeded `opacity`/`transform`, or use a [portal](https://react.dev/reference/react-dom/createPortal) in React to render the modal outside the parent's tree).

## Methodologies for organizing and scoping styles

All of them solve the same problem — "how do I keep my classes from colliding between components" — from different layers:

- **BEM** (`Block__Element--Modifier`): a naming convention in plain CSS, no build tool. Avoids collisions through discipline, not tooling.
  ```css
  .card { }
  .card__title { }
  .card--featured { }
  ```
- **CSS Modules**: each `.module.css` gets processed at build time and its classes get automatically renamed to something unique (`.card_a3f1x`) — the scoping is guaranteed by the tool, not the convention.
  ```jsx
  import styles from './Card.module.css';
  <div className={styles.card}>...</div>
  ```
- **CSS-in-JS** (e.g. styled-components): styles are written in JS, placed alongside the component, and can depend on props at runtime — but that has a performance cost (generating CSS on the client) that CSS Modules doesn't have.
  ```jsx
  const Card = styled.div`
    background: ${props => props.featured ? '#fef3c7' : 'white'};
  `;
  ```
- **SASS/SCSS**: a preprocessor — adds variables, nesting, and mixins that compile to plain CSS before reaching the browser. Doesn't solve scoping on its own (usually combined with BEM or CSS Modules for that), solves plain CSS's lack of reusability.
  ```scss
  $color-primary: #6b21a8;
  .card {
    &__title { color: $color-primary; }
    &:hover { opacity: 0.9; }
  }
  ```
