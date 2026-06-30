---
type: comparison-dimension
dimension: quorum systems
protocols: [FastPaxos, EPaxos, EPaxosStar, Mencius, SwiftPaxos, Pando]
tags: [quorum]
---

# Quorum Systems

## What this dimension means
Quorum systems define which evidence sets can choose, commit, recover, or read.

## Why it matters
Every fast path here buys latency by strengthening quorum intersections or metadata agreement.

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Classic and fast quorums | Any two quorums intersect; fast rounds require intersection of one arbitrary quorum with two fast quorums | Prevents two fast values surviving recovery | Fast progress needs nonfaulty fast quorum | Model `Quorum(i)` by round | [[FastPaxos-2006]] |
| [[EPaxos]] | Majority plus fast quorum over `N = 2F + 1` | Fast path requires matching attributes | Preserves one tuple per instance | Majority recovery | Separate command leader from quorum members | [[EPaxos-2013]] |
| [[EPaxosStar]] | Parameterized slow/recovery quorum `n - f` and fast quorum `n - e` | Optimized protocol requires `n >= max{2e + f - 1, 2f + 1}` | Recovery validation preserves agreement and visibility | `e` bounds fast-path failures; `f` bounds overall crash resilience | Model `e` and `f` independently | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | Paxos quorum plus owner-authored `SKIP` | Paper states quorum size `f + 1` with `n = 2f + 1`; `SKIP` is safe by simple-consensus value restriction | Quorum evidence preserves chosen non-`no-op`; coordinator `SKIP` proves `no-op` without majority agreement | Progress needs live quorum and revocation of suspected coordinators | Model `SKIP` as owner evidence, not as a quorum certificate | [[Mencius-2008]] |
| [[SwiftPaxos]] | Majority slow quorums; leader-including fast quorums | Fast quorum intersection size is greater than `N/2` | Preserves dependency-path agreement | Slow quorum fallback | Model fast quorum membership per ballot | [[SwiftPaxos-2024]] |
| [[Pando]] | Phase 1a, Phase 1b, Phase 2 quorums | 1a intersects 2 in one site; 1b intersects 2 in `k` splits | Recovers chosen erasure-coded values | Needs available 1b and 2 quorums | Distinguish value id from split count | [[Pando-2020]] |

## Main patterns
Fast paths need either larger quorums, leader inclusion, identical metadata, or a fallback quorum that can reconstruct prior choices.

## Fast quorum sizes
| Protocol | Common configuration | Fast quorum size |
|---|---|---|
| [[FastPaxos]] | `N = 3f + 1` acceptors | `2f + 1`; more generally, with fast quorum size `N - E` |
| [[EPaxos]] | `N = 2F + 1` replicas | `F + floor((F + 1)/2)` total, including the command leader; non-leader replies are one fewer |
| [[EPaxosStar]] | General `n, e, f` | `n - e`; optimized correctness requires `n >= max{2e + f - 1, 2f + 1}` |
| [[Mencius]] | `n = 2f + 1` servers | no Fast Paxos-style fast quorum; classic/recovery quorum is `f + 1`, while `SKIP` can be learned from the coordinator |
| [[SwiftPaxos]] C1 | `N = 2f + 1` replicas | any leader-including set with size `> 3N/4`, i.e. at least `floor(3N/4) + 1` |
| [[SwiftPaxos]] C2 | `N = 2f + 1` replicas | a unique fixed majority fast quorum of size `f + 1`, including the leader |
| [[Pando]] | Erasure-coded storage with split threshold `k` | no SMR fast quorum; Phase 1a fast-read/discovery quorum has size `max(k, f + 1)`, while Phase 1b/Phase 2 quorums are at least `f + k` |

Counting convention: these rows count total quorum membership unless explicitly saying "non-leader replies." This matters for [[EPaxos]], where the paper states the fast-path quorum size as including the command leader.

In [[EPaxosStar]], `f` and `e` are separate budgets. `f` is the total crash-failure resilience target, while `e <= f` is the number of failures under which conflict-free commands should still execute on the fast path. This is why the fast quorum is written as `n - e`: after `e` failures, exactly `n - e` processes remain available, so a fast quorum of size `n - e` is the largest quorum that can still be collected in the `e`-faulty fast run. Slow and recovery quorums use `n - f` because they must remain available under the full `f`-failure resilience target.

The boundary choice `e = f` is allowed, but it changes the optimized EPaxos* bound to `n >= max{3f - 1, 2f + 1}`. For `f >= 2`, the minimal configuration is `n = 3f - 1`, with fast quorum size `n - f = 2f - 1`. So `e = f` minimizes the fast quorum for fixed `n`, but the lower bound may force `n` upward.

For [[FastPaxos]] with `N = 2f + 1` acceptors and classic quorum size `f + 1`, the fast quorum can be defined, but it must have size at least `N - floor(f/2)`, equivalently `floor((3f + 1)/2) + 1 = f + floor((f + 1)/2) + 1`. This is one larger than the [[EPaxos]] fast quorum under the same `N = 2f + 1` counting convention, and it has the same size as [[SwiftPaxos]] C1 under `N = 2f + 1`. The SwiftPaxos C1 fast quorum must include the ballot leader; Fast Paxos fast quorums do not have this leader-inclusion requirement. The protocols use the quorum evidence for different safety arguments. This supports fast learning only while at most `floor(f/2)` acceptors are unavailable; after `f` failures, only the classic quorum size remains available.

## Important exceptions
PANDO's Phase 1a quorum is intentionally too small for full recovery; it is a fast read/discovery quorum.

## Common pitfalls
Do not treat EPaxos dependencies, SwiftPaxos dependency paths, and Fast Paxos collisions as the same mechanism.

## Relevance to new protocol design
Decide first what evidence recovery must reconstruct, then derive intersections.

## Open questions
- TODO: Add exact quorum-size derivations for Fast Paxos `N, F, E`.

## Related pages
[[quorum]], [[quorum-intersection]]



