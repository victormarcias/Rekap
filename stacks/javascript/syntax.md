# JavaScript

## Variables — `var` vs `let` vs `const`

- **`var`**: function-scoped (not block-scoped), can be redeclared, suffers full *hoisting* — avoid it in modern code.
- **`let`**: block-scoped (`{}`), can be reassigned.
- **`const`**: block-scoped, the reference can't be reassigned — but if it's an object or array, its **content can still be mutated**.

```js
const arr = [1, 2, 3];
arr.push(4);   // ✅ valid — const protects the reference, not the content
arr = [];      // ❌ TypeError: Assignment to constant variable
```

## Arrays

```js
const nums = [1, 2, 3, 4];

nums.map(n => n * 2);            // [2, 4, 6, 8] — transforms each element
nums.filter(n => n % 2 === 0);   // [2, 4] — keeps the ones matching the condition
nums.reduce((acc, n) => acc + n, 0); // 10 — accumulates everything into a single value

const combined = [...nums, 5, 6]; // spread: a new array, without mutating the original
```

## Objects

```js
const name = 'Ana';
const user = { name, age: 30 };   // shorthand: { name } instead of { name: name }

const updated = { ...user, age: 31 }; // spread: shallow copy + override of one field

user.address?.city;               // optional chaining: doesn't blow up if address is undefined/null
user.nickname ?? 'No nickname';     // nullish coalescing: uses the default only if it's null/undefined (not if it's '' or 0)
```

## Functions

```js
function greet(name, greeting = 'Hello') { // default param
  return `${greeting}, ${name}`;
}

function sumAll(...nums) {  // rest params: collects the rest of the arguments into an array
  return nums.reduce((a, b) => a + b, 0);
}

sumAll(1, 2, 3); // 6
```

## Arrow Functions

Shorter syntax, and one fundamental difference from `function`: they don't have their own `this` — they inherit it from the context where they were defined (*lexical this*), instead of depending on how they're called.

```js
const square = x => x * x;      // a single parameter, no parentheses required
const add = (a, b) => a + b;    // implicit return on a single line

class Counter {
  count = 0;
  incrementBad = function() { this.count++; }; // ❌ `this` depends on who calls the function
  incrementGood = () => { this.count++; };      // ✅ `this` is always the Counter instance
}
```

## Destructuring

```js
const [first, second] = [1, 2, 3]; // array destructuring — first=1, second=2

const { name, age = 18 } = user;   // object destructuring, with a default if the field doesn't exist
const { name: userName } = user;   // renaming while destructuring

function greet({ name, greeting = 'Hello' }) { // destructuring directly in the parameters
  return `${greeting}, ${name}`;
}
```
