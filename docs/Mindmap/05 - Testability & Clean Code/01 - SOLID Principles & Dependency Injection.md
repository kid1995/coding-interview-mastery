---
title: SOLID Principles & Dependency Injection
mindmap_id: solid-principles-dependency-injection
node_type: topic
category: Testability & Clean Code
parent: "[[00 - Testability & Clean Code (Overview)]]"
tags: [coding-interview, testability-clean-code]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# SOLID Principles & Dependency Injection

> Five design principles for managing responsibility and dependency — the two things that determine whether a class can be tested in isolation.

## Definition & Core Concepts

SOLID is an acronym for five object-oriented design principles introduced by Robert C. Martin and popularized as the antidote to code that is rigid, fragile, and immobile — three properties that also happen to make code nearly untestable. All five matter for maintainability, but two of them are the direct load-bearing principles for testability:

**S — Single Responsibility Principle (SRP):** A class should have only one reason to change. In practice this means a class should do one job — not "one method" but one *axis of change*. A class that both calculates a price and formats it for an invoice PDF has two reasons to change (pricing rules changing, invoice layout changing) and, worse for testing, you cannot exercise the pricing logic without also dragging in PDF rendering. SRP is what keeps a unit small enough that a test can set up its inputs and assert its outputs without an avalanche of unrelated setup.

**O — Open/Closed Principle:** Software entities should be open for extension but closed for modification. New behavior is added by adding new code (a new implementation of an interface), not by editing existing, already-tested code. This matters for testability because adding a feature doesn't require re-verifying every test that already passed against the old code path.

**L — Liskov Substitution Principle:** Subtypes must be substitutable for their base types without altering the correctness of the program. For testing this is what makes test doubles trustworthy: a stub or fake that implements an interface is only a safe substitute for the real implementation if it honors the same contract (same pre/post-conditions) that Liskov substitution requires of any subtype.

**I — Interface Segregation Principle:** Clients shouldn't be forced to depend on methods they don't use. Narrow, role-specific interfaces are also narrower to fake or stub — a `PaymentProcessor` interface with one method (`charge`) is trivial to stub; a bloated interface with twenty unrelated methods forces every test double to implement (or explicitly no-op) methods it doesn't care about.

**D — Dependency Inversion Principle (DIP):** High-level modules should not depend on low-level modules; both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions. This is the principle most directly responsible for testability, because it is what makes swapping in a test double possible *at all*. If an `OrderService` (high-level policy) directly instantiates a `PostgresOrderRepository` (low-level detail) inside its constructor, there is no seam — no interface boundary — at which a test can intervene. If instead `OrderService` depends on an `OrderRepository` interface, a test can supply an in-memory fake that implements that same interface, and `OrderService` never knows the difference.

**Dependency Injection (DI) vs. Dependency Inversion Principle (DIP) — a distinction worth being precise about:** DIP is a *principle* — a statement about which direction dependencies in your source code should point (toward abstractions, not concrete details). Dependency Injection is a *technique* — the mechanical act of supplying a class's dependencies from the outside (typically via constructor parameters, sometimes via field/setter injection or a DI framework) rather than having the class construct them itself. You can apply DIP without any DI framework at all (manual "poor man's DI" — passing an interface reference into a constructor is enough). And you can use a DI framework or container while still violating DIP, if what's being injected is a concrete class rather than an abstraction. DI is *how* you mechanically wire an application so that DIP holds true at runtime; DIP is the *design rule* that tells you dependencies should point at abstractions in the first place. They are related — DI is usually the tool that makes DIP practical in a real codebase — but conflating them is a common interview mistake.

**Constructor injection vs. field injection:** constructor injection passes dependencies as constructor parameters, making them explicit, immutable (assignable to `final`/`readonly` fields), and impossible to forget — the class literally cannot be instantiated without its required collaborators, which also means a test cannot forget to supply a double. Field/setter injection (common in older Spring code via `@Autowired` on a field) hides the dependency list, allows a half-constructed object to exist before injection runs, and makes required-vs-optional dependencies ambiguous.

## Best Practices

