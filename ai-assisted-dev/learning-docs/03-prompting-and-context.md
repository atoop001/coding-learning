# Chapter 3: Prompting and Context

## Overview

Verification (Chapter 2) is what makes AI output safe to use. This chapter is about what makes it *good* to begin with, so you spend less time verifying garbage: giving the tool the context it actually needs, working in a loop instead of one giant ask, and breaking large tasks into pieces a model can reliably handle. It also covers a distinction most learners miss — prompting to get *output* versus prompting to *learn* are different skills with different techniques — and closes with the harder question: when the right move is not to prompt the AI at all.

## Definitions & Explanations

**Context is the model's entire world for that request.** An LLM has no persistent memory of your codebase, your team's conventions, or your intent unless you (or the tool) put it in front of the model *this time*. Every gap between "what I meant" and "what I typed" gets filled in by the model's best statistical guess — which is often wrong in ways that look confident (Chapter 1). Good prompting is mostly the discipline of not leaving gaps you could have filled.

**What to include as context, roughly in order of how often it's forgotten:**

- **Constraints** — language/framework version, style conventions, what you're not allowed to change (e.g., "don't touch the function signature, other code calls it"), performance or dependency limits ("no new npm packages").
- **Existing code** — the actual function/file/class you're modifying, not a description of it. A model given the real code writes code that fits; a model given a paraphrase writes code that fits the paraphrase.
- **Error output, verbatim** — the full stack trace or error message, not "it's throwing an error." The exact text often contains the one detail (a line number, a type name, an error code) that turns a guess into a diagnosis.
- **The actual goal, not just the mechanism** — "add input validation to this form" gives the model far less to work with than "reject empty names and emails without an @, and show the error inline without a page reload." Say what "done" looks like.

**Iterating vs. restarting.** When output is close but not right, you have two moves: refine in place ("keep the structure, but handle the empty-list case") or start over with a better initial prompt. Iterating preserves context the model already has and is usually faster for small misses. Restarting is often better when the *approach* itself is wrong — an agent bolting fixes onto a flawed foundation tends to produce more tangled code than one prompt that states the constraint up front. A rough signal: if you're on your third patch to the same fundamental issue, stop patching and restate the problem with that constraint included from the start.

**Decomposing tasks.** Large, vague asks ("build me a login system") give the model maximum freedom to guess wrong about things you actually care about — password rules, session handling, error messages — and maximum surface area for you to verify (Chapter 2) once it's done. Smaller, sequenced asks ("write the password-hashing function first; once that's right, wire it into the login route") keep each step small enough to actually read and test before moving on. This isn't just a prompting tactic — it's the verification workflow's prerequisite. You can't apply Chapter 2's five steps carefully to five hundred lines that landed all at once.

**Prompting for output vs. prompting for learning.** These pull in different directions and it's worth naming both:

- *Prompting for output*: "write me X." Optimized for getting working code fast. Appropriate once you understand the problem and want the typing done.
- *Prompting for learning*: "explain what this regex does, piece by piece," "quiz me on closures until I get three right in a row," "review my solution and tell me what's fragile about it — don't rewrite it." Optimized for building understanding you'll still have without the tool. This is a legitimate, deliberate use of the same tools and belongs in your regular rotation, not just when stuck — see Chapter 5.

Mixing the two without noticing is the trap: asking for output when you actually needed to learn the underlying concept gets you code that works today and a gap that costs you in an interview or on the next task that's 10% different.

**When *not* to use AI.** Two categories are worth calling out explicitly, because the pressure to reach for AI by default is real and not always right:

- **Fundamentals you're still learning.** If you're two weeks into learning loops, asking AI to write the loop skips the exact rep your brain needed. This isn't a purity rule — it's that the entire value of practicing fundamentals is the struggle, and outsourcing the struggle outsources the learning (Chapter 5 goes deeper on this).
- **Security-critical code you can't yet evaluate.** Authentication, authorization, cryptography, payment handling, anything touching secrets. AI can absolutely help you learn *about* these, but generating production code you cannot yet independently verify (because verifying it requires expertise you don't have yet) inverts the whole model this track teaches: you'd be shipping code whose correctness you're trusting rather than checking. Learn the fundamentals of the specific area first (`testing-debugging-security/` chapters 10–12), then use AI on it with real verification ability behind you. Watch specifically for *insecure-by-default* output: a hallucinated API call throws an error and announces itself; an insecure pattern like string-concatenated SQL just runs, correctly, on every input you happen to try (see the example below).

## Code Examples

### Vague prompt vs. context-rich prompt, same task

```text
VAGUE:
"Add validation to my signup form."

CONTEXT-RICH:
"Add client-side validation to the signup form in SignupForm.jsx
(pasted below). Requirements: email must contain '@' and a '.'
after it; password must be 8+ characters; both fields show an
inline error message below the input on blur, not on every
keystroke. Don't add a new npm dependency — use plain JS. Here's
the current component: [paste SignupForm.jsx]"
```

