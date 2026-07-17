# Chapter 9: Utility Types

## Overview

TypeScript ships a standard library of *type transformers* — utility types that derive new types from existing ones: `Partial<T>` makes everything optional, `Pick<T, K>` selects properties, `ReturnType<F>` extracts what a function returns, and a dozen more.

Why this matters professionally: real codebases have one canonical type (say, `User`) and many *views* of it — the creation payload (no `id` yet), the update payload (everything optional), the public profile (no password), the database row. Without utility types you'd copy-paste variants that silently drift apart. With them, every variant is *derived*, so changing `User` updates everything, and the compiler tells you what broke. This is DRY applied to types — a skill interviewers explicitly probe.

All utility types are built from mapped types, conditional types, and `keyof` — machinery you can inspect (Ctrl-click them in your editor!) and eventually write yourself. This chapter covers using them well; the peek under the hood at the end demystifies the magic.

## Definitions & Explanations

### The object reshapers

- **`Partial<T>`** — All properties optional. The type of "an update patch."
- **`Required<T>`** — All properties required (strips `?`). The type of "a config after defaults are applied."
- **`Readonly<T>`** — All properties `readonly` (shallow). For values that must not be mutated after creation.
- **`Pick<T, K>`** — Keep only keys `K` (a union of literal key names). "The subset I need."
- **`Omit<T, K>`** — Remove keys `K`. "Everything except…". Pick and Omit are duals; prefer whichever names the *smaller* list.
- **`Record<K, V>`** — An object type with keys `K` and values `V`. `Record<string, number>` is an open dictionary; `Record<"a" | "b", number>` requires *exactly* those keys — the typed replacement for index signatures when keys are known.

### The union filters

- **`Exclude<T, U>`** — Remove members of union `T` assignable to `U`. `Exclude<"a" | "b" | "c", "a">` → `"b" | "c"`.
- **`Extract<T, U>`** — Keep only members assignable to `U`.
- **`NonNullable<T>`** — Remove `null` and `undefined` from `T`.

### The function/promise introspectors

- **`ReturnType<F>`** — The return type of function type `F`. Use `ReturnType<typeof myFn>` to name a type you never declared.
- **`Parameters<F>`** — The parameter list as a tuple type.
- **`Awaited<T>`** — What a promise resolves to (recursively unwraps). `Awaited<Promise<string>>` → `string`.
- **`ConstructorParameters<C>` / `InstanceType<C>`** — Same ideas for classes.

### The key operator that powers it all

- **`keyof T`** — The union of `T`'s property names as literal types. `keyof User` → `"id" | "name" | "email"`. You met it with generics (Chapter 7); utility types lean on it constantly.
- **`typeof` (type position)** — Gets the type of a *value*: `typeof config` where `config` is a const. Combined: `keyof typeof config` = the keys of an actual object. Essential for deriving types from runtime constants.

## Code Examples

One canonical type, many derived views:

```ts
interface User {
  id: string;
  email: string;
  name: string;
  passwordHash: string;
  createdAt: Date;
  bio?: string;
}
```

### Partial — update patches

```ts
// Callers send only the fields they're changing:
function updateUser(id: string, patch: Partial<User>): void {
  // patch: { id?: string; email?: string; name?: string; ... all optional }
}

updateUser("u1", { name: "Ada L." });        // OK — just one field
updateUser("u1", {});                        // OK — empty patch is legal (watch this!)
updateUser("u1", { nmae: "typo" });
// Error: Object literal may only specify known properties…
// 'nmae' does not exist in type 'Partial<User>'.  ← still typo-proof!
```

### Omit / Pick — payloads and public views

```ts
// Creation payload: the server generates id/createdAt, so callers must not send them:
type NewUser = Omit<User, "id" | "createdAt">;

function createUser(data: NewUser): User {
  return { ...data, id: crypto.randomUUID(), createdAt: new Date() };
}

createUser({ email: "a@b.c", name: "Ada", passwordHash: "…" });   // OK
createUser({ id: "hack", email: "a@b.c", name: "Ada", passwordHash: "…" });
// Error: 'id' does not exist in type 'NewUser'.

// Public profile: whitelist what's safe to expose (Pick = allowlist — safer for
// security-sensitive filtering than Omit's blocklist, which fails open when
// someone later adds a sensitive field):
type PublicProfile = Pick<User, "id" | "name" | "bio">;

function toPublic(user: User): PublicProfile {
  return { id: user.id, name: user.name, bio: user.bio };
}
```

