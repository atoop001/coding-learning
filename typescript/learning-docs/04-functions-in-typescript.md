# Chapter 4: Functions in TypeScript

## Overview

Functions are where TypeScript earns its keep. Every function signature is a contract: what goes in, what comes out. Once parameter and return types are locked down, the compiler verifies every call site in the codebase — which is exactly the checking JavaScript never gave you.

This chapter covers parameter and return annotations, optional/default/rest parameters, typing functions as values (callbacks), `this` typing, async functions, and **overloads** — plus how TypeScript checks that one function type is safely usable where another is expected.

## Definitions & Explanations

**Parameter types** — Annotations on inputs: `function repeat(text: string, times: number)`. Parameters are the one place inference can't save you — without annotations they become `any` (an error under strict mode's `noImplicitAny`). *Always type parameters.*

**Return types** — Usually inferred accurately. Annotating them is optional but valuable on exported/public functions: the annotation is a promise, and if your implementation drifts, the error appears *in the function* rather than at 40 call sites.

**Optional parameters (`?`)** — `function f(a: string, b?: number)`. Inside the function, `b` is `number | undefined`. Optional parameters must come after required ones.

**Default parameters** — `function f(a: string, b = 10)`. The type of `b` is inferred (`number`) from the default; callers may omit it, but inside the function it is *never* undefined. Prefer defaults over optionals when a sensible fallback exists.

**Rest parameters** — `function sum(...nums: number[])`. Typed as an array (or tuple for fixed patterns, Chapter 10).

**Function type expressions** — The type *of a function*, used for callbacks and variables: `(input: string) => boolean`. Parameter names in the type are for documentation only; only positions and types matter.

**`void` return in callbacks** — A callback typed as returning `void` may actually return anything; the type just says "the caller will ignore it." This deliberate looseness lets `array.forEach(x => list.push(x))` compile even though `push` returns a number.

**Contextual typing** — When you pass an arrow function where a function type is expected, TypeScript infers the parameters *from the expected type* — that's why `names.map(n => n.length)` needs no annotation on `n`.

**Function overloads** — Multiple declared signatures for one implementation, used when the return type depends on the argument types in ways a single signature can't express. Overloads are a sharp tool; unions, optional parameters, and generics cover most cases more simply.

**`this` parameter** — TypeScript can type what `this` must be inside a function: `function handler(this: HTMLButtonElement, e: Event)`. It's a fake first parameter, erased at compile time.

**Async functions** — Always return `Promise<T>`. Annotate as `async function f(): Promise<string>`, never `: string`.

## Code Examples

### The bread and butter

```ts
// Parameters annotated, return inferred (string):
function repeat(text: string, times: number) {
  return text.repeat(times);
}

repeat("ab", 3);     // OK: "ababab"
repeat("ab");        // Error: Expected 2 arguments, but got 1.
repeat(3, "ab");     // Error: Argument of type 'number' is not assignable
                     //        to parameter of type 'string'.

// Exported functions: annotate the return as a stable contract.
export function slugify(title: string): string {
  return title.toLowerCase().replace(/\s+/g, "-");
}
```

### Optional vs default parameters

```ts
// Optional — absence is meaningful, no natural fallback:
function findUser(id: string, options?: { includeDeleted: boolean }) {
  // options is: { includeDeleted: boolean } | undefined
  const includeDeleted = options?.includeDeleted ?? false;
  // ...
}

// Default — there IS a natural fallback (prefer this when possible):
function paginate(items: string[], pageSize = 20) {
  // pageSize: number — never undefined in here
  return items.slice(0, pageSize);
}

paginate(["a", "b"]);       // OK, pageSize = 20
paginate(["a", "b"], 5);    // OK
paginate(["a", "b"], "5");  // Error: 'string' not assignable to 'number'.
```

### Rest parameters

```ts
function joinPath(...segments: string[]): string {
  return segments.join("/");
}

joinPath("usr", "local", "bin"); // "usr/local/bin"
joinPath();                      // OK — empty array
joinPath("usr", 2);              // Error on the 2
```

### Typing callbacks and contextual inference

```ts
// A named function type keeps signatures readable:
type Predicate<T> = (value: T) => boolean;   // (generics: Chapter 7)

function filterStrings(items: string[], keep: (value: string) => boolean) {
  return items.filter(keep);
}

// Contextual typing: 's' is inferred as string from the expected type —
// no annotation needed on the arrow function:
filterStrings(["a", "bb", "ccc"], s => s.length > 1);

filterStrings(["a", "bb"], (s: number) => s > 1);
// Error: Types of parameters 's' and 'value' are incompatible.
```

### The `void`-return looseness (and why)

```ts
declare function onTick(cb: () => void): void;

const results: number[] = [];
// This arrow returns number (push's return), but that's fine —
// void-typed callback means "return value is ignored", not "must return undefined":
onTick(() => results.push(1)); // OK

// But a declared void FUNCTION cannot return a value:
function tick(): void {
  return 5;
  // Error: Type 'number' is not assignable to type 'void'.
}
```

### Async functions

```ts
interface Post { id: number; title: string }

async function fetchPost(id: number): Promise<Post> {
  const res = await fetch(`/api/posts/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json() as Promise<Post>;  // json() returns Promise<any> — see Ch. 12
}

