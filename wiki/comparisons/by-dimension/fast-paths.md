---
type: comparison-dimension
dimension: fast paths
protocols: [FastPaxos, FPaxos, OmniPaxos, GPaxos, EPaxos, EPaxosStar, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia, CURP, Hermes, Copilot, Avicenna, Bodega, Jetpack, Hydra, HydraPaxos, WPaxos]
tags: [fast-path]
---

# Fast Paths

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Acceptors vote after `phase2a any` | No collision for two-delay learn | Collision recovery must pick safe value | Frequent collisions hurt progress | Model `any` separately | [[FastPaxos-2006]] |
| [[FPaxos]] | Ordinary stable-leader Phase 2 over a smaller or structured `Q2` | Every valid recovery `Q1` intersects every valid deciding `Q2` | Keeps Paxos agreement while permitting same-phase disjoint quorums | Smaller `Q2` can improve common latency/load but larger `Q1` makes leader recovery harder | This is not a leader-bypassing fast round or fewer-message-delay path | [[Flexible-Paxos-2016]], §§3-5 |
| [[OmniPaxos]] | Stable Sequence Paxos Accept phase pipelines commands through one leader-to-majority round trip | Prepared QC leader and promised majority | Ordinary majority acceptance preserves a growing prefix | Partial connectivity is handled by BLE/Prepare rather than a second commit predicate | Non-example: pipelined common path, not a fast quorum or leader bypass | [[Omni-Paxos-2023]], §§4.1.2, 7.1 |
| [[GPaxos]] | Proposers send commands directly to acceptors in fast ballots; learners lub compatible c-struct evidence | Concurrent commands are non-interfering/compatible at the c-struct level | Recovery must preserve lower-ballot choosable c-structs, not just equal values | Interfering commands collide and need higher-ballot leader recovery | Model compatibility and lub/glb explicitly; equality is too strong | [[Generalized-Paxos-2005]] |
| [[EPaxos]] | Matching PreAccept attributes | Non-conflicting or identically observed conflicts | Same `(cmd, seq, deps)` committed | Falls back to Accept; execution may still wait on dependencies | Interference relation drives deps; measure execution latency separately | [[EPaxos-2013]], [[EPaxos-Revisited-2021]] |
| [[EPaxosStar]] | Matching `PreAcceptOK` dependency sets equal to `initDep[id]` | Conflict-free command and at most `e` fast-path failures in synchronous run | Same `(cmd, dep)` plus visibility for conflicting commands | Executes at submitter by `t + 2 Delta` under the `e`-fast condition | Do not add EPaxos sequence numbers; model dependency SCC execution | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | Local rotating coordinator path; one-way `SKIP` learning for `no-op` | Each instance has one coordinator allowed to propose non-`no-op` | Avoids Fast Paxos-style collisions by construction | Idle coordinators must skip or be revoked to avoid gaps | Do not model `SKIP` as a fast quorum; it is owner evidence under simple consensus | [[Mencius-2008]] |
| [[PigPaxos]] | Normal Multi-Paxos path through relay aggregation | Stable leader and live majority; relays are transport helpers | No additional fast-path safety burden beyond Paxos majority acceptance | Reduces leader load but can add relay-hop latency | Do not classify relay aggregation as a Fast Paxos-style fast quorum | [[PigPaxos-2021]] |
| [[Atlas]] | Dependency union over a fast quorum of size `floor(n/2) + f` | Every dependency in `union_Q dep` appears in at least `f` fast-quorum replies | Makes the fast proposal recoverable after `f` failures | For `f = 1`, every command can take the fast path; higher `f` may fall back | Model `union_Q dep = union^f_Q dep`, not matching replies | [[Atlas-2020]] |
| [[SwiftPaxos]] | Matching FastAck dependency paths | Fast quorum agrees with leader | Same deps and acyclic graph | SlowAck fallback | Dependency paths matter | [[SwiftPaxos-2024]] |
| [[Pando]] | Phase 1a fast read; delegated write overlap | Chosen value detectable from nearby quorum | Reads only return chosen values | Phase 1b/write-back fallback | Split reconstruction predicate | [[Pando-2020]] |
| [[Rabia]] | Weak-MVC terminates after Proposal/State/vote exchanges | All replicas have same proposal, or no proposal has a majority | Binary agreement returns one request or `⊥` for the slot | Stable networks keep PQ heads aligned and reduce `⊥` slots | Count fast path as three message delays; weak validity permits `⊥` | [[Rabia-2021]] |

