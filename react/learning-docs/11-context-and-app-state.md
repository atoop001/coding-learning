# Chapter 11: Context & App-Wide State

## Overview

Props work brilliantly until a value is needed *everywhere* — the theme, the signed-in user, the current locale. Threading those through every intermediate component ("prop drilling") clutters code with props that components merely pass along. **Context** lets a parent make a value available to any depth of descendant. This chapter covers creating and typing context, the provider pattern, context + reducer for app state, performance characteristics, and — importantly — when *not* to use context.

## Definitions & Explanations

### What context is

Context is a **broadcast channel down a subtree**: a `Provider` supplies a value; any descendant can read it with `useContext`. It is *not* a state manager — it's a transport. The state still lives in normal `useState`/`useReducer` somewhere; context just delivers it without drilling.

Mechanics:

- `createContext<T>(defaultValue)` creates the channel. The default is used only when a component reads context **with no provider above it** — usually a bug, which is why the nullable-context pattern below turns it into a loud error.
- `<MyContext.Provider value={x}>` (or in React 19, `<MyContext value={x}>`) supplies `x` to descendants.
- `useContext(MyContext)` reads the nearest provider's value. **Whenever that value changes (by `Object.is`), every component reading the context re-renders.** This sentence drives all context performance advice.

### The professional pattern: provider component + custom hook

Raw context use scatters `useContext(ThemeContext)!` and null checks everywhere. Instead, wrap both ends:

1. A **provider component** (`<ThemeProvider>`) that owns the state and renders the `Provider`.
2. A **custom hook** (`useTheme()`) that reads the context and *throws* if there's no provider — converting a silent bug into an immediate, well-described crash.

Consumers then import one hook and never touch the context object. TypeScript stays clean: create the context as `createContext<ThemeContextValue | null>(null)` and let the hook narrow away the `null`.

### Context + reducer = lightweight app state

For genuinely app-wide state (auth session, cart, notifications), the standard recipe is `useReducer` in a provider, exposing `state` and `dispatch` via context — often as **two separate contexts**, because many components only dispatch (dispatch is stable → they never re-render from state changes).

### When context — and when not

Good fits: theme, locale, auth/current user, feature flags, toast API, values that are *read widely but change rarely*.

Poor fits:

- **Fast-changing state read by few components** (form inputs, mouse position) — every change re-renders all readers.
- **Avoiding two levels of props.** Drilling through one or two components is fine — it's explicit and refactorable. Context adds indirection; don't pay that cost for minor convenience. Also consider **component composition** (slot props, Chapter 6) which often removes the drilling entirely.
- **Server data** — that's a cache problem; React Query solves it better (Chapter 8).

When context + reducer starts feeling heavy (selectors, many stores, performance tuning), that's the cue for a dedicated library like Zustand or Redux Toolkit (Chapter 14) — same concepts, better ergonomics.

## Code Examples

### Theme context, the full professional pattern

```tsx
// theme.tsx
import { createContext, useContext, useState, type ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextValue {
  theme: Theme;
  toggleTheme: () => void;
}

// null default → reading without a provider is detectable
const ThemeContext = createContext<ThemeContextValue | null>(null);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>(() =>
    // respect the OS preference on first load
    window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light',
  );

  const toggleTheme = () => setTheme(t => (t === 'light' ? 'dark' : 'light'));

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// The ONLY way consumers touch this context.
export function useTheme(): ThemeContextValue {
  const ctx = useContext(ThemeContext);
  if (ctx === null) {
    throw new Error('useTheme must be used inside <ThemeProvider>');
  }
  return ctx; // narrowed to ThemeContextValue — no ! anywhere
}
```

```tsx
// App.tsx — provider at the top; any descendant can read
import { ThemeProvider, useTheme } from './theme';

function ThemeToggle() {
  const { theme, toggleTheme } = useTheme(); // no props needed, any depth
  return <button onClick={toggleTheme}>Switch to {theme === 'light' ? 'dark' : 'light'}</button>;
}

function App() {
  return (
    <ThemeProvider>
      <Shell>            {/* Shell never mentions theme — no drilling */}
        <ThemeToggle />
      </Shell>
    </ThemeProvider>
  );
}

function Shell({ children }: { children: React.ReactNode }) {
  const { theme } = useTheme();
  return <div className={`app app--${theme}`}>{children}</div>;
}
```

### Auth: context + reducer, split state/dispatch contexts

