# Chapter 2: npm & Modules

## Overview

Chapter 1's scripts were single files. Real backends are dozens of files of your own code plus hundreds of packages of other people's code, and the thing that holds all of it together is npm and the module system. This chapter covers the two halves of that: **npm** — the package manager and registry that installs, records, and locks third-party code — and **modules** — how Node splits code across files and how ES modules (`import`/`export`, which you know from the JavaScript track) coexist with Node's older CommonJS system (`require`, which you *will* meet in the wild, in docs, and in interviews). You'll learn to read a `package.json` the way you'd read a recipe, understand what the `^` in `"express": "^5.1.0"` actually promises, know why the lockfile exists and why it's committed to git while `node_modules` never is, and structure a multi-file project of your own modules. Every subsequent chapter assumes this one: an Express app *is* a `package.json`, a dependency list, and a folder of your own modules.

## Definitions & Explanations

- **Package** — a folder of reusable code with a `package.json` describing it. Your project is a package; everything you install is a package. The words "package," "dependency," "module" (loosely), and "library" overlap heavily in practice.

- **npm** — three things sharing one name: (1) the **registry**, a giant public database of packages at npmjs.com; (2) the **CLI tool** `npm` that came with your Node install; (3) colloquially, the whole ecosystem. Alternatives to the CLI exist (`pnpm`, `yarn`) but talk to the same registry; this track uses plain `npm`.

- **`package.json`** — the manifest file at your project root. It records the project's name and version, its dependencies, its npm scripts, and — critically for this track — `"type": "module"`, which switches Node to treating `.js` files as ES modules. It is *hand-editable JSON*: no comments allowed, double quotes required, trailing commas forbidden.

- **Dependency** — a package your code needs *at runtime* to work (Express, a database driver). Recorded under `"dependencies"`.

- **Dev dependency (devDependency)** — a package needed only while *developing*: test runners, linters, type-checkers. Recorded under `"devDependencies"` via `npm install -D`. The distinction matters in production: deployment steps can skip devDependencies entirely (`npm install --omit=dev`), making installs smaller and faster.

- **Semver (semantic versioning)** — the `MAJOR.MINOR.PATCH` version convention, e.g. `5.1.0`. The *promise*: **PATCH** bumps (5.1.0 → 5.1.1) fix bugs, **MINOR** bumps (5.1 → 5.2) add features without breaking anything, **MAJOR** bumps (5 → 6) may break your code. It's a social contract, not physics — maintainers occasionally break it — which is why lockfiles exist.

- **Version ranges** — what you write in `package.json`:
  - `"5.1.0"` — exactly 5.1.0, nothing else.
  - `"~5.1.0"` — 5.1.x: patch updates only.
  - `"^5.1.0"` — 5.x.y where x ≥ 1: any minor/patch, never a new major. This is npm's default and what you'll almost always see.

- **`package-lock.json` (lockfile)** — the exact, resolved version of *every* package in your tree, including dependencies-of-dependencies, with integrity hashes. `package.json` says "any Express 5.x is fine"; the lockfile says "we tested with exactly 5.1.0, and these exact 90 transitive packages." Committing it means teammates, CI, and production all install *identical* code.

- **`node_modules/`** — the folder where installed packages physically live. It is huge, machine-generated, and 100% reproducible from the lockfile — which is why it goes in `.gitignore`, always, no exceptions.

- **npm scripts** — named shell commands stored under `"scripts"` in `package.json` and run with `npm run <name>`. They're the project's public verbs: `npm run dev`, `npm test`, `npm start`. Scripts can run any binaries installed by your dependencies without a path, because npm puts `node_modules/.bin` on the PATH while a script runs.

- **Module** — a single file with its own scope. Nothing in a module leaks out unless exported; nothing comes in unless imported. Node has two module systems:

- **ES modules (ESM)** — the standard you learned in the JS track: `import express from "express"`, `export function …`. Node treats `.js` files as ESM when the nearest `package.json` has `"type": "module"` (or when the file is named `.mjs`). **This track uses ESM everywhere.**

- **CommonJS (CJS)** — Node's original, pre-standard module system: `const express = require("express")`, `module.exports = …`. It's synchronous, older, and still everywhere — older tutorials, many published packages, legacy codebases at jobs. You need to *read* it fluently even though you'll *write* ESM.

