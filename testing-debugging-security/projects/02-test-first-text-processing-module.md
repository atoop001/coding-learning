# Project 2: Test-First Build of a Markdown-Lite Converter

## Description

In Project 1 you wrote tests for existing code. Now flip the order: build a module *test-first* — for every feature, write a failing test, then write just enough code to pass it, then clean up. The target is a "Markdown-lite" converter: a function that turns a small subset of Markdown into HTML. Text processing is ideal TDD territory: rich edge cases, pure functions, instant feedback. Pick ONE language (Python + pytest or JS + Vitest) and go deep.

`render(text)` supports, in this order of implementation:

1. Plain paragraphs: blank-line-separated text → `<p>...</p>`
2. Headings: `# Title` → `<h1>Title</h1>`, `##`/`###` likewise (max 3 levels)
3. Bold: `**word**` → `<strong>word</strong>`
4. Italic: `*word*` → `<em>word</em>` (careful: must not break bold!)
5. Unordered lists: consecutive `- item` lines → `<ul><li>item</li>...</ul>`
6. Inline code: `` `code` `` → `<code>code</code>` — with **no** bold/italic processing inside
7. Escaping: literal `<`, `>`, `&` in input must be HTML-escaped in output (a preview of Chapter 11's XSS thinking — user text must never become markup)

## Difficulty

**Beginner-to-intermediate.** Estimated effort: 6–9 hours across several sessions.

## Chapters used

- Chapter 2 — Writing Your First Unit Tests
- Chapter 3 — Designing Testable Code (pure core; `render` does no I/O)
- Chapter 4 — Edge Cases & Test Design
- Chapter 7 — Debugging Fundamentals (you *will* debug regex/parsing surprises)

## Requirements checklist

- [ ] Strict red-green cycle for every feature: commit history (or a `TDD-LOG.md`) shows the test existed and failed before the code that passes it
- [ ] `render` is a pure function: string in, string out, no printing, no file access
- [ ] Features 1–7 implemented in order, each with its own `describe`/test group
- [ ] Edge cases per feature, including at minimum: empty string; heading with no space (`#Title` — decide!); `####` (beyond max level); unclosed `**bold`; single `*` alone; empty list item; adjacent lists separated by a blank line
- [ ] The bold/italic interaction has dedicated tests (`***both***`, `**a *b* c**` — decide and document the supported behavior)
- [ ] Inline code protection tested: `` `**not bold**` `` stays literal inside `<code>`
- [ ] Escaping tested with hostile-looking input: `render("<script>alert(1)</script>")` produces escaped text, not a script tag
- [ ] At least one parameterized table test with 8+ rows
- [ ] At least one bug you created along the way is documented in `TDD-LOG.md`: symptom, how a test caught it (or should have), the fix
- [ ] Final suite: 30+ tests, all green, runs in under 2 seconds

## Hints

- Resist implementing ahead of the tests — the discipline feels slow for a day, then the safety net starts paying: refactoring feature 3's regex while features 1–2 stay verified is the whole experience this project exists to give you.
- Order of transformations matters enormously (escape first? code spans first?). When an ordering bug bites, that's a feature: write the failing test, then fix.
- Keep a "test ideas" scratch list; while implementing one case you'll think of three more. Don't chase them immediately — list them, finish the current cycle.
- `repr()`/`JSON.stringify` your actual-vs-expected strings when a test fails mysteriously — invisible whitespace and newline differences are the classic culprits here.
- If a regex becomes unreadable, that's a design signal (Chapter 3): consider splitting into smaller passes, each individually testable.

## Stretch goals

- Add links: `[text](url)` → `<a href="url">text</a>` — with the URL scheme allowlisted to `http`/`https` (reject `javascript:` — Chapter 11 foreshadowing) and tests proving it.
- Add a tiny CLI shell (`python md.py input.md > out.html`) and one integration test using `tmp_path` — keeping the core pure.
- Fuzz it: feed 500 random ASCII strings to `render` and assert it never throws and the output never contains a raw unescaped `<` from input text.
