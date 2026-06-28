# New Protocol Ideas

This page is a speculative backlog grounded in the ingested wiki. Claims about existing protocols should point to source pages; new design moves are marked as `Idea`, `Hypothesis`, `TODO`, or `Unclear`.

## Design Ingredients From The Wiki

- [[EPaxosStar|EPaxos*]] separates full crash tolerance `f` from fast-path failure budget `e`, with fast quorums `n - e`, slow/recovery quorums `n - f`, and optimized bound `n >= max{2e + f - 1, 2f + 1}`.
- [[SwiftPaxos]] uses leader-including fast quorums and dependency-path evidence to simplify dependency recovery relative to fully leaderless fast evidence.
- [[Mencius]] assigns ownership by instance and makes `SKIP` safe through an owner/non-owner value restriction, not through a fast quorum certificate.
- [[Pando]] separates discovery-style Phase 1a evidence from stronger Phase 1b reconstruction evidence.
- [[EPaxos-Revisited-2021]] warns that fast commit latency and execution latency are different: dependency chains can make a committed command execute late.
- [[quorum-intersection]] and [[commit-rules]] suggest the design order: define the committed object and recovery predicate first, then derive quorum intersections.

## Idea: Evidence-Split Fast SMR

Motivation: dependency-based SMR often pays a high recovery complexity cost because the fast path must leave enough evidence for later recovery. [[Pando]] suggests a separation between cheap discovery and stronger reconstruction, while [[EPaxosStar|EPaxos*]] shows that validated recovery can be the central safety mechanism.

Source mechanisms: [[Pando]] Phase 1a/Phase 1b split, [[EPaxosStar|EPaxos*]] validation, [[fast-paths]], [[recovery-rules]], [[quorum-systems]].

Design sketch: a command leader tries to fast-commit a command using a compact fast certificate containing command id, payload id, dependency summary, conflict witnesses, and a certificate class. Recovery first asks a small discovery quorum whether the certificate is clearly safe, clearly absent, or ambiguous. Only ambiguous cases trigger a larger reconstruction quorum that gathers full dependency evidence and runs validation.

Hypothesis: separating "is there dangerous fast evidence?" from "reconstruct all metadata needed for commit" may reduce normal-case metadata and make recovery proofs modular.

Proof obligations:

- Define the agreed object: likely `(cmd, dep, certClass)` or `(cmd, dep)` plus recoverable certificate evidence.
- Prove that any committed fast certificate is visible to every recovery path that might choose a conflicting command.
- Derive discovery/reconstruction quorum inequalities from [[quorum-intersection]], not by copying [[Pando]] storage quorums directly.
- Prove validation preserves EPaxos*-style visibility for conflicting committed commands.

Risks:

- If the discovery quorum is too weak, recovery may miss a possible fast commit.
- If the certificate omits too much dependency information, reconstruction may become equivalent to full EPaxos* recovery.
- Pando is storage-oriented, not SMR; its split-quorum idea cannot be transferred without a new dependency visibility proof.

TODO:

- Work out a minimal abstract model with certificate states `Absent`, `Possible`, `Recoverable`, and `Committed`.
- Derive candidate inequalities for `n`, `f`, `e`, discovery quorum size, and reconstruction quorum size.

## Idea: Conflict-Class Anchors

Motivation: [[EPaxos]] distributes command leadership well, but recovery is subtle; [[SwiftPaxos]] simplifies fast evidence by requiring leader involvement. A middle ground may anchor only commands that can conflict with one another.

Source mechanisms: EPaxos command leaders in [[leaderless-protocols]], SwiftPaxos leader-including fast quorums in [[leader-roles]], the fusion sketch in [[FastPaxos-EPaxos-SwiftPaxos]], and conflict classes in [[conflict-handling]].

Design sketch: assign each conflict class, key range, shard, or commutativity domain an anchor. Any command may be submitted to any replica, but fast evidence for commands in the same conflict class must include the class anchor or an anchor-signed dependency proposal. Non-conflicting classes proceed independently. Recovery for a command contacts a quorum plus the relevant anchor evidence when the anchor is live, and falls back to a full recovery quorum when it is not.

Hypothesis: per-conflict anchors could preserve much of EPaxos's load distribution while making recovery for actually conflicting commands closer to SwiftPaxos.

Proof obligations:

- Define conflict-class membership and show that all potentially interfering commands share at least one anchor or a deterministic anchor-selection rule.
- Prove that anchor evidence cannot hide a conflicting committed command from recovery.
- Prove agreement for the per-command dependency object and acyclicity or deterministic execution for committed dependency components.
- Specify fallback when the anchor crashes or a command spans multiple conflict classes.

Risks:

- Hot conflict classes may recreate a single-leader bottleneck.
- Multi-key commands may need multiple anchors, which can introduce deadlock or dependency cycles.
- If anchors are dynamic, reconfiguration may become the hardest part of the protocol.

TODO:

- Start with a static key-to-anchor function and single-key commands.
- Compare two agreed objects: EPaxos-style `(cmd, seq, deps)` versus SwiftPaxos-style dependency paths.

## Idea: Rotating Dependency Ownership

Motivation: [[Mencius]] avoids per-instance collisions by assigning each log slot to a coordinator, while [[EPaxosStar|EPaxos*]] avoids a global log but needs dependency recovery. A hybrid could rotate ownership over dependency slots or conflict classes rather than over every global log index.

