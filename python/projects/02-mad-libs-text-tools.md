# Project 2: Mad Libs & Text Toolbox

## Description

Build a two-part terminal program around text manipulation. Part one is a **Mad Libs** engine: story templates containing placeholders like `{noun}` and `{verb}` are filled in from user prompts, producing (usually ridiculous) stories. Part two is a **Text Toolbox**: a menu-driven set of utilities that analyze or transform any text the user types — word counts, vowel counts, title-casing, reversing, palindrome checks, and a simple word-frequency report.

Using it should feel like a small Swiss-army knife: pick a tool from a menu, feed it text, get a clean, nicely formatted result, return to the menu.

## Difficulty & Effort

**Difficulty:** Beginner
**Estimated effort:** 3–5 hours

## Chapters Used

- `03-strings-and-f-strings.md` — the core of the whole project
- `04-operators-and-conditionals.md` — menu branching
- `05-loops-and-iteration.md` — menu loop, counting loops
- `06-functions.md` — one function per tool
- `07-lists-and-tuples.md` — storing templates, splitting into word lists
- `08-dictionaries-and-sets.md` — word-frequency counting (light use)

## Requirements Checklist

### Mad Libs

- [ ] At least three different story templates are stored in the code (multi-line strings with named placeholders)
- [ ] The user picks a template from a numbered list
- [ ] The program prompts once per placeholder, naming the kind of word needed (e.g. "Give me a plural noun:")
- [ ] Empty answers are re-asked, not accepted
- [ ] The completed story prints framed by a decorative border built with string repetition
- [ ] The same placeholder type appearing twice in a template asks twice (two different nouns, not one reused)

### Text Toolbox

- [ ] A menu loop offers at least six tools plus "back/quit"; invalid choices are handled politely
- [ ] Word count and character count (with and without spaces) tool
- [ ] Vowel/consonant counter tool (case-insensitive; ignores digits and punctuation)
- [ ] Transformations tool: show the input in UPPER, lower, Title Case, and reversed
- [ ] Palindrome checker that ignores case, spaces, and punctuation ("Never odd or even" → yes)
- [ ] Word-frequency tool that prints the top 5 most common words with their counts
- [ ] Every tool is its own function that *returns* its result; only the menu code prints

## Hints

- For Mad Libs, gather answers first, then use `.format(**answers)` on the template or an f-string built at the end — either works; the dict-based `.format` approach scales better to many placeholders.
- To find the placeholders in a template without hardcoding them per story, consider storing each template as a pair: the text *and* a list of its placeholder names.
- The palindrome cleaner is a one-line comprehension once you see it: keep only alphabetic characters, lowercased, joined back together. Compare against its reverse via slicing.
- `text.split()` with no argument handles multiple spaces and newlines gracefully — better than `split(" ")`.
- For frequency counting, the `counts.get(word, 0) + 1` pattern from Chapter 8 is exactly right; sorting a dict's items by value was shown in the same chapter.
- Keep `main()` thin: it should read like a table of contents of your functions.

## Stretch Goals

- Load story templates from a text file instead of hardcoding them (peek ahead at Chapter 11)
- A "story so far" mode that saves each completed Mad Lib to a running session list and can replay them all
- Add a word-ladder tool: given two words, report which letters differ and at which positions
- A crude "reading level" estimate: average word length and average sentence length (split on `.`), mapped to easy/medium/hard
- Support piping a whole paragraph into the toolbox at once and running *all* tools on it as a single report
