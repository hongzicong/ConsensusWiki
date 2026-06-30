---
type: comparison-dimension
dimension: fast paths
protocols: [FastPaxos, EPaxos, EPaxosStar, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia]
tags: [fast-path]
---

# Fast Paths

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Acceptors vote after `phase2a any` | No collision for two-delay learn | Collision recovery must pick safe value | Frequent collisions hurt progress | Model `any` separately | [[FastPaxos-2006]] |
| [[EPaxos]] | Matching PreAccept attributes | Non-conflicting or identically observed conflicts | Same `(cmd, seq, deps)` committed | Falls back to Accept; execution may still wait on dependencies | Interference relation drives deps; measure execution latency separately | [[EPaxos-2013]], [[EPaxos-Revisited-2021]] |
| [[EPaxosStar]] | Matching `PreAcceptOK` dependency sets equal to `initDep[id]` | Conflict-free command and at most `e` fast-path failures in synchronous run | Same `(cmd, dep)` plus visibility for conflicting commands | Executes at submitter by `t + 2 Delta` under the `e`-fast condition | Do not add EPaxos sequence numbers; model dependency SCC execution | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | Local rotating coordinator path; one-way `SKIP` learning for `no-op` | Each instance has one coordinator allowed to propose non-`no-op` | Avoids Fast Paxos-style collisions by construction | Idle coordinators must skip or be revoked to avoid gaps | Do not model `SKIP` as a fast quorum; it is owner evidence under simple consensus | [[Mencius-2008]] |
| [[PigPaxos]] | Normal Multi-Paxos path through relay aggregation | Stable leader and live majority; relays are transport helpers | No additional fast-path safety burden beyond Paxos majority acceptance | Reduces leader load but can add relay-hop latency | Do not classify relay aggregation as a Fast Paxos-style fast quorum | [[PigPaxos-2021]] |
| [[Atlas]] | Dependency union over a fast quorum of size `floor(n/2) + f` | Every dependency in `union_Q dep` appears in at least `f` fast-quorum replies | Makes the fast proposal recoverable after `f` failures | For `f = 1`, every command can take the fast path; higher `f` may fall back | Model `union_Q dep = union^f_Q dep`, not matching replies | [[Atlas-2020]] |
| [[SwiftPaxos]] | Matching FastAck dependency paths | Fast quorum agrees with leader | Same deps and acyclic graph | SlowAck fallback | Dependency paths matter | [[SwiftPaxos-2024]] |
| [[Pando]] | Phase 1a fast read; delegated write overlap | Chosen value detectable from nearby quorum | Reads only return chosen values | Phase 1b/write-back fallback | Split reconstruction predicate | [[Pando-2020]] |
| [[Rabia]] | Weak-MVC terminates after Proposal/State/vote exchanges | All replicas have same proposal, or no proposal has a majority | Binary agreement returns one request or `⊥` for the slot | Stable networks keep PQ heads aligned and reduce `⊥` slots | Count fast path as three message delays; weak validity permits `⊥` | [[Rabia-2021]] |

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

[[Atlas]] is closest to [[EPaxos]] in message shape, but its fast predicate is weaker than exact metadata matching and stronger than arbitrary disagreement: the final dependency union must be reconstructible because every included dependency appears at least `f` times.

Summary: [[FastPaxos]] is a fast single-value consensus mechanism; [[EPaxos]] turns fast consensus into per-command SMR by committing ordering metadata; [[Atlas]] tunes dependency fast quorums around a small site-failure budget; [[SwiftPaxos]] keeps dependency-based SMR but adds leader-including fast quorums and SlowAck repair to make disagreement cheaper to resolve.

[[Rabia]] is a different fast-path shape: it does not make a two-delay Paxos-style decision. Its common case is three message delays, and the fast decision may be `⊥` when replicas do not observe a majority proposal.


