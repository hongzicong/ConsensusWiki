---
type: comparison-dimension
dimension: quorum systems
protocols: [FastPaxos, FPaxos, OmniPaxos, GPaxos, EPaxos, EPaxosStar, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia, CURP, Hermes, Copilot, Avicenna, Bodega, Jetpack, Hydra, HydraPaxos, WPaxos]
tags: [quorum]
---

# Quorum Systems

## What this dimension means
Quorum systems define which evidence sets can choose, commit, recover, or read.

## Why it matters
Every fast path here buys latency by strengthening quorum intersections or metadata agreement.

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Classic and fast quorums | Any two quorums intersect; fast rounds require intersection of one arbitrary quorum with two fast quorums | Prevents two fast values surviving recovery | Fast progress needs nonfaulty fast quorum | Model `Quorum(i)` by round | [[FastPaxos-2006]] |
| [[FPaxos]] | Separate Phase 1 and Phase 2 quorum families; only cross-phase pairs must intersect | General: every `Q1` intersects every `Q2`; simple thresholds: `\|Q1\| + \|Q2\| > N` | Later Phase 1 sees evidence from every earlier deciding Phase 2 quorum | Stable leader needs `Q2`; leader replacement needs `Q1` and `Q2`; structured availability depends on placement | Never assume `Q1-Q1` or `Q2-Q2` intersection; record phase and family membership | [[Flexible-Paxos-2016]], §§3-5 |
| [[OmniPaxos]] | Fixed majorities for BLE observation, Prepare/log adoption, and Accept/decision | QC server is directly linked to at least a majority of correct servers; at least one QC server is the progress threshold | Prepare majority intersects every earlier deciding majority and carries the chosen prefix | One QC server can lead even if its followers are not mutually connected | Distinguish a majority of heartbeat replies, a QC center, and a fully connected majority | [[Omni-Paxos-2023]], §§3-5 |
| [[Hydra]] | Application-defined quorum from every receiver group for sequencer addition/removal | Group protocol determines quorum size; whole group required for uniform per-receiver drop detection | Fixes the terminal sequence-number frontier before configuration transition | Reconfiguration stalls without one quorum from every group | Hydra sequencers are not acceptors; do not invent a global `n,f` formula | [[Hydra-2023]], §4.4 and Appendix B |
| [[HydraPaxos]] | Majority replies over `n = 2f + 1`, including leader | `f + 1` consistent replies survive `f` crashes | Couples Hydra order with durable replica evidence | Live leader and majority are required; drops trigger extra recovery | Inherited NOPaxos recovery quorum is not restated | [[Hydra-2023]], §6 |
| [[WPaxos]] | Grid-like flexible quorums across zones and nodes | `Q1` uses `f_n + 1` nodes in `Z - f_z` zones; `Q2` uses `l - f_n` nodes in `f_z + 1` zones | Cross-phase overlap reveals or blocks every earlier per-object decision | Stable owner needs local `Q2`; stealing/recovery needs wide `Q1` then `Q2` | Model zone placement, not size alone; no same-phase intersection is required | [[WPaxos-2020]], §3.1 |
| [[GPaxos]] | Ballot-dependent classic and fast quorums over c-struct votes | Any two quorums intersect; for fast `k`, any two `k`-quorums and any `m`-quorum have non-empty triple intersection | Ensures lower-ballot choosable c-structs are extended by later safe values | Fast progress needs compatible quorum evidence; classic progress needs stable leader/live quorum | Model chosen as prefix evidence over c-structs, not equality of values | [[Generalized-Paxos-2005]] |
| [[EPaxos]] | Majority plus fast quorum over `N = 2F + 1` | Fast path requires matching attributes | Preserves one tuple per instance | Majority recovery | Separate command leader from quorum members | [[EPaxos-2013]] |
| [[EPaxosStar]] | Parameterized slow/recovery quorum `n - f` and fast quorum `n - e` | Optimized protocol requires `n >= max{2e + f - 1, 2f + 1}` | Recovery validation preserves agreement and visibility | `e` bounds fast-path failures; `f` bounds overall crash resilience | Model `e` and `f` independently | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | Paxos quorum plus owner-authored `SKIP` | Paper states quorum size `f + 1` with `n = 2f + 1`; `SKIP` is safe by simple-consensus value restriction | Quorum evidence preserves chosen non-`no-op`; coordinator `SKIP` proves `no-op` without majority agreement | Progress needs live quorum and revocation of suspected coordinators | Model `SKIP` as owner evidence, not as a quorum certificate | [[Mencius-2008]] |
| [[PigPaxos]] | Ordinary Paxos majority quorum carried through relay aggregates | `N = 2f + 1`; leader must count unique replica votes, not relay messages | Relay groups do not alter quorum intersection or chosen-value rules | Relay failures can force retries, but a live majority remains enough for progress under timing assumptions | Model relays as transport/aggregation nodes, not quorum members with extra voting power | [[PigPaxos-2021]] |
| [[Atlas]] | Fast quorum `floor(n/2) + f`, slow quorum `f + 1`, recovery quorum `n - f` | `1 <= f <= floor((n - 1)/2)`; fast quorum includes initial coordinator | Recovery can find slow accepted proposals and reconstruct fast dependency unions | Small `f` gives smaller quorums but more than `f` unavailable sites may block | Track the remembered initial fast quorum per command | [[Atlas-2020]] |
| [[SwiftPaxos]] | Majority slow quorums; leader-including fast quorums | Fast quorum intersection size is greater than `N/2` | Preserves dependency-path agreement | Slow quorum fallback | Model fast quorum membership per ballot | [[SwiftPaxos-2024]] |
| [[Pando]] | Phase 1a, Phase 1b, Phase 2 quorums | 1a intersects 2 in one site; 1b intersects 2 in `k` splits | Recovers chosen erasure-coded values | Needs available 1b and 2 quorums | Distinguish value id from split count | [[Pando-2020]] |
| [[Rabia]] | Each step waits for `n - f`; proposal/state majority is `⌊n/2⌋ + 1`; decision vote threshold is `f + 1` | `n ≥ 2f + 1`; no leader inclusion | Prevents two concrete proposals or binary decisions for the same slot | Non-faulty majority lets Weak-MVC keep advancing | Model `⊥` as a valid weak outcome, not a recovered value | [[Rabia-2021]] |

