# Chapter 12: Tooling & Real-World Patterns

## Overview

You now know the language. This final chapter is about *practicing* it professionally: the strict-mode flags and what each buys you, linting TypeScript with ESLint, working with the DOM and `fetch` under strict settings, validating external data at runtime (the boundary problem), and the pragmatic playbook for migrating an existing JavaScript project to TypeScript — the exact task many juniors get handed first.

The through-line: **TypeScript guarantees stop at the edges of your program.** Inside, the compiler proves things. At the edges — network responses, user input, DOM queries, JSON files, third-party callbacks — you must *establish* the types with checks. Professionals are recognizable by how they handle edges.

## Definitions & Explanations

### The strict family (and friends)

`"strict": true` enables a bundle; the notable members:

- **`noImplicitAny`** — Untyped parameters/variables error instead of silently becoming `any`. The floor of type safety.
- **`strictNullChecks`** — `null`/`undefined` are real types you must handle. The single biggest bug-eliminator.
- **`strictFunctionTypes`** — Sound checking of function-type assignability (parameter contravariance).
- **`strictPropertyInitialization`** — Class properties must be definitely assigned (Chapter 8).
- **`useUnknownInCatchVariables`** — `catch (e)` gives `unknown`, not `any` (Chapter 6).

Worth enabling beyond `strict`:

- **`noUncheckedIndexedAccess`** — Index reads are `T | undefined` (Chapter 10). Honest arrays/dictionaries.
- **`noImplicitOverride`** — Subclass overrides must say `override`, catching renamed-base-method drift.
- **`exactOptionalPropertyTypes`** — Distinguishes "absent" from "explicitly undefined". Stricter modeling; some ecosystem friction.
- **`noUnusedLocals` / `noUnusedParameters`** — Hygiene; often delegated to ESLint instead.

### Linting

**typescript-eslint** connects ESLint to TS. Beyond style, its *type-aware* rules catch real bugs the compiler allows: floating (unawaited) promises, `if` conditions that are always truthy, unsafe `any` flows, misused enums. Baseline professional setup: ESLint + `typescript-eslint`'s `recommendedTypeChecked` config + Prettier (formatting) kept separate from linting (correctness).

### The boundary problem & runtime validation

`res.json()` returns `Promise<any>`. Annotating it `as User` is a *claim, not a check* — the server can ship anything. Three professional postures:
1. **Trust + type** (`as User`) — acceptable for internal APIs you control end-to-end; crashes surface far from the cause when the contract breaks.
2. **Manual guards** — `unknown` + type guard functions (Chapter 6). Fine for small surfaces.
3. **Schema validation** — a library like **Zod**: define a schema once, get *both* the runtime validator and the inferred static type (`z.infer`). The industry-standard answer; know its shape even if you haven't memorized its API.

The same discipline applies to configuration, not just network responses: `process.env.ANYTHING` is typed `string | undefined` — Node has no idea which variables your deployment actually set — so treat startup config as a boundary too. Validate it once, at process start, into a fully-typed config object; fail fast with a clear error if something required is missing, rather than letting `undefined` leak into business logic and surface as a confusing crash minutes later.

### DOM typing essentials

`lib.dom.d.ts` types the entire browser API. The recurring patterns: `querySelector` returns `Element | null` (handle the null; use the generic parameter or `instanceof` for specific elements); event handlers receive typed events but `e.target` is `EventTarget | null` (narrow it); `addEventListener("click", …)` infers `MouseEvent` from the event-name literal — a real-world payoff of literal types.

### Migration strategy

TypeScript is designed for *incremental* adoption: `allowJs` lets `.ts` and `.js` coexist; `checkJs` (or `// @ts-check` per-file) type-checks JS via inference + JSDoc; you convert file-by-file, loosest-config-first, tightening flags as the converted fraction grows. The anti-pattern is the big-bang rewrite; the pro move is strangling the JS gradually while the app keeps shipping.

### Testing TypeScript with Vitest

