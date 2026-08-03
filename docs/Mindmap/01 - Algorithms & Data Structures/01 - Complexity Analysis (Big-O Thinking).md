---
title: Complexity Analysis (Big-O Thinking)
mindmap_id: complexity-analysis
node_type: topic
category: Algorithms & Data Structures
parent: "[[00 - Algorithms & Data Structures (Overview)]]"
tags: [coding-interview, algorithms-data-structures]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Complexity Analysis (Big-O Thinking)

> The vocabulary for describing how an algorithm's running time and memory usage scale as input size grows — the lens every other technique in this vault is evaluated through.

## Definition & Core Concepts
Big-O notation describes the **upper bound** on how an algorithm's resource usage (time or space) grows as the input size `n` grows toward infinity. It deliberately throws away constant factors and lower-order terms because those depend on hardware, language, and compiler — what's left is the *shape* of growth, which is what actually determines whether an algorithm chokes on large inputs. Formally, `f(n) = O(g(n))` means there exist constants `c` and `n₀` such that `f(n) ≤ c·g(n)` for all `n ≥ n₀`. In interview practice you almost never need the formal definition — you need the intuition: "if I double the input, what happens to the work done?"

Big-O has two lesser-used siblings that round out the picture. **Big-Ω (Omega)** describes a *lower* bound — the best case, or a guarantee that an algorithm can never do better than this. **Big-Θ (Theta)** describes a *tight* bound — when the upper and lower bounds match, meaning the algorithm's growth rate is precisely characterized in both directions. In casual interview speech "Big-O" is usually used loosely to mean Big-Θ (i.e. "this is exactly how it scales"), but knowing the distinction shows precision: e.g., linear search is O(n) worst case, but Ω(1) best case (the target is the first element), so its tight bound Θ(n) only applies when you're talking about the general/average case.

**Deriving complexity from code shape:**
- **Sequential statements** add: two O(n) loops back-to-back is still O(n) (you drop the 2× constant).
- **Nested loops multiply**: a loop inside a loop, each running roughly n times, is O(n²) — because for each of n outer iterations you do n inner iterations. Three nested loops over the same n is O(n³).
- **Loops that shrink the problem by a constant factor** (e.g. `i *= 2` each iteration, or repeatedly halving a search range) give O(log n) — because you can only halve/double a quantity `log₂(n)` times before hitting the boundary.
- **Recursion**: model it as a recurrence relation. A recursive function that makes 1 call on a problem of size n-1 and does O(1) work per call is O(n) (like factorial). A function that makes 2 recursive calls each on a problem of size n-1 (like naive Fibonacci) is O(2ⁿ) — the call tree branches exponentially. A function that makes 2 calls each on size n/2 and does O(n) work to combine them (like merge sort) is O(n log n) by the Master Theorem.
- **Space complexity** counts *extra* memory beyond the input — including the call stack for recursive solutions. A recursive function with recursion depth n uses O(n) stack space even if it allocates no other data structures.

**Common complexity classes, from best to worst, each with a canonical example:**

| Complexity | Name | Example algorithm |
|---|---|---|
| O(1) | Constant | Array index access `arr[i]`; hash map lookup (average case) |
| O(log n) | Logarithmic | Binary search on a sorted array |
| O(n) | Linear | Single pass through an array to find the max element |
| O(n log n) | Linearithmic | Merge sort, heap sort, or any comparison-based sort's optimal bound |
| O(n²) | Quadratic | Bubble sort; brute-force pair-checking (nested loop over all pairs) |
| O(2ⁿ) | Exponential | Naive recursive Fibonacci; generating all subsets of a set |

**Time–space tradeoffs**: very often you can trade memory for speed. The classic example is turning an O(n²) brute-force "check every pair" algorithm into O(n) time by spending O(n) extra space on a hash map that remembers what you've already seen (see [[02 - Arrays, Hashing & Strings]]). Memoization in dynamic programming is the same trade: O(n) or O(n·m) extra space to avoid recomputing overlapping subproblems, turning exponential recursion into polynomial time.

