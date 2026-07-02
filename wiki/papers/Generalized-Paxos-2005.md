---
type: paper
title: Generalized Consensus and Paxos
authors: Leslie Lamport
year: 2005
venue: Microsoft Research Technical Report MSR-TR-2005-33
source: raw/gpaxos.pdf
protocols: [Generalized Paxos]
tags: [paxos, fast-consensus, generalized-consensus, quorum, command-structure]
status: ingested
---

# Generalized Consensus and Paxos

## One-sentence summary
Generalized Paxos extends Paxos from choosing one value or one total command sequence to learning a growing [[command-structure]] so non-interfering concurrent commands can be learned in two message delays.

## Why this paper matters
The paper gives the theoretical basis for later dependency and commutativity-based SMR protocols: it separates agreement on command histories from agreement on a single total order, and it states the quorum intersections needed for fast ballots over generalized values.

## System model
Asynchronous message-passing system with proposers, acceptors, learners, and possible leaders. Messages can be lost but not corrupted. Processes may stop and do nothing, but do not execute the algorithm incorrectly.

## Fault model
Non-Byzantine crash/stop failures. Safety is required without timing assumptions or extra failure assumptions. Liveness requires enough nonfaulty processes and eventually delivered messages.

## Timing assumptions
Safety is asynchronous. Liveness for classic ballots follows ordinary Paxos-style assumptions: eventually one nonfaulty leader uses a sufficiently large classic ballot, communicates with an `m`-quorum, and the relevant proposer/learner messages are delivered.

## Main idea
Instead of choosing a sequence of commands, GPaxos chooses monotonically growing c-structs. A c-struct can represent a command history in which non-interfering commands are unordered, so different learners may learn compatible extensions rather than identical total prefixes.

## Protocol roles
- Proposers issue commands through `SendProposal(C)`.
- Acceptors vote in numbered ballots and move only to higher ballot numbers.
- Leaders run phase 1 and suggest safe c-structs in classic voting.
- Learners learn c-structs chosen by quorum evidence and update their learned value by lub.

## Message types
- `propose`: `("propose", C)` from proposer to leaders and/or acceptors.
- Phase 1a: `("1a", m)` from ballot leader to acceptors.
- Phase 1b: `("1b", m, a, bA_a)` in the abstract distributed algorithm, optimized to `("1b", m, a, bal[a], val[a])`.
- Phase 2a: `("2a", m, v)` from leader to acceptors.
- Phase 2b: `("2b", m, a, v)` from acceptor to learners.

## Local state
The abstract algorithm uses `learned`, `propCmd`, ballot array `bA`, and `minTried`/`maxTried`. The optimized algorithm keeps only needed data: leaders keep `maxStarted` and `maxVal`; acceptors keep `mbal[a]`, `bal[a]`, and `val[a]`.

## Normal path
A newly selected leader chooses a higher ballot number, sends phase 1a to an `m`-quorum, receives phase 1b evidence, computes a safe c-struct using `ProvedSafe(Q, m, beta)`, starts phase 2 with phase 2a messages, and then proceeds in either a classic or fast ballot.

## Fast path
If the leader chooses a fast ballot, proposers send proposals directly to an `m`-quorum of acceptors. Acceptors append proposed commands by `Phase2bFast` and send phase 2b messages to learners. A learner can learn a c-struct containing command `C` in two message delays when it receives compatible c-structs containing `C` from an `m`-quorum.

## Slow path
In a classic ballot, proposers send proposals to the leader. The leader extends `maxTried[m]` and sends phase 2a messages; acceptors execute `Phase2bClassic`; learners learn after receiving phase 2b messages from an `m`-quorum. The paper states this path takes three message delays from proposal to learning.

## Recovery path
Recovery uses a higher-numbered ballot. The leader gathers phase 1b evidence from an `m`-quorum and must choose a c-struct in `ProvedSafe(Q, m, beta)`. For fast-ballot collisions, the leader can start a higher fast ballot and send a phase 2a c-struct that orders the conflicting commands, after which acceptors vote classically and then resume fast operation.

