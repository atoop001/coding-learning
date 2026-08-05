# Testing, Debugging & Security Basics

A self-paced learning track for building **professional-quality habits**: writing code that is tested, debuggable, and defensively secure. The through-line is a mindset shift — from "it runs on my machine" to "I have evidence it works and stays safe when the world isn't friendly."

This is a **defensive, educational** track. The security material teaches you to find and fix vulnerabilities in **your own** code. You practice only on systems you own or have explicit permission to test.

## Who this is for & prerequisites

- You know **some JavaScript and/or Python** from earlier tracks — you can write functions, loops, and a small web app.
- You're on **Windows**; all setup uses PowerShell, VS Code, and Chrome/Edge — no exotic tooling.
- Everything is illustrated in **both** Python (pytest) and JavaScript (Vitest/Playwright); you can follow the language you're stronger in and use the other for contrast.

These habits are not a separate subject — **they apply to every other track you do.** Once tests, a debugger workflow, and a security checklist are part of how you build, every future project gets more solid for free.

## How to use this track

Study the **chapters in order** (each is primary study material, not a summary — read actively, run the code, do the exercises). Interleave the **projects** as you reach the chapters they draw on. Chapters are in `learning-docs/`; projects are in `projects/`.

Every chapter contains: Overview, Definitions & explanations, runnable commented Code examples (security chapters show *vulnerable → why → fixed*), Common pitfalls with corrections, and 3–5 Practice exercises (no solutions — the struggle is the learning). Every project contains: Description, Difficulty + effort, Chapters used, a Requirements checklist, Hints (nudges, not answers), and Stretch goals. **No project ships solution code** — you write it.

## Chapters (in order)

1. `learning-docs/01-why-testing-and-the-testing-mindset.md` — why tests exist, kinds of tests (unit/integration/e2e), the testing pyramid, the testing mindset.
2. `learning-docs/02-your-first-unit-tests.md` — installing & running pytest and Vitest, Arrange–Act–Assert, naming, assertions.
3. `learning-docs/03-designing-testable-code.md` — pure functions, side effects at the edges, dependency injection lite, seams.
4. `learning-docs/04-edge-cases-and-test-design.md` — boundaries, equivalence classes, error paths, parameterized tests.
5. `learning-docs/05-test-doubles-stubs-mocks-fakes.md` — stubs/mocks/fakes, mocking HTTP and time, when mocking goes too far.
6. `learning-docs/06-integration-and-end-to-end-testing.md` — real files & SQLite, Flask `test_client`, intro to Playwright, the cost model.
7. `learning-docs/07-debugging-fundamentals.md` — reading stack traces, reproducing bugs, binary-search debugging, rubber-ducking, print vs debugger.
8. `learning-docs/08-using-real-debuggers.md` — VS Code breakpoints for Python & Node, watches, conditional breakpoints, browser DevTools.
9. `learning-docs/09-systematic-debugging-and-prevention.md` — minimal repros, regression tests, logging well, assertions, `git bisect`.
10. `learning-docs/10-security-mindset-and-threat-basics.md` — what attackers want, trust boundaries, never trust input, defense in depth.
11. `learning-docs/11-common-web-vulnerabilities.md` — XSS, SQL injection, CSRF (vulnerable→fixed in your own apps), the OWASP Top 10 as a map.
12. `learning-docs/12-secrets-auth-and-safe-practices.md` — password hashing, env vars vs hardcoded secrets, `.gitignore`, HTTPS, `npm`/`pip` audit.

## Projects (easiest → hardest)

1. `projects/01-test-suite-for-a-utility-library.md` — write a professional test suite for a small spec'd utility library, in pytest *and* Vitest. *(Ch. 1, 2, 4)*
2. `projects/02-test-first-text-processing-module.md` — build a Markdown-lite converter strictly test-first. *(Ch. 2, 3, 4)*
3. `projects/03-debugging-gauntlet.md` — build a bug-prone Grade Book fast, then hunt and regression-test every bug systematically. *(Ch. 4, 7, 8, 9)*
4. `projects/04-mock-based-api-client-testing.md` — build & test a weather API client entirely with doubles; no network in the unit suite. *(Ch. 3, 4, 5, 6)*
5. `projects/05-security-audit-of-an-earlier-project.md` — threat-model, audit, and harden one of your own past web apps. *(Ch. 6, 9, 10, 11, 12)*
6. `projects/06-capstone-professional-quality-project.md` — take a real project to professional quality: full test pyramid, debugged, hardened, documented. *(all chapters)*

## Suggested cadence

A comfortable part-time pace is **8–10 weeks**; compress or stretch to taste.

| Week(s) | Chapters | Project work |
|--------|----------|-------------|
| 1 | 1–2 | Start Project 1 |
| 2 | 3–4 | Finish Project 1; start Project 2 |
| 3 | 5 | Finish Project 2 |
| 4 | 6 | Project 4 (mock-heavy — pairs with Ch. 5–6) |
| 5 | 7–8 | Start Project 3 (debugging gauntlet) |
| 6 | 9 | Finish Project 3 |
| 7 | 10–11 | Start Project 5 (audit) |
| 8 | 12 | Finish Project 5 |
| 9–10 | review all | Project 6 (capstone) |

Guidance, not law: do the exercises before moving on, don't rush the debugging and security chapters (they reward slow reading), and revisit earlier chapters when a project makes their ideas concrete.

## The habits this track leaves you with

- Write a failing test before you fix a bug — always.
- Read the whole stack trace before touching code.
- Never trust input; escape output; parameterize queries.
- Keep secrets out of source and git history.
- Claim "done" only with evidence you actually ran.

Carry these into every other track. That's the point.
