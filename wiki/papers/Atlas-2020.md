---
type: paper
title: State-Machine Replication for Planet-Scale Systems
authors: [Vitor Enes, Carlos Baquero, Tuanir Franca Rezende, Alexey Gotsman, Matthieu Perrin, Pierre Sutra]
year: 2020
venue: EuroSys '20
source: raw/atlas.pdf
protocols: [Atlas]
tags: [consensus, smr, leaderless, fast-path, geo-replication]
status: ingested
---

# State-Machine Replication for Planet-Scale Systems

## One-sentence summary
Atlas is a leaderless SMR protocol for planet-scale deployments that makes the fast quorum size depend on the tolerated concurrent site failures `f`, not only on the total site count `n`.

## Why this paper matters
Atlas is a useful bridge between [[EPaxos]]-style dependency SMR and Flexible Paxos-style quorum tuning: it keeps command coordinators leaderless, reduces fast quorum size to `floor(n/2) + f`, and gives an explicit recovery rule for preserving fast-path dependency decisions.

## System model
The system has `n` processes `P = {1, ..., n}` in an asynchronous distributed system. In a geo-distributed deployment, each process represents a data center. The process set is static; the paper notes that classical reconfiguration techniques can be added.

The service is a deterministic state machine with commands. Processes submit commands for clients and eventually trigger `execute(c)`.

## Fault model
At most `f` processes may crash, where `1 <= f <= floor((n - 1)/2)`. Processes are non-Byzantine. If more than `f` transient outages occur, the paper states that Atlas may block but safety is not compromised.

## Timing assumptions
Safety is asynchronous and quorum/recovery based. Liveness/performance require enough reachable sites to collect the needed fast, slow, or recovery quorums.

## Main idea
Each command has a coordinator that collects dependency reports from a fast quorum. The coordinator commits in one round trip if every dependency in the proposed union is reported by at least `f` fast-quorum processes; otherwise it runs a Flexible Paxos-style slow path. Recovery gathers `n - f` replies and either preserves an accepted consensus proposal, reconstructs a possible fast-path proposal, or chooses `noOp`.

## Protocol roles
- Client.
- Process / replica / data center.
- Initial command coordinator.
- Recovery coordinator.
- Acceptors for the per-command consensus instance.

## Message types
- `MCollect(id, c, past, Q)`.
- `MCollectAck(id, dep[id])`.
- `MConsensus(id, c, D, b)`.
- `MConsensusAck(id, b)`.
- `MCommit(id, c, D)`.
- `MRec(id, c, b)`.
- `MRecAck(id, cmd[id], dep[id], quorum[id], abal[id], b)`.

## Local state
For each command identifier `id`, a process stores:
- `cmd[id]`, initially `noOp`.
- `phase[id]`, initially `start`, with phases `start`, `collect`, `recover`, `commit`, `execute`.
- `dep[id]`, a dependency set.
- `quorum[id]`, the fast quorum that saw the initial collect message, or empty if unknown.
- `bal[id]`, the current ballot.
- `abal[id]`, the last accepted consensus ballot.

Identifiers are pairs `<i, l>`, where `i` is the submitting process and `l` is its local sequence number.

## Normal path
On `submit(c)`, the initial coordinator assigns `id`, computes `past = conflicts(c)`, chooses a fast quorum `Q` of size `floor(n/2) + f` including itself, and sends `MCollect`.

Each fast-quorum process computes `dep[id] = conflicts(c) union past`, stores the command and quorum, enters `collect`, and replies with `MCollectAck`.

The coordinator waits for all replies from `Q` and computes:

```text
D = union_Q dep
```

It then either commits by the fast path or runs the slow path.

## Fast path
Atlas commits in one round trip when:

```text
union_Q dep = union^f_Q dep
```

where:

```text
union^f_Q dep = {id | count(id) >= f}
count(id) = |{j in Q | id in dep_j}|
```

This means each dependency included in the final union was reported by at least `f` fast-quorum processes. Unlike EPaxos-style matching-reply fast paths, Atlas can take the fast path even when non-commuting commands are observed in different orders, as long as the dependency union is recoverable under `f` failures. When `f = 1`, the condition always holds and the fast quorum is a majority.

## Slow path
If the fast-path condition fails, the coordinator runs a per-identifier consensus instance using Flexible Paxos. The initial coordinator skips Phase 1 and sends `MConsensus` at its reserved ballot to a slow quorum of size `f + 1`. After `f + 1` `MConsensusAck` replies, it broadcasts `MCommit`.

The paper also gives a slow-path optimization: instead of proposing `union_Q dep`, the coordinator may propose `union^f_Q dep` to prune dependencies reported by fewer than `f` fast-quorum processes.