### Record — typed dictionaries

```ts
// Known keys — Record enforces completeness:
type Theme = "light" | "dark" | "highContrast";

const backgrounds: Record<Theme, string> = {
  light: "#ffffff",
  dark: "#111111",
  highContrast: "#000000",
};
// Delete a line → Error: Property 'highContrast' is missing.
// Add a theme to the union → every Record<Theme, …> in the codebase errors
// until updated. This is exhaustiveness for objects. 🎯

// Open dictionary (unknown keys):
const wordCounts: Record<string, number> = {};
wordCounts["hello"] = (wordCounts["hello"] ?? 0) + 1;

// Compound values:
const usersById: Record<string, User> = {};
```

### Readonly and Required

```ts
type FrozenUser = Readonly<User>;
declare const admin: FrozenUser;
admin.name = "eve";
// Error: Cannot assign to 'name' because it is a read-only property.
// (Shallow! admin.createdAt.setFullYear(1999) still compiles — Date is mutable.)

// Required — after defaults are applied, nothing is optional anymore:
interface Options { retries?: number; timeout?: number; verbose?: boolean }

function withDefaults(opts: Options): Required<Options> {
  return { retries: 3, timeout: 5000, verbose: false, ...opts };
}
const o = withDefaults({ retries: 5 });
o.timeout.toFixed();   // no undefined-check needed — Required<> proved it exists
```

### ReturnType / Parameters / Awaited — types you never wrote

```ts
// A function whose return shape you didn't name:
function buildQuery(table: string, limit: number) {
  return { sql: `SELECT * FROM ${table} LIMIT ${limit}`, params: [], startedAt: Date.now() };
}

// Name it after the fact — stays in sync with the implementation forever:
type Query = ReturnType<typeof buildQuery>;
// { sql: string; params: never[]; startedAt: number }

function logQuery(q: Query) { console.log(q.sql); }

// Parameters — forward arguments without restating types:
type BuildArgs = Parameters<typeof buildQuery>;   // [table: string, limit: number]
function buildAndLog(...args: BuildArgs) {
  logQuery(buildQuery(...args));
}

// Awaited — what an async function actually yields:
async function fetchUser(id: string): Promise<User> { /* … */ return {} as User; }
type FetchedUser = Awaited<ReturnType<typeof fetchUser>>;   // User, not Promise<User>
```

### Union surgery: Exclude / Extract / NonNullable

```ts
type Status = "draft" | "review" | "published" | "archived";

type ActiveStatus = Exclude<Status, "archived">;      // "draft" | "review" | "published"
type EndState = Extract<Status, "published" | "archived">; // those two only

type MaybeName = string | null | undefined;
type Name = NonNullable<MaybeName>;                   // string

// Realistic: filter a discriminated union by discriminant:
type Event2 =
  | { kind: "click"; x: number; y: number }
  | { kind: "keypress"; key: string }
  | { kind: "scroll"; deltaY: number };

type MouseEvent2 = Extract<Event2, { kind: "click" | "scroll" }>;
// the click and scroll members, full shapes intact
```

### keyof typeof — deriving types from runtime constants

```ts
// One source of truth: a value. Types derived, never duplicated:
const ROLES = {
  admin: 3,
  editor: 2,
  viewer: 1,
} as const;

type Role = keyof typeof ROLES;              // "admin" | "editor" | "viewer"
type RoleLevel = (typeof ROLES)[Role];       // 3 | 2 | 1

function canEdit(role: Role): boolean {
  return ROLES[role] >= ROLES.editor;
}
canEdit("admin");   // OK
canEdit("guest");
// Error: Argument of type '"guest"' is not assignable to parameter of type 'Role'.
```

### Composing utilities & a peek under the hood

