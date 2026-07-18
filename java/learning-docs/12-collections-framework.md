# Chapter 12: The Collections Framework (List, Set, Map)

## Overview

Fixed-size arrays get old fast. The **Java Collections Framework** is the standard library's set of growable, feature-rich data structures — the equivalent of Python's `list`/`set`/`dict` or JS's `Array`/`Set`/`Map` — and it's used in essentially every Java method you'll ever write professionally. This chapter covers the three core interfaces (`List`, `Set`, `Map`), their workhorse implementations (`ArrayList`, `HashSet`, `HashMap`), iteration, and the `equals`/`hashCode` contract that makes hashing work.

## Definitions & Explanations

### The big picture: interfaces + implementations

The framework is designed exactly along Chapter 10 lines — interfaces define contracts, classes implement them:

| Interface | Meaning | Main implementations | Python/JS analogue |
|-----------|---------|----------------------|--------------------|
| `List<E>` | ordered, indexed, duplicates OK | `ArrayList`, `LinkedList` | `list` / `Array` |
| `Set<E>`  | no duplicates | `HashSet` (unordered), `TreeSet` (sorted), `LinkedHashSet` (insertion order) | `set` / `Set` |
| `Map<K,V>`| key → value pairs | `HashMap`, `TreeMap`, `LinkedHashMap` | `dict` / `Map` |

The `<E>` / `<K,V>` are **generics** (fully explained in Chapter 13): `List<String>` is "a list of Strings," checked by the compiler. Collections hold *objects only* — for primitives you use the wrappers: `List<Integer>`, not `List<int>` (autoboxing makes this mostly painless).

**Program to the interface** — declare variables and parameters as the interface type:

```java
List<String> names = new ArrayList<>();   // ✅ swap implementation later, nothing else changes
ArrayList<String> names = new ArrayList<>(); // works, but needlessly locks you in
```

The `<>` "diamond" on the right lets the compiler infer the type parameter.

### List — ordered sequences

```java
List<String> names = new ArrayList<>();
names.add("Ada");                 // append           (Python: append / JS: push)
names.add(0, "Grace");            // insert at index
names.get(1)                      // read by index    — NOT names[1]!
names.set(1, "Adele");            // replace by index
names.remove("Grace");            // remove by VALUE...
names.remove(0);                  // ...or by INDEX (see pitfalls!)
names.size()                      // length           — not .length or length()
names.contains("Adele")           // membership test
names.indexOf("Adele")            // first position or -1
names.isEmpty()
```

`ArrayList` is a resizable array (fast get/set, fast append; slow inserts in the middle) — the right default. `LinkedList` is rarely the right choice; know it exists.

`List.of("a", "b", "c")` creates a compact **immutable** list — great for fixed data and test inputs; calling `add` on it throws `UnsupportedOperationException`.

### Set — uniqueness

```java
Set<String> tags = new HashSet<>();
tags.add("java");
tags.add("java");          // returns false; still one copy
tags.contains("java")      // true — and FAST (hashing), even with millions of elements
```

Use a `Set` whenever the question is "have I seen this before?" or "no duplicates allowed." `HashSet` has no defined order; `TreeSet` keeps elements sorted (requires `Comparable` elements).

### Map — key/value lookup

```java
Map<String, Integer> stock = new HashMap<>();
stock.put("apples", 12);              // insert or overwrite
stock.get("apples")                   // 12 — or NULL if key absent (not an error!)
stock.getOrDefault("kiwis", 0)        // 0 — null-safe reading
stock.containsKey("apples")           // true
stock.remove("apples");
stock.size();
stock.keySet()                        // Set<String> of keys
stock.values()                        // Collection<Integer> of values
stock.entrySet()                      // Set<Map.Entry<K,V>> — for iterating pairs
```

Handy mutation helpers: `stock.merge("apples", 1, Integer::sum)` and `stock.putIfAbsent(k, v)`. The counting idiom below is one you'll write constantly.

### Iteration

```java
for (String name : names) { ... }                     // for-each works on all collections

for (Map.Entry<String, Integer> e : stock.entrySet()) {
    System.out.println(e.getKey() + " = " + e.getValue());
}
```

**The removal rule:** you may not structurally modify a collection while for-each iterating it (`ConcurrentModificationException`). Use an explicit `Iterator` with `it.remove()`, or the one-liner `names.removeIf(n -> n.isBlank());`.

### equals & hashCode — why hashing works

`HashSet`/`HashMap` locate elements by `hashCode()` first, then confirm with `equals()`. For your own classes to behave as keys/set-elements, you must override **both**, consistently: equal objects ⇒ equal hash codes. Easiest correct route today: make the class a **record** (Java 16+), which generates `equals`, `hashCode`, and `toString` for you:

```java
public record Point(int x, int y) { }   // immutable, value-semantics, done
```

Otherwise generate both overrides with IntelliJ (Code → Generate) or `Objects.equals`/`Objects.hash`.

## Code Examples

