# Testing in React

For the general theory (test pyramid, mocks/stubs, fixtures, brittle vs resilient tests) see [Testing — General Concepts](../system-design/testing.md). Here the focus is on what specifically changes when testing a React UI.

## What we test

In frontend, what's worth testing is **behavior visible to the user** — what gets rendered, what happens when the user interacts — not internal implementation details (internal function names, a hook's internal state). A test that breaks because a component was refactored internally without changing its external behavior is a brittle test (see [Brittle vs resilient tests](../system-design/testing.md#7-brittle-tests-vs-resilient-tests)).

## Jest — the basics

Jest is the **test runner**: it discovers test files, runs them, and provides the `describe`/`test`/`expect` that structures the test. RTL (below) runs **on top of** Jest — Jest executes, RTL finds elements and simulates interactions.

```js
describe('add', () => {
  test('adds two positive numbers', () => {
    expect(add(2, 3)).toBe(5);
  });

  test('throws an error with an invalid argument', () => {
    expect(() => add(2, "a")).toThrow();
  });
});
```

Common `expect` matchers: `toBe` (strict equality, `===`), `toEqual` (structural equality — compares an object/array's content, not the reference), `toContain` (an array/string contains something), `toBeNull`/`toBeUndefined`, `toThrow` (the function throws an error).

**Mocks with `jest.fn()` / `jest.mock()`**: replacing a function or an entire module with a fake version controlled by the test — the same [Mock](../system-design/testing.md#2-test-doubles--mock-vs-stub-vs-fake-vs-spy) concept explained in the general theory, here with Jest's concrete syntax.

```js
const fetchUser = jest.fn(() => Promise.resolve({ id: 1, name: 'Vic' }));

test('calls fetchUser exactly once', async () => {
  await loadProfile(fetchUser);
  expect(fetchUser).toHaveBeenCalledTimes(1);
});

// jest.mock() replaces an entire module — useful to avoid hitting a real API in the test
jest.mock('./api', () => ({
  fetchUser: jest.fn(() => Promise.resolve({ id: 1, name: 'Vic' })),
}));
```

**Vitest is API-compatible**: the same code above (`describe`, `test`, `expect`, even `vi.fn()` instead of `jest.fn()` with an alias) runs the same in Vitest — that's why migrating a project from Jest to Vitest is usually mechanical, not a rewrite.

## Unit Testing with RTL (React Testing Library)

RTL's philosophy is to test the component **the way a real user would use it**: find elements by visible text, role, or label (not CSS classes or internal IDs), and simulate real interactions instead of calling internal functions directly.

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('shows the error message when submitting without an email', async () => {
  render(<LoginForm />);

  // finds by role/text, not CSS selector — so the test keeps working
  // even if the component's classes or internal structure change
  await userEvent.click(screen.getByRole('button', { name: /log in/i }));

  expect(screen.getByText(/email is required/i)).toBeInTheDocument();
});
```

`getByRole`/`getByLabelText` force the component to be accessible in order to test it — a side benefit: if RTL can't "find" the button because it has no clear role/label, a screen reader couldn't either.

## End to End Testing with Cypress

While RTL tests an isolated component (mounted in a simulated DOM, with no real backend), Cypress runs the **full** app in a real browser, against a real server (or mocked at the network level) — it simulates the entire real-user flow, end to end.

```js
// cypress/e2e/login.cy.js
describe('Login', () => {
  it('allows logging in and redirects to the dashboard', () => {
    cy.visit('/login');
    cy.get('input[name="email"]').type('user@test.com');
    cy.get('input[name="password"]').type('password123');
    cy.get('button[type="submit"]').click();

    cy.url().should('include', '/dashboard');
    cy.contains('Welcome').should('be.visible');
  });
});
```

The cost of E2E is that it's slower and more fragile against infrastructure changes (if the backend is down, the test fails even though the frontend is fine) — that's why it sits at the narrow tip of the [test pyramid](../system-design/testing.md#1-test-pyramid): few E2E tests covering critical flows (login, checkout), many more unit tests with RTL covering the rest.

## E2E Tools: Selenium vs Cypress vs Playwright

| | Selenium | Cypress | Playwright |
|---|---|---|---|
| Released | 2004 — the oldest | 2017 | 2020, Microsoft |
| Architecture | Runs *outside* the browser (WebDriver protocol) | Runs *inside* the browser (same event loop) | Runs outside, modern CDP-like protocol |
| Multi-browser | Yes — Chrome, Firefox, Safari, Edge | Limited, historically tied to Chromium | Yes, natively — Chromium, Firefox, WebKit |
| Element auto-wait | No, has to be waited manually | ✅ | ✅ |
| Speed | Slower | Fast | Very fast |
| CI parallelization | Possible, but you build your own infra | Needs Cypress Cloud (paid) to do it well | Native and free |
| Languages | Java, Python, C#, JS, Ruby... (the most polyglot) | JS/TS only | JS/TS, Python, Java, C# |
| Today's trend | Legacy — heavily installed in large, established companies | Still widely used | Gaining ground fast in new projects |

None fully replaces the other — today's default choice on a new project is usually Playwright, unless the team already has experience/infrastructure built on Cypress or needs Selenium's polyglot support (e.g. a QA team already writing in Java).
