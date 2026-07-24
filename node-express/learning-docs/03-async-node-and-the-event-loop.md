# Chapter 3: Async Node & the Event Loop

## Overview

You met the event loop, promises, and `async`/`await` in the JavaScript track — in the browser, where the stakes were "the page freezes for one user." In Node the same machinery carries much higher stakes: a server is *one* JavaScript process handling requests from *hundreds* of users on a single thread, and it can only do that because nearly everything slow (disk, network, database) happens off-thread while the event loop keeps cycling. Block that loop for 500 ms and every user of your API waits 500 ms. This chapter re-grounds your async knowledge in a server context: Node's error-first callback heritage and why you'll still see it, the `fs/promises` API you'll actually use, timers, top-level `await`, the difference between CPU-bound and IO-bound work, and — most importantly — a demonstration you'll run yourself of what blocking the loop does. Chapter 4 builds a real HTTP server on top of exactly these ideas.

## Definitions & Explanations

- **The event loop** — the scheduler at Node's heart. One thread runs your JavaScript; the loop's job is: run a piece of JS to completion, then check "did any timers fire? did any file reads finish? did any network data arrive?", run the callbacks queued by those events, repeat. It's the same concept as in the browser, but here the "events" are mostly *server* events: incoming requests, completed disk reads, database replies.

- **Single-threaded (for your JS)** — only one piece of *your* JavaScript executes at any instant. There's no parallel JS stepping on your variables — which is why you'll never write a mutex in this track. Node itself uses additional threads internally (via **libuv**, its C library) for file I/O and DNS, but you don't touch them; you just receive their results as events.

- **Non-blocking I/O** — when Node starts a slow operation (read a file, query a database), it *initiates* it, hands the waiting to the operating system or a libuv thread, and immediately returns to the loop to serve other work. The result arrives later as an event. Contrast with **blocking I/O**, where the whole thread sits idle mid-operation — fine in a one-user script, fatal in a server.

- **IO-bound work** — work that is mostly *waiting*: disk, network, databases. Node excels here, because waiting costs the loop nothing.

- **CPU-bound work** — work that is mostly *computing*: hashing, image resizing, parsing a 300 MB file, a gigantic loop. This runs *on* your one JS thread, blocks it, and is Node's genuine weakness. Small doses are fine; heavy jobs belong in a worker thread or a separate service (deployment-devops track territory — for now, just learn to recognize the hazard).

- **Error-first callback** — Node's original async convention, predating promises: pass a function whose *first* parameter is the error (or `null`): `fs.readFile(path, (err, data) => { … })`. You'll see it in older code, older docs, and a few stubborn APIs. Recognize it; don't imitate it.

- **Callback hell** — the pyramid of nested error-first callbacks that motivated promises. If you see code indenting rightward through five anonymous functions, you're looking at the reason `async/await` exists.

- **Promise** — (recap) an object representing a value that will arrive later, in states pending → fulfilled/rejected. Node's modern APIs return promises natively.

- **`fs/promises`** — the promise-returning version of the file-system module: `import { readFile, writeFile } from "node:fs/promises"`. This is the file API this track uses. The old callback API lives at `node:fs`; a synchronous API (`readFileSync`) also exists — legitimate in scripts and startup code, dangerous inside a running server.

- **`async`/`await`** — (recap) syntax that lets promise-based code read top-to-bottom. `await` pauses *that async function only* — the event loop keeps running everything else. This is the crucial server insight: `await` yields, it does not block.

- **Top-level `await`** — in ES modules you may `await` outside any function, at the top level of a file. Ideal for startup work: read config, open a database, *then* start serving.

- **Timers** — `setTimeout(fn, ms)` / `setInterval(fn, ms)`, same as the browser, with the same honesty requirement: `ms` is a *minimum*, not a guarantee — the callback runs when the loop next gets to it. A busy loop = late timers, which is exactly how we'll *measure* blocking below.

- **Concurrency vs. parallelism** — Node gives you *concurrency*: many operations in flight, overlapping in time, one JS instruction running at once. It does not give your JS *parallelism* (multiple instructions at the same instant). A Node server handles 1,000 concurrent connections by never waiting around, not by having 1,000 threads.

- **Sequential vs. concurrent awaiting** — `await a(); await b();` runs the operations one after another. `await Promise.all([a(), b()])` starts both and waits for both. When operations don't depend on each other, `Promise.all` is often a free 2–10× speedup.

## Code Examples

Assume a project with `"type": "module"` (Chapter 2) for everything below.

### The three eras of reading a file

