# Chapter 2: Basic Types & Type Annotations

## Overview

This chapter covers the atoms of TypeScript: the primitive types, arrays, objects, and the special types (`any`, `unknown`, `void`, `null`, `undefined`, `never`) — plus *when to annotate and when to let inference work*. Everything in later chapters is built from these pieces, so getting the mental model right here pays off everywhere.

The core skill is not memorizing syntax (there's little of it) — it's developing judgment about **where types come from**: sometimes you declare them, but most of the time TypeScript infers them, and good TypeScript code annotates only where it adds safety or clarity.

## Definitions & Explanations

**Type annotation** — Explicitly writing a type after a colon: `let name: string`. You're telling the compiler "hold me to this."

**Type inference** — The compiler deducing a type from context: `let name = "Ada"` infers `string`. Inference is not a lesser form of typing — an inferred type is checked exactly as strictly as an annotated one.

**The primitives** — TypeScript mirrors JavaScript's primitives, all lowercase:
- `string`, `number`, `boolean` — the big three.
- `bigint`, `symbol` — same as their JS counterparts.
- `null` and `undefined` — each is its own type with exactly one value. Under strict mode (`strictNullChecks`), `null`/`undefined` are *not* assignable to other types — this single feature eliminates the most common JS bug class.

**`any`** — The escape hatch. A value of type `any` opts out of type checking entirely: you can call it, index it, assign it anywhere. `any` is *contagious* — operations on `any` produce more `any`. Treat it as a code smell.

**`unknown`** — The type-safe counterpart of `any`. It means "could be anything," but unlike `any`, you can't *do* anything with it until you narrow it (Chapter 6). Prefer `unknown` for genuinely unknown data (user input, parsed JSON, caught errors).

**`void`** — The return type of a function that returns nothing useful. Slightly different from `undefined` in ways that rarely matter day-to-day; use `void` for "no meaningful return value."

**`never`** — The type of values that cannot exist: a function that always throws, an infinite loop's "return," or the type left over after all possibilities are narrowed away. It signals impossibility and powers exhaustiveness checking (Chapter 6).

**Array types** — `number[]` or the equivalent `Array<number>`. Both mean "an array whose elements are all numbers, of any length."

**Object type annotations** — Inline shapes like `{ name: string; age: number }`. Properties can be optional (`age?: number`) or read-only (`readonly id: string`). Chapter 3 covers naming these shapes with interfaces/aliases.

**Literal types** — A specific value used as a type: `"north"`, `42`, `true`. `let dir: "north" | "south"` allows exactly two values. Covered in depth in Chapter 5, but you'll see them immediately because of...

**Widening** — `let x = "hi"` infers `string` (x can be reassigned), but `const x = "hi"` infers the literal type `"hi"` (it can never change). This `let`-widens/`const`-narrows behavior surprises everyone once.

## Code Examples

### Primitives and inference

```ts
// Annotated (explicit):
let username: string = "ada";
let attempts: number = 3;
let isAdmin: boolean = false;

// Idiomatic (inferred — identical safety, less noise):
let username2 = "ada";     // string
let attempts2 = 3;         // number
let isAdmin2 = false;      // boolean

username2 = 42;
// Error: Type 'number' is not assignable to type 'string'.
```

### const vs let inference (widening)

```ts
let mode = "dark";    // type: string        (could be reassigned)
const mode2 = "dark"; // type: "dark"        (literal — can never change)

// This matters when passing values to functions expecting literals:
function setTheme(theme: "dark" | "light") { /* ... */ }

setTheme(mode2); // OK: "dark" is assignable to "dark" | "light"
setTheme(mode);
// Error: Argument of type 'string' is not assignable to
// parameter of type '"dark" | "light"'.
```

### Arrays

```ts
const scores: number[] = [90, 85, 77];
const names = ["Ada", "Grace"];        // inferred: string[]

scores.push("100");
// Error: Argument of type 'string' is not assignable to parameter of type 'number'.

// Mixed arrays need a union element type (Chapter 5):
const mixed: (number | string)[] = [1, "two", 3];

// Empty arrays need help — there's nothing to infer from:
const queue = [];            // inferred: any[]  ⚠️ avoid
const queue2: string[] = []; // ✅ tell it what it will hold
```

### Object shapes, optional and readonly properties

```ts
// Inline object type annotation:
let point: { x: number; y: number } = { x: 0, y: 0 };

point.z = 5;
// Error: Property 'z' does not exist on type '{ x: number; y: number }'.

// Optional property — may be absent, and if present may need checking:
function describe(user: { name: string; nickname?: string }) {
  // user.nickname is: string | undefined
  console.log(user.nickname?.toUpperCase() ?? user.name);
}

describe({ name: "Grace" });                      // OK — nickname omitted
describe({ name: "Grace", nickname: "Amazing" }); // OK

// readonly — compile-time immutability:
const config: { readonly apiUrl: string } = { apiUrl: "https://api.example.com" };
config.apiUrl = "http://evil.com";
// Error: Cannot assign to 'apiUrl' because it is a read-only property.
```

### null and undefined under strict mode

```ts
// strictNullChecks makes null/undefined explicit citizens:
let title: string = "Hello";
title = null;
// Error: Type 'null' is not assignable to type 'string'.

// If null is genuinely possible, say so with a union:
let selected: string | null = null;
selected = "item-4"; // OK

// The compiler then forces you to handle it:
console.log(selected.toUpperCase());
// Error: 'selected' is possibly 'null'.

if (selected !== null) {
  console.log(selected.toUpperCase()); // OK — narrowed (Chapter 6)
}
```

### any vs unknown — the crucial difference

```ts
// any: checking is OFF. All of these compile — and all can crash at runtime.
let a: any = JSON.parse('"just a string"');
a.foo.bar;        // compiles ✔ crashes at runtime ✘
a();              // compiles ✔ crashes at runtime ✘
const n: number = a; // compiles ✔ — the lie spreads into n

// unknown: checking is ON. You must prove what it is first.
let u: unknown = JSON.parse('"just a string"');
u.toUpperCase();
// Error: 'u' is of type 'unknown'.

if (typeof u === "string") {
  u.toUpperCase(); // OK — inside this block, u is string
}
```

### void and never

```ts
// void — returns nothing worth using:
function logEvent(name: string): void {
  console.log(`event: ${name}`);
}

// never — never returns at all:
function fail(message: string): never {
  throw new Error(message);
}

function processValue(value: string | number) {
  if (typeof value === "string") return value.trim();
  if (typeof value === "number") return value.toFixed(2);
  // Here, value has type never — every possibility is handled.
  return fail("unreachable");
}
```

### When to annotate — practical rules

```ts
// ✅ ANNOTATE: function parameters (inference can't help — callers vary)
function area(width: number, height: number) {
  return width * height; // ✅ DON'T annotate obvious returns — inferred
}

// ✅ ANNOTATE: empty containers and variables initialized later
let result: string;
const cache: Map<string, number> = new Map();

// ✅ ANNOTATE: exported/public API boundaries (locks in your contract)
export function parsePrice(input: string): number { /* ... */ return 0; }

// ❌ DON'T annotate what inference already knows — it's noise:
const count: number = 5;        // ❌ redundant
const count2 = 5;               // ✅
```

## Common Pitfalls

**Pitfall 1: Using `any` to silence errors.**
This is the #1 habit that ruins TypeScript codebases. Every `any` is a hole through which runtime bugs return.

```ts
// ❌ "It works now"
function handle(data: any) { return data.items[0].name; }

// ✅ Describe what you actually expect
function handle2(data: { items: { name: string }[] }) {
  return data.items[0]?.name;
}
```

If you truly don't know the shape yet, use `unknown` and narrow.

**Pitfall 2: Using uppercase wrapper types.**
JS developers see `String` and `Number` (the constructor functions) and use them as types. Don't — they refer to the boxed object wrappers, which are almost never what you mean.

```ts
// ❌
let name: String = "Ada";
// ✅
let name2: string = "Ada";
```

**Pitfall 3: Forgetting optional means "possibly undefined."**

```ts
function greet(user: { nickname?: string }) {
  // ❌ Compiles under loose configs, crashes when nickname is absent:
  // return user.nickname.trim();
  // Error (strict): 'user.nickname' is possibly 'undefined'.

  // ✅
  return user.nickname?.trim() ?? "friend";
}
```

**Pitfall 4: Annotating everything.**
Newcomers from Java/C# habits (or just anxiety) annotate every variable. This buries meaningful annotations under noise and can even *hide bugs* by silencing inference of more specific types. Annotate parameters, public APIs, and empty containers; infer the rest.

**Pitfall 5: Expecting `const` objects to be deeply immutable.**
`const` prevents reassignment of the *binding*, not mutation — same as JS. And `readonly` is compile-time only.

```ts
const user = { name: "Ada" };
user.name = "Grace";   // fine — const doesn't freeze contents
// For compile-time deep freezing of literals, `as const` exists (Chapter 5).
```

**Pitfall 6: Trusting `JSON.parse` and `fetch`.**
`JSON.parse` returns `any`. Assigning it straight into a typed variable *feels* safe but checks nothing — the type is an unverified claim. Assign to `unknown` and validate, or use a validation library (Chapter 12).

## Practice Exercises

1. **Inference audit.** Write ten variable declarations mixing `let`/`const`, primitives, arrays, and object literals, with zero annotations. Predict each inferred type on paper, then verify by hovering in your editor. Pay special attention to `let` vs `const` string/number declarations.

2. **Shape a profile.** Write a function `formatProfile` that takes one parameter typed inline as an object with: required `username: string`, required `followers: number`, optional `bio: string`, and readonly `id: string`. Return a one-line summary string. Demonstrate (in comments) two calls that compile and two that produce compiler errors, quoting the errors.

3. **`unknown` gauntlet.** Write a function `describeValue(value: unknown): string` that returns `"text: <value>"` for strings, `"number: <value>"` for numbers, `"array of N items"` for arrays, and `"something else"` otherwise — using only `typeof` and `Array.isArray` checks. It must compile under strict mode with no `any` and no assertions.

4. **Null discipline.** Write `function findFirstLong(words: string[], minLength: number): string | null` that returns the first word at least `minLength` long, or `null`. Then write a second function that calls it and safely prints the result's uppercase form or `"none found"` — the compiler must force you to handle the `null` case.

5. **De-noise a file.** Take this over-annotated snippet and rewrite it with the minimal set of annotations that keeps it fully type-safe under strict mode; justify each annotation you keep:
   ```ts
   const rate: number = 0.2;
   const label: string = "VAT";
   const amounts: number[] = [10, 20, 30];
   const total: number = amounts.reduce((acc: number, n: number): number => acc + n, 0);
   const message: string = `${label}: ${(total * rate).toFixed(2)}`;
   ```
