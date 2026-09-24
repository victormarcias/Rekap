# Code Splitting / Lazy Loading

Partir el bundle de JS en pedazos que se cargan **cuando hacen falta**, en vez de mandar toda la app en un único archivo desde el primer request. El usuario que entra al login no necesita descargar el código del panel de administración todavía.

## `React.lazy` + `Suspense`

```jsx
import { lazy, Suspense } from 'react';

// el import() dinámico le dice al bundler (Webpack/Vite) que este componente
// va en un chunk separado, no en el bundle principal
const AdminPanel = lazy(() => import('./AdminPanel'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      {/* el chunk de AdminPanel recién se pide por red la primera vez
          que este componente intenta renderizarse */}
      <AdminPanel />
    </Suspense>
  );
}
```

`Suspense` muestra el `fallback` mientras el chunk todavía no llegó — es el mismo mecanismo que usan los frameworks con SSR/streaming para no bloquear el render completo esperando una sola parte lenta.

## Dónde suele aplicarse

- **Rutas**: cada página de un router carga su propio chunk — la más común y la de mayor impacto (nadie necesita el JS del checkout si está mirando la home).
- **Componentes pesados condicionales**: un modal complejo, un editor de texto rico, un gráfico con una librería grande — cosas que no se ven en el primer render.

```jsx
// dividido por ruta con React Router
const Home = lazy(() => import('./pages/Home'));
const Checkout = lazy(() => import('./pages/Checkout'));

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/checkout" element={<Checkout />} /> {/* su JS ni se pide hasta entrar acá */}
</Routes>
```

## No confundir con Tree Shaking ni Module Federation

Los tres reducen cuánto JS termina corriendo en el browser, pero en momentos distintos:

- **[Tree shaking](tree-shaking.es.md)**: en **build time**, elimina código que nunca se usa en ningún lado — no llega ni a existir en ningún bundle.
- **Code splitting**: el código sí se usa, pero se **difiere** su descarga hasta el momento en que hace falta — sigue siendo parte de la app, solo que en otro archivo.
- **[Module Federation](module-federation.es.md)**: parte la app en piezas que ni siquiera comparten el mismo build/deploy — code splitting divide el bundle de una sola app; Module Federation compone bundles de apps independientes en runtime.
