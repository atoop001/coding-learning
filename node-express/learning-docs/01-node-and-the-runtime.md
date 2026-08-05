# Chapter 1: Node.js & the Runtime

## Overview

Everything you've written in the JavaScript track so far ran inside a browser (or was pasted into a console attached to one). This chapter moves your JavaScript out of the browser and onto your own machine as a first-class program — the same way Python or Java programs run. That shift is what Node.js provides, and it's the foundation of every backend job posting that says "Node" or "Express." By the end of this chapter you'll have Node installed via a version manager (the professional way), understand what Node actually *is* and how it differs from the browser environment you know, and be able to write scripts that read command-line arguments, print output, read environment variables, and exit with meaningful status codes. None of this is Express yet — deliberately. Express is a library that sits *on top of* everything in the next three chapters, and it will make far more sense once you know what it's built from.

## Definitions & Explanations

- **Runtime (JavaScript runtime)** — the environment that actually executes your JavaScript: an engine to run the language itself, plus a set of built-in APIs the code can call. The browser is one runtime (engine + DOM, `fetch`, `localStorage`…). Node.js is another runtime (engine + file system, networking, processes…). Same language, different toolbox.

- **Node.js** — a JavaScript runtime built on Chrome's V8 engine that runs *outside* the browser. It was created in 2009 specifically to build fast network servers, which is why its standard library is full of server-ish things: files, sockets, HTTP, streams, child processes.

- **V8** — the open-source JavaScript engine from Google Chrome. It parses your JS and compiles it to fast machine code. Node embeds V8, which is why Node's language behavior (syntax, `Array.prototype.map`, promises…) matches modern Chrome almost exactly. The *engine* is shared; the *APIs around it* are not.

- **Server-side JavaScript** — JavaScript that runs on a machine you control (your laptop during development, a rented server in production) instead of inside a visitor's browser. Server code can do things browser code must never be allowed to do: read arbitrary files, talk to databases directly, hold secrets.

- **LTS (Long-Term Support)** — Node releases in even-numbered major versions designated for long-term maintenance; check nodejs.org for the current LTS (this track assumes it and was written against Node 22+). Production apps and this entire track use LTS. Odd-numbered versions are short-lived experiments; skip them.

- **nvm-windows** — a *Node version manager* for Windows. Instead of installing Node once, globally, forever, you install nvm-windows and let it install and switch between Node versions. Real jobs involve multiple projects pinned to different Node versions; a version manager is how professionals handle that.

- **REPL (Read-Eval-Print Loop)** — an interactive prompt: it **R**eads a line of code, **E**valuates it, **P**rints the result, and **L**oops. You get one by running `node` with no arguments. It's Node's equivalent of the browser DevTools console — perfect for experiments, useless for real programs.

- **`process`** — a global object (no import needed) representing *the currently running Node program*. It's your window into how the program was started and your lever for how it ends: `process.argv` (arguments), `process.env` (environment variables), `process.exit()` (quit with a status code), `process.stdout` / `process.stdin` (raw output/input streams).

- **Command-line arguments (argv)** — extra words typed after your program's name when launching it: in `node convert.js input.txt --verbose`, the arguments are `input.txt` and `--verbose`. They're how users configure a program per-run without editing code. `argv` is short for "argument vector," a name inherited from C.

- **Environment variables** — named values that live in your *shell session* rather than in your code, e.g. `PATH`, or a database password you don't want in a file. Programs read them at startup via `process.env`. They're the standard way to configure the *same code* differently on your laptop vs. a production server (much more on this in Chapter 11).

- **Exit code (status code)** — a number a program reports when it ends. `0` means "succeeded"; anything non-zero means "failed." Shells, CI systems, and other programs make decisions based on it — it's how `npm test` knows your tests failed.

- **Global object** — in the browser, globals hang off `window`. In Node there is no `window`; the global object is called `globalThis` (and, Node-specifically, `global`). `console`, `setTimeout`, `fetch`, and `process` are all available globally in Node.

### How Node differs from the browser

