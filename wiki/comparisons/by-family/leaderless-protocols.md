---
type: comparison-family
family: leaderless protocols
protocols: [EPaxos, EPaxosStar]
tags: [leaderless]
---

# Leaderless Protocols

[[EPaxos]] and [[EPaxosStar]] are the currently ingested leaderless SMR protocols. [[FastPaxos]] can bypass the coordinator in prepared fast rounds, but still has coordinators per round. [[SwiftPaxos]] and [[Pando]] use leaders/delegates in important paths.

[[EPaxos-Revisited-2021]] is the main cautionary evaluation source: leaderless fast-path commit can improve median latency while still producing poor tail execution latency when conflicts, batching, or dependency chains appear.

[[Making-Democracy-Work-2025]] is the main correctness-oriented leaderless source: it argues that original EPaxos recovery is ambiguous and buggy, and gives EPaxos* with validation-based recovery and the optimal `n >= max{2e + f - 1, 2f + 1}` bound.
