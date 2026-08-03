---
title: Algorithms & Data Structures
mindmap_id: algorithms-data-structures
node_type: category
parent: "[[00 - Coding Interview Mastery Mind Map (Index)]]"
children: ["[[01 - Complexity Analysis (Big-O Thinking)]]", "[[02 - Arrays, Hashing & Strings]]", "[[03 - Two Pointers, Sliding Window & Binary Search]]", "[[04 - Trees, Graphs & Traversal]]", "[[05 - Dynamic Programming & Recursion]]"]
tags: [coding-interview, category]
source: "coding-interview-mastery vault"
created: 2026-08-03
---

# Algorithms & Data Structures

> The theoretical foundation for solving LeetCode-style algorithm and data-structure interview problems.

## Overview
Algorithm interviews are not really testing whether you remember a specific solution — they're testing whether you can recognize *which family of technique* a new, never-before-seen problem belongs to, and whether you can reason precisely about the time and space cost of your approach. The problems themselves are infinite in surface variety (strings, grids, trees, intervals, graphs...) but the underlying toolbox is small and stable. This category is that toolbox.

The topics below are deliberately ordered from "lens" to "technique families." Complexity analysis isn't a technique you apply to a problem — it's the vocabulary you use to *evaluate* every other technique, so it comes first. After that, the remaining four notes cover the actual pattern families interviewers draw from: narrowing a search space (two pointers / sliding window / binary search), traversing a structure (arrays/hashing/strings as the simplest structure, then trees/graphs as the recursive ones), and avoiding recomputation (dynamic programming, built on recursion fundamentals).

Most "hard" interview problems are not a single technique in isolation — they're two of these combined (e.g. binary search on the answer + a greedy/DP feasibility check, or DFS + backtracking + memoization). Once you can name the individual techniques fluently, spotting the combinations becomes much easier. The goal of this category is fluency in naming: given an unfamiliar problem statement, you should be able to say "this smells like sliding window because we want a contiguous subarray satisfying a constraint" before you've written a single line of code.

## Topics in This Category
- [[01 - Complexity Analysis (Big-O Thinking)]] — the vocabulary (Big-O/Θ/Ω) and habits for reasoning about time and space cost of any algorithm.
- [[02 - Arrays, Hashing & Strings]] — the foundational data layouts, and how a hash map converts brute-force O(n²) "have I seen this before" checks into O(n).
- [[03 - Two Pointers, Sliding Window & Binary Search]] — techniques that narrow a search space instead of exhaustively scanning it.
- [[04 - Trees, Graphs & Traversal]] — BFS vs DFS, backtracking, and topological sort for exploring hierarchical and networked structures.
- [[05 - Dynamic Programming & Recursion]] — recognizing optimal substructure and overlapping subproblems, and avoiding recomputation via memoization or tabulation.

## How These Topics Fit Together
Think of solving an unseen problem as a funnel. First you use **complexity analysis** to state the brute-force solution's cost out loud (this is often expected in interviews, even before you optimize) — this also tells you what target complexity you're aiming for. Then you ask: is the input a flat sequence I can scan once with extra memory? That's **arrays & hashing** territory. Is it a sequence where the answer lives in a *contiguous* or *sorted* sub-region, so I can avoid re-scanning by moving pointers intelligently? That's **two pointers / sliding window / binary search**. Is the input inherently hierarchical or networked (a tree, grid, or graph) where I need to *visit* elements according to some connectivity rule? That's **trees & graphs**. And whenever the same sub-problem would otherwise be solved repeatedly — whether inside a recursive tree exploration or a linear scan — you reach for **dynamic programming** to cache and reuse that work.

These aren't mutually exclusive boxes. A classic "hard" problem like "sliding window maximum" combines sliding window with a monotonic deque; "word break" combines recursion with DP memoization; "course schedule" combines graph traversal with topological sort. Fluency in the individual families is what lets you recognize and assemble the combination under interview time pressure.

## References
- [Big O Cheat Sheet – Time Complexity Chart](https://www.freecodecamp.org/news/big-o-cheat-sheet-time-complexity-chart/)
- [15 LeetCode Patterns](https://blog.algomaster.io/p/15-leetcode-patterns)
