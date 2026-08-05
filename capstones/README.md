# Capstones

Large cross-track projects that combine everything you've learned into finished,
shareable, portfolio-worthy work. Unlike track projects, capstones span multiple
tracks, take weeks rather than hours, and end with something you can put in front
of an employer: a live URL, a public GitHub history, or a written comparison.

Each capstone lives in its own folder with a `README.md` spec — description,
phased milestones with checklists, a "definition of shipped," hints, stretch
goals, and guidance on telling the story in interviews. **No solution code** —
you write everything. Keep your project code inside the capstone's folder (or in
its own repo, linked from the spec's folder).

## The Capstones

| # | Capstone | Effort | Start after |
|---|----------|--------|-------------|
| 1 | [Tutoring Business Platform](01-tutoring-business-platform/README.md) | 40–80 h | react + node-express + sql + testing-debugging-security |
| 2 | [Portfolio & Blog with a Real Backend](02-portfolio-and-blog/README.md) | 15–25 h | node-express (databases + auth) |
| 3 | [Learning Progress Dashboard](03-learning-progress-dashboard/README.md) | 10–15 h (v1) | react — v1; node-express — v2 |
| 4 | [Open-Source Contribution Program](04-open-source-contributions/README.md) | 15–30 h | git-github + one language track |
| 5 | [Public API with a Public Face](05-public-api/README.md) | 25–40 h | node-express + sql + deployment-devops + testing-debugging-security |
| 6 | [Data Pipeline & Public Dashboard](06-data-pipeline-dashboard/README.md) | 25–40 h | python + sql + deployment-devops |
| 7 | [Same App, Twice](07-same-app-twice/README.md) | 30–50 h | java OR csharp + a finished backend + testing-debugging-security |

Deliberately absent from every "Start after" column: **data-structures-algorithms**
(interview prep, not a project dependency), **how-the-web-works** (conceptual
grounding that helps but never blocks), and **rust** (a stretch track). None of
the three gates a capstone.

## Suggested Order

1. **Capstone 2 (Portfolio & Blog)** first — the quickest to ship end-to-end,
   and it becomes the home where every later capstone gets its write-up.
2. **Capstone 3 (Progress Dashboard) v1** after the React track — small, useful
   daily, and upgradeable later.
3. **Capstone 1 (Tutoring Platform)** is the north star — the flagship project
   with a real user and the strongest interview story. Start it once
   node-express, sql, and react are done; ship it in phases.
4. **Capstones 5 and 6** deepen the backend and data sides — pick whichever
   excites you more; you don't need both before job applications.
5. **Capstone 7 (Same App, Twice)** last — it requires a finished backend plus
   the Java or C# track, and it's the strongest proof that your skills
   transfer across languages.

Meanwhile, in the background: start **Capstone 4 (Open Source)** early and keep
it simmering alongside whichever numbered capstone you're on. Review turnaround
on real pull requests makes it a slow-burn project, not a sequential step — it
runs in parallel with everything above rather than waiting its turn.

## Keeping It Free

Every capstone here is shippable at **$0/month** as of mid-2026 — but free tiers
change constantly, so verify current terms before committing to any platform.
The combination that works today:

- **Static sites & SPAs** — Cloudflare Pages or GitHub Pages (no card required,
  generous limits).
- **Node/Express APIs** — Render's free web service. It spins down after ~15 idle
  minutes and cold-starts in 30–60 seconds; fine for a portfolio project — mention
  it in your README so reviewers aren't surprised by the first load.
- **Databases** — Neon (free Postgres that scales to zero when idle but doesn't
  expire) or Turso (free SQLite with the most generous limits). Avoid Render's
  free Postgres (expires after 30 days); Supabase's free projects pause after
  7 idle days. Railway and Fly.io no longer offer true free tiers.
- **Scheduled jobs** — GitHub Actions cron on a public repo: free, no limits that
  matter at this scale.
- **Domains** — the platforms' free subdomains (`*.pages.dev`, `*.onrender.com`,
  `*.github.io`) cost nothing. A custom domain is the one unavoidable spend
  (~$10–11/year at Porkbun or Cloudflare Registrar) and only Capstone 1 asks for
  one — if you already own a domain, point a subdomain at it for free.

You do **not** need all seven to be employable. Capstones 1 + 2 + 4 shipped well
beat all seven half-finished.
