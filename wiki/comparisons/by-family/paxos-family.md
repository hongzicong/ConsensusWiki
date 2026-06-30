---
type: comparison-family
family: paxos
protocols: [FastPaxos, EPaxos, EPaxosStar, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando]
tags: [paxos]
---

# Paxos Family

## Family overview
These protocols preserve Paxos-style safety through quorum evidence while optimizing wide-area latency or cost.

## Protocol table
| Protocol | Main idea | What it changes | Strength | Weakness | Source |
|---|---|---|---|---|---|
| [[FastPaxos]] | Fast rounds | Lets acceptors vote directly after `any` | Two-delay learning without collision | Collision recovery complexity | [[FastPaxos-2006]] |
| [[EPaxos]] | Leaderless dependency order | Any replica leads commands | Load balance and low WAN commit latency | Complex dependency recovery | [[EPaxos-2013]] |
| [[EPaxosStar]] | Validated leaderless dependency order | Simplifies/fixes EPaxos recovery with `bal`/`abal` and validation | Optimal `f`-resilient `e`-fast bound | Recovery still has subtle validation cases | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | Rotating coordinator per log slot | Partitions sequence instances among servers and adds cheap `no-op` skipping | Balances WAN bandwidth/CPU and avoids a single leader bottleneck | Any server failure leaves owned slots to revoke or skip | [[Mencius-2008]] |
| [[PigPaxos]] | Randomized relay aggregation for Multi-Paxos | Changes leader/follower communication, not Paxos quorum logic | Reduces leader message load while reusing Paxos safety proof | Relay hops and failures can increase latency/tail latency | [[PigPaxos-2021]] |
| [[Atlas]] | Leaderless dependency order with small site-failure budget | Fast quorum `floor(n/2) + f`, slow quorum `f + 1`, recovery quorum `n - f` | Low planet-scale latency when concurrent site failures are rare | May block if more than `f` sites are unreachable | [[Atlas-2020]] |
| [[SwiftPaxos]] | Leader-including fast quorum dependencies | Keeps dependency graph acyclic | Low latency and simpler execution | Quadratic messages | [[SwiftPaxos-2024]] |
| [[Pando]] | Erasure-coded quorum split recovery | Separates Phase 1a/1b/2 quorums | Near-optimal cost/latency | Single-key focus | [[Pando-2020]] |


