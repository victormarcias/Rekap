# Stacks

A review by language/stack — setup, syntax, and fundamentals — plus the concrete backend architecture implementation in a specific framework (`stacks/<framework>/`, e.g. FastAPI, NestJS). Max 2 levels: never `stacks/<language>/<framework>/`, the framework stands on its own even if it belongs to one particular language.

- [💛 JavaScript](javascript/syntax.md) — variables, arrays, objects, functions, arrow functions, destructuring
- [💚 Node.js](node/runtime.md) — Event Loop, single-threaded non-blocking, CommonJS vs ESM, streams, buffers
- [🐍 Python](python/) — setup, syntax, Python 2 vs 3, GIL, mutability, OOP, decorators, generators
- [🔷 TypeScript](typescript/) — function typing, unions, utility types, generics, `tsconfig.json` configuration
- [⚡ FastAPI](fastapi/) — concrete backend architecture implementation in Python (endpoints, sync/async, auth, testing)
- [🐈 NestJS](nestjs/) — concrete backend architecture implementation in TypeScript/Node (modules, DI, Pipes/Guards/Interceptors/Exception Filters)
- [📱 React Native](react-native/fundamentals.md) — core components, styling, Expo vs bare, navigation, the bridge
- [🔗 n8n](n8n/) — node-based automation, self-hostable, AI Agent node

See also: [Compared type system](type-system-comparison.md) (TypeScript/Python/Swift/Kotlin/Java side by side), [Overview: layers of a stack](stack-layers.md) (runtime vs framework vs ORM vs frontend — why "Node vs FastAPI vs Next.js" isn't an even comparison).
