---
type: comparison-dimension
dimension: leader roles
protocols: [FastPaxos, FPaxos, OmniPaxos, GPaxos, EPaxos, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia, CURP, Copilot, Avicenna, Bodega, Jetpack, Hydra, HydraPaxos, WPaxos]
tags: [leader]
---

# Leader Roles

| Protocol | Leader role | Fast-path leader involvement | Source |
|---|---|---|---|
| [[FastPaxos]] | Coordinator prepares fast/classic rounds | Can be bypassed by proposer-to-acceptor votes after `any` | [[FastPaxos-2006]] |
| [[FPaxos]] | Proposer completing Phase 1 becomes stable leader; leader runs repeated Phase 2 with chosen flexible quorums | Leader remains on the normal path; flexibility changes acceptor evidence, not proposer authority | [[Flexible-Paxos-2016]] |
| [[OmniPaxos]] | BLE elects the highest-ballot quorum-connected candidate without testing log freshness; Sequence Paxos then synchronizes it | Leader remains on every stable replication path; followers need only connect to that leader, not elect it in BLE | [[Omni-Paxos-2023]], §§4-5 |
| [[GPaxos]] | Leader per ballot runs phase 1, computes safe c-structs, and resolves collisions | In fast ballots, proposers send directly to acceptors after leader starts the ballot | [[Generalized-Paxos-2005]] |
| [[EPaxos]] | No fixed leader; command leader per instance | Command leader collects PreAccept replies | [[EPaxos-2013]] |
| [[Mencius]] | Deterministic rotating coordinator per instance, e.g. instance `cn + p` belongs to server `p` | Local coordinator proposes directly for its owned slots; idle coordinators send or piggyback `SKIP` | [[Mencius-2008]] |
| [[PigPaxos]] | Stable Multi-Paxos leader makes decisions; randomly selected relays only disseminate and aggregate messages | Leader remains on every commit path; relays reduce leader communication load but do not choose values | [[PigPaxos-2021]] |
| [[Atlas]] | No distinguished leader; initial coordinator per command and recovery coordinator on takeover | Initial coordinator belongs to its fast quorum and collects all fast-quorum replies | [[Atlas-2020]] |
| [[SwiftPaxos]] | Fixed leader per ballot | Leader belongs to all fast quorums in the discussed design | [[SwiftPaxos-2024]] |
| [[Pando]] | Delegate/leader used for Phase 2 or conflict fallback | Reads can avoid leader; writes may partially delegate | [[Pando-2020]] |
| [[Rabia]] | No leader or command leader | All replicas participate symmetrically in Weak-MVC | [[Rabia-2021]] |
| [[CURP]] | Strong master serializes and executes updates; witnesses do not order | Master remains on execution path, but clients bypass backup sync by recording unordered durability at witnesses | [[CURP-2019]] |
| [[Copilot]] | Two distinguished replicas each own a separate log and redundantly order/execute every command | Both pilots run independent fast paths; either can take over specific blocking entries from the other log | [[Copilot-2020]] |
| [[Avicenna]] | One real leader orders executable commands; the deterministic next leader is shadow leader; roles rotate by phase | Real leader remains on the normal commit path; shadow processing is independent and becomes authoritative only after safe rotation | [[Avicenna-2026]] |
| [[Bodega]] | One ballot leader orders writes; the roster additionally assigns arbitrary per-key read responders, with the leader implicit for all keys | Local reads bypass the leader at stable/caught-up responders; writes still traverse the leader and cover responders | [[Bodega-2026]] |
| [[Jetpack]] | Retains the host's stable-view proposer set; all proposers must receive/promise each fast command; recovery coordinator bridges proposer sets | Clients bypass leader serialization for fast evidence, but host proposers concurrently insert the command and later commit it | [[Jetpack-2026]] |
| [[Hydra]] | No ordering leader; several active sequencers independently stamp messages; configuration service controls membership | Any sequencer may handle a groupcast; receiver order merges all sequencers | [[Hydra-2023]], §§3-5 |
| [[HydraPaxos]] | Network sequencers order requests; one replica leader executes and drives unresolved-drop agreement | Leader is bypassed for ordering but must appear in the consistent reply majority | [[Hydra-2023]], §6 |
| [[WPaxos]] | Every node may own a subset of objects; each object has one ballot leader and independent log | Current owner remains on Phase 2; another node takes over one object through `Q1` [[object-stealing]] | [[WPaxos-2020]], §§3.2-3.3 |


