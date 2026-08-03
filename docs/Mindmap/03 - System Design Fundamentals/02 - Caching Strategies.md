---
title: Caching Strategies
mindmap_id: caching-strategies
node_type: topic
category: System Design Fundamentals
parent: "[[00 - System Design Fundamentals (Overview)]]"
tags: [coding-interview, system-design]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Caching Strategies

> A cache trades storage and staleness for speed — the strategy you pick determines exactly how much staleness you're accepting, and where.

## Definition & Core Concepts

A cache stores a copy of data that's expensive to compute or fetch, closer to where it's needed, so repeated requests for the same data don't pay that cost again. The interesting design decisions aren't "should I add a cache" — they're **how does data get into the cache, and how does the cache stay in sync with the source of truth**. Four canonical strategies answer that:

- **Cache-aside (lazy loading)**: the application checks the cache first; on a miss, it reads from the database, then writes the result into the cache for next time. The cache only ever contains data that's actually been requested, which keeps it small and relevant, but every cache miss pays the full read latency plus a write to the cache. Writes to the database, importantly, do **not** automatically update the cache — the application must invalidate or update the cached entry itself, or it will serve stale data until it expires. This is the most common pattern because the cache and the database can be different technologies with no coupling between them.
- **Read-through**: conceptually the same flow as cache-aside (check cache, fall back to source on miss), but the cache itself owns the logic for loading from the source — the application only ever talks to the cache, never to the database directly for reads. This centralizes the loading logic but requires a caching layer that supports it (many managed caching services do).
- **Write-through**: every write goes to the cache *and* the database synchronously, as a single logical operation, before the write is considered complete. This keeps the cache always consistent with the database, at the cost of added write latency (every write now pays two round trips) — and it caches data that may never be read, since writes populate the cache regardless of read demand.
- **Write-behind (write-back)**: the write goes to the cache first and is acknowledged immediately; the cache asynchronously flushes the change to the database later, often batched. This gives the lowest write latency and can absorb write bursts, but introduces a window where the database is stale relative to the cache, and a cache failure before the flush completes can mean **data loss** — this is the riskiest pattern and needs a durable buffer (e.g., a write-ahead log) to be safe.

