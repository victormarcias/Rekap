# Renderizado: SSR vs CSR vs SSG vs ISR vs SPA

Dónde y cuándo se genera el HTML de una página — en el servidor, en build time, o en el browser — y qué implica cada elección. Concepto genérico de arquitectura frontend; lo implementan Next.js, Nuxt, SvelteKit, Remix, cada uno con su propia sintaxis.

## CSR vs SSR

- **CSR (Client-Side Rendering)**: el servidor manda un HTML casi vacío (`<div id="root"></div>`) más un bundle de JS. El browser tiene que descargar y ejecutar ese JS antes de que aparezca cualquier contenido — pantalla en blanco mientras tanto, y nada que un crawler de buscador pueda leer sin ejecutar JS.
- **SSR**: el servidor ejecuta la app y devuelve el HTML **ya renderizado con el contenido real** para ese request. El usuario ve contenido apenas llega la respuesta, sin esperar a que el JS cargue — mejor First Contentful Paint y SEO out-of-the-box.

## SPA (Single Page Application)

Una app que carga un único HTML inicial y después maneja toda la navegación entre "páginas" en el cliente, con JS, sin pedirle un documento nuevo al servidor en cada cambio de URL (un router de cliente como React Router intercepta la navegación y solo actualiza el DOM). CSR casi siempre implica una SPA; SSR/SSG pueden convivir con el mismo patrón de SPA **después** del primer render — el HTML inicial viene renderizado, pero la navegación posterior sigue siendo client-side, sin recargar la página completa.

Se contrapone al modelo tradicional de **MPA (Multi-Page Application)**, donde cada link es un request nuevo al servidor que devuelve un documento HTML completo — más simple, pero pierde estado de UI (scroll, inputs sin submitear) en cada navegación.

## Hidratación

El HTML que llega del servidor es contenido estático — todavía no tiene los event listeners de React attacheados. **Hidratación** es el paso donde el JS del cliente "toma control" de ese HTML ya existente y lo vuelve interactivo, sin volver a crear los nodos DOM desde cero (reutiliza lo que el servidor ya mandó).

```jsx
// Ej. con Next.js — el mismo componente corre en el servidor (primer render) y en el cliente (hidratación)
export default function ProductPage({ product }) {
  return <h1>{product.name}</h1>; // en el servidor: HTML real. En el cliente: se hidrata sobre ese HTML.
}

export async function getServerSideProps() {
  const product = await fetchProduct(); // corre en el servidor, en cada request
  return { props: { product } };
}
```

Si el HTML que devuelve el servidor no coincide exactamente con lo que el cliente renderizaría (ej. usar `Date.now()` o `Math.random()` directo en el render), la hidratación falla con un *hydration mismatch* — un bug clásico de SSR.

## Isomorfismo (Universal apps)

Que el **mismo código de componente** corra sin cambios tanto en el servidor como en el cliente — es justamente lo que el ejemplo de arriba muestra: `ProductPage` no sabe ni le importa si lo está ejecutando Node en el servidor o el browser durante la hidratación. Antes de que esto fuera estándar (frameworks como Angular.js clásico o Backbone corrían solo en el cliente), había que escribir y mantener dos implementaciones del mismo render — una en el servidor (ej. en un template engine) y otra en JS del cliente — con el riesgo constante de que se desincronizaran. "Isomorphic" y "Universal" se usan como sinónimos en la práctica.

## SSG, la variante prima

**Static Site Generation**: el mismo concepto de "renderizar en el servidor" pero en **build time**, no por request — el HTML se genera una vez y se sirve igual para todos (con una CDN por delante, ver [CDN](../devops/cdn.es.md)). Sirve cuando el contenido no depende del usuario ni cambia entre requests (un blog, landing pages); SSR hace falta cuando sí depende (un dashboard con datos del usuario logueado).

**El límite de SSG puro**: si el contenido cambia (ej. se actualiza el precio de un producto), la única forma de reflejarlo es rebuildear y redeployar **todo el sitio de nuevo** — aunque solo haya cambiado una página entre miles.

## ISR (Incremental Static Regeneration)

Resuelve justo ese límite: en vez de que todo el sitio quede fijo hasta el próximo deploy completo, cada página estática puede **regenerarse sola, en background, cada tanto** — sin rebuildear el resto del sitio.

```jsx
// Ej. con Next.js (Pages Router)
export async function getStaticProps() {
  const product = await fetchProduct();
  return {
    props: { product },
    revalidate: 60, // esta página puede volver a generarse en background, como mucho cada 60 segundos
  };
}
```

Cómo funciona en la práctica: el primer usuario que pide la página después de que pasaron los 60 segundos **sigue recibiendo la versión vieja cacheada** (no espera nada) — pero dispara, en paralelo, una regeneración en background. El próximo usuario ya recibe la versión actualizada. Este patrón se llama *stale-while-revalidate*: nunca hay un usuario esperando a que el servidor renderice, a cambio de tolerar que a veces se sirva contenido levemente desactualizado por un ratito.

**SSG vs ISR, la diferencia real**: SSG genera una vez en build time y ahí queda fijo hasta el próximo deploy manual. ISR también genera en build time, pero además se puede volver a regenerar sola después, sin deploy — es SSG con una fecha de vencimiento configurable por página, en vez de un solo build congelado para todo el sitio.

## Trade-off

SSR mueve trabajo del cliente al servidor: mejor experiencia inicial, pero cada request ahora cuesta CPU del servidor (en vez de ser un archivo estático servido por una CDN) y agrega la complejidad de la hidratación. No es gratis — es elegir dónde pagar el costo de renderizar.

Una ventaja poco obvia de SSR para troubleshooting: si algo sale mal en el HTML que le llega al usuario, es mucho más probable que el problema esté del lado del servidor (el `getServerSideProps`, la query a la DB, el fetch) porque ahí es donde se generó ese HTML — reduce el espacio de búsqueda comparado con CSR, donde el bug puede estar en cualquier punto de una cadena larga de JS ejecutándose en el cliente.
