# linearizability

Linearizability gives each operation a single point between invocation and response; PANDO targets linearizable key-value operations, while EPaxos/SwiftPaxos target SMR order.

[[PigPaxos]] targets the same linearizable SMR behavior as Multi-Paxos: one stable leader orders operations into log slots, and relay aggregation does not change the order/commit semantics.

[[Atlas]] targets linearizable SMR by ensuring validity, integrity, and acyclic ordering of real-time plus conflicting execution order. Commands that commute need not execute in the exact same order at every process.

[[Rabia]] targets log-based SMR linearizability by making replicas execute the same sequence of non-`⊥` decided requests in slot order. Duplicate client requests are skipped using unique IDs.

[[CURP]] preserves primary-backup linearizability while replying before backup sync. Its proof relies on witness durability for completed unsynced operations, master-side sync before non-commuting dependent observations, and exactly-once duplicate filtering during replay.

[[Copilot]] proves real-time order across its two logs with dependency compatibility: after command `a` completes at `P.i`, a later command `b` in the other log cannot commit with a dependency before `P.i`. Per-log agreement, cross-log orientation, deterministic cycle priority, and `(cliid, cid)` deduplication then give one total client-operation order.

[[Avicenna]] proves that a command committed at real-log index `k` remains at `k` in every later phase. This committed-prefix invariant gives a common total execution order. If `a` completes before `b` is proposed, the later real leader already preserves the committed prefix containing `a`, so it assigns `b` a later index. The shadow log is irrelevant to linearizability because it is never executed.

[[Bodega]] proves the new local-read case by comparing a prior acknowledged write's ballot to the reader's stable roster ballot. A higher ballot is impossible by majority lease uniqueness; an equal ballot's write quorum covers the responder; and a lower ballot's write majority intersects the responder's majority grantor subset, whose `thresh_p` forces catch-up through the write. Ordinary fallback operations inherit classic consensus linearizability.

[[Jetpack]] requires a later conflicting command to execute after a fast-replied command. In the same view, the fast superquorum prevents another conflict from fast-committing and proposer ordering places it later on the host path. In a higher view, the recovery set/stability marker must commit before new commands are accepted. Host linearizability then applies.

[[HydraPaxos]] guarantees linearizability for deterministic state machines. Hydra supplies one delivery order or an ordered gap; replicas resolve every gap to the recovered operation or an agreed `NO-OP`, and a client completes only after consistent replies from a majority including the executing leader.

[[WPaxos]] provides per-object linearizability, not one global total order. Each object's commands execute by increasing slot under one ballot owner at a time; different object logs may advance independently. Multi-object atomicity is outside the basic algorithm.

## Related pages
[[PigPaxos]], [[HydraPaxos]], [[WPaxos]], [[Atlas]], [[Rabia]], [[CURP]], [[Copilot]], [[Avicenna]], [[Bodega]], [[Jetpack]], [[roster-lease]], [[view-change-hazard]], [[agreement]], [[recovery]], [[quorum]]
