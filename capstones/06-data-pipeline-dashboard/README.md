# Capstone 6: Data Pipeline & Public Dashboard

Build a scheduled data pipeline that collects real-world data every day without you touching it, stores it in a SQL database, and serves a public dashboard charting the accumulated history.

The architecture in one line:

```
GitHub Actions (cron) → Python collector → validate → SQL database → public dashboard
```

A Python collector runs on a schedule (GitHub Actions cron — free, and you'll learn why it's the right choice), pulls from a real external API, validates and cleans what it gets, and appends it to your database. A web dashboard turns months of that data into charts anyone can visit.

The magic of this project is time. On day one it's a script and an empty table. After three months of unattended runs, you own a dataset that does not exist anywhere else in exactly this form — and a live URL proving your automation ran reliably while you slept.

## Why This Project

Most portfolio projects handle data that arrives clean, on demand, in exactly the shape the code expects. Real systems don't get that luxury. This capstone forces you through the full messy-data lifecycle — **fetch → validate → clean → store → visualize** — with each stage defending against the failures of the stage before it.

What it proves to an employer:

- **You can handle unreliable inputs.** External APIs go down, return garbage, and change their response shape without warning. Your collector has to survive all of that without crashing or corrupting the database.
- **You can automate.** Scheduled, unattended execution is the core of data engineering, DevOps, and half of backend work. "It ran every day for four months without me" is a sentence very few junior candidates can say.
- **You understand data modeling.** Deciding which fields to keep, what types they are, and how raw data relates to cleaned data is schema design under real constraints.
- **You differentiate yourself.** Everyone has a to-do app. Nobody else has *your* dataset. After N months, your dashboard shows history that can't be recreated by cloning a repo.
- **It's adjacent-skill gold.** Data engineering sits right next to web development, and touching it credibly widens the set of jobs you can interview for.

## Difficulty & Estimated Effort

**Advanced.** Roughly **25–40 hours** of active building, spread over several weeks — plus the ongoing accumulation period, which costs you nothing but calendar time.

Rough breakdown:

- Phase 1 (source & contract): 3–5 hours — mostly reading and writing, not coding
- Phase 2 (collector): 8–12 hours — the engineering heart of the project
- Phase 3 (automation): 4–6 hours — short in code, long in "why didn't the workflow trigger?"
- Phase 4 (dashboard): 8–12 hours
- Phase 5 (ship & accumulate): 2–5 hours of work, 4+ weeks of patience

The build itself is not the hard part; the hard part is making something robust enough to leave alone. Plan the timeline honestly: the "4+ weeks unattended" requirement in the Definition of Shipped means you should get the pipeline running early, then build the dashboard while data accumulates in the background.

## Prerequisites

Complete (or be comfortable with the material in) these tracks first:

- **python** — through the APIs and files chapters. The collector is an API-consuming, database-writing Python program.
- **sql** — schema design, inserts, constraints, and aggregate queries. Your dashboard queries are GROUP BY practice with real stakes.
- **deployment-devops** — the CI/CD chapters. This matters more than it may seem: your scheduler *is* a CI workflow. GitHub Actions cron uses the same YAML, secrets, and runner model you learned for CI, pointed at a different job.
- **javascript** or **react** — for the dashboard. Either works; pick the one you want more practice in.

A quick self-check — you're ready if you can do these without much looking-up:

- Call a JSON API from Python, check the status code, and pull fields out of the response
- Write a `CREATE TABLE` with types and a `UNIQUE` constraint, and query it with `GROUP BY`
- Read a GitHub Actions workflow file and explain what triggers it and what each step does
- Fetch data in the browser and render something from it

If any of those feel shaky, spend an evening back in that track first — this project punishes shaky foundations by failing at 6 a.m. when you're not watching.

## Phased Milestones

Work the phases in order. Each phase produces something verifiable before the next begins.

### Phase 1 — Source Selection & Contract

Before any code: decide exactly what you're collecting and what shape it has. This written contract is what every later phase validates against.

