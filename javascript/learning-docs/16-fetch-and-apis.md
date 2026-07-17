# Chapter 16: Fetch & Working with APIs / JSON

## Overview

Real apps live on data that comes from somewhere else: weather services, product catalogs, user databases. On the web, that data is served by **APIs** (Application Programming Interfaces) over HTTP, almost always as **JSON**. JavaScript's built-in **`fetch`** function is how you request it.

This chapter combines everything: async/await (Chapter 15) to wait for responses, error handling (Chapter 12) for the many ways networking fails, objects/arrays (Chapters 7–8) to work with the data, and DOM manipulation (Chapters 10–11) to display it. When you can fetch an API and render its data, you can build genuinely useful applications — and that's exactly what employers want to see in a portfolio.

## Definitions & Explanations

### HTTP in two minutes

A client (your JS) sends a **request** to a URL; a server sends back a **response**.

- **Methods** (verbs): `GET` (read data — the default), `POST` (create), `PUT`/`PATCH` (update), `DELETE` (remove).
- **Status codes** (response results):
  - `2xx` success — `200 OK`, `201 Created`
  - `4xx` client error — `400 Bad Request`, `401 Unauthorized`, `404 Not Found`, `429 Too Many Requests`
  - `5xx` server error — `500 Internal Server Error`
- **Headers**: metadata on requests/responses — e.g. `Content-Type: application/json`.

### JSON

**JSON** (JavaScript Object Notation) is a text format for data that looks almost like JS literals:

```json
{ "name": "Ada", "age": 36, "tags": ["math", "computing"], "active": true }
```

Rules that differ from JS: keys **must** be double-quoted; strings must use double quotes; no comments, no trailing commas, no `undefined`, no functions.

- `JSON.parse(text)` — JSON string → JavaScript object (throws `SyntaxError` on invalid input).
- `JSON.stringify(obj)` — object → JSON string (`JSON.stringify(obj, null, 2)` pretty-prints).

### `fetch` — the two-step dance

```js
const response = await fetch(url);   // step 1: get the Response (headers, status)
const data = await response.json();  // step 2: read & parse the body (also async!)
```

Both steps are asynchronous — the body may still be streaming in when the response object arrives.

Key `Response` properties: `response.ok` (true for status 200–299), `response.status`, `response.statusText`, and body readers `.json()`, `.text()`.

### The critical quirk: fetch doesn't reject on 404

`fetch` only **rejects** on *network-level* failures (no connection, DNS failure, CORS block). A `404` or `500` is, to fetch, a perfectly good response that happens to carry bad news. **You must check `response.ok` yourself** and throw if it's false. Forgetting this is the #1 fetch bug.

### Sending data

```js
await fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ title: "Hi" }),   // body must be a STRING
});
```

### Practice APIs and CORS

Free, no-auth APIs for practice: `jsonplaceholder.typicode.com` (fake blog data), `restcountries.com`, `api.open-meteo.com` (weather), `pokeapi.co`. Real APIs often need an **API key** (sent as a header or query parameter — keep keys out of public code).

**CORS** (Cross-Origin Resource Sharing): browsers block JS from reading responses from a different domain unless that server opts in via headers. If you hit a CORS error, it's the *server's* policy — the practice APIs above all allow browser access.

## Code Examples

### 1. The canonical GET request

```js
async function getPost(id) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`);

  if (!response.ok) {                                   // ← never skip this!
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }

  return response.json();   // returns a promise; caller can await it
}

// Usage:
try {
  const post = await getPost(1);
  console.log(post.title);         // property access on parsed JSON — just an object
} catch (err) {
  console.error("Could not load post:", err.message);
}
```

### 2. Fetching a list and shaping the data

```js
async function getUserNames() {
  const res = await fetch("https://jsonplaceholder.typicode.com/users");
  if (!res.ok) throw new Error(`HTTP ${res.status}`);

  const users = await res.json();          // an array of objects

  // From here it's just Chapter 7 — array methods on plain data:
  return users
    .filter((u) => u.address.city !== "Gwenborough")
    .map((u) => ({ name: u.name, email: u.email.toLowerCase() }));
}

console.log(await getUserNames());
```

### 3. Fetch → DOM: the full loop

```html
<button id="load">Load users</button>
<p id="status"></p>
<ul id="users"></ul>
```

```js
const statusEl = document.querySelector("#status");
const listEl = document.querySelector("#users");

async function loadUsers() {
  statusEl.textContent = "Loading...";            // 1. loading state
  listEl.innerHTML = "";

  try {
    const res = await fetch("https://jsonplaceholder.typicode.com/users");
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const users = await res.json();

    for (const user of users) {                   // 2. success state: render
      const li = document.createElement("li");
      li.textContent = `${user.name} (${user.email})`;  // textContent = safe
      listEl.append(li);
    }
    statusEl.textContent = `Loaded ${users.length} users ✅`;
  } catch (err) {
    statusEl.textContent = `Failed to load: ${err.message} ❌`;  // 3. error state
  }
}

