---
type: comparison-dimension
dimension: fault models
protocols: [FastPaxos, FPaxos, OmniPaxos, GPaxos, EPaxos, EPaxosStar, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia, CURP, Hermes, Copilot, Avicenna, Bodega, Jetpack, Hydra, HydraPaxos, WPaxos]
tags: [failure-model]
---

# Fault Models

## What this dimension means
The fault model states which failures the protocol tolerates and which components may fail without violating safety.

## Why it matters
Fault assumptions determine quorum sizes, recovery evidence, and which liveness claims can be modeled.

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Acceptors may fail non-Byzantinely | Non-Byzantine faults; safety independent of timing | Quorum intersection protects chosen values despite failed acceptors | Progress requires enough live acceptors and eventual recovery coordination | Parameterize acceptor count and failure budget | [[FastPaxos-2006]] |
| [[FPaxos]] | Proposers/acceptors may fail and messages may be lost | Non-Byzantine asynchronous model; tolerance is phase/quorum-system-specific | Cross-phase intersection preserves agreement despite failures | Current leader needs `Q2`; recovery needs `Q1`; simple quorum full recovery guaranteed through `\|Q2\| - 1` failures when `Q2` is smaller | For grids/structured families, failure placement matters more than count | [[Flexible-Paxos-2016]], §§2-5 |
| [[OmniPaxos]] | Servers fail and recover; individual bidirectional links may partition while endpoints remain alive | Non-Byzantine fail-recovery with persistent storage; messages may drop/delay; session-based FIFO perfect links | Ballots, majority intersection, and prefix synchronization preserve safety | Partial synchrony plus at least one QC server enables progress; paper gives no explicit `N,f` formula | Model process state and link connectivity independently; alive does not imply quorum-connected | [[Omni-Paxos-2023]], §§2-5 |
| [[GPaxos]] | Proposers, acceptors, learners, and leaders may stop; messages may be lost | Non-Byzantine crash/stop failures; messages are not corrupted | Quorum intersection and safe c-struct selection preserve compatibility despite failed acceptors | Progress needs a nonfaulty leader/live quorum, and fast recovery needs collision evidence to reach a leader | Model failures as omission/stop behavior, not Byzantine equivocation | [[Generalized-Paxos-2005]] |
| [[EPaxos]] | Replicas are both acceptors and command leaders | Non-Byzantine replica failures | Majority/fast quorum evidence preserves one safe tuple per instance | Any surviving replica can lead new commands or recovery if quorums are available | Model failed command leaders separately from failed quorum members | [[EPaxos-2013]] |
| [[EPaxosStar]] | Separates full crash resilience `f` from fast-path failure budget `e` | Non-Byzantine crash failures | Bound `n >= max{2e + f - 1, 2f + 1}` supports validated recovery | Fast path is guaranteed under at most `e` failures in the synchronous fast run; slow/recovery handles up to `f` | Keep `e` and `f` as distinct parameters | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | Every server owns infinitely many future coordinator slots | Non-Byzantine crash-recovery with stable storage; `n = 2f + 1` tolerates `f` failures | Paxos quorum and revocation preserve chosen values despite failed coordinators | Any failed server creates slots that must be skipped or revoked; long-term recovery needs checkpoint/state transfer | Model crashed owner separately from unavailable quorum members | [[Mencius-2008]] |
| [[PigPaxos]] | Followers and relays may silently crash | Non-Byzantine crash failures; `N = 2f + 1` tolerates `f` failures | Safety unchanged because failed relays only affect message delivery | Relay failures can hide an entire group's replies until retry, affecting latency/tail behavior | Model relay failure as loss/delay of a batch of follower replies | [[PigPaxos-2021]] |
| [[Atlas]] | Site/data-center processes may crash or be temporarily unreachable | Non-Byzantine crashes; `1 <= f <= floor((n - 1)/2)` | More than `f` failures may block but do not violate safety | Small `f` is the intended planet-scale deployment assumption | Model `f` as an availability budget, not as majority-of-`n` by default | [[Atlas-2020]] |
| [[SwiftPaxos]] | Replicas fail non-Byzantinely | Non-Byzantine faults | Leader-including fast quorums and slow quorums preserve dependency evidence | Eventual stable ballot/leader needed for progress | Record leader failure separately from dependency evidence loss | [[SwiftPaxos-2024]] |
| [[Pando]] | Storage/data sites may fail | Non-Byzantine data-site failures | Intersecting quorums preserve enough coded splits of chosen values | Reads/writes need available Phase 1b/Phase 2 quorums | Model failed sites as missing splits, not corrupted values | [[Pando-2020]] |
| [[Rabia]] | Replicas may crash fail-stop | At most `f` crashes with `n ≥ 2f + 1`; Byzantine faults excluded | Binary agreement and weak validity preserve slot outputs despite crashes | Weak-MVC terminates with probability 1 when a majority is non-faulty | Keep the common-coin adversary assumption explicit | [[Rabia-2021]] |

