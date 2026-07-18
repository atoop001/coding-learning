# Chapter 4: Loops

## Overview

Loops repeat work. Java has four looping constructs: `while`, `do-while`, the classic three-part `for`, and the enhanced *for-each* loop. Python people will miss `for x in range(...)` — Java's classic `for` is its (more verbose, more flexible) equivalent, and it's identical to JavaScript's. This chapter also covers `break`, `continue`, nested loops, and labeled breaks.

## Definitions & Explanations

### while — check first, then run

```java
while (condition) {
    // runs zero or more times
}
```

Use when you don't know in advance how many iterations you need (e.g., "keep asking until input is valid").

### do-while — run first, then check

```java
do {
    // runs AT LEAST once
} while (condition);      // note the trailing semicolon
```

Python has no equivalent; JS has the same construct. Perfect for menus: show the menu at least once, repeat until the user quits.

### Classic for — counted loops

```java
for (int i = 0; i < 10; i++) {
    // init; condition; update
}
```

- **init** runs once before the loop (`int i = 0` — `i` is scoped to the loop only).
- **condition** is checked before every iteration; loop ends when false.
- **update** runs after each iteration.

Equivalent of Python's `for i in range(10)`. Counting down: `for (int i = 10; i > 0; i--)`. Stepping by 2: `i += 2` in the update slot.

### Enhanced for (for-each) — iterate a collection or array

```java
int[] scores = {90, 85, 72};
for (int score : scores) {        // read: "for each score in scores"
    System.out.println(score);
}
```

This is Java's closest match to Python's `for x in xs` / JS's `for...of`. Use it whenever you need the *elements* but not the *index*. You cannot modify the array through the loop variable, and you don't get the position — use the classic `for` when you need either.

### break, continue, and labels

- `break` — exit the innermost loop immediately.
- `continue` — skip to the next iteration of the innermost loop.
- **Labeled break** — exit an *outer* loop from inside a nested one (Java's answer to Python's for-else workarounds):

```java
outer:
for (int row = 0; row < 5; row++) {
    for (int col = 0; col < 5; col++) {
        if (grid[row][col] == target) {
            break outer;      // exits BOTH loops
        }
    }
}
```

### Infinite loops

`while (true) { ... }` is the idiomatic infinite loop, exited with `break` or `return`. Common for game loops and menu loops.

## Code Examples

### The four loop types side by side

```java
public class LoopTour {
    public static void main(String[] args) {
        // 1. Classic for: print 1..5
        for (int i = 1; i <= 5; i++) {
            System.out.print(i + " ");
        }
        System.out.println();

        // 2. while: halve until small
        int n = 100;
        while (n > 1) {
            n = n / 2;
        }
        System.out.println("ended at " + n);   // 1

        // 3. do-while: runs even though condition starts false
        int x = 99;
        do {
            System.out.println("ran once with x=" + x);
        } while (x < 10);

        // 4. for-each over an array
        String[] langs = {"Java", "Python", "JavaScript"};
        for (String lang : langs) {
            System.out.println("I know some " + lang);
        }
    }
}
```

### Input validation loop (very common pattern)

```java
import java.util.Scanner;

public class ValidatedInput {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int age = -1;

        // Loop until we get a sane value; hasNextInt avoids crashes on "abc"
        while (age < 0 || age > 120) {
            System.out.print("Enter your age (0-120): ");
            if (in.hasNextInt()) {
                age = in.nextInt();
            } else {
                in.next();   // discard the bad token, or we'd loop forever
                System.out.println("That's not a number.");
            }
        }
        System.out.println("Thanks! Age recorded: " + age);
        in.close();
    }
}
```

### Nested loops: multiplication table

