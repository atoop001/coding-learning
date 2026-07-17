# Chapter 15: Asynchronous JavaScript

## Overview

Some operations take time: downloading data, reading files, waiting for a timer. JavaScript runs on a **single thread** — one thing at a time — so if it simply *waited* for slow operations, the whole page would freeze. Instead, JavaScript is **asynchronous**: it starts a slow operation, keeps going, and deals with the result *later*, when it arrives.

This chapter builds the async mental model from the ground up: the event loop, then the three historical layers of async style — **callbacks**, **promises**, and **async/await**. Every modern web app fetches data; this chapter is the prerequisite for doing that (Chapter 16) without confusion.

## Definitions & Explanations

### The event loop (the 90-second version)

JavaScript has:

- a **call stack** — the code currently running;
- **web APIs / background tasks** — timers, network requests, handled outside your code;
- a **task queue** — completed background work waiting to re-enter;
- the **event loop** — moves queued work onto the stack *only when the stack is empty*.

Consequences you must internalize:

1. `setTimeout(fn, 0)` does **not** run immediately — `fn` waits in the queue until all current synchronous code finishes.
2. Async results are **never available "on the next line"**. Code after starting an async operation runs *before* the result exists.

### Layer 1: Callbacks

The oldest pattern: pass a function to be called when the work finishes.

```js
setTimeout(() => console.log("2 seconds passed"), 2000);
```

Callbacks work, but sequencing several async steps nests them deeper and deeper — **callback hell** ("pyramid of doom") — and error handling must be wired manually through every level.

### Layer 2: Promises

A **Promise** is an object representing a value that will exist *eventually*. It's in one of three states:

- **pending** — still working;
- **fulfilled** — done, has a value;
- **rejected** — failed, has a reason (an error).

Once settled (fulfilled or rejected), a promise never changes again.

You consume promises with:

- **`.then(fn)`** — runs `fn` with the value when fulfilled; returns a **new promise**, so `.then`s **chain flat** instead of nesting.
- **`.catch(fn)`** — handles a rejection from *anywhere earlier* in the chain.
- **`.finally(fn)`** — runs either way; ideal for cleanup (hide the spinner).

Combinators for multiple promises:

- **`Promise.all([...])`** — wait for *all*; rejects immediately if *any* rejects.
- **`Promise.allSettled([...])`** — wait for all, never rejects; gives per-promise status.
- **`Promise.race([...])`** — settles as soon as the *first* one settles.

### Layer 3: `async` / `await`

Syntax that makes promise code *look* synchronous:

