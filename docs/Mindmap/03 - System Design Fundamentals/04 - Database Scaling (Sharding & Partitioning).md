---
title: Database Scaling (Sharding & Partitioning)
mindmap_id: database-scaling-sharding-partitioning
node_type: topic
category: System Design Fundamentals
parent: "[[00 - System Design Fundamentals (Overview)]]"
tags: [coding-interview, system-design]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Database Scaling (Sharding & Partitioning)

> Partitioning splits a table within one database; sharding splits data across many databases — and the shard key you pick on day one is one of the hardest things to change later.

## Definition & Core Concepts

**Partitioning** is dividing a single database's data into smaller, more manageable segments (partitions) that still live within the same database instance. The database engine handles routing queries to the right partition transparently. Partitioning is primarily about *manageability and query performance within one machine*: a query that only needs last month's orders can skip scanning years of older partitions (partition pruning), maintenance operations (index rebuilds, backups) can run per-partition, and hot/cold data can be tiered onto different storage. Partitioning does **not**, by itself, let you exceed the capacity — CPU, RAM, disk, write throughput — of a single machine, because everything still runs on that one instance.

**Sharding** is the next step: splitting data across **multiple separate database instances (shards)**, each running on its own machine, typically with the same schema but a disjoint subset of rows. This is what actually breaks through a single machine's ceiling, because you're no longer bound by any one server's CPU, memory, disk, or write throughput — you add more shards to add more capacity. The cost is architectural complexity: queries that need data from more than one shard (a cross-shard join, a global count, a multi-user transaction) become expensive or require application-level coordination, since the database engine no longer sees the whole dataset from one place.

The decision of **which column(s) determine where a row lives — the shard key** — is the single most consequential choice in a sharded system, because resharding a live system with the wrong key is one of the hardest operational problems in distributed data systems (it means moving large volumes of data between machines while the system stays online and correct). Common sharding strategies:

- **Range-based sharding**: rows are assigned to shards based on ranges of the key (e.g., user IDs 1–1M on shard A, 1M–2M on shard B). Simple, and range queries stay efficient (a query for "users 100–200" hits one shard), but new data with a monotonically increasing key (like an auto-incrementing ID or a timestamp) concentrates all new writes onto whichever shard currently owns the "latest" range — a classic **hotspot**.
- **Hash-based sharding**: the shard key is hashed, and the hash determines the shard. This spreads writes evenly (a good hash function has no natural "latest" concentration point) but destroys range-query locality — "give me users 100–200" now has to fan out to every shard, since consecutive keys land on effectively random shards.
- **Directory-based sharding**: a separate lookup service maps each key (or key range) to its shard explicitly. This is the most flexible — shards can be rebalanced by just updating the directory — but the directory itself becomes a critical, must-be-highly-available component and an extra hop on every query.
- **Geographic/entity-based sharding**: rows are grouped by a real-world dimension (region, tenant/customer ID in a multi-tenant system) so that related data — everything for one customer, or one geographic region — lives together, which is what makes queries scoped to that entity efficient and avoids cross-shard joins for the most common access pattern.

**Hotspotting** — one shard receiving disproportionate load — is the central risk to design against, and it can happen even with a hash-based key if the *real-world* access pattern is skewed (e.g., one celebrity user's data drives far more reads/writes than a typical user's, regardless of how evenly the hash spreads keys on paper).

Sharding and **replication** solve different problems and are normally combined, not substitutes for each other: sharding scales capacity (spreading the total dataset and total write throughput across machines) and provides *fault isolation* (a shard going down only affects the subset of data/users on that shard, not the whole system); replication provides *fault tolerance* and read scale *within* each shard (each shard typically has its own replicas, so losing one machine doesn't lose that shard's data). A mature sharded architecture is really a grid: N shards, each replicated M times.

## Best Practices

