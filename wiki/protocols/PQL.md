---
type: protocol
name: PQL
family: Paxos / quorum-lease read optimization
papers: [Paxos-Quorum-Leases-2014]
tags: [paxos, quorum-lease, local-read, strict-serializability]
---

# PQL

## Short description

Paxos Quorum Leases enable per-object local reads by requiring writes to synchronously notify every active holder before commitment.

## Problem solved

Improve read locality beyond one leader while avoiding write-all cost for every object.

## System model

`N` Multi-Paxos replicas, separately addressable objects, declared read/write sets, ordered log execution, asynchronous message loss/delay. Composite operations execute atomically.

## Fault model

Non-Byzantine crash/nonresponse. Majority availability needs `⌊N/2⌋ + 1` live replicas; an active failed holder may additionally block affected writes until expiration.

## Timing assumptions

Unsynchronized clocks with sufficiently similar rates and drift guard. Holder usability must end before grantor obligation expiry. Timely messages are needed for activation/renewal, not base Paxos safety.

## Roles

Stable leader/proposer; acceptors/learners; lease grantors; per-object holders; clients. Roles overlap.

## Message types

Paxos `Prepare`, replies, `Accept`, `AcceptOK`/accept replies, `Committed`; configuration consensus; `Guard`, `GuardACK`, `Promise`, `PromiseReply`; renewal `t′`/`seq_ACK`; heartbeats and forwarded reads.

## Local state

Paxos accepted/committed/executed log and ballots; configuration ID/mapping; per-peer grant/guard/lease timers; acknowledgement history; promise-reported accepted frontier; read-frequency counters and pending interfering operations.

## Normal path

Leader commits writes after majority acceptance and required holder acknowledgements. A caught-up active holder serves reads locally when no unresolved interfering write makes its state stale.

## Fast path

One holder executes the entire read atomically with leases for every read object; no new quorum round. Client read bypasses the leader. No separate fast write path or identical-value read quorum exists.

## Slow path

Forward non-local reads or commit them through Paxos; wait for interfering writes to commit/execute. Multi-object writes cover the union of corresponding holder sets.

## Recovery

Stop renewing failed-holder promises after a grace period, agree a new configuration, and let old obligations expire. Restarted replicas wait `t_wait = t_guard + t_lease` before accepting/proposing. New leader uses ordinary Prepare/value recovery with applicable lease obligations. Newly active holders catch up through grantors' highest accepted instances.

## Commit condition

Ordinary Paxos accepting majority plus synchronous notification acknowledgements from every holder required by active promises discovered in replies. In the normal configured single-object case, all object holders are included. Holder notification and Paxos choice are distinct conditions; local-read response is not a new consensus decision.

## Quorum requirement

- Total `N`; derived crash budget `N - (⌊N/2⌋ + 1)`.
- Classic/recovery: `⌊N/2⌋ + 1`; distinct fast write quorum: not applicable.
- Activation: `⌊N/2⌋` peer promises plus implicit self, forming a grantor majority.
- Holders: arbitrary subset; implementation uses leader plus `⌊N/2⌋` read-popular replicas.
- Write: flexible accepting majority plus fixed current binding holder obligations.
- Majority intersection relays obligations; the read contacts one holder.

## Safety intuition

Every write majority intersects an active reader's grantor majority, forcing notification before commit. An older accepted write is covered by the acceptance frontier carried in activation promises. Timer containment prevents a grantor abandoning an obligation still used by a holder. Together with atomic execution, this preserves strict serializability.

## Liveness intuition

Stable leadership and a responsive majority suffice after obstructing leases expire. Timely renewals maintain local reads. Failed-holder blocking is bounded by `t_grace + t_guard + t_lease` under the assumed clocks and failure-detection policy.

## Strengths

Per-object read locality; one-replica reads; notification can share the normal write round; automatic holder selection from read frequency.

## Weaknesses

Clock assumptions; holder failures delay affected writes; holder unions inflate multi-object cost; all-to-all renewal and configuration management; concurrent writes can delay reads.

## Differences from related protocols

[[Bodega]] protects a ballot-tagged roster and preserves responder authority across writes using different read-release rules. PQL uses timed notification obligations and delays reads around unresolved interference. [[PQR]] instead queries an ordinary majority and rinses a selected replica without leases or holder-covering writes.

## Open questions

Quantitative drift/persistence specification and §4 footnote 6's implicit-vote accounting remain unresolved; see [[Paxos-Quorum-Leases-2014]] and [[unresolved-confusions]].

## Sources

[[Paxos-Quorum-Leases-2014]] - `raw/pql.pdf`, §§2-4, especially §§3.2-3.9 and Figures 4-5.
