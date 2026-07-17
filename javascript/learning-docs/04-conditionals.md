# Chapter 4: Conditionals

## Overview

Programs become useful when they make decisions: show the dashboard *if* the user is logged in; apply a discount *if* the cart is big enough; display an error *if* the input is invalid. **Conditionals** are the structures that let code choose between paths.

This chapter covers `if`/`else if`/`else`, the `switch` statement, guard clauses, and how to structure branching logic so it stays readable as it grows. Everything builds directly on the comparison and logical operators from Chapter 3.

## Definitions & Explanations

### The `if` statement

```js
if (condition) {
  // runs only when condition is truthy
}
```

The condition is any expression. JavaScript converts its result to a boolean (truthy/falsy rules from Chapter 2). The braces `{}` mark a **block** — a group of statements treated as a unit.

### `else` and `else if`

```js
if (conditionA) {
  // A was truthy
} else if (conditionB) {
  // A was falsy, B was truthy
} else {
  // neither was truthy
}
```

Conditions are checked **top to bottom**, and only the *first* matching branch runs. Order matters: put the most specific conditions first.

### The `switch` statement

When you're comparing one value against many exact options, `switch` can be cleaner than a long `else if` chain:

```js
switch (value) {
  case option1:
    // runs if value === option1
    break;
  case option2:
    // runs if value === option2
    break;
  default:
    // runs if nothing matched
}
```

Key facts:

- Comparison uses **strict equality** (`===`).
- **`break` is required** to stop after a match — without it, execution "falls through" into the next case (occasionally useful, usually a bug).
- `default` handles "none of the above" (like `else`).

### Guard clauses

A **guard clause** handles the "bad" or "edge" case early and exits, so the main logic isn't buried in nesting:

```js
function processOrder(order) {
  if (!order) return "No order provided";     // guard
  if (order.items === 0) return "Cart empty"; // guard
  // main logic here, at the top level, unindented
}
```

