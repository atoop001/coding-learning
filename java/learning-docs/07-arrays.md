# Chapter 7: Arrays

## Overview

An array is Java's most basic collection: a **fixed-size**, ordered block of elements, all the same type. Coming from Python lists or JS arrays, the two shocks are: (1) the size is fixed forever at creation, and (2) all elements share one declared type. Arrays are the foundation under everything — `ArrayList` (Chapter 12) is built on them — and they still appear constantly in method signatures (`String[] args`!), interview questions, and performance-sensitive code.

## Definitions & Explanations

### Declaring and creating

```java
int[] scores = new int[5];              // 5 slots, all initialized to 0
String[] names = new String[3];         // 3 slots, all initialized to null
double[] temps = {21.5, 19.0, 25.3};    // literal syntax: size inferred (3)
int[] empty = {};                        // legal: zero-length array
```

Key facts:

- `new int[5]` zero-fills: numeric arrays get `0`/`0.0`, `boolean[]` gets `false`, object arrays get `null`. (Contrast with local variables, which have *no* default.)
- The **length is fixed**. There is no `append`/`push`. "Growing" means creating a bigger array and copying (which is exactly what `ArrayList` automates).
- `arr.length` is a **field**, not a method — no parentheses. (Strings use `length()` *with* parentheses. Yes, this inconsistency annoys everyone.)

### Indexing

Zero-based, like Python/JS. `scores[0]` is the first, `scores[scores.length - 1]` the last. Out-of-range access throws `ArrayIndexOutOfBoundsException` at runtime — no negative-index magic, no silent `undefined`.

### Arrays are objects

An array variable holds a *reference*. Assignment copies the reference, not the contents:

```java
int[] a = {1, 2, 3};
int[] b = a;             // b and a point to the SAME array
b[0] = 99;               // a[0] is now also 99!
int[] c = a.clone();     // ✅ actual (shallow) copy
```

### The `java.util.Arrays` utility class

Import `java.util.Arrays` for the everyday helpers:

```java
Arrays.toString(arr)          // printable form: [1, 2, 3] — println(arr) prints garbage!
Arrays.sort(arr)              // in-place ascending sort
Arrays.fill(arr, 7)           // set every element
Arrays.copyOf(arr, newLen)    // resized copy (truncates or zero-pads)
Arrays.equals(a, b)           // element-wise comparison (== compares references!)
Arrays.binarySearch(arr, x)   // fast search — array must be sorted first
```

### Multidimensional arrays

Java's 2D array is an array *of arrays* (rows), like nested lists in Python:

```java
int[][] grid = new int[3][4];          // 3 rows × 4 columns, zero-filled
grid[1][2] = 5;                        // row 1, column 2
int[][] jagged = { {1}, {2, 3}, {4, 5, 6} };   // rows may differ in length!
```

Iterate with nested loops; `grid.length` is the row count, `grid[r].length` is that row's column count.

## Code Examples

### Core operations

```java
import java.util.Arrays;

public class ArrayBasics {
    public static void main(String[] args) {
        int[] scores = {88, 94, 67, 75, 91};

        // Printing: use Arrays.toString, never println(arr)
        System.out.println(Arrays.toString(scores));    // [88, 94, 67, 75, 91]
        System.out.println(scores.length);              // 5 (no parentheses!)

        // Read and write by index
        scores[2] = 70;
        System.out.println("first=" + scores[0] + " last=" + scores[scores.length - 1]);

        // Sort (in place — mutates the array)
        Arrays.sort(scores);
        System.out.println(Arrays.toString(scores));    // [70, 75, 88, 91, 94]

        // Copy before sorting when you need the original preserved
        int[] original = {3, 1, 2};
        int[] sorted = original.clone();
        Arrays.sort(sorted);
        System.out.println(Arrays.toString(original) + " vs " + Arrays.toString(sorted));
    }
}
```

### Classic array algorithms (write these by hand at least once)

