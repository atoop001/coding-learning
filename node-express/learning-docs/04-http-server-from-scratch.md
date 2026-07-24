# Chapter 4: Building an HTTP Server from Scratch

## Overview

Before you touch Express, you're going to build a working web server using nothing but Node's built-in `http` module. This is deliberate. Express is a thin layer over `http` — every `app.get()` you'll ever write ultimately becomes the kind of code in this chapter. If you skip this step, Express feels like magic, and magic is impossible to debug. If you do this chapter properly, Express will feel like what it actually is: a convenience library that saves you from writing the same URL-parsing and body-collecting boilerplate over and over. By the end you'll have handled raw requests and responses, routed by hand, parsed URLs and JSON bodies from the raw stream, and — most importantly — you'll understand exactly which pain points frameworks exist to solve.

## Definitions & Explanations

**HTTP (HyperText Transfer Protocol)** is the text-based request/response protocol the web runs on. A client (browser, `fetch`, curl) opens a connection, sends a request, and the server sends back exactly one response. Then they're done — HTTP itself has no memory between requests (it's **stateless**).

**Request** — what the client sends. It has four parts: a **method** (GET, POST, PUT, DELETE, ...), a **path** (`/users/42?sort=name`), **headers** (metadata like `Content-Type: application/json`), and optionally a **body** (data, usually on POST/PUT).

**Response** — what the server sends back: a **status code** (200, 404, 500...), **headers**, and optionally a **body**.

**Status codes** are three-digit numbers grouped by first digit: `2xx` success (200 OK, 201 Created, 204 No Content), `3xx` redirection (301, 302), `4xx` client errors (400 Bad Request, 404 Not Found), `5xx` server errors (500 Internal Server Error). You choose these — nothing forces you to send the right one, which is why sloppy servers send 200 with an error message inside. Don't be that server.

**The `http` module** is Node's built-in HTTP implementation. `http.createServer(handler)` returns a server object; the handler function runs once per incoming request.

**`req` (http.IncomingMessage)** — the request object Node hands you. It's a **readable stream**: headers arrive immediately (`req.method`, `req.url`, `req.headers`), but the body arrives in chunks over time. You have to collect it yourself.

**`res` (http.ServerResponse)** — the response object. It's a **writable stream**: you set the status and headers (`res.writeHead(...)`), write body data (`res.write(...)`), and finish with `res.end(...)`. Until you call `end()`, the client sits there waiting — forever, if you forget.

**Routing** — deciding what code runs based on the method + path combination. The `http` module gives you zero routing; `req.url` is just a string and it's your job to match it.

**The `URL` class** — Node's (and the browser's) standard URL parser. `new URL(req.url, 'http://localhost')` splits a raw URL string into `.pathname` and `.searchParams` so you don't hand-parse `?` and `&` yourself.

**Content-Type** — the header that tells the receiver how to interpret the body: `text/html`, `application/json`, `text/plain`, etc. If you send JSON with a `text/html` content type, browsers and clients will do the wrong thing with it.

**Port** — a number (1–65535) identifying which program on a machine should receive network traffic. Development servers conventionally use 3000, 5000, or 8080. Only one process can listen on a port at a time — the `EADDRINUSE` error means something else already has it.

## Code Examples

### The smallest possible server

```js
// server.js  (package.json has "type": "module")
import http from 'node:http';

const server = http.createServer((req, res) => {
  // This function runs for EVERY request: any method, any path.
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from Node\n');
});

server.listen(3000, () => {
  console.log('Listening on http://localhost:3000');
});
```

Run it and test it from a **second** PowerShell window (the first one is busy running the server):

```powershell
node server.js

# In another PowerShell tab:
Invoke-RestMethod http://localhost:3000/
# Or with curl — note the .exe! In PowerShell, plain `curl` is an alias
# for Invoke-WebRequest and takes different flags. curl.exe is the real curl.
curl.exe http://localhost:3000/
```

Stop the server with `Ctrl+C`. Notice the process doesn't exit on its own — `listen()` keeps the event loop alive on purpose.

