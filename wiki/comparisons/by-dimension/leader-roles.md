---
type: comparison-dimension
dimension: leader roles
protocols: [FastPaxos, GPaxos, EPaxos, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia, CURP]
tags: [leader]
---

# Leader Roles

| Protocol | Leader role | Fast-path leader involvement | Source |
|---|---|---|---|
| [[FastPaxos]] | Coordinator prepares fast/classic rounds | Can be bypassed by proposer-to-acceptor votes after `any` | [[FastPaxos-2006]] |
| [[GPaxos]] | Leader per ballot runs phase 1, computes safe c-structs, and resolves collisions | In fast ballots, proposers send directly to acceptors after leader starts the ballot | [[Generalized-Paxos-2005]] |
| [[EPaxos]] | No fixed leader; command leader per instance | Command leader collects PreAccept replies | [[EPaxos-2013]] |
| [[Mencius]] | Deterministic rotating coordinator per instance, e.g. instance `cn + p` belongs to server `p` | Local coordinator proposes directly for its owned slots; idle coordinators send or piggyback `SKIP` | [[Mencius-2008]] |
| [[PigPaxos]] | Stable Multi-Paxos leader makes decisions; randomly selected relays only disseminate and aggregate messages | Leader remains on every commit path; relays reduce leader communication load but do not choose values | [[PigPaxos-2021]] |
| [[Atlas]] | No distinguished leader; initial coordinator per command and recovery coordinator on takeover | Initial coordinator belongs to its fast quorum and collects all fast-quorum replies | [[Atlas-2020]] |
| [[SwiftPaxos]] | Fixed leader per ballot | Leader belongs to all fast quorums in the discussed design | [[SwiftPaxos-2024]] |
| [[Pando]] | Delegate/leader used for Phase 2 or conflict fallback | Reads can avoid leader; writes may partially delegate | [[Pando-2020]] |
| [[Rabia]] | No leader or command leader | All replicas participate symmetrically in Weak-MVC | [[Rabia-2021]] |
| [[CURP]] | Strong master serializes and executes updates; witnesses do not order | Master remains on execution path, but clients bypass backup sync by recording unordered durability at witnesses | [[CURP-2019]] |


