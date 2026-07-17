# Chapter 2: Variables & Data Types

## Overview

Programs work with data: names, prices, scores, on/off switches. To use data, you need somewhere to *put* it — that's what **variables** are for. And every piece of data has a **type** that determines what you can do with it (you can multiply numbers, but multiplying two names makes no sense).

This chapter covers how to declare variables with `let` and `const`, JavaScript's core data types, and how to inspect and convert between types. Nearly every bug you'll hit as a beginner traces back to a misunderstanding of variables or types, so time invested here pays off across the entire track.

## Definitions & Explanations

### Variables: labeled boxes for values

A **variable** is a named container for a value. You *declare* a variable (create it), optionally *initialize* it (give it a first value), and later *read* or *reassign* it.

JavaScript has three declaration keywords:

- **`let`** — declares a variable whose value **can be reassigned**.
- **`const`** — declares a variable whose value **cannot be reassigned**. Use this by default; it signals "this won't change," which makes code easier to reason about.
- **`var`** — the old (pre-2015) way. It has confusing scoping behavior (covered in Chapter 6). **Avoid it in new code**; you'll only see it in older tutorials and codebases.

Rule of thumb: **use `const` unless you know the value needs to change, then use `let`.**

### Naming rules and conventions

- Names can contain letters, digits, `_`, and `$`, but cannot *start* with a digit.
- Names are case-sensitive: `score` and `Score` are different variables.
- Convention: use **camelCase** (`firstName`, `totalPrice`) for variables and functions.
- Use descriptive names. `let d = 5` says nothing; `let daysUntilDeadline = 5` tells a story.

### The primitive data types

JavaScript has seven *primitive* types. The five you'll use constantly:

1. **`number`** — all numbers: integers and decimals alike. `42`, `3.14`, `-7`. JavaScript has only one number type (no separate "integer" and "float" types). Special values: `Infinity`, `-Infinity`, and `NaN` ("Not a Number" — the result of a failed numeric operation).
2. **`string`** — text, wrapped in quotes: `"hello"`, `'hello'`, or backticks `` `hello` ``. All three work; backticks unlock extra powers (Chapter 9).
3. **`boolean`** — exactly two values: `true` and `false`. Used for decisions.
4. **`undefined`** — the value of a variable that has been declared but not given a value. It means "nothing here *yet* — nobody set this."
5. **`null`** — an intentional "no value." It means "nothing here *on purpose* — someone set this to empty."

Two more you'll meet later: **`bigint`** (huge integers) and **`symbol`** (unique identifiers) — rare in everyday code.

Everything that isn't a primitive is an **object** (including arrays and functions) — Chapters 7 and 8 cover those in depth.

### `typeof` — asking a value its type

The `typeof` operator returns a string describing a value's type: `typeof 42` is `"number"`. One famous quirk: `typeof null` is `"object"` — a bug from 1995 that's kept for compatibility. Remember it; it shows up in interviews.

### Dynamic typing

JavaScript is **dynamically typed**: variables don't have fixed types — *values* do. The same variable can hold a number now and a string later (though doing so is usually a bad idea for readability).

### Type conversion (coercion)

Values can be converted between types:

- **Explicit conversion** — you ask for it: `Number("42")`, `String(42)`, `Boolean(1)`.
- **Implicit coercion** — JavaScript does it automatically, e.g. `"5" + 1` becomes `"51"` (number converted to string, then joined). Implicit coercion is a top source of beginner bugs.

**Truthy and falsy:** when JavaScript needs a boolean (e.g., in an `if`), every value converts to `true` or `false`. The **falsy** values are: `false`, `0`, `-0`, `""` (empty string), `null`, `undefined`, and `NaN`. *Everything else* is truthy — including `"0"`, `"false"`, and empty arrays `[]`.

## Code Examples

### 1. Declaring and using variables

```js
// const: for values that never get reassigned
const siteName = "My Learning Journal";

// let: for values that will change
let visitCount = 0;
visitCount = visitCount + 1; // reassignment is fine with let
console.log(siteName, "visits:", visitCount); // My Learning Journal visits: 1

// const prevents reassignment:
const pi = 3.14159;
// pi = 3; // ❌ TypeError: Assignment to constant variable.
```

### 2. The primitive types in action

```js
const price = 19.99;            // number
const productName = "Notebook"; // string
const inStock = true;           // boolean

let shippingDate;               // declared but not initialized
console.log(shippingDate);      // undefined

let discount = null;            // intentionally empty: "no discount"

console.log(typeof price);        // "number"
console.log(typeof productName);  // "string"
console.log(typeof inStock);      // "boolean"
console.log(typeof shippingDate); // "undefined"
console.log(typeof discount);     // "object"  <-- the famous quirk!
```

