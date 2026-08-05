# Project 6: Capstone — Bring a Past Project to Professional Quality

## Description

The capstone integrates everything in the track. Take one real project you care about — ideally the same app you audited in Project 5, or a comparably substantial one — and bring it to the standard you'd be comfortable putting on a resume or handing to another developer: a genuine test suite across the pyramid, systematically debugged, security-hardened, well-logged, and documented. This is not a from-scratch build; it's a *transformation*, and the discipline is doing it in reviewable increments, each verified, never claiming "done" without evidence (Chapter 9). The output is a project whose README you could show anyone and a git history that tells the story of it becoming professional.

## Difficulty

**Advanced (capstone).** Estimated effort: 15–25 hours over one to two weeks.

## Chapters used

**All twelve.** Specifically: pyramid & mindset (1), unit tests (2), testable design (3), edge cases (4), doubles (5), integration/E2E (6), debugging (7–8), prevention/logging/assertions (9), and the full security trio (10–12).

## Requirements checklist

**Testing — build the pyramid (Chapters 1–6)**
- [ ] Unit tests covering the core logic, with edge/boundary/error cases per Chapter 4 (the wide base)
- [ ] Integration tests for real seams: DB access against a temp/in-memory database, file I/O via `tmp_path`, and (for web apps) route tests via `test_client`
- [ ] At least 2 end-to-end tests (Playwright for a web UI, or full CLI-invocation tests) covering the top user journeys — and a written justification of why *those* journeys
- [ ] External dependencies (APIs, clock, randomness) isolated behind injected seams and covered with appropriate doubles
- [ ] Meaningful coverage (aim ~80%+ on core modules) with a written note on what the uncovered code is and why it's acceptable
- [ ] Whole suite runs with one command per language and is fast enough to run often (state the number)
- [ ] A GitHub Actions workflow runs the full test suite on every push — a red build stops the story here; treat it like a failing test, not a formality

**Debugging & robustness (Chapters 7–9)**
- [ ] Structured logging added (levels, values in messages, `log.exception` in catch blocks); a `LOGGING.md` note on what runs at INFO vs DEBUG
- [ ] Assertions guarding at least three real invariants; input validation (raise/400) clearly distinguished from assertions
- [ ] A `BUGS-FIXED.md` documenting at least three bugs found during hardening, each with a regression test that failed first
- [ ] No swallowed exceptions, no leftover debug prints, no `debugger;` statements (grep to prove it)

**Security hardening (Chapters 10–12)**
- [ ] The Project-5 audit checklist applied (or re-applied) with all High/Critical findings fixed and locked with tests
- [ ] Passwords via bcrypt/argon2; secrets in `.env` + environment with `.env.example` and `.gitignore`; session cookies hardened; parameterized queries only; output escaping verified
- [ ] `pip-audit`/`npm audit` clean or triaged, with a note on your update cadence going forward
- [ ] A one-page `SECURITY.md` summarizing the threat model and the defenses in place

**Documentation & delivery**
- [ ] A README a stranger could use: what it does, setup on Windows, how to run it, how to run the tests, environment variables required
- [ ] Git history in small, verified, well-messaged commits — the story of the transformation
- [ ] A closing `RETROSPECTIVE.md`: what the project looked like before vs after, the three habits that made the biggest difference, and what you'd still improve with more time

## Hints

- Sequence deliberately: get the app under a basic test suite *first* (a safety net), then debug, then refactor for testability/security, then add integration/E2E on top. Hardening untested code is how you introduce new bugs.
- Every claim of "done" needs evidence you actually ran (Chapter 9's verification habit): paste the passing command output into the relevant doc rather than asserting from memory. This is exactly the professional standard the track is teaching.
- Increments over heroics: one behavior, one test, one commit. A capstone done in a single 20-hour marathon is neither reviewable nor honest about what works.
- Let testability and security reinforce each other — the seams you add for doubles (Chapter 3) are the same seams that let you inject a hardened dependency; the input validation you add for security is the same code your error-path tests exercise.
- Right-size the effort (Chapter 5's over-mocking / Chapter 10's paranoia warnings both apply): this is a real project made solid, not a NASA rewrite. Aim for "another developer would trust this," not "provably perfect."
- Use the todo list / your progress file to track the four workstreams; the capstone is as much a project-management exercise as a technical one.
- A CI workflow file has three parts: a **trigger** (`on: push` — when does this run), a **job** running on a fresh VM (`runs-on: ubuntu-latest` — what environment), and **steps** in order (checkout the code, install dependencies, run the suite). If any step exits non-zero, the whole run is marked failed — that's your red build.

## Stretch goals

- Add `audit`/`pip-audit` to the same CI workflow so dependency vulnerabilities also fail the build, not just test failures.
- Add coverage gating and a status badge to the README.
- Write a `CONTRIBUTING.md` describing your testing and security expectations for contributors — teaching the standard is the deepest form of learning it.
- Present the before/after to someone (or record a short walkthrough): explaining the transformation out loud is the final rubber-duck that reveals whatever you still don't fully understand.
