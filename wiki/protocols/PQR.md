---
type: protocol
name: PQR
family: Paxos / client-driven quorum-read optimization
papers: [Linearizable-Quorum-Reads-2019]
tags: [paxos, quorum-read, rinse, linearizability]
---

# PQR

## Short description

Paxos Quorum Reads discover the latest accepted slot from followers and rinse a dirty observation by waiting for execution at one replica.

## Problem solved

Offload leader reads without leases. A committed-only quorum can be stale; accepted-only reads can expose a value that is not yet chosen.

## System model

Multi-Paxos log with ordered execution, clients coordinating reads, follower accepted/executed progress, optional per-key frontiers. Highest accepted slot can exceed gaps; execution cannot skip unresolved prior slots.

## Fault model

Non-Byzantine Paxos crashes/nonresponse. Majority host tolerance is inherited, but excluding the leader can reduce discovery availability. Separate PQR crash/restart handling is not fully specified.

## Timing assumptions

Lease-free read safety; no synchronized clocks or drift bound. Progress needs responsive read quorums, host Paxos recovery, and eventual selected-prefix execution.

## Roles

Client read coordinator; follower progress/data responder and rinse target; unchanged write leader.

## Message types

Read query/progress-and-data reply; single-node execution query/confirmation/retry; unchanged host propose/promise, accept/accepted, commit. The paper does not name concrete PQR wire structs.

## Local state

Host log/ballots and accepted, committed, executed state; read-facing highest accepted/executed slots globally or per object; client quorum reports and selected frontier.

## Normal path

Client queries a quorum of followers, selects maximum accepted slot, waits for that slot to execute at any one follower, and returns execution-justified data. Writes use unchanged Multi-Paxos.

## Fast path

One client-to-quorum RTT when first-round accepted/executed progress is clean and already establishes rinse evidence. Per-object progress avoids waiting on unrelated writes. No leader, lease, identical-value voting, or special fast write quorum is required.

## Slow path

Dirty reports trigger a separate rinse query; retry until execution. Two RTTs is common, not a worst-case guarantee. No client read-repair write is added.

## Recovery

Host Paxos selects safe values and resolves gaps. PQR waits for executed state rather than choosing a recovery value. The PDF leaves rinse-target failover, restart/epoch fencing, and binding provisional data to a value chosen after recovery underspecified.

## Commit condition

Write commit remains host Phase 2 quorum acceptance. Read completion requires discovery intersecting all possible write quorums plus execution confirmation for the selected frontier; clean first-round evidence can provide that confirmation. Do not return an unchosen accepted value.

## Quorum requirement

- Total `n`/`N`; standard host majority `⌊n/2⌋ + 1`; derived host crash budget `n - (⌊n/2⌋ + 1)`.
- Standard read: `⌊n/2⌋ + 1` total-membership replicas, drawn from followers in the intended path; paper wording is ambiguous about the denominator.
- Rinse: one replica, after discovery; recovery: unchanged host Phase 1 quorum.
- No distinct fast write quorum; rinse-free reads use the same discovery quorum.
- Flexible extension: read quorum intersects every `Q2`; host separately preserves `Q1`/`Q2` intersection. Derived threshold rule `q_read + q_write > N`.
- Paper examples: `(N,Q2,q_read) = (5,4,2)` and `(7,5,3)`.
- Sets may be flexibly selected subject to these intersections; leader inclusion is avoided on reads, not required. Host writes retain leader coordination.

## Safety intuition

Discovery intersects every earlier completed write's acceptance quorum, revealing at least its slot. Rinse makes the observed frontier a chosen executed prefix before return. Committed-only discovery can miss a write; accepted-only return can expose a provisional value. Full recovery/data-binding proof remains TODO in the source extraction.

## Liveness intuition

Read completion follows when a discovery quorum responds and one replica eventually executes the selected prefix. The read path depends on host write recovery without involving the leader directly in each read. A follower-only majority may be unavailable while a leader-including write majority survives.

## Strengths

No lease clock assumptions; unchanged writes; leader offload; clean reads finish in one RTT; flexible larger writes allow smaller read quorums.

## Weaknesses

Read fanout/client work; dirty reads may repeatedly wait; host gaps/recovery delay execution; follower-only availability caveat; incomplete algorithm/proof details in the short paper.

## Differences from related protocols

[[PQL]] and [[Bodega]] make writes cover authorized local readers. PQR instead charges discovery work to reads and leaves writes unchanged. Its one-replica rinse is an execution certificate after quorum discovery, not permission for arbitrary one-replica reads or single-replica recovery.

## Open questions

Read-value binding after overwrite; per-key/multi-key semantics; leader-exclusion fallback; restart/reconfiguration/compaction fencing. See [[Linearizable-Quorum-Reads-2019]] and [[unresolved-confusions]].

## Sources

[[Linearizable-Quorum-Reads-2019]] - `raw/pqr.pdf`, §§3-6, Figure 2 and Table 1.
