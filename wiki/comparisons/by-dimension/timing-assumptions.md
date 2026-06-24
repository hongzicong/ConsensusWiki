---
type: comparison-dimension
dimension: timing assumptions
protocols: [FastPaxos, EPaxos, EPaxosStar, SwiftPaxos, Pando]
tags: [timing, liveness]
---

# Timing Assumptions

## What this dimension means
Timing assumptions separate safety, which is usually asynchronous, from progress claims that need synchrony, partial synchrony, stable leadership, or available quorums.

## Why it matters
A model that bakes timing into safety can prove the wrong theorem, while a model with no timing assumptions may be unable to state the intended liveness result.

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Fast and classic rounds | Asynchronous safety; progress needs favorable conditions/coordinator recovery | Safety is quorum-based, not timing-based | Collisions and failures require eventual recovery progress | Keep safety invariant independent of timers | [[FastPaxos-2006]] |
| [[EPaxos]] | Command leaders with PreAccept/Accept recovery | Asynchronous safety; progress with available quorums and recovery | Timing only affects which dependencies are observed | Conflict timing affects fast-path rate and execution latency | Model latency metrics separately from commit safety | [[EPaxos-2013]], [[EPaxos-Revisited-2021]] |
| [[EPaxosStar]] | Synchronous fast path with validated recovery | Partial synchrony; `e`-faulty synchronous fast guarantee | Agreement/visibility must hold regardless of timing | Fast execution by `t + 2 Delta` is conditional on the fast-run assumptions | State `Delta` only in liveness/performance lemmas | [[Making-Democracy-Work-2025]] |
| [[SwiftPaxos]] | Stable ballot leader with fast/slow ack paths | Asynchronous safety; eventual stable ballot for progress | Dependency safety is independent of message delays | SlowAck/Sync help progress after disagreement | Model eventual leader stability as an assumption, not a safety premise | [[SwiftPaxos-2024]] |
| [[Pando]] | Quorum reads/writes over geo-storage sites | Quorum safety; progress with available quorums | Intersection, not timing, preserves chosen values | Latency benefits depend on nearby available quorums and write-back | Separate quorum availability from network-delay optimization | [[Pando-2020]] |

## Main patterns
Safety claims are mostly quorum/intersection claims; timing assumptions enter liveness, fast-path availability, and performance.

## Important exceptions
[[EPaxosStar|EPaxos*]] explicitly states an `e`-faulty synchronous fast-path guarantee, so its timing assumptions are part of the fast-path performance theorem.

## Common pitfalls
Do not use timeout behavior as evidence that a value is safe to recover unless the paper explicitly proves that connection.

## Relevance to new protocol design
State safety invariants without timing first, then add liveness assumptions as separate hypotheses.

## Open questions
- TODO: Extract exact timing theorem statements for each paper that gives formal liveness or latency bounds.

## Related pages
[[liveness]], [[fast-paths]], [[leader-roles]], [[protocol-catalog]]