Source mechanisms: Mencius ownership and `SKIP`, EPaxos* dependency graph, [[commit-rules]], [[proof-techniques]], [[rocq-modeling-notes]].

Design sketch: define ownership over `(class, epoch)` dependency slots. The owner for a slot may propose the ordering fence, dependency summary, or `no-op`/`skip` for that class and epoch. Commands still commit as dependency-graph nodes, but execution can use class-local fences to cut long dependency chains and make idle periods explicit.

Hypothesis: owner-authored no-op evidence could help dependency protocols distinguish "no known conflict in this class/epoch" from "missing evidence because the coordinator failed."

Proof obligations:

- Model the owner restriction: only the owner can certify a non-empty fence or an owner-authored no-op for its slot.
- Prove a dependency visibility invariant across commands that cross slot boundaries.
- Keep chosen, learned, committed, and executable states separate, as in [[Mencius]] modeling notes.
- Show that revocation of a failed owner preserves any possible prior fence or no-op.

Risks:

- The design may import the worst of both worlds: Mencius gaps plus EPaxos dependency recovery.
- Slot fences may serialize workloads that would otherwise commute.
- Owner-authored no-ops are not quorum certificates; modeling them incorrectly would produce a false proof.

TODO:

- Build a tiny TLA+ sketch with `owner(class, epoch)` and in-order execution per class.
- Check whether class-local no-ops reduce execution wait chains observed in [[EPaxos-Revisited-2021]] style workloads.

## Idea: Execution-Aware Fast Commit

Motivation: [[EPaxos-Revisited-2021]] shows that fast commit alone can be misleading if commands wait behind unresolved dependencies. A protocol could make the fast path optimize for commit-to-execute latency rather than only quorum response latency.

Source mechanisms: EPaxos commit/execution distinction in [[fast-paths]] and [[commit-rules]], SwiftPaxos acyclic dependency graph, EPaxos* SCC execution over committed dependencies.

Design sketch: fast commit requires not only matching dependency metadata but also an execution-readiness predicate: dependencies are already committed, known to be concurrently certifiable, or bounded by a small unresolved-dependency budget. If the predicate fails, the protocol deliberately takes a slow/repair path that resolves dependencies before announcing fast success.

Hypothesis: giving up some fast commits may improve tail execution latency and simplify evaluation-sensitive invariants.

Proof obligations:

- Define execution readiness without using timing as safety evidence.
- Prove that the readiness predicate is monotonic or specify how later recovery handles non-monotonic readiness observations.
- Preserve the base agreement/visibility proof even when fast commit is denied for performance reasons.

Risks:

- The fast path may become too conservative under benign concurrency.
- Readiness evidence may require extra messages that erase the latency benefit.
- The protocol must avoid making liveness depend on a perfectly accurate conflict predictor.

TODO:

- Separate safety predicate `CanCommit` from performance predicate `LikelyExecutableSoon`.
- Evaluate first as a policy layered over EPaxos* rather than as a new safety core.

## Idea: First-Class Abort / No-Op Recovery

Motivation: [[EPaxosStar|EPaxos*]] may recover `Nop` and resubmit, while [[Mencius]] uses owner-authored `SKIP` to fill gaps cheaply. Treating abort/no-op as a first-class recovery outcome may make dependency protocols easier to model.

Source mechanisms: EPaxos* `Nop` recovery, Mencius `SKIP`, [[recovery-rules]], [[recoverability]].

Design sketch: every command id has an explicit terminal outcome: `Commit(cmd, dep)`, `Abort(id)`, or `Noop(id)`. Recovery may choose `Abort`/`Noop` only when validation shows that preserving the original payload is unsafe or unavailable. A client-facing resubmission rule gives the command a fresh id, so agreement remains per id while validity is stated over terminal outcomes.

Hypothesis: explicit terminal outcomes may simplify recovery proofs by avoiding half-committed command identifiers that block dependency execution.

Proof obligations:

- State validity carefully: an aborted id does not mean the client operation is chosen.
- Prove abort/no-op outcomes do not break visibility for commands that depended on the original id.
- Prove resubmission cannot duplicate a non-idempotent operation if the original command later becomes visible.

Risks:

- Client semantics may become subtle for operations with externally visible retries.
- Too-eager aborts may harm liveness or fairness.
- A no-op dependency node may still need to participate in SCC execution.

TODO:

- Compare two abstractions: no-op as an ordinary committed command versus no-op as a separate terminal state.
- Add Rocq lemmas for "terminal outcome uniqueness" and "aborted ids cannot later commit payload."

## Cross-Cutting Modeling Plan

For any of these ideas, start with a tiny abstraction before adding optimizations:

- State: command ids, payload ids, dependency sets or paths, evidence records, terminal outcomes, and quorum memberships.
- Messages: proposal, fast evidence, slow evidence, recovery request/reply, validation, commit, abort/no-op.
- Core invariants: per-id agreement, visibility for conflicting committed commands, recovery preservation of possible committed evidence, and execution determinism over dependency components.
- Liveness assumptions: keep synchrony, failure detectors, stable anchors/leaders, and quorum availability outside the safety invariant.
- First validation target: search small TLA+ configurations for counterexamples, then port stable invariants into Rocq/Coq.
