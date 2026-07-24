# Chapter 5: Express Fundamentals

## Overview

In Chapter 4 you built a server by hand and collected a list of pain points: manual URL parsing, no dynamic routes, repeated JSON boilerplate, an if/else chain that wouldn't scale. Express is the standard answer to all of that. It's the most widely used Node web framework — deliberately small, unopinionated, and everywhere in job postings and tutorials, which makes it the right first framework even though newer ones exist (Fastify, Hono). This chapter covers the core: creating an app, defining routes with clean pattern matching, reading route parameters and query strings, sending responses correctly, and organizing files so your project doesn't become one giant `server.js`. Everything maps directly onto what you did manually last chapter — keep asking "what is Express doing for me here?" and you'll always know.

## Definitions & Explanations

**Express** is a web framework for Node: a library that wraps `http.createServer` and gives you routing, a middleware pipeline (Chapter 6), and response helpers. Under the hood, `app` is literally a request handler function you could pass to `http.createServer` yourself.

**`app`** — the object returned by calling `express()`. You register routes and middleware on it, then call `app.listen(port)` (which creates the underlying `http` server for you).

**Route** — a combination of an HTTP method, a path pattern, and a handler: `app.get('/users/:id', handler)`. Express checks incoming requests against routes *in the order you defined them* and runs the first match.

**Route handler** — a function `(req, res) => { ... }` Express calls when a route matches. Same `req`/`res` idea as Chapter 4, but both objects are extended with helpers.

**Route parameters (`req.params`)** — named, dynamic segments in a path pattern. In `/users/:id`, the `:id` segment matches any single path segment, and its value appears as `req.params.id`. **Always a string**, even when it looks like a number.

**Query string (`req.query`)** — the parsed `?key=value&other=thing` portion of the URL, as an object: `req.query.key`. Also always strings. Query strings are for *optional* modifiers (filtering, sorting, paging); route params are for *identifying* a resource.

**`res.json(data)`** — stringifies `data`, sets `Content-Type: application/json`, and ends the response. Replaces three lines of Chapter 4 boilerplate.

**`res.status(code)`** — sets the status code and returns `res` so you can chain: `res.status(201).json(newThing)`.

**`res.send(body)`** — flexible sender: strings become HTML/text, objects become JSON. Fine for quick tests; in an API, prefer the explicit `res.json` so intent is obvious.

**`res.redirect(url)`** — sends a `302` (or a code you choose) with a `Location` header telling the client to go elsewhere. Mostly used in server-rendered apps, rarely in JSON APIs.

**`res.sendStatus(code)`** — sets the status and sends its standard text as the body (`res.sendStatus(204)` for "success, no content to return").

**`express.Router()`** — a mini-app: an object you attach routes to, then mount on the main app with `app.use('/api/notes', notesRouter)`. This is how Express apps split routes across files.

**`node --watch`** — Node's built-in file watcher (stable in Node 22): reruns your server when files change, so you don't manually Ctrl+C and restart after every edit. The third-party tool **nodemon** does the same and predates it; you'll see both in the wild.

## Code Examples

### Setup and a first app

```powershell
mkdir express-demo; cd express-demo
npm init -y
npm install express
# Tell Node we're using ES modules (or edit package.json by hand):
npm pkg set type=module
```

```js
// server.js
import express from 'express';

const app = express();

app.get('/', (req, res) => {
  res.send('<h1>Hello from Express</h1>');
});

app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', uptime: process.uptime() });
});

app.listen(3000, () => {
  console.log('Listening on http://localhost:3000');
});
```

Run with auto-restart during development:

```powershell
node --watch server.js
```

