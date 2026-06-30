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

For formal modeling, define the fast path by its commit predicate and recovery obligation: if something fast-commits, every later recovery quorum must contain enough evidence to preserve the same decision or compatible metadata.

[[PigPaxos]] is a useful non-example: its relay aggregation optimizes the common Multi-Paxos path, but it is not a Fast Paxos-style fast path because the commit predicate is still ordinary majority acceptance.

## Related pages
[[FastPaxos]], [[EPaxos]], [[EPaxosStar]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[fast-paths]], [[slow-path]], [[recovery]], [[quorum]]
