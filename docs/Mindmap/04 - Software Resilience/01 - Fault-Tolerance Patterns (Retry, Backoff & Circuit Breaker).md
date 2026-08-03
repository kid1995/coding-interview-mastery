---
title: Fault-Tolerance Patterns (Retry, Backoff & Circuit Breaker)
mindmap_id: fault-tolerance-patterns-retry-backoff-circuit-breaker
node_type: topic
category: Software Resilience
parent: "[[00 - Software Resilience (Overview)]]"
tags: [coding-interview, software-resilience]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Fault-Tolerance Patterns (Retry, Backoff & Circuit Breaker)

> How to recover from a transient failure automatically, without your recovery attempt becoming the next outage.

## Definition & Core Concepts

Most failures a client sees when calling a network dependency are **transient**: a dropped packet,
a momentary GC pause on the server, a brief overload during a deploy. The naive fix — "if it
fails, call it again immediately" — is also the most dangerous one. If a downstream service is
struggling under load and every one of its thousands of callers immediately retries a failed call,
the retries land on top of the original traffic and make the overload worse, not better. This is a
**retry storm**: the retry logic that was supposed to add resilience instead amplifies the outage,
because it couples the callers' behavior to the very failure they're trying to route around.

**Exponential backoff** breaks that coupling by waiting longer between each successive retry
(e.g. 100ms, 200ms, 400ms, 800ms) instead of retrying at a fixed interval, giving the downstream
service room to recover instead of being hit at a constant rate. But backoff alone still has a
subtler failure mode: if every client backs off on the exact same schedule, they retry in
lockstep, and the downstream service now sees synchronized waves of traffic instead of one
sustained storm. **Jitter** — adding randomness to each client's backoff interval — spreads those
retries out over time so they don't collide, which is why "exponential backoff" without jitter is
an incomplete answer in an interview.

Retry with backoff and jitter still assumes the failure is worth retrying at all. If a downstream
service is not just briefly overloaded but genuinely down or in a bad state, continuing to retry —
even slowly — just keeps sending doomed requests and keeps consuming the caller's own resources
(threads, connections) waiting on calls that will fail. The **circuit breaker** pattern solves
this by tracking the failure rate of calls to a dependency and tripping into one of three states:

- **Closed** — the normal state. Calls pass through to the dependency, and failures are counted.
  If the failure rate crosses a configured threshold, the breaker trips to Open.
- **Open** — calls fail immediately without even attempting the network call, for a configured
  timeout period. This protects both the caller (no more threads blocked waiting on a dead
  dependency) and the callee (no more load added to a service that's already struggling).
- **Half-Open** — after the timeout elapses, the breaker lets a small number of probe requests
  through. If they succeed, the breaker closes and normal traffic resumes; if they fail, it trips
  back to Open and the timeout restarts.

Retry and circuit breaker are not competing solutions to the same problem — they compose. Retry
with backoff+jitter handles the common case of a single transient blip. The circuit breaker sits
above that and handles the case where the failures aren't transient: once the failure rate is high
enough to trip the breaker, it stops the retries from happening at all for a while, which is what
actually protects a genuinely struggling downstream service instead of just spacing out the damage.

## Best Practices

- Always pair retry with exponential backoff **and** jitter — backoff without jitter still
  produces synchronized retry waves under load.
- Cap the maximum number of retry attempts (or a maximum total elapsed retry time / "retry
  budget") — unbounded retries just delay the eventual failure while consuming resources.
- Never retry blindly; retry only operations that are safe to repeat. This is where fault
  tolerance depends on [[03 - Idempotency & Failure Recovery]] — retrying a non-idempotent write
  (e.g. "charge the customer") on a request that actually succeeded but whose *response* was lost
  can create a duplicate side effect, not just a duplicate attempt.
- Distinguish retryable failures (timeouts, 502/503/504, connection resets) from non-retryable
  ones (4xx client errors, business-rule rejections) — retrying a request that will deterministically
  fail again just wastes the retry budget.
- Set a circuit breaker's failure threshold and half-open probe interval deliberately, based on the
  dependency's real latency/error profile, rather than accepting a library's default — a threshold
  that's too sensitive trips on normal noise, and one that's too loose never protects anything.
- Retry at a single point in the call stack, not at every layer independently — if a client retries
  and the service it calls also retries its own downstream call, failures can multiply
  combinatorially instead of linearly.

## Real-World Use Case

**Case study:** Amazon's own engineering guidance in the Amazon Builders' Library describes how
AWS designs retry/backoff/timeout behavior for its own services: they pick an acceptable rate of
"false timeouts" (e.g. 0.1%) and size the timeout against the corresponding latency percentile of
the downstream service, and they explicitly call out that when retries happen at multiple layers
of a call chain without coordination, the effective retry count multiplies and can turn a small
backend blip into a large amplified load spike — which is precisely the retry-storm failure mode
this note describes.

## Hands-On Practice

Sketch a service that calls a flaky downstream payments API:

1. Wrap the call to the payments API in a retry loop with a maximum of 3 attempts, exponential
   backoff starting at 100ms (100ms, 200ms, 400ms), and random jitter of ±20% added to each wait.
2. Only enter the retry loop for responses classified as transient (timeout, connection reset,
   5xx) — a 4xx (e.g. "invalid card") should fail immediately with no retry.
3. Wrap the whole retrying call in a circuit breaker: failure threshold of, say, 50% over the last
   20 calls trips it to Open for 30 seconds, after which one probe call is allowed through
   (Half-Open) to decide whether to close or re-open.
4. To verify it actually works: in a test/staging environment, make the payments API stub return
   HTTP 500 for every call, and assert that (a) each logical request only produces the expected
   number of retries with correctly-spaced, jittered delays, and (b) after the configured number
   of failures the circuit breaker opens and subsequent calls fail *fast*, with no further network
   attempts, until the timeout elapses and a Half-Open probe is sent.

## Exam Tips

- If a candidate proposes "just retry on failure" with no mention of backoff, that's a red flag —
  call out the retry-storm risk explicitly; it's a very common gap interviewers probe for.
- Backoff without jitter is a half-answer — many candidates remember "exponential backoff" but
  forget jitter, and jitter is what actually prevents synchronized retry waves.
- Timeout and retry solve different problems and are often confused: a timeout bounds how long you
  wait for one attempt; retry decides whether to make another attempt after that. You generally
  need both — retry without a timeout on the underlying call can still hang indefinitely on the
  first attempt.
- Retry and circuit breaker are complementary, not redundant — be ready to explain what each one
  adds on top of the other, not just define them separately.

## References
- [Circuit Breaker pattern — Azure Architecture Center, Microsoft Learn](https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker)
- [Implement the Circuit Breaker pattern — .NET Microservices Architecture, Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-circuit-breaker-pattern)
- [Timeouts, retries, and backoff with jitter — Marc Brooker, Amazon Builders' Library (PDF)](https://d1.awsstatic.com/onedam/marketing-channels/website/aws/en_US/product-categories/developer-tools/approved/pdfs/timeouts-retries-and-backoff-with-jitter.pdf)

## Related
- Parent: [[00 - Software Resilience (Overview)]]
- Siblings: [[02 - Bulkheads, Timeouts & Graceful Degradation]], [[03 - Idempotency & Failure Recovery]], [[04 - Observability & Chaos Engineering]]
