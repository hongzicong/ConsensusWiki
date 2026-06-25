# New Protocol Ideas

- Explore whether PANDO-style separated discovery/recovery quorums can be adapted to dependency-based SMR.
- Explore leader-including fast quorums as a way to simplify EPaxos recovery while preserving some leaderless load distribution.

## Designing Better Fast Paths

A better [[fast-path]] should be optimized for the whole commit-to-execute path, not only the first quorum response. [[EPaxos-Revisited-2021]] suggests that a command can commit quickly yet execute late because dependency chains or conflicting commands remain unresolved.

Promising directions:

- Separate fast-path failure budget from full fault tolerance, as in [[EPaxosStar|EPaxos*]] with `e <= f`, so the common case pays for a smaller synchronous-failure target while recovery still handles `f` crashes.
- Make fast evidence recovery-friendly. A fast quorum should leave enough evidence for a later recovery quorum to distinguish "definitely committed" from "only partially observed."
- Consider leader-including or leader-observed fast quorums, as in [[SwiftPaxos]], when that simplifies recovery more than it hurts load balance.
- Separate discovery from reconstruction, inspired by [[Pando]]: a small quorum may detect that the fast path is safe, while a larger recovery quorum reconstructs enough metadata only on fallback.
- Commit ordering metadata only when it is stable enough to execute, or explicitly track the difference between commit latency and execution latency.

Main proof obligations:

- agreement for the per-instance or per-command committed object,
- visibility/order for conflicting commands,
- recovery preservation of any possibly committed fast-path value,
- acyclicity or deterministic execution over dependency components,
- liveness after fallback under the full `f`-failure model.

Open design sketch: use a small "fast certificate" that contains command id, dependency summary, conflict witnesses, and a leader/delegate observation bit. Recovery would first test the certificate against a cheap discovery quorum; only ambiguous cases would gather full dependency evidence from a larger recovery quorum. TODO: derive exact quorum inequalities and validate against [[quorum-intersection]].

## EPaxos and SwiftPaxos Fusion

Question: can [[EPaxos]] keep its distributed command leaders while borrowing [[SwiftPaxos]]'s leader-including fast quorums, dependency-path evidence, and `SlowAck` repair?

Candidate design: each command has an EPaxos-style command leader, but fast-path evidence must include either:

- a current ballot leader/delegate, or
- a conflict-class anchor responsible for commands that may interfere, or
- a dependency-path certificate strong enough for SwiftPaxos-style recovery.

Potential benefit: retain EPaxos's load distribution for non-conflicting commands while simplifying recovery and avoiding some dependency-cycle/execution-lag issues using SwiftPaxos-style acyclicity.

Main risk: if the anchor is global, the protocol becomes closer to SwiftPaxos and loses EPaxos's leaderless load balancing; if the anchor is per command, recovery may become as subtle as original EPaxos. TODO: specify the agreed object and prove whether recovery preserves dependency visibility.
