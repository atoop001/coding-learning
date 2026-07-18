# Chapter 10: useReducer & Complex State

## Overview

When one piece of state has many *kinds* of updates — or several `useState` variables must always change together — scattered setters become fragile. `useReducer` centralizes all update logic in one pure function driven by typed actions. This chapter covers the reducer pattern, typing actions as discriminated unions, choosing between `useState` and `useReducer`, and modeling state so invalid states can't exist.

## Definitions & Explanations

### The reducer pattern

A **reducer** is a pure function `(state, action) => newState`:

- **State** — the single object describing this feature right now.
- **Action** — a plain object describing *what happened* (`{ type: 'added', text: '...' }`), not *how state should change*. Past-tense event names (`'added'`, `'moved'`) keep this framing honest.
- **Dispatch** — the function components call to send an action: `dispatch({ type: 'added', text })`.

```ts
const [state, dispatch] = useReducer(reducer, initialState);
```

React calls your reducer with the current state and the dispatched action; the return value becomes the next state (triggering a re-render, with all `useState` rules intact: immutability, batching, snapshots).

### Why bother?

- **Centralized transitions**: every way the state can change is in one readable function, instead of scattered across a dozen handlers. New requirement "completing a task also clears its edit draft"? One place to enforce it.
- **Impossible states become unwritable**: the reducer is a chokepoint where you enforce invariants.
- **Components get dumber**: they translate user intent into actions; they don't know how state updates. Deep children can receive `dispatch` alone (it's stable — never changes identity) instead of five callbacks.
- **Testable**: a reducer is a pure function — test it with plain assertions, no rendering.

### Typing reducers in TypeScript

Type actions as a **discriminated union** — this is where your TS-track knowledge pays off hard:

```ts
type Action =
  | { type: 'added'; text: string }
  | { type: 'toggled'; id: string }
  | { type: 'removed'; id: string };
```

In the `switch (action.type)`, each case narrows `action` to exactly the right payload, and dispatching a misspelled type or wrong payload is a compile error. Add an exhaustiveness check (`default: return assertNever(action)` or just `satisfies never`) so forgetting a case fails the build.

### `useState` vs `useReducer` — the decision

Use `useState` when: independent values, simple updates, form fields, toggles.
Prefer `useReducer` when:

- Multiple state variables must change **together** (or you keep writing three `set` calls in a row).
- The **next state depends on the previous** in nontrivial ways (moving items, undo).
- The same update logic is triggered from **many places**.
- You're passing many update callbacks down; passing one `dispatch` is cleaner.
- You want the logic **unit-tested**.

They're not rivals — a page commonly has `useReducer` for its core model and `useState` for a stray input.

## Code Examples

### A task-board reducer, fully typed

```tsx
import { useReducer } from 'react';

interface Task {
  id: string;
  title: string;
  column: 'todo' | 'doing' | 'done';
}

interface BoardState {
  tasks: Task[];
  filter: string;
}

type BoardAction =
  | { type: 'task_added'; title: string }
  | { type: 'task_moved'; id: string; to: Task['column'] }
  | { type: 'task_removed'; id: string }
  | { type: 'filter_changed'; filter: string }
  | { type: 'board_cleared' };

const initialState: BoardState = { tasks: [], filter: '' };

// Pure function: no fetches, no Date.now()/random ideally passed in via action,
// no mutation — always return new objects for changed parts.
function boardReducer(state: BoardState, action: BoardAction): BoardState {
  switch (action.type) {
    case 'task_added': {
      const title = action.title.trim();
      if (!title) return state; // invariant enforced in ONE place
      const task: Task = { id: crypto.randomUUID(), title, column: 'todo' };
      return { ...state, tasks: [...state.tasks, task] };
    }
    case 'task_moved':
      return {
        ...state,
        tasks: state.tasks.map(t => (t.id === action.id ? { ...t, column: action.to } : t)),
      };
    case 'task_removed':
      return { ...state, tasks: state.tasks.filter(t => t.id !== action.id) };
    case 'filter_changed':
      return { ...state, filter: action.filter };
    case 'board_cleared':
      return initialState;
    default: {
      // Exhaustiveness: if a new action type is added and unhandled, this errors at compile time.
      const _exhaustive: never = action;
      return _exhaustive;
    }
  }
}

function Board() {
  const [state, dispatch] = useReducer(boardReducer, initialState);

  // Derived data still computed at render — reducers don't change that rule.
  const visible = state.tasks.filter(t =>
    t.title.toLowerCase().includes(state.filter.toLowerCase()),
  );

  return (
    <div>
      <input
        placeholder="Filter…"
        value={state.filter}
        onChange={e => dispatch({ type: 'filter_changed', filter: e.currentTarget.value })}
      />
      <button onClick={() => dispatch({ type: 'task_added', title: 'New task' })}>Add</button>

      {(['todo', 'doing', 'done'] as const).map(col => (
        <section key={col}>
          <h3>{col}</h3>
          <ul>
            {visible.filter(t => t.column === col).map(t => (
              <li key={t.id}>
                {t.title}
                {col !== 'done' && (
                  <button onClick={() => dispatch({ type: 'task_moved', id: t.id, to: 'done' })}>
                    ✓
                  </button>
                )}
                <button onClick={() => dispatch({ type: 'task_removed', id: t.id })}>✕</button>
              </li>
            ))}
          </ul>
        </section>
      ))}
    </div>
  );
}
```

