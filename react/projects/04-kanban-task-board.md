# Project 4: Kanban Task Board

## Description

Build a three-column kanban board (Backlog / In Progress / Done): create tasks, move them between columns, edit titles inline, filter and search, undo mistakes. All board logic lives in one thoroughly-typed reducer — this project is where `useReducer`, discriminated-union actions, lifted state, and component decomposition click together into real architecture.

## Difficulty

**Intermediate** — estimated effort: 8–12 hours.

## Chapters Used

- Chapter 4: State with useState (for local UI bits)
- Chapter 5: Events & Forms
- Chapter 6: Component Communication & Design
- Chapter 7: useEffect & the Component Lifecycle (persistence)
- Chapter 10: useReducer & Complex State

## Requirements Checklist

### The reducer (the heart of the project)
- [ ] Board state in a single `useReducer` at the top: tasks plus any board-level fields you design
- [ ] `interface Task` with at least: id, title, column (union type), priority (`'low' | 'med' | 'high'`), createdAt
- [ ] Actions as a discriminated union — at minimum: `task_added`, `task_moved`, `task_renamed`, `task_deleted`, `priority_changed`, `board_cleared`
- [ ] `switch` with an exhaustiveness check (`never`) so an unhandled action type fails compilation
- [ ] Invariants enforced **in the reducer only**: no empty/whitespace titles, `task_moved` to the task's current column is a no-op returning the same reference, renaming a nonexistent id returns state unchanged
- [ ] Reducer is pure: no `Date.now()`/`randomUUID()` inside it — nondeterministic values are created at dispatch time and travel in the action payload
- [ ] Zero mutations (spread/map/filter only)

### Components
- [ ] Decomposition: at least `Board`, `Column`, `TaskCard`, `NewTaskForm`, `BoardToolbar` — each in its own file with typed props
- [ ] Deep components receive `dispatch` (or thin intent callbacks) — no drilling of five separate setters
- [ ] `TaskCard` move buttons: ◀ / ▶ move one column, hidden/disabled at the ends
- [ ] Inline title editing: double-click a title to swap in a controlled input; Enter commits (dispatch), Escape cancels (local draft state stays local to the card — decide what's local vs board state and comment why)

### Filtering & derived data
- [ ] Toolbar search box (task text) and priority filter — visible tasks are **derived** at render, the reducer state never stores a filtered list
- [ ] Column headers show live counts (visible/total per column)
- [ ] Columns render an empty-state slot when they have no visible tasks

### Persistence & effects
- [ ] Board persists to `localStorage`: lazy-init via `useReducer`'s third argument, write-back in an effect with honest dependencies
- [ ] Corrupted/missing stored data falls back to a sensible initial board without crashing

### Quality
- [ ] Stable keys (task ids) everywhere; no console warnings, StrictMode on and clean
- [ ] `npm run build` passes; no `any`
- [ ] Write 2–3 Vitest tests for the reducer: at least one action and one invariant (e.g. `task_added` appends correctly, renaming a nonexistent id returns the same reference). It's a pure function — no rendering, no Testing Library needed. Chapter 13 formally teaches the Vitest/Testing Library setup; skim its "Testing components" section for the install command, but a reducer test is just `expect(reducer(state, action)).toEqual(...)` — you don't need the rest of the stack to start

## Hints

- Write the reducer and its types **first**, before any component — if you can play the whole board in your head via dispatched actions, the UI becomes a thin shell.
- Test the "no-op returns same reference" invariants informally in the console: `boardReducer(s, action) === s` should be `true` for rejected actions. This exact property makes undo (stretch) clean.
- The inline-edit draft is the classic "local vs lifted" decision: the *draft* is UI state (local `useState` in the card), the *committed title* is board state (reducer). Mixing them up causes either lost edits or dispatch-per-keystroke.
- For move buttons, a `const COLUMNS = ['backlog', 'doing', 'done'] as const` array lets you compute prev/next columns instead of hard-coding six conditionals.
- If the effect that persists to localStorage loops or fires unexpectedly, re-read Chapter 7's dependency section — the fix is never omitting the dep.

## Stretch Goals

- [ ] **Undo/redo** by wrapping your reducer in a higher-order history reducer (Chapter 10's `withUndo` pattern) with buttons that disable at the boundaries
- [ ] Drag-and-drop between columns using native HTML5 drag events (`onDragStart`/`onDragOver`/`onDrop`) dispatching the same `task_moved` action as the buttons — good architecture means DnD adds *zero* reducer changes
- [ ] A "column WIP limit" (e.g. In Progress max 3) enforced in the reducer, with the UI explaining rejected moves
- [ ] Task detail modal (due date, description) — decide: more reducer state or component-local?
- [ ] Full reducer coverage with Vitest — beyond the required 2–3, test *every* action and invariant, including reference-equality of no-ops (Chapter 13 formalizes the stack; you already started above)
