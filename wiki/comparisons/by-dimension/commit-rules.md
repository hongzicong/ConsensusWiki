---
type: comparison-dimension
dimension: commit rules
protocols: [FastPaxos, FPaxos, OmniPaxos, GPaxos, EPaxos, EPaxosStar, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia, CURP, Hermes, Copilot, Avicenna, Bodega, Jetpack, Hydra, HydraPaxos, WPaxos]
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
| [[FPaxos]] | Value decided after every acceptor in one valid Phase 2 quorum accepts `(proposal, value)` | Chosen `Q2` belongs to a family intersecting every valid `Q1`; no majority or same-phase intersection required | Any later Phase 1 can discover and preserve the decision | Smaller `Q2` improves stable-leader progress but may enlarge recovery `Q1` | Commit predicate is ordinary Paxos acceptance with different quorum geometry, not a fast round | [[Flexible-Paxos-2016]], §§3, 5 |
| [[OmniPaxos]] | An index is chosen after a majority accepts it; leader advances `decidedIdx` and broadcasts `Decide`; the preceding prefix is decided too | Fixed configuration, FIFO replication order, and a prepared leader | Later Prepare adopts a log containing every chosen prefix | Stable leader pipelines entries; `SS` ends further decisions in the old configuration | Separate accepted, chosen, leader-observed decided, and follower-learned decided states | [[Omni-Paxos-2023]], §§4.1-4.2, 6 |
| [[GPaxos]] | A c-struct `v` is chosen at ballot `m` when an `m`-quorum has voted c-structs extending `v`; learners learn lubs of chosen c-structs | Fast path requires compatible c-struct evidence, not identical values | Preserves generalized consistency: learned c-structs remain compatible | Classic ballots progress with stable leader; fast collisions require higher ballot | Model chosen predicate with prefix relation `v <= beta_a[m]` | [[Generalized-Paxos-2005]] |
| [[EPaxos]] | Fast-commit on matching PreAccept attributes; otherwise Accept/Commit after majority evidence | Matching `(cmd, seq, deps)` for fast path or majority in slow path | Commits one tuple for an instance and orders interfering commands through dependencies | Slow path recovers when fast evidence disagrees or is incomplete | Separate commit from execution readiness; dependencies may delay execution | [[EPaxos-2013]], [[EPaxos-Revisited-2021]] |
| [[EPaxosStar]] | Fast-commit when a quorum of size `n - e` returns dependency sets equal to `initDep[id]`; recovery may commit `Nop` | Optimized bound `n >= max{2e + f - 1, 2f + 1}` | Validation preserves agreement and visibility among conflicting commands | `Waiting` messages and eventual recovery coordinator stability avoid recovery cycles | Model validation as part of the commit/recovery evidence, not as an optimization | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | In-order commit after learning an instance and all previous instances; optional out-of-order commit for commutable requests | Simple consensus restricts non-coordinators to `no-op`; out-of-order commit requires application commutativity | Preserves a single learned sequence, or equivalent orders for commutable operations | `SKIP` and revocation fill gaps that would otherwise block commit | Separate chosen, learned, in-order committed, and out-of-order executable | [[Mencius-2008]] |
| [[PigPaxos]] | Ordinary Multi-Paxos commit after a majority of unique Phase 2b accepted votes | Relays may aggregate replies, but leader counts replica votes and suppresses duplicates | Preserves the same chosen-value rule as Paxos | Relay/follower timeouts only delay majority collection or trigger retries | Model relay aggregate as a set of voter ids/evidence, not as one vote | [[PigPaxos-2021]] |
| [[Atlas]] | Fast commit after all fast-quorum replies satisfy `union_Q dep = union^f_Q dep`; slow commit after `f + 1` consensus acks | Fast quorum includes coordinator; slow path uses [[FPaxos]] Phase 2 | Commits one `(cmd, dep)` per identifier and makes conflicting commands dependency-visible | Fallback handles non-recoverable dependency unions | Separate `MCommit` from batch execution readiness | [[Atlas-2020]] |
| [[SwiftPaxos]] | Commit/execute from matching dependency-path evidence; SlowAck and Sync repair disagreement | Leader-including fast quorum or slow quorum evidence | Preserves agreed dependencies and acyclic committed dependency graph | Fallback is available when fast evidence does not match | Track dependency paths, not only direct dependency sets | [[SwiftPaxos-2024]] |
| [[Pando]] | A write is chosen when enough Phase 2 quorum evidence exists; reads can return when chosen value is reconstructible | Erasure-coded quorum intersections recover enough splits | Later proposals/read repair must preserve chosen value identity and reconstructability | Progress needs available Phase 1b/Phase 2 quorums | Distinguish value identity from individual coded splits | [[Pando-2020]] |
| [[Rabia]] | A slot is decided when Weak-MVC returns; binary `1` maps to a majority proposal, binary `0` maps to `⊥` | Non-faulty majority and common coin; weak validity allows null slots | Agreement is on the slot output, including `⊥` | Probabilistic termination with expected five rounds | Do not require ordinary validity for every slot | [[Rabia-2021]] |