- [ ] Pick your data source and confirm it has an official, documented API
- [ ] Read the API's terms of service and note anything relevant (storage rights, attribution, commercial limits) in your project notes
- [ ] Record the rate limits and design your fetch frequency to sit comfortably under them
- [ ] Make a few manual test calls (browser or `curl`) and save example responses to a `samples/` folder
- [ ] Write a **data dictionary**: for every field you'll keep — name, type, units, allowed range, whether null is acceptable, and the source field it maps from
- [ ] Design the SQL schema: a **raw** table (what the API sent, minimally processed) and a **cleaned** table (validated, typed, unit-normalized rows the dashboard reads)
- [ ] Decide your natural key — the column(s) that uniquely identify one observation (e.g., `city + observation_date`) — and enforce it with a uniqueness constraint

A data dictionary row can be as simple as:

```
temp_max_c | REAL | °C | -60 to 60 | not null | maps from daily.temperature_2m_max
```

Ten of those lines are worth more than any amount of code, because they turn "clean the data" from a vibe into a checklist.

**Phase 1 is done when:** someone else could read your data dictionary and schema and know exactly what a valid row looks like — without asking you anything.

### Phase 2 — The Collector

A Python script that can be run at any time, any number of times, and always leaves the database in a correct state.

- [ ] Fetch one period's data from the API using the documented endpoint
- [ ] Validate the response against your data dictionary: required fields present, types correct, values in range — reject (and log) anything that fails
- [ ] Handle the API being down or slow: timeouts, non-200 responses, and malformed JSON must produce a clear logged error and a nonzero exit code — never a half-written database or a silent success
- [ ] Make the collector **idempotent**: running it twice for the same period must not create duplicate rows (your uniqueness constraint from Phase 1 is the safety net; your insert logic should handle the conflict deliberately)
- [ ] Store the raw response as well as the cleaned rows
- [ ] Log what happened on every run: period fetched, rows inserted, rows skipped, validation failures
- [ ] Write tests for the validation logic and the idempotency behavior (these are the tests the Definition of Shipped requires)

A useful self-test before moving on: run the collector three times in a row, then yank your network connection and run it a fourth time. Row count unchanged after runs two and three, a clean loud failure on run four, and a log that tells the story — that's a collector ready for automation.

**Phase 2 is done when:** the self-test above passes and the validation/idempotency tests run green.

### Phase 3 — Automation

Turn "I run a script" into "a machine runs my script." This is where the deployment-devops track pays off.

**Why GitHub Actions cron?** You need something that executes a script on a schedule, for free, without keeping a computer on. Consider the alternatives honestly:

- A cron/Task Scheduler job on your own PC dies when the machine sleeps — and on a Windows laptop, it will sleep.
- A rented server ($5+/month) works but costs money and needs patching, and you'd be maintaining a machine to run one script a day.
- Serverless schedulers (cloud functions + a timer) are closer, but bring an account, billing setup, and a deployment story of their own.
- **GitHub Actions** gives you scheduled workflow runs on GitHub's machines, free for public repos, with built-in secret storage, run logs, and failure emails — using YAML you already know from CI.

The tradeoffs to know about Actions: scheduled runs can start minutes late under load (fine for daily data), and a scheduled workflow on a repo with no recent activity is auto-disabled after ~60 days (a good reason to keep committing). For daily or hourly collection, it's the obvious choice — and being able to *argue* that choice is worth as much as making it.

- [ ] Write a workflow that runs the collector on a cron schedule (e.g., `cron: '15 6 * * *'` — daily at 06:15 UTC; avoid `0 0` since on-the-hour slots are the most congested and most delayed)
- [ ] Store the database connection string (and any API key) as **repository secrets** — never in the workflow file, never in code, referenced only as environment variables like `<your-connection-string-secret>`
- [ ] Make the workflow fail visibly when the collector fails (nonzero exit = red run), and set up failure notifications so you actually find out (GitHub emails on workflow failure by default — confirm it reaches you, or wire up something better)
- [ ] Add a `workflow_dispatch` trigger so you can run the collector manually from the Actions tab
- [ ] Design a **backfill strategy**: when a scheduled run is missed (outage, disabled workflow, API downtime), how do you fill the gap? Ideally the collector accepts a date range, and idempotency makes re-running any period safe
- [ ] Test the backfill: manually skip a day (disable the workflow), then recover the gap using your strategy
- [ ] Watch it run on schedule for several consecutive days before moving on

