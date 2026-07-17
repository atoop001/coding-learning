# Chapter 5: Loops & Iteration

## Overview

Computers excel at doing the same thing many times without complaint. **Loops** are how you tell them to: process every item in a list, retry until something succeeds, count from 1 to 1,000,000. Without loops you'd copy-paste code; with them, three lines can do a million steps.

This chapter covers `for`, `while`, `do...while`, the modern `for...of` and `for...in` loops, and the loop-control keywords `break` and `continue`. Loops pair naturally with arrays (Chapter 7), but the mechanics stand on their own and are worth mastering first.

## Definitions & Explanations

### The classic `for` loop

```js
for (initialization; condition; update) {
  // body — runs once per iteration
}
```

- **Initialization** runs once, before anything else — typically `let i = 0`.
- **Condition** is checked *before* each iteration; when falsy, the loop ends.
- **Update** runs *after* each iteration — typically `i++`.

`i` (for *index*) is the traditional counter name. The pattern `for (let i = 0; i < n; i++)` runs the body exactly `n` times, with `i` taking values `0` through `n - 1`.

### The `while` loop

```js
while (condition) {
  // runs as long as condition stays truthy
}
```

Use `while` when you *don't know in advance* how many iterations you need — "keep asking until the answer is valid," "keep halving until below 1."

### The `do...while` loop

```js
do {
  // runs at least once, THEN checks the condition
} while (condition);
```

Same as `while`, but the body always executes at least once because the check happens *after*. Classic use: prompting a user at least once.

### `for...of` — looping over values

```js
for (const item of iterable) {
  // item is each VALUE in turn
}
```

Works on anything *iterable*: arrays, strings, Maps, Sets. This is the cleanest way to visit every element when you don't need the index.

### `for...in` — looping over keys

```js
for (const key in object) {
  // key is each property NAME (a string)
}
```

Designed for object property names. **Do not use `for...in` on arrays** — it gives you index *strings* (`"0"`, `"1"`) plus any extra properties, in no guaranteed order. Mnemonic: **`of` = values, `in` = keys**.

### `break` and `continue`

- **`break`** exits the loop entirely, immediately.
- **`continue`** skips the rest of the current iteration and jumps to the next one.

### Infinite loops

