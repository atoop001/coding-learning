# Chapter 13: Performance & Quality — Memoization, Error Boundaries, Testing

## Overview

Professional React is not just features that work — it's apps that stay fast, fail gracefully, and are protected by tests. This chapter covers React's rendering cost model and the memoization trio (`memo`, `useMemo`, `useCallback`), profiling with React DevTools, error boundaries, and component testing with **Vitest + React Testing Library** — the standard stack for Vite apps.

## Definitions & Explanations

### The render cost model

When state changes, React re-renders that component **and its entire descendant subtree** by default. Re-rendering is usually cheap (it's function calls + diffing, not DOM rebuilding). Therefore:

> **Rule zero: don't optimize until you've measured a real problem.** Premature `useMemo` everywhere adds noise and its own overhead. First fix *structure*: colocate state (Chapter 6), push state down, pass children through — these eliminate re-renders without any memoization API.

### The memoization trio

- **`memo(Component)`** — wraps a component so it *skips re-rendering when its props are shallowly equal* to last time. Only helps if props actually stay equal — which is where the other two come in.
- **`useMemo(fn, deps)`** — caches the *result* of a computation between renders; recomputes only when deps change. Two uses: genuinely expensive calculations, and *referential stability* (keeping the same object/array reference so `memo` children and effect deps don't fire).
- **`useCallback(fn, deps)`** — `useMemo` for functions: returns the same function reference between renders until deps change. Exists because inline handlers are new references every render, which defeats `memo` on children that receive them.

They work as a *system*: `memo` on the child is pointless if the parent passes a fresh inline object/handler each render; `useCallback` in the parent is pointless if no memoized child (or effect dep) consumes it.

> **React Compiler note:** the React Compiler (stable since 2025) automates most of this memoization. New projects increasingly enable it — but you must still understand the model, both for existing codebases and to understand what the compiler is doing for you.

### Profiling

React DevTools (browser extension) has a **Profiler** tab: record, interact, stop, and inspect which components rendered, why (props? state? parent?), and how long they took. Enable "Highlight updates when components render" for a live view. Measure → find the actually-slow thing → fix *that*.

### Error boundaries

A runtime error during rendering unmounts your **entire app** to a blank screen. An **error boundary** catches render-phase errors in its subtree and shows a fallback UI instead. Two facts:

1. Error boundaries are the one place class components survive in modern React — there's no hook equivalent. In practice everyone uses the tiny **`react-error-boundary`** package and never writes the class.
2. They catch errors in *rendering, lifecycles, and constructors* of the subtree — **not** in event handlers, async code, or the server (use try/catch and your fetch-state unions for those).

Place boundaries at meaningful seams: around routes/pages, around independent widgets — so one crashed widget doesn't take out the page.

### Testing components

The stack: **Vitest** (runner, Jest-compatible API, native Vite integration) + **React Testing Library** (renders components into a DOM and queries them *the way a user would*) + **jsdom** (fake browser) + **@testing-library/user-event** (realistic interactions).

Testing Library's philosophy: *test behavior, not implementation*. Query by **role and accessible name** (`getByRole('button', { name: /save/i })`), not by class names or component internals. Tests written this way survive refactors and double as accessibility checks.

Query cheat sheet: `getBy*` (must exist, throws otherwise) / `queryBy*` (may be absent — use for "not rendered" assertions) / `findBy*` (async — waits, for post-fetch UI).

## Code Examples

### Structure first: fixing re-renders without memo

```tsx
// ❌ Typing in the input re-renders <ExpensiveTree> on every keystroke
function Page1() {
  const [query, setQuery] = useState('');
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.currentTarget.value)} />
      <ExpensiveTree />
    </div>
  );
}

// ✅ Push the state down into a leaf — ExpensiveTree is untouched by typing
function Page2() {
  return (
    <div>
      <SearchInput />       {/* owns query state now */}
      <ExpensiveTree />
    </div>
  );
}

// ✅ Or pass children through: children are created by Page3's PARENT,
// so Wrapper's own state changes don't re-render them.
function Wrapper({ children }: { children: React.ReactNode }) {
  const [query, setQuery] = useState('');
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.currentTarget.value)} />
      {children}
    </div>
  );
}
```

### The trio used together, correctly

```tsx
import { memo, useCallback, useMemo, useState } from 'react';

interface Row { id: string; name: string; score: number }

// memo: skip re-render when props are shallow-equal
const RowItem = memo(function RowItem({ row, onSelect }: {
  row: Row;
  onSelect: (id: string) => void;
}) {
  console.log('render row', row.id); // watch this in the console
  return <li onClick={() => onSelect(row.id)}>{row.name}: {row.score}</li>;
});

function Leaderboard({ rows }: { rows: Row[] }) {
  const [selected, setSelected] = useState<string | null>(null);
  const [minScore, setMinScore] = useState(0);

  // useMemo: stable reference AND avoids re-sorting on unrelated renders
  const visible = useMemo(
    () => rows.filter(r => r.score >= minScore).toSorted((a, b) => b.score - a.score),
    [rows, minScore],
  );

  // useCallback: stable handler so memo'd rows don't all re-render
  // when `selected` changes. Functional update avoids depending on state.
  const handleSelect = useCallback((id: string) => {
    setSelected(prev => (prev === id ? null : id));
  }, []);

  return (
    <div>
      <input type="number" value={minScore}
             onChange={e => setMinScore(Number(e.currentTarget.value))} />
      <p>Selected: {selected ?? 'none'}</p>
      <ul>
        {visible.map(row => <RowItem key={row.id} row={row} onSelect={handleSelect} />)}
      </ul>
    </div>
  );
}
// Behavior: selecting a row re-renders ONLY the Leaderboard shell —
// every RowItem's props are referentially unchanged, so memo skips them all.
```

### Error boundaries with react-error-boundary

```tsx
// npm install react-error-boundary
import { ErrorBoundary, type FallbackProps } from 'react-error-boundary';

function WidgetFallback({ error, resetErrorBoundary }: FallbackProps) {
  return (
    <div role="alert" className="widget widget--crashed">
      <p>This panel crashed: {error instanceof Error ? error.message : 'Unknown error'}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function Dashboard() {
  return (
    <main>
      {/* Each widget crashes alone; the rest of the page survives. */}
      <ErrorBoundary FallbackComponent={WidgetFallback}>
        <RevenueChart />
      </ErrorBoundary>
      <ErrorBoundary FallbackComponent={WidgetFallback}>
        <ActivityFeed />
      </ErrorBoundary>
    </main>
  );
}
```

### Test setup (Vite + Vitest + Testing Library)

```powershell
npm install -D vitest jsdom @testing-library/react @testing-library/user-event @testing-library/jest-dom
```

```ts
// vite.config.ts — add the test block (needs: /// <reference types="vitest/config" />)
export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',        // simulate a browser
    globals: true,               // describe/it/expect without imports
    setupFiles: './src/test-setup.ts',
  },
});
```

```ts
// src/test-setup.ts — adds matchers like toBeInTheDocument()
import '@testing-library/jest-dom/vitest';
```

### Testing a component like a user

```tsx
// Counter.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, expect, it } from 'vitest';
import Counter from './Counter';

describe('Counter', () => {
  it('starts at zero and increments on click', async () => {
    const user = userEvent.setup();
    render(<Counter />);

    // Query by ROLE + accessible NAME — resilient and a11y-checking
    const button = screen.getByRole('button', { name: /increment/i });
    expect(screen.getByText(/count: 0/i)).toBeInTheDocument();

    await user.click(button);
    await user.click(button);

    expect(screen.getByText(/count: 2/i)).toBeInTheDocument();
  });
});
```

### Testing async UI (fetching component, mocked fetch)

```tsx
import { render, screen } from '@testing-library/react';
import { afterEach, describe, expect, it, vi } from 'vitest';
import UserDetail from './UserDetail';

describe('UserDetail', () => {
  afterEach(() => vi.restoreAllMocks());

  it('shows loading, then the user', async () => {
    vi.spyOn(globalThis, 'fetch').mockResolvedValue(
      new Response(JSON.stringify({ id: 1, name: 'Ada', email: 'ada@example.com' })),
    );

    render(<UserDetail userId={1} />);

    expect(screen.getByText(/loading/i)).toBeInTheDocument();
    // findBy* awaits the async state transition
    expect(await screen.findByRole('heading', { name: 'Ada' })).toBeInTheDocument();
    expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
  });

  it('shows the error state on HTTP failure', async () => {
    vi.spyOn(globalThis, 'fetch').mockResolvedValue(new Response('nope', { status: 500 }));
    render(<UserDetail userId={1} />);
    expect(await screen.findByRole('alert')).toHaveTextContent(/HTTP 500/);
  });
});
```

(For serious API mocking, look at **MSW** — Mock Service Worker — which intercepts at the network level and works in both tests and the browser.)

## Common Pitfalls

1. **Memoizing everything, measuring nothing.** Blanket `useMemo`/`useCallback` clutters code and can be *slower* (dependency comparisons aren't free). Profile first; most components never need any of it.

2. **`memo` defeated by a fresh prop.** `<MemoChild style={{ margin: 8 }} onPick={() => ...} />` — new object + new function every render; the memo never hits. Hoist constants to module scope, `useMemo`/`useCallback` the rest — or notice you didn't need `memo` at all.

3. **`useMemo` for correctness.** If your app *breaks* without `useMemo`, something else is wrong (usually an impure render or a mis-dependencied effect). Memoization is an optimization; React may discard cached values.

4. **Testing implementation details.** Asserting on state variables, instance internals, or CSS classes makes tests break on every refactor. Assert what the user sees and does. If you can't query it by role/label/text, that's often an accessibility smell.

5. **Forgetting `await` with `user-event` / `findBy`.** Un-awaited interactions cause flaky "act(...)" warnings and tests that pass for the wrong reason. Every `user.*` call and every `findBy*` returns a promise — await them.

6. **One app-level error boundary only.** Better than nothing, but a crashed like-button then blanks the whole app. Add boundaries per route and per independent widget.

7. **Expecting error boundaries to catch handler/async errors.** They don't. Handler errors need try/catch; fetch failures are your Chapter 8 unions.

## Practice Exercises

1. Build a list of 2,000 generated rows with a filter input. Use the DevTools Profiler to record typing before and after: (a) pushing state down, (b) adding `memo` + `useCallback` + `useMemo`. Note the flame-graph difference in a comment.
2. Take the Leaderboard example and *break* the memoization three different ways (inline handler, inline object prop, spreading a fresh object). Verify each break via the row `console.log`, then fix them.
3. Create a `<BuggyWidget>` that throws when a "crash" button's state flips, wrap it in an error boundary with reset, and confirm the rest of the page keeps working. Then prove to yourself that a `throw` inside the *click handler itself* is NOT caught — and handle it properly.
4. Set up Vitest + Testing Library in a project, then write tests for your Chapter 5 login form: renders both fields; shows a validation error only after blur; submit with valid input calls a mocked `onSubmit` callback with the typed values.
5. Write tests for Chapter 8's `UserDetail` covering all three union states — success, HTTP error, and network rejection (`mockRejectedValue`) — using `findBy*`/`queryBy*` correctly (no arbitrary timeouts).
