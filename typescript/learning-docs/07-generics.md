# Chapter 7: Generics

## Overview

You've already been *using* generics: `string[]` is `Array<string>`, `Promise<User>`, `Map<string, number>`. This chapter teaches you to *write* them. Generics are type-level parameters — they let one function, interface, or class work over many types **while preserving the relationship between inputs and outputs**.

Without generics you face a lose-lose choice: write `firstOfStrings`, `firstOfNumbers`, `firstOfUsers`… (duplication), or write `first(arr: any[]): any` (safety gone). Generics resolve it: `first<T>(arr: T[]): T` — one implementation, full type fidelity, for types that don't even exist yet.

Every serious TypeScript codebase and every library you'll ever use (React, Express, Prisma, fetch wrappers) leans on generics constantly. Reading them fluently is required for professional work; writing them well is a differentiator.

## Definitions & Explanations

**Type parameter** — A placeholder type introduced in angle brackets: the `T` in `function first<T>(arr: T[]): T`. Conventional single letters: `T` (type), `K` (key), `V` (value), `E` (element) — but descriptive names (`TItem`, `TResponse`) are fine and often clearer in complex signatures.

**Generic function** — A function with type parameters. The magic is **inference at the call site**: `first([1, 2, 3])` infers `T = number` automatically. You almost never write `first<number>([1,2,3])` explicitly — if you find yourself doing it constantly, the signature may be poorly designed.

**Generic interface / type alias** — Reusable shapes parameterized by type: `interface ApiResponse<T> { data: T; status: number }`. Unlike functions, *usage sites must supply the argument*: `ApiResponse<User>`.

**Generic class** — Same idea on classes: `class Stack<T>` with `push(item: T)` / `pop(): T | undefined`. The parameter is fixed per instance: a `Stack<string>` accepts only strings forever.

