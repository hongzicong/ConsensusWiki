---
type: protocol
name: HydraPaxos
family: network-ordered Paxos / SMR
papers: [Hydra-2023]
tags: [smr, network-ordering, crash-fault, one-rtt, nopaxos]
---

# HydraPaxos

## Short description

HydraPaxos is a NOPaxos-derived crash-tolerant SMR protocol that uses [[Hydra]] groupcast for common-order delivery and application-level consensus only for durability and dropped positions.

## Problem solved

It preserves NOPaxos's one-round-trip normal path while removing dependence on one network sequencer and shortening sequencer failover.

## System model

`2f + 1` replicas run a deterministic state machine and form one Hydra receiver group. Clients groupcast operations; every replica logs Hydra deliveries, while one leader executes and replies.

## Fault model

Up to `f` crash failures among `2f + 1` replicas. Hydra handles packet loss and sequencer/link failures through ordered drop notification and sequencer reconfiguration. Byzantine behavior is excluded.

## Timing assumptions

Safety inherits Hydra's non-retrograde clock assumption and Paxos/NOPaxos quorum reasoning. Progress requires a live replica majority, an available leader, Hydra progress, and completion of drop recovery after loss.

## Roles

Client, leader replica, follower replica, Hydra sequencers/receivers, and Hydra configuration service.

## Message types

Hydra groupcast operation, replica reply, `DROP-NOTIFICATION`, missing-message query/response, consensus messages for committing a missing position as `NO-OP`, and Hydra reconfiguration traffic.

## Local state

Hydra receiver frontier/buffer plus an ordered operation log, deterministic application state, leader role, client deduplication/reply state inherited from NOPaxos, and drop-recovery state.

## Normal path

Hydra delivers an operation in common order. Every replica appends it; the leader executes it. The client commits after receiving consistent replies from a majority that includes the leader.

## Fast path

One client round trip in the no-drop, stable-configuration case. The client majority supplies durability while Hydra supplies order.

## Slow path

A drop notification pauses the affected ordered position. Replicas first try to recover the missing operation from peers; if unavailable, the leader drives agreement to fill the position with `NO-OP`.

## Recovery

Any replica detecting the gap queries other replicas. The safe result is either the actual groupcast message, if some replica retained it, or a consensus-chosen `NO-OP`. Sequencer failure is recovered by Hydra's configuration protocol rather than NOPaxos's original sequencer-failure procedure.

## Commit condition

A client treats an operation as committed after consistent replies from `f + 1` replicas, including the leader. A missing position is skipped only after the replicas agree to commit `NO-OP`.

## Quorum requirement

- Total replicas: `n = 2f + 1`.
- Normal commit: majority `f + 1`, including the leader.
- Recovery: peer evidence followed by leader-driven consensus when no copy is recovered.
- Unclear: the Hydra paper does not restate the exact NOPaxos recovery-evidence quorum or message predicate.

## Safety intuition

Hydra gives every replica the same order or an ordered indication of a missing position. Majority replies make a completed operation survive `f` crashes. Agreement on `NO-OP` prevents replicas from inconsistently executing a late or partially delivered message.

## Liveness intuition

The normal path needs a live majority and leader. Drops add a recovery/consensus round, while failed sequencers add Hydra removal latency; remaining sequencers resume without waiting for network rerouting.

## Strengths

- One-round-trip normal operation.
- `f`-crash tolerance with `2f + 1` replicas.
- Multiple active sequencers and routing diversity.
- Explicit missing-position resolution.

## Weaknesses

- Requires deterministic execution and Hydra-capable networking/host libraries.
- Drops force recovery and possibly consensus on `NO-OP`.
- Depends on Hydra clock and configuration assumptions.
- Full inherited NOPaxos recovery details are not restated in the Hydra paper.

## Differences from related protocols

Unlike [[Hydra]], HydraPaxos commits replicated state-machine operations. Unlike ordinary Multi-Paxos, network ordering removes leader serialization from request ordering, although the leader still executes and must be present in the reply quorum.

## Open questions

- Ingest NOPaxos to make the inherited recovery proof and durable-state rules explicit.
- Specify the exact recovery quorum and late-message fencing rule.
- Quantify the safety boundary between Hydra group delivery and replica commit.

## Sources

- [[Hydra-2023]] - §6 for HydraPaxos and Appendix B for Hydra ordering/drop safety.
