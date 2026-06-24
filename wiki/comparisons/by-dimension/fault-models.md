---
type: comparison-dimension
dimension: fault models
protocols: [FastPaxos, EPaxos, EPaxosStar, SwiftPaxos, Pando]
tags: [failure-model]
---

# Fault Models

## What this dimension means
The fault model states which failures the protocol tolerates and which components may fail without violating safety.

## Why it matters
Fault assumptions determine quorum sizes, recovery evidence, and which liveness claims can be modeled.

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Acceptors may fail non-Byzantinely | Non-Byzantine faults; safety independent of timing | Quorum intersection protects chosen values despite failed acceptors | Progress requires enough live acceptors and eventual recovery coordination | Parameterize acceptor count and failure budget | [[FastPaxos-2006]] |
| [[EPaxos]] | Replicas are both acceptors and command leaders | Non-Byzantine replica failures | Majority/fast quorum evidence preserves one safe tuple per instance | Any surviving replica can lead new commands or recovery if quorums are available | Model failed command leaders separately from failed quorum members | [[EPaxos-2013]] |
| [[EPaxosStar]] | Separates full crash resilience `f` from fast-path failure budget `e` | Non-Byzantine crash failures | Bound `n >= max{2e + f - 1, 2f + 1}` supports validated recovery | Fast path is guaranteed under at most `e` failures in the synchronous fast run; slow/recovery handles up to `f` | Keep `e` and `f` as distinct parameters | [[Making-Democracy-Work-2025]] |
| [[SwiftPaxos]] | Replicas fail non-Byzantinely | Non-Byzantine faults | Leader-including fast quorums and slow quorums preserve dependency evidence | Eventual stable ballot/leader needed for progress | Record leader failure separately from dependency evidence loss | [[SwiftPaxos-2024]] |
| [[Pando]] | Storage/data sites may fail | Non-Byzantine data-site failures | Intersecting quorums preserve enough coded splits of chosen values | Reads/writes need available Phase 1b/Phase 2 quorums | Model failed sites as missing splits, not corrupted values | [[Pando-2020]] |

## Main patterns
All currently ingested protocols assume non-Byzantine failures.

## Important exceptions
No BFT consensus paper has been ingested yet; Byzantine assumptions should remain out of protocol pages until sourced.

## Common pitfalls
Do not infer Byzantine tolerance from erasure coding or from quorum intersection alone.

## Relevance to new protocol design
Choose the fault model before choosing quorum formulas; changing the fault model changes both safety and recovery obligations.

## Open questions
- TODO: Ingest at least one BFT protocol before populating the [[bft-consensus]] family page.

## Related pages
[[failure-model]], [[quorum-systems]], [[recovery-rules]], [[protocol-catalog]]
