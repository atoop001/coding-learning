# Project 1: Number Guessing Game (Console)

## Description

Build a number guessing game that runs entirely in the console (Node.js or the browser console). The computer picks a secret random number between 1 and 100; the player makes guesses and receives "too high" / "too low" feedback until they find it. The game tracks how many guesses were used, gives a rating at the end ("lucky!", "solid", "took a while..."), and offers replayability by being wrapped in a function you can call again.

It should *feel* like a friendly little game: clear prompts, encouraging feedback, and no crashes no matter what nonsense is entered.

Since you haven't learned DOM or events yet, the "interface" is functions and the console. In Node, you can either hard-code a series of guesses to test with, use `prompt-sync` if you're comfortable installing it, or run the game in the browser console using `prompt()` — any of these is acceptable.

## Difficulty & Effort

- **Difficulty:** Beginner
- **Estimated effort:** 2–4 hours

## Chapters Used

- `01-getting-started.md` — running JS, console
- `02-variables-and-data-types.md` — numbers, strings, conversion of input
- `03-operators-and-expressions.md` — comparisons, modulo, ternaries
- `04-conditionals.md` — the core feedback logic
- `05-loops-and-iteration.md` — the guess loop
- `06-functions.md` — structuring the game into functions

## Requirements Checklist

- [ ] The secret number is a random integer from 1 to 100 inclusive (verify your formula's edges!)
- [ ] The player's guess is compared and produces exactly one of: "too high", "too low", or "correct"
- [ ] Guessing continues in a loop until the number is found
- [ ] The total number of guesses is counted and reported at the end
- [ ] Input is converted from string to number, and invalid input (not a number, out of 1–100 range) is rejected with a helpful message *without* counting as a guess
- [ ] A final rating message varies by guess count (e.g., ≤5 / ≤10 / more) using conditionals
- [ ] The game logic is organized into at least three functions (e.g., `pickSecret()`, `checkGuess(guess, secret)`, `playGame()`)
- [ ] `checkGuess` returns a value rather than only logging — so it could be tested by calling it directly
- [ ] Constants like the min, max, and rating thresholds are named variables (`const`), not magic numbers scattered around
- [ ] Running `playGame()` twice works cleanly with a fresh secret number each time

## Hints

- `Math.random()` gives a decimal from 0 up to (not including) 1. Think through what to multiply by and what to add so both 1 and 100 are possible. Test your formula in a loop that runs 1000 times and records the min and max produced.
- "Keep going until something happens" is the signature of a `while` loop — reread the dice-rolling example in Chapter 5.
- `Number(input)` plus `Number.isNaN(...)` covers most input validation. Decide what happens *before* the comparing starts — a guard clause keeps the loop body clean.
- If a wrong input shouldn't count as a guess, think about `continue` — or about where in the loop you increment the counter.
- Hard-coded test mode: write `playWithGuesses(secret, guessArray)` that feeds guesses from an array instead of prompting. This makes the game testable and is genuinely how developers verify logic.

## Stretch Goals

- **Difficulty levels:** let the player choose a range (1–50, 1–100, 1–1000) and scale the rating thresholds mathematically.
- **Optimal play detector:** report how the player's guess count compares to binary search (`Math.ceil(Math.log2(range))` guesses).
- **Warmer/colder mode:** instead of high/low, report whether the latest guess is *closer or farther* than the previous guess.
- **Guess history:** store all guesses in an array and print them at the end, flagging any repeated guesses.
- **Two-player mode:** player 1 "sets" the number (as a function argument), player 2 guesses.