document.querySelector("#load").addEventListener("click", loadUsers);
```

Every data-driven UI has those three states — **loading, success, error**. Handling all three is what makes an app feel professional.

### 4. POST — creating data

```js
async function createTodo(title) {
  const res = await fetch("https://jsonplaceholder.typicode.com/todos", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ title, completed: false, userId: 1 }),
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();          // servers typically echo back the created object
}

const created = await createTodo("Master fetch");
console.log(created);          // { title: "Master fetch", completed: false, userId: 1, id: 201 }
```

### 5. Parallel requests

```js
async function getPostWithComments(postId) {
  const base = "https://jsonplaceholder.typicode.com";

  // Independent requests → run them in parallel (Chapter 15!)
  const [postRes, commentsRes] = await Promise.all([
    fetch(`${base}/posts/${postId}`),
    fetch(`${base}/posts/${postId}/comments`),
  ]);
  if (!postRes.ok || !commentsRes.ok) throw new Error("Request failed");

  const [post, comments] = await Promise.all([postRes.json(), commentsRes.json()]);
  return { ...post, comments };            // merge into one convenient object
}

const full = await getPostWithComments(1);
console.log(`${full.title} — ${full.comments.length} comments`);
```

### 6. Query parameters, built safely

```js
async function searchCountries(name) {
  // URL/URLSearchParams handle encoding (spaces, accents, &, etc.) for you:
  const url = new URL("https://restcountries.com/v3.1/name/" + encodeURIComponent(name));
  url.searchParams.set("fields", "name,capital,population");

  const res = await fetch(url);
  if (res.status === 404) return [];       // for THIS api, 404 = "no matches" — expected
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

for (const c of await searchCountries("new zea")) {
  console.log(`${c.name.common} — capital ${c.capital?.[0]}, pop ${c.population.toLocaleString()}`);
}
```

## Common Pitfalls

### 1. Not checking `response.ok`

```js
// ❌ 404 sails through; res.json() then parses an error page or unexpected body
const data = await (await fetch(url)).json();

// ✅ Always gate on ok/status before reading the body:
const res = await fetch(url);
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const data = await res.json();
```

### 2. Forgetting the second `await`

```js
const res = await fetch(url);
const data = res.json();          // ❌ data is a pending Promise
console.log(data.title);          // undefined
const real = await res.json();    // ✅ (note: a body can only be read ONCE)
```

### 3. Reading the body twice

```js
const res = await fetch(url);
const a = await res.json();
const b = await res.json();       // ❌ TypeError: body stream already read
// ✅ Read once, reuse the variable (or res.clone() before the first read).
```

### 4. Sending an object instead of a string

```js
fetch(url, { method: "POST", body: { title: "hi" } });        // ❌ becomes "[object Object]"
fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ title: "hi" }),                       // ✅
});
```

### 5. Guessing the response shape

APIs nest data in unexpected ways (`data.results[0].name.common`...). Don't guess: `console.log` the parsed response once (or check the API docs), *then* write property accesses — with optional chaining (`?.`) for anything that might be missing.

### 6. Treating every 404 as a crash

For some endpoints, 404 legitimately means "no results" (see the countries example). Decide per-API: expected-miss → return an empty result; genuine failure → throw. This is Chapter 12's throw-vs-return decision applied to HTTP.

## Practice Exercises

Use `https://jsonplaceholder.typicode.com` unless noted.

1. **Single fetch.** Write `getUser(id)` that fetches `/users/:id`, checks `response.ok`, and returns `"<name> <email>"`. Call it with id 3 (works) and id 999 (should produce a clean error message via try/catch, not a crash).

2. **Album browser.** Fetch `/albums?userId=1` and `/users/1` **in parallel**. Print the user's name followed by a numbered list of their album titles. Then repeat sequentially and (with `console.time`) compare how long each version takes.

3. **JSON round-trip.** Build a JS object describing your favorite meal (with a nested object and an array). `JSON.stringify` it (pretty-printed), then `JSON.parse` it back, and demonstrate one thing that JSON cannot represent (e.g., a function property or `undefined` field disappearing).

4. **Todo dashboard.** Fetch `/todos?userId=2`, then using array methods report: total todos, how many are completed, the percentage complete, and the titles of the first three incomplete ones. Render the report into the DOM with a loading message while fetching and an error message path you can test by breaking the URL.

5. **Mini search UI.** Build a page with an input and a button that searches `https://restcountries.com/v3.1/name/<query>`. Show loading / results (country name, capital, population) / "no matches" / error states. Bonus: trigger search on Enter and ignore empty queries.
