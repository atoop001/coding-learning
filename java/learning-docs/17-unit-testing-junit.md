# Chapter 17: Unit Testing with JUnit

## Overview

Professional Java development is test-driven or at minimum test-covered — no serious employer ships untested code, and "how do you test?" is a guaranteed interview topic. **JUnit 5** (a.k.a. JUnit Jupiter) is the standard framework: you write small methods that call your code and *assert* what should be true; a test runner executes them all and reports green/red.

If you've seen pytest or Jest, the shape is identical: test files mirror source files, assertions compare expected vs actual, and a failing test points you straight at a bug. This chapter covers writing tests, the assertion vocabulary, test structure and naming, parameterized tests, and testing exceptions.

## Definitions & Explanations

### What a unit test is

A **unit test** exercises one small piece (usually one method/class) in isolation:

```java
@Test
void withdrawalReducesBalance() {
    BankAccount acct = new BankAccount("Ada", 100.0);   // Arrange
    acct.withdraw(30.0);                                 // Act
    assertEquals(70.0, acct.getBalance());               // Assert
}
```

The **Arrange–Act–Assert** pattern structures nearly every test: set up objects, do the thing, check the outcome. Tests are regular Java methods marked `@Test`, in classes conventionally named `<ClassUnderTest>Test`, living in a parallel source tree (`src/test/java` mirrors `src/main/java` — Maven's layout, Chapter 18).

### Getting JUnit

JUnit is a library, not part of the JDK. Realistically you get it one of two ways:

- **IntelliJ**: place the caret on a class → Alt+Enter → "Create Test" → IntelliJ offers to add JUnit 5 to the project. Fine for now.
- **Maven/Gradle** (Chapter 18, and the capstone project): declare `junit-jupiter` as a test dependency — the professional route.

Run tests in IntelliJ with the green gutter arrows, or `mvn test` once you're on Maven.

### The assertion vocabulary

All static methods on `org.junit.jupiter.api.Assertions` (static-import them):

```java
assertEquals(expected, actual)          // the workhorse — NOTE THE ORDER
assertEquals(expected, actual, delta)   // doubles: tolerance for float error
assertNotEquals(a, b)
assertTrue(condition)   assertFalse(condition)
assertNull(x)           assertNotNull(x)
assertThrows(ExceptionType.class, () -> code)   // exception expected — returns it for inspection
assertIterableEquals(expectedList, actualList)
assertArrayEquals(expectedArr, actualArr)
assertAll(...)                          // group assertions; reports ALL failures, not just first
```

Every assertion takes an optional final `String` message shown on failure — use it when the values alone won't explain the problem.

### Lifecycle annotations

```java
@BeforeEach  void setUp()    { }   // runs before EVERY test — build fresh fixtures here
@AfterEach   void tearDown() { }   // after every test — cleanup (delete temp files...)
@BeforeAll   static void once()    { }   // once per class, must be static
@AfterAll    static void done()    { }   // once, after everything
@Disabled("reason")                 // skip a test, visibly
@DisplayName("withdraw rejects negatives")   // pretty name in reports
```

Each test method gets a **brand-new instance** of the test class — tests can't accidentally share state through fields, which enforces independence.

### Parameterized tests — one test, many inputs

```java
@ParameterizedTest
@ValueSource(ints = {2, 4, 6, 100})
void evenNumbersAreEven(int n) {
    assertTrue(MathKit.isEven(n));
}

@ParameterizedTest
@CsvSource({ "1,one", "5,five", "-3,minus three" })
void spellsNumbers(int input, String expected) {
    assertEquals(expected, Speller.spell(input));
}
```

(Needs the `junit-jupiter-params` artifact; included in the `junit-jupiter` aggregate.)

### What makes a GOOD test

- **Fast** — milliseconds; no sleeping, no network.
- **Independent** — any order, any subset, same results.
- **Repeatable** — no randomness without a fixed seed, no "works on my machine."
- **One behavior per test** — a failure should identify the broken behavior by test name alone.
- **Tests behavior, not implementation** — assert on outputs and state, not private internals. If a refactor that preserves behavior breaks your test, the test was wrong.

Name tests as behavior sentences: `withdrawFailsWhenAmountExceedsBalance`, not `test1`.

## Code Examples

### Class under test + its test class

```java
// src/main/java — Temperature.java  (the production code)
public class Temperature {
    private final double celsius;

    public Temperature(double celsius) {
        if (celsius < -273.15) {
            throw new IllegalArgumentException("below absolute zero: " + celsius);
        }
        this.celsius = celsius;
    }

    public double getCelsius() { return celsius; }
    public double getFahrenheit() { return celsius * 9 / 5 + 32; }
    public boolean isFreezing() { return celsius <= 0; }
}
```

```java
// src/test/java — TemperatureTest.java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;   // static import: bare assertEquals etc.

class TemperatureTest {

    @Test
    void convertsBoilingPointToFahrenheit() {
        Temperature t = new Temperature(100.0);
        // doubles need a tolerance (3rd argument) — float math is inexact
        assertEquals(212.0, t.getFahrenheit(), 0.0001);
    }

    @Test
    void zeroCelsiusIsFreezing() {
        assertTrue(new Temperature(0.0).isFreezing());
    }

    @Test
    void slightlyAboveZeroIsNotFreezing() {
        assertFalse(new Temperature(0.1).isFreezing());
    }

    @Test
    void rejectsTemperaturesBelowAbsoluteZero() {
        // assertThrows: the lambda is run, MUST throw the given type
        IllegalArgumentException e = assertThrows(
                IllegalArgumentException.class,
                () -> new Temperature(-300.0));
        assertTrue(e.getMessage().contains("absolute zero"));   // message quality matters too
    }

    @Test
    void acceptsExactlyAbsoluteZero() {          // boundary testing — the edges hide the bugs
        assertDoesNotThrow(() -> new Temperature(-273.15));
    }
}
```

### Fixtures with @BeforeEach

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;
import java.util.List;

class PlaylistTest {

    private Playlist playlist;          // fresh for every test — see setUp

    @BeforeEach
    void setUp() {
        playlist = new Playlist(3);     // capacity 3 (from Ch. 8's exercise)
        playlist.add(new Song("Aja", "Steely Dan", 480));
    }

    @Test
    void addingWithinCapacitySucceeds() {
        assertTrue(playlist.add(new Song("Peg", "Steely Dan", 237)));
        assertEquals(2, playlist.size());
    }

    @Test
    void addingBeyondCapacityFails() {
        playlist.add(new Song("b", "x", 100));
        playlist.add(new Song("c", "x", 100));
        assertFalse(playlist.add(new Song("overflow", "x", 100)));
        assertEquals(3, playlist.size(), "size must not grow past capacity");
    }

    @Test
    void totalDurationSumsAllSongs() {
        playlist.add(new Song("Peg", "Steely Dan", 237));
        assertEquals(717, playlist.totalDurationSeconds());
    }
}
```

### Parameterized boundary sweep

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;
import static org.junit.jupiter.api.Assertions.assertEquals;

class GraderTest {
    @ParameterizedTest(name = "score {0} -> grade {1}")
    @CsvSource({
        "100, A", "90, A",          // boundaries of each band —
        "89, B",  "80, B",          // exactly where off-by-one bugs live
        "79, C",  "70, C",
        "69, D",  "60, D",
        "59, F",  "0, F"
    })
    void assignsCorrectLetterGrade(int score, String expected) {
        assertEquals(expected, Grader.letterFor(score));
    }
}
```

Ten cases, one test method — and the failure report names the exact input that broke.

## Common Pitfalls

### 1. Expected and actual swapped

```java
assertEquals(acct.getBalance(), 70.0);   // ❌ compiles & passes/fails identically...
assertEquals(70.0, acct.getBalance());   // ✅ ...but failure MESSAGES lie when swapped:
// "expected: <35.0> but was: <70.0>" would blame the wrong side
```

JUnit's convention: **expected first, actual second.**

### 2. Comparing doubles without a delta

```java
assertEquals(0.3, 0.1 + 0.2);            // ❌ fails! 0.30000000000000004
assertEquals(0.3, 0.1 + 0.2, 1e-9);     // ✅ tolerance
```

### 3. Tests depending on each other

```java
static List<String> shared = new ArrayList<>();   // ❌ static state survives across tests
@Test void a() { shared.add("x"); }
@Test void b() { assertEquals(1, shared.size()); }  // passes/fails depending on ORDER
```

**Fix:** instance fields + `@BeforeEach` rebuild. If a test only passes when run after another, it's broken.

### 4. Testing everything through printed output

Code that only `System.out.println`s is hard to test. **Design for testability**: methods *return* values; a thin `main` does the printing. (This is why Chapter 5 made you extract `fizzBuzzOf(int)` — that method is trivially testable; a print-only loop isn't.)

### 5. One giant test method

```java
@Test void testEverything() { /* 40 asserts */ }   // ❌ first failure hides the other 39
```

Split per behavior; use `assertAll` when several assertions genuinely describe one outcome.

### 6. Only testing the happy path

The bugs live at the edges: empty lists, zero, negative numbers, `null`, capacity limits, absolute zero. For every method ask: what's the weirdest legal input, and the most likely illegal one? Write both tests. `assertThrows` makes illegal-input contracts explicit.

## Practice Exercises

1. **Test your MathKit.** Write a JUnit test class for Chapter 5's `MathKit` (`max3`, `isEven`, `factorial`, `clamp`): happy paths, boundaries (`clamp` at exactly min/max; `factorial(0)`), and error cases (`factorial(-1)` — decide the contract, then assert it with `assertThrows`). Aim for 10+ focused test methods.
2. **Bug hunt by test.** Write this deliberately buggy method, then find both bugs *with tests before reading the code closely*: `static double average(int[] xs) { int sum = 0; for (int i = 1; i < xs.length; i++) sum += xs[i]; return sum / xs.length; }`. Write failing tests exposing each bug, fix the method, watch them go green.
3. **Test-first Caesar.** Redo Chapter 6's Caesar cipher **test-first**: write `CaesarTest` with cases for basic shift, wrap-around (`z` + 1), case preservation, non-letters untouched, and the encrypt/decrypt round trip — all failing. Only then implement until green. Note in a comment how the tests changed what you wrote.
4. **Parameterize the leap years.** Convert Chapter 3's leap-year logic into `static boolean isLeap(int year)` and cover it with one `@ParameterizedTest` + `@CsvSource` including 1900 (no), 2000 (yes), 2024 (yes), 2025 (no), 4 (yes), 100 (no). Add `@DisplayName`s that read as sentences.
5. **Fixture discipline.** Take your Chapter 12 gradebook (`Map<String, List<Integer>>` operations) and test it with a `@BeforeEach`-built fixture of 3 students. Include: average correctness, best-student tie behavior (define it!), unknown-student lookups, and adding to a brand-new student. Then deliberately convert your fixture field to `static`, watch tests interfere (or order-depend), revert, and record the lesson in a comment.
