# Chapter 5: Events & Forms

## Overview

Events are how users change state; forms are where most of that happens. This chapter covers React's typed event system, controlled inputs (the pattern behind every text field, checkbox, and select you'll write), form submission, and basic validation patterns — all fully typed.

## Definitions & Explanations

### React events

You attach handlers with camelCase props: `onClick`, `onChange`, `onSubmit`, `onKeyDown`, `onBlur`. React hands your handler a **SyntheticEvent** — a cross-browser wrapper with the same API as native events (`preventDefault`, `stopPropagation`, `target`, `currentTarget`). Events propagate (bubble) through the *component* tree just like the DOM.

Key TypeScript types (all generic over the element type):

| Handler | Event type |
|---|---|
| `onChange` on `<input>`/`<textarea>`/`<select>` | `ChangeEvent<HTMLInputElement>` (or `HTMLTextAreaElement`, `HTMLSelectElement`) |
| `onSubmit` on `<form>` | `FormEvent<HTMLFormElement>` |
| `onClick` | `MouseEvent<HTMLButtonElement>` |
| `onKeyDown` | `KeyboardEvent<HTMLInputElement>` |
| `onFocus`/`onBlur` | `FocusEvent<HTMLInputElement>` |

You rarely write these by hand — **inline handlers get the types inferred**. Extract a handler to a named function and you annotate the parameter once. `e.target` vs `e.currentTarget`: `currentTarget` is the element the handler is attached to and is precisely typed; prefer it.

### Passing handlers correctly

`onClick={handleClick}` passes the function. `onClick={handleClick()}` **calls it during render** — a classic bug (often an infinite render loop when the handler sets state). To pass arguments, wrap in an arrow: `onClick={() => remove(id)}`.

### Controlled vs uncontrolled inputs

- **Controlled input** — React state is the single source of truth: `value={text}` + `onChange={e => setText(e.currentTarget.value)}`. The input displays exactly what state says; every keystroke flows through state. This enables instant validation, formatting, disabling submit, character counts, etc. *Default choice in this track.*
- **Uncontrolled input** — the DOM holds the value; you read it when needed (via `defaultValue` and a ref, Chapter 9, or `FormData` on submit). Less code for simple fire-and-forget forms.
- Checkboxes are controlled via `checked` (boolean), not `value`. Selects via `value` on the `<select>` itself. Number inputs still yield **strings** — convert deliberately.

### Form submission

Always handle submission on the `<form>`'s `onSubmit` (not the button's `onClick`) and call `e.preventDefault()` to stop the browser's full-page navigation. This keeps Enter-to-submit working and is better for accessibility.

### Validation patterns

- Validate on submit for simple forms; validate on blur or on change for richer UX.
- Keep an `errors` object in state, typed as `Partial<Record<FieldName, string>>`.
- Only show a field's error after it's been *touched* (blurred once) or after a submit attempt — nagging users about a field they haven't reached yet is bad UX.

### Accessibility basics

Accessibility (a11y) isn't a separate feature bolted on later — for the events and forms in this chapter it's mostly a byproduct of using the right element for the job:

