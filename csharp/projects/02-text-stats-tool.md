# Project 2: Text Statistics Tool

## Description

Build a console tool that analyzes a block of text the user pastes in (multi-line input, terminated by an empty line) and reports detailed statistics: character/word/sentence counts, longest and shortest words, average word length, letter frequency, and estimated reading time. This project makes you fluent with strings, arrays, loops, and decomposing a problem into methods.

## Difficulty

**Beginner** — estimated effort: 3–5 hours.

## Chapters Used

- 04 Loops
- 05 Methods
- 06 Strings & String Interpolation
- 07 Arrays & Lists

## Requirements Checklist

- [ ] The program reads multiple lines of input until the user enters an empty line, accumulating them into one text
- [ ] Reports total characters (with and without spaces)
- [ ] Reports word count (split on whitespace, ignoring empty entries)
- [ ] Reports sentence count (count of `.`, `!`, `?` terminators — document your rule in a comment)
- [ ] Reports the longest and shortest word (ties: first occurrence wins)
- [ ] Reports average word length with one decimal place
- [ ] Reports the 5 most frequent letters with counts, case-insensitive, ignoring non-letters
- [ ] Reports estimated reading time at 200 words per minute, formatted as "X min Y sec"
- [ ] Displays everything as an aligned, readable report (use alignment specifiers like `{value,8}`)
- [ ] Each statistic is computed by its own method taking the text/words as a parameter and returning a value — `Main` only orchestrates and prints
- [ ] Empty input (user enters nothing) is handled with a friendly message, no crash, no division by zero

## Hints

- Accumulate input with a `List<string>` of lines, then `string.Join(" ", lines)` — or use `StringBuilder`.
- `text.Split(new[] { ' ', '\t', '\n' }, StringSplitOptions.RemoveEmptyEntries)` gives clean words; consider also trimming punctuation off each word with `Trim('.', ',', '!', '?', ';', ':')`.
- For letter frequency, an `int[26]` indexed by `c - 'a'` works — or wait, you know arrays: `char.IsLetter` and `char.ToLower` are your gatekeepers. (A Dictionary is cleaner but belongs to a later chapter — either is acceptable.)
- Finding top-5 without LINQ: loop 5 times, each pass find the max count not yet reported, mark it used.
- Reading time: total words / 200 gives minutes as a double; take the fractional part × 60 for seconds. Watch integer division.
- Test with edge cases: one word, one giant word, text that is only punctuation.

## Stretch Goals

- Add a "words appearing only once" count and a most-common-word report (excluding "the", "a", "and" — keep a small stop-word array).
- Detect and report the longest sentence (by word count).
- Add a `--simple` mode chosen at startup that prints just counts, exercising a boolean flag through your methods.
- Compute a rough readability score (e.g., Flesch: 206.835 − 1.015 × (words/sentences) − 84.6 × (syllables/words), approximating syllables as vowel groups).
- Support reading the text from a file path instead of pasted input once you reach Chapter 16 — design your methods now so only the input step would change.