- Marking a function **`async`** makes it always return a promise.
- Inside an async function, **`await promise`** pauses *that function* (not the whole program!) until the promise settles, then gives you the value — or **throws** the rejection, which means ordinary **`try/catch` works** for async errors (resolving Chapter 12's cliffhanger).

`async/await` is the style you'll write day-to-day; promises are what it runs on; callbacks are what you'll meet in older APIs and in `setTimeout`/event listeners.

## Code Examples

### 1. Proving the order of execution

```js
console.log("1: start");

setTimeout(() => console.log("3: timer done"), 0);   // queued, even at 0ms!

console.log("2: end");

// Output: 1, 2, 3 — the timer callback waits for the stack to clear.
```

### 2. Callback style — and why it hurts

```js
// A fake async operation, callback style:
function getUserCb(id, callback) {
  setTimeout(() => callback({ id, name: "Ada" }), 500);
}
function getPostsCb(userId, callback) {
  setTimeout(() => callback(["Post 1", "Post 2"]), 500);
}

// Sequencing = nesting. Add error params and it gets worse fast:
getUserCb(1, (user) => {
  getPostsCb(user.id, (posts) => {
    console.log(`${user.name} has ${posts.length} posts`);
    // a third step would nest again... the pyramid of doom
  });
});
```

### 3. Making and consuming promises

```js
// Wrap setTimeout into a promise — a utility you'll reuse forever:
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

// Promise-based versions of the fake API:
function getUser(id) {
  return delay(500).then(() => ({ id, name: "Ada" }));
}
function getPosts(userId) {
  return delay(500).then(() => ["Post 1", "Post 2"]);
}

// Chaining: each .then returns a promise, so the steps stay FLAT:
getUser(1)
  .then((user) => {
    console.log("Got user:", user.name);
    return getPosts(user.id);          // return a promise → next .then waits for it
  })
  .then((posts) => {
    console.log("Got posts:", posts);
  })
  .catch((err) => {
    console.error("Something failed:", err.message); // one handler for the whole chain
  })
  .finally(() => {
    console.log("Done either way");
  });
```

### 4. Rejection

```js
function riskyOperation() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      Math.random() > 0.5
        ? resolve("payload")
        : reject(new Error("network glitch"));   // reject with an Error object
    }, 300);
  });
}

riskyOperation()
  .then((data) => console.log("OK:", data))
  .catch((err) => console.log("Recovered from:", err.message));
```

### 5. The same flow with async/await

```js
async function showUserActivity() {
  try {
    const user = await getUser(1);         // pause here until resolved
    console.log("Got user:", user.name);

    const posts = await getPosts(user.id); // then pause here
    console.log(`${user.name} has ${posts.length} posts`);

    return posts.length;                   // async fn → this becomes a resolved promise
  } catch (err) {
    console.error("Something failed:", err.message);  // catches rejections too!
    return 0;
  } finally {
    console.log("Done either way");
  }
}

showUserActivity();
console.log("This prints FIRST — showUserActivity is paused, not the program");
```

### 6. Sequential vs. parallel

```js
async function sequential() {
  const a = await delay(1000).then(() => "A");   // 1s
  const b = await delay(1000).then(() => "B");   // +1s
  return [a, b];                                  // total ~2s
}

async function parallel() {
  const pa = delay(1000).then(() => "A");   // start both immediately...
  const pb = delay(1000).then(() => "B");
  const [a, b] = await Promise.all([pa, pb]); // ...then wait for both
  return [a, b];                              // total ~1s
}
// Rule: if step B doesn't need step A's result, run them in parallel.
```

### 7. `allSettled` for independent tasks

```js
async function loadDashboard() {
  const results = await Promise.allSettled([
    getUser(1),
    riskyOperation(),
    getPosts(1),
  ]);

  for (const r of results) {
    if (r.status === "fulfilled") console.log("✅", r.value);
    else console.log("❌", r.reason.message);   // one failure doesn't sink the rest
  }
}
```

## Common Pitfalls

### 1. Using an async value before it exists

```js
let user;
getUser(1).then((u) => { user = u; });
console.log(user);        // ❌ undefined — the .then hasn't run yet!

// ✅ Use the value WHERE it becomes available:
getUser(1).then((u) => console.log(u));
// or: const u = await getUser(1); (inside an async function)
```

This is *the* async beginner bug. The result only exists inside the `.then`/after the `await`.

### 2. Forgetting `await`

```js
async function main() {
  const user = getUser(1);          // ❌ user is a Promise object, not the data
  console.log(user.name);           // undefined
  const real = await getUser(1);    // ✅
  console.log(real.name);           // "Ada"
}
```

If you ever log something and see `Promise { <pending> }`, you forgot an `await` (or a `.then`).

### 3. Forgetting to return inside a `.then` chain

```js
getUser(1)
  .then((user) => { getPosts(user.id); })   // ❌ no return → next then gets undefined
  .then((posts) => console.log(posts));     // undefined

getUser(1)
  .then((user) => getPosts(user.id))        // ✅ return the promise
  .then((posts) => console.log(posts));
```

### 4. Unhandled rejections

```js
riskyOperation();   // ❌ if it rejects: "Unhandled promise rejection" — silent failure risk
// ✅ Every promise chain needs a .catch (or try/catch around its await).
```

### 5. `await` inside `forEach`

```js
// ❌ forEach ignores async — all fire at once, and you can't await the whole thing:
ids.forEach(async (id) => { await getUser(id); });
console.log("done");   // lies — nothing has finished

// ✅ Sequential: for...of.  Parallel: map + Promise.all
for (const id of ids) { await getUser(id); }              // one by one
await Promise.all(ids.map((id) => getUser(id)));          // all at once
```

### 6. Accidentally serializing independent work

```js
const user = await getUser(1);       // ⚠️ fine...
const news = await getNews();        // ...but news never needed user!
// ✅ Start both, then await: const [u, n] = await Promise.all([getUser(1), getNews()]);
```

## Practice Exercises

1. **Predict the order.** Without running, number the outputs of: a script that logs `"A"`, sets a 0ms timeout logging `"B"`, logs `"C"`, sets a 100ms timeout logging `"D"`, then logs `"E"`. Verify, and explain in one sentence why `"B"` is where it is.

2. **`delay` toolkit.** Write the `delay(ms)` promise wrapper yourself. Use it to write `countdown(n)` — an async function that logs `n`, `n-1`, ..., `"Go!"` with one second between each line.

3. **Fake API, three ways.** Write `fetchTemperature(city)` that resolves after 1s with a random temperature, but rejects (Error `"Sensor offline"`) 25% of the time. Consume it (a) with `.then/.catch`, (b) with `async/await` + `try/catch`, (c) five cities at once with `Promise.allSettled`, printing a line per city.

4. **Chain vs. parallel.** Using `delay`, simulate: `login()` (1s, returns a user), `loadProfile(user)` (1s, needs the user), and `loadWeather()` (1s, needs nothing). Write an async function that finishes in ~2s total, not ~3s, by parallelizing correctly. Log timestamps to prove it.

5. **Retry helper.** Write `withRetry(fn, attempts)` — an async function that calls the promise-returning `fn`; on rejection it retries up to `attempts` total tries, logging each failure, and throws the last error if all fail. Test it on your flaky `fetchTemperature`.
