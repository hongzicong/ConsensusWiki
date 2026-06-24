---
type: comparison-dimension
dimension: fast paths
protocols: [FastPaxos, EPaxos, EPaxosStar, SwiftPaxos, Pando]
tags: [fast-path]
---

# Fast Paths

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Acceptors vote after `phase2a any` | No collision for two-delay learn | Collision recovery must pick safe value | Frequent collisions hurt progress | Model `any` separately | [[FastPaxos-2006]] |
| [[EPaxos]] | Matching PreAccept attributes | Non-conflicting or identically observed conflicts | Same `(cmd, seq, deps)` committed | Falls back to Accept; execution may still wait on dependencies | Interference relation drives deps; measure execution latency separately | [[EPaxos-2013]], [[EPaxos-Revisited-2021]] |
| [[EPaxosStar]] | Matching `PreAcceptOK` dependency sets equal to `initDep[id]` | Conflict-free command and at most `e` fast-path failures in synchronous run | Same `(cmd, dep)` plus visibility for conflicting commands | Executes at submitter by `t + 2 Delta` under the `e`-fast condition | Do not add EPaxos sequence numbers; model dependency SCC execution | [[Making-Democracy-Work-2025]] |
| [[SwiftPaxos]] | Matching FastAck dependency paths | Fast quorum agrees with leader | Same deps and acyclic graph | SlowAck fallback | Dependency paths matter | [[SwiftPaxos-2024]] |
| [[Pando]] | Phase 1a fast read; delegated write overlap | Chosen value detectable from nearby quorum | Reads only return chosen values | Phase 1b/write-back fallback | Split reconstruction predicate | [[Pando-2020]] |

## Main patterns
The fast path is safe only when the fast evidence already contains enough information for any later recovery to avoid incompatible choices.

## FastPaxos vs EPaxos vs SwiftPaxos
| Dimension | [[FastPaxos]] | [[EPaxos]] | [[SwiftPaxos]] |
|---|---|---|---|
| Object agreed on | One value in one consensus instance | Command plus `(seq, deps)` attributes | Command plus dependency paths |
| Who proposes | Proposers send values directly after a fast round is opened with `phase2a any` | Any replica can be the command leader for its own instance | Clients propagate commands; fast-quorum replicas propose dependencies, with a fixed ballot leader central to agreement |
| Fast evidence | Same value accepted by a fast quorum | Matching `PreAcceptOK` attributes from a fast quorum | Matching `FastAck` dependency-path evidence from a leader-including fast quorum |
| Conflict condition | Different proposed values collide | Interfering commands cause different `seq`/`deps` observations | Disagreement with the leader's dependency proposal triggers repair |
| Fallback | Recovery/classic round selects a safe value | Accept phase with majority after unioning deps and raising seq | `SlowAck` can adopt the leader's proposal; slow quorum fallback also exists |
| Main safety burden | Quorum triple-intersection and safe phase-2a value selection | Same tuple per instance plus dependency ordering of interfering commands | Same dependencies, dependency-path recovery, and acyclic committed dependency graph |
| Modeling pitfall | Treating `any` as a normal value | Counting commit latency while ignoring execution waiting on dependencies | Modeling direct deps only instead of dependency paths and acyclicity |

Summary: [[FastPaxos]] is a fast single-value consensus mechanism; [[EPaxos]] turns fast consensus into per-command SMR by committing ordering metadata; [[SwiftPaxos]] keeps dependency-based SMR but adds leader-including fast quorums and SlowAck repair to make disagreement cheaper to resolve.
