# Chapter 5: Unions, Intersections & Literal Types

## Overview

This chapter is where TypeScript stops looking like "JavaScript with labels" and starts modeling *real domain logic*. Union types express "one of these"; intersection types express "all of these at once"; literal types shrink `string`/`number` down to exact values. Combined, they let you encode business rules — "an order is pending, shipped, or cancelled, and only shipped orders have a tracking number" — so precisely that invalid states *cannot compile*.

The crown jewel is the **discriminated union**, the single most important pattern in production TypeScript. Master it here; Chapter 6 (narrowing) shows how the compiler works with you to consume these types safely.

## Definitions & Explanations

**Union type (`A | B`)** — A value that is one of several types. `string | number` holds a string *or* a number. Crucially, until you determine which one it is, you may only do things valid for *every* member of the union. This restriction is a feature: it forces you to handle all cases.

**Literal type** — A single exact value as a type: `"pending"`, `404`, `true`. Alone they're trivial; in unions they shine: `type Status = "pending" | "shipped" | "cancelled"` is a self-documenting, typo-proof enum-like type (and the idiomatic alternative to `enum` — Chapter 10).

**Intersection type (`A & B`)** — A value that satisfies *all* constituents simultaneously. `Timestamped & Named` has every property of both. Intersections compose object types; intersecting incompatible primitives (`string & number`) produces `never` (impossible).

**Discriminated union (tagged union)** — A union of object types that each carry a common literal-typed property (the *discriminant*, conventionally `kind`, `type`, or `status`). Checking that one property tells the compiler exactly which member you have, unlocking its specific properties. This is how you model state machines, API results, events, and Redux-style actions.

**`as const`** — A suffix assertion that makes TypeScript infer the *narrowest* possible type: literals stay literal, arrays become readonly tuples, objects become deeply readonly. Essential for keeping literal types from widening to `string`/`number`.

**Widening (recap from Ch. 2)** — Mutable positions widen literals: `let s = "up"` is `string`, not `"up"`. Function calls expecting `"up" | "down"` then reject it. Fixes: `const`, `as const`, or an explicit annotation.

**Union of function results / nullable types** — `T | null` and `T | undefined` are just unions; everything in this chapter (and narrowing in the next) applies to them uniformly.

## Code Examples

### Unions: only common operations allowed

```ts
function formatId(id: string | number) {
  // Allowed: things BOTH string and number support.
  console.log(id.toString());   // OK — both have toString

  id.toUpperCase();
  // Error: Property 'toUpperCase' does not exist on type 'string | number'.
  //   Property 'toUpperCase' does not exist on type 'number'.

  // To use member-specific operations, narrow first (Chapter 6):
  if (typeof id === "string") {
    return id.toUpperCase();  // id: string here
  }
  return id.toFixed(0);       // id: number here
}
```

### Literal unions replace magic strings

```ts
// ❌ JavaScript habit — any typo compiles, fails silently at runtime:
function alignJs(direction: string) { /* ... */ }
alignJs("lfet"); // oops — runs, does nothing

// ✅ TypeScript — the type IS the documentation and the validation:
type Direction = "left" | "right" | "center";

function align(direction: Direction) { /* ... */ }

align("left");  // OK — and your editor autocompletes the options
align("lfet");
// Error: Argument of type '"lfet"' is not assignable to parameter of type 'Direction'.
```

### Widening bites, `as const` fixes

```ts
type Method = "GET" | "POST";
declare function request(url: string, method: Method): void;

const req1 = { url: "/api", method: "POST" };
request(req1.url, req1.method);
// Error: Argument of type 'string' is not assignable to parameter of type 'Method'.
// Why: object properties are mutable → "POST" widened to string.

// Fix 1 — as const freezes the literals (and makes properties readonly):
const req2 = { url: "/api", method: "POST" } as const;
request(req2.url, req2.method); // OK — method is type "POST"

// Fix 2 — annotate the object with a proper type:
const req3: { url: string; method: Method } = { url: "/api", method: "POST" };
request(req3.url, req3.method); // OK
```

### Intersections: composing capabilities

```ts
type Timestamped = { createdAt: Date; updatedAt: Date };
type SoftDeletable = { deletedAt: Date | null };

type Article = { title: string; body: string } & Timestamped & SoftDeletable;

const post: Article = {
  title: "Hello",
  body: "...",
  createdAt: new Date(),
  updatedAt: new Date(),
  deletedAt: null,
}; // must satisfy ALL parts — omit any property and it errors

// Intersection of incompatible primitives = never (no possible value):
type Impossible = string & number; // never
```

### The discriminated union — the pattern to internalize

```ts
// Each member: a literal 'status' discriminant + only the fields valid for that state.
type FetchState =
  | { status: "idle" }
  | { status: "loading"; startedAt: number }
  | { status: "success"; data: string[] }
  | { status: "error"; message: string; retryable: boolean };

function render(state: FetchState): string {
  // Checking the discriminant narrows to the exact member:
  switch (state.status) {
    case "idle":
      return "Ready.";
    case "loading":
      return `Loading since ${state.startedAt}…`;   // startedAt exists ONLY here
    case "success":
      return `Got ${state.data.length} items`;      // data exists ONLY here
    case "error":
      return state.retryable ? `Retrying: ${state.message}` : state.message;
  }
}

// Invalid states are UNREPRESENTABLE — this cannot be constructed:
const bad: FetchState = { status: "success", message: "?" };
// Error: Object literal may only specify known properties…
// 'message' does not exist in type '{ status: "success"; data: string[] }'.
```

