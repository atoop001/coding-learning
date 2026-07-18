# Chapter 6: Component Communication & Design

## Overview

A single component with state is easy. Real apps are trees of dozens of components that must share and coordinate state. This chapter covers the fundamental patterns: one-way data flow, **lifting state up**, callback props, `children`-based composition, and how to decide *where state should live* — the everyday architecture skill of a React developer.

## Definitions & Explanations

### One-way data flow

Data in React flows **down** the tree via props. Changes flow **up** via callback props (functions passed down that children call). There is no two-way binding. This makes data movement traceable: to find where a value can change, find the component that owns its state.

### Lifting state up

When two components need the same changing data, the state must live in their **closest common ancestor**. The ancestor owns the state and passes:

- the *value* down to whoever displays it, and
- a *callback* down to whoever changes it.

The children become simpler — often stateless. This is "lifting state up," and it's the answer to at least half of all "how do I get X from component A to component B?" questions.

### Controlled components (the general concept)

Chapter 5 showed controlled *inputs*. The same idea generalizes: any component can be **controlled** (parent owns the state; component receives `value` + `onChange`-style props) or **uncontrolled** (component owns its own state internally). Library-quality components often support both. When designing, ask: *does the parent need to know or influence this state?* If yes → controlled.

### State colocation (the flip side of lifting)

Lift state **as high as necessary, but as low as possible**. State used by only one component should stay in that component — hoisting everything to `App` makes every keystroke re-render the whole app and turns `App` into a god component. When state gets too high, push it back down ("colocation").

### Children as a composition tool

Passing `children` (and other `ReactNode` "slot" props) isn't just convenience — it's an architecture tool:

- **Avoids prop drilling**: instead of passing `user` through `Layout → Sidebar → UserPanel`, the top component composes `<Layout sidebar={<UserPanel user={user} />} />`. `Layout` never touches `user`.
- **Performance side-benefit**: children created by a parent are *the same element objects* across the wrapper's own re-renders, so a stateful wrapper re-rendering doesn't re-render composed-in children.

### Designing component APIs

Props are your component's public API. Guidelines:

- Name event props `onXxx` (`onSelect`, `onRemove`) and pass the *minimal* data the parent needs (`onRemove(id)`, not the whole event).
- Accept the narrowest data that suffices (`title: string`, not `todo: Todo`, if only the title is used) — narrower props mean fewer reasons to re-render and easier reuse.
- Prefer a `variant`/`status` union prop over many booleans (`variant: 'primary' | 'ghost'` beats `primary?: boolean; ghost?: boolean` which allows impossible combinations).
- Spread-through props for wrappers: `interface ButtonProps extends ComponentPropsWithoutRef<'button'> { variant: Variant }` lets your `Button` accept everything a real `<button>` does.

## Code Examples

### Lifting state up: filter shared by two siblings

```tsx
import { useState } from 'react';

interface Course { id: string; title: string; level: 'beginner' | 'advanced' }

// Child 1: writes the state via a callback — stateless itself.
function LevelFilter({ level, onChange }: {
  level: Course['level'] | 'all';
  onChange: (level: Course['level'] | 'all') => void;
}) {
  return (
    <select value={level} onChange={e => onChange(e.currentTarget.value as Course['level'] | 'all')}>
      <option value="all">All levels</option>
      <option value="beginner">Beginner</option>
      <option value="advanced">Advanced</option>
    </select>
  );
}

// Child 2: reads the (derived) result — also stateless.
function CourseList({ courses }: { courses: Course[] }) {
  if (courses.length === 0) return <p>No courses match.</p>;
  return <ul>{courses.map(c => <li key={c.id}>{c.title}</li>)}</ul>;
}

// Parent: the closest common ancestor OWNS the state.
function CourseBrowser({ courses }: { courses: Course[] }) {
  const [level, setLevel] = useState<Course['level'] | 'all'>('all');

  const visible = level === 'all' ? courses : courses.filter(c => c.level === level);

  return (
    <section>
      <LevelFilter level={level} onChange={setLevel} />
      <CourseList courses={visible} />
    </section>
  );
}
```

Note the shape: value down (`level`), callback down (`onChange`), derived data down (`visible`). Neither child has state; both stay trivially testable and reusable.

### Callback props carrying data up

