# Project 2: Habit Tracker Dashboard

## Description

Build a small dashboard for tracking daily habits (e.g. "code 1 hour", "read", "exercise"): add habits, log completions with counter buttons, see live-derived stats, and filter the view. This is your first fully stateful app — the project drills `useState`, immutable updates, derived data, events, and controlled inputs.

## Difficulty

**Beginner+** — estimated effort: 5–8 hours.

## Chapters Used

- Chapters 1–3 (assumed fluency)
- Chapter 4: State with useState
- Chapter 5: Events & Forms

## Requirements Checklist

### Data & state
- [ ] `interface Habit { id: string; name: string; target: number; completions: number }` (or similar — design it yourself and defend it in a comment)
- [ ] All habits in a single `useState<Habit[]>` in the top-level component
- [ ] Ids generated with `crypto.randomUUID()` **at creation time**, never during render
- [ ] Every update is immutable — no `push`, `splice`, or property mutation anywhere

### Adding habits (controlled form)
- [ ] A form with a controlled text input (habit name) and a controlled number input (daily target, min 1, max 20)
- [ ] Submission on the **form's** `onSubmit` with `preventDefault` (Enter must work)
- [ ] Validation: empty/whitespace names rejected, duplicate names (case-insensitive) rejected, with an inline error message that clears when the user fixes the input
- [ ] The form resets after a successful add

### Tracking interactions
- [ ] Each habit row shows name, a progress indication (`completions / target`), and `+` / `−` buttons
- [ ] `−` cannot take completions below 0; `+` can exceed the target (over-achieving is allowed and styled differently)
- [ ] Both buttons use **functional updates** (`prev => ...`)
- [ ] A "reset day" button sets all completions to 0 (one immutable pass)
- [ ] A delete button per habit, removing it immutably

### Derived data (computed in render — zero extra state)
- [ ] Stat tiles: total habits, habits completed today (completions ≥ target), overall completion percentage
- [ ] A "best habit" highlight (highest completions/target ratio) — handle the empty-list case
- [ ] A filter control (`all | done | in-progress`) as one state variable; the filtered list is derived, not stored

### Quality
- [ ] Correct keys on all lists; no console warnings
- [ ] Empty state when there are no habits, and a separate "no matches" state when the filter hides everything
- [ ] `npm run build` passes clean

## Hints

- Decide which values are *state* vs *derived* before coding: the moment you're tempted to `useState` a total, stop — it's derived.
- The `+`/`-` handlers need the habit's id. `onClick={() => increment(habit.id)}` — remember why `onClick={increment(habit.id)}` is a bug.
- One generic update helper `updateHabit(id: string, patch: (h: Habit) => Habit)` used by `+`, `−`, and reset keeps the immutable `map` logic in one place.
- For duplicate-name validation, derive the check from existing state at submit time — don't maintain a parallel "names" array.
- If your stat tiles disagree with the list after some interaction, you almost certainly mutated something — check for hidden mutations (`sort` is a classic).

## Stretch Goals

- [ ] Persist habits to `localStorage` using **lazy state initialization** for the read (writing back can be a plain call inside the update helper for now — you'll formalize this with an effect in Chapter 7)
- [ ] Add a per-habit streak counter and an "end day" button that archives today's numbers into a `history: number[]` on each habit (immutably) before resetting
- [ ] Batching demo: an "auto-fill day" dev button that completes every habit to target — implement it once with a naive loop of plain `set` calls, observe the bug, then fix with functional updates and write a comment explaining the difference
- [ ] Sort controls (by name, by progress) using non-mutating sorts (`toSorted`)
