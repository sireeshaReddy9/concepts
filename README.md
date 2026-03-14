# JavaScript Notes

## Basics
JavaScript is both an **interpreted and compiled** language.

**Memory sizing:** 1 GB = 10⁹ bytes | 1 byte = 8 bits

---

## Data Types

JavaScript has **8 data types** — 7 primitives and 1 non-primitive.

### Primitive Types
| Type | Example | Notes |
|---|---|---|
| `Number` | `42`, `3.14` | Integers and floats are the same type |
| `String` | `"hello"` | Sequence of characters |
| `Boolean` | `true`, `false` | Yes/no values |
| `Undefined` | `undefined` | Variable declared but not assigned |
| `Null` | `null` | Intentional absence of value |
| `BigInt` | `100n` | For integers beyond `Number.MAX_SAFE_INTEGER` |
| `Symbol` | `Symbol("id")` | Unique identifiers |

### Non-Primitive
| Type | Example | Notes |
|---|---|---|
| `Object` | `{}`, `[]`, functions | Stored by reference in heap memory |

### Key Notes
- **Variables** are stored in **stack memory**; their **values** live in **heap memory**
- `typeof` — checks the data type of a value
- `NaN` — occurs when trying to convert a string to a number; use `parseInt()` instead
  - `parseInt(" 1 2 3")` → `1` (parses until it hits a non-numeric character)
- **BigInt** — for large integers only; append `n` → e.g. `100n`

```js
typeof 42;           // "number"
typeof "hello";      // "string"
typeof true;         // "boolean"
typeof undefined;    // "undefined"
typeof null;         // "object"  ← known JS quirk
typeof {};           // "object"
typeof [];           // "object"
typeof function(){}; // "function"
```

---

## Null vs Undefined

| | `null` | `undefined` |
|---|---|---|
| Meaning | Intentional empty value | Variable declared but never assigned |
| Set by | Developer explicitly | JavaScript engine (or developer) |
| `typeof` | `"object"`  | `"undefined"` |

```js
let a;         // undefined — JS sets this automatically
let b = null;  // null — you explicitly said "no value"

// Checking for undefined — prefer strict check
if (value === undefined) { }  // explicit and clear
if (!value) { }               // also catches 0, "", null, false
```
---

## Truthy and Falsy Values

**Falsy** values (evaluate to `false` in a boolean context):

```js
false
0
""        // empty string
null
undefined
NaN
```

**Everything else is truthy**, including:

```js
"0"       // non-empty string
[]        // empty array
{}        // empty object
-1        // any non-zero number
```

---

## `let` vs `var` vs `const`

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function-scoped | Block-scoped | Block-scoped |
| Hoisting |  Hoisted as `undefined` |  Hoisted, uninitialized (TDZ) | Hoisted, uninitialized (TDZ) |
| TDZ |  None |  Yes |  Yes |
| Re-declarable | Yes |  No |  No |
| Re-assignable |  Yes |  Yes |  No |

> **TDZ (Temporal Dead Zone)** = the region from where a variable is hoisted up until it is initialized. Accessing it in this zone throws a `ReferenceError`.

### Why You Must NOT Use `var`

1. **Leaks out of blocks** — `var` is function-scoped, not block-scoped, so it comes out of `if`, `for`, etc.
2. ** bugs** — accessing a `var` before its declaration gives `undefined` instead of an error, hiding bugs.
3. **Re-declaration allowed** — accidentally re-declaring the same variable causes no error, making bugs hard to trace for programmer.

```js
// var leaks out of block scope
if (true) {
  var x = 10;
}
console.log(x); // 10 — leaks out

// let stays inside block
if (true) {
  let y = 10;
}
console.log(y); // ReferenceError 
```

### Why Global Variables Are Bad

- Any part of the codebase can accidentally overwrite them
- Makes code hard to debug 
- Causes naming conflicts in large codebases or when using third-party libraries
- Pollutes the `window` object in browsers
---

## Scopes in JavaScript

**Scope** determines where a variable is accessible.

### Types of Scope

