# Project 1: Console Quiz Game

## Description

Build an interactive multiple-choice quiz that runs in the terminal. The program asks a series of questions, accepts answers, gives immediate feedback, and reports a final score with a grade. This is your first complete program: it exercises input/output, types, branching, loops, and basic methods — everything from the opening chapters working together.

## Difficulty

**Beginner** — estimated effort: 2–4 hours.

## Chapters Used

- 01 Getting Started
- 02 Variables & Types
- 03 Operators & Control Flow
- 04 Loops
- 05 Methods
- 06 Strings (light use)

## Requirements Checklist

- [ ] The game greets the player, asks for their name, and uses it in later messages
- [ ] At least 8 hardcoded questions, each with 4 answer options labeled A–D
- [ ] Questions are presented one at a time; the player answers by typing a letter
- [ ] Input is case-insensitive and trimmed ("b", " B " both work)
- [ ] Invalid input (not A–D) re-prompts without consuming the question or crashing
- [ ] Correct answers give positive feedback; wrong answers show the correct option
- [ ] A running score is tracked and shown after each question (e.g., "Score: 3/5")
- [ ] After the last question, a summary shows total score, percentage (one decimal place), and a letter grade via a switch expression
- [ ] The player is asked whether to play again; "y" restarts with score reset
- [ ] All question-asking logic lives in at least two well-named methods (e.g., one that asks a single question and returns whether it was correct)
- [ ] No compiler warnings

## Hints

- Store each question's text, four options, and correct letter in parallel arrays for now (e.g., `string[] questions`, `string[,] options` or four separate arrays, `char[] answers`). You'll rebuild this with classes in a later project — feel the awkwardness on purpose.
- A method with signature like `static bool AskQuestion(string question, string[] options, char correct)` keeps `Main` short.
- A `do/while` loop is the natural fit for both "re-ask until valid input" and "play again?".
- For the percentage, remember integer division (Chapter 2's pitfall) — cast before dividing.
- `Console.ReadLine()?.Trim().ToUpper()` normalizes input in one line; compare against `"A"`..`"D"` or take `[0]` after checking length.

## Stretch Goals

- Shuffle question order each playthrough using `Random`.
- Add a per-question countdown display like "Question 3 of 8".
- Track wrong answers and re-ask only those at the end as a "review round".
- Add categories (e.g., science, history) and let the player pick one before starting.
- Keep a session-best score and show "New high score!" when beaten (memory only — file persistence comes later in the track).