```tsx
interface Item { id: string; name: string }

function ItemRow({ item, onRemove }: { item: Item; onRemove: (id: string) => void }) {
  return (
    <li>
      {item.name}
      {/* Child reports WHAT happened with minimal data; parent decides what it means */}
      <button onClick={() => onRemove(item.id)} aria-label={`Remove ${item.name}`}>✕</button>
    </li>
  );
}

function ItemManager() {
  const [items, setItems] = useState<Item[]>([
    { id: 'a', name: 'Alpha' }, { id: 'b', name: 'Beta' },
  ]);

  return (
    <ul>
      {items.map(item => (
        <ItemRow key={item.id} item={item}
                 onRemove={id => setItems(prev => prev.filter(i => i.id !== id))} />
      ))}
    </ul>
  );
}
```

### Controlled vs uncontrolled component (an accordion)

```tsx
// Uncontrolled: owns its own open/closed state. Simple, but parent can't coordinate panels.
function Accordion({ title, children }: { title: string; children: React.ReactNode }) {
  const [open, setOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setOpen(o => !o)}>{title} {open ? '▾' : '▸'}</button>
      {open && <div>{children}</div>}
    </div>
  );
}

// Controlled: parent owns which panel is open → "only one open at a time" is possible.
function AccordionC({ title, open, onToggle, children }: {
  title: string; open: boolean; onToggle: () => void; children: React.ReactNode;
}) {
  return (
    <div>
      <button onClick={onToggle}>{title} {open ? '▾' : '▸'}</button>
      {open && <div>{children}</div>}
    </div>
  );
}

function Faq() {
  const [openId, setOpenId] = useState<string | null>(null);
  const toggle = (id: string) => setOpenId(cur => (cur === id ? null : id));
  return (
    <>
      <AccordionC title="What is React?" open={openId === 'a'} onToggle={() => toggle('a')}>…</AccordionC>
      <AccordionC title="Why TypeScript?" open={openId === 'b'} onToggle={() => toggle('b')}>…</AccordionC>
    </>
  );
}
```

### Slots instead of prop drilling

```tsx
function AppShell({ nav, children }: { nav: React.ReactNode; children: React.ReactNode }) {
  // AppShell knows layout, nothing about users or pages.
  return (
    <div className="shell">
      <nav className="shell__nav">{nav}</nav>
      <main className="shell__main">{children}</main>
    </div>
  );
}

function App() {
  const [user] = useState({ name: 'Ada' });
  return (
    <AppShell nav={<UserMenu name={user.name} />}>
      <h1>Dashboard for {user.name}</h1>
    </AppShell>
  );
}
// `user` never passed through AppShell — no drilling.

function UserMenu({ name }: { name: string }) {
  return <span>Signed in as {name}</span>;
}
```

## Common Pitfalls

1. **Duplicating shared state in two components.** Two siblings each keeping their own copy of "selected item" and trying to sync them with effects is a bug factory. One owner, lifted to the common ancestor — always.

2. **Copying props into state to "edit" them.** `const [name, setName] = useState(props.name)` silently ignores later prop changes. If a child must edit a draft of parent data, either make it fully controlled, or name the prop `initialX` and accept that it's initial-only (optionally use `key` on the child to reset it when the source changes).

3. **Lifting everything to `App`.** Global-by-default state means maximal re-renders and tangled code. Colocate; lift only when a second component genuinely needs the value.

4. **Passing setters with vague names.** `<Child setStuff={setStuff} />` couples the child to the parent's state shape. Prefer intent-revealing callbacks: `onAdd(item)`, `onToggle(id)` — the parent maps intent to state changes.

5. **Boolean prop explosion.** `<Button primary large danger>` allows nonsense combinations. Use unions: `variant: 'primary' | 'danger'`, `size: 'sm' | 'lg'`.

6. **Children re-created by inline definitions.** Defining a component *inside* another component (`function Parent() { function Inner() {...} ... }`) creates a brand-new component type each render — React unmounts and remounts it, destroying its state. Always define components at module top level.

## Practice Exercises

1. Build a `TabGroup` where the parent owns `activeTab`, tab buttons are one child component, and the panel is another. Then add keyboard support (←/→ moves the active tab) without moving any state.
2. Take a two-component page — a `SearchBox` and a `ResultsList`, each currently owning private state — and refactor to lift the query into the parent. The list should show live-filtered results from a hard-coded array.
3. Design a `ConfirmDialog` component two ways: uncontrolled (a `trigger` prop renders the button that opens it) and controlled (`open` + `onClose` props). Use both in a demo page and note which felt better for what.
4. Build a star-rating widget as a fully controlled component (`value: number`, `onChange: (n: number) => void`, `max?: number`). Use it twice on one page bound to two different state variables to prove it holds no state of its own.
5. You have `<App> → <Page> → <Toolbar> → <ExportButton>` and `ExportButton` needs `documentTitle` from `App`. Sketch three solutions (drilling, slot composition, context — you'll meet context in Chapter 11) and write a sentence on the trade-off of each.
