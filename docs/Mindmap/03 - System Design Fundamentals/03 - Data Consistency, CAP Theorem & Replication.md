---
title: Data Consistency, CAP Theorem & Replication
mindmap_id: data-consistency-cap-theorem-replication
node_type: topic
category: System Design Fundamentals
parent: "[[00 - System Design Fundamentals (Overview)]]"
tags: [coding-interview, system-design]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Data Consistency, CAP Theorem & Replication

> Once data lives on more than one machine, you can't have perfect correctness and perfect uptime at the same time during a network failure — CAP theorem forces you to say, in advance, which one you give up.

## Definition & Core Concepts

The **CAP theorem** states that a distributed data store can only guarantee two of the following three properties at once: **Consistency** (every read receives the most recent write or an error — all nodes see the same data at the same time), **Availability** (every request receives a non-error response, without the guarantee that it contains the most recent write), and **Partition tolerance** (the system continues operating despite network partitions — messages between nodes being lost or delayed). The crucial insight most candidates miss: in any real distributed system, network partitions **will** happen — hardware fails, cables get cut, data centers lose connectivity to each other. Partition tolerance therefore isn't really an optional third choice; it's a fact of distributed systems life. The real decision CAP theorem forces is: **when a partition occurs, do you choose Consistency (reject/delay requests to nodes that might be out of sync) or Availability (answer anyway, possibly with stale data)?** This is why CAP is more usefully read as "C vs A, under P" rather than "pick any two."

This maps directly onto two consistency models:

- **Strong consistency**: every read reflects the latest completed write, no matter which replica serves it. This requires coordination between nodes on every write (or every read) to guarantee agreement, which adds latency and, under a partition, means some nodes must refuse to serve requests rather than risk returning stale data. Systems that choose this are "CP."
- **Eventual consistency**: replicas may temporarily disagree, but if no new writes occur, all replicas will *eventually* converge to the same value. This allows every node to keep answering requests even when it can't confirm it has the latest data, trading a temporary correctness gap for continuous availability. Systems that choose this are "AP."

Neither is "better" in the abstract — the right choice is a function of what the data represents. A bank account balance generally needs strong consistency (double-spending is unacceptable); a social media "like" count or a product view counter can tolerate eventual consistency (a few seconds of staleness is invisible to the user and the cost of coordinating every replica on every increment would be enormous for no real benefit).

