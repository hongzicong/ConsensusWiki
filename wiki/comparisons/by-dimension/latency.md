---
type: comparison-dimension
dimension: latency
protocols: [Paxos, N2Paxos, Mencius, FastPaxosPlus, GPaxos, EPaxos, CURP-N2Paxos, SwiftPaxos]
tags: [latency, fast-path, slow-path, contention, stable-run]
---

# Latency

## What this dimension means

The comparison below uses the metric from [[SwiftPaxos-2024]], Table 1: after the system has become stable, latency is the maximum time from command submission until the client can deliver a response. Let `δ` be the upper bound on one message delay in the stable system.

The run classes are:

- **Sequential:** when a command is submitted, all previously submitted commands are already committed everywhere.
- **Conflict-free:** all concurrent commands commute.
- **Contention-free:** concurrent conflicting commands may exist, but all replicas receive them in the same order.
- **General:** the worst stable run, including conflicting commands received in different orders.

They are strictly nested:

`sequential ⊊ conflict-free ⊊ contention-free ⊊ general`.

## Why it matters

Sequential latency measures an unloaded service. Conflict-free latency measures concurrency that does not require ordering. Contention-free latency captures a practically important middle case: hot commands conflict, but the network delivers them consistently. General latency exposes the stable slow-path cost when disagreement or loss of a fast-path participant prevents the optimistic path.

## Comparison table

| Protocol | Sequential | Conflict-free | Contention-free | General | Source |
|---|---:|---:|---:|---:|---|
| Paxos | `4δ` | `4δ` | `4δ` | `4δ` | [[SwiftPaxos-2024]], Table 1 |
| N²Paxos | `3δ+1` | `3δ+1` | `3δ+1` | `3δ+1` | [[SwiftPaxos-2024]], Table 1 |
| [[Mencius]] | `2δ+1` | `4δ+1` | `4δ+1` | `4δ+1` | [[SwiftPaxos-2024]], Table 1 |
| FastPaxos+ | `2δ+1` | `3δ+1` | `3δ+1` | `3δ+1` | [[SwiftPaxos-2024]], Table 1 |
| [[GPaxos]] | `2δ+1` | `2δ+1` | `2δ+1` | `6δ+1` | [[SwiftPaxos-2024]], Table 1 |
| [[EPaxos]] | `2δ+1` | `2δ+1` | `2δ+1` | `O(nδ)` | [[SwiftPaxos-2024]], Table 1 |
| [[CURP]] + N²Paxos | `2δ` | `2δ` | `3δ+1` | `3δ+1` | [[SwiftPaxos-2024]], Table 1 |
| [[SwiftPaxos]] | `2δ` | `2δ` | `2δ` | `3δ` | [[SwiftPaxos-2024]], Table 1 and Proposition 11 |

The source table merges adjacent cells with the same value; the values are expanded above across all covered classes. Preserve `+1` literally: in the paper's notation it denotes one additional message delay for a non-colocated client.

## Main patterns

- [[SwiftPaxos]] is the only row in this table that keeps `2δ` through the contention-free class and also has a `3δ` fixed general bound.
- [[GPaxos]] and [[EPaxos]] also avoid a penalty for same-order conflicts, but their `+1` remains and their general bounds rise to `6δ+1` and `O(nδ)`, respectively.
- [[CURP]] + N²Paxos matches SwiftPaxos at `2δ` only through conflict-free runs; even same-order conflicting concurrency moves it to `3δ+1`.
- FastPaxos+ and [[Mencius]] get their lowest latency only in sequential runs. Their latency rises as soon as commands are concurrent, even when concurrent commands commute.
- Paxos and N²Paxos do not change latency across the four classes because the compared paths do not exploit commutativity or common receive order.

## Important exceptions

