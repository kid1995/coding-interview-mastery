---
title: Coding Interview Mastery Mind Map (Index)
mindmap_id: coding-interview-mastery
node_type: root
tags: [coding-interview, moc]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
---

# Coding Interview Mastery — Mind Map Index

> A Map of Content (MOC) covering the theory needed to solve: Elevator-Saga-style simulation and
> scheduling problems, LeetCode-style algorithm exercises, system design interview questions,
> software resilience, and how to write easily-testable code. Built with the `study-mindmap-vault`
> skill — no pre-existing syllabus, the tree below was designed from scratch across these 5 areas.

## How to use this vault

- Same tree-mirroring shape as this skill's other vaults (see `EXAMS/GCP/Mindmap/`): this index is
  the root, each category below is a folder, each folder has an `00 - <Category> (Overview).md`
  plus one note per leaf topic.
- Every note links to its parent and siblings via wikilinks — navigate with Obsidian's graph view
  or backlinks the same way you'd navigate a mind map.
- "Real-World Use Case" sections are labeled either **Case study:** (verified, cited, real
  published example) or **Illustrative scenario:** (a realistic but not independently verified
  pattern) — never presented as fact without that label.
- Practice, don't just read: [play.elevatorsaga.com](https://play.elevatorsaga.com/) for the
  simulation/scheduling category, [leetcode.com](https://leetcode.com/) for the algorithms
  category, and mock system-design interviews for that category — this vault is the theory that
  makes the practice click, not a substitute for it.

## Categories

1. [[00 - Algorithms & Data Structures (Overview)|Algorithms & Data Structures]] — Complexity analysis, arrays/hashing/strings, two pointers/sliding window/binary search, trees/graphs/traversal, dynamic programming & recursion. The LeetCode toolbox.
2. [[00 - Simulation & Scheduling (Overview)|Simulation & Scheduling]] — State machines, event-driven simulation, scheduling algorithms (SCAN/LOOK/greedy dispatch), queueing theory. The Elevator Saga toolbox.
3. [[00 - System Design Fundamentals (Overview)|System Design Fundamentals]] — Scalability & load balancing, caching, CAP theorem & consistency, database sharding, async messaging. The "design X" interview toolbox.
4. [[00 - Software Resilience (Overview)|Software Resilience]] — Retry/backoff/circuit breaker, bulkheads/timeouts/graceful degradation, idempotency, observability & chaos engineering. How to survive real-world failure.
5. [[00 - Testability & Clean Code (Overview)|Testability & Clean Code]] — SOLID & dependency injection, test doubles & mocking, TDD workflow, hexagonal/clean architecture. How to write code that's easy to test by design.

## Cross-references

- `DEV-BRAIN/03-Architecture/System-Design/01 - CAP theorem and consistency models.md` already
  exists in this user's DEV-BRAIN vault — this vault's own
  [[03 - Data Consistency, CAP Theorem & Replication|CAP Theorem note]]
  is scoped to interview-practice depth; read the DEV-BRAIN note too for the deeper
  architecture-focused treatment, since they cover the same theorem from different angles rather
  than duplicating each other.
- Companion git repo: `coding-interview-mastery` (docs-only snapshot of this vault, see its README).
