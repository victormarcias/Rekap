# JavaScript

## Variables — `var` vs `let` vs `const`

- **`var`**: scope de función (no de bloque), se puede redeclarar, sufre *hoisting* completo — evitarlo en código moderno.
- **`let`**: scope de bloque (`{}`), se puede reasignar.
- **`const`**: scope de bloque, no se puede reasignar la referencia — pero si es un objeto o array, su **contenido sí se puede mutar**.

```js
const arr = [1, 2, 3];
arr.push(4);   // ✅ válido — const protege la referencia, no el contenido
arr = [];      // ❌ TypeError: Assignment to constant variable
```

## Arrays

```js
const nums = [1, 2, 3, 4];

nums.map(n => n * 2);            // [2, 4, 6, 8] — transforma cada elemento
nums.filter(n => n % 2 === 0);   // [2, 4] — se queda con los que cumplen la condición
nums.reduce((acc, n) => acc + n, 0); // 10 — acumula todo en un solo valor

const combined = [...nums, 5, 6]; // spread: array nuevo, sin mutar el original
```

## Objects

```js
const name = 'Ana';
const user = { name, age: 30 };   // shorthand: { name } en vez de { name: name }

const updated = { ...user, age: 31 }; // spread: copia superficial + override de un campo

user.address?.city;               // optional chaining: no explota si address es undefined/null
user.nickname ?? 'Sin apodo';     // nullish coalescing: usa el default solo si es null/undefined (no si es '' o 0)
```

## Functions

```js
function greet(name, greeting = 'Hola') { // default param
  return `${greeting}, ${name}`;
}

function sumAll(...nums) {  // rest params: junta el resto de argumentos en un array
  return nums.reduce((a, b) => a + b, 0);
}

sumAll(1, 2, 3); // 6
```

## Arrow Functions

Sintaxis más corta, y una diferencia de fondo con `function`: no tienen su propio `this` — heredan el del contexto donde se definieron (*lexical this*), en vez de depender de cómo se las llama.

```js
const square = x => x * x;      // un solo parámetro, sin paréntesis obligatorios
const add = (a, b) => a + b;    // return implícito en una sola línea

class Counter {
  count = 0;
  incrementBad = function() { this.count++; }; // ❌ `this` depende de quién llama a la función
  incrementGood = () => { this.count++; };      // ✅ `this` siempre es la instancia de Counter
}
```

## Destructuring

```js
const [first, second] = [1, 2, 3]; // array destructuring — first=1, second=2

const { name, age = 18 } = user;   // object destructuring, con default si el campo no existe
const { name: userName } = user;   // renombrar al desestructurar

function greet({ name, greeting = 'Hola' }) { // destructuring directo en los parámetros
  return `${greeting}, ${name}`;
}
```
