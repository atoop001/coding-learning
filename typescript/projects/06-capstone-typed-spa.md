# Project 6: Capstone — "Shelf": a Typed Single-Page App

## Description

Build **Shelf**, a complete single-page application for tracking a personal collection (pick your domain: books, films, games, recipes — anything with rich fields). This is the capstone: every chapter converges here, and the result is a portfolio piece you can walk an interviewer through.

Shelf is a multi-view SPA with hash-based routing (`#/collection`, `#/item/:id`, `#/add`, `#/stats`), full CRUD with validated forms, search/filter/sort, `localStorage` persistence with runtime validation, data enrichment from a real third-party API (e.g., Open Library for books, TVMaze for shows — both free and keyless), and at least one third-party npm library integrated **through its types** (a date library, a chart library, or similar).

The professional framing: strict-plus compiler settings, type-aware linting, a layered architecture (domain types / storage / API / UI as separate modules with explicit contracts), typed error handling at every boundary, and a written README that explains your type-design decisions — because explaining *why* is what distinguishes a hire.

It should feel like a real product: fast, keyboard-friendly, survives refresh, tolerates a corrupted localStorage and a dead network without white-screening.

## Difficulty & Effort

- **Difficulty:** Advanced (for this track)
- **Estimated effort:** 15–25 hours — plan multiple sessions

## Chapters Used

All of them, materially:

- `01`–`04` (project setup, annotations, shapes, functions — assumed fluent)
- `05-unions-intersections-literal-types.md` (route + async-data state unions)
- `06-narrowing-and-type-guards.md` (guards for storage/API payloads, exhaustive rendering)
- `07-generics.md` (reusable storage/fetch helpers)
- `08-classes-in-typescript.md` (service layer against interfaces)
- `09-utility-types.md` (derived form/patch/summary types)
- `10-enums-tuples-advanced-objects.md` (satisfies-checked config, honest index access)
- `11-modules-declaration-files-third-party.md` (module architecture, @types, augmentation)
- `12-tooling-and-real-world-patterns.md` (strict flags, ESLint, boundaries, DOM)

## Requirements Checklist

### Foundation
- [ ] Vite + TypeScript, `strict`, `noUncheckedIndexedAccess`, `noImplicitOverride`, `verbatimModuleSyntax` — all passing via a `check` script
- [ ] ESLint with `typescript-eslint` type-checked preset; zero errors, zero `eslint-disable` without a justifying comment
- [ ] Layered modules with explicit boundaries: `domain/` (types + pure logic), `storage/`, `api/`, `ui/`, `router/` — UI imports domain, never the reverse; `import type` used consistently
- [ ] A GitHub Actions workflow (`.github/workflows/`) that runs on every push: install deps, `tsc --noEmit`, and the Vitest suite — the build fails if either fails (Pitfall 7 from Chapter 12, enforced instead of just known)

### Domain modeling
- [ ] A canonical item type with at least 7 fields including: a literal-union status (e.g., `"wishlist" | "owned" | "in-progress" | "finished"`), a rating, tags, timestamps
- [ ] Derived types via utilities — `NewItem`, `ItemPatch`, a `Pick`ed list-card summary — no duplicated shapes anywhere in the codebase
- [ ] At least one discriminated union beyond routing/async (e.g., items of different kinds with kind-specific fields, or a typed domain event log)
- [ ] A `Result<T, E>` type (or typed error classes — one strategy, uniform) used across storage and API layers

### Routing & app state
- [ ] Route state is a discriminated union parsed from `location.hash` by a `parseRoute(hash: string): Route` function — unknown hashes map to a `not-found` member, never a crash
- [ ] Async data is modeled as `idle | loading | success | error` unions — no `data?: X; error?: Y` bags
- [ ] Central exhaustive rendering per route with `never` checks; adding a route without handling it must not compile

