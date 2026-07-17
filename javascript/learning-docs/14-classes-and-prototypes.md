# Chapter 14: Classes & Prototypes

## Overview

When your program manages many objects of the same *kind* — many users, many tasks, many products — you want a blueprint that stamps them out consistently, with shared behavior. That's what **classes** provide: a template combining data (properties) and behavior (methods).

JavaScript's `class` syntax (added in ES6) looks like classes in other languages, but under the hood it's built on **prototypes** — JavaScript's original object-linking mechanism. You'll write classes daily and rarely touch prototypes directly, but understanding the prototype layer explains many "weird" behaviors and is a favorite interview topic.

## Definitions & Explanations

### Class syntax

```js
class Task {
  constructor(title) {        // runs once per `new Task(...)`
    this.title = title;       // instance properties: unique per object
    this.done = false;
  }

  complete() {                // method: shared by ALL instances
    this.done = true;
  }
}

const t = new Task("Learn classes");  // `new` creates an instance
```

- **`constructor`** initializes each new instance. `this` inside it refers to the object being created.
- **Methods** defined in the class body are shared — stored once, usable by every instance.
- **`new`** is required. Calling `Task("x")` without `new` throws.
- `t instanceof Task` → `true` — checks what class created an object.

### `this` in classes

Inside a method, `this` is the instance the method was called on (`t.complete()` → `this` is `t`). The classic gotcha: if you *detach* a method from its object (e.g., pass `t.complete` as a callback), `this` gets lost. Fixes: wrap in an arrow (`() => t.complete()`), or bind (`t.complete.bind(t)`), or define the method as an arrow-function class field.

### Getters, setters, static members

- **Getter** — a method accessed like a property: `get fullName() {...}` → `user.fullName` (no parentheses). Great for computed values.
- **Setter** — intercepts assignment: `set age(v) {...}` → `user.age = 30` runs your validation code.
- **Static** — belongs to the class itself, not instances: `static compare(a, b) {...}` → `Task.compare(t1, t2)`. Use for helpers and factory functions.

### Private fields

Prefix with `#` to make a field truly inaccessible from outside:

```js
class Wallet {
  #balance = 0;          // only code inside the class can touch #balance
}
```

### Inheritance: `extends` and `super`

```js
class AdminUser extends User { ... }
```

- The child class gets everything the parent has.
- `super(...)` in the child constructor calls the parent constructor — **required before using `this`**.
- `super.method()` calls the parent's version of an overridden method.
- Prefer *shallow* hierarchies (one level). Deep inheritance trees are widely considered a design smell — often composition ("has-a") beats inheritance ("is-a").

### Prototypes — what's really happening

Every object has a hidden link to a **prototype** object. When you read `t.complete`, JavaScript looks on `t` itself, then on its prototype, then the prototype's prototype, and so on up the **prototype chain** until found (or `undefined`).

- Class methods live on `Task.prototype`; instances link to it. That's why methods are shared: one copy, looked up through the chain.
- `Object.getPrototypeOf(t) === Task.prototype` → `true`.
- The chain ends at `Object.prototype` (home of `toString`, `hasOwnProperty`), then `null`.
- Arrays work the same way: `[1,2,3]` finds `map` on `Array.prototype`.

`class` is (mostly) **syntactic sugar** over this mechanism — a nicer way to write what was always possible with constructor functions and prototypes.

## Code Examples

### 1. A complete basic class

```js
class Book {
  constructor(title, author, pages) {
    this.title = title;
    this.author = author;
    this.pages = pages;
    this.currentPage = 1;
  }

  read(count) {
    this.currentPage = Math.min(this.currentPage + count, this.pages);
    return this;                      // returning this enables chaining
  }

  get progress() {                    // computed property
    return `${Math.round((this.currentPage / this.pages) * 100)}%`;
  }

  describe() {
    return `"${this.title}" by ${this.author} — ${this.progress} read`;
  }
}

const book = new Book("Dune", "Frank Herbert", 412);
book.read(100).read(50);              // chaining thanks to `return this`
console.log(book.progress);           // "37%" — getter: no parentheses
console.log(book.describe());         // "Dune" by Frank Herbert — 37% read

const other = new Book("Emma", "Jane Austen", 474);
console.log(other.progress);          // "0%"... each instance has its own data
console.log(book.read === other.read); // true — but methods are SHARED
```

### 2. Private fields with getters/setters

```js
class Thermostat {
  #temperature = 20;                  // private field with a default

  get temperature() {
    return this.#temperature;
  }

  set temperature(value) {            // validation on assignment
    if (typeof value !== "number") throw new TypeError("Temperature must be a number");
    if (value < 5 || value > 30) throw new RangeError("Temperature must be 5–30");
    this.#temperature = value;
  }
}

const t = new Thermostat();
t.temperature = 23;                   // runs the setter
console.log(t.temperature);           // 23 — runs the getter
// t.#temperature                     // ❌ SyntaxError — truly private
// t.temperature = 99                 // ❌ RangeError from the setter
```

### 3. Static members

