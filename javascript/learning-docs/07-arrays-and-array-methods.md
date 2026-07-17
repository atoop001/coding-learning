# Chapter 7: Arrays & Array Methods

## Overview

Real programs juggle *collections*: a list of tasks, a cart of products, a feed of messages. In JavaScript, the workhorse collection is the **array** — an ordered list of values. Arrays come with a rich toolbox of built-in methods, and three of them — **`map`**, **`filter`**, and **`reduce`** — are so central to modern JavaScript (and to job interviews) that they get their own spotlight in this chapter.

You already know loops (Chapter 5) and functions (Chapter 6); array methods combine both ideas: they are functions on arrays that take *your* function and apply it to every element.

## Definitions & Explanations

### Array basics

An array is written with square brackets, elements separated by commas:

```js
const colors = ["red", "green", "blue"];
```

- Elements are accessed by **index**, starting at **0**: `colors[0]` is `"red"`.
- `colors.length` is the number of elements (3), so the last index is `length - 1`.
- Arrays can hold any mix of types (numbers, strings, objects, other arrays), though same-type arrays are easier to work with.
- Arrays are objects — a `const` array can't be *reassigned*, but its *contents* can change.

### Mutating vs. non-mutating methods

This distinction matters constantly:

- **Mutating methods change the original array**: `push` (add to end), `pop` (remove from end), `unshift` (add to front), `shift` (remove from front), `splice` (remove/insert anywhere), `sort`, `reverse`.
- **Non-mutating methods return a new array/value and leave the original alone**: `map`, `filter`, `slice`, `concat`, `includes`, `indexOf`, `join`, `find`, `some`, `every`, `reduce`.

Modern style favors non-mutating methods — they make code easier to reason about because data doesn't change behind your back.

### The big three

- **`map(fn)`** — transforms every element. Returns a **new array of the same length** where each element is `fn(originalElement)`. "Turn each X into a Y."
- **`filter(fn)`** — selects elements. Returns a **new array containing only** the elements for which `fn(element)` returned truthy. "Keep the ones that pass the test."
- **`reduce(fn, initialValue)`** — boils an array down to a **single value** (number, string, object — anything). `fn(accumulator, element)` runs per element; whatever it returns becomes the accumulator for the next round.

### Searching and testing

- `find(fn)` — the **first element** matching the test (or `undefined`).
- `findIndex(fn)` — its index (or `-1`).
- `includes(value)` — boolean: is this exact value present?
- `some(fn)` — boolean: does **at least one** element pass?
- `every(fn)` — boolean: do **all** elements pass?

### `sort` — powerful but tricky

`sort()` sorts **in place** (mutates!) and, by default, compares elements **as strings** — so `[10, 2, 1].sort()` gives `[1, 10, 2]`. For numbers, pass a comparator: `arr.sort((a, b) => a - b)`. To sort without mutating, copy first: `[...arr].sort(...)` or use `arr.toSorted(...)` (newer).

## Code Examples

### 1. Creating, reading, updating

```js
const tasks = ["buy milk", "walk dog"];

console.log(tasks[0]);           // "buy milk"
console.log(tasks.length);       // 2
console.log(tasks[tasks.length - 1]); // "walk dog" — last element
console.log(tasks.at(-1));       // "walk dog" — cleaner way to get the last

tasks.push("write code");        // add to end → length 3
tasks.unshift("wake up");        // add to front → length 4
const done = tasks.shift();      // removes & returns "wake up"
console.log(tasks);              // ["buy milk", "walk dog", "write code"]

tasks[1] = "feed dog";           // replace by index
console.log(tasks.includes("buy milk")); // true
console.log(tasks.indexOf("write code")); // 2
```

### 2. `map` — transform every element

```js
const prices = [10, 25, 3.5];

// Each price → price with tax (same length, new array)
const withTax = prices.map((p) => p * 1.08);
console.log(withTax);  // [10.8, 27, 3.78]
console.log(prices);   // [10, 25, 3.5] — untouched!

// Map to a different type: numbers → display strings
const labels = prices.map((p) => `$${p.toFixed(2)}`);
console.log(labels);   // ["$10.00", "$25.00", "$3.50"]

// The callback also receives the index as a second argument:
const numbered = ["intro", "setup", "build"].map((step, i) => `${i + 1}. ${step}`);
console.log(numbered); // ["1. intro", "2. setup", "3. build"]
```

### 3. `filter` — keep what passes

```js
const scores = [93, 47, 88, 62, 75, 39];

const passing = scores.filter((s) => s >= 60);
console.log(passing); // [93, 88, 62, 75]

const words = ["apple", "hi", "banana", "ok", "cherry"];
const longWords = words.filter((w) => w.length > 3);
console.log(longWords); // ["apple", "banana", "cherry"]
```

### 4. `reduce` — many values, one result

