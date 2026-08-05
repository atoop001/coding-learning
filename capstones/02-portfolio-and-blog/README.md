# Capstone 2: Portfolio & Blog with a Real Backend

Build your own corner of the internet: a personal portfolio site with a blog, where
articles live in a SQL database, get written in markdown through an admin panel that
only you can log into, and appear on a fast, clean public site — deployed with CI/CD
on a domain you own.

This is **not** a static template you fork and fill in. You are building the whole
machine: database, server, auth, rendering, deployment. That is the point.

---

## Why This Project

Every developer needs a portfolio site. Most people satisfy that requirement with a
static template — and hiring managers can smell a template from across the room. It
tells them nothing except that you can edit a config file.

Doing it with a real backend turns a checkbox into a demonstration:

- **It proves full-stack range.** One project shows you can model data in SQL, build
  and secure an API, render a frontend, and ship the whole thing to production with
  tests running in CI. That is the entire job description of a junior web developer,
  in one URL.
- **It compounds.** Every project you build after this one gets a write-up here. Your
  next capstone gets a portfolio entry and a blog post about what was hard and how you
  solved it. Six months from now, this site is a living record of your growth — which
  is exactly what interviewers want to see and almost never get.
- **You are the customer.** You will actually use the admin panel. Nothing teaches you
  about rough edges in your own software faster than being its daily user.
- **It never stops paying rent.** Client projects end. Toy apps rot. This site stays
  live, stays linked from your resume, and stays worth improving for your entire career.

This is the recommended **warm-up capstone** — the first one to ship end-to-end. It
touches every layer without any single layer being brutally hard, and finishing it
gives you the permanent home that showcases everything else you build.

## Difficulty & Estimated Effort

**Intermediate. Roughly 15–25 hours** of focused work, spread over 2–4 weeks of
real life.

This should be the **first capstone you attempt**. The scope is deliberately contained:
one user (you), one content type (articles), a handful of pages. The challenge is not
any single feature — it is carrying a project across *every* layer, from empty folder
to live domain, without abandoning it at 80%. That last 20% (deployment, DNS, CI,
polish) is where most learners have never been. Going there is the whole value.

If a phase takes longer than the estimate, that is normal. Log what ate the time —
it makes a great first blog post.

## Prerequisites

You should have completed, or be comfortable with the material in:

- **html-css** — semantic markup, layout with flexbox/grid, responsive design.
- **javascript** — the core track. You will write plenty of it on both ends.
- **node-express** — at minimum through the **databases** and **auth** chapters.
  You need to know how to build routes, talk to a database from Express, hash
  passwords, and manage a session or token.
- **sql** — basics: tables, `SELECT`/`INSERT`/`UPDATE`/`DELETE`, a simple schema.
  This project needs one or two tables; you do not need joins mastery.
- **deployment-devops** — chapters 1 (foundations), 7–8 (deploying an app +
  database), and 12 (CI). You will lean on these hard in Phase 4.

**React is optional.** A server-rendered site (Express + a template engine) or plain
HTML with a little vanilla JS is a completely legitimate — arguably *better* — fit
for a content site like this. If you have done the react track and want to use it,
fine, but do not block this capstone on learning React first. Blogs were solved
problems long before frontend frameworks existed.

## Phased Milestones

Work the phases in order. Each phase ends with something you can look at and show
someone. Do not start Phase 2 until Phase 1's checklist is done — especially the
writing. Content-first is not a slogan here; it is the thing that stops you from
building an admin panel for articles that never get written.

### Phase 1 — Design & Content (before any code)

The goal: know exactly what you are building, and have real content to build it for.
Placeholder text hides design problems; real text exposes them.

- [ ] Sketch every page on paper or in a simple tool: homepage, article list, article
      detail, portfolio/projects page, about page, admin login, admin editor. Boxes
      and arrows are enough — you are deciding what exists, not how it is styled.
- [ ] Write **two real blog posts** in markdown, in plain `.md` files, before building
      anything. Good first topics: "What I learned building X" from any earlier track
      project, or "Why I'm learning web development." 400+ words each, honestly written.