```ts
// Utilities nest — read inside-out:
type UserPatch = Partial<Omit<User, "id" | "passwordHash">>;
// "any subset of User's fields, except the untouchable ones"

// Under the hood, Partial is just a mapped type — you could write it yourself:
type MyPartial<T> = { [K in keyof T]?: T[K] };
// "for each key K in T, the same property, made optional"
// Similarly: type MyPick<T, K extends keyof T> = { [P in K]: T[P] };
// Custom variant the stdlib lacks — make selected keys optional:
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
type UserDraft = PartialBy<User, "email" | "name">;
```

## Common Pitfalls

**Pitfall 1: Hand-writing derived types.**

```ts
// ❌ Will silently drift when User changes:
interface UserPatchManual { email?: string; name?: string; bio?: string }
// ✅ Derived — cannot drift:
type UserPatch2 = Partial<Omit<User, "id" | "passwordHash" | "createdAt">>;
```

If a type is conceptually "X but modified," *derive* it from X.

**Pitfall 2: `Partial<T>` as a lazy "some stuff might be missing" type.**
`Partial` means *every* field may be absent — including ones your code then reads unconditionally. If only some fields are optional, say exactly that (`PartialBy` above, or restructure). Overusing `Partial` pushes undefined-handling everywhere.

**Pitfall 3: Expecting `Readonly`/`Partial` to be deep.**
Both are shallow. `Readonly<User>` freezes `User`'s own properties; nested objects stay mutable. Deep variants exist in libraries (`type-fest`) or as custom recursive mapped types — know that the built-ins don't recurse.

**Pitfall 4: `Omit` with misspelled keys fails silently.**

```ts
type Oops = Omit<User, "paswordHash">;   // typo — compiles! Omit's K isn't
// constrained to keyof T, so nothing is removed and no error is raised. ⚠️
// Pick<User, "pasword"> DOES error (its K extends keyof T).
// Guard: define a strict version once:
type StrictOmit<T, K extends keyof T> = Omit<T, K>;
```

This asymmetry (Pick checks keys, Omit doesn't) is a notorious gotcha — especially painful in security filtering.

**Pitfall 5: `Record<string, T>` when the keys are known.**
`Record<string, Color>` accepts any string key and (without `noUncheckedIndexedAccess`) claims every lookup succeeds. If keys are a known set, use a literal-union key: you get completeness checking, autocomplete, and safe lookups.

**Pitfall 6: Reaching for `ReturnType` instead of naming the type.**
`ReturnType<typeof f>` is superb for types outside your control or intentionally inferred. But for your own public API, an explicit named interface is clearer, stabler documentation. Utility types serve design — they don't replace it.

## Practice Exercises

1. **The four views.** Given a `Product` interface you define (id, sku, name, priceCents, description, internalNotes, createdAt — pick sensible types), derive without redeclaring any property: `NewProduct` (no id/createdAt), `ProductPatch` (optional everything except id, which is excluded), `PublicProduct` (no internalNotes), and `FrozenProduct` (fully readonly). Write one function consuming each.

2. **Exhaustive lookup table.** Define `type Weekday = "mon" | ... | "fri"` and a `Record<Weekday, { open: string; close: string }>` of opening hours. Verify: removing a day errors; adding `"sat"` to the union errors at the table until filled in. Then write `hoursFor(day: Weekday)` and confirm no undefined-handling is required — and explain why `Record<string, …>` would have lost that guarantee.

3. **Introspection chain.** Write `async function loadSettings()` returning an object literal (don't declare its type). Using only `typeof`, `ReturnType`, and `Awaited`, produce a named `Settings` type, then write a synchronous `printSettings(s: Settings)`. Confirm changing the object literal automatically updates what `printSettings` accepts.

4. **Union surgery.** Given `type ApiEvent = { type: "created"; id: string } | { type: "updated"; id: string; changes: string[] } | { type: "deleted"; id: string } | { type: "ping" }`, use `Extract`/`Exclude` to derive: events that carry an `id`; events that don't; and a `type EventName = ApiEvent["type"]`. Write a handler that accepts only id-carrying events.

5. **Build your own.** Implement from scratch (then compare to the peek-under-the-hood section): `MyReadonly<T>`, `MyRecord<K extends string, V>`, and `RequireAtLeastOne<T, K extends keyof T>` (hard — the type where at least one of the K fields must be present). Test each with assignments that should and shouldn't compile.