### Testing the reducer (no React needed)

```ts
import { describe, expect, it } from 'vitest';

describe('boardReducer', () => {
  it('trims titles and rejects empty ones', () => {
    const s1 = boardReducer(initialState, { type: 'task_added', title: '  hi  ' });
    expect(s1.tasks[0].title).toBe('hi');
    const s2 = boardReducer(initialState, { type: 'task_added', title: '   ' });
    expect(s2).toBe(initialState); // same reference: nothing changed
  });

  it('moves the right task', () => {
    const s1 = boardReducer(initialState, { type: 'task_added', title: 'a' });
    const s2 = boardReducer(s1, { type: 'task_moved', id: s1.tasks[0].id, to: 'done' });
    expect(s2.tasks[0].column).toBe('done');
  });
});
```

### Undo with a history reducer (state shape doing the work)

```tsx
interface Undoable<T> {
  past: T[];
  present: T;
  future: T[];
}

type UndoAction<A> = { type: 'undo' } | { type: 'redo' } | { type: 'do'; action: A };

// A higher-order reducer: wraps ANY reducer with undo/redo.
function withUndo<T, A>(reducer: (s: T, a: A) => T) {
  return (state: Undoable<T>, action: UndoAction<A>): Undoable<T> => {
    switch (action.type) {
      case 'undo': {
        if (state.past.length === 0) return state;
        const previous = state.past[state.past.length - 1];
        return { past: state.past.slice(0, -1), present: previous,
                 future: [state.present, ...state.future] };
      }
      case 'redo': {
        if (state.future.length === 0) return state;
        const [next, ...rest] = state.future;
        return { past: [...state.past, state.present], present: next, future: rest };
      }
      case 'do': {
        const next = reducer(state.present, action.action);
        if (next === state.present) return state; // no-op actions don't pollute history
        return { past: [...state.past, state.present], present: next, future: [] };
      }
    }
  };
}
```

This is the payoff of pure reducers: undo/redo required *zero* changes to `boardReducer`.

### Lazy initialization

```tsx
// Third argument = init function, receives the second argument.
const [state, dispatch] = useReducer(
  boardReducer,
  'board-v1',                        // passed to init
  (key): BoardState => {
    const raw = localStorage.getItem(key);
    return raw ? (JSON.parse(raw) as BoardState) : initialState;
  },
);
```

## Common Pitfalls

1. **Mutating state inside the reducer.** `state.tasks.push(task); return state;` returns the same reference — React sees "no change" and skips the re-render. Reducers obey exactly the same immutability rules as `useState`.

2. **Side effects in the reducer.** Fetching, `localStorage.setItem`, dispatching other actions, or even `new Date()` inside a reducer breaks purity (and misbehaves under StrictMode's double-invoke). Effects go in `useEffect`/handlers; nondeterminism (ids, timestamps) is fine at *dispatch time*, passed in the action payload — the id example above bends this rule; putting `crypto.randomUUID()` in the action payload is the stricter habit.

3. **Setter-shaped actions.** `{ type: 'set_tasks', tasks }` recreates `useState` with extra steps and moves logic back into components. Actions describe *events* (`'task_added'`); the reducer decides state changes.

4. **Forgetting a switch case.** Without an exhaustiveness check, an unhandled action silently returns... whatever falls through. Always `default` with a `never` assertion (and return `state` in JS-interop code).

5. **One global mega-reducer.** A reducer holding the modal flag, the theme, the search text, and the tasks re-renders everything for any change and reads like a god object. Reducers should own one cohesive feature's state; multiple `useReducer`s are fine.

6. **Async in the reducer.** Reducers are synchronous. "Fetch then update" = fetch in an effect/handler, then `dispatch({ type: 'loaded', data })`.

## Practice Exercises

1. Convert Chapter 4's `TodoApp` from three `useState`s to one `useReducer` with actions `added`, `toggled`, `removed`, `clearedCompleted`. Enforce "no empty titles" in the reducer only.
2. Write Vitest tests for that reducer covering every action plus one unknown-input edge (e.g., toggling a non-existent id returns unchanged state — assert reference equality).
3. Build a multi-step wizard reducer: state `{ step: 1 | 2 | 3; answers: Record<string, string> }`, actions `next`, `back`, `answered`. Make it impossible (in the reducer) to go past step 3, below 1, or `next` while the current step's answer is empty.
4. Wrap exercise 1's reducer with the `withUndo` higher-order reducer and add Undo/Redo buttons that disable correctly at history boundaries.
5. Take a component of yours (or invent one) with 4+ `useState` calls that update together, and refactor to `useReducer`. In a comment block, list which pitfalls from this chapter the old version risked.
