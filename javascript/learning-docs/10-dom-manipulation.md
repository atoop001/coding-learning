# Chapter 10: DOM Manipulation

## Overview

Everything so far has lived in the console. Now JavaScript meets the web page. The **DOM** (Document Object Model) is the browser's live, in-memory representation of your HTML — a tree of objects that JavaScript can read and modify. Change the DOM, and the page on screen changes instantly.

DOM manipulation is *the* skill that turns static pages into applications: showing and hiding panels, injecting lists of data, updating a score display, building whole interfaces from arrays of objects. Every frontend framework (React, Vue, etc.) is ultimately doing DOM manipulation for you — understanding the raw version makes you a far stronger developer.

## Definitions & Explanations

### The DOM tree

When the browser loads HTML, it parses it into a tree of **nodes**. Each element (`<div>`, `<p>`, `<button>`) becomes an **element node** with parent/child/sibling relationships mirroring the HTML nesting. The whole tree is reachable through the global **`document`** object.

### Selecting elements

The two selectors to learn (they accept **CSS selector syntax** — the same strings you use in stylesheets):

- **`document.querySelector(sel)`** — returns the **first** matching element, or `null` if none match.
- **`document.querySelectorAll(sel)`** — returns a **NodeList** of *all* matches (loop with `for...of` or `forEach`; convert with `[...list]` to use `map`/`filter`).

Selector examples: `"#signup"` (id), `".card"` (class), `"button"` (tag), `".card h2"` (descendant), `"input[type=email]"` (attribute).

Older APIs you'll see in tutorials: `getElementById`, `getElementsByClassName`. `querySelector` covers everything they do.

### Reading & changing content

- **`el.textContent`** — the plain text inside an element. Reading gives you the text; assigning replaces it. **Safe** — anything you assign is treated as literal text.
- **`el.innerHTML`** — the HTML markup inside. Assigning parses the string as HTML. Powerful but **dangerous with user input** (script injection — see pitfalls).
- **`input.value`** — the current text in form fields (`<input>`, `<textarea>`, `<select>`). Note: this is *not* `textContent`.

### Changing appearance

- **`el.style.propertyName = "..."`** — sets inline CSS. CSS names are camelCased: `background-color` → `el.style.backgroundColor`.
- **`el.classList`** — the better approach: define appearance in CSS classes and toggle them:
  - `el.classList.add("active")`
  - `el.classList.remove("active")`
  - `el.classList.toggle("active")` — add if missing, remove if present
  - `el.classList.contains("active")` — boolean check

### Attributes

- `el.getAttribute("href")` / `el.setAttribute("href", "...")`
- Common ones have direct properties: `img.src`, `a.href`, `input.disabled`, `input.placeholder`.
- **`data-*` attributes** let you attach custom data to elements in HTML (`data-id="42"`), read via `el.dataset.id`.

### Creating, inserting, removing

- `document.createElement("li")` — make a new element (not yet on the page).
- `parent.append(child)` — insert at the end (also `prepend`, `before`, `after`).
- `el.remove()` — delete from the page.

## Code Examples

All examples assume this HTML (save as `index.html` with `<script src="app.js" defer></script>` in the head):

```html
<h1 id="title">My App</h1>
<p class="status">Loading...</p>
<ul id="list"></ul>
<input id="name-input" placeholder="Your name" />
<button id="add-btn">Add</button>
<div id="card" class="card">Hello</div>
```

### 1. Selecting and reading

```js
const title = document.querySelector("#title");
console.log(title.textContent);      // "My App"

const status = document.querySelector(".status");
console.log(status.tagName);         // "P"

const missing = document.querySelector("#nope");
console.log(missing);                // null — always possible; check before using!
```

### 2. Changing text and styles

```js
const status = document.querySelector(".status");

status.textContent = "Ready ✅";              // replace the text
status.style.color = "green";                 // inline style (camelCase!)
status.style.fontWeight = "bold";

// Better: toggle classes defined in your CSS
// .highlight { background: yellow; padding: 4px; }
const card = document.querySelector("#card");
card.classList.add("highlight");
card.classList.toggle("highlight");           // now removed again
console.log(card.classList.contains("highlight")); // false
```

### 3. Working with `querySelectorAll`

```js
// Suppose the page has several <li class="item"> elements
const items = document.querySelectorAll(".item");

console.log(items.length);

// NodeList supports forEach directly:
items.forEach((item, i) => {
  item.textContent = `Item #${i + 1}`;
});

