# Chapter 8: Classes in TypeScript

## Overview

You know JavaScript classes: constructors, methods, `extends`, `super`, getters, `static`, `#private` fields. TypeScript keeps all of that and adds a typed layer: property declarations, access modifiers (`public` / `private` / `protected`), `readonly`, parameter properties, `abstract` classes, and the ability for a class to `implements` an interface contract.

Two framing points before the syntax:

1. **A class creates both a value and a type.** `class User {}` gives you the constructor (a runtime value) *and* the instance type `User` usable in annotations. This dual nature is unique to classes (and enums) — interfaces are types only; functions are values only.
2. **Classes are one tool, not the default.** Idiomatic TypeScript uses plain objects + interfaces + functions for data, and classes where they earn it: stateful services, things with lifecycles, encapsulated invariants, or ecosystem expectations (error subclasses, some frameworks). Don't port Java habits wholesale.

## Definitions & Explanations

**Property declarations** — Unlike JS, TypeScript requires instance properties to be declared with types in the class body. Under `strictPropertyInitialization` (part of strict), every declared property must be definitely assigned — in the declaration or the constructor — or typed to include `undefined`.

**Access modifiers** —
- `public` (default) — accessible everywhere. Rarely written explicitly except in parameter properties.
- `private` — accessible only inside this class. **Compile-time only**: erased in JS, visible in devtools, bypassable with bracket access. Contrast with JS `#private` fields, which are enforced at runtime. Both are used in real codebases; `private` is more common in app code, `#` when hard privacy matters.
- `protected` — accessible in this class *and its subclasses*, not from outside.

**`readonly` properties** — Assignable only at declaration or in the constructor; immutable (at compile time) afterward. Perfect for IDs and injected dependencies.

**Parameter properties** — TypeScript shorthand: putting a modifier on a constructor parameter (`constructor(private repo: UserRepo)`) declares the property *and* assigns it in one stroke. Massive boilerplate saver, standard in service classes.

**`implements`** — Declares that a class satisfies an interface, making the compiler verify every member. Key subtlety: `implements` *checks* the class; it doesn't change types or inject anything. Multiple interfaces allowed: `class A implements B, C`.

**`abstract` classes** — Cannot be instantiated; exist to be extended. May mix implemented methods (shared logic) with `abstract` members (each subclass must supply). Choose an abstract class over an interface when there's *shared implementation* to inherit; choose an interface when there's only a contract.

**Getters/setters, static, generics** — All work as in JS, now typed. `static` members belong to the constructor value; generic classes (`class Box<T>`) fix their type argument per instance (Chapter 7).

**Structural typing still rules** — Even class instances are compared by shape. A plain object matching `User`'s public shape is assignable to `User`… *unless* the class has `private`/`protected` members, which make it effectively nominal (only real instances qualify). This is occasionally used deliberately (branding).

## Code Examples

### The basics, typed

```ts
class Timer {
  // Property declarations — required, with types:
  label: string;
  private startedAt: number | null = null;
  readonly createdAt = new Date();          // inferred Date, readonly

  constructor(label: string) {
    this.label = label;
    // strictPropertyInitialization: if we forgot to assign 'label',
    // Error: Property 'label' has no initializer and is not definitely
    // assigned in the constructor.
  }

  start(): void {
    this.startedAt = Date.now();
  }

  elapsed(): number {
    if (this.startedAt === null) throw new Error(`${this.label} not started`);
    return Date.now() - this.startedAt;     // narrowing works on properties too
  }
}

const t = new Timer("db-query");   // t: Timer  (class as a type!)
t.start();

t.startedAt;
// Error: Property 'startedAt' is private and only accessible within class 'Timer'.
t.createdAt = new Date();
// Error: Cannot assign to 'createdAt' because it is a read-only property.
```

### Parameter properties — the boilerplate killer

