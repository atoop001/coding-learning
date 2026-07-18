# Chapter 9: useRef, Custom Hooks & the Rules of Hooks

## Overview

This chapter completes your core-hooks toolkit: `useRef` for values that persist without causing renders (and for reaching actual DOM nodes), the **Rules of Hooks** that make all hooks work, and **custom hooks** — the primary way React code gets reused and the pattern that separates tidy codebases from tangled ones.

## Definitions & Explanations

### `useRef`: a box that survives renders

```ts
const ref = useRef<T>(initialValue); // ref.current: T
```

A ref is a mutable object `{ current: T }` that React preserves across renders — like state, it persists; unlike state, **changing `ref.current` does not trigger a re-render**, and you may mutate it freely.

Two distinct use cases:

1. **DOM refs** — attach to an element with `ref={myRef}`; after mount, `myRef.current` is the DOM node. Used for focus, scrolling, measuring, text selection, media playback — imperative escape hatches React doesn't model declaratively. Type: `useRef<HTMLInputElement>(null)`.
2. **Instance variables** — remember a value across renders without it being "reactive": interval ids, previous values, "has this run before" flags, latest-callback holders.

**The dividing line:** if the *UI* should change when the value changes → state. If not → ref. Never read/write `ref.current` during rendering (except lazy-init patterns); do it in handlers and effects.

### The Rules of Hooks

1. **Only call hooks at the top level** of a component or custom hook — never inside conditions, loops, nested functions, or after an early return.
2. **Only call hooks from React functions** — components (capitalized) or custom hooks (`useXxx`), not arbitrary helpers.

*Why:* React identifies each hook by **call order**. Render 1 calls `useState, useState, useEffect` — React stores three slots. If a condition skips the first `useState` on render 2, every subsequent hook reads the *wrong slot*: state scrambles across hooks. The rules guarantee identical call order every render. The `eslint-plugin-react-hooks` rules enforce this — keep them on and treat violations as errors.

Conditional *behavior* is fine — put the condition inside the hook (`useEffect(() => { if (!enabled) return; ... })`) or split the conditional part into a child component you conditionally render.

### Custom hooks

A **custom hook** is just a function whose name starts with `use` and that calls other hooks. There is no magic registration — the naming convention is what lets the linter check the rules, and signals "this function has hook semantics."

What they give you:

- **Reuse of stateful logic** (not state itself — each *call site* gets independent state, exactly as if the hooks were written inline).
- **Separation of concerns**: components describe UI; hooks encapsulate how state/effects work (`useWindowWidth`, `useDebouncedValue`, `useLocalStorage`).
- **Typed contracts**: a hook's parameters and return type document the logic's API. Return tuples `as const` for `useState`-like pairs, or named-property objects for richer returns.

A custom hook is warranted when: the same state+effect combo appears twice; or a component's logic obscures its JSX; or you want to test logic apart from rendering.

## Code Examples

### DOM ref: focus management

```tsx
import { useRef } from 'react';

function SearchBar() {
  // null until React attaches the node after first render
  const inputRef = useRef<HTMLInputElement>(null);

  function focusSearch() {
    inputRef.current?.focus(); // optional chaining: null-safe and honest
  }

  return (
    <div>
      <input ref={inputRef} placeholder="Search…" />
      <button onClick={focusSearch}>Focus the search box</button>
    </div>
  );
}
```

### Instance-variable ref: tracking without re-rendering

```tsx
import { useEffect, useRef, useState } from 'react';

function VideoAnalytics({ src }: { src: string }) {
  const [playing, setPlaying] = useState(false);
  // How many times playback started — analytics data, never displayed:
  // a ref, because changing it must NOT re-render.
  const playCount = useRef(0);

  function handlePlay() {
    playCount.current += 1;         // mutate freely; no render
    setPlaying(true);               // UI state; renders
  }

  useEffect(() => {
    return () => console.log(`sent analytics: ${playCount.current} plays of ${src}`);
  }, [src]);

  return <video src={src} onPlay={handlePlay} onPause={() => setPlaying(false)}
                controls data-playing={playing} />;
}
```

### Custom hook: `useDebouncedValue`

