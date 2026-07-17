# Chapter 17: Modules (import/export)

## Overview

So far, your programs have lived in one file. Real projects don't: they're split into **modules** — separate files that each own one piece of the app (data logic here, DOM rendering there, utilities in a third) and explicitly share only what others need. Modules keep code organized, prevent naming collisions, make testing easier, and are simply how all modern JavaScript is written — every framework, library, and professional codebase uses them.

This chapter covers **ES Modules** (ESM) — the standard `import`/`export` system — how to use them in the browser and Node, and the design habits that make modules worth having.

## Definitions & Explanations

### What is a module?

A **module** is a JavaScript file with its own **module scope**: top-level variables in a module are *private to that file* by default (unlike classic scripts, where top-level `var`/function declarations pollute the shared global scope). To share something, you **export** it; to use something shared, you **import** it.

Modules also automatically run in *strict mode* and each module is evaluated **once** — no matter how many files import it, the code runs a single time and every importer gets the same instance (great for shared state, see below).

### Named exports

A module can export any number of named bindings:

```js
// math.js
export const PI = 3.14159;
export function square(x) { return x * x; }

// Or export a list at the bottom (equivalent):
function cube(x) { return x ** 3; }
export { cube };
```

Import them **by exact name** in curly braces:

```js
// main.js
import { PI, square } from "./math.js";
```

- Rename while importing: `import { square as sq } from "./math.js";`
- Import everything under a namespace: `import * as math from "./math.js";` → `math.square(4)`.

### Default exports

Each module may additionally have **one** default export — "the main thing this module is":

```js
// task.js
export default class Task { ... }

// main.js — no braces, and YOU choose the local name:
import Task from "./task.js";
```

Style guidance: many teams prefer **named exports everywhere** — they autocomplete better, rename-refactor safely, and typos fail loudly. Use default exports sparingly (e.g., when a file truly is one thing, like a single component).

### Module specifiers

- `"./utils.js"` / `"../lib/helpers.js"` — **relative paths**: your own files. In the browser, the leading `./` and the `.js` extension are **required**.
- `"lodash"` — **bare specifiers**: npm packages, resolved by Node or a bundler (Chapter 18).

### Using modules in the browser

```html
<script type="module" src="main.js"></script>
```

`type="module"` changes the rules: imports work, the script is deferred automatically, top-level `await` is allowed, and — important — **modules don't load from `file://` pages** in most browsers. Run a local server: `npx serve`, `python -m http.server`, or the VS Code "Live Server" extension.

### Using modules in Node

Either name files `.mjs`, or (usual practice) put `"type": "module"` in your `package.json`. You'll also encounter the older CommonJS system (`require`/`module.exports`) in tutorials and legacy code — recognize it, but write ESM.

### Modules as singletons — shared state

Because a module runs once, exporting a mutable object gives every importer the *same* object. This is a simple, legitimate way to share app state between files — and also a source of spooky bugs if done casually. Prefer exporting *functions that manage* state over the raw state itself.

## Code Examples

A small multi-file app — a task list split the way a real project would be:

### 1. A pure-logic module

```js
// tasks.js — knows NOTHING about the DOM. Just data in, data out.
let nextId = 1;                       // module-private: not exported!

export function createTask(title) {
  if (!title || title.trim() === "") {
    throw new Error("Task title required");
  }
  return { id: nextId++, title: title.trim(), done: false };
}

export function toggleTask(tasks, id) {
  return tasks.map((t) => (t.id === id ? { ...t, done: !t.done } : t));
}

export function stats(tasks) {
  const done = tasks.filter((t) => t.done).length;
  return { total: tasks.length, done, remaining: tasks.length - done };
}
```

### 2. A utilities module

```js
// format.js — small reusable helpers
export function plural(n, word) {
  return `${n} ${word}${n === 1 ? "" : "s"}`;
}

export function checkbox(done) {
  return done ? "[x]" : "[ ]";
}
```

### 3. A rendering module

```js
// render.js — ALL the DOM code lives here
import { stats } from "./tasks.js";
import { plural, checkbox } from "./format.js";

export function renderTasks(listEl, tasks) {
  listEl.innerHTML = "";
  for (const t of tasks) {
    const li = document.createElement("li");
    li.textContent = `${checkbox(t.done)} ${t.title}`;
    li.dataset.id = t.id;
    listEl.append(li);
  }
}

export function renderStats(el, tasks) {
  const s = stats(tasks);
  el.textContent = `${plural(s.remaining, "task")} remaining of ${s.total}`;
}
```

### 4. The entry point wires it together

