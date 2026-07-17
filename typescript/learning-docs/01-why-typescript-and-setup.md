# Chapter 1: Why TypeScript, and Getting Set Up

## Overview

TypeScript is JavaScript with a static type system layered on top. Every valid JavaScript program is (almost) valid TypeScript, but TypeScript lets you *describe the shape of your data* so the compiler can catch entire categories of bugs before your code ever runs.

You already know JavaScript, so you already know the pain TypeScript solves:

- `undefined is not a function` at runtime, in production, on a Friday.
- Passing arguments in the wrong order and finding out three functions deep.
- Refactoring a function's parameters and hunting every call site by hand.
- Reading someone else's code and having no idea what `options` is supposed to contain.

TypeScript matters professionally because it is the de facto standard for serious JavaScript work. Most large frontend codebases (React, Angular, Vue ecosystems), most Node.js backends at scale, and virtually every job posting mentioning JavaScript will also mention TypeScript. Employers treat it as table stakes.

Key mental model: **TypeScript types exist only at compile time.** The compiler checks your code, then *erases* every type annotation and emits plain JavaScript. Types never exist at runtime. This is called *type erasure*, and it explains many TypeScript behaviors that surprise newcomers (for example, you cannot `if (x instanceof SomeInterface)` — interfaces don't exist at runtime).

## Definitions & Explanations

**Static typing** — Types are checked before the program runs (at "compile time"), as opposed to JavaScript's dynamic typing where type errors surface while the program runs.

**The TypeScript compiler (`tsc`)** — The official tool that (1) type-checks your `.ts` files and (2) transpiles them to `.js`. These are two separate jobs: you can type-check without emitting files (`--noEmit`), and other tools (esbuild, Vite, swc) often do the transpiling while `tsc` only checks types.

**`tsconfig.json`** — The configuration file at the root of a TypeScript project. It tells `tsc` which files to include, what JavaScript version to output, how strict to be, and dozens of other options. The presence of a `tsconfig.json` is what makes a folder "a TypeScript project."

**Type inference** — TypeScript figures out types you didn't write. `let x = 5` infers `x: number` with zero annotations. A huge amount of good TypeScript is *not* writing annotations, because inference already knows.

**Compile error vs. runtime error** — A compile error means `tsc` refuses to bless your code (though it can still emit JS by default!). A runtime error is a normal JavaScript error. TypeScript's whole job is converting would-be runtime errors into compile errors.

**Strict mode** — A family of compiler flags (bundled under `"strict": true`) that make the checker significantly more rigorous. Professional codebases run strict. Learn strict from day one; loosening later is easy, tightening later is painful.

## Setting Up

### Installing

```bash
# Per-project (recommended — versions pinned in package.json)
npm init -y
npm install --save-dev typescript

# Check the version
npx tsc --version
```

### Creating a project

```bash
npx tsc --init
```

This generates a `tsconfig.json` full of commented options. A clean, modern starting point looks like this:

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",          // JS version to emit
    "module": "ESNext",          // module syntax to emit
    "moduleResolution": "bundler", // how imports are resolved
    "strict": true,              // ALL the strict flags — non-negotiable
    "outDir": "./dist",          // where compiled JS goes
    "rootDir": "./src",          // where source TS lives
    "sourceMap": true,           // enables debugging TS in devtools
    "esModuleInterop": true,     // smooths CommonJS/ESM import friction
    "skipLibCheck": true,        // don't type-check node_modules .d.ts files
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"]
}
```

### Running TypeScript

Three common workflows:

```bash
# 1. Compile then run the output
npx tsc                 # emits dist/*.js
node dist/index.js

# 2. Watch mode — recompile on save
npx tsc --watch

# 3. Run TS directly (great for learning and scripts)
npx tsx src/index.ts    # tsx is a fast runner; install with: npm i -D tsx
# Node 23+ can also run .ts files natively with type stripping:
node src/index.ts
```

For quick experiments, the **TypeScript Playground** (typescriptlang.org/play) is invaluable — it shows the emitted JS and errors live. Use it constantly while working through these chapters.

## Code Examples

### Your first TypeScript file

```ts
// src/index.ts

// This is just JavaScript — and it's also valid TypeScript.
function greet(name: string): string {
  return `Hello, ${name}!`;
}

console.log(greet("Ada"));

// The payoff — the compiler catches mistakes JS would run happily:
greet(42);
// Error: Argument of type 'number' is not assignable to parameter of type 'string'.