- The numbers are the comparative results reported by the SwiftPaxos paper, not an independent re-derivation from every cited protocol's original paper.
- `FastPaxos+` is the paper's named variant; do not silently substitute the base [[FastPaxos]] protocol.
- The `CURP + N²Paxos` row is a composition described by the SwiftPaxos paper. It is not the standalone [[CURP]] primary-backup protocol, whose own paper reports a one-RTT commutative-update path under a different setup.
- [[Atlas]], [[EPaxosStar]], [[PigPaxos]], [[Rabia]], and standalone [[CURP]] are absent from Table 1. [[Pando]] is a storage protocol rather than the same SMR command-response object. Their four-class cells remain `Unclear` until derived under this exact metric.
- General latency assumes the system is already stable. It excludes leader election, recovery before stabilization, and the unbounded delays possible in a fully asynchronous run.

## Common pitfalls

- Do not equate client response, local commit, commit everywhere, and dependency-ready execution; dependency protocols can separate these events.
- Do not read `contention-free` as conflict-free. It allows conflicting concurrency and constrains only the replicas' receive order.
- Do not algebraically rewrite `2δ+1` as `3δ`; the paper deliberately uses `+1` to mark the non-colocated-client hop.
- Do not compare `O(nδ)` with fixed-delay bounds without making the replica count `n` explicit.
- Do not infer a latency for a protocol merely because its message flow resembles a row in this table.

## Relevance to new protocol design

The four classes reveal which favorable property a fast path actually exploits. A conflict-free-only optimization helps mostly cold or sharded items. A contention-free optimization can also help hot items when replicas observe conflicts in a common order. A low general bound limits the penalty when that spontaneous order disappears or a fast-path participant is unavailable.

## Optimization space

- **Inference — `2δ` is a natural floor under this metric:** when the client must first deliver a command to replicas and then receive quorum-backed response evidence, the information flow already consumes two message delays. A `1δ` claim would have to eliminate one of those hops through materially different placement, pre-established authority, or completion semantics; this is not stated as a formal lower bound.
- **Idea — enlarge the `2δ` domain:** replace exact matching with recoverable non-matching evidence, such as an [[Atlas]]-style threshold union or a deterministic normalization of [[SwiftPaxos]] dependency paths. The proof obligation is that every later recovery quorum reconstructs the same conflict-visible, acyclic result.
- **Idea — specialize hot conflict classes:** install an epoch-fenced per-class anchor only after repeated disagreement or dependency debt, while cold classes remain leaderless. This is more likely to bound hot-item execution tails than to lower the nominal `3δ` general response bound. Multi-key commands and anchor transitions are the main safety risks.
- **Idea — optimize execution readiness:** commit an execution-ready dependency certificate or an SCC batch instead of measuring only early command commit. This may trade a small median-commit penalty for a lower tail from response to deterministic execution.
- **Idea — preserve the fast path under partial disconnection:** use versioned quorum profiles, failure-aware quorum placement, or witness quorums whose recovery intersections are explicit. This targets fast-path availability and real WAN delay rather than the theoretical message-delay count.
- **Hypothesis — stable general `2δ` needs a new ordering assumption:** when conflicting commands reach replicas in different orders, two-hop client/quorum communication leaves no extra hop for replicas to reconcile those observations. A pre-established serializer, ordered multicast, lease/clock premise, or changed response semantics may avoid that hop; absent such a mechanism, no `2δ` general claim is made.

See [[new-protocol-ideas]] for the focused latency roadmap and proof obligations.

## Open questions

- Re-derive each row from the original protocol pseudocode under the same client-response metric.
- Extend the matrix to [[Atlas]], [[EPaxosStar]], [[PigPaxos]], [[Rabia]], and standalone [[CURP]] without mixing decision, commit, execution, and response latency.
- Determine how execution dependencies change end-to-end latency when a command is committed but not yet executable.
- Prove or refute a `2δ` stable-general lower bound under an explicit client, quorum, failure, and response model.

## Related pages

[[fast-paths]], [[slow-path]], [[timing-assumptions]], [[conflict-handling]], [[commit-rules]], [[leader-roles]]
