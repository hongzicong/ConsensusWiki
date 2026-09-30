---
type: paper
title: Fast Paxos
authors: Leslie Lamport
year: 2006
venue: Distributed Computing
source: raw/fastpaxos.pdf
protocols: [Fast Paxos]
tags: [paxos, fast-consensus, quorum]
status: ingested
---

# Fast Paxos

## One-sentence summary
Fast Paxos extends classic Paxos so that, in collision-free executions, a proposed value can be learned in two message delays.

## Why this paper matters
It isolates the quorum-intersection condition needed for fast consensus. This makes it a central source for [[fast-path]], [[quorum]], [[recovery]], and [[proof-techniques]].

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
Acceptors track the highest promised round and accepted round/value. Coordinators track current round and chosen phase 2a value, including special `any` or `none` choices.

## Normal path
In a classic round, the coordinator runs phase 1 then sends one value in phase 2a to a quorum. In a prepared fast round, the coordinator sends `any`; proposers send values directly to acceptors; acceptors vote for the first value accepted in that fast round.

## Fast path
A value is fast-chosen when a fast quorum votes for the same value in a fast round. In the absence of collision, learning takes two message delays.

## Slow path
If collision occurs or the fast path is not enabled, the system recovers using a classic round or a recovery round that selects a safe value using phase 1 evidence.

## Recovery path
Collision recovery can be coordinated, uncoordinated, or performed by starting a new higher-numbered round. The coordinator's phase 2a selection rule must preserve possible values from lower rounds.

Figure 2 (§3.1, printed p. 20) gives the exact selection rule. Let `Q` be the new round's Phase 1 quorum, `k` the highest accepted round reported, and `V` the values reported at `k`. If `k = 0`, select any proposed value. If `V` is a singleton, retain that value. Otherwise retain the unique value `v` for which some old `k`-quorum `R` has every member of `R ∩ Q` reporting acceptance of `v` at `k` (`O4(v)`). If no value satisfies this predicate, select any proposed value. Arbitrary tie-breaking among replies is not a replacement for this rule.

§3.2 (printed pp. 19–21) describes coordinated and uncoordinated collision-recovery optimizations using old Phase 2b messages as Phase 1 evidence under their stated round conditions. These optimizations do not remove the general higher-round mechanism. §3.3 (printed pp. 21–23) obtains progress with a live classic quorum and eventual stable leadership by eventually starting a sufficiently high classic round. §3.4.1 explicitly permits switching to classic Paxos and executing Phase 1 for all instances when too few acceptors remain for fast rounds. The appendix specifies `Phase1a`, `Phase1b`, `Phase2a`, `Phase2b`, `IsPickableVal`, and both collision-recovery actions.

## Commit rule
A value is chosen in round `i` iff an `i`-quorum of acceptors votes for it in that round.

## Quorum system
The paper's quorum requirement is:
- For any rounds `i` and `j`, any `i`-quorum and any `j`-quorum have non-empty intersection.
- If `j` is a fast round, then any `i`-quorum and any two `j`-quorums have non-empty intersection.

With `N` acceptors, classic quorums of size `N - F`, and fast quorums of size `N - E`, the requirements support progress when enough corresponding quorums are nonfaulty; the paper notes examples including `N > 3F` when `E = F`.

For these cardinality-based families and `E ≤ F`, §3.4.1 gives exactly `N > 2F` and `N > 2E + F`. Maximizing classic fault tolerance gives `F = ceil(N/2) - 1` and `E = floor(N/4)`. Thus `(N, classic size, fast size)` is `(5,3,4)`, `(9,5,7)`, or `(13,7,10)`. These fast sizes apply when any subset of sufficient size is a quorum, not to every restricted quorum family.

Derived application of the paper's general intersection rule: a single fixed majority as the only fast quorum for a round can coexist with arbitrary classic majorities. The two old fast quorums in the triple-intersection requirement are then the same set. Losing one of its designated members disables that round's fast path, but does not itself preclude recovery through a classic majority and the Figure 2 selection rule. Changing or expanding the set of permitted fast quorums requires rechecking the intersection requirements and preserving evidence of the old round's quorum family.

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

## Relationship to other protocols
[[FastPaxos|Fast Paxos]] is a base for later fast-consensus and leaderless/near-leaderless systems such as [[EPaxos]], [[SwiftPaxos]], and [[Pando]].

## Limitations
Fast latency is not guaranteed under collisions. Implementation choices for recovery and message routing affect cost.

## Open questions
- A concrete SMR implementation must additionally specify slot recovery, command-data availability, execution deduplication, and client reply routing; the single-value consensus rules alone do not define those engineering details.

## Related pages
[[FastPaxos|Fast Paxos]], [[quorum]], [[fast-path]], [[recovery]], [[agreement]], [[quorum-intersection]]
