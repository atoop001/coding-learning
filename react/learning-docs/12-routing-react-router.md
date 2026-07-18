# Chapter 12: Routing with React Router

## Overview

A single-page application swaps *components*, not HTML pages — but users still expect URLs that work: bookmarkable, shareable, back-button friendly. **React Router** maps URLs to component trees. This chapter covers route configuration, `Link`/`NavLink` navigation, URL params, nested routes with `Outlet`, programmatic navigation, search params, 404s, and protected routes.

## Definitions & Explanations

### Client-side routing

In an SPA there is one `index.html`; the router:

1. Intercepts link clicks and updates the address bar via the History API (no server round-trip).
2. Renders the component tree matching the new URL.
3. Handles back/forward buttons for free.

The URL becomes **state you don't own** — and it's *shareable* state. A good rule: if a piece of state should survive refresh or be linkable (current page, selected item, active tab, search filters), it probably belongs in the URL, not in `useState`.

> Install: `npm install react-router-dom` (this chapter targets v6+/v7 component-based routing).

### Core vocabulary

- **`<BrowserRouter>`** — wraps the app, connects React Router to the browser's history. One per app, at the root.
- **`<Routes>` / `<Route path element>`** — declarative URL-to-component mapping. Best match wins.
- **`<Link to>` / `<NavLink to>`** — anchor replacements that navigate without reloading. `NavLink` knows when it's active (for nav highlighting). Using a raw `<a href>` internally causes a full page reload and loses all state.
- **Dynamic segments** — `path="courses/:courseId"`; read with `useParams()`. Params are always `string | undefined` — parse and validate.
- **Nested routes + `<Outlet />`** — child routes render *inside* the parent's layout at the `Outlet` position. This is how shared shells (nav, sidebars) persist across pages.
- **Index route** — `<Route index element={...} />`: what renders at the parent's own path.
- **`useNavigate()`** — programmatic navigation (after form submit, login, etc.). `navigate('/dash')`, `navigate(-1)` for back, `{ replace: true }` to not add a history entry.
- **`useSearchParams()`** — read/write the query string (`?q=react&page=2`) like URL-backed state.
- **Catch-all** — `path="*"` for the 404 page.

(React Router also offers a data-router API — `createBrowserRouter`, loaders/actions — and full framework modes. Learn the component basics here first; loaders are a natural next step and are mentioned in Chapter 14.)

## Code Examples

### App skeleton with layout, nesting, params, and 404

```tsx
// main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>,
);
```

```tsx
// App.tsx
import { Routes, Route, Link, NavLink, Outlet } from 'react-router-dom';

function App() {
  return (
    <Routes>
      {/* Layout route: no path of its own beyond "/", children render in its Outlet */}
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />                 {/* at "/"        */}
        <Route path="courses" element={<CoursesLayout />}>
          <Route index element={<CourseList />} />          {/* /courses      */}
          <Route path=":courseId" element={<CourseDetail />} /> {/* /courses/42 */}
        </Route>
        <Route path="about" element={<About />} />
        <Route path="*" element={<NotFound />} />           {/* everything else */}
      </Route>
    </Routes>
  );
}

function Layout() {
  return (
    <div>
      <nav>
        {/* NavLink gets an `isActive` flag for styling the current section */}
        <NavLink to="/" end className={({ isActive }) => (isActive ? 'active' : '')}>
          Home
        </NavLink>
        <NavLink to="/courses" className={({ isActive }) => (isActive ? 'active' : '')}>
          Courses
        </NavLink>
        <NavLink to="/about">About</NavLink>
      </nav>
      <main>
        <Outlet /> {/* the matched child route renders here */}
      </main>
    </div>
  );
}

function NotFound() {
  return (
    <section>
      <h1>404 — nothing here</h1>
      <Link to="/">Back home</Link>
    </section>
  );
}
```

### URL params, validated

```tsx
import { useParams, Link } from 'react-router-dom';

const COURSES = [
  { id: 1, title: 'React Fundamentals' },
  { id: 2, title: 'Advanced Hooks' },
] as const;

function CourseList() {
  return (
    <ul>
      {COURSES.map(c => (
        <li key={c.id}>
          {/* Relative link: from /courses, this goes to /courses/1 */}
          <Link to={String(c.id)}>{c.title}</Link>
        </li>
      ))}
    </ul>
  );
}

function CourseDetail() {
  // useParams returns Record<string, string | undefined> — ALWAYS validate.
  const { courseId } = useParams();
  const id = Number(courseId);
  const course = COURSES.find(c => c.id === id);

  if (!course) {
    // Bad ids are user input like any other — render a real not-found state.
    return <p role="alert">No course with id “{courseId}”. <Link to="..">All courses</Link></p>;
  }

  return (
    <article>
      <h2>{course.title}</h2>
      {/* ".." = up one route level, like a filesystem */}
      <Link to="..">← All courses</Link>
    </article>
  );
}
```

