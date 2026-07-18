# Chapter 2: JSX & Components

## Overview

JSX is the markup-in-TypeScript syntax React uses to describe UI, and components are the functions that produce it. This chapter covers the rules of JSX (they're stricter than HTML), how to define and *type* component props, and how small components compose into larger UIs — the core skill of React development.

## Definitions & Explanations

### What JSX actually is

JSX looks like HTML inside TypeScript, but it is **syntax sugar for function calls**. `<h1 className="big">Hi</h1>` compiles to something like `jsx('h1', { className: 'big', children: 'Hi' })`. Two consequences:

1. JSX produces **values** (of type `ReactNode` / `JSX.Element`). You can store them in variables, return them, pass them around, put them in arrays.
2. Because it's really TypeScript, the file extension is `.tsx`, and TS type-checks your markup — misspell a prop and you get a compile error.

### The rules of JSX

- **One root element per return.** A component returns one value. To group siblings without adding a DOM node, use a **Fragment**: `<>...</>`.
- **Every tag closes.** `<img />`, `<input />`, `<br />` — self-closing is mandatory.
- **camelCase attributes.** `class` → `className`, `for` → `htmlFor`, `tabindex` → `tabIndex`, `onclick` → `onClick`. (Exceptions: `data-*` and `aria-*` keep their dashes.)
- **`{}` embeds expressions**, not statements. Any TS *expression* works: variables, function calls, ternaries, `&&`. You cannot put `if`/`for` statements inside JSX (do that logic above the `return`).
- **`style` takes an object**, not a string: `style={{ marginTop: 8, color: 'tomato' }}` — camelCased CSS properties, numbers become px.
- **Comments** inside JSX use `{/* ... */}`.

### Props: a component's inputs

**Props** (properties) are the arguments you pass to a component, written like HTML attributes. Inside the component they arrive as a single object. Props are **read-only** — a component never modifies its own props; data flows *down* the tree (one-way data flow).

In TypeScript you type props with an interface or type alias. Useful prop types you'll use constantly:

- `string`, `number`, `boolean` — plain data.
- Optional props: `subtitle?: string`, often with a default value in destructuring.
- Union types for variants: `variant: 'primary' | 'danger'`.
- `ReactNode` — "anything renderable" (elements, strings, numbers, fragments). The type of the special `children` prop.
- Function props (callbacks) — covered in Chapters 5–6.

### `children`: components that wrap things

Whatever you put *between* a component's opening and closing tags arrives as the `children` prop. This is how you build wrappers — cards, layouts, modals — that don't need to know what's inside them.

### Composition

React apps are built by **composing** small, focused components into bigger ones, exactly like composing functions. Prefer many small components with clear props over one giant one. A good heuristic: if you'd give a chunk of markup a name when describing the page aloud ("the avatar", "the stat row"), it's probably a component.

## Code Examples

### A typed component with props

```tsx
// Badge.tsx
interface BadgeProps {
  label: string;
  // Union type: only these two values compile
  tone: 'success' | 'warning';
  // Optional with a default supplied in destructuring
  count?: number;
}

function Badge({ label, tone, count = 0 }: BadgeProps) {
  return (
    <span className={`badge badge--${tone}`}>
      {label}
      {/* Render the count only when it's positive — ternary keeps it explicit */}
      {count > 0 ? <strong> ({count})</strong> : null}
    </span>
  );
}

export default Badge;
```

Usage — TypeScript catches mistakes at compile time:

```tsx
<Badge label="Tests" tone="success" count={12} />
<Badge label="Lint" tone="warning" />
{/* <Badge label="Oops" tone="error" />  ← compile error: 'error' not assignable */}
{/* <Badge tone="success" />             ← compile error: label is required   */}
```

Note the difference between `count="12"` (a **string**) and `count={12}` (a **number**). Non-string props must use braces.

### Fragments and expressions

```tsx
function UserSummary({ name, unread }: { name: string; unread: number }) {
  const greeting = unread > 0 ? `You have mail, ${name}` : `All clear, ${name}`;

  // Fragment: two siblings, no wrapper div polluting the DOM
  return (
    <>
      <h2>{greeting}</h2>
      <p>{unread} unread {unread === 1 ? 'message' : 'messages'}</p>
    </>
  );
}
```

### `children` and wrapper components

```tsx
import type { ReactNode } from 'react';

interface PanelProps {
  title: string;
  children: ReactNode; // anything renderable
}

function Panel({ title, children }: PanelProps) {
  return (
    <section className="panel">
      <h3 className="panel__title">{title}</h3>
      <div className="panel__body">{children}</div>
    </section>
  );
}

// Usage: Panel doesn't know or care what it wraps.
function Dashboard() {
  return (
    <Panel title="Today">
      <p>3 chapters remaining</p>
      <Badge label="Streak" tone="success" count={7} />
    </Panel>
  );
}
```

### Composing a realistic piece of UI

```tsx
interface Stat {  // plain data type — not a component
  label: string;
  value: number;
}

function StatItem({ label, value }: Stat) {
  return (
    <div className="stat">
      <dt>{label}</dt>
      <dd>{value}</dd>
    </div>
  );
}

function ProfileHeader({ name, title }: { name: string; title: string }) {
  return (
    <header>
      <h1>{name}</h1>
      <p className="muted">{title}</p>
    </header>
  );
}

// The page is just composition — read it top-down like an outline.
function ProfilePage() {
  return (
    <main>
      <ProfileHeader name="Ada Lovelace" title="Analyst & Programmer" />
      <dl className="stats">
        <StatItem label="Projects" value={12} />
        <StatItem label="Followers" value={340} />
      </dl>
    </main>
  );
}
```

## Common Pitfalls

1. **`class` instead of `className`.** The most common day-one error. TS will flag it; in plain JS it silently half-works, which is worse.

2. **Returning multiple root elements.**
   ```tsx
   // ❌ Doesn't compile — two roots
   return (
     <h1>Title</h1>
     <p>Body</p>
   );
   // ✅ Wrap in a fragment
   return (
     <>
       <h1>Title</h1>
       <p>Body</p>
     </>
   );
   ```

3. **Trying to put statements in `{}`.** `{ if (x) { ... } }` is invalid. Compute above the return, or use a ternary/`&&` expression.

4. **Mutating props.** `props.count++` (or reassigning a destructured prop and expecting the UI to change) does nothing useful — props are read-only inputs. If a value needs to change over time, it's *state* (Chapter 4), owned by some component and passed down.

5. **Forgetting braces for non-string props.** `<Badge count="3" />` passes the string `"3"`. TypeScript saves you here — another reason the `-ts` template matters.

6. **One mega-component.** A 300-line `App.tsx` full of nested divs is a design smell. Extract named components early; props are the seams.

7. **Rendering objects directly.** `<p>{user}</p>` where `user` is an object throws at runtime ("Objects are not valid as a React child"). Render fields: `{user.name}`.

## Practice Exercises

1. Build a `PriceTag` component with props `amount: number`, `currency: 'USD' | 'EUR' | 'GBP'`, and optional `onSale?: boolean` (default `false`). When on sale, render the word "SALE" next to the price. Format the number with `toLocaleString`.
2. Build a `MediaObject` layout component that takes `image: ReactNode` and `children: ReactNode` and renders them side by side. Use it twice with different content to prove it's generic.
3. Recreate a GitHub-style repo card (name, description, language dot, star count) as a composition of at least three sub-components, each with typed props. No state, hard-code the data.
4. Take this invalid JSX and fix every problem without changing what it displays: `return (<div class="box"><img src={url}><p>Items: {if (n > 0) { n } }</p></div><footer>end</footer>);`
5. Write a `Layout` component with **two** `ReactNode` props — `sidebar` and `children` — and use it to render a page. When is a named "slot" prop better than `children` alone?
