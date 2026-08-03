---
title: Arrays, Hashing & Strings
mindmap_id: arrays-hashing-strings
node_type: topic
category: Algorithms & Data Structures
parent: "[[00 - Algorithms & Data Structures (Overview)]]"
tags: [coding-interview, algorithms-data-structures]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Arrays, Hashing & Strings

> The most fundamental data layout (contiguous, indexable memory) plus the single highest-leverage trick in interview algorithms: trading O(n) space for a hash map to collapse O(n²) brute force into O(n).

## Definition & Core Concepts
An **array** is a contiguous block of memory holding elements of the same type, addressable by index in O(1) time. This O(1) random access is *the* defining property that makes arrays different from linked lists — you pay for it with O(n) insertion/deletion in the middle (everything after the gap must shift). A **string** is, for algorithmic purposes, just an array of characters, so almost every array technique (two pointers, sliding window, hashing) applies directly to strings — the extra wrinkle is that strings are often *immutable* in high-level languages (Python, Java, JavaScript), which changes the cost model for building output incrementally (see Best Practices).

A **hash map** (hash table / dictionary) stores key-value pairs and, via a hash function, provides average-case O(1) insert, lookup, and delete regardless of how many elements it holds. It works by computing `hash(key) mod table_size` to pick a "bucket," then handling the rare case where two different keys land in the same bucket (a **collision**) either by chaining (each bucket holds a small linked list) or open addressing (probe to the next free slot). A **hash set** is the same structure with no attached value — it only answers "have I seen this key before?"

**The core pattern: "have I seen this before?"** A huge fraction of brute-force O(n²) algorithms have this shape: for each element, scan the rest of the array/string to check some condition against it (a matching pair, a duplicate, a required complement). The hash map insight is that "scan the rest of the array" is exactly what a hash set answers in O(1): instead of re-scanning, you build the set *as you go*, so by the time you reach element `i`, the set already contains "everything I've seen so far," and checking membership is O(1) instead of O(n). This single substitution is what turns Two Sum, Contains Duplicate, and dozens of variants from O(n²) into O(n).