greet();
// Error: Expected 1 arguments, but got 0.
```

### Type erasure in action

```ts
// TypeScript source:
const age: number = 30;
function double(n: number): number {
  return n * 2;
}

// Compiled JavaScript output (types are simply gone):
// const age = 30;
// function double(n) {
//   return n * 2;
// }
```

Nothing about types survives compilation. There is no runtime cost and no runtime enforcement — a `fetch` response claiming to be a `User` is only a *promise you made to the compiler*, not a guarantee (Chapter 12 covers validating external data).

### Inference doing the work

```ts
let city = "Lisbon";       // inferred: string
let year = 2026;           // inferred: number
let flags = [true, false]; // inferred: boolean[]

city = 99;
// Error: Type 'number' is not assignable to type 'string'.

// Return types are inferred too:
function add(a: number, b: number) {
  return a + b;            // inferred return type: number
}
```

### Why strict matters — a real bug caught

```ts
// With "strict": false, this compiles and crashes at runtime.
// With "strict": true, TypeScript stops you:

function firstUpper(s: string) {
  return s.toUpperCase();
}

const maybeName = ["ada", "grace"].find(n => n.startsWith("z"));
// maybeName is: string | undefined  (find can fail!)

firstUpper(maybeName);
// Error (strict): Argument of type 'string | undefined' is not
// assignable to parameter of type 'string'.

// The fix — handle the undefined case, like you should have anyway:
if (maybeName !== undefined) {
  firstUpper(maybeName); // OK
}
```

### A note on `noEmitOnError`

```bash
# By default tsc emits JS even when there are type errors
# (types are advisory). To make errors block output:
npx tsc --noEmitOnError
```

In real projects, CI runs `tsc --noEmit` as a pure type-check gate while a bundler produces the actual JS.

## Common Pitfalls

**Pitfall 1: Treating TypeScript errors as suggestions.**
Because `tsc` can emit JS despite errors, JS developers sometimes ignore red squiggles "for now." Don't. An error means the compiler found a code path that can misbehave. Fix it or explicitly justify it — never accumulate them.

**Pitfall 2: Starting with `"strict": false` to "learn gradually."**
Non-strict TypeScript silently types many things as `any`, which disables checking. You end up learning a false version of the language and getting little safety. Start strict; the errors are your teacher.

```jsonc
// ❌ "I'll turn it on later" (you won't — there'll be 900 errors by then)
{ "compilerOptions": { "strict": false } }

// ✅
{ "compilerOptions": { "strict": true } }
```

**Pitfall 3: Expecting types to exist at runtime.**

```ts
interface User { name: string }

// ❌ Interfaces are erased — this is not even valid syntax:
// if (data instanceof User) { ... }

// ✅ Check the actual runtime shape:
if (typeof data === "object" && data !== null && "name" in data) { /* ... */ }
```

**Pitfall 4: Confusing `tsc` errors with runtime behavior.**
`tsc` never runs your code. A program can type-check perfectly and still have logic bugs, and (via `any` or bad assertions) a program can type-check and still throw type errors at runtime. TypeScript reduces risk; it doesn't eliminate testing.

**Pitfall 5: Editing compiled output.**
If `outDir` is `dist/`, never hand-edit files in `dist/` — the next compile overwrites them. Add `dist/` to `.gitignore`.

## Practice Exercises

1. **Setup drill.** Create a new folder, initialize npm, install TypeScript, generate a `tsconfig.json`, and configure it with `strict: true`, `rootDir: "src"`, and `outDir: "dist"`. Write a `src/index.ts` that exports a `greet(name)` function and logs a greeting. Compile it and run the output with Node. Inspect the emitted JS and confirm the types were erased.

2. **Break it on purpose.** In your `greet` project, call `greet(123)` and `greet()` and read the two compiler errors carefully. Then run `tsc --noEmitOnError` and confirm no JS is produced.

3. **Inference safari.** Without writing a single type annotation, declare: a string, a number, an array of numbers, an object with two properties, and a function that concatenates two strings. Hover each in your editor (or paste into the TypeScript Playground) and write down the inferred type of each.

4. **Watch mode workflow.** Run `tsc --watch` in one terminal. In another, repeatedly introduce and fix a type error, observing how fast feedback arrives. Then set up `tsx` (or `node --watch` on a recent Node) so saving the file re-runs the program.

5. **Strict vs. loose.** Take the `firstUpper`/`find` example from this chapter. Paste it into the Playground twice: once with `strict` on, once off (use the TS Config panel). Note exactly which error appears or disappears, and explain in one sentence *why* the strict version is safer.
