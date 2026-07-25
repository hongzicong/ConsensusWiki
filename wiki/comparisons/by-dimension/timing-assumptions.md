---
type: comparison-dimension
dimension: timing assumptions
protocols: [FastPaxos, FPaxos, OmniPaxos, GPaxos, EPaxos, EPaxosStar, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia, CURP, Hermes, Copilot, Avicenna, Bodega, Jetpack, Hydra, HydraPaxos, WPaxos]
tags: [timing, liveness]
---

# Timing Assumptions

## What this dimension means
Timing assumptions separate safety, which is usually asynchronous, from progress claims that need synchrony, partial synchrony, stable leadership, or available quorums.

## Why it matters
A model that bakes timing into safety can prove the wrong theorem, while a model with no timing assumptions may be unable to state the intended liveness result.

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Fast and classic rounds | Asynchronous safety; progress needs favorable conditions/coordinator recovery | Safety is quorum-based, not timing-based | Collisions and failures require eventual recovery progress | Keep safety invariant independent of timers | [[FastPaxos-2006]] |
| [[FPaxos]] | Stable-leader Phase 2 over a selected quorum; Phase 1 on election/recovery | Asynchronous safety; additional synchrony/stable proposer assumptions for progress | Safety depends only on persistent promises and cross-phase intersection | Smaller targeted `Q2` can reduce latency, but slow/failing selected members cause retransmission; leader loss requires available `Q1` | Do not convert performance gains into fewer protocol phases or synchronous safety | [[Flexible-Paxos-2016]], §§2-6 |
| [[OmniPaxos]] | Periodic BLE heartbeats elect a QC ballot; Sequence Paxos then prepares and pipelines | Partial synchrony; eventually a heartbeat duration `T_delay` has no late connected replies | Sequence safety is ballot/quorum based; suffix optimization additionally assumes FIFO sessions | Stable periods let QC status converge and one QC server complete election/Prepare | Ignore late heartbeats for the old round; do not infer failure from missing links alone | [[Omni-Paxos-2023]], §§3, 5.2 |
| [[GPaxos]] | Fast and classic ballots over c-structs | Asynchronous safety; classic progress with eventual unique leader/live quorum; fast progress when compatible evidence is collected or collisions are observed | Safety depends on safe c-struct votes and quorum intersections, not timing | Interfering commands and failures require higher-ballot recovery | Keep collision detection/liveness separate from generalized consistency proof | [[Generalized-Paxos-2005]] |
| [[EPaxos]] | Command leaders with PreAccept/Accept recovery | Asynchronous safety; progress with available quorums and recovery | Timing only affects which dependencies are observed | Conflict timing affects fast-path rate and execution latency | Model latency metrics separately from commit safety | [[EPaxos-2013]], [[EPaxos-Revisited-2021]] |
| [[EPaxosStar]] | Synchronous fast path with validated recovery | Partial synchrony; `e`-faulty synchronous fast guarantee | Agreement/visibility must hold regardless of timing | Fast execution by `t + 2 Delta` is conditional on the fast-run assumptions | State `Delta` only in liveness/performance lemmas | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | Coordinated Paxos over FIFO channels with unreliable failure detector | Asynchronous safety; FIFO/eventual delivery and eventually accurate failure detector for liveness | Failure-detector mistakes do not break safety; FIFO is used for skip piggybacking optimizations | Progress needs idle coordinators to skip and suspected faulty coordinators to be revoked | Keep FIFO assumptions local to optimizations, not the base chosen-value invariant | [[Mencius-2008]] |
| [[PigPaxos]] | Relay and leader timeouts with random relay retries | Paxos safety independent of timing; liveness assumes eventual bounded message delays | Timeouts do not justify safety decisions | Relay timeout `T_r` must be below leader timeout; random retries find healthy relays under adequate synchrony | Keep timeout/retry state out of chosen-value proof | [[PigPaxos-2021]] |
| [[Atlas]] | Leaderless command coordination with recovery | Asynchronous safety; progress with available fast/slow/recovery quorums | Dependency and recovery invariants are quorum-based | If more than `f` sites are unreachable, Atlas may block until enough sites return | Keep the site-outage budget out of safety assumptions | [[Atlas-2020]] |
| [[SwiftPaxos]] | Stable ballot leader with fast/slow ack paths | Asynchronous safety; eventual stable ballot for progress | Dependency safety is independent of message delays | SlowAck/Sync help progress after disagreement | Model eventual leader stability as an assumption, not a safety premise | [[SwiftPaxos-2024]] |
| [[Pando]] | Quorum reads/writes over geo-storage sites | Quorum safety; progress with available quorums | Intersection, not timing, preserves chosen values | Latency benefits depend on nearby available quorums and write-back | Separate quorum availability from network-delay optimization | [[Pando-2020]] |
| [[Rabia]] | Randomized Weak-MVC over datacenter replicas | Asynchronous safety; probabilistic termination; stable network only for performance | Safety does not require synchrony | Expected five rounds; fast-path rate depends on aligned PQ heads | Do not encode stable network as a safety assumption | [[Rabia-2021]] |

