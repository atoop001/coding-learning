# Chapter 1: How AI Coding Tools Work

## Overview

You can use these tools well without a machine learning degree, but you do need a practitioner's mental model — not of the math, but of the *shape* of the tool's competence. This chapter builds that model: what a large language model (LLM) is actually doing when it writes code (predicting, not looking up), the landscape of tools built on that capability (autocomplete, chat, and agentic tools), and the specific ways this prediction machine fails. Every later chapter assumes you understand *why* AI output needs verification, not just *that* it does. This chapter is the why.

## Definitions & Explanations

**Large language model (LLM)** — a model trained on enormous amounts of text (including huge amounts of public code) to predict the next token (roughly, a word-piece) given everything before it. Ask it to write a function, and it is generating the most statistically plausible continuation of your prompt, token by token, based on patterns it absorbed during training. It is not querying a database of known-correct code, not running your code to check it works, and not "looking anything up" unless the tool explicitly gives it a search or execution capability alongside the model itself.

This single fact explains almost every failure mode below. A model that predicts plausible text will, by construction, sometimes produce *very* plausible text that is wrong — because "looks like working code" and "is working code" are correlated but not identical, and the model was optimized for the former.

**Training data and its cutoff** — LLMs learn from a snapshot of text collected up to some date (the "training cutoff"), then get deployed and used for months or years afterward. During that gap, libraries release new major versions, APIs get deprecated, best practices shift, and security advisories get published. The model doesn't know any of this happened unless the tool feeds it fresh information at request time (see "grounding" below). Ask an ungrounded model about "the current way to do X" and you get *last year's* current way, stated with the same confidence as if it were still true.

**Hallucination** — the model generates something that doesn't exist: a function that isn't in the library, a parameter that was never added, an npm package that was never published, a config option invented whole-cloth. This isn't the model "lying" — it has no concept of true or false, only of what's statistically likely to follow your prompt. A plausible-sounding method name for a well-known library is *exactly* the kind of thing the model is good at generating, hallucinated or not, because both real and hallucinated names look the same to a next-token predictor.

**Grounding / retrieval augmentation** — some tools reduce staleness and hallucination by giving the model live access to real information at request time: web search, your actual open files, your project's installed dependency versions, or a documentation index. A grounded answer is more trustworthy than an ungrounded one, but grounding narrows the failure rate — it does not eliminate it. The model can still misread what it retrieved, retrieve the wrong thing, or blend retrieved facts with a hallucinated detail in the same sentence.

**The tool landscape** — three broad categories, distinguished by how much autonomy and context they get:

