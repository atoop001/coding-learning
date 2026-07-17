# Chapter 3: Interfaces & Type Aliases

## Overview

Inline object types (`{ name: string; age: number }`) get unwieldy fast. TypeScript gives you two ways to *name* types so they can be reused, documented, and composed: **interfaces** and **type aliases**. They overlap heavily — one of the most common beginner questions is "which do I use?" — and this chapter answers that directly, along with the mechanics: extending, combining, optional/readonly members, index signatures, and how TypeScript decides whether two types are compatible at all (structural typing).

Naming your types is not bureaucracy. A well-named `interface Invoice` is executable documentation: every reader and every call site learns exactly what an invoice contains, and the compiler enforces it everywhere.

## Definitions & Explanations

**Interface** — A named description of an object's shape:

```ts
interface User {
  id: string;
  name: string;
}
```

Interfaces can be extended (`interface Admin extends User`), implemented by classes (Chapter 8), and *merged* (declaring the same interface twice combines them — mostly relevant for augmenting library types, Chapter 11).

**Type alias** — A name for *any* type, not just object shapes:

```ts
type User = { id: string; name: string }; // object shape — same as interface
type ID = string | number;                // union — interfaces can't do this
type Point = [number, number];            // tuple
type Handler = (event: string) => void;   // function type
```

**Structural typing ("duck typing")** — TypeScript compares types by *shape*, not by name. If a value has all the properties `User` requires, it *is* a `User` — no matter how it was created or what it's called. This is fundamentally different from Java/C# nominal typing and is the key to understanding almost all assignability questions.

**Excess property checking** — A special strictness applied to *object literals* passed directly where a specific type is expected: unknown extra properties are flagged. This exists to catch typos (`colour` vs `color`). It only applies to fresh literals — variables holding extra properties pass fine (structural typing).

**Optional (`?`) and readonly members** — Same as Chapter 2, but now in named types. `phone?: string` means the property may be absent; `readonly id: string` forbids assignment after creation (compile-time only).

**Index signatures** — Describe objects used as dictionaries with unknown keys:

```ts
interface WordCounts {
  [word: string]: number;
}
```

**Extending vs intersecting** — Interfaces compose with `extends`; type aliases compose with `&` (intersection, Chapter 5). Results are usually equivalent; `extends` gives clearer error messages and catches incompatible overrides earlier.

### So… interface or type? The practical rules

1. **Object shapes that represent domain entities** (User, Order, Config, component props): either works; pick one and be consistent. `interface` is the conventional default in many codebases and produces friendlier errors.
2. **Unions, tuples, primitives, function types, mapped/conditional types**: must use `type` — interfaces literally cannot express these.
3. **Library/public API types that consumers may need to augment**: `interface` (declaration merging).
4. **Never mix styles for the same concept** in one codebase — consistency beats the marginal differences.

A defensible team rule: *use `interface` for object shapes, `type` for everything else.* Another equally defensible rule: *use `type` everywhere and `interface` only when merging is needed.* Know both; follow your team.

## Code Examples

### From inline chaos to named clarity

```ts
// ❌ Before — repeated, unnamed, drift-prone:
function sendReceipt(user: { id: string; email: string; name: string }) { /* ... */ }
function banUser(user: { id: string; email: string; name: string }) { /* ... */ }

// ✅ After — one source of truth:
interface User {
  id: string;
  email: string;
  name: string;
}

function sendReceipt2(user: User) { /* ... */ }
function banUser2(user: User) { /* ... */ }
```

### Structural typing — shape is everything

```ts
interface Named {
  name: string;
}

function greet(entity: Named) {
  console.log(`Hello, ${entity.name}`);
}

// None of these were "declared as Named" — they just fit the shape:
greet({ name: "Ada" });                     // OK — wait, see excess check below!
const dog = { name: "Rex", legs: 4 };
greet(dog);                                 // OK — extra properties fine via variable
class City { constructor(public name: string) {} }
greet(new City("Lisbon"));                  // OK — classes are shapes too
```

### Excess property checking — literals are special

```ts
interface Config {
  host: string;
  port?: number;
}

// Fresh literal, directly passed → excess check applies:
const c1: Config = { host: "localhost", protocl: "https" };
// Error: Object literal may only specify known properties,
// and 'protocl' does not exist in type 'Config'.  ← caught a typo!

// Same data via a variable → plain structural typing, no complaint:
const raw = { host: "localhost", protocl: "https" };
const c2: Config = raw; // OK (extra property is just ignored by the type)
```

This asymmetry confuses everyone once. Rationale: a literal you typed by hand with an unknown key is *probably* a mistake; a pre-existing object with extra data is normal.

### Extending interfaces

```ts
interface Entity {
  readonly id: string;
  createdAt: Date;
}

interface User extends Entity {
  email: string;
}

interface AdminUser extends User {
  permissions: string[];
}

const admin: AdminUser = {
  id: "u_1",
  createdAt: new Date(),
  email: "root@example.com",
  permissions: ["*"],
};

admin.id = "u_2";
// Error: Cannot assign to 'id' because it is a read-only property.
```

### Type aliases beyond objects

