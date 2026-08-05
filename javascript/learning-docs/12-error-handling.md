# Chapter 12: Error Handling

## Overview

Things go wrong. Users type garbage, networks fail, data arrives in unexpected shapes, and — let's be honest — our own code has bugs. **Error handling** is how a program deals with failure gracefully instead of crashing: catching problems, reporting them clearly, and recovering when possible.

JavaScript's tools for this are `try`/`catch`/`finally`, the `throw` statement, the built-in `Error` types, and custom error classes. Solid error handling separates hobby scripts from professional code — and it becomes *essential* once you start fetching data from APIs (Chapter 16).

This chapter also introduces **breakpoint debugging** — a more powerful alternative to `console.log` for tracking down *why* something went wrong in the first place.

## Definitions & Explanations

### What is an exception?

When something impossible happens — calling a method on `undefined`, referencing a variable that doesn't exist — JavaScript **throws an exception**. Unless something catches it, the exception crashes the current operation: the script stops (in the browser, the rest of that event handler dies; in Node, the program may exit).

### `try...catch`

```js
try {
  // code that MIGHT throw
} catch (error) {
  // runs ONLY if something in try threw
  // `error` is the thrown value — usually an Error object
}
```

- If nothing throws, `catch` is skipped entirely.
- If a statement in `try` throws, execution jumps *immediately* to `catch` — remaining `try` lines are skipped.
- After `catch` finishes, the program **continues normally**. The crash was contained.

### `finally`

```js
try { ... } catch (e) { ... } finally {
  // runs ALWAYS: after success, after a caught error,
  // even if try/catch contains a return
}
```

Use `finally` for cleanup that must happen no matter what: hiding a loading spinner, closing a resource, re-enabling a button.

### `throw` — raising your own errors

You aren't limited to the errors JavaScript produces. When *your* function receives bad input or reaches an invalid state, throw your own:

```js
throw new Error("Withdrawal amount must be positive");
```

You *can* throw any value (`throw "oops"`), but **always throw `Error` objects** — they carry a `message` and a **stack trace** (the file/line trail showing where the error originated), which strings do not.

### Built-in error types

- **`Error`** — the general-purpose base type.
- **`TypeError`** — a value had the wrong type ("x is not a function", "cannot read properties of undefined").
- **`RangeError`** — a value was out of range.
- **`ReferenceError`** — used a variable that doesn't exist.
- **`SyntaxError`** — invalid code (thrown at parse time — mostly not catchable) or invalid JSON via `JSON.parse`.

An error object has `.name` (`"TypeError"`), `.message`, and `.stack`.

### Custom error classes

For your own application's failure categories, extend `Error`:

```js
class ValidationError extends Error {
  constructor(message) {
    super(message);              // pass message to Error
    this.name = "ValidationError";
  }
}
```

Then callers can distinguish error kinds with `instanceof` and react differently to a validation problem vs. an unexpected bug. (Classes get full treatment in Chapter 14 — for now, follow the recipe.)

### Errors as API design: throw vs. return

Two respectable strategies for a function that can fail:

1. **Throw** — for *exceptional* situations the caller likely can't anticipate. Forces callers to handle it or crash.
2. **Return a marker** — `null`, `undefined`, `-1`, or a result object like `{ ok: false, error: "..." }` — for *expected*, routine failures (e.g., "search found nothing").

Rule of thumb: expected outcomes → return values; broken contracts and impossible states → throw.

### Debugging with breakpoints

`console.log` is great for a quick check, but it means editing code, guessing in advance what to print, and re-running. **Breakpoints** let you pause a running program at an exact line and inspect *everything* in scope — no code changes required.

- **Setting a breakpoint**: in DevTools (Chapter 1), open the **Sources** tab, find your file, and click the line number. Execution pauses there the next time that line runs.
- **Stepping**: **Step over** runs the current line without diving into function calls it makes; **Step into** follows the call into the function; **Step out** finishes the current function and pauses back in its caller.
- **Watch expressions**: pin any expression (`user.name`, `total > 100`) in the Watch panel — DevTools re-evaluates it every time execution pauses, so you don't have to retype it.
- **The `debugger;` statement**: write `debugger;` directly in your code. If DevTools is open, execution pauses there exactly like a manual breakpoint — handy for pausing deep inside logic you can't easily click a line number for (e.g., inside a loop, on a specific iteration). Remove it before shipping.
- **Call stack**: while paused, the Call Stack panel lists the chain of function calls that led here — invaluable for "how did execution even reach this line?"

