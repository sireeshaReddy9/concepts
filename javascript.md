# JavaScript 

## Basics
JavaScript is both an **interpreted and compiled** language.

**Memory sizing:** 1 GB = 10⁹ bytes | 1 byte = 8 bits

---

## Data Types

| Category | Types |
|---|---|
| Primitive | Integer, Real number, Character, Boolean (yes/no), String, BigInt |
| Built-in via libraries | Audio, Video |

### Key Notes
- **Variables** are stored in **stack memory**; their **values** live in **heap memory**
- `typeof` — checks the data type of a value
- `NaN` — occurs when trying to convert a string to a number; use `parseInt()` instead
  - `parseInt(" 1 2 3")` → `1` (parses until it hits a non-numeric character)
- **BigInt** — for large integers only; append `n` → e.g. `100n`
- **Falsy values:** `[]`, `""`, `0`, `NaN`, `null` → evaluate to `false`; everything else is truthy

---

## Functions

### Four Types

```js
// 1. Function Declaration
function name() {}

// 2. Function Expression
let add = function() {};

// 3. Arrow Function
let res = (a, b) => a + b;

// 4. Immediately Invoked Function Expression (IIFE)
(function add() {
  console.log(2 + 3);
})();
```

---

## `let` vs `var`

| Feature | `var` | `let` |
|---|---|---|
| Hoisting |  Hoisted, initialized as `undefined` |  Hoisted, but **not** initialized |
| TDZ (Temporal Dead Zone) |  No TDZ |  Has TDZ — reference error if accessed before declaration |
| Block scope |  Breaches block scope |  Respects block scope |

> **TDZ** = the region from where a variable is declared up until it is initialized.
- `var` can be redeclared, which causes unexpected behaviour
- `var` does not respect block scope, which makes it hard to debug for programmers.
---

## Functions as Objects (First-Class Citizens)

Functions can be stored in variables, passed to other functions, and returned from functions.

### Key Properties
- `function.name` — name of the function
- `function.length` — number of parameters

### Methods: `call`, `apply`, `bind`

```js
function exmp(fun) {   // Higher-order function
  fun(1, 2);
}

function add(a, b) {
  console.log(a + b);  // 'add' is a callback function
}

exmp(add);
```

---

## Closures

If an inner function needs a variable from an outer function, that variable **persists** even after the outer function has finished executing — forming a **closure**.

```js
function outer() {
  let x = 10;
  return function inner() {
    console.log(x); // x is still accessible via closure
  };
}
```

---

## Callbacks

A function that is **passed as an argument** to another function and called inside it.

---

## Arrays

### Iterating
```js
for (let a of arr) { }
```

### Common Methods

```js
let arr = [1, 2, 3, 4, 5];

arr.push(6);              // Add to end
arr.unshift(0);           // Add to start
arr.pop();                // Remove & return last element
arr.shift();              // Remove first element
arr.splice(2, 2, 60, 70); // [1, 2, 60, 70, 5] — (startIndex, deleteCount, ...items)
arr.indexOf(0);           // First occurrence (-1 if not found)
arr.lastIndexOf(3);       // Last occurrence
arr.includes(5);          // true / false
```

### Shallow Copy vs Deep Copy

```js
// Shallow copy — both variables point to the same reference
let ar2 = ar1.slice();
let ar2 = [...ar1];

// Deep copy — fully independent copy including nested contents
let ar2 = structuredClone(ar1);
```

---

## String Interning

JavaScript ensures only **one unique copy** of each string literal exists in memory. This makes **string comparisons faster** and reduces memory usage.
