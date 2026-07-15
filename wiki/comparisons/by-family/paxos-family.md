---
type: comparison-family
family: paxos
protocols: [FastPaxos, FPaxos, OmniPaxos, GPaxos, EPaxos, EPaxosStar, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Copilot, Avicenna, Bodega, Jetpack, HydraPaxos, WPaxos]
tags: [paxos]
---

# Paxos Family

## Family overview
These protocols preserve Paxos-style safety through quorum evidence while optimizing wide-area latency or cost.

## Protocol table
| Protocol | Main idea | What it changes | Strength | Weakness | Source |
|---|---|---|---|---|---|
| [[FastPaxos]] | Fast rounds | Lets acceptors vote directly after `any` | Two-delay learning without collision | Collision recovery complexity | [[FastPaxos-2006]] |
| [[GPaxos]] | Generalized command-history consensus | Replaces single values/total sequences with compatible c-structs | Two-delay learning for non-interfering concurrent commands | Abstract c-struct recovery and checkpointing complexity | [[Generalized-Paxos-2005]] |
| [[EPaxos]] | Leaderless dependency order | Any replica leads commands | Load balance and low WAN commit latency | Complex dependency recovery | [[EPaxos-2013]] |
| [[EPaxosStar]] | Validated leaderless dependency order | Simplifies/fixes EPaxos recovery with `bal`/`abal` and validation | Optimal `f`-resilient `e`-fast bound | Recovery still has subtle validation cases | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | Rotating coordinator per log slot | Partitions sequence instances among servers and adds cheap `no-op` skipping | Balances WAN bandwidth/CPU and avoids a single leader bottleneck | Any server failure leaves owned slots to revoke or skip | [[Mencius-2008]] |
| [[PigPaxos]] | Randomized relay aggregation for Multi-Paxos | Changes leader/follower communication, not Paxos quorum logic | Reduces leader message load while reusing Paxos safety proof | Relay hops and failures can increase latency/tail latency | [[PigPaxos-2021]] |
| [[Atlas]] | Leaderless dependency order with small site-failure budget | Fast quorum `floor(n/2) + f`, slow quorum `f + 1`, recovery quorum `n - f` | Low planet-scale latency when concurrent site failures are rare | May block if more than `f` sites are unreachable | [[Atlas-2020]] |
| [[SwiftPaxos]] | Leader-including fast quorum dependencies | Keeps dependency graph acyclic | Low latency and simpler execution | Quadratic messages | [[SwiftPaxos-2024]] |
| [[Pando]] | Erasure-coded quorum split recovery | Separates Phase 1a/1b/2 quorums | Near-optimal cost/latency | Single-key focus | [[Pando-2020]] |
| [[Copilot]] | Redundant pilot/copilot logs with dependency merge and per-entry takeover | Applies Paxos independently to two logs and adds EPaxos-inspired cross-log prefix dependencies | Normal client latency despite one slow replica | Duplicated work and subtle ambiguous fast-evidence recovery | [[Copilot-2020]] |
| [[Avicenna]] | Single executable log plus independent shadow processing | Restricts normal acceptors so the next leader can recover from two non-standby logs | Multi-Paxos normal latency plus one-fail-slow tolerance | Shadow-processing throughput cost; full crash liveness may await Armageddon phase | [[Avicenna-2026]] |
| [[Bodega]] | Majority-leased roster of leader and per-key read responders | Adds responder-covering writes and threshold-certified local reads to classic consensus | Local linearizable reads survive interfering writes | Distant/slow responders extend write commit; leases require bounded drift | [[Bodega-2026]] |
| [[Jetpack]] | Parallel fast layer plus unchanged original consensus path | Adds proposer promises, same-view superquorums, and a prioritized recovery-set marker | Retrofittable 1-RTT fast commit | Host prerequisites, contention, and multi-leader processing overhead | [[Jetpack-2026]] |
| [[FPaxos]] | Cross-phase-only quorum intersection | Separates Phase 1 and Phase 2 quorum families while retaining Paxos value selection | Smaller/topology-aware steady-state quorums | Larger Phase 1 may reduce leader-recovery availability | [[Flexible-Paxos-2016]] |
| [[OmniPaxos]] | Quorum-connected election plus prefix log synchronization | Separates election, growing-log replication, and reconfiguration | Stable progress with one QC server under modeled partial partitions | Partial synchrony, heartbeat convergence, and complete migration still required | [[Omni-Paxos-2023]] |
| [[HydraPaxos]] | Network pre-orders operations through multi-sequencer [[Hydra]] groupcast | Replaces application request ordering with ordered-unreliable network delivery | One client RTT without one network serialization point | Depends on Hydra clocks/configuration and extra agreement for drops | [[Hydra-2023]] |
| [[WPaxos]] | Per-object leaders over WAN flexible quorums | Uses wide Phase 1 to move ownership and local Phase 2 for repeated commits | Adapts placement to access locality with per-object linearizability | Object migration adds WAN latency and can thrash | [[WPaxos-2020]] |


