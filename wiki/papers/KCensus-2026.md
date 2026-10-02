---
type: paper
title: "KCensus: Synthesizing Latency-Optimal Consensus Fast Paths [Extended Version]"
authors: [Clément Burgelin, Antoine Murat, Gal Sela, Marcos K. Aguilera, Rachid Guerraoui]
year: 2026
venue: "arXiv extended version; to appear at EuroSys 2027"
source: raw/KCensus.pdf
protocols: [KCensus, KSMR]
tags: [smr, fast-path, adopt-commit, knowledge, recoverability, synthesis, geo-replication]
status: ingested
---

# KCensus: Synthesizing Latency-Optimal Consensus Fast Paths

## One-sentence summary
KCensus synthesizes compatible per-proposer fast paths by tracking who has witnessed which acceptances, then preserves possible fast decisions using a frozen census of that evidence before fallback consensus.

## Why this paper matters
The design space includes both quorum membership and evidence dissemination. Quorum intersection alone prevents simultaneous conflicting fast commits but need not leave enough evidence for recovery after crashes. KCensus characterizes the extra evidence needed, synthesizes strategies for a measured topology and objective, and implements them in the KSMR replication engine. The optimality result is scoped to stable, known latencies and single-proposer executions, not arbitrary workloads (§§3–6, Appendix B).

The source is the 38-page arXiv v2 dated 28 September 2026, with a first-page note saying the proceedings paper will appear at EuroSys '27. The filename/year follows the supplied preprint rather than the future proceedings year. [Version metadata](https://arxiv.org/abs/2609.31302v2); [proceedings DOI](https://doi.org/10.1145/3842654.3848528). Protocol extraction below uses `raw/KCensus.pdf`.

## System model
Point-to-point message passing among `n ≥ 2f + 1` voting processes. Links between correct processes are FIFO and lossless. The core is one-shot adopt-commit, with a fixed requirement dictionary per proposer and instance. KSMR composes it with fallback consensus for each slot of each shard's log. Commands execute deterministically; independent keys can use independent logs (§§2.2–2.3, 4, 6).

## Fault model
At most `f` voting processes crash; no Byzantine behavior. Non-voting clients may propose and forward evidence but are never required acceptors or witnesses, and their failures do not consume this budget (§4.5). Crash restart, durable-state restoration, and membership changes are not specified by the displayed core algorithms. Do not infer crash-recovery support from the word recovery.

## Timing assumptions
Safety holds under full asynchrony. Adopt-commit termination needs eventual suspicion of crashed required processes; false suspicion can safely trigger adoption. Deterministic SMR termination relies on partial synchrony and the fallback protocol. Optimization additionally needs periods stable enough to measure latencies/rates and install strategies. Appendix B's optimality setting assumes fixed known latencies, permanent crashes, and accurate failure detection (§2.3; Appendix B, Definitions 4–6).

## Main idea
For proposer `p`, requirement `R_p` maps witness `q` to the acceptors whose acceptance of `p`'s value `q` must record. The proposer must learn that each such record exists. Normal form requires `R_p[q] ⊆ R_p.keys()` and `q ∈ R_p[q]`; define `Q_p = R_p.keys()`.

Algorithm 1 imposes:

```text
valid(R_p) iff |Q_p| > f
I = Q_p ∩ Q_q
W_p = {w ∈ Q_p : R_p[w] ∩ I ≠ ∅}
W_q = {w ∈ Q_q : R_q[w] ∩ I ≠ ∅}
compatible(R_p, R_q) iff |W_p ∪ W_q| > f
```

Compatibility can be asymmetric in where the evidence is stored. It counts distinct witnesses of intersection acceptances, including witnesses outside the intersection, rather than requiring the intersection itself to exceed `f`. See [[knowledge-requirement]] (§3.1).

## Protocol roles
- Voting process: accepts at most one value per instance and maintains first- and second-order evidence.
- Proposer: starts evidence dissemination, checks its requirement, or initiates adoption; no fixed fast-path leader is required.
- Non-voting proposer: external client that does not contribute recoverability evidence.
- Adopter: proposer collecting frozen replies; several adopters may run concurrently.
- Fallback leader: KSMR uses Paxos; §6 describes highest-ID concurrent-proposer election overlapped with adopt-commit.
- Execution delegate: optional nearby replica performs commitment/execution for a non-voting logical proposer using requirements synthesized for proposer–delegate pairs (§§4–6).