| [[CURP]] | All-`f` witness durability plus primary-backup sync | Primary-backup has one master plus `f` backups; fast path records to all `f` witnesses; recovery chooses one backup and one selected witness | Every fast-completed unsynced operation appears in the selected witness, while backup sync preserves ordered history | Any witness rejection/failure can force slow sync; recovery waits if no witness is reachable | Do not union multiple witnesses in primary-backup recovery; one witness preserves the commutative replay set | [[CURP-2019]] |
| [[Hermes]] | Read-one/write-all over leased live membership `M_e` | Coordinator applies locally and waits for `\|M_e\| - 1` remote ACKs; RM alone uses a majority protocol to change `M_e` | Every replica authorized to serve a local read has seen the write or a higher timestamp before client completion | One slow member blocks until retransmission or RM removes it after lease expiry | Do not replace all-members evidence with a majority; cross-epoch safety is an RM/lease obligation | [[Hermes-2020]], §§2.4, 3.1-3.4 |
| [[Copilot]] | Fast dependency quorum plus majority regular/recovery quorums | `n = 2f + 1`; fast quorum `f + floor((f + 1) / 2)` includes proposer; regular and Prepare quorums are `f + 1` | Fast/recovery intersection gives at least `floor((f + 1) / 2)` reports, but ambiguous partial fast evidence requires cross-log inspection | A majority remains sufficient after `f` failures; one slow participant is bypassed by redundant pilots/takeover | Count the proposing pilot in the fast quorum; do not model recovery as a count-only predicate | [[Copilot-2020]], §§3.2–3.4 |
| [[Avicenna]] | Restricted normal acceptor set plus periodic all-replica Armageddon phases | `n = 2f + 1`; normal non-standbys `f + 2`; commit `f + 1` including leader; normal merge `2` non-standby logs; Armageddon merge `f + 1` | Every normal commit intersects any two non-standby logs; every Armageddon commit intersects an `f + 1` merge set | One slow follower can be omitted; repeated rotation reaches an all-replica phase after multiple crashes | Distinguish acceptor-set size, commit quorum, and rotation evidence; `OverCommitted` uses all `f + 2` non-standbys | [[Avicenna-2026]], §§4.1, 4.3-4.4, Appendix A.2 |
| [[Bodega]] | Flexible majority write quorum extended to cover all current per-key responders; majority-held roster leases | Odd `n`; `m = ceil(n / 2)`; leader is implicit responder; stable responder holds `m` grants and catches up through thresholds from some `m` grantors | Majority intersection makes the roster unique and transfers older-ballot write visibility; responder coverage handles same-ballot writes | After old leases expire, a healthy-majority roster can exclude failed responders and resume normal consensus | Model write quorum, roster-lease quorum, threshold subset, and optional `m` early notifications as distinct evidence sets | [[Bodega-2026]], §§3.2-3.3, 4.3 |
| [[Jetpack]] | Same-view fast superquorum constrained to contain all host proposers; majority recovery-set quorums | `n = 2f + 1`; fast at least `f + ceil(f/2) + 1`; prepare/accept recovery `f + 1` | Every fast commit appears in at least `ceil(f/2) + 1` recovery replies; proposer inclusion enforces the original-path promise | Missing fast evidence or conflict falls back to host; view change pauses until marker commit | Track view and proposer membership, not size alone; acknowledgements across views never combine | [[Jetpack-2026]], §§3.3-4.3, Appendix B.3 |

