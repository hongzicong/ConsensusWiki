---
type: comparison-family
family: fast consensus
protocols: [FastPaxos, EPaxos, Atlas, SwiftPaxos, Pando, Rabia]
tags: [fast-path]
---

# Fast Consensus

## Family overview
Fast consensus protocols reduce common-case message delays by collecting stronger evidence early.

## Shared mechanism
A fast path succeeds only when evidence is strong enough for later recovery: same value, same dependencies, recoverable dependency union, same dependency paths, or enough chosen splits.

[[Rabia]] adds a randomized variant: the fast path can also succeed by deciding that no concrete request should occupy the current slot, returning `⊥` and retrying proposals later.

## Modeling implications
Define the fast evidence predicate before modeling optimizations.

[[Atlas]] is a useful reminder that "unambiguous" need not mean identical replies. Its fast path permits differing dependency reports when every dependency in the final union is reported by at least `f` fast-quorum members.