```js
// main.js — the only file the HTML loads
import { createTask, toggleTask } from "./tasks.js";
import { renderTasks, renderStats } from "./render.js";

let tasks = [createTask("Learn modules"), createTask("Split my code")];

const listEl = document.querySelector("#list");
const statsEl = document.querySelector("#stats");

function refresh() {
  renderTasks(listEl, tasks);
  renderStats(statsEl, tasks);
}

listEl.addEventListener("click", (e) => {
  const li = e.target.closest("li");
  if (!li) return;
  tasks = toggleTask(tasks, Number(li.dataset.id));
  refresh();
});

refresh();
```

```html
<!-- index.html -->
<ul id="list"></ul>
<p id="stats"></p>
<script type="module" src="main.js"></script>
```

Notice the architecture the modules *enforce*: logic (`tasks.js`) can be tested without a browser; rendering (`render.js`) can change without touching logic; `main.js` is thin glue. This separation is the habit employers look for.

### 5. Default export example

```js
// storage.js — one main thing: a tiny localStorage wrapper
export default {
  save(key, value) {
    localStorage.setItem(key, JSON.stringify(value));
  },
  load(key, fallback = null) {
    const raw = localStorage.getItem(key);
    return raw === null ? fallback : JSON.parse(raw);
  },
};

// main.js
import storage from "./storage.js";   // any name you like — it's the default
storage.save("tasks", tasks);
```

### 6. Namespace import & re-export

```js
import * as fmt from "./format.js";
console.log(fmt.plural(3, "item"));   // "3 items"

// index.js "barrel" file — re-export a folder's public API from one place:
export { createTask, toggleTask, stats } from "./tasks.js";
export { renderTasks, renderStats } from "./render.js";
// Now consumers just: import { createTask, renderTasks } from "./index.js";
```

## Common Pitfalls

### 1. Forgetting `type="module"`

```html
<script src="main.js"></script>
<!-- ❌ "SyntaxError: Cannot use import statement outside a module" -->
<script type="module" src="main.js"></script>  <!-- ✅ -->
```

### 2. Opening the HTML as a file

Double-clicking `index.html` gives `file://...` — module imports will fail with CORS errors. **Run a local server** (`npx serve`, Live Server, `python -m http.server`) and open `http://localhost:...`.

### 3. Wrong path or missing extension

```js
import { square } from "math";        // ❌ bare specifier — browser can't resolve it
import { square } from "./math";      // ❌ browsers need the extension
import { square } from "./math.js";   // ✅ relative path + .js
```

### 4. Braces vs. no braces

```js
// math.js: export default function add() {} + export const PI = 3.14
import add from "./math.js";          // default → no braces
import { PI } from "./math.js";       // named → braces, exact name
import PI from "./math.js";           // ❌ silently imports the DEFAULT and calls it PI!
import { add } from "./math.js";      // ❌ error: no named export "add"
```

That third line is nasty: it "works" but gives you the wrong thing. Know which kind each export is.

### 5. Circular imports

If `a.js` imports from `b.js` and `b.js` imports from `a.js`, one side may see `undefined` during initialization. Usually a design smell — extract the shared piece into a third module both can import.

### 6. Sharing mutable state accidentally

```js
// config.js
export const settings = { theme: "dark" };
// Any importer can do settings.theme = "neon" and EVERY other importer sees it.
// ✅ Prefer controlled access:
let theme = "dark";
export const getTheme = () => theme;
export const setTheme = (t) => { theme = t; };
```

## Practice Exercises

For all of these, serve your files locally and use `type="module"`.

1. **First split.** Create `greetings.js` exporting `hello(name)` and `goodbye(name)` (named exports) and a default export `defaultGreeting` (a string). In `main.js`, import all three (both import styles) and log them. Deliberately misspell an import name once to see the error you get.

2. **Refactor the calculator.** Take your Chapter 12 `calculate(a, op, b)` (with its custom errors) and split it into `calculator.js` (logic + error classes) and `main.js` (a few test calls with try/catch). Export the error classes too, and use `instanceof` in `main.js` to prove imported classes work across files.

3. **Utility library.** Build `strings.js` (with `capitalize`, `slugify` from Chapter 9) and `numbers.js` (with `clamp`, `formatMoney`). Create a barrel `index.js` re-exporting all of them, and a `main.js` that imports everything from the barrel and demonstrates each function.

4. **Module-scope experiment.** Create a module `counter.js` with a private `let count = 0` and exported `increment()` / `current()`. Import it into **two** different modules that both increment, and show (by logging from `main.js`) that they share one count. Explain in a comment why.

5. **Re-architect a project.** Take your Chapter 11 delegated shopping list (or any exercise app you've built) and split it into at least three modules: pure logic, rendering, and a `main.js` entry point. The test of success: the logic module must contain zero references to `document`.