```ts
// ❌ The long way — name each property three times:
class OrderServiceVerbose {
  private repo: OrderRepo;
  private logger: Logger;
  constructor(repo: OrderRepo, logger: Logger) {
    this.repo = repo;
    this.logger = logger;
  }
}

// ✅ Parameter properties — declare + assign in the signature:
class OrderService {
  constructor(
    private readonly repo: OrderRepo,
    private readonly logger: Logger,
  ) {}

  async cancel(orderId: string): Promise<void> {
    this.logger.info(`cancelling ${orderId}`);
    await this.repo.delete(orderId);
  }
}

interface OrderRepo { delete(id: string): Promise<void> }
interface Logger { info(msg: string): void }
```

### `implements` — contract checking

```ts
interface Storage2 {
  get(key: string): string | null;
  set(key: string, value: string): void;
}

class MemoryStorage implements Storage2 {
  private data = new Map<string, string>();

  get(key: string): string | null {
    return this.data.get(key) ?? null;
  }

  set(key: string, value: string): void {
    this.data.set(key, value);
  }
}

// Forget a member and the CLASS errors immediately:
class BrokenStorage implements Storage2 {
  get(key: string): string | null { return null; }
  // Error: Class 'BrokenStorage' incorrectly implements interface 'Storage2'.
  //   Property 'set' is missing in type 'BrokenStorage'.
}

// The payoff — code depends on the interface, swaps implementations freely:
function seed(store: Storage2) {
  store.set("theme", "dark");
}
seed(new MemoryStorage());        // works with any conforming class
```

Note: `implements` does **not** auto-type your members. Write parameter/return types on the methods; `implements` then verifies compatibility.

### `protected` and inheritance

```ts
class Repository<T extends { id: string }> {
  protected items = new Map<string, T>();

  add(item: T): void {
    this.items.set(item.id, item);
  }

  findById(id: string): T | undefined {
    return this.items.get(id);
  }
}

interface Article { id: string; title: string; published: boolean }

class ArticleRepository extends Repository<Article> {
  findPublished(): Article[] {
    // 'items' is protected → visible here in the subclass:
    return [...this.items.values()].filter(a => a.published);
  }
}

const repo = new ArticleRepository();
repo.items;
// Error: Property 'items' is protected and only accessible within
// class 'Repository<T>' and its subclasses.
```

### Abstract classes — shared skeleton, enforced gaps

```ts
abstract class ReportGenerator {
  // Shared, concrete logic (why we chose abstract class over interface):
  generate(data: string[]): string {
    const body = this.formatBody(data);
    return `${this.header()}\n${body}\n--- ${data.length} rows ---`;
  }

  protected header(): string {
    return `Report ${new Date().toISOString()}`;
  }

  // The gap every subclass MUST fill:
  protected abstract formatBody(data: string[]): string;
}

class CsvReport extends ReportGenerator {
  protected formatBody(data: string[]): string {
    return data.join(",");
  }
}

class MarkdownReport extends ReportGenerator {
  protected formatBody(data: string[]): string {
    return data.map(d => `- ${d}`).join("\n");
  }
}

new ReportGenerator();
// Error: Cannot create an instance of an abstract class.

const report: ReportGenerator = new CsvReport();   // abstract type as annotation ✅
report.generate(["a", "b"]);
```

### Class as value AND type; structural quirks

```ts
class Point {
  constructor(public x: number, public y: number) {}
}

// Type position — instance type:
const p: Point = new Point(1, 2);

// Value position — the constructor itself:
const PointClass: typeof Point = Point;
const p2 = new PointClass(3, 4);

// Structural typing: a lookalike object passes for a class with only public members…
const fake: Point = { x: 0, y: 0 };   // OK! shape matches

// …but one private member makes the class nominal:
class Id {
  private brand = true;
  constructor(public value: string) {}
}
const fakeId: Id = { value: "x", brand: true };
// Error: Property 'brand' is private in type 'Id' but not in type '{...}'.
```

### Extending built-ins: custom errors