### Programmatic navigation

```tsx
import { useNavigate } from 'react-router-dom';

function NewCourseForm() {
  const navigate = useNavigate();
  const [title, setTitle] = useState('');

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const id = saveCourse(title); // pretend this persists and returns an id
    // replace:true → Back won't return to the now-stale form
    navigate(`/courses/${id}`, { replace: true });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={title} onChange={e => setTitle(e.currentTarget.value)} />
      <button type="submit">Create</button>
    </form>
  );
}
```

### Search params as shareable state

```tsx
import { useSearchParams } from 'react-router-dom';

function CourseList({ courses }: { courses: { id: number; title: string; level: string }[] }) {
  const [searchParams, setSearchParams] = useSearchParams();
  // URL is the source of truth — refresh/share/back all work with no extra state.
  const level = searchParams.get('level') ?? 'all';

  const visible = level === 'all' ? courses : courses.filter(c => c.level === level);

  return (
    <div>
      <select
        value={level}
        onChange={e => {
          const v = e.currentTarget.value;
          // Functional form + delete keeps URLs clean (no ?level=all noise)
          setSearchParams(prev => {
            v === 'all' ? prev.delete('level') : prev.set('level', v);
            return prev;
          }, { replace: true }); // filter tweaks shouldn't spam history
        }}
      >
        <option value="all">All</option>
        <option value="beginner">Beginner</option>
        <option value="advanced">Advanced</option>
      </select>
      <ul>{visible.map(c => <li key={c.id}>{c.title}</li>)}</ul>
    </div>
  );
}
```

### Protected routes (context from Chapter 11 + router)

```tsx
import { Navigate, useLocation } from 'react-router-dom';

function RequireAuth({ children }: { children: React.ReactNode }) {
  const auth = useAuth();          // Chapter 11's hook
  const location = useLocation();

  if (auth.status !== 'authenticated') {
    // Remember where they were headed so login can send them back.
    return <Navigate to="/login" replace state={{ from: location.pathname }} />;
  }
  return <>{children}</>;
}

// Usage:
<Route path="dashboard" element={<RequireAuth><Dashboard /></RequireAuth>} />
```

## Common Pitfalls

1. **`<a href>` for internal links.** Full page reload: all state gone, app re-boots. Internal navigation is always `Link`/`NavLink`/`navigate`; plain `<a>` is only for external URLs.

2. **Route state that should be URL state.** Storing the selected course id in `useState` means refresh loses it and it can't be shared. If it should survive refresh, it belongs in the path or search params.

3. **Forgetting `end` on the root `NavLink`.** `<NavLink to="/">` matches *every* path by prefix, so Home renders as active everywhere. `end` (or `to="/"` with `end`) restricts it to exact matches.

4. **Trusting `useParams` blindly.** `:courseId` can be `"999"`, `"abc"`, or missing after a refactor. `Number(...)` + existence check + not-found UI, every time. (TS types it `string | undefined` for a reason.)

5. **Duplicating layout in every page instead of nesting.** Copy-pasting `<Nav />` into each page component loses persistent-layout benefits (nav state, no re-mount flicker). One layout route + `Outlet`.

6. **Deploy 404s.** A Vite SPA on static hosting must rewrite all paths to `index.html`, or deep links like `/courses/2` 404 on refresh. Every host has a rewrite setting (`_redirects` on Netlify, `vercel.json` rewrites, etc.) — details in Chapter 14.

7. **Navigating during render.** Calling `navigate()` in the component body is a side effect during render (React warns). Navigate in handlers/effects, or render `<Navigate />` declaratively as in `RequireAuth`.

## Practice Exercises

1. Build a three-page site (Home, Notes, About) with a persistent nav layout, active-link styling, and a styled 404 including a "back home" link.
2. Add `/notes/:noteId` detail pages over a hard-coded array: list links to details, details validate the id, invalid ids show inline not-found. Add a "next note →" link computed from the data.
3. Move a search input's value into `?q=` with `useSearchParams` so refresh and back/forward preserve the query. Decide and justify: `replace` or push for each keystroke?
4. Combine Chapters 11+12: add `/login` (fake form → dispatch `logged_in` → navigate to the page stored in `location.state.from`) and protect `/dashboard` with `RequireAuth`. Test the full loop including deep-linking straight to `/dashboard` while logged out.
5. Add a nested settings section: `/settings` layout with its own sub-nav and child routes `profile` and `appearance` (reuse the theme toggle from Chapter 11). Both `Outlet`s — the app's and settings' — should be in play at once; draw the render tree for `/settings/appearance`.
