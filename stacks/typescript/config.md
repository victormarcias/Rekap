# TypeScript — Configuration (`tsconfig.json`)

The most used and most misunderstood options — not exhaustive, `tsconfig.json` has dozens of flags, most already set up by the project's template and rarely need to be touched by hand.

## `strict`

The single most important flag — turns on a bundle of checks (`noImplicitAny`, `strictNullChecks`, `strictFunctionTypes`, among others) at once. Without `strict`, TypeScript lets things slide like a variable with no explicit type (implicit `any`) or assigning `null` to something typed as `string` — the difference between TS actually checking things and TS as decoration.

```json
{ "compilerOptions": { "strict": true } }
```

```ts
// without strict: this compiles with no complaints
function greet(name) { return `Hello, ${name}`; } // name is implicit "any"

// with strict: TypeScript requires the type
function greet(name: string) { return `Hello, ${name}`; } // ✅
```

## `target`

What JS version the code compiles to — defines which modern syntax (optional chaining, `async`/`await`, etc.) gets transformed into something older, and which is left as-is because the target engine already supports it.

```json
{ "compilerOptions": { "target": "ES2020" } }
```

Targeting an old `target` (`ES5`) generates more compatible but heavier JS (more transformed code); a modern one (`ES2020`+) generates less extra code, but assumes a more recent runtime (current browsers, recent Node).

## `module`

The module system of the generated JS — `CommonJS` (`require`/`module.exports`) or `ESNext` (`import`/`export`). Connects directly with [CommonJS vs ESM](../node/runtime.md#commonjs-vs-es-modules): a classic Node backend usually compiles to CommonJS, a modern frontend with Vite/webpack uses ESNext because the bundler needs real ES Modules to do tree shaking.

```json
{ "compilerOptions": { "module": "ESNext" } }
```

## `noEmit`

Tells `tsc` "don't generate any `.js` files, just check types and tell me if there's an error." Used when another tool (Vite, esbuild) is the one actually compiling — `tsc` runs separately (in the editor or in CI) purely as an auditor.

```json
{ "compilerOptions": { "noEmit": true } }
```

## `esModuleInterop`

Fixes a compatibility clash between CommonJS and ES Modules: without this flag, importing an old CommonJS-written package with modern syntax (`import express from 'express'`) can fail or bring in something odd. It's turned on almost always — worth knowing *why* it exists, not just enabling it because "everyone does."

```json
{ "compilerOptions": { "esModuleInterop": true } }
```

```ts
// without esModuleInterop, this might not work as expected with CommonJS packages
import express from 'express';

// the alternative without the flag would be CommonJS's more verbose syntax
import * as express from 'express';
```

## `paths` / `baseUrl`

Import aliases — avoids long chains of `../../../` when importing something from another folder.

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```

```ts
// ❌ without an alias
import { Button } from '../../../components/Button';

// ✅ with the alias configured
import { Button } from '@/components/Button';
```

**The common gotcha**: configuring this in `tsconfig.json` only teaches the alias to TypeScript (for type checking and autocomplete) — the bundler (Vite, webpack) doesn't automatically know about it, the same alias needs to be configured there too, or the real build fails even though `tsc` doesn't complain.

## `outDir` / `rootDir`

Where the compiled JS ends up and where the source code starts from — relevant in a Node backend that compiles to a `dist/` folder and runs that in production (`node dist/index.js`), instead of running `.ts` directly.

```json
{
  "compilerOptions": {
    "rootDir": "./src",
    "outDir": "./dist"
  }
}
```

## `include` / `exclude`

Which files enter the compilation — typically `src/` is included and `node_modules` and test files are excluded, to avoid wasting time compiling/checking code that isn't needed.

```json
{
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.test.ts"]
}
```

---
Related: [Type System](types.md), [CommonJS vs ESM (Node)](../node/runtime.md#commonjs-vs-es-modules).