**Replication** is the mechanism that makes any of this possible — keeping copies of the same data on multiple nodes, for both fault tolerance (a node dying doesn't lose data) and scale (reads can be spread across replicas). The dominant model is **leader-follower (primary-replica) replication**: one node (the leader) accepts all writes and then propagates them to one or more follower nodes, which serve reads. This is simple to reason about (writes have one, unambiguous, order — whatever order they hit the leader in) but the leader is a single point of write availability, and replication to followers can be **synchronous** (the leader waits for followers to confirm before acknowledging the write — stronger consistency, higher write latency, and the leader is blocked if a follower is slow or down) or **asynchronous** (the leader acknowledges immediately and replicates in the background — lower latency, but a leader crash before replication completes can lose the most recent writes, and followers may briefly serve stale reads). Some systems instead use **multi-leader** or **leaderless** replication to avoid a single write bottleneck, at the cost of needing conflict resolution when concurrent writes to the same data disagree — a meaningfully harder problem that most interview answers don't need to go into, but should be able to name.

## Best Practices

- **Choose CP or AP based on what the data represents, not based on which sounds more rigorous.** Strong consistency is not a universal "safer" default — it costs latency and availability everywhere, all the time, to protect against a partition scenario that may be rare, so applying it to data that doesn't need it is over-engineering.
- **Default to eventual consistency for anything the user won't notice being briefly stale**, and reserve strong consistency for the specific data paths where staleness causes a real, user-visible or business-critical problem (payments, inventory counts near zero, auth/session state).
- **Use synchronous replication only for the specific writes that truly require zero data loss on leader failure**; broad synchronous replication multiplies write latency by the slowest follower, everywhere, for every write.
- **Design for leader failover explicitly.** A leader-follower system needs a plan for what happens when the leader dies — automatic promotion of a follower, and a way to avoid "split brain" (two nodes both believing they're the leader) during the transition.
- **State your consistency guarantee, don't leave it implicit.** A common production bug is a service assuming strong consistency from a datastore that's documented as eventually consistent (or vice versa) — the guarantee should be an explicit, written contract, not folklore.

## Real-World Use Case

Illustrative scenario: an e-commerce platform uses strong consistency (a CP-leaning path, via synchronous writes to the primary before acknowledging) for the inventory decrement that happens at checkout — because two customers concurrently "winning" the last unit of stock is a real business problem (overselling), not a cosmetic one. The same platform uses eventual consistency (an AP-leaning path, via asynchronous replication) for the "customers also viewed" recommendation counters and review star-rating aggregates — a few seconds of staleness there is invisible to the shopper and not worth the availability and latency cost of coordinating every replica on every view event.

## Hands-On Practice

**Design exercise: "Design the data layer for a collaborative document editor (like Google Docs) — what consistency guarantees does each piece of data need?"**

1. **Break the system into distinct data types before picking one consistency model for everything.** A collaborative editor isn't one data problem — it has document content (frequently, concurrently written), the list of active collaborators/presence indicators, and access-control metadata (who can edit).
2. **Document content**: this needs strong, ordered consistency for *conflict resolution correctness* — concurrent edits must be merged deterministically (this is typically solved with operational transformation or CRDTs rather than plain leader-follower replication alone, but the underlying storage layer still benefits from a clear write-ordering guarantee so merges are deterministic across replicas).
3. **Presence/"who's currently viewing"**: this is a strong candidate for eventual consistency and even availability-over-correctness — if a collaborator's cursor position is a second stale, or their "online" status briefly lags, nobody notices, and this data changes so frequently that paying a strong-consistency cost for it would be wasted overhead.
4. **Access control (who can edit this doc)**: this needs strong consistency — a permission revocation that takes 10 seconds to propagate to all replicas means a removed collaborator could still write to the document during that window, which is a security issue, not just a UX inconvenience.
5. **State the replication model for the strongly-consistent paths.** Leader-follower with synchronous replication (or a consensus protocol like Raft/Paxos for the access-control store specifically) so a permission change is confirmed durable and visible before the write is acknowledged.
6. **Name the tradeoff out loud.** Applying strong consistency uniformly across all three data types would be simpler to reason about but would make presence updates unnecessarily slow and would hurt availability during any partition — the point of the exercise is showing you'd choose per data type, not per system.

## Exam Tips

- The single most common CAP theorem mistake: treating it as "pick 2 of 3" in general, when in practice partition tolerance is non-negotiable for any real distributed system — the actual decision is C vs A *during* a partition. Say this explicitly; it signals you understand the theorem rather than having memorized the acronym.
- Don't apply CAP theorem to a system that has no meaningful partition risk (e.g., a single-node database with no replicas) — it's frequently misapplied to systems where it simply doesn't govern the relevant tradeoff, and naming when it *doesn't* apply is itself a sign of understanding.
- When asked "would you use SQL or NoSQL," resist answering in the abstract — connect it back to which consistency model each specific piece of data in the system actually needs, since that's usually the real question underneath.
- Be ready to distinguish "the CAP theorem's C" (linearizability / every read sees the latest write) from the "C" in ACID (transactional consistency / constraints are upheld) — these are different concepts that share a letter and are a classic terminology trap.
- If you propose eventual consistency, always state the bound (how long until convergence, or under what conditions) — "eventually" without a bound is not an engineering answer.

## References
- [CAP Theorem vs BASE Consistency Model in Distributed System — GeeksforGeeks](https://www.geeksforgeeks.org/system-design/cap-theorem-vs-base-consistency-model-distributed-system/)
- [CAP Theorem — MongoDB](https://www.mongodb.com/resources/basics/databases/cap-theorem)

## Related
- Parent: [[00 - System Design Fundamentals (Overview)]]
- Siblings: [[01 - Scalability & Load Balancing]], [[02 - Caching Strategies]], [[04 - Database Scaling (Sharding & Partitioning)]], [[05 - Asynchronous Messaging & Microservices Communication]]
