# Capstone 4: Open-Source Contribution Program

Three merged pull requests to real open-source projects you did not write, climbing a
difficulty ladder: a documentation fix, a bug fix with a test, and a small feature.

This is not an app you build. It is a program you run. The deliverable is your public
GitHub contribution history — a set of green "Merged" badges on other people's
repositories that any hiring manager can click and verify in thirty seconds.

If you completed the deployment-devops capstone, you have already done the hardest part
once: you adopted a codebase you didn't write, got it running, and fixed one thing. This
capstone turns that one-time exercise into a repeatable skill — and makes the results
public.

## Why This Project

Ask hiring managers their #1 complaint about junior developers and you will hear the same
answer over and over: "They can build toy projects, but they can't work in an *existing*
codebase." Every solo project on your GitHub — no matter how polished — leaves that
question unanswered. You wrote all of it. Of course you can navigate it.

A merged PR in a stranger's codebase answers it. Publicly. It proves, with evidence no
interviewer has to take your word for:

- **Code reading.** You oriented yourself in tens of thousands of lines of unfamiliar
  code, found the right place to make a change, and changed only that.
- **Communication.** You described a problem and a fix clearly enough that a busy
  maintainer — who owes you nothing — chose to spend their time reviewing it.
- **Professional git workflow.** Fork, branch, rebase, force-push-with-lease, respond to
  review, squash. The actual workflow of a real team, not the solo `git commit -m "stuff"`
  loop.
- **Receiving code review.** Someone with more experience critiqued your work in public
  and you responded professionally, made the changes, and got it over the line. This is
  the single most job-like thing you can do without having a job.

No solo project can prove any of these. Three merged PRs prove all four, three times.

## Difficulty & Estimated Effort

**Intermediate–Advanced.** Roughly **15–30 hours of your own work, spread over 1–2
months.**

The spread matters more than the hours. Maintainers review PRs on their own schedule —
a response can take three days or three weeks. That makes this a **background project**:
do not block on it. Run it alongside your other tracks and capstones. Submit a PR, set a
reminder to check on it, and go work on something else. The rhythm looks like short,
intense bursts (orienting, writing the change) separated by long quiet stretches
(waiting for review).

Plan for setbacks. A PR getting rejected or going stale is a normal outcome, not a
failure state — which is why the milestones below have you pick backup projects up front.

## Prerequisites

- **git-github track — the whole thing**, especially the pull-request workflow: forking,
  branching, keeping a fork in sync with upstream, rebasing, and amending commits after
  review. You will use all of it, in public, where mistakes are visible.
- **command-line track.** You'll be running unfamiliar build tools, test runners, and
  scripts from README instructions written for someone who already knows the project.
- **At least one language track completed** (javascript, python, typescript, etc.). You
  need one language you're comfortable reading at length, and you'll pick target projects
  written in it.
- **testing-debugging-security track — strongly recommended before Rung 2.** The bug-fix
  rung requires reproducing a bug and writing a failing test first. If you haven't
  practiced that in your own code, doing it for the first time in a stranger's codebase
  is much harder.

## Phased Milestones

### Phase 1: Target Selection

Choosing the right projects is half the battle. A great contribution to a dead repo
merges never; a mediocre first attempt at a welcoming project gets shepherded to merge
by friendly maintainers.

Selection criteria — a good target project is one where **all** of these hold:

- **You actually use it.** A CLI tool, library, framework, or docs site that's already in
  your daily workflow. You'll spot real gaps because you hit them yourself, and your
  motivation survives the slow parts.
- **Small enough to hold in your head** — ideally under ~50k lines of code. You can check
  with a line-count tool or just by browsing: if the `src/` directory has hundreds of
  files, look for something smaller for your first rungs.
- **Active maintainers who merge outsiders' PRs.** Look at the last 20 merged PRs. Are
  any from people who aren't core team? How long did review take? Recent activity from
  outside contributors is the single best predictor that yours will get attention.
- **A `CONTRIBUTING.md` exists.** Its presence signals the project *wants* contributions
  and tells you exactly how to make them acceptable.
- **Labeled `good first issue` / `help wanted` issues exist.** Even if you don't take one
  for Rung 1, their existence means maintainers deliberately leave room for newcomers.

Checklist:

- [ ] List every open-source tool and library you personally use (editor plugins, CLI
      tools, libraries from your learning tracks, docs sites — everything)
- [ ] Filter that list against the five criteria above; shortlist candidates
- [ ] Pick **3 candidate projects** and write **one paragraph each** in
      `contribution-log.md` explaining why it qualifies: how you use it, rough size,
      evidence of outsider PRs being merged, link to its CONTRIBUTING.md, link to its
      good-first-issue label