```js
class Money {
  constructor(cents) {
    this.cents = cents;
  }

  static fromDollars(dollars) {       // factory: called on the CLASS
    return new Money(Math.round(dollars * 100));
  }

  static ZERO = new Money(0);         // static property

  add(other) {
    return new Money(this.cents + other.cents);
  }

  toString() {                        // customizes string conversion
    return `$${(this.cents / 100).toFixed(2)}`;
  }
}

const price = Money.fromDollars(19.99);
const tip = Money.fromDollars(3);
console.log(`${price.add(tip)}`);     // "$22.99" — toString invoked automatically
console.log(`${Money.ZERO}`);         // "$0.00"
```

### 4. Inheritance

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound.`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);                      // MUST call parent constructor first
    this.breed = breed;
  }
  speak() {                           // override
    return `${this.name} barks!`;
  }
  describe() {
    return `${super.speak()} (Actually: ${this.speak()})`; // parent version via super
  }
}

const rex = new Dog("Rex", "Border Collie");
console.log(rex.speak());             // "Rex barks!"
console.log(rex.describe());          // "Rex makes a sound. (Actually: Rex barks!)"
console.log(rex instanceof Dog);      // true
console.log(rex instanceof Animal);   // true — the chain includes Animal
```

### 5. Peeking under the hood at prototypes

```js
class Point {
  constructor(x, y) { this.x = x; this.y = y; }
  length() { return Math.sqrt(this.x ** 2 + this.y ** 2); }
}

const p = new Point(3, 4);

// Instance data lives ON p; methods live on the prototype:
console.log(Object.keys(p));                            // ["x", "y"] — no "length"!
console.log(Object.getPrototypeOf(p) === Point.prototype); // true
console.log(p.length());                                // 5 — found via the chain

// The chain: p → Point.prototype → Object.prototype → null
console.log(p.toString());  // "[object Object]" — inherited from Object.prototype
console.log(p.hasOwnProperty("x"));       // true
console.log(p.hasOwnProperty("length"));  // false — it's on the prototype
```

### 6. Classes with the DOM — a realistic component

```js
class Counter {
  #count = 0;

  constructor(rootElement) {
    this.root = rootElement;
    this.root.innerHTML = `<button>+1</button> <span>0</span>`; // our own markup — safe
    // Arrow function keeps `this` bound to the Counter instance:
    this.root.querySelector("button").addEventListener("click", () => this.increment());
  }

  increment() {
    this.#count++;
    this.root.querySelector("span").textContent = this.#count;
  }
}

// Turn every <div class="counter"> on the page into a live widget:
document.querySelectorAll(".counter").forEach((el) => new Counter(el));
```

## Common Pitfalls

### 1. Forgetting `new`

```js
class User { constructor(n) { this.name = n; } }
// const u = User("Ada");   // ❌ TypeError: Class constructor cannot be invoked without 'new'
const u = new User("Ada");  // ✅
```

### 2. Losing `this` when passing methods around

```js
class Logger {
  prefix = "[app]";
  log(msg) { console.log(this.prefix, msg); }
}
const logger = new Logger();

setTimeout(logger.log, 100, "hi");        // ❌ this is undefined → crash
setTimeout(() => logger.log("hi"), 100);  // ✅ arrow preserves the call form
setTimeout(logger.log.bind(logger), 100, "hi"); // ✅ or bind
```

### 3. Forgetting `super()` in a subclass constructor

```js
class Cat extends Animal {
  constructor(name) {
    // this.name = name;   // ❌ ReferenceError: must call super before 'this'
    super(name);           // ✅ first line
  }
}
```

### 4. Putting shared mutable data in the wrong place

```js
class TeamBad {
  static members = [];          // ❌ ONE array shared by every team!
}
class Team {
  constructor() {
    this.members = [];          // ✅ per-instance array, made in the constructor
  }
}
```

Similarly, a class field initialized to an object/array is per-instance (fine), but a *static* one is shared — know which you want.

### 5. Overusing classes

Not everything needs a class. A bag of related functions? Use plain functions and objects. One-off state? A closure (Chapter 13) may be simpler. Reach for classes when you'll create *multiple instances* sharing *behavior*.

## Practice Exercises

1. **Playlist class.** Build `class Playlist` with a constructor taking a name; methods `add(song)`, `remove(song)`, and `play()` returning `"Now playing: <first song>"` or `"Playlist empty"`; and a getter `size`. Songs are strings stored in an instance array. Create two playlists and verify their independence.

2. **Temperature with guards.** Write a `Temperature` class storing celsius in a private `#celsius` field, with getter/setter `celsius` (setter throws on non-numbers or values below −273.15) and a getter `fahrenheit` computed from celsius. Add a static `fromFahrenheit(f)` factory.

3. **Shape hierarchy.** Create `class Shape` with `constructor(name)` and a method `area()` that throws `"Not implemented"`. Extend it with `Rectangle(w, h)` and `Circle(r)` overriding `area()`. Write a loop over an array of mixed shapes printing `"<name>: <area>"` — this is polymorphism in action.

4. **Prototype exploration.** Using your `Rectangle` class: show with `Object.keys` which properties live on the instance; verify with `Object.getPrototypeOf` that the instance links to `Rectangle.prototype` and that *its* prototype is `Shape.prototype`; and confirm `area` is not an "own" property of the instance. Comment each finding.

5. **Bank account, class edition.** Rebuild Chapter 13's `createBankAccount` closure as a class with a private `#balance`, methods `deposit`/`withdraw` (with the same validations, throwing Errors), and a getter `balance`. Then write two sentences (as comments) on which version you prefer and why.
