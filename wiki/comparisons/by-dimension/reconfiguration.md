---
type: comparison-dimension
dimension: reconfiguration
protocols: [OmniPaxos, Hermes, Hydra, WPaxos]
tags: [reconfiguration, log-migration, configuration]
---

# Reconfiguration

## What this dimension means

Reconfiguration changes the replicas responsible for future consensus while preserving the decided history and defining when the new configuration may start.

## Why it matters

Membership change joins two safety domains. The protocol must close the old log, transfer sufficient decided state, prevent concurrent incompatible decisions, and restore liveness even when old and new members have limited connectivity.

## Comparison table

| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[Hydra]] | Configuration service removes a sequencer after per-group quorum reports or adds one after choosing a cross-group flush frontier | Numbered configurations; receivers stop old-stream processing or pause delivery during transition | Final flush closes every old sequence gap; addition frontier is later than all quorum-delivered old messages | Needs service communication with a quorum of every receiver group | This changes sequencers, not necessarily application replica membership | [[Hydra-2023]], §4.4 and Appendix B |
| [[WPaxos]] | General two-phase joint old/new quorum configuration; one-zone/row changes may collapse to one phase | Pipelined slots; old/new configurations must not decide independently | Joint quorum phase preserves one decision history; special case requires `Q'1 ∪ Q1 = Q'1` and `Q'2 ∪ Q2 = Q'2` | Two phases add latency; restricted common changes are cheaper | Separate object stealing from membership/quorum reconfiguration | [[WPaxos-2020]], §5.3 |
| [[OmniPaxos]] | Decide stop-sign `SS` in old Sequence Paxos instance; migrate decided log through cross-configuration service layer; start new instance after complete log arrives | Fixed configuration per instance; `SS` names next membership; at least one new member can gather the complete decided log | No entry may be decided after chosen `SS`; new servers start only with the complete old decided prefix | Any reachable old/new server may contribute log segments, avoiding old-leader-only migration | Separate old-config stop, data availability, migration completion, and new-config start predicates | [[Omni-Paxos-2023]], §6 |
| [[Hermes]] | RM waits for old leases to expire, installs `m-update(live_nodes, epoch_id)`, and fences messages by epoch; a joiner first shadows writes while copying chunks | Majority-based RM can update only in the primary partition; every newly listed live node must become operational before writes finish | Old/minority replicas stop serving before the new write-all set completes; shadowing prevents missed concurrent writes during state transfer | Writes stall while listed members lack the new epoch; joiner serves only after full catch-up | Separate membership agreement, epoch activation, write acknowledgment, and bulk state-transfer completion | [[Hermes-2020]], §§2.4, 3.4 |

## Main patterns

OmniPaxos separates the consensus decision to reconfigure from bulk state transfer. Hermes separates leased RM epoch change from shadow-replica state transfer. Hydra closes sequencer streams at a clock/sequence frontier. WPaxos uses joint old/new quorum configurations and permits only algebraically nested special cases to skip a phase.

## Important exceptions

The paper's formal appendix proves Sequence Paxos safety. The cross-configuration mechanism is described with correctness conditions but is not presented as one mechanized end-to-end proof.

## Common pitfalls

- Do not let a new server start merely because it learned `SS`; it must fetch the complete decided log.
- Do not migrate unchosen suffixes as authoritative state.
- Do not permit post-`SS` decisions in the old configuration.
- Do not assume the old leader is the only safe migration source; decided entries are immutable.

## Relevance to new protocol design

Separate the ordering barrier from data movement. This enables parallel migration, alternative storage placement, and version isolation without weakening the old/new configuration boundary.

## Open questions

- How should snapshots or erasure-coded state replace complete-log migration?
- What availability condition is sufficient when old and new configurations do not overlap?
- How should repeated or concurrent reconfiguration requests be serialized?

## Related pages

[[OmniPaxos]], [[Omni-Paxos-2023]], [[Hermes]], [[Hermes-2020]], [[reliable-membership]], [[Hydra]], [[Hydra-2023]], [[WPaxos]], [[WPaxos-2020]], [[sequence-consensus]], [[recovery]], [[agreement]], [[liveness]]
