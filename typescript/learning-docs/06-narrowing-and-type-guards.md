# Chapter 6: Narrowing & Type Guards

## Overview

Chapter 5 taught you to build precise union types. This chapter teaches the other half of the loop: **consuming** them. *Narrowing* is TypeScript's ability to refine a broad type into a more specific one by analyzing your ordinary runtime checks — `typeof`, `===`, `in`, `instanceof`, truthiness, early returns. You mostly write the JavaScript you'd write anyway; the compiler follows along, shrinking the type as control flow proceeds.

Beyond the built-in checks, you can teach the compiler new checks with **custom type guards** (`value is Type`), **assertion functions** (`asserts value is Type`), and get whole-program safety nets via **exhaustiveness checking** with `never`. This chapter is the day-to-day working experience of TypeScript — the constant, quiet conversation between your `if` statements and the checker.

## Definitions & Explanations

**Narrowing** — Refinement of a variable's type within a region of code, based on checks the compiler understands. Also called *control-flow analysis*: TypeScript tracks each variable's possible types through branches, assignments, returns, and throws.

**Type guard** — Any expression the compiler recognizes as narrowing. The built-ins:

- **`typeof` guard** — `typeof x === "string"` narrows to `string`. Remember JS quirks apply: `typeof null === "object"`, `typeof [] === "object"`.
- **Equality guard** — `x === "loading"`, `x === null`, `x !== undefined`. Comparing to a literal narrows unions to matching members. This is what powers discriminated-union `switch`es.
- **Truthiness guard** — `if (x)` removes `null`, `undefined`, `""`, `0`, `NaN`, `false` from the type. Convenient but dangerous when falsy values are *valid* (see pitfalls).
- **`in` guard** — `"radius" in shape` narrows object unions by property presence.
- **`instanceof` guard** — Narrows by prototype chain; works for classes and built-ins (`Date`, `Error`, `Map`), *not* for interfaces/type aliases (erased!).
- **`Array.isArray`** — Narrows to an array type.

**Discriminant narrowing** — Checking a shared literal property (`state.status === "error"`) narrows a discriminated union to the matching member(s). The most important guard in application code.

**Custom type guard (type predicate)** — A function returning `value is SomeType` instead of `boolean`. When it returns true, the compiler narrows the argument at the call site. You write the runtime logic; the predicate tells the compiler what a `true` result *means*. Power and responsibility: if your logic is wrong, the compiler now believes a lie.

**Assertion function** — Declared `asserts value is SomeType` (or `asserts value`): if it *returns* (doesn't throw), the compiler narrows from then on. Ideal for validation helpers: `assertIsUser(data); // data is User below this line`.

**Exhaustiveness checking** — In a `switch` over a discriminated union, after all cases are handled the value's type is `never`. Assigning it to a `never`-typed variable in the `default` branch makes the compiler *error when someone later adds a union member you forgot to handle*. This turns "we forgot to update the switch" from a runtime surprise into a compile failure.

**Narrowing invalidation** — Narrowing is per-code-path and can be lost: reassignment resets it, and closures (callbacks) may not preserve it for mutable (`let`) variables, because the callback could run after a reassignment.

## Code Examples

### The built-in guards, tour

```ts
function describe(input: string | number | string[] | null) {
  if (input === null) {
    return "nothing";              // input: null
  }
  if (typeof input === "string") {
    return input.toUpperCase();    // input: string
  }
  if (Array.isArray(input)) {
    return `${input.length} items`; // input: string[]
  }
  return input.toFixed(2);         // input: number — all that's left!
}
```

Note the last line: no check needed. Elimination is narrowing too — the compiler tracked what *remains*.

### Discriminant narrowing + exhaustiveness — the production pattern

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rect"; width: number; height: number }
  | { kind: "triangle"; base: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rect":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default: {
      // If every case is handled, shape is 'never' here and this compiles.
      // Add a 'kind: "ellipse"' member later WITHOUT a case → compile error here. 🎯
      const unhandled: never = shape;
      throw new Error(`Unhandled shape: ${JSON.stringify(unhandled)}`);
    }
  }
}
```

This `default: never` trick is the cheapest large-scale safety net TypeScript offers. Use it in every switch over a union you expect to grow.

### `in` and `instanceof`

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim();   // Fish
  } else {
    animal.fly();    // Bird
  }
}

function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toISOString()); // Date
  } else {
    console.log(x.trim());        // string
  }
}
```

### Truthiness: convenient and treacherous

```ts
function printLines(text: string | null) {
  if (text) {
    console.log(text.split("\n")); // string — null AND "" excluded
  }
}

// ⚠️ The classic bug — 0 is valid data but falsy:
function applyDiscount(percent: number | undefined) {
  if (percent) {                       // ❌ 0 falls into the "no discount" branch!
    console.log(`-${percent}%`);
  }
}
// ✅ Check for what you actually mean:
function applyDiscount2(percent: number | undefined) {
  if (percent !== undefined) {
    console.log(`-${percent}%`);       // 0% prints correctly
  }
}
```

### Custom type guards — teaching the compiler