| [[CURP]] | Client completes after record acceptance by all `f` witnesses plus master result, or after backup sync to `f` backups | Request-level commutativity and exactly-once RPC ids | Completed operations survive master crash and replay cannot contradict visible results | Fast path requires all witnesses; slow path waits for backup replication | Treat client completion separately from later ordered backup commit | [[CURP-2019]] |
| [[Hermes]] | Coordinator replies after every current live member acknowledges its `TS`; later `VAL` only restores matching local readability | Stable leased epoch; coordinator has applied locally; each follower ACKs even a stale/lower `INV` | No authorized reader can return an older value after completion; higher timestamps safely supersede lower concurrent writes | All-members ACKs block on failure until RM changes membership | Model client completion, coordinator state, and follower validation separately; RMW commit additionally requires highest concurrent timestamp | [[Hermes-2020]], §§3.1-3.2, 3.6 |
| [[Copilot]] | Fast commit after `f + floor((f + 1) / 2)` compatible `FastAcceptOk` replies; regular/takeover commit after `f + 1` `AcceptOk` replies | `n = 2f + 1`; fast quorum includes proposer; regular dependency is the `(f + 1)`-th sorted suggestion | Commits one command/dependency per log entry and preserves cross-log order | Slow Accept handles incompatible/missing fast evidence; takeover unblocks execution | Track commit, dependency-finality, execution readiness, and duplicate suppression separately | [[Copilot-2020]], §§3.2–3.4 |
| [[Avicenna]] | Real leader commits after `f + 1` `AcceptOK`s from `f + 2` non-standbys including itself; shadow leader analogously shadow-commits but shadow entries never execute | One real leader per phase; `n = 2f + 1`; commit is broadcast to all replicas | Commit quorum intersects later merge evidence, preserving the real-log value across phases | One slow follower is bypassed; stopped commits trigger rotation | Separate real commit, execution after the committed prefix, shadow commit, and `OverCommitted = f + 2` truncation evidence | [[Avicenna-2026]], §§4.1, 4.4 |
| [[Bodega]] | Leader commits a write after at least `m` matching `AcceptReply`s including self and replies from every responder for that key | Stable ballot/roster; `m = ceil(n / 2)`; leader is implicit responder | Same-ballot local-read responders necessarily store every committed key write | Slow/distant responders extend write latency until a higher roster removes them safely | Keep consensus commit separate from local-read eligibility and from `m` optional `AcceptNote`s used for early read release | [[Bodega-2026]], §§3.2, 4.3 |
| [[Jetpack]] | Client fast-commits after a same-view superquorum of at least `f + ceil(f/2) + 1` includes every host proposer; otherwise waits for unchanged host commit | Conflict-free against both paths; host satisfies receipt/proposal/log ordering prerequisites | Original proposers promise no concurrent conflict before the fast command; host later commits it consistently | Conflict or missing evidence leaves the original path live; recovery marker gates a new view | Separate fast client commit, eventual original commit, recovery-set acceptance, and new-view activation | [[Jetpack-2026]], §§3.2-4.3 |
| [[Hydra]] | No consensus commit; receiver delivers `p` only below the minimum `(clock, sequencer ID)` frontier and after exposing same-sequencer gaps | Strictly monotonic clocks and per-sequencer counters | Establishes ordered delivery/drop evidence only | Flushes or reconfiguration advance stalled frontiers | Do not equate Hydra delivery with durable application commit | [[Hydra-2023]], §§4.1-4.4 |
| [[HydraPaxos]] | Client commits after consistent replies from `f + 1` of `2f + 1` replicas, including leader | Deterministic execution and stable Hydra delivery; missing positions resolved first | Majority durability plus common order yields linearizable SMR | Drops add message recovery or `NO-OP` agreement | Separate receiver delivery, replica logging, leader execution, and client commit | [[Hydra-2023]], §6 |
| [[WPaxos]] | Per-object owner commits `(o,b,s)` after matching `2b` replies from every node in one `q2 ∈ Q2`, then broadcasts `3` | Stable per-object ballot; `Q2` satisfies cross-phase intersection with every `Q1` | Later owner sees or blocks the committed slot through `Q1 ∩ Q2` | Nearby `Q2` keeps stable-owner latency local; higher ballot rejects and retries | Logs may commit out of order but execute without slot gaps | [[WPaxos-2020]], §§4.2-4.4 |

## Main patterns
Fast commit rules need stronger evidence than slow commit rules because recovery must be able to distinguish a real commit from a merely possible partial execution.

## Important exceptions
[[Pando]] is not an SMR command protocol; its commit rule is about erasure-coded storage versions rather than command execution order.

[[Rabia]] is an SMR command protocol, but its commit rule is weak-validity-based: committing `⊥` is intentional slot forfeiture, not a safety failure.

## Common pitfalls
Do not treat "committed" and "ready to execute" as the same state in dependency-based SMR protocols.

## Relevance to new protocol design
Define the commit predicate before choosing quorum sizes; otherwise recovery evidence may be too weak to prove [[agreement]].

## Open questions
- TODO: Extract exact pseudocode line numbers for [[SwiftPaxos]] and [[Pando]] commit preconditions.

## Related pages
[[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[recovery-rules]], [[agreement]], [[recoverability]]


