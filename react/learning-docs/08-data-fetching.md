# Chapter 8: Data Fetching

## Overview

Almost every real app loads data from a server. This chapter covers fetching in effects done *properly* — loading/error states, race conditions, aborting stale requests — modeling request state with discriminated unions, and why production apps usually graduate to a server-state library like **TanStack Query (React Query)**. The manual version teaches you what those libraries actually solve.

## Definitions & Explanations

### Why fetching belongs in an effect (for now)

Rendering must stay pure, so network calls can't happen in the component body. Fetching data *because the component is on screen* is an external synchronization — an effect. (Fetching *because the user clicked* belongs in the event handler, per Chapter 7.)

### The three-and-a-half states of a request

Every fetch UI must handle: **loading**, **error**, **success** — and usually **empty** (success with no results). Model these as one discriminated union rather than separate booleans, so impossible combinations (`isLoading && isError`) cannot be represented:

```ts
type FetchState<T> =
  | { status: 'loading' }
  | { status: 'error'; message: string }
  | { status: 'success'; data: T };
```

### `fetch` refresher, React edition

- `fetch` rejects only on *network* failure. HTTP 404/500 resolve fine — **you must check `res.ok`** and throw yourself.
- `res.json()` returns `Promise<any>`; immediately assign it to a typed value (`const data: User[] = await res.json()`) or validate with a schema library (e.g. zod) at the boundary.
- `useEffect(async () => ...)` is invalid; define an async function inside the effect and call it, or use `.then` chains.

### Race conditions

If the user types "a", then "ab", two requests fly. If "a" resolves *after* "ab", your UI shows results for "a" under a query of "ab". This is a **race condition** and it *will* happen in production. Two fixes:

1. **Ignore flag** — cleanup sets a boolean; late responses are discarded.
2. **AbortController** — cleanup actually cancels the in-flight request (saves bandwidth too, and is the modern default). Aborting rejects the promise with an `AbortError` you must swallow.

The pattern generalizes: *any* effect that awaits something must assume the world changed while it awaited.

### React Query, conceptually

Manual effect-fetching leaves you to build: caching, request deduplication, refetch-on-focus, retries, pagination, mutations + invalidation, and shared state across components. **TanStack Query** treats server data as a *cache keyed by query keys* and gives you all of that:

```tsx
const { data, isPending, isError } = useQuery({
  queryKey: ['user', userId],           // identity of this data
  queryFn: () => fetchUser(userId),     // how to get it
});
```

Rules of thumb: learn the manual pattern (interviews and understanding), use React Query (or SWR, or router loaders / framework server components) in real apps. Server state and client state are different problems — don't store server responses in global client stores.

## Code Examples

### A complete, correct fetch-in-effect

```tsx
import { useEffect, useState } from 'react';

interface User { id: number; name: string; email: string }

type FetchState<T> =
  | { status: 'loading' }
  | { status: 'error'; message: string }
  | { status: 'success'; data: T };

function UserDetail({ userId }: { userId: number }) {
  const [state, setState] = useState<FetchState<User>>({ status: 'loading' });

  useEffect(() => {
    const controller = new AbortController();
    setState({ status: 'loading' }); // reset when userId changes

    async function load() {
      try {
        const res = await fetch(
          `https://jsonplaceholder.typicode.com/users/${userId}`,
          { signal: controller.signal },        // ties the request to cleanup
        );
        if (!res.ok) throw new Error(`HTTP ${res.status}`); // fetch won't do this for you
        const data: User = await res.json();   // typed at the boundary
        setState({ status: 'success', data });
      } catch (err) {
        if (err instanceof DOMException && err.name === 'AbortError') return; // expected on cleanup
        setState({ status: 'error', message: err instanceof Error ? err.message : 'Unknown error' });
      }
    }

    void load();
    return () => controller.abort(); // cancels the stale request when userId changes / unmount
  }, [userId]); // honest deps: the effect reads userId

  // Exhaustive rendering over the union — TS knows `data` exists only on success.
  switch (state.status) {
    case 'loading': return <p aria-busy="true">Loading user…</p>;
    case 'error':   return <p role="alert">Failed to load: {state.message}</p>;
    case 'success': return (
      <article>
        <h2>{state.data.name}</h2>
        <p>{state.data.email}</p>
      </article>
    );
  }
}
```

Why this shape matters: change `userId` quickly between 1 → 2 → 3 and the abort in cleanup guarantees only the response for 3 can reach state. StrictMode's double-mount also passes cleanly — the first mount's request is aborted, the second completes.

### Search with debounce + race protection

```tsx
function BookSearch() {
  const [query, setQuery] = useState('');
  const [state, setState] = useState<FetchState<string[]>>({ status: 'success', data: [] });

  useEffect(() => {
    if (query.trim() === '') { setState({ status: 'success', data: [] }); return; }

    const controller = new AbortController();
    // Debounce: wait 300ms of quiet before firing; cleanup cancels the timer too.
    const timer = setTimeout(async () => {
      setState({ status: 'loading' });
      try {
        const res = await fetch(
          `https://openlibrary.org/search.json?q=${encodeURIComponent(query)}&limit=10`,
          { signal: controller.signal },
        );
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const json = await res.json();
        const titles: string[] = json.docs.map((d: { title: string }) => d.title);
        setState({ status: 'success', data: titles });
      } catch (err) {
        if (err instanceof DOMException && err.name === 'AbortError') return;
        setState({ status: 'error', message: 'Search failed' });
      }
    }, 300);

    return () => { clearTimeout(timer); controller.abort(); };
  }, [query]);

  return (
    <div>
      <input value={query} onChange={e => setQuery(e.currentTarget.value)} placeholder="Search books…" />
      {state.status === 'loading' && <p>Searching…</p>}
      {state.status === 'error' && <p role="alert">{state.message}</p>}
      {state.status === 'success' && (
        state.data.length === 0
          ? <p className="muted">{query ? 'No results.' : 'Type to search.'}</p>
          : <ul>{state.data.map(t => <li key={t}>{t}</li>)}</ul>
      )}
    </div>
  );
}
```

### The same component with React Query (for comparison)

```tsx
// npm install @tanstack/react-query
// main.tsx: wrap the app in <QueryClientProvider client={new QueryClient()}>
import { useQuery } from '@tanstack/react-query';

