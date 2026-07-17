# TypeScript Learning Track

A self-paced track taking you from "solid JavaScript developer" to "professionally employable TypeScript developer." The focus is real-world web development: strict-mode typing, honest handling of nulls and external data, and the patterns used in production codebases and probed in interviews.

## Prerequisite

**Solid JavaScript is assumed.** This track does not re-teach functions, closures, classes, promises/async-await, modules, array methods, or the DOM — complete the JavaScript track first. Every chapter builds directly on that knowledge and teaches only what TypeScript adds.

## Structure

- `learning-docs/` — 12 chapters. Primary study material: read, type out the examples, do the exercises.
- `projects/` — 6 guided project specs, easiest to hardest. No solutions provided — the spec + compiler + your chapters are the resources, just like a real ticket.

## Chapters (in order)

| # | File | Covers |
|---|------|--------|
| 1 | `learning-docs/01-why-typescript-and-setup.md` | Why TS, tsc, tsconfig, running TS, type erasure, strict from day one |
| 2 | `learning-docs/02-basic-types-and-annotations.md` | Primitives, arrays, objects, any/unknown/void/never, inference, when to annotate |
| 3 | `learning-docs/03-interfaces-and-type-aliases.md` | Naming shapes, interface vs type, structural typing, excess property checks, index signatures |
| 4 | `learning-docs/04-functions-in-typescript.md` | Parameter/return types, optional/default/rest, callbacks, async, overloads, `this` |
| 5 | `learning-docs/05-unions-intersections-literal-types.md` | Unions, intersections, literal types, `as const`, discriminated unions |
| 6 | `learning-docs/06-narrowing-and-type-guards.md` | Control-flow narrowing, all guard kinds, custom predicates, assertions, exhaustiveness with `never` |
| 7 | `learning-docs/07-generics.md` | Generic functions/interfaces/classes, constraints, `keyof`, when NOT to genericize |
| 8 | `learning-docs/08-classes-in-typescript.md` | Access modifiers, readonly, parameter properties, `implements`, abstract classes |
| 9 | `learning-docs/09-utility-types.md` | Partial, Pick, Omit, Record, Required, Readonly, ReturnType, Awaited, Exclude/Extract, `keyof typeof` |
| 10 | `learning-docs/10-enums-tuples-advanced-objects.md` | Enums (and modern alternatives), tuples, indexed access types, `satisfies`, `noUncheckedIndexedAccess` |
| 11 | `learning-docs/11-modules-declaration-files-third-party.md` | Typed modules, `import type`, `.d.ts` files, `@types`, `declare module`, augmentation |
| 12 | `learning-docs/12-tooling-and-real-world-patterns.md` | Strict flags in depth, ESLint, DOM + fetch under strict mode, runtime validation, migrating JS to TS |

## Projects (in order)

| # | File | Builds | Difficulty |
|---|------|--------|-----------|
| 1 | `projects/01-convert-js-utils-to-strict-ts.md` | Convert a JS utility library to strict TS | Beginner |
| 2 | `projects/02-typed-task-manager.md` | Task manager with discriminated-union state lifecycle | Beginner-plus |
| 3 | `projects/03-generic-collection-and-cache-library.md` | Generic collection + TTL cache library | Intermediate |
| 4 | `projects/04-typed-api-client.md` | Typed fetch API client with validated boundaries | Intermediate-plus |
| 5 | `projects/05-dom-quiz-app-strict-ts.md` | Browser quiz app, strict TS in the DOM, no framework | Intermediate-plus |
| 6 | `projects/06-capstone-typed-spa.md` | Capstone SPA: routing, persistence, third-party APIs & types | Advanced |

## Suggested Cadence

Read chapters in order, doing the exercises as you go, and interleave projects at these checkpoints:

1. **Chapters 1–4** → **Project 1** (convert JS utils). Cements setup, annotations, shapes, and function typing.
2. **Chapters 5–6** → **Project 2** (task manager). Discriminated unions + narrowing are the heart of applied TS — practice them immediately.
3. **Chapters 7–9** → **Project 3** (generic library). Generics, classes, and utility types in one build.
4. Still after 7–9 (plus skimming Ch. 12's fetch section) → **Project 4** (API client). Utility types and boundary validation against a real API.
5. **Chapters 10–12** → **Project 5** (DOM quiz app). Strict TS meets the browser.
6. **Everything** → **Project 6** (capstone SPA). Budget multiple sessions; the README and type-design write-up are part of the deliverable — it's your portfolio piece.

Rough overall pacing: 4–8 weeks at ~1 hour/day. Don't rush chapters 5–7; they carry the most weight per page.

## Ground Rules for the Whole Track

- `"strict": true` always. Projects 3+ also enable `noUncheckedIndexedAccess`.
- `any`, `as`, and `!` are treated as debt: avoid them, and comment any survivor with a justification.
- Type out every code example yourself — reading TypeScript is not learning TypeScript; arguing with the compiler is.
- When the compiler errors, read the message fully before changing code. The error text is the curriculum.
