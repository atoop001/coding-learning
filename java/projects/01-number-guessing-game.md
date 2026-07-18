# Project 1: Number Guessing Game

## Description

A console game: the program picks a secret number between 1 and 100, and the player guesses until they find it, getting "higher"/"lower" feedback after each guess. Track the number of guesses, congratulate the player at the end, and offer a rematch. This is the classic first project — small enough to finish in one sitting, big enough to exercise everything from the first four chapters.

## Difficulty

**Beginner** — estimated effort: 2–4 hours.

## Chapters used

- 01 (Getting Started) — compiling and running
- 02 (Variables & Types) — ints, booleans, Scanner input
- 03 (Operators & Control Flow) — comparisons, if/else
- 04 (Loops) — the game loop, replay loop, input validation loop

## Requirements checklist

- [ ] Program generates a random secret number from 1 to 100 (`(int)(Math.random() * 100) + 1` or `java.util.Random`)
- [ ] Player is prompted for guesses via `Scanner`
- [ ] After each wrong guess, print "Too high" or "Too low"
- [ ] Correct guess prints a win message including how many guesses it took
- [ ] Non-numeric input does not crash the program (re-prompt instead; hint below)
- [ ] Guesses outside 1–100 are rejected with a message and don't count as attempts
- [ ] After winning, ask "Play again? (y/n)" and restart with a new secret number on "y"
- [ ] Total games played and best (lowest) guess count are reported when the player quits
- [ ] Code is split into at least `main` plus one helper method (e.g., reading a valid guess)

## Hints

- Structure: an outer `do-while` (play again?) wrapping an inner `while` (guessing until correct).
- For crash-proof input *before* you've learned exceptions: `scanner.hasNextInt()` tells you whether the next token is a number; if it isn't, consume it with `scanner.next()` and re-prompt.
- Keep the "best score" in a variable initialized to something impossible-to-beat, like `Integer.MAX_VALUE`, and update it after each win.
- Print a running attempt counter ("Guess #3: ") so the player feels the pressure.
- Test the edges: guess 1, guess 100, guess the secret first try.

## Stretch goals

- Difficulty levels: easy (1–50, unlimited guesses), normal (1–100), hard (1–500, max 10 guesses — losing is possible).
- "Warmer/colder" mode: compare the distance of the current guess to the distance of the previous guess.
- Reverse mode: *you* think of a number, and the **computer** guesses using binary search, with you answering h/l/c. (You'll re-derive an actual algorithm.)
- ASCII victory banner drawn with nested loops.