## Main patterns
Fast paths need either larger quorums, leader inclusion, identical metadata, or a fallback quorum that can reconstruct prior choices.

## Fast quorum sizes
| Protocol | Common configuration | Fast quorum size |
|---|---|---|
| [[FastPaxos]] | `N = 3f + 1` acceptors | `2f + 1`; more generally, with fast quorum size `N - E` |
| [[GPaxos]] | `N` acceptors | either classic and fast quorums both `floor(2N/3) + 1`, or classic `floor(N/2) + 1` with fast `ceil(3N/4)` |
| [[EPaxos]] | `N = 2F + 1` replicas | `F + floor((F + 1)/2)` total, including the command leader; non-leader replies are one fewer |
| [[EPaxosStar]] | General `n, e, f` | `n - e`; optimized correctness requires `n >= max{2e + f - 1, 2f + 1}` |
| [[Mencius]] | `n = 2f + 1` servers | no Fast Paxos-style fast quorum; classic/recovery quorum is `f + 1`, while `SKIP` can be learned from the coordinator |
| [[PigPaxos]] | `N = 2f + 1` replicas | no Fast Paxos-style fast quorum; classic quorum remains `floor(N/2) + 1`; relay partial thresholds must satisfy `sum_{i=1}^R g_i >= floor(N/2) + 1` |
| [[Atlas]] | General `n, f`, with `1 <= f <= floor((n - 1)/2)` | `floor(n/2) + f`; slow quorum `f + 1`; recovery quorum `n - f` |
| [[SwiftPaxos]] C1 | `N = 2f + 1` replicas | any leader-including set with size `> 3N/4`, i.e. at least `floor(3N/4) + 1` |
| [[SwiftPaxos]] C2 | `N = 2f + 1` replicas | a unique fixed majority fast quorum of size `f + 1`, including the leader |
| [[Pando]] | Erasure-coded storage with split threshold `k` | no SMR fast quorum; Phase 1a fast-read/discovery quorum has size `max(k, f + 1)`, while Phase 1b/Phase 2 quorums are at least `f + k` |
| [[Rabia]] | `n ≥ 2f + 1` replicas | no fast quorum in the Paxos sense; each Weak-MVC communication step waits for `n - f`, while fast termination needs aligned majority-derived binary state |

| [[CURP]] | Primary-backup with one master, `f` backups, and `f` witnesses | fast witness set is all `f` witnesses; backup sync writes to `f` backups; recovery uses one backup and one selected witness |
| [[Hermes]] | Current leased membership `M_e`; typical deployment 3-7 replicas | no subset fast quorum: normal write/replay requires all `\|M_e\|` members, implemented as coordinator local application plus `\|M_e\| - 1` remote ACKs |
| [[Copilot]] | `n = 2f + 1` replicas | fast `f + floor((f + 1) / 2)` including proposer; regular Accept and takeover Prepare/Accept each use `f + 1` |
| [[Avicenna]] | `n = 2f + 1` replicas | normal acceptor set `f + 2`; real/shadow commit `f + 1`; normal rotation `2` non-standby logs; Armageddon rotation `f + 1` logs |
| [[Bodega]] | odd `n`; `m = ceil(n / 2)` and `f = floor(n / 2)` | Prepare `m`; write `m` plus all key responders; stable roster `m` lease grants; catch-up thresholds from a size-`m` grantor subset |
| [[Jetpack]] | `n = 2f + 1` replicas | fast `f + ceil(f/2) + 1` and all proposers; original/recovery `f + 1`; recovery candidate threshold `ceil(f/2) + 1` |
| [[FPaxos]] | arbitrary acceptor set `A` | general cross-phase quorum families; simple thresholds satisfy `\|Q1\| + \|Q2\| > N`; minimum `\|Q1\| = N - \|Q2\| + 1` |
| [[OmniPaxos]] | fixed configuration `N` | Prepare, Accept, and BLE observation use a majority; no separate symbolic `f` or fast quorum; progress requires at least `1` QC server |
| [[Hydra]] | receiver-group size is application-defined | one quorum from every group for sequencer reconfiguration; whole group for uniform drop detection; no sequencing commit quorum |
| [[HydraPaxos]] | `n = 2f + 1` replicas | normal commit `f + 1` consistent replies including leader; exact inherited drop-recovery quorum is Unclear |
| [[WPaxos]] | `Z` zones, `l` nodes/zone; total `Zl` | `\|Q1\| = (f_n + 1)(Z - f_z)`; `\|Q2\| = (l - f_n)(f_z + 1)`; every `Q1` intersects every `Q2` |