- **Pick a shard key that avoids hotspots before you need to reshard — not after.** Resharding a live system is one of the hardest operational problems in distributed data engineering (moving data between machines while staying online and consistent); the cost of getting the shard key wrong is paid much later, at a much less convenient time, than the cost of thinking hard about it up front.
- **Exhaust cheaper scaling options before sharding.** Sharding is usually the last resort after vertical scaling, read replicas, and caching have been tried and are no longer sufficient — it's the most operationally complex option on the list, and reaching for it first is a common interview (and real-world) overreach.
- **Design around your query patterns, not just your write-distribution needs.** A shard key that spreads writes evenly but forces every common read query to fan out across all shards (scatter-gather) has just traded one bottleneck for another — evaluate both the write-hotspot risk and the read/query-locality cost together.
- **Keep frequently-joined data on the same shard when possible** (e.g., shard by customer/tenant ID so all of one customer's data is co-located) to avoid cross-shard joins, which are expensive or require application-level stitching.
- **Plan for resharding from the start, even if you don't need it yet** — e.g., using consistent hashing (see [[01 - Scalability & Load Balancing]]) or a directory-based approach so that adding shards later remaps a small fraction of data instead of nearly all of it.
- **Treat cross-shard transactions as an exception, not a norm.** If the application frequently needs atomicity across shards, that's a sign the sharding boundary doesn't match the natural transactional boundary of the data — reconsider the shard key or the scope of what needs to be transactional.

## Real-World Use Case

Illustrative scenario: a multi-tenant SaaS platform shards its primary database by `tenant_id` (an entity-based/hash-based hybrid: hash the tenant ID to assign it to one of N shards, but keep every row for that tenant on the same shard). This keeps the overwhelmingly common query pattern — "give me all data for tenant X" — entirely within one shard, avoids cross-shard joins for normal application queries, and provides natural fault isolation: a shard outage affects only the tenants assigned to it, not the whole platform. The tradeoff the team accepts explicitly: a single very large tenant (a hotspot) can outgrow its shard's capacity faster than smaller tenants, which they mitigate by allowing a large tenant to be manually migrated to its own dedicated shard when it crosses a size threshold.

## Hands-On Practice

**Design exercise: "The users table for a social app has grown to 500M rows and a single database can no longer handle the write throughput at peak. Design the sharding approach."**

1. **Confirm sharding is actually warranted first.** State explicitly that you'd check whether a bigger instance (vertical scaling), read replicas (if the bottleneck is reads, not writes), or a cache layer in front of hot reads would resolve this more cheaply — the prompt specifies *write* throughput at peak is the bottleneck, which read replicas and caching don't fix, so sharding is genuinely warranted here, not a default reach.
2. **Choose a candidate shard key: `user_id`.** It's the natural primary access pattern (nearly every query is scoped to a user) and it's present on essentially every row.
3. **Evaluate range vs hash for this key.** If `user_id`s are sequentially assigned (auto-increment or a time-ordered ID scheme), pure range-based sharding would concentrate all new user signups — and their write activity — onto whichever shard owns the newest range, a hotspot. Hash-based sharding on `user_id` spreads new users evenly across shards instead.
4. **Check the read-pattern cost of hashing.** Since the dominant query is "fetch this one user's data" (a point lookup by `user_id`), hashing doesn't hurt query locality here the way it would for a range-scan-heavy workload — this makes hash-based sharding a strong fit for this specific access pattern, not just a default choice.
5. **Address celebrity/whale hotspots explicitly.** Even with even hash distribution, a small number of high-follower users can generate disproportionate read/write load on their shard regardless of key design — call out that this needs a separate mitigation (e.g., extra caching or read replicas specifically for hot keys) rather than claiming the shard key alone solves it.
6. **Combine with replication.** Each shard gets its own leader-follower replica set (see [[03 - Data Consistency, CAP Theorem & Replication]]) so a single machine failure within a shard doesn't lose that shard's data or availability — sharding for write-scale and fault isolation, replication for durability and read-scale within each shard.

## Exam Tips

- Candidates very often reach for sharding before they've exhausted cheaper options — always say out loud that you considered vertical scaling, read replicas, and caching first, and explain specifically why they don't solve *this* bottleneck (usually: they don't fix write throughput, only reads or overall capacity headroom).
- Don't conflate partitioning and sharding — interviewers listen for whether you know partitioning stays within one instance while sharding spans multiple; using the terms interchangeably is a quick signal of shallow understanding.
- When you propose a shard key, always state the hotspot risk for that specific key and workload — a shard key proposed with no hotspot analysis is an incomplete answer, since hotspotting is the primary failure mode of sharding.
- Range vs hash isn't a "pick one forever" choice — be ready to explain the read-vs-write tradeoff (range preserves query locality but risks write hotspots on sequential keys; hash spreads writes but breaks range-query locality) for the specific access pattern in the prompt.
- Remember sharding and replication are complementary, not alternatives — a design that only sharded (no replication per shard) or only replicated (no sharding, so still capacity-bound to one machine's total dataset) is incomplete for a truly large-scale system.

## References
- [Difference Between Database Sharding and Partitioning — GeeksforGeeks](https://www.geeksforgeeks.org/system-design/difference-between-database-sharding-and-partitioning/)
- [Shard Keys — MongoDB Manual](https://www.mongodb.com/docs/manual/core/sharding-shard-key/)

## Related
- Parent: [[00 - System Design Fundamentals (Overview)]]
- Siblings: [[01 - Scalability & Load Balancing]], [[02 - Caching Strategies]], [[03 - Data Consistency, CAP Theorem & Replication]], [[05 - Asynchronous Messaging & Microservices Communication]]
