# TypeScript — Configuración (`tsconfig.json`)

Las opciones que más se usan y más se malentienden — no es exhaustivo, `tsconfig.json` tiene decenas de flags, la mayoría vienen ya armados por el template del proyecto y rara vez hace falta tocarlos a mano.

## `strict`

El flag más importante de todos — prende un combo de chequeos (`noImplicitAny`, `strictNullChecks`, `strictFunctionTypes`, entre otros) de una sola vez. Sin `strict`, TypeScript deja pasar cosas como una variable sin tipo explícito (`any` implícito) o asignar `null` a algo tipado como `string` — la diferencia entre TS chequeando de verdad y TS de adorno.

```json
{ "compilerOptions": { "strict": true } }
```

```ts
// sin strict: esto compila sin quejarse
function saludar(nombre) { return `Hola, ${nombre}`; } // nombre es "any" implícito

// con strict: TypeScript exige el tipo
function saludar(nombre: string) { return `Hola, ${nombre}`; } // ✅
```

## `target`

A qué versión de JS se compila el código — define qué sintaxis moderna (optional chaining, `async`/`await`, etc.) se transforma a algo más viejo, y cuál se deja tal cual porque el motor destino ya la soporta.

```json
{ "compilerOptions": { "target": "ES2020" } }
```

Apuntar a un `target` viejo (`ES5`) genera JS más compatible pero más pesado (más código transformado); uno moderno (`ES2020`+) genera menos código de más, pero asume un runtime más reciente (navegadores actuales, Node reciente).

## `module`

El sistema de módulos del JS que se genera — `CommonJS` (`require`/`module.exports`) o `ESNext` (`import`/`export`). Conecta directo con [CommonJS vs ESM](../node/runtime.md#commonjs-vs-es-modules): un backend Node clásico suele compilar a CommonJS, un frontend moderno con Vite/webpack usa ESNext porque el bundler necesita ES Modules reales para hacer tree shaking.

```json
{ "compilerOptions": { "module": "ESNext" } }
```

## `noEmit`

Le dice a `tsc` "no generes ningún archivo `.js`, solo chequeá tipos y avisame si hay error". Se usa cuando otra herramienta (Vite, esbuild) es la que realmente compila — `tsc` corre aparte (en el editor o en CI) únicamente como auditor.

```json
{ "compilerOptions": { "noEmit": true } }
```

## `esModuleInterop`

Arregla un choque de compatibilidad entre CommonJS y ES Modules: sin este flag, importar un paquete viejo escrito en CommonJS con sintaxis moderna (`import express from 'express'`) puede fallar o traer algo raro. Se activa casi siempre — vale la pena saber *por qué* existe, no solo prenderlo porque "todos lo hacen".

```json
{ "compilerOptions": { "esModuleInterop": true } }
```

```ts
// sin esModuleInterop, esto puede no funcionar como se espera con paquetes CommonJS
import express from 'express';

// la alternativa sin el flag sería la sintaxis más verbosa de CommonJS
import * as express from 'express';
```

## `paths` / `baseUrl`

Alias de imports — evita cadenas largas de `../../../` al importar algo de otra carpeta.

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```

```ts
// ❌ sin alias
import { Button } from '../../../components/Button';

// ✅ con el alias configurado
import { Button } from '@/components/Button';
```

**El gotcha común**: configurar esto en `tsconfig.json` solo le enseña el alias a TypeScript (para el chequeo de tipos y el autocompletado) — el bundler (Vite, webpack) no lo sabe automáticamente, hace falta configurar el mismo alias ahí también, o el build real falla aunque `tsc` no se queje.

## `outDir` / `rootDir`

Dónde termina el JS compilado y desde dónde arranca el código fuente — relevante en un backend Node que compila a una carpeta `dist/` y corre eso en producción (`node dist/index.js`), en vez de correr `.ts` directo.

```json
{
  "compilerOptions": {
    "rootDir": "./src",
    "outDir": "./dist"
  }
}
```

## `include` / `exclude`

Qué archivos entran a la compilación — típicamente se incluye `src/` y se excluye `node_modules` y los archivos de test, para no perder tiempo compilando/chequeando código que no hace falta.

```json
{
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.test.ts"]
}
```

---
Relacionado: [Sistema de tipos](tipos.md), [CommonJS vs ESM (Node)](../node/runtime.md#commonjs-vs-es-modules).
