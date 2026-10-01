---
type: concept
tags: [knowledge, quorum, recoverability, fast-path]
---

# knowledge-requirement

In [[KCensus]], a knowledge requirement `R_p` describes what proposer `p` must know about witnesses' acceptance records before fast commitment. It maps each required witness `w` to a set of acceptors `R_p[w]`. The witness must have recorded those acceptances, and `p` must learn that it has done so ([[KCensus-2026]], §3.1).

## Normal form and validity

```text
Q_p = R_p.keys()
w ∈ R_p[w] ⊆ Q_p for every w ∈ Q_p
valid(R_p) iff |Q_p| > f
```

Thus every required witness also accepts the value. Requirement validity is a resilience constraint, distinct from the consensus property [[validity]].

## Pairwise compatibility

For `I = Q_p ∩ Q_q`, define `W_p = {w ∈ Q_p : R_p[w] ∩ I ≠ ∅}` and similarly `W_q`. Requirements are compatible iff `|W_p ∪ W_q| > f`. Count each witness once, even if it appears in both sets or witnesses several intersection acceptors. This implies nonempty quorum intersection but allows an intersection smaller than `f+1` when other processes record its acceptances.

Every `n-f` recovery census intersects this distinguishing-witness set. Frozen reports either preserve the fast value or eliminate an incompatible candidate; quorum size alone does not capture this guarantee (§4.4; [[knowledge-census-recovery]]).

## Concrete example

For `n=5, f=2`, the paper uses:

```text
R_1 = {1:{1,2,5}, 2:{2}, 5:{5}}
R_4 = {4:{3,4,5}, 3:{3}, 5:{5}}
I = {5}
W_1 = {1,5}; W_4 = {4,5}
|W_1 ∪ W_4| = 3 > 2
```

Both quorums have three members, yet their one-member intersection is sufficient because processes 1 and 4 add witnesses outside it (§4.3). This is an example of a jointly compatible pair, not a full five-proposer configuration.

## Modeling cautions

- Record actual first-order evidence separately from another process's knowledge of it.
- Do not merge evidence for different values, slots, or requirement configurations.
- Requirements are constant within the displayed one-shot instance.
- Non-voting clients are excluded from required witness and acceptor sets.
- [[witness]] also describes CURP's different durability role; the terms are not interchangeable.

## Related pages
[[KCensus]], [[KCensus-2026]], [[recoverability]], [[quorum]], [[quorum-intersection]], [[knowledge-census-recovery]], [[adopt-commit-abstraction]]
