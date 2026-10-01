---
type: comparison-family
family: fast consensus
protocols: [FastPaxos, GPaxos, EPaxos, Atlas, SwiftPaxos, Pando, Rabia, CURP, Copilot, Jetpack, KCensus]
tags: [fast-path]
---

# Fast Consensus

## Family overview
Fast consensus protocols reduce common-case message delays by collecting stronger evidence early.

[[GPaxos]] is the bridge between [[FastPaxos]] and dependency-based SMR protocols: its fast evidence can be a compatible set of command histories rather than one identical value.

[[CURP]] is included as a boundary case rather than a Paxos-family protocol: it achieves 1 RTT primary-backup updates through unordered witness durability and commutativity, not through ordered consensus fast quorums.

[[Copilot]] is a dual-leader fast-consensus variant: each pilot can fast-commit one cross-log dependency, while ping-pong batching makes their concurrent proposals compatible and fast takeover prevents execution from waiting on a slow pilot.

[[Jetpack]] is a plugin boundary case: it layers a Fast-Paxos-style superquorum over an independent host protocol. Same-view proposer promises give the fast result, while a majority-agreed recovery set and stability marker carry the promise across host view changes.

## Shared mechanism
A fast path succeeds only when evidence is strong enough for later recovery: same value, compatible c-structs, same dependencies, recoverable dependency union, same dependency paths, or enough chosen splits.

[[Rabia]] adds a randomized variant: the fast path can also succeed by deciding that no concrete request should occupy the current slot, returning `⊥` and retrying proposals later.

## Modeling implications
Define the fast evidence predicate before modeling optimizations.

[[Atlas]] is a useful reminder that "unambiguous" need not mean identical replies. Its fast path permits differing dependency reports when every dependency in the final union is reported by at least `f` fast-quorum members.

## Synthesized evidence strategies

[[KCensus]] makes the recoverability evidence itself a synthesis variable: each proposer has a [[knowledge-requirement]], pairs must retain more than `f` witnesses of intersection acceptances, and an `n-f` frozen census supplies safe fallback inputs. It can choose different dissemination patterns by topology and objective. Unlike a fixed larger-quorum recipe, membership cardinality alone cannot validate a strategy. Its stable single-proposer optimality theorem does not promise the best conflict/recovery latency ([[KCensus-2026]], §§3–5, Appendix B).