The second version removes almost every decision the model would otherwise have to guess at — and every guess it makes is a spot Chapter 2's workflow now has to catch.

### Decomposing a task instead of asking for it whole

```text
INSTEAD OF:
"Build a REST API for a todo app with a database, auth, and tests."

TRY, IN SEQUENCE:
1. "Design the Todo model fields — just the schema, no code yet.
    I want title, done, created_at, and owner_id."
2. "Now write the SQLAlchemy model for that schema."
   [read it, run it, test it -- Chapter 2 -- before continuing]
3. "Now write the GET /todos route that returns only the
    logged-in user's todos, using this model." [paste model]
   [verify again before continuing]
4. ...
```

Each step is small enough to actually apply the five-step workflow to before the next one builds on it — which also means a wrong assumption gets caught at step 2, not discovered underneath three more layers at step 4.

### Prompting for learning instead of output

```text
INSTEAD OF:
"Write a function that debounces a callback in JavaScript."

TRY:
"I want to write a debounce function myself. Explain the concept —
what problem it solves and the general approach — but don't give
me the code. I'll write it and show you my attempt for feedback."

... [you write your attempt] ...

"Here's my debounce function: [paste]. Don't fix it — tell me what's
wrong with it and why, and I'll try again."
```

This gets you the understanding a solved problem can't; the code you eventually land on is one you can actually explain (which matters a great deal in Chapter 6).

### Iterating vs. restarting, recognized in the moment

```text
# Iterate (small, local miss):
"Good, but this crashes on an empty list. Handle that case."

# Restart (the approach itself was wrong):
"Stop -- this is doing the filtering in JavaScript after fetching
all rows, which won't scale. Let's restart: I need the filtering
done in the SQL query itself. Here's the table schema: [paste]."
```

### Insecure-by-default: code that runs fine but shouldn't ship

Not every AI mistake looks like a mistake. Some output executes without error, returns the right answer on the inputs you happen to test, and is still wrong in a way that only shows up under attack:

```python
# AI-GENERATED (runs fine, returns correct results, still vulnerable):
query = f"SELECT * FROM users WHERE username = '{username}'"
cursor.execute(query)

# FIXED -- parameterized: SQL text and data travel separately.
query = "SELECT * FROM users WHERE username = ?"
cursor.execute(query, (username,))
```

The takeaway: hallucinations announce themselves by breaking. Insecure defaults don't -- you have to already know what a parameterized query looks like to notice the first version is wrong at all. See `testing-debugging-security/learning-docs/11-common-web-vulnerabilities.md` for the full SQL-injection pattern, and the equivalent problem for unescaped output.

## Common Pitfalls

- **Describing code instead of pasting it.** "I have a function that takes a list and sorts it somehow" gives the model nothing to actually fit its answer to. Correction: paste the real code, every time it exists.
- **Omitting the error message and re-explaining it in your own words.** Your paraphrase drops the one specific detail (exact exception type, line number) that would have made the diagnosis immediate. Correction: paste the full, verbatim error/stack trace.
- **Asking for everything at once, then being unable to review any of it carefully.** A giant multi-file agentic task that "worked" is much harder to verify than five small ones. Correction: decompose, and apply Chapter 2 between steps, not just at the end.
- **Patching a fundamentally wrong approach three, four, five times.** Diminishing returns set in fast once the foundation is wrong; you end up with tangled code and comparable effort either way. Correction: recognize the restart signal and restate the problem, including the constraint that broke it, from scratch.
- **Defaulting to "prompt for output" even when the actual goal is understanding.** Using AI to skip a concept you're still building costs you later — in an interview, in the next feature that's a slight variation, in your own confidence. Correction: notice which mode you're in and choose deliberately (see Chapter 5 for the full argument).
- **Reaching for AI on security-critical code before you can evaluate it.** Generating an auth check you can't independently verify inverts the whole workflow this track teaches. Correction: build the fundamentals first; bring AI in once you can genuinely review its output in that domain.

## Practice Exercises

1. Take a real bug or feature request from any project you've built. Write a vague, one-line prompt for it and a context-rich version (constraints, existing code, error output, actual goal). Run both through an AI tool and compare the two outputs for how much verification work each would need.
2. Pick a task that's naturally 3+ steps (e.g., "add pagination to a list endpoint"). Decompose it into sequenced prompts and work through them one at a time, applying Chapter 2's five steps between each step before moving to the next.
3. Choose a concept you're shaky on. Do a "prompting for learning" session: ask the AI to explain it without code, write your own attempt, then ask it to critique (not rewrite) your attempt. Note what you understood afterward that you didn't before.
4. Recall or create a moment where you patched an AI-generated solution more than twice for what turned out to be the same underlying issue. Rewrite the *first* prompt you should have used, with that constraint included from the start.
5. Write three short scenarios (one line each) where you'd deliberately choose *not* to use AI, based on this chapter's two categories. Be specific to your own current skill level, not generic.
