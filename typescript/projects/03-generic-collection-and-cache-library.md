# Project 3: Generic Collection & Cache Library

## Description

Build `tinystore` — a small, reusable, fully generic data library with two pillars:

1. **`TypedCollection<T>`** — an in-memory collection of items with typed CRUD, querying, sorting, and aggregation. Think "a minimal lodash-meets-database-table," where `collection.sortBy("age")` only accepts real keys of `T` and returns properly typed results.
2. **`Cache<K, V>`** — a key-value cache with TTL (time-to-live) expiry, max-size eviction, and a `getOrCompute` pattern.

The library must know nothing about any concrete data shape — no `User` or `Task` types inside the library code — yet consuming it with your own types should feel telepathic: autocomplete on your property names, compile errors on typos, precise inferred return types everywhere. This is the project where Chapter 7 stops being theory.

Deliverable is a library package plus a demo consumer that uses it with at least two different domain types.

## Difficulty & Effort

- **Difficulty:** Intermediate
- **Estimated effort:** 6–9 hours

## Chapters Used

- `04-functions-in-typescript.md` (callbacks, overload judgment)
- `06-narrowing-and-type-guards.md` (guards in filtering)
- `07-generics.md` (core of the project)
- `08-classes-in-typescript.md` (generic classes, access modifiers, implements)
- `09-utility-types.md` (Partial for updates, Record, ReturnType)

## Requirements Checklist

### TypedCollection<T>
- [ ] `class TypedCollection<T extends { id: string }>` — the constraint gives the class `item.id` access while staying shape-agnostic
- [ ] `add(item: T): T`, `getById(id: string): T | undefined`, `remove(id: string): boolean`
- [ ] `update(id: string, patch: Partial<Omit<T, "id">>): T | undefined` — a typed patch that cannot change the id
- [ ] `where(predicate: (item: T) => boolean): T[]`
- [ ] `sortBy<K extends keyof T>(key: K, direction?: "asc" | "desc"): T[]` — misspelled keys must not compile
- [ ] `pluck<K extends keyof T>(key: K): T[K][]` — the return type must be the actual property type array
- [ ] `groupBy<K extends keyof T>(key: K): Map<T[K], T[]>` (or an equivalent Record-based signature — justify your choice in a comment)
- [ ] Internal storage is `private`; no way to mutate the collection except through methods

### Cache<K, V>
- [ ] `class Cache<K, V>` with `get(key: K): V | undefined`, `set(key: K, value: V, ttlMs?: number): void`, `has`, `delete`, `clear`, `size`
- [ ] Entries expire after their TTL (checked lazily on read is fine); a default TTL is configurable via a constructor options object
- [ ] Max-size eviction: when full, setting a new key evicts the oldest entry (document your definition of "oldest")
- [ ] `getOrCompute(key: K, compute: () => V): V` — returns cached value or computes, stores, and returns
- [ ] An `interface CacheStats { hits: number; misses: number; evictions: number }` exposed via a readonly accessor
- [ ] The cache implements a separately-declared `interface KeyValueStore<K, V>` (the contract the demo codes against)

### Library discipline
- [ ] The library files contain zero domain-specific types and zero `any`
- [ ] Type parameters follow the "appears at least twice" rule — no decorative generics (audit and state this in a comment)
- [ ] Public methods have explicit return type annotations
- [ ] `tsconfig` has `strict: true` and `noUncheckedIndexedAccess: true`, both passing

### Demo consumer (`src/demo.ts`)
- [ ] Uses `TypedCollection` with TWO different domain types you define (e.g., `Product` and `Employee`)
- [ ] Demonstrates: hovering-verified inferred types for `pluck`/`sortBy`/`groupBy` results (record them in comments)
- [ ] Demonstrates at least four distinct compile errors from misuse (typo'd key in `sortBy`, wrong patch shape in `update`, wrong value type into `Cache`, id mutation attempt) — quoted in comments
- [ ] Uses `Cache<string, ReturnType<...>>` at least once, deriving the value type from an existing function instead of restating it
- [ ] `getOrCompute` demo shows a hit and a miss reflected in `CacheStats`

## Hints

- Write `TypedCollection` for a hard-coded concrete type FIRST, get it fully working, then generify it. Extracting `T` from working code is far easier than designing in the abstract.
- If `sortBy` comparison errors on `a[key] < b[key]`, remember `T[K]` could be anything — decide a policy: constrain to string/number keys with a helper type, or accept a comparator. Either is defensible; the reasoning is the lesson.
- Inside `Cache`, storing `{ value: V; expiresAt: number | null; insertedAt: number }` per entry keeps expiry logic clean. `Map` preserves insertion order — helpful for "oldest".
- `getOrCompute`'s subtlety: an expired entry must count as a miss and be recomputed.
- If you feel the pull to write `function wrap<T>(x: T): T`-style helpers that relate nothing — resist; that's Pitfall 1 from Chapter 7.
- `Map<T[K], T[]>` vs `Record`: try Record first and notice what constraint it forces on `T[K]`. That discovery is worth a comment.

## Stretch Goals

- [ ] `findBy<K extends keyof T>(key: K, value: T[K]): T | undefined` — a fully key-value-typed lookup
- [ ] Add an async `getOrCompute(key, compute: () => Promise<V>): Promise<V>` overload — and prevent the "thundering herd" (two concurrent calls computing twice)
- [ ] Emit typed events (`"added" | "removed" | "evicted"`) with a generic `on(event, handler)` whose handler payload type depends on the event name (template for advanced generics)
- [ ] Set `declaration: true`, generate `.d.ts` files, and consume the built library from a separate scratch project as if it were an npm package (previews Chapter 11)
