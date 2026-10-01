---
type: comparison-family
family: leaderless protocols
protocols: [EPaxos, EPaxosStar, Atlas, Rabia, KCensus]
tags: [leaderless]
---

# Leaderless Protocols

[[EPaxos]], [[EPaxosStar]], [[Atlas]], and [[Rabia]] are the currently ingested leaderless SMR protocols. [[FastPaxos]] can bypass the coordinator in prepared fast rounds, but still has coordinators per round. [[SwiftPaxos]] and [[Pando]] use leaders/delegates in important paths. [[Copilot]] is not leaderless: it uses two distinguished, redundant pilots.

[[EPaxos-Revisited-2021]] is the main cautionary evaluation source: leaderless fast-path commit can improve median latency while still producing poor tail execution latency when conflicts, batching, or dependency chains appear.

[[Making-Democracy-Work-2025]] is the main correctness-oriented leaderless source: it argues that original EPaxos recovery is ambiguous and buggy, and gives EPaxos* with validation-based recovery and the optimal `n >= max{2e + f - 1, 2f + 1}` bound.

[[Atlas-2020]] is the main planet-scale quorum-tuning source: it keeps the leaderless dependency shape but uses fast quorums of `floor(n/2) + f`, slow quorums of `f + 1`, and recovery quorums of `n - f`.

[[Rabia-2021]] is the main randomized leaderless source: it removes command leaders and separate fail-over by deciding each slot with Weak-MVC, permitting `⊥` slots when proposals do not line up.

## KCensus fast-path qualification

[[KCensus]] permits any proposer to use a synthesized fast path without a fixed global leader, including non-voting clients. Its KSMR implementation nevertheless uses a Paxos leader on fallback. Classify the fast path and fallback separately rather than treating it as leaderless in the same sense as Rabia ([[KCensus-2026]], §§4–6).
