# Chapter 1: Why React & Project Setup

## Overview

You already know modern JavaScript and TypeScript. This chapter explains the *problem* React solves, why manually manipulating the DOM stops scaling, and how to create and understand a Vite + React + TypeScript project on Windows. By the end you will have a running dev server and know what every file in the scaffold does.

## Definitions & Explanations

### The problem: keeping the DOM in sync with data

In vanilla JS, your app's *state* (data in variables) and the *UI* (the DOM) are two separate things that you must keep synchronized by hand:

```js
// Vanilla JS: imperative DOM updates
let count = 0;
const btn = document.querySelector('#btn')!;
const label = document.querySelector('#label')!;

btn.addEventListener('click', () => {
  count++;
  label.textContent = `Count: ${count}`; // YOU must remember to update the DOM
});
```

For a counter this is fine. But real apps have dozens of pieces of state, each affecting many DOM nodes. Forget one update and the UI silently shows stale data. This class of bug — *state/UI desynchronization* — is what React eliminates.

### The React model: UI as a function of state

React inverts the relationship. You never say "change this DOM node." You say "given this state, the UI *is* this," and React figures out which DOM changes are needed:

> **UI = f(state)** — you describe *what* the UI looks like for any state; React handles *how* to update the DOM.

Key terms:

- **Declarative** — you declare the desired result ("show `Count: 3`"), not the steps to get there. Vanilla DOM code is **imperative** (step-by-step instructions).
- **Component** — a reusable piece of UI defined as a TypeScript function that returns markup. Apps are trees of components.
- **Virtual DOM / reconciliation** — React keeps a lightweight description of the UI in memory. When state changes, it re-runs your component functions, diffs the new description against the old one, and applies only the minimal real-DOM changes. You don't manage this; you just benefit from it.
- **Re-render** — React calling your component function again to get the updated description. Re-rendering is cheap; it's not the same as rebuilding the DOM.
- **Function components + hooks** — the modern way to write React (since 2019). Components are plain functions; *hooks* (functions starting with `use`) give them state and other capabilities. React also has legacy **class components** — you'll see them in old codebases and Stack Overflow answers, but you should not write new ones. This track is 100% function components.

### Why Vite?

**Vite** is the standard build tool for React SPAs today. It gives you:

- Instant dev server startup and hot module replacement (HMR — edits appear in the browser without a full reload, preserving state).
- Built-in TypeScript and JSX handling.
- An optimized production build (`vite build`) via Rollup.

(`create-react-app` is deprecated; don't use it.)

## Code Examples

### Creating the project (Windows / PowerShell)

```powershell
# Requires Node.js 20+ (check with: node --version)
npm create vite@latest my-first-react-app -- --template react-ts
cd my-first-react-app
npm install
npm run dev   # opens dev server, usually http://localhost:5173
```

### Project anatomy

```text
my-first-react-app/
├─ index.html          # THE page. Vite serves this; note <div id="root"> and the script tag
├─ package.json        # dependencies (react, react-dom) + scripts (dev, build, preview)
├─ tsconfig.json       # TS config for your app code (strict mode on — keep it on)
├─ tsconfig.node.json  # TS config for vite.config.ts itself
├─ vite.config.ts      # Vite configuration; the react() plugin enables JSX + fast refresh
├─ public/             # static files copied verbatim to the build (favicons etc.)
└─ src/
   ├─ main.tsx         # entry point: mounts <App /> into #root
   ├─ App.tsx          # your root component
   ├─ App.css          # styles imported by App.tsx
   └─ index.css        # global styles imported by main.tsx
```

### The entry point, explained

```tsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';
import './index.css';

// Find the empty <div id="root"> in index.html and hand it to React.
// The "!" tells TS we know the element exists.
createRoot(document.getElementById('root')!).render(
  // StrictMode runs extra dev-only checks (e.g. double-invoking components
  // to surface impure code). It renders nothing and does nothing in production.
  <StrictMode>
    <App />
  </StrictMode>,
);
```

### Your first component from scratch

Replace `App.tsx` with:

```tsx
// src/App.tsx
// A component is a plain function that returns JSX (next chapter).
// Name must be Capitalized — that's how React tells components from HTML tags.
function App() {
  const learner = 'future React developer';
  const topics = 3;

  return (
    <main>
      <h1>Hello, {learner}!</h1>
      {/* Curly braces embed any TS expression into the markup */}
      <p>You have {topics} topics to cover this week.</p>
    </main>
  );
}

export default App;
```

Save and watch the browser update instantly — that's HMR.

### The same counter, React-style (a preview of Chapter 4)

```tsx
import { useState } from 'react';

function Counter() {
  // useState gives the component memory that survives re-renders.
  const [count, setCount] = useState(0);

  // No DOM manipulation anywhere. We change state; React updates the DOM.
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

Compare with the vanilla version at the top: there is no `querySelector`, no `textContent` assignment, and no way for the label to drift out of sync with `count`.

### Scripts you'll use constantly

```powershell
npm run dev       # dev server with HMR
npm run build     # type-check + production build into dist/
npm run preview   # serve the production build locally to sanity-check it
```

## Common Pitfalls

1. **Editing `index.html` expecting to build the UI there.** In a React SPA, `index.html` stays nearly empty; all UI lives in components under `src/`. If you find yourself adding markup to `index.html`, stop.

2. **Thinking in imperative steps.** Beginners ask "how do I change the text of that element?" The React answer is always: *which state, when changed, should make that text different?* You change data; React changes DOM.

3. **Panicking about StrictMode double-execution.** In development, StrictMode intentionally runs components (and later, effects) twice to expose impure code. Seeing two console.logs is *not* a bug. Don't remove StrictMode to "fix" it.

4. **Lowercase component names.** `function app()` used as `<app />` is treated as an unknown HTML tag, not your component. Components must start with a capital letter.

5. **Skipping the TypeScript template.** `--template react` (no `-ts`) creates a JS project. Professional React work is TypeScript; always use `react-ts` and keep `strict: true`.

## Practice Exercises

1. Create a fresh Vite + React + TS project named `react-playground`. Run the dev server, then open `dist/` after `npm run build` and inspect what a production build actually contains.
2. Delete everything inside `App.tsx` and rewrite it from memory as a component that renders your name in an `<h1>`, today's focus in a `<p>`, and the current year computed with `new Date().getFullYear()` inside curly braces.
3. In `main.tsx`, temporarily remove `<StrictMode>` and add a `console.log('App rendered')` inside `App`. Observe the log count with and without StrictMode, then put StrictMode back.
4. Write (in comments or a notes file) the imperative vanilla-JS version of a "like button" that toggles between "Like" and "Liked ♥", then sketch — without running it — what the React version would look like. Which piece of state drives the UI?
5. Break the project on purpose three ways — rename `App` to lowercase `app`, remove the `export default`, and return *two* top-level elements from the component — and read each error message carefully so you recognize them later.
