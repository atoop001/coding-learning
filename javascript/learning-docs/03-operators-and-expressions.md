# Chapter 3: Operators & Expressions

## Overview

Operators are the verbs of programming: they add, compare, combine, and transform values. An **expression** is any piece of code that produces a value — `2 + 3`, `price * quantity`, `age >= 18`. Almost every line of JavaScript you write will contain expressions built from operators.

This chapter covers arithmetic, assignment, comparison, and logical operators — plus the crucial difference between `==` and `===`, which is one of the most common interview questions and beginner stumbling blocks in JavaScript.

## Definitions & Explanations

### Expressions vs. statements

- An **expression** produces a value: `5 * 2`, `"a" + "b"`, `x > 10`.
- A **statement** performs an action: `let x = 5;`, `if (...) {...}`.

Expressions can be nested inside statements. Understanding this distinction helps later when you learn that some constructs (like arrow functions and ternaries) are expressions and can go anywhere a value can.

### Arithmetic operators

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Addition | `5 + 2` | `7` |
| `-` | Subtraction | `5 - 2` | `3` |
| `*` | Multiplication | `5 * 2` | `10` |
| `/` | Division | `5 / 2` | `2.5` |
| `%` | Remainder (modulo) | `5 % 2` | `1` |
| `**` | Exponentiation | `5 ** 2` | `25` |

**Modulo (`%`)** deserves special attention: it returns the *remainder* after division. It's the standard tool for "is this number even?" (`n % 2 === 0`), for cycling through values, and for extracting digits.

**Increment/decrement:** `x++` adds 1 to `x`; `x--` subtracts 1. Prefer the clearer `x += 1` when starting out.

### Assignment operators

`=` assigns. Compound forms combine an operation with assignment:

```js
x += 5;  // x = x + 5
x -= 5;  // x = x - 5
x *= 2;  // x = x * 2
x /= 2;  // x = x / 2
x %= 3;  // x = x % 3
```

### Comparison operators

Comparisons produce booleans (`true`/`false`):

- `>` greater than, `<` less than, `>=` at least, `<=` at most
- `===` **strict equality** — equal value AND equal type
- `!==` **strict inequality**
- `==` **loose equality** — converts types before comparing (avoid!)
- `!=` **loose inequality** (avoid!)

**The golden rule: always use `===` and `!==`.** Loose equality performs implicit type coercion with rules almost nobody remembers correctly (`"" == 0` is `true`; `null == undefined` is `true`; `"1" == 1` is `true`). Strict equality has no surprises.

### Logical operators

- `&&` (AND) — `a && b` is true only if *both* are true.
- `||` (OR) — `a || b` is true if *at least one* is true.
- `!` (NOT) — flips a boolean: `!true` is `false`.

**Short-circuiting:** `&&` and `||` don't actually return `true`/`false` — they return one of their *operands*:

- `a && b`: if `a` is falsy, return `a` (stop early); otherwise return `b`.
- `a || b`: if `a` is truthy, return `a` (stop early); otherwise return `b`.

This enables common patterns like default values: `const name = input || "Anonymous";`

### Nullish coalescing: `??`

`a ?? b` returns `b` only when `a` is `null` or `undefined` — *not* for other falsy values like `0` or `""`. This is usually what you want for defaults:

```js
const count = savedCount ?? 10; // keeps 0 if savedCount is 0; uses 10 only if null/undefined
```

### The conditional (ternary) operator

`condition ? valueIfTrue : valueIfFalse` — an expression version of if/else, great for choosing between two values:

```js
const label = age >= 18 ? "adult" : "minor";
```

### Operator precedence

Just like math, operators have priority: `*` before `+`, comparisons before logical operators. `2 + 3 * 4` is `14`, not `20`. When in doubt, **use parentheses** — they cost nothing and make intent obvious.

## Code Examples

### 1. Arithmetic in practice

```js
const items = 7;
const perBox = 3;

console.log(items / perBox);            // 2.3333... boxes (fractional)
console.log(Math.floor(items / perBox)); // 2 full boxes
console.log(items % perBox);            // 1 item left over

// Is a number even?
const n = 42;
console.log(n % 2 === 0); // true — even

// Compound assignment
let score = 100;
score += 50;   // 150
score *= 2;    // 300
score -= 25;   // 275
console.log(score);
```

### 2. Strict vs loose equality

```js
console.log(5 === 5);      // true
console.log(5 === "5");    // false — different types
console.log(5 == "5");     // true  — loose equality coerced the string. Avoid!

console.log(0 == "");      // true  (?!) — why we avoid ==
console.log(0 === "");     // false — sane

console.log(null == undefined);  // true
console.log(null === undefined); // false

// The one legitimate use of ==: checking for "null or undefined" at once
let value = null;
console.log(value == null); // true if value is null OR undefined
// But being explicit is clearer for beginners:
console.log(value === null || value === undefined); // same result
```

