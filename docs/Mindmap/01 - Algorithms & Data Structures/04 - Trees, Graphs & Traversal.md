---
title: Trees, Graphs & Traversal
mindmap_id: trees-graphs-traversal
node_type: topic
category: Algorithms & Data Structures
parent: "[[00 - Algorithms & Data Structures (Overview)]]"
tags: [coding-interview, algorithms-data-structures]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Trees, Graphs & Traversal

> How to systematically visit every relevant node in a hierarchical or networked structure — the choice between BFS and DFS, backtracking as DFS-with-undo, and topological sort as the classic "ordering with dependencies" pattern.

## Definition & Core Concepts
A **tree** is a connected, acyclic graph with a designated root — every node (except the root) has exactly one parent, which is what makes recursive traversal so natural (each subtree is itself a smaller tree). A **graph** generalizes this: nodes (vertices) connected by edges, which may be directed or undirected, cyclic or acyclic, weighted or unweighted. Graphs are commonly represented as an **adjacency list** (a hash map or array from each node to a list of its neighbors — efficient for sparse graphs, O(V + E) space) or an **adjacency matrix** (a V×V grid where `matrix[i][j]` indicates an edge — O(V²) space, but O(1) edge-existence checks, better for dense graphs).

**BFS (Breadth-First Search)** explores level by level, using a **queue** (FIFO): visit all neighbors of the start node before moving to their neighbors, and so on outward in expanding "rings." Because it explores in order of distance from the start, BFS is the correct tool whenever you need the **shortest path in an unweighted graph** or a **level-order traversal** of a tree. Each node is enqueued once and dequeued once, giving O(V + E) time.

**DFS (Depth-First Search)** explores as deep as possible along one branch before backtracking, using a **stack** (explicit, or implicitly via recursion — the call stack *is* the stack). DFS is the right tool for **exhaustive exploration** — visiting every node/path regardless of distance, detecting cycles, computing connectivity, or exploring all possible states in a search/constraint problem (backtracking). Like BFS, DFS is O(V + E) time, but its space usage is O(h) (max recursion/stack depth) rather than O(width of the widest level) — this can matter for very wide, shallow graphs versus very deep, narrow ones.

**When to use which** — the decision isn't arbitrary:
- Need the *shortest* path, minimum number of steps/levels, or "closest" node satisfying a condition → **BFS**. Its level-by-level nature guarantees the first time you reach a target, you've reached it via the shortest unweighted path.
- Need to explore *all* paths, check if *any* path exists, detect cycles, compute connected components, or the problem is naturally recursive (subtree properties, path sums) → **DFS**.
- Need shortest path in a *weighted* graph → neither plain BFS nor DFS suffices; that's Dijkstra's algorithm (a weighted generalization of BFS using a priority queue instead of a plain queue) — worth knowing exists even if it's a level beyond core BFS/DFS.