```ts
class ApiError extends Error {
  constructor(
    message: string,
    public readonly status: number,
    public readonly url: string,
  ) {
    super(message);
    this.name = "ApiError";
  }
}

try {
  throw new ApiError("Not Found", 404, "/api/users/9");
} catch (err) {
  if (err instanceof ApiError) {           // instanceof works — classes are runtime!
    console.error(`${err.status} from ${err.url}`);
  }
}
```

## Common Pitfalls

**Pitfall 1: Believing `private` hides data at runtime.**
`private` is a compile-time promise. `JSON.stringify` serializes it; `(obj as any).secret` reads it; devtools show it. For genuine runtime privacy (library boundaries, security-adjacent code), use JS `#fields`. Know which you're using and why.

**Pitfall 2: Forgetting property declarations.**

```ts
class Bad {
  constructor() {
    this.count = 0;
    // Error: Property 'count' does not exist on type 'Bad'.
  }
}
// ✅ Declare it: class Good { count = 0; }  (or count: number; + constructor assign)
```

In JS you conjure properties in the constructor; in TS the class body is the source of truth.

**Pitfall 3: Silencing `strictPropertyInitialization` with `!`.**

```ts
class Widget {
  element!: HTMLElement;   // "definite assignment assertion" — a promise, unchecked
}
```

The `!` is legitimate when a framework assigns the property outside the constructor (lifecycle hooks, DI). As a reflex to make errors go away, it reintroduces exactly the undefined-property crashes strict mode prevents. Prefer initializing in the constructor or typing as `| null`.

**Pitfall 4: Losing `this` when passing methods as callbacks.**
Pure JS problem, still yours in TS — and the types won't always save you:

```ts
class Counter2 {
  count = 0;
  increment() { this.count++; }
}
const c = new Counter2();
const fn = c.increment;
fn(); // runtime: this is undefined → TypeError (TS may error under strict, but not always)
// ✅ bind it, wrap it, or declare as arrow property: increment = () => { this.count++ }
```

**Pitfall 5: Interface-itis / class-itis.**
Two opposite mistakes: (a) writing a class for every data shape (a `User` with no behavior should be an interface + plain objects — cheaper, serializable, spreadable); (b) avoiding classes so hard you reimplement stateful encapsulation with closures everywhere. The rule: **data → interfaces; behavior + state + invariants → maybe a class.**

**Pitfall 6: Expecting `implements` to do inheritance.**
`implements` copies nothing — no method bodies, no defaults. If you want inherited behavior, you want `extends` (possibly of an abstract class). A class can do both: `class A extends Base implements Contract`.

## Practice Exercises

1. **Bank account with invariants.** Build `class BankAccount` with a `readonly` `iban`, a private balance, `deposit(amount)` / `withdraw(amount)` that throw on invalid amounts or insufficient funds, and a getter `balance`. Prove from outside the class that direct balance manipulation is a compile error, and that a negative-deposit call throws at runtime.

2. **Parameter-property refactor.** Take a class written the "long way" (declare fields, assign in constructor — write one with 3 dependencies yourself) and refactor it to parameter properties with appropriate `private readonly` modifiers, confirming behavior and types are unchanged.

3. **Contract + swap.** Define `interface NotificationChannel { send(to: string, message: string): Promise<void> }`. Implement `EmailChannel` and `ConsoleChannel` (the latter just logs). Write `notifyAll(channels: NotificationChannel[], msg: string)` and run it with both. Then remove a method from one class and record the `implements` error verbatim.

4. **Abstract shapes.** Create `abstract class Shape` with concrete `describe(): string` (uses area) and abstract `area(): number`, plus subclasses `Circle` and `Rectangle` using parameter properties. Store mixed shapes in a `Shape[]` and print descriptions. Explain in one sentence why an interface alone wouldn't have been enough here.

5. **Nominal branding.** Using the private-member trick from this chapter, create a `UserId` class wrapping a string such that a raw string (or a lookalike object) cannot be passed where a `UserId` is required. Write `function loadUser(id: UserId)` and demonstrate the rejection of `loadUser("u1" as any as never)` — then reflect: when might this ceremony be worth it in a real app?