Compare this to Chapter 4: no `writeHead`, no `JSON.stringify`, no URL parsing, and unmatched routes automatically get a 404 (a plain-text one — we'll customize it below).

### Routing: params, query strings, and all four main methods

```js
import express from 'express';

const app = express();
app.use(express.json()); // parse JSON bodies into req.body (full story in Ch. 6)

// Fake in-memory data for now — a real database arrives in Chapter 9.
let books = [
  { id: 1, title: 'The Pragmatic Programmer', read: true },
  { id: 2, title: 'Designing Data-Intensive Applications', read: false },
];
let nextId = 3;

// GET a collection, with optional filtering via the query string:
// /api/books            -> all books
// /api/books?read=true  -> only read ones
app.get('/api/books', (req, res) => {
  let result = books;
  if (req.query.read !== undefined) {
    // req.query.read is the STRING "true" or "false", never a boolean.
    const wantRead = req.query.read === 'true';
    result = books.filter((b) => b.read === wantRead);
  }
  res.json(result);
});

// GET one item by route param. :id matches one path segment.
app.get('/api/books/:id', (req, res) => {
  const id = Number(req.params.id); // params are strings — convert deliberately
  const book = books.find((b) => b.id === id);
  if (!book) {
    return res.status(404).json({ error: `No book with id ${id}` });
    // ^ note the `return` — without it, execution would continue below
  }
  res.json(book);
});

// POST creates. 201 + the created resource is the conventional response.
app.post('/api/books', (req, res) => {
  const { title } = req.body ?? {};
  if (typeof title !== 'string' || title.trim() === '') {
    return res.status(400).json({ error: 'title is required' });
  }
  const book = { id: nextId++, title: title.trim(), read: false };
  books.push(book);
  res.status(201).json(book);
});

// PUT replaces/updates an existing resource.
app.put('/api/books/:id', (req, res) => {
  const id = Number(req.params.id);
  const book = books.find((b) => b.id === id);
  if (!book) return res.status(404).json({ error: `No book with id ${id}` });
  const { title, read } = req.body ?? {};
  if (title !== undefined) book.title = title;
  if (read !== undefined) book.read = Boolean(read);
  res.json(book);
});

// DELETE. 204 = "done, nothing to say" — no body.
app.delete('/api/books/:id', (req, res) => {
  const id = Number(req.params.id);
  const exists = books.some((b) => b.id === id);
  if (!exists) return res.status(404).json({ error: `No book with id ${id}` });
  books = books.filter((b) => b.id !== id);
  res.sendStatus(204);
});

// Catch-all 404 — registered LAST, matches anything nothing else did.
app.use((req, res) => {
  res.status(404).json({ error: 'Not found' });
});

app.listen(3000, () => console.log('http://localhost:3000'));
```

Exercise the API from PowerShell:

```powershell
Invoke-RestMethod http://localhost:3000/api/books
Invoke-RestMethod http://localhost:3000/api/books/2
Invoke-RestMethod 'http://localhost:3000/api/books?read=true'   # quote URLs with ? or & !
Invoke-RestMethod -Uri http://localhost:3000/api/books -Method Post `
  -ContentType 'application/json' -Body '{"title":"Eloquent JavaScript"}'
Invoke-RestMethod -Uri http://localhost:3000/api/books/1 -Method Delete
```

(In bash you'd use `curl` with `-X POST -d '...'`; the only PowerShell trap is that unquoted `?`/`&` in URLs get eaten by the shell.)

### Naive → better: splitting routes out of server.js

Everything in one file works until roughly the second resource. The standard first refactor is one router per resource:

```js
// routes/books.js
import { Router } from 'express';

const router = Router();

// Paths here are RELATIVE to wherever the router gets mounted.
router.get('/', (req, res) => { /* list books */ });
router.get('/:id', (req, res) => { /* one book */ });
router.post('/', (req, res) => { /* create */ });

export default router;
```

```js
// server.js — now just wiring
import express from 'express';
import booksRouter from './routes/books.js'; // .js extension required in ESM!

const app = express();
app.use(express.json());

app.use('/api/books', booksRouter); // router handles everything under this prefix

app.use((req, res) => res.status(404).json({ error: 'Not found' }));

app.listen(3000, () => console.log('http://localhost:3000'));
```

A sane starting structure, which grows into the full layered layout in Chapter 13:

```
express-demo/
├── package.json
├── server.js          # app setup + listen
└── routes/
    ├── books.js
    └── users.js
```

## Common Pitfalls

1. **Forgetting `return` before early responses.** `res.status(404).json(...)` does *not* stop the function. Code after it keeps running, often calling `res.json` a second time → `ERR_HTTP_HEADERS_SENT`. Habit: `return res.status(...)...` for every early exit.
2. **Comparing `req.params.id` or `req.query` values to numbers with `===`.** They're always strings, so `req.params.id === 42` is never true. Convert with `Number(...)` first — and then check `Number.isInteger(...)` because `Number('abc')` is `NaN`, not an error.
3. **Forgetting `app.use(express.json())`, then reading `req.body`.** It'll be `undefined` and destructuring it throws. Express does not parse bodies unless you ask (Chapter 6 explains why this is middleware).
4. **Route order mistakes.** Express matches top-down. If you define `app.get('/api/books/:id')` *before* `app.get('/api/books/stats')`, then `/api/books/stats` matches `:id` with `id === "stats"`. Put literal routes before parameterized ones.
5. **Not quoting URLs with query strings in PowerShell.** `Invoke-RestMethod http://x/api?a=1&b=2` — PowerShell treats `&` as a statement separator and the command breaks confusingly. Always single-quote URLs containing `?` or `&`.
6. **Omitting the `.js` extension in ESM imports.** `import booksRouter from './routes/books'` throws `ERR_MODULE_NOT_FOUND`. In Node ES modules, relative imports need full extensions (this differs from what bundlers allowed you in front-end code).
7. **Using `res.send` for everything in a JSON API.** It works (objects become JSON), but it hides intent and behaves differently for strings vs objects. In an API, `res.json` for data, `res.sendStatus` for empty responses — consistency pays off when debugging.

## Practice Exercises

1. Rebuild your Chapter 4 exercise server (quotes + greet + todos) in Express. Diff the two files and annotate: which Express feature replaced which chunk of manual code? Your list from Chapter 4, exercise 6 should now have most boxes ticked.
2. Build a `/api/movies` resource with all five routes from the books example (list, get one, create, update, delete) over an in-memory array. Test every route *and every failure path* (bad id, missing fields, deleting twice) from PowerShell, checking status codes with `-StatusCodeVariable` or `curl.exe -i`.
3. Add query-string features to your movies list route: `?genre=scifi` filters, `?sort=year` sorts, and `?limit=5` caps the result count. Decide and document what happens when someone passes garbage like `?limit=banana`.
4. Split the movies app into `server.js` + `routes/movies.js` using `express.Router()`. Then add a second tiny resource (e.g., `/api/genres`) in its own router file to prove the structure scales.
5. Create a route `GET /api/movies/:id/similar` that returns other movies sharing a genre with the given movie. This exercises nested paths, params, and reusing your lookup logic — notice any copy-paste between routes and think about where shared logic should live (Chapter 13 will formalize the answer).
6. Deliberately reproduce three errors, and write down (in comments) the exact symptom of each: (a) missing `return` before an early `res.json`; (b) `:id` route defined before a literal sibling route; (c) reading `req.body` without `express.json()`. You will hit all three for real someday — recognizing them instantly is the payoff.