```java
public class ArrayAlgorithms {

    public static int findMax(int[] values) {
        int max = values[0];                 // seed with FIRST element, not 0
        for (int i = 1; i < values.length; i++) {
            if (values[i] > max) max = values[i];
        }
        return max;
    }

    public static double average(int[] values) {
        int sum = 0;
        for (int v : values) sum += v;
        return (double) sum / values.length; // cast prevents int division
    }

    // Linear search: index of target, or -1
    public static int indexOf(int[] values, int target) {
        for (int i = 0; i < values.length; i++) {
            if (values[i] == target) return i;
        }
        return -1;
    }

    // Reverse in place with two pointers
    public static void reverse(int[] values) {
        int left = 0, right = values.length - 1;
        while (left < right) {
            int tmp = values[left];
            values[left] = values[right];
            values[right] = tmp;
            left++;
            right--;
        }
    }

    public static void main(String[] args) {
        int[] data = {4, 9, 2, 7, 5};
        System.out.println(findMax(data));       // 9
        System.out.println(average(data));       // 5.4
        System.out.println(indexOf(data, 7));    // 3
        reverse(data);
        System.out.println(java.util.Arrays.toString(data)); // [5, 7, 2, 9, 4]
    }
}
```

### 2D array: tic-tac-toe board

```java
public class Board {
    public static void main(String[] args) {
        char[][] board = {
            {'X', 'O', ' '},
            {' ', 'X', 'O'},
            {' ', ' ', 'X'}
        };

        // Print the board with separators
        for (int r = 0; r < board.length; r++) {
            for (int c = 0; c < board[r].length; c++) {
                System.out.print(board[r][c]);
                if (c < board[r].length - 1) System.out.print("|");
            }
            System.out.println();
            if (r < board.length - 1) System.out.println("-----");
        }

        // Check the main diagonal for a win
        boolean diagonalWin = board[0][0] != ' '
                && board[0][0] == board[1][1]
                && board[1][1] == board[2][2];
        System.out.println("Diagonal win: " + diagonalWin);   // true
    }
}
```

## Common Pitfalls

### 1. Printing an array directly

```java
int[] a = {1, 2, 3};
System.out.println(a);                    // ❌ [I@6d06d69c  (type + hash, useless)
System.out.println(Arrays.toString(a));   // ✅ [1, 2, 3]
```

### 2. Off-by-one / out of bounds

```java
int[] a = new int[5];
a[5] = 1;      // ❌ ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 5
a[4] = 1;      // ✅ valid indices are 0..4
```

Loop condition must be `i < a.length`, never `<=`.

### 3. Expecting arrays to grow

```java
int[] nums = new int[3];
nums[3] = 42;                              // ❌ there is no room — arrays don't grow
nums = Arrays.copyOf(nums, 4);             // ✅ make a bigger copy first
nums[3] = 42;                              // (or just use ArrayList — Ch. 12)
```

### 4. Reference copy when you meant content copy

```java
int[] backup = data;          // ❌ NOT a backup — same array, two names
int[] backup = data.clone();  // ✅
```

### 5. Comparing arrays with `==` or `.equals()`

```java
int[] a = {1, 2}; int[] b = {1, 2};
a == b                 // false — different objects
a.equals(b)            // false — array equals() is just ==, sadly
Arrays.equals(a, b)    // ✅ true — element-wise
```

### 6. Forgetting object-array slots start as null

```java
String[] words = new String[3];
int len = words[0].length();   // ❌ NullPointerException — slot is null
words[0] = "hi";               // ✅ fill before use
```

## Practice Exercises

1. **Stats pack.** Write methods `min`, `max`, `sum`, and `average` over an `int[]`, plus a `main` that runs them on `{12, 4, 19, 4, 7, 21}` and prints labeled results. Decide (and document in a comment) what your methods should do when given an empty array.
2. **Above average.** Using your `average`, print all elements greater than the array's average, and count them. Two passes over the array — why can't it be done cleanly in one? (Answer in a comment.)
3. **Frequency counter.** Given an `int[]` of dice rolls (values 1–6), build a `int[7]` tally where index `i` holds how many times `i` was rolled, then print a text histogram: `3: ****` per face. Generate 100 rolls with `(int)(Math.random() * 6) + 1`.
4. **Manual resize.** Write `static int[] push(int[] arr, int value)` that returns a new array one longer with `value` at the end (no `Arrays.copyOf` — copy with a loop). Start from `{}` and push 1, 2, 3, printing after each push. You've just built the core of `ArrayList`.
5. **Matrix transpose.** Given a rectangular `int[3][4]`, produce its `int[4][3]` transpose (rows become columns), printing both nicely. Then explain in a comment why transposing a *jagged* array would be harder.