- [ ] Rank them: primary target for Rung 1, plus two backups
- [ ] Fork and clone your primary target; get it building and its test suite passing
      locally on your Windows machine (note: some projects assume Linux — if the dev
      setup fights you on Windows, WSL or a devcontainer may be the path of least
      resistance, and your deployment-devops capstone prepared you for exactly this)

### Phase 2: Rung 1 — Documentation PR

The goal of this rung is not the typo fix. It's learning to orient in a codebase you've
never seen — and the docs PR is proof you did. **The orientation log matters as much as
the PR.**

- [ ] Spend a focused session orienting in the codebase, and **document HOW you
      oriented** in `contribution-log.md` as you go: where you started (README?
      entry-point file? tests?), what you read in what order, what confused you, what
      finally made the structure click. This log is interview gold — "walk me through how
      you approach an unfamiliar codebase" is a real question, and you'll have a real
      answer.
- [ ] Find a **real** documentation gap — not a manufactured one. Good sources: something
      that confused *you* during setup, a README instruction that's outdated, a missing
      or broken code example, an error message the docs don't explain, a typo cluster.
      If you got stuck during Phase 1 setup, the fix for whatever stuck you is probably
      your PR.
- [ ] Check the issue tracker and open PRs first — make sure nobody has already fixed it
- [ ] Read `CONTRIBUTING.md` end to end and **follow it exactly**: branch naming, commit
      message format, PR template, sign-offs, changelog entries — whatever it asks
- [ ] Make the change on a branch, verify it renders/builds correctly, and open the PR
      with a short, clear description of what was wrong and why the new version is better
- [ ] Respond to every review comment within a day or two; make requested changes
      graciously, even trivial-seeming ones
- [ ] **Get it merged.** Record the PR link in `contribution-log.md`

### Phase 3: Rung 2 — Bug Fix with a Test

Now you change behavior, not just words. This can be in the same project or a different
one from your shortlist.

- [ ] Browse the issue tracker for **confirmed, reported bugs** — ideally ones with
      reproduction steps, a maintainer acknowledgment, or a `bug` + `good first issue`
      label combination. Avoid vague reports ("it's slow sometimes") and anything
      touching core architecture.
- [ ] **Reproduce the bug locally** and record your reproduction steps in
      `contribution-log.md`. If you can't reproduce it, pick a different bug — never fix
      what you can't see fail.
- [ ] Comment on the issue saying you can reproduce it and would like to work on a fix
      (this both claims it and adds value — a confirmed repro helps the maintainers even
      if you never finish)
- [ ] **Write the failing test first.** A test that fails for exactly the reason the
      issue describes, in the project's existing test style and framework. Run it; watch
      it fail.
- [ ] Write the smallest fix that makes the test pass without breaking the rest of the
      suite
- [ ] Open the PR: link the issue (`Fixes #123`), describe the root cause in a sentence
      or two, describe the fix, and point at the test. Root-cause-then-fix descriptions
      are what separate professional PRs from drive-by patches.
- [ ] Respond to review, iterate, **get it merged**, record the link

### Phase 4: Rung 3 — Small Feature or Good-First-Issue

The top rung adds behavior that didn't exist. The critical new skill here is
**discussing design before writing code** — the thing juniors most often skip and most
regret skipping.

- [ ] Find a labeled `good first issue` (or a small, maintainer-approved feature request)
      in one of your target projects
- [ ] **Claim it by commenting first.** Ask if it's still wanted and whether anyone is
      working on it. Wait for a go-ahead — some projects assign issues, some just say
      "go for it"
- [ ] **Discuss your approach with the maintainers BEFORE coding.** Two or three
      sentences in the issue thread: "I'm planning to add X in module Y, following the
      pattern used by Z — does that sound right?" Five minutes of alignment here saves
      you from building the wrong thing and having a week of work rejected.
- [ ] Implement in small, reviewable commits, **with tests**, matching the project's
      existing patterns and style — even where you'd personally do it differently
- [ ] Open the PR with: link to the issue, summary of the agreed approach, notes on
      anything you deviated from and why, and what you tested
- [ ] Respond to review — expect more rounds than the earlier rungs; features attract
      more scrutiny than fixes
- [ ] **Get it merged.** Record the link. You're done — and you're now demonstrably
      someone who can be handed a ticket in an existing codebase and ship it.

## Definition of Shipped

This capstone is shipped when every box below is checked:

- [ ] **Three merged PRs**, one per rung (docs → bug fix with test → small feature), in
      real open-source projects you did not write, with all three links recorded