- **Module specifier** — the string you import from, resolved three ways: bare names (`"express"`) come from `node_modules`; relative paths (`"./db.js"`) are your own files — **in ESM the `.js` extension is required**; and `node:`-prefixed names (`"node:fs"`) are Node's built-ins (the prefix is convention now: it makes built-ins unambiguous).

- **`npx`** — runs a package's command-line tool without permanently installing it (`npx serve .`), or runs an already-installed dev tool from your terminal.

## Code Examples

### Creating a project

```powershell
mkdir quote-tool
cd quote-tool
npm init -y      # generates a default package.json without asking questions
```

Then edit `package.json` to add the one line this whole track depends on, plus a script:

```json
{
  "name": "quote-tool",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node src/index.js"
  }
}
```

Without `"type": "module"`, `import` statements in `.js` files throw `SyntaxError: Cannot use import statement outside a module`. This is the single most common day-one Node error for people coming from a frontend background.

### Installing dependencies vs. dev dependencies

```powershell
npm install chalk            # runtime dependency (colored terminal output)
npm install -D vitest        # dev-only: the test runner from Chapter 12
```

Resulting `package.json` fragment:

```json
{
  "dependencies":    { "chalk": "^5.4.1" },
  "devDependencies": { "vitest": "^3.1.4" }
}
```

Notice you didn't write those version numbers — npm picked the latest and prefixed `^`. Also notice two new arrivals: `node_modules/` (never commit) and `package-lock.json` (always commit). Your `.gitignore` needs at minimum:

```gitignore
node_modules/
```

### Reading semver ranges

```json
{
  "dependencies": {
    "express": "^5.1.0",   // any 5.x.y ≥ 5.1.0 — normal choice
    "pg": "~8.14.0",       // only 8.14.x — cautious choice
    "left-pad": "1.3.0"    // exactly 1.3.0 — paranoid choice (and JSON can't
  }                        //   actually contain these comments — remove them!)
}
```

The naive mental model is "the version I have is the version written there." The accurate model: `package.json` states a *range you'd accept*, `package-lock.json` records the *exact version you actually have*. `npm install` obeys the lockfile when present; `npm update` moves you to the newest versions the ranges allow and rewrites the lockfile.

### Your own modules

A small project split sensibly:

```text
quote-tool/
├── package.json
└── src/
    ├── index.js       (entry point: wiring, I/O)
    ├── quotes.js      (data)
    └── format.js      (pure logic)
```

```js
// src/quotes.js — named exports for a data module
export const quotes = [
  { text: "Simplicity is prerequisite for reliability.", author: "Dijkstra" },
  { text: "Make it work, make it right, make it fast.", author: "Kent Beck" },
];

export function randomQuote() {
  return quotes[Math.floor(Math.random() * quotes.length)];
}
```

```js
// src/format.js — a default export suits a module with one obvious job
export default function formatQuote({ text, author }) {
  return `"${text}" — ${author}`;
}
```

```js
// src/index.js — the entry point imports both styles.
// Note the ./ and the .js extension: both are REQUIRED in Node ESM.
// (Bundler-based frontend code often let you omit them — Node does not.)
import { randomQuote } from "./quotes.js";
import formatQuote from "./format.js";
import chalk from "chalk";              // bare specifier → node_modules

console.log(chalk.cyan(formatQuote(randomQuote())));
```

```powershell
npm run start     # or its shorthand: npm start
```

### Reading CommonJS (so old code doesn't scare you)

The same `format.js` in the CJS dialect you'll see in older tutorials:

```js
// CommonJS — recognize it, don't write it in this track
const chalk = require("chalk");           // ≈ import

function formatQuote({ text, author }) {
  return `"${text}" — ${author}`;
}

module.exports = formatQuote;             // ≈ export default
module.exports.somethingElse = 42;        // ≈ a named export (roughly)
```

Rules of thumb when the two worlds meet: ESM files can `import` most CJS packages just fine (Node bridges it), but CJS files **cannot** `require()` an ES module. If a tutorial mixes `require` and `import` in one file, the tutorial is broken.

### npm scripts as the project's control panel

```json
{
  "scripts": {
    "start": "node src/index.js",
    "dev": "node --watch src/index.js",
    "test": "vitest run"
  }
}
```

