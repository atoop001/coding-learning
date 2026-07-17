# Chapter 11: Modules, Declaration Files & Typing Third-Party Libraries

## Overview

Real projects are many files importing each other and dozens of npm packages. This chapter covers how TypeScript handles both: module syntax with types (including type-only imports), how the compiler *resolves* imports, and the machinery that types the outside world — **declaration files (`.d.ts`)**, the **DefinitelyTyped/`@types` ecosystem**, **declaration merging/augmentation** for extending library types, and **ambient declarations** for globals.

This is the chapter that turns "TypeScript on my toy files" into "TypeScript in a real codebase." It's also where beginners hit the most confusing error messages (`Cannot find module`, `Could not find a declaration file`, `esModuleInterop`…), so we decode those explicitly.

## Definitions & Explanations

**ES modules in TS** — `import`/`export` work exactly as in modern JS, and *types flow through them*: import a function, get its full signature; import an interface, use it in annotations. Any file with a top-level `import` or `export` is a module (its names are file-scoped); a file with neither is a *script* whose declarations are global — a classic accidental-global gotcha.

**Type-only imports** — `import type { User } from "./user"` (or inline: `import { type User, createUser }`). Guarantees the import is erased at compile time. Matters for: avoiding circular-dependency runtime issues, letting single-file transpilers (esbuild/swc, `isolatedModules`/`verbatimModuleSyntax`) safely drop type imports, and communicating intent. Habit worth building: if you only use it in type positions, import it with `type`.

**Module resolution** — How the compiler turns `"./user"` or `"zod"` into a file. Relative specifiers resolve from the importing file; bare specifiers resolve through `node_modules`. The `moduleResolution` setting (`"bundler"` for bundler-built apps, `"nodenext"` for Node ESM — which requires explicit `.js` extensions in specifiers) determines the exact rules. Most confusing errors in this area are settings mismatches, not typos.

**Declaration files (`.d.ts`)** — Files containing *only types*: signatures, interfaces, no implementations. They describe JS to the TypeScript compiler. Three sources:
1. **Bundled** — the package ships its own (`"types"` field in its package.json). Most modern libraries.
2. **`@types/*` packages** — community-written declarations from **DefinitelyTyped** for JS libraries that don't ship types: `npm i -D @types/lodash`. The compiler picks them up automatically by name convention.
3. **Yours** — via `declare module`, for untyped packages or your own globals.