## Message types
Algorithms 2–6 name `Accept&Spread(v, knowledge_of_q)`, `DoAdopt()`, `Freeze()`, and `Frozen(me, accepted, known_acceptors[me])`. `WeakFailureDetector::PotentialFailure(q)` is a local event. `Commit(v)` and `Adopt(v)` are outputs; Algorithm 7 produces `Decide(v)` through direct commit or fallback. Slot/shard scoping, commit announcements, latency tables, and Paxos messages belong to the surrounding KSMR implementation (§6), not additional core algorithm message definitions.

## Local state
- `accepted = ⊥`: immutable after accepting the first non-bottom value.
- `known_acceptors`: array of sets, initially empty. At process `p`, entry `[p]` is its own witnessed acceptances; entry `[q]` is its knowledge of what `q` witnessed, all for `accepted`.
- `REQUIREMENTS[proposer]`: constant normal-form requirements for the instance; recovery accesses the original proposer's requirement through `v.proposer`.
- Frozen status: after `Freeze`, no further `Accept&Spread` handling for that instance; repeated freeze requests can still be answered.
- Propose-call input, pending adoption notification, and collected frozen replies.
- KSMR adds per-shard slots/logs, topology/rate information, dissemination graphs, and fallback Paxos state; full state-machine definitions for these optimizations are not displayed (§§4, 6).

## Normal path
The voting-process core follows Algorithms 2–3 (pp. 5–6):

```text
on Propose(v):
    send Accept&Spread(v, empty knowledge) to self
    wait until can_commit(v) or DoAdopt received
    if released by can_commit(v): output Commit(v)
    else: run the recovery path below

on Accept&Spread(v, K) from q, if not frozen:
    if accepted == ⊥: accepted = v
    if accepted != v: send DoAdopt() to all, including self
    else:
        old = snapshot(known_acceptors)
        known_acceptors[me] ∪= {me} ∪ K[q]
        for each r: known_acceptors[r] ∪= K[r]
        if known_acceptors != old:
            send Accept&Spread(v, known_acceptors) to others
```

These are atomic handlers between waits in Appendix A.1. The snapshot notation makes the paper's `old_acceptors` comparison explicit; it must not alias the mutable array.

## Fast path
Proposer `p` commits its value only after accepting it and covering every entry of `R_p` with its second-order evidence. Evidence may travel through more than one hop; there is no universal fixed fast-quorum cardinality or one-RTT theorem for all topologies. In the evaluated deployments most fast paths take one round trip and others slightly more (§§4.3, 6).

The optimizer simulates single-proposer dissemination under measured latencies, enumerates budgets at evidence-arrival events, discards invalid requirements, and selects a mutually compatible combination minimizing the objective. It prunes dominated budgets and uses branch-and-bound. A deterministic dissemination graph then removes unnecessary messages and uses latency shortcuts; it preserves required evidence and commit checks. If an outside relay is needed, its failure must also trigger abandonment. Identifier-only evidence is buffered until the value payload has been received and logged (§§5.1–5.4).

## Slow path
Conflicting values or suspicion of a required witness cause `DoAdopt`. After adoption, Algorithm 7 submits the adopted value to fallback consensus; adoption itself is not a decision. KSMR uses Paxos and overlaps its first phase with adopt-commit. The implementation freezes processes when they observe conflict or finish their dissemination-graph role, continues prescribed forwarding, and piggybacks frozen state on the last graph message. This is an implementation refinement of the full-broadcast template, not a replacement for its frozen-evidence obligation (§6).

## Recovery path
Algorithms 2, 3, and 6:

```text
on Freeze() from q:
    stop handling Accept&Spread for this instance
    send DoAdopt() to all, including self
    send Frozen(me, accepted, known_acceptors[me]) to q

on adoption branch at proposer with original input v:
    send Freeze() to all, including self
    wait for n - f Frozen replies from distinct voting processes
    for each non-⊥ candidate u appearing in replies:
        R = REQUIREMENTS[u.proposer]
        reject u if a reply (p, val != u, A) has
                    ({p} ∪ A) ∩ R.keys() != ∅
        reject u if a reply (p, val == u, A) has
                    p ∈ R.keys() and R[p] ⊈ A
        if not rejected: output Adopt(u) and return
    output Adopt(v)
```

Missing evidence is meaningful because a frozen witness cannot later acquire it. Any census intersects every valid requirement and every compatibility witness set. If any value committed, it appears in every census, passes both tests, and every conflicting value fails. If none passes, adopting the proposer's own input preserves validity. Without a fast commit, the abstraction permits different adopters to choose differently; fallback consensus resolves that disagreement. Do not infer an unconditional unique-candidate theorem (§4.4; Lemmas A.11–A.14).