### List fundamentals

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ListDemo {
    public static void main(String[] args) {
        List<String> tasks = new ArrayList<>();
        tasks.add("write report");
        tasks.add("review PR");
        tasks.add("deploy");
        tasks.add(1, "coffee break");            // insert at index 1

        System.out.println(tasks);               // collections print nicely, unlike arrays!
        System.out.println("count: " + tasks.size());
        System.out.println("first: " + tasks.get(0));

        tasks.remove("coffee break");
        Collections.sort(tasks);                 // in-place alphabetical sort
        System.out.println(tasks);

        // Iterate with index when you need positions
        for (int i = 0; i < tasks.size(); i++) {
            System.out.println((i + 1) + ". " + tasks.get(i));
        }
    }
}
```

### Word frequency — the classic Map idiom

```java
import java.util.HashMap;
import java.util.Map;

public class WordCount {
    public static void main(String[] args) {
        String text = "the quick brown fox jumps over the lazy dog the end";

        Map<String, Integer> counts = new HashMap<>();
        for (String word : text.split("\\s+")) {
            // read-modify-write, null-safe:
            counts.merge(word, 1, Integer::sum);
            // equivalent long form:
            // counts.put(word, counts.getOrDefault(word, 0) + 1);
        }

        for (Map.Entry<String, Integer> e : counts.entrySet()) {
            System.out.println(e.getKey() + ": " + e.getValue());
        }
        System.out.println("'the' appeared " + counts.getOrDefault("the", 0) + " times");
    }
}
```

### Set for de-duplication

```java
import java.util.*;

public class DedupDemo {
    public static void main(String[] args) {
        List<String> visitors = List.of("ada", "bob", "ada", "cy", "bob", "ada");

        Set<String> unique = new HashSet<>(visitors);      // constructor de-dupes
        System.out.println("visits: " + visitors.size());  // 6
        System.out.println("unique: " + unique.size());    // 3

        // TreeSet: unique AND sorted
        Set<String> sortedUnique = new TreeSet<>(visitors);
        System.out.println(sortedUnique);                  // [ada, bob, cy]

        // Membership: the whole point of sets
        System.out.println(unique.contains("cy"));         // true, O(1)
    }
}
```

### Safe removal while iterating

```java
import java.util.*;

public class RemovalDemo {
    public static void main(String[] args) {
        List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6));
        // note: new ArrayList<>(...) makes a MUTABLE copy of the immutable List.of

        // ❌ for (Integer n : nums) if (n % 2 == 0) nums.remove(n);   // ConcurrentModificationException

        nums.removeIf(n -> n % 2 == 0);    // ✅ the modern one-liner (lambda: Ch. 15)
        System.out.println(nums);           // [1, 3, 5]
    }
}
```

## Common Pitfalls

### 1. Array habits on lists

```java
list[0]           // ❌ no bracket indexing — list.get(0)
list.length       // ❌ — list.size()
List<int> xs      // ❌ no primitives in generics — List<Integer>
```

### 2. `List<Integer>.remove` — index or value?

```java
List<Integer> xs = new ArrayList<>(List.of(10, 20, 30));
xs.remove(1);                    // removes INDEX 1 (the 20) — overload takes int
xs.remove(Integer.valueOf(30));  // ✅ removes the VALUE 30
```

Nasty because both compile. With any other element type, `remove(Object)` is unambiguous.

### 3. `Map.get` returning null

```java
int n = counts.get("missing");           // ❌ NullPointerException on unboxing null
int n = counts.getOrDefault("missing", 0); // ✅
```

### 4. Mutating `List.of` results

```java
List<String> xs = List.of("a", "b");
xs.add("c");                     // ❌ UnsupportedOperationException — it's immutable
List<String> xs = new ArrayList<>(List.of("a", "b"));   // ✅ mutable copy
```

Same trap with `Arrays.asList` (fixed-size view over an array).

### 5. Custom objects lost in HashSets

```java
class Point { int x, y; /* no equals/hashCode */ }
Set<Point> s = new HashSet<>();
s.add(new Point(1, 2));
s.contains(new Point(1, 2));     // ❌ false! default equals is identity (==)
```

**Fix:** override `equals` and `hashCode` together — or use a `record`. This is a top-tier interview question.

### 6. Modifying while iterating

```java
for (String s : list) {
    if (s.isEmpty()) list.remove(s);   // ❌ ConcurrentModificationException
}
list.removeIf(String::isEmpty);        // ✅
```

## Practice Exercises

1. **Todo list.** A console app (menu loop from Chapter 4) over a `List<String>`: add task, list tasks numbered, remove by number, quit. Guard against invalid indices. Then add "mark done" by *replacing* the entry with a "[x] "-prefixed version using `set`.
2. **Unique words, sorted.** Read a paragraph, lowercase it, strip punctuation (`replaceAll("[^a-z ]", "")`), split, and print: total word count, unique word count, and all unique words alphabetically (choose the right Set implementation — no manual sorting allowed).
3. **Gradebook map.** `Map<String, List<Integer>>` mapping student names to their scores. Support: add score for student (creating the list on first sight — look up `computeIfAbsent`), print each student's average, and print the best student. Test with at least 3 students × 3 scores.
4. **Inventory with records.** Define `record Item(String sku, String name)` and build a `Map<Item, Integer>` inventory. Demonstrate that two separately-constructed `Item` objects with the same fields act as the same key (explain in a comment why this works for records but failed in pitfall 5).
5. **Implementation shoot-out.** Insert 100,000 random ints into an `ArrayList` and a `HashSet`, then time 1,000 `contains` calls against each (`System.nanoTime()` before/after). Report the two timings and explain the difference in one comment. (Sanity check: the set should win massively.)