```js
// 1. Global Scope — accessible everywhere
const globalVar = "I'm global";

// 2. Function Scope — accessible only inside the function
function myFunc() {
  const funcVar = "I'm function-scoped";
}

// 3. Block Scope — accessible only inside the block {}
if (true) {
  let blockVar = "I'm block-scoped";
  const alsoBlock = "me too";
}

// 4. Lexical (Closure) Scope — inner functions access outer variables
function outer() {
  const outerVar = "outer";
  function inner() {
    console.log(outerVar); //  accessible via lexical scope
  }
}
```

### Scope Chain
When a variable is not found in the current scope, JavaScript looks up the **scope chain** until it reaches the global scope. If not found anywhere, it throws a `ReferenceError`.

---

## Functions

### Different Ways of Declaring a Function

```js
// 1. Function Declaration — hoisted fully
function add(a, b) {
  return a + b;
}

// 2. Function Expression — NOT hoisted
const add = function(a, b) {
  return a + b;
};

// 3. Arrow Function — no own `this` takes from surrounding scope
const add = (a, b) => a + b;

// 4. Named Function Expression
const add = function addNumbers(a, b) {
  return a + b;
};

// 5. IIFE (Immediately Invoked Function Expression)
(function() {
  console.log("Runs immediately");
})();

// 6. Method (function inside an object)
const obj = {
  greet: function() { console.log("Hello"); },
  greetArrow: () => console.log("Hello")
};
```

### Function Hoisting

```js
// Function Declarations are fully hoisted — can be called before declaration
sayHello(); 
function sayHello() {
  console.log("Hello");
}

// Function Expressions are NOT hoisted
greet(); // TypeError: greet is not a function
const greet = function() {
  console.log("Hi");
};
```

### What Happens Without a Return Statement

If a function has no `return` statement, it **returns `undefined`** automatically.

### Named vs Anonymous Functions

``` js
// Named function — has a name property, better stack traces
function multiply(a, b) { return a * b; }
const multiply = function multiplyNums(a, b) { return a * b; };

// Anonymous function — no name, harder to debug
const multiply = function(a, b) { return a * b; };
const multiply = (a, b) => a * b; // arrow functions are technically anonymous
```

### Arrow Functions vs Regular Functions

| Feature | Regular Function | Arrow Function |
|---|---|---|
| `this` binding | Has its own `this` | Inherits `this` from surrounding scope |
| `arguments` object |  Available |  Not available |
| Use as constructor |  Yes (`new`) |  No |
| Hoisting | Declaration is hoisted | Not hoisted |

``` js
const obj = {
  name: "Alice",
  regularFn: function() {
    console.log(this.name); // "Alice" — `this` is obj
  },
  arrowFn: () => {
    console.log(this.name); // undefined — `this` is outer scope
  }
};
```
---

## Pass by Value vs Pass by Reference

``` js
// PRIMITIVES — passed by value (a copy is made)
let a = 10;
let b = a;
b = 20;
console.log(a); // 10 — unchanged

// OBJECTS/ARRAYS — passed by reference (same memory address)
let arr1 = [1, 2, 3];
let arr2 = arr1;
arr2.push(4);
console.log(arr1); // [1, 2, 3, 4] — affected!

// To avoid this, use a deep copy
let arr3 = structuredClone(arr1); // independent copy
arr3.push(5);
console.log(arr1); // [1, 2, 3, 4] — unchanged

// Same with objects passed to functions
function updateName(person) {
  person.name = "Bob"; // mutates the original object
}
const user = { name: "Alice" };
updateName(user);
console.log(user.name); // "Bob" — original was mutated
```

---

## Closures

If an inner function needs a variable from an outer function, that variable **persists** even after the outer function has finished executing — forming a **closure**.

``` js
function outer() {
  let count = 0;
   function inner() {
    count++;
    console.log(count);
  };
return inner;
}

const increment = outer();
increment(); // 1
increment(); // 2
increment(); // 3 — count persists across calls
```
---

## `===` vs `==`