## Commit rule
Algorithm 4: `can_commit(v)` is true iff `accepted = v` and, for every `q ∈ REQUIREMENTS[me].keys()`, `REQUIREMENTS[me][q] ⊆ known_acceptors[q]`. This is a local evidence predicate, not just a count of votes and not a cryptographic certificate. Algorithm 7 decides directly on `Commit(v)` or on a fallback decision after `Adopt(v)`. Client completion additionally requires execution; delegation addresses that cost (§6).

## Quorum system
- Total voting processes: `n ≥ 2f + 1`; evaluation uses `2f + 1`.
- Fast quorum: `Q_p = R_p.keys()`, with `|Q_p| > f` plus pairwise compatibility; membership is synthesized per proposer, not any set of that cardinality.
- Recovery census: any `n - f` distinct voting processes, without leader inclusion.
- Classic fallback: Paxos majority in KSMR (§6).
- No universally required fast-path leader; the requirement specifies all mandatory witnesses/acceptors.
- The template fixes one requirement per proposer per instance. §5.3 discusses multiple requirements, any one sufficient for commit and all alternatives subject to compatibility; optimizing robust sets is future work.
- Non-voting clients never appear in the required witness/acceptor sets (§4.5).

## Conflict handling
The one-shot instance accepts one value, with conflicting arrivals triggering adoption. KSMR assigns separate logs to non-interfering operations, e.g. different keys. A command that does not win a slot is proposed again in a later slot. When adoption establishes no possible fast commit, §6 permits the fallback leader to batch known conflicting commands into one Paxos proposal. This is slot consensus with sharding, not EPaxos dependency agreement.

## Safety argument
Evidence is truthful, value-specific, and monotone; no process accepts two values. Pairwise compatibility implies quorum overlap, so two fast commits cannot disagree. Freeze makes each witness's report an upper bound on all evidence it could have supplied. More than `f` intersection witnesses force every size-`n-f` census to preserve any committed value and eliminate competing ones. Adopt-commit agreement plus fallback agreement/validity then gives consensus agreement (Appendix A). See [[knowledge-census-recovery]].

## Liveness argument
Every correct proposer eventually commits or adopts. A freeze or conflicting acceptance disseminates `DoAdopt`; recovery receives `n - f` replies. If no such event occurs, correct required witnesses eventually disseminate sufficient evidence, while a crashed required process is eventually suspected. Actual consensus termination additionally requires the fallback's liveness assumptions; it is not deterministic asynchronous consensus without a failure detector (Theorems A.20 and A.23).

## Key proof ideas
- Observations A.4–A.6 and Corollary A.7: one acceptance, truthful evidence, monotonicity, and remote reports bounded by the witness's final local state.
- Lemma A.11: every census meets the pair's distinguishing-witness set.
- Lemmas A.12–A.14: eliminate rivals, find the committed value, and never eliminate that value.
- Theorems A.20–A.22: adopt-commit termination, validity, and conditional agreement.
- Theorem A.23: compose adopt-commit with fallback consensus.
- Theorem B.1: weighted objective latency optimality among deterministic consensus protocols in the stable-network setting. The proof derives requirements from causal communication and uses indistinguishable executions when validity or compatibility would fail.
- Theorems C.1–C.2: sufficiency and necessity of validity/compatibility for recoverable fast paths under that derived-requirement model. This is an existence/characterization result, not proof that an arbitrary implementation with those quorum sizes is safe.

The supplied paper provides mathematical proofs; this ingestion does not supply or claim a machine-checked proof.

## Important formulas
In addition to Algorithm 1 and the commit/recovery rules above, Problem 1 minimizes:

```text
b★ = arg min_(b_1,...,b_n) ∈ B_1 × ... × B_n  Σ_(p=1)^n P_p b_p
subject to compatible(R_(p,b_p), R_(q,b_q)) for every p,q ∈ {1,...,n}
```

Here `P_p` is proposer probability; invalid candidates were already removed. Changing the objective can target median or tail latency. Appendix B's metric is proposal-to-decision time over single-correct-proposer executions weighted by proposer probability, distinct from arbitrary-concurrency client execution latency.

§3.2 interprets unrestricted cardinality-based Fast Paxos fast quorums using `2q - n > f`, equivalently `q > (n + f)/2`. This is not KCensus's own cardinality rule and does not replace the full quorum-family conditions in [[FastPaxos]].

