# Chapter 3: Rendering Lists & Conditional Rendering

## Overview

Real UIs are mostly "show this collection" and "show this only when...". This chapter covers turning arrays into elements with `.map()`, the `key` prop and *why* it matters (not just that React yells at you), and the idiomatic patterns for conditional rendering — including the sharp edges of `&&`.

## Definitions & Explanations

### Lists: JSX elements are just values

Since JSX produces values, an **array of JSX elements** renders naturally. The idiom is `array.map(item => <Component ... />)` — you already know `.map()` from the JS track; React adds nothing new except the `key` requirement.

### `key`: React's identity tag for list items

When a list re-renders, React must match each *new* item against the *old* items to decide what to reuse, move, add, or remove. The `key` prop is the identity it uses for that matching.

- Keys must be **stable** (same item ⇒ same key across renders), **unique among siblings**, and ideally come from your data (`item.id`).
- Keys are consumed by React — they are *not* passed to your component as a prop.
- **Why index-as-key is dangerous:** if you use the array index and the array is reordered, filtered, or has items inserted/removed at the front, the keys no longer follow the *items* — they follow the *positions*. React then matches old item state (like the text typed into an input, or which row is expanded) to the wrong item. Data appears to "jump" between rows. Index is only acceptable for lists that never reorder, never filter, and have no per-item state — and even then an id is safer.
- A `key` change also has a constructive use: changing a component's key forces React to **remount** it (throw away its state and start fresh).

### Conditional rendering patterns

React has no special directive syntax (no `v-if`); you use TypeScript itself:

| Pattern | Use when |
|---|---|
| Early `return null` | The whole component may render nothing |
| `condition ? <A /> : <B />` | Two alternatives |
| `condition && <A />` | Render-or-nothing (beware falsy numbers — see pitfalls) |
| `if`/`else` assigning to a variable before `return` | Logic too complex for inline ternaries |
| A lookup object/map keyed by a union type | Many variants (status → element) |

`null`, `undefined`, `false`, and `true` render **nothing**. But `0` and `NaN` render as text — the classic `&&` bug.

### Typing the data

List UIs start with a typed model. Define an interface for the item, type the array, and let inference flow through `.map()`. Discriminated unions (from the TS track) shine for "state of the screen" rendering: `{ status: 'loading' } | { status: 'error'; message: string } | { status: 'ready'; items: Item[] }`.

## Code Examples

### Basic list with proper keys

```tsx
interface Task {
  id: string;        // stable identity from the data — this becomes the key
  title: string;
  done: boolean;
}

function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul>
      {tasks.map(task => (
        // key goes on the OUTERMOST element returned by map
        <li key={task.id} className={task.done ? 'done' : ''}>
          {task.title}
        </li>
      ))}
    </ul>
  );
}
```

### Extracting the item component (key stays at the call site)

```tsx
function TaskRow({ task }: { task: Task }) {
  // Note: there is no `key` prop here — key is for React, not for you.
  return <li className={task.done ? 'done' : ''}>{task.title}</li>;
}

function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul>
      {/* key belongs where the array is mapped, on the component itself */}
      {tasks.map(task => <TaskRow key={task.id} task={task} />)}
    </ul>
  );
}
```

### Conditional rendering, all the idioms

```tsx
interface InboxProps {
  user: { name: string } | null;
  unread: number;
  error?: string;
}

function Inbox({ user, unread, error }: InboxProps) {
  // 1. Early return: nothing to show at all
  if (!user) return null;

  // 2. if/else into a variable: complex branch kept out of the JSX
  let banner: React.ReactNode = null;
  if (error) {
    banner = <p role="alert" className="error">{error}</p>;
  } else if (unread > 99) {
    banner = <p className="notice">Over 99 unread — consider inbox zero?</p>;
  }

  return (
    <section>
      {banner}
      <h2>Welcome back, {user.name}</h2>

      {/* 3. Ternary: two alternatives */}
      {unread > 0
        ? <p>You have {unread} unread messages.</p>
        : <p>Inbox zero. Nice.</p>}

      {/* 4. && guard — safe here because the left side is a real boolean */}
      {unread > 0 && <button>Mark all read</button>}
    </section>
  );
}
```

### Variant lookup with a union type

```tsx
type Status = 'idle' | 'loading' | 'success' | 'error';

const statusUi: Record<Status, React.ReactNode> = {
  idle:    <p>Ready when you are.</p>,
  loading: <p aria-busy="true">Loading…</p>,
  success: <p className="ok">Saved ✔</p>,
  error:   <p role="alert">Something went wrong.</p>,
};

function SaveIndicator({ status }: { status: Status }) {
  return <div className="save-indicator">{statusUi[status]}</div>;
}
```

### Filtering + empty states (a pattern you'll write weekly)

```tsx
function CompletedTasks({ tasks }: { tasks: Task[] }) {
  const completed = tasks.filter(t => t.done);

  // Always design the empty state explicitly — blank space looks broken.
  if (completed.length === 0) {
    return <p className="muted">Nothing completed yet — keep going!</p>;
  }

  return (
    <ol>
      {completed.map(t => <li key={t.id}>{t.title}</li>)}
    </ol>
  );
}
```

## Common Pitfalls

1. **Index as key on a dynamic list.**
   ```tsx
   // ❌ Deleting the first todo makes every remaining input show the wrong text
   {todos.map((todo, i) => <TodoRow key={i} todo={todo} />)}
   // ✅ Identity from data
   {todos.map(todo => <TodoRow key={todo.id} todo={todo} />)}
   ```
   If your data truly has no id, generate one **when the item is created** (`crypto.randomUUID()`), never during render.

2. **Generating keys during render.** `key={Math.random()}` or `key={crypto.randomUUID()}` in the `.map()` gives every item a *new* identity each render — React destroys and rebuilds every row, killing state and performance.

3. **The `&&` zero bug.**
   ```tsx
   // ❌ When items.length is 0, this renders the text "0"
   {items.length && <ItemList items={items} />}
   // ✅ Make the left side an actual boolean
   {items.length > 0 && <ItemList items={items} />}
   ```

4. **Key on the wrong element.** Putting `key` on an inner element inside the mapped fragment/element does nothing. It must be on the top-level thing returned from the `.map()` callback. Mapping to fragments? Use the long form: `<Fragment key={item.id}>…</Fragment>`.

5. **Non-unique keys.** Using `item.name` when two items can share a name causes silent mis-rendering. Unique per sibling list, always.

6. **Deeply nested ternaries.** Three levels of `? :` inside JSX is unreadable. Promote to `if`/`else` with variables, an early return, or a lookup map.

## Practice Exercises

1. Given `interface Country { code: string; name: string; population: number }`, render a table of countries sorted by population (descending), with correct keys. Add a "no data" empty state.
2. Build a `Tag` filter UI: an array of tags, each clickable-looking (no state yet — accept a `selected: string[]` prop) where selected tags render with a different class. Which value did you use for keys, and why is it safe?
3. Write a `TrafficLight` component taking `light: 'red' | 'amber' | 'green'` that renders three circles with only the active one lit — once using ternaries, then refactored to a `Record` lookup. Which reads better?
4. Deliberately reproduce the index-as-key bug: hard-code an array of three items each rendering an `<input defaultValue={item.title} />`, use index keys, and render a button that (for now) just logs — then in Chapter 4 revisit this with real deletion and watch the inputs desync.
5. Model a screen with the discriminated union `{ status: 'loading' } | { status: 'error'; message: string } | { status: 'ready'; names: string[] }` and render all three branches exhaustively — use TypeScript's `never` trick so adding a fourth status is a compile error.