// await unwraps the promise:
async function main() {
  const post = await fetchPost(1);   // post: Post
  const pending = fetchPost(2);      // pending: Promise<Post>
  pending.title;
  // Error: Property 'title' does not exist on type 'Promise<Post>'.
}
```

### Function overloads — when one signature can't say it

```ts
// Goal: makeDate(timestamp) OR makeDate(year, month, day) — but never two args.
// A union signature can't forbid the two-argument call; overloads can.

// Overload signatures (what callers see):
function makeDate(timestamp: number): Date;
function makeDate(year: number, month: number, day: number): Date;
// Implementation signature (hidden from callers; must cover both):
function makeDate(a: number, b?: number, c?: number): Date {
  return b !== undefined && c !== undefined
    ? new Date(a, b - 1, c)
    : new Date(a);
}

makeDate(1700000000000);   // OK — first overload
makeDate(2026, 7, 17);     // OK — second overload
makeDate(2026, 7);
// Error: No overload expects 2 arguments, but overloads do exist
// that expect either 1 or 3 arguments.
```

```ts
// Overloads also model input-dependent RETURN types:
function firstElement(list: string): string;
function firstElement<T>(list: T[]): T | undefined;
function firstElement(list: string | unknown[]) {
  return list[0];
}

const c = firstElement("hello");   // c: string
const n = firstElement([1, 2, 3]); // n: number | undefined
```

Rule of thumb: reach for overloads only after unions, optional params, and generics fail. Every overload multiplies maintenance cost.

### Typing `this`

```ts
interface Counter { count: number }

// 'this' is a phantom parameter — erased in the emitted JS:
function increment(this: Counter, by: number) {
  this.count += by;
}

const counter: Counter = { count: 0 };
increment.call(counter, 5);   // OK
increment(5);
// Error: The 'this' context of type 'void' is not assignable
// to method's 'this' of type 'Counter'.
```

## Common Pitfalls

**Pitfall 1: Leaving parameters untyped.**
Under strict mode this errors (`noImplicitAny`); under loose configs it silently produces `any` and you've written JavaScript with extra steps.

```ts
// ❌ function total(items) { ... }        // items: any
// ✅ function total(items: number[]) { ... }
```

**Pitfall 2: Confusing "optional parameter" with "parameter that can be null."**

```ts
function f(a?: string) {}   // callers may OMIT a; inside it's string | undefined
f();            // OK
f(undefined);   // OK
f(null);        // Error — null was never in the type. If null is a real input,
                // declare it: function f(a: string | null)
```

**Pitfall 3: Forgetting `Promise<>` on async return annotations.**

```ts
// ❌ async function getName(): string { return "x"; }
// Error: The return type of an async function must be the global Promise<T> type.
// ✅
async function getName(): Promise<string> { return "x"; }
```

Relatedly: forgetting `await` gives you a `Promise<T>` where you expected `T` — TypeScript's error messages ("Property 'x' does not exist on type 'Promise<...>'") are your missing-await detector. Read them.

**Pitfall 4: Overloading when a union would do.**

```ts
// ❌ Ceremony for nothing:
function len(x: string): number;
function len(x: unknown[]): number;
function len(x: string | unknown[]) { return x.length; }

// ✅ One signature says the same thing:
function len2(x: string | unknown[]): number { return x.length; }
```

**Pitfall 5: Assuming the implementation signature is callable.**
The implementation signature of an overloaded function is invisible to callers. If you write two overloads and an implementation accepting more combinations, callers can only use the two declared forms. Also, the checker verifies the implementation only *loosely* against overloads — keep overload lists small and tested.

**Pitfall 6: Writing `Function` as a type.**

```ts
// ❌ 'Function' accepts anything callable and returns any — nearly as bad as any:
function run(cb: Function) { return cb(); }

// ✅ Describe the actual signature:
function run2(cb: () => void) { cb(); }
```

## Practice Exercises

1. **Signature bootcamp.** Write and fully type: (a) `clamp(value, min, max)` returning a number; (b) `initials(fullName)` returning the uppercase first letters of each word; (c) `retry(fn, attempts)` where `fn` is `() => boolean` and `retry` returns `boolean`. Add one deliberately wrong call to each and record the compiler error.

2. **Optional vs default vs union.** Design `formatCurrency(amount, currency?, locale?)`. Decide for each parameter whether it should be optional, defaulted, or required — and justify. Inside, handle every `undefined` the compiler surfaces without using `!`.

3. **Callback workout.** Write `mapAndFilter(items: string[], transform: (s: string) => string, keep: (s: string) => boolean): string[]`. Call it with arrow functions that use *no* parameter annotations, and confirm contextual typing infers them. Then intentionally return a number from `transform` and note the error.

4. **Overload design.** Implement `parseInput` with overloads so that: `parseInput(raw: string)` returns `string[]` (splits on commas), and `parseInput(raw: string, asNumbers: true)` returns `number[]`. Verify that `parseInput("1,2", true)` is typed `number[]` and `parseInput("a,b")` is typed `string[]`. Then write one sentence on whether a generic or union could replace these overloads.

5. **Async contract.** Write `async function loadConfig(path: string): Promise<{ port: number; host: string }>` that reads/pretends-to-read a config (you can fake with a resolved value). Call it once *with* `await` and once *without*, access `.port` on both results, and explain the compiler's differing responses.
