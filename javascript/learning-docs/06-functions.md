# Chapter 6: Functions

## Overview

Functions are the most important building block in JavaScript. A **function** packages a piece of behavior — some inputs, some steps, an output — under a name, so you can run it whenever you want without rewriting it. Functions let you break big problems into small named pieces, avoid repetition, and build programs that read like a list of intentions rather than a wall of steps.

This chapter covers the three ways to define functions (declarations, expressions, arrow functions), parameters and return values, and **scope** — the rules governing which variables are visible where. Scope is also the foundation for closures (Chapter 13), so learn it well here.

## Definitions & Explanations

### Anatomy of a function

```js
function add(a, b) {   // "add" = name; a, b = parameters
  return a + b;        // return = the function's output
}

const result = add(2, 3); // calling (invoking) with arguments 2 and 3
```

- **Parameters** are the named placeholders in the definition (`a`, `b`).
- **Arguments** are the actual values you pass when calling (`2`, `3`).
- **`return`** immediately stops the function and hands back a value. A function with no `return` (or a bare `return;`) produces `undefined`.

### Three ways to define a function

**1. Function declaration**

```js
function greet(name) {
  return `Hello, ${name}!`;
}
```

Declarations are **hoisted**: JavaScript registers them before running the file, so you can call them *above* where they're written. Good default for standalone, named operations.

**2. Function expression**

```js
const greet = function (name) {
  return `Hello, ${name}!`;
};
```