Vitest is the standard test runner for TS/Vite projects — native ESM, TS support with zero config (no `ts-jest` transform step), and a Jest-compatible API (`describe`, `it`, `expect`, `vi` for mocking). Install it as a dev dependency; a `vitest.config.ts` (or the `test` block inside `vite.config.ts`) points it at your test files, conventionally `*.test.ts` beside the source they cover.

The reason typed testing matters beyond "tests exist": in a JS test suite, a fixture object silently drifting out of sync with the interface it represents is only caught when the test happens to exercise the drifted field. In a TS test suite, the *fixture itself* is compiler-checked against the interface — rename a field, add a required property, or narrow a union, and every stale fixture becomes a compile error before the test runner starts. Tests stop being just a runtime check; they're a second consumer of your types, and that consumer forces your interfaces to keep matching reality.

Typed mocks work the same way: `vi.fn<SomeInterface["method"]>()` (pulling the signature straight off the interface or a `typeof` of the real function) means a mock's call signature is checked against the function it replaces — call it with the wrong argument types, or resolve the wrong shape, and TS objects at compile time, not just an assertion failing at runtime.

## Code Examples

### A production-grade tsconfig, annotated

```jsonc
{
  "compilerOptions": {
    // Language & output
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],   // browser project; drop DOM for Node

    // Safety — the point of the exercise
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,

    // Interop & hygiene
    "esModuleInterop": true,
    "isolatedModules": true,        // every file independently transpilable
    "verbatimModuleSyntax": true,   // forces correct import type usage
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,

    "outDir": "dist",
    "sourceMap": true
  },
  "include": ["src"]
}
```

### DOM: strict TypeScript in the browser

```ts
// querySelector: typed by CSS selector? No — by the GENERIC you provide (or Element):
const form = document.querySelector("#signup");
// form: Element | null — two problems to handle: null, and too-general Element.

form.addEventListener("submit", () => {});
// Error: 'form' is possibly 'null'.

// ✅ Handle null explicitly (fail fast beats mystery crashes later):
const form2 = document.querySelector<HTMLFormElement>("#signup");
if (!form2) throw new Error("#signup form missing from page");
// form2: HTMLFormElement — .elements, .submit() etc. now available.

// Inputs — the value lives on HTMLInputElement, not Element:
const email = document.querySelector<HTMLInputElement>("#email");
console.log(email?.value.trim());

// Event maps: the "submit" literal selects SubmitEvent automatically:
form2.addEventListener("submit", (e) => {
  e.preventDefault();          // e: SubmitEvent — typed, no annotation needed
});

// e.target needs narrowing — it's EventTarget | null:
document.addEventListener("click", (e) => {
  if (e.target instanceof HTMLButtonElement) {
    console.log(`clicked button: ${e.target.textContent}`);   // narrowed ✅
  }
});

// createElement: the tag literal picks the element type:
const img = document.createElement("img");   // HTMLImageElement
img.src = "/logo.png";                       // .src exists — typed!
```

### Fetch: the honest typed wrapper

```ts
interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

// Level 1 — trust + type. Compiles; verifies NOTHING at runtime:
async function getTodoTrusting(id: number): Promise<Todo> {
  const res = await fetch(`/api/todos/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json() as Promise<Todo>;   // ← a promise to the compiler, not a check
}

// Level 2 — unknown + guard. The boundary is defended:
function isTodo(v: unknown): v is Todo {
  return (
    typeof v === "object" && v !== null &&
    typeof (v as Record<string, unknown>).id === "number" &&
    typeof (v as Record<string, unknown>).title === "string" &&
    typeof (v as Record<string, unknown>).completed === "boolean"
  );
}

