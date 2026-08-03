---
title: State Machines & Event-Driven Simulation
mindmap_id: state-machines-event-driven-simulation
node_type: topic
category: Simulation & Scheduling
parent: "[[00 - Simulation & Scheduling (Overview)]]"
tags: [coding-interview, simulation-scheduling]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# State Machines & Event-Driven Simulation

> Model a real-time controller as an explicit set of states and transitions, and drive it by events rather than by polling — this is the structural backbone every scheduling problem sits on top of.

## Definition & Core Concepts

A **finite state machine (FSM)** is a model of computation built from four pieces: a finite set of **states** (the distinct "modes" the system can be in), **events/triggers** (inputs that can cause the system to change), **transitions** (rules of the form "if in state A and event E occurs, move to state B"), and **actions** (side effects performed on a transition, or on entering/exiting a state — e.g. "open the doors," "notify the caller"). The system is always in exactly one state at any point in time, which is the whole point: it collapses "what could be happening right now" from a combinatorial explosion of boolean flags down to one known value you can switch on. See the classic reference at [state-machine.com/fsm](https://www.state-machine.com/fsm) for the formal treatment (states, events, transitions, actions, and the state topology).

**Event-driven design** is the complementary idea about *how* the state machine gets its inputs. In an event-driven system, the program doesn't sit in a loop constantly asking "has anything changed yet?" (polling); instead, it registers handlers and the runtime (or your own dispatch loop) invokes them only when something actually happens — a button press, a network packet, a timer firing. See [Wikipedia: Event-driven programming](https://en.wikipedia.org/wiki/Event-driven) for the general model of event producers, an event loop/dispatcher, and event handlers.

Why do these two ideas belong together for problems like Elevator Saga? Because an elevator controller is naturally described as: "the elevator is in one of a handful of states (idle, moving up, moving down, doors open, doors closing), and it changes state only in reaction to specific events (arrived at floor, button pressed, doors finished closing)." If you instead write it as a pile of mutable variables (`isMoving`, `targetFloor`, `doorsAreOpen`, `lastDirection`, ...) checked with nested `if`/`else` in a tight loop, you get combinatorial state explosion — some combinations of those boolens are meaningless or unreachable, but nothing in the code says so, and every new feature risks breaking a combination you didn't think about. An explicit FSM makes illegal states structurally hard to represent and makes the *set* of reachable states and transitions something you can literally enumerate, diagram, and unit-test one transition at a time.

Event-driven design fits the same problem for a second reason: real-time controllers don't control their own tempo. Passengers press buttons whenever they want; elevators arrive at floors whenever their physics says so. A polling design ("every tick, check every possible condition") wastes cycles doing nothing most of the time and — worse — can miss or double-handle events that occur between polls if the polling interval isn't tightly matched to event timing. An event-driven design instead says "when floor_button_pressed fires, run this handler," which is both more efficient and a more faithful model of what's actually happening in the world.

## Best Practices

- **Enumerate states explicitly** (e.g. as an enum/string union: `"idle" | "moving_up" | "moving_down" | "doors_open"`), never as an implicit combination of independent booleans. If you catch yourself writing `if (isMoving && !doorsOpen && direction === "up")`, that's a sign the state should be named directly.
- **Keep transitions total and explicit.** For every (state, event) pair, know — or explicitly assert/ignore — what happens. An event arriving in a state that "shouldn't" receive it (e.g. `floor_button_pressed` while doors are still opening) is a real scenario in a live system; decide on purpose whether to queue it, ignore it, or handle it, rather than let it fall through undefined behavior.
- **Separate transition logic from action/side-effect code.** The "what state do we go to" decision should be a pure function of (current state, event) wherever possible; the "what do we do about it" (call `goToFloor()`, update a UI) should be a separate step. This is what makes the state machine unit-testable without mocking a whole runtime.
- **Model timers and "idle" as first-class events**, not as special cases. Elevator Saga's `"idle"` event and its `update(dt, ...)` tick are both just more inputs to the same dispatch logic — treat them uniformly rather than bolting on special-case code paths.
- **Avoid deeply nested conditionals as a proxy for states.** More than roughly 2 levels of nested `if`/`switch` driven by mutable flags is usually a sign an implicit state machine wants to become an explicit one.
- **Keep the event handler idempotent-aware.** Event-driven systems occasionally re-deliver or race events (e.g. two floor arrivals in the same tick); decide explicitly whether a handler is safe to run twice and guard it if not.

## Real-World Use Case

Illustrative scenario: model a single elevator car's controller as an FSM with states `Idle`, `MovingUp`, `MovingDown`, and `DoorsOpen`. Events: `floor_button_pressed(floor)` (someone requested a floor), `arrived_at_floor(floor)`, `doors_should_open`, `doors_closed`. Transitions: from `Idle`, a `floor_button_pressed` event for a floor above the current one transitions to `MovingUp` (action: start moving); from `MovingUp`, an `arrived_at_floor` event matching the target transitions to `DoorsOpen` (action: open doors) — otherwise it stays in `MovingUp`; from `DoorsOpen`, a `doors_closed` event transitions back to `Idle` if the request queue is empty, or directly to `MovingUp`/`MovingDown` if more requests remain. Drawing this as a small directed graph (states as nodes, events as labeled edges) makes it trivial to spot missing cases — e.g. "what happens if a button is pressed for the current floor while doors are already open?" — before you've written a single line of dispatch code.

Case study: the elevator itself is the textbook example used across control-systems and software-design teaching material precisely because its state space is small enough to draw on a whiteboard but rich enough to expose every classic FSM pitfall (unreachable states, missing transitions, races between near-simultaneous events).

## Hands-On Practice

Sketch (in JS-ish pseudocode) a minimal FSM-driven elevator controller shaped to fit Elevator Saga's actual callback contract, verified against its documentation: `init(elevators, floors)` is called once at challenge start, and `update(dt, elevators, floors)` is called repeatedly during the challenge ([play.elevatorsaga.com/documentation.html](https://play.elevatorsaga.com/documentation.html)).

```js
// Per-elevator state, driven by real Elevator Saga events.
function makeElevatorFSM(elevator) {
  let state = "idle";

  elevator.on("idle", () => {
    state = "idle";
    // Decision logic (covered in the Scheduling Algorithms note) picks
    // the next target here, e.g. elevator.goToFloor(nextFloor).
  });

  elevator.on("floor_button_pressed", (floorNum) => {
    // A passenger inside pressed a button — queue it regardless of
    // current state; goToFloor() appends to destinationQueue.
    elevator.goToFloor(floorNum);
  });

  elevator.on("passing_floor", (floorNum, direction) => {
    // Optional: could decide to stop early if this floor has a
    // matching pending hall call in the same direction.
  });

  elevator.on("stopped_at_floor", (floorNum) => {
    state = "doors_open"; // conceptual state; doors are handled by the game engine
  });

  return { getState: () => state };
}

function init(elevators, floors) {
  elevators.forEach(makeElevatorFSM);
  floors.forEach((floor) => {
    floor.on("up_button_pressed", () => { /* record a pending hall call */ });
    floor.on("down_button_pressed", () => { /* record a pending hall call */ });
  });
}

function update(dt, elevators, floors) {
  // Most logic lives in event handlers above; update() is mainly a
  // fallback tick for time-based decisions (e.g. timeouts, rebalancing).
}
```

Note the events (`idle`, `floor_button_pressed`, `passing_floor`, `stopped_at_floor` on elevators; `up_button_pressed`, `down_button_pressed` on floors) and methods (`goToFloor`, `stop`, `currentFloor`, `destinationQueue`) above are taken directly from the live Elevator Saga documentation, not guessed. Try extending this skeleton so the `state` variable actually gates behavior (e.g. ignore new `goToFloor` calls while conceptually "docking") and unit-test the transition table in isolation from the game engine by calling the handler functions directly with fake floor numbers.

## Exam Tips

- If asked to "design a controller for X," name the state machine explicitly (states + transitions) before writing any code — interviewers are often specifically checking whether you reach for structure instead of ad-hoc flags.
- A common trap: conflating "state" with "variable." Not every mutable field is a state; a state machine should have a small, enumerable set of *mutually exclusive* states. If two "states" can be true simultaneously, they're not states — split them into orthogonal dimensions or merge them into one enum.
- Watch for race/ordering edge cases explicitly: what happens when two events arrive "simultaneously" (e.g. a button press exactly as doors finish closing)? A strong answer names an explicit ordering/priority rule rather than assuming it away.
- Event-driven does not mean "no polling ever" — real systems often combine both (event handlers for the common case, a periodic tick for timeouts/fallbacks, as Elevator Saga's `update(dt, ...)` demonstrates). Don't over-claim "pure event-driven is always better."
- Be ready to explain *why* event-driven design suits real-time controllers specifically: it matches the system's actual causal structure (things happen because of external triggers, not because your loop decided to check), and it avoids wasted work and missed/duplicated events inherent to naive polling.

## References
- [State Machines - state-machine.com](https://www.state-machine.com/fsm)
- [Event-driven programming - Wikipedia](https://en.wikipedia.org/wiki/Event-driven)
- [Elevator Saga documentation](https://play.elevatorsaga.com/documentation.html)

## Related
- Parent: [[00 - Simulation & Scheduling (Overview)]]
- Siblings: [[02 - Scheduling Algorithms (SCAN, LOOK & Greedy Dispatch)]], [[03 - Queueing Theory & Discrete-Event Simulation]]
