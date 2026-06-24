---
type: comparison-dimension
dimension: leader roles
protocols: [FastPaxos, EPaxos, SwiftPaxos, Pando]
tags: [leader]
---

# Leader Roles

| Protocol | Leader role | Fast-path leader involvement | Source |
|---|---|---|---|
| [[FastPaxos]] | Coordinator prepares fast/classic rounds | Can be bypassed by proposer-to-acceptor votes after `any` | [[FastPaxos-2006]] |
| [[EPaxos]] | No fixed leader; command leader per instance | Command leader collects PreAccept replies | [[EPaxos-2013]] |
| [[SwiftPaxos]] | Fixed leader per ballot | Leader belongs to all fast quorums in the discussed design | [[SwiftPaxos-2024]] |
| [[Pando]] | Delegate/leader used for Phase 2 or conflict fallback | Reads can avoid leader; writes may partially delegate | [[Pando-2020]] |