async function getTodoChecked(id: number): Promise<Todo> {
  const res = await fetch(`/api/todos/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const data: unknown = await res.json();      // honest type
  if (!isTodo(data)) throw new Error("API contract violation: not a Todo");
  return data;                                  // narrowed — REALLY a Todo now
}

// Level 3 — schema validation (Zod): validator and type from ONE definition:
import { z } from "zod";

const TodoSchema = z.object({
  id: z.number(),
  title: z.string(),
  completed: z.boolean(),
});
type TodoZ = z.infer<typeof TodoSchema>;        // static type derived from schema

async function getTodoValidated(id: number): Promise<TodoZ> {
  const res = await fetch(`/api/todos/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return TodoSchema.parse(await res.json());    // throws with a precise report if wrong
}
```

### Environment variables: validating config at startup

```ts
// process.env.ANYTHING is `string | undefined` — Node can't know what
// your deployment actually set. Treat it like any other external boundary.

interface AppConfig {
  readonly apiBaseUrl: string;
  readonly port: number;
  readonly logLevel: "debug" | "info" | "warn" | "error";
}

function requireEnv(name: string): string {
  const value = process.env[name];
  if (value === undefined || value === "") {
    throw new Error(`Missing required environment variable: ${name}`);
  }
  return value;
}

function loadConfig(): AppConfig {
  const logLevel = process.env.LOG_LEVEL ?? "info";
  if (!["debug", "info", "warn", "error"].includes(logLevel)) {
    throw new Error(`Invalid LOG_LEVEL: ${logLevel}`);
  }

  return {
    apiBaseUrl: requireEnv("API_BASE_URL"),       // e.g. "https://api.example.test"
    port: Number(process.env.PORT ?? "3000"),
    logLevel: logLevel as AppConfig["logLevel"],  // narrowed by the check above ↑
  };
}

// Call ONCE at startup — fail fast beats a mystery crash three requests in:
export const config = loadConfig();
```

```ts
// The Zod version: one schema gives both validation and the static type —
// same trade-off as the fetch boundary above.
import { z } from "zod";

const ConfigSchema = z.object({
  API_BASE_URL: z.string().url(),
  PORT: z.coerce.number().int().positive().default(3000),
  LOG_LEVEL: z.enum(["debug", "info", "warn", "error"]).default("info"),
});

// parse() throws a detailed, field-by-field error if anything required
// is missing or malformed — exactly what you want at boot, not mid-request:
export const env = ConfigSchema.parse(process.env);
// env: { API_BASE_URL: string; PORT: number; LOG_LEVEL: "debug"|"info"|"warn"|"error" }
```

### Type-aware linting catching what tsc can't

```ts
// tsc is FINE with all of these. typescript-eslint is not:

async function save(): Promise<void> { /* ... */ }

function onClick() {
  save();
  // @typescript-eslint/no-floating-promises:
  // Promise is created but not awaited/handled — errors vanish silently. 🐛
}

function check(items: string[]) {
  if (items) { /* ... */ }
  // @typescript-eslint/no-unnecessary-condition:
  // 'items' is always truthy (string[] is never null here) —
  // you probably meant items.length. 🐛
}
```

```jsonc
// eslint.config.js sketch (flat config):
// import tseslint from 'typescript-eslint';
// export default tseslint.config(
//   ...tseslint.configs.recommendedTypeChecked,
//   { languageOptions: { parserOptions: { projectService: true } } },
// );
```

### Migrating a JS project — the playbook

```jsonc
// Step 1: TS "wraps" the JS project — nothing converted yet:
{
  "compilerOptions": {
    "allowJs": true,          // .js files are part of the program
    "checkJs": false,         // …but not checked yet
    "strict": false,          // start permissive — tighten later
    "noEmit": true            // bundler keeps building; tsc only checks
  },
  "include": ["src"]
}
```

```ts
// Step 2: convert file-by-file, LEAVES first (no internal dependencies):
//   git mv src/utils/format.js src/utils/format.ts
// Fix the errors that appear — each one is a latent bug or a missing type.

// Step 3: JSDoc bridges files you can't convert yet — checked by TS, still .js:
/**
 * @param {number} cents
 * @param {string} [currency]
 * @returns {string}
 */
function formatPrice(cents, currency = "EUR") {
  return `${(cents / 100).toFixed(2)} ${currency}`;
}

// Step 4: ratchet the config as coverage grows — never loosen:
//   checkJs: true  →  strict: true  →  noUncheckedIndexedAccess: true
// Step 5: when the last .js is gone, delete allowJs. Migration complete.
```

```ts
// Temporary escape hatch, budgeted and visible — with expiry culture:
// @ts-expect-error TODO(#412): legacy payload shape, remove after API v2
const total = legacyComputeTotal(order);
// Prefer @ts-expect-error over @ts-ignore: it ERRORS when the underlying
// problem is fixed, so suppressions can't outlive their reason. 🎯
```

### Vitest: a typed test, fixtures, and mocks

```ts
// npm install -D vitest
// vitest.config.ts (or the `test` block in vite.config.ts):
// import { defineConfig } from "vitest/config";
// export default defineConfig({ test: { environment: "node" } });

// src/pricing.ts
export interface Order {
  id: string;
  items: { unitPriceCents: number; quantity: number }[];
  discountPercent: number;
}

export function totalCents(order: Order): number {
  const subtotal = order.items.reduce(
    (sum, item) => sum + item.unitPriceCents * item.quantity,
    0
  );
  return Math.round(subtotal * (1 - order.discountPercent / 100));
}
```

```ts
// src/pricing.test.ts
import { describe, it, expect } from "vitest";
import { totalCents, type Order } from "./pricing";

// A typed fixture: `satisfies` checks the object against Order's shape
// while keeping literal inference (handy if you narrow it further later).
const baseOrder = {
  id: "ord_1",
  items: [{ unitPriceCents: 1000, quantity: 2 }],
  discountPercent: 0,
} satisfies Order;

describe("totalCents", () => {
  it("sums unit price * quantity with no discount", () => {
    expect(totalCents(baseOrder)).toBe(2000);
  });

  it("applies a percent discount", () => {
    const discounted: Order = { ...baseOrder, discountPercent: 10 };
    expect(totalCents(discounted)).toBe(1800);
  });
});
```

```ts
// Typed mocks: vi.fn<...>() types the mock's call signature.
import { vi, expect, it } from "vitest";

interface UserRepo {
  findById(id: string): Promise<{ id: string; name: string } | null>;
}

it("greets a found user", async () => {
  const findById = vi.fn<UserRepo["findById"]>();
  //                 ^ signature pulled straight off the interface —
  //                   the mock can't drift from the contract it stands in for.
  findById.mockResolvedValue({ id: "u1", name: "Ada" });

  const repo: UserRepo = { findById };
  const user = await repo.findById("u1");

  expect(user?.name).toBe("Ada");
  expect(findById).toHaveBeenCalledWith("u1");
});
```

```ts
// How typing catches drift: rename a field on Order (unitPriceCents -> unitPrice)
// and baseOrder above fails to compile — BEFORE any test runs:
//
//   Object literal may only specify known properties, and
//   'unitPriceCents' does not exist in type 'Order'.
//
// A JS test suite would happily run the old fixture and either pass
// wrongly (the field silently undefined, math coincidentally still working)
// or fail at runtime with a far less specific message.
```

## Common Pitfalls

**Pitfall 1: `as User` on fetch results, believed to be a check.**
The single most dangerous habit in applied TypeScript: assertions at boundaries create *confidently wrong* types that crash three modules away from the cause. At every boundary, choose a posture consciously (trust / guard / schema) — and default to validation for anything external.

**Pitfall 2: `document.querySelector(...)!` everywhere.**
Non-null asserting every DOM query resurrects `Cannot read properties of null` with worse stack traces. Write one helper — `function mustFind<T extends Element>(sel: string): T` that throws a *descriptive* error — and use it: same convenience, actual diagnostics.

**Pitfall 3: Floating promises.**
`save()` without `await` in a `void` context swallows rejections. tsc accepts it; production incidents love it. Enable `no-floating-promises` (needs type-aware linting) and treat every unawaited promise as a decision, marked `void save()` when intentional.

**Pitfall 4: Migration via `any`-storm.**
Renaming `.js` → `.ts` and slapping `any`/`@ts-ignore` on every error produces *negative* value: the codebase looks typed, checks nothing, and blocks later tightening. Convert fewer files properly rather than many files fraudulently. Track `any`-count as a burn-down metric.

**Pitfall 5: `@ts-ignore` outliving its reason.**
`@ts-ignore` suppresses forever, silently — even after the underlying issue is gone (or a *new*, different error appears on that line!). Use `@ts-expect-error` (+ comment + ticket), which fails the build once the error disappears.

**Pitfall 6: Strictness theater — flags on, escape hatches everywhere.**
A repo with `strict: true` plus hundreds of `as any`, `!`, and suppressions is *less* safe than an honest loose config, because readers trust guarantees that don't exist. Strictness is a budgetary discipline: every escape hatch is debt, made visible, tracked, and paid down.

**Pitfall 7: Skipping the "does it actually run?" check.**
tsc passing means types are consistent, not that code is correct or even that the bundler/runtime config works (ESM/CJS mismatches, wrong `lib`, missing DOM types show up at run time). CI needs *both* `tsc --noEmit` and real execution (tests, a smoke run).

**Pitfall 8: `as SomeType` on test fixtures.**
Casting a fixture (`{ id: "1" } as User`) to dodge a compile error defeats the entire point of typed tests — the fixture no longer proves it matches the interface, and it silently rots the same way a JS fixture would. Build fixtures with `satisfies` (or a literal that genuinely matches the type) so the compiler keeps checking them; if the object won't compile, that's the test telling you the fixture — or the interface — is out of date.

## Practice Exercises

1. **Flag safari.** Create a scratch project and write short snippets that compile with `strict: false` but error under: `noImplicitAny`, `strictNullChecks`, `strictPropertyInitialization`, and `noUncheckedIndexedAccess` (one snippet each). For each, state the runtime bug the flag pre-empted.

2. **DOM contact form.** Build (HTML + strict TS) a small form with name/email inputs and a submit handler that: prevents default, reads and trims values, validates non-emptiness, and writes errors or a success message into a `<p role="status">`. Zero `!`, zero `as`, zero `any` — every null handled explicitly.

3. **Boundary defense in depth.** Against any public JSON API (or a local JSON file served statically), implement the same "get one record" function three times: trusting (`as`), guarded (`unknown` + predicate), and schema-validated (Zod or a hand-rolled mini-validator). Then corrupt the data (rename one field) and document exactly how each version fails. One paragraph: which failure mode do you want in production, and why?

4. **Lint the invisible.** Set up typescript-eslint with `recommendedTypeChecked` in a scratch project. Write code triggering `no-floating-promises` and `no-unnecessary-condition`, confirm tsc alone accepts both, and fix each properly (not via disable-comments). Record the two rule names and what each caught.

5. **Micro-migration.** Write (or grab) a 3-file vanilla-JS mini-project (e.g., a shopping list with storage helpers). Migrate it: `allowJs` wrapper config first, then convert leaf → middle → entry file, then ratchet to full `strict`. Keep a migration log: every error the compiler raised and whether it was a real latent bug or just missing type info. Finish with zero `any`, zero suppressions — and a working app.

6. **Typed test suite.** Pick two pure functions from an earlier project (e.g., Project 2 or 3) and write Vitest tests for them: at least one `satisfies`-checked fixture, one edge case, and one typed mock (`vi.fn<...>()`) replacing a dependency the function takes as a parameter. Then deliberately rename a field on the underlying interface and note the exact compile error your fixture produces before you fix it.

7. **Break it on purpose.** Take `getTodoChecked` above (or your own boundary function) and write Vitest tests that mock `fetch` with `vi.fn<typeof fetch>()`: one case with a well-formed response, and one with a malformed response that should throw. Confirm the malformed case is actually exercising your type guard — not passing by luck.