### Routing by hand

```js
import http from 'node:http';

const server = http.createServer((req, res) => {
  // Parse the URL once. req.url is only the path+query ("/about?x=1"),
  // so we must supply a base for the URL constructor.
  const url = new URL(req.url, `http://${req.headers.host}`);

  if (req.method === 'GET' && url.pathname === '/') {
    res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
    res.end('<h1>Home</h1>');
  } else if (req.method === 'GET' && url.pathname === '/api/time') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    // JSON must be stringified by hand — res.end only takes strings/Buffers.
    res.end(JSON.stringify({ now: new Date().toISOString() }));
  } else if (req.method === 'GET' && url.pathname === '/search') {
    // Query strings: /search?q=node&limit=5
    const q = url.searchParams.get('q') ?? '';
    const limit = Number(url.searchParams.get('limit') ?? 10);
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ query: q, limit }));
  } else {
    // The fallthrough. Without this, unmatched requests hang forever.
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Not found' }));
  }
});

server.listen(3000, () => console.log('http://localhost:3000'));
```

Already you can feel the shape of the problem: every route repeats method-checking, path-checking, content-type setting, and stringifying. This if/else chain will not scale to 40 routes.

### Reading a request body (the hard way)

The body doesn't exist as a property on `req` — it streams in. You collect chunks until the `end` event:

```js
import http from 'node:http';

// Helper: collect the whole body as a string, then parse it.
function readBody(req) {
  return new Promise((resolve, reject) => {
    let data = '';
    req.on('data', (chunk) => {
      data += chunk; // chunk is a Buffer; += coerces it to a string
      if (data.length > 1_000_000) {
        // Never trust the client: without a cap, someone can stream
        // gigabytes at you and exhaust memory.
        reject(new Error('Body too large'));
        req.destroy();
      }
    });
    req.on('end', () => resolve(data));
    req.on('error', reject);
  });
}

const server = http.createServer(async (req, res) => {
  if (req.method === 'POST' && req.url === '/api/echo') {
    try {
      const raw = await readBody(req);
      const parsed = JSON.parse(raw); // throws on invalid JSON — hence try/catch
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ youSent: parsed }));
    } catch {
      // Invalid JSON is the CLIENT's fault: 400, not 500.
      res.writeHead(400, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: 'Invalid JSON body' }));
    }
    return;
  }
  res.writeHead(404, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ error: 'Not found' }));
});

server.listen(3000, () => console.log('http://localhost:3000'));
```

Test the POST route from PowerShell:

```powershell
# Invoke-RestMethod builds the request and parses the JSON response for you
Invoke-RestMethod -Uri http://localhost:3000/api/echo -Method Post `
  -ContentType 'application/json' -Body '{"name":"Ada"}'

# The curl.exe equivalent (quoting JSON in PowerShell needs the backslash-escaped
# inner quotes, or use single quotes outside):
curl.exe -X POST http://localhost:3000/api/echo -H "Content-Type: application/json" -d '{\"name\":\"Ada\"}'
```

### Naive → better: a tiny route table

Instead of a growing if/else chain, store routes as data. This is one small step toward what Express does:

```js
import http from 'node:http';

// Each key is "METHOD pathname"; each value is a handler function.
const routes = {
  'GET /': (req, res) => sendJson(res, 200, { message: 'home' }),
  'GET /api/health': (req, res) => sendJson(res, 200, { status: 'ok' }),
  'POST /api/notes': async (req, res) => {
    const body = JSON.parse(await readBody(req));
    sendJson(res, 201, { created: body }); // 201 = Created
  },
};

function sendJson(res, status, data) {
  // One place that sets the header and stringifies — no more repetition.
  res.writeHead(status, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify(data));
}

function readBody(req) {
  return new Promise((resolve, reject) => {
    let data = '';
    req.on('data', (c) => (data += c));
    req.on('end', () => resolve(data));
    req.on('error', reject);
  });
}

