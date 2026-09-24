# Stacks

Repaso por lenguaje/stack — setup, sintaxis y fundamentos — más la implementación concreta de arquitectura backend en un framework puntual (`stacks/<framework>/`, ej. FastAPI, NestJS). Máximo 2 niveles: nunca `stacks/<lenguaje>/<framework>/`, el framework va suelto aunque sea propio de un lenguaje en particular.

- [💛 JavaScript](javascript/syntax.es.md) — variables, arrays, objects, functions, arrow functions, destructuring
- [💚 Node.js](node/runtime.es.md) — Event Loop, single-threaded no bloqueante, CommonJS vs ESM, streams, buffers
- [🐍 Python](python/) — setup, sintaxis, Python 2 vs 3, GIL, mutabilidad, OOP, decorators, generators
- [🔷 TypeScript](typescript/) — tipado de funciones, unions, utility types, generics, configuración de `tsconfig.json`
- [⚡ FastAPI](fastapi/) — implementación concreta de arquitectura backend en Python (endpoints, sync/async, auth, testing)
- [🐈 NestJS](nestjs/) — implementación concreta de arquitectura backend en TypeScript/Node (módulos, DI, Pipes/Guards/Interceptors/Exception Filters)
- [📱 React Native](react-native/fundamentals.es.md) — componentes core, styling, Expo vs bare, navegación, el bridge
- [🔗 n8n](n8n/) — automatización node-based, self-hosteable, nodo AI Agent

Ver también: [Sistema de tipos comparado](type-system-comparison.es.md) (TypeScript/Python/Swift/Kotlin/Java lado a lado), [Panorama: capas de un stack](stack-layers.es.md) (runtime vs framework vs ORM vs frontend — por qué "Node vs FastAPI vs Next.js" no es una comparación pareja).
