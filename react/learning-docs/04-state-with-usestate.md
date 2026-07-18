# Chapter 4: State with useState

## Overview

State is a component's memory — data that changes over time and drives the UI. This chapter covers the `useState` hook: how to declare and type state, why updates must be immutable, how React batches updates, and the difference between passing a value and passing an updater function. This is the heart of React; take your time here.

## Definitions & Explanations

### What state is (and isn't)

A **state variable** is a value React preserves between re-renders of a component, paired with a setter that both changes the value *and* schedules a re-render. Contrast with:

- **Local variables** — recreated on every render; changes to them trigger nothing.
- **Props** — inputs owned by the parent; read-only.
- **Derived data** — anything computable from existing state/props (a filtered list, a total). Derived data should be *calculated during render*, not stored as separate state.

### The `useState` contract

```ts
const [value, setValue] = useState<Type>(initialValue);
```

- On the **first** render, `value` is `initialValue`. On later renders, it's whatever was last set.
- Calling `setValue(next)` does **not** change `value` immediately. It tells React "on the next render, this state is `next`" and schedules that render. Within the currently running function, `value` is a **snapshot** — a constant. This is the single most important mental model in React: *state is a snapshot per render*.
- TypeScript usually infers the type from the initial value. Provide an explicit type argument when it can't: `useState<User | null>(null)`, `useState<string[]>([])`.

### Functional updates

`setValue` accepts either a value or a function `prev => next`. Use the **updater function** whenever the next state depends on the previous state:

```ts
setCount(c => c + 1); // always based on the latest state, even if called 3× in a row
```

### Batching

React **batches** state updates: all `set` calls made during one event handler (and, in React 18+, timeouts/promises too) are applied together in a single re-render. So three `setCount(count + 1)` calls in one click handler produce **one** re-render and — because each read the same snapshot — a total increment of **one**. Three `setCount(c => c + 1)` calls produce one re-render and an increment of three.

### Immutability: why you must not mutate

React decides whether state "changed" using `Object.is` comparison on the state value. If you *mutate* an object or array and set it back, it's the **same reference** — React sees no change and may skip re-rendering; components receiving it as a prop and memoization also break. The rule:

> **Treat all state as read-only. To update objects/arrays, create new ones** (spread, `map`, `filter`, `toSorted`, etc. — your JS-track immutability skills apply directly).

### Lazy initialization

`useState(expensiveCompute())` runs `expensiveCompute()` on *every* render (its result is only used on the first). Pass the function itself for one-time init: `useState(() => expensiveCompute())` — e.g. reading `localStorage`.

### Where state lives

Each rendered *instance* of a component has its own state. Two `<Counter />`s are independent. State is tied to the component's **position in the tree** and its `key` — unmount it (or change its key) and the state is gone.

## Code Examples

### Counter, snapshots, and batching made visible

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  function handleTriplePlain() {
    // All three read the SAME snapshot of `count` (say 0):
    setCount(count + 1); // "set to 1"
    setCount(count + 1); // "set to 1" again
    setCount(count + 1); // "set to 1" again  → result: 1
  }

  function handleTripleFunctional() {
    // Each updater receives the latest pending value:
    setCount(c => c + 1); // 0 → 1
    setCount(c => c + 1); // 1 → 2
    setCount(c => c + 1); // 2 → 3  → result: 3, in ONE re-render
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleTriplePlain}>+3 (broken)</button>
      <button onClick={handleTripleFunctional}>+3 (correct)</button>
    </div>
  );
}
```

### Typing state that starts empty

```tsx
interface User {
  id: string;
  name: string;
}

function Profile() {
  // Inference alone would give `null` — annotate the union explicitly.
  const [user, setUser] = useState<User | null>(null);
  const [tags, setTags] = useState<string[]>([]); // not never[]!

  if (!user) return <button onClick={() => setUser({ id: '1', name: 'Ada' })}>Log in</button>;

  return (
    <div>
      <h2>{user.name}</h2> {/* narrowed: user is User here */}
      <button onClick={() => setTags(t => [...t, 'new-tag'])}>Add tag</button>
      <p>{tags.join(', ')}</p>
    </div>
  );
}
```

### Immutable updates: objects and arrays

```tsx
interface Todo { id: string; title: string; done: boolean }