## Relationship to other protocols
- [[FastPaxos]] relies on sufficiently large fast intersections; KCensus can place additional intersection evidence outside them.
- [[SwiftPaxos]] uses leader-including fast-quorum strategies; KCensus synthesizes proposer-specific evidence patterns and uses a different fallback structure.
- [[EPaxos]] uses proposer participation to spread acceptance evidence beyond intersections; KCensus uses this as an explanatory example (§3.2), not a proof of original EPaxos recovery. [[EPaxosStar]] records the later recovery correction.
- [[FPaxos]] varies phase quorum families; KCensus optimizes who witnesses which acceptances as well as membership.
- [[Jetpack]] adds a host-protocol fast path with view promises; KCensus instead instantiates adopt-commit with synthesized requirements and an explicit census.
- [[CURP]] witnesses hold unordered operations; KCensus witnesses are voting processes recording acceptance knowledge. These meanings must stay separate.

## Limitations
- Optimality excludes contention, varying/unknown delays, inaccurate failure detection in the stable comparison, and processing/queueing costs not captured by link latency (§5.2; Appendix B).
- Baselines are reimplementations in the same Rust KSMR codebase, with Pando erasure coding disabled and EPaxos non-thrifty. Default tests use 100,000 keys, 16-byte requests/replies, and 1,000 aggregate requests/s; the key space is partitioned per client for the default conflict-free workload (§7).
- Four seven-replica deployments show average reductions of 15% NH, 15% EA, 12% EU, and 9% NA versus the fastest baseline in each. The reported maximum 16% comes from the scalability study (§§7.1, 7.3).
- NH with three replica failures: SwiftPaxos is 3% faster. This evaluates post-failure latency across fault placements, not a timed sustained-recovery trace (§7.2).
- NH Zipf 0.99, 50/50 reads/writes: KCensus write mean/p99 is 158/385 ms versus SwiftPaxos 146/209 ms. Optimal conflict-free paths do not ensure lowest conflict tail latency (§7.5).
- At 31 replicas, requirements optimization takes up to 144 ms in the reported deployments; evidence metadata raises memory to 39.2 MiB/server. These are paper measurements, not repository benchmarks (§§7.4, 7.7).

## Open questions
- TODO: specify the exact slot/epoch boundary for installing new requirement tables while old instances are in flight; §6 describes cross-shard consensus installation but gives no transition-level fencing rule.
- TODO: inspect the implementation for the refinement relating early freeze, continued graph forwarding, and piggybacked Paxos Phase 1 to Algorithms 2–7.
- TODO: make the per-instance value identity/proposer binding explicit before modeling: Algorithm 6 uses `v.proposer`, but the abstract value type and repeated logical-value proposals are not spelled out.
- TODO: extract restart persistence, garbage collection, catch-up, deduplication, and exact read-state evidence from the artifact before claiming a complete executable SMR specification.

## Local topology-selection audit (2026-10-01)
Arena's `kcensus.Synthesize` uses compatible per-proposer requirements and latency-budget frontiers, matching the general §5 search approach. Do not describe a distinct strategy-space restriction solely because the search is a local implementation; the concrete differences established here are objective/input scope and execution support. The runtime calls it with uniform weights for voting proposers, while the external selector's client weights do not reach synthesis. It routes clients to nearest replicas rather than preserving them as non-voting logical proposers with §6 delegated execution. The bundled five-replica ingress receives 3/1/6 clients at Japan/Paris/California and none at India/South Africa, so equal offered rates per client do not imply equal ingress traffic per voting proposer. Actual proposal rates can further change through conflicts and reproposals and have not been measured by this audit.

The synthesizer applies a shortest-path transform to the replica matrix, but `graph.go` sends direct evidence edges and explicitly omits shortest-path relay/event-graph optimization. On Arena's 5/9/13 topologies, 2/8/10 directed replica pairs gain shortcut savings (up to 33.5/49/49 ms one way). Consequently, the predicted budget can assume a route not realized by this port. This finding concerns performance-model consistency; it does not establish a safety failure. Sources: paper §§5.2, 5.4 and 6; local `requirements.go`, `replica.go`, `graph.go`; `report/baseline/configuration-audit.md` in the parent repository. No runtime behavior changed in this audit.

## Related pages
[[KCensus]], [[knowledge-requirement]], [[knowledge-census-recovery]], [[recoverability]], [[adopt-commit-abstraction]], [[quorum-systems]], [[fast-paths]], [[recovery-rules]], [[commit-rules]], [[proof-techniques]], [[latency]], [[fast-consensus]]
