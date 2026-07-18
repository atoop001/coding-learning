# React Learning Track

A complete, self-paced React course with TypeScript from the first line — 14 study chapters and 7 guided projects, ordered from React-beginner to job-ready. Function components and hooks only, built and tooled with Vite, aimed squarely at professional front-end employability.

## Prerequisites

This track assumes you have **completed the JavaScript and TypeScript tracks** in this workspace:

- **JavaScript**: solid modern JS — arrow functions, destructuring, spread, array methods (`map`/`filter`/`reduce`), immutability habits, promises/`async`-`await`, modules, closures.
- **TypeScript**: working knowledge — interfaces/type aliases, union and literal types, generics, discriminated unions, narrowing, strict mode.

React leans on all of it constantly (typed props, discriminated-union state, generic hooks). If any of those bullets feel shaky, revisit them first — this track does not re-teach them.

**Environment**: Windows, Node.js 20+, VS Code recommended (with the ESLint extension), and the React Developer Tools browser extension.

## Chapters (learning-docs/)

Study in order — each chapter assumes the previous ones.

| # | File | Covers |
|---|------|--------|
| 1 | `01-why-react-and-setup.md` | The problem React solves, declarative UI, Vite + React + TS scaffold, project anatomy |
| 2 | `02-jsx-and-components.md` | Rules of JSX, typing props, `children`, composition |
| 3 | `03-lists-and-conditional-rendering.md` | `.map()` rendering, keys and why they matter, conditional patterns, `&&` pitfalls |
| 4 | `04-state-with-usestate.md` | `useState`, typing state, snapshots, immutable updates, batching, derived data |
| 5 | `05-events-and-forms.md` | Typed handlers, controlled inputs, form submission, validation patterns |
| 6 | `06-component-communication.md` | Lifting state up, callback props, controlled components, slots, component API design |
| 7 | `07-useeffect-and-lifecycle.md` | Lifecycle, dependencies, cleanup, StrictMode, when NOT to use an effect |
| 8 | `08-data-fetching.md` | Fetch in effects, loading/error unions, race conditions, abort, React Query preview |
| 9 | `09-useref-custom-hooks.md` | `useRef` (DOM + instance vars), Rules of Hooks, writing custom hooks |
| 10 | `10-usereducer-complex-state.md` | Reducers, discriminated-union actions, testing reducers, undo patterns |
| 11 | `11-context-and-app-state.md` | Context + provider/hook pattern, context + reducer, when context vs props |
| 12 | `12-routing-react-router.md` | React Router: pages, params, nested routes, `Outlet`, search params, protected routes |
| 13 | `13-performance-and-quality.md` | `memo`/`useMemo`/`useCallback`, DevTools profiling, error boundaries, Vitest + Testing Library |
| 14 | `14-ecosystem-and-next-steps.md` | State libraries (Zustand/RTK), Next.js & SSR concepts, styling, deployment, CI, career next steps |

## Projects (projects/)

Guided specs with requirement checklists, hints, and stretch goals — **no solution code**; the struggle is the curriculum.

| # | File | Builds | After chapters |
|---|------|--------|----------------|
| 1 | `01-profile-card-page.md` | Static profile page from typed data — composition, lists, conditionals | 1–3 |
| 2 | `02-habit-tracker-dashboard.md` | Stateful tracker dashboard — state, immutability, forms, derived stats | 4–5 |
| 3 | `03-multi-step-form.md` | Three-step validated wizard — lifted state, controlled components | 4–6 |
| 4 | `04-kanban-task-board.md` | Kanban board on a typed reducer — actions, invariants, persistence | 7 + 10 |
| 5 | `05-api-explorer.md` | Paginated/searchable API browser — fetching, races, abort, custom hooks | 7–9 |
| 6 | `06-themed-multi-page-app.md` | Multi-page notes app — routing, context (theme/toast/auth), URL state | 11–12 |
| 7 | `07-capstone-study-tracker.md` | Full SPA: routing, API data, context, reducers, tests, **deployed** | 1–14 |

## Suggested Cadence (~10–12 weeks at 8–10 hrs/week)

| Weeks | Study | Build |
|-------|-------|-------|
| 1 | Chapters 1–3 + exercises | Project 1 |
| 2 | Chapters 4–5 + exercises | Project 2 |
| 3 | Chapter 6 + exercises | Project 3 |
| 4 | Chapter 7 (take it slow — twice is normal) | Project 3 stretch goals |
| 5 | Chapters 8–9 + exercises | Start Project 5* |
| 6 | Chapter 10 + exercises | Project 4 |
| 7 | Chapters 11–12 + exercises | Project 6 |
| 8 | Chapter 13 + exercises (testing setup!) | Add tests to Project 4 or 6 |
| 9 | Chapter 14 + exercises (deploy something small) | Capstone: spec + walking skeleton |
| 10–12 | — | Capstone build, test, deploy, README |

\* Projects 4 and 5 can be swapped — 5 follows the fetching chapters, 4 follows the reducer chapter; do them in whichever order matches your reading.

Rhythm that works: read a chapter → do its exercises in a scratch Vite app → build the project *without* re-reading first, consulting chapters only when stuck. Recall beats recognition.

## Rules of Engagement

1. **Type everything.** No `any`, strict mode always. TypeScript is your fastest reviewer.
2. **Do the exercises.** Chapters teach concepts; your fingers learn React.
3. **No copy-paste from AI/tutorials for project requirements.** Reference docs are fine; the point of each checklist item is that *you* produce it.
4. **Keep StrictMode and the `react-hooks` lint rules on.** They catch the exact bugs the chapters warn about.
5. **Finish and deploy the capstone.** One shipped, tested, explained app outweighs five abandoned repos in a job search.

## After This Track

Chapter 14 maps the road: TanStack Query in depth, a state library (Zustand), Tailwind + a component library, then Next.js — plus accessibility, CI, and monitoring as standing professional habits. Your capstone README's architecture section becomes your interview script.
