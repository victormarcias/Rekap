# TypeScript — Type System

## Function typing

Each parameter and the return value can be explicitly typed — the compiler rejects a call that doesn't satisfy the contract before the code ever runs.

```ts
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

calculateTotal(10, "2"); // ❌ compile-time error: Argument of type 'string' is not assignable to parameter of type 'number'

// optional parameters (?) and default values
function greet(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}`;
}

// functions as values: the type describes the full signature
const add: (a: number, b: number) => number = (a, b) => a + b;
```

Why it matters: without this, a type error (passing a string where a number is expected) is only discovered at runtime, potentially in production — with typing, the error shows up in the editor while the code is being written.

## Type Aliases

Naming a type with `type` to reuse it without repeating the full shape every time — doesn't create a new type, it's just a readable name for something that could already be written inline.

```ts
// ❌ without an alias: repeat the same shape in every function that needs it
function update(user: { id: string; name: string; email: string }) { }
function create(user: { id: string; name: string; email: string }) { }

// ✅ with an alias: defined once, reused
type User = { id: string; name: string; email: string };
function update(user: User) { }
function create(user: User) { }
```

**`type` vs `interface`**: for describing an object's shape they overlap quite a bit. The practical difference: `type` can name anything (unions, tuples, primitives), while `interface` only describes objects, but can be extended/reopened in several places in the code (useful in libraries). Simple rule: `interface` for an object's shape that might grow over time, `type` for everything else.

## Literal Types

Unlike `string` (which accepts any possible string), a literal type pins down the **exact** allowed value.

```ts
type Direction = "up" | "down" | "left" | "right"; // union of string literals

let dir: Direction = "up";        // ✅
let dir2: Direction = "diagonal";  // ❌ error — not one of the 4 allowed values

type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6; // number literals also exist
```

**`as const`**: by default, TypeScript infers the most general possible type for a value (`string`, `number`). `as const` forces the exact literal type instead of the general one.

```ts
let a = "hello";           // inferred type: string
let b = "hello" as const;   // inferred type: "hello" — the exact literal

const config = { mode: "dark" };            // config.mode is type string
const config2 = { mode: "dark" } as const;   // config2.mode is type "dark"
```

Literal types are the basis of the union examples in the section below (`"idle" | "loading" | "error"`) — they were already being used there, here they're named as their own concept.

## Enum vs Union of literals

To represent "a value from a closed set of options" there are two paths — they're not equivalent, they differ in **runtime cost**.

```ts
// enum: generates real JS code that exists in production
enum Direction { Up = "UP", Down = "DOWN" }

// union of literals: purely compile-time, disappears when compiled
type Direction2 = "up" | "down";
```

`enum` isn't just a type — it compiles to a real JS object:

```js
// this is what the enum above generates, in the compiled JS
var Direction;
(function (Direction) {
    Direction["Up"] = "UP";
    Direction["Down"] = "DOWN";
})(Direction || (Direction = {}));
```

A union of literals generates nothing — it's information the compiler uses and discards, zero runtime cost. That's why several style guides (Google's, among others) directly recommend avoiding `enum` in favor of unions of literals.

Two specific problems with `enum`, besides the cost:

- **Reverse mapping in numeric enums** (`enum Direction { Up, Down }`, no strings): TS creates an automatic bidirectional mapping (`Direction.Up === 0` and also `Direction[0] === "Up"`) — confuses more than it helps, almost nobody uses that second direction.
- **`const enum`** (the zero-runtime-cost variant) breaks with modern bundlers (Vite/esbuild) that require `isolatedModules` — many setups ban it outright.

**When `enum` is actually worth it**: when you need to iterate the values at runtime (`Object.values(Direction)`), or when the value corresponds to something real and numeric that already exists elsewhere (e.g. a DB column with integers). Outside those specific cases, a union of literals is the most recommended default today.

## Unions and Intersections

- **Union (`|`)**: the value can be **one of several** possible types.
- **Intersection (`&`)**: the value must satisfy **all** the combined types at once.

```ts
// Union: a request's state can only be one of these three string literals
type RequestState = "idle" | "loading" | "error" | "success";