## Commit rule
A c-struct `v` is chosen at ballot number `m` iff there exists an `m`-quorum `Q` such that `v <= beta_a[m]` for every acceptor `a` in `Q`. Learners learn `v` when they receive phase 2b evidence from an `m`-quorum with each reported c-struct extending `v`.

## Quorum system
For all ballot numbers `k` and `m`:
- Any `k`-quorum and any `m`-quorum intersect.
- If `k` is a fast ballot number, then any two `k`-quorums and any `m`-quorum have non-empty triple intersection: `Q1 ∩ Q2 ∩ R != {}` for all `Q1, Q2 in Quorum(k)` and `R in Quorum(m)`.

The implementation section gives two common quorum choices for `N` acceptors:
- Classic and fast quorums both of size at least `floor(2N/3) + 1`.
- Classic quorums of size at least `floor(N/2) + 1` and fast quorums of size at least `ceil(3N/4)`.

## Conflict handling
For command histories, concurrent non-interfering commands remain compatible and can be learned without imposing a total order. Interfering concurrent commands can produce incompatible c-structs such as `w * C * D` and `w * D * C`; then no c-struct containing either command is chosen in that ballot, and a leader resolves the collision in a higher ballot.

## Safety argument
The core invariant is that acceptors vote only for c-structs safe at their ballot number. Safe means every lower-ballot c-struct that is still choosable is a prefix of the value being voted for. If the ballot array is safe, all chosen values are compatible, giving generalized consistency.

## Liveness argument
Classic-ballot liveness is essentially ordinary Paxos liveness under an eventually unique nonfaulty leader and eventually delivered messages with a quorum. Fast-ballot liveness requires the leader to observe incompatible phase 2b evidence when a collision occurs, start a higher ballot, and get the conflicting commands chosen.

## Key proof ideas
- Generalized consensus replaces equality/total-prefix agreement with compatibility of learned c-structs.
- `ProvedSafe(Q, m, beta)` computes safe values using only quorum evidence from phase 1b replies.
- Conservative ballot arrays ensure `ProvedSafe` is non-empty even after classic ballots.
- The abstract algorithm proves invariants over `minTried`, `maxTried`, `bA`, and `learned`; the distributed and optimized algorithms refine it.

## Important formulas
- Prefix relation: `v <= w` iff `w = v * sigma` for some command sequence `sigma`.
- Compatibility: `v` and `w` are compatible iff they have a common upper bound.
- Chosen at `m`: exists `Q in Quorum(m)` such that `v <= beta_a[m]` for all `a in Q`.
- Fast quorum assumption: `Q1 ∩ Q2 ∩ R != {}` for any two fast `k`-quorums and any `m`-quorum.
- Example quorum sizing: both quorum types `floor(2N/3) + 1`, or classic `floor(N/2) + 1` with fast `ceil(3N/4)`.

## Relationship to other protocols
[[GPaxos]] generalizes [[FastPaxos]] by letting fast ballots choose compatible command structures rather than requiring the same value. [[EPaxos]], [[Atlas]], and [[SwiftPaxos]] can be read as later SMR protocols that make command dependencies explicit, though their proof and recovery rules are protocol-specific and should not be copied from GPaxos without justification.

## Limitations
Arbitrary c-structs can be large; practical implementations need checkpointing and compact representation. Fast progress still needs compatible fast-quorum evidence, so interfering concurrent commands trigger leader recovery.

## Open questions
- TODO: Add a dedicated [[command-structure]] concept page if more GPaxos-derived pages use the c-struct algebra.
- TODO: Extract exact TLA+ module names and invariant statements from Appendix C if formal-model notes are expanded.

## Related pages
[[GPaxos]], [[FastPaxos]], [[quorum]], [[fast-path]], [[conflict]], [[dependency]], [[recovery]], [[quorum-intersection]], [[fast-consensus]], [[paxos-family]]