| [[CURP]] | Masters, backups, and witnesses may fail stop | Primary-backup deployment has one master plus `f` backups and `f` witnesses; Byzantine faults excluded | Fast-completed operations survive because they are recorded at all witnesses or synced to backups | Immediate recovery tolerates up to `f` failures, but fast path needs all witnesses to accept | Model witness failure as fast-path unavailability, not as safety loss | [[CURP-2019]] |
| [[Hermes]] | Replicas crash-stop; messages may reorder, duplicate, or drop; links may partition | Partially synchronous, non-Byzantine; leased RM permits only the primary partition to serve | Timestamps/replay handle message faults; lease expiry and epochs fence removed/minority replicas | All-live-member ACKs block until retransmission or RM removes a failed member | Preserve the paper's distinct `n - 1` node-failure claim and its stated "less than `floor(n/2)`" partition/RM limit; do not invent `n = 2f + 1` | [[Hermes-2020]], §§2.4, 3.4 |
| [[Copilot]] | Any replica, including either pilot, may crash or respond slowly | Crash failures; `n = 2f + 1`; Byzantine behavior excluded; performance target covers one slow replica | Ballots, quorum evidence, and cross-log recovery preserve linearizability | Majority communication gives crash liveness; proactive dual paths and takeover give `1`-slowdown-tolerance | Model "slow but responsive" separately from crash and from message-link delay | [[Copilot-2020]], §§2.2, 3.1, 3.6 |
| [[Avicenna]] | Any replica may crash or remain correct but become fail-slow; gray fail-slow faults may affect clients but not heartbeats | `n = 2f + 1`; up to `f` crashes for progress; one fail-slow replica for the latency goal; Byzantine faults excluded | Phase/log invariants preserve linearizability regardless of detection accuracy | Slow follower is bypassed; slow real leader is detected by client-visible counterfactual latency and rotated | Keep crash liveness, `ε-1` fail-slow performance, and gray-failure observability as separate claims | [[Avicenna-2026]], §§2-4, Appendix A |
| [[Bodega]] | Replicas may fail-slow/stop and links may fail in an asynchronous network | Odd `n`; any minority failures for availability; Byzantine faults outside scope; leases additionally require bounded drift | Consensus plus roster uniqueness/catch-up preserve linearizable reads | Expiration permits failed leaders/responders to be removed into a healthy-majority roster | Partial partitions may threaten activation liveness under heartbeat-only failure detection; model pre-vote/rerouting separately | [[Bodega-2026]], §§2.1-2.2, 4.3, 7 |
| [[Jetpack]] | Replicas may crash, slow, or be partitioned; host protocol detects view changes | `n = 2f + 1`; tolerate `f` crashes; Byzantine and interactive transaction models excluded | View-tagged evidence and prioritized recovery preserve fast replies across failures | Host path and view-change assumptions determine progress; fast failure falls back | Host performance properties need not extend to fast path, e.g. Copilot slowdown tolerance | [[Jetpack-2026]], §§1-4 |
| [[Hydra]] | Packet loss/reordering/duplication plus sequencer or link failure; non-Byzantine configuration service | Receiver-group fault tolerance is application-defined; service assumed fault tolerant | Ordered drop detection and configuration transition preserve safety despite partial delivery | Needs service access to a quorum of every receiver group | Separate unreliable groupcast from receiver-replica fault tolerance | [[Hydra-2023]], §§4.1, 4.4 and Appendix B |
| [[HydraPaxos]] | Replica crash failures plus Hydra network/sequencer failures | `n = 2f + 1`, tolerate `f` replica crashes | Majority replies and resolved gaps preserve deterministic SMR | Live majority/leader and Hydra progress required | Byzantine behavior excluded; inherited NOPaxos details are partly summarized | [[Hydra-2023]], §6 |
| [[WPaxos]] | Node crash/recovery and arbitrary message loss in a zone-structured deployment | Configure `f_z` zone failures and `f_n` node failures per zone; worst/best total tolerance depends on placement | Ballots and cross-phase intersection preserve per-object safety | Existing owners may remain partially available with `Q2` even when no `Q1` survives | Record zone/node coordinates and failure placement; Byzantine behavior excluded | [[WPaxos-2020]], §§1, 3.1, 5.3 |

## Main patterns
All currently ingested protocols assume non-Byzantine failures.

## Important exceptions
No BFT consensus paper has been ingested yet; Byzantine assumptions should remain out of protocol pages until sourced.

## Common pitfalls
Do not infer Byzantine tolerance from erasure coding or from quorum intersection alone.

## Relevance to new protocol design
Choose the fault model before choosing quorum formulas; changing the fault model changes both safety and recovery obligations.

## Open questions
- TODO: Ingest at least one BFT protocol before populating the [[bft-consensus]] family page.

## Related pages
[[failure-model]], [[quorum-systems]], [[recovery-rules]], [[protocol-catalog]]


