# Chapter 8: Objects

## Overview

If arrays are *lists*, objects are *records*. An **object** groups related data under named labels: a user has a `name`, an `email`, and an `age`; a product has a `title`, a `price`, and a `stock` count. Objects are the backbone of JavaScript — nearly all real-world data you'll handle (API responses, form data, app state) arrives shaped as objects, arrays of objects, or objects containing arrays.

This chapter covers creating objects, reading and writing properties, methods, iterating over objects, copying them safely, and the crucial concept of **reference semantics** — the source of many "why did my data change?!" bugs.

## Definitions & Explanations

### Object literals

The most common way to create an object is the **object literal**:

```js
const user = {
  name: "Ada",          // key: value
  email: "ada@math.org",
  age: 36,
};
```

- Each entry is a **property**: a **key** (also called a *property name* — always a string under the hood) and a **value** (any type, including arrays, other objects, and functions).
- Trailing commas after the last property are allowed and common.

### Reading and writing properties

- **Dot notation**: `user.name` — clean and standard when you know the key.
- **Bracket notation**: `user["name"]` — required when the key is stored in a variable, contains spaces/dashes, or is computed:

```js
const key = "email";
console.log(user[key]);        // bracket: uses the VARIABLE's value as the key
console.log(user.key);         // ❌ looks for a property literally named "key"
```

- Assigning to a property that doesn't exist **creates** it: `user.city = "London";`
- `delete user.age;` removes a property (rarely needed).
- Reading a missing property gives `undefined` (no error).

### Methods and `this`

A function stored on an object is called a **method**:

```js
const counter = {
  count: 0,
  increment() {           // shorthand method syntax
    this.count += 1;      // `this` = the object the method was called on
  },
};
```

Inside a regular method, **`this` refers to the object before the dot** at the call site (`counter.increment()` → `this` is `counter`). Arrow functions do *not* get their own `this`, so **don't use arrow functions for methods that need `this`**.

### Objects are references

This is the big one. Primitives (numbers, strings, booleans) are copied *by value*. Objects (and arrays!) are copied **by reference** — the variable holds a *pointer* to the object, not the object itself:

```js
const a = { score: 1 };
const b = a;        // b points to the SAME object
b.score = 99;
console.log(a.score); // 99 — "both" changed, because there's only one object
```

To actually copy an object one level deep: `const copy = { ...original };` (spread). For fully nested copies, use `structuredClone(original)`.

This also explains object equality: `{a:1} === {a:1}` is `false` — two different objects, even with identical contents. `===` on objects asks "are these the *same* object?"

### Iterating and introspecting

- `Object.keys(obj)` → array of keys.
- `Object.values(obj)` → array of values.
- `Object.entries(obj)` → array of `[key, value]` pairs — great with `for...of`.
- `for (const key in obj)` — iterates keys directly.
- `"name" in obj` or `obj.name !== undefined` — does the property exist?

### Shorthand & computed properties

```js
const name = "Ada";
const user = { name };            // shorthand for { name: name }

const field = "score";
const stats = { [field]: 100 };   // computed key → { score: 100 }
```

## Code Examples

### 1. Building and using a record

```js
const book = {
  title: "Eloquent JavaScript",
  author: "Marijn Haverbeke",
  pages: 472,
  isRead: false,
  tags: ["programming", "javascript"],   // arrays inside objects: totally normal
};

console.log(book.title);                  // "Eloquent JavaScript"
console.log(book.tags[0]);                // "programming"

book.isRead = true;                       // update
book.rating = 5;                          // create a new property
console.log(Object.keys(book));           // ["title","author","pages","isRead","tags","rating"]
```

### 2. Nested objects and safe access

```js
const order = {
  id: 1042,
  customer: {
    name: "Grace",
    address: { city: "Portland", zip: "97201" },
  },
  items: [
    { sku: "A1", qty: 2 },
    { sku: "B7", qty: 1 },
  ],
};

console.log(order.customer.address.city); // "Portland"
console.log(order.items[1].sku);          // "B7"

// Optional chaining (?.) avoids crashes when something may be missing:
console.log(order.shipping?.carrier);     // undefined — no error!
// Without ?. this would throw: Cannot read properties of undefined
```

### 3. Methods and `this`

```js
const bankAccount = {
  owner: "Sam",
  balance: 100,

  deposit(amount) {
    this.balance += amount;
    return this.balance;
  },

  withdraw(amount) {
    if (amount > this.balance) {
      return "Insufficient funds";
    }
    this.balance -= amount;
    return this.balance;
  },

  summary() {
    return `${this.owner}: $${this.balance.toFixed(2)}`;
  },
};

bankAccount.deposit(50);
console.log(bankAccount.withdraw(30));  // 120
console.log(bankAccount.summary());     // "Sam: $120.00"
```