**`declare`** — The keyword meaning "this exists at runtime; here's its type; emit nothing." `declare const VERSION: string`, `declare function gtag(...): void`, `declare module "legacy-lib" { … }`. Ambient declarations in `.d.ts` files describe environments (Node's types, DOM types — `lib.dom.d.ts` is why `document` type-checks).

**Declaration merging & module augmentation** — Interfaces with the same name in the same scope *merge*. This is the mechanism for extending third-party types: `declare module "express" { interface Request { user?: AuthUser } }` merges your property into Express's `Request`. Augmenting globals uses `declare global { interface Window { … } }` from within a module.

**Generating your own `.d.ts`** — `"declaration": true` in tsconfig makes `tsc` emit declaration files alongside JS — required when *publishing* a library so consumers get your types.

**`skipLibCheck`** — Skips type-checking *inside* declaration files (yours still get checked at usage). Standard practice: `true` — it avoids breakage from mismatched dependency type versions and speeds builds.

## Code Examples

### Typed modules, end to end

```ts
// ── src/models/user.ts ──────────────────────────────
export interface User {
  id: string;
  name: string;
  email: string;
}

export function displayName(user: User): string {
  return user.name || user.email.split("@")[0] || "anonymous";
}

export const GUEST: User = { id: "guest", name: "Guest", email: "guest@local" };
```

```ts
// ── src/app.ts ──────────────────────────────────────
// Values and types cross the boundary together, checked:
import { displayName, GUEST, type User } from "./models/user";
//                            ^^^^ type-only — guaranteed erased

const u: User = { id: "u1", name: "Ada", email: "ada@example.com" };
console.log(displayName(u));

displayName({ id: "u2" });
// Error: Argument of type '{ id: string; }' is not assignable to parameter
// of type 'User'. Property 'name' is missing…   ← cross-file checking! 🎯
```

```ts
// Pure type-only import — nothing of ./models/user appears in emitted JS:
import type { User } from "./models/user";

// ⚠️ import type gives you ONLY the type side:
import type { GUEST } from "./models/user";
console.log(GUEST);
// Error: 'GUEST' cannot be used as a value because it was imported using 'import type'.
```

### The `@types` workflow — decoding the famous error

```ts
import _ from "lodash";
// Error TS7016: Could not find a declaration file for module 'lodash'.
// 'node_modules/lodash/lodash.js' implicitly has an 'any' type.
//   Try `npm i --save-dev @types/lodash` …
```

The library is installed and *works* — TypeScript just has no type information for it. The fix ladder:

```bash
# 1. Best case — the error already told you:
npm i -D @types/lodash          # now _.chunk([1,2,3], 2) is fully typed

# 2. Check whether types are bundled (then nothing to install — a different
#    error like a version/resolution mismatch is at play).

# 3. No types exist anywhere → write a minimal declaration (next section).
```

### Writing declarations for an untyped library

```ts
// ── src/types/tiny-slugger.d.ts ─────────────────────
// A minimal module declaration: type ONLY what you use.
declare module "tiny-slugger" {
  export interface SlugOptions {
    separator?: string;
    lowercase?: boolean;
  }
  export function slugify(input: string, options?: SlugOptions): string;
  export default slugify;
}
```

```ts
// Now this compiles, typed:
import slugify from "tiny-slugger";
slugify("Hello World", { separator: "_" });   // OK
slugify(42);
// Error: Argument of type 'number' is not assignable to parameter of type 'string'.
```

```ts
// Escape hatch (LAST resort — everything from it is any):
declare module "some-legacy-lib";
```

### Ambient declarations — typing globals

```ts
// ── src/types/globals.d.ts ──────────────────────────
// Script-injected analytics that "just exists" at runtime:
declare function gtag(command: "event", name: string, params?: Record<string, unknown>): void;

// Build-time constants injected by a bundler (Vite/webpack define):
declare const __APP_VERSION__: string;
```

```ts
// Usable anywhere, typed, zero emitted code:
gtag("event", "signup", { plan: "pro" });
console.log(`v${__APP_VERSION__}`);
gtag("evnt", "typo");
// Error: Argument of type '"evnt"' is not assignable to parameter of type '"event"'.
```

### Module augmentation — extending someone else's types

```ts
// ── src/types/express-augment.d.ts ──────────────────
// Express's Request has no 'user' — but our auth middleware adds one.
// Merge it in, instead of casting (req as any).user at 50 call sites:
import type { AuthUser } from "../auth/types";

declare module "express-serve-static-core" {
  interface Request {
    user?: AuthUser;      // MERGED into the library's interface
  }
}
```

```ts
// Now every handler in the codebase sees it, typed and optional:
app.get("/me", (req, res) => {
  if (!req.user) return res.status(401).end();
  res.json({ name: req.user.name });     // narrowed to AuthUser after the guard
});
```

```ts
// Augmenting the global scope from inside a module needs 'declare global':
// ── src/types/window-augment.d.ts ──
export {};                       // make this file a module
declare global {
  interface Window {
    __APP_STATE__: { booted: boolean };
  }
}
// elsewhere: window.__APP_STATE__.booted  ✅ typed
```

### Publishing types: emitting `.d.ts`

```jsonc
// tsconfig.json for a library:
{
  "compilerOptions": {
    "declaration": true,        // emit .d.ts next to each .js
    "declarationMap": true,     // lets consumers jump-to-source
    "outDir": "dist"
  }
}
```

```ts
// dist/index.d.ts (generated) — what your consumers' compilers read:
// export interface User { id: string; name: string; email: string; }
// export declare function displayName(user: User): string;
```

```jsonc
// package.json points at them:
{ "main": "dist/index.js", "types": "dist/index.d.ts" }
```

### `verbatimModuleSyntax` and Node ESM extensions

```ts
// Under "moduleResolution": "nodenext" (Node ESM), relative imports need
// the OUTPUT extension — .js even though the source is .ts:
import { helper } from "./helper.js";    // ✅ resolves helper.ts, emits ./helper.js
import { helper } from "./helper";       // ❌ Error under nodenext
// Under "bundler" resolution, extensionless imports are fine. Know your setting!
```

## Common Pitfalls

**Pitfall 1: Treating "Could not find a declaration file" as fatal (or nuking it with `any`).**
The library works; only types are missing. Follow the ladder: `@types` package → bundled-types check → minimal `declare module` of your own. The lazy `declare module "x";` wildcard should embarrass you slightly every time you read it.

**Pitfall 2: A file with no imports/exports becomes a global script.**
Write `const config = {...}` in a file with no import/export and those names leak into *every* file's scope — colliding confusingly ("Cannot redeclare block-scoped variable"). Fix: ensure every file is a module (any real import/export, or the idiom `export {}`).

**Pitfall 3: Casting instead of augmenting.**

```ts
// ❌ At every call site, forever, unchecked:
const user = (req as any).user;
// ✅ One augmentation file; typed everywhere (see express example above).
```

If you're repeatedly asserting the same "extra" property on a library type, that's augmentation's job.

**Pitfall 4: `@types` version drift.**
`@types/foo` majors track `foo` majors. Upgrading a library without its `@types` (or vice versa) yields bizarre errors deep in `node_modules`. Keep them in lockstep; `skipLibCheck: true` reduces (not eliminates) the blast radius. Also: `@types` belong in `devDependencies` for apps.

**Pitfall 5: Confusing type-only and value imports.**
Importing a class with `import type` then calling `new` on it fails ("cannot be used as a value"). Inverse smell: importing an interface *without* `type` is harmless today but can create circular-import runtime surprises and breaks under `verbatimModuleSyntax`. Rule: type positions only → `import type`.

**Pitfall 6: Editing library `.d.ts` files in `node_modules`.**
It "works" until the next `npm install` silently reverts it. Fixes belong in your own augmentation files (tracked in git), or upstream as a DefinitelyTyped PR — a genuinely good first open-source contribution, and visible résumé material.

**Pitfall 7: Extension/resolution mismatch errors.**
"Relative import paths need explicit file extensions" → you're under `nodenext`; add `.js`. "Cannot find module './x.js'" in a bundler project → you're under `bundler`; drop it. The error is about *configuration*, not your code being wrong.

## Practice Exercises

1. **Multi-module refactor.** Take any single-file exercise from Chapters 5–9 (e.g., the payment methods union) and split it into `types.ts`, `logic.ts`, and `index.ts` with explicit exports. Use `import type` wherever only types cross. Verify a deliberate cross-file type error is caught, then check the emitted JS to confirm type-only imports vanished.

2. **`@types` field trip.** In a scratch project, install a popular JS-only library (e.g., `lodash`). Reproduce the TS7016 error, then fix it with the right `@types` package. Explore: hover three lodash functions and write down their generic signatures; find where the `.d.ts` lives on disk.

3. **Declare the undeclarable.** Pick (or invent) a tiny untyped npm package — or simulate one with a plain `.js` file plus `require`. Write a `declare module` file typing exactly the two functions you use, including an options object with optional properties. Demonstrate one wrong call failing to compile.

4. **Augment Express (or fake it).** Recreate the middleware scenario: a library interface (write a mini `declare module "mini-server" { interface Request { path: string } }` yourself), then augment it from another file with `user?: { id: string }`. Show both properties type-checking on one `Request` value. Bonus: augment `Window` with a typed `__DEBUG__` flag via `declare global`.

5. **Ship a typed mini-library.** Configure a scratch package with `declaration: true`, write a small module (two exported functions + one interface), build it, and inspect the generated `.d.ts`. Then, in a *second* scratch folder, install/link it and confirm the consumer gets autocomplete and errors from your types. Write down the two package.json fields that made it work.