const server = http.createServer(async (req, res) => {
  const url = new URL(req.url, `http://${req.headers.host}`);
  const handler = routes[`${req.method} ${url.pathname}`];
  if (!handler) return sendJson(res, 404, { error: 'Not found' });
  try {
    await handler(req, res);
  } catch (err) {
    console.error(err); // log the real error server-side...
    sendJson(res, 500, { error: 'Internal server error' }); // ...send a generic one out
  }
});

server.listen(3000, () => console.log('http://localhost:3000'));
```

This is genuinely better — but notice what it still *can't* do: dynamic paths like `/api/notes/42` (the route table only matches exact strings), middleware that runs before every route, or content negotiation. Solving those well is hundreds of lines of fiddly code. **That's why frameworks exist**: Express is essentially this route-table idea, finished properly, tested by millions of apps, with pattern matching (`/notes/:id`), a middleware pipeline, and helpers like `res.json()`.

## Common Pitfalls

1. **Forgetting `res.end()`.** The request hangs until the client times out. Every code path through your handler must eventually call `end()` — exactly once. Symptom: `Invoke-RestMethod` just sits there.
2. **Calling `res.writeHead()` or `res.end()` twice.** You'll get `ERR_HTTP_HEADERS_SENT` or `write after end`. This usually means a missing `return` after handling a route, so execution fell through into the 404 branch too.
3. **Treating `req.url` as just the path.** It includes the query string — `req.url === '/search'` fails for `/search?q=x`. Always parse with `new URL(req.url, base)` and compare `url.pathname`.
4. **Assuming the body is available synchronously.** There is no `req.body` in raw Node. Beginners write `JSON.parse(req.body)` and get `undefined is not valid JSON`. The body is a stream; you must collect `data` events and wait for `end`.
5. **Sending JSON without `JSON.stringify` or with the wrong Content-Type.** `res.end({ ok: true })` throws — `end` takes strings or Buffers only. And a JSON string sent as `text/html` will confuse every client that respects headers.
6. **Wrong status codes for errors.** Returning 200 with `{"error": "not found"}` in the body, or 500 for bad client input. Rule of thumb: client sent something wrong → 4xx; your code broke → 5xx.
7. **Testing with `curl` instead of `curl.exe` in PowerShell.** Plain `curl` is aliased to `Invoke-WebRequest`, which rejects flags like `-X` and `-d` with confusing errors. Use `curl.exe`, or lean into `Invoke-RestMethod` — it's the more PowerShell-native tool anyway.

## Practice Exercises

1. Build a server with three routes: `GET /` returns an HTML welcome page, `GET /api/quote` returns a random quote from a hard-coded array as JSON, and everything else returns a JSON 404. Verify the Content-Type headers with `curl.exe -i`.
2. Add a `GET /greet` route that reads a `name` query parameter (`/greet?name=Ada`) and responds `{ "greeting": "Hello, Ada!" }`. If `name` is missing, respond 400 with a JSON error explaining what's required.
3. Build `POST /api/todos` that accepts a JSON body like `{ "title": "..." }`, assigns it an incrementing id, stores it in an in-memory array, and returns 201 with the created todo. Add `GET /api/todos` to list them. (Restarting the server wipes the array — that's expected; databases come in Chapter 9.)
4. Handle the failure cases for exercise 3: invalid JSON → 400; missing/empty `title` → 400 with a message; anything your code throws → 500 with a generic message (and `console.error` the real error). Test each case deliberately with `Invoke-RestMethod` and confirm the status codes with `curl.exe -i`.
5. Extend the route-table version to support one dynamic route: `GET /api/todos/42` should find the todo with id 42 (hint: check if the pathname *starts with* `/api/todos/` and extract the rest, converting it to a number). Return 404 if no todo has that id. Notice how awkward this is — you're about to appreciate `:id` in Express.
6. Write down (in a comment block at the top of your file) every piece of boilerplate you had to repeat across routes. Keep this list; after Chapter 5, revisit it and note which Express feature eliminated each item.
