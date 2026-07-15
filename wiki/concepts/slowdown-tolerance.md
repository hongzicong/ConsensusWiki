# slowdown-tolerance

[[slowdown-tolerance]] is a performance property for an RSM whose replicas may keep responding but much more slowly than normal. It is distinct from crash tolerance: every failed replica is slow, but a slow replica may still send heartbeats and protocol messages, preventing ordinary failure detectors from replacing it.

[[Copilot-2020]] defines a replica as slow when its local receive-to-send response time exceeds its normal response time by a threshold `t`. Link traversal time is excluded from that replica-speed definition.

For `s`-slowdown-tolerance, sort replicas so `r1, ..., rs` are the `s` slowest. Let `T` describe current RSM response-time behavior and let `T'` describe the counterfactual system obtained by replacing those `s` replicas with clones of `r(s+1)`. The RSM is `s`-slowdown-tolerant when the difference between `T` and `T'` is close to zero. The paper leaves "close" qualitative rather than giving a fixed metric or bound.

For end-to-end RSM slowdown tolerance, redundancy must cover:
- receiving a command,
- ordering it,
- executing it,
- replying to the client.

[[Copilot]] targets `s = 1` with two proactive paths through a pilot and copilot. Per-entry fast takeover prevents the fast path from waiting indefinitely for unresolved ordering work from the slow path; null dependency elimination avoids repeated takeovers once duplicates from a continually slow pilot become semantically irrelevant.

[[Avicenna-2026]] argues that the replacement baseline above is unachievable in some geo-distributed placements. It instead defines `ε-s-Fail-Slow Fault Tolerance`: client latency with `s` fail-slow replicas may exceed the latency of the corresponding system with those replicas removed by at most `ε`. Removing replicas represents ideal instantaneous isolation while preserving a physically realizable geographic configuration.

[[Avicenna]] targets `s = 1`. A single real leader orders commands, an independent shadow leader estimates the alternative latency through [[counterfactual-evaluation]], and the system rotates when real performance is sufficiently worse. A slow follower is bypassed by the `f + 1`-of-`f + 2` non-standby quorum; a slow real leader is replaced by the shadow leader.

Do not equate slowdown tolerance with safety or ordinary liveness. A protocol may remain safe and eventually live while its client latency grows with a slow participant. Slowdown tolerance asks whether performance remains close to the counterfactual without that slow participant.

## Related pages
[[Copilot]], [[Avicenna]], [[counterfactual-evaluation]], [[failure-model]], [[liveness]], [[latency]], [[leader]], [[recovery]]
