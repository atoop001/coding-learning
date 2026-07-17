# Project 4: Quiz App (State Machine + Error Handling)

## Description

Build a multiple-choice quiz app. Questions live in an array of objects; the app shows one question at a time with clickable answer options, gives immediate right/wrong feedback (correct answer highlighted green, a wrong pick highlighted red), tracks the score, shows a progress indicator ("Question 3 of 10"), and ends with a results screen offering a restart.

This project adds two new muscles. First, **screen states**: the app moves through start → question → feedback → next question → results, and the UI must always match the current state. Second, **defensive programming**: your quiz data is loaded through a validator that *throws* descriptive errors for malformed questions (missing fields, no correct answer, duplicate options), and the app catches problems and degrades gracefully — skipping a bad question rather than crashing the quiz.

## Difficulty & Effort

- **Difficulty:** Intermediate
- **Estimated effort:** 6–9 hours

## Chapters Used

- `04-conditionals.md` — state transitions and feedback logic
- `06-functions.md` — one function per screen/action
- `07-arrays-and-array-methods.md` — question handling, scoring with `filter`/`reduce`
- `08-objects.md` — question objects, answer records
- `10-dom-manipulation.md` — building option buttons dynamically
- `11-events.md` — delegated answer clicks
- `12-error-handling.md` — validation with `throw`, `try/catch`, a custom error class

## Requirements Checklist

- [ ] At least 8 questions defined as objects, e.g. `{ question, options: [...], correctIndex }` (or a similar shape you document in a comment)
- [ ] A `validateQuestion(q)` function that **throws a custom error** (e.g., `class QuizDataError extends Error`) with a specific message for: missing/empty question text, fewer than 2 options, `correctIndex` out of range, or non-unique options
- [ ] On startup, all questions pass through the validator inside `try/catch`; invalid ones are excluded and logged (`console.warn`) with their error message — the quiz runs with the valid remainder
- [ ] If *zero* valid questions remain, the app shows a friendly error screen instead of starting
- [ ] Start screen with a Start button; quiz shows one question at a time with its options as buttons
- [ ] Progress indicator ("Question X of Y") and current score are always visible during play
- [ ] Clicking an option: locks further answers for that question, highlights the correct option, highlights the player's wrong pick differently (CSS classes), and updates the score
- [ ] A "Next" button appears after answering; the last question's Next leads to the results screen
- [ ] Results screen shows score, percentage, and a message tier (e.g., ≥80% / ≥50% / below) — plus per-question review: each question with ✓/✗
- [ ] Restart button resets all state (score, index, answer records) and returns to the start screen — playing twice must not double-count anything
- [ ] Answer clicks use event delegation on the options container
- [ ] All user-facing text is inserted via `textContent`

## Hints

- Model state as a handful of variables: `questions`, `currentIndex`, `score`, `answers` (array recording each pick), and maybe `phase` (`"start" | "question" | "answered" | "results"`). Write `renderCurrent()` that draws whatever the state says — resist the urge to imperatively patch the screen from many places.
- "Lock further answers" is easiest as: if this question already has an entry in `answers`, ignore clicks. That's one guard clause in the click handler.
- Build the per-question review with `map` over your `answers` array joined against `questions` — indexes line up if you push an answer record for every question.
- For the validator: collect requirements as separate `if (...) throw new QuizDataError(...)` lines. Test it deliberately by adding two intentionally-broken questions to your data and confirming the console warns and the quiz still runs.
- To check option uniqueness, compare `options.length` with `new Set(options).size`.
- Showing/hiding screens: give each screen `<section>` a class like `screen`, plus `hidden` (with `.hidden { display: none }`), and write `showScreen(id)` that hides all and reveals one.

## Stretch Goals

- **Shuffle:** randomize question order and option order per run (careful: shuffling options means recomputing which index is correct — store the correct *text* through the shuffle).
- **Timer:** 15 seconds per question; timeout counts as wrong and auto-advances.
- **Categories/difficulty:** tag questions and let the player pick a subset on the start screen.
- **High score:** keep the best score in `localStorage`.
- **Streak feedback:** show "🔥 3 in a row!" style messages driven by your answers array.