| Operator | Name | Behavior |
|---|---|---|
| `===` | Strict equality | Checks **value AND type** — no type coercion |
| `==` | Loose equality | Checks value only — **converts types** first |

``` js
0 == "0";           // true  — type coercion
0 === "0";          // false — different types

null == undefined;  // true  
null === undefined; // false 

// Always use === unless you have a specific reason not to
```

---

## For Loops

```js
const arr = [10, 20, 30];

// 1. Classic for loop — best for index-based control
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}

// 2. for...of — iterates over VALUES (arrays, strings, etc.)
for (let value of arr) {
  console.log(value);
}

// 3. for...in — iterates over KEYS/INDICES (use for objects)
const obj = { a: 1, b: 2 };
for (let key in obj) {
  console.log(key, obj[key]); // "a" 1, "b" 2
}

// 4. forEach — array method, no break/continue
arr.forEach((value, index) => {
  console.log(index, value);
});

// 5. while loop — when you don't know iterations in advance
let i = 0;
while (i < arr.length) {
  console.log(arr[i]);
  i++;
}
```

---

## forEach vs map/filter / reduce

| Method | Use When | Returns |
|---|---|---|
| `forEach` | You only need **side effects** (logging, DOM updates) | `undefined` |
| `map` | You need a **new transformed array** based on condition | New array (same length) |
| `filter` | You need a **subset** of the array based on the condition| New array (shorter or equal) |
| `reduce` | You need to **combine** all elements into one value | Single value |

``` js
const nums = [1, 2, 3, 4, 5];

// forEach — just do something, don't collect a result
nums.forEach(n => console.log(n));

// map — transform each element
const doubled = nums.map(n => n * 2); // [2, 4, 6, 8, 10]

// filter — keep only matching elements
const evens = nums.filter(n => n % 2 === 0); // [2, 4]

// reduce — collapse to a single value
const total = nums.reduce((acc, n) => acc + n, 0); // 15
```

---

## Immutable vs Mutable Methods

- **Mutable** methods modify the **original** array/object directly
- **Immutable** methods return a **new** value, leaving the original unchanged

> As a rule: prefer immutable operations — they make code easier to reason about and prevent bugs from accidental mutation.

---

## Array Utility Methods

### Basics

``` js
const arr = [1, 2, 3, 4, 5];

// pop — removes & returns last element | MUTABLE
arr.pop(); // returns 5; arr is now [1, 2, 3, 4]

// push — adds element(s) to end | MUTABLE
arr.push(6); // arr is now [1, 2, 3, 4, 5, 6]

// concat — merges arrays into new array | IMMUTABLE
const arr2 = arr.concat([6, 7]); // [1, 2, 3, 4, 5, 6, 7]

// slice(start, end) — extracts portion | IMMUTABLE
const sliced = arr.slice(1, 3); // [2, 3] — end index is exclusive

// splice(start, deleteCount, ...items) — adds/removes in place | MUTABLE
arr.splice(2, 1, 99); // replaces 1 element at index 2 with 99

// join — converts array to string | IMMUTABLE
arr.join(", "); // "1, 2, 3, 4, 5"

// flat — flattens nested arrays | IMMUTABLE
[1, [2, [3, [4]]]].flat();           // [1, 2, [3, [4]]] — one level
[1, [2, [3, [4]]]].flat(Infinity);   // [1, 2, 3, 4] — fully flat
```

### Finding

``` js
const arr = [10, 20, 30, 20, 40];

// find — returns FIRST element matching condition | IMMUTABLE
arr.find(n => n > 15); // 20

// indexOf — first index of value (-1 if not found) | IMMUTABLE
arr.indexOf(20); // 1

// includes — checks if value exists | IMMUTABLE
arr.includes(30); // true

// findIndex — index of first match (-1 if not found) | IMMUTABLE
arr.findIndex(n => n > 15); // 1
```

### Higher Order Functions

