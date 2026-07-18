# Chapter 7: useEffect & the Component Lifecycle

## Overview

`useEffect` lets a component synchronize with things *outside* React — timers, subscriptions, the document title, browser APIs, network requests. It is also the most misused hook in React. This chapter covers the lifecycle of a function component, dependency arrays, cleanup functions, the classic effect mistakes, and — crucially — the many situations where you should **not** reach for an effect at all.

## Definitions & Explanations

### The lifecycle of a function component

A component instance goes through: **mount** (first render + DOM insertion) → zero or more **re-renders** (state/props/context changed) → **unmount** (removed from the DOM, state destroyed). Function components don't have lifecycle *methods*; instead:

- The function body runs on **every** render — it must be pure (no side effects during render).
- **Effects** run *after* React has updated the DOM (they never block painting... roughly).
- **Cleanup functions** run before the effect re-runs, and on unmount.

### The `useEffect` contract

```ts
useEffect(() => {
  // side effect: runs AFTER the render is committed
  return () => {
    // cleanup: runs before the NEXT run of this effect, and on unmount
  };
}, [dep1, dep2]); // dependency array
```

Dependency array semantics:

- **Omitted** — effect runs after *every* render. Rarely what you want.
- **`[]`** — runs after the first render only (plus its cleanup on unmount). "On mount."
- **`[a, b]`** — runs after the first render and after any render where `a` or `b` changed (compared with `Object.is`).

**The honest mental model:** the dependency array is *not* a way to choose when the effect runs. It is a declaration of every reactive value (props, state, and things derived from them) the effect *reads*. You don't pick dependencies — the code inside determines them. Lying (omitting a real dependency) causes stale-value bugs; the ESLint rule `react-hooks/exhaustive-deps` exists to catch this. If the correct dependencies make the effect run "too often," restructure the code — don't silence the rule.

### Cleanup, and why StrictMode double-runs effects

Anything an effect starts (interval, subscription, event listener, connection) must be stopped in cleanup, or it leaks — and after several remounts you'll have several intervals stacked up. In development, StrictMode deliberately runs *mount → cleanup → mount* once, specifically to expose missing cleanup. If your app misbehaves under StrictMode, the effect is buggy, not StrictMode.

### When NOT to use an effect

This deserves its own law: **if you can compute it during render, or do it in the event handler, you don't need an effect.**

- **Derived data**: filtering/totals/formatting from existing state → compute in render (Chapter 4). Not `useEffect` + a second state.
- **Reacting to a user action**: POSTing on submit, showing a toast on click → do it *in the handler*. Effects are for "this component is on screen" sync, not "the user did something."
- **Syncing two state variables**: `useEffect(() => setB(f(a)), [a])` causes an extra render and drift; derive `b` or lift the computation.
- **Resetting state when a prop changes**: use a `key` on the component instead.
- Legitimate effect use-cases: timers, subscriptions (websocket, `window` events, media queries), manually controlling non-React widgets, `document.title`, and data fetching (Chapter 8 — and even that has better tools).

## Code Examples

### Document title (a "true" external sync)

```tsx
import { useEffect, useState } from 'react';

function TitleCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // The effect READS `count`, so `count` MUST be a dependency.
    document.title = `Clicked ${count} times`;
  }, [count]);

  return <button onClick={() => setCount(c => c + 1)}>+1</button>;
}
```

### Interval with cleanup (and a stale-closure fix)

```tsx
function Stopwatch() {
  const [seconds, setSeconds] = useState(0);
  const [running, setRunning] = useState(false);

  useEffect(() => {
    if (!running) return; // no interval when paused — nothing to clean up either

    const id = setInterval(() => {
      // ✅ functional update: no need to depend on `seconds`,
      // and the interval never reads a stale snapshot.
      setSeconds(s => s + 1);
    }, 1000);

    // Cleanup runs when `running` flips or the component unmounts.
    return () => clearInterval(id);
  }, [running]);

  return (
    <div>
      <p>{seconds}s</p>
      <button onClick={() => setRunning(r => !r)}>{running ? 'Pause' : 'Start'}</button>
      <button onClick={() => setSeconds(0)}>Reset</button>
    </div>
  );
}
```

### Subscribing to a browser API