**Backtracking** is DFS augmented with **explicit undo**: you make a choice (add it to a partial solution), recurse deeper, and when you return from that recursive call, you **undo the choice** before trying the next option at that level. This "try → recurse → undo" shape is what lets you explore an entire tree of possibilities (permutations, combinations, subsets, board configurations) without needing to copy the partial solution at every step — you mutate a single shared structure and restore it afterward. Pruning (returning early when a partial solution can't possibly succeed) is what keeps backtracking from being pure brute force.

**Topological sort** produces a linear ordering of nodes in a **directed acyclic graph (DAG)** such that for every directed edge `u → v`, `u` comes before `v` in the ordering. It's the standard tool for "what order must these dependent tasks run in" problems. Two common implementations: **Kahn's algorithm** (BFS-based: repeatedly remove nodes with in-degree 0, decrementing their neighbors' in-degrees) and **DFS-based** (do a DFS, and prepend each node to the result as its DFS call *finishes*, i.e. a reversed post-order). Kahn's algorithm has the added benefit of naturally detecting a cycle (if you can't remove all nodes, a cycle exists — this is exactly how "Course Schedule" style problems check feasibility).

## Best Practices
- **State your representation choice.** For sparse graphs (most interview problems), default to an adjacency list — it's more space-efficient and iterating neighbors is faster than scanning a matrix row.
- **Always track visited nodes** in graph BFS/DFS (unlike tree traversal, where the tree structure itself prevents revisiting) — otherwise cycles cause infinite loops. A hash set of visited node IDs is the standard tool; mark a node visited **when you enqueue/push it**, not when you dequeue/pop it, or you can enqueue the same node multiple times before it's ever processed (a subtle but common bug that bloats the queue without causing wrong answers, hiding the bug from small test cases).
- **Backtracking complexity is often exponential by nature of the problem** (generating all permutations is inherently O(n!)) — the goal of backtracking isn't to make it polynomial, it's to make the *constant factor* as small as possible via pruning, and to make sure you're not doing *additional* unnecessary work (like copying the whole partial-solution array at every recursive call instead of mutating and undoing in place).
- **The undo step is the most commonly forgotten line in backtracking code** — every "add to partial solution" or "mark visited" before a recursive call needs a matching "remove"/"unmark" after it returns, or subsequent branches see stale state.
- **Topological sort only exists for DAGs.** If Kahn's algorithm finishes without processing all nodes, the graph has a cycle and no valid ordering exists — this is the mechanism, not a side note, for solving "Course Schedule" (can all courses be finished?).
- **BFS shortest-path claim only holds for unweighted graphs** (or graphs where every edge has equal weight) — using plain BFS on a weighted graph and assuming the first-reached path is shortest is a common and serious mistake.

## Real-World Use Case
Case study: BFS is the standard algorithm taught for computing shortest paths in unweighted graphs, and its level-order exploration underlies real applications like finding the shortest connection chain in a social network or the fewest moves to solve a puzzle state. [ref: GeeksforGeeks BFS article] Topological sort's dependency-ordering is directly how real build systems and package managers work: illustrative scenario — a build tool resolving compile order for modules with `depends_on` relationships is solving exactly the topological-sort problem, and a circular dependency in that system is detected the same way "Course Schedule" detects an infeasible course plan (Kahn's algorithm fails to process all nodes).

## Hands-On Practice
Three canonical problem types, one walked through in depth:
1. **Binary Tree Level Order Traversal** — BFS with a queue, walked through: setup — push the root into a queue. Approach — while the queue isn't empty, record the current queue size (this is exactly one "level"), pop that many nodes, collect their values, and push each of their children; repeat. Complexity of result: O(n) time (every node enqueued/dequeued once), O(w) space where w is the widest level (worst case O(n) for a wide tree).
2. **Number of Islands** — DFS (or BFS) flood-fill from each unvisited land cell, marking connected land as visited, counting how many separate flood-fills are needed. O(rows × cols) time and space.
3. **Course Schedule** (topological sort / cycle detection) — build an adjacency list from prerequisite pairs, compute in-degree for each course, run Kahn's algorithm (BFS from all in-degree-0 nodes), and check whether the number of processed nodes equals the total number of courses; if not, a cycle exists and the schedule is infeasible. O(V + E) time.

## Exam Tips
- If the problem says "shortest," "minimum number of steps," or "fewest," think BFS first — reaching for DFS here is a very common miss that still "works" on small test cases (finds *a* path) but is wrong or needlessly complex for guaranteeing the *shortest* one.
- If the problem says "all possible," "every combination/permutation," or "does there exist a valid arrangement," think backtracking (DFS + undo), and be ready to state the exponential complexity honestly rather than pretending it's polynomial.
- A classic recursion-depth gotcha: DFS on a very deep/skewed tree or graph (e.g. a linked-list-shaped tree) can blow the call stack in languages with limited recursion depth (Python's default ~1000) — mention iterative DFS with an explicit stack as a fallback if asked about very large inputs.
- Don't forget the "mark visited on push, not pop" rule for graph BFS — interviewers sometimes probe this exact edge case with a small graph that has multiple paths to the same node.
- For backtracking, always state the branching factor and depth to justify your complexity claim (e.g. "generating subsets is O(2ⁿ) because each of n elements is independently included or excluded") — this shows the complexity isn't guesswork.

## References
- [Breadth First Search or BFS for a Graph — GeeksforGeeks](https://www.geeksforgeeks.org/dsa/breadth-first-search-or-bfs-for-a-graph/)
- [When to use DFS or BFS to solve a Graph Problem — GeeksforGeeks](https://www.geeksforgeeks.org/dsa/when-to-use-dfs-or-bfs-to-solve-a-graph-problem/)
- [Print the DFS Traversal step-wise (Backtracking also) — GeeksforGeeks](https://www.geeksforgeeks.org/dsa/print-the-dfs-traversal-step-wise-backtracking-also/)

## Related
- Parent: [[00 - Algorithms & Data Structures (Overview)]]
- Siblings: [[01 - Complexity Analysis (Big-O Thinking)]], [[02 - Arrays, Hashing & Strings]], [[03 - Two Pointers, Sliding Window & Binary Search]], [[05 - Dynamic Programming & Recursion]]
