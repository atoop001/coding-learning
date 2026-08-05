# Project 4: Typed API Client

## Description

Build a typed client library for a real public REST API — recommended: JSONPlaceholder (`jsonplaceholder.typicode.com`, free, no auth) with its `/users`, `/posts`, `/comments`, `/todos` resources. The client wraps `fetch` behind a clean, promise-based, fully typed API:

```
const api = createClient({ baseUrl: "..." });
const posts = await api.posts.list({ userId: 3 });   // posts: Post[]
const created = await api.posts.create({ title, body, userId }); // no id — server assigns it
```

The heart of the project is the **boundary discipline** from Chapter 12: responses arrive as `unknown` and are validated by type guards before your types are trusted, errors are modeled as a discriminated `Result`/error union rather than thrown strings, and every request/response variant type (`NewPost`, `PostPatch`, `PublicUser`…) is *derived* from canonical types with utility types — never duplicated.

Using the finished client should feel like an SDK: everything autocompletes, illegal payloads don't compile, and a network failure or malformed response produces a precise, typed error instead of a mystery crash.

## Difficulty & Effort

- **Difficulty:** Intermediate-plus
- **Estimated effort:** 7–10 hours

## Chapters Used

- `04-functions-in-typescript.md` (async signatures)
- `05-unions-intersections-literal-types.md` (error unions, literal method types)
- `06-narrowing-and-type-guards.md` (validating unknown responses)
- `07-generics.md` (the shared request function)
- `09-utility-types.md` (core: deriving payload types)
- `12-tooling-and-real-world-patterns.md` (fetch patterns, boundary validation, typed testing with Vitest)

## Requirements Checklist

### Canonical types & derived views
- [ ] Canonical interfaces for at least three resources (e.g., `User`, `Post`, `Todo`) matching the real API responses
- [ ] `NewPost = Omit<Post, "id">` -style creation payload types — derived, not redeclared
- [ ] `PostPatch = Partial<Omit<Post, "id">>` -style update payloads — derived
- [ ] At least one `Pick`-based "summary" view used in a list-rendering helper
- [ ] HTTP method type is a literal union (`"GET" | "POST" | "PUT" | "PATCH" | "DELETE"`), not `string`

### The core request machinery
- [ ] ONE generic private function (e.g., `request<T>(path, options, validate: (data: unknown) => data is T)`) that all resource methods delegate to — no copy-pasted fetch logic
- [ ] `res.json()` is received as `unknown` — never `as T` without validation
- [ ] Every response passes a type guard before being returned as `T`; guard failure produces a typed error, not a crash
- [ ] Errors form a discriminated union with at least: `{ kind: "network" }` (fetch threw), `{ kind: "http"; status: number }` (non-2xx), `{ kind: "validation"; detail: string }` (guard failed)
- [ ] Public methods return `Promise<Result<T, ApiError>>` (or throw a single typed `ApiError` class — choose ONE strategy, state it in a comment, apply it uniformly)

### The client surface
- [ ] `createClient(config)` factory taking a config object (baseUrl, optional default headers, optional timeout)
- [ ] Resource namespaces: at minimum `users.list()`, `users.get(id)`, `posts.list(filter?)`, `posts.create(newPost)`, `posts.update(id, patch)`, `todos.list(filter?)`
- [ ] List filters are typed options objects (e.g., `{ userId?: number; completed?: boolean }`) translated to query strings
- [ ] Zero `any`, zero unvalidated `as` on external data, zero `!`

### Proof & demo
- [ ] `src/demo.ts`: fetches users, creates a post, patches it, lists a user's incomplete todos — logging typed results, with every error branch handled (the compiler must force this)
- [ ] Deliberately break one validator (e.g., demand a field the API doesn't send) and document the resulting typed validation error in a comment — proving guards actually run
- [ ] Comment block quoting 3+ compile errors from misuse (sending `id` in a create payload; typo'd filter property; treating a `Result` as the raw value)
- [ ] Write Vitest tests for `request`/one resource method: mock `fetch` with a typed mock (`vi.fn<typeof fetch>()`), and cover one success case and one validation-failure case using `satisfies`-checked fixtures — a handful of focused tests, not a full suite (Chapter 12's testing section)

## Hints

- Write the guards for primitives-in-objects once (`isNumber`, `isString`, an `isObject` helper) and compose them — hand-validating every field inline gets miserable fast, and that pain is the setup for the Zod stretch goal.
- Validating an array: check `Array.isArray`, then `every(isPost)`. Give `isPost` real teeth — JSONPlaceholder is well-behaved, so simulate hostility (see the broken-validator requirement).
- Generic `request<T>` + a passed-in guard keeps validation at the boundary while the rest of the client stays clean. Note how the guard parameter makes `T` inferable.
- For the `Result`-vs-throw decision: `Result` gives compiler-forced handling (Chapter 5); a typed error class gives idiomatic `try/catch` with `instanceof` narrowing (Chapters 6/8). Both are professional; mixing them is not.
- JSONPlaceholder fakes writes (POST returns an id but nothing persists) — perfect: your types describe the *contract*, and the demo still works.
- Query-string building from a typed filter object: `Object.entries` + `URLSearchParams`; mind that values need `String(...)` conversion.
- For the tests: `vi.fn<typeof fetch>()` gives you a mock whose `mockResolvedValue` must return something shaped like a `Response` (at minimum `ok` and a `json()` method) — the mismatch if you get it wrong is itself a good demonstration of typed mocks catching drift.

## Stretch Goals

- [ ] Add request timeout via `AbortController`, surfacing as `{ kind: "timeout" }` in the error union
- [ ] Add a retry policy (max attempts, backoff) for `network`/5xx errors only — typed config, no `any`
- [ ] Swap hand-written guards for Zod schemas with `z.infer` — measure how much code disappears; keep the public API identical (consumers shouldn't notice)
- [ ] Integrate Project 3's `Cache` to memoize GETs with a TTL, keyed by URL — two of your own libraries composing
- [ ] Type the endpoint map with template literal types (`` `GET ${string}` ``-style keys) as an advanced experiment
