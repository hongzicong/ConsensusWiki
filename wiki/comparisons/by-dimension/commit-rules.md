---
type: comparison-dimension
dimension: commit rules
protocols: [FastPaxos, EPaxos, EPaxosStar, Mencius, SwiftPaxos, Pando]
tags: [commit-rule, quorum, fast-path, recovery]
---

# Commit Rules

## What this dimension means
A commit rule states when a value, command, dependency set, or storage version is known strongly enough that later recovery must preserve it.

## Why it matters
Commit predicates are the bridge between protocol message evidence and [[agreement]], [[recoverability]], and executable formal models.

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | A value is chosen in a round when a quorum for that round accepts it; fast rounds use fast quorums | Fast-round collision-free common case, with recovery after collisions | Later rounds must select a value safe with respect to possibly chosen lower-round values | Progress needs a coordinator/recovery path after collision | Model `any` and the safe phase-2a value predicate separately | [[FastPaxos-2006]] |
| [[EPaxos]] | Fast-commit on matching PreAccept attributes; otherwise Accept/Commit after majority evidence | Matching `(cmd, seq, deps)` for fast path or majority in slow path | Commits one tuple for an instance and orders interfering commands through dependencies | Slow path recovers when fast evidence disagrees or is incomplete | Separate commit from execution readiness; dependencies may delay execution | [[EPaxos-2013]], [[EPaxos-Revisited-2021]] |
| [[EPaxosStar]] | Fast-commit when a quorum of size `n - e` returns dependency sets equal to `initDep[id]`; recovery may commit `Nop` | Optimized bound `n >= max{2e + f - 1, 2f + 1}` | Validation preserves agreement and visibility among conflicting commands | `Waiting` messages and eventual recovery coordinator stability avoid recovery cycles | Model validation as part of the commit/recovery evidence, not as an optimization | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | In-order commit after learning an instance and all previous instances; optional out-of-order commit for commutable requests | Simple consensus restricts non-coordinators to `no-op`; out-of-order commit requires application commutativity | Preserves a single learned sequence, or equivalent orders for commutable operations | `SKIP` and revocation fill gaps that would otherwise block commit | Separate chosen, learned, in-order committed, and out-of-order executable | [[Mencius-2008]] |
| [[SwiftPaxos]] | Commit/execute from matching dependency-path evidence; SlowAck and Sync repair disagreement | Leader-including fast quorum or slow quorum evidence | Preserves agreed dependencies and acyclic committed dependency graph | Fallback is available when fast evidence does not match | Track dependency paths, not only direct dependency sets | [[SwiftPaxos-2024]] |
| [[Pando]] | A write is chosen when enough Phase 2 quorum evidence exists; reads can return when chosen value is reconstructible | Erasure-coded quorum intersections recover enough splits | Later proposals/read repair must preserve chosen value identity and reconstructability | Progress needs available Phase 1b/Phase 2 quorums | Distinguish value identity from individual coded splits | [[Pando-2020]] |

## Main patterns
Fast commit rules need stronger evidence than slow commit rules because recovery must be able to distinguish a real commit from a merely possible partial execution.

## Important exceptions
[[Pando]] is not an SMR command protocol; its commit rule is about erasure-coded storage versions rather than command execution order.

## Common pitfalls
Do not treat "committed" and "ready to execute" as the same state in dependency-based SMR protocols.

## Relevance to new protocol design
Define the commit predicate before choosing quorum sizes; otherwise recovery evidence may be too weak to prove [[agreement]].

## Open questions
- TODO: Extract exact pseudocode line numbers for [[SwiftPaxos]] and [[Pando]] commit preconditions.

## Related pages
[[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[recovery-rules]], [[agreement]], [[recoverability]]


