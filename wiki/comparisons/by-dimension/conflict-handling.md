---
type: comparison-dimension
dimension: conflict handling
protocols: [FastPaxos, EPaxos, Mencius, SwiftPaxos, Pando]
tags: [conflict]
---

# Conflict Handling

| Protocol | Conflict meaning | Resolution |
|---|---|---|
| [[FastPaxos]] | Concurrent proposals accepted in a fast round | Collision recovery chooses a safe value |
| [[EPaxos]] | Non-commuting commands observed in different orders | Dependencies and sequence numbers order execution; [[EPaxos-Revisited-2021]] shows conflict rate depends on workload, topology, load, batching, and timing |
| [[Mencius]] | Consensus-instance contention is avoided by owner assignment; execution conflicts only matter for optional out-of-order commit | Non-owners can only propose `no-op`; commutable operations may execute out of sequence when dependencies permit |
| [[SwiftPaxos]] | Conflicting commands with differing dependency proposals | Leader proposal plus FastAck/SlowAck evidence; acyclic deps |
| [[Pando]] | Concurrent writes/proposals for a version | Higher proposal numbers recover chosen values; fallback leader |