- **Inline autocomplete** (GitHub Copilot's ghost-text, similar features in most modern editors) — predicts the next few lines as you type, based on the current file and nearby open files. Fast, low-context, easy to accept without reading (which is exactly the danger — see Chapter 2).
- **Chat assistants** (ChatGPT, Claude, and similar in a browser or side panel) — you paste in context and ask a question or request code; you copy the answer back yourself. More context than autocomplete if you supply it, but only what you paste — it doesn't see your repo unless you show it.
- **Agentic coding tools** (Claude Code, Cursor's agent mode, GitHub Copilot Workspace, and similar) — can read your actual files, run commands, execute tests, and make multi-file edits across a whole task with much less hand-holding. Most context, most capability, and correspondingly the highest cost of un-reviewed trust: an agent that's confidently wrong can touch a dozen files before you notice, not just one function.

None of these categories is "safe" and none is "dangerous" in an absolute sense — all three produce the same class of errors below, just at different scales and speeds. Fast, low-context tools make small mistakes often; capable, high-context tools make bigger mistakes less often but with more blast radius when they happen.

**Confident tone regardless of correctness** — this is the trap that makes all of the above dangerous rather than merely imperfect. Nothing about how these models are trained or deployed makes them *express less confidence* when they're about to be wrong. A hallucinated API is described in the same calm, authoritative voice as a real one. There is no built-in "I'm not sure" signal you can rely on — tone is not evidence.

## Code Examples

The clearest way to see these failure modes is to see plausible-looking, wrong output next to what's actually true. These are illustrative examples of the *shape* of the problem — you'll produce your own live examples in Chapter 2's worked example and in Project 1.

```python
# Prompted: "give me a fast way to check if a list has duplicates in Python"
#
# Plausible AI output — reads fine, uses a real stdlib module:
from collections import Counter

def has_duplicates(items):
    counts = Counter(items)
    return any(count > 1 for count in counts.values())

# This works. But it's O(n) time AND does more work than needed —
# it counts every occurrence when it only needs to know "seen before?".
# A model optimizing for "plausible, working-looking code" has no
# pressure to also produce the *simplest correct* code:
def has_duplicates(items):
    seen = set()
    for item in items:
        if item in seen:
            return True
        seen.add(item)
    return False
```

```javascript
// Prompted: "parse this date string with Node's built-in date library"
//
// Hallucinated output — confident, specific, and wrong:
const { parseISODate } = require("date-fns/parseISO"); // <- not a real export shape
const date = parseISODate("2026-03-14");

// date-fns exports `parseISO` directly, not a `parseISODate` destructured
// from a `/parseISO` subpath. The function name is a plausible BLEND of
// the real function name and a plausible file path — a classic hallucination
// pattern: real pieces, wrong combination.
const { parseISO } = require("date-fns");
const date = parseISO("2026-03-14");
```

```python
# Staleness example: an ungrounded model asked in mid-2026 might still
# describe the pre-2024 way to pin dependencies in a requirements.txt-based
# project as "the modern way," because more of its training text described
# that era than describes whatever the current convention is by the time
# you're reading this. The fix isn't memorizing today's convention —
# it's habitually checking the tool's own current docs (Chapter 2) instead
# of trusting the model's unprompted claim about "the current way."
```

## Common Pitfalls

- **Assuming tone signals confidence.** A hedge like "I believe this should work" and a flat assertion "this is correct" carry the same actual evidence about correctness — none, from the tone alone. Correction: never adjust how much you verify based on how the AI *sounds*.
- **Treating a well-known library as low-risk for hallucination.** Popular libraries actually get hallucinated *more* often in specific detail, because the model has seen many versions, many similar-but-different APIs, and many blog posts describing old versions — all blended into one plausible-sounding answer. Correction: verify API surface especially for libraries you "already know," where you're most likely to skim past a wrong detail.
- **Assuming a grounded/agentic tool can't hallucinate.** Search and file access reduce the rate; they don't remove the mechanism. The model still generates a token-by-token guess about what to do with what it retrieved. Correction: grounding earns faster review, not skipped review.
- **Forgetting the context window is finite.** Long chats or huge files get silently truncated, summarized, or dropped from what the model actually "sees," and it will still answer confidently about content it no longer has. Correction: for long sessions, periodically re-state the constraints that matter instead of assuming they're still "in view."
- **Conflating "it compiles/runs" with "it's correct."** A hallucinated function name throws immediately; a hallucinated *default value* or a subtly wrong algorithm runs fine and fails later, in production, on an input you didn't try. Correction: running without error is the weakest evidence of correctness there is (more in Chapter 2).

## Practice Exercises

1. Open any AI chat tool and ask it for a niche function from a library you actually know well (e.g., a specific `pandas`, `lodash`, or `requests` method). Check the official docs afterward. Did it match exactly — method name, parameter names, and defaults?
2. Ask an AI tool "what's the current recommended way to do X" for something in your primary language that has changed conventions in the last 2–3 years (state management in React, package management in Python, etc.). Compare its answer against the framework's own current docs. Note any staleness.
3. Deliberately ask an AI tool to use a library or function you're fairly sure doesn't exist (invent a plausible-sounding name). See whether it (a) tells you it doesn't exist, (b) hallucinates a plausible implementation anyway, or (c) hedges. Record which happened.
4. Pick one tool from each category — an inline autocomplete, a chat assistant, and (if you have access) an agentic tool — and give all three the same small task. Compare how much context each one had access to without you doing anything extra, and how that shaped the output.
5. Find (or recall) a real moment where AI-generated code looked right to you at a glance but wasn't. Write two sentences: what made it *look* right, and what you'd check now, having read this chapter, that you didn't check then.
