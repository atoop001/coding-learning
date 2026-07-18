# Project 5: API Explorer

## Description

Build a browser for a public REST API — recommended: the PokéAPI, JSONPlaceholder, or the Open Library API — with live search, a paginated list, and a detail panel. Every request handles loading, error, and empty states; stale responses can never win; requests are aborted on change. You'll finish by extracting the machinery into custom hooks. This is the project that makes you trustworthy with async UI.

## Difficulty

**Intermediate+** — estimated effort: 8–12 hours.

## Chapters Used

- Chapter 7: useEffect & the Component Lifecycle
- Chapter 8: Data Fetching
- Chapter 9: useRef, Custom Hooks & Rules of Hooks
- (Foundation: Chapters 4–6)

## Requirements Checklist

### Request-state modeling
- [ ] A reusable discriminated union `FetchState<T>` (`loading | error | success`) — **no** separate `isLoading`/`isError` booleans anywhere
- [ ] API response types defined explicitly; `res.json()` assigned to a typed value at the boundary (no `any` escaping into components)
- [ ] `res.ok` checked on every request; HTTP errors produce your error state with the status visible

### List view
- [ ] Fetch a paginated list in an effect with honest dependencies
- [ ] Pagination controls (next/prev at minimum; page numbers welcome) — page is state, and changing it triggers a new request with the old one **aborted**
- [ ] Loading state that doesn't blank the whole page on page change (e.g. dim the stale list or show inline skeletons — your choice, note it)
- [ ] Error state with a working **Retry** button that doesn't duplicate fetch logic
- [ ] Empty state ("no results") distinct from the error state

### Search
- [ ] A controlled search input **debounced** (~300ms) before it triggers requests
- [ ] Race-condition proof: rapid typing can never display results for an earlier query — use `AbortController` in cleanup (verify in the Network tab that stale requests show as cancelled)
- [ ] Clearing the search returns to the normal list without a spurious request for `""`

### Detail panel
- [ ] Selecting a list item loads its detail (a second endpoint) into a panel or separate view
- [ ] Detail resets to loading when the selection changes; switching selections quickly never shows mismatched data
- [ ] Selection state design: comment on where it lives and why (you'll move it to the URL in Project 6's world)

### Custom hooks (extract after it works inline)
- [ ] `useDebouncedValue<T>(value, delayMs)` — generic, reused for search
- [ ] `useFetch<T>(url: string | null): FetchState<T>` (or a domain-specific equivalent) — handles abort, reset-on-url-change, and `null` = "don't fetch"; used by both list and detail
- [ ] Hooks obey the Rules of Hooks (linter clean) and live in `src/hooks/`

### Quality
- [ ] StrictMode on; the app behaves correctly with dev double-mounting (this is your missing-cleanup detector)
- [ ] Stable keys from API ids; `npm run build` clean

## Hints

- Build ugly-but-correct first: one component, inline effect, all three states rendered as plain text. Extract hooks and polish only once the Network tab shows correct abort behavior.
- The "don't blank the list while paging" requirement is a design decision about what `loading` means when you *already have data*. One honest option: keep the last success in state alongside the new request's status. Sketch the state shape before coding.
- For Retry, a `retryToken` state included in the effect's dependency array is an honest, linter-clean trigger — incrementing it re-runs the effect.
- `AbortError` handling: check `err.name === 'AbortError'` and return silently — treating aborts as errors makes cancelled-on-purpose requests flash your error UI.
- If your `useFetch` seems to need to fetch conditionally ("only when a thing is selected"), remember: hooks can't be called conditionally, but a `null` URL parameter meaning "idle" keeps the hook unconditional and the logic conditional.

## Stretch Goals

- [ ] A tiny in-memory cache inside `useFetch` (a module-level `Map<string, T>` keyed by URL) so revisiting a page/detail is instant — then note in a comment two problems your cache has that real libraries solve (staleness, invalidation, dedupe...)
- [ ] Port the whole app to **TanStack Query** on a branch: compare line count, cancelled requests, and behavior on window refocus; write a short comparison in the README
- [ ] Keyboard navigation for the list (↑/↓ moves selection, Enter opens detail) using refs for scroll-into-view
- [ ] Offline detection (`useOnlineStatus` from Chapter 7's exercises) with a banner and automatic retry when back online
- [ ] Request timing display ("loaded in 230ms") using a ref for the start timestamp
