# Node.js

Runtime de JavaScript fuera del browser — corre sobre **V8** (el motor de JS de Chrome) + **libuv** (la librería en C que le da el I/O asíncrono). Permite usar JS del lado del servidor.

## Single-threaded, pero no bloqueante (Event Loop)

Node ejecuta el código JS en **un solo hilo** — pero las operaciones de I/O (leer un archivo, una query a la DB, un request de red) no las hace ese hilo: se las delega a libuv, que las maneja por afuera (con el sistema operativo o un thread pool interno), y avisa al hilo principal cuando terminan. Por eso un solo proceso de Node puede atender miles de conexiones simultáneas sin abrir un thread por request — mientras una operación de I/O está "en vuelo", el hilo principal sigue libre para atender otra cosa.

```js
const fs = require('fs');

// ❌ bloqueante: el proceso entero se congela hasta que termina de leer el archivo
const data = fs.readFileSync('archivo.txt');
console.log('esto espera a que termine la lectura');

// ✅ no bloqueante: Node sigue ejecutando código mientras el archivo se lee en background
fs.readFile('archivo.txt', (err, data) => {
  console.log('esto corre recién cuando termina la lectura');
});
console.log('esto se imprime ANTES que el callback de arriba');
```

**La consecuencia práctica**: Node es muy bueno para cargas con mucho I/O (APIs que esperan a una DB, proxies, streaming) y mediocre para trabajo pesado de CPU (cálculos intensivos bloquean el único hilo y congelan todo lo demás mientras tanto — para eso existen los `worker_threads` o separar ese trabajo a otro proceso).

## `process.nextTick` vs Promises vs `setTimeout` vs `setImmediate`

El orden en que corren no es intuitivo — se confunde seguido.

```js
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('nextTick'));

console.log('código síncrono');

// Orden real:
// código síncrono
// nextTick       <- siempre antes que cualquier otra cosa async
// promise        <- microtask, después de nextTick, antes de pasar a la siguiente fase del event loop
// setTimeout      <- depende, pero típicamente antes que setImmediate en el main module
// setImmediate
```

`process.nextTick` y las Promises son **microtasks** — se vacían por completo antes de que el Event Loop avance a la siguiente fase. `setTimeout`/`setImmediate` son **macrotasks** — cada uno vive en una fase distinta del Event Loop (timers vs check), por eso su orden relativo puede variar según el contexto.

## CommonJS vs ES Modules

Node soporta los dos sistemas de módulos, con reglas distintas de carga.

```js
// CommonJS (el histórico, default en archivos .js sin configurar nada)
const fs = require('fs');
module.exports = { greet };

// ES Modules (el estándar de JS moderno — necesita "type": "module" en package.json, o extensión .mjs)
import fs from 'fs';
export function greet() { }
```

Diferencia de fondo: `require()` es **síncrono** (carga y ejecuta el módulo ahí mismo, bloqueando); `import` es parte de la spec de ES Modules y permite *tree shaking* real (ver [Tree Shaking](../../frontend-react/tree-shaking.es.md)) porque el grafo de dependencias se puede analizar estáticamente, sin ejecutar código.

## npm y `package.json`

- **`dependencies`**: lo que la app necesita para correr en producción.
- **`devDependencies`**: herramientas de desarrollo (test runner, linter, bundler) — no viajan a producción.
- **`package-lock.json`**: fija las versiones **exactas** instaladas (incluyendo dependencias transitivas) para que `npm install` sea reproducible entre máquinas — sin este archivo, `^1.2.3` en `package.json` podría resolver a una versión distinta cada vez.

```json
{
  "dependencies": { "express": "^4.18.0" },
  "devDependencies": { "jest": "^29.0.0" }
}
```

`^4.18.0` acepta actualizaciones de minor/patch (`4.x.x`, no `5.0.0`) — semver (`^`/`~`) define cuánto margen de auto-actualización tolera cada dependencia.

## Streams

Procesar datos de a **pedazos (chunks)**, sin cargar todo en memoria de una — la alternativa a leer un archivo entero con `readFileSync` cuando ese archivo pesa gigabytes.

```js
const fs = require('fs');

// ❌ carga el archivo completo en memoria antes de mandar la respuesta
app.get('/download', (req, res) => {
  const data = fs.readFileSync('archivo-grande.csv');
  res.send(data);
});

// ✅ streaming: manda el archivo en pedazos a medida que se leen del disco,
// sin nunca tener el archivo completo en memoria a la vez
app.get('/download', (req, res) => {
  fs.createReadStream('archivo-grande.csv').pipe(res);
});
```

## Buffers

Representación de datos **binarios** en memoria — lo que hay "debajo" de un string cuando se lee un archivo, una imagen, o datos de una conexión de red antes de decodificarlos. Los streams de arriba mueven Buffers internamente, no strings.

```js
const buf = Buffer.from('hola');
buf.toString();       // 'hola'
buf.length;             // 4 — bytes, no caracteres (importa con Unicode multi-byte)
```

---
Relacionado: [Tree Shaking](../../frontend-react/tree-shaking.es.md), [Sintaxis general (Python)](../python/syntax.es.md) para el equivalente de módulos/contextos en otro lenguaje.
