---
title: System Design Fundamentals
mindmap_id: system-design-fundamentals
node_type: category
parent: "[[00 - Coding Interview Mastery Mind Map (Index)]]"
children: ["[[01 - Scalability & Load Balancing]]", "[[02 - Caching Strategies]]", "[[03 - Data Consistency, CAP Theorem & Replication]]", "[[04 - Database Scaling (Sharding & Partitioning)]]", "[[05 - Asynchronous Messaging & Microservices Communication]]"]
tags: [coding-interview, category]
source: "coding-interview-mastery vault"
created: 2026-08-03
---

# System Design Fundamentals

> The core toolkit for "design X" interview questions — how large systems scale, stay available, and stay consistent.

## Overview

System design interviews don't test whether you know a specific product's architecture — they test whether you can reason about tradeoffs under constraints that don't exist in a LeetCode problem: unreliable networks, servers that crash mid-request, data too large for one machine, and traffic that spikes 100x without warning. The five topics in this category are the load-bearing vocabulary for that reasoning. Every "design a URL shortener / design Twitter / design a ride-sharing app" question decomposes into some combination of: how do we distribute load across machines, how do we avoid hitting the database for every request, what consistency guarantee can we actually promise once data lives on more than one node, how do we split data that no longer fits on one machine, and how do services talk to each other without falling over together.

This matters beyond interviews too. Production incidents are disproportionately caused by getting these fundamentals wrong: a cache with no invalidation strategy serving stale data for hours, a load balancer with no health checks routing traffic into a crashed instance, a "CP" database deployed where the business actually needed "AP," a shard key that concentrates 90% of traffic on one node, or a synchronous call chain that turns one slow downstream service into a cascading outage. Understanding these concepts deeply — not just the buzzwords, but *why* each tradeoff exists — is what separates an engineer who can design resilient systems from one who can only operate systems someone else designed.

The five nodes below are not independent — a real design answer usually threads through all of them in sequence, and the interviewer is listening for whether you can move fluidly between the layers rather than reciting facts about just one.

## Topics in This Category

- [[01 - Scalability & Load Balancing]] — how to add capacity (vertical vs horizontal) and how a load balancer spreads traffic across it
- [[02 - Caching Strategies]] — how to keep frequently-read data close to the request and off the database, and why invalidation is the hard part
- [[03 - Data Consistency, CAP Theorem & Replication]] — what guarantees you can make about data correctness once it's copied across nodes, and the AP/CP tradeoff under a network partition
- [[04 - Database Scaling (Sharding & Partitioning)]] — how to split data itself across machines when one database can no longer hold or serve it all
- [[05 - Asynchronous Messaging & Microservices Communication]] — how services talk to each other, and when decoupling with a queue beats a direct synchronous call

## How These Topics Fit Together

Think of a request's journey through a system as a narrative these five topics tell in order. First, **scalability and load balancing** get the request to a healthy server at all — this is the entry point, the layer that turns "one server" into "a fleet of servers" and decides which one handles this request. Once the request reaches application code, **caching** is the first line of defense against hammering the database — most reads in a well-designed system never reach persistent storage at all. When a request *does* need the database, and that database has been replicated or sharded for scale, **the CAP theorem and consistency model** determine what answer you're actually allowed to promise the caller — is it the absolute latest write, or could it be a few seconds stale. When the data itself has outgrown a single database instance, **sharding** is the mechanism that splits it across many, and it inherits every consistency and replication concern from the previous layer. Finally, when one service's work depends on another service's work but shouldn't be blocked waiting for it, **asynchronous messaging** decouples them — trading immediate consistency for resilience, so that a spike or outage in one component doesn't cascade into all of them.

A strong interview answer moves through this chain deliberately: start simple (single server, single DB), identify the first bottleneck, apply the cheapest fix that solves it (usually caching or a read replica before sharding), and only reach for the next layer of complexity when you can articulate *why* the simpler layer isn't enough. That "graduated complexity" narrative — not a laundry list of buzzwords — is what these five notes are built to support.

## References
- [Horizontal and Vertical Scaling in System Design — GeeksforGeeks](https://www.geeksforgeeks.org/system-design/system-design-horizontal-and-vertical-scaling/)
- [CAP Theorem — MongoDB](https://www.mongodb.com/resources/basics/databases/cap-theorem)
