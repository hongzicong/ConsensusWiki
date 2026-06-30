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
- [[SwiftPaxos]] uses matching FastAck dependency evidence, with SlowAck fallback.

For formal modeling, define the fast path by its commit predicate and recovery obligation: if something fast-commits, every later recovery quorum must contain enough evidence to preserve the same decision or compatible metadata.

## Related pages
[[FastPaxos]], [[EPaxos]], [[EPaxosStar]], [[SwiftPaxos]], [[Pando]], [[fast-paths]], [[slow-path]], [[recovery]], [[quorum]]
