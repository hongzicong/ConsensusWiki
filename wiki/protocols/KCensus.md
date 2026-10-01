---
type: protocol
name: KCensus
family: synthesized fast consensus / adopt-commit
papers: [KCensus-2026]
tags: [fast-path, knowledge, recoverability, synthesis, smr]
---

# KCensus

## Short description
Knowledge Census (KCensus) synthesizes compatible per-proposer acceptance-evidence requirements. Its adopt-commit template either fast-commits or freezes a recovery quorum and adopts a safe value for fallback consensus. KSMR is the paper's sharded SMR engine built from these instances, not a different quorum algorithm ([[KCensus-2026]], §§3–6).

## Problem solved
Optimize common-case consensus latency for topology, proposer distribution, and a selected mean/median/tail objective while preserving enough evidence to recover after crashes. The design varies both quorum membership and who witnesses acceptances; it does not claim optimal contentious-workload latency.

## System model
`n ≥ 2f + 1` voting processes; point-to-point FIFO, lossless links between correct processes. Each instance has fixed normal-form requirements. The core decides one value; KSMR adds one instance per shard/log slot (§§2.3, 4, 6).

## Fault model
Up to `f` voting-process crashes, no Byzantine faults. Non-voting clients may fail without counting toward `f`, because no requirement depends on their acceptance/evidence. Restart/persistence semantics are not specified in the core (§4.5).

## Timing assumptions
Asynchronous safety; eventual crash suspicion for adopt-commit termination; partial synchrony for deterministic SMR termination. Stable known latencies and accurate failure detection are additional assumptions of the optimality theorem, not of safety (§2.3; Appendix B).

## Roles
Voting acceptors also witness and relay evidence. Any proposer starts a fast path or census; no global leader is mandatory. External non-voting clients can propose. KSMR uses a Paxos fallback leader and optionally an execution delegate chosen near a client (§§4.5, 6).

## Message types
`Accept&Spread(v,K)`, `DoAdopt()`, `Freeze()`, and `Frozen(p,val,A)`; `PotentialFailure(q)` is a detector event. Adopt-commit emits `Commit(v)` or `Adopt(v)`; its consensus wrapper emits `Decide(v)` (Algorithms 2–7).

## Local state
Per instance: `accepted = ⊥`; `known_acceptors[r] = ∅` for every voting process; fixed `REQUIREMENTS`; frozen flag; proposal input and census replies. At process `p`, `[p]` contains its own witnessed acceptances, while `[q]` records what it knows `q` witnessed. Evidence is value-specific, not a count or a dependency set. Values retain the original proposer identity used by recovery. KSMR additionally tracks shard slots/logs, dissemination graphs, topology, and Paxos state (§§4, 6).

## Normal path
Algorithms 2–3, in transition order:

1. Proposer sends `Accept&Spread(v,{})` to itself and waits for commitment evidence or `DoAdopt`.
2. On the first spread message, an unfrozen voting process sets `accepted` to its value permanently.
3. On a spread message for another value, it broadcasts `DoAdopt`, without merging that value's evidence.
4. On a spread message `(v,K)` from `q` for its accepted value, it unions `{me} ∪ K[q]` into its own entry, then unions `K[r]` into every entry `[r]`.
5. Any evidence growth broadcasts the updated array. A proposer commits when its requirement is covered.

Non-voters omit their self-acceptance from tracked evidence, iterate only voting processes, and ignore `Freeze`; the census counts only voters (§4.5).

## Fast path
Satisfy the exact per-proposer requirement below. The optimized runtime uses a dissemination graph rather than all broadcasts, including relay shortcuts when measured links violate triangle inequality. Monitor needed external relays too. Compact edge/proposer identifiers do not authorize commit before the value is received and logged (§5.4).

The paper's five-process example uses `R_1 = {1:{1,2,5}, 2:{2}, 5:{5}}` and `R_4 = {4:{3,4,5}, 3:{3}, 5:{5}}`. Their quorum intersection is `{5}`, but the distinct intersection witnesses are `{1,4,5}`, sufficient for `f = 2`. In Figure 4, process 1 can commit at `t = 2`, while process 4 cannot (§4.3, pp. 5–6).

## Slow path
Conflict, received freeze, or suspicion of a required process triggers adoption. `Adopt(v)` feeds the same slot's fallback consensus and does not itself decide it. KSMR uses Paxos, overlapping leader election with evidence exchange, then running Phase 2 after adoption. A losing command retries a later slot; non-interfering commands use separate shard logs. Batching conflicting proposals is allowed when the census rules out every fast commitment (§6).

## Recovery
Any proposer entering adoption sends `Freeze` to all voters and waits for `n-f` distinct `Frozen(p,val,A)` replies. A frozen voter stops processing all spread messages for the instance, broadcasts `DoAdopt`, and replies with its accepted value plus its own first-order evidence. It remains able to answer freeze requests.