- **Semantic elements over `<div onClick>`.** A `<button>` is focusable, keyboard-operable (Enter/Space fire it), and announced correctly by screen readers, all for free. A `<div onClick={...}>` gives you none of that — you'd have to bolt on `tabIndex={0}`, `role="button"`, and an `onKeyDown` handler for Enter/Space just to match what `<button>` already does natively. Default to the native interactive element (`button`, `a`, `input`, `select`); reach for ARIA roles only when no native element fits.
- **Every input needs a label.** Either wrap the input in a `<label>` (as this chapter's examples do) or pair an explicit `htmlFor`/`id`. An input with only a `placeholder` is not labeled — screen readers don't treat placeholder text as a label, and it vanishes the moment the user starts typing.
- **Keyboard reachability.** Every interactive element — buttons, links, form controls, custom widgets — must be reachable via Tab and operable without a mouse. If you can't complete a form using only Tab, Shift+Tab, Enter, and Space, something's wrong: a missing `tabIndex`, a click-only handler, or a non-interactive element pretending to be a control.
- **`getByRole` as a free audit.** Chapter 13's Testing Library queries (`getByRole('button', { name: /save/i })`) only find elements with an accessible role and name — the same information a screen reader relies on. If you can't query your component by role or label text, that's usually a sign a real assistive-tech user can't operate it either.

## Code Examples

### Controlled text input, minimal

```tsx
import { useState } from 'react';

function NameField() {
  const [name, setName] = useState('');

  return (
    <label>
      Name
      {/* value + onChange = controlled. State is the source of truth. */}
      <input
        value={name}
        onChange={e => setName(e.currentTarget.value)}
      />
      <span>{name.length} chars</span>
    </label>
  );
}
```

### Every input kind, one form object

```tsx
import { useState, type ChangeEvent, type FormEvent } from 'react';

interface SignupForm {
  email: string;
  role: 'student' | 'tutor';
  hoursPerWeek: string;   // number inputs still give strings — parse on submit
  acceptTerms: boolean;
}

const initialForm: SignupForm = {
  email: '',
  role: 'student',
  hoursPerWeek: '5',
  acceptTerms: false,
};

function Signup() {
  const [form, setForm] = useState<SignupForm>(initialForm);

  // One generic change helper for text-ish inputs, keyed by field name.
  function updateField(e: ChangeEvent<HTMLInputElement | HTMLSelectElement>) {
    const { name, value } = e.currentTarget;
    setForm(prev => ({ ...prev, [name]: value })); // immutable merge
  }

  function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault(); // stop full-page reload
    const hours = Number(form.hoursPerWeek); // deliberate conversion
    console.log('submitting', { ...form, hoursPerWeek: hours });
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Email
        <input name="email" type="email" value={form.email} onChange={updateField} />
      </label>

      <label>
        Role
        <select name="role" value={form.role} onChange={updateField}>
          <option value="student">Student</option>
          <option value="tutor">Tutor</option>
        </select>
      </label>

      <label>
        Hours/week
        <input name="hoursPerWeek" type="number" min={1} max={40}
               value={form.hoursPerWeek} onChange={updateField} />
      </label>

      <label>
        {/* Checkbox: controlled via `checked`, read via `checked` */}
        <input type="checkbox" checked={form.acceptTerms}
               onChange={e => setForm(p => ({ ...p, acceptTerms: e.currentTarget.checked }))} />
        I accept the terms
      </label>

      {/* Derived validity disables submit — no extra state needed */}
      <button type="submit" disabled={!form.acceptTerms || form.email === ''}>
        Sign up
      </button>
    </form>
  );
}
```

### Validation with touched fields

```tsx
type Errors = Partial<Record<'email' | 'password', string>>;

function validate(email: string, password: string): Errors {
  const errors: Errors = {};
  if (!/^\S+@\S+\.\S+$/.test(email)) errors.email = 'Enter a valid email.';
  if (password.length < 8) errors.password = 'At least 8 characters.';
  return errors;
}

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [touched, setTouched] = useState<{ email?: boolean; password?: boolean }>({});
  const [submitted, setSubmitted] = useState(false);

  // Errors are DERIVED — recomputed every render, never stored stale.
  const errors = validate(email, password);
  const show = (field: keyof Errors) => (touched[field] || submitted) && errors[field];

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setSubmitted(true);
    if (Object.keys(errors).length === 0) {
      console.log('valid — send it');
    }
  }

  return (
    <form onSubmit={handleSubmit} noValidate>
      <label>
        Email
        <input type="email" value={email}
               onChange={e => setEmail(e.currentTarget.value)}
               onBlur={() => setTouched(t => ({ ...t, email: true }))}
               aria-invalid={Boolean(show('email'))} />
      </label>
      {show('email') && <p role="alert">{errors.email}</p>}

      <label>
        Password
        <input type="password" value={password}
               onChange={e => setPassword(e.currentTarget.value)}
               onBlur={() => setTouched(t => ({ ...t, password: true }))} />
      </label>
      {show('password') && <p role="alert">{errors.password}</p>}

      <button type="submit">Log in</button>
    </form>
  );
}
```

### Callback props: child events, parent handlers (preview of Chapter 6)

```tsx
interface SearchBoxProps {
  onSearch: (query: string) => void; // typed callback prop
}

function SearchBox({ onSearch }: SearchBoxProps) {
  const [q, setQ] = useState('');
  return (
    <form onSubmit={e => { e.preventDefault(); onSearch(q.trim()); }}>
      <input value={q} onChange={e => setQ(e.currentTarget.value)} placeholder="Search…" />
    </form>
  );
}
```

## Common Pitfalls

1. **Calling instead of passing.**
   ```tsx
   <button onClick={save()}>Save</button>      // ❌ runs during render
   <button onClick={save}>Save</button>        // ✅
   <button onClick={() => save(id)}>Save</button> // ✅ with args
   ```

2. **`value` without `onChange`.** React makes the input read-only and warns. If you truly want an initial-but-editable value, that's `defaultValue` (uncontrolled). Don't mix `value` and `defaultValue`.

3. **A controlled input whose value can become `undefined`.** Going from `undefined` to a string flips the input from uncontrolled to controlled mid-life (React warns, cursor jumps). Initialize string state as `''`, never `undefined`.

4. **Submitting via button `onClick`.** You lose Enter-to-submit and native form semantics. Put logic in `<form onSubmit>` and remember `preventDefault`. Also: a `<button>` inside a form defaults to `type="submit"` — an innocent "clear" button will submit the form unless you mark it `type="button"`.

5. **Trusting `type="number"` to give numbers.** `e.currentTarget.value` is a string (and can be `''`). Keep it as string state; convert with `Number()` at the boundary where you need math.

6. **Showing all errors immediately.** A form that's covered in red before the user types anything feels hostile. Gate errors behind touched/submitted as in the example.

## Practice Exercises

1. Build a `TemperatureConverter` with two controlled inputs (Celsius and Fahrenheit) that stay in sync — typing in either updates the other. Decide: one state variable or two? Justify.
2. Build a "add tag" input: type a tag, press Enter to add it to a list (no duplicates, trimmed, no empties), click a tag to remove it. Use `onKeyDown` and proper keys.
3. Extend the Signup example with a `password` + `confirmPassword` pair and a derived "passwords match" validation shown only after both fields are touched.
4. Build a pizza order form: size (radio group), toppings (checkbox group stored as `string[]`), quantity (number), delivery notes (textarea, max 200 chars with live count). Log a fully typed order object on submit.
5. Rewrite exercise 4's submission using the uncontrolled approach — `FormData` in the submit handler, no per-field state — and write two sentences on when you'd prefer each style.
6. Pick a component from a project you've already built (or one of this chapter's examples). Unplug your mouse and try to complete its main task using only Tab, Shift+Tab, Enter, and Space. Note every place you got stuck, and fix at least one.
