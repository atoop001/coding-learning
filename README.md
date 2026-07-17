# Coding Language Learning Workspace

A self-paced workspace for learning programming languages, aimed at web development
and career readiness. Each language has its own folder with two subfolders:

- **`learning-docs/`** — numbered chapters ordered beginner → advanced. Each chapter
  has thorough definitions, plain-English explanations, runnable code examples,
  common pitfalls, and practice exercises.
- **`projects/`** — numbered guided project specs, ordered by difficulty. Each spec
  bundles several chapters together and gives you a description, requirements
  checklist, hints, and stretch goals — **no solution code**. You write everything.

## Recommended Learning Path

1. **`html-css/`** — start here. Every web page is built on HTML and CSS, and it's
   the fastest way to see visible results.
2. **`javascript/`** — the core language of the web. Start once you're comfortable
   building static pages.
3. **`python/`** — can run *alongside* JavaScript. Seeing the same concepts
   (variables, loops, functions) in two languages reinforces them. Also your route
   into back-end web development.
4. **`typescript/`** — last. It builds directly on JavaScript, so finish the JS
   track (or most of it) first.

## How to Use This Workspace

1. Read a chapter, type out the examples yourself (don't copy-paste), and do the
   practice exercises.
2. When you reach a project whose "Chapters used" you've covered, build it. Work
   through the requirements checklist top to bottom; use hints only when stuck.
3. Keep your project code in the language's `projects/` folder (e.g.,
   `javascript/projects/03-todo-app/` next to the spec).
4. Each track's `README.md` lists its chapters and projects in order with a
   suggested cadence.

## Adding More Languages

Ask Claude to add a new track (e.g., "add a Go track") — it will follow the same
structure: `<language>/learning-docs/` + `<language>/projects/`.

## Progress

Track-level progress lives in `.claude/progress.json` and shows in the Claude Code
statusline. Tell Claude "mark X done" as you finish chapters and projects.