- Prefer **constructor injection** for required dependencies — it makes dependencies visible in the type's public contract and impossible to omit, and it plays nicely with immutability (assign to `final` fields).
- Use field/setter injection sparingly, and really only for genuinely optional dependencies with a sensible default.
- Depend on the **smallest interface that describes what you need**, not the concrete class or a "kitchen sink" interface (this is ISP working in service of DIP) — a class that only needs to read one row shouldn't depend on a repository interface with fifteen write methods.
- Keep constructors free of logic beyond assignment. A constructor that calls out to a database or opens a socket removes your test seam — you can no longer instantiate the class in a test without that side effect happening.
- Apply SRP by asking "what is this class's *one* reason to change?" — if you find yourself using "and" to describe what a class does, it likely has more than one responsibility and needs splitting before it can be tested cleanly.
- Don't over-apply DIP to primitives or fundamentally stable types (there's no value in injecting an interface around `Math` or a `String` formatter) — reserve abstraction seams for the things that actually vary across environments: I/O, time, randomness, external services.

## Real-World Use Case

Illustrative scenario: a `SubscriptionRenewalService` that needs to (a) calculate whether a subscription is due for renewal, and (b) charge the customer's card via a third-party payment gateway. Written without SOLID in mind, this is one class that both contains the renewal-date arithmetic and directly instantiates a concrete `StripeClient` inside a method — testing the renewal-date logic (which is the part with real business complexity and edge cases: leap years, timezones, proration) means either hitting Stripe's real API in a unit test or not testing that logic at all. Applying SRP splits it into a `RenewalPolicy` (pure logic, no I/O) and a `PaymentGateway` interface implemented by a `StripePaymentGateway` adapter; applying DIP means `SubscriptionRenewalService` takes a `PaymentGateway` in its constructor instead of constructing `StripeClient` itself. Now `RenewalPolicy` is tested with plain unit tests and zero network calls, and `SubscriptionRenewalService` is tested by injecting a fake `PaymentGateway` that just records whether `charge()` was called with the right amount.

## Hands-On Practice

BEFORE — hard to test, violates DIP (constructs its own dependency) and mixes concerns:

```
class OrderService {
  private db = new PostgresOrderRepository(connectionStringFromEnv());

  placeOrder(order) {
    validate(order);
    this.db.save(order);
    sendConfirmationEmail(order.customerEmail); // also does email — SRP violation
  }
}
```
To unit test `placeOrder`, you need a real Postgres instance and a real SMTP server reachable from the test — or no test at all.

AFTER — dependencies passed in via constructor, single responsibility per class:

```
class OrderService {
  constructor(private repository: OrderRepository, private notifier: OrderNotifier) {}

  placeOrder(order) {
    validate(order);
    this.repository.save(order);
    this.notifier.notifyConfirmation(order);
  }
}
```
A test now does:
```
const fakeRepo = new InMemoryOrderRepository();
const spyNotifier = new SpyOrderNotifier();
const service = new OrderService(fakeRepo, spyNotifier);

service.placeOrder(sampleOrder);

assert(fakeRepo.contains(sampleOrder));
assert(spyNotifier.wasNotifiedFor(sampleOrder));
```
No database, no network — the test runs in milliseconds and is deterministic.

## Exam Tips

- If asked "how would you make this class testable," don't just say "add a mock" — say *why*: because the class currently constructs its own concrete dependency, violating DIP, so there is no seam for a test double. Naming the principle shows you understand the cause, not just the fix.
- Don't conflate DI and DIP in an interview answer — DIP is the *what/why* (depend on abstractions), DI is the *how* (constructor/field injection wires it up). Interviewers listen for this distinction specifically.
- Watch for "SRP means one method per class" — that's wrong. SRP is about one *reason to change* (one axis of responsibility), not a method count. A class can have several small methods that all serve the same single responsibility.
- Don't over-engineer: injecting an interface for something that will only ever have one implementation and never needs a test double (e.g., a stateless math helper) is unnecessary ceremony — apply DIP where it buys real testability or real flexibility, not everywhere reflexively.
- Liskov violations often hide inside test doubles themselves: a hand-rolled stub that doesn't honor the real implementation's contract (e.g., returns `null` where the real implementation would throw) can make a test pass while the real integration would fail — mention this if asked about the risks of test doubles.

## References
- [S.O.L.I.D: The First 5 Principles of Object-Oriented Design — DigitalOcean](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
- [Dependency Inversion Principle Explained — Stackify](https://stackify.com/dependency-inversion-principle/)

## Related
- Parent: [[00 - Testability & Clean Code (Overview)]]
- Siblings: [[02 - Test Doubles & Mocking Strategy]], [[03 - Test-Driven Development (TDD) Workflow]], [[04 - Hexagonal & Clean Architecture for Testability]]