``` js
const nums = [1, 2, 3, 4, 5];

// forEach — side effects only, returns undefined | IMMUTABLE
nums.forEach((n, i) => console.log(i, n));

// filter — keep matching elements | IMMUTABLE
const evens = nums.filter(n => n % 2 === 0); // [2, 4]

// map — transform each element | IMMUTABLE
const squares = nums.map(n => n * n); // [1, 4, 9, 16, 25]

// reduce — collapse to one value | IMMUTABLE
const sum = nums.reduce((acc, n) => acc + n, 0); // 15

// sort — sorts in place | MUTABLE 
const sorted = [3, 1, 4, 1, 5].sort((a, b) => a - b); // [1, 1, 3, 4, 5]
//  sort mutates the original — clone first if you want to preserve it
const safeSorted = [...nums].sort((a, b) => a - b);
```

### Array Methods Chaining

``` js
const people = [
  { name: "Alice", age: 25, score: 80 },
  { name: "Bob",   age: 17, score: 95 },
  { name: "Carol", age: 30, score: 60 },
  { name: "Dave",  age: 22, score: 90 },
];

// Get names of adults with score > 70, sorted alphabetically
const result = people
  .filter(p => p.age >= 18)   // adults only
  .filter(p => p.score > 70)  // high scorers
  .map(p => p.name)           // extract names
  .sort();                    // alphabetical

console.log(result); // ["Alice", "Dave"]
```

---

## String Utility Methods

> All string methods are **IMMUTABLE** — strings in JavaScript cannot be changed in place; every method returns a new string.

``` js
const str = "  Hello, World!  ";

str.length;                        // 17
str.toUpperCase();                 // "  HELLO, WORLD!  "
str.toLowerCase();                 // "  hello, world!  "
str.trim();                        // "Hello, World!"
str.trimStart();                   // "Hello, World!  "
str.trimEnd();                     // "  Hello, World!"

str.includes("World");             // true
str.startsWith("  Hello");         // true
str.endsWith("!  ");               // true

str.indexOf("o");                  // 4
str.lastIndexOf("o");              // 8

str.slice(2, 7);                   // "Hello"
str.replace("World", "JS");        // "  Hello, JS!  "
str.replaceAll("l", "r");          // "  Herro, Worrd!  "

str.split(", ");                   // ["  Hello", "World!  "]
"abc".repeat(3);                   // "abcabcabc"
"5".padStart(4, "0");              // "0005"
"5".padEnd(4, "0");                // "5000"
```

### Template Literals

``` js
const name = "Alice";
const age = 25;

// Embed expressions with ${}
const greeting = `Hello, ${name}! You are ${age} years old.`;

// Multi-line strings 
const multiLine = `Line 1
Line 2
Line 3`;

// Expression evaluation inside ${}
const result = `2 + 2 = ${2 + 2}`;
```

### String Interning
JavaScript ensures only **one unique copy** of each string literal exists in memory. This makes **string comparisons faster** and reduces memory usage.

---

## Object Utility Methods

> Object methods that return new objects are **IMMUTABLE**; direct property assignment is **MUTABLE**.

``` js
const user = { name: "Alice", age: 25, city: "NYC" };

// Keys, values, entries | IMMUTABLE
Object.keys(user);    // ["name", "age", "city"]
Object.values(user);  // ["Alice", 25, "NYC"]
Object.entries(user); // [["name","Alice"], ["age",25], ["city","NYC"]]

// Merge objects | IMMUTABLE — returns new object
const updated = Object.assign({}, user, { age: 26 });
const updated2 = { ...user, age: 26 }; // preferred — spread syntax

// Freeze — prevents any modification | PREVENTS mutation
const frozen = Object.freeze(user);
frozen.name = "Bob"; // silently fails (throws in strict mode)

// Check property existence
"name" in user;              // true
user.hasOwnProperty("age");  // true

// Delete a property | MUTABLE
delete user.city;

// Convert entries back to object | IMMUTABLE
const entries = [["a", 1], ["b", 2]];
Object.fromEntries(entries); // { a: 1, b: 2 }
```

---

## Spread Operator

