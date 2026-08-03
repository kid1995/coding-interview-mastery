---
title: Test-Driven Development (TDD) Workflow
mindmap_id: test-driven-development-tdd-workflow
node_type: topic
category: Testability & Clean Code
parent: "[[00 - Testability & Clean Code (Overview)]]"
tags: [coding-interview, testability-clean-code]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Test-Driven Development (TDD) Workflow

> A disciplined write-test-first workflow where testability problems surface immediately, as a design signal, instead of being discovered after the fact.

## Definition & Core Concepts

TDD, formalized by Kent Beck as part of Extreme Programming, is a workflow in which a test for a small piece of behavior is written *before* the implementation code that satisfies it. The cycle repeats in three tight steps, commonly called **Red-Green-Refactor**:

1. **Red** — write a test for the next small increment of behavior you want, and run it. It must fail (usually because the code it needs doesn't exist yet), and it must fail for the *right* reason — a compile error or a wrong-behavior assertion, not an accidental typo in the test itself. Seeing red first is not a formality; it's proof the test is actually capable of failing, and therefore capable of meaningfully passing later.
2. **Green** — write the *minimum* implementation code needed to make that test pass. Not the most elegant, general, or complete solution — the smallest one that turns the test green. Resisting the urge to build more than the current test demands is what keeps steps small and keeps you from designing ahead of the evidence you actually have.
3. **Refactor** — with the safety net of a passing test suite, clean up the code (and the test, if needed) without changing behavior: remove duplication, improve names, extract methods. Martin Fowler notes that skipping this third step is "the most common way ... to screw up TDD" — without it, the codebase accumulates the exact expedient shortcuts Green-step minimalism produces, and testability erodes over time instead of improving.

**Why writing the test first is a design activity, not just a testing activity:** to write a test before the implementation exists, you must first decide what the *caller's-eye view* of the code looks like — its name, its parameters, its return type, what it depends on. This forces interface design to happen from the perspective of how the code will actually be used, before you've committed to any particular internal implementation. Fowler's framing is exact here: "thinking about the test first forces us to think about the interface to the code first," which is "a key element of good design that many programmers struggle with" when writing implementation first and testing as an afterthought. A class that's awkward to instantiate in a test — because it needs seventeen constructor arguments, or reaches out to a real database in its constructor — reveals that awkwardness *immediately*, while the design is still cheap to change, rather than months later when a bug report reveals the same class is also awkward to reuse or extend.

**Baby steps discipline:** if you find yourself stuck in "red" for more than a few minutes (unsure how to make the test pass) or stuck in "green" writing a large chunk of code before the test passes, the step was too big. The fix is not to push through — it's to back up, write a smaller test for a smaller slice of the behavior, and get back to a fast Red-Green-Refactor rhythm. TDD's value comes largely from the tight feedback loop; a "step" that takes twenty minutes to go green has stopped being TDD in any meaningful sense and become "write code, then write a test for it," with all of the design-feedback benefit lost.

**How TDD and testability reinforce each other:** the core insight tying this whole category together is that when you're doing real TDD and a piece of code is hard to test, that difficulty is *information* — it's telling you the design has a problem (a hidden dependency, a class doing too much, a missing seam) — not that "testing is just annoying here." The correct response to "this is hard to test" under TDD is to fix the design (extract an interface, inject a dependency, split a responsibility) — the same fixes covered in the SOLID/DI and hexagonal architecture notes — not to reach for ever more elaborate mocking to force a test onto a badly-shaped unit.

## Best Practices

- Keep each Red step genuinely small — one assertion, one new behavior, not "implement the whole feature and then write a big test for it."
- Let test names describe *behavior*, not implementation (`returns_zero_discount_for_new_customers`, not `test1` or `testCalculateDiscountMethod`) — a well-named test doubles as a specification.
- Don't skip Refactor "just this once" — the compounding cost of always skipping it is exactly how TDD-in-name-only codebases end up just as messy as non-TDD ones.
- When a test is hard to write, resist immediately reaching for a bigger mocking framework — first ask whether the design needs a dependency extracted or a responsibility split (see the SOLID/DI note).
- Maintain a running list of the next few test cases you intend to write — Fowler notes "sequencing the tests properly is a skill," and picking tests that drive toward the interesting edge cases of the design, rather than random order, produces a cleaner emergent design.
- Run the full test suite (or at least the fast unit layer) after every Green and Refactor step — TDD's safety net only works if you actually keep checking it.

## Real-World Use Case

Illustrative scenario: implementing a `discount()` function for an e-commerce cart. A red-green-refactor walkthrough for one rule ("orders over $100 get 10% off"):
- **Red**: write `expect(discount(120)).toBe(12)` and run it — fails, because `discount` doesn't exist yet.
- **Green**: implement the smallest thing that passes — `function discount(total) { return total > 100 ? total * 0.1 : 0; }`.
- **Refactor**: nothing meaningfully duplicated yet at this size, so this step may be a no-op the first time through — which is fine; refactor is "as needed," not "mandatory busywork every cycle."
- Next **Red**: write `expect(discount(50)).toBe(0)` to lock in the boundary case (already passes with the current implementation, which is itself useful information — it confirms the boundary is already handled, and the test now guards it against regression).
- Next **Red**: write `expect(discount(100)).toBe(0)` to pin down the exact boundary (is $100 "over $100"? the test forces you to decide and encode the decision, rather than leaving it ambiguous).

This progression is small enough that each Red-Green cycle takes under a minute, and the accumulated test list becomes a readable specification of every boundary condition the discount rule actually handles.

## Hands-On Practice

A single tiny Red-Green-Refactor cycle for an `isPalindrome` helper:

**Red:**
```
test('empty string is a palindrome', () => {
  expect(isPalindrome('')).toBe(true);
});
```
Fails — `isPalindrome` is undefined.

**Green** (minimum code to pass):
```
function isPalindrome(s) {
  return true; // deliberately hardcoded — passes the ONE test that exists so far
}
```
This looks silly in isolation, but it's honest: no test yet demands anything smarter, and TDD says build only what current tests demand.

**Next Red:**
```
test('single character is a palindrome', () => {
  expect(isPalindrome('a')).toBe(true);
});
test('non-palindrome returns false', () => {
  expect(isPalindrome('abc')).toBe(false);
});
```
The hardcoded `return true` now fails the third test — forcing real logic.

**Green:**
```
function isPalindrome(s) {
  return s === [...s].reverse().join('');
}
```
**Refactor:** rename for clarity, maybe extract a `reverse(s)` helper if reused elsewhere — behavior unchanged, all tests still green.

## Exam Tips

- A common trap: candidates describe TDD as "write tests after to prove the code works" — that's just testing, not TDD. TDD's defining feature is that the test exists *before* the implementation and actively shapes it.
- Be ready to explain *why* red must come first (proves the test can actually fail / catches false-positive tests) — interviewers sometimes ask "why not just write the test and the code together?" specifically to see if you understand this.
- Don't conflate "100% coverage" with "well-designed under TDD" — TDD's design benefit comes from the *first-write* discipline, not from a coverage percentage measured after the fact; you can hit high coverage with implementation-first code and never get the interface-design benefit TDD provides.
- If asked to live-code with TDD, actually narrate small Red-Green-Refactor steps out loud rather than writing the whole solution and retrofitting a test — interviewers are often specifically evaluating the discipline of small steps, not just the final code.
- Recognize "I'm stuck" during a live TDD exercise as a signal to shrink the test, not push through with a bigger implementation leap — naming this discipline explicitly is a strong signal in an interview setting.

## References
- [Test Driven Development — Martin Fowler's Bliki](https://martinfowler.com/bliki/TestDrivenDevelopment.html)
- [What is Test Driven Development (TDD)? — Agile Alliance](https://agilealliance.org/glossary/tdd/)

## Related
- Parent: [[00 - Testability & Clean Code (Overview)]]
- Siblings: [[01 - SOLID Principles & Dependency Injection]], [[02 - Test Doubles & Mocking Strategy]], [[04 - Hexagonal & Clean Architecture for Testability]]
