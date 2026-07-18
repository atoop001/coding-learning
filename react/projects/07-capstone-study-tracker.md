# Project 7: Capstone — Study Tracker SPA

## Description

Build **StudyTrack**: a complete single-page application for managing your own learning across this workspace's tracks (JavaScript, TypeScript, React). Track subjects, chapters, and projects; log study sessions; visualize progress; pull in supplementary data from a public API; theme it; test it; profile it; and **deploy it publicly**. This is the portfolio piece — treat it like a job: plan, build in vertical slices, test as you go, ship, and write a README that explains your decisions.

## Difficulty

**Advanced** — estimated effort: 25–40 hours over 2–4 weeks.

## Chapters Used

All of them — Chapters 1–14. Explicit new muscle: Chapter 13 (memoization, error boundaries, testing) and Chapter 14 (deployment, ecosystem judgment).

## Requirements Checklist

### Product scope
- [ ] **Subjects & items**: CRUD for subjects (e.g. "React") each containing trackable items (chapters/projects) with a status (`'not-started' | 'in-progress' | 'done'`) — model it with typed interfaces you design and document
- [ ] **Study sessions**: log sessions (subject, minutes, date, optional note) via a validated form; sessions list with filtering by subject and date range
- [ ] **Dashboard**: derived stats — total hours, per-subject progress percentages, current streak (consecutive study days), most-studied subject — all computed from state, none stored
- [ ] **External data**: at least one public-API integration that enriches the app (e.g. programming quotes on the dashboard, GitHub repo stats for your project repos, or dictionary lookups for a glossary page) — full loading/error/empty handling, abort-safe
- [ ] **Settings**: theme (light/dark, persisted), data management (export/import all app data as JSON, plus a guarded "wipe everything")

### Architecture (document each choice in the README)
- [ ] Routing: React Router with a persistent layout, nested routes, dynamic segments (`/subjects/:subjectId`), URL-held filter state, protected route or 404 handling as appropriate
- [ ] Core domain state in `useReducer`(s) with discriminated-union actions, exhaustive switches, enforced invariants, localStorage persistence
- [ ] Context for cross-cutting concerns only (theme, toasts, and state/dispatch delivery) — throwing hooks, stable provider values
- [ ] At least three custom hooks extracted (candidates: `useLocalStorage`, `useFetch`/query wrapper, `useDebouncedValue`, `useMediaQuery`) in `src/hooks/`
- [ ] Server data handled with either your abort-safe fetch pattern **or** TanStack Query — state your choice and its trade-off in the README
- [ ] A `src/` layout you can defend (e.g. `components/`, `features/`, `hooks/`, `lib/`) — consistency matters more than the specific scheme

### Quality bar (this is what makes it a capstone)
- [ ] **Tests**: Vitest + Testing Library configured; minimum coverage: every reducer action (pure unit tests), the session form (validation + submit behavior via `user-event`), one async component across all its fetch states (mocked fetch), and one routing behavior (e.g. bad id → not-found UI). Aim for 15+ meaningful tests
- [ ] **Error boundaries**: at least route-level, with a reset path; one deliberately isolated widget boundary on the dashboard
- [ ] **Performance pass**: profile with React DevTools; fix at least one real re-render issue structurally or with the memo trio, and record before/after observations in the README
- [ ] **Accessibility basics**: semantic landmarks, labeled inputs, keyboard-operable interactions, visible focus, `role="alert"` for errors — your Testing Library queries double as the audit
- [ ] TypeScript strict, no `any`, ESLint with `react-hooks` rules clean, StrictMode on

### Shipping
- [ ] Deployed to Netlify/Vercel (or similar) with SPA fallback configured — deep links work cold
- [ ] CI (GitHub Actions): typecheck + lint + tests on every push; badge in the README
- [ ] README: screenshots, feature list, architecture decisions (state placement policy!), how to run/test locally, live URL, and a short "what I'd do next" section

## Hints

- **Plan first**: write a one-page spec — pages, routes, data model, state placement table — before any code. Then build a walking skeleton (routes + layout + deploy) in the first sitting, and deploy from day one.
- Build in vertical slices ("log a session end-to-end") rather than horizontal layers ("all reducers first") — a slice is demoable and testable; a layer is neither.
- Reducer-first for the domain: when you can simulate a week of studying purely by dispatching actions in a test file, the UI becomes straightforward.
- Steal from your own projects: Project 4's reducer discipline, Project 5's fetch machinery, Project 6's provider stack are all designed to be re-derivable here. Re-typing them from memory is the point.
- Write tests *with* features, not after — the reducer tests especially will catch refactor mistakes for the rest of the build.
- Scope discipline: the checklist is the contract. New ideas go in the README's "what I'd do next" — a finished 90% beats an abandoned 140%.

## Stretch Goals

- [ ] A calendar heat-map (GitHub-style) of study activity, built with plain divs + derived data
- [ ] Undo for destructive actions via a history-wrapped reducer, surfaced as a toast with an "Undo" button (toast + dispatch working together)
- [ ] Pomodoro timer page (refs + intervals + cleanup) that auto-logs a session on completion
- [ ] Import this workspace's actual `progress.json` files format as a supported import source
- [ ] Route-level code splitting with `lazy()`/`Suspense`; compare bundle chunks in the build output
- [ ] MSW for network mocking in tests instead of `vi.spyOn(fetch)`
- [ ] Lighthouse pass ≥90 on performance and accessibility; document what you fixed