```ts
// Things interfaces cannot name:
type ID = string | number;
type Status = "draft" | "published" | "archived";
type Pair = [number, number];
type Comparator<T> = (a: T, b: T) => number;   // generics: Chapter 7
type MaybeUser = User | null;

// And object shapes, where they work like interfaces:
type Point = {
  x: number;
  y: number;
};

// Composing aliases with intersections (≈ extends):
type Timestamped = { createdAt: Date; updatedAt: Date };
type Post = Timestamped & {
  title: string;
  status: Status;
};
```

### Function types and callable interfaces

```ts
// Alias form (most common):
type Logger = (message: string, level?: "info" | "warn" | "error") => void;

const consoleLogger: Logger = (msg, level = "info") => {
  console.log(`[${level}] ${msg}`);
};

// Interface form (rarer, allows hybrid callable + properties):
interface Counter {
  (): number;          // callable
  reset(): void;       // ...with methods attached
  count: number;
}
```

### Index signatures for dictionary objects

```ts
interface Inventory {
  [productId: string]: number;
}

const stock: Inventory = { apples: 10, pears: 4 };
stock["plums"] = 7;            // OK — any string key allowed
const n = stock["nonexistent"]; // typed number — but is undefined at runtime! ⚠️
// Enable "noUncheckedIndexedAccess" to make this number | undefined (Chapter 12).

// Mixing known and indexed members — known members must fit the signature:
interface Scores {
  [subject: string]: number;
  average: number;   // OK, number fits
  // note: string;   // Error: 'string' is not assignable to 'number'
}
```

### Interface merging (preview of Chapter 11)

```ts
interface Window { myAppVersion: string }
// Elsewhere, the built-in Window interface already exists —
// this declaration MERGES into it rather than conflicting:
window.myAppVersion = "1.0.0"; // now type-checks

// type aliases would NOT merge:
// type Window = { ... }  // Error: Duplicate identifier 'Window'.
```

## Common Pitfalls

**Pitfall 1: Thinking types are classes.**
JS developers coming via OOP languages expect `interface User` to be instantiable or checkable at runtime. It's neither — it's erased. `new User()` is meaningless; `x instanceof User` is a syntax error. Interfaces describe; classes construct (Chapter 8).

**Pitfall 2: Fighting excess property checks with `any` or assertions.**

```ts
interface Options { title: string }

// ❌ Nukes all checking to sneak one property through:
const opts: Options = { title: "Hi", subtitle: "there" } as any;

// ✅ If subtitle is legitimate, add it to the type (optional if appropriate):
interface Options2 { title: string; subtitle?: string }
```

If the compiler flags an excess property, the fix is almost always in the *type* or a genuine typo — not an assertion.

**Pitfall 3: Using index signatures when you know the keys.**

```ts
// ❌ Throws away everything the compiler could check:
interface Theme { [key: string]: string }

// ✅ Enumerate what you actually have:
interface Theme2 { primary: string; secondary: string; background: string }
// (When keys are a known union, Record<K, V> is even better — Chapter 9.)
```

**Pitfall 4: Duplicating shapes instead of extending.**
Copy-pasting `id`/`createdAt` into ten interfaces means ten places to update. Factor shared members into a base interface (or intersection) once.

**Pitfall 5: Optional properties as a substitute for modeling.**
An interface where everything is optional (`title?`, `body?`, `error?`, `data?`) usually means several distinct states crammed into one type. Model each state separately and union them — Chapter 5's discriminated unions do this properly.

```ts
// ❌ Which combinations are valid? Nobody knows:
interface FetchState { loading?: boolean; data?: string; error?: string }

// ✅ Preview of Chapter 5:
type FetchState2 =
  | { status: "loading" }
  | { status: "success"; data: string }
  | { status: "error"; error: string };
```

**Pitfall 6: Assuming `readonly` protects at runtime.**
`readonly` (and everything else here) is erased. JS code, JSON tampering, or an `as any` can still mutate. For runtime immutability use `Object.freeze` — and remember its return type `Readonly<T>` is checked, its behavior is shallow.

## Practice Exercises

1. **Model a music library.** Define types/interfaces for `Artist` (id, name, optional country), `Album` (id, title, year, artist reference), and `Track` (id, title, durationSeconds, album reference). Use `extends` or intersections to factor the shared `id` into a base. Create two fully valid sample objects and one that the compiler rejects (note the error).

2. **interface vs type triage.** For each of the following, decide whether an interface can express it, and write the correct declaration: (a) a value that is either a `string` or a `string[]`; (b) a callback taking a `number` and returning `void`; (c) a config object with `apiKey` and optional `timeout`; (d) a pair `[latitude, longitude]`; (e) the string values `"asc" | "desc"`. State the rule you applied for each.

3. **Excess property detective.** Construct an example where assigning an object literal to a typed variable errors, but assigning the *same data* through an intermediate variable compiles. Then explain in two sentences why TypeScript treats the cases differently and which one caught a plausible bug.

4. **Dictionary with rules.** Define a `Settings` type using an index signature where every value is `string | boolean`, plus a *required, known* member `version: string`. Verify the compiler makes you keep the known member compatible with the index signature, and write three assignments (two valid, one invalid).

5. **Refactor for reuse.** You inherit this code — refactor it into named, composed types with zero repeated shape definitions, keeping all call sites compiling:
   ```ts
   function renderCard(item: { id: string; title: string; imageUrl?: string }) {}
   function saveItem(item: { id: string; title: string; imageUrl?: string; draft: boolean }) {}
   function shareItem(item: { id: string; title: string }, to: { id: string; email: string }) {}
   ```