Here the function is a *value* assigned to a variable. Not hoisted (you can't call it before the assignment line). This shows a deep truth: **functions are values** in JavaScript — they can be stored in variables, passed to other functions, and returned from functions (Chapter 13 builds on this heavily).

**3. Arrow function** (ES6)

```js
const greet = (name) => {
  return `Hello, ${name}!`;
};

// Short form: single expression? Drop the braces and `return`:
const greetShort = (name) => `Hello, ${name}!`;

// One parameter? Parentheses optional:
const double = n => n * 2;

// Zero parameters? Empty parentheses required:
const now = () => new Date();
```

Arrow functions are compact and are the standard style for short functions passed as arguments (e.g., to array methods in Chapter 7). They also handle the `this` keyword differently (relevant in Chapter 14) — for now, know they exist and practice the syntax.

### Default parameters

```js
function greet(name = "friend") {
  return `Hello, ${name}!`;
}
greet();        // "Hello, friend!"
greet("Rosa");  // "Hello, Rosa!"
```

Defaults apply when the argument is missing or `undefined`.

### Scope

**Scope** is the region of code where a variable is accessible.

- **Global scope** — declared outside any function/block; visible everywhere. Keep globals to a minimum.
- **Function scope** — parameters and variables declared inside a function exist only during that call.
- **Block scope** — `let` and `const` are confined to the nearest `{ }` block (including `if` blocks and loop bodies). This is one reason to avoid `var`, which ignores block boundaries and is only function-scoped.

Inner scopes can *read* outer variables; outer scopes can never see inner variables. When names collide, the innermost one wins (**shadowing**).

## Code Examples

### 1. From repetition to function

```js
// ❌ Repetitive:
console.log(`Total: $${(19.99 * 1.08).toFixed(2)}`);
console.log(`Total: $${(5.49 * 1.08).toFixed(2)}`);

// ✅ Extracted into a function:
function withTax(price, taxRate = 0.08) {
  return (price * (1 + taxRate)).toFixed(2);
}

console.log(`Total: $${withTax(19.99)}`);       // Total: $21.59
console.log(`Total: $${withTax(5.49)}`);        // Total: $5.93
console.log(`Total: $${withTax(100, 0.2)}`);    // custom tax rate: $120.00
```

### 2. Return early, return often

```js
function describeAge(age) {
  if (typeof age !== "number" || Number.isNaN(age)) {
    return "Please provide a number."; // guard clause — exits immediately
  }
  if (age < 0) return "Ages can't be negative.";
  if (age < 13) return "child";
  if (age < 20) return "teenager";
  return "adult";
}

console.log(describeAge(9));      // "child"
console.log(describeAge("nine")); // "Please provide a number."
```

### 3. The three definition styles side by side

```js
// Declaration — hoisted, good for "main" named operations
function area(width, height) {
  return width * height;
}

// Expression — a function stored in a variable
const perimeter = function (width, height) {
  return 2 * (width + height);
};

// Arrow — concise, great for short helpers
const diagonal = (w, h) => Math.sqrt(w * w + h * h);

console.log(area(3, 4));      // 12
console.log(perimeter(3, 4)); // 14
console.log(diagonal(3, 4));  // 5
```

### 4. Functions calling functions

```js
const clean = (text) => text.trim().toLowerCase();

const isValidUsername = (raw) => {
  const name = clean(raw);                 // reuse the helper
  return name.length >= 3 && !name.includes(" ");
};

function registrationMessage(raw) {
  return isValidUsername(raw)
    ? `Welcome, ${clean(raw)}!`
    : "Invalid username.";
}

console.log(registrationMessage("  Ada  "));   // "Welcome, ada!"
console.log(registrationMessage(" x "));        // "Invalid username."
```

### 5. Scope in action

```js
const appName = "TaskMaster";      // global — visible everywhere

function makeReport() {
  const generatedAt = new Date();  // function scope — only inside makeReport

  if (true) {
    const label = "REPORT";        // block scope — only inside this if-block
    console.log(label, appName);   // ✅ can read outer scopes
  }

  // console.log(label);           // ❌ ReferenceError — label is gone
  return `${appName} report at ${generatedAt.toLocaleTimeString()}`;
}

console.log(makeReport());
// console.log(generatedAt);       // ❌ ReferenceError — function scope
```

### 6. Shadowing

```js
let message = "outer";

function demo() {
  let message = "inner";   // a NEW variable that shadows the outer one
  console.log(message);    // "inner"
}

demo();
console.log(message);      // "outer" — unchanged
```

## Common Pitfalls

### 1. Forgetting to `return`

```js
// ❌ logs inside, returns undefined
function addBroken(a, b) {
  const sum = a + b;
  console.log(sum);      // printing is NOT returning!
}
const x = addBroken(2, 3); // prints 5, but...
console.log(x);            // undefined

// ✅
function add(a, b) {
  return a + b;
}
```

`console.log` shows a value to *you*; `return` hands the value to the *code that called the function*. They are completely different.

### 2. Calling vs. referencing

```js
const sayHi = () => "Hi!";

console.log(sayHi);    // ❌ prints the function itself: () => "Hi!"
console.log(sayHi());  // ✅ parentheses CALL it: "Hi!"
// Exception: when passing a function to another function (Chapter 13),
// you deliberately omit the parentheses.
```

### 3. Arrow function returning an object

```js
// ❌ The braces are read as a function body, not an object:
const makeUser = (name) => { name: name };   // returns undefined!

// ✅ Wrap the object in parentheses:
const makeUserFixed = (name) => ({ name: name });
console.log(makeUserFixed("Ada")); // { name: "Ada" }
```

### 4. Unreachable code after `return`

```js
function process(items) {
  return items.length;
  console.log("done");   // ❌ never runs — return already exited
}
```

### 5. Relying on hoisting with function expressions

```js
sayHello();                       // ✅ works — declarations are hoisted
function sayHello() { console.log("hello"); }

sayGoodbye();                     // ❌ ReferenceError
const sayGoodbye = () => console.log("bye");
// Expressions/arrows exist only after their line runs. Define before use.
```

## Practice Exercises

1. **Three ways.** Write a function `celsiusToFahrenheit(c)` (formula: `c * 9/5 + 32`) three times: as a declaration, as a function expression, and as a one-line arrow function. Verify all three give the same answers for 0, 100, and 37.

2. **Clamp.** Write `clamp(value, min, max)` that returns `value` limited to the range: below `min` returns `min`, above `max` returns `max`, otherwise `value` itself. Then give `min` and `max` defaults of 0 and 100.

3. **Password checker.** Write `checkPassword(pw)` returning a descriptive string. Use guard clauses: shorter than 8 characters → "too short"; no digit → "needs a number" (hint: loop over characters, or peek ahead at `/[0-9]/.test(pw)`); otherwise "ok". No nested ifs allowed.

4. **Scope prediction.** Without running it, predict every logged value; then verify.
   ```js
   let count = 1;
   function outer() {
     let count = 2;
     if (true) {
       let count = 3;
       console.log(count);
     }
     console.log(count);
   }
   outer();
   console.log(count);
   ```

5. **Compose a pipeline.** Write three small arrow functions: `trim(s)`, `capitalize(s)` (first letter uppercase), and `exclaim(s)` (appends `"!"`). Then write `shout(s)` that applies all three in order using function calls, and test it on `"  hello there  "`.