```js
const cart = [12.99, 4.5, 30, 8.25];

// Sum: accumulator starts at 0, each step adds one price
const total = cart.reduce((acc, price) => acc + price, 0);
console.log(total); // 55.74

// Max without Math.max:
const max = cart.reduce((best, p) => (p > best ? p : best), cart[0]);
console.log(max); // 30

// Counting occurrences into an object (a very common interview pattern):
const votes = ["yes", "no", "yes", "yes", "no"];
const tally = votes.reduce((acc, v) => {
  acc[v] = (acc[v] ?? 0) + 1;
  return acc;                 // must return the accumulator!
}, {});
console.log(tally); // { yes: 3, no: 2 }
```

### 5. Chaining: the real power

Because `map`/`filter` return arrays, you can chain them:

```js
const products = [
  { name: "Laptop",   price: 900, inStock: true  },
  { name: "Mouse",    price: 25,  inStock: true  },
  { name: "Monitor",  price: 180, inStock: false },
  { name: "Keyboard", price: 55,  inStock: true  },
];

// "Names of in-stock products under $100, cheapest first"
const result = products
  .filter((p) => p.inStock && p.price < 100)
  .sort((a, b) => a.price - b.price)
  .map((p) => p.name);

console.log(result); // ["Mouse", "Keyboard"]

// Total value of in-stock inventory:
const inventoryValue = products
  .filter((p) => p.inStock)
  .reduce((sum, p) => sum + p.price, 0);
console.log(inventoryValue); // 980
```

### 6. find / some / every / join / slice

```js
const users = [
  { id: 1, name: "Ada",  admin: false },
  { id: 2, name: "Linus", admin: true },
];

console.log(users.find((u) => u.id === 2));        // { id: 2, name: "Linus", admin: true }
console.log(users.some((u) => u.admin));            // true — at least one admin
console.log(users.every((u) => u.admin));           // false — not all admins

console.log(["a", "b", "c"].join(" → "));           // "a → b → c"
console.log([1, 2, 3, 4, 5].slice(1, 3));           // [2, 3] — end index excluded, original untouched
```

## Common Pitfalls

### 1. Expecting `map`/`filter` to change the original

```js
const nums = [1, 2, 3];
nums.map((n) => n * 2);
console.log(nums);       // ❌ still [1, 2, 3] — the result was thrown away!

const doubled = nums.map((n) => n * 2); // ✅ capture the returned array
```

### 2. Using `map` when you mean `forEach` (or vice versa)

```js
// map is for TRANSFORMING; if you're only doing side effects (logging,
// updating the page), use forEach and don't pretend to build an array:
nums.forEach((n) => console.log(n));      // ✅ side effects
const strs = nums.map((n) => String(n));  // ✅ transformation

// ❌ Code smell: a map whose result is ignored
nums.map((n) => console.log(n));
```

### 3. Default `sort` on numbers

```js
// Default sort compares elements AS STRINGS:
console.log([1, 10, 2].sort());                // ❌ [1, 10, 2] — "1" < "10" < "2" alphabetically
console.log([1, 10, 2].sort((a, b) => a - b)); // ✅ [1, 2, 10] — numeric comparator
```

Also remember `sort` mutates — copy first if you need the original: `const sorted = [...nums].sort((a, b) => a - b);`

### 4. Forgetting to return the accumulator in `reduce`

```js
// ❌ acc becomes undefined after the first iteration
const bad = [1, 2, 3].reduce((acc, n) => { acc + n; }, 0); // undefined + ...
// ✅ return it (or use an expression body with no braces)
const good = [1, 2, 3].reduce((acc, n) => acc + n, 0); // 6
```

### 5. Out-of-range indexes and `find` misses

```js
const arr = [1, 2, 3];
console.log(arr[10]);                    // undefined — no error, easy to miss
const hit = arr.find((n) => n > 100);    // undefined when nothing matches
// ✅ Always handle the "not found" case:
if (hit === undefined) console.log("No match found");
```

## Practice Exercises

1. **Playlist manager.** Start with `const playlist = ["Track A", "Track B"]`. Using mutating methods: add two tracks to the end, one to the front, remove the last one, and print the final list numbered `1.`, `2.`, ... (use `map` + `join`, or `entries`).

2. **Temperature conversion.** Given `const celsius = [0, 12, 24, 30, -5]`, use `map` to produce Fahrenheit values, then `filter` to keep only "warm" readings above 70°F, then print how many there are.

3. **Word stats.** Given `const words = ["sky", "javascript", "loop", "arrays", "of", "reduce"]`: (a) use `filter` for words of 5+ letters; (b) use `map` to uppercase everything; (c) use `reduce` to find the total number of characters across all words; (d) use `reduce` to find the longest word.

4. **Grade book.** Given an array of `{ name, score }` objects (write 5 of your own), produce: the names of everyone scoring 80+, the class average via `reduce`, whether *anyone* failed (below 60) via `some`, and the top student via `reduce` — each as a separate chained expression.

5. **Cash register.** Given `const transactions = [4.75, -2, 13.5, 8, -5.25, 20]` (negatives are refunds): compute total revenue (sum of positives only), total refunds, the final balance, and a formatted receipt line for each transaction like `"+ $4.75"` / `"- $2.00"` joined with newlines. Use only array methods — no `for` loops.