function TodoApp() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [draft, setDraft] = useState('');

  function addTodo() {
    if (!draft.trim()) return;
    // ✅ new array containing a new object
    setTodos(prev => [...prev, { id: crypto.randomUUID(), title: draft.trim(), done: false }]);
    setDraft('');
  }

  function toggleTodo(id: string) {
    // ✅ new array; the changed item is a new object; others are reused as-is
    setTodos(prev => prev.map(t => (t.id === id ? { ...t, done: !t.done } : t)));
  }

  function removeTodo(id: string) {
    setTodos(prev => prev.filter(t => t.id !== id)); // ✅ filter returns a new array
  }

  // ✅ DERIVED data — computed each render, NOT stored in state
  const remaining = todos.filter(t => !t.done).length;

  return (
    <div>
      <input value={draft} onChange={e => setDraft(e.target.value)} placeholder="New todo" />
      <button onClick={addTodo}>Add</button>
      <p>{remaining} remaining</p>
      <ul>
        {todos.map(t => (
          <li key={t.id}>
            <label>
              <input type="checkbox" checked={t.done} onChange={() => toggleTodo(t.id)} />
              {t.title}
            </label>
            <button onClick={() => removeTodo(t.id)}>✕</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Lazy initialization from localStorage

```tsx
function usePersistedName() {
  // ❌ useState(localStorage.getItem('name') ?? '') reads storage every render
  // ✅ initializer function runs exactly once, on mount
  const [name, setName] = useState<string>(() => localStorage.getItem('name') ?? '');
  return [name, setName] as const;
}
```

## Common Pitfalls

1. **Expecting state to change immediately after `set`.**
   ```tsx
   function handleClick() {
     setCount(count + 1);
     console.log(count); // ❌ still the OLD value — snapshot!
   }
   ```
   If you need the next value now, compute it first: `const next = count + 1; setCount(next); use(next);`

2. **Mutating state in place.**
   ```tsx
   // ❌ Same reference — React may not re-render; memoized children definitely won't
   todos.push(newTodo);
   setTodos(todos);
   user.name = 'Bob';
   setUser(user);
   // ✅
   setTodos([...todos, newTodo]);
   setUser({ ...user, name: 'Bob' });
   ```
   Watch for hidden mutators: `push`, `pop`, `splice`, `sort`, `reverse`, and assigning to nested properties. Prefer `toSorted`/`toReversed`, or copy first.

3. **Storing derived data in state.** Keeping `remaining` in its own `useState` alongside `todos` invites desync bugs — you'll forget to update one of them. Store the *source of truth*; compute the rest during render.

4. **Stale closures with plain updates.** Any callback that captures `count` and runs later (interval, timeout, event listener) will use the value from the render when it was created. Functional updates (`setCount(c => c + 1)`) sidestep the whole class of bug.

5. **Duplicating props into state.** `const [name] = useState(props.name)` freezes the first value forever — when the parent passes a new prop, your copy doesn't update. Just use the prop; state-from-props is only for deliberate "initial value" semantics (name it `initialName` to signal that).

6. **Too many `useState`s for one concept.** Five booleans (`isLoading`, `isError`, `isSuccess`...) that must be toggled in sync is a smell — model one `status` union instead (and later, `useReducer`, Chapter 10).

## Practice Exercises

1. Build a "character counter" textarea showing `X / 280` that turns red past the limit. Which values are state and which are derived?
2. Build an accumulator with buttons `+1`, `+10`, `×2`, `Reset`, and an "Undo last" button — keep a history array of previous values in state, immutably.
3. Create a `ShoppingList` where each item has `id`, `name`, `qty`. Support add, remove, increment/decrement qty (min 1), all immutably. Show the total item count derived from the list.
4. Reproduce the batching demo: one button that calls a plain `setCount(count + 1)` three times, one that uses updater functions, plus a render counter (`console.log` in the body) to prove there's a single re-render per click.
5. Write a `Poll` component: `options: string[]` prop, a `votes` state of type `Record<string, number>` initialized lazily from the prop, click a name to vote (immutably), and show percentages derived at render time.
