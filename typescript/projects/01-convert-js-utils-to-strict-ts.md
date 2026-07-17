# Project 1: Convert a JS Utility Library to Strict TypeScript

## Description

Your first TypeScript project mirrors a classic first professional task: take working-but-untyped JavaScript and make it strictly typed without changing its behavior.

You will first write (in plain JavaScript) a small utility library of ~8 functions — string helpers, array helpers, and a couple of object helpers. Then you'll set up a strict TypeScript project and convert the library file-by-file, fixing every error the compiler raises honestly (no `any`, no assertions). Along the way you'll discover that several of your JS functions had latent bugs — undefined returns, unhandled empty inputs — that the compiler forces you to confront.

Using the finished library should feel like using lodash with great types: calling any function from another file gives precise autocomplete, and misuse fails to compile.

## Difficulty & Effort

- **Difficulty:** Beginner (TS) — you know the JS already
- **Estimated effort:** 3–5 hours

## Chapters Used

- `01-why-typescript-and-setup.md` (tsconfig, tsc, running TS)
- `02-basic-types-and-annotations.md` (annotations, inference, null/undefined)
- `03-interfaces-and-type-aliases.md` (naming object shapes)
- `04-functions-in-typescript.md` (parameter/return types, optionals, defaults)

## Requirements Checklist

### Setup
- [ ] Project initialized with npm and a dev dependency on `typescript`
- [ ] `tsconfig.json` with `strict: true`, `rootDir: "src"`, `outDir: "dist"`, `sourceMap: true`
- [ ] An npm script `build` that runs `tsc` and a script `check` that runs `tsc --noEmit`
- [ ] The compiled output runs with plain `node dist/index.js`

### The JavaScript starting point (write this first, as `.js`)
- [ ] `capitalize(text)` — first letter uppercase
- [ ] `truncate(text, maxLength)` — cut with `"…"` suffix when too long
- [ ] `chunk(items, size)` — split an array into arrays of at most `size`
- [ ] `unique(items)` — remove duplicates, order preserved
- [ ] `lastOf(items)` — last element (what does it do on an empty array? The compiler will ask.)
- [ ] `pluckField(objects, fieldName)` — extract one property from each object
- [ ] `groupByField(objects, fieldName)` — group objects into a dictionary by a string field
- [ ] `range(start, end, step)` — array of numbers, `step` optional

### The conversion
- [ ] Every function converted to `.ts` with typed parameters and (for exported functions) explicit return types
- [ ] Functions that can fail to produce a value return `T | undefined` or `T | null` — chosen deliberately and used consistently
- [ ] Optional parameters use `?` or defaults appropriately (justify each choice in a code comment)
- [ ] At least one shared object shape is named with an `interface` or `type` and reused
- [ ] Zero `any` (implicit or explicit), zero `as` assertions, zero `!` non-null assertions
- [ ] `strict` mode passes: `npm run check` exits clean

### Proof it works
- [ ] A `src/demo.ts` that imports and exercises every function with logged output
- [ ] A comment block in `demo.ts` listing at least three calls that do NOT compile (with the error message quoted) — e.g., wrong argument types, missing arguments
- [ ] A `NOTES.md` listing every latent bug or ambiguity the compiler forced you to resolve during conversion (aim for at least three)

## Hints

- Convert leaf functions first (ones that don't call your other functions) — errors stay local.
- When `lastOf([])` makes the compiler complain, that's not the compiler being pedantic — decide what the function should truly return and encode it in the type.
- For `pluckField` and `groupByField`, you don't know generics yet — it's fine to type them for a concrete named shape (e.g., objects with a `string` field you specify). Note in `NOTES.md` how they feel restrictive; Project 3 fixes this properly.
- If you're tempted to write `any` to move on, write the most specific type you can instead — even a union of two cases beats `any`.
- `groupByField`'s return type is a dictionary — Chapter 3's index signatures cover this.

## Stretch Goals

- [ ] Add `noUncheckedIndexedAccess: true` and fix the new errors it surfaces (especially in `chunk`, `lastOf`, `groupByField`)
- [ ] Write a `slugify(text)` function whose allowed separator is the literal union `"-" | "_"`
- [ ] Enable `declaration: true` and inspect the generated `.d.ts` — confirm it reads like documentation
- [ ] Time yourself re-doing the conversion of two functions from scratch; note how much faster the second pass is
