# Chapter 1: Getting Started with JavaScript

## Overview

JavaScript is the programming language of the web. It runs in every browser, powers interactive websites, and — thanks to Node.js — also runs on servers, in build tools, and on the command line. If you want to build websites, web apps, or get hired as a web developer, JavaScript is non-negotiable.

This chapter gets you set up to *run* JavaScript in the two places you will use throughout this track:

1. **The browser** — where JavaScript was born and where your projects will live.
2. **Node.js** — a way to run JavaScript directly on your computer, outside a browser.

You will also meet the browser DevTools, which will be your debugging companion for the rest of this track.

Why this matters: everything else in this course assumes you can open a console, run a line of code, and see the result. Getting comfortable with that feedback loop *now* makes every later chapter faster.

## Definitions & Explanations

### What is JavaScript?

JavaScript (often abbreviated **JS**) is a high-level, interpreted programming language. Let's unpack those words:

- **High-level**: you write human-readable instructions; you don't manage memory or talk to hardware directly.
- **Interpreted**: your code is read and executed by an *engine* (like V8 in Chrome and Node.js) rather than compiled to a standalone program ahead of time.

JavaScript is standardized under the name **ECMAScript** (ES). You'll see terms like "ES6" or "ES2015" — these refer to versions of the language standard. Modern JavaScript (ES6 and later) is what this track teaches.

Important: **JavaScript is not Java.** They are unrelated languages that share part of a name for historical marketing reasons.

### Where JavaScript runs

- **Browser**: Every browser ships a JavaScript engine. JS in the browser can react to clicks, change what's on the page, fetch data, and more.
- **Node.js**: A runtime that takes the same engine Chrome uses (V8) and lets it run on your machine without a browser. Used for servers, scripts, and tooling.

### The developer console

Every browser includes **Developer Tools** (DevTools). The **Console** tab is an interactive JavaScript prompt: type code, press Enter, see the result immediately. This is called a **REPL** (Read–Evaluate–Print Loop).

- Open DevTools: `F12`, or `Ctrl+Shift+I` (Windows/Linux), or `Cmd+Option+I` (Mac). Then click the **Console** tab.

### Statements and comments

A **statement** is a single instruction, like "print this text". Statements usually end with a semicolon (`;`). Semicolons are technically optional in many cases, but this track uses them consistently — it avoids a category of subtle bugs.

A **comment** is text the engine ignores — notes for humans:

- `// single-line comment`
- `/* multi-line comment */`

### `console.log`

`console.log(...)` prints values to the console. It's the simplest way to see what your code is doing, and you will use it constantly.

## Code Examples

### 1. Your first line of JavaScript (browser console)

Open the browser console and type:

```js
// Prints text to the console
console.log("Hello, world!");
```

You should see `Hello, world!` printed. You just ran JavaScript.

### 2. The console as a calculator

```js
// The console evaluates expressions and shows their result
2 + 3;          // 5
10 * 4;         // 40
100 / 3;        // 33.333333333333336 (more on this later!)
"Java" + "Script"; // "JavaScript" — + joins strings together
```

### 3. Running JS in a web page

Create a folder for practice, and inside it create `index.html`:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My First JS Page</title>
  </head>
  <body>
    <h1>Check the console!</h1>

    <!-- The script tag loads and runs JavaScript.
         Placing it at the end of <body> ensures the page
         content exists before the script runs. -->
    <script src="app.js"></script>
  </body>
</html>
```

And `app.js` in the same folder:

```js
// This runs automatically when the page loads
console.log("Hello from app.js!");
console.log("The current time is:", new Date());
```

Open `index.html` in your browser (double-click it, or drag it into a browser window), open DevTools, and check the Console tab.

### 4. Running JS with Node.js

Install Node.js from [nodejs.org](https://nodejs.org) (choose the LTS version). Then, in a terminal:

```bash
# Check that Node is installed — prints a version like v22.x.x
node --version

# Start the Node REPL (interactive prompt). Type JS, press Enter.
node
```

Inside the REPL:

```js
console.log("Hello from Node!");
2 ** 10;   // 1024 — exponentiation
.exit      // leaves the REPL
```

Run a file with Node:

```bash
# Runs app.js and prints its output to the terminal
node app.js
```

### 5. A tiny taste of what's coming

Don't worry about understanding every detail — this is a preview:

```js
// A variable stores a value
let name = "Ada";

// A template literal embeds values in a string
console.log(`Hello, ${name}!`); // Hello, Ada!

// A function bundles reusable behavior
function greet(person) {
  return `Welcome, ${person}!`;
}

console.log(greet("Grace")); // Welcome, Grace!
```

## Common Pitfalls

### 1. Editing code in the console and expecting it to persist

Code typed into the console disappears when you refresh the page. The console is for *experimenting*. Real code belongs in `.js` files loaded by your HTML (or run with Node).

### 2. Script tag placed before the content it needs

```html
<!-- ❌ Problem: script runs before <h1> exists -->
<head>
  <script src="app.js"></script>
</head>
<body>
  <h1 id="title">Hi</h1>
</body>
```

If `app.js` tries to work with the `<h1>`, it fails because the page hasn't been built yet. Fix: put the script at the end of `<body>`, or use the `defer` attribute:

```html
<!-- ✅ defer waits until the page is parsed -->
<head>
  <script src="app.js" defer></script>
</head>
```

### 3. Confusing the file path in `src`

```html
<!-- ❌ Wrong: app.js is in a "js" folder but src doesn't say so -->
<script src="app.js"></script>

<!-- ✅ Correct: path is relative to the HTML file -->
<script src="js/app.js"></script>
```

If your script "does nothing," check the browser console — a red 404 error means the file wasn't found.

### 4. Ignoring error messages

Beginners often panic at red console errors. Don't — errors are your friends. They tell you *what* went wrong and *which line* it happened on. Read them top to bottom; the first line is usually the most important.

### 5. Typos in `console.log`

```js
Console.log("hi");  // ❌ ReferenceError: Console is not defined
console.Log("hi");  // ❌ TypeError: console.Log is not a function
console.log("hi");  // ✅ JavaScript is case-sensitive!
```

## Practice Exercises

1. **Console explorer.** Open your browser's DevTools console. Use it to compute: the number of seconds in a year, your age in days (roughly), and the result of `"5" + 5`. Write down anything that surprises you — we'll explain it in Chapter 2.

2. **First page.** Create a folder called `practice-01` with an `index.html` and an `app.js`. Make the page display a heading, and make the script log three different messages to the console, including one that uses `new Date()`.

3. **Node hello.** Create a file `hello.js` that logs a short introduction of yourself (name, one hobby, one goal for learning JS) across three `console.log` calls. Run it with `node hello.js`.

4. **Break it on purpose.** In your `app.js`, deliberately misspell `console.log` and reload the page. Read the error in the console. Then change the `src` path in your HTML to a wrong filename and observe what error appears. Fix both.

5. **Comment practice.** Take your `app.js` and add a single-line comment above each statement explaining what it does, plus one multi-line comment at the top describing the file's purpose.
