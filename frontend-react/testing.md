# Testing en React

Para la teoría general (test pyramid, mocks/stubs, fixtures, tests frágiles vs robustos) ver [Testing — Conceptos Generales](../system-design/testing.md). Acá el foco es qué cambia específicamente al testear una UI de React.

## Qué testeamos

En frontend, lo que vale la pena testear es **comportamiento visible para el usuario** — qué se renderiza, qué pasa cuando el usuario interactúa — no detalles internos de implementación (nombres de funciones internas, estado interno de un hook). Un test que rompe porque se refactorizó un componente por dentro sin cambiar su comportamiento externo es un test frágil (ver [Tests frágiles vs robustos](../system-design/testing.md#7-tests-frágiles-vs-tests-robustos)).

## Jest — lo básico

Jest es el **test runner**: quién descubre los archivos de test, los ejecuta, y da el `describe`/`test`/`expect` que arma la estructura del test. RTL (más abajo) corre **arriba** de Jest — Jest ejecuta, RTL busca elementos y simula interacciones.

```js
describe('sumar', () => {
  test('suma dos números positivos', () => {
    expect(sumar(2, 3)).toBe(5);
  });

  test('lanza un error con un argumento inválido', () => {
    expect(() => sumar(2, "a")).toThrow();
  });
});
```

Matchers comunes de `expect`: `toBe` (igualdad estricta, `===`), `toEqual` (igualdad estructural — compara el contenido de un objeto/array, no la referencia), `toContain` (un array/string contiene algo), `toBeNull`/`toBeUndefined`, `toThrow` (la función lanza un error).

**Mocks con `jest.fn()` / `jest.mock()`**: reemplazar una función o un módulo entero por una versión falsa controlada por el test — el mismo concepto de [Mock](../system-design/testing.md#2-test-doubles--mock-vs-stub-vs-fake-vs-spy) explicado en la teoría general, acá con la sintaxis concreta de Jest.

```js
const fetchUsuario = jest.fn(() => Promise.resolve({ id: 1, name: 'Vic' }));

test('llama a fetchUsuario una sola vez', async () => {
  await cargarPerfil(fetchUsuario);
  expect(fetchUsuario).toHaveBeenCalledTimes(1);
});

// jest.mock() reemplaza un módulo entero — útil para no pegarle a una API real en el test
jest.mock('./api', () => ({
  fetchUsuario: jest.fn(() => Promise.resolve({ id: 1, name: 'Vic' })),
}));
```

**Vitest es API-compatible**: el mismo código de arriba (`describe`, `test`, `expect`, incluso `vi.fn()` en vez de `jest.fn()` con un alias) corre igual en Vitest — es por eso que migrar un proyecto de Jest a Vitest suele ser mecánico, no una reescritura.

## Unit Testing con RTL (React Testing Library)

La filosofía de RTL es testear el componente **como lo usaría un usuario real**: buscar elementos por texto, rol o label visible (no por clases CSS o IDs internos), y simular interacciones reales en vez de llamar funciones internas directamente.

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('muestra el mensaje de error al submitear sin email', async () => {
  render(<LoginForm />);

  // busca por rol/texto, no por selector CSS — así el test sigue funcionando
  // aunque cambien las clases o la estructura interna del componente
  await userEvent.click(screen.getByRole('button', { name: /ingresar/i }));

  expect(screen.getByText(/el email es obligatorio/i)).toBeInTheDocument();
});
```

`getByRole`/`getByLabelText` obligan a que el componente sea accesible para poder testearlo — un beneficio colateral: si RTL no puede "encontrar" el botón porque no tiene un rol/label claro, un screen reader tampoco podría.

## End to End Testing con Cypress

Mientras RTL testea un componente aislado (montado en un DOM simulado, sin backend real), Cypress corre la app **completa** en un browser real, contra un servidor real (o mockeado a nivel de red) — simula el flujo entero de un usuario real, de punta a punta.

```js
// cypress/e2e/login.cy.js
describe('Login', () => {
  it('permite loguearse y redirige al dashboard', () => {
    cy.visit('/login');
    cy.get('input[name="email"]').type('user@test.com');
    cy.get('input[name="password"]').type('password123');
    cy.get('button[type="submit"]').click();

    cy.url().should('include', '/dashboard');
    cy.contains('Bienvenido').should('be.visible');
  });
});
```

El costo de E2E es que es más lento y más frágil ante cambios de infraestructura (si el backend está caído, el test falla aunque el frontend esté perfecto) — por eso va en la punta angosta de la [test pyramid](../system-design/testing.md#1-test-pyramid): pocos tests E2E cubriendo los flujos críticos (login, checkout), muchos más unit tests con RTL cubriendo el resto.

## Herramientas E2E: Selenium vs Cypress vs Playwright

| | Selenium | Cypress | Playwright |
|---|---|---|---|
| Generación | 2004 — el más viejo | 2017 | 2020, Microsoft |
| Arquitectura | Corre *afuera* del browser (protocolo WebDriver) | Corre *adentro* del browser (mismo event loop) | Corre afuera, protocolo moderno tipo CDP |
| Multi-browser | Sí — Chrome, Firefox, Safari, Edge | Limitado, históricamente atado a Chromium | Sí, nativo — Chromium, Firefox, WebKit |
| Auto-wait de elementos | No, hay que esperarlo manual | ✅ | ✅ |
| Velocidad | Más lento | Rápido | Muy rápido |
| Paralelización en CI | Posible, pero infraestructura propia | Necesita Cypress Cloud (pago) para hacerlo bien | Nativo y gratis |
| Lenguajes | Java, Python, C#, JS, Ruby... (el más políglota) | Solo JS/TS | JS/TS, Python, Java, C# |
| Tendencia hoy | Legacy — muy instalado en empresas grandes ya armadas | Sigue muy usado | Ganando terreno rápido en proyectos nuevos |

Ninguno reemplaza al otro por completo — la elección hoy en un proyecto nuevo suele ser Playwright por default, salvo que el equipo ya tenga experiencia/infraestructura hecha en Cypress o necesite el soporte políglota de Selenium (ej. un equipo de QA que ya escribe en Java).