### 4. Iterating over an object

```js
const stock = { apples: 10, bananas: 0, cherries: 42 };

// Object.entries + for...of — the go-to pattern
for (const [fruit, qty] of Object.entries(stock)) {
  console.log(`${fruit}: ${qty > 0 ? qty : "OUT OF STOCK"}`);
}

// Turn an object into derived data with array methods:
const totalItems = Object.values(stock).reduce((a, b) => a + b, 0);
console.log(totalItems); // 52

const available = Object.keys(stock).filter((k) => stock[k] > 0);
console.log(available); // ["apples", "cherries"]
```

### 5. References, copies, and equality

```js
const original = { theme: "dark", fontSize: 14 };

// ❌ Not a copy — an alias:
const alias = original;
alias.theme = "light";
console.log(original.theme); // "light" — original changed too!

// ✅ Shallow copy with spread:
const copy = { ...original };
copy.fontSize = 20;
console.log(original.fontSize); // 14 — safe

// ⚠️ Shallow means nested objects are still shared:
const config = { colors: { bg: "black" } };
const shallow = { ...config };
shallow.colors.bg = "white";
console.log(config.colors.bg);  // "white" — nested object is shared!

// ✅ Deep copy when you need full independence:
const deep = structuredClone(config);

// Equality is by identity:
console.log({ a: 1 } === { a: 1 }); // false — two different objects
```

### 6. Arrays of objects — the shape of real data

```js
const todos = [
  { id: 1, text: "Learn objects", done: true },
  { id: 2, text: "Practice spread", done: false },
  { id: 3, text: "Build a project", done: false },
];

const remaining = todos.filter((t) => !t.done).length;
console.log(`${remaining} tasks remaining`); // "2 tasks remaining"

// Update one item WITHOUT mutating (the pattern used in React and elsewhere):
const updated = todos.map((t) => (t.id === 2 ? { ...t, done: true } : t));
console.log(updated[1].done); // true
console.log(todos[1].done);   // false — original untouched
```

## Common Pitfalls

### 1. Dot notation with a variable key

```js
const settings = { volume: 7 };
const prop = "volume";
console.log(settings.prop);   // ❌ undefined — looks for key "prop"
console.log(settings[prop]);  // ✅ 7
```

### 2. Mutating a shared object by accident

```js
function resetScore(player) {
  player.score = 0;    // ❌ mutates the caller's object!
  return player;
}

// ✅ Return a modified copy instead:
function resetScoreSafe(player) {
  return { ...player, score: 0 };
}
```

### 3. Comparing objects with `===`

```js
const a = { x: 1 };
const b = { x: 1 };
console.log(a === b);  // false — identity, not contents

// ✅ For simple cases compare the relevant fields,
// or serialize: JSON.stringify(a) === JSON.stringify(b) (crude but common)
```

### 4. Arrow functions as methods

```js
const timer = {
  seconds: 0,
  tick: () => {
    this.seconds++;    // ❌ `this` is NOT timer here — arrow functions
  },                   //    don't get their own `this`
};

const timerFixed = {
  seconds: 0,
  tick() {             // ✅ shorthand method — `this` works
    this.seconds++;
  },
};
```

### 5. Reading deep into possibly-missing data

```js
const user = { name: "Ada" };
// console.log(user.address.city);   // ❌ TypeError: Cannot read properties of undefined
console.log(user.address?.city);     // ✅ undefined — optional chaining
console.log(user.address?.city ?? "Unknown"); // ✅ with a default
```

## Practice Exercises

1. **Your library.** Create an object describing your favorite movie: title, year, director (a nested object with name and country), and an array of genres. Log a sentence built from at least four of those fields with a template literal.

2. **Inventory report.** Given `const pantry = { rice: 2, pasta: 0, beans: 5, flour: 1 }`, print each item on its own line as `"item × qty"`, compute the total number of items, and build an array of item names that are out of stock. Use `Object.entries` / `Object.values`.

3. **Playlist object.** Create a `playlist` object with a `songs` array (start empty), plus methods `add(title)`, `remove(title)`, and `describe()` returning `"N songs: A, B, C"`. Use `this` throughout, and verify the methods work in sequence.

4. **Reference detective.** Write code that demonstrates (a) that assigning an object to a second variable does *not* copy it, (b) that spread makes a working shallow copy, and (c) a case where the shallow copy still shares nested data. Add comments explaining each observation.

5. **Contact updater.** Given an array of contact objects `{ id, name, email, favorite }` (make 4), write: `toggleFavorite(contacts, id)` returning a *new* array with just that contact's `favorite` flipped (no mutation); and `findByEmail(contacts, email)` returning the contact or the string `"not found"`. Prove the original array is unchanged after `toggleFavorite`.
