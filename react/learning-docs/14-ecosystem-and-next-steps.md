# Chapter 14: The Ecosystem & Next Steps

## Overview

You now know React itself. This final chapter maps the landscape around it: state-management libraries and when they're warranted, frameworks like Next.js (conceptually — what server rendering changes), styling options, deploying a Vite app for real, and a professional-readiness checklist to guide what you learn next. Less code, more judgment — the skill interviews probe.

## Definitions & Explanations

### State management beyond context

You already have a decision ladder; libraries extend it, not replace it:

1. **Local state** (`useState`/`useReducer`) — default; most state should stay here.
2. **Lifted state / composition** — for shared state among close relatives.
3. **URL state** (router) — for anything shareable/bookmarkable.
4. **Server cache** (**TanStack Query**) — for anything fetched. Most apps' "global state problem" is 80% server data; solving it with a cache removes most of the need for a store.
5. **Client store libraries** — for genuinely global, fast-changing *client* state (complex editors, carts, cross-cutting UI state):
   - **Zustand** — tiny hook-based store; components subscribe via selectors so only readers of a changed slice re-render (fixing context's broadcast problem). The pragmatic default today.
   - **Redux Toolkit (RTK)** — the modern, batteries-included Redux: your reducer skills transfer directly (actions, immutable updates via Immer, devtools time-travel). Common in large/older codebases; worth reading even if you don't choose it.
   - **Jotai / others** — atom-based fine-grained state; niche but growing.

Interview-ready summary: *"Local first, server state in a query cache, URL for shareable state, and a small store like Zustand only for real cross-cutting client state."*

### Form libraries

The controlled-input pattern from Chapter 5 (state + `onChange` + a hand-rolled `errors`/`touched` object) works great at the scale you've been building. Past roughly five fields, cross-field validation, conditional fields, or nested arrays, most teams reach for **React Hook Form** instead: it manages field state internally via uncontrolled refs (far fewer re-renders per keystroke) while still exposing a simple `register()`/`handleSubmit()` API, and it's almost always paired with **Zod** for schema-based validation — one schema defines the rules *and* infers the TypeScript type (`z.infer<typeof schema>`), so the shape can't drift from the validation. Recognition level for this track: know the two names, know why teams reach for them (less boilerplate than what you wrote by hand in Chapter 5), and know they compose (`zodResolver` wires a Zod schema into React Hook Form) — not mastery.

### Frameworks and server rendering, conceptually

Your Vite SPA renders entirely in the browser: the server sends an empty HTML shell + JS. Trade-offs: simple hosting, great for apps behind login; but slower first paint and weak SEO for content sites. Frameworks address this:

- **SSR (server-side rendering)** — the server renders HTML for the first load; the browser then **hydrates** it (attaches React so it becomes interactive).
- **SSG (static site generation)** — pages pre-rendered at build time; served from a CDN.
- **RSC (React Server Components)** — components that run *only* on the server (can read databases directly, ship zero JS); interleaved with `'use client'` interactive components. This is React's current architectural direction.
- **Next.js** — the dominant React framework: file-based routing, RSC by default, server actions for mutations, image/font optimization. **Everything in this track transfers** — client components in Next.js are exactly the React you've learned.
- **React Router (framework mode) / Remix, TanStack Start, Astro** — alternatives with the same fundamental ideas (loaders/actions for data, progressive enhancement, islands).

When asked "SPA or framework?": content-heavy/public/SEO → framework; internal tools and dashboards → a Vite SPA is a perfectly professional answer.

### Styling options (know the categories)

- **Plain CSS / CSS Modules** (`Button.module.css`) — scoped classes, zero runtime, built into Vite. Solid default.
- **Tailwind CSS** — utility classes in JSX; extremely common in job listings; pairs with component libraries like **shadcn/ui** + Radix for accessible primitives.
- **CSS-in-JS** (styled-components, Emotion) — declining for new work (runtime cost, RSC friction) but present in existing codebases.

### Deploying a Vite app

`npm run build` produces `dist/` — plain static files. Any static host serves them. Two things always need attention:

1. **SPA fallback**: every route must rewrite to `index.html` (Chapter 12's deploy-404 pitfall).
2. **Environment variables**: Vite inlines `import.meta.env.VITE_*` at *build* time — they are public, never secrets. Secrets belong on a backend or serverless proxy.

### Professional-readiness checklist

Beyond this track, employable front-end work adds: **accessibility** (semantic HTML, keyboard support, ARIA only when needed — Testing Library has been nudging you here already); **linting/formatting** (ESLint with `react-hooks` rules + Prettier, enforced in CI); **git hygiene + PR reviews**; **basic CI** (typecheck + lint + test on push); **error monitoring** (Sentry-class tooling); and enough **HTTP/browser fundamentals** (caching, CORS, cookies vs tokens) to debug real integrations.

## Code Examples

### Zustand in thirty seconds

```tsx
// npm install zustand
import { create } from 'zustand';

interface CartState {
  items: { id: string; qty: number }[];
  add: (id: string) => void;
  clear: () => void;
}

const useCart = create<CartState>()(set => ({
  items: [],
  add: id => set(s => {
    const existing = s.items.find(i => i.id === id);
    return {
      items: existing
        ? s.items.map(i => (i.id === id ? { ...i, qty: i.qty + 1 } : i))
        : [...s.items, { id, qty: 1 }],
    };
  }),
  clear: () => set({ items: [] }),
}));

// Selector subscription: this component re-renders ONLY when the count changes.
function CartBadge() {
  const count = useCart(s => s.items.reduce((n, i) => n + i.qty, 0));
  return <span className="badge">{count}</span>;
}

// No provider needed; any component anywhere can read/write.
function AddButton({ id }: { id: string }) {
  const add = useCart(s => s.add);
  return <button onClick={() => add(id)}>Add to cart</button>;
}
```

Note what carried over: immutable updates, functional set, derived data in selectors — the concepts were the hard part and you already own them.

### Next.js, recognizably (reading knowledge)

```tsx
// app/courses/[id]/page.tsx — a SERVER component: async, no hooks, zero client JS.
// File location IS the route: /courses/42
export default async function CoursePage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const course = await fetchCourse(id);      // runs on the server — can hit the DB
  return (
    <main>
      <h1>{course.title}</h1>
      <EnrollButton courseId={course.id} />  {/* interactive island */}
    </main>
  );
}
```

```tsx
// app/courses/[id]/enroll-button.tsx
'use client';                                // ← everything below is the React you know
import { useState } from 'react';

export function EnrollButton({ courseId }: { courseId: string }) {
  const [enrolled, setEnrolled] = useState(false);
  return (
    <button onClick={() => setEnrolled(true)} disabled={enrolled}>
      {enrolled ? 'Enrolled ✔' : 'Enroll'}
    </button>
  );
}
```

### Environment variables in Vite

```ts
// .env.local  (gitignored)          .env.production
// VITE_API_URL=http://localhost:3001    VITE_API_URL=https://api.example.com

// src/api.ts — typed access; remember: BUILD-time inlined, PUBLIC.
const API_URL: string = import.meta.env.VITE_API_URL;
export const getJson = (path: string) => fetch(`${API_URL}${path}`).then(r => {
  if (!r.ok) throw new Error(`HTTP ${r.status}`);
  return r.json();
});
```

### Deploying to Netlify / Vercel / GitHub Pages

```powershell
npm run build          # emits dist/
npm run preview        # sanity-check the production bundle locally
```

```text
# Netlify — public/_redirects (copied into dist/ automatically):
/*  /index.html  200

# Vercel — vercel.json:
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }

# GitHub Pages — vite.config.ts needs base: '/<repo-name>/',
# and use HashRouter OR a 404.html fallback trick for deep links.
```

Both Netlify and Vercel also do it with zero config when you connect the git repo: they detect Vite, run the build, deploy `dist/`, and give you preview deploys per PR — set this up once and every push ships.

### CI in one file (quality gate)

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run build      # includes tsc type-checking
      - run: npx eslint src
      - run: npx vitest run
```

## Common Pitfalls

1. **Reaching for Redux/Zustand on day one.** Most apps need none of it — local state + React Query + URL state covers the majority. Adding a store "because real apps have one" is résumé-driven architecture; be able to *justify* every layer.

2. **Putting server data in a client store.** Copying fetch results into Zustand/Redux recreates caching by hand — staleness, invalidation, dedupe all become your bugs. Server state lives in the query cache; stores hold client state.

3. **Secrets in `VITE_` vars.** Anything in the bundle is readable by every visitor (`dist/` is plain text — go look). API keys with billing attached go behind a server you control.

4. **Learning Next.js before React.** RSC + client components confuse learners who can't yet tell "which parts of my knowledge apply where." You've done it in the right order — Next.js will now feel like React plus a server, not a new religion.

5. **Skipping the deploy until "the end."** Deploy the walking skeleton in week one of any project; discovering the SPA-fallback issue or an env mixup on launch day is self-inflicted. CI + auto-deploy makes shipping boring — the goal.

6. **Framework-hopping instead of shipping.** The hiring signal is deployed, tested, explained projects — not a tour of five state libraries. Finish the capstone; write a README that explains your architectural *choices*.

## Practice Exercises

1. Deploy your Chapter 12 routed app to Netlify or Vercel. Verify deep-linking (`/courses/2` fresh in a new tab) and the 404 page both work in production. Document the rewrite config you needed.
2. Take your Chapter 11 cart or auth context and port it to Zustand. Compare in writing: lines of code, provider ceremony, and what re-renders when (verify with the DevTools Profiler).
3. Add the CI workflow above to a repo, then intentionally push (a) a type error, (b) a failing test. Confirm both are caught and fix them.
4. Spend an evening with the Next.js tutorial and port two pages of your routed app. Write a half-page comparison: what got easier, what got harder, what stayed identical.
5. Write your "state placement policy" as a one-page reference: for each of ten example values (modal open, JWT, search text, fetched product list, theme, form draft, selected tab, WebSocket messages, cart, feature flag), name where it lives (local / lifted / URL / query cache / store / context) and defend the choice in one sentence each. This document is interview gold.
