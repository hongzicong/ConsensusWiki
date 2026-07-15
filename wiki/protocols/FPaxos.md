---
type: protocol
name: FPaxos
family: Paxos / flexible quorums
papers: [Flexible-Paxos-2016]
tags: [paxos, quorum, flexible-quorum, recovery, agreement]
---

# FPaxos

## Short description

Flexible Paxos generalizes Paxos so Phase 1 quorums need intersect Phase 2 quorums, while quorums within the same phase may be disjoint.

## Problem solved

Majorities in both Paxos phases impose unnecessary steady-state latency, load, and topology constraints. FPaxos lets deployments shrink or reshape common Phase 2 quorums by paying a controlled Phase 1 recovery tradeoff.

## System model

Asynchronous proposer/acceptor/learner consensus with persistent promises and accepted values. Multi-Paxos aggregates Phase 1 to establish a stable leader and runs Phase 2 per log slot.

## Fault model

Non-Byzantine process failure and message loss. Safety is failure-independent; progress requires a live quorum for the current phase.

## Timing assumptions

Asynchronous safety; Paxos-style eventual synchrony/stable leadership for progress. Quorum targeting can reduce common latency but may retransmit after slow/failing members.

## Roles

Client, proposer/stable leader, acceptor, learner.

## Message types

`prepare`/`1a`, `promise`/`1b`, `propose`/`2a`, `accept`/`2b`, and host client/learn notifications.

## Local state

Highest promised ballot, highest accepted ballot/value, proposer ballot and Phase 1 replies, selected quorum families, per-slot Multi-Paxos state.

## Normal path

A stable leader sends each slot's value to one valid `Q2` and decides after every member accepts. Same-phase `Q2` quorums need not intersect.

## Fast path

No Fast Paxos-style path. FPaxos optimizes the ordinary stable-leader Phase 2 by changing quorum size/shape, not by bypassing the leader or a phase.

## Slow path

Leader election or recovery runs ordinary Phase 1 over a valid `Q1`, often larger than `Q2`, then resumes Phase 2.

## Recovery

Gather promises from `Q1`, preserve the value with the highest accepted ballot, and propose it through `Q2`. Cross-phase intersection exposes any earlier chosen value.

## Commit condition

One value is decided after all members of any valid `Q2 ∈ Quorum2` accept it at the proposal ballot.

## Quorum requirement

```text
∀Q1 ∈ Quorum1, ∀Q2 ∈ Quorum2: Q1 ∩ Q2 ≠ ∅
```

For simple threshold quorums, `|Q1| + |Q2| > N`; same-phase intersection is unnecessary.

## Safety intuition

Every later Phase 1 meets the Phase 2 evidence for an earlier decision. Their common acceptor either blocks the older proposal after a higher promise or reports the accepted value so the higher proposer adopts it.

## Liveness intuition

Current-leader progress needs a `Q2`; replacing the leader needs `Q1` and `Q2`. Shrinking `Q2` improves steady-state availability/latency but can reduce leader-recovery availability.

## Strengths

- Strictly generalizes ordinary Paxos quorum geometry.
- Smaller/faster steady-state Phase 2 quorums.
- Supports disjoint same-phase quorums and topology-aware systems.
- Minimal algorithm/code changes.
- Orthogonal to many Paxos variants.

## Weaknesses

- Availability becomes phase-specific and sometimes failure-placement-specific.
- Small `Q2` can make leader recovery require nearly all acceptors.
- Quorum selection and retransmission policy affect real performance.
- Dynamic quorum choice needs a separate safe announcement mechanism.
- No Byzantine treatment.

## Differences from related protocols

Unlike [[FastPaxos]], FPaxos changes classic Phase 1/Phase 2 quorum geometry without a leaderless fast round. Unlike [[GPaxos]], it chooses one value per instance. Unlike [[PigPaxos]], it changes which evidence is sufficient rather than how messages are aggregated.

## Open questions

- Which quorum families best match correlated failures and WAN latency?
- How should dynamic quorum versions be recovered across ballots?
- How should FPaxos compose with relay, lease, and fast-quorum optimizations?

## Sources

- [[Flexible-Paxos-2016]] - source paper, especially §§3-7 and the TLA+ appendix.