| [[CURP]] | Client records request at all witnesses while master speculatively executes | Operation is commutative with all unsynced operations and all `f` witnesses accept | Recovery can replay from any one selected witness without changing visible results | Conflicts or witness failures fall back to backup sync | Model fast evidence as unordered durability, not as ordered consensus acceptance | [[CURP-2019]] |
| [[Hermes]] | Local `Valid` reads and `INV -> ACK` client-visible writes from any replica; `VAL` follows off path | Every current live member acknowledges the timestamp; RM epoch is stable | All authorized readers are invalidated before completion, and replay can reproduce the exact write | Message loss replays; member failure waits for RM; ordinary conflicts do not cause fallback | Boundary case: fast normal path, but no smaller fast quorum or conflict slow path | [[Hermes-2020]], §§3.1-3.4 |
| [[Copilot]] | Pilot fast-commits an initial cross-log dependency after `f + floor((f + 1) / 2)` compatible `FastAcceptOk` replies | Replicas have not accepted an incompatible later entry from the other pilot; ping-pong batching usually aligns proposals | Initial dependency is recoverable, with ambiguous partial evidence resolved through the other log | Incompatibility or insufficient fast replies triggers majority Accept; slow pilot work can be taken over | Separate fast commit from execution readiness and from takeover of unresolved predecessors | [[Copilot-2020]], §§3.2–3.4 |
| [[Avicenna]] | Ordinary real-leader `Accept`/`AcceptOK` normal path; independent sampled shadow processing | Stable live real leader and `f + 1` responsive non-standbys | Single leader assigns one value per index; shadow log never executes | Slow real leader or stopped commits trigger phase rotation, not a conflict slow path | Do not classify the two-message-delay Multi-Paxos normal path or fast rotation as a Fast Paxos-style fast quorum | [[Avicenna-2026]], §§4.1, 4.3 |
| [[Bodega]] | One-server local read at a stable, caught-up key responder; optional `m` `AcceptNote`s release a read before Commit arrives | Majority roster leases, threshold catch-up, responder-covering writes, and newest key write committed or inevitably committing | Local response observes every prior acknowledged write without changing write consensus | Uncertain reads hold or redirect; failed responders require higher-roster lease revocation/expiration | This is a client-read fast path, not a Fast Paxos-style value-selection path | [[Bodega-2026]], §§3.2-3.3, 4.3 |
| [[Jetpack]] | Client broadcasts to shim replicas while all host proposers concurrently insert the command; first successful path returns | Same-view, conflict-free acknowledgements from `f + ceil(f/2) + 1` including every proposer; host satisfies PR 1/PR 2 | Original path promises no concurrent conflict before the fast command and later commits it; recovery preserves promise/order | Any rejection/missing evidence uses already-running host path; view change recovers before resume | Model fast view separately from host view and include the proposer-membership constraint | [[Jetpack-2026]], §§3-5 |
| [[Hydra]] | One sequencer stamp plus receiver-local frontier delivery | Traffic or flushes advance every sequencer frontier; clocks never go backward | Gives order/drop evidence but no application commit | Idle sequencers need flushes; failed sequencers need reconfiguration | Boundary case: fast ordering primitive, not consensus fast path | [[Hydra-2023]], §§4.3-4.4 |
| [[HydraPaxos]] | Client groupcasts; replicas log; leader executes; client waits for consistent majority including leader | Stable configuration, no drop, live `f + 1` replicas | Majority durability plus Hydra order supports one-RTT completion | Drop notification triggers recovery or consensus `NO-OP` | Not a Fast Paxos quorum; ordering is pre-established by the network | [[Hydra-2023]], §6 |
| [[WPaxos]] | Current object owner repeats Phase 2 over nearby `Q2` without rerunning Phase 1 | Stable ownership, available local/nearby `Q2`, no higher ballot | Ordinary Paxos cross-phase intersection protects every commit | Remote first access or locality shift triggers WAN `Q1` object stealing | Non-example: low-latency stable-leader path, not leader bypass or fast quorum | [[WPaxos-2020]], §§3.2-4.3 |

