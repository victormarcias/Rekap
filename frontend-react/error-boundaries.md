# Error Boundaries

Un error boundary es un componente que **atrapa errores de JS lanzados durante el render** de sus componentes hijos, y muestra una UI de fallback en vez de que toda la app se rompa en blanco. Sin uno, un error en cualquier componente tira abajo el árbol entero de React que lo contiene.

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true }; // dispara el render de fallback en el próximo render
  }

  componentDidCatch(error, info) {
    logErrorToService(error, info); // efecto secundario: loguear, reportar a Sentry, etc.
  }

  render() {
    if (this.state.hasError) return <h2>Algo salió mal.</h2>;
    return this.props.children;
  }
}

function App() {
  return (
    <ErrorBoundary>
      <Dashboard /> {/* si Dashboard tira un error de render, solo esta parte se reemplaza */}
    </ErrorBoundary>
  );
}
```

## Por qué es una clase (no hay hook equivalente)

`getDerivedStateFromError` y `componentDidCatch` no tienen todavía un equivalente en hooks — es una de las pocas razones legítimas para seguir escribiendo un class component en código moderno de React. En la práctica casi nadie lo escribe a mano: se usa la librería `react-error-boundary`, que expone la misma funcionalidad con una API de componente reutilizable.

```jsx
import { ErrorBoundary } from 'react-error-boundary';

<ErrorBoundary fallback={<h2>Algo salió mal.</h2>} onError={logErrorToService}>
  <Dashboard />
</ErrorBoundary>
```

## Qué NO atrapa

Un error boundary solo atrapa errores durante el **render**, en lifecycle methods, y en constructores de componentes hijos. No atrapa errores en: event handlers (`onClick`), código async (`setTimeout`, promesas), server-side rendering, ni errores lanzados en el error boundary mismo. Los errores de un `onClick` se manejan con un `try/catch` normal dentro del handler.

## Dónde ubicarlos

No hace falta un único error boundary global — es común poner uno alrededor de secciones independientes de la UI (ej. un widget que consume una API externa poco confiable), así un error ahí no tira abajo el resto de la página que sí está funcionando.

---
Relacionado: [Hooks](hooks.md), [Diagnóstico Frontend](../diagnostico/frontend.md).