### Persistence & boundaries (make-or-break section)
- [ ] `localStorage` read path: `unknown` → validation (guards or a schema library) → typed data; a corrupted/legacy payload is detected and handled gracefully (report + reset, not a crash) — demonstrate by hand-corrupting storage and documenting the behavior
- [ ] A generic typed storage helper (e.g., `Store<T>` taking a validator) reused for every persisted key
- [ ] Third-party API integration with the full Project-4 discipline: validated responses, discriminated error union incl. network/http/validation, all failure branches rendered in UI
- [ ] Item enrichment flow: user searches the external API, picks a match, fields pre-fill the add/edit form — mapped from the API's shape to yours in one typed mapping function

### Third-party types in practice
- [ ] At least one npm library consumed through bundled or `@types` declarations — with a comment noting which kind it ships and one signature you relied on
- [ ] At least one type augmentation: extend `Window` (e.g., a typed debug handle) via `declare global`, or augment a library's interface — in a dedicated `.d.ts`/types file
- [ ] If any dependency lacks types: a minimal hand-written `declare module` for the parts you use (if all your deps are typed, simulate this once in a scratch file and link it in the README)

### Features & quality bar
- [ ] Full CRUD with a validated form (required fields, ranges, uniqueness where sensible) — validation errors are typed per-field, rendered inline, and cleared on fix
- [ ] Search (text), filter (status + tags), and sort (at least 3 typed sort keys via `keyof`-constrained helpers) — composable, reflected in the URL where reasonable
- [ ] A stats view computed with typed aggregation (counts by status via `Record<Status, number>`, average rating, etc.)
- [ ] Keyboard navigable; focus moves sanely on route change; no dead interactive elements
- [ ] Zero `any`, zero `!`, `as` only where narrowing is provably impossible and commented (target ≤ 2 in the whole app)
- [ ] README: setup/run instructions, architecture sketch, and a "Type Design Decisions" section explaining ≥5 concrete choices (why a union here, why Omit there, Result vs throw, etc.)

## Hints

- **Order of attack:** domain types + pure logic (testable with zero DOM) → storage layer → router skeleton with dummy views → list/detail views → forms → external API → stats → polish. Working software at every stage.
- Steal from yourself: Project 2's state unions, Project 3's generic patterns, Project 4's client discipline, Project 5's DOM helpers. Adapting your own code into a new architecture is the real exercise.
- `parseRoute` is a parser at a boundary — the hash is user-controllable input. Treat `#/item/abc` where `abc` doesn't exist as a *routing success* but a *data miss* (a `not-found` render), and keep those failures distinct in your types.
- Form state is not domain state: a half-filled form is all-strings-and-maybes. Give it its own type and one conversion function (`formToNewItem: (f: FormState) => Result<NewItem, FieldErrors>`) — trying to reuse `Item` for form state is the classic mistake this project should teach you to avoid.
- For `FieldErrors`, a mapped type over your form's keys (`Partial<Record<keyof FormState, string>>`) keeps error rendering typo-proof.
- When the render-everything-on-change approach from Project 5 starts hurting at this scale, contain it: render per-region (header/main), and note the pain in your README — it's your best "why frameworks?" interview answer.
- Budget real time for the corrupted-storage requirement. Migration/validation of persisted data is disproportionately valuable experience — it's most of what "senior" data handling looks like.
- For the CI workflow: a GitHub Actions workflow file needs a trigger (`on: push`), a job with a runner (`runs-on: ubuntu-latest`), and steps that check out the repo, set up Node, install deps, then run your `check` script and your test script — in that order, so a type error fails fast before tests even start. GitHub's own "Building and testing Node.js" quickstart docs show the exact anatomy; adapt it rather than inventing one from scratch.

## Stretch Goals

- [ ] Undo/redo via an immutable state history (readonly types will keep you honest)
- [ ] Import/export the collection as a JSON file — export typed, import fully validated with per-record error reporting
- [ ] A tiny typed event bus (`on`/`emit` where payload type depends on event name) decoupling storage writes from UI updates
- [ ] Swap hand-rolled validation for Zod across every boundary; measure the diff in lines and in confidence
- [ ] Unit tests (Vitest) for domain logic and every validator — including hostile inputs
- [ ] Deploy it (GitHub Pages/Netlify) and put the link on your CV — the entire point of the track
