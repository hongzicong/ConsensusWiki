---
type: protocol
name: SwiftPaxos
family: paxos / dependency-based SMR
papers: [SwiftPaxos-2024]
tags: [consensus]
---

# SwiftPaxos

## Short description
Leader/ballot protocol with leader-including fast quorums, FastAck/SlowAck evidence, and acyclic dependencies.

## Problem solved
Agreement/replication with lower wide-area latency than a simple single-leader design.

## System model
See primary paper page [[SwiftPaxos-2024]]; this page records protocol-level facts only.

## Fault model
Non-Byzantine faults.

## Timing assumptions
Safety is asynchronous; liveness depends on the progress assumptions of the source paper.

## Roles
- Client: submits a command with a unique identifier by broadcasting `Propagate(c)` to all replicas. A co-located client can receive the result from its local replica after execution; a non-co-located client can accept the leader's optimistic `Reply` once quorum evidence confirms the ordering.
- Leader: fixed per ballot as `leader(b)`. The leader belongs to every fast and slow quorum, computes its own dependency proposal, broadcasts `FastAck`, and may optimistically execute pending commands to send `Reply`.
- Fast-quorum replica: computes a local dependency proposal from previously received conflicting commands and broadcasts `FastAck`.
- Slow-quorum replica: does not need to make its own proposal; after receiving the leader's `FastAck`, it adopts the leader's dependency proposal and broadcasts `SlowAck`.

## Message types
Normal-operation messages:

- `Propagate(c)`: client-to-replicas command submission.
- `FastAck(b, id, D, P)`: replica proposal/acknowledgement for command `id` in ballot `b`, carrying direct dependencies `D` and dependency paths `P`.
- `SlowAck(b, id)`: acknowledgement that the sender is in sync with the leader's proposal for `id`.
- `Reply(b, id, P, r)`: leader-to-client optimistic response, carrying dependency paths `P` and result `r`.

Recovery messages include `NewLeader`, `NewLeaderAck`, and `Sync`.

## Local state
See [[SwiftPaxos-2024]].

## Normal path
1. The client broadcasts `Propagate(c)` to all replicas.
2. A receiving replica records `c` in `PREACCEPT`.
3. If the replica is in a fast quorum for the current ballot, it computes dependencies `D` from locally known conflicting commands and dependency paths `P`, then broadcasts `FastAck(b, id(c), D, P)`.
4. If the receiving replica is the leader, it also appends `c` to `pending_log`, optimistically computes a result, sends `Reply(b, id(c), P, r)` to the client, and broadcasts its `FastAck` to replicas.

## Fast path
The fast path happens when a fast quorum, which always includes the leader, converges on the same dependency proposal.

Message flow:

1. `client -> all replicas`: `Propagate(c)`.
2. `fast-quorum replicas -> replicas/client`: `FastAck(b, id, D, P)`. The leader's `FastAck` is sent to replicas; follower fast-quorum replicas also send it to the submitting client.
3. `leader -> client`: `Reply(b, id, P, r)` for optimistic execution.
4. A replica commits after it has processed the leader's `FastAck`, then receives matching `FastAck` evidence from all followers in some fast quorum and `D = dep[id]` is already committed.
5. A non-co-located client accepts the leader's `Reply` after receiving matching dependency paths from the followers of a fast quorum.

In the stable, contention-free case this gives two message delays from submission: one delay for `Propagate`, one delay for matching `FastAck`/`Reply` evidence.

Read-only optimization: the speculative execution step does not have to run at the leader. A read-only command can be tentatively executed at any fast-quorum replica, and the client accepts that tentative result only when the fast-quorum dependency-path evidence matches. This reduces leader load for read-heavy workloads while keeping the command ordered through the same dependency evidence.

## Slow path
SwiftPaxos has two slow-path commit shapes.

Double voting inside a fast quorum:

1. The client still starts with `Propagate(c)`.
2. Fast-quorum replicas may initially send different `FastAck` proposals because they received conflicting commands in different orders.
3. When a fast-quorum replica receives the leader's `FastAck(b, id, D, P)`, it compares `D` with its own `dep[id]`.
4. If it disagrees, it overwrites/adopts the leader's dependency proposal, moves the command to `ACCEPT`, and broadcasts `SlowAck(b, id)` to replicas and the client.
5. A replica can commit once it has matching leader proposal evidence from all members of a fast quorum, where each follower either sent a matching `FastAck` or corrected itself with `SlowAck`, and the dependencies are committed.

Slow quorum fallback:

1. A slow-quorum follower receives the leader's `FastAck`.
2. It adopts the leader's dependency proposal and broadcasts `SlowAck(b, id)`.
3. Any replica can commit after receiving the leader's `FastAck` plus `SlowAck`s from the followers of some slow quorum.
4. A non-co-located client can accept the leader's `Reply` after receiving the corresponding slow-quorum `SlowAck` evidence.

The key safety reason double voting is allowed is that the leader is in every fast quorum. If a fast-quorum replica's proposal differs from the leader's, then no fast quorum could already have unanimously fast-committed that different proposal.

## Recovery
See [[SwiftPaxos-2024]].

## Commit condition
See [[SwiftPaxos-2024]].

## Quorum requirement
See [[SwiftPaxos-2024]] and [[quorum-systems]].

## Safety intuition
Quorum intersection prevents incompatible committed outcomes; dependency-based protocols additionally prove compatible execution order.

## Liveness intuition
Requires enough nonfaulty replicas/data sites and eventual stable recovery/leadership where applicable.

## Strengths
Lower latency in its target common case.
Read-only commands can be speculatively served by nearby fast-quorum replicas, reducing leader saturation in read-heavy workloads.

## Weaknesses
More complex recovery and quorum reasoning than classic Paxos.

## Differences from related protocols
See [[protocol-catalog]] and dimension pages.

## Modeling notes
Start from abstract state, message evidence, quorum assumptions, and commit predicate before optimizing the real protocol.

## Open questions
- TODO: Expand this protocol page with examples after more query-driven use.

## Sources
- [[SwiftPaxos-2024]]