```java
public class TimesTable {
    public static void main(String[] args) {
        // header row
        System.out.print("    ");
        for (int col = 1; col <= 9; col++) {
            System.out.printf("%4d", col);
        }
        System.out.println();

        for (int row = 1; row <= 9; row++) {
            System.out.printf("%4d", row);          // row label
            for (int col = 1; col <= 9; col++) {
                System.out.printf("%4d", row * col);
            }
            System.out.println();                    // end the row
        }
    }
}
```

### Accumulator patterns

```java
public class Accumulators {
    public static void main(String[] args) {
        int[] temps = {21, 25, 19, 30, 27};

        int sum = 0;
        int max = temps[0];              // seed with first element, not 0!
        int countAbove25 = 0;

        for (int t : temps) {
            sum += t;
            if (t > max) {
                max = t;
            }
            if (t > 25) {
                countAbove25++;
            }
        }

        double average = (double) sum / temps.length;  // cast avoids int division
        System.out.println("avg=" + average + " max=" + max
                + " daysAbove25=" + countAbove25);
    }
}
```

## Common Pitfalls

### 1. Off-by-one errors

```java
for (int i = 0; i <= arr.length; i++) { }   // ❌ runs one time too many
for (int i = 0; i < arr.length; i++) { }    // ✅ indices 0 .. length-1
```

`arr[arr.length]` throws `ArrayIndexOutOfBoundsException`. Java arrays are 0-indexed; the last valid index is `length - 1`.

### 2. Semicolon after the loop header

```java
for (int i = 0; i < 3; i++);        // ❌ empty body; loop runs 3 times doing nothing
{
    System.out.println("hi");        // prints once, after the loop
}
```

### 3. Forgetting to update the loop variable in `while`

```java
int i = 0;
while (i < 10) {
    System.out.println(i);           // ❌ infinite loop — i never changes
}
```

**Fix:** `i++;` inside the body. If your program hangs, this is suspect #1. (Press Ctrl+C in the terminal to kill it.)

### 4. Modifying a for-each variable expecting the array to change

```java
int[] nums = {1, 2, 3};
for (int n : nums) {
    n = n * 2;                       // ❌ changes only the local copy
}
// nums is still {1, 2, 3}. Use a classic for with index assignment:
for (int i = 0; i < nums.length; i++) {
    nums[i] = nums[i] * 2;           // ✅
}
```

### 5. `break` inside nested loops only exits the inner one

```java
for (...) {
    for (...) {
        break;        // ❌ if you meant to stop everything — only inner loop exits
    }
}
// ✅ use a labeled break (see above) or a boolean flag / extracted method.
```

### 6. Floating-point loop counters

```java
for (double d = 0.0; d != 1.0; d += 0.1) { }   // ❌ may never equal exactly 1.0
for (int i = 0; i < 10; i++) { double d = i / 10.0; }   // ✅ count with ints
```

## Practice Exercises

1. **FizzBuzz, complete.** Loop 1 to 100 printing Fizz/Buzz/FizzBuzz/number (rules from Chapter 3). This is a real interview warm-up question — practice until you can write it in under two minutes.
2. **Countdown launcher.** Print a rocket countdown from 10 to 1 with a classic `for` counting down, then "Liftoff!". Then rewrite it with a `while` loop. Which reads better and why? (Comment your answer.)
3. **Menu loop.** Using `do-while` and `Scanner`, show a menu: `1) Say hello  2) Show time  3) Quit`. Repeat until the user picks 3. Reject unknown choices with a message. (Hint: `System.currentTimeMillis()` or `java.time.LocalTime.now()` for option 2.)
4. **Prime checker.** Read an `int n > 1` and determine if it's prime by testing divisors from 2 up to `Math.sqrt(n)`. Use `break` as soon as a divisor is found. Then extend it: print all primes from 2 to 100 using a nested loop.
5. **Star pyramid.** Using nested loops only (no string tricks), print a centered pyramid of `*` of height `h`: row 1 has one star, row `h` has `2h-1` stars, each row padded with leading spaces so the pyramid is centered. Test with `h = 5`.