## Recovery path
A process calls `recover(id)`, chooses a higher ballot it owns, and sends `MRec` to all. It waits for a recovery quorum of `n - f` `MRecAck` replies.

Selection rule:
- If any reply has `abal != 0`, choose the command/dependencies accepted at the highest accepted ballot.
- Else, if some reply contains a non-empty initial fast quorum `Q0`, choose dependencies by unioning either the whole recovery quorum if the initial coordinator replied, or `Q intersect Q0` if the initial coordinator did not reply.
- Else choose `(noOp, empty dependencies)`.

The key recovery observation is that a fast-path proposal can be reconstructed from at least `floor(n/2)` fast-quorum processes that are not the initial coordinator.

## Commit rule
Fast path: after all `MCollectAck` replies from the fast quorum and `union_Q dep = union^f_Q dep`, the coordinator broadcasts `MCommit(id, cmd[id], D)`.

Slow path: after `f + 1` `MConsensusAck` replies in the coordinator's ballot, the coordinator broadcasts `MCommit(id, cmd[id], dep[id])`.

A process that receives `MCommit` stores the command/dependencies and marks the identifier `commit`.

## Quorum system
- Total processes: `n`.
- Fault tolerance: `f`, with `1 <= f <= floor((n - 1)/2)`.
- Fast quorum: `floor(n/2) + f`, including the initial coordinator.
- Slow quorum: `f + 1`.
- Recovery quorum: `n - f`.
- NFR read quorum: plain majority, independent of `f`.
- Recovery intersection: a recovery quorum of size `n - f` intersects a fast quorum of size `floor(n/2) + f` in at least `floor(n/2)` processes if the initial coordinator is absent from the recovery quorum.

## Conflict handling
Commands conflict when they do not commute. Atlas records dependencies between conflicting commands and executes committed commands only after dependencies are committed/executed or in the same batch.

The dependency invariant is:

```text
If MCommit(id, c, D) and MCommit(id', c', D') have been sent,
id != id', conflict(c, c'), c != noOp, and c' != noOp,
then either id' in D or id in D', or both.
```

## Safety argument
The paper presents these key invariants:
- Per identifier, all `MCommit` messages agree on the command and dependency set.
- For distinct conflicting committed non-`noOp` commands, one is in the dependency set of the other.
- If a process executes a batch containing `c` before a batch containing `c'`, then `id'` is not in `dep[id]`.
- The batch containing a given command is the same at every process.

These imply that the relation combining real-time order and per-process conflicting execution order is acyclic, giving the SMR ordering property.

## Liveness argument
The paper's main liveness/performance claims assume enough responsive processes to collect the needed quorums. If more than `f` sites are unavailable, Atlas may block until enough sites are reachable.

## Key proof ideas
- The fast-path condition makes each included dependency survive the loss of up to `f` fast-quorum processes.
- Recovery quorums of `n - f` are larger than slow quorums so they can discover any `f + 1` accepted slow-path proposal.
- If a fast quorum is known and the initial coordinator did not answer recovery, the intersection `Q intersect Q0` contains enough non-coordinator fast-quorum replies to reconstruct a possible fast-path proposal.
- `noOp` prevents an unknown command payload from permanently blocking commands that depend on its identifier.

## Important formulas
```text
1 <= f <= floor((n - 1)/2)
fast quorum size = floor(n/2) + f
slow quorum size = f + 1
recovery quorum size = n - f
union^f_Q dep = {id | count(id) >= f}
count(id) = |{j in Q | id in dep_j}|
fast path condition: union_Q dep = union^f_Q dep
```

## Relationship to other protocols
Atlas shares the leaderless per-command coordination shape of [[EPaxos]], but differs in two central ways: its fast quorum size is `floor(n/2) + f`, and its fast path allows non-matching dependency replies when each final dependency is reported by at least `f` fast-quorum members.

Compared with leader-based Flexible Paxos, Atlas also uses the small slow quorum `f + 1`, but this is only the fallback for a leaderless dependency protocol; recovery uses `n - f`.

## Limitations
- Atlas trades off high simultaneous site-failure tolerance for lower latency under small `f`.
- It may block if more than `f` sites are unreachable.
- Non-fault-tolerant reads require a transitive conflict relation.
- Dependency/batch execution can separate commit latency from execution latency.

## Open questions
- TODO: Extract the full technical-report proof of Invariants 1 and 2 from reference [9] if the extended version is added to `raw/`.
- TODO: Compare Atlas recovery directly with EPaxos recovery once the exact EPaxos recovery page is expanded.

## Related pages
[[Atlas]], [[EPaxos]], [[quorum-systems]], [[fast-paths]], [[recovery-rules]], [[commit-rules]], [[conflict-handling]], [[quorum-intersection]]
