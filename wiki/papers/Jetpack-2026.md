---
type: paper
title: "Jetpack: Consensus Made Generally Fast"
authors: Ze Tang; Zihao Zhang; Weihai Shen; Jicheng Shi; Shuai Mu
year: 2026
venue: 20th USENIX Symposium on Operating Systems Design and Implementation (OSDI '26)
source: raw/jetpack.pdf
protocols: [Jetpack]
tags: [fast-path, paxos, plugin, view-change, recovery, commutativity, linearizability]
status: ingested
---

# Jetpack: Consensus Made Generally Fast

## One-sentence summary

Jetpack adds a parallel 1-RTT, conflict-free fast path to compatible existing consensus protocols, using original-proposer promises during stable views and an explicitly ordered recovery-set protocol to preserve those promises across view changes.

## Why this paper matters

Fast paths are normally designed together with their host consensus protocols. Jetpack identifies the structural conditions needed to retrofit one: a host must preserve proposer insertion order through log/execution order, while the fast layer must make same-view evidence recoverable and force recovered fast commands ahead of stale conflicting work after a view change.

The paper's central warning is the [[view-change-hazard]]. Recovering the fast-committed value is necessary but insufficient; recovery must also preserve the relative order promised to the client. The paper demonstrates how prior extensions can lose a fast-committed command or place an older conflicting command before it.

## System model

- `n = 2f + 1` crash-fault-tolerant replicas; progress requires a responsive majority.
- Asynchronous network: messages may be delayed arbitrarily, and replicas may crash, slow down, or be isolated by partitions.
- Each replica runs a Jetpack shim colocated with one replica of an existing original consensus protocol.
- The original protocol provides views containing a monotonic identifier, membership, and stable proposer set.
- Commands are independent state-machine operations; the application supplies `Conflict(c1, c2)`.
- Interactive transactions and Byzantine faults are outside scope.

Source: `raw/jetpack.pdf`, §§2-3.

## Fault model

- Up to `f` crash failures with `2f + 1` replicas.
- Fail-slow behavior and network partitions are handled through the original protocol's view-change/failure-detection machinery.
- Byzantine replicas are not tolerated.
- Jetpack's fast path is not automatically fail-slow tolerant even when the original path is; for example, [[Copilot]] retains slowdown tolerance only on its original path.

## Timing assumptions

Safety is stated over the asynchronous model and depends on quorum evidence, view tags, ordering prerequisites, and recovery markers rather than timeout values. Liveness inherits the original protocol's view-change and majority-progress assumptions.

Fast-path latency is 1 client RTT when a conflict-free command reaches the required superquorum. The original path continues independently and may finish first. Recovery temporarily pauses Jetpack command processing; the naive algorithm has three exchanges before resubmission, while the optimized implementation merges metadata work into two RPCs.

Source: `raw/jetpack.pdf`, §§2-4 and Appendix B.2.

## Main idea

Each fast-path command is issued concurrently to a Fast-Paxos-style layer and to every original-path proposer. A Jetpack replica acknowledges only if the command's view matches and it conflicts with no uncommitted command in either path. An original proposer receiving it also proposes it through the original path.

The client fast-commits only after a same-view superquorum acknowledges and all original proposers have made the ordering promise. Otherwise the command simply completes through the unmodified original path. The original path eventually commits every fast-path command, making the two paths converge.

## Protocol roles

- **Client library:** tracks the current view, chooses fast/original path, broadcasts fast commands, collects acknowledgements, waits for whichever path commits first, and may adapt the fast-attempt rate.
- **Jetpack replica:** shim colocated with an original replica; stores fast commands, checks conflicts, issues promises, forwards to a paired proposer, pauses for recovery, and garbage-collects finalized commands.
- **Original-path proposer:** proposes commands in the host protocol and must honor receipt/proposal/log ordering prerequisites.
- **Original replica:** retains the host protocol's voting, replication, failure detection, view change, and execution behavior.
- **Recovery coordinator:** pauses the last normal view, agrees on its recovery set, resubmits it to the new proposer, and resumes the fast path after a stability marker commits.

## Message types

- Client fast/original command and commit/reply messages, all view-tagged.
- Fast-path acknowledgement/promise and rejection.
- Host-protocol proposal/replication/commit messages.
- Garbage-collection notification after original-path finalization.
- Recovery: `BeginRecovery`, `BeginRecoveryOK`, `Prepare(v_n)`, `PrepareOK`, `Accept(v_n, rset_vn)`, `AcceptOK`, and `FinishRecovery(view)`.

Source: `raw/jetpack.pdf`, §§3.3, 5 and Figure 14.

## Local state

- Jetpack's current fast-path view, independent from each colocated original replica's view.
- Mode: normal or recovery.
- Per-view fast-path log of acknowledged commands.
- Per-view accepted/final recovery set `rset_vn`.
- Command pool containing all in-flight fast- and original-path commands for conflict checks.
- Current original view, membership, and proposer set.
- Original log position/status and host-protocol state.
- Client-side recent latency, CPU, and attempt-rate samples for adaptive selection.

## Normal path

Original-only commands enter the host protocol unchanged. Fast-path commands are broadcast to all Jetpack replicas and forwarded to every original proposer. A matching-view replica checks all uncommitted commands in both paths; if none conflict, it stores the command in its fast log and acknowledges. Proposers also place the command on the original path regardless of whether the fast check succeeds.

The client returns on the first successful path. Fast success gives 1 RTT; if the fast path fails, the already-running original path later commits the command. After original commitment/execution, Jetpack observes the reply and removes the finalized command from every command pool.

## Fast path

A command `c` fast-commits when:

- all acknowledgements from both paths use the same view;
- at least `f + ⌈f/2⌉ + 1` Jetpack replicas acknowledge;
- the acknowledging set includes every current original-path proposer; and
- each acknowledgement found `c` conflict-free against uncommitted commands in both paths.

The fast-path promise for multi-decree consensus is:

> If the fast path commits `c`, the original path will not propose any concurrent conflicting command before `c` in the log.

Non-conflicting commands may be reordered because `Conflict` must report when order can change state or response.

## Slow path

If views mismatch, a conflict is detected, a proposer rejects the promise, or the client cannot collect the required superquorum, fast commitment fails. The command is not abandoned: it continues through the original path as a regular command, which supplies ordering, conflict resolution, batching, slowdown tolerance, or other host-specific behavior.

Adaptive clients reduce fast attempts under CPU saturation or when observed fast latency is not better, preserving original-path throughput when the fast path offers little benefit.

## Recovery path

Jetpack handles proposer election and membership reconfiguration with the same three-phase procedure:

1. **Detect and pause:** identify `v_n`, the last normal view from the most recent committed recovery-set marker; send `BeginRecovery` to `v_n`'s membership and wait for `f + 1` acknowledgements.
2. **Agree on recovery set:** send `Prepare(v_n)` and collect `f + 1` replies. Preserve any already accepted recovery set; otherwise include commands appearing in at least `⌈f/2⌉ + 1` replies. Send the set in `Accept` and finalize after `f + 1` `AcceptOK`s.
3. **Resubmit and resume:** submit the agreed set to the new original proposer, or submit `no-op` if empty. Only after this entry commits as the stability marker does the coordinator send `FinishRecovery` and enable normal processing in the new view.

Accepted recovery sets are returned by later prepares, giving one recovery set for the last normal view even across cascading failed views. Recovery of fast commands has priority over any other new-view command.

Source: `raw/jetpack.pdf`, §4.3 and Appendix B.

## Commit rule

- **Fast commit:** same-view acknowledgements from a superquorum of at least `f + ⌈f/2⌉ + 1`, including all current original proposers.
- **Original commit:** unchanged host-protocol rule.
- **Recovery-set acceptance:** `f + 1` `AcceptOK`s in the last normal view's replica set.
- **Recovery candidate threshold:** command appears in at least `⌈f/2⌉ + 1` of `f + 1` prepare replies when no accepted set exists.
- **New-view activation:** recovered set or `no-op` must commit through the original path as a stability marker before Jetpack resumes.

Fast client commitment may precede original commitment, but the original path must eventually commit the same command with the promised relative order.

## Quorum system

- Total replicas: `n = 2f + 1`.
- Original/recovery majority: `f + 1`.
- Minimum fast superquorum: `f + ⌈f/2⌉ + 1`.
- Proposer constraint: every current original proposer belongs to the fast acknowledgement set; with many proposers this can strengthen the effective quorum beyond the numerical minimum.
- Minimum fast/recovery intersection: `⌈f/2⌉ + 1`.
- Recovery candidate threshold: `⌈f/2⌉ + 1` reports.

The fast quorum is not fixed to one membership set, but it is view-specific and proposer-constrained. Acknowledgements from different views cannot be combined.

## Conflict handling

The application defines `Conflict(c1, c2)` so it is true whenever relative order can change state or a response. Jetpack replicas check new fast commands against all in-flight commands from both paths. Any conflict can block the superquorum; the host protocol then orders and commits the command normally.

For key-value workloads, two operations conflict when they access the same key and at least one writes. Range queries, SQL predicates, aliases, and opaque dependencies require application-specific indexes or coarser conflict scopes. Incorrect under-approximation would invalidate safety.

## Safety argument

Jetpack assumes the original protocol provides durability, fixed committed positions, in-order execution, and linearizability. It also requires:

- **PR 1:** for conflicting commands proposed by one proposer, proposal order determines log/execution order.
- **PR 2:** if a proposer receives `A` before `B`, it proposes `A` before `B`.
- **Principle 1:** fast-path view is independent; a fast commit uses same-view evidence from both paths.
- **Principle 2:** a new view recovers the last normal view's fast commands and commits the recovery set/marker before other command processing.

The fast-path invariant states that if `A` fast-commits in view `v`, no concurrent conflicting original-path command commits before `A` during `v`.

The proof establishes:

- **C1 Durability:** a fast-committed command appears on the fast superquorum; every recovery majority sees it in at least `⌈f/2⌉ + 1` replies and resubmits it.
- **C2 Execution consistency:** original recovery cannot place an uncommitted conflicting command before `A`, and fast recovery is complete and excludes a conflicting fast-committed command.
- **C3 Linearizability:** a later conflicting command in the same view cannot fast-commit ahead of `A`, while a higher view cannot accept it until `A` is recovered and committed.

Source: `raw/jetpack.pdf`, §§3.3-4 and Appendix B.3.

## Liveness argument

The original path remains live under its own assumptions because Jetpack does not block or change its commit rules during normal operation. A failed fast attempt simply waits for that path. During a view change, Jetpack intentionally pauses until a majority agrees on and the original path commits the recovery set/marker.

Recovery can be retried because accepted recovery sets are carried forward. Cascading failed views target the same last normal view. Progress therefore additionally needs eventual completion of the original view change, a responsive majority of the last normal view for recovery evidence, and a responsive new-view proposer/membership to commit the marker.

## Key proof ideas

- Treat a fast commit as an ordering promise by the original proposer set, not merely a separate decision.
- Make fast evidence view-specific; never combine acknowledgements across views.
- Derive the recovery threshold from fast/recovery intersection.
- Persist one recovery set with a majority before resubmission.
- Commit a stability marker before accepting new-view work, giving recovered commands priority over stale uncommitted entries.
- Separate host compatibility prerequisites (`PR 1`, `PR 2`) from Jetpack's own recovery principles.
- For shared-log multi-leader hosts, prove every fast command must reach every leader, yielding `Ω(M)` per-command processing for `M` leaders.

## Important formulas

Replica and quorum sizes:

```text
n = 2f + 1
q_original = q_recovery = f + 1
q_fast = f + ⌈f/2⌉ + 1
```

Intersection:

```text
|Q_fast ∩ Q_recovery|
  ⩾ (f + ⌈f/2⌉ + 1) + (f + 1) - (2f + 1)
   = ⌈f/2⌉ + 1
```

Recovery includes a possible fast command when it appears in:

```text
⌈f/2⌉ + 1 of the f + 1 PrepareOK replies
```

Shared-log multi-leader lower bound:

```text
per-command processing cost = Ω(M)
```

where `M` is the number of leaders, under client-side 1 RTT, order consistency, and non-intrusive integration.

## Relationship to other protocols

Jetpack uses a Fast-Paxos-style superquorum but delegates slow-path ordering and ordinary recovery to its host. It can layer onto [[Mencius]] and [[Copilot]], though multi-leader hosts incur extra duplicate processing. [[EPaxos]] is incompatible as presented because its dependency/topological execution can violate `PR 1`. [[SwiftPaxos]] and several other protocols already contain 1-RTT fast paths and do not need Jetpack.

The design resembles [[CURP]] in running a fast layer beside an original path, but Jetpack focuses on reusable view-change requirements. It identifies hazards in a sketched CURP-on-Raft extension without challenging CURP's primary-backup design.

## Limitations

- `PR 1` is essential for meaningful application. If buffering violates `PR 2`, Jetpack must delay its fast reply until the host actually proposes the command, reducing or eliminating the 1-RTT advantage.
- Requires an application-sound conflict predicate.
- Does not target Byzantine protocols or interactive transactions.
- Fast path adds messages, CPU, and command-pool memory; throughput can fall under saturation.
- The fast path does not inherit every host performance property, such as Copilot slowdown tolerance.
- Shared-log multi-leader integration has unavoidable `Ω(M)` per-command processing and duplicate suppression.
- View changes pause fast processing and add recovery downtime.
- Fast success falls with contention or insufficient same-view/proposer acknowledgements.

## Open questions

- Can [[EPaxos]] execution be constrained to satisfy `PR 1` without losing its dependency advantages?
- Can promises be represented durably in the host log to simplify or shorten Jetpack recovery?
- Can a BFT version derive analogous proposer and recovery thresholds?
- How should transactional conflict/order promises span shards?
- Can multi-leader hosts avoid `Ω(M)` processing by relaxing non-intrusiveness or client-side 1 RTT?
- Can conflict predicates be verified or conservatively synthesized from application code?

## Related pages

[[Jetpack]], [[view-change-hazard]], [[FastPaxos]], [[fast-path]], [[slow-path]], [[conflict]], [[quorum]], [[leader]], [[recovery]], [[agreement]], [[linearizability]], [[recoverability]], [[quorum-intersection]], [[adopt-commit-abstraction]], [[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[commit-rules]], [[recovery-rules]], [[proof-techniques]], [[latency]]
