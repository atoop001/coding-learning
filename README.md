# Coding Language Learning Workspace

A self-paced workspace for learning programming languages and core developer
skills, aimed at web development and career readiness. Each track has its own
folder with two subfolders:

- **`learning-docs/`** — numbered chapters ordered beginner → advanced. Each chapter
  has thorough definitions, plain-English explanations, runnable code examples,
  common pitfalls, and practice exercises.
- **`projects/`** — numbered guided project specs, ordered by difficulty. Each spec
  bundles several chapters and gives you a description, requirements checklist,
  hints, and stretch goals — **no solution code**. You write everything.

## Tracks

**Languages**

| Track | What it's for |
|-------|---------------|
| `html-css/` | Structure and styling of every web page |
| `javascript/` | The core language of the web |
| `typescript/` | Professional typed JavaScript (learn after JS) |
| `python/` | Back-end web, automation, data, careers |
| `sql/` | Databases — storing and querying real data |
| `java/` | Enterprise back-ends, Android, big-company jobs |
| `csharp/` | .NET web, Windows apps, Unity (alternative to Java) |
| `rust/` | Systems programming — the advanced challenge track |

**Developer skills**

| Track | What it's for |
|-------|---------------|
| `git-github/` | Version control and the professional PR workflow |
| `command-line/` | Terminal fluency (PowerShell + Bash) |
| `how-the-web-works/` | HTTP, DNS, APIs — what actually happens between browser and server |
| `data-structures-algorithms/` | Efficiency, problem-solving, interview prep |
| `react/` | The most in-demand front-end framework (after JS + TS) |
| `node-express/` | Back-end JavaScript — build real APIs with Node.js + Express |
| `testing-debugging-security/` | Professional quality habits for every language |
| `deployment-devops/` | Docker, CI/CD, and cloud — shipping projects to real URLs |
| `ai-assisted-dev/` | Working with AI coding tools — verifying output, professional workflow, interviews |

**Capstones**

The `capstones/` folder holds 7 large cross-track project specs — finished,
shareable, portfolio-grade builds (a real business app, a public API, a data
pipeline, open-source contributions, and more). See `capstones/README.md` for
the list and suggested order. Start the first one (Portfolio & Blog) once you've
finished the node-express track's database and auth chapters.

## Recommended Learning Path

1. **Foundations** — `html-css/`, with `git-github/` and `command-line/` alongside
   (both pay off immediately and apply to everything).
2. **Core programming** — `javascript/`, with `how-the-web-works/` alongside.
   `python/` can also run in parallel; seeing the same concepts in two languages
   reinforces them.
3. **Professional web dev** — `typescript/`, then `sql/` and `node-express/`
   (back-end JS builds directly on both), with `testing-debugging-security/`
   alongside.
4. **Specialize & ship** — `react/` (front-end jobs),
   `deployment-devops/` (put your projects on real URLs — now a baseline hiring
   expectation), `data-structures-algorithms/` (interview prep), and
   `ai-assisted-dev/` (how to use and verify AI coding tools — 2026 interviews
   ask about this directly; its verification habits are worth skimming early).
5. **Broaden** — `java/` **or** `csharp/` (they teach similar concepts — pick one
   first), and finally `rust/`, the hardest track, once you're confident.

## How to Use This Workspace

1. Read a chapter, type out the examples yourself (don't copy-paste), and do the
   practice exercises.
2. When you reach a project whose "Chapters used" you've covered, build it. Work
   through the requirements checklist top to bottom; use hints only when stuck.
3. Keep your project code in the track's `projects/` folder (e.g.,
   `javascript/projects/03-todo-app/` next to the spec).
4. Each track's `README.md` lists its chapters and projects in order with a
   suggested cadence.

## Syncing Across Devices

This workspace lives at `https://github.com/atoop001/coding-learning`.

- Start each session: `rtk git pull`
- End each session: `rtk git add . && rtk git commit -m "what you did" && rtk git push`
- New device: `git clone https://github.com/atoop001/coding-learning.git`

## Progress

Track-level progress lives in `.claude/progress.json` and shows in the Claude Code
statusline. Tell Claude "mark X done" as you finish chapters and projects.