``` js
// Arrays — expand elements
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]
const copy = [...arr1];              // shallow copy

// Objects — expand properties
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3 };
const merged = { ...obj1, ...obj2 };  // { a: 1, b: 2, c: 3 }
const overridden = { ...obj1, b: 99 }; // { a: 1, b: 99 }

// Function arguments
function sum(a, b, c) { return a + b + c; }
const nums = [1, 2, 3];
sum(...nums); // 6
```

---

## Default Parameters

``` js
// With default parameters — cleaner
function greet(name = "Guest") {
  console.log(`Hello, ${name}`);
}
greet();         // "Hello, Guest"
greet("Alice");  // "Hello, Alice"
greet(undefined); // "Hello, Guest" — undefined triggers the default
greet(null);      // "Hello, null"  — null does NOT trigger the default
```

---

## Destructuring

``` js
// Array destructuring
const [a, b, c] = [1, 2, 3];
const [first, , third] = [1, 2, 3]; // skip elements with comma
const [head, ...rest] = [1, 2, 3, 4]; // rest: [2, 3, 4]

// Object destructuring
const { name, age } = { name: "Alice", age: 25 };
const { name: userName, age: userAge } = user; // rename on extraction
const { name = "Guest", role = "user" } = {};  // default values

// Nested destructuring
const { address: { city } } = { address: { city: "NYC" } };

// In function parameters
function greet({ name, age = 0 }) {
  console.log(`${name} is ${age}`);
}
greet({ name: "Alice", age: 25 });
```

---

## Error Handling

### try / catch / finally

``` js
try {
  // Code that might throw an error
  const result = JSON.parse("invalid json");
} catch (error) {
  // Runs when an error is thrown — handle it gracefully
  console.error("Something went wrong:", error.message);
} finally {
  // Always runs, whether an error occurred or not — use for cleanup
  console.log("Done");
}
```

### Importance of the catch Block

- Without a catch, an uncaught error **crashes the entire program**
- The catch block lets you **handle the error gracefully** — show a message, retry, log, or fallback
- Always use catch around: I/O operations, JSON parsing, network calls, user input processing
- Never leave a catch block empty — at least, log the error

### Throwing Errors

``` js
// throw new Error() — preferred 
throw new Error("Something went wrong");

// throw "string" — avoid 
throw "Something went wrong";
```

**Difference between `throw new Error(...)` and `throw "..."`:**

| | `throw new Error("msg")` | `throw "msg"` |
|---|---|---|
| Type | `Error` object | String |
| `.message` property |  Available |  None |
| Stack trace |  Available |  None |
| `catch (e)` gives you | `e.message`, `e.stack` | Just the string |
| Recommended |  Always use this |  Avoid |

``` js
// Custom error with context
function divide(a, b) {
  if (b === 0) {
    throw new Error("Division by zero is not allowed");
  }
  return a / b;
}

try {
  divide(10, 0);
} catch (err) {
  console.error(err.message); // "Division by zero is not allowed"
  console.error(err.stack);   // full stack trace
}
```

### Reading Stack Traces

```
TypeError: Cannot read properties of undefined (reading 'name')
    at getUserName (app.js:15:20)   ← where the error actually happened
    at printUser (app.js:22:5)      ← called from here
    at main (app.js:30:3)           ← called from here
```

**How to read it:**
1. **First line** — the error type (`TypeError`) and what went wrong
2. **Stack lines** — read top to bottom: the top is where it crashed, the rest is the call chain leading there
3. **`file:line:column`** — open that file and go to that exact line
---

## `console` Methods

```js
console.log("General output");           // Standard logging
console.error("Error message");          // Red output, goes to stderr
console.warn("Warning message");         // Yellow/orange warning
console.info("Info message");            // Informational (same as log in most envs)
console.debug("Debug message");          // Verbose debug output

console.table([{ a: 1 }, { a: 2 }]);    // Displays array/object as a table
console.dir(someObject);                 // Shows object properties interactively

console.group("Group label");            // Starts a collapsible group
console.log("inside group");
console.groupEnd();                      // Ends the group

console.time("label");                   // Starts a timer
// ... some code ...
console.timeEnd("label");               // Logs elapsed time

console.count("label");                  // Counts how many times this line is hit
console.assert(1 === 2, "Math broke");  // Logs error only if condition is false
console.clear();                         // Clears the console
```

