---
title: Test Doubles & Mocking Strategy
mindmap_id: test-doubles-mocking-strategy
node_type: topic
category: Testability & Clean Code
parent: "[[00 - Testability & Clean Code (Overview)]]"
tags: [coding-interview, testability-clean-code]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Test Doubles & Mocking Strategy

> The vocabulary and judgment for replacing real collaborators in a test — and knowing when replacing them helps versus when it makes tests brittle.

## Definition & Core Concepts

Gerard Meszaros's taxonomy, popularized by Martin Fowler's "Mocks Aren't Stubs," identifies five kinds of test double, each answering a different question about what a collaborator needs to do during a test:

- **Dummy** — an object passed around to satisfy a parameter list but never actually used by the code path under test (e.g., a `Logger` argument a constructor requires but the test doesn't care about). Its only job is to compile/run without doing anything meaningful.
- **Fake** — a working implementation, but one that takes a shortcut unsuitable for production — the canonical example is an in-memory repository standing in for a real database. A fake genuinely *behaves* like the real thing (you can save an item and read it back), it's just not production-grade (no persistence across restarts, no real transactions).
- **Stub** — provides pre-canned answers to calls made during the test, and typically has no real logic beyond returning whatever was configured (e.g., `paymentGateway.charge() returns success`). A stub doesn't care whether or how it was called, only what to return when called.
- **Spy** — a stub that additionally records how it was called (call count, arguments passed) so a test can inspect that record afterward. A spy sits between a stub and a mock: it doesn't fail the test on its own, but it gives you data to assert against.
- **Mock** — an object "pre-programmed with expectations which form a specification of the calls they are expected to receive"; a mock can fail the test itself (via its own verification step) if the expected calls don't happen as specified. Mocks are inherently about verifying *behavior* (were the right calls made?), not *state* (what value resulted?).

**State verification vs. behavior verification** is the deeper distinction underlying all five: state verification exercises the system under test and then inspects the resulting state — of the object itself, or of a fake/stub collaborator — to decide if the test passed (dummies, fakes, and stubs are typically used this way). Behavior verification instead asserts that specific interactions occurred — that a particular method was called, with particular arguments, a particular number of times (spies and, especially, mocks are built for this). Neither is universally "more correct" — the right choice depends on what actually matters for the piece of behavior you're testing: if the *outcome* is what matters (an order ended up saved), assert state; if the *interaction itself* is the observable contract (an email-sending side effect that has no other observable trace), you may have no choice but to verify behavior.

**Classicist (Detroit school) vs. mockist (London school) TDD:** these are two different philosophies about how liberally to reach for test doubles.
- **Classicist / Detroit school** (associated with Kent Beck) prefers using real objects wherever they're cheap, fast, and deterministic, and reaches for a test double only when the real collaborator is genuinely inconvenient (slow, non-deterministic, external, or hard to construct). Assertions tend to be state-based, checking on real outcomes.
- **Mockist / London school** (associated with Steve Freeman and Nat Pryce) mocks essentially every collaborator with interesting behavior, regardless of how easy the real thing would be to use, because the point is to test a unit fully isolated from its neighbors and to let the mock's expected-interaction list double as an executable specification of that unit's contract with its collaborators. Assertions tend to be interaction/behavior-based.

The tradeoff: classicist tests exercise more real code per test (closer to an integration test in spirit) and are less likely to break on a pure refactor, but a failing classicist test can be harder to localize (which of several real collaborators actually broke?). Mockist tests isolate failures precisely to one unit and drive interface design early, but they couple the test to the *implementation* of how a unit talks to its collaborators — so a refactor that changes *how* a unit calls its dependencies (without changing observable behavior) can break a pile of mockist tests even though nothing is actually wrong. This is the single most common source of "brittle tests that break on every refactor."

## Best Practices

- Prefer **fakes over mocks** when a fast, deterministic in-memory real implementation is feasible (an in-memory repository beats a mocked repository — it actually exercises save/read logic instead of just recording that `save` was called).
- Reserve mocking for things that are genuinely necessary to fake: external services you don't control (payment gateways, third-party APIs), non-deterministic inputs (system clock, random number generation, UUID generation), and slow/expensive I/O (real network calls, disk access) that would make the test suite slow if run for real.
- Don't mock what you don't own loosely — when mocking a third-party SDK, wrap it behind your own narrow interface first (an adapter), and mock *your* interface, not the SDK's own classes directly; SDK internals change in ways you don't control, and mocking them ties your tests to library internals.
- Avoid mocking value objects, pure functions, or simple data holders — there's no behavior worth verifying there; just use the real thing.
- Assert on behavior (spy/mock call verification) only when the interaction *is* the observable contract (e.g., "an audit log entry must be written") — if the same result could be verified by checking resulting state, prefer the state check; it survives refactors that classicist-style testing is designed to tolerate.
- Name test doubles by role, not by mocking-framework mechanics — a variable named `fakeUserRepository` communicates intent better than `mockObj1`.

## Real-World Use Case

Illustrative scenario: a `WeatherAlertService` that reads the current time (`Clock.now()`), calls a third-party weather API, and, if a storm warning is active, writes a record to a database and sends a push notification. In a classicist-leaning test suite, the database would likely be a real in-memory fake (fast, deterministic, and it's your own code, cheap to fake well), the third-party weather API would be **stubbed or mocked** because it's an external, non-deterministic, rate-limited network dependency that has no place adding latency or flakiness to a unit test, and `Clock` would be **stubbed** to return a fixed instant because "storm warnings expire after 2 hours" logic is only deterministically testable if time itself is controlled rather than left to the real wall clock. The push-notification call, if the test cares only that the *right* customers were notified, is a good candidate for a spy (record calls, assert on them afterward) rather than a strict mock with rigid call-order expectations.

## Hands-On Practice

BEFORE — over-mocked, brittle test coupled to implementation detail, not behavior:

```
test('applies discount', () => {
  const mockCalculator = mock(DiscountCalculator);
  when(mockCalculator.roundToNearestCent(10.999)).thenReturn(11.00);
  when(mockCalculator.applyPercentage(11.00, 0.1)).thenReturn(9.90);

  const service = new PricingService(mockCalculator);
  service.priceItem(10.999, 0.1);

  verify(mockCalculator.roundToNearestCent(10.999)).calledOnce();
  verify(mockCalculator.applyPercentage(11.00, 0.1)).calledOnce();
});
```
This test breaks the moment `PricingService` refactors the *order* it calls `roundToNearestCent` and `applyPercentage` in, even if the final price is still correct — the test is asserting implementation steps, not the actual outcome.

AFTER — state-based assertion using the real `DiscountCalculator` (cheap, deterministic, no reason to fake it), only mocking what's genuinely external:

```
test('applies discount', () => {
  const service = new PricingService(new DiscountCalculator()); // real object, classicist style

  const finalPrice = service.priceItem(10.999, 0.1);

  expect(finalPrice).toBe(9.90); // asserts the outcome, survives any internal refactor
});
```
If `PricingService` also called an external `TaxRatesApi`, *that* dependency is the one worth stubbing — it's external, possibly slow, and its response needs to be fixed for a deterministic test.

## Exam Tips

- A very common interview trap: candidates say "I'd mock the database" as a reflex, without justifying *why*. Always state the reason — speed (no real I/O), determinism (no shared state or network flakiness between test runs), and isolation (avoiding side effects that other tests could observe). An unjustified "I'd mock X" reads as cargo-culting, not understanding.
- Don't confuse "high mock count" with "well-tested" — a suite full of mocks verifying call sequences can have 100% line coverage and still fail to catch a real bug, because it never exercises real interaction between components. Coverage measures lines executed, not correctness verified.
- Be ready to explain the classicist/mockist tradeoff concretely: mockist tests localize failures and double as interface specs but are coupled to implementation and break on pure refactors; classicist tests survive refactors better but a failure can require more digging to localize.
- If asked to fix a "flaky test," check first whether it depends on real wall-clock time, real randomness, or a real network call that should have been a stub/fake — this is the most common root cause of flakiness, and naming it correctly signals real understanding of test doubles' purpose.
- Know the taxonomy well enough to place any hand-rolled test helper into the right bucket (dummy/fake/stub/spy/mock) — interviewers sometimes hand you a class and ask "what kind of test double is this?"

## References
- [Mocks Aren't Stubs — Martin Fowler](https://martinfowler.com/articles/mocksArentStubs.html)

## Related
- Parent: [[00 - Testability & Clean Code (Overview)]]
- Siblings: [[01 - SOLID Principles & Dependency Injection]], [[03 - Test-Driven Development (TDD) Workflow]], [[04 - Hexagonal & Clean Architecture for Testability]]
