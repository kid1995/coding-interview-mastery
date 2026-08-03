---
title: Dynamic Programming & Recursion
mindmap_id: dynamic-programming-recursion
node_type: topic
category: Algorithms & Data Structures
parent: "[[00 - Algorithms & Data Structures (Overview)]]"
tags: [coding-interview, algorithms-data-structures]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Dynamic Programming & Recursion

> How to recognize when a problem is solving the same sub-problem over and over, and how to stop paying for that repetition — via recursion fundamentals, memoization (top-down), and tabulation (bottom-up).

## Definition & Core Concepts
**Recursion** is a function that calls itself on a smaller version of the same problem. Every correct recursive function needs two things: a **base case** (the smallest input, solved directly, with no further recursive call — this is what stops the recursion) and a **recursive case** (how to reduce the problem toward the base case, and how to combine the result of the recursive call into the current answer). Each call adds a **stack frame** to the call stack, holding that call's local variables and its return address; this is why recursion has an implicit O(depth) space cost even when it allocates no explicit data structures, and why extremely deep recursion can overflow the stack. Recursion tends to beat iteration when the problem is *naturally* self-similar (tree traversal, divide-and-conquer, backtracking) — the code mirrors the problem's structure and is easier to prove correct. Iteration tends to beat recursion when the recursion would be simple tail-recursion over a linear structure (most languages used in interviews, notably Python, don't optimize tail calls, so a simple loop is both clearer and avoids stack-depth limits) or when the recursive solution's naive form does **redundant work** — which is exactly the problem dynamic programming solves.

**Dynamic Programming (DP)** applies when a problem has two properties simultaneously:
1. **Optimal substructure** — the optimal solution to the whole problem can be built from optimal solutions to its subproblems (e.g. the shortest path to node X is the shortest path to some predecessor, plus one edge).
2. **Overlapping subproblems** — a naive recursive solution calls itself with the *same arguments* many times. This is the tell-tale sign: draw the recursion tree for naive Fibonacci and you'll see `fib(3)` computed repeatedly across many branches — that repetition, multiplied across an exponentially branching tree, is what makes naive recursive Fibonacci O(2ⁿ) despite there only being n *distinct* subproblems.

If a problem has overlapping subproblems, you can cache each distinct subproblem's answer the first time you compute it, and return the cached value on every subsequent identical call — collapsing what was exponential work into work proportional to the number of *distinct* subproblems. This is DP's entire value proposition; without overlapping subproblems (e.g. plain divide-and-conquer like merge sort, where subproblems don't repeat), caching buys nothing.

**Top-down (memoization)**: keep the natural recursive structure, but wrap it with a cache (a hash map or array keyed by the function's arguments). Before doing real work, check if this exact input has been solved before; if so, return the cached answer immediately; otherwise compute it, store it in the cache, and return it. This is usually the *easier* transformation to reach for from a brute-force recursive solution — you're changing existing code minimally (add a cache check at the top, add a cache write before each return).

**Bottom-up (tabulation)**: build a table (usually a 1D or 2D array) iteratively, starting from the base cases and filling in every subproblem in an order that guarantees each subproblem's dependencies are already computed by the time you need them. This avoids recursive call overhead and stack-depth limits entirely, and often allows an additional space optimization (if `dp[i]` only depends on `dp[i-1]` and `dp[i-2]`, you don't need the whole array — two variables suffice).

**The standard 4-step DP template:**
1. **Define the state** — what does `dp[i]` (or `dp[i][j]`) *mean*, in a sentence? (e.g. "the minimum coins needed to make amount i.") Getting this definition precise is the single most important step — most DP struggles trace back to a fuzzy state definition.
2. **Write the recurrence** — how does `dp[i]` relate to smaller states? (e.g. `dp[i] = min(dp[i - coin] + 1 for coin in coins if coin <= i)`.)
3. **Identify the base case(s)** — the smallest state(s) that can't be decomposed further (e.g. `dp[0] = 0`).
4. **Determine the answer** — which cell of the table holds the final answer, and in what order must you fill the table so every dependency is ready when needed (usually smallest state first for bottom-up).

## Best Practices
- **Always define the state in words before writing any code.** "dp[i] = ___" filled in as a plain sentence forces you to be precise about what's being optimized/counted, and makes the recurrence almost fall out naturally once the state is right.
- **Draw the recursion tree (even just mentally) for a small example** to confirm overlapping subproblems actually exist — if every recursive call has genuinely distinct arguments, DP doesn't help; that's a signal you're facing plain divide-and-conquer or exhaustive search instead.
- **Memoization key correctness**: the cache key must capture *every* piece of state that affects the answer — missing a dimension (e.g. memoizing on index alone when the answer also depends on "remaining budget") silently returns wrong cached answers for what looks like the same call.
- **Watch for mutable default argument bugs** when memoizing with a dict passed as a default parameter in Python — the cache can leak across unrelated calls to the function if not scoped carefully (e.g. reset per top-level call, or use `functools.lru_cache`/a fresh dict).
- **Bottom-up fill order matters.** If `dp[i][j]` depends on `dp[i-1][j]` and `dp[i][j-1]`, you must fill row-by-row, left-to-right (or any order respecting both dependencies) — filling in the wrong order reads uninitialized/stale cells.
- **Space optimization is a real interview follow-up.** After a correct O(n·m) space bottom-up solution, be ready to note whether you only ever need the previous row/column (common in 1D "rolling array" DP), reducing space to O(m) or O(1).

## Real-World Use Case
Case study: memoization — caching the result of expensive function calls keyed by their arguments — is a general software-engineering technique used far beyond textbook DP problems, from caching expensive database query results to memoizing pure computed values in UI frameworks (e.g. React's `useMemo`) to avoid redundant recomputation on every render. [ref: GeeksforGeeks memoization tutorial] The underlying idea — "don't recompute something you've already computed for these exact inputs" — is identical whether the "subproblem" is `fib(30)` or a cache key in a production HTTP caching layer.

## Hands-On Practice
Three canonical problem types, one walked through with the full 4-step template:
1. **Climbing Stairs** — state: `dp[i]` = number of distinct ways to reach step i. Recurrence: `dp[i] = dp[i-1] + dp[i-2]` (you can arrive at step i from one step below via a 1-step move, or two steps below via a 2-step move). Base cases: `dp[0] = 1`, `dp[1] = 1`. Answer: `dp[n]`. Complexity of result: O(n) time, O(n) space naively (or O(1) with rolling variables, since each state only needs the previous two).
2. **Coin Change** — state: `dp[i]` = minimum number of coins to make amount i (or infinity if impossible). Recurrence: `dp[i] = min(dp[i - coin] + 1)` over every coin denomination ≤ i. Base case: `dp[0] = 0`. Answer: `dp[amount]` (or -1 if it's still infinity). Complexity of result: O(amount × number of coin denominations) time, O(amount) space.
3. **Longest Common Subsequence** — 2D state: `dp[i][j]` = length of the longest common subsequence of the first i characters of string A and first j characters of string B. Recurrence: if `A[i-1] == B[j-1]`, `dp[i][j] = dp[i-1][j-1] + 1`; otherwise `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`. Base case: `dp[0][*] = dp[*][0] = 0`. O(n·m) time and space.

## Exam Tips
- The most reliable "is this DP?" signal in interview problems: the question asks for a **count**, **minimum/maximum**, or **yes/no feasibility** over choices made across a sequence — not "list all of them" (that phrasing usually wants backtracking instead, see [[04 - Trees, Graphs & Traversal]]).
- Always state the brute-force recursive solution's exponential complexity first, then say explicitly "this has overlapping subproblems — I can memoize" before writing the optimized version; this is the DP-specific version of the general "state brute force, then optimize" habit from [[01 - Complexity Analysis (Big-O Thinking)]].
- A common off-by-one bug in tabulation is sizing the table `n` instead of `n + 1` (forgetting to allocate a slot for the base case at index 0), causing an index-out-of-range or silently wrong base case.
- Don't reach for full 2D DP tables when a 1D rolling-variable version suffices (e.g. Fibonacci-shaped recurrences) — interviewers often ask for the space-optimized version as an explicit follow-up, so mention it proactively.
- Recursion depth: for problems with large n (e.g. n > a few thousand), a bottom-up/iterative DP solution sidesteps recursion's call-stack limits entirely — worth mentioning if the interviewer asks about very large inputs.

## References
- [What is Memoization? A Complete Tutorial — GeeksforGeeks](https://www.geeksforgeeks.org/dsa/what-is-memoization-a-complete-tutorial/)
- [Recursion — Chapter 7, Invent With Python](https://inventwithpython.com/recursion/chapter7.html)

## Related
- Parent: [[00 - Algorithms & Data Structures (Overview)]]
- Siblings: [[01 - Complexity Analysis (Big-O Thinking)]], [[02 - Arrays, Hashing & Strings]], [[03 - Two Pointers, Sliding Window & Binary Search]], [[04 - Trees, Graphs & Traversal]]