| | Browser | Node |
|---|---|---|
| Engine | V8 (Chrome), others | V8 |
| DOM / `document` / `window` | ✅ | ❌ — there's no page |
| `alert`, `prompt`, `localStorage` | ✅ | ❌ |
| File system access | ❌ (sandboxed) | ✅ (`node:fs`) |
| Start a TCP/HTTP server | ❌ | ✅ (`node:http`, Chapter 4) |
| `fetch`, `console`, timers, Promises | ✅ | ✅ |
| Who runs it | Your site's visitors | You (your machine/server) |
| Security stance | Untrusted sandbox | Full trust — it's *your* program |

The mental shift: browser JS *reacts to a user on a page*. Node JS *is a program you launch*, which runs top to bottom, maybe waits for events (like incoming HTTP requests), and eventually exits.

## Code Examples

### Installing Node with nvm-windows (PowerShell)

Don't use the plain installer from nodejs.org. Install **nvm-windows** (search "nvm-windows releases" on GitHub, run `nvm-setup.exe`), then in a **new** PowerShell window:

```powershell
nvm install lts        # installs the current LTS (Node 22.x)
nvm use 22             # activate it (may prompt for admin — that's normal on Windows)
node --version         # v22.x.x
npm --version          # comes bundled with Node
```

> Bash note: on macOS/Linux the equivalent tool is `nvm` (a different project, same idea). Commands are nearly identical.

### The REPL

```powershell
node
```

```text
> 2 + 2
4
> const greet = (name) => `Hello, ${name}`
undefined
> greet("Node")
'Hello, Node'
> .exit
```

`.exit` (or Ctrl+C twice) leaves the REPL. Use it exactly like you used the DevTools console: to answer "what does this expression do?" in five seconds.

### Your first script

```powershell
mkdir hello-node
cd hello-node
```

Create `hello.js`:

```js
// hello.js — a complete Node program. No HTML file, no <script> tag.
const now = new Date();
console.log(`Hello from Node at ${now.toLocaleTimeString()}`);
console.log(`This file is running on Node ${process.version}`);
```

Run it:

```powershell
node hello.js
```

The program runs top to bottom and exits. That's the whole lifecycle of a simple script — nothing keeps it alive because nothing is waiting for events.

### Reading command-line arguments

```js
// args.js
// process.argv is an array:
//   [0] full path to the node executable
//   [1] full path to this script
//   [2+] the arguments you actually care about
console.log(process.argv);

const args = process.argv.slice(2); // the conventional first line of every CLI script
const [name] = args;

if (!name) {
  // Write usage errors to stderr, not stdout — stderr is for humans/diagnostics,
  // stdout is for the program's actual output (this matters when output is piped).
  console.error("Usage: node args.js <name>");
  process.exit(1); // non-zero = failure. Scripts that fail silently are a trap.
}

console.log(`Hello, ${name}!`);
```

```powershell
node args.js Ada       # Hello, Ada!
node args.js           # Usage message, and check the exit code:
echo $LASTEXITCODE     # 1   (PowerShell's variable for the last exit code)
```

> Bash note: the equivalent of `$LASTEXITCODE` is `$?`.

### Environment variables

```js
// env.js
// Naive: crash later, mysteriously, when GREETING is undefined somewhere deep in the code.
// Better: read config at the top, validate immediately, fail loudly.
const greeting = process.env.GREETING ?? "Hello";
const target = process.env.TARGET;

if (!target) {
  console.error("Missing required environment variable: TARGET");
  process.exit(1);
}

console.log(`${greeting}, ${target}!`);
```

```powershell
$env:GREETING = "Howdy"
$env:TARGET = "server"
node env.js            # Howdy, server!
```

> Bash note: `GREETING=Howdy TARGET=server node env.js` sets variables for one command only. PowerShell has no inline form — you set `$env:NAME` first (it lasts for the whole shell session).

### stdout vs. console.log, and reading stdin

```js
// stdin-shout.js — reads everything piped into it, prints it uppercased.
// console.log() is just process.stdout.write() plus a newline.
let input = "";
process.stdin.setEncoding("utf8");

for await (const chunk of process.stdin) {  // top-level await works in ES modules
  input += chunk;
}

process.stdout.write(input.toUpperCase());
```