**When breakpoints beat `console.log`**: bugs that depend on the *full state* at one moment (many variables, not just one you thought to log), bugs inside loops or recursive calls where you'd need dozens of log lines, and situations where you don't yet know what value is even worth printing. `console.log` still wins for quick one-off checks and for watching a value change across many calls over time — a timeline breakpoints don't give you as naturally.

## Code Examples

### 1. Catching a crash

```js
function parseUserJson(json) {
  try {
    const user = JSON.parse(json);      // throws SyntaxError on invalid JSON
    return user.name.toUpperCase();     // throws TypeError if name is missing
  } catch (error) {
    console.log("Could not process user:", error.message);
    return "UNKNOWN";
  }
}

console.log(parseUserJson('{"name": "Ada"}'));   // "ADA"
console.log(parseUserJson("not json at all"));   // "UNKNOWN" (SyntaxError caught)
console.log(parseUserJson('{"age": 30}'));       // "UNKNOWN" (TypeError caught)
// Note: the program keeps running after each failure. That's the point.
```

### 2. Throwing on bad input

```js
function divide(a, b) {
  if (typeof a !== "number" || typeof b !== "number") {
    throw new TypeError("divide() needs two numbers");
  }
  if (b === 0) {
    throw new RangeError("Cannot divide by zero");
  }
  return a / b;
}

try {
  console.log(divide(10, 2));   // 5
  console.log(divide(10, 0));   // throws — next line never runs
  console.log("unreachable");
} catch (err) {
  console.log(`${err.name}: ${err.message}`); // "RangeError: Cannot divide by zero"
}
```

### 3. `finally` for guaranteed cleanup

```js
function saveData(data) {
  const button = document.querySelector("#save-btn");
  button.disabled = true;                 // prevent double-clicks
  button.textContent = "Saving...";

  try {
    if (!data || data.length === 0) {
      throw new Error("Nothing to save");
    }
    console.log("Saved:", data);
    return "ok";
  } catch (err) {
    console.log("Save failed:", err.message);
    return "failed";
  } finally {
    // Runs in EVERY case — success, failure, even after those returns:
    button.disabled = false;
    button.textContent = "Save";
  }
}
```

### 4. Custom error classes and selective handling

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;                  // extra context is free to add!
  }
}

function validateSignup(form) {
  if (form.username.length < 3) {
    throw new ValidationError("Username too short", "username");
  }
  if (!form.email.includes("@")) {
    throw new ValidationError("Invalid email", "email");
  }
  return true;
}

try {
  validateSignup({ username: "jo", email: "jo@site.com" });
} catch (err) {
  if (err instanceof ValidationError) {
    // Expected kind of failure — show it to the user nicely
    console.log(`Please fix the ${err.field} field: ${err.message}`);
  } else {
    // Unexpected bug — don't hide it! Re-throw for visibility.
    throw err;
  }
}
```

### 5. Result objects — the non-throwing style

```js
function safeParseNumber(input) {
  const n = Number(input);
  if (Number.isNaN(n)) {
    return { ok: false, error: `"${input}" is not a number` };
  }
  return { ok: true, value: n };
}

const result = safeParseNumber("42px");
if (result.ok) {
  console.log("Double is", result.value * 2);
} else {
  console.log("Input problem:", result.error);   // no try/catch needed
}
```

### 6. Realistic: robust user input in an app

```js
function getQuantity(rawInput) {
  const qty = Number(rawInput);
  if (Number.isNaN(qty)) throw new ValidationError("Quantity must be a number", "qty");
  if (!Number.isInteger(qty)) throw new ValidationError("Quantity must be whole", "qty");
  if (qty < 1 || qty > 99) throw new ValidationError("Quantity must be 1–99", "qty");
  return qty;
}