---

## Modules — `require` and `module.exports`

``` js
// ---- math.js (exporting) ----

const add = (a, b) => a + b;
const subtract = (a, b) => a - b;

// Export multiple things as an object
module.exports = { add, subtract };

// Or export a single value
module.exports = function multiply(a, b) { return a * b; };


// ---- app.js (importing) ----

// Import the whole exported object
const math = require("./math");
math.add(2, 3); // 5

// Destructure on import
const { add, subtract } = require("./math");
add(2, 3); // 5

// Import built-in or npm modules (no ./ needed)
const fs = require("fs");
const express = require("express");
```

---

## Searching MDN (Mozilla Developer Network)

MDN is the most reliable reference for JavaScript, HTML, and CSS.

**How to use it effectively:**
- Search Google: `mdn Array.map` or `mdn String.slice`
- Go directly to [developer.mozilla.org](https://developer.mozilla.org)
- Every method page includes: **syntax**, **parameters**, **return value**, **browser compatibility**, and **examples**
- Always check the **"Examples"** section — it shows real usage
- Check **"Specifications"** to understand if a method is a modern addition

---

## Best Practices

### Indentation
- Use **2 or 4 spaces** consistently — pick one, never mix
- Never use tabs and spaces together in the same project

### Variable Naming
- Use **camelCase** for variables and functions: `userName`, `getTotal`
- Use **PascalCase** for classes and constructors: `UserAccount`
- Use **UPPER_SNAKE_CASE** for constants: `MAX_RETRIES`

### Loop Variable Naming
- Don't always reach for `i` — use meaningful names: `userIndex`, `itemIndex`
- For `for...of` loops, name the variable after what it represents: `for (let user of users)`

### General Best Practices
- **Prefer `const`** — use `let` only when you need to reassign; never use `var`
- **One thing per function** — keep functions small and focused
- **Early returns** — avoid deep nesting by returning early from functions
- **Avoid magic numbers** — use named constants: `const MAX_RETRIES = 3`
- **Always handle errors** — never leave empty catch blocks
- **Use strict equality** (`===`) over loose (`==`)
- **Avoid global variables** — scope everything appropriately
- **Comment the WHY**, not the WHAT — your code should explain itself; comments explain intent

---

## Debugging Strategies

1. **Read the error message carefully** — the type, message, and stack trace tell you exactly where and what went wrong
2. **`console.log` strategically** — log values just before the error line to inspect actual state
3. **Use `console.table`** for arrays of objects — much easier to scan than plain logs
4. **Use the browser/Node debugger** — set breakpoints and step through code line by line
5. **Isolate the problem** — comment out code until the error disappears, then narrow in
6. **Check your assumptions** — use `typeof value` and `console.log(value)` to verify what you think is there
7. **Search the exact error message** — paste it into Google or MDN
8. **Rubber duck debugging** — explain the code out loud; you often find the bug yourself while explaining
9. **Check for typos** — especially in variable and function names (JS is case-sensitive)
10. **Verify data shapes** — when working with objects/arrays, log the full structure before accessing nested properties

---

## Functions as Objects (First-Class Citizens)

Functions can be stored in variables, passed to other functions, and returned from functions.

### Key Properties
- `function.name` — name of the function
- `function.length` — number of declared parameters

### Methods: `call`, `apply`, `bind`

```js
function exmp(fun) {  // Higher-order function
  fun(1, 2);
}

function add(a, b) {
  console.log(a + b); // add is a callback function here
}

exmp(add);
```

---

## Callbacks

A function that is **passed as an argument** to another function and called inside it.

---

## Shallow Copy vs Deep Copy

```js
// Shallow copy — copies references, changes affect both
let ar2 = ar1.slice();
let ar2 = [...ar1];

// Deep copy — fully independent, nested contents also copied
let ar2 = structuredClone(ar1);
```