```js
// era-1-callbacks.js — how Node code looked from 2009 until ~2017.
import { readFile } from "node:fs";

readFile("notes.txt", "utf8", (err, data) => {
  if (err) {                       // you must check, every time, by hand
    console.error("failed:", err.message);
    return;                        // forget this `return` and you fall through — a classic bug
  }
  console.log(data.length, "characters");
});
```

```js
// era-3-await.js — how you will write it. (Era 2, raw .then() chains, you know
// from the JS track; it's the same promises without the nice syntax.)
import { readFile } from "node:fs/promises";

try {
  const data = await readFile("notes.txt", "utf8");   // top-level await: fine in ESM
  console.log(data.length, "characters");
} catch (err) {
  console.error("failed:", err.message);
  process.exit(1);
}
```

Same behavior, but errors flow through `try/catch` like synchronous code, and there's no pyramid.

### Proving `await` doesn't block

```js
// interleave.js — watch the loop keep working while a "slow" operation waits.
function pretendQuery(label, ms) {
  // A stand-in for a database call: resolves after `ms` of *waiting*, not computing.
  return new Promise((resolve) => setTimeout(() => resolve(label), ms));
}

console.log("start");

const p = pretendQuery("query result", 1000).then((r) => console.log("got:", r));

// This line runs IMMEDIATELY — the await/promise above yielded to the loop.
console.log("the loop is free while the query waits");

setTimeout(() => console.log("a timer fired at ~500ms, mid-query"), 500);

await p;
console.log("end");
```

Run `node interleave.js` and read the output order carefully. That interleaving is a Node server's entire business model.

### Sequential vs. concurrent: a real difference you can time

```js
// naive-vs-better.js
import { readFile } from "node:fs/promises";

// Naive: three independent reads, forced into single file (≈ sum of the times).
console.time("sequential");
const a1 = await readFile("a.txt", "utf8");
const b1 = await readFile("b.txt", "utf8");
const c1 = await readFile("c.txt", "utf8");
console.timeEnd("sequential");

// Better: start all three, await all three (≈ the slowest single time).
console.time("concurrent");
const [a2, b2, c2] = await Promise.all([
  readFile("a.txt", "utf8"),
  readFile("b.txt", "utf8"),
  readFile("c.txt", "utf8"),
]);
console.timeEnd("concurrent");
```

