---
title: Software Resilience
mindmap_id: software-resilience
node_type: category
parent: "[[00 - Coding Interview Mastery Mind Map (Index)]]"
children: ["[[01 - Fault-Tolerance Patterns (Retry, Backoff & Circuit Breaker)]]", "[[02 - Bulkheads, Timeouts & Graceful Degradation]]", "[[03 - Idempotency & Failure Recovery]]", "[[04 - Observability & Chaos Engineering]]"]
tags: [coding-interview, category]
source: "coding-interview-mastery vault"
created: 2026-08-03
---

# Software Resilience

> The theory behind making software survive real-world failure instead of just working on the happy path.

## Overview

Most interview prep and most tutorials teach the happy path: the request arrives, the database
answers, the downstream API responds in 40ms, everyone goes home. Production systems don't live
there. Networks partition mid-request. A downstream service you don't own gets slow for no reason
you control. A deploy on someone else's team takes their API from 99.9% availability to 60% for
eleven minutes. None of that is a bug in your code — it's the normal operating condition of any
system with more than one machine in it. Resilience is the set of deliberate design decisions that
decide what happens to *your* system when that normal condition occurs: does one slow dependency
take down your whole application, or does it degrade one feature while everything else keeps
working?

This is why resilience has to be designed in, not discovered during an incident. A system that
"just works" in every demo and every test run has usually never been asked what it does when a
call it depends on times out, returns garbage, or simply never returns at all — and the first time
it's asked that question for real is 2am during an outage, which is the most expensive possible
time to learn the answer. The patterns in this category are the vocabulary for answering that
question in advance: on paper, in a design review, in an interview — and then verifying, rather
than assuming, that the answer holds under real failure.

Interviewers probe this category specifically because it separates candidates who can write
*correct* code from candidates who can write code that survives contact with a distributed system.
A solution that's O(n) and passes every test case but falls over the instant a dependency is slow,
or silently double-charges a customer on retry, is not a complete solution to a real-world backend
problem — it's a solution to a simplified version of it that doesn't actually exist in production.

## Topics in This Category

- [[01 - Fault-Tolerance Patterns (Retry, Backoff & Circuit Breaker)]] — how to recover from a
  transient failure automatically without making the outage worse
- [[02 - Bulkheads, Timeouts & Graceful Degradation]] — how to contain the blast radius when a
  dependency fails and automatic recovery isn't enough
- [[03 - Idempotency & Failure Recovery]] — the property that makes all of the above safe to retry
  in the first place
- [[04 - Observability & Chaos Engineering]] — how you verify all of the above actually works,
  instead of just hoping it does

## How These Topics Fit Together

Read these four notes as one continuous argument rather than four unrelated buzzwords. **Retry,
backoff, and circuit breakers** are the first line of defense: most failures in a distributed
system are transient (a packet drop, a momentary GC pause, a brief overload), and a well-designed
client can recover from those automatically, as long as it retries carefully instead of naively —
naive immediate retries are exactly what turns a brief blip into a full outage. **Bulkheads and
timeouts** are the second line of defense, for when the failure isn't transient: they don't try to
recover the call, they contain it, so that one dependency stuck in a bad state can't exhaust a
shared resource pool and take unrelated features down with it, and **graceful degradation** is
what the user sees when that containment kicks in — a reduced experience instead of a blank error
page.

Neither of those first two layers is safe without the third: **idempotency**. Every retry in layer
one, and every fallback-and-retry-later path in layer two, is only safe if replaying the operation
doesn't cause a second side effect — a second charge, a second email, a second shipped order.
Idempotency is the property that makes "just retry it" a valid answer instead of a data-corruption
bug waiting to happen, which is why note 03 explicitly loops back to note 01.

None of the first three layers can be trusted on faith. **Observability** is how you know your
system is actually behaving the way you designed it to — logs, metrics, and traces are what let
you see a circuit breaker open, a bulkhead saturate, or a retry storm forming, in real time instead
of after a postmortem. And **chaos engineering** is how you stop hoping your resilience design
works and start proving it: deliberately injecting the failures you designed for, in a controlled
way, to confirm the circuit actually opens, the bulkhead actually isolates, and the fallback
actually serves a degraded-but-working response — because an untested failure path is, in
practice, an undesigned one.

## References
- [Circuit Breaker pattern — Azure Architecture Center, Microsoft Learn](https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker)
- [Google SRE — book overview](https://sre.google/books/)
