# Chapter 5: Loops & Iteration

## Overview

Computers excel at repetition. Loops let you run a block of code many times: once per item in a collection, once per line in a file, or until a condition changes. Almost every interesting program contains loops — processing every expense, retrying a network request, running a game until the player quits.

Python has two loop statements: `for` (loop *over* something — a known collection or range) and `while` (loop *while* a condition holds — an unknown number of times). You'll also learn `break`, `continue`, `range()`, `enumerate()`, and how to build the loop-until-valid input pattern you'll reuse forever.

## Definitions & Explanations

**`for` loop** — iterates over each item of an *iterable* (a string, list, range, file, ...), binding the item to a variable each pass:

```python
for item in iterable:
    # body runs once per item, with `item` set to the current one
```

Use `for` when you know *what* you're looping over.

**`while` loop** — repeats as long as its condition is truthy:

```python
while condition:
    # body runs, then condition is re-checked
```

Use `while` when you loop until something *happens* — user types "quit", a guess is correct, a number gets small enough. The body must eventually make the condition false, or you get an **infinite loop** (stop one with Ctrl+C in the terminal).

**`range()`** — generates a sequence of integers, most often to loop a specific number of times:

- `range(5)` → 0, 1, 2, 3, 4 (five numbers, starts at 0, stops *before* 5)
- `range(2, 6)` → 2, 3, 4, 5
- `range(10, 0, -2)` → 10, 8, 6, 4, 2 (third argument is the step)

**`break`** — immediately exits the nearest enclosing loop.

**`continue`** — skips the rest of the body and jumps to the next iteration.

**`else` on a loop** (occasionally useful, often surprising) — the `else` block after a loop runs only if the loop finished *without* hitting `break`. Handy for search loops: "if we never found it...".

**`enumerate(iterable)`** — yields `(index, item)` pairs, so you get positions without manual counters: `for i, name in enumerate(names):`.

**`zip(a, b)`** — pairs up items from two (or more) iterables: `for name, score in zip(names, scores):`. Stops at the shorter one.

**Accumulator pattern** — the most common loop shape: initialize a variable before the loop (`total = 0`, `result = ""`, `found = None`), update it inside, use it after.

**Nested loops** — a loop inside a loop; the inner one completes fully for each pass of the outer one. Needed for grids, tables, and comparing all pairs.

## Code Examples

### `for` over collections and strings

```python
# Looping over a list (lists formally in Chapter 7 — intuition suffices here)
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(f"I like {fruit}")

# Looping over the characters of a string
vowel_count = 0
for ch in "programming is repetitive":
    if ch in "aeiou":
        vowel_count += 1
print(f"Vowels: {vowel_count}")        # Vowels: 8
```

### `range` — counting loops

```python
# Print a 5-times table
for i in range(1, 11):                 # 1 through 10
    print(f"5 x {i:>2} = {5 * i}")

# Countdown
for n in range(5, 0, -1):
    print(n)
print("Liftoff!")

# Sum 1..100 with an accumulator
total = 0
for n in range(1, 101):
    total += n
print(total)                           # 5050
```

### `while` — loop until something happens

```python
# Halving until small
value = 1000
steps = 0
while value > 1:
    value = value / 2
    steps += 1
print(f"Reached {value} after {steps} halvings")

# The classic input-validation loop — memorize this shape:
while True:                            # deliberately infinite...
    raw = input("Enter a number 1-10: ").strip()
    if raw.isdigit() and 1 <= int(raw) <= 10:
        number = int(raw)
        break                          # ...until we explicitly escape
    print("Invalid — try again.")
print(f"You chose {number}")
```

### `break`, `continue`, and loop-`else`

```python
# continue: skip uninteresting items
for n in range(1, 11):
    if n % 2 == 0:
        continue                       # skip evens
    print(n)                           # 1 3 5 7 9

# break + else: searching
secret = 7
for guess in [2, 9, 4, 7, 5]:
    print(f"trying {guess}...")
    if guess == secret:
        print("Found it!")
        break
else:
    # runs ONLY if the for loop never hit break
    print("Not in the list.")
```

### `enumerate` and `zip`