For that top-level `for await` to work, tell Node this is an ES module — either name the file `stdin-shout.mjs`, or (better, and the track's default) create a `package.json` containing `{ "type": "module" }` in the folder. Chapter 2 explains this properly.

```powershell
"hello from a pipe" | node stdin-shout.mjs   # HELLO FROM A PIPE
Get-Content hello.js | node stdin-shout.mjs  # your own source, shouted
```

This is your first taste of why Node programs compose with the shell: they read streams in and write streams out, like any other command-line tool.

## Common Pitfalls

1. **Installing Node from the nodejs.org installer instead of nvm-windows.** It works today, but the first time a project needs a different Node version you'll be uninstalling/reinstalling by hand. Correction: install nvm-windows once; from then on Node versions are one `nvm install` away. (If you already have a global install, uninstall it first — nvm-windows will warn you about conflicts.)

2. **Trying to use `document`, `window`, `alert`, or `localStorage` in Node.** They don't exist — there is no page. `ReferenceError: document is not defined` almost always means "this code was written for a browser." Correction: for anything DOM-shaped, that logic belongs in your frontend; Node code works with files, requests, and data instead.

3. **Reading arguments from the wrong index of `process.argv`.** `process.argv[0]` is the Node executable and `[1]` is your script path — your first *real* argument is `[2]`. Correction: start every CLI script with `const args = process.argv.slice(2);`.

4. **Forgetting that `process.env` values are always strings (or `undefined`).** `process.env.PORT` is `"3000"`, not `3000`, and `process.env.DEBUG = "false"` is a *truthy string*. Correction: convert explicitly — `Number(process.env.PORT)` — and compare strings exactly: `process.env.DEBUG === "true"`.

5. **Exiting with code 0 on failure (or never setting an exit code at all).** If your script prints "ERROR: file not found" but exits 0, every script, CI job, and npm hook that calls it will believe it succeeded. Correction: `process.exit(1)` (or throw — an uncaught error also produces a non-zero exit) whenever the program couldn't do its job.

6. **Editing `$env:PATH` problems away by reusing an old PowerShell window.** After installing nvm-windows or switching versions, an already-open terminal may still see the old (or no) Node. Correction: open a fresh PowerShell window after installs; when things look impossible, `Get-Command node` shows exactly which `node.exe` you're running.

7. **Assuming `console.log` is only for debugging.** In CLI programs, stdout *is your user interface* and other programs may consume it. Mixing status chatter ("Processing…", "Done!") into stdout corrupts piped output. Correction: real output → `console.log`/`process.stdout`; diagnostics and errors → `console.error` (which writes to stderr).

## Practice Exercises

1. **Version check.** Install Node 22 LTS via nvm-windows. Then use `nvm list`, install one *other* LTS version (e.g. 20), switch between them with `nvm use`, prove the switch worked with `node --version`, and switch back to 22.

2. **REPL scavenger hunt.** In the REPL, evaluate: `globalThis === global`, `typeof window`, `typeof process`, `process.platform`, `process.cwd()`, and `Object.keys(process.env).length`. For each result, write one sentence (in a comment or notebook) explaining what it tells you about the Node environment.

3. **`greet.js`.** A script run as `node greet.js <name> [--shout]`. It prints `Hello, <name>!`; with `--shout` it prints it uppercased. With no name it prints a usage message to **stderr** and exits with code 1. Verify the exit code from PowerShell with `$LASTEXITCODE` in both the success and failure cases.

4. **`sum.js`.** Accepts any quantity of numeric arguments (`node sum.js 4 8 15 16`) and prints their sum. Non-numeric arguments should cause an error message naming the bad argument and a non-zero exit. Think about: what does `Number("16")` give you, and what does `Number("banana")` give you?

5. **`envcheck.js`.** Define an array of required variable names, e.g. `["APP_NAME", "APP_MODE"]`. The script reports every missing variable (all of them, not just the first) to stderr and exits 1 if any are missing; otherwise it prints a one-line summary like `APP_NAME=tracker (mode: dev)`. Test all paths by setting/removing `$env:` variables.

6. **`linecount.mjs`.** Using the stdin pattern from this chapter, read piped input and print only the number of lines received. Test with `Get-Content somefile.txt | node linecount.mjs`. Compare your count against PowerShell's own `(Get-Content somefile.txt | Measure-Object -Line).Lines` — if they differ by one, form a theory about trailing newlines.
