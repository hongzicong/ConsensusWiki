---
type: protocol
name: OmniPaxos
family: paxos
papers: [Omni-Paxos-2023]
tags: [smr, partial-connectivity, leader-election, sequence-consensus, reconfiguration]
---

# OmniPaxos

## Short description

OmniPaxos is a leader-based SMR system that combines quorum-connected Ballot Leader Election, prefix-preserving Sequence Paxos, and service-layer log migration.

## Problem solved

It preserves progress under partial network partitions that leave at least one server directly connected to a majority, including cases where the old leader remains alive, the only viable candidate has a stale log, or disconnected candidates repeatedly gossip higher terms.

## System model

Servers in one fixed configuration run BLE and Sequence Paxos. A cross-configuration service layer stores the replicated log and migrates decided entries. Communication uses bidirectional session-based FIFO perfect links.

## Fault model

Non-Byzantine fail-recovery servers with persistent storage; messages may be lost or delayed and individual links may be partitioned. The paper states majority requirements and does not define a separate `N,f` formula.

## Timing assumptions

Safety follows from ballots, log-prefix adoption, and majority intersection. Liveness requires partial synchrony, a converged heartbeat delay, one quorum-connected server, and available replication majorities.

## Roles

- client;
- Sequence Paxos leader;
- Sequence Paxos follower;
- BLE participant;
- cross-configuration service layer.

## Message types

- Sequence Paxos: `Prepare`, `Promise`, `AcceptSync`, `Accept`, `Accepted`, `Decide`, `PrepareReq`.
- BLE: `HBRequest`, `HBReply`, and the local `Leader` event.
- Reconfiguration: stop-sign log entry `SS`.

## Local state

Persistent replication state: `log[]`, `promisedRnd`, `acceptedRnd`, `decidedIdx`. Leader volatile state: `currentRnd`, `promises{}`, `maxProm`, `accepted[]`, `buffer[]`. BLE persists leader ballot `l` and tracks `r`, `b`, `qc`, `delay`, and `ballots{}`.

## Normal path

After Prepare, the leader appends each command and pipelines `Accept` messages. A majority of `Accepted` indices lets the leader advance `decidedIdx`; `Decide` informs followers and implicitly decides the preceding prefix.

## Fast path

No distinct fast quorum or conflict-free path. The stable leader decides in one replication round trip to a majority, but remains on every command path.

## Slow path

Leader election, crash recovery, or link reconnection triggers Prepare and log synchronization. The leader adopts the highest `acceptedRnd` log, breaking ties by longest `logIdx`, then sends `AcceptSync` before new accepts.

## Recovery

Any QC server may be elected even with a stale log. A majority of promises exposes every chosen prefix; the leader preserves that prefix and may overwrite only unchosen suffixes. A recovering/reconnected follower sends `PrepareReq` and resynchronizes with the current leader.

## Commit condition

An index is chosen after a majority has accepted it. The leader marks it decided after observing that majority. FIFO ordering means deciding one index decides all earlier indices. A chosen `SS` closes the current configuration.

## Quorum requirement

- Prepare, Accept, recovery, and BLE observation use majorities within a fixed configuration.
- The leader counts its own promise and acceptance.
- Majority intersection carries a chosen prefix into every later Prepare.
- BLE progress requires at least one QC server, defined as a correct server directly linked to at least a majority of correct servers, including itself.
- No fast or flexible quorum family is defined.

## Safety intuition

Sequence Paxos changes Paxos P2 from equal values to prefix-comparable sequences. Prepare adopts the highest-numbered accepted log in a majority, so every later proposal extends any earlier chosen sequence. Per-server integrity plus majority intersection gives uniform prefix agreement.

## Liveness intuition

BLE separates connectivity from log freshness. QC servers elect the highest QC ballot after stable heartbeat rounds; Sequence Paxos then repairs the elected server's log. Different QC majorities may temporarily elect different leaders, but an overlapping server promises the maximum ballot and stabilizes replication.

## Strengths

- Progress with one QC server under the modeled partial-connectivity scenarios.
- Stale but well-connected servers remain eligible leaders.
- One-round-trip pipelined stable replication.
- Parallel, leader-independent migration of decided log segments.
- Clear component boundaries for implementation and reasoning.

## Weaknesses

- Partial-synchrony and QC-existence requirements for progress.
- Non-Byzantine model and FIFO-session assumption.
- Heartbeat traffic and convergence are necessary even though measured overhead was small.
- Reconfiguration requires complete decided-state transfer before a new server starts.
- Majority quorums are fixed; the paper does not derive flexible or topology-aware quorum formulas.

## Differences from related protocols

- Unlike Multi-Paxos, the replicated object is a strictly growing sequence rather than independent slots.
- Unlike Raft/Zab, a leader need not already have the maximum log.
- Unlike VR/Zab, the elected leader need not be elected by a majority of QC servers.
- Unlike [[FPaxos]], the optimization is component decoupling and connectivity-aware election, not cross-phase quorum flexibility.
- Unlike [[FastPaxos]], clients do not bypass the leader through a separate fast ballot.

## Open questions

- Formalize end-to-end safety across BLE, Sequence Paxos, `SS`, and migration.
- Combine QC-aware election with snapshots, flexible quorums, or topology-aware priorities.
- Quantify behavior under half-duplex/asymmetric and rapidly changing connectivity.

## Sources

- [[Omni-Paxos-2023]] - primary paper, especially §§3-6 and Appendix A.
