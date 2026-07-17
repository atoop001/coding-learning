# Language Learning Workspace — Design

**Date:** 2026-07-17
**Status:** Approved

## Purpose

A self-paced workspace for learning programming languages, aimed at a learner with
some basics (has dabbled, wants a structured path) whose goals are web development
and career/employability. Each language gets a folder containing ordered learning
docs (beginner → advanced) and guided project specs that exercise the material.

## Languages (initial set)

| Track | Chapters | Projects | Notes |
|-------|----------|----------|-------|
| html-css | ~14 | 6–8 | Foundation track; HTML and CSS combined |
| javascript | ~18 | 6–8 | Core web language |
| python | ~18 | 6–8 | Back-end web, automation, career demand |
| typescript | ~12 | 6–8 | Assumes JavaScript knowledge; starts at types |

Recommended learning order: HTML/CSS → JavaScript (Python can run alongside) → TypeScript last.

## Folder structure

```
D:\atoop\coding-projects\
├── README.md                  # learning path + how to use the workspace
├── html-css\
│   ├── learning-docs\         # 01-….md, 02-….md (numbered chapters)
│   └── projects\              # 01-….md (numbered by difficulty)
├── javascript\  (same layout)
├── typescript\  (same layout)
└── python\      (same layout)
```

## Learning doc format (each chapter)

Numbered markdown files (`01-variables.md` style), one topic area per file, ordered
as a learning path. Every chapter contains:

- **Overview** — what the topic is and why it matters
- **Definitions & explanations** — thorough, plain-English, assumes only prior chapters
- **Code examples** — runnable, commented, progressing from simple to realistic
- **Common pitfalls** — mistakes beginners actually make
- **Practice exercises** — 3–5 small exercises (no solutions)

## Project spec format (guided specs, no solutions)

Numbered markdown files ordered by difficulty. Each project bundles several
chapters so it is fully fleshed out rather than a one-topic toy. Every spec contains:

- **Description** — what you're building and what it should feel like to use
- **Chapters used** — which learning docs it exercises
- **Requirements checklist** — concrete, checkable requirements
- **Hints** — nudges for the tricky parts, not answers
- **Stretch goals** — optional extensions
- **No solution code** — the learner writes everything

## Generation approach

Four parallel general-purpose subagents, one per language track (the tracks are
fully independent). The orchestrator writes the root README and progress tracker,
then commits the workspace to git.

## Progress tracking

`.claude/progress.json` tracks the learner's journey per track (read chapters,
complete projects), rendered by the statusline. Milestones M1–M4 map to the four
tracks in recommended learning order.
