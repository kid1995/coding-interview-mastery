---
title: Simulation & Scheduling
mindmap_id: simulation-scheduling
node_type: category
parent: "[[00 - Coding Interview Mastery Mind Map (Index)]]"
children: ["[[01 - State Machines & Event-Driven Simulation]]", "[[02 - Scheduling Algorithms (SCAN, LOOK & Greedy Dispatch)]]", "[[03 - Queueing Theory & Discrete-Event Simulation]]"]
tags: [coding-interview, category]
source: "coding-interview-mastery vault"
created: 2026-08-03
---

# Simulation & Scheduling

> The theory behind real-time dispatch/scheduling problems like Elevator Saga — modeling state, making dispatch decisions under uncertainty, and reasoning about queues over time.

## Overview

Most LeetCode-style problems ask you to transform a static input into a static output. This category is different: it covers problems where a program has to keep making decisions over *time*, in response to *events it doesn't fully control*, while other events keep arriving. The canonical example — and the one this whole folder is designed around — is [Elevator Saga](https://play.elevatorsaga.com/), a browser game where you write a JavaScript controller for a bank of elevators and watch it live or die against passenger demand you can't predict. It looks like a toy, but it's a compressed version of real systems: ride-hailing dispatch, warehouse robot routing, disk-arm scheduling, load balancers, and job schedulers all share the same bones — a controller, a queue of requests, and a decision loop that has to be both correct and fast under load.

The three topics here build on each other. First you need a way to *represent* the controller itself — not as a tangle of `if` statements, but as an explicit set of states and transitions (state machines), driven by events rather than by polling in a tight loop (event-driven design). Once you can represent the controller cleanly, you need *decision logic* for what it should actually do when multiple requests compete for its attention — this is where SCAN, LOOK, and greedy dispatch strategies come in, along with the central tension between throughput and fairness. Finally, you need a *lens* to evaluate whether your decisions are actually good once real load (many passengers, many floors, bursty arrivals) hits the system — that's queueing theory and discrete-event simulation, which give you the vocabulary (arrival rate, service rate, utilization, wait time) to reason about a system quantitatively instead of just eyeballing it.

None of this is really about elevators. It's about writing resilient, testable code for anything that has to react to a stream of events over time and make good-enough decisions fast. If you can model an elevator controller cleanly, you can model a task queue, a matchmaking system, or a rate limiter using the exact same tools.

## Topics in This Category

- [[01 - State Machines & Event-Driven Simulation]] — how to represent a real-time controller as explicit states, transitions, and event handlers instead of ad-hoc conditional logic.
- [[02 - Scheduling Algorithms (SCAN, LOOK & Greedy Dispatch)]] — the core decision algorithms (SCAN, LOOK, greedy nearest-request) for choosing what a dispatcher does next, and the throughput-vs-fairness tradeoffs between them.
- [[03 - Queueing Theory & Discrete-Event Simulation]] — the math and simulation technique for reasoning about queues, wait times, and utilization once real load hits your scheduler.

## How These Topics Fit Together

Think of it as three layers stacked on top of each other. **State machines** give you the *structure*: a clean, testable representation of "what state is this elevator/worker/connection in right now, and what happens when event X arrives." Without this layer, real-time controllers degenerate into a pile of mutable booleans and nested conditionals that are nearly impossible to reason about or unit-test. **Scheduling algorithms** give you the *decision logic* that lives inside that structure: when the state machine is in an "idle, choosing next target" state, which algorithm picks the target? This is where SCAN-style discipline, LOOK's early reversal, and greedy nearest-request dispatch come in, each with different throughput/fairness/complexity tradeoffs. **Queueing theory** gives you the *evaluation lens*: once your state machine and scheduling algorithm are wired together and running against a stream of arrivals, how do you know if it's actually good? Queueing theory's arrival rate (λ), service rate (μ), and utilization (ρ = λ/μ) let you predict — and discrete-event simulation lets you empirically verify — whether your design will keep up under load or fall over.

In an Elevator Saga run, all three show up simultaneously: each elevator is a state machine, your `update()` callback runs a scheduling algorithm to pick the next floor, and the challenge's scoring (transported count vs. elapsed time) is literally a queueing-theory throughput metric in disguise.

## References
- [Elevator Saga](https://play.elevatorsaga.com/) — the practical exercise this category prepares you for
- [Elevator algorithm — Wikipedia](https://en.wikipedia.org/wiki/Elevator_algorithm) — origin and definition of SCAN-style scheduling