// Convert to a real array for map/filter:
const texts = [...items].map((el) => el.textContent);
console.log(texts);
```

### 4. Creating elements from data

This pattern — *array of data → DOM elements* — is the heart of every list-driven UI:

```js
const list = document.querySelector("#list");

const todos = [
  { text: "Learn the DOM", done: true },
  { text: "Build a to-do app", done: false },
  { text: "Get hired", done: false },
];

for (const todo of todos) {
  const li = document.createElement("li");
  li.textContent = todo.text;
  if (todo.done) {
    li.classList.add("done");        // e.g. .done { text-decoration: line-through; }
  }
  list.append(li);
}
```

### 5. Reading input and updating the page

```js
const input = document.querySelector("#name-input");
const title = document.querySelector("#title");

// (Chapter 11 covers events properly — here's a preview so this is runnable)
document.querySelector("#add-btn").addEventListener("click", () => {
  const name = input.value.trim();       // .value for form fields!
  if (name === "") return;               // guard: ignore empty input

  title.textContent = `Welcome, ${name}!`;
  input.value = "";                      // clear the field
});
```

### 6. Rebuilding a list (render pattern)

```js
const list = document.querySelector("#list");
let fruits = ["apple", "banana", "cherry"];

function render() {
  list.innerHTML = "";                   // clear existing children
  for (const fruit of fruits) {
    const li = document.createElement("li");
    li.textContent = fruit;
    list.append(li);
  }
}

render();                                // draw the initial state
fruits.push("date");                     // change the data...
render();                                // ...and re-draw
```

Keeping your **data in JavaScript** (arrays/objects) and treating the DOM as a *view* you re-render is a habit that scales all the way up to React.

## Common Pitfalls

### 1. Script runs before the HTML exists

```js
// ❌ If your <script> is in <head> WITHOUT defer:
const btn = document.querySelector("#add-btn");
console.log(btn); // null — the button hasn't been parsed yet!
```

Fix: add `defer` to the script tag, or place the script at the end of `<body>`.

### 2. Forgetting `querySelector` needs CSS syntax

```js
document.querySelector("title");    // ❌ selects the <title> TAG, not id="title"
document.querySelector("#title");   // ✅ # for ids
document.querySelector(".status");  // ✅ . for classes
```

### 3. `textContent` vs `value` confusion

```js
const input = document.querySelector("#name-input");
console.log(input.textContent);  // ❌ "" — inputs don't hold text content
console.log(input.value);        // ✅ what the user typed
```

### 4. `innerHTML` with user input (XSS)

```js
const userInput = '<img src=x onerror="alert(\'hacked!\')">';
el.innerHTML = userInput;        // ❌ executes attacker-controlled markup!
el.textContent = userInput;      // ✅ renders it as harmless literal text
```

Rule: `innerHTML` only with strings **you** wrote; `textContent` for anything from a user or an API.

### 5. Not handling `null` from selectors

```js
const el = document.querySelector("#typo-in-id");
el.textContent = "hi";           // ❌ TypeError: Cannot set properties of null

// ✅ fail loudly and clearly while developing:
if (!el) {
  console.error("Element #typo-in-id not found — check the HTML id!");
}
```

### 6. Expecting a live copy of `.length`-style data

Changing your JavaScript array does **not** update the page by itself. The DOM only changes when you explicitly modify it — hence the `render()` pattern above.

## Practice Exercises

For all exercises, create your own small HTML file and a linked JS file.

1. **Greeting board.** Create a page with an `<h1>`, a `<p>`, and a `<button>`. Using only JS: set the heading to your name, set the paragraph to today's date (`new Date().toDateString()`), and give the paragraph a CSS class you defined that changes its color.

2. **Zebra list.** Put 8 `<li>` items in a `<ul>` in your HTML. With `querySelectorAll` and `forEach`, give every even-indexed item a class `stripe` (define it in CSS with a background color), and append " ⭐" to the text of the last item.

3. **Data-driven menu.** Define an array of 5 objects `{ name, price, vegetarian }`. Render them into a list: each item shows `"Name — $price"`, vegetarian dishes get a 🌱 suffix, and dishes over $15 get a class `premium`. Write it as a `render()` function; then change the data and call `render()` again to prove it re-draws.

4. **Character counter.** Add an `<input>` and a `<p>` to your page. Using the `addEventListener("input", ...)` preview from this chapter, make the paragraph always display `"N characters"` for the current input length, turning red (via a class) past 20 characters.

5. **Theme switcher.** Add a button that toggles a `dark` class on `document.body`. Define `body.dark` styles in CSS (dark background, light text). Bonus: make the button's own label flip between "Dark mode" and "Light mode" using `classList.contains`.