**Phase 3 is done when:** you can show a row of green scheduled runs in the Actions tab that happened while your computer was off.

### Phase 4 — The Dashboard

A public page that makes months of numbers legible at a glance.

- [ ] Build a page that reads the cleaned table and renders **at least 3 chart types** (e.g., a time-series line for the main metric, a bar chart of aggregates by week or month, and a distribution or comparison chart)
- [ ] Add **date-range selection** so a visitor can zoom from "all time" to "last 7 days"
- [ ] Show a **stats summary**: latest value, all-time high/low, average over the selected range, total data points collected, and when the pipeline last ran
- [ ] Make it **mobile-readable** — charts resize, nothing requires horizontal scrolling, text is legible on a phone
- [ ] Attribute the data source visibly on the page

How the dashboard reads the database depends on your Phase 1 storage choice (see Hints): a SQLite file can be turned into static JSON at build time, while hosted Postgres wants a small API layer or a backend-rendered page. Neither is wrong; just make the read path deliberate.

**Phase 4 is done when:** you can hand your phone to someone, say "here's a year — sorry, a month — of <your data>," and they can explore it without instructions.

### Phase 5 — Ship & Accumulate

- [ ] Deploy the dashboard to a public URL
- [ ] Let the pipeline run unattended for **4+ consecutive weeks** — resist the urge to babysit; the point is proving it doesn't need you
- [ ] Add a **data-quality check** that catches anomalies: gaps in the date sequence, values outside data-dictionary ranges, duplicate keys, a run that inserted zero rows — and surface failures the same way collector failures surface
- [ ] Write the project README: what it collects, the architecture (with a diagram — fetch → validate → store → serve is four boxes and three arrows; Mermaid renders natively on GitHub), the data dictionary, and how to run everything locally

**Phase 5 is done when:** the Definition of Shipped below is fully checked — that's the same thing.

## Definition of Shipped

This capstone is done when every box is checked:

- [ ] The pipeline has run unattended, on schedule, for **4+ consecutive weeks**
- [ ] The dashboard is live at a public URL anyone can visit
- [ ] The data dictionary is published in the README
- [ ] The collector has automated tests covering validation and idempotency logic
- [ ] No secrets are committed anywhere in the repo's history — connection strings and keys live only in repository secrets and local `.env` files that are gitignored

Note the first box is a calendar requirement, not a code requirement — which is exactly why Phase 3 should ship as early as possible.

## Hints

Nudges, not answers. Reach for these when you're stuck, not before.

- **Choosing a data source.** Pick something you personally find interesting — you're going to look at this dashboard for months. Ideas, roughly in order of friction:

  | Source | Why it's good | Friction |
  |---|---|---|
  | **Weather history for your city (Open-Meteo)** | No API key required, generous limits, clean JSON. The best first choice. | Lowest |
  | Air quality (Open-Meteo AQ, OpenAQ) | Same ease, more socially interesting data | Low |
  | Currency or crypto rates | Numeric, frequent, easy to chart | Low |
  | GitHub stats of a favorite project (stars, issues, releases) | Official API, well documented | Low–medium |
  | A sports team's stats | Fun, seasonal patterns | Medium — check the API's terms carefully |
  | Local gas prices | Genuinely useful, unique | Medium–high — official APIs are rare; see below |

