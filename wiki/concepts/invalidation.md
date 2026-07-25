# invalidation

An [[invalidation]] makes a local replica temporarily unable to answer reads for a key. It can replace read quorums only when the update protocol ensures every replica still authorized to serve is invalidated before the write becomes client-visible.

In [[Hermes]], `INV(key, epoch_id, TS, RMW_flag, value)` carries both the ordering metadata and the complete value needed for recovery. A follower adopts a higher timestamp and enters `Invalid`, acknowledges every `INV`, and returns to `Valid` only for a `VAL` whose timestamp exactly equals its local timestamp.

This resembles a lightweight lock but has different conflict semantics. Concurrent ordinary writes do not fail: lexicographic `[version, cid]` timestamps select the same order at every replica. Only RMWs may abort. An invalidated replica can also replay the original timestamp/value pair, so coordinator failure does not strand the key permanently.

Invalidation safety depends on [[reliable-membership]]: every current live member must be invalidated, and removed replicas must stop serving before a smaller live set commits.

## Related pages

[[Hermes]], [[Hermes-2020]], [[reliable-membership]], [[fast-path]], [[recovery]], [[conflict]], [[linearizability]]