(You'll formally learn functions in Chapter 6; for now, read `return` as "stop and hand back this value.")

### Nesting and its limits

Conditionals can be nested inside conditionals, but deep nesting (3+ levels) becomes hard to read. Tools to flatten it: combine conditions with `&&`/`||`, use guard clauses, or extract logic into functions.

## Code Examples

### 1. Basic branching

```js
const temperature = 31;

if (temperature > 30) {
  console.log("It's hot — drink water!");
} else if (temperature > 15) {
  console.log("Nice weather.");
} else if (temperature > 0) {
  console.log("Chilly — grab a jacket.");
} else {
  console.log("Freezing!");
}
// Prints: "It's hot — drink water!" and nothing else.
```

### 2. Combining conditions

```js
const age = 22;
const hasTicket = true;
const isVip = false;

// AND: both must hold
if (age >= 18 && hasTicket) {
  console.log("Welcome to the show.");
}

// OR: either is enough
if (hasTicket || isVip) {
  console.log("You may enter the lobby.");
}

// NOT + grouping: parentheses make intent explicit
if (!(age >= 18)) {
  console.log("Sorry, adults only.");
}
```

### 3. `switch` for exact matches

```js
const day = "saturday";

switch (day) {
  case "saturday":
  case "sunday":               // two cases sharing one body (intentional fall-through)
    console.log("Weekend! 🎉");
    break;
  case "friday":
    console.log("Almost there...");
    break;
  default:
    console.log("Regular workday.");
}
// Prints: "Weekend! 🎉"
```

### 4. Guard clauses vs. nested ifs

```js
// ❌ Deeply nested — hard to follow
function canRentCarNested(age, hasLicense, hasInsurance) {
  if (age >= 21) {
    if (hasLicense) {
      if (hasInsurance) {
        return "Approved";
      } else {
        return "Need insurance";
      }
    } else {
      return "Need a license";
    }
  } else {
    return "Too young";
  }
}

// ✅ Guard clauses — each requirement checked and dismissed in turn
function canRentCar(age, hasLicense, hasInsurance) {
  if (age < 21) return "Too young";
  if (!hasLicense) return "Need a license";
  if (!hasInsurance) return "Need insurance";
  return "Approved";
}

console.log(canRentCar(25, true, false)); // "Need insurance"
```

### 5. A realistic example: form validation logic

```js
const username = "jo";
const password = "hunter2";
const passwordConfirm = "hunter2";

let error = null; // null means "no error so far"

if (username.length < 3) {
  error = "Username must be at least 3 characters.";
} else if (password.length < 8) {
  error = "Password must be at least 8 characters.";
} else if (password !== passwordConfirm) {
  error = "Passwords do not match.";
}

if (error) {
  console.log("❌ " + error);
} else {
  console.log("✅ Form is valid!");
}
// Prints: ❌ Username must be at least 3 characters.
```

### 6. Choosing values vs. choosing actions

```js
const hour = 14;

// When each branch just picks a VALUE, a ternary or lookup is tidier:
const greeting = hour < 12 ? "Good morning" : hour < 18 ? "Good afternoon" : "Good evening";
console.log(greeting); // "Good afternoon"

// When branches perform different ACTIONS, use if/else:
if (hour < 9) {
  console.log("Office closed — opening at 9.");
} else {
  console.log("Office open.");
}
```

## Common Pitfalls

### 1. Forgetting `break` in `switch`

```js
const fruit = "apple";
switch (fruit) {
  case "apple":
    console.log("Apples are red or green");
  case "banana":                              // ❌ no break above — this runs too!
    console.log("Bananas are yellow");
    break;
}
// Prints BOTH lines. Add `break` after each case body unless
// you are deliberately sharing a body between cases.
```

### 2. Wrong branch order

```js
const score = 95;
// ❌ The first (broadest) condition swallows everything:
if (score > 50) {
  console.log("Pass");
} else if (score > 90) {
  console.log("Excellent!");   // unreachable — never runs!
}

// ✅ Most specific first:
if (score > 90) {
  console.log("Excellent!");
} else if (score > 50) {
  console.log("Pass");
}
```

### 3. Comparing a variable to multiple values incorrectly

```js
const color = "red";
// ❌ This is ALWAYS true: ("blue" is truthy on its own)
if (color === "red" || "blue") { /* ... */ }

// ✅ Each side of || must be a complete comparison:
if (color === "red" || color === "blue") { /* ... */ }

// ✅ Or, with a preview of arrays (Chapter 7):
if (["red", "blue"].includes(color)) { /* ... */ }
```

### 4. Accidental semicolon after `if`

```js
const loggedIn = false;
if (loggedIn);                  // ❌ this semicolon ENDS the if!
{
  console.log("Welcome back!"); // runs unconditionally
}
// ✅ No semicolon between the condition and the block.
```

### 5. Redundant boolean comparisons

```js
if (isActive === true) { /* works, but noisy */ }
if (isActive) { /* ✅ same thing, idiomatic */ }

if (isActive === false) { /* noisy */ }
if (!isActive) { /* ✅ */ }

// Also: never write  return condition ? true : false;
// just write         return condition;
```

## Practice Exercises

1. **Movie ratings.** Given `const rating = "PG-13"` and `const age = 11`, write conditionals that print whether the viewer can watch: `"G"` — anyone; `"PG"` — anyone, but note "parental guidance suggested" under 10; `"PG-13"` — 13+; `"R"` — 17+. Test with several rating/age combinations.

2. **FizzBuzz decision.** Given a number `n`, print `"FizzBuzz"` if divisible by both 3 and 5, `"Fizz"` if only by 3, `"Buzz"` if only by 5, otherwise the number itself. (Think carefully about branch order!) Test with 9, 10, 15, and 7.

3. **Switch the calculator.** Given `const a = 8`, `const b = 3`, and `const op = "*"`, use a `switch` on `op` to compute and print the result for `"+"`, `"-"`, `"*"`, `"/"`, with a `default` that prints "Unknown operator". Make division print a special message when `b` is 0.

4. **Refactor to guards.** Take this description and implement it *first* with nested ifs, *then* refactored with guard clauses: a gym checks in a visitor only if they have a membership, the membership is not expired, and the gym is not at capacity. Each failure has its own message.

5. **Ticket pricer.** Prices: children under 5 free; ages 5–12 pay $8; 13–64 pay $15; 65+ pay $10; and on Wednesdays everyone gets 25% off. Given `age` and `dayOfWeek`, compute and print the final price. Decide yourself where `if/else` fits best and where a ternary is cleaner.