`node --watch` (built into Node 22) restarts the process whenever a file changes — you'll live in `npm run dev` from Chapter 5 onward. `npm start` and `npm test` are special enough to not need `run`; everything else is `npm run <name>`.

## Common Pitfalls

1. **Forgetting `"type": "module"` and hitting `Cannot use import statement outside a module`.** Correction: add `"type": "module"` to `package.json` the moment you create it. Make it part of your project-setup ritual: `npm init -y`, add type, create `.gitignore`.

2. **Omitting `./` or the `.js` extension in imports of your own files.** `import { x } from "utils.js"` makes Node hunt `node_modules` for a package named `utils.js`; `import { x } from "./utils"` fails with `ERR_MODULE_NOT_FOUND` because ESM requires full filenames. Correction: your files are always `./name.js` (or `../lib/name.js`) — dot-slash and extension, every time.

3. **Committing `node_modules` (or deleting `package-lock.json` to "fix" things).** The first bloats your repo by hundreds of MB; the second throws away the only record of exactly which versions worked. Correction: `.gitignore` the former, commit the latter. When installs act genuinely haunted, the reset ritual is: delete `node_modules`, keep the lockfile, `npm install`.

4. **Installing everything as a regular dependency.** Test runners and linters in `"dependencies"` get installed on production servers for no reason. Correction: ask "does the app *run* with this at 3 a.m. in production?" No → `npm install -D`.

5. **Editing `package.json` with JavaScript habits.** JSON has no comments, no single quotes, no trailing commas — one stray comma and every npm command fails with a parse error pointing at a confusing position. Correction: let VS Code's JSON validation underline mistakes, and re-read the reddest line.

6. **Assuming `npm install somepackage` from a random tutorial is safe.** Typosquatting (malicious packages named `expres` or `chaIk`) is a real attack. Correction: check the package's page on npmjs.com — weekly downloads in the thousands+ and a linked repo are your quick sanity signals. Copy install commands from official docs, not comment sections.

7. **Confusing `npm install` (obey the lockfile) with `npm update` (advance the lockfile).** Running `npm update` casually can pull in new minor versions of 200 packages the day before a deadline. Correction: `npm install` for day-to-day; `npm update` deliberately, alone, with time to run your tests afterward. `npm outdated` shows what *would* move before you commit to it.

## Practice Exercises

1. **Manifest autopsy.** Pick any popular package (Express itself is a good choice), find its repository on GitHub, and read its `package.json` top to bottom. Identify: its dependencies vs devDependencies (what's each *for*?), its scripts, and its version. Which of its dependencies use `^`, and are any pinned exactly?

2. **Range reasoning.** Given `"^4.17.1"`, `"~4.17.1"`, and `"4.17.1"`, write down for each whether these releases would be acceptable installs: `4.17.2`, `4.18.0`, `4.17.0`, `5.0.0`. Check your answers with npm's semver calculator (search "npm semver calculator").

3. **Build `word-tool`.** Create a fresh ESM project with `src/index.js`, `src/analyze.js` (exports functions `wordCount`, `longestWord`), and `src/clean.js` (default-exports a function that lowercases text and strips punctuation). `index.js` reads a string from `process.argv`, cleans it, and prints both analyses. Add `start` and `dev` scripts and run it through both.

4. **Translation drill.** Take your `analyze.js` and rewrite it, on paper or in a scratch file, in CommonJS (`require`/`module.exports`). Then take some CJS snippet from an older Stack Overflow answer and translate it to ESM. You're done when both directions feel mechanical.

5. **Lockfile experiment.** In a scratch project, install `chalk`, note the exact version in `package-lock.json`, then delete `node_modules` and reinstall. Same version? Now change `package.json` to an older range like `"chalk": "^4.0.0"`, reinstall, and observe what changed in both the lockfile and `node_modules/chalk/package.json`. Write one sentence stating what each of the two files is *for*.

6. **Script plumbing.** Add a script `"check": "node src/check.js"` where `check.js` exits 0 if a `data/` folder exists and 1 otherwise (look up `node:fs`'s `existsSync` — a preview of Chapter 3). Confirm that `npm run check`'s exit status reflects your script's (`$LASTEXITCODE` in PowerShell), and explain in a sentence why npm propagating exit codes matters for CI.