For each non-bottom candidate `v` in replies, let `R = REQUIREMENTS[v.proposer]`. Algorithm 6 rules out `v` if either:

- A report with `val != v` has `({p} ∪ A) ∩ R.keys() != ∅`.
- A report with `val == v` has `p ∈ R.keys()` but `R[p] ⊈ A`.

Adopt the first candidate not ruled out; if none survives, adopt the original input. Both tests are necessary: a report can invalidate a required acceptor indirectly, or show that a required witness froze before acquiring enough evidence. Any committed value must be the surviving candidate in every census; without a fast commit, adopters need not agree until fallback runs (§4.4; Appendix A).

Freeze prevents evidence growth at the respondent; it does not revoke a commit already enabled by existing evidence. Treating it as an unconditional global fast-commit cancellation is not the displayed algorithm.

## Commit condition
At proposer `p`, Algorithm 4 requires:

```text
accepted_p = v
and for every q ∈ R_p.keys(): R_p[q] ⊆ known_acceptors_p[q]
```

The requirements must have passed validity/compatibility checks before use. Matching value votes alone do not suffice. Algorithm 7 maps this commit directly to a decision; otherwise fallback consensus decides the adopted input. Service execution/client reply is a later condition, or performed by the execution delegate (§6).

## Quorum requirement
Normal form and safety constraints (Algorithm 1):

```text
Q_p = R_p.keys()
for each w ∈ Q_p: w ∈ R_p[w] ⊆ Q_p
|Q_p| > f
I = Q_p ∩ Q_q
W_p = {w ∈ Q_p : R_p[w] ∩ I ≠ ∅}
W_q = {w ∈ Q_q : R_q[w] ∩ I ≠ ∅}
|W_p ∪ W_q| > f
```

Fast sets are fixed by the selected requirements for the instance, may differ by proposer, and have no universal leader inclusion. A size-`f+1` set is not automatically a legal fast quorum. The census is any size-`n-f` voter set; fallback KSMR Paxos uses a majority. §5.3 permits multiple compatible alternatives per proposer but leaves optimizing their robustness to future work.

## Safety intuition
Processes accept once; evidence only records actual acceptances of that same value. Compatible fast quorums intersect, precluding two different fast commits. Recovery needs more: every census sees at least one of the pair's more-than-`f` distinguishing witnesses. Frozen evidence preserves a real commit and eliminates alternatives. The fallback receives only that value if a fast commit exists, and otherwise resolves competing adopted values. See [[knowledge-census-recovery]] for named lemmas and proof scope.

## Liveness intuition
Either healthy witnesses eventually supply the requirement, or conflict/freeze/failure suspicion triggers a census with `n-f` reachable voters. Adopt-commit then returns. The fallback's eventual leader/progress assumptions complete consensus. False suspicion harms fast-path performance but does not break safety (Theorems A.20–A.23).

## Strengths
- Synthesizes different strategies for different proposer locations and rates.
- Trades evidence dissemination against quorum size using explicit compatibility checks.
- Separates requirement safety from communication-graph optimization.
- Supports direct non-voting clients and delegated execution.
- Proves stable single-proposer weighted-latency optimality under the paper's model (§5; Appendix B).

## Weaknesses
- Requirements depend on a topology/rate estimate and must be installed consistently.
- Evidence tracking and dissemination add CPU/memory costs; optimizer work grows with deployment size.
- Same-slot conflicts require adoption and fallback, and can substantially increase tail latency.
- Mathematical core proofs do not by themselves verify early-freeze optimizations, dynamic installation, durable restart, or the complete KSMR runtime.

## Differences from related protocols
[[FastPaxos]] and [[SwiftPaxos]] encode particular fast-quorum patterns; KCensus searches across acceptance-evidence patterns. [[EPaxos]]/[[Atlas]] agree on dependency metadata, whereas KSMR agrees on log-slot commands and uses sharding. [[FPaxos]] changes cross-phase quorum geometry without this second-order witness matrix. [[Jetpack]] is a host shim with view barriers, not a requirement optimizer. [[CURP]]'s dedicated unordered durability witnesses are distinct from KCensus voting witnesses. These are comparisons of the ingested mechanisms, not claims that their recovery protocols are interchangeable.

## Open questions
The paper describes but does not fully specify requirement-installation fencing, the optimized early-freeze/Paxos-overlap refinement, and proposer-tagged value identity. Restart persistence, log catch-up/compaction, exact read evidence, and deduplication require artifact inspection. Track these in [[unresolved-confusions]].

## Sources
[[KCensus-2026]] (`raw/KCensus.pdf`, arXiv v2): §§2–3 assumptions/requirements; Algorithms 1–6 core; §5 synthesis; §6 KSMR; Appendix A correctness and Algorithm 7; Appendix B optimality; Appendix C characterization. Related-protocol comparisons use their corresponding wiki notes.
