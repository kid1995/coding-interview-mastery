---
title: Observability & Chaos Engineering
mindmap_id: observability-chaos-engineering
node_type: topic
category: Software Resilience
parent: "[[00 - Software Resilience (Overview)]]"
tags: [coding-interview, software-resilience]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Observability & Chaos Engineering

> How you verify your resilience design actually works, instead of just hoping it does.

## Definition & Core Concepts

Observability is usually described as three pillars, and each one answers a different question a
resilient system needs answered:

- **Logs** — what exactly happened, for one specific event, in detail. Best for answering "what
  went wrong in this one request" after you already know roughly where to look.
- **Metrics** — aggregate numeric trends over time (request rate, error rate, p99 latency). Cheap
  to store and query at scale, which is what makes them the right tool for alerting — you want to
  know "error rate just crossed 5%" without scanning every individual log line to compute it.
- **Traces** — the path a single request took across every service it touched, with timing for
  each hop. This is what makes latency debugging in a microservices call graph tractable: without
  a trace, "why was this request slow" requires manually correlating logs across N services by
  timestamp and hoping they line up.

The reason these have to exist **before** an incident, not during one, is that you cannot
retroactively instrument a system that's already on fire. If a service doesn't emit a metric for
"circuit breaker state" or "bulkhead pool utilization," you find that out exactly when you need it
most — mid-outage, trying to understand why requests are failing, with no data describing the
failure you're currently living through. Observability isn't a debugging feature bolted on after
the fact; it's part of the resilience design itself, because every pattern in this category (retry
counts, circuit breaker trips, bulkhead saturation, degraded-mode activations) is only verifiable
if it's observable.

**Chaos engineering** is the practice of deliberately injecting failure into a system — killing an
instance, adding artificial latency to a dependency, returning errors from a service that normally
works fine — in order to verify that the resilience mechanisms you designed (retry, circuit
breaker, bulkhead, graceful degradation) actually behave the way you think they do, rather than
just trusting the design on paper. This is the same distinction as unit-testing the happy path
versus testing the failure path: a system that has never actually been asked "what happens when
this dependency goes down" has an *assumed* answer to that question, not a *verified* one, and
those two are not the same thing — the assumed answer is frequently wrong (a fallback path with a
bug in it that's never been exercised, a circuit breaker threshold that never actually trips).

Underneath both of these is the **SRE mindset**: designing for failure as the default expectation
of a distributed system, not an edge case to handle if there's time left over. At scale, "will
something fail" isn't the right question — something is *always* failing somewhere in a large
enough system; the right question is "does the system as a whole keep working when its individual
parts don't," and that question can only be answered with both the visibility to see failures
happening (observability) and the discipline to test that the system's response to them is
correct (chaos engineering), rather than assumed.

## Best Practices

- Instrument at write-time, as part of building the feature, not as a follow-up task — code
  written without logging/metrics/tracing in mind usually needs a larger retrofit later than
  building it in from the start would have cost.
- Correlate logs, metrics, and traces with a shared identifier (a trace/request ID) so an alert on
  a metric can be followed straight to the specific traces and logs that explain it, instead of
  three disconnected data sources.
- Alert on symptoms that matter to users (SLOs like error rate or latency), not on every possible
  internal cause — alerting on causes produces noisy, low-signal pages that get ignored.
- Emit metrics for the resilience mechanisms themselves — circuit breaker state transitions,
  bulkhead pool saturation, retry counts, fallback/degraded-mode activations — not just for
  business metrics, since those are exactly what you need visible during an incident.
- Run chaos experiments with a stated hypothesis and a defined, limited blast radius (start in
  staging, or a small percentage of production traffic) and a way to abort immediately — chaos
  engineering is a controlled experiment, not "randomly break things and see what happens."
- Run chaos experiments repeatedly, on a schedule, not as a one-time exercise — resilience
  mechanisms can regress silently as code changes, and only a repeated experiment catches that.

## Real-World Use Case

**Case study:** Netflix's Chaos Monkey, introduced in 2011 and later expanded into the "Simian
Army," randomly terminates production instances during business hours specifically to verify that
Netflix's systems tolerate the loss of an individual instance without customer-visible impact.
The explicit premise, as Netflix's own engineering team described it, was that instance failure in
a large cloud deployment is a *when*, not an *if* — so rather than hoping their resilience design
handled it, they built a tool that forced it to happen continuously, turning an assumed capability
into a continuously re-verified one.

## Hands-On Practice

Take the flaky-payments-API service built up across the earlier notes in this category (retry +
backoff, circuit breaker, bulkhead, timeout, graceful degradation, idempotency) and add the
verification layer:

1. Add structured logs around the payments call that include a trace/request ID, the circuit
   breaker's current state, and the outcome (success / retried / fell back to degraded response).
2. Add metrics for: payments call error rate, payments call p99 latency, circuit breaker state
   (closed/open/half-open) over time, and bulkhead pool utilization for the payments client.
3. Add distributed tracing so a single checkout request's trace shows the time spent in the
   payments call specifically, separate from the rest of the request.
4. Run a chaos experiment in staging: configure the payments API stub to return 500s for 2 minutes,
   with a stated hypothesis ("the circuit breaker will open within N failed calls, the bulkhead
   will keep the product-catalog endpoint's latency unaffected, and checkout will serve the
   degraded 'payment processing' response instead of a 500"). Confirm the hypothesis against the
   metrics/traces you just added, and treat any mismatch as a resilience bug, not just an
   observability gap.

## Exam Tips

- A common tell: a candidate mentions adding monitoring only in response to being asked "how would
  you have caught this incident," rather than describing it as part of the original design — call
  out that observability needs to be built in upfront, not reconstructed after the fact.
- "We have logs" is not the same claim as "we have observability" — be ready to name what metrics
  and traces add on top of logs specifically (aggregate trend visibility, and cross-service request
  correlation, respectively), since interviewers often probe whether a candidate is conflating the
  three pillars.
- Chaos engineering is not "turn things off randomly" — a strong answer describes a hypothesis, a
  bounded blast radius, and a way to measure the outcome; describing it as unstructured is a
  common misconception worth correcting explicitly.
- Be ready to connect this note back to the rest of the category: observability and chaos
  engineering aren't a fifth independent resilience mechanism, they're how you verify the other
  three (retry/circuit breaker, bulkhead/timeout/degradation, idempotency) actually hold.

## References
- [Site Reliability Engineering — Google SRE](https://sre.google/books/)
- [Table of Contents — Google SRE book](https://sre.google/sre-book/table-of-contents/)
- [The Netflix Simian Army — Netflix Technology Blog](https://netflixtechblog.com/the-netflix-simian-army-16e57fbab116)

## Related
- Parent: [[00 - Software Resilience (Overview)]]
- Siblings: [[01 - Fault-Tolerance Patterns (Retry, Backoff & Circuit Breaker)]], [[02 - Bulkheads, Timeouts & Graceful Degradation]], [[03 - Idempotency & Failure Recovery]]
