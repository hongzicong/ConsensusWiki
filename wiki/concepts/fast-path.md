# fast-path

A [[fast-path]] is the common-case route that commits or learns with fewer message delays than the fallback [[slow-path]]. It is not merely "the normal case"; it is a path whose evidence is already strong enough to make later [[recovery]] safe.

Typical fast-path requirements:
- a sufficiently large or specially shaped [[quorum]],
- matching votes or metadata,
- no unresolved conflict/collision, or a dependency rule that safely records the conflict,
- a fallback rule when the fast evidence is ambiguous.

Examples:
- [[FastPaxos]] can learn in a fast round when a fast quorum accepts the same value.
- [[EPaxos]] can fast-commit when replicas return matching `(cmd, seq, deps)` attributes; execution can still wait on dependencies.
- [[EPaxosStar]] can fast-commit when a quorum of size at least `n - e` returns dependency sets equal to `initDep[id]`.
- [[Atlas]] can fast-commit with non-matching dependency replies when `union_Q dep = union^f_Q dep`.
- [[SwiftPaxos]] uses matching FastAck dependency evidence, with SlowAck fallback.
- [[Rabia]] can terminate Weak-MVC in three message delays; the fast outcome may be a request or a forfeited `⊥` slot.
- [[CURP]] can complete primary-backup updates in 1 RTT when all witnesses durably record the request and the master can safely execute it speculatively.
- [[Copilot]] can fast-commit an initial cross-log dependency after `f + floor((f + 1) / 2)` compatible `FastAcceptOk` replies including the proposing pilot.

For formal modeling, define the fast path by its commit predicate and recovery obligation: if something fast-commits, every later recovery quorum must contain enough evidence to preserve the same decision or compatible metadata.

[[PigPaxos]] is a useful non-example: its relay aggregation optimizes the common Multi-Paxos path, but it is not a Fast Paxos-style fast path because the commit predicate is still ordinary majority acceptance.

[[GPaxos]] is a positive fast-path example: in a fast ballot, proposers send directly to acceptors and learners can learn in two message delays when a quorum reports compatible c-structs containing the proposed command.

[[CURP]] is a useful boundary case: it has a true 1 RTT fast path, but the evidence is unordered witness durability plus commutativity, not an ordered consensus quorum.

[[Copilot]] shows why fast evidence and recovery evidence must be modeled together. A recovery majority may see only `floor((f + 1) / 2)` fast accepts from a committed fast quorum; in the ambiguous count range, safe recovery also examines the other pilot's log.

[[Bodega]] is a read-path boundary case: a stable, caught-up responder answers locally when its newest key write is committed or has `m` early accept notifications. This is a true client-latency fast path, but it does not change the write-consensus value-selection rule.

[[Jetpack]] makes the fast path a reusable shim: a command fast-commits after same-view acknowledgements from `f + ceil(f/2) + 1` replicas including all host proposers. The host path runs concurrently and must later honor the same conflict order across view changes.

[[FPaxos]] is a non-example: a smaller stable-leader Phase 2 quorum can reduce latency and load, but it does not bypass the leader, eliminate a phase, or introduce a separate fast-round commit predicate.

[[OmniPaxos]] is also a non-example. Its prepared Sequence Paxos leader decides through one pipelined leader-to-majority round trip, but there is no separate fast quorum, client bypass, or conflict-dependent commit predicate.

[[Hydra]] is an ordering-layer boundary case: its normal message uses one sequencer pass, but that produces ordered delivery/drop evidence rather than application commit. [[HydraPaxos]] turns the evidence into a one-client-RTT SMR path by waiting for consistent majority replies including the leader.

[[WPaxos]] is another non-example: after [[object-stealing]] establishes a stable owner, repeated local Phase 2 lowers WAN latency, but the owner remains on the path and no separate fast quorum or conflict-dependent predicate exists.

## Related pages
[[FastPaxos]], [[FPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], [[WPaxos]], [[GPaxos]], [[EPaxos]], [[EPaxosStar]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[CURP]], [[Copilot]], [[Bodega]], [[Jetpack]], [[fast-paths]], [[slow-path]], [[recovery]], [[quorum]]