## Best Practices
- **State the brute force complexity out loud first**, even if it's obviously bad. Interviewers want to hear "the naive approach is O(n²) because we check every pair — can I think about how to do better?" before you jump to the optimized solution. It shows you're reasoning, not pattern-matching.
- **Watch for "secretly quadratic" code.** A single loop that calls `.contains()` or slicing (`arr[:i]`) on a Python list/array inside it is still O(n²) overall, because each `.contains()` or slice is itself O(n). This is one of the most common "optimized but not really" mistakes — swapping a nested `for` loop for a `while` loop with an inner linear operation doesn't change the underlying complexity.
- **String concatenation in a loop** (`result += char` in a loop, in languages where strings are immutable like Python/Java) is O(n²) overall because each concatenation copies the whole string so far. Use a list/StringBuilder and join/build once at the end for O(n).
- **Don't confuse average case and worst case.** Hash map lookup is O(1) average but O(n) worst case (all keys collide into one bucket). Quicksort is O(n log n) average but O(n²) worst case on adversarial/already-sorted input. State which one you mean.
- **Drop constants and lower-order terms, but don't drop them too early in your head** — an algorithm that's O(n) with a huge constant factor (e.g. it does 1000 operations per element) can be slower in practice than an O(n log n) algorithm for realistic input sizes. This matters for real systems, less so for interview correctness, but it's worth mentioning if asked about real-world performance.
- **Account for the cost of built-in operations you call.** `sorted(arr)` is O(n log n), not free; `x in some_list` is O(n) but `x in some_set` is O(1) average — the underlying data structure of a "helper" call changes your total complexity.

## Real-World Use Case
Illustrative scenario: a relational database's query planner chooses between a sequential table scan (O(n) — read every row) and an index lookup via a B-tree (O(log n) — descend the tree) when deciding how to execute a `WHERE` clause. The same asymptotic reasoning taught for interview problems is what lets a database estimate, for a given table size, which physical execution plan will actually be cheaper — this is why adding an index to a large, frequently-filtered column can turn a slow query fast without changing a single line of application code.

## Hands-On Practice
Three problem types where naming the complexity class correctly is the entire interview signal:
1. **Two Sum** — brute force checks all pairs: O(n²) time, O(1) space. Optimized with a hash map: O(n) time, O(n) space. Being able to state both, and explain the space-for-time trade, is the point of the problem.
2. **Merge Two Sorted Lists** — walking both lists once with two pointers is O(n + m) time, O(1) extra space (excluding the output) — recognizing that this is linear, not quadratic, despite touching two different lists, is the key insight.
3. **Contains Duplicate** — brute force is O(n²) (compare every pair). Sorting first gives O(n log n) time, O(1) extra space (if sorting in place). A hash set gives O(n) time, O(n) space. Walk through: setup (read the array once), approach (insert into a set, check membership before inserting), complexity of result (single pass, O(n) time because set operations are O(1) average, O(n) space for the set).

## Exam Tips
- When asked "can you do better?", the interviewer is almost always fishing for you to trade space for time (arrays → hash map) or to trade a nested loop for a single pass with pointers (two pointers/sliding window) or a divide step (binary search, merge sort).
- Common off-by-one-adjacent mistake: conflating O(n) and O(n log n) when a solution *sorts first, then scans*. Sorting dominates: O(n log n) + O(n) = O(n log n), not O(n).
- Don't over-claim O(1) space when your solution uses recursion — the call stack counts. A recursive in-order tree traversal is O(h) space where h is tree height, not O(1), even though no explicit data structure is allocated.
- If an interviewer asks for the *tightest* bound, that's a Big-Θ question in disguise — don't just say "it's O(n²)" if the algorithm always does exactly n²/2 operations regardless of input; say "it's Θ(n²), both upper and lower bound."
- Amortized complexity is a common trap: a dynamic array's `append` is O(1) *amortized* even though occasional resizes cost O(n) — know the difference between "always O(1)" and "O(1) on average over a sequence of operations."

## References
- [Big O Cheat Sheet – Time Complexity Chart](https://www.freecodecamp.org/news/big-o-cheat-sheet-time-complexity-chart/)
- [Big O Notation — AlgoMap](https://algomap.io/lessons/big-o-notation)

## Related
- Parent: [[00 - Algorithms & Data Structures (Overview)]]
- Siblings: [[02 - Arrays, Hashing & Strings]], [[03 - Two Pointers, Sliding Window & Binary Search]], [[04 - Trees, Graphs & Traversal]], [[05 - Dynamic Programming & Recursion]]