Compare with the "bag of optionals" anti-pattern from Chapter 3 — `{ loading?: boolean; data?: string[]; error?: string }` — where `{ loading: true, error: "x" }` is a perfectly legal lie. Discriminated unions make the impossible impossible.

### Modeling operation results (the Result pattern)

```ts
type ParseResult =
  | { ok: true; value: number }
  | { ok: false; error: string };

function parsePrice(input: string): ParseResult {
  const n = Number(input);
  return Number.isFinite(n) && n >= 0
    ? { ok: true, value: n }
    : { ok: false, error: `Invalid price: "${input}"` };
}

const result = parsePrice("19.99");
result.value;
// Error: Property 'value' does not exist on type 'ParseResult'.
// The compiler FORCES the check:
if (result.ok) {
  console.log(result.value * 100);  // ok: true branch → value exists
} else {
  console.error(result.error);      // ok: false branch → error exists
}
```

### Unions of literal numbers, template literal types

```ts
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
type HttpSuccess = 200 | 201 | 204;

// Template literal types build string unions from other unions:
type Size = "small" | "large";
type Variant = "primary" | "danger";
type ClassName = `btn-${Size}-${Variant}`;
// "btn-small-primary" | "btn-small-danger" | "btn-large-primary" | "btn-large-danger"

const c: ClassName = "btn-small-danger"; // OK
const d: ClassName = "btn-medium-primary";
// Error: not assignable to type 'ClassName'.
```

## Common Pitfalls

**Pitfall 1: Reaching for `string` when a literal union is the truth.**
If a parameter can only meaningfully be one of a handful of values, `string` is a lie that disables autocomplete and typo-catching. Enumerate the values.

**Pitfall 2: Expecting union members' properties to be directly accessible.**

```ts
type Shape = { kind: "circle"; radius: number } | { kind: "square"; side: number };

function area(s: Shape) {
  return Math.PI * s.radius ** 2;
  // Error: Property 'radius' does not exist on type 'Shape'.
  // ❌ Beginners then write: (s as any).radius — destroying the safety.
  // ✅ Narrow on the discriminant:
  // return s.kind === "circle" ? Math.PI * s.radius ** 2 : s.side ** 2;
}
```

**Pitfall 3: Confusing `|` and `&` on object types.**
Intuition trips people here: `A | B` (union) is *fewer guarantees* (could be either, so you can only use common members), while `A & B` (intersection) is *more guarantees* (has everything). "Or means fewer usable properties; And means more" — say it until it sticks.

**Pitfall 4: Bags of optionals instead of discriminated unions.**
Covered above — if certain properties only make sense together, encode that as union members, not as independent `?` fields. Interview-relevant: this is often called "making illegal states unrepresentable."

**Pitfall 5: Forgetting that mutation breaks literal inference.**

```ts
let method = "GET";           // string — let widens
const methods = ["GET", "POST"];        // string[] — array elements widen too
const methods2 = ["GET", "POST"] as const; // readonly ["GET", "POST"] ✅
```

When a "list of allowed values" needs to stay literal (e.g., to derive a type from it: `type Method = typeof methods2[number]`), you need `as const`.

**Pitfall 6: Intersecting types with conflicting members.**

```ts
type A = { id: string };
type B = { id: number };
type AB = A & B;   // compiles! but id: string & number = never
const x: AB = { id: 1 };
// Error: Type 'number' is not assignable to type 'never'.
```

The intersection itself doesn't error — the impossibility surfaces only when you try to construct a value. If an intersection suddenly demands `never`, look for a member-type conflict.

## Practice Exercises

1. **Payment methods.** Model `PaymentMethod` as a discriminated union: `card` (last4: string, expiry: string), `paypal` (email: string), `banktransfer` (iban: string, verified: boolean). Write `describePayment(p: PaymentMethod): string` using a `switch` on the discriminant. Confirm that accessing `email` outside the `paypal` branch is a compiler error.

2. **Widen and rescue.** Write a `move(direction: "up" | "down" | "left" | "right")` function. Then create the situations where a call fails due to widening — via a `let` variable and via an object property — and fix each with a different technique (`const`, annotation, `as const`).

3. **Result, not throw.** Write `divide(a: number, b: number)` returning `{ ok: true; value: number } | { ok: false; error: "division-by-zero" }`. Then write a caller that computes `divide(10, x)` and logs the value doubled — proving the compiler forces you to handle the failure branch before touching `value`.

4. **Compose with intersections.** Define `WithId`, `WithTimestamps`, and `WithAuthor` as small object types, then build `Comment` and `Post` types from them via `&`. Deliberately create a conflict (give two parts different types for the same property name) and observe where and how the error appears.

5. **Template literal API.** Using template literal types, define `Route` as all combinations of `"GET" | "POST"` plus a space plus `"/users" | "/posts" | "/login"` (e.g., `"GET /users"`). Write `handle(route: Route)` and verify autocomplete offers all six values while `"DELETE /users"` is rejected.