Counting convention: these rows count total quorum membership unless explicitly saying "non-leader replies." This matters for [[EPaxos]], where the paper states the fast-path quorum size as including the command leader.

[[Copilot]] uses the same stated fast-quorum size as [[EPaxos]] under `n = 2f + 1`, but its recovery obligation is different. A recovery majority intersects a committing fast quorum in at least `floor((f + 1) / 2)` replicas. Counts in `[floor((f + 1) / 2), f)` are inconclusive, so recovery examines the first possibly incompatible entry in the other pilot's log.

[[Avicenna]] obtains fast rotation by shrinking the normal acceptor universe rather than enlarging its commit quorum. For a commit set `A` and merge set `B`, `(f + 1) + 2 > f + 2` guarantees intersection in a normal phase. In an Armageddon phase, `(f + 1) + (f + 1) > 2f + 1` restores ordinary-majority-style recovery.

[[Bodega]] uses majority intersection twice. First, two sets of `m` lease grantors cannot support different stable rosters. Second, a new responder's chosen size-`m` grantor subset intersects every older write majority, so some reported `thresh_p` covers that write's slot. Same-ballot visibility comes from the separate requirement that the write quorum include every responder.

[[Jetpack]] derives its candidate threshold directly:

```text
|Q_fast intersect Q_recovery|
  >= (f + ceil(f/2) + 1) + (f + 1) - (2f + 1)
   = ceil(f/2) + 1
```

This intersection is meaningful only within the same fast-path view. Principle 1 prevents stale/new acknowledgements from jointly satisfying a commit whose evidence no single view can recover.

[[FPaxos]] is the basic reminder that quorum intersection is a proof obligation, not a synonym for majority. For simple thresholds, shrinking common Phase 2 directly enlarges Phase 1. Grid or weighted families can change this size tradeoff, but every allowed cross-phase membership pair must still intersect.

In [[EPaxosStar]], `f` and `e` are separate budgets. `f` is the total crash-failure resilience target, while `e <= f` is the number of failures under which conflict-free commands should still execute on the fast path. This is why the fast quorum is written as `n - e`: after `e` failures, exactly `n - e` processes remain available, so a fast quorum of size `n - e` is the largest quorum that can still be collected in the `e`-faulty fast run. Slow and recovery quorums use `n - f` because they must remain available under the full `f`-failure resilience target.

In [[Atlas]], `f` is the tolerated number of concurrent site failures, chosen independently of `n` within `1 <= f <= floor((n - 1)/2)`. The fast quorum size `floor(n/2) + f` is large enough that, after removing up to `f` fast-quorum members including the initial coordinator, recovery can still reconstruct a fast-path dependency union from at least `floor(n/2)` non-coordinator fast-quorum replies.

The boundary choice `e = f` is allowed, but it changes the optimized EPaxos* bound to `n >= max{3f - 1, 2f + 1}`. For `f >= 2`, the minimal configuration is `n = 3f - 1`, with fast quorum size `n - f = 2f - 1`. So `e = f` minimizes the fast quorum for fixed `n`, but the lower bound may force `n` upward.

For [[FastPaxos]] and [[GPaxos]] with `N = 2f + 1` acceptors and classic quorum size `f + 1`, the fast quorum can be defined, but it must have size at least `N - floor(f/2)`, equivalently `floor((3f + 1)/2) + 1 = f + floor((f + 1)/2) + 1`. This is one larger than the [[EPaxos]] fast quorum under the same `N = 2f + 1` counting convention, and it has the same size as [[SwiftPaxos]] C1 under `N = 2f + 1`. The SwiftPaxos C1 fast quorum must include the ballot leader; Fast Paxos and GPaxos fast quorums do not have this leader-inclusion requirement. The protocols use the quorum evidence for different safety arguments. This supports fast learning only while at most `floor(f/2)` acceptors are unavailable; after `f` failures, only the classic quorum size remains available.

## Important exceptions
PANDO's Phase 1a quorum is intentionally too small for full recovery; it is a fast read/discovery quorum.

## Common pitfalls
Do not treat EPaxos dependencies, SwiftPaxos dependency paths, and Fast Paxos collisions as the same mechanism.

## Relevance to new protocol design
Decide first what evidence recovery must reconstruct, then derive intersections.

## Open questions
- TODO: Add exact quorum-size derivations for Fast Paxos `N, F, E`.

## Related pages
[[quorum]], [[quorum-intersection]]



