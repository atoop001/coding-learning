# Chapter 10: Enums, Tuples & Advanced Object Typing

## Overview

This chapter rounds out your type-modeling toolkit with three clusters:

1. **Enums** — TypeScript's named-constant feature, notable as one of the few TS constructs that *generates runtime code*. You'll learn how they work, their sharp edges, and why modern codebases often prefer literal unions or `as const` objects instead. (Knowing *the debate itself* is interview-relevant.)
2. **Tuples** — fixed-length, per-position-typed arrays: coordinates, key-value pairs, React-style `[value, setter]` returns, labeled and optional elements, rest elements.
3. **Advanced object typing** — the techniques that make object types precise at scale: indexed access types (`T[K]`), `satisfies`, deeper `as const` patterns, and `noUncheckedIndexedAccess`.

## Definitions & Explanations

**Numeric enum** — `enum Direction { Up, Down }` — members auto-number from 0. Compiles to a real JS object with *reverse mappings* (`Direction[0] === "Up"`). Quirk-rich (see pitfalls).

**String enum** — `enum Level { Info = "INFO", Warn = "WARN" }` — every member explicitly a string. No reverse mapping, more predictable, debuggable values in logs. If you use enums at all, prefer string enums.

**`const enum`** — Erased at compile time; usages are inlined as literals. Zero runtime cost, but incompatible with some toolchains (Babel/esbuild's single-file transpilation, `isolatedModules` constraints). Many teams ban them; know they exist, avoid by default.

**The modern alternatives** — A literal union (`type Level = "info" | "warn"`) gives the same compile-time safety with zero runtime code; an `as const` object (`const Level = { Info: "info", … } as const`) additionally gives you a runtime value to iterate/validate against, with types derived via `keyof typeof`. These are what most current style guides recommend. Enums still appear widely in existing codebases (and Angular/Nest ecosystems), so you must *read* them fluently either way.

**Tuple type** — `[string, number]`: an array with a fixed length whose element types are fixed *per position*. Distinct from `(string | number)[]` (any length, any mix — order and length unchecked!).

**Labeled tuple elements** — `[lat: number, lng: number]`: labels are documentation only (they don't create properties) but dramatically improve signatures and editor hints.

**Optional & rest elements** — `[string, number?]` (length 1–2); `[string, ...number[]]` (first fixed, rest variable). Rest-in-tuples also types functions like `(...args: [string, number]) => void`.

**`readonly` tuples** — `readonly [number, number]`: no push/pop/mutation. `as const` on an array literal produces exactly this.

**Indexed access types** — `T[K]` reads a property's *type*: `User["email"]` → `string`; `Config["db"]["port"]` digs deep; `Items[number]` extracts an array's element type. Keeps derived types glued to their source.

**`satisfies`** — `expr satisfies SomeType` checks the expression against a type **without changing its inferred type**. The best of both: validation *and* precise inference. Contrast: an annotation (`const x: T = …`) validates but *widens* to `T`; `as const` narrows but validates nothing.

**`noUncheckedIndexedAccess`** — A compiler flag making index-signature and array-by-index reads return `T | undefined`, reflecting runtime truth. Stricter than default; recommended for new projects; pairs naturally with this chapter's material.

## Code Examples

### Enums — mechanics and quirks

```ts
// Numeric enum — generates a runtime object with REVERSE mappings:
enum Direction {
  Up,      // 0
  Down,    // 1
  Left,    // 2
  Right,   // 3
}

const d: Direction = Direction.Up;
console.log(Direction.Up);      // 0
console.log(Direction[0]);      // "Up" — reverse mapping exists in emitted JS

// String enum — no reverse mapping, log-friendly values:
enum LogLevel {
  Debug = "DEBUG",
  Info = "INFO",
  Warn = "WARN",
  Error = "ERROR",
}

function log(level: LogLevel, msg: string) {
  console.log(`[${level}] ${msg}`);
}

log(LogLevel.Warn, "disk almost full");   // "[WARN] disk almost full"
log("WARN", "hello");
// Error: Argument of type '"WARN"' is not assignable to parameter of type 'LogLevel'.
// ⚠️ String enums are NOMINAL-ish: even the exact right string is rejected —
// callers MUST import and use the enum. Feature or friction, depending on taste.
```

### The modern alternatives, side by side

```ts
// Alternative 1 — literal union: zero runtime code, structural, simple:
type LogLevelU = "debug" | "info" | "warn" | "error";
function logU(level: LogLevelU, msg: string) { /* ... */ }
logU("warn", "hi");           // plain strings work — no import ceremony

// Alternative 2 — as const object: when you ALSO need a runtime value
// (iteration, validation, display names):
const LOG_LEVELS = {
  debug: 0,
  info: 1,
  warn: 2,
  error: 3,
} as const;

type LogLevelK = keyof typeof LOG_LEVELS;              // "debug" | "info" | "warn" | "error"

function shouldLog(msgLevel: LogLevelK, minLevel: LogLevelK): boolean {
  return LOG_LEVELS[msgLevel] >= LOG_LEVELS[minLevel]; // runtime use of the object
}
const allLevels = Object.keys(LOG_LEVELS) as LogLevelK[]; // iterable at runtime
```

Decision rule: need only compile-time checking → literal union. Need runtime iteration/lookup too → `as const` object. Existing codebase uses enums → follow the codebase.

### Tuples — position-typed arrays

```ts
// Array vs tuple — the difference is everything:
const pointA: number[] = [3, 5, 8, 13];        // any length OK
const pointT: [number, number] = [3, 5];       // exactly two numbers

const bad: [number, number] = [3, 5, 8];
// Error: Type '[number, number, number]' is not assignable to type '[number, number]'.
//   Source has 3 element(s) but target allows only 2.

const mixed: [string, number, boolean] = ["age", 30, true];  // per-position types
const [label, value, visible] = mixed;
// label: string, value: number, visible: boolean — destructuring is typed! ✅

// Labeled elements — self-documenting signatures:
type GeoPoint = [lat: number, lng: number];
function distance(a: GeoPoint, b: GeoPoint): number { /* … */ return 0; }
// Editor shows: distance(a: [lat: number, lng: number], …) — no more "which is which?"
```

### Tuples as return values (the React-style pattern)

```ts
// Returning multiple values with distinct types:
function useToggle(initial: boolean): [boolean, () => void] {
  let state = initial;
  const toggle = () => { state = !state; };
  return [state, toggle];
}

const [isOpen, toggleOpen] = useToggle(false);
// isOpen: boolean, toggleOpen: () => void

// ⚠️ Without the return annotation, TS infers (boolean | (() => void))[]
// — an ARRAY, losing position information. Tuple returns usually need
// an explicit annotation or 'as const'. This is the #1 tuple gotcha.
```

### Optional, rest, and readonly tuple elements

```ts
type Range = [start: number, end?: number];          // length 1 or 2
const r1: Range = [5];
const r2: Range = [5, 10];

type Command = [name: string, ...args: string[]];    // 1+, first is the name
const c1: Command = ["git"];
const c2: Command = ["git", "commit", "-m", "msg"];

// Rest tuples type argument lists:
function run(...cmd: Command) { /* name = cmd[0] */ }
run("npm", "install", "typescript");

// readonly tuples & as const:
const ORIGIN = [0, 0] as const;      // readonly [0, 0]
ORIGIN.push(1);
// Error: Property 'push' does not exist on type 'readonly [0, 0]'.
```

### Indexed access types — types that follow the data

```ts
interface ApiConfig {
  baseUrl: string;
  auth: {
    kind: "bearer" | "basic";
    token: string;
  };
  endpoints: { name: string; path: string }[];
}

// Extract pieces WITHOUT redeclaring them:
type Auth = ApiConfig["auth"];                    // { kind: …; token: string }
type AuthKind = ApiConfig["auth"]["kind"];        // "bearer" | "basic"
type Endpoint = ApiConfig["endpoints"][number];   // { name: string; path: string }
//                                     ^^^^^^ [number] = "element type of the array"

function connect(auth: ApiConfig["auth"]) { /* … */ }
// If ApiConfig changes, connect's parameter follows automatically.
```

### `satisfies` — validate without widening

```ts
type RGB = readonly [number, number, number];
type ColorName = "primary" | "danger" | "muted";

// ❌ Annotation: validated but WIDENED — we lose which keys exist:
const palette1: Record<ColorName, RGB | string> = {
  primary: [0, 96, 255],
  danger: "#cc0022",
  muted: [90, 90, 90],
};
palette1.danger.toUpperCase();
// Error: Property 'toUpperCase' does not exist on type 'string | RGB'.
// The annotation forgot that danger is specifically a string. 😞

// ✅ satisfies: validated AND precisely inferred:
const palette2 = {
  primary: [0, 96, 255],
  danger: "#cc0022",
  muted: [90, 90, 90],
} satisfies Record<ColorName, RGB | string>;

palette2.danger.toUpperCase();   // OK — TS remembers danger is a string
palette2.primary[0].toFixed();   // OK — and primary is a number tuple

// And it still CHECKS — typos and wrong shapes are caught:
const palette3 = {
  primry: [0, 96, 255],
} satisfies Record<ColorName, RGB | string>;
// Error: 'primry' does not exist in type 'Record<ColorName, …>'.
```

### `noUncheckedIndexedAccess` — honest lookups

```ts
// With the flag OFF (default): TS pretends every lookup succeeds:
const scores: Record<string, number> = { ada: 10 };
const s = scores["nobody"];   // typed number — actually undefined at runtime! 💥

// With the flag ON: lookups are T | undefined; the compiler forces handling:
// const s: number | undefined
// s.toFixed();               // Error: 's' is possibly 'undefined'.
const s2 = scores["nobody"] ?? 0;   // ✅ handle it

// Arrays too:
const first = [10, 20, 30][99];     // number | undefined under the flag — true!
```

## Common Pitfalls

**Pitfall 1: Numeric enums accept arbitrary numbers (older TS) and reverse-map surprisingly.**
Historically `const d: Direction = 42` compiled (fixed in TS 5.0 for out-of-range literals, but computed values can still sneak through), and `Object.keys(Direction)` on a numeric enum yields *both* names and stringified numbers (`"0","1","Up","Down"…`) thanks to reverse mappings — a classic iteration bug. String enums and `as const` objects avoid both.

**Pitfall 2: Using enums when a literal union does the job.**
Extra import ceremony, runtime code, and nominal friction for no added safety. Default to literal unions; upgrade to an `as const` object when you need runtime iteration; use enums when the codebase already does.

**Pitfall 3: Tuple returns silently inferred as arrays.**
Shown above — `return [state, toggle]` without annotation infers a union-element *array*, and destructurers get the union type everywhere. Annotate the return type or `as const`. If a downstream type looks like `(A | B)[]` where you expected `[A, B]`, this is what happened.

**Pitfall 4: Using tuples where objects belong.**
`[string, string, number, boolean]` — quick, which one is "verified"? Positional typing beyond 2–3 elements is write-only code. Objects with named properties scale; tuples are for genuinely positional data (coordinates, pairs, fixed returns).

**Pitfall 5: `satisfies` vs `as` confusion.**
`as T` is an *assertion* — "compiler, trust me," checks almost nothing, can hide real errors. `satisfies T` is a *check* — full validation, no type change. When you're tempted to write `as` on a literal to "make it fit," `satisfies` is almost always what you actually want. Reserve `as` for genuine knowledge the compiler lacks (and prefer narrowing even then).

**Pitfall 6: Trusting index reads because the flag is off.**
`users[i].name` and `dict[key].total` are potential runtime crashes that default TS blesses. Either enable `noUncheckedIndexedAccess` (recommended for new code) or cultivate the habit of `?.` / explicit checks on dynamic lookups. Know your project's setting — it changes what the compiler guarantees.

## Practice Exercises

1. **Enum triage.** Write the same "order status" concept three ways: a string enum, a literal union, and an `as const` object with derived type. For each, demonstrate (a) a function accepting the type, (b) a valid call, (c) an invalid call with its error. Then write ≤4 sentences ranking the three for a brand-new codebase, with reasons.

2. **CSV row tuples.** Model a CSV user row as a labeled tuple `[id: number, email: string, active: boolean, tags?: string]`. Write `parseRow(cells: string[]): Row | null` performing the conversions with real validation, and `formatRow(row: Row): string` reversing it. Confirm destructuring a `Row` yields the right per-position types.

3. **Fix the inference.** Write `function minMax(nums: number[])` that returns the smallest and largest value. First return `[min, max]` with no annotation and inspect the inferred type; explain the problem in one sentence; then fix it two different ways (return annotation; `as const`) and note the resulting type of each fix.

4. **Config with satisfies.** Define `type Endpoint = { path: string; method: "GET" | "POST"; cache?: boolean }` and build a `const endpoints = { … } satisfies Record<string, Endpoint>` with 3+ entries. Show that (a) a typo'd method fails to compile, (b) `endpoints.users.path` autocompletes with the precise key names, which a `Record` annotation would have lost. Then derive `type EndpointName = keyof typeof endpoints`.

5. **Indexed-access safari.** Given a moderately nested `AppState` interface you design (user, settings, a list of notifications), derive via indexed access only: the notification element type, the settings theme type, and the type of `AppState["user"]["email"]`. Then enable `noUncheckedIndexedAccess` in a scratch tsconfig and record which of your existing lookups start erroring — fix each without `!`.
