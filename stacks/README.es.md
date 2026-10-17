# Stacks

Repaso por lenguaje/stack — setup, sintaxis y fundamentos — más la implementación concreta de arquitectura backend en un framework puntual (`stacks/<framework>/`, ej. FastAPI, NestJS). Máximo 2 niveles: nunca `stacks/<lenguaje>/<framework>/`, el framework va suelto aunque sea propio de un lenguaje en particular.

- [⚡ FastAPI](fastapi/) — implementación concreta de arquitectura backend en Python (endpoints, sync/async, auth, testing)
- [💛 JavaScript](javascript/README.es.md) — variables, arrays, objects, functions, arrow functions, destructuring
- [🐈 NestJS](nestjs/) — implementación concreta de arquitectura backend en TypeScript/Node (módulos, DI, Pipes/Guards/Interceptors/Exception Filters)
- [💚 Node.js](node/README.es.md) — Event Loop, single-threaded no bloqueante, CommonJS vs ESM, streams, buffers
- [🐍 Python](python/) — setup, sintaxis, Python 2 vs 3, GIL, mutabilidad, OOP, decorators, generators
- [📱 React Native](react-native/README.es.md) — componentes core, styling, Expo vs bare, navegación, el bridge
- [🔷 TypeScript](typescript/) — tipado de funciones, unions, utility types, generics, configuración de `tsconfig.json`

Ver también: [Sistema de tipos comparado](type-system-comparison.es.md) (TypeScript/Python/Swift/Kotlin/Java lado a lado), [Panorama: capas de un stack](stack-layers.es.md) (runtime vs framework vs ORM vs frontend — por qué "Node vs FastAPI vs Next.js" no es una comparación pareja).