| [[CURP]] | Witness recording overlaps master execution | Asynchronous safety; network may delay/drop messages; 1 RTT is a successful normal-case path | Commutativity and recovery evidence, not timing, preserve linearizability | Dropped/delayed witness or master messages cause retry or sync fallback | Keep latency theorem separate from recovery safety proof | [[CURP-2019]] |
| [[Hermes]] | Logical timestamps order updates; `mlt` triggers retry/replay; LSCs support RM leases | Partial synchrony with loosely synchronized clocks for the main RM design; no-LSC read-validation variant is sketched | Per-key order and stale-message filtering do not use physical time; cross-epoch serving safety relies on the RM lease contract | Failure removal waits for lease expiry; no-LSC reads wait for a later committed write or majority membership check | Keep logical timestamps, retry timeout, and membership lease time as three distinct mechanisms | [[Hermes-2020]], §§2.4, 3.4, 8 |
| [[Copilot]] | Dual ordering paths, ping-pong timeout, and per-entry takeover timeout | Asynchronous safety; eventual partial synchrony and eventual delivery for liveness | Timeouts only trigger recovery; ballots and evidence determine safe values | Timely majority plus eventual stable takeover/view-change behavior; one slow path should not raise latency materially | Separate the qualitative slowdown metric `T ≈ T'` from eventual liveness | [[Copilot-2020]], §§2.2, 3.1, 3.4, 4.2 |
| [[Avicenna]] | Real/shadow latency windows, early-report timer, and commit-progress timer trigger rotation | Asynchronous safety; no clock synchronization; liveness assumes a majority communicates within timeouts and messages arrive before receiver timeouts | Timing only chooses when to rotate; phase-tagged merge evidence chooses safe log values | Persistent clients plus repeated deterministic rotations eventually reach a live leader/majority, including an Armageddon phase if needed | Local latency durations avoid synchronized clocks; objective parameters are performance policy, not proof premises | [[Avicenna-2026]], §§2, 4.2-4.4, Appendix A.3 |
| [[Bodega]] | Guard/renew/revoke roster leases piggyback on heartbeat failure detection | Asynchronous consensus plus bounded clock-speed drift; no synchronized timestamps or bounded skew required | Grantor expiration never precedes grantee expiration, preventing overlapping stable rosters during change | Explicit revoke is fast; unresponsive peers require expiration; partial partitions may need pre-vote/rerouting | Preserve `avg. RTT < t_hb_send ≪ t_hb_fail < t_guard = t_lease`; do not turn lease time into a consensus safety bound | [[Bodega-2026]], §§2.2, 3.3, 7 |
| [[Jetpack]] | Parallel fast/host paths during stable views; command processing pauses during recovery | Asynchronous safety; 1 RTT is a successful fast-path latency; liveness inherits eventual host view/majority progress | Safety uses view equality, quorums, ordering prerequisites, and marker commit rather than timeout bounds | Failed fast attempts use host path; new view waits for recovery-set/no-op original commit | Separate latency claim from host liveness; naive recovery uses three exchanges and optimized metadata recovery two RPCs | [[Jetpack-2026]], §§2-4, Appendix B.2 |
| [[Hydra]] | Sequencer physical clocks order cross-sequencer traffic; timeout detects failure | Safety requires clocks never move backward, not bounded skew; loose synchronization affects delivery latency | `(clock, sequencer ID)` order is safe under skew because receivers wait for every frontier | Clock skew/idle sequencers delay `c_min`; flushes and removal restore progress | Do not infer synchronized clocks or a skew safety bound | [[Hydra-2023]], §§4.3-4.5 |
| [[HydraPaxos]] | One-RTT stable no-drop path above Hydra; recovery after drops/failure | Hydra clock premise plus Paxos-style live-majority progress | Timing does not choose missing-message fate | Slow frontier or drop adds flush/recovery latency | Separate Hydra delivery latency from client commit latency | [[Hydra-2023]], §6 |
| [[WPaxos]] | Rare WAN Phase 1 moves ownership; repeated Phase 2 uses leader-local/nearby zones | Asynchronous Paxos safety; progress needs live `Q1`,`Q2`, message delivery, and resolved contention | Geography changes latency, not safe-value selection | Skewed locality benefits; oscillating locality adds repeated WAN steals | Do not convert local Phase 2 into a synchrony assumption or Fast Paxos path | [[WPaxos-2020]], §§3-6 |

## Main patterns
Safety claims are mostly quorum/intersection claims; timing assumptions enter liveness, fast-path availability, and performance.

## Important exceptions
[[EPaxosStar]] explicitly states an `e`-faulty synchronous fast-path guarantee, so its timing assumptions are part of the fast-path performance theorem.

## Common pitfalls
Do not use timeout behavior as evidence that a value is safe to recover unless the paper explicitly proves that connection.

## Relevance to new protocol design
State safety invariants without timing first, then add liveness assumptions as separate hypotheses.

## Open questions
- TODO: Extract exact timing theorem statements for each paper that gives formal liveness or latency bounds.

## Related pages
[[liveness]], [[fast-paths]], [[leader-roles]], [[protocol-catalog]]


