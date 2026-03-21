# JavaScript Async Programming
### A Complete Technical Reference — Callbacks · Promises · Async/Await · Event Loop

---

## 1. How JavaScript Executes Code

JavaScript is **single-threaded** — it can only do one thing at a time. All code runs on a single thread called the **main thread**, executing line-by-line, top to bottom.

### The Call Stack
When a function is called, JS pushes it onto the Call Stack. When it returns, it's popped off. Last in, first out.

```js
fun2();   // pushed → executes → popped
fun4();   // pushed → executes → popped
```

>  If a function blocks the stack (like `fun1`'s `while` loop for 5 seconds), nothing else can run. The page freezes.

---

## 2. Synchronous vs Asynchronous

### Synchronous (Sync)
Code runs one line at a time. Each line must finish before the next starts.

```js
fun2();  // executes immediately
fun4();  // waits for fun2 to finish
```

### Asynchronous (Async)
Code is scheduled to run later without blocking the current flow.

```js
setTimeout(fun1, 2000);  // scheduled for later
fun2();                  // runs immediately — doesn't wait for fun1
```

> In our code: `fun2()` and `fun4()` run right away. `fun3()` (1000ms) fires **before** `fun1()` (2000ms) — even though fun1 was registered first.

---

## 3. Ways to Make Code Async

- `setTimeout` / `setInterval` — schedule code after a delay
- `fetch` / `XMLHttpRequest` — network calls
- **Callbacks** — pass a function to call when done
- **Promises** — represent a future value
- **async/await** — cleaner syntax on top of promises
- **Event listeners** — react to user/browser events

---

## 4. Web Browser APIs

The browser gives JS extra powers beyond the language itself. These run **outside** the JS engine — in the browser's internals.

| API | What it does |
|-----|-------------|
| `setTimeout` / `setInterval` | Timer managed by the browser. Callback is queued after delay. |
| `fetch` | Network request. `.then()` callback queued when done. |
| `DOM APIs` | `getElementById`, `addEventListener` — interact with the page. |
| `localStorage`, `console`, `geolocation` | All browser-provided capabilities. |

> Browser APIs run **in parallel** with your JS. They put the callback into the task queue when done — JS picks it up when the call stack is empty.

---

## 5. The Event Loop

The event loop has one job: **check if the Call Stack is empty, and if so, push the next task from the queue.**

### How it works:
1. JS runs synchronous code — Call Stack fills and empties
2. Async tasks (setTimeout, fetch) go to **Web APIs** (browser handles them)
3. When done, their callbacks move to the **Task Queue**
4. Event Loop watches the Call Stack
5. When stack is empty → moves next callback from Queue onto Stack
6. It runs, finishes, gets popped → event loop checks again

> `setTimeout(fn, 0)` doesn't run immediately — it goes to the queue and only runs **after all synchronous code** completes.

### Micro vs Macro Task Queue

| Queue | Priority | Examples |
|-------|----------|---------|
| **Microtask** | Higher — drains completely first | Promise `.then()` callbacks |
| **Macrotask** | Lower — one task per cycle | `setTimeout`, `setInterval`, I/O |

---

## 6. Callbacks and Callback Hell

### 6.1 Inversion of Control
When you pass a callback to a function, you hand over control. You're trusting that function to:
- Call your callback (not too early, not too late)
- Call it only once
- Call it with the right arguments
- Not swallow errors

With third-party libraries, **you can't guarantee any of this**. This is Inversion of Control — you lose control of when and how your code runs.

### 6.2 Callback Hell

When async operations depend on each other, callbacks get nested — creating the "pyramid of doom":

```js
loadDashboard(function() {
  fetchUser(function() {
    fetchFriendList(function() {
      fetchPosts(function() {
        fetchComments(function() {
          // deeply nested...
        });
      });
    });
  });
});
```

**Problems:**
- Hard to read and maintain
- Error handling must be repeated at every level
- Difficult to run things in parallel
- Trust issues with each outer function

---

## 7. Promises

### 7.1 What is a Promise?
A Promise is an object that represents the **eventual completion (or failure)** of an async operation. It's a placeholder for a value that doesn't exist yet.

>  Think of it like a restaurant receipt. You place your order (start async work), get a receipt (the Promise), and come back for food when it's ready (`.then`). You don't have to stand there blocking the counter.

### 7.2 Creating a Promise

```js
let prm = new Promise((resolve, reject) => {
  // async work here
  if (success) resolve(result);  // fulfills the promise
  else reject(error);            // rejects the promise
});
```

The executor function runs **immediately** (synchronously). Call `resolve()` on success, `reject()` on failure.

### 7.3 Three States of a Promise

| State | Meaning | Trigger |
|-------|---------|---------|
| **Pending** | Still waiting | Initial state |
| **Fulfilled** | Success  | `resolve()` was called |
| **Rejected** | Failure  | `reject()` was called |

Once a promise settles (fulfilled or rejected), it **cannot change state**.

### 7.4 Consuming a Promise

```js
prm
  .then(result => console.log(result))   // runs on success
  .catch(error => console.log(error))    // runs on failure
  .finally(() => console.log('done'));   // always runs
```

---

## 8. Promise Chaining

### 8.1 Chaining with `.then`

Each `.then()` returns a new promise. If you return a promise from `.then()`, the chain waits for it.

```js
step1()
  .then(step2)   // waits for step1, then runs step2
  .then(step3)   // waits for step2, then runs step3
  .then(() => console.log('all steps successful'))
  .catch(() => console.log('error occurred'));
```

> In your code: step1 (4s) → step2 (2s) → step3 (3s) = ~9s total. Sequential, one after another.

### 8.2 Error Handling with `.catch`

`.catch()` catches any error thrown **anywhere above it** in the chain.

**With `.catch` — error is caught:**
```js
step1()
  .then(() => { throw new Error('oops'); })
  .catch(err => console.log(err));  //  catches it
```

**Without `.catch` — error is silently lost:**
```js
step1()
  .then(() => { throw new Error('oops'); })
// Unhandled promise rejection — warning in browser, crash in Node.js
```

### 8.3 Why `.catch` Must Be at the End

`.catch()` only catches errors from the steps **above** it.

```js
// BAD — won't catch errors from step2 or step3
step1()
  .catch(...)
  .then(step2)
  .then(step3)

//  GOOD — catches errors from all 3 steps
step1()
  .then(step2)
  .then(step3)
  .catch(...)
```

### 8.4 `.finally` Block

Runs **whether fulfilled or rejected**. Perfect for cleanup — hiding a loading spinner, closing a DB connection. It receives no arguments.

```js
step1()
  .then(step2)
  .catch(err => console.log(err))
  .finally(() => hideSpinner());  // always runs
```

---

## 9. Multiple Promises

### 9.1 Sequential (Chaining)
Promises run one after another. Total time = sum of all times. Use when each step depends on the previous.

### 9.2 `Promise.all` — Parallel, fail-fast

```js
Promise.all([step1(), step2(), step3()])
  .then(results => console.log(results))  // array of results, in order
  .catch(err => console.log(err));
```

- All run **in parallel** → total time = slowest promise
- Returns array of all results
-  If **ANY** promise rejects → immediately rejects, others are ignored

### 9.3 `Promise.allSettled` — Parallel, wait for all

```js
Promise.allSettled([step1(), step2(), step3()])
  .then(results => results.forEach(r => console.log(r.status, r.value)));
```

- Waits for **all** promises regardless of success/failure
- Returns `{ status: 'fulfilled', value }` or `{ status: 'rejected', reason }`
- **Never rejects itself** — always resolves

### 9.4 `Promise.any` — First success wins

- Resolves as soon as the **first** promise fulfills
- Rejects only if **all** promises reject (throws `AggregateError`)

### 9.5 `Promise.race` — First to settle wins

- Settles (resolve **or** reject) as soon as the **first** promise settles
- Useful for timeouts — race your `fetch` against a `setTimeout` rejection

### 9.6 `Promise.resolve` / `Promise.reject`

```js
Promise.resolve('value')  // instantly fulfilled promise
Promise.reject('error')   // instantly rejected promise
```

Useful for wrapping non-promise values or forcing a rejected state in tests/mocks.

---

## 10. Error Handling — Why It Matters Most

Async errors are **silent by default**. A rejected promise with no `.catch` is lost forever — no crash, no log, no clue.

-  Always add `.catch()` at the end of every promise chain
-  Never ignore the error argument in `.catch.`
-  Log the full error object, not just a string
-  In production, send errors to a monitoring service

> Your `loadDashboard` example shows why: one missing error check = silent failure with no feedback to the user. This is the most dangerous bug in async code.

---

## 11. Promisifying Callback Functions

**Promisify** = wrap an old callback-based function inside a Promise.

### Promisifying `setTimeout`:

```js
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

delay(2000).then(() => console.log('2 seconds passed'));
```

### Promisifying `fs.readFile` (Node.js):

```js
function readFile(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}
```

> Node.js has `util.promisify()` built-in for this. Modern Node.js also has `fs.promises` module with native promise support.

---

## 12. Async / Await

`async/await` is **syntactic sugar on top of promises**. Makes async code look and feel synchronous — much easier to read.

```js
async function run() {
  try {
    await step1();  // pauses here until step1 settles
    await step2();
    await step3();
  } catch (error) {
    console.log(error);  // catches any rejection
  }
}
```

Same behavior as promise chaining — but cleaner. `await` pauses execution of the async function (not the main thread!) until the promise settles.

### Real API call from your code:

```js
async function calls() {
  try {
    let res = await fetch(url);          // wait for response
    let data = await res.json();          // wait for parsed JSON
    console.log(data.abilities.ability);  // use the data
  } catch (error) {
    console.log(error);
  }
}
```

> `await` only works inside an `async` function. An `async` function always returns a promise, even if you don't explicitly return one.

---

## Quick Reference Summary

| Concept | Use When | Watch Out For |
|---------|----------|---------------|
| Callbacks | Simple one-off async ops | Nesting → callback hell |
| Promises | Sequential / parallel async | Forgetting `.catch` |
| `Promise.all` | Run multiple in parallel | One reject = all fail |
| `Promise.allSettled` | Need all results always | Check each `.status` |
| `Promise.race` | First one wins (timeout) | Any settle triggers it |
| `Promise.any` | First success wins | Rejects if ALL fail |
| `async/await` | Complex async flow | Must be inside async fn |

---
