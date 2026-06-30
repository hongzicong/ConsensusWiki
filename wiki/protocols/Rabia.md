---
type: protocol
name: Rabia
family: randomized leaderless SMR
papers: [Rabia-2021]
tags: [smr, randomized-consensus, weak-validity, leaderless]
---

# Rabia

## Short description
Leaderless SMR protocol that uses randomized weak multi-valued consensus, allowing `⊥` slots to avoid separate fail-over and recovery machinery.

## Problem solved
Total ordering of client requests for datacenter state-machine replication, with a design intended to simplify log compaction, snapshotting, reconfiguration, and failure handling.

## System model
Asynchronous message-passing replicas maintain a log. Clients send requests to proxy replicas, which forward requests to all replicas. Replicas agree slot-by-slot using Weak-MVC.

## Fault model
Fail-stop crash faults; at most `f` replicas may fail; `n ≥ 2f + 1`. Byzantine faults are outside the model.

## Timing assumptions
Safety is asynchronous. Liveness is probabilistic with a non-faulty majority. High performance assumes a stable datacenter network where replicas usually observe the same oldest pending request.

## Roles
Clients, proxy replicas, and replicas. There is no distinguished leader or command leader.

## Message types
`Proposal`, `State`, and `vote` inside Weak-MVC; request forwarding and catch-up messages outside the core consensus instance.

## Local state
Priority queue `PQ_i`, current slot `seq`, log entries, dictionary for already logged requests, and per-Weak-MVC `state`, `vote`, and phase counter `p`.

## Normal path
For each slot, propose the oldest request not already in the log, run Weak-MVC, write the result to the log, requeue the proposal if the output is `⊥` or another request, then advance to the next slot.

## Fast path
Three message delays: exchange proposals, exchange binary states, exchange votes. It returns a real request if a majority proposal is found and binary consensus decides `1`; it returns `⊥` if the binary decision is `0`.

## Slow path
Additional randomized binary-consensus phases. If no non-`?` vote is observed, replicas use `CommonCoinFlip(p)` to choose the next state.

## Recovery
No separate fail-over protocol. Decision evidence is carried by the Weak-MVC phase structure: once a value is decided, later phases are value-locked on that value for non-decided replicas. Slow replicas can ask peers for decided proposals.

## Commit condition
Weak-MVC returns for a slot. For binary value `1`, the slot contains the proposal that appeared in at least `⌊n/2⌋ + 1` Phase 0 Proposal messages. For binary value `0`, the slot contains `⊥`.

## Quorum requirement
With `n ≥ 2f + 1`, each communication step waits for at least `n - f` messages. Proposal/state majority threshold is `⌊n/2⌋ + 1`; decision requires a non-`?` value in at least `f + 1` vote messages.

## Safety intuition
Majority proposal evidence prevents two different concrete requests from both being returned for the same slot. Binary agreement plus value-locking prevents different replicas from deciding different binary outcomes.

## Liveness intuition
Each binary-consensus phase has probability at least `1/2` of leading to termination, so each slot terminates with probability 1 when a majority is alive.

## Strengths
- No leader fail-over protocol.
- Simple log compaction and reconfiguration story compared with leader-based SMR.
- Fast path is common in stable datacenter networks.
- Machine-checked safety proof outline using Ivy and Coq.

## Weaknesses
- `O(n^2)` communication.
- Three message delays on the fast path, compared with two for Multi-Paxos/EPaxos in favorable cases.
- Performance depends on network stability.
- Weak validity allows `⊥` slots, so log density is workload/network dependent.

## Differences from related protocols
Unlike [[EPaxos]], [[Atlas]], and [[SwiftPaxos]], Rabia does not attach dependency metadata to commands. Unlike [[Mencius]], it does not allocate slots to rotating coordinators. Unlike [[FastPaxos]], it does not recover collisions through higher rounds; it can forfeit ambiguous slots and retry requests later.

## Open questions
- TODO: Model whether `⊥` slots should be treated as committed no-ops or as a separate weak-validity outcome in reusable proof abstractions.

## Sources
- [[Rabia-2021]]
