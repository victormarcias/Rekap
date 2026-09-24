# Node.js

JavaScript runtime outside the browser — runs on **V8** (Chrome's JS engine) + **libuv** (the C library that gives it async I/O). Lets you use JS on the server side.

## Single-threaded, but non-blocking (Event Loop)

Node runs JS code on **a single thread** — but I/O operations (reading a file, a DB query, a network request) aren't done by that thread: they're delegated to libuv, which handles them externally (with the operating system or an internal thread pool), and notifies the main thread when they finish. That's why a single Node process can handle thousands of simultaneous connections without opening a thread per request — while an I/O operation is "in flight," the main thread stays free to handle something else.

```js
const fs = require('fs');

// ❌ blocking: the entire process freezes until it finishes reading the file
const data = fs.readFileSync('file.txt');
console.log('this waits for the read to finish');

// ✅ non-blocking: Node keeps executing code while the file is read in the background
fs.readFile('file.txt', (err, data) => {
  console.log('this only runs once the read finishes');
});
console.log('this prints BEFORE the callback above');
```

**The practical consequence**: Node is very good for I/O-heavy loads (APIs waiting on a DB, proxies, streaming) and mediocre for heavy CPU work (intensive calculations block the single thread and freeze everything else in the meantime — that's what `worker_threads` or offloading that work to another process are for).

## `process.nextTick` vs Promises vs `setTimeout` vs `setImmediate`

The order they run in isn't intuitive — often confused.

```js
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('nextTick'));

console.log('synchronous code');

// Real order:
// synchronous code
// nextTick       <- always before anything else async
// promise        <- microtask, after nextTick, before moving to the next event loop phase
// setTimeout      <- depends, but typically before setImmediate in the main module
// setImmediate
```

`process.nextTick` and Promises are **microtasks** — they're fully drained before the Event Loop advances to the next phase. `setTimeout`/`setImmediate` are **macrotasks** — each lives in a different Event Loop phase (timers vs check), which is why their relative order can vary depending on context.

## CommonJS vs ES Modules

Node supports both module systems, with different loading rules.

```js
// CommonJS (the historical one, default in .js files with no config)
const fs = require('fs');
module.exports = { greet };

// ES Modules (the modern JS standard — needs "type": "module" in package.json, or a .mjs extension)
import fs from 'fs';
export function greet() { }
```

The underlying difference: `require()` is **synchronous** (loads and executes the module right there, blocking); `import` is part of the ES Modules spec and enables real *tree shaking* (see [Tree Shaking](../../frontend-react/tree-shaking.md)) because the dependency graph can be statically analyzed, without executing code.

## npm and `package.json`

- **`dependencies`**: what the app needs to run in production.
- **`devDependencies`**: development tools (test runner, linter, bundler) — don't ship to production.
- **`package-lock.json`**: pins the **exact** installed versions (including transitive dependencies) so `npm install` is reproducible across machines — without this file, `^1.2.3` in `package.json` could resolve to a different version each time.

```json
{
  "dependencies": { "express": "^4.18.0" },
  "devDependencies": { "jest": "^29.0.0" }
}
```

`^4.18.0` accepts minor/patch updates (`4.x.x`, not `5.0.0`) — semver (`^`/`~`) defines how much auto-update margin each dependency tolerates.

## Streams

Processing data in **chunks**, without loading it all into memory at once — the alternative to reading an entire file with `readFileSync` when that file is gigabytes in size.

```js
const fs = require('fs');

// ❌ loads the entire file into memory before sending the response
app.get('/download', (req, res) => {
  const data = fs.readFileSync('big-file.csv');
  res.send(data);
});

// ✅ streaming: sends the file in chunks as they're read from disk,
// never holding the entire file in memory at once
app.get('/download', (req, res) => {
  fs.createReadStream('big-file.csv').pipe(res);
});
```

## Buffers

**Binary** data representation in memory — what's "underneath" a string when reading a file, an image, or data from a network connection before decoding it. The streams above move Buffers internally, not strings.

```js
const buf = Buffer.from('hello');
buf.toString();       // 'hello'
buf.length;             // 5 — bytes, not characters (matters with multi-byte Unicode)
```

---
Related: [Tree Shaking](../../frontend-react/tree-shaking.md), [General Syntax (Python)](../python/syntax.md) for the module/context equivalent in another language.