## Main patterns
The fast path is safe only when the fast evidence already contains enough information for any later recovery to avoid incompatible choices.

## FastPaxos vs GPaxos vs EPaxos vs SwiftPaxos
| Dimension | [[FastPaxos]] | [[GPaxos]] | [[EPaxos]] | [[SwiftPaxos]] |
|---|---|---|---|---|
| Object agreed on | One value in one consensus instance | Growing compatible c-struct / command history | Command plus `(seq, deps)` attributes | Command plus dependency paths |
| Who proposes | Proposers send values directly after a fast round is opened with `phase2a any` | Proposers send commands directly to acceptors in a fast ballot | Any replica can be the command leader for its own instance | Clients propagate commands; fast-quorum replicas propose dependencies, with a fixed ballot leader central to agreement |
| Fast evidence | Same value accepted by a fast quorum | Compatible c-structs from a fast quorum containing the command | Matching `PreAcceptOK` attributes from a fast quorum | Matching `FastAck` dependency-path evidence from a leader-including fast quorum |
| Conflict condition | Different proposed values collide | Interfering commands produce incompatible c-structs | Interfering commands cause different `seq`/`deps` observations | Disagreement with the leader's dependency proposal triggers repair |
| Fallback | Recovery/classic round selects a safe value | Higher ballot selects a safe c-struct using `ProvedSafe` | Accept phase with majority after unioning deps and raising seq | `SlowAck` can adopt the leader's proposal; slow quorum fallback also exists |
| Main safety burden | Quorum triple-intersection and safe phase-2a value selection | Compatibility, lub/glb algebra, and safe c-struct selection | Same tuple per instance plus dependency ordering of interfering commands | Same dependencies, dependency-path recovery, and acyclic committed dependency graph |
| Modeling pitfall | Treating `any` as a normal value | Replacing compatibility with equality or total-order prefixes | Counting commit latency while ignoring execution waiting on dependencies | Modeling direct deps only instead of dependency paths and acyclicity |

[[Atlas]] is closest to [[EPaxos]] in message shape, but its fast predicate is weaker than exact metadata matching and stronger than arbitrary disagreement: the final dependency union must be reconstructible because every included dependency appears at least `f` times.

Summary: [[FastPaxos]] is a fast single-value consensus mechanism; [[GPaxos]] generalizes fast consensus to compatible command histories; [[EPaxos]] turns commutativity into per-command SMR by committing ordering metadata; [[Atlas]] tunes dependency fast quorums around a small site-failure budget; [[SwiftPaxos]] keeps dependency-based SMR but adds leader-including fast quorums and SlowAck repair to make disagreement cheaper to resolve.

[[Rabia]] is a different fast-path shape: it does not make a two-delay Paxos-style decision. Its common case is three message delays, and the fast decision may be `⊥` when replicas do not observe a majority proposal.

[[CURP]] is another boundary case: its 1 RTT path is not a consensus fast quorum, but unordered witness durability combined with master-side speculative execution and commutativity.

[[Hermes]] is a write-all boundary case. Its one-exposed-RTT write is the normal `INV -> ACK` path over every live member, not an optimistic subset. Conflict never selects a slow path for ordinary writes; only failures, invalid states, and RMW races change behavior.

[[Copilot]] is a dual-leader dependency case: its fast evidence concerns one cross-log prefix dependency per entry, while its no-slowdown fast-path rate is improved by ping-pong batching rather than application commutativity.

[[Avicenna]] is another boundary case. It matches Multi-Paxos normal-case latency but has no conflict-dependent fast/slow consensus split; its optimization is fast leader rotation obtained from a restricted acceptor universe.

[[Bodega]] is a read-oriented boundary case. Its write commits remain classic leader-based consensus plus responder coverage; the fast path is the absence of network messages on a safe local read.

[[Jetpack]] turns fast consensus into a layer. Its early evidence is not sufficient by itself: all host proposers must make a same-view ordering promise, and a later recovery-set marker must preserve that promise across views.

[[FPaxos]] belongs here as a non-example. It can make the normal Multi-Paxos replication quorum faster to collect, but the protocol still uses leader-driven Phase 2 and ordinary higher-ballot Phase 1 recovery.