```ts
interface User {
  id: string;
  name: string;
}

// Runtime check + compile-time meaning in one place:
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value && typeof (value as { id: unknown }).id === "string" &&
    "name" in value && typeof (value as { name: unknown }).name === "string"
  );
}

const data: unknown = JSON.parse('{"id":"u1","name":"Ada"}');

if (isUser(data)) {
  console.log(data.name.toUpperCase()); // data: User ✅
} else {
  console.log("not a user");            // data: unknown
}

// Type guards also power array filtering:
const inputs: (User | null)[] = [ { id: "u1", name: "Ada" }, null ];
const users = inputs.filter((u): u is User => u !== null);
// users: User[]  — without the predicate, filter would return (User | null)[]
```

### Assertion functions

```ts
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error(`Expected string, got ${typeof value}`);
  }
}

function shout(input: unknown) {
  assertIsString(input);
  // From here on, input: string — because if it weren't, we'd have thrown.
  return input.toUpperCase();
}
```

### Narrowing errors in catch blocks

```ts
// catch variables are 'unknown' (under useUnknownInCatchVariables, part of strict):
try {
  riskyOperation();
} catch (err) {
  err.message;
  // Error: 'err' is of type 'unknown'.

  if (err instanceof Error) {
    console.error(err.message);        // ✅ narrowed to Error
  } else {
    console.error("Unknown failure", err);
  }
}
declare function riskyOperation(): void;
```

### Where narrowing is lost

```ts
function process(id: string | null) {
  if (id === null) return;
  // id: string — narrowing survives the early return ✅

  id = null;
  // ⚠️ reassignment: id is string | null again (and here, just null)

  setTimeout(() => {
    // For 'let'/param variables, callbacks may not keep narrowing —
    // the compiler can't prove the callback runs before another reassignment.
  }, 100);
}

// ✅ Robust habit: copy into a const after narrowing.
function process2(id: string | null) {
  if (id === null) return;
  const safeId = id;            // const string — immune to invalidation
  setTimeout(() => console.log(safeId.length), 100); // fine
}
```

## Common Pitfalls

**Pitfall 1: `typeof x === "object"` believing it excluded `null`.**

```ts
function keys(x: unknown) {
  if (typeof x === "object") {
    // ❌ x is: object | null — typeof null is "object"!
    // Object.keys(x)  → Error: 'x' is possibly 'null'.
  }
  if (typeof x === "object" && x !== null) {
    Object.keys(x);   // ✅
  }
}
```

**Pitfall 2: Using `instanceof` with interfaces.**
Interfaces are erased; `data instanceof User` where `User` is an interface is a compile error ("'User' only refers to a type"). Use a custom type guard or `in` checks. This is the runtime/compile-time divide again — internalize it.

**Pitfall 3: Truthiness on numbers and strings.**
`if (count)` treats `0` as absent; `if (name)` treats `""` as absent. When falsy values are legitimate data, compare explicitly (`!== undefined`, `!== null`, `!= null` for both at once).

**Pitfall 4: Lying type predicates.**

```ts
// ❌ The compiler trusts you COMPLETELY — this "guard" checks nothing:
function isUser(v: unknown): v is User { return true; }
```

A wrong predicate is worse than `any` because it *looks* safe. Keep guards' runtime logic honest and test them. (Chapter 12 introduces schema validators like Zod that generate correct guards for you.)

**Pitfall 5: Skipping the exhaustiveness check "because the switch works."**
Without `default: const x: never = value`, adding a union member later silently falls through (often returning `undefined`). The check costs three lines and converts a class of future bugs into compile errors. Non-negotiable in professional code.

**Pitfall 6: Narrowing, then refactoring into a callback and losing it.**
Extracting `if`-guarded code into `.forEach`/`.map` callbacks can resurface "possibly undefined" errors that were previously narrowed. Copy the narrowed value to a `const` first (see `process2` above) rather than sprinkling `!`.

**Pitfall 7: The non-null assertion `!` as a reflex.**
`value!.foo` means "trust me, it's not null" — with zero runtime backing. Every `!` is a potential `TypeError` and a place where the compiler was overruled. Acceptable rarely (e.g., right after a check the compiler can't see); a habit of it defeats strict null checking entirely. Prefer narrowing, `?.`, `??`, or restructuring.

## Practice Exercises

1. **Guard gauntlet.** Write `stringify(value: string | number | boolean | Date | string[] | null): string` producing a sensible string for each case, using at least four different guard kinds (`typeof`, `instanceof`, `Array.isArray`, equality). No `any`, no assertions, no `!`.

2. **Exhaustive vehicles.** Define `Vehicle = { type: "car"; seats: number } | { type: "truck"; payloadKg: number } | { type: "motorcycle"; cc: number }` and write `describeVehicle` with a `switch` including the `never` exhaustiveness pattern. Then add a fourth member `{ type: "bus"; capacity: number }` and confirm the compiler pinpoints the unhandled case. Fix it.

3. **Honest guard.** Write `isNonEmptyStringArray(value: unknown): value is string[]` that verifies: it's an array, has length ≥ 1, and every element is a string. Demonstrate it narrowing an `unknown` from `JSON.parse`, and show one malformed input it correctly rejects at runtime.

4. **The zero bug.** Write `formatScore(score: number | undefined)` twice: once with a truthiness check (buggy for 0) and once correctly. Write the exact input where their behavior differs and the output each produces. One sentence: why did the buggy version type-check?

5. **Assertion pipeline.** Write `assertPositiveNumber(value: unknown): asserts value is number` (throws unless value is a number > 0). Use it in a `sqrt(input: unknown)` function that compiles cleanly and returns `Math.sqrt(input)` after the assertion. Then explain the difference in *caller experience* between this and a `isPositiveNumber` predicate version.
