---
type: comparison-dimension
dimension: conflict handling
protocols: [FastPaxos, FPaxos, OmniPaxos, GPaxos, EPaxos, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia, CURP, Hermes, Copilot, Avicenna, Bodega, Jetpack, Hydra, HydraPaxos, WPaxos]
tags: [conflict]
---

# Conflict Handling

| Protocol | Conflict meaning | Resolution |
|---|---|---|
| [[FastPaxos]] | Concurrent proposals accepted in a fast round | Collision recovery chooses a safe value |
| [[FPaxos]] | Different proposers race with distinct proposal numbers, possibly using disjoint Phase 2 quorums | Cross-phase acceptor either blocks the lower ballot after a higher promise or reports the earlier accepted value so the higher proposer adopts it |
| [[OmniPaxos]] | Client commands are serialized by one Sequence Paxos leader; competing leaders use different ballots and may leave divergent unchosen suffixes | Higher-ballot Prepare adopts the highest accepted log from a majority, preserves chosen prefixes, and overwrites only unchosen suffixes during `AcceptSync` |
| [[GPaxos]] | Concurrent commands are problematic only when their c-structs are incompatible, e.g. interfering histories `w * C * D` and `w * D * C` | Compatible commands can be learned without total ordering; incompatible fast votes require a higher ballot that orders the commands |
| [[EPaxos]] | Non-commuting commands observed in different orders | Dependencies and sequence numbers order execution; [[EPaxos-Revisited-2021]] shows conflict rate depends on workload, topology, load, batching, and timing |
| [[Mencius]] | Consensus-instance contention is avoided by owner assignment; execution conflicts only matter for optional out-of-order commit | Non-owners can only propose `no-op`; commutable operations may execute out of sequence when dependencies permit |
| [[PigPaxos]] | Client operation concurrency is serialized by the stable Multi-Paxos leader | No dependency conflict handling; leader assigns commands to log slots and relays only aggregate acknowledgements |
| [[Atlas]] | Non-commuting commands observed before one another at fast-quorum processes | Dependency union plus batch execution; fast path can still succeed if each final dependency is reported at least `f` times |
| [[SwiftPaxos]] | Conflicting commands with differing dependency proposals | Leader proposal plus FastAck/SlowAck evidence; acyclic deps |
| [[Pando]] | Concurrent writes/proposals for a version | Higher proposal numbers recover chosen values; fallback leader |
| [[Rabia]] | Replicas propose different oldest pending requests for a slot | If no majority proposal is visible, decide `⊥`, requeue the request, and try later |
| [[CURP]] | Non-commuting operations among unsynced master operations or witness records | Witness rejects or master syncs to backups before replying; only mutually commutative unsynced requests can be replayed unordered |
| [[Hermes]] | Concurrent same-key updates may begin from the same version at different coordinators | Lexicographic `[version, cid]` gives one endpoint order; ordinary writes never abort, writes outrank racing RMWs, and only the highest concurrent RMW commits |
| [[Copilot]] | Cross-log dependencies are incompatible if neither entry is ordered after the other, which could yield different combined orders | Replica rejects the initial dependency and suggests a later prefix; regular Accept persists the selected dependency; deterministic pilot priority orders cycles |
| [[Avicenna]] | Different real-log commands at one index can arise only across phases; shadow order may differ but is never executable | Preserve committed evidence during merge; otherwise choose the highest-phase accepted command; one real leader prevents same-phase collisions |
| [[Bodega]] | A local read overlaps an in-flight write to the same key and cannot yet know whether the local accepted value will commit | Hold the read until Commit or `m` early accept notifications; client timeout duplicates it at another responder/leader |
| [[Jetpack]] | Application-defined commands whose relative order can change state or response; checked against all in-flight commands in both paths | Reject fast acknowledgement and let the unchanged host protocol order/commit; recovery marker keeps stale conflicts behind recovered fast commands |
| [[Hydra]] | Groupcasts with overlapping destination-group sets | Order every overlapping pair by `(clock, sequencer ID)`; do not order disjoint groupcasts |
| [[HydraPaxos]] | Deterministic state-machine operations plus missing ordered positions | Execute in Hydra order; recover the missing operation or agree on `NO-OP` before advancing |
| [[WPaxos]] | Commands on the same object and concurrent attempts to own that object | One per-object `(ballot, slot)` log orders commands; higher ballot wins steals, ID tie-break/backoff handles dueling |


