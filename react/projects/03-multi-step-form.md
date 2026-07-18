# Project 3: Multi-Step Form with Validation

## Description

Build a three-step "tutoring signup" wizard: (1) personal details, (2) subject & schedule preferences, (3) review & confirm. Steps validate before advancing, going back preserves answers, and the review step shows everything typed so far. This project stresses form patterns, validation UX, lifting state up, and component design — the exact skills of most real-world front-end tickets.

## Difficulty

**Intermediate** — estimated effort: 6–10 hours.

## Chapters Used

- Chapter 4: State with useState
- Chapter 5: Events & Forms
- Chapter 6: Component Communication & Design
- (Optional stretch: Chapter 9 refs, Chapter 10 reducer)

## Requirements Checklist

### Architecture
- [ ] One parent `Wizard` component **owns all form state and the current step** — step components are controlled: they receive values + typed `onChange`-style callbacks and hold no form state of their own
- [ ] A single typed model for the whole form (e.g. `interface SignupData` with nested or flat fields — your call, defend it in a comment)
- [ ] Step components live in separate files with explicit props interfaces
- [ ] A progress indicator component (e.g. "Step 2 of 3" + step names, current one highlighted) driven purely by props

### Step 1 — Personal details
- [ ] Controlled inputs: full name, email, age (number input — remember it yields strings)
- [ ] Validation: name non-empty, plausible email (simple regex is fine), age between 5 and 120
- [ ] Errors appear only after a field is **touched** (blurred) or after attempting to advance — never on first paint

### Step 2 — Preferences
- [ ] Subject: a `<select>` over a union type (`'math' | 'science' | 'english' | 'coding'`)
- [ ] Session days: checkbox group stored as an array (or Set) of a `Day` union type
- [ ] Preferred format: radio group (`'online' | 'in-person'`)
- [ ] Notes: textarea with a live character counter, max 300, counter turns warning-colored at 250+
- [ ] Validation: at least one day selected

### Step 3 — Review & confirm
- [ ] Read-only summary of **every** answer, formatted for humans (days joined nicely, subject label not the raw value)
- [ ] Each summary section has an "Edit" button that jumps back to that step (answers intact on return)
- [ ] A terms checkbox that must be checked before the final submit enables
- [ ] Submit logs the complete typed object and shows a success screen (form gone, friendly message)

### Navigation rules
- [ ] "Next" is blocked while the current step is invalid — either disabled with visible reasons, or clickable but surfacing all errors (choose one and note why)
- [ ] "Back" always works and never loses data
- [ ] Refresh-loss is acceptable for the base project (see stretch)

### Quality
- [ ] Validation logic lives in **pure functions** (`validateStep1(data): Errors`) outside components — imaginable as unit-testable
- [ ] No `any`; errors typed as `Partial<Record<FieldName, string>>` or similar
- [ ] Labels properly associated with inputs (`<label>` wrapping or `htmlFor`) — click a label, its field focuses

## Hints

- Sketch the state shape on paper first: one `data` object + one `step` number + one `touched` structure is usually enough. Resist per-field `useState` sprawl at this scale.
- A single generic field-update helper (`update('email', value)`) typed with `keyof SignupData` keeps handlers thin — this is a nice TS generics workout.
- Derive validity at render time from the data (`const errors = validateStep(step, data)`) rather than storing errors in state — stored errors go stale.
- "Jump back to edit" is just `setStep(n)` — the data never left the parent, which is the whole point of lifting state.
- If a step component starts knowing what step number it is, or holds its own copy of a value, you've broken the controlled pattern — push it back up.

## Stretch Goals

- [ ] Persist in-progress answers to `localStorage` so refresh restores the wizard (lazy init + write-through)
- [ ] Focus management: on entering a step, focus its first field; on failed advance, focus the first invalid field (Chapter 9 refs)
- [ ] Refactor step/data transitions to a `useReducer` with actions like `field_changed`, `step_advanced`, `step_rewound`, `submitted` — enforce "can't advance while invalid" inside the reducer (Chapter 10)
- [ ] An animated progress bar and per-step slide transition (pure CSS)
- [ ] Make the wizard generic: a config array of step definitions drives the progress indicator and navigation instead of hard-coded step numbers
