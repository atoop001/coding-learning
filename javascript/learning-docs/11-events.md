# Chapter 11: Events

## Overview

The DOM lets you change the page; **events** let the page respond to the user. A click, a keypress, typing into a field, submitting a form, scrolling — each of these fires an *event*, and JavaScript can listen for them and run code in response. Events are what make a page *interactive*, and event-driven thinking ("when X happens, do Y") is the mental model behind all UI programming.

This chapter covers `addEventListener`, the event object, the most useful event types, form handling with `preventDefault`, and **event delegation** — the professional pattern for handling events on dynamic lists.

## Definitions & Explanations

### Listeners

An **event listener** is a function you register to run when a specific event happens on a specific element:

```js
element.addEventListener("click", (event) => {
  // runs every time element is clicked
});
```

- First argument: the **event type** as a string (`"click"`, `"input"`, `"submit"`, ...).
- Second argument: the **handler** — a function (often an arrow function). You pass the function itself — *no parentheses* — because the browser will call it later.
- You can attach multiple listeners to one element, and the same handler to many elements.
- `element.removeEventListener(type, handler)` unregisters — but only if you pass the *same function reference*, which is why named functions matter for removal.

### The event object

The browser calls your handler with an **event object** describing what happened. Useful properties:

- `event.target` — the element where the event actually occurred (e.g., the exact button clicked).
- `event.currentTarget` — the element the listener is attached to (these differ during delegation!).
- `event.key` — for keyboard events, which key (`"Enter"`, `"a"`, `"Escape"`).
- `event.preventDefault()` — cancels the browser's default action (following a link, submitting/reloading a form).

### Event types you'll actually use

| Event | Fires when |
|---|---|
| `click` | element is clicked (works on almost anything) |
| `input` | a form field's value changes — every keystroke |
| `change` | a field's value is committed (checkbox toggled, select chosen, input blurred) |
| `submit` | a `<form>` is submitted (fires on the **form**, not the button) |
| `keydown` | a key is pressed (check `event.key`) |
| `mouseover` / `mouseout` | pointer enters / leaves an element |
| `DOMContentLoaded` | on `document`: the HTML is fully parsed |

### Bubbling and delegation

When you click a `<button>` inside a `<li>` inside a `<ul>`, the click event **bubbles**: it fires on the button, then the li, then the ul, then up to `document`. This enables **event delegation**: instead of attaching a listener to every item in a list (including items created later!), attach **one** listener to the parent and use `event.target` to figure out which child was involved. Delegation is *the* answer to "why doesn't my listener work on elements I added afterwards?"

### Forms and `preventDefault`

By default, submitting a form reloads the page — a leftover from pre-JavaScript days. For interactive apps you almost always want:

```js
form.addEventListener("submit", (event) => {
  event.preventDefault(); // stop the reload
  // read fields, validate, update the page
});
```

Listen for `submit` on the form (not `click` on the button) — it also catches the user pressing Enter.

## Code Examples

Assume this HTML (script loaded with `defer`):

```html
<button id="counter-btn">Clicked 0 times</button>

<form id="signup">
  <input id="username" placeholder="Username" />
  <button type="submit">Sign up</button>
</form>
<p id="form-msg"></p>

<input id="search" placeholder="Type to search..." />
<p id="search-echo"></p>

<ul id="todo-list">
  <li>Learn events <button class="delete">✕</button></li>
  <li>Practice delegation <button class="delete">✕</button></li>
</ul>
```

### 1. Click counter — state + event + render

```js
const btn = document.querySelector("#counter-btn");
let count = 0;                        // state lives in JS, not in the DOM

btn.addEventListener("click", () => {
  count++;
  btn.textContent = `Clicked ${count} time${count === 1 ? "" : "s"}`;
});
```

### 2. Live input echo

```js
const search = document.querySelector("#search");
const echo = document.querySelector("#search-echo");

search.addEventListener("input", (event) => {
  // event.target is the input; .value is its current text
  echo.textContent = event.target.value === ""
    ? ""
    : `Searching for: ${event.target.value}`;
});
```

### 3. Keyboard events

```js
search.addEventListener("keydown", (event) => {
  if (event.key === "Enter") {
    console.log("Search submitted:", search.value);
  }
  if (event.key === "Escape") {
    search.value = "";               // clear on Esc
    echo.textContent = "";
  }
});
```

### 4. Form submission done right