function handleAddToCart(rawInput) {
  try {
    const qty = getQuantity(rawInput);
    console.log(`Added ${qty} to cart ✅`);
  } catch (err) {
    if (err instanceof ValidationError) {
      console.log(`⚠️ ${err.message}`);      // friendly message, app keeps running
    } else {
      console.error("Unexpected error:", err); // log real bugs loudly
      throw err;
    }
  }
}

handleAddToCart("3");     // Added 3 to cart ✅
handleAddToCart("3.5");   // ⚠️ Quantity must be whole
handleAddToCart("lots");  // ⚠️ Quantity must be a number
```

## Common Pitfalls

### 1. Swallowing errors silently

```js
try {
  riskyOperation();
} catch (e) {}          // ❌ empty catch: failures vanish without a trace

// ✅ At MINIMUM log it; ideally handle it or re-throw:
try {
  riskyOperation();
} catch (e) {
  console.error("riskyOperation failed:", e);
}
```

Silent catches produce the worst bugs: things "just don't work" with no clue why.

### 2. Wrapping everything in one giant try

```js
// ❌ Which of these 30 lines failed? Who knows.
try { /* entire program */ } catch (e) { console.log("something broke"); }
```

Keep `try` blocks *small and targeted* around the specific operations that can throw, so the catch knows what it's handling.

### 3. Throwing strings

```js
throw "something went wrong";        // ❌ no stack trace, no .name, breaks instanceof
throw new Error("something went wrong"); // ✅
```

### 4. Using exceptions for normal control flow

```js
// ❌ "Not found" is a normal outcome, not an emergency:
function findUser(list, id) {
  const u = list.find((x) => x.id === id);
  if (!u) throw new Error("not found");
  return u;
}
// ✅ Return undefined/null for routine misses; let the caller decide:
function findUserBetter(list, id) {
  return list.find((x) => x.id === id) ?? null;
}
```

### 5. Expecting `try/catch` to catch errors from *later* (async) code

```js
try {
  setTimeout(() => {
    throw new Error("boom");   // ❌ NOT caught — it throws after try/catch finished
  }, 1000);
} catch (e) {
  console.log("never runs");
}
```

`try/catch` only guards code that throws *while the try block runs*. Handling asynchronous failures needs promises and `async/await` — Chapter 15 covers it. Remember this pitfall; it will make Chapter 15 click.

## Practice Exercises

1. **Safe JSON.** Write `tryParse(json)` that returns the parsed object on success, and `null` on invalid JSON — using try/catch internally so callers never crash. Test with valid JSON, `"{oops"`, and `""`.

2. **Strict calculator.** Write `calculate(a, op, b)` supporting `+ - * /`. Throw a `TypeError` for non-number operands, a `RangeError` for division by zero, and a plain `Error` with message `"Unknown operator: X"` otherwise. Write a wrapper that calls it inside try/catch and prints either `"Result: N"` or `"[ErrorName] message"` for each of five test cases you design.

3. **Custom errors.** Create `EmptyCartError` and `PaymentDeclinedError` classes extending `Error`. Write `checkout(cart, cardValid)` that throws the first when the cart array is empty and the second when `cardValid` is false. In the caller, use `instanceof` to show a different friendly message for each, and re-throw anything unrecognized.

4. **Cleanup guarantee.** Write a function that simulates a "loading" state: log `"loading started"`, then run a callback that may or may not throw, and — using `finally` — always log `"loading finished"` afterwards. Demonstrate both the success path and the failure path.

5. **Validator suite.** Write `validateProfile(profile)` that checks: `name` is a non-empty string, `age` is a number 13–120, `email` contains `"@"`. Instead of stopping at the first problem, collect *all* failures into an array; if any exist, throw one `ValidationError` whose message joins them with `"; "`. Show a caller printing the combined message.

6. **Breakpoint hunt.** Take your `getQuantity`/`handleAddToCart` functions (or the Validator suite above) and deliberately introduce a subtle bug — e.g., an off-by-one in the range check. Instead of adding `console.log` calls, set a breakpoint on the first line of the function in DevTools Sources, reload, and use step over/into plus a watch expression on the input variable to find the bug. Then add a `debugger;` statement inside the loop of a function that processes an array, and step through one iteration at a time.
