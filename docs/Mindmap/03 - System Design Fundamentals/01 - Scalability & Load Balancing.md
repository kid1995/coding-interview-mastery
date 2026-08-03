---
title: Scalability & Load Balancing
mindmap_id: scalability-load-balancing
node_type: topic
category: System Design Fundamentals
parent: "[[00 - System Design Fundamentals (Overview)]]"
tags: [coding-interview, system-design]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Scalability & Load Balancing

> Scaling is how you add capacity; load balancing is how you make sure that capacity actually gets used.

## Definition & Core Concepts

**Scalability** is a system's ability to handle growing load — more users, more data, more requests per second — by adding resources. There are two fundamentally different ways to add resources, and the choice between them shapes almost every later architecture decision:

- **Vertical scaling (scaling up)**: make the existing machine bigger — more CPU, more RAM, faster disks. It's the simplest option because the application doesn't change at all; a single, more powerful box just replaces the old one. But it has a hard ceiling (there's a biggest machine you can buy), it's a single point of failure (one box, one crash, total outage), and cost grows faster than linearly as you approach the top of the hardware curve — the biggest instances cost disproportionately more per unit of capacity.
- **Horizontal scaling (scaling out)**: add *more* machines and spread the load across them. This has no hard ceiling — in principle you keep adding boxes — and it naturally gives you redundancy, since losing one of ten machines is a 10% capacity hit, not a total outage. The cost is architectural: the application must now be able to run correctly as multiple, independent processes. That's only possible if those processes are **stateless** — no request should depend on data that only lives in one instance's memory or local disk, because the next request from the same user might land on a different machine entirely. Session data, uploaded files, and any in-memory cache must move to a shared, external store (a database, a distributed cache like Redis, or object storage) before horizontal scaling is safe.

Once you have multiple machines, something has to decide which machine handles each incoming request — that's a **load balancer (LB)**. A load balancer sits in front of a pool of servers, accepts incoming connections, and routes each one to a backend instance using some algorithm:

- **Round robin**: cycle through the server list in order, one request each. Simple, but ignores whether a server is already overloaded or handling a slow request — a slow server still gets its "fair share" of new requests.
- **Least connections**: send the new request to whichever server currently has the fewest active connections. This adapts to servers that are running slow (their connection count backs up), unlike plain round robin.
- **Weighted round robin / weighted least connections**: same idea, but servers are given weights so a more powerful machine gets proportionally more traffic — useful in a heterogeneous fleet (mixed instance sizes) or during a gradual rollout where new nodes should get less traffic until proven healthy.
- **Consistent hashing**: route based on a hash of some request attribute (e.g., a user ID or cache key) so the same key consistently lands on the same backend. This matters enormously when backends hold local state that benefits from being hit repeatedly by the same key — sharded caches and sharded databases both rely on consistent hashing so that adding or removing one node only remaps a small fraction of keys, instead of reshuffling everything (as plain modulo-hashing would).

Load balancers exist at more than one layer of a system, and interview candidates often only think of the outermost one:

- **Client-facing (edge) load balancing**: the first hop from the internet into your infrastructure — e.g., a Layer 7 (HTTP-aware) load balancer that can route based on URL path, headers, or cookies, or a Layer 4 (TCP/UDP) load balancer that only sees IP and port. DNS-based load balancing (returning different IPs to different clients) is also common at this layer, though it's coarser and slower to react than a dedicated LB because of DNS caching.
- **Service-to-service (internal) load balancing**: once inside the system, one microservice calling another still needs to pick which instance of that downstream service to call. This is often handled client-side (the caller has a list of healthy instances and picks one, common in service-mesh architectures) rather than through a single central LB, to avoid adding an extra network hop for every internal call.

Underpinning all of this is the **health check**: the load balancer periodically pings each backend (an HTTP endpoint, a TCP handshake, or a custom probe) and stops routing traffic to any instance that fails to respond correctly. Without health checks, a load balancer keeps sending live traffic into a crashed or degraded instance, turning a partial failure into a customer-visible one.

## Best Practices

- **Design for statelessness from day one**, even before you need to scale horizontally. Retrofitting a stateful application to be stateless under deadline pressure is far harder than building it stateless from the start.
- **Never run a single load balancer instance** — the load balancer itself is a single point of failure unless it's deployed redundantly (an active-active or active-passive pair, or a managed/anycast LB service that's inherently distributed).
- **Use health checks that reflect real readiness, not just process liveness.** A process that's running but can't reach its database should fail its health check — a check that only confirms "the process didn't crash" will happily route traffic into a broken instance.
- **Prefer least-connections or consistent hashing over plain round robin** once request processing times vary meaningfully between requests — round robin's blindness to server load becomes a real problem under uneven traffic.
- **Use consistent hashing whenever backends are not interchangeable** — i.e., whenever a backend holds local cache or shard state tied to specific keys. Plain hashing (`hash(key) % N`) remaps almost every key when `N` changes; consistent hashing remaps roughly `1/N` of keys, which is the difference between a smooth scale-out and a cache stampede.
- **Support connection draining during deploys and scale-down events** — when an instance is about to be removed, stop sending it *new* requests but let its in-flight requests finish, instead of dropping them.
- **Be deliberate about session affinity (sticky sessions).** It solves state-locality problems cheaply in the short term, but it undermines the even-distribution benefit of load balancing and creates uneven load if some "sticky" users are far heavier than others. Prefer externalizing session state over relying on affinity.