**Frequency counting** is the sibling pattern: instead of a set (yes/no), use a hash map from element → count. This answers questions like "what's the most common character," "are two strings anagrams of each other" (compare frequency maps for equality), or "does this window contain at most k distinct characters." In many languages this is a built-in (Python's `collections.Counter`), but knowing how to build one by hand (`map[char] = map.get(char, 0) + 1`) is expected.

**Grouping by a computed key** extends frequency counting: instead of counting, you bucket original elements by some derived signature. The canonical example is Group Anagrams — the signature is the sorted string (or a 26-length character-count tuple), and every word producing the same signature goes in the same bucket (a hash map from signature → list of original strings).

## Best Practices
- **Recognize the trigger phrase.** "Find if there exists," "count pairs/triples," "have you seen," "is this a duplicate," "what's the frequency of" — these almost always mean brute force is O(n²) or O(n³) and a hash map gets you to O(n) or O(n log n).
- **Watch for the "secretly still O(n²)" trap**: using `list.index(x)` or `x in some_list` (not a set) inside a loop is still linear *per call*, so wrapping it in an outer loop is still O(n²) even though the code "uses a lookup." The lookup must be against a *hash-based* structure (set/dict), not a list, to get O(1).
- **String immutability matters for complexity.** In Python and Java, `s += char` inside a loop reallocates and copies the entire string each time, making a loop that builds a string of length n cost O(n²) total. Use a list/array of characters (or `StringBuilder` in Java) and join/build once at the end.
- **Know your hash map's worst case.** Average O(1) assumes a reasonably uniform hash function and low load factor; a pathological/adversarial input (or a bad custom hash function) can degrade to O(n) per operation. This is rarely the interview's point, but it's worth knowing exists.
- **Two Sum specifically**: the "one-pass hash map" version (check if `target - num` is already in the map *before* inserting the current number) is O(n) with a single pass, versus a "two-pass" version (build the whole map first, then scan) which also works but is a slightly weaker mental model to build on for follow-ups (like Two Sum II on a sorted array, which instead wants two pointers — see [[03 - Two Pointers, Sliding Window & Binary Search]]).
- **Prefix sums** are a related array technique worth knowing alongside hashing: precompute cumulative sums so any range-sum query becomes O(1) instead of O(n) per query. Combined with a hash map of prefix-sum frequencies, this solves "subarray sum equals k" in O(n) — a strong signal that hashing generalizes beyond just "have I seen this element."

## Real-World Use Case
Case study: hash tables are the core data structure behind virtually every in-memory key-value cache (e.g. Redis, Memcached) and every language's built-in dictionary/map type (Python `dict`, Java `HashMap`, JavaScript `Map`) — the average O(1) lookup guarantee is precisely why these are the default choice for "look something up by key fast," from session caches to database indexes' in-memory components. [ref: Hash Table Data Structure — GeeksforGeeks]

## Hands-On Practice
Three canonical problem types this pattern family solves:
1. **Two Sum** — given an array and a target, find two indices whose values sum to target. Setup: iterate once, keeping a hash map of `value → index` seen so far. Approach: for each number, compute `complement = target - num`; if `complement` is already in the map, you've found your pair; otherwise insert the current number and its index. Complexity of result: O(n) time (single pass, O(1) hash operations), O(n) space for the map.
2. **Group Anagrams** — given a list of strings, group the ones that are anagrams of each other. Setup: a hash map from a canonical signature to a list of original strings. Approach: for each string, compute its signature (sort the characters, or build a 26-count tuple for lowercase English letters), and append the original string to `map[signature]`. Complexity of result: O(n · k log k) time where k is average string length (dominated by sorting each string), or O(n · k) if using the count-tuple signature instead of sorting.
3. **Longest Consecutive Sequence** — given an unsorted array, find the length of the longest run of consecutive integers. Setup: put all numbers in a hash set for O(1) membership checks. Approach: for each number, only start counting a sequence if `num - 1` is NOT in the set (i.e., it's the start of a run) — then count upward (`num+1`, `num+2`, ...) while each is present. Complexity of result: O(n) time overall, because the inner "count upward" loop only runs for true sequence starts, so every element is visited a constant number of times across the whole algorithm — this "each element visited O(1) amortized times" argument is a common and important complexity-analysis trick.

## Exam Tips
- If your first instinct is a nested loop with an `if this[i] == this[j]` style check, stop and ask: "can a hash set/map answer this membership question instead?" before writing any code.
- Anagram problems: don't forget that "sort the string" and "compare character-count arrays" are both valid O(1)-alphabet-size techniques — sorting costs O(k log k) per string, counting costs O(k), so counting is asymptotically better but sorting is often less code to write under time pressure. State the trade-off if asked.
- A common off-by-one/logic bug in "longest consecutive sequence"-style problems: forgetting the `num - 1 not in set` guard, which turns an O(n) amortized algorithm into an accidental O(n²) one (every element re-walks the same run from scratch).
- Don't over-apply hashing where order matters and you'd need a *sorted* structure instead — e.g. "find the smallest element ≥ x" needs a sorted array/binary search or a balanced BST/treemap, not a plain hash map, because hash maps have no notion of order.
- When asked to check for duplicates, always mention the three options and their trade-offs: sort first (O(n log n) time, O(1) extra space if in-place), hash set (O(n) time, O(n) space) — this shows you understand the time/space trade-off explicitly rather than reaching for one option by habit.

## References
- [Hash Table Data Structure — GeeksforGeeks](https://www.geeksforgeeks.org/dsa/hash-table-data-structure/)
- [15 LeetCode Patterns](https://blog.algomaster.io/p/15-leetcode-patterns)

## Related
- Parent: [[00 - Algorithms & Data Structures (Overview)]]
- Siblings: [[01 - Complexity Analysis (Big-O Thinking)]], [[03 - Two Pointers, Sliding Window & Binary Search]], [[04 - Trees, Graphs & Traversal]], [[05 - Dynamic Programming & Recursion]]
