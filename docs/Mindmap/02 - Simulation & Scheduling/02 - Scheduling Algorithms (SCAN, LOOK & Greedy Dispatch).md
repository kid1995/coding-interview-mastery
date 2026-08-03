---
title: Scheduling Algorithms (SCAN, LOOK & Greedy Dispatch)
mindmap_id: scheduling-algorithms-scan-look-greedy-dispatch
node_type: topic
category: Simulation & Scheduling
parent: "[[00 - Simulation & Scheduling (Overview)]]"
tags: [coding-interview, simulation-scheduling]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Scheduling Algorithms (SCAN, LOOK & Greedy Dispatch)

> The core decision logic for any dispatcher choosing what to do next — literally named after elevators, and the single most important theory node for solving Elevator Saga well rather than just passing it.

## Definition & Core Concepts

**SCAN (the "elevator algorithm")** is a disk-scheduling algorithm named after — and directly modeled on — how elevators traditionally behave: the arm (or elevator) moves in one direction, servicing every pending request it passes along the way, continues to the *end* of its range (the last cylinder, or the top/bottom floor), and only then reverses direction and sweeps back the other way, again servicing everything it passes. See [Wikipedia: Elevator algorithm](https://en.wikipedia.org/wiki/Elevator_algorithm) — it's literally named this because it was inspired by, and is still used to describe, real elevator dispatch behavior, and it originated as a way to minimize the seek time of a hard disk's read/write arm moving across cylinders (see [GeeksforGeeks: SCAN (Elevator) Disk Scheduling Algorithm](https://www.geeksforgeeks.org/dsa/scan-elevator-disk-scheduling-algorithms/)).

**LOOK** is SCAN's practical refinement: instead of always sweeping all the way to the physical end of the range (floor 1 or the top floor) before reversing, a LOOK-based scheduler reverses as soon as there are *no more pending requests* in the current direction — it "looks ahead" to see if continuing is even worthwhile. This avoids wasted travel to a floor nobody requested. See [Baeldung: The SCAN Algorithm](https://www.baeldung.com/cs/scan-algorithm) for the SCAN/LOOK distinction and worked examples. In elevator terms: SCAN always rides to the top even if the highest pending request is floor 5 of 10; LOOK reverses right after floor 5.

**Greedy nearest-request dispatch** is a different family entirely: rather than committing to a sweep direction, the dispatcher (or, in a multi-elevator system, the assignment logic) simply sends the elevator/car to whichever *pending request is currently closest* (by distance, or by estimated time-to-arrival). This is simple, reacts instantly to new information, and often *looks* highly efficient on any single decision — but it has a well-known structural weakness: **starvation**. If requests keep arriving near the elevator's current position, a request that is merely a little farther away can be repeatedly "jumped" by newer, closer requests and wait indefinitely, even though nothing in the algorithm intended to ignore it — it's simply never the *nearest* one at any given decision point.

The reason SCAN/LOOK exist at all is precisely to solve what greedy dispatch does not: they give an explicit **fairness guarantee**. Because a SCAN/LOOK sweep commits to a direction and services everything it passes before reversing, every request is guaranteed to be picked up within, at most, one full sweep cycle — there's a provable upper bound on wait time that greedy dispatch cannot offer.

## Best Practices

- **Name the tradeoff explicitly: throughput vs. fairness vs. worst-case wait.** Greedy nearest-request tends to minimize *average* wait time under light, evenly-distributed load (it's always doing the locally-optimal thing), but has an unbounded *worst-case* wait time (starvation). SCAN/LOOK bounds the worst case at the cost of sometimes making a "locally silly" move (passing a very close request to first finish the current direction, if that request is now behind the sweep).
- **Combine strategies rather than picking one absolutely.** Many real systems (and strong Elevator Saga solutions) use LOOK-style directional discipline for *within-sweep ordering*, plus a greedy or load-aware rule for *which idle elevator gets assigned a new hall call* in a multi-elevator bank. The two decisions (direction discipline vs. car assignment) are different problems and can use different strategies.
- **Prevent starvation directly if you must use greedy dispatch:** add an aging/priority boost (e.g. increase a request's effective priority the longer it waits, or cap max wait time and force-service anything past a threshold) so no request can be perpetually out-competed by newer, closer ones.
- **Use LOOK over plain SCAN whenever the physical/traversal cost of "going to the end" is non-trivial** — SCAN's only advantage over LOOK is implementation simplicity (no need to track "is there anything left in this direction"); LOOK strictly dominates on efficiency once you're willing to track pending requests per direction.
- **Distinguish hall calls from car calls in a multi-elevator design.** A hall call (someone pressed "up" on floor 5) needs an *assignment* decision (which elevator should answer it — nearest? least loaded? already heading that way?); a car call (a passenger inside pressed floor 8) only needs to be added to that specific elevator's own SCAN/LOOK queue. Conflating the two leads to inefficient bank-wide behavior.
- **Measure, don't assume.** "Greedy nearest-request is good enough" is a claim about a specific load profile — validate it against the load you actually expect (see Queueing Theory & Discrete-Event Simulation) rather than asserting it from intuition alone.

## Real-World Use Case

Case study: SCAN's origin is a real, well-documented one — it was developed for hard disk drive scheduling, where a physical read/write arm has to move across concentric cylinders to service pending I/O requests, and minimizing total arm travel (seek time) directly translates into disk throughput. The algorithm was named "elevator" because disk-scheduling researchers explicitly borrowed the mental model of a building elevator sweeping up and down servicing calls in order, rather than jumping erratically between the nearest requests (see [Wikipedia: Elevator algorithm](https://en.wikipedia.org/wiki/Elevator_algorithm) and [GeeksforGeeks: SCAN Elevator Disk Scheduling](https://www.geeksforgeeks.org/dsa/scan-elevator-disk-scheduling-algorithms/)). This is a rare case in CS pedagogy where the metaphor and the literal application (real elevators) are both genuinely accurate uses of the same algorithm.

Illustrative scenario: a single elevator in a 10-floor building is at floor 3, moving up, with pending requests at floors 6 and 9 and a just-arrived new request at floor 4 (behind it, relative to direction) plus one at floor 2. Under SCAN, the elevator continues up through 6 and 9, then all the way to floor 10 (even though nothing is pending there), then reverses and services 4 and 2 on the way down. Under LOOK, it continues up through 6 and 9, then reverses immediately after 9 (no need to touch floor 10) and services 4 and 2. Under pure greedy nearest-request from floor 3, the elevator would go to floor 4 first (distance 1) — reasonable here, but in a busier system this is exactly the pattern that lets floor 9's request keep losing to whatever new request appears closest, next.

## Hands-On Practice

Implement a LOOK-style dispatcher against Elevator Saga's real API (verified via [play.elevatorsaga.com/documentation.html](https://play.elevatorsaga.com/documentation.html)): `elevator.goToFloor(floorNum, forceNow)` queues (or, with `forceNow=true`, prioritizes) a destination in `elevator.destinationQueue`; `elevator.currentFloor()` returns the current floor; `elevator.destinationDirection()` returns `"up"`/`"down"`/`"stopped"`; hall calls arrive as `"up_button_pressed"`/`"down_button_pressed"` events on floor objects; car calls arrive as `"floor_button_pressed"(floorNum)` events on elevator objects.

```js
function init(elevators, floors) {
  elevators.forEach((elevator) => {
    elevator.on("floor_button_pressed", (floorNum) => {
      elevator.goToFloor(floorNum); // car call: just enqueue
    });

    elevator.on("idle", () => {
      dispatchNextSweepTarget(elevator, floors);
    });
  });

  floors.forEach((floor) => {
    floor.on("up_button_pressed", () => assignHallCall(elevators, floor, "up"));
    floor.on("down_button_pressed", () => assignHallCall(elevators, floor, "down"));
  });
}

// LOOK: when idle, look for the nearest pending request in the current
// sweep direction; reverse only when nothing remains that way.
function dispatchNextSweepTarget(elevator, floors) {
  const here = elevator.currentFloor();
  const pending = getPendingRequests(); // app-level queue of floor numbers
  const ahead = pending.filter((f) => f > here).sort((a, b) => a - b);
  const behind = pending.filter((f) => f < here).sort((a, b) => b - a);

  if (elevator.destinationDirection() !== "down" && ahead.length > 0) {
    elevator.goToFloor(ahead[0]);
  } else if (behind.length > 0) {
    elevator.goToFloor(behind[0]); // reverse: nothing left ahead
  } else if (ahead.length > 0) {
    elevator.goToFloor(ahead[0]);
  }
  // else: truly idle, nothing pending.
}

// Naive greedy car assignment for hall calls — cheap, but can starve
// far elevators' best-fit requests under bursty load (see Exam Tips).
function assignHallCall(elevators, floor, direction) {
  const target = floor.floorNum();
  const nearest = elevators.reduce((best, e) =>
    Math.abs(e.currentFloor() - target) < Math.abs(best.currentFloor() - target) ? e : best
  );
  nearest.goToFloor(target);
}
```

Exercise: run this in the actual game at [play.elevatorsaga.com](https://play.elevatorsaga.com/), then deliberately break the greedy `assignHallCall` fairness by simulating a burst of requests near one elevator — watch a far-away request wait far longer than any single request "should," and add an aging rule (e.g. track wait time per pending hall call and force-assign anything older than N seconds) to fix it.

## Exam Tips

- The single most common trap: presenting greedy nearest-request as an unqualified "good" answer. A strong response always names its weakness (starvation) unprompted and explains *why* it happens (the request is never the *nearest* at any decision point, not because it's ignored on purpose).
- Know the SCAN vs. LOOK distinction cold: SCAN always travels to the physical end of the range before reversing; LOOK reverses as soon as no more requests remain in the current direction. LOOK is strictly more efficient; SCAN is simpler to implement and reason about (no need to track "anything left ahead?").
- Be able to state the guarantee SCAN/LOOK gives that greedy doesn't: a *bounded worst-case wait* (roughly, at most one full sweep), vs. greedy's unbounded worst case.
- If asked to design a multi-elevator dispatcher, don't collapse "which elevator answers this hall call" and "in what order does one elevator visit its own queue" into a single algorithm — they're separate decisions (car assignment vs. within-car sweep discipline) and naming that distinction is a strong signal.
- Direction-reversal edge cases to watch for: what happens when a request arrives for a floor *behind* the elevator while it's mid-sweep? (Under strict SCAN/LOOK, it waits for the reversal — don't silently interrupt the sweep, or you lose the fairness guarantee that's the whole point of the algorithm.)

## References
- [Elevator algorithm - Wikipedia](https://en.wikipedia.org/wiki/Elevator_algorithm)
- [SCAN (Elevator) Disk Scheduling Algorithms - GeeksforGeeks](https://www.geeksforgeeks.org/dsa/scan-elevator-disk-scheduling-algorithms/)
- [The SCAN Algorithm - Baeldung on Computer Science](https://www.baeldung.com/cs/scan-algorithm)
- [Elevator Saga](https://play.elevatorsaga.com/)
- [Elevator Saga documentation](https://play.elevatorsaga.com/documentation.html)

## Related
- Parent: [[00 - Simulation & Scheduling (Overview)]]
- Siblings: [[01 - State Machines & Event-Driven Simulation]], [[03 - Queueing Theory & Discrete-Event Simulation]]
