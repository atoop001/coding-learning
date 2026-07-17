# Project 5: Quiz App — Strict TypeScript in the DOM

## Description

Build a browser quiz application in strict TypeScript with **no framework** — plain DOM APIs, which is precisely where TypeScript's strictness meets the messy real world of `Element | null`, `EventTarget`, and user input.

The app: a start screen (choose category and number of questions), a question screen (one multiple-choice question at a time, progress indicator, optional countdown timer per question), and a results screen (score, per-question review, play again). Question data lives in a local typed dataset (an `as const`-friendly array or a JSON file you validate on load). The app's flow is a state machine modeled as a discriminated union — the same discipline as Project 2, now driving real UI rendering.

Using it should feel crisp: keyboard-accessible answers, an obviously-correct/incorrect flash on selection, no dead buttons, and — invisibly to users but centrally to you — not a single `!`, `as any`, or unhandled null in the codebase.

## Difficulty & Effort

- **Difficulty:** Intermediate-plus
- **Estimated effort:** 8–12 hours

## Chapters Used

- `05-unions-intersections-literal-types.md` (app state machine)
- `06-narrowing-and-type-guards.md` (DOM narrowing, exhaustive rendering)
- `08-classes-in-typescript.md` (optional: timer/service classes)
- `09-utility-types.md` (Record for lookups, Readonly for the dataset)
- `10-enums-tuples-advanced-objects.md` (as const dataset, satisfies)
- `12-tooling-and-real-world-patterns.md` (core: DOM + strict mode, tooling)

## Requirements Checklist

### Setup & tooling
- [ ] Vite (or equivalent) TypeScript project; `strict: true` AND `noUncheckedIndexedAccess: true` in tsconfig
- [ ] `"lib"` includes DOM; a `check` script running `tsc --noEmit` passes clean
- [ ] Code split into modules: at minimum `types.ts`, `state.ts`, `render.ts`, `main.ts` — with `import type` used where only types cross

### Data model
- [ ] `Question` type: text, four options, index (or id) of the correct answer, category, difficulty (`"easy" | "medium" | "hard"` literal union)
- [ ] The question dataset is declared with `satisfies` (or `as const` + a checked type) so both validity is enforced AND literal inference is preserved
- [ ] App state is a discriminated union: at least `{ screen: "start" }`, `{ screen: "playing"; ...progress fields }`, `{ screen: "results"; ...answer record }` — playing-only fields must not exist on other states
- [ ] The player's answers are recorded in a typed structure that pairs question with chosen option (tuples or objects — justify the choice in a comment)

### State machine & rendering
- [ ] ONE central `render(state: AppState)` function with an exhaustive `switch` (with the `never` default) — every screen renders from state, no ad-hoc DOM mutation scattered around
- [ ] State transitions happen through pure functions (e.g., `answerQuestion(state, optionIndex): AppState`) that only accept the states they're valid in — illegal transitions are compile errors, not runtime checks
- [ ] Score and per-question review on the results screen derive from state (no separate mutable counters)

### DOM discipline (the heart of the project)
- [ ] A `mustFind<T extends Element>(selector: string): T`-style helper that throws a descriptive error — used instead of scattered null-checks or `!`
- [ ] All `querySelector` uses are element-type-specific (generic parameter or `instanceof` narrowing) — reading `.value` off a bare `Element` must not appear
- [ ] Event handlers narrow `e.target` properly (e.g., option buttons via `instanceof HTMLButtonElement` and/or `closest`) — no `as HTMLButtonElement` on targets
- [ ] Dynamic option elements are built via `createElement` (typed), not innerHTML string concatenation
- [ ] Keyboard support: keys 1–4 select answers; the key-to-option mapping is a typed `Record`
- [ ] Zero `any`, zero `!`, zero unchecked `as` (DOM-related `as` allowed ONLY if accompanied by a comment proving why narrowing was impossible — target: none)

### Behavior
- [ ] Progress indicator ("Question 3 of 10") always accurate
- [ ] Selected answer shows correct/incorrect feedback before advancing
- [ ] Results screen lists every question with the player's answer and the correct one, visually distinguished
- [ ] "Play again" fully resets to a valid start state (no ghost state from the previous round)
- [ ] If a per-question timer is included: expiry counts as incorrect and advances — timer cleanup verified (no timers firing after screen change)

## Hints

- Render-from-state means: blow away and rebuild the app container's contents on every state change. It's not efficient — it's *correct and simple*, and it's why frameworks exist. Note the pain; it's the motivation for Project 6.
- `noUncheckedIndexedAccess` will challenge you at `questions[currentIndex]` — resist `!`. Options: a `currentQuestion(state)` helper that throws descriptively, or restructure so the current question is part of the `playing` state itself. The second is more interesting.
- Store the state in one `let state: AppState` at module scope, with a single `setState(next: AppState)` that reassigns and calls `render` — a 5-line Redux.
- For click handling on options, one listener on the container using `e.target` + `closest("button")` is cleaner than N listeners — and a better narrowing exercise.
- `satisfies readonly Question[]` on the dataset catches a wrong `correctIndex` type or a missing option at author time — try breaking one entry to see.
- The timer, if built as a class: `private handle: number | null`, and every start cancels any previous — Chapter 8's encapsulation earning its keep.

## Stretch Goals

- [ ] Persist high scores to `localStorage` — with the read path treating stored JSON as `unknown` and validating (storage can contain anything; Chapter 12 applies)
- [ ] Load questions from the Open Trivia DB API instead of the local dataset, reusing the guard/validation approach from Project 4 (their payloads need decoding and shuffling)
- [ ] Category filter typed via `keyof typeof` from a categories constant, with a `Record<Category, Question[]>` index
- [ ] Add ESLint with `typescript-eslint` type-checked rules; fix everything it finds and note which rules fired
- [ ] Animate transitions between screens without breaking render-from-state purity
