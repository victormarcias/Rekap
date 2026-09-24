# HTML

## Semántica general y containers

Usar la etiqueta que describe el **rol** del contenido, no una que solo lo hace ver bien. Un `<div>` no dice nada sobre qué es ese bloque; `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>` sí — y ese significado lo aprovechan los screen readers, los crawlers de buscadores y hasta el propio navegador (ej. navegación por landmarks).

```html
<!-- ❌ todo es un div, no hay información sobre la estructura -->
<div class="header">...</div>
<div class="main-content">...</div>

<!-- ✅ la estructura se lee sin necesitar las clases -->
<header>...</header>
<main>
  <article>
    <h1>Título del post</h1>
    <section>...</section>
  </article>
  <aside>Contenido relacionado</aside>
</main>
<footer>...</footer>
```

`<article>` vs `<section>`: `<article>` es contenido que tiene sentido por sí solo fuera de la página (un post, un producto); `<section>` es una agrupación temática dentro de algo más grande, sin sentido propio aislada.

## Forms

Los elementos de formulario (`<input>`, `<select>`, `<textarea>`, `<label>`) traen gratis validación básica del browser, accesibilidad (un `<label>` bien asociado hace que el screen reader anuncie el campo, y que tocar el texto también enfoque el input) y semántica que el navegador usa para autocompletar.

```html
<!-- el for=id conecta el label con el input — necesario para accesibilidad -->
<label for="email">Email</label>
<input id="email" type="email" required autocomplete="email" />

<!-- type correcto = teclado correcto en mobile + validación nativa -->
<input type="email" />   <!-- valida formato de email -->
<input type="tel" />     <!-- teclado numérico en mobile -->
<input type="number" min="0" max="100" />
```

Un `<div onClick>` simulando un botón pierde todo esto — no es focuseable con Tab, no responde a Enter/Espacio, y un screen reader no sabe que es interactivo. Usar `<button>` (o `<input type="submit">`) siempre que algo dispare una acción.

## SEO

Lo que un crawler de buscador puede indexar depende de que el HTML tenga la información en el lugar que espera, no solo de que "se vea bien":

```html
<head>
  <title>Título único de la página (aparece en el resultado de búsqueda)</title>
  <meta name="description" content="Resumen de 1-2 líneas, aparece debajo del título en el buscador" />
  <link rel="canonical" href="https://miapp.com/producto/123" />
</head>
```

- **Un solo `<h1>` por página**, jerarquía de headings sin saltos (`h1` → `h2` → `h3`, no `h1` directo a `h3`) — el crawler arma un índice del contenido a partir de esa jerarquía.
- Contenido que solo aparece después de ejecutar JS puede no ser indexado por todos los crawlers — ver [CSR vs SSR](rendering.es.md#csr-vs-ssr), porque CSR manda un HTML casi vacío.

## Tab navigation

El orden en que Tab recorre la página sigue el **orden del DOM**, no el orden visual — si CSS reposiciona un elemento (`order` en flex, `position: absolute`), el foco puede "saltar" en un orden que no coincide con lo que se ve, confundiendo a cualquiera que navegue solo con teclado.

```html
<!-- tabindex="0": suma el elemento al orden natural de tab (útil en elementos no interactivos por defecto, como un div que actúa de botón) -->
<div role="button" tabindex="0">Acción custom</div>

<!-- tabindex="-1": sacable del tab, pero foco-able por JS (ej. mover el foco a un modal recién abierto) -->
<div tabindex="-1" id="modal">...</div>

<!-- tabindex positivo (2, 3...): ❌ evitar — fuerza un orden manual que rompe
     apenas se agrega/mueve un elemento en el medio, y pisa el orden natural del DOM -->
```

## Anchors

`<a href>` navega (cambia de URL, funciona con "abrir en pestaña nueva", el browser lo indexa como link real) — un `onClick` en un `<span>` o `<div>` no hace nada de eso. Regla simple: si navega, es un `<a>`; si dispara una acción sin cambiar de página, es un `<button>`.

```html
<!-- ✅ navega a otra URL -->
<a href="/producto/123">Ver producto</a>

<!-- ❌ común pero incorrecto: usar <a> sin href real para disparar JS -->
<a href="#" onClick={...}>Guardar</a>  <!-- confunde navegación con acción -->

<!-- ✅ si es una acción, es un botón -->
<button onClick={...}>Guardar</button>
```

---
Relacionado: [Accesibilidad](accessibility.es.md), [Renderizado: SSR vs CSR vs SSG vs SPA](rendering.es.md).