(Create the three files first: `"aaa" | Set-Content a.txt` etc. With tiny local files the gap is small; with three database queries at 50 ms each it's 150 ms vs 50 ms, *per request*.)

The rule: operations that depend on each other → sequential `await`. Independent operations → `Promise.all`. One caveat to remember for later: `Promise.all` rejects as soon as *any* input rejects; `Promise.allSettled` waits for everyone and reports each outcome.

### Watching yourself block the event loop

This is the most important example in the chapter. The heartbeat timer *should* tick every 100 ms — unless something hogs the thread.

```js
// block-demo.js
let last = Date.now();
setInterval(() => {
  const now = Date.now();
  const drift = now - last - 100;            // how late is this 100ms heartbeat?
  console.log(`tick (late by ${drift}ms)`);
  last = now;
}, 100);

setTimeout(() => {
  console.log("starting CPU-bound work on the main thread...");
  const until = Date.now() + 2000;
  while (Date.now() < until) { /* pure computation: the loop cannot run */ }
  console.log("done. notice what the heartbeat did.");
}, 400);

setTimeout(() => process.exit(0), 3000);
```

Run it. The heartbeat ticks happily, goes *silent for two full seconds*, then reports being ~2000 ms late. In a server, every one of your users lives inside that heartbeat: during the busy-loop, no requests are accepted, no responses are sent, nothing happens. `JSON.parse` on a huge payload, a synchronous `readFileSync` of a big file inside a route, an accidental O(n²) over a large array — all are miniature versions of this `while` loop.

### Sync APIs: the legitimate use and the trap

```js
// Legitimate: a one-shot script or server STARTUP, before any users exist.
import { readFileSync } from "node:fs";
const config = JSON.parse(readFileSync("config.json", "utf8")); // fine here

// The trap (preview of Chapter 5 — just recognize the shape):
// app.get("/report", (req, res) => {
//   const big = readFileSync("huge-report.csv", "utf8"); // ⛔ blocks EVERY user
//   res.send(process(big));
// });
// Inside anything request-handling: fs/promises + await, always.
```

### Awaitable timers: the modern sleep

```js
// sleep-demo.js — the clean way to pause without callbacks.
import { setTimeout as sleep } from "node:timers/promises";

console.log("fetching pretend data...");
await sleep(750);                       // yields to the loop for ~750ms; blocks nothing
console.log("done. this pattern is great for retry/backoff:");

for (let attempt = 1; attempt <= 3; attempt++) {
  console.log(`attempt ${attempt}`);
  const succeeded = attempt === 3;       // pretend the third try works
  if (succeeded) break;
  await sleep(200 * attempt);            // wait a little longer each retry
}
console.log("connected");
```

Retry-with-backoff shows up constantly in real backends (flaky networks, databases still starting up). Note it *requires* an awaitable sleep — the callback form of `setTimeout` can't express "wait, then continue this same function" without nesting.

## Common Pitfalls

1. **Forgetting to `await` a promise.** `const data = readFile("x.txt", "utf8");` gives you a Promise object, and `data.length` is `undefined` — or worse, the operation's errors vanish as an "unhandled rejection." Correction: if a function is async, its call almost certainly needs `await`; when a value logs as `Promise { <pending> }`, you know exactly what you forgot.

2. **Importing from `node:fs` when you meant `node:fs/promises`.** Same function names, incompatible styles: the `node:fs` version expects a callback and returns `undefined`, so `await readFile(...)` silently awaits `undefined`. Correction: in this track, file imports come from `"node:fs/promises"` unless you're deliberately using a `*Sync` function.

3. **Using `forEach` with async callbacks.** `items.forEach(async (item) => { await save(item); })` fires all the saves and *returns immediately* — nothing waits, errors escape, "it works sometimes." Correction: `for (const item of items) { await save(item); }` for sequential, or `await Promise.all(items.map((item) => save(item)))` for concurrent.

4. **Wrapping an `await` in `try/catch` but only logging.** Catching, printing, and continuing as if the data existed just moves the crash three lines down. Correction: a catch block must *handle* — return early, exit non-zero, substitute a real fallback value — not just narrate.

5. **Sequential `await`s for independent work.** Three unrelated 100 ms lookups awaited one-by-one cost 300 ms every single time. Correction: spot `await` lines where the later ones don't use the earlier results; bundle them into `Promise.all`.

6. **Believing `setTimeout(fn, 100)` is a promise or a guarantee.** It returns a timer handle, not a promise (`await setTimeout(...)` from the global does nothing useful), and 100 ms is a floor, not a schedule. Correction: for awaitable delays use `import { setTimeout as sleep } from "node:timers/promises"` then `await sleep(100)` — and treat all timer durations as "no sooner than."

7. **Doing heavy CPU work on the main thread because "async will handle it."** `async` keywords don't move computation off-thread — a 2-second loop inside an async function blocks exactly like `block-demo.js`. Correction: async only helps *waiting*. For genuinely heavy computation, the answers are worker threads or separate processes — out of scope for now; the in-scope skill is *noticing* when work is CPU-bound.

## Practice Exercises

1. **Predict-then-run.** Before running `interleave.js`, write down the exact output order you expect, including where the 500 ms timer lands. Run it. If anything surprised you, trace through the event-loop steps until the surprise dissolves — then add a second query of 300 ms and predict again.

2. **`json-peek.js`.** A script run as `node json-peek.js <file.json>` that reads the file with `fs/promises`, parses it, and prints the top-level keys and the type of each value. Handle *both* failure modes distinctly — file missing vs. invalid JSON — with different messages and a non-zero exit. (Which of the two errors comes from `readFile` and which from `JSON.parse`?)

3. **Callback translation.** Take the `era-1-callbacks.js` example and extend it in callback style to: read `a.txt`, then write its uppercased contents to `b.txt`, then read `b.txt` back and log it — three nested levels. Feel the pyramid. Now rewrite it flat with `fs/promises` and `await`. Keep both versions; they're your before/after exhibit.

4. **Race the loop.** Modify `block-demo.js`: replace the busy-loop with (a) `await` of a 2-second promise-based sleep, then (b) a synchronous `JSON.parse(JSON.stringify(...))` of a giant generated array (build one with `Array.from({ length: 5_000_000 }, (_, i) => i)`). For each, predict whether the heartbeat stutters before you run it. Explain the difference in one sentence: which one *waits* and which one *works*?

5. **Concurrent downloader.** Using global `fetch` (available in Node 22) and an array of 3–4 URLs (e.g. `https://jsonplaceholder.typicode.com/todos/1` through `/4`), fetch them sequentially and time it, then with `Promise.all` and time it. Then kill one URL (typo the domain) and observe how each version fails. Switch the concurrent version to `Promise.allSettled` and report per-URL success/failure.

6. **`copy-dir.js` (stretch).** Copy every file from one folder to another: `node copy-dir.js src dest`. You'll need `readdir`, `mkdir`, and `copyFile` from `fs/promises`. First version: sequential loop. Second: `Promise.all`. Decide — and justify in a comment — whether concurrent copying is actually safe here, and what happens if `dest` doesn't exist.