function handleState(state: RequestState) {
  if (state === "loading") { /* ... */ }
  // TypeScript knows, inside this if, that state can't be any other value —
  // autocomplete and exhaustive checking of the possible cases
}

// Intersection: combines the props of two types into one
type WithId = { id: string };
type WithName = { name: string };
type User = WithId & WithName; // { id: string; name: string }
```

**Narrowing**: inside an `if (typeof x === "string")` or an `if ("prop" in obj)`, TypeScript automatically "shrinks" the union type to the option that applies in that block — no manual casting needed.

## Nullable Types

TypeScript doesn't have a separate "optional" type — the absence of a value is modeled with an explicit union against `null` and/or `undefined`.

```ts
function findUser(id: string): User | undefined {
  // returns undefined if not found
}

const user = findUser("123");
user.name;   // ❌ error: 'user' is possibly 'undefined'
user?.name;   // ✅ optional chaining — accesses only if not null/undefined
user!.name;   // ⚠️ non-null assertion — "trust me, it's not null" — no real check, risk of a runtime error if you're wrong
```

With [`strict`](config.md#strict) on, TypeScript forces you to explicitly handle the `null`/`undefined` case before using the value — without `strict`, it lets the direct access through and the error only shows up at runtime.

## `unknown`

The safe type for "I don't know what this is yet" — unlike `any`, which turns off type checking entirely, `unknown` forces you to **verify the type before you can use it**.

```ts
function process(value: unknown) {
  value.toUpperCase();          // ❌ error: 'value' is of type 'unknown'

  if (typeof value === "string") {
    value.toUpperCase();         // ✅ already narrowed to string inside this if
  }
}

function processAny(value: any) {
  value.toUpperCase();          // ✅ compiles anyway, even if value is a number — no real protection
}
```

`unknown` is the right choice when the type genuinely isn't known ahead of time (e.g. an external API's raw response) — `any` should be the exception, not the default.

## `never`

The type of something that **never produces a value** — a function that always throws an exception, or a branch of code that's mathematically impossible to reach.

```ts
function fail(message: string): never {
  throw new Error(message); // never returns, always throws
}

// typical use: exhaustive checking of a union — if a new case is added
// and not handled here, TypeScript flags a compile-time error
type State = "idle" | "loading" | "error";

function handle(state: State) {
  switch (state) {
    case "idle": return "waiting";
    case "loading": return "loading";
    case "error": return "failed";
    default:
      const _exhaustive: never = state; // if a case is missing, this line doesn't compile
      return _exhaustive;
  }
}
```

## Utility Types

Generic types that come with TypeScript for transforming an existing type without rewriting it by hand — avoid duplicating an object's shape for every variant needed (e.g. "all fields optional" or "only these two fields").

```ts
type User = { id: string; name: string; email: string };

type PartialUser = Partial<User>;  // { id?: string; name?: string; email?: string }
type ContactOnly = Pick<User, "name" | "email">; // { name: string; email: string }
type NoEmail = Omit<User, "email">;  // { id: string; name: string }
type UsersById = Record<string, User>; // { [key: string]: User }
```

`Partial` is the most common one in practice: an "update" function usually receives only the fields that change, not the whole object.

## Generics (advanced types)

A "parameterized" type — instead of fixing ahead of time what type a function or structure works with, it's defined once and instantiated with the real type at each use, without losing type checking (unlike using `any`, which turns off checking entirely).

```ts
// without generics: you'd have to repeat the function for each type, or use any and lose safety
function first<T>(list: T[]): T {
  return list[0];
}

first([1, 2, 3]);       // inferred type: number
first(["a", "b"]);      // inferred type: string

// generics in an interface — a Response that can wrap any type of data
interface ApiResponse<T> {
  data: T;
  error: string | null;
}

const response: ApiResponse<User> = { data: user, error: null };
```

**Conditional types** (`T extends U ? X : Y`) and **mapped types** (`{ [K in keyof T]: ... }`, the basis of how the Utility Types above are implemented) are the way to write logic at the type level — used a lot when building libraries, less often in day-to-day application code.

---
Related: [Configuration (tsconfig.json)](config.md), [Compared type system (Python/Swift/Kotlin/Java)](../type-system-comparison.md), [Frontend React](../../frontend-react/README.md) for where this applies in components.
