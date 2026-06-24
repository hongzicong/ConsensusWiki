---
type: comparison-family
family: fast consensus
protocols: [FastPaxos, EPaxos, SwiftPaxos, Pando]
tags: [fast-path]
---

# Fast Consensus

## Family overview
Fast consensus protocols reduce common-case message delays by collecting stronger evidence early.

## Shared mechanism
A fast path succeeds only when evidence is unambiguous: same value, same dependencies, same dependency paths, or enough chosen splits.

## Modeling implications
Define the fast evidence predicate before modeling optimizations.
