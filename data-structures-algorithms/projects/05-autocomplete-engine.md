# Project 5: Autocomplete Engine

## Description

Build the thing every search box has: type a prefix, get ranked suggestions instantly. You'll implement it twice with different structures — first on a **BST** (with a range-style prefix walk), then on a **trie** (prefix tree), the structure purpose-built for this job — load a real word list into both, wire in frequency-based ranking with a heap, and benchmark the two engines against each other and against a naive linear scan. An interactive prompt loop gives the payoff: suggestions as you type.

## Difficulty

**Intermediate–Advanced.** Estimated effort: 8–12 hours.

## Chapters used

- Chapter 5 (hash maps for children and frequencies)
- Chapter 6 (recursion: trie traversal and collection)
- Chapter 9 (BSTs, ordered traversal, range queries)
- Chapter 10 (heap for top-k ranking)

## Requirements checklist

### Engine A — BST-based
- [ ] A BST keyed on words (string comparison), with insert and in-order traversal, built by you (adapting your Chapter 9 work is fine — no library trees)
- [ ] `suggest(prefix, k)`: return up to k words starting with `prefix`, using a **pruned** range walk (skip subtrees that cannot contain the prefix range — like Chapter 9's `range_query`), never a full traversal
- [ ] Build the tree from a shuffled word list, and document why inserting an alphabetized list instead would be catastrophic (measure the height both ways)

### Engine B — trie-based
- [ ] A `TrieNode` with a dict of children and an end-of-word flag (plus a frequency field); a `Trie` with `insert(word, freq)`, `contains(word)`, and `starts_with(prefix)`
- [ ] `suggest(prefix, k)`: walk to the prefix node (O(len(prefix))), then collect completions beneath it recursively
- [ ] Ranking: suggestions ordered by frequency (highest first), ties alphabetical — selected with a **heap-based top-k** (your own heap or `heapq`), not by sorting all completions
- [ ] `delete(word)` that unflags the word and prunes now-useless nodes

### Data, interface, and proof
- [ ] Load a real word list (e.g., a `words.txt` — thousands of entries; frequencies can be synthetic, e.g., inverse rank) with a documented file format
- [ ] Interactive loop: user types a prefix, sees top 5 suggestions; typing more narrows them; handles prefixes with zero matches gracefully
- [ ] Baseline for honesty: a `LinearEngine` that scans the whole list with `startswith` per query
- [ ] Benchmark all three engines: build time, memory proxy (node/entry count), and per-query latency for short (1-char), medium (3-char), and long prefixes; results table in `RESULTS.md` with a paragraph interpreting *why* the trie wins where it wins
- [ ] Shared test suite run against all engines: same queries, same expected suggestion sets (before ranking), plus ranking-specific tests for the trie

## Hints

- BST prefix trick: all words with prefix `p` fall in the range `[p, p + "￿")` lexicographically — your Chapter 9 `range_query` pruning logic applies almost verbatim, with "collect at most k" as an early exit.
- In the trie collector, build the current word incrementally as you recurse (pass `path + ch`, or append/pop a char list — the Chapter 6 backtracking pattern). Collect `(freq, word)` pairs and let the heap do selection.
- Top-k with a min-heap of size k: the root is the weakest current suggestion; a candidate must beat it to enter (Chapter 10's pattern). Mind the tie rule — you may need a key tuple that inverts one component.
- `delete` pruning: recurse to the leaf, then remove child links on the way back *only if* the child has no children and isn't itself a word-end. Post-order thinking (children before parents).
- No word list handy? Generate one: random pronounceable strings work, but also grab any long public-domain text and split it into unique words — frequency then comes free (count occurrences — Chapter 5).

## Stretch goals

- Case-insensitive matching that still displays original casing.
- "Did you mean": if a prefix has no matches, suggest words at edit distance 1 from the typed string (generate variants, check the trie — connects to Chapter 5's spell-checker exercise).
- Weight recency: `record_selection(word)` bumps a word's frequency so the engine learns from use; decide where that write goes in each engine and what it costs.
- Compressed trie (radix tree): merge single-child chains into multi-character edges; re-measure node count against the plain trie.
- Serialize the trie to disk and lazy-load it, so startup doesn't rebuild from the word list.