**Cache invalidation is the hard part** — famously one of the "two hard things in computer science." The core problem: the cache and the source of truth are two copies of the same fact, and every write to one is an opportunity for them to disagree. Common invalidation approaches: a **TTL (time-to-live)** that expires entries automatically (simple, bounds staleness, but doesn't guarantee freshness within the TTL window); **explicit invalidation** on write (the application or database triggers a delete/update of the cache entry when the source changes — more accurate, but requires every write path to remember to do it); and **event-driven invalidation** (a change-data-capture stream or message from the database triggers cache updates, decoupling invalidation from the write path itself).

**Eviction policies** decide what gets removed when the cache is full (a separate problem from invalidation, which is about *correctness*; eviction is about *capacity*). The dominant policy is **LRU (Least Recently Used)** — evict whichever entry hasn't been accessed in the longest time, on the assumption that recently-used data is likely to be used again soon. Variants exist for specific access patterns: **LFU (Least Frequently Used)** for data with a stable "hot set" that isn't well captured by recency alone, and **FIFO** for simple cases where recency/frequency don't matter.

Caches exist at multiple layers of a system, and a strong answer names which layer solves which problem: **client-side** (browser cache, mobile app cache — avoids the network entirely), **CDN** (caches static or semi-static content geographically close to the user — solves latency from network distance), **application-layer** (an in-process or distributed cache like Redis/Memcached in front of the database — solves database load), and **database-layer** (the database's own internal buffer pool/query cache — usually not something you configure directly, but it's why "just add an index" sometimes isn't enough if the working set doesn't fit in memory).

## Best Practices

- **Always set a TTL and decide on an invalidation strategy before adding a cache — not after.** A cache with no expiration and no invalidation plan is a bug waiting to happen: it will eventually serve data that's provably wrong, and by the time that's noticed in production, it's usually hard to tell how long it's been wrong.
- **Match the strategy to the read/write ratio.** Cache-aside fits read-heavy, unpredictable-key workloads (most web apps). Write-through fits workloads where read-after-write consistency matters and write volume is manageable. Write-behind fits write-heavy workloads that can tolerate eventual durability (e.g., analytics counters) — never use write-behind for data where losing the last few writes is unacceptable (e.g., financial transactions).
- **Guard against cache stampede / thundering herd**: when a hot key expires, many concurrent requests can all miss simultaneously and hammer the database at once. Mitigate with request coalescing (only one request repopulates the cache, others wait), staggered TTLs (jitter to avoid synchronized expiry), or serving stale-while-revalidate.
- **Cache the right granularity.** Caching a whole expensive query result is simple but invalidates as one unit; caching smaller, composable pieces (e.g., per-object rather than per-page) invalidates more precisely but adds complexity — pick based on how often the underlying pieces change independently.
- **Treat the cache as disposable.** The system must remain correct (if slower) if the cache is completely empty or unavailable — never let the cache become a hidden source of truth that the database doesn't have.
- **Monitor hit rate, not just latency.** A low hit rate means the cache isn't earning its complexity and operational cost; it's a signal to revisit TTL, key granularity, or whether this data is actually cacheable.

## Real-World Use Case

Illustrative scenario: a social media product page shows a user's profile, including a follower count that changes frequently but doesn't need to be exact to the second. The team uses cache-aside with a short TTL (e.g., 30–60 seconds) for the follower count specifically, accepting brief staleness in exchange for removing near-constant read load from the database, while the profile's rarely-changing fields (bio, join date) are cached with explicit invalidation on write since they change infrequently and users notice staleness there more. Static assets (profile photos) are served from a CDN entirely outside the application cache layer, since they never change once uploaded and benefit most from being geographically close to the requester.

## Hands-On Practice

**Design exercise: "Design the caching layer for a product detail page that gets 10,000 reads/sec and the underlying product data (price, description, stock count) updates about once a minute."**

1. **Characterize the workload first.** Extremely read-heavy (10K reads/sec) against infrequent writes (~once/minute) — this ratio strongly favors cache-aside over write-through, since write-through would add latency to writes that don't need it for a workload dominated by reads.
2. **Pick the strategy: cache-aside.** On a cache miss, load from the database and populate the cache; subsequent reads for up to a minute are served from cache without touching the database at all — this alone likely removes 99%+ of read load from the database.
3. **Set the TTL relative to the update frequency and business tolerance.** Since the source updates roughly once a minute, a TTL of 30–60 seconds bounds staleness to about one update cycle; if stock count going stale by even a few seconds is unacceptable (e.g., overselling risk), explicit invalidation on write is added *in addition to* the TTL as a safety net, not instead of it.
4. **Address stampede risk explicitly.** At 10K reads/sec, a naive expiry means potentially thousands of simultaneous requests hitting the database the instant a hot product's cache entry expires — mitigate with request coalescing (a lock or "in-flight" marker so only one request repopulates) or a small jitter on the TTL so not all keys expire in lockstep.
5. **Decide on granularity.** Cache the product data as one object keyed by product ID (not the whole rendered page), so unrelated changes elsewhere on the page don't force a cache miss on data that hasn't changed.
6. **Name the failure mode out loud.** If the cache layer goes down entirely, the system should degrade to reading straight from the database (slower, but correct) rather than fail outright — this is the kind of resilience statement interviewers listen for.

## Exam Tips

- When asked "how would you cache this," always ask (or state your assumption) about the **read/write ratio** and the **staleness tolerance** first — the right strategy is a direct function of those two answers, not a fixed default.
- Cache invalidation is usually the actual crux of a caching question — don't stop at "we'd cache this with a TTL" without addressing what happens on write, since that's where interviewers probe next.
- Don't confuse eviction (LRU, capacity-driven) with invalidation (correctness-driven) — they solve different problems and candidates frequently conflate them.
- Write-behind is a common wrong answer when candidates want "the fastest option" without registering the durability risk — flag the tradeoff explicitly if you propose it.
- Remember caching exists at multiple layers (client, CDN, application, database) — a mature answer places the right kind of data at the right layer instead of assuming "cache" always means "Redis in front of the database."

## References
- [Caching — Hello Interview (System Design Core Concepts)](https://www.hellointerview.com/learn/system-design/core-concepts/caching)
- [Caching Strategies — system-design.space](https://system-design.space/en/chapter/caching-strategies/)

## Related
- Parent: [[00 - System Design Fundamentals (Overview)]]
- Siblings: [[01 - Scalability & Load Balancing]], [[03 - Data Consistency, CAP Theorem & Replication]], [[04 - Database Scaling (Sharding & Partitioning)]], [[05 - Asynchronous Messaging & Microservices Communication]]