```tsx
function useWindowWidth(): number {
  const [width, setWidth] = useState(() => window.innerWidth);

  useEffect(() => {
    const onResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', onResize);
    // Without this cleanup, every mount adds ANOTHER listener forever.
    return () => window.removeEventListener('resize', onResize);
  }, []); // reads no reactive values → empty array is truthful

  return width;
}

function Layout() {
  const width = useWindowWidth(); // custom hook — formally introduced in Chapter 9
  return width < 600 ? <MobileNav /> : <DesktopNav />;
}
```

### The "you didn't need an effect" gallery

```tsx
// ❌ Derived state via effect: extra render, risk of drift
function Cart1({ items }: { items: { price: number }[] }) {
  const [total, setTotal] = useState(0);
  useEffect(() => { setTotal(items.reduce((s, i) => s + i.price, 0)); }, [items]);
  return <p>{total}</p>;
}
// ✅ Just compute it
function Cart2({ items }: { items: { price: number }[] }) {
  const total = items.reduce((s, i) => s + i.price, 0);
  return <p>{total}</p>;
}

// ❌ Event logic smuggled into an effect
function Save1({ draft }: { draft: string }) {
  const [saveRequested, setSaveRequested] = useState(false);
  useEffect(() => {
    if (saveRequested) { void fetch('/api/save', { method: 'POST', body: draft }); setSaveRequested(false); }
  }, [saveRequested, draft]);
  return <button onClick={() => setSaveRequested(true)}>Save</button>;
}
// ✅ Do it in the handler
function Save2({ draft }: { draft: string }) {
  return <button onClick={() => void fetch('/api/save', { method: 'POST', body: draft })}>Save</button>;
}

// ❌ Resetting state when a prop changes
function Profile1({ userId }: { userId: string }) {
  const [comment, setComment] = useState('');
  useEffect(() => { setComment(''); }, [userId]); // extra render, easy to forget fields
  return <textarea value={comment} onChange={e => setComment(e.currentTarget.value)} />;
}
// ✅ key remounts the subtree with fresh state
function ProfilePage({ userId }: { userId: string }) {
  return <ProfileEditor key={userId} userId={userId} />;
}
function ProfileEditor({ userId }: { userId: string }) {
  const [comment, setComment] = useState('');
  return <textarea value={comment} onChange={e => setComment(e.currentTarget.value)} />;
}
```

## Common Pitfalls

1. **The infinite effect loop.**
   ```tsx
   // ❌ Effect sets state → re-render → new [] object dep → effect runs → …
   const [list, setList] = useState<string[]>([]);
   useEffect(() => { setList([...list, 'x']); }, [list]);
   ```
   Any effect that unconditionally sets a state variable in its own dependency array loops forever. Restructure: derive the value, or ask why the effect exists at all.

2. **Object/array/function dependencies re-created every render.** `const options = { page }` in the body makes `options` a new object each render, so `[options]` fires every time. Depend on the primitive (`[page]`), move the object *inside* the effect, or memoize it (Chapter 13).

3. **Lying about dependencies.** Omitting `query` from deps because "I only want this on mount" means the effect forever sees the *first* `query` (stale closure). The fix is never to silence the linter — it's to move code, use functional updates, or split effects.

4. **Missing cleanup.** Intervals, listeners, and subscriptions without cleanup stack up on every remount. StrictMode's double-mount makes this visible immediately — treat any dev-mode double behavior as a missing-cleanup alarm, not an annoyance.

5. **One mega-effect.** Cramming the timer, the analytics ping, and the title update into one effect couples unrelated concerns and over-triggers all of them. One effect per synchronized thing, each with its own honest deps.

6. **`async` directly on the effect.** `useEffect(async () => …)` is invalid — an async function returns a Promise, but React expects the return value to be a cleanup function. Define an async function inside and call it (details next chapter).

## Practice Exercises

1. Build a `Clock` component showing the current time, updating every second, with correct cleanup. Prove there's exactly one interval by logging inside it and toggling the clock's mount from a parent button.
2. Write a `useLocalStorage(key, initial)`-style behavior *without* extracting a hook yet: a component whose `theme` state initializes lazily from `localStorage` and is written back in an effect whenever it changes. What is the honest dependency array?
3. Build `useOnlineStatus()` using the `online`/`offline` window events and render a banner when offline. Test with your browser DevTools' network throttling.
4. Take this buggy code and fix it two ways (once with deps, once by restructuring): a search box whose effect runs `console.log('searching for', query)` but has `[]` deps and therefore always logs the initial query.
5. Audit gallery: list five effects from tutorials or your own past code (or invent them) and classify each as "legit external sync" or "should not be an effect," with the replacement pattern named.