### 3. Logical operators and short-circuiting

```js
const isLoggedIn = true;
const isAdmin = false;

console.log(isLoggedIn && isAdmin); // false — needs both
console.log(isLoggedIn || isAdmin); // true — needs one
console.log(!isLoggedIn);           // false

// Short-circuit values:
console.log("hello" && 42);   // 42      ("hello" is truthy, so return second operand)
console.log(0 && 42);         // 0       (0 is falsy, stop and return it)
console.log("" || "default"); // "default"
console.log("Sam" || "default"); // "Sam"

// || vs ?? for defaults:
const savedVolume = 0; // user turned the volume all the way down
console.log(savedVolume || 50);  // 50 ❌ — treats 0 as "missing"!
console.log(savedVolume ?? 50);  // 0  ✅ — only null/undefined trigger the default
```

### 4. Ternary expressions

```js
const stock = 3;

// Instead of a 4-line if/else just to pick a string:
const message = stock > 0 ? `${stock} in stock` : "Out of stock";
console.log(message); // "3 in stock"

// Ternaries can be used anywhere an expression fits:
console.log(`Shipping: ${stock > 0 ? "2-3 days" : "unavailable"}`);

// Don't nest ternaries deeply — it becomes unreadable. Use if/else instead.
```

### 5. A realistic pricing calculation

```js
// Compute an order total with a member discount and free-shipping rule
const unitPrice = 15;
const quantity = 4;
const isMember = true;

const subtotal = unitPrice * quantity;             // 60
const discount = isMember ? subtotal * 0.1 : 0;    // 6
const shipping = subtotal - discount >= 50 ? 0 : 5.99;
const total = subtotal - discount + shipping;

console.log(`Subtotal: $${subtotal.toFixed(2)}`);  // $60.00
console.log(`Discount: -$${discount.toFixed(2)}`); // -$6.00
console.log(`Shipping: $${shipping.toFixed(2)}`);  // $0.00
console.log(`Total:    $${total.toFixed(2)}`);     // $54.00
```

## Common Pitfalls

### 1. Using `=` when you mean `===`

```js
let role = "user";
if (role = "admin") {           // ❌ assignment! role is now "admin", and
  console.log("Access granted"); //    "admin" is truthy so this always runs
}

if (role === "admin") { /* ✅ comparison */ }
```

### 2. Relying on `==`

```js
const input = ""; // user left the field blank
if (input == 0) {
  console.log("You entered zero"); // ❌ runs! "" == 0 is true
}
if (input === 0) { /* ✅ never confuses "" with 0 */ }
```

### 3. `+` with mixed types

```js
console.log(1 + 2 + "3");  // "33" — 1+2 is 3, then 3 + "3" concatenates
console.log("1" + 2 + 3);  // "123" — left to right, string wins immediately
console.log("10" - 5);     // 5 — minus has no string meaning, so it converts!
// Lesson: + is the only arithmetic operator that concatenates. Convert first.
```

### 4. Floating point comparisons

```js
console.log(0.1 + 0.2 === 0.3); // ❌ false! (0.30000000000000004)

// ✅ compare with a tolerance:
console.log(Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON); // true
// Or work in integers (cents instead of dollars) for money.
```

### 5. Chaining comparisons like math class

```js
const age = 25;
// ❌ Looks right, is wrong:
// console.log(18 <= age <= 30); // true even for age = 100!
// (18 <= age) is true; true <= 30 becomes 1 <= 30 → true.

// ✅ combine two comparisons explicitly:
console.log(age >= 18 && age <= 30); // true
```

## Practice Exercises

1. **Predict then verify.** Without running them, write down the result of: `7 % 3`, `2 ** 3 ** 2`, `"5" - 2`, `"5" + 2`, `10 / 4`, `!!"hello"`. Then check each in the console and explain any misses.

2. **Even/odd/leap.** Given a variable `year`, write an expression that is `true` when the year is a leap year (divisible by 4, except centuries unless divisible by 400). Test with 2024, 1900, and 2000.

3. **Grade ternary.** Given `const score = 78`, use a single ternary to produce `"pass"` or `"fail"` (pass is 60+). Then extend it with the ternary inside a template literal: `` `You scored 78: pass` ``.

4. **Default settings.** You receive `let fontSize` that may be `undefined`, `0`, or a positive number. Using `??`, produce a final size that defaults to 16 only when the value is missing — verify that an explicit `0` is preserved. Then show what `||` would have done differently.

5. **Cart rules.** Given `subtotal`, `isMember` (boolean), and `couponCode` (string, possibly empty), compute: 10% discount if member, an extra $5 off if `couponCode` is exactly `"SAVE5"`, free shipping when the discounted subtotal is at least $35 (otherwise $4.99). Log a formatted summary. Use only operators from this chapter — no `if` statements.
