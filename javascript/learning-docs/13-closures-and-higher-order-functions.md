# Chapter 13: Closures & Higher-Order Functions

## Overview

This chapter is where JavaScript gets *deep* — and where interviews get serious. Two related superpowers flow from the fact that **functions are values**:

1. **Higher-order functions** — functions that take other functions as arguments, or return new functions. You've already used them (`map`, `filter`, `addEventListener`); now you'll write your own.
2. **Closures** — a function "remembers" the variables from the scope where it was created, even after that scope has finished. Closures power private state, function factories, callbacks, and most advanced patterns you'll meet.

If Chapter 6's scope rules felt abstract, this is where they pay off.

## Definitions & Explanations

### Functions as values, one more time

A function in JavaScript is an object you can:

- store in a variable or array,
- pass as an argument to another function,
- return from a function.

A function passed as an argument is called a **callback** ("call me back when you're ready"). You already pass callbacks to `map`, `filter`, `forEach`, and `addEventListener`.

### Higher-order functions (HOFs)

A **higher-order function** is any function that *takes* a function and/or *returns* a function. Why bother?

- **Behavior injection**: `map` knows how to loop; *you* supply what to do with each element. The looping logic is written once, the per-element behavior varies.
- **Customization**: a function that returns a function lets you "pre-configure" behavior (`makeGreeter("Hello")` → a greeting function).

### Closures

Formal definition: **a closure is a function together with the variables from its surrounding scope that it references**. Practically:

```js
function makeCounter() {
  let count = 0;                // local to makeCounter...
  return function () {
    count++;                    // ...but the inner function still uses it
    return count;
  };
}

const counter = makeCounter();  // makeCounter has RETURNED, its call is over...
counter();                      // 1  ...yet count lives on!
counter();                      // 2
```

Normally a function's local variables die when the call ends. But if an inner function still references them, JavaScript **keeps them alive** for that inner function. The inner function "closes over" `count` — hence *closure*.

Key facts:

- Each call to the outer function creates a **fresh** set of variables. `makeCounter()` twice → two independent counters.
- The closure captures the **variable itself**, not a snapshot of its value — if the variable changes later, the closure sees the change.
- Closures are the closest thing pre-class JavaScript had to **private state**: nothing outside `makeCounter` can touch `count` except through the returned function.

### Where you're already using closures

- Every event handler that references variables outside itself.
- Every `map`/`filter` callback using a variable from enclosing code.
- Debounce/throttle utilities, memoization, module patterns — all closures.

## Code Examples

### 1. Writing your own higher-order function

```js
// A HOF that takes a callback: run any function n times
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);                  // call the callback, passing the index
  }
}

repeat(3, (i) => console.log(`Run #${i + 1}`));
// Run #1
// Run #2
// Run #3

// Same machinery, different behavior — that's the point of HOFs:
let total = 0;
repeat(5, (i) => { total += i; });
console.log(total); // 0+1+2+3+4 = 10
```

### 2. Reimplementing `map` (a classic interview exercise)

```js
function myMap(array, transform) {
  const result = [];
  for (const item of array) {
    result.push(transform(item));
  }
  return result;
}

console.log(myMap([1, 2, 3], (n) => n * 10));      // [10, 20, 30]
console.log(myMap(["a", "b"], (s) => s.toUpperCase())); // ["A", "B"]
// Once you can write this, map/filter/reduce stop being magic.
```

### 3. Functions returning functions (factories)

```js
// A factory: pre-configure behavior, get back a specialized function
function makeMultiplier(factor) {
  return (n) => n * factor;     // closes over `factor`
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

console.log(double(7));  // 14
console.log(triple(7));  // 21
console.log(makeMultiplier(100)(5)); // 500 — call the returned function immediately

// Real-world flavor: tax calculators per region
const withUkVat = makeMultiplier(1.2);
const withDeVat = makeMultiplier(1.19);
console.log(withUkVat(100)); // 120
```

### 4. Closures for private state

```js
function createBankAccount(initialBalance) {
  let balance = initialBalance;      // PRIVATE — unreachable from outside

  return {
    deposit(amount) {
      if (amount <= 0) throw new Error("Deposit must be positive");
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) throw new Error("Insufficient funds");
      balance -= amount;
      return balance;
    },
    getBalance() {
      return balance;
    },
  };
}