### 3. Numbers and their quirks

```js
console.log(10 / 3);      // 3.3333333333333335
console.log(0.1 + 0.2);   // 0.30000000000000004 (!!)
// Computers store decimals in binary; tiny rounding errors are normal.
// For display, round the result:
console.log((0.1 + 0.2).toFixed(2)); // "0.30" (a string!)

console.log(1 / 0);           // Infinity
console.log("abc" * 2);       // NaN — the math failed
console.log(NaN === NaN);     // false — NaN equals nothing, even itself
console.log(Number.isNaN(NaN)); // true — the right way to check for NaN
```

### 4. Explicit type conversion

```js
// String -> Number
const input = "42";               // form inputs always arrive as strings!
const asNumber = Number(input);   // 42
console.log(asNumber + 1);        // 43

console.log(Number("3.14"));      // 3.14
console.log(Number(""));          // 0    (surprising!)
console.log(Number("hello"));     // NaN

// parseInt / parseFloat read numbers off the front of a string
console.log(parseInt("42px", 10)); // 42
console.log(parseFloat("3.5kg"));  // 3.5

// Number -> String
console.log(String(99));          // "99"
console.log((99).toString());     // "99"

// Anything -> Boolean
console.log(Boolean(0));          // false
console.log(Boolean("hello"));    // true
console.log(Boolean(""));         // false
```

### 5. A realistic mini-program

```js
// A tiny checkout summary
const itemName = "Wireless Mouse";
const unitPrice = 24.5;      // dollars
let quantity = 3;

const subtotal = unitPrice * quantity;
const taxRate = 0.08;
const total = subtotal * (1 + taxRate);

console.log("Item:", itemName);
console.log("Subtotal:", subtotal.toFixed(2));  // "73.50"
console.log("Total with tax:", total.toFixed(2)); // "79.38"
```

## Common Pitfalls

### 1. Adding a string to a number

```js
const age = "30";        // came from a form or prompt — it's a string!
console.log(age + 5);    // ❌ "305" — string concatenation, not addition

const ageNum = Number(age);
console.log(ageNum + 5); // ✅ 35
```

Anything from user input (forms, `prompt()`, URL parameters) is a string. Convert before doing math.

### 2. Using a variable before declaring it

```js
console.log(score); // ❌ ReferenceError: Cannot access 'score' before initialization
let score = 10;
```

Declare variables before you use them, at the top of the block where they're needed.

### 3. Reassigning a `const` — or thinking `const` means "frozen"

```js
const limit = 100;
// limit = 200; // ❌ TypeError

// But note: const prevents REASSIGNMENT, not modification of object contents:
const settings = { theme: "dark" };
settings.theme = "light"; // ✅ allowed! The variable still points to the same object.
```

### 4. Confusing `null` and `undefined`

```js
let a;            // undefined — nobody assigned anything
let b = null;     // null — deliberately set to "empty"

console.log(a == b);  // true  (loose equality treats them as similar)
console.log(a === b); // false (they are different types)
```

Use `null` when *you* want to represent "no value." Expect `undefined` when something was never set.

### 5. Trusting truthiness too much

```js
let itemsInCart = 0;

if (itemsInCart) {
  console.log("You have items!");
} else {
  console.log("Cart is empty"); // ❌ runs even though 0 is a valid count... 
}
// The intent was "is this variable set?", but 0 is falsy.
// ✅ Be explicit about what you're checking:
if (itemsInCart !== undefined) {
  console.log("Cart count is known:", itemsInCart);
}
```

## Practice Exercises

1. **Profile card.** Declare variables for your name, age, favorite language (`"JavaScript"`, presumably), and whether you're currently employed (boolean). Pick `const` or `let` appropriately for each, and log them all with labels.

2. **Type detective.** Predict the output of `typeof` for each of these, *then* verify in the console: `42`, `"42"`, `true`, `undefined`, `null`, `NaN`, `3.14`, `""`. Note every prediction you got wrong.

3. **Converter.** Given `const rawInput = "  19.99 "` (note the spaces), convert it to a number and compute the price of 4 units. Then format the result as a string with exactly 2 decimal places.

4. **Truthy or falsy?** For each value — `0`, `"0"`, `""`, `" "`, `null`, `undefined`, `NaN`, `-1`, `"false"` — predict whether it's truthy or falsy, then verify each with `Boolean(value)` in the console.

5. **Swap.** Declare `let x = 5` and `let y = 10`. Write code that swaps their values (so `x` is 10 and `y` is 5) using a third temporary variable. Log the values before and after.