**Constraint (`extends`)** — Restricts what a type parameter can be, granting you capabilities inside: `<T extends { id: string }>` means "any T, as long as it has a string id" — and now `item.id` type-checks in the body. Unconstrained `T` supports almost no operations (that's the point: it must be *anything*).

**`keyof` with generics** — `<T, K extends keyof T>` links a key parameter to an object parameter, giving perfectly typed property access: `getProp(user, "name")` returns the *exact type of `name`*, and `getProp(user, "nmae")` is a compile error. This duo is the backbone of typed data utilities.

**Default type parameters** — `interface Box<T = string>` lets `Box` mean `Box<string>` when unspecified.

**Generic type argument vs. union parameter** — `function f<T extends string | number>(x: T)` vs `function f(x: string | number)`: the generic *remembers which one* (return types can depend on it); the union forgets. Use a union unless something in the signature depends on the specific type — unnecessary generics are noise (see pitfalls).

## Code Examples

### The motivating example

```ts
// ❌ Duplication:
function firstString(arr: string[]): string | undefined { return arr[0]; }
function firstNumber(arr: number[]): number | undefined { return arr[0]; }

// ❌ any — output type severed from input:
function firstAny(arr: any[]): any { return arr[0]; }
const x = firstAny([1, 2, 3]); // x: any — checking off, autocomplete dead

// ✅ Generic — one function, relationship preserved:
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const n = first([10, 20, 30]);        // n: number | undefined   (T inferred = number)
const s = first(["a", "b"]);          // s: string | undefined   (T = string)
const u = first([{ id: 1 }]);         // u: { id: number } | undefined

n?.toFixed(1);   // OK — the compiler KNOWS n is a number if defined
s?.toFixed(1);
// Error: Property 'toFixed' does not exist on type 'string'.
```

### Multiple type parameters

```ts
// Relate two types — here, transforming element types:
function map<T, U>(items: T[], fn: (item: T) => U): U[] {
  const out: U[] = [];
  for (const item of items) out.push(fn(item));
  return out;
}

const lengths = map(["apple", "fig"], s => s.length);
// T = string inferred from array, U = number inferred from the callback's return.
// lengths: number[]

// Building a pair:
function pair<A, B>(a: A, b: B): [A, B] {
  return [a, b];
}
const p = pair("age", 30); // p: [string, number]
```

### Constraints: demanding capabilities

```ts
// Unconstrained T allows almost nothing:
function longest<T>(a: T, b: T) {
  return a.length > b.length ? a : b;
  // Error: Property 'length' does not exist on type 'T'.
}

// Constrain to types that HAVE length:
function longest2<T extends { length: number }>(a: T, b: T): T {
  return a.length > b.length ? a : b;
}

longest2("alice", "bob");        // OK — strings have length; returns string
longest2([1, 2, 3], [4, 5]);     // OK — arrays have length; returns number[]
longest2(10, 20);
// Error: Argument of type 'number' is not assignable to
// parameter of type '{ length: number; }'.
```

### The `keyof` power combo

```ts
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: "u1", name: "Ada", age: 36 };

const name = getProp(user, "name"); // name: string   — exact property type!
const age = getProp(user, "age");   // age: number

getProp(user, "email");
// Error: Argument of type '"email"' is not assignable to
// parameter of type '"id" | "name" | "age"'.   ← keyof produced this union

// Realistic use — a typed sort-by:
function sortBy<T, K extends keyof T>(items: T[], key: K): T[] {
  return [...items].sort((a, b) => (a[key] < b[key] ? -1 : 1));
}
sortBy([user], "age");    // OK
sortBy([user], "height"); // Error — typo-proof refactoring-proof sorting
```

### Generic interfaces and type aliases

```ts
interface ApiResponse<T> {
  data: T;
  status: number;
  fetchedAt: Date;
}

interface Paginated<T> {
  items: T[];
  page: number;
  totalPages: number;
}

interface User { id: string; name: string }

// Compose them — generics nest naturally:
type UserListResponse = ApiResponse<Paginated<User>>;

declare const res: UserListResponse;
res.data.items[0]?.name;   // fully typed all the way down

// Generic Result (from Chapter 5, now reusable for ANY value/error):
type Result<T, E = string> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function tryParseJson(text: string): Result<unknown, Error> {
  try {
    return { ok: true, value: JSON.parse(text) };
  } catch (e) {
    return { ok: false, error: e instanceof Error ? e : new Error(String(e)) };
  }
}
```

### A generic class

```ts
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  get size(): number {
    return this.items.length;
  }
}

const history = new Stack<string>();
history.push("/home");
history.push("/settings");
const last = history.pop();  // last: string | undefined

history.push(42);
// Error: Argument of type 'number' is not assignable to parameter of type 'string'.
```

### Generic functions as parameters & default parameters

```ts
// Accepting a generic-shaped callback:
function firstMatching<T>(items: T[], predicate: (item: T) => boolean): T | undefined {
  for (const item of items) if (predicate(item)) return item;
  return undefined;
}

// Defaults — Events can carry any payload, default none:
interface AppEvent<TPayload = void> {
  name: string;
  payload: TPayload;
}
type Click = AppEvent<{ x: number; y: number }>;
type Logout = AppEvent;            // payload: void
```

## Common Pitfalls

**Pitfall 1: `any` in generic clothing — the useless generic.**

```ts
// ❌ T appears ONCE — it relates nothing to nothing:
function log<T>(value: T): void { console.log(value); }
// ✅ Same safety, less ceremony:
function log2(value: unknown): void { console.log(value); }
```

Rule of thumb (from the TS docs): a type parameter should appear **at least twice** — it exists to *relate* positions (param↔return, param↔param). If it appears once, delete it.

**Pitfall 2: Assuming `T` has properties without a constraint.**
Inside an unconstrained generic, `T` could be `number`, `null`, anything — so `value.name` errors. That error is correct! Either constrain (`T extends { name: string }`) or reconsider whether you need a generic at all.

**Pitfall 3: Returning a concrete type where `T` is promised.**

```ts
function makeDefault<T extends { id: string }>(): T {
  return { id: "new" };
  // Error: '{ id: string }' is assignable to the constraint of type 'T',
  // but 'T' could be instantiated with a different subtype.
}
```

The caller chooses `T` — they might pick `{ id: string; name: string }`, and your `{ id: "new" }` wouldn't satisfy it. If the function *produces* the value, it usually shouldn't be generic over it (return the concrete type), or it must receive a factory for `T`.

**Pitfall 4: Over-genericizing.**
JS developers discovering generics generify everything. Each parameter adds reading cost. Ask: "does any output type *depend on* an input type?" No → union or concrete types. `function len<T extends string | unknown[]>(x: T): number` gains nothing over `function len(x: string | unknown[]): number`.

**Pitfall 5: Fighting inference with explicit arguments.**
Writing `first<number>([1, 2, 3])` everywhere signals mistrust of inference (or a signature that hides inference sites in bad positions). Explicit type arguments are appropriate mainly when there's nothing to infer from — e.g., `new Stack<string>()` on an empty container, or `tryParse<Config>(text)`-style APIs.

**Pitfall 6: Expecting generics to exist at runtime.**
`T` is erased like every other type. You cannot `if (typeof T === ...)`, cannot `new T()`, cannot make runtime decisions based on the type argument. If behavior must vary by type, pass a value (a factory, a discriminant string, a validator) — values survive to runtime; types don't.

## Practice Exercises

1. **Utility warm-ups.** Implement with full generic typing (no `any`): `last<T>(arr: T[]): T | undefined`; `unique<T>(arr: T[]): T[]`; `zip<A, B>(a: A[], b: B[]): [A, B][]`. Verify inferred result types with hovers on at least two differently-typed calls each.

2. **Constrained lookup.** Implement `pluck<T, K extends keyof T>(items: T[], key: K): T[K][]` (extract one property from every object). Show a call on an array of `{ id: number; title: string }` returning `string[]`, and a misspelled key producing a compile error naming the valid keys.

3. **Generic cache shape.** Define `interface Cache<K, V>` with methods `get(key: K): V | undefined`, `set(key: K, value: V): void`, `has(key: K): boolean`, and implement it as `class MemoryCache<K, V>` backed by a `Map`. Instantiate a `MemoryCache<string, { count: number }>` and prove wrong value shapes are rejected. (This becomes the seed of Project 3.)

4. **Result, generalized.** Using the generic `Result<T, E>` from this chapter, write `safeDivide(a: number, b: number): Result<number>` and `parseAge(input: string): Result<number, "not-a-number" | "out-of-range">`. Write one consumer for each that the compiler forces to handle the error branch.

5. **Spot the fake generic.** For each signature, say whether the type parameter earns its place, and rewrite the ones that don't: (a) `identity<T>(x: T): T`; (b) `count<T>(xs: T[]): number`; (c) `defaults<T>(partial: Partial<T>, fallback: T): T` *(Partial arrives in Chapter 9 — reason from the name)*; (d) `toStr<T extends number | boolean>(x: T): string`. Justify each verdict in one sentence.