## Real-World Use Case

Illustrative scenario: an e-commerce checkout service starts on a single application server backed by a single database. As Black-Friday-style traffic grows, the team first scales the app server vertically (bigger instance) because it requires no code changes — this buys time but hits a ceiling and remains a single point of failure. They then move to horizontal scaling: multiple stateless application instances behind a Layer 7 load balancer, with session state moved out of local memory into a shared Redis store so any instance can serve any user's request. The load balancer uses least-connections routing (checkout requests vary widely in processing time) and active health checks against a `/health` endpoint that verifies both the app process and its database connection. During deploys, connection draining ensures in-flight checkouts complete on the old instance before it's terminated.

## Hands-On Practice

**Design exercise: "Design the traffic-handling layer for a read-heavy news website that just went from 10K to 2M daily users overnight after a viral story."**

1. **Identify the bottleneck first.** A sudden 200x traffic spike on a single web server will exhaust CPU/connections long before the database does for a read-heavy site — so the first fix is capacity, not caching or sharding yet.
2. **Vertical vs horizontal, and why horizontal wins here.** A bigger single server buys some headroom but caps out and stays a single point of failure during a viral spike that could get worse — horizontal scaling behind a load balancer is the right call, and it also gives redundancy during a high-visibility traffic event.
3. **Make the app stateless before scaling out.** Confirm no server holds per-user session or cached article state locally; move anything like that to a shared cache/session store first, otherwise adding instances will produce inconsistent behavior per user.
4. **Choose the LB layer and algorithm.** A client-facing Layer 7 LB in front of the web tier, using least-connections (article rendering times vary by content) with active health checks so a slow or crashed instance is pulled out of rotation automatically.
5. **Plan for autoscaling, not a fixed fleet size.** Since the spike is unpredictable in magnitude and duration, the instance pool should scale out based on a metric like CPU or request latency, and the LB's health checks are what make newly-added instances safe to receive traffic immediately.
6. **State what you're deferring and why.** Caching (topic 02) and read replicas would be the *next* layer to reach for once database read load — not app-tier capacity — becomes the bottleneck; naming this explicitly shows the interviewer you're reasoning about bottlenecks in order, not reciting every technique at once.

## Exam Tips

- Don't jump straight to "add a load balancer" without first asking whether the app tier is even stateless — if it isn't, that's the actual first problem to solve, and naming it shows deeper understanding.
- When asked to pick a load balancing algorithm, justify it against the actual traffic pattern (uniform request cost → round robin is fine; variable request cost → least connections; need for key-locality → consistent hashing) rather than picking one by default.
- Candidates often forget health checks entirely and only discuss the routing algorithm — always mention how the LB knows a backend is unhealthy, since that's frequently the actual cause of a described outage in the prompt.
- Don't present vertical scaling as "wrong" — it's the correct, low-effort first step in many real systems. The trap is not knowing *when* to graduate from it (hardware ceiling, cost curve, single point of failure) to horizontal scaling.
- Remember load balancing isn't only "internet to server" — service-to-service load balancing inside a microservices architecture is a distinct and frequently-tested sub-topic.

## References
- [System Design: Horizontal and Vertical Scaling — GeeksforGeeks](https://www.geeksforgeeks.org/system-design/system-design-horizontal-and-vertical-scaling/)

## Related
- Parent: [[00 - System Design Fundamentals (Overview)]]
- Siblings: [[02 - Caching Strategies]], [[03 - Data Consistency, CAP Theorem & Replication]], [[04 - Database Scaling (Sharding & Partitioning)]], [[05 - Asynchronous Messaging & Microservices Communication]]
