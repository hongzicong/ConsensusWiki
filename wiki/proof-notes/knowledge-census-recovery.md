---
type: proof-note
protocols: [KCensus]
tags: [adopt-commit, knowledge, recoverability, indistinguishability]
---

# Knowledge Census Recovery

Source: [[KCensus-2026]], §3.1, Algorithms 1–7, Appendices A–C. This note extracts the paper's arguments; it is not a mechanized proof or a verification of the implementation.

## Obligation and assumptions

Adopt-commit agreement is conditional: if any call commits `v`, every returning call adopts or commits `v`. If none commits, adopters may disagree. Termination and proposal validity are separate obligations. The template assumes `n ≥ 2f+1`, crash faults, reliable FIFO links between correct processes, atomic handlers between waits, and constant valid/pairwise-compatible normal-form requirements. Eventual crash suspicion is used for termination, not safety.

## Evidence invariants

1. Each process changes `accepted` at most once, from `⊥` to a proposed value (Observation A.4 and provenance from A.1).
2. All acceptors named in local knowledge accepted that process's value (A.5).
3. Knowledge entries only grow (A.6).
4. A remote view of witness `w`'s knowledge is contained in `w`'s final local knowledge (Corollary A.7).
5. A fast commit implies every member of its requirement key set accepted the value (Lemma A.8; Corollary A.9).
6. Freeze ends all further acceptance/evidence growth at that witness for this instance. A frozen snapshot covers any earlier record on which a proposer could rely.

These are not satisfied by a set of value votes alone: the origin and meaning of each witness record matter.

## Census argument

Let `C` contain the `n-f` distinct frozen respondents. Any required set `Q` has `|Q| > f`, so `C ∩ Q ≠ ∅`. For two requirements, let `W` be the union of witnesses required to observe their quorum intersection. Compatibility gives `|W| > f`, hence `C ∩ W ≠ ∅` (Lemma A.11).

Suppose `v` fast-committed:

- **Presence:** a respondent in `C ∩ Q_v` reports `v` (Lemma A.13).
- **Preservation:** no frozen report can disprove `v`. Required witnesses have all needed evidence, and no reported other-value acceptor can belong to `Q_v` (Lemma A.14).
- **Exclusion:** for candidate `u != v`, take a distinguishing witness in `C ∩ W`. If its obligation comes from `R_v`, its report exposes an acceptor in `Q_u` that accepted `v`, triggering Algorithm 6's first test. If its obligation comes from `R_u`, it either reports a different accepted value or lacks required evidence for `u`, triggering the first or second test (Lemma A.12).

The logic covers a later fast commit too: a frozen witness cannot subsequently fill missing evidence, but already sufficient evidence can still enable commitment. Do not replace freeze with a global prohibition on committing.

## Composition and termination

Theorems A.20–A.22 establish adopt-commit termination, validity, and agreement. Algorithm 7 maps commits to decisions and adopts to fallback proposals. If no fast commit exists, fallback agreement decides; if one exists, all fallback inputs equal it and fallback validity preserves it (Theorem A.23). Do not model `Adopt` as `Decide`, or require equal adopted values in executions with no commit.

## Lower bound and synthesis

The recoverability argument compares states with different commits: they must differ at more than `f` processes, or crashing the differing processes leaves indistinguishable survivors unable to choose the required different continuations (§3.1).

Appendix B derives normal-form requirements from each proposer's causal message history in a single-proposer execution. Too few required processes or distinguishing witnesses permit executions violating agreement through indistinguishability. Under fixed known latency, permanent faults, perfect failure detection in the stable state, and the weighted single-proposer objective, Theorem B.1 concludes optimality among deterministic consensus protocols. The proof explicitly explains how delayed/suppressed messages preserve FIFO and eventual delivery to correct processes. Theorems C.1–C.2 state sufficiency/necessity for recoverable fast paths using this construction.

This argument does not establish optimal processing cost, conflict latency, or recovery duration. Nor does it transfer correctness to an arbitrary protocol that merely shares quorum cardinalities.

## Implementation gaps to preserve

- Keep proposer identity attached to values when selecting `REQUIREMENTS[v.proposer]`.
- Count unique voting respondents; non-voting clients do not contribute to the failure bound or census.
- Maintain a fixed requirement configuration per modeled instance; §6's dynamic table installation needs an explicit boundary/refinement.
- Verify that compact graph messages, early freeze, continued forwarding, and Paxos piggybacking preserve the abstract evidence invariants before reusing the core proof.
- The mathematical appendices do not substitute for a kernel-checked model/proof of full KSMR.

## Related pages
[[KCensus]], [[knowledge-requirement]], [[recoverability]], [[quorum-intersection]], [[adopt-commit-abstraction]], [[proof-techniques]], [[unresolved-confusions]]
