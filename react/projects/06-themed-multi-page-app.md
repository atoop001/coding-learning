# Project 6: Themed Multi-Page Notes App

## Description

Build "NoteSpace" — a multi-page notes application with real URLs: a notes list, per-note detail pages, tag filtering via search params, a settings section, and a fake-auth-protected area — plus app-wide theming and a toast system delivered through context. This project integrates routing and context into the state skills you already have, producing your first app that *feels* like a real product.

## Difficulty

**Advanced-intermediate** — estimated effort: 10–15 hours.

## Chapters Used

- Chapter 11: Context & App-Wide State
- Chapter 12: Routing with React Router
- (Foundation: Chapters 4–10, especially 6 and 10)

## Requirements Checklist

### Routing structure
- [ ] `react-router-dom` with a persistent layout route (header nav + `Outlet`); nav uses `NavLink` with active styling (`end` on Home)
- [ ] Routes: `/` (dashboard/recent notes), `/notes` (list), `/notes/:noteId` (detail), `/notes/new` (create form), `/settings` with **nested** children `/settings/profile` and `/settings/appearance`, a `/login` page, and a styled `*` 404
- [ ] Zero internal `<a href>` — `Link`/`NavLink`/`navigate` only
- [ ] `useParams` ids validated: unknown note ids render an in-app not-found with a link back (no crash)
- [ ] Creating a note navigates to its new detail page with `{ replace: true }`; deleting a note navigates back to the list
- [ ] Tag filter and note search live in **search params** (`/notes?tag=react&q=hooks`) — refresh, back/forward, and pasted URLs all reproduce the view; think about `replace` vs push per keystroke

### Notes state
- [ ] Notes in a `useReducer` (Chapter 10 discipline: union actions, exhaustive switch, pure, immutable) provided app-wide via context — split state and dispatch contexts
- [ ] Persistence to `localStorage` (lazy init + effect write-back)
- [ ] `interface Note` with at least id, title, body, tags, createdAt, updatedAt

### Theme context
- [ ] `ThemeProvider` + `useTheme()` with the professional pattern: `createContext<... | null>(null)` and a **throwing** hook
- [ ] Light/dark themes applied via a `data-theme` attribute + CSS custom properties; toggle lives in `/settings/appearance`, initial value respects `prefers-color-scheme`, choice persists
- [ ] The provider's `value` object is referentially stable across unrelated re-renders (memoized or state/updater split)

### Toast context
- [ ] `ToastProvider` exposing `useToast(): { show(message, tone?) }`; toasts stack, auto-dismiss with proper cleanup, and are dismissible by click
- [ ] Toasts fired from at least three places (note created, note deleted, login) — proving the value of app-wide delivery

### Fake auth + protected routes
- [ ] Auth context (Chapter 11's pattern) with an `AuthState` union including a `'pending'` status during a fake ~500ms login
- [ ] `/settings/*` requires auth: a `RequireAuth` wrapper redirects to `/login` remembering the intended destination, and login returns you there
- [ ] Logged-in user's name appears in the header; logout works from anywhere

### Quality
- [ ] Provider nesting composed into one tidy `<AppProviders>` component
- [ ] Every context consumer imports only the hook, never the context object
- [ ] StrictMode clean, keys correct, `npm run build` passes, no `any`
- [ ] Write a small Vitest test for whichever piece of theming or notes logic you consider most breakable — e.g. the notes reducer's tag-filter derivation, or `ThemeProvider`'s persisted-toggle behavior. Reducer/pure-logic pieces need no rendering; a context/component piece needs Testing Library's `render`. Chapter 13 formalizes the full stack and setup — treat this as a look-ahead using just its "Test setup" and "Testing a component like a user" sections

## Hints

- Order of construction matters: routes with hard-coded data first, then the notes reducer + context, then theme, then toasts, then auth. Each layer should work before the next starts.
- "Which state goes in the URL?" — apply the rule: if it should survive refresh or be shareable (filters, selected note), URL; if it's app data, reducer; if it's a cross-cutting service (theme, toasts, auth), context. Write your placement decisions as comments — interviewers ask exactly this.
- The tag filter UI and the notes list both derive from `searchParams` + reducer state at render time — resist mirroring URL state into `useState` (a classic double-source-of-truth bug).
- For `RequireAuth` + pending status: decide what renders while auth is `'pending'` — redirecting *during* pending is the classic flash-of-login bug.
- If toggling the theme visibly re-renders the whole app in the Profiler, that's expected (everything reads the CSS variables anyway) — but if typing in the note editor re-renders the header, check your provider value stability.

## Stretch Goals

- [ ] Note editing with unsaved-changes detection: navigating away from a dirty editor asks for confirmation
- [ ] A command palette (Ctrl+K) overlay for jumping to any note — global key listener with cleanup, focus trapped in the palette
- [ ] Sort options for the list stored in search params alongside filters, with typed parsing/defaulting of all params in one helper
- [ ] Route-level code splitting with `lazy()` + `<Suspense>` for the settings section; verify the separate chunk in the build output
- [ ] Migrate to `createBrowserRouter` with a loader for the notes list as a taste of data-router mode
