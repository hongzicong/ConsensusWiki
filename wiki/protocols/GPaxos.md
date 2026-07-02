---
type: protocol
name: Generalized Paxos
family: paxos / fast consensus / generalized consensus
papers: [Generalized-Paxos-2005]
tags: [consensus, paxos, fast-path, command-structure]
---

# GPaxos

## Short description
Generalized Paxos is Paxos over command structures: learners agree on compatible growing histories, allowing non-interfering concurrent commands to be learned on a fast path.

## Problem solved
State-machine replication when a total order is stronger than necessary because many concurrent commands commute or do not interfere.

## System model
Asynchronous message passing with proposers, acceptors, learners, and possible leaders.

## Fault model
Non-Byzantine crash/stop failures. Safety does not depend on timing.

## Timing assumptions
Classic liveness follows ordinary Paxos-style eventual leader and message-delivery assumptions. Fast liveness additionally requires detecting incompatible fast votes and recovering through a higher ballot.

## Roles
Proposers issue commands; acceptors vote in ballots; leaders run phase 1 and classic/recovery actions; learners learn quorum-chosen c-structs.

## Message types
`propose`, `1a`, `1b`, `2a`, and `2b`.

## Local state
Optimized acceptors keep `mbal`, `bal`, and `val`; leaders keep the largest started ballot and its max suggested c-struct; learners keep `learned`.

## Normal path
A leader starts a ballot by collecting phase 1b evidence from a quorum, computes a safe starting c-struct, sends phase 2a, and then either continues classic leader-mediated voting or enables fast proposer-to-acceptor voting.

## Fast path
In a fast ballot, proposers send commands directly to an `m`-quorum of acceptors. Acceptors append commands to their current c-structs and send phase 2b messages to learners. Compatible c-struct evidence from a quorum lets a learner learn in two message delays.

## Slow path
In a classic ballot, the leader receives proposals and sends extended c-structs in phase 2a messages; acceptors vote; learners learn from quorum phase 2b evidence.

## Recovery
A leader starts a higher ballot, collects phase 1b evidence, computes `ProvedSafe(Q, m, beta)`, and selects an extension safe with respect to lower-ballot choosable values. Fast collisions are resolved by ordering the conflicting commands in a higher ballot.

## Commit condition
A c-struct `v` is chosen at ballot `m` when all acceptors in some `m`-quorum have voted c-structs extending `v`.

## Quorum requirement
Any two quorums for any ballot numbers intersect. If `k` is fast, then any two `k`-quorums and any `m`-quorum have non-empty triple intersection. Common examples are both classic/fast quorums of size `floor(2N/3) + 1`, or classic `floor(N/2) + 1` with fast `ceil(3N/4)`.

## Safety intuition
Safe-value selection ensures that higher ballots extend every lower-ballot value that might still be chosen. Because chosen c-structs remain compatible, learners never learn incompatible histories.

## Liveness intuition
With a stable leader and a live quorum, classic ballots make Paxos-like progress. Fast ballots make two-delay progress while acceptors produce compatible c-structs; collisions require leader intervention.

## Strengths
- Captures commutativity directly instead of forcing an arbitrary total order.
- Provides two-message-delay learning for compatible concurrent commands.
- Gives a reusable quorum/safe-value framework for later fast SMR designs.

## Weaknesses
- C-struct algebra and `ProvedSafe` are harder to implement and model than single-value Paxos.
- Interfering concurrent commands still require collision recovery.
- Large command histories require compact representation and checkpointing.

## Differences from related protocols
Unlike [[FastPaxos]], GPaxos can fast-learn compatible but not identical command histories. Unlike [[EPaxos]], [[Atlas]], and [[SwiftPaxos]], the paper presents a general c-struct abstraction rather than a concrete dependency metadata protocol. Unlike [[Mencius]], it does not avoid contention by pre-assigning total-order slots to rotating leaders.

## Open questions
- TODO: Add a formal note for `ProvedSafe(Q, m, beta)` and conservative ballot arrays.

## Sources
- [[Generalized-Paxos-2005]]
