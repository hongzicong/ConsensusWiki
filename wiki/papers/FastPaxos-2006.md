---
type: paper
title: Fast Paxos
authors: Leslie Lamport
year: 2006
venue: Distributed Computing
source: raw/fastpaxos.pdf
protocols: [Fast Paxos]
tags: [paxos, fast-consensus, quorum, tla]
status: ingested
---

# Fast Paxos

## One-sentence summary
Fast Paxos extends classic Paxos so that, in collision-free executions, a proposed value can be learned in two message delays.

## Why this paper matters
It isolates the quorum-intersection condition needed for fast consensus and gives a formal TLA+ specification. This makes it a central source for [[fast-path]], [[quorum]], [[recovery]], and [[proof-techniques]].

## System model
Asynchronous message-passing system with proposers, acceptors, coordinators, and learners. Roles are logical: one process may play several roles.

## Fault model
Non-Byzantine faults. Safety must hold despite any number of failures; progress needs enough nonfaulty agents that can communicate.

## Timing assumptions
Safety is asynchronous. Progress assumes eventual leader behavior and a good set whose agents are nonfaulty and can communicate.

## Main idea
A coordinator may start a fast round by sending a phase 2a `any` message. Acceptors can then accept the first proposed value they receive directly, allowing learners to learn after proposer-to-acceptor and acceptor-to-learner delays.

## Protocol roles
Proposers propose values; acceptors choose by voting; coordinators start rounds and recover from collisions; learners learn chosen values.

## Message types
`phase1a`, `phase1b`, `phase2a`, `phase2b`, proposal messages, and special `phase2a any` messages in fast rounds.

## Local state
Acceptors track the highest promised round and accepted round/value. Coordinators track current round and chosen phase 2a value, including `any` or `none` in the TLA+ model.

## Normal path
In a classic round, the coordinator runs phase 1 then sends one value in phase 2a to a quorum. In a prepared fast round, the coordinator sends `any`; proposers send values directly to acceptors; acceptors vote for the first value accepted in that fast round.

## Fast path
A value is fast-chosen when a fast quorum votes for the same value in a fast round. In the absence of collision, learning takes two message delays.

## Slow path
If collision occurs or the fast path is not enabled, the system recovers using a classic round or a recovery round that selects a safe value using phase 1 evidence.

## Recovery path
Collision recovery can be coordinated, uncoordinated, or performed by starting a new higher-numbered round. The coordinator's phase 2a selection rule must preserve possible values from lower rounds.

## Commit rule
A value is chosen in round `i` iff an `i`-quorum of acceptors votes for it in that round.

## Quorum system
The paper's quorum requirement is:
- For any rounds `i` and `j`, any `i`-quorum and any `j`-quorum have non-empty intersection.
- If `j` is a fast round, then any `i`-quorum and any two `j`-quorums have non-empty intersection.

With `N` acceptors, classic quorums of size `N - F`, and fast quorums of size `N - E`, the requirements support progress when enough corresponding quorums are nonfaulty; the paper notes examples including `N > 3F` when `E = F`.

## Conflict handling
Competing proposals can collide. A fast consensus algorithm cannot always be fast under collision, so recovery chooses a safe value from phase 1 evidence.

## Safety argument
Safety generalizes classic Paxos: the phase 2a value-selection rule ensures that if any value may have been or may yet be chosen in a lower round, higher rounds can only choose a compatible value.

## Liveness argument
Progress is conditional on eventual stable leadership and communication among a good set. Frequent collisions can make classic Paxos preferable.

## Key proof ideas
The key invariant is that a higher round cannot choose a value different from a value that has been or might yet be chosen in a lower round. Fast rounds require triple-intersection-style quorum reasoning because two fast quorums may have voted for different values.

## Important formulas
- Chosen in round `i`: an `i`-quorum voted for the value.
- Quorum requirement: `(a)` any two quorums intersect; `(b)` if `j` is fast, any `i`-quorum and any two `j`-quorums intersect.
- TLA+ constants include `FastNum`, `Quorum(i)`, `Coord(i)`.

## Relationship to other protocols
[[Fast Paxos]] is a base for later fast-consensus and leaderless/near-leaderless systems such as [[EPaxos]], [[SwiftPaxos]], and [[Pando]].

## Modeling notes

### Rocq/Coq modeling notes
Model rounds, quorum families, accepted votes, and a `pickable` predicate for the phase 2a rule. The hardest obligation is the fast-round quorum intersection case.

### TLA+ modeling notes
The appendix is directly usable as a modeling reference; keep `any` and `none` as distinguished values outside `Val`.

## Limitations
Fast latency is not guaranteed under collisions. Implementation choices for recovery and message routing affect cost.

## Open questions
- TODO: Capture the exact size constraints for all `(N, F, E)` examples from Section 3.4 in a separate proof note.

## Related pages
[[Fast Paxos]], [[quorum]], [[fast-path]], [[recovery]], [[agreement]], [[quorum-intersection]]