```js
const form = document.querySelector("#signup");
const msg = document.querySelector("#form-msg");

form.addEventListener("submit", (event) => {
  event.preventDefault();            // no page reload!

  const username = document.querySelector("#username").value.trim();

  if (username.length < 3) {
    msg.textContent = "❌ Username must be at least 3 characters.";
    return;                          // guard clause — stop here
  }

  msg.textContent = `✅ Welcome, ${username}!`;
  form.reset();                      // clears all fields
});
```

### 5. Event delegation — one listener, many items

```js
const list = document.querySelector("#todo-list");

// ONE listener on the parent handles every delete button —
// including buttons on items added in the future.
list.addEventListener("click", (event) => {
  // Was the actual clicked thing a delete button?
  if (event.target.classList.contains("delete")) {
    // .closest() walks UP the tree to the nearest matching ancestor
    event.target.closest("li").remove();
  }
});

// Prove it works for dynamically added items:
const li = document.createElement("li");
li.innerHTML = `Added later <button class="delete">✕</button>`; // our own HTML — safe
list.append(li);
// Its delete button works with no extra wiring. That's delegation.
```

### 6. Toggling with `change`

```js
// <input type="checkbox" id="agree" /> <button id="continue" disabled>Continue</button>
const agree = document.querySelector("#agree");
const cont = document.querySelector("#continue");

agree.addEventListener("change", () => {
  cont.disabled = !agree.checked;    // checkbox state → button state
});
```

## Common Pitfalls

### 1. Calling the handler instead of passing it

```js
function handleClick() { console.log("clicked!"); }

btn.addEventListener("click", handleClick());  // ❌ runs NOW, passes undefined
btn.addEventListener("click", handleClick);    // ✅ passes the function itself
// With arguments, wrap it:
btn.addEventListener("click", () => greet("Ada")); // ✅
```

### 2. Forgetting `preventDefault` on forms

Symptoms: "my page flashes and everything resets when I click submit." The form reloaded the page. Add `event.preventDefault()` as the first line of the submit handler.

### 3. Listening on the button for form submission

```js
// ❌ Misses Enter-key submissions and can double-fire:
submitBtn.addEventListener("click", handler);
// ✅ The form's submit event catches both clicks and Enter:
form.addEventListener("submit", handler);
```

### 4. Attaching listeners to elements that don't exist yet

```js
// ❌ .card items are rendered later — this finds nothing at startup:
document.querySelectorAll(".card").forEach((c) =>
  c.addEventListener("click", select)
);
// ✅ Delegate to a stable ancestor instead:
container.addEventListener("click", (e) => {
  const card = e.target.closest(".card");
  if (card) select(card);
});
```

### 5. `target` vs `currentTarget`

```js
list.addEventListener("click", (e) => {
  console.log(e.target);        // the exact element clicked (maybe a <span> inside an <li>)
  console.log(e.currentTarget); // always the <ul> the listener is on
});
// Use closest() to normalize: const li = e.target.closest("li");
```

### 6. Piling up duplicate listeners

Registering the same anonymous handler inside code that runs repeatedly (e.g., inside `render()`) adds a **new** listener each time — clicks start firing 2×, 3×, ... Attach listeners **once** at startup, or delegate to a parent that never gets re-created.

## Practice Exercises

1. **Mood button.** A button cycles through moods on each click: 😐 → 🙂 → 😄 → 🤩 → back to 😐. Store the moods in an array and track the current index in a variable. Display the current mood in the button's own text.

2. **Live character limit.** An input and a counter paragraph: show `"N / 100"` as the user types (`input` event). At over 100 characters add a `too-long` CSS class to the counter and disable a nearby "Post" button; going back under re-enables it.

3. **Color keys.** Listen for `keydown` on `document`. Pressing `r`, `g`, or `b` sets the page background to that color; pressing `Escape` resets it; any other key logs `"Unmapped key: X"` using `event.key`.

4. **Validated signup form.** Build a form with username, email, and password fields plus a message area. On submit (with `preventDefault`): validate username ≥ 3 chars, email contains `"@"`, password ≥ 8 chars. Show the *first* failing rule as an error, or a success message and `form.reset()` when all pass.

5. **Delegated shopping list.** An input, an "Add" button, and an empty `<ul>`. Adding creates an `<li>` with the text and a "✕" button. Using **one** delegated listener on the `<ul>`: clicking "✕" removes that item; clicking the item's text toggles a `done` class (strikethrough via CSS). Verify items added at any time all behave correctly.
