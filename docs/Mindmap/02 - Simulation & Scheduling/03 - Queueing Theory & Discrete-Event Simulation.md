---
title: Queueing Theory & Discrete-Event Simulation
mindmap_id: queueing-theory-discrete-event-simulation
node_type: topic
category: Simulation & Scheduling
parent: "[[00 - Simulation & Scheduling (Overview)]]"
tags: [coding-interview, simulation-scheduling]
source: "coding-interview-mastery vault — designed from scratch, verified via web research"
created: 2026-08-03
review_next: 2026-08-17
---

# Queueing Theory & Discrete-Event Simulation

> The math and simulation technique for reasoning about queues over time — the bridge between "I solved one elevator problem" and "I understand scheduling systems in general."

## Definition & Core Concepts

**Discrete-event simulation (DES)** is a way of simulating a system by advancing simulated time in jumps directly to the next moment something happens (the next *event*), rather than stepping forward in small fixed increments and checking "did anything happen yet?" at each one. A DES engine keeps a future-event list (often a priority queue ordered by scheduled time), pops the earliest event, advances the simulation clock straight to that event's timestamp, executes the event's effects (which may schedule new future events), and repeats. See [MathWorks: Role of Queues in SimEvents Models](https://www.mathworks.com/help/simevents/gs/role-of-queues-in-simevents-models.html) for how a DES engine and queues interact in a concrete modeling tool, and [Simio textbook: Queueing Theory](https://textbook.simio.com/SASMAA7/ch-queueing.html) for the underlying theory. This is directly relevant to Elevator Saga: nothing forces you to simulate at fixed small time steps — the meaningful moments are "a button was pressed" or "an elevator arrived at a floor," and a DES-style mental model (react to the next event, not to every tick) is exactly what event-driven design (see the State Machines note) gives you for free.

**Queueing theory** is the mathematical study of waiting lines — how requests (customers, jobs, passengers) accumulate in front of a limited-capacity server and how long they wait. The foundational quantities:
- **Arrival rate (λ, lambda)** — how often new requests arrive, on average (requests per unit time).
- **Service rate (μ, mu)** — how many requests a single server can complete, on average, per unit time.
- **Utilization (ρ, rho) = λ / μ** — the fraction of time the server is busy. If ρ ≥ 1, arrivals are coming in at or faster than the server can clear them, and the queue grows without bound over time (the system is "unstable"); a healthy system needs ρ < 1, and — critically — average wait time doesn't grow linearly as ρ approaches 1, it grows sharply (nonlinearly), which is why systems that look "fine" at 70% utilization can degrade badly at 90%.

Why this matters beyond one elevator: a single elevator is, structurally, a *server* — it has a service rate (how many passenger-trips per minute it can complete) and faces an arrival rate (how often passengers show up). A bank of elevators is a *multi-server queueing system*. The exact same vocabulary applies unmodified to a web server handling requests, a job queue processing background tasks, or a customer support system routing tickets — which is precisely why this topic is the "bridge" from one specific simulation problem to general scheduling/systems thinking.

## Best Practices

- **Always ask "what's my λ and my μ?" before optimizing a scheduler.** A scheduling algorithm that's excellent at ρ = 0.4 (light load) can fall apart at ρ = 0.9 (heavy load) — know which regime you're actually designing for before picking SCAN/LOOK vs. greedy dispatch.
- **Use discrete-event simulation to validate a scheduler empirically, not just analytically.** Closed-form queueing formulas (like M/M/1 results) assume clean statistical distributions (e.g. Poisson arrivals); real systems (and Elevator Saga's passenger generators) rarely match those assumptions exactly. A DES-style test harness — generate synthetic arrival events, run your real dispatch code against them, measure actual wait times — catches gaps that formulas miss.
- **Track both average and worst-case (tail) wait time**, not just the mean. A scheduler can have an excellent average wait while starving a small fraction of requests badly (see the Scheduling Algorithms note on greedy-dispatch starvation) — queueing metrics should always include a percentile (e.g. p95/p99 wait time), not just the mean.
- **Model burstiness, not just average rate.** A system can be stable (ρ < 1) "on average" and still fail during a burst if arrivals aren't smooth — size buffers/queues and reason about worst-case bursts, not just steady-state averages.
- **Prefer event-driven/DES-style advancement over fixed-timestep polling whenever event timing is irregular** — it's both more efficient (no wasted "nothing happened" checks) and more numerically exact (you never "miss" an event by landing between two fixed steps).

## Real-World Use Case

Illustrative scenario: an office building's elevator bank sees roughly 2 passenger arrivals per minute across all floors during a normal morning (λ = 2/min), and each elevator can complete roughly 3 passenger trips per minute (μ = 3/min per car). With 1 elevator, ρ = 2/3 ≈ 0.67 — busy but stable. During the 8:45-9:00 morning rush, arrivals might spike to λ = 5/min while μ stays at 3/min — ρ > 1 with a single car, meaning the queue (people waiting in the lobby) grows for as long as the spike lasts, no matter how good the dispatch algorithm is. This is the queueing-theory explanation for why "my elevator logic works fine in testing but falls apart during the rush-hour Elevator Saga levels" — the levels are specifically designed to push ρ toward or past 1, at which point the *scheduling algorithm's* job shifts from "minimize average wait" to "prevent the queue from cascading and answer requests roughly in a fair order" — exactly the throughput-vs-fairness tradeoff covered in the Scheduling Algorithms note.

This generalizes directly: a background job queue with λ = 100 jobs/min arriving and a single worker completing μ = 80 jobs/min (ρ = 1.25) will show an ever-growing backlog no matter how cleverly the single worker orders its work — the fix is adding capacity (more workers/servers), not a smarter single-server algorithm. Recognizing "this is a ρ ≥ 1 problem, not an algorithm problem" is a core piece of systems-design judgment that transfers well beyond elevators.

## Hands-On Practice

Build a minimal discrete-event simulation harness to stress-test a scheduling algorithm outside the Elevator Saga UI, so you can push λ far past what the game's browser levels let you try:

```js
// Minimal DES: a sorted future-event list, advancing straight to the
// next event's timestamp rather than stepping through fixed dt ticks.
class DiscreteEventSim {
  constructor() {
    this.events = []; // [{ time, type, payload }]
    this.clock = 0;
  }

  schedule(time, type, payload) {
    this.events.push({ time, type, payload });
    this.events.sort((a, b) => a.time - b.time); // priority queue in spirit
  }

  run(handlers) {
    while (this.events.length > 0) {
      const event = this.events.shift();
      this.clock = event.time; // jump straight to the next event
      handlers[event.type]?.(event.payload, this);
    }
  }
}

// Generate Poisson-ish passenger arrivals: exponential inter-arrival
// times give a mean arrival rate of `lambda` per unit time.
function scheduleArrivals(sim, lambda, untilTime) {
  let t = 0;
  while (t < untilTime) {
    t += -Math.log(Math.random()) / lambda; // exponential interarrival
    sim.schedule(t, "passenger_arrival", { requestedAt: t });
  }
}

const sim = new DiscreteEventSim();
const waitTimes = [];
scheduleArrivals(sim, /* lambda */ 2.5, /* untilTime */ 60);
sim.run({
  passenger_arrival: (payload, s) => {
    // Feed this into your real dispatch logic (SCAN/LOOK/greedy) and,
    // when the passenger is actually served, record:
    // waitTimes.push(servedAt - payload.requestedAt);
  },
});
```

Exercise: run this harness at a few values of λ against both a greedy-dispatch and a LOOK-based scheduler implementation, plot (or just log) mean and p95 wait time at each λ, and find the λ where each scheduler's p95 wait time starts climbing sharply — that's your empirical ρ-approaching-1 breaking point, and comparing where it happens for each algorithm is direct, concrete evidence for the throughput-vs-fairness tradeoff discussed in the Scheduling Algorithms note.

## Exam Tips

- If a system-design or simulation question mentions "the queue keeps growing" or "requests are timing out under load," check for ρ ≥ 1 first (arrival rate outpacing service rate) before assuming the scheduling *algorithm* is the bug — no algorithm fixes an under-provisioned system.
- Know the difference between *average* wait time and *tail* (p95/p99) wait time, and be ready to say why a scheduler can look great on the former while starving a minority of requests on the latter (ties directly back to greedy-dispatch starvation).
- Be able to explain discrete-event simulation's core efficiency argument in one sentence: simulated time jumps straight to the next meaningful event instead of stepping through every small time increment, so simulation cost scales with the number of *events*, not the length of *time* simulated.
- A frequent trap: assuming steady-state/average-case queueing math (like closed-form results for simple queue models) applies directly to bursty, non-Poisson real traffic. A strong answer flags that closed-form formulas assume specific arrival distributions and that empirical simulation is how you validate a scheduler against real (bursty, non-uniform) load.
- Connect utilization explicitly to elevator-bank sizing: "how many elevators do we need" is a queueing-theory capacity question (get ρ comfortably below 1 for the peak arrival rate you must support), separate from "how should each elevator behave" (the scheduling-algorithm question).

## References
- [Role of Queues in SimEvents Models - MathWorks](https://www.mathworks.com/help/simevents/gs/role-of-queues-in-simevents-models.html)
- [Queueing Theory - Simio textbook](https://textbook.simio.com/SASMAA7/ch-queueing.html)
- [Elevator Saga](https://play.elevatorsaga.com/)

## Related
- Parent: [[00 - Simulation & Scheduling (Overview)]]
- Siblings: [[01 - State Machines & Event-Driven Simulation]], [[02 - Scheduling Algorithms (SCAN, LOOK & Greedy Dispatch)]]