```tsx
import { useEffect, useState } from 'react';

// Generic: works for strings, numbers, objects…
function useDebouncedValue<T>(value: T, delayMs = 300): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delayMs);
    return () => clearTimeout(id); // typing again cancels the pending update
  }, [value, delayMs]);

  return debounced;
}

// The component reads like a description of the UI again:
function LiveSearch() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebouncedValue(query, 300);

  // fetch with debouncedQuery in deps (Chapter 8 pattern)…
  return <input value={query} onChange={e => setQuery(e.currentTarget.value)} />;
}
```

### Custom hook: `useLocalStorage` (state + persistence bundled)

```tsx
import { useEffect, useState } from 'react';

function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    // Lazy init: read storage once; survive bad JSON gracefully.
    try {
      const raw = localStorage.getItem(key);
      return raw !== null ? (JSON.parse(raw) as T) : initialValue;
    } catch {
      return initialValue;
    }
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  // Tuple return mirrors useState — `as const` keeps the tuple types exact.
  return [value, setValue] as const;
}

// Independent state per call site:
function Settings() {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
  const [fontSize, setFontSize] = useLocalStorage('fontSize', 16);
  return (
    <div data-theme={theme}>
      <button onClick={() => setTheme(t => (t === 'light' ? 'dark' : 'light'))}>
        Theme: {theme}
      </button>
      <input type="range" min={12} max={24} value={fontSize}
             onChange={e => setFontSize(Number(e.currentTarget.value))} />
    </div>
  );
}
```

### Custom hook extracting a whole concern: `useToggleList`

```tsx
// Manages a set of selected ids — reusable across tag pickers, checklists, filters.
function useToggleList(initial: string[] = []) {
  const [selected, setSelected] = useState<ReadonlySet<string>>(new Set(initial));

  function toggle(id: string) {
    setSelected(prev => {
      const next = new Set(prev);            // copy: Sets must be replaced, not mutated
      next.has(id) ? next.delete(id) : next.add(id);
      return next;
    });
  }

  const clear = () => setSelected(new Set());

  // Object return: named, self-documenting, order-independent.
  return { selected, toggle, clear, count: selected.size };
}
```

## Common Pitfalls

1. **Using a ref where state belongs.** "I put the counter in a ref and the UI doesn't update" — correct, that's the definition. If it renders, it's state.

2. **Using state where a ref belongs.** Storing an interval id or "previous scroll position" in state causes pointless re-renders (and sometimes loops). If nothing rendered depends on it, use a ref.

3. **Reading refs during render.** `<p>{renderCount.current}</p>` while also incrementing it in the body is impure and breaks under StrictMode/concurrent rendering. Refs are for handlers and effects.

4. **Conditional hooks.**
   ```tsx
   // ❌ hook order differs between renders — state scrambles
   if (isOpen) { const [pos, setPos] = useState(0); }
   // ✅ hook always runs; condition lives inside, or in a child component
   const [pos, setPos] = useState(0);
   ```
   The early-return variant is sneakier: `if (!user) return null;` *above* other hooks is the same bug.

5. **Expecting custom hooks to share state.** Two components calling `useLocalStorage('theme', ...)` each have their own React state; they only coincidentally read the same storage key (and won't sync live). Shared *live* state needs lifting or context (Chapter 11).

6. **`use`-prefixing plain utilities.** `useFormatDate(date)` that calls no hooks shouldn't be a hook — the prefix subjects it to the rules needlessly. Conversely, a helper that *does* call hooks **must** be `use`-named or the linter can't protect you.

7. **DOM ref possibly-null confusion.** `inputRef.current.focus()` errors under strict null checks — the ref is `null` before mount and after unmount. Always `?.` or guard.

## Practice Exercises

1. Build a `Stopwatch` storing the interval id in a ref and elapsed ms in state, with Start/Stop/Reset — Start must be idempotent (clicking twice doesn't double-speed it).
2. Write `usePrevious<T>(value: T): T | undefined` using a ref updated in an effect, and use it to render "went up/went down" next to a number input.
3. Build `useCountdown(seconds)` returning `{ remaining, running, start, pause, reset }`. Use it in two unrelated components to prove independence of state per call site.
4. Refactor Chapter 8's fetch component into `useFetch<T>(url)` returning your `FetchState<T>` union, with abort in cleanup. Then explain in comments why `useFetch` in two components fetches twice — and which tool from Chapter 8 fixes that properly.
5. Create a form component with 3 fields where the first invalid field gets focused on submit, via DOM refs. Bonus: type a single `Record<FieldName, RefObject<HTMLInputElement>>` instead of three separate refs.