- [ ] Write the copy for your about page and one or two portfolio project entries
      (name, description, tech used, link, what you'd do differently).
- [ ] Define the article data model on paper: what columns does an `articles` table
      need? Think through: title, slug, markdown body, published/draft state,
      created/updated timestamps. Decide it now; write the `CREATE TABLE` in Phase 2.
- [ ] Decide your stack for the frontend (server-rendered templates, vanilla JS, or
      React) and write one sentence justifying it. You will reuse that sentence in
      interviews.

### Phase 2 — Backend & Admin

The goal: you can log in, write a post in markdown, save it as a draft, and publish
it — all stored in SQL.

- [ ] Set up the Express project and the database. Create the `articles` table from
      your Phase 1 schema (plus a `users` table, even though it will only ever hold
      one row — you).
- [ ] Build CRUD for articles: create, read (one and all), update, delete. Store the
      **raw markdown** in the database — render it later, at display time.
- [ ] Implement single-admin auth: a login page, hashed password, and a session or
      token that protects every admin route. There is no registration flow — you
      create the one admin account with a script or a seed command.
- [ ] Implement draft vs published: a draft is visible in the admin, invisible on the
      public site. Publishing sets the state (and, ideally, a published-at timestamp).
- [ ] Build the admin panel UI: list of all articles with their state, an editor page
      with a big textarea for markdown, and save/publish/delete actions. It only needs
      to be usable by you — plain and functional beats pretty here.
- [ ] Write tests for the parts that would embarrass you if they broke: auth actually
      blocks unauthenticated requests to admin routes, drafts do not appear in public
      queries, CRUD round-trips work.

### Phase 3 — Public Site

The goal: a site you would not be embarrassed to send to a stranger.

- [ ] Homepage: who you are in one or two sentences, your most recent posts, and a
      clear path to the portfolio and about pages.
- [ ] Article list page: published posts only, newest first, with titles, dates, and
      a short excerpt or opening line.
- [ ] Article detail page at a readable URL (see Hints on slugs), with the markdown
      rendered to HTML — headings, lists, links, and **code blocks** all displaying
      properly, since you will be writing about code.
- [ ] Portfolio/projects page built from your Phase 1 copy: each project with a
      description, tech list, and links to the live thing and the repo.
- [ ] About page. A photo is optional; a genuine paragraph is not.
- [ ] Responsive: usable on a phone. Check every page at a narrow width, not just
      the homepage.
- [ ] Decent typography: a readable body font at 16px+, a comfortable line length
      (roughly 60–75 characters), enough line height and whitespace. Typography *is*
      the design of a blog — an hour spent here outperforms ten hours of decoration.

### Phase 4 — Ship

The goal: real URL, real HTTPS, real automation. This phase is where the project
becomes a credential instead of a folder on your laptop.

- [ ] Deploy the app and the database to a host, using what you learned in
      deployment-devops chapters 7–8. Environment variables for every secret; the
      production database is not SQLite-on-a-whim unless your host supports it durably.
- [ ] Buy a domain (they cost about as much as a pizza per year) and point it at
      your deployment. HTTPS must work — most modern hosts handle certificates for
      you, but *verify* the padlock, and verify that plain `http://` redirects.
- [ ] Set up CI so your tests run on every push (deployment-devops chapter 12). A
      failing test should block or at least loudly flag a deploy.
- [ ] Log into your admin panel **on the live site** and publish your two posts for
      real. Not seeded, not inserted by hand into the database — published through
      the front door, using the thing you built.
- [ ] Run an automated accessibility audit (axe DevTools or Lighthouse) on every
      public page, fix what it finds, and note the audit and its results in the
      README.
- [ ] Send the URL to one real person and watch them use it. Fix the first
      confusing thing they hit.

## Definition of Shipped

You are done when every box below is checked. Not before. "Basically done except
deployment" is another way of saying "not done."

- [ ] The site is live at a custom domain with working HTTPS.
- [ ] At least **2 published posts**, written by you, published through the admin panel.
- [ ] The admin panel works from any device — you can log in from your phone and
      fix a typo in a published post.
- [ ] Tests exist and are **green in CI** on the latest push to `main`.
- [ ] The repo has a README with screenshots, a short description of the stack and
      why you chose it, and instructions for running it locally.
- [ ] **No secrets in the repo** — no database passwords, session secrets, or API
      keys in any commit, including old ones. Search your git history, not just the
      current tree. If something leaked, rotate it; deleting the file is not enough.

## Hints

Nudges, not answers. Reach for these when you are stuck or about to over-build.

- **Rendering markdown safely is a security problem, not just a formatting one.**
  Markdown converts to raw HTML, and if anything in that pipeline ever contains
  attacker-controlled input, you have an XSS hole. "But only I write posts" is true
  today — build the habit anyway: use a well-maintained markdown library, and run
  its output through an HTML sanitizer before it hits the page. Two lines of
  dependency wiring now, versus a class of vulnerability you never think about again.
- **Slugs make URLs.** `/articles/17` works but `/articles/why-i-learned-sql` is
  better for readers and for search engines. Generate the slug from the title when
  an article is created (lowercase, hyphens, strip punctuation), store it in its own
  unique column, and look articles up by it. Decide early what happens if you retitle
  a published post — the boring answer (the slug stays frozen) is the right one,
  because cool URIs don't change.
- **You have one user. Build for one user.** No registration, no password reset
  emails, no roles, no user profiles. A single admin row, a login form, and a
  protected route prefix is the entire auth system. Every hour spent on multi-user
  plumbing is an hour stolen from Phase 3 and 4, which are the phases people see.
- **SEO basics are cheap and worth it.** Every page needs a real `<title>` and a
  meta description; article pages should get them from the article itself. Add
  Open Graph tags so your posts unfurl nicely when you link them from social sites —
  that unfurl is free advertising for your work. Semantic HTML (`<article>`, `<h1>`
  once per page, real heading order) does the rest.
- **Resist the redesign spiral.** You will be tempted, around hour ten, to throw out
  your design and start the CSS over. Don't. Pick one font pairing and a two-color
  palette in Phase 1, write it down, and declare visual changes out of scope until
  after the Definition of Shipped is fully checked. Shipped-and-plain beats
  gorgeous-and-local every single time, and you can restyle a live site forever.
- **Automated accessibility scans catch maybe a third of real issues.** axe and
  Lighthouse are great at finding missing alt text, low-contrast colors, and
  unlabeled form fields — but they cannot tell you whether a keyboard user can
  actually get through your site. Also tab through every page using only the
  keyboard: can you reach every link and button, is focus always visible, and
  does the order make sense?
- **When you get stuck in Phase 4, it is almost always environment variables or
  DNS.** Check what the production process actually sees (log the presence — never
  the value — of each required variable at startup), and remember DNS changes can
  take a while to propagate. Neither of these means you broke your code.

## Stretch Goals

Only after the Definition of Shipped is fully checked. Each of these is a nice
afternoon-to-weekend project — and each one is a blog post.

- **RSS feed.** Genuinely simple to generate from your articles table, beloved by
  the exact kind of technical person who might one day interview you.
- **Tags and search.** Tag articles, filter the list by tag, and add a simple
  search. This will teach you more SQL — a many-to-many table for tags, `LIKE` or
  full-text search for the search box.
- **"Reply by email" instead of comments.** Comment systems invite spam and
  moderation work. A `mailto:` link at the end of each post ("Thoughts? Email me")
  gives you reader contact with zero infrastructure. It is a design *decision*, and
  you can write about why you made it.
- **Dark mode.** CSS custom properties plus `prefers-color-scheme`, with an optional
  manual toggle. A tidy, contained exercise in CSS architecture.
- **Privacy-friendly analytics.** Self-hosted or a lightweight privacy-respecting
  service — no invasive tracking on your own site. Knowing which posts get read
  changes what you write next.
- **Image uploads in the admin.** Paste or upload an image while writing a post and
  get back a markdown image reference. Teaches file handling, storage, and why you
  should never trust an uploaded file's claimed type.

## Telling the Story

A finished project you cannot talk about is worth half a finished project.

**On your resume**, this is one line under Projects, built as *what you did, how,
and the proof*:

> Designed and shipped a full-stack portfolio and blog platform (Node/Express, SQL,
> server-rendered frontend) with markdown authoring, single-admin authentication,
> and CI-tested deployments to a custom domain — live at yourdomain.example

Adjust the stack words to match your actual choices. The load-bearing part is
"designed and shipped" plus a URL they can click. Almost no junior resume has a
clickable, self-built, full-stack URL on it. Yours will.

**The blog posts are interview artifacts.** Write about what you build — starting
with this project. "How I handled markdown rendering safely," "What broke when I
first deployed," "Why I chose server-rendered over React for my blog." When an
interviewer asks "tell me about a technical decision you made," you will not be
reaching for a memory; you will be summarizing something you already wrote, and you
can send them the link afterward. A candidate who writes clearly about their own
tradeoffs reads as a mid-level thinker regardless of years of experience.

**Every later capstone links back here.** When you finish the next capstone, it gets
an entry on your portfolio page and a write-up on your blog. That is the compounding
loop: this site is not one project among several — it is the front door to all of
them. Keep it alive, keep it current, and let every future thing you build make it
more valuable.

Now go check off Phase 1. The two blog posts come first — everything else is just
plumbing to get them onto the internet.