- **Play it straight: APIs, terms of service, and courtesy.** This part is not optional, and it's a professional habit rather than a legal technicality:

  - **Prefer official APIs over scraping — always.** An API is a published contract: the provider is telling you how they want to be accessed. Scraping HTML is fragile (it breaks when the page changes), often against the site's terms, and a bad look in an interview when you explain your architecture.
  - **Read the terms of service before you write a line of code.** Look specifically for: whether automated access is allowed, whether you may store and republish the data, and any attribution requirements. If the terms forbid what you're planning, pick a different source. There are plenty.
  - **Check `robots.txt`** if you're fetching anything that isn't a documented API endpoint. If it disallows the path, that's your answer.
  - **Respect rate limits — and stay far below them.** You're making one small request per scheduled run; there is no reason to ever be a burden. Set a descriptive `User-Agent`, don't retry in a tight loop, and if the API returns a 429, back off and let the next scheduled run try again.
  - **Attribute your source** on the dashboard and in the README. It's polite, often required, and makes your project look more professional, not less.

  If a source you love has no official API and scraping is your only option, choose a different source. The project is about the pipeline, not the data.

- **Questions to answer before committing to a source:**

  - Does it update at least daily? (Weekly data means a very slow feedback loop and a sparse dashboard.)
  - Is history *not* freely downloadable in bulk? If anyone can grab 10 years as a CSV, your accumulated copy is less special — still fine for learning, weaker as a differentiator.
  - Can you describe, in one sentence, what a single row of your cleaned table represents? ("One day of weather for one city.") If you can't, the source is too fuzzy.
  - Will you still care about this data in four months? Be honest — abandoned dashboards show their neglect.

- **Idempotency is a schema decision first, code decision second.** If your table has a uniqueness constraint on the natural key (say, `city + date`), duplicates are *impossible*, and your insert code just has to decide what to do on conflict — skip, or update. Look up your database's "upsert" (insert-or-update-on-conflict) support. Getting this guarantee from the schema is far more reliable than checking "does this row already exist?" in Python first, because the check-then-insert approach has a gap between the check and the insert.

- **Keep the raw responses.** Storage is cheap; hindsight is expensive. If you discover in month three that your cleaning logic mangled a field, raw responses let you rebuild the cleaned table from scratch. If you only stored cleaned data, those months are gone forever. Raw rows also make debugging "why does Tuesday look weird?" trivial: you can see exactly what the API said that day.

- **UTC everywhere; convert at the edges.** Store all timestamps in UTC in the database and convert to local time only when displaying. Time-series data with mixed or ambiguous timezones is quietly corrupted data — daylight-saving transitions alone will produce duplicate or missing hours if you store local time. Also be precise about what a "day" means: the API's day boundary and yours may differ, and off-by-one-day bugs in daily data are maddening to spot.

- **SQLite-in-repo vs. hosted Postgres — an honest tradeoff.** Two legitimate storage layers:
  - *SQLite committed to the repo:* zero cost, zero accounts, no credentials to leak. The Actions runner opens the file, appends rows, and commits it back. Downsides: your git history grows with every run, concurrent writers would collide (fine for a single daily job), and the dashboard needs a build step or small export to read it.
  - *Hosted Postgres:* the dashboard can query it live, no commit-the-database weirdness, and it's closer to how production systems actually work — but you manage credentials as secrets, and free tiers can idle, expire, or change terms mid-project. As of mid-2026, Neon is the safe free pick (it scales to zero when idle but doesn't expire, and your daily run wakes it right back up); avoid Render's free Postgres (expires after 30 days), and know that Supabase pauses free projects after 7 idle days.
  - SQLite-in-repo is charmingly simple for daily-granularity data; hosted Postgres is better practice for the job you want. Pick one *on purpose* and write down why — the reasoning is interview material either way.

- **"The API changed its response shape" — what it feels like.** One day a field you depend on is renamed, nested one level deeper, or switches from a number to a string. Without validation, the failure is silent and delayed: nulls or garbage flow into your cleaned table for weeks before a chart looks wrong enough to notice. *With* validation against your data dictionary, the collector fails loudly on day one, the workflow goes red, you get an email, and your stored data stays clean. That is the entire argument for Phase 2's validation step. It will happen to you eventually — treat it as a feature of the project, not a disaster.

- **Log for your future self.** In month two, when a run fails at 6 a.m., "Error" is useless, while "2026-09-14: fetched 2026-09-13, validation failed: temp_max_c=612.0 out of range [-60, 60]" solves the mystery in one glance. Every log line should answer: what period, what happened, how many rows.