const account = createBankAccount(100);
account.deposit(50);
console.log(account.getBalance()); // 150
console.log(account.balance);      // undefined — there is NO direct access!
// The only doors into `balance` are the three methods. That's encapsulation.
```

### 5. Independent closures

```js
function makeIdGenerator(prefix) {
  let nextId = 1;
  return () => `${prefix}-${nextId++}`;
}

const userIds = makeIdGenerator("user");
const orderIds = makeIdGenerator("order");

console.log(userIds());  // "user-1"
console.log(userIds());  // "user-2"
console.log(orderIds()); // "order-1"  — its own separate nextId
console.log(userIds());  // "user-3"  — unaffected by orderIds
```

### 6. A practical HOF: `once`

```js
// Ensures a function can only ever run one time (e.g., app initialization)
function once(fn) {
  let called = false;
  let result;
  return (...args) => {           // ...args gathers any arguments into an array
    if (!called) {
      called = true;
      result = fn(...args);       // ...spread passes them along
    }
    return result;                // later calls return the first result
  };
}

const initialize = once(() => {
  console.log("Setting up app (expensive)!");
  return "ready";
});

initialize(); // "Setting up app (expensive)!" → "ready"
initialize(); // (silent) → "ready"
initialize(); // (silent) → "ready"
```

### 7. Closures in event handlers (you've been doing this!)

```js
function setupButton(name) {
  const btn = document.createElement("button");
  btn.textContent = name;
  let clicks = 0;                                  // per-button private counter

  btn.addEventListener("click", () => {
    clicks++;                                      // closure over `clicks` AND `name`
    console.log(`${name} clicked ${clicks} times`);
  });

  document.body.append(btn);
}

setupButton("Alpha");
setupButton("Beta");   // each button remembers ITS OWN name and count
```

## Common Pitfalls

### 1. Calling the callback too early

```js
repeat(3, console.log("hi"));   // ❌ logs "hi" once NOW, passes undefined
repeat(3, () => console.log("hi")); // ✅ passes a function to call later
```

### 2. Expecting a closure to snapshot a value

```js
let discount = 0.1;
const applyDiscount = (price) => price * (1 - discount);

discount = 0.5;                    // changed AFTER the function was created
console.log(applyDiscount(100));   // 50, not 90! Closures see the LIVE variable.
// If you need a snapshot, copy it: 
const makeDiscounter = (rate) => (price) => price * (1 - rate); // rate is fixed per call
```

### 3. The classic loop/`var` trap

```js
// ❌ With var, all three callbacks share ONE i (var ignores block scope):
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);   // 3, 3, 3
}

// ✅ let creates a fresh i per iteration — each closure captures its own:
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);   // 0, 1, 2
}
// This single example is why modern JS uses let, and a very common interview question.
```

### 4. Thinking two factory calls share state

```js
const c1 = makeCounter();
const c2 = makeCounter();
c1(); c1();
console.log(c2());   // 1, not 3 — every makeCounter() call creates fresh state.
// (This is a feature; just don't be surprised by it.)
```

### 5. Accidental shared state through an outer variable

```js
// ❌ Both handlers close over the SAME `selected` — is that intended?
let selected = null;
buttonA.addEventListener("click", () => { selected = "A"; });
buttonB.addEventListener("click", () => { selected = "B"; });
// Sometimes shared state is exactly what you want (app state!) —
// the pitfall is not REALIZING it's shared. Be deliberate.
```

## Practice Exercises

1. **`myFilter`.** Implement `myFilter(array, test)` from scratch with a loop (no built-in `filter`). It must return a new array of elements for which `test(element)` is truthy. Verify against the built-in on two examples.

2. **Greeting factory.** Write `makeGreeter(greeting)` returning a function that takes a name: `const hi = makeGreeter("Hi"); hi("Ada")` → `"Hi, Ada!"`. Create two differently-configured greeters and show they don't interfere.

3. **Secret counter.** Build `createScoreboard()` that returns `{ addPoint, subtractPoint, getScore, reset }` with the score held privately in a closure. Prove there is no way to set the score to 1000 directly from outside.

4. **Rate limiter.** Write `atMost(n, fn)` that returns a wrapped version of `fn` which runs normally the first `n` times and after that just returns the string `"limit reached"` without calling `fn`. Test with a function that logs.

5. **Trace the closure.** Without running it, predict the full output, then verify and explain each line:
   ```js
   function outer() {
     let x = 10;
     const inner = () => { x += 5; return x; };
     x = 20;
     return inner;
   }
   const f = outer();
   const g = outer();
   console.log(f());
   console.log(f());
   console.log(g());
   ```
