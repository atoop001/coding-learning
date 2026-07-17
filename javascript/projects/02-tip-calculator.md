# Project 2: Tip Calculator (DOM + Events)

## Description

Build a web page that calculates tips and splits bills — your first real user interface. The user enters a bill amount, chooses a tip percentage (preset buttons like 10% / 15% / 20% plus a custom input), and optionally the number of people splitting the bill. The page instantly shows: tip amount, total, and per-person amounts, updating live as inputs change.

It should feel responsive and forgiving: results update as you type, silly inputs (negative bill, zero people) produce a gentle inline message rather than `NaN` splattered across the screen, and the currently selected tip button is visually highlighted.

## Difficulty & Effort

- **Difficulty:** Beginner+
- **Estimated effort:** 3–5 hours

## Chapters Used

- `02-variables-and-data-types.md` — string→number conversion, `toFixed`
- `03-operators-and-expressions.md` — the arithmetic, ternaries
- `04-conditionals.md` — validation branching
- `06-functions.md` — pure calculation functions separated from UI code
- `09-strings-and-template-literals.md` — formatted output
- `10-dom-manipulation.md` — selecting, updating text, toggling classes
- `11-events.md` — `input` and `click` events

## Requirements Checklist

- [ ] HTML page with: bill input, at least three preset tip buttons, a custom tip input, a "number of people" input (default 1), and a results area
- [ ] JS is in a separate file loaded with `defer` (or at end of body)
- [ ] Typing in the bill field updates results immediately (`input` event — no "Calculate" button needed)
- [ ] Clicking a preset tip button selects that percentage and visually highlights the active button via a CSS class (only one active at a time)
- [ ] Entering a custom tip deselects the preset buttons and uses the custom value
- [ ] Results show tip amount, total, and per-person total, all formatted to exactly 2 decimal places with a currency symbol
- [ ] All input values are explicitly converted with `Number(...)` before math
- [ ] Invalid states (empty/negative bill, people < 1, tip < 0) show a friendly message in the results area instead of numbers — and never show `NaN`
- [ ] The math lives in a pure function like `calculateTip(bill, tipPercent, people)` that returns an object `{ tip, total, perPerson }` and touches no DOM — callable and verifiable from the console
- [ ] A single `render()` (or `update()`) function is the only place that writes results into the DOM

## Hints

- Structure: read inputs → validate → calculate → render. If every event handler just calls one `update()` function that does all four steps, the app stays simple.
- For the preset buttons, give them a shared class and a `data-tip="15"` attribute; one delegated listener on their container can read `event.target.dataset.tip`. Chapter 11's delegation example is nearly this.
- "Highlight only one button": first remove the active class from *all* buttons (`querySelectorAll` + loop), then add it to the clicked one.
- An empty input's `.value` is `""`, and `Number("")` is `0` — decide deliberately whether an empty bill means 0 or means "don't show results yet." (Hint: check for `""` *before* converting.)
- Beware `people = 0` — what does dividing by it produce? Your validation should catch it before the division does.

## Stretch Goals

- **Rounding modes:** a toggle for "round per-person total up" (nobody likes owing $13.334) using `Math.ceil` on cents.
- **Reset button:** returns every field and the UI to its initial state.
- **Bill history:** store each calculation in an array and render the last 5 as a small list.
- **Keyboard support:** pressing 1/2/3 selects the preset tips; Escape resets.
- **Currency selector:** a `<select>` for $/€/£ that reformats output (later, compare with `Intl.NumberFormat`).