```python
tasks = ["write resume", "practice Python", "apply to jobs"]

# Numbered menu — enumerate's second argument sets the starting number
for i, task in enumerate(tasks, 1):
    print(f"{i}. {task}")

names = ["Ana", "Ben", "Cleo"]
scores = [91, 78, 85]
for name, score in zip(names, scores):
    print(f"{name}: {score}")
```

### Nested loops

```python
# A multiplication table grid
for row in range(1, 6):
    for col in range(1, 6):
        print(f"{row * col:>4}", end="")   # end="" keeps us on the same line
    print()                                 # bare print() moves to the next line
#    1   2   3   4   5
#    2   4   6   8  10
#    ...
```

### A realistic mini-program: menu loop

```python
# todo_mini.py — the menu-loop shape you'll reuse in projects
tasks = []

while True:
    print("\n1) Add task  2) List tasks  3) Quit")
    choice = input("> ").strip()

    if choice == "1":
        task = input("Task: ").strip()
        if task:
            tasks.append(task)
            print("Added.")
    elif choice == "2":
        if not tasks:
            print("(no tasks yet)")
        for i, t in enumerate(tasks, 1):
            print(f"{i}. {t}")
    elif choice == "3":
        print("Bye!")
        break
    else:
        print("Unknown option.")
```

## Common Pitfalls

**1. Infinite `while` loops from forgetting the update**

```python
n = 10
while n > 0:
    print(n)
    # forgot n -= 1  → prints 10 forever. Ctrl+C to stop.
```

Rule of thumb: when you write `while`, immediately ask "what line makes this condition eventually false?"

**2. Off-by-one with `range`**

`range(1, 10)` does **not** include 10. If you want 1 through 10, write `range(1, 11)`. Check your loop's first and last value mentally before trusting it.

**3. Modifying a list while looping over it**

```python
nums = [1, 2, 3, 4, 5, 6]
for n in nums:
    if n % 2 == 0:
        nums.remove(n)        # skips items! Result: [1, 3, 5] only by luck; often wrong
# FIX: build a new list instead
odds = []
for n in nums:
    if n % 2 != 0:
        odds.append(n)
# (Chapter 9's comprehensions make this one line.)
```

**4. Using a manual counter when `enumerate` exists**

```python
# Clunky and error-prone:
i = 0
for name in names:
    print(i, name)
    i += 1
# Idiomatic:
for i, name in enumerate(names):
    print(i, name)
```

**5. `break` only exits ONE loop**

In nested loops, `break` in the inner loop returns you to the outer loop, not out of everything. To exit both, set a flag, or restructure into a function and `return` (Chapter 6).

**6. Shadowing the loop variable's purpose**

```python
total = 0
for total in range(5):    # oops — reusing `total` as the loop variable destroys the accumulator
    ...
```

Keep loop variables (`i`, `n`, `item`, `line`) distinct from accumulators (`total`, `count`, `result`).

**7. Empty-body confusion with `pass`**

If you need a syntactically required block with nothing in it (rare in loops, common while sketching), use `pass`:

```python
for item in data:
    pass    # TODO: handle later — placeholder so the file still runs
```

## Practice Exercises

1. **Star triangle.** Using a loop and string repetition, print a right triangle of `*` with 5 rows (1 star, then 2, ... then 5). Then invert it (5 down to 1).
2. **Sum and average.** Ask the user for numbers one at a time; an empty input ends entry. Afterwards print the count, the sum, and the average (careful: what if they entered nothing at all?).
3. **Guess validation.** Write a loop that keeps asking "Even number between 2 and 20:" until the input is genuinely a number, even, and in range. Reject each bad input with a *specific* message (not a number / not even / out of range).
4. **First negative.** Given `readings = [4, 8, 15, 16, -23, 42]`, use a `for` loop with `break` to print the index of the first negative value, and a loop-`else` to print "no negatives" if there isn't one. Test both cases.
5. **FizzBuzz, complete.** Print the numbers 1–50, but print "Fizz" for multiples of 3, "Buzz" for multiples of 5, and "FizzBuzz" for multiples of both. This is a genuinely common interview warm-up — write it until you can do it without notes.
