---
type: comparison-dimension
dimension: conflict handling
protocols: [FastPaxos, EPaxos, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia]
tags: [conflict]
---

# Conflict Handling

| Protocol | Conflict meaning | Resolution |
|---|---|---|
| [[FastPaxos]] | Concurrent proposals accepted in a fast round | Collision recovery chooses a safe value |
| [[EPaxos]] | Non-commuting commands observed in different orders | Dependencies and sequence numbers order execution; [[EPaxos-Revisited-2021]] shows conflict rate depends on workload, topology, load, batching, and timing |
| [[Mencius]] | Consensus-instance contention is avoided by owner assignment; execution conflicts only matter for optional out-of-order commit | Non-owners can only propose `no-op`; commutable operations may execute out of sequence when dependencies permit |
| [[PigPaxos]] | Client operation concurrency is serialized by the stable Multi-Paxos leader | No dependency conflict handling; leader assigns commands to log slots and relays only aggregate acknowledgements |
| [[Atlas]] | Non-commuting commands observed before one another at fast-quorum processes | Dependency union plus batch execution; fast path can still succeed if each final dependency is reported at least `f` times |
| [[SwiftPaxos]] | Conflicting commands with differing dependency proposals | Leader proposal plus FastAck/SlowAck evidence; acyclic deps |
| [[Pando]] | Concurrent writes/proposals for a version | Higher proposal numbers recover chosen values; fallback leader |
| [[Rabia]] | Replicas propose different oldest pending requests for a slot | If no majority proposal is visible, decide `⊥`, requeue the request, and try later |