- **Version your cleaning logic mentally.** When you change how a field is cleaned, note the date in the README. If a chart shows a step-change on that date, you'll know whether it's the world changing or your code changing — a distinction real data teams care about a lot.

- **Do the math in SQL, not in the browser.** The dashboard should ask the database for weekly averages, not download every row and average them in JavaScript. Aggregation is what SQL is *for*, and after six months of data the difference stops being theoretical. A handful of well-shaped queries (one per chart) keeps the page fast forever.

- **Secrets hygiene from commit one.** Add `.env` to `.gitignore` before the first commit, and commit an `.env.example` with placeholder values like `DATABASE_URL=<your-connection-string-here>` so the setup is documented without the secret. A key committed "just for a second" lives in git history permanently — the fix (history rewriting, key rotation) is far more painful than the prevention.

- **Start the clock early.** Nothing in Phases 4–5 blocks Phase 3. The moment the collector works, automate it — every day it runs is a day of data the finished dashboard gets for free. Building the dashboard against three weeks of real accumulated data (with its real gaps and quirks) also beats building it against three rows of test data.

## Stretch Goals

Only after the Definition of Shipped is fully met:

- **Anomaly alerts** — when the data-quality check trips (a gap, an out-of-range value, a zero-row run), send yourself an email or a Discord webhook message instead of waiting to notice the dashboard looks off.
- **Comparison overlays** — chart this week against the same week last year, or this month against the trailing 12-month average. This is where accumulated history starts paying visible dividends.
- **Public export endpoint** — serve the cleaned dataset as CSV and JSON from a documented URL. Now your project is a data *source* other people could build on.
- **A second data source, joined to the first** — e.g., weather joined with air quality for the same city and dates. Cross-source joins surface all the alignment problems (timezones, granularity, missing days) that real data work is made of.
- **Write it up** — find one genuine pattern in your data and publish a short post about it, with charts. "I collected this for six months and here's what I found" is a portfolio piece *and* a conversation starter.

## Telling the Story

This project's resume value compounds, so phrase it that way:

> Built and operate a scheduled data pipeline (Python, SQL, GitHub Actions) collecting daily <your data> since <month year> — <N>+ data points and counting — with schema validation, idempotent writes, and a public dashboard at <url>.

Update N and the start month each time you touch your resume. "Running unattended since March" is a claim that gets stronger every week you don't touch the code.

Interview angles to prepare:

- **The upstream change that broke collection.** At some point the API will hiccup or change — and your validation will catch it. That incident is your best story: how you found out (red workflow run, notification), what the failure actually was, how raw-response storage or your backfill strategy limited the damage, and what you changed afterward. If it genuinely never breaks, tell the design story instead: walk through what *would* happen if a field were renamed tomorrow, stage by stage, and where the failure would surface.
- **Why idempotency matters.** Explain what a retried or double-scheduled run would do to a naive pipeline, and how your natural-key-plus-upsert design makes reruns safe by construction rather than by care.
- **A tradeoff you made deliberately.** Your storage-layer choice (SQLite-in-repo vs. hosted Postgres) is perfect for this: two defensible options, and you can argue both sides before explaining your pick.
- **The scheduler-as-CI insight.** Explaining that a cron-triggered GitHub Actions workflow is the same machinery as CI — same YAML, same secrets model, same logs — shows you understand the tools rather than having followed a tutorial.

A 90-second demo script for interviews:

1. Open the live dashboard — point at the date axis: "this has been collecting since <month>."
2. Open the Actions tab — scroll the wall of green scheduled runs: "unattended, daily."
3. Open one run's log — show it reporting what it fetched and inserted.
4. Open the data dictionary in the README — "this is the contract every run is validated against."
5. Close with the incident story or the storage tradeoff, whichever conversation invites.

When it's live, put the dashboard URL in your GitHub profile README and your resume header. A recruiter who clicks it sees months of real, accumulating data — proof of follow-through that no snapshot project can fake.