- [ ] `contribution-log.md` exists in this folder (`capstones/04-open-source-contributions/`)
      and contains, for each project you worked in:
  - [ ] The Phase 1 candidate paragraphs (why you chose each target)
  - [ ] Your **orientation process** — how you learned your way around, written while it
        was happening, not reconstructed later
  - [ ] Every substantive piece of **review feedback you received and how you addressed
        it** (quote or paraphrase the comment, then what you changed)
  - [ ] **Lessons learned** — what you'd do differently on the next contribution
- [ ] All three PRs are visible on your public GitHub profile (they will be, if your
      commits use the email tied to your GitHub account — check this early, not after
      three merges)

## Hints

**Reading an unfamiliar codebase.** Don't try to read it top to bottom — nobody does.
Find the entry point (`main`, `index`, the CLI's argument parser, the server's route
table) and follow **one single request or command all the way through the stack**, taking
notes as you go. One complete vertical slice teaches you more than skimming every
directory. And read the **tests as documentation**: a test file for a module is a list of
worked examples of how that module is supposed to behave, written by the people who wrote
the module. Often clearer than the docs.

**Etiquette.** Keep PRs small — one concern per PR, always. A 20-line PR gets reviewed
this week; a 400-line PR gets reviewed never. Match the project's existing style even
when you disagree with it, and **never argue with maintainer style preferences** — their
house, their rules, and "you're right, changed" costs you nothing. Say thank you when
review comments teach you something; reviewers remember pleasant contributors.

**When a PR sits ignored for weeks.** This will happen and it isn't personal —
maintainers are usually unpaid volunteers with day jobs. After about two weeks of
silence, leave **one polite ping**: "Gentle bump — happy to make any changes needed."
Then, whether or not it lands, **move on to one of your backup projects** and keep the
program moving. This is exactly why Phase 1 had you pick three candidates. A stale PR
can still merge months later; your progress shouldn't wait for it.

**Handling rejection.** Some PRs get closed. Common reasons: it duplicated planned work,
it didn't fit the project's direction, or the maintainer just didn't want that change.
Respond with grace — "understood, thanks for taking a look" — and log what you learned in
`contribution-log.md`. A politely-handled rejection is still a public artifact of
professionalism, and interviewers can see how you behaved. The only truly bad outcomes
are arguing in the thread or deleting the evidence.

**On Windows specifically.** Before investing in a target project, check its
CONTRIBUTING.md and CI config for Windows support. If the dev environment is
Linux-assumed, that's not a dealbreaker — WSL exists — but factor the setup cost into
your Phase 1 ranking.

## Stretch Goals

- **Become a repeat contributor to one project.** A second and third merged PR in the
  same repo changes your story from "contributed once" to "contributes to" — present
  tense. Some projects grant triage or commit rights to reliable regulars.
- **Triage someone else's issue.** Pick an unconfirmed bug report, try to reproduce it,
  and comment with your findings (environment, steps, result). Maintainers deeply value
  this, and it's contribution without writing a line of code.
- **Review someone else's PR.** Leave a genuinely useful comment on an open PR — a bug
  you spotted, a test case that's missing, a confirmation that it works on Windows.
  Being on the giving end of review teaches you what reviewers look for in yours.
- **Write a short blog post per contribution** — what the project is, what you changed,
  what the review taught you. If you built the blog in Capstone 2, this is exactly what
  it's for; each post links the merged PR, and each PR-shaped story compounds your
  public evidence.

## Telling the Story

**Resume bullets — link the PRs directly.** Merged PRs are the rare resume claim that is
instantly verifiable, so make them one click away:

- "Contributed bug fix with regression test to <project> (12k stars) — merged: <PR link>"
- "Implemented <small feature> in <project> after design discussion with maintainers —
  merged: <PR link>"
- "Three merged PRs across open-source projects in <language>: <profile or gist link>"

Put the links in the PDF, on your GitHub profile README, and on LinkedIn. "Open-source
contributor" with no links is noise; with links it's proof.

**The interview narrative.** When asked about working with others' code — and you will
be — don't recite the ladder. Instead, **pick the hardest review comment you received and
tell that story**: what the maintainer pushed back on, what you initially thought, what
you came to understand, and what you now do differently because of it. That one story
demonstrates code reading, humility, communication, and growth in ninety seconds — and
almost no other junior candidate has one. Your `contribution-log.md` is where that story
is kept fresh; this is why you wrote things down while they happened.

The quiet superpower of this capstone: when an interviewer asks "have you ever worked in
a large existing codebase?", you don't say yes. You send a link.
