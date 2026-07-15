---
type: comparison-dimension
dimension: latency
protocols: [Paxos, FPaxos, OmniPaxos, N2Paxos, Mencius, FastPaxosPlus, GPaxos, EPaxos, CURP-N2Paxos, SwiftPaxos, Copilot, Avicenna, Bodega, Jetpack, HydraPaxos, WPaxos]
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
| [[Copilot]] | `Unclear` | `Unclear` | `Unclear` | `Unclear` | [[Copilot-2020]] uses a different slowdown-focused empirical metric |
| [[Avicenna]] | `Unclear` | `Unclear` | `Unclear` | `Unclear` | [[Avicenna-2026]] proves/evaluates a Multi-Paxos-shaped normal path and fail-slow counterfactual metric, not this four-class table |
| [[Bodega]] | `Unclear` | `Unclear` | `Unclear` | `Unclear` | [[Bodega-2026]] gives a separate read/write/interference latency model, not this command-concurrency table |
| [[Jetpack]] | `Unclear` | `Unclear` | `Unclear` | `Unclear` | [[Jetpack-2026]] states a client-RTT plugin fast path and host fallback, not this `δ`-class model |
| [[FPaxos]] | `Unclear` | `Unclear` | `Unclear` | `Unclear` | [[Flexible-Paxos-2016]] changes quorum collection size/shape but does not derive this four-class client metric |
| [[OmniPaxos]] | `Unclear` | `Unclear` | `Unclear` | `Unclear` | [[Omni-Paxos-2023]] reports one stable leader-to-majority round trip and separate partition-recovery timeouts, not this four-class metric |
| [[HydraPaxos]] | `Unclear` | `Unclear` | `Unclear` | `Unclear` | [[Hydra-2023]] reports one client RTT in the no-drop normal case, not the four-class latency metric |
| [[WPaxos]] | `Unclear` | `Unclear` | `Unclear` | `Unclear` | [[WPaxos-2020]] reports locality-dependent WAN latency, not the four stable-run classes |

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
- [[Copilot]] evaluates whether latency remains near its no-slowdown baseline when any one replica is slow. That `T` versus counterfactual `T'` notion is useful but cannot be silently converted into the four stable-run `δ` classes above.
- [[Avicenna]] reports the same normal-case message-delay latency as Multi-Paxos and defines `ε-s` fail-slow tolerance against removing the slow replicas. Neither claim supplies the four workload-class cells above, so they remain `Unclear`.
- [[Bodega]] models local reads and interfering writes rather than these four command-run classes. Its cells remain `Unclear`; its source formulas are recorded separately here.
- [[Jetpack]] states 1 client RTT for successful conflict-free fast commitment and otherwise inherits host latency. That does not uniquely determine these four `δ` classes across every compatible host.
- [[FPaxos]] retains the Paxos phase/message shape. Smaller or better-placed `Q2` can lower measured steady-state latency, but the exact `δ` row depends on the surrounding Multi-Paxos client/leader path and is not given by the source.
- [[OmniPaxos]] reports stable Sequence Paxos replication and partial-partition recovery using different metrics. These results cannot be converted into the SwiftPaxos four-class `δ` table without specifying client placement and response semantics.
- General latency assumes the system is already stable. It excludes leader election, recovery before stabilization, and the unbounded delays possible in a fully asynchronous run.

## Bodega read-oriented latency model

[[Bodega-2026]], Table 1 uses different symbols and assumes each protocol's most read-optimized configuration while tolerating `f = floor(n / 2)` faults. For Bodega it states:

```text
write latency W = l + N
quiescent read latency R = c
interfered read latency R* = c ~ c + m/2
read degradation period D* = m/2
```

Here `l` is client-leader RTT, `c` is client-nearest-server RTT, `m` is time to establish a simple majority, and `N` is time to form an all-node quorum. These formulas must not be substituted into the `δ` table above.

## Jetpack plugin latency model

[[Jetpack-2026]] contrasts a successful 1-client-RTT fast commit with the host protocol's original path, typically 2 client RTTs for a remote leader. Both paths run concurrently, so failure of the fast path does not add a sequential retry round. Contention, view mismatch, proposer membership, host locality, and saturation determine which path returns first; the paper reports up to 60% average commit-latency reduction across its integrations.

## FPaxos quorum-latency model

[[Flexible-Paxos-2016]] does not reduce protocol phases. It reduces the number of Phase 2 acceptors a stable leader waits for and allows topology-aware quorum membership. The prototype reports lower average latency and higher throughput for smaller `Q2`, but also notes that sending only to a selected quorum can add retransmission delay if those acceptors are slow or failed. Leader-change latency/availability depends separately on `Q1`.

## OmniPaxos replication and recovery model

[[Omni-Paxos-2023]] states that stable Sequence Paxos decides through one round trip from the leader to a majority and pipelines later entries. In its evaluation, BLE heartbeat traffic contributed at most `0.02%` of total I/O. Under the tested partial-connectivity scenarios, quorum-loss recovery averaged four heartbeat rounds, while constrained-election recovery took three election timeouts. These are component-specific empirical/model claims, not entries in the four stable workload classes above.

## HydraPaxos and WPaxos latency models

[[HydraPaxos]] states one client round trip in a stable no-drop configuration. Its receiver frontier, client/leader placement, and drop recovery are not parameters in the four-class table, so no `δ` cells are inferred.

[[WPaxos]] latency depends on ownership state: a current local owner runs nearby Phase 2, while a remote first access or locality shift adds WAN Phase 1 [[object-stealing]]. This locality/ownership dimension is also absent from the four-class table.

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
