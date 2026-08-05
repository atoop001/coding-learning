# Project 5: Interactive Form Validator

## Description

Build a signup form with professional-grade, real-time validation — the kind of polish that separates portfolio projects from tutorials. Fields: username, email, password, confirm-password, and an age/date-of-birth field. Each field validates as the user interacts with it, showing specific inline error messages ("Password needs at least one number") and success styling, while a live password-strength meter responds to every keystroke. The submit button stays disabled until the whole form is valid; a successful submit shows a summary instead of reloading the page.

The engineering focus: validation rules are **data + pure functions**, not a tangle of if-statements inside event handlers. Each rule is a small function; each field has a list of rules; one generic engine runs any field's rules and reports the failures. This rule-driven design uses closures and higher-order functions in a way you'll recognize later in every form library.

## Difficulty & Effort

- **Difficulty:** Intermediate
- **Estimated effort:** 6–9 hours

## Chapters Used

- `06-functions.md` — pure predicate functions
- `07-arrays-and-array-methods.md` — running rule lists with `filter`/`map`/`every`
- `09-strings-and-template-literals.md` — string checks (`length`, `includes`, `trim`), message building
- `10-dom-manipulation.md` — classes, messages, meter rendering
- `11-events.md` — `input`, `blur` (focus loss), `submit`
- `12-error-handling.md` — collected validation results
- `13-closures-and-higher-order-functions.md` — **rule factories** like `minLength(8)` returning validator functions

## Requirements Checklist

- [ ] Form with username, email, password, confirm-password, and age (or birth date) fields, plus a disabled-by-default submit button
- [ ] Validators are **pure functions** taking a value and returning either `null`/`true` for pass or an error-message string for fail (pick one convention and use it everywhere) — zero DOM access inside them
- [ ] At least two validators are produced by **factory functions** using closures — e.g., `minLength(n)`, `maxLength(n)`, `mustContain(charSet, label)` — so the same factory configures rules for different fields
- [ ] Each field is described by a config object mapping it to an array of rule functions; one generic `validateField(value, rules)` returns all failing messages
- [ ] Username: 3–20 chars, no spaces. Email: contains `@` with characters before/after and a `.` after the `@` (simple checks are fine — no regex required). Password: ≥8 chars, at least one digit, at least one uppercase. Confirm: matches password. Age: 13+
- [ ] Validation runs on `blur` the first time a field is visited, and on every `input` after a field has shown an error (so users aren't scolded mid-typing but errors clear promptly)
- [ ] Failing fields get an error class + the *specific* first failing message displayed next to the field; passing fields get a success class
- [ ] Password strength meter (weak/medium/strong) updates on every keystroke, scored from length and character variety, rendered as text and a colored bar
- [ ] The confirm-password field re-validates when the *password* field changes too
- [ ] Submit button enabled only when every field passes; on submit, `preventDefault` and render a success summary of the entered values (mask the password)
- [ ] The whole rules setup is testable from the console: `validateField("ab", usernameRules)` returns the expected messages without any page interaction

## Hints

- Start with zero UI: define the factories and rule lists, then in the console run values through `validateField` until the messages are right. The DOM wiring afterwards is mostly plumbing.
- A rule factory is Chapter 13's `makeMultiplier` in disguise: a function that takes the configuring value (like `n`) and returns *another* function that closes over it and does the real work later. Non-validator reminder of the shape: `const multiplyBy = (n) => (x) => x * n;` — `multiplyBy(3)` hands you back a one-argument function that remembers `3` forever, so `multiplyBy(3)(5)` is `15`. `minLength(n)` has the identical skeleton — outer function captures `n`, inner function takes the real argument (here, the field's `value`) and returns the pass/fail result. Revisit Chapter 13 if the closure part feels shaky.
- "Has a digit" without regex: `[..."0123456789"].some(d => value.includes(d))` — and that generalizes into a `mustContain(chars, label)` factory.
- Track per-field UI state (e.g., `touched`) in an object keyed by field name, so you know whether to validate on `input` yet.
- For "form fully valid," reuse the same machinery: run every field's rules and check `every(...)` — don't maintain a separate `isValid` flag that can drift out of sync.
- Strength score idea: +1 for length ≥ 8, +1 for ≥ 12, +1 for a digit, +1 for an uppercase, +1 for a symbol → map score ranges to weak/medium/strong.

## Stretch Goals

- **Debounced username availability:** simulate an async "is this username taken?" check with a Promise + `setTimeout` (peek at Chapter 15), showing a spinner state.
- **Shake animation** on attempting to submit an invalid form (CSS class + `animationend` cleanup).
- **Rule composition:** an `optional(rule)` wrapper that passes empty values through, for non-required fields.
- **Accessibility pass:** wire errors with `aria-describedby` / `aria-invalid` and test tab-only navigation.
- **Show/hide password toggle** with the eye-icon pattern.
