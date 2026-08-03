---
title: Two Pointers, Sliding Window & Binary Search
mindmap_id: two-pointers-sliding-window-binary-search
node_type: topic
category: Algorithms & Data Structures
parent: "[[00 - Algorithms & Data Structures (Overview)]]"
tags: [coding-interview, algorithms-data-structures]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Two Pointers, Sliding Window & Binary Search

> Three techniques united by one idea: instead of exhaustively re-scanning the input for every candidate answer, use structure (sortedness, contiguity, monotonicity) to eliminate large chunks of the search space in one move.

## Definition & Core Concepts
**Two pointers** uses two indices moving through a sequence (usually sorted, or with some other exploitable structure) instead of nested loops. There are two common shapes:
- **Converging pointers**: one starts at the left, one at the right, and they move toward each other based on a comparison, eliminating one end of the range per step. Classic use: Two Sum on a *sorted* array — if `arr[left] + arr[right] > target`, decrement `right` (you need a smaller sum); if `< target`, increment `left`. Each comparison eliminates one element from consideration, giving O(n) instead of O(n²).
- **Fast/slow pointers ("tortoise and hare")**: both pointers start together but move at different speeds (typically fast moves 2 steps for every 1 slow step). This detects cycles in linked lists (if there's a cycle, fast eventually laps slow) and finds the middle of a list in one pass (when fast reaches the end, slow is at the midpoint).

**Sliding window** is two-pointers specialized for **contiguous subarrays/substrings**. You maintain a window `[left, right]` and a running aggregate (a sum, a character-frequency map, a count of distinct elements) over the elements currently inside it:
- **Fixed-size window**: the window size k is given up front. Slide it one step at a time — add the new right element, remove the element falling off the left — updating the aggregate incrementally instead of recomputing it from scratch each time (this incremental update is what makes it O(n) instead of O(n·k)).
- **Variable-size window**: you grow `right` to expand the window, and shrink `left` (in a `while` loop) whenever the window violates a constraint (e.g. "at most k distinct characters," "sum exceeds target"). Because `left` and `right` each only move forward and each traverses the array at most once over the whole algorithm, total work is O(n) even though there's a nested-looking `while` inside a `for` — this "amortized O(n), not O(n²)" argument is the single most important complexity insight for this pattern and the one most commonly mis-stated as O(n²) by mistake.

**Binary search** exploits a **monotonic** (sorted, or "sorted enough") structure to eliminate half the remaining search space on every comparison, giving O(log n). The textbook form searches a sorted array for a target value: compare the middle element to the target, and discard the half that can't contain it. The far more powerful interview variant is **"binary search on the answer"**: instead of searching an array, you search a *range of possible answers* to an optimization problem (e.g. "what's the minimum capacity that lets me ship all packages within D days?"). This works whenever the underlying question is monotonic — i.e., there's a threshold value X such that every value ≥ X satisfies some checkable condition ("feasible") and every value < X doesn't. You binary search over X, using a (usually O(n)) feasibility check as the comparison, giving O(n log(range)) overall instead of testing every candidate value one by one.

## Best Practices
- **Recognize the trigger conditions.** Two pointers/binary search usually require sorted input (or you sort first, paying O(n log n)). Sliding window requires the answer to live in a *contiguous* run of the array/string — if the problem allows picking non-contiguous elements, sliding window doesn't apply and you likely need DP or a different structure.
- **The variable-window "shrink" loop is the most common bug source.** Forgetting to shrink the window (or shrinking with an `if` instead of a `while`) means the window can stay invalid, silently corrupting the answer. Always ask: "after I expand, could the window now be invalid in a way that requires shrinking more than once?"
- **Off-by-one on window size**: window length is `right - left + 1`, not `right - left` — a very common slip when computing "longest substring" style answers.
- **Binary search boundary bugs**: `while left <= right` vs `while left < right`, and whether you update with `mid` or `mid ± 1`, must be chosen consistently or you get infinite loops or an off-by-one wrong answer. Always sanity-check with a 1-element and 2-element array by hand.
- **"Binary search on the answer" requires proving monotonicity first** — don't reach for it just because the problem asks for a min/max; verify (even just informally, in your head) that the feasibility condition really does partition the range into "no's" then "yes's" with no flip-flopping, or the binary search is invalid.
- **Fast/slow pointers**: always null-check `fast` AND `fast.next` before advancing fast by two, or you'll dereference past the end of the list.

## Real-World Use Case
Case study: binary search's O(log n) behavior is the canonical example used to teach logarithmic-time algorithms, and the same divide-and-eliminate-half idea ships directly in standard libraries — Python's `bisect` module and Java's `Collections.binarySearch` both implement it for searching sorted sequences. Illustrative scenario: the "narrow the search space" idea behind sliding window also shows up outside pure algorithms — a monitoring dashboard computing a rolling "average latency over the last 5 minutes" is conceptually a fixed-size sliding window over a stream of timestamped events, incrementally adding new events and dropping ones that fall out of the window rather than recomputing the average from scratch.

## Hands-On Practice
Three canonical problem types, one walked through in depth:
1. **Two Sum II (Input Array Is Sorted)** — converging two pointers, walked through: setup — `left = 0`, `right = n - 1`. Approach — compute `sum = arr[left] + arr[right]`; if it equals target, return the pair; if it's less than target, increment `left` (need a bigger sum); if greater, decrement `right` (need a smaller sum); repeat until pointers meet. Complexity of result: O(n) time (each step eliminates exactly one element, pointers can move at most n total steps combined), O(1) extra space — strictly better than the O(n) space of the hash-map Two Sum approach, because sortedness gives you structure to exploit instead.
2. **Longest Substring Without Repeating Characters** — variable-size sliding window with a hash set/map tracking characters currently in the window; expand `right`, and whenever a duplicate is found, shrink `left` past the previous occurrence. O(n) time, O(min(n, alphabet size)) space.
3. **Koko Eating Bananas** (binary search on the answer) — search over possible eating speeds `[1, max(piles)]`; for each candidate speed, an O(n) check computes whether Koko can finish all piles within H hours; binary search over the speed range using that feasibility check gives O(n log(max(piles))) overall, dramatically better than testing every possible speed linearly.

## Exam Tips
- If a problem says "contiguous subarray/substring" and asks for a max/min/count satisfying some constraint, say "sliding window" out loud before coding — it signals pattern recognition to the interviewer.
- If a problem gives a *sorted* array and asks for a pair/triple with some sum property, two pointers beats a hash map on space (O(1) vs O(n)) — mention both and explain why you'd pick pointers here.
- Don't binary-search on unsorted data without either sorting first (and paying O(n log n)) or recognizing you actually want "binary search on the answer" over a *value range*, not the array itself — conflating these two is a common beginner mistake.
- For fast/slow pointer cycle detection (Floyd's algorithm), know that after detecting a cycle, finding its *start* requires resetting one pointer to the head and advancing both at the same speed — a frequently-forgotten follow-up.
- State window-size and complexity explicitly when you finish a sliding-window solution; interviewers specifically listen for "O(n), each pointer moves forward at most n times total" to confirm you're not secretly presenting an O(n²) solution.

## References
- [Sliding Window Technique: A Comprehensive Guide — LeetCode Discuss](https://leetcode.com/discuss/post/3722472/sliding-window-technique-a-comprehensive-ix2k/)
- [15 LeetCode Patterns](https://blog.algomaster.io/p/15-leetcode-patterns)
- [Big O Cheat Sheet – Time Complexity Chart](https://www.freecodecamp.org/news/big-o-cheat-sheet-time-complexity-chart/)

## Related
- Parent: [[00 - Algorithms & Data Structures (Overview)]]
- Siblings: [[01 - Complexity Analysis (Big-O Thinking)]], [[02 - Arrays, Hashing & Strings]], [[04 - Trees, Graphs & Traversal]], [[05 - Dynamic Programming & Recursion]]