async function fetchUser(id: number): Promise<User> {
  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

function UserDetailRQ({ userId }: { userId: number }) {
  const { data, isPending, isError, error } = useQuery({
    queryKey: ['user', userId],   // change userId → new cache entry, auto refetch
    queryFn: () => fetchUser(userId),
  });

  if (isPending) return <p aria-busy="true">Loading…</p>;
  if (isError) return <p role="alert">{error.message}</p>;
  return <article><h2>{data.name}</h2><p>{data.email}</p></article>;
}
// Gone: the state union, the AbortController, the race handling, the reset-on-change.
// Gained: caching, dedupe, retries, refetch-on-focus — for free.
```

## Common Pitfalls

1. **No `res.ok` check.** A 500 response happily `json()`s into garbage and your "success" UI renders nonsense. Always `if (!res.ok) throw`.

2. **Booleans instead of a union.** `isLoading` + `isError` + `data` lets you forget to unset `isLoading` on error, showing a spinner forever. The discriminated union makes such states unrepresentable — and TS narrows `data` for you.

3. **Ignoring races.** Any effect keyed on user input that fetches without abort/ignore will eventually show stale results. If you saw double-fetch weirdness in StrictMode and "fixed" it by removing StrictMode — the real bug was the missing cleanup.

4. **Forgetting to reset state on dependency change.** Switching from user 1 to user 2 without `setState({ status: 'loading' })` shows user 1's data under user 2's URL while loading.

5. **Fetching in a handler *and* mirroring into an effect** (or fetching the same data in five components). Duplicate requests, divergent copies. Lift the fetch, pass data down — or use React Query, whose cache dedupes by key.

6. **Trusting `any` from `res.json()`.** Unvalidated network data is the #1 source of runtime type errors in TS apps. At minimum assert a type at the boundary; for robustness, parse with zod and fail into your error state.

7. **Chained effects as a data pipeline.** Effect A fetches, sets state; effect B watches that state, fetches more… Cascades of renders, impossible to reason about. Do sequential awaits in *one* effect (or one query function).

## Practice Exercises

1. Build `PostList` fetching `https://jsonplaceholder.typicode.com/posts?userId={id}` with a user-id `<select>` (1–10), full union state handling, abort-on-change, and an empty state. Verify in the Network tab that switching users mid-flight cancels the old request.
2. Add "Retry" to the error state of exercise 1 without duplicating fetch logic. (Hint: a `retryCount` state in the dependency array is one honest approach.)
3. Extend `BookSearch` with a "cancelled vs failed" distinction in the UI, and log every request+outcome to the console; type quickly and confirm no stale response ever wins.
4. Write `useFetch<T>(url: string): FetchState<T>` as a custom hook (formalized next chapter) and use it in two unrelated components.
5. Install `@tanstack/react-query`, port exercise 1, then in DevTools' Network tab document three behaviors it gives you that your manual version lacked (e.g. cache hit on revisit, refetch on window focus, dedupe of simultaneous mounts).