If the condition never becomes falsy, the loop runs forever and freezes the page/terminal (stop with the browser's "kill page" or `Ctrl+C` in Node). Every loop needs something that moves it toward stopping.

## Code Examples

### 1. Counting with `for`

```js
// Count 1 to 5
for (let i = 1; i <= 5; i++) {
  console.log("Count:", i);
}

// Count down
for (let i = 5; i >= 1; i--) {
  console.log("T-minus", i);
}

// Step by more than 1: even numbers 0..10
for (let i = 0; i <= 10; i += 2) {
  console.log(i);
}
```

### 2. Accumulating a result

The single most common loop pattern: start with an "accumulator," update it every iteration.

```js
// Sum of 1..100
let sum = 0;
for (let i = 1; i <= 100; i++) {
  sum += i;
}
console.log(sum); // 5050

// Building a string
let row = "";
for (let i = 0; i < 8; i++) {
  row += i % 2 === 0 ? "⬜" : "⬛";
}
console.log(row); // ⬜⬛⬜⬛⬜⬛⬜⬛
```

### 3. `while` for unknown iteration counts

```js
// How many times can you halve 1000 before reaching 1 or less?
let value = 1000;
let steps = 0;

while (value > 1) {
  value /= 2;
  steps++;
}
console.log(`Took ${steps} halvings`); // Took 10 halvings

// do...while: simulate rolling dice until you get a 6 (runs at least once)
let roll;
let attempts = 0;
do {
  roll = Math.floor(Math.random() * 6) + 1; // random integer 1–6
  attempts++;
} while (roll !== 6);
console.log(`Rolled a 6 after ${attempts} attempt(s)`);
```

### 4. `for...of` over arrays and strings

```js
const groceries = ["milk", "bread", "eggs", "coffee"];

// Clean value iteration — no index bookkeeping
for (const item of groceries) {
  console.log("Buy:", item);
}

// When you DO need the index, use entries():
for (const [index, item] of groceries.entries()) {
  console.log(`${index + 1}. ${item}`);
}

// Strings are iterable too — character by character:
for (const ch of "hi!") {
  console.log(ch); // h, i, !
}
```

### 5. `break` and `continue`

```js
const pins = ["1234", "0000", "9999", "4321"];
const target = "9999";

// break: stop searching once found
for (const pin of pins) {
  if (pin === target) {
    console.log("Found the PIN!");
    break; // no point checking the rest
  }
  console.log("Checked", pin, "- not it");
}

// continue: skip items that don't apply
for (let i = 1; i <= 10; i++) {
  if (i % 3 !== 0) continue; // skip non-multiples of 3
  console.log(i); // 3, 6, 9
}
```

### 6. Nested loops

```js
// A multiplication table (rows × columns)
for (let row = 1; row <= 3; row++) {
  let line = "";
  for (let col = 1; col <= 3; col++) {
    line += `${row}×${col}=${row * col}  `;
  }
  console.log(line);
}
// 1×1=1  1×2=2  1×3=3
// 2×1=2  2×2=4  2×3=6
// 3×1=3  3×2=6  3×3=9
```

Each full run of the inner loop happens once per outer iteration — 3 × 3 = 9 body executions here. Nested loops multiply work quickly; be mindful with large data.

## Common Pitfalls

### 1. Off-by-one errors

```js
const letters = ["a", "b", "c"];

// ❌ <= runs one time too many; letters[3] is undefined
for (let i = 0; i <= letters.length; i++) {
  console.log(letters[i]); // a, b, c, undefined
}

// ✅ Arrays are 0-indexed; valid indexes are 0 .. length-1
for (let i = 0; i < letters.length; i++) {
  console.log(letters[i]);
}
```

### 2. The accidental infinite loop

```js
let i = 0;
// ❌ i never changes — runs forever, freezes the tab
// while (i < 5) {
//   console.log(i);
// }

// ✅ something inside must move toward the exit condition
while (i < 5) {
  console.log(i);
  i++;
}
```

### 3. Modifying the counter inside the body

```js
// ❌ Changing i in two places makes behavior hard to predict
for (let i = 0; i < 10; i++) {
  console.log(i);
  i += 2; // now it advances by 3 per iteration — confusing!
}
// ✅ Put ALL counter changes in the update slot: for (let i = 0; i < 10; i += 3)
```

### 4. `for...in` on an array

```js
const scores = [90, 85, 77];

// ❌ keys, as strings — "0", "1", "2"
for (const s in scores) {
  console.log(s + 1); // "01", "11", "21" — string concatenation!
}

// ✅ for...of gives the values
for (const s of scores) {
  console.log(s + 1); // 91, 86, 78
}
```

### 5. Expecting `break` to exit nested loops

```js
// ❌ break only exits the INNER loop
for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) break;   // exits inner loop only; outer keeps going
  }
}

// ✅ Use a label when you genuinely need to exit both:
outer:
for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i + j === 3) break outer;
  }
}
// (Better yet: restructure into a function and use return — Chapter 6.)
```

## Practice Exercises

1. **Countdown launcher.** Print a countdown from 10 to 1, then print `"Liftoff! 🚀"`. Next, modify it to skip odd numbers on the way down.

2. **Sum and average.** Loop over `const temps = [21, 24, 19, 26, 23, 22, 25]` with `for...of` to compute the total and the average temperature, then print both. Also find and print the highest value — without using `Math.max`.

3. **Times-table grid.** Using nested loops, print a full 10×10 multiplication table, one row per line. Bonus: pad the numbers so columns align (`String(n).padStart(4)` may help).

4. **Guess counter.** Using a `while` (or `do...while`) loop, repeatedly generate a random number 1–100 until it lands within 5 of 50, counting attempts. Print each miss as `"too high"` / `"too low"` and finish with the attempt count.

5. **Vowel census.** Given `const sentence = "Loops make computers do the boring work"`, iterate its characters with `for...of` and count how many are vowels. Use `continue` to skip spaces, and print the vowel count and the total non-space letter count.
