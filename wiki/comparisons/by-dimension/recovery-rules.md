---
type: comparison-dimension
dimension: recovery rules
protocols: [FastPaxos, EPaxos, EPaxosStar, SwiftPaxos, Pando]
tags: [recovery]
---

# Recovery Rules

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Higher round chooses pickable safe value | Phase 1 evidence from quorum | Preserves possible lower chosen value | Stable coordinator helps | Encode pickable predicate | [[FastPaxos-2006]] |
| [[EPaxos]] | Explicit Prepare / TryPreAccept | Majority evidence | Preserves safe tuple per instance | Timeouts recover failed leaders | Distinguish pre-accepted vs accepted | [[EPaxos-2013]] |
| [[EPaxosStar]] | Recover evidence plus validation of possible fast-path dependencies | Recovery quorum `n - f`; possible fast evidence from at least `Q_size - e` members | Preserves agreement and dependency visibility; invalidating commands force `Nop` | `Waiting` messages break recovery cycles; per-command leader detector eventually stabilizes | Treat validation as evidence gathering, not tentative pre-accept | [[Making-Democracy-Work-2025]] |
| [[SwiftPaxos]] | NewLeaderAck plus Sync | Recovery majority | Carries prior accepted dependency paths | Eventually stable ballot | Sync can add/remove deps | [[SwiftPaxos-2024]] |
| [[Pando]] | Phase 1b reconstructs prior value | `k` splits in intersection | Later proposer recovers chosen value | Available 1b and 2 quorums | Value identity and split count | [[Pando-2020]] |
