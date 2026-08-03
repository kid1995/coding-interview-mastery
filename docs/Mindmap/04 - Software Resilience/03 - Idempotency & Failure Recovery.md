---
title: Idempotency & Failure Recovery
mindmap_id: idempotency-failure-recovery
node_type: topic
category: Software Resilience
parent: "[[00 - Software Resilience (Overview)]]"
tags: [coding-interview, software-resilience]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Idempotency & Failure Recovery

> The property that makes it safe to retry an operation you're not sure actually failed.

## Definition & Core Concepts

An operation is **idempotent** if applying it once has the same effect as applying it any number
of times. `SET balance = 100` is idempotent — run it once or ten times, the balance ends up 100
either way. `balance += 10` is not — run it once and the balance goes up by 10; run it three times
by accident and it goes up by 30. This distinction sounds academic until you connect it to
[[01 - Fault-Tolerance Patterns (Retry, Backoff & Circuit Breaker)]]: every retry is, by
definition, re-sending an operation whose outcome you're not certain of. Maybe the request never
reached the server. Maybe it reached the server, was processed successfully, and only the
*response* was lost on the way back — from the client's point of view those two situations are
indistinguishable, and it must decide whether to retry anyway. If the operation is idempotent,
retrying is always safe: worst case, you re-apply an operation that already happened and nothing
changes. If it's not, that same retry can silently double-charge a customer, ship an order twice,
or create a duplicate record — the retry logic that was supposed to add resilience becomes the
bug. This is the load-bearing reason idempotency belongs in the same category as retry and circuit
breakers, not a separate one: **retries are not safe by default, they're safe only when the
operation being retried is idempotent.**

Not every real-world operation is naturally idempotent (a payment charge, sending an email, or
appending to a log are inherently "do it once" actions), so systems make them idempotent
explicitly with an **idempotency key**: the client generates a unique key for each *logical*
operation (not per network attempt — the same key is reused across retries of the same logical
request) and sends it along with the request. The server checks whether it has already processed
that key; if so, it returns the stored result of the original attempt instead of processing the
request again. This turns a naturally non-idempotent operation ("charge $50") into an idempotent
one from the caller's perspective ("charge $50, identified as attempt `abc-123`") — repeating the
call with the same key is now guaranteed safe.

The subtle trap is assuming idempotency at one hop is enough. A request might be idempotent at the
API's front door — the dedupe check on the charge itself works correctly — but if processing that
charge fans out internally to three downstream actions (write a ledger entry, send a confirmation
email, notify a fulfillment service), and *those* aren't also deduplicated against the same key,
a retried request can still produce a duplicate email or a duplicate fulfillment notification even
though the primary charge was correctly deduplicated. Idempotency has to be enforced **end-to-end
through the call graph**, at every hop that has a side effect — not just at the entry point where
it's easiest to add.

## Best Practices

- Require an idempotency key for every mutating operation exposed to a client that might retry —
  don't rely on the operation being "naturally" idempotent unless you've actually verified it is.
- Store the idempotency key *together with the result* of the original request, not just a flag
  that it happened — a retried request should get back the same response as the original, not a
  generic "already processed" error.
- Make the dedupe check and the write atomic (e.g. a unique constraint on the idempotency key in
  the same transaction as the write) — checking "have I seen this key" as a separate step before
  the write creates a race window where two concurrent retries can both pass the check.
- Set an explicit TTL/expiry policy for stored idempotency records — keeping them forever is
  wasteful, but expiring them too soon reopens the double-processing window for a legitimately
  late retry.
- Trace every side effect the operation triggers, not just its primary write, and make sure
  each one is either idempotent on its own or deduplicated against the same key.
- Prefer operations that are naturally idempotent by design (e.g. `PUT`/"set" semantics) over
  needing an idempotency key at all, wherever the API shape allows it.

## Real-World Use Case

**Case study:** Stripe's API lets clients send an `Idempotency-Key` header on requests that create
side effects (like creating a charge). If the same key is reused within Stripe's retention window,
Stripe returns the result of the original request instead of creating a second charge — this is
explicitly designed so that a client can safely retry a request after a network timeout without
knowing whether the original attempt actually succeeded, which is exactly the "was it processed or
not" ambiguity described above.

## Hands-On Practice

Sketch a payments service that a client calls to charge a card, where the network between client
and server is unreliable enough that the client sometimes can't tell if its request succeeded:

1. Require every "create charge" request to include a client-generated `idempotencyKey`.
2. On the server, before processing, look up `(endpoint, idempotencyKey)` in a dedupe store within
   the same transaction as the charge write. If found, return the previously stored response
   immediately without reprocessing. If not found, process the charge and atomically store the key
   alongside the resulting response.
3. Make sure the downstream "send confirmation email" step is also keyed off the same
   idempotencyKey (or made naturally idempotent, e.g. "upsert the confirmation record" instead of
   "append an email send") so a retried request doesn't send two emails even though the charge
   itself was correctly deduplicated.
4. To verify it actually works: fire the same charge request twice concurrently (simulating a
   client retry racing the original attempt), and assert that exactly one charge exists in the
   ledger, both callers receive an identical response, and exactly one confirmation email was
   queued.

## Exam Tips

- If a candidate's answer to "how do you make this resilient to network failures" is "add
  retries" with no mention of idempotency, treat that as an incomplete answer — it's a very common
  gap and a strong signal interviewers watch for.
- Don't assume an operation is idempotent just because it uses `PUT` or looks like a "set"
  operation — check whether it has *any* side effect beyond updating the primary resource (an
  email, a webhook, a counter increment somewhere else); those side effects need their own
  dedupe.
- Be ready to trace idempotency through a multi-hop call graph, not just describe it at a single
  API boundary — interviewers may explicitly probe "what if this fans out to three services" to
  see if the guarantee actually holds end-to-end.
- Idempotency keys need a defined lifetime — an answer that stores keys forever (unbounded growth)
  or never expires them is missing an operational detail interviewers may ask about directly.

## References
- [Designing robust and predictable APIs with idempotency — Stripe Blog](https://stripe.com/blog/idempotency)
- [Role of Idempotent APIs in Modern System Design — GeeksforGeeks](https://www.geeksforgeeks.org/system-design/role-of-idempotent-apis-in-modern-systems-design/)

## Related
- Parent: [[00 - Software Resilience (Overview)]]
- Siblings: [[01 - Fault-Tolerance Patterns (Retry, Backoff & Circuit Breaker)]], [[02 - Bulkheads, Timeouts & Graceful Degradation]], [[04 - Observability & Chaos Engineering]]
