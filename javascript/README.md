# JavaScript Learning Track

A self-paced path from "I've dabbled" to job-ready web development JavaScript. It has two halves that interleave:

- **`learning-docs/`** — 18 chapters of primary study material. Read in order; each assumes only the chapters before it. Do the practice exercises — they're where the learning actually happens.
- **`projects/`** — 8 guided project specs, easiest to hardest, with requirement checklists but **no solution code**. Each bundles several chapters; build them at the checkpoints below.

## Chapters (read in order)

| # | File | Topic |
|---|------|-------|
| 01 | `learning-docs/01-getting-started.md` | Running JS in the browser console & Node, DevTools |
| 02 | `learning-docs/02-variables-and-data-types.md` | `let`/`const`, primitives, conversion, truthy/falsy |
| 03 | `learning-docs/03-operators-and-expressions.md` | Arithmetic, `===`, logical ops, `??`, ternary |
| 04 | `learning-docs/04-conditionals.md` | `if`/`else`, `switch`, guard clauses |
| 05 | `learning-docs/05-loops-and-iteration.md` | `for`, `while`, `for...of`, `break`/`continue` |
| 06 | `learning-docs/06-functions.md` | Declarations, expressions, arrows, parameters, scope |
| 07 | `learning-docs/07-arrays-and-array-methods.md` | Arrays; `map`/`filter`/`reduce` and friends |
| 08 | `learning-docs/08-objects.md` | Object literals, methods, `this`, references & copies |
| 09 | `learning-docs/09-strings-and-template-literals.md` | String methods, template literals, `split`/`join` |
| 10 | `learning-docs/10-dom-manipulation.md` | Selecting, updating, creating elements |
| 11 | `learning-docs/11-events.md` | `addEventListener`, forms, event delegation |
| 12 | `learning-docs/12-error-handling.md` | `try`/`catch`/`finally`, `throw`, custom errors |
| 13 | `learning-docs/13-closures-and-higher-order-functions.md` | Callbacks, factories, private state |
| 14 | `learning-docs/14-classes-and-prototypes.md` | `class`, inheritance, the prototype chain |
| 15 | `learning-docs/15-asynchronous-javascript.md` | Event loop, callbacks, promises, `async`/`await` |
| 16 | `learning-docs/16-fetch-and-apis.md` | `fetch`, JSON, HTTP, loading/error states |
| 17 | `learning-docs/17-modules.md` | `import`/`export`, module architecture |
| 18 | `learning-docs/18-modern-js-and-tooling.md` | Destructuring, spread/rest, npm, bundlers, what's next |

## Projects (build in order)

| # | File | Project | Difficulty |
|---|------|---------|-----------|
| 1 | `projects/01-number-guessing-game.md` | Number guessing game (console) | Beginner |
| 2 | `projects/02-tip-calculator.md` | Tip calculator (DOM + events) | Beginner+ |
| 3 | `projects/03-todo-list-app.md` | To-do list app | Intermediate− |
| 4 | `projects/04-quiz-app.md` | Quiz app with error handling | Intermediate |
| 5 | `projects/05-form-validator.md` | Interactive form validator | Intermediate |
| 6 | `projects/06-weather-dashboard.md` | Weather dashboard (fetch/async) | Intermediate+ |
| 7 | `projects/07-expense-tracker.md` | Modular expense tracker (classes + modules + localStorage) | Advanced− |
| 8 | `projects/08-capstone-recipe-box.md` | Capstone: "Recipe Box" single-page app | Advanced |

## Suggested cadence

Assuming ~5–8 focused hours per week (adjust freely — the checkpoints matter, not the calendar):

| Phase | Study | Then build | Rough time |
|-------|-------|-----------|------------|
| 1. Language core | Chapters 01–06 | **Project 1** — Number guessing game | Weeks 1–3 |
| 2. Data & text | Chapters 07–09 | (exercises only — Project 3 is coming) | Weeks 4–5 |
| 3. The browser | Chapters 10–11 | **Project 2** — Tip calculator, then **Project 3** — To-do list | Weeks 5–7 |
| 4. Robustness | Chapter 12 | **Project 4** — Quiz app | Week 8 |
| 5. Deeper functions | Chapter 13 | **Project 5** — Form validator | Weeks 9–10 |
| 6. Async & APIs | Chapters 15–16 (yes, before 14 is fine — or read 14 first, either order works after 13) | **Project 6** — Weather dashboard | Weeks 11–12 |
| 7. Structure | Chapters 14, 17 | **Project 7** — Expense tracker | Weeks 13–14 |
| 8. Polish & capstone | Chapter 18 | **Project 8** — Recipe Box capstone | Weeks 15–17 |

## How to work

- **Type every example.** Reading code teaches recognition; typing it teaches recall. Break the examples on purpose and read the errors.
- **Exercises before projects.** Each project assumes you did the exercises of its chapters.
- **Projects are specs, not tutorials.** Getting stuck is the point; the Hints sections nudge without spoiling. Budget being stuck ~20–30 minutes before looking anything up.
- **Use git from Project 2 onward.** Commit small and often — employers read commit history.
- **Finish the capstone properly** (README, deployed if possible). One polished project outweighs five abandoned ones in a portfolio.

## After the track

TypeScript → a framework (React is the most-hired) → deeper git/GitHub workflow → build and deploy 1–2 more original projects. Chapter 18's closing section has the details.
