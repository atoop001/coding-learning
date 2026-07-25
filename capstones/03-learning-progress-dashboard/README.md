# Capstone 3: Learning Progress Dashboard

Build the tool you already wish existed: a web app that tracks your progress through this
exact workspace — every chapter read and every project completed across all 16 tracks —
with streaks, completion percentages, and charts you'll actually look at every morning.

This capstone is deliberately meta. The curriculum you're studying becomes the data set,
and you become the user. That's not a gimmick; it's the whole point.

---

## Why This Project

**You are the user.** Most learner projects have imaginary requirements ("a store that
sells... shoes, I guess?"). Here, every requirement is real because you'll hit it
yourself: "I can't tell which track I should finish next," "I lost my streak because the
date math was wrong," "I marked the wrong chapter done and can't undo it." Real
requirements produce real design decisions, and real design decisions are what interviews
ask about.

**Dogfooding finds bugs specs can't.** You'll use this dashboard daily. Daily use
surfaces the bugs that only appear in the wild — the streak that breaks at midnight, the
chart that lies when a track has zero chapters started, the progress bar that renders
wrong on your phone. Fixing bugs you personally suffered from is the fastest way to learn
what "production quality" actually means.

**It demonstrates two skills employers screen for.** First, data modeling: you'll design
entities (tracks, chapters, projects, completion events) and live with the consequences
of your schema. Second, data visualization: turning raw completion events into progress
bars, percentages, streaks, and charts that communicate at a glance.

**The v1 → v2 migration is itself the resume story.** You'll build v1 with data in
localStorage, then upgrade to a full Express + SQL backend — *without rewriting the UI*.
If you design the data layer well, the swap is nearly invisible to your components.
"I migrated a production app from local storage to a REST API behind a stable interface,
with zero UI rewrites" is a sentence that gets follow-up questions in interviews, and
you'll be able to answer every one of them because you lived it.

---

## Difficulty & Estimated Effort

**Difficulty: Intermediate.**

| Stage | Scope | Estimated effort |
|---|---|---|
| v1 (frontend-only) | React + TypeScript + localStorage, charts, streaks | ~10–15 hours |
| v2 (full-stack upgrade) | Express + SQL API, auth, cross-device sync | adds ~10–20 hours |

You do **not** need to do both stages back to back. v1 is a complete, shippable,
daily-usable product on its own. Build v1 right after finishing the React track, use it
for weeks, and come back for v2 once you've finished node-express. The gap between
stages is a feature — you'll return to your own code cold, which is its own lesson.

---

## Prerequisites

**For v1 (frontend-only):**
- `javascript` — the whole track (arrays, dates, and array methods get a workout here)
- `typescript` — types for your data model are the backbone of this project
- `react` — components, state, effects, lifting state, controlled inputs

**For v2 (full-stack upgrade), add:**
- `node-express` — through the databases and auth chapters at minimum
- `sql` — schema design, joins, aggregate queries
- `deployment-devops` — to ship it to a live URL (needed for v1's deploy too; you can
  learn just enough for a static deploy first and circle back)

If you haven't finished the v2 prerequisites yet: stop reading at Phase 2, build v1, and
enjoy it. Phases 3–4 will still be here.

---

## Phased Milestones

Work through the phases in order. Each checkbox is roughly a work session or less.

### Phase 1 — Data Model & Catalog

The unglamorous phase that determines whether every later phase is easy or painful.
Get the entities right before writing any UI.

- [ ] On paper (or in a markdown scratch file), design your core entities: **Track**
      (id, name, order), **Chapter** (id, trackId, number, title), **Project** (id,
      trackId, name), and **CompletionEvent** (what was completed, and — critically —
      **when**, as a date). Decide the ID scheme (slugs like `react/03-hooks` are fine).
- [ ] Write TypeScript interfaces for all four entities. These types will outlive both
      versions of the app, so take your time naming things.
- [ ] Hand-build a JSON catalog of the full curriculum: open the workspace root, list the
      16 track folders (`command-line`, `csharp`, `data-structures-algorithms`,
      `deployment-devops`, `git-github`, `how-the-web-works`, `html-css`, `java`,
      `javascript`, `node-express`, `python`, `react`, `rust`, `sql`,
      `testing-debugging-security`, `typescript`), and for each one record its chapters
      from `learning-docs/` and its projects from `projects/`. Yes, by hand. You'll
      learn your own curriculum's shape in the process.
- [ ] Validate the catalog against your TypeScript types (a tiny script or just
      importing it into a typed constant is enough — the compiler is your validator).
- [ ] Decide what a "completion" record looks like separately from the catalog. The
      catalog is static curriculum data; completions are *your* data. Keep them in
      separate structures from day one — in v2 they'll live in different database tables.

A sample catalog entry shape (illustrative only — design your own):

```json
{
  "id": "react",
  "name": "React",
  "chapters": [{ "id": "react/01-components", "number": 1, "title": "Components" }],
  "projects": [{ "id": "react/project-recipe-box", "name": "Recipe Box" }]
}
```

### Phase 2 — Dashboard v1 (React + localStorage)

The complete product, minus the server. Everything a user (you) needs, running entirely
in the browser.

- [ ] Scaffold a React + TypeScript app (Vite is the sensible default).
- [ ] Build the data layer **as an interface first**: define something like a
      `ProgressStore` with methods for reading completions, adding a completion event,
      and removing one. Then write the localStorage implementation of that interface.
      Your components should only ever talk to the interface. (This one decision is
      what makes Phase 3 a swap instead of a rewrite.)
- [ ] Render the track list: one card or row per track with a per-track progress bar
      (chapters done / total, projects done / total) and an overall completion
      percentage across the whole curriculum.
- [ ] Add the interaction: mark a chapter or project done (recording a dated completion
      event), and un-mark it (mistakes happen — you'll make one within a week).
- [ ] Implement streak logic as **pure functions** that take a list of dated completion
      events and return the current streak, longest streak, and per-day history.
      No React, no localStorage, no Date.now() inside these functions — dates in,
      numbers out.
- [ ] Write tests for the streak and percentage functions before wiring them into the
      UI. Cover: empty history, a single event, multiple events on one day, a gap day,
      and events "today."
- [ ] Add at least two charts: e.g., a bar chart of completion percentage per track, and
      a line or area chart of cumulative completions over time. Pick chart types that
      answer a question you actually have.
- [ ] Handle the empty state gracefully — the app should look intentional on day one,
      before any progress exists.
- [ ] Start using it. Every day. Seriously — enter your real current progress and make
      updating it part of finishing a study session.

### Phase 3 — Full-Stack v2 (Express + SQL + Auth)

Same UI, new engine. The measure of success for this phase: how *little* your React
components change.

- [ ] Design the SQL schema: tables for users, completion events, and (your choice)
      either a tracks/chapters catalog in the database or the catalog kept as static
      JSON served by the API. Write down the reasoning either way.
- [ ] Build the Express API: endpoints to fetch completions, create a completion event,
      and delete one. Match the shape of your `ProgressStore` interface — the API is
      just that interface over HTTP.
- [ ] Add authentication (session or token — whatever your node-express track taught).
      Registration can be minimal; this app has approximately one user.
- [ ] Write the API-backed implementation of `ProgressStore` in the frontend and swap it
      in. If Phase 2's checkbox about the interface was done honestly, this is a small
      diff. If it's a big diff, that's a lesson too — write down why.
- [ ] Build a one-time migration path: export your real localStorage data and import it
      into the database. Do not lose your streak history to your own upgrade.
- [ ] Verify cross-device sync: mark a chapter done on your PC, see it on your phone.
- [ ] Keep the localStorage implementation around behind a flag or fallback — offline
      grace is a nice-to-have, and keeping both implementations honest keeps the
      interface honest.

### Phase 4 — Ship

- [ ] Deploy it: frontend and (for v2) API + database on a live URL. For v1 alone, a
      static host is enough.
- [ ] Set up CI: on every push, run the tests and a production build. A pull request
      that breaks the streak math should fail loudly before it reaches you.
- [ ] Use the deployed app — not localhost — as your daily driver for **at least two
      weeks**. Keep a running list of every friction point and bug you notice.
- [ ] Ship at least one improvement that came from that list, not from this spec. That
      improvement is the proof the dogfooding worked.

---

## Definition of Shipped

This capstone is done when every box below is checked — not before.

- [ ] Live URL that loads for someone who isn't you (v1: static site is fine; v2: API
      and database live too).
- [ ] Your **real** progress data is in it — actual chapters you've read, actual dates.
      No lorem-ipsum completions.
- [ ] Charts render correctly for the real data, including edge cases: untouched tracks,
      a 100% track, and a day with multiple completions.
- [ ] README in the project repo with screenshots, a short architecture note, and (post
      v2) a paragraph on the localStorage → API migration.
- [ ] Automated tests for the streak and percentage logic. These are pure functions —
      dates and events in, numbers out — which makes them the easiest code in the whole
      app to test. There is no excuse for skipping this one.
- [ ] No secrets committed. Database URLs, session secrets, API keys — all in environment
      variables with placeholders like `<your-database-url>` in the example config, and
      the real env file in `.gitignore`. Check the git history, not just the working tree.

---

## Hints

Nudges, not answers. If a hint doesn't make sense yet, build until it does.

- **Design the data layer as an interface from day one.** The single highest-leverage
  decision in this project. If your components call `localStorage.getItem` directly,
  Phase 3 is a rewrite. If they call `store.getCompletions()`, Phase 3 is a new file.
- **Store completion *events* with dates, not booleans.** A `done: true` flag can tell
  you *what* is finished but never *when* — and streaks, history charts, and "completions
  this week" all need *when*. An append-able list of dated events can always be reduced
  to booleans; booleans can never be inflated back into history.
- **Chart library:** Recharts is the path of least resistance in React and plenty for
  this project. Chart.js (via react-chartjs-2) is a fine alternative. Hand-rolled SVG or
  D3 are rewarding but easily double the phase — save them for a stretch goal.
- **Timezone gotchas will eat your streak logic alive.** "Did I complete something
  yesterday?" depends on what "yesterday" means, and `Date.toISOString()` answers in
  UTC, not your local day. Decide early whether a "day" is a UTC day or a local day,
  convert consistently at one boundary, and add a test for an event at 11:30 PM. Your
  future self, checking the dashboard at midnight, will discover whether you got this
  right.
- **Keep the catalog boring.** Static JSON, checked into the repo, hand-maintained. When
  you add a chapter to a track in real life, you edit the JSON. Automating catalog
  generation from the filesystem is a fun trap — it's a stretch goal, not a foundation.
- **Un-completing must work.** Any state you can enter with one click, you must be able
  to leave. Deleting an event is easier than un-setting a boolean tangled into derived
  state — another argument for events.

---

## Stretch Goals

Only after the Definition of Shipped is fully checked:

- **Import/export JSON backup** — one button to download all your completion events, one
  to restore them. (You'll have built half of this for the v2 migration anyway.)
- **Reading-time estimates per chapter** — add estimated minutes to the catalog and show
  "about 3 hours left in TypeScript" per track.
- **Goal setting** — "finish TypeScript by September," with an on-track/behind indicator
  computed from your actual completion rate.
- **Heatmap calendar** — a GitHub-style contributions grid of your study days. Your
  dated completion events make this almost free; the rendering is the fun part.
- **Public share page** — a read-only view of your progress at a shareable URL, so the
  dashboard doubles as a public learning log. (Careful: this makes auth boundaries
  suddenly matter.)

---

## Telling the Story

When this goes on your resume and portfolio, lead with the parts that are rare, not the
parts that are common. Every bootcamp graduate has "built a React app." Very few can say:

- *"Designed an event-based progress data model and migrated the app from localStorage
  to a REST API + SQL backend behind a stable storage interface — zero changes to UI
  components."* That's a bullet about architecture, not tools.
- *"Used in production daily for N months; shipped X improvements driven by my own usage."*
  Update N and X honestly over time. Real usage numbers, even tiny ones, beat feature
  lists.
- *"Test-covered streak and completion logic as pure functions."* Small bullet, but it
  signals you know what deserves tests and why.

In interviews, the migration is your best story arc: the naive version (booleans in
localStorage) you avoided, the interface decision that paid off, the timezone bug you
hit at midnight, the moment you saw your data sync to your phone. Walk through it as a
sequence of decisions and consequences — that's what "experience" sounds like.

And the closing line that no side project with fake data can match: **"I still use it
every day."** Then open the live URL and show them today's streak.