```tsx
// auth.tsx
import { createContext, useContext, useReducer, type Dispatch, type ReactNode } from 'react';

interface User { id: string; name: string }

type AuthState =
  | { status: 'anonymous' }
  | { status: 'authenticated'; user: User };

type AuthAction =
  | { type: 'logged_in'; user: User }
  | { type: 'logged_out' };

function authReducer(_state: AuthState, action: AuthAction): AuthState {
  switch (action.type) {
    case 'logged_in':  return { status: 'authenticated', user: action.user };
    case 'logged_out': return { status: 'anonymous' };
  }
}

// Two channels: readers of state vs senders of actions re-render independently.
const AuthStateContext = createContext<AuthState | null>(null);
const AuthDispatchContext = createContext<Dispatch<AuthAction> | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(authReducer, { status: 'anonymous' });
  return (
    <AuthStateContext.Provider value={state}>
      {/* dispatch is identity-stable → this provider's value NEVER changes */}
      <AuthDispatchContext.Provider value={dispatch}>
        {children}
      </AuthDispatchContext.Provider>
    </AuthStateContext.Provider>
  );
}

export function useAuth(): AuthState {
  const ctx = useContext(AuthStateContext);
  if (ctx === null) throw new Error('useAuth must be used inside <AuthProvider>');
  return ctx;
}

export function useAuthDispatch(): Dispatch<AuthAction> {
  const ctx = useContext(AuthDispatchContext);
  if (ctx === null) throw new Error('useAuthDispatch must be used inside <AuthProvider>');
  return ctx;
}
```

```tsx
// Consumers pick what they need:
function LoginButton() {
  const dispatch = useAuthDispatch(); // never re-renders on auth changes
  return (
    <button onClick={() => dispatch({ type: 'logged_in', user: { id: '1', name: 'Ada' } })}>
      Log in
    </button>
  );
}

function Greeting() {
  const auth = useAuth(); // re-renders when auth changes — as it should
  if (auth.status === 'anonymous') return <p>Welcome, guest.</p>;
  return <p>Hello, {auth.user.name}!</p>; // union narrowing again
}

// Route guarding preview (Chapter 12 makes this real):
function RequireAuth({ children }: { children: React.ReactNode }) {
  const auth = useAuth();
  if (auth.status !== 'authenticated') return <LoginButton />;
  return <>{children}</>;
}
```

### Keeping provider values stable

```tsx
// ❌ New object every render → ALL consumers re-render whenever
//    ThemeProvider's parent re-renders, even if theme didn't change.
<ThemeContext.Provider value={{ theme, toggleTheme }}>

// ✅ Memoize the value object (useMemo/useCallback detailed in Chapter 13)
const value = useMemo(() => ({ theme, toggleTheme }), [theme]);
<ThemeContext.Provider value={value}>
```

(When the provider re-renders *only because its own state changed* — like our `ThemeProvider`, whose parent is static — this matters less; make it a habit anyway for providers that sit under changing parents.)

## Common Pitfalls

1. **Meaningful `createContext` defaults.** `createContext<ThemeContextValue>({ theme: 'light', toggleTheme: () => {} })` compiles happily, and a missing provider silently no-ops your toggle for hours of confused debugging. Use `null` + throwing hook.

2. **Inline provider `value` objects under re-rendering parents.** As above — a fresh `{}` each render defeats the change check. Memoize the object, or provide state and dispatch separately.

3. **One giant AppContext.** Theme + user + cart + modals in one context means changing *anything* re-renders *everything* that reads *any* of it. Split by concern; nest the providers (composing them into an `<AppProviders>` component keeps `main.tsx` tidy).

4. **Context as a reflex against any drilling.** Passing a prop through two layers is not a problem to engineer away. Reach for composition first, context when the value is genuinely cross-cutting.

5. **High-frequency values in context.** Keystroke-level or animation-level data through context = app-wide re-render storms. Keep it local, or use a store library with selector subscriptions.

6. **Provider placed too low.** A `RequireAuth` in the header can't see an `AuthProvider` wrapping only the main panel. Providers must wrap *every* consumer — usually near the root for app-wide concerns.

## Practice Exercises

1. Implement the full `ThemeProvider`/`useTheme` pattern, persist the choice with your Chapter 9 `useLocalStorage` hook, and apply the theme by setting `data-theme` on `<html>` in an effect. Confirm the throwing hook works by briefly rendering a consumer outside the provider.
2. Build a `ToastProvider` exposing `useToast(): { show: (msg: string) => void }`. Toasts stack, auto-dismiss after 4s (cleanup!), and can be dismissed by click. Which of state/dispatch do consumers of `show` actually need?
3. Extend `AuthProvider` with a fake async login (`setTimeout` 500ms): add a `'pending'` status to the union and disable the login button while pending. What changed in the reducer, actions, and consumers?
4. Take a 4-level drilling scenario (`App → Layout → Sidebar → UserBadge` needing `user`) and solve it three ways: drilling, slot composition, context. Time yourself; write which you'd ship and why.
5. Build a `LocaleProvider` (`'en' | 'es' | 'fr'`) with a `useT()` hook returning a typed translate function `(key: 'greeting' | 'farewell') => string` backed by a `Record<Locale, Record<Key, string>>` dictionary. Add a language `<select>` that lives three components away from the text it changes.
