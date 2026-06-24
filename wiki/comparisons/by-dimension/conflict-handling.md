---
type: comparison-dimension
dimension: conflict handling
protocols: [FastPaxos, EPaxos, SwiftPaxos, Pando]
tags: [conflict]
---

# Conflict Handling

| Protocol | Conflict meaning | Resolution |
|---|---|---|
| [[FastPaxos]] | Concurrent proposals accepted in a fast round | Collision recovery chooses a safe value |
| [[EPaxos]] | Non-commuting commands observed in different orders | Dependencies and sequence numbers order execution; [[EPaxos-Revisited-2021]] shows conflict rate depends on workload, topology, load, batching, and timing |
| [[SwiftPaxos]] | Conflicting commands with differing dependency proposals | Leader proposal plus FastAck/SlowAck evidence; acyclic deps |
| [[Pando]] | Concurrent writes/proposals for a version | Higher proposal numbers recover chosen values; fallback leader |
