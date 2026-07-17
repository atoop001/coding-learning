# Project 1: Number Guessing Game

## Description

Build a terminal game where the computer secretly picks a number between 1 and 100 and the player tries to guess it. After each guess the game says "higher" or "lower" until the player finds it, then reports how many guesses it took and offers a rematch.

Using it should feel snappy and forgiving: garbage input never crashes the game, the messages are friendly and specific, and a play session flows naturally from round to round. This is a small program, but a *finished* one — polish counts.

## Difficulty & Effort

**Difficulty:** Beginner
**Estimated effort:** 2–4 hours

## Chapters Used

- `01-getting-started.md` — running scripts, `input()`/`print()`
- `02-variables-and-data-types.md` — ints, type conversion
- `03-strings-and-f-strings.md` — clean output formatting
- `04-operators-and-conditionals.md` — comparisons, branching
- `05-loops-and-iteration.md` — the game loop, input-validation loop
- `10-modules-packages-pip-venv.md` — only the `random` module section

## Requirements Checklist

- [ ] The computer picks a random integer from 1 to 100 (inclusive on both ends) using the `random` module
- [ ] The player is told the valid range before their first guess
- [ ] After each wrong guess, the game prints whether the secret is higher or lower
- [ ] The game counts guesses and reports the total when the player wins
- [ ] Non-numeric input prints a specific message ("that's not a number") and re-asks without counting as a guess
- [ ] Out-of-range guesses (e.g. 250) print a specific message and re-ask without counting as a guess
- [ ] Guessing the same number twice in a round triggers a gentle "you already tried that" message
- [ ] After winning, the player is asked to play again (accepting `y`/`yes` in any capitalization); the count and secret reset for a new round
- [ ] Declining a rematch prints a goodbye message including the best (lowest) guess count across all rounds this session
- [ ] The program never crashes, no matter what is typed at any prompt

## Hints

- Structure first: an outer loop for "play another round?", an inner loop for guesses within a round. Sketch it with comments before writing code.
- The input-validation loop pattern from Chapter 5 (`while True:` ... `break` on success) handles the "re-ask without counting" requirements naturally — validate *before* incrementing the guess counter.
- `.strip().lower()` on the play-again answer makes `"  YES "` work.
- Tracking "already guessed" wants a collection you can `append` to and check with `in` — you know one.
- To track the best score across rounds, think about what needs to live *outside* the outer loop, and when it should update.
- Test like a saboteur: empty input, `"abc"`, `"3.7"`, `"-5"`, `"101"`, repeated guesses.

## Stretch Goals

- Difficulty levels: easy (1–50, unlimited guesses), normal (1–100), hard (1–200 with a 10-guess limit and a lose condition)
- After each round, rate the player's performance vs. the theoretical optimum (~7 guesses via halving for 1–100) — e.g. "efficient!" or "room to improve"
- Reverse mode: the *player* thinks of a number and the *computer* guesses using binary search, with the player answering h/l/c (higher/lower/correct)
- A "warmer/colder" mode that compares the distance of this guess to the previous one instead of saying higher/lower
