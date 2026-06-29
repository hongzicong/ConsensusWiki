# New Protocol Ideas

This page is a speculative backlog grounded in the ingested wiki. Claims about existing protocols should point to source pages; new design moves are marked as `Idea`, `Hypothesis`, `TODO`, or `Unclear`.

Current design goal: promote only ideas that either make the safe fast path faster than the current baseline, or keep the fast path at the same 1RTT shape while making the slow path, fallback, recovery, or execution repair faster.

## Target Filter

| Track | Target | What counts as success | What does not count |
|---|---|---|---|
| A. Faster fast path | Improve the common case. | Same safety target with fewer waited-for remote replies, lower tail-latency quorum choice, less metadata processing on the critical path, or a narrower failure budget that is stated explicitly. | Adding an online anchor, validator, owner approval, or extra pre-commit check. |
| B. Same 1RTT fast path, faster slow path | Preserve the conflict-free 1RTT fast path. | Fast path is observationally equivalent to the baseline, while mismatch/failure/recovery uses fewer rounds, smaller evidence, or cheaper validation. | Making every clean command pay for recovery metadata that requires a new round or larger quorum. |
| Not a primary candidate | May still be useful as a model or policy. | Helps background repair, execution, or evaluation but is not a protocol-level latency win yet. | Slower fast path with only a vague recovery benefit. |

Important caveat: for replicated SMR, a safe durable commit normally still needs evidence from other replicas. So "faster than 1RTT" should be treated as `Unclear` unless the design changes the semantics, for example speculative execution, leases, local-only operations, or a weaker failure/visibility target. The more realistic Track A goal is usually "still one proposal/reply exchange, but waits for a faster/smaller/closer evidence set."

## Fast-Path Non-Regression Gate

Use this gate before promoting any idea.

| Gate | Requirement |
|---|---|
| Message delays | No extra client-visible communication step before fast commit or fast execution relative to the baseline protocol. |
| Waited-for quorum | Do not wait for more replies than the baseline fast quorum unless the design explicitly changes the fault target and derives a better bound. |
| Online roles | Do not add a new online anchor, owner, validator, or sequencer to the conflict-free fast path unless that role is already part of the chosen baseline, such as the SwiftPaxos ballot leader. |
| Validation | Recovery validation, reconstruction, and dependency repair must run only after mismatch, timeout, failure, or background execution repair. |
| Metadata | Extra metadata is acceptable only when piggybacked on existing fast-path messages and bounded enough that it does not change the protocol round structure. |
| Safety | The fast evidence must still be sufficient for recovery to preserve any possibly committed command. |

## Baselines From The Wiki

| Baseline | Fast-path shape | Slow/fallback shape | Improvement target |
|---|---|---|---|
| [[EPaxos]] | Command leader collects matching PreAccept attributes from a fast quorum. | Accept with majority after disagreement; recovery is subtle. | Same 1RTT fast path, but cheaper disagreement handling and recovery. |
| [[EPaxosStar|EPaxos*]] | Matching `PreAcceptOK` dependency sets from quorum size `n - e`; optimized bound `n >= max{2e + f - 1, 2f + 1}`. | Recover/Validate/Waiting; may recover `Nop`. | Preserve `n - e` fast quorum while reducing validation/recovery cost. |
| [[SwiftPaxos]] | Matching FastAck dependency-path evidence from a leader-including fast quorum. | SlowAck/Sync repair disagreement. | Borrow repair ideas without putting a new leader/anchor on EPaxos-style fast path. |
| [[FastPaxos]] | Values are accepted directly in an opened fast round. | Collision recovery chooses a safe value in a higher round. | Faster collision recovery, not a slower fast round. |
| [[Mencius]] | Local rotating coordinator path; `SKIP` is owner evidence, not a fast quorum certificate. | Revocation fills failed-owner gaps. | Use cheap owner evidence for gaps/recovery without requiring owner approval on every dependency fast path. |

## Design Ingredients From The Wiki

- [[EPaxosStar|EPaxos*]] separates full crash tolerance `f` from fast-path failure budget `e`, with fast quorums `n - e`, slow/recovery quorums `n - f`, and optimized bound `n >= max{2e + f - 1, 2f + 1}`.
- [[SwiftPaxos]] uses leader-including fast quorums and dependency-path evidence to simplify dependency recovery relative to fully leaderless fast evidence.
- [[Mencius]] assigns ownership by instance and makes `SKIP` safe through an owner/non-owner value restriction, not through a fast quorum certificate.
- [[Pando]] separates discovery-style Phase 1a evidence from stronger Phase 1b reconstruction evidence.
- [[EPaxos-Revisited-2021]] warns that fast commit latency and execution latency are different: dependency chains can make a committed command execute late.
- [[quorum-intersection]] and [[commit-rules]] suggest the design order: define the committed object and recovery predicate first, then derive quorum intersections.

## Track A: Faster Fast Path Candidates

These ideas try to improve the common case itself. They are higher risk because quorum lower bounds can easily erase the apparent win.

### Candidate A1: Latency-Tiered Fast Quorums

Status: `Idea`, strongest Track A candidate if the quorum algebra works.

Motivation: EPaxos-style protocols can be 1RTT but still slow when the fast quorum waits for geographically distant or slow replicas. A fast path can be "faster" without fewer message delays if it waits for a lower-latency quorum that still satisfies the recovery intersections.

Source mechanisms: [[quorum-systems]], [[quorum-intersection]], [[EPaxosStar|EPaxos*]] `e`-fast quorum separation, [[SwiftPaxos]] fixed/leader-including fast quorum variants.

Design sketch: define a family of admissible fast quorums ranked by latency tier. The command leader chooses the lowest-latency admissible quorum for the command's conflict class, region, or shard. Recovery quorums must intersect every admissible fast quorum enough to preserve committed dependency evidence. If some tier is unavailable, the protocol falls back to the ordinary baseline fast quorum or slow path.

Why it might be faster: still 1RTT, but the command leader can stop after replies from closer/faster replicas instead of waiting for a globally uniform fast quorum.

Proof obligations:

- Define the exact fast-quorum family and recovery-quorum family.
- Prove all required fast-fast and fast-recovery intersections for dependency visibility.
- State whether the failure budget changes from `f`/`e`; if yes, mark the weaker availability target explicitly.
- Show that quorum choice cannot hide conflicting commands in different regions/classes.

Risks:

- A "nearby" quorum may be fast but too weak for recovery.
- Fixed fast quorums can become hotspots or availability bottlenecks.
- If the protocol silently lowers the failure target, the idea is not a fair improvement over EPaxos*/EPaxos.

TODO:

- Start with two latency tiers and derive inequalities for `n`, `f`, `e`, fast-tier size, and recovery size.
- Compare against SwiftPaxos C2 fixed-majority fast quorum and EPaxos* `n - e`.

### Candidate A2: Class-Local `e` Budgets

Status: `Idea`, promising only if the model exposes the weaker fast guarantee.

Motivation: [[EPaxosStar|EPaxos*]] already separates full crash tolerance `f` from fast-path failure budget `e`. Some conflict classes may deserve a smaller `e` for lower fast-path waiting, while others keep a stronger fast path.

Source mechanisms: EPaxos* `e`-fast design, [[quorum-systems]], [[conflict-handling]].

Design sketch: assign each conflict class a fast-path budget `e_c`. Low-risk or local classes use a smaller waited-for fast quorum only if the derived lower bound and recovery intersections remain valid for that class. High-risk classes keep the baseline `e` and quorum size. Multi-class commands use the maximum required evidence or deliberately take the slow path.

Why it might be faster: lower fast-path failure tolerance for selected classes may allow smaller or lower-latency fast quorums, but only if the quorum formulas actually permit it.

Proof obligations:

- Define class membership and multi-class command rules.
- Derive per-class quorum bounds instead of copying the global EPaxos* formula blindly.
- Prove a command in class `c` cannot be recovered using assumptions for a different class `d`.

Risks:

- The algebra may show no real quorum reduction under the same `n` and `f`.
- Multi-key commands can collapse back to the slowest/largest class.
- Different `e_c` values complicate Rocq/TLA+ models.

TODO:

- Build a small table for `f = 1, 2, 3` comparing global `e` vs class-local `e_c`.
- Mark any improvement that changes the failure target as a weaker-mode optimization, not a strict win.

### Candidate A3: Precomputed Owner Evidence For Gaps Only

Status: `Conditional`; can improve apparent fast execution only if owner evidence is prepared before the command arrives.

Motivation: [[Mencius]] uses owner-authored `SKIP` to fill idle slots cheaply. Dependency protocols sometimes wait to execute because dependencies or gaps are unresolved.

Source mechanisms: [[Mencius]] `SKIP`, [[EPaxos-Revisited-2021]] execution-delay warning, [[commit-rules]].

Design sketch: conflict-class owners periodically publish signed or quorum-backed "no known command in epoch" evidence outside the request critical path. A new command can then fast-commit in 1RTT and use already-published gap evidence to become executable sooner.

Why it might be faster: not faster commit, but potentially faster fast-path execution if dependency gaps are already closed before the command arrives.

Proof obligations:

- Specify whether owner evidence is single-owner evidence, quorum evidence, or merely a hint.
- Prove precomputed gap evidence cannot erase a concurrent conflicting command.
- Keep chosen, committed, and executable states separate.

Risks:

- If the command must wait for fresh owner evidence, the idea fails the fast-path gate.
- Owner evidence may serialize hot classes.
- This is more an execution-latency improvement than a commit-latency improvement.

TODO:

- Treat this as an execution optimization until a commit-latency proof exists.

### Candidate A4: Fast-Quorum Hedges With Early Stopping

Status: `Idea`, useful if it improves tail latency without changing the quorum predicate.

Motivation: a 1RTT fast path can be slow because the command leader sends to a broad set and waits for the slowest member needed to complete a fast quorum. Instead of changing safety, the leader can send to multiple candidate fast-quorum supersets and stop as soon as any admissible fast quorum returns matching evidence.

Source mechanisms: [[quorum-systems]], [[fast-paths]], [[EPaxosStar|EPaxos*]] fast quorum `n - e`, and SwiftPaxos fast-quorum variants.

Design sketch: define a set of admissible fast quorums. A command leader sends the same PreAccept/FastAck-style message to a larger witness set, possibly prioritized by latency. The first matching admissible quorum commits. Late replies become hints for recovery or execution repair but are not needed for clean fast commit.

Why it might be faster: same 1RTT and same quorum size, but lower order-statistic latency because the leader is not tied to one fixed reply set.

Proof obligations:

- Prove every admissible fast quorum satisfies the same fast-fast and fast-recovery intersections as the baseline.
- Show that early stopping on one quorum cannot ignore conflicting evidence that would have made the command unsafe.
- Specify how duplicate or late replies affect recovery state.

Risks:

- More messages may increase network load and worsen tail latency under saturation.
- If the admissible quorum family is too broad, intersection proofs may fail.
- This is an implementation-level latency win unless the quorum family itself is part of the protocol model.

TODO:

- Model quorum selection as nondeterministic choice among admissible fast quorums.
- Compare fixed quorum, nearest quorum, and hedged quorum under the same proof predicate.

### Candidate A5: Metadata-Compressed Fast Evidence

Status: `Idea`; faster only if metadata processing or reply size is a real bottleneck.

Motivation: dependency protocols may keep the same network RTT but pay high critical-path cost in dependency-set comparison, serialization, and merge logic. A command can be 1RTT in message delays yet slower in practice if fast replies carry large dependency metadata.

Source mechanisms: EPaxos `(seq, deps)`, SwiftPaxos dependency paths, EPaxos* `(cmd, dep)`, [[commit-rules]].

Design sketch: fast replies carry a compact dependency digest plus a bounded conflict summary. Full dependency evidence is stored locally and fetched only if the digest mismatches or recovery needs reconstruction. A clean fast commit requires matching digest evidence from the fast quorum and a deterministic rule mapping the digest back to the committed dependency object.

Why it might be faster: still 1RTT, same waited-for quorum, but less fast-path bandwidth and comparison work.

Proof obligations:

- Prove digest equality implies equality of the dependency object used by commit, or include a recovery rule for digest collision/ambiguity.
- Define when full dependency evidence must be materialized.
- Avoid relying on cryptographic hash assumptions in a pure safety proof unless they are modeled as explicit assumptions.

Risks:

- If digest equality is not logically tied to dependency equality, the proof becomes probabilistic or assumption-heavy.
- Full evidence fetch on mismatch could make slow path worse unless paired with Candidate B1.
- Compression may hide the modeling object that recovery actually needs.

TODO:

- Start with symbolic digests as injective functions in Rocq/TLA+.
- Later decide whether practical cryptographic hashes are acceptable engineering assumptions.

### Candidate A6: Read/Commutativity Leases For Local Fast Execution

Status: `Unclear`; can be faster than ordinary fast commit only for restricted operations.

Motivation: some operations commute or are read-only. If their conflict class has a lease-like guarantee that no conflicting writer can commit concurrently, a replica may answer locally or execute after a smaller evidence check.

Source mechanisms: [[conflict-handling]], [[linearizability]], Mencius commutativity-based out-of-order commit notes in [[commit-rules]].

Design sketch: maintain leases or epochs for read-only/commutative classes. Operations whose conflict predicate is empty within the current lease can complete locally or with a smaller confirmation path. Non-commutative or lease-expired operations use the baseline protocol.

Why it might be faster: for a restricted operation class, the path may avoid the ordinary dependency fast quorum.

Proof obligations:

- Define exactly which operations commute or are read-only.
- Prove leases cannot overlap with conflicting writes in a way that violates linearizability.
- State timing assumptions; lease safety may require clock or synchrony assumptions not used by baseline consensus safety.

Risks:

- This is not a general consensus fast path.
- Lease timing assumptions may weaken the model.
- Incorrect commutativity classification breaks safety.

TODO:

- Keep this as a separate restricted-operation track until the timing assumptions are explicit.

## Track B: Same 1RTT Fast Path, Faster Slow Path

These are the strongest current candidates. They preserve the baseline fast path and try to make mismatch, conflict, failure, or recovery cheaper.

### Candidate B1: Fast-Path-Neutral Evidence Split

Status: `Primary Idea`.

Motivation: dependency-based SMR often pays high recovery complexity because the fast path must leave enough evidence for later recovery. [[Pando]] suggests a separation between cheap discovery and stronger reconstruction, while [[EPaxosStar|EPaxos*]] shows that validated recovery can be the central safety mechanism.

Source mechanisms: [[Pando]] Phase 1a/Phase 1b split, [[EPaxosStar|EPaxos*]] validation, [[fast-paths]], [[recovery-rules]], [[quorum-systems]].

Fast-path rule: the normal path remains the baseline EPaxos*/EPaxos-style proposal plus matching fast replies. Certificate class, dependency digest, or conflict-witness summary may be piggybacked, but the command leader must not wait for a separate discovery quorum, reconstruction quorum, validator, or anchor before fast commit.

Slow-path improvement: recovery first asks a cheap classification question: clearly safe, clearly absent, or ambiguous. Only ambiguous cases gather full dependency evidence and run expensive validation.

Hypothesis: recovery can often avoid full EPaxos*-style validation while keeping the conflict-free fast path unchanged.

Proof obligations:

- Define the agreed object: likely `(cmd, dep, certClass)` or `(cmd, dep)` plus recoverable certificate evidence.
- Prove that any committed fast certificate is visible to every recovery path that might choose a conflicting command.
- Derive discovery/reconstruction quorum inequalities from [[quorum-intersection]], not by copying [[Pando]] storage quorums directly.
- Prove validation preserves EPaxos*-style visibility for conflicting committed commands.

TODO:

- Work out a minimal abstract model with certificate states `Absent`, `Possible`, `Recoverable`, and `Committed`.
- Derive candidate inequalities for `n`, `f`, `e`, discovery quorum size, and reconstruction quorum size.

### Candidate B2: SlowAck-Style Adoption For EPaxos*

Status: `Primary Idea`, potentially the cleanest slow-path latency win.

Motivation: [[SwiftPaxos]] can repair disagreement using SlowAck/Sync-style evidence. EPaxos* has correct validation-based recovery, but validation may be heavier than needed for simple mismatches.

Source mechanisms: SwiftPaxos SlowAck/Sync from [[fast-paths]] and [[recovery-rules]], EPaxos* validation from [[EPaxosStar|EPaxos*]].

Fast-path rule: unchanged EPaxos*/EPaxos fast path. Clean matching fast replies still commit in 1RTT.

Slow-path improvement: when fast replies disagree but contain a deterministically mergeable dependency object, replicas can adopt a canonical dependency proposal in one repair round instead of entering full recovery/validation. Full validation remains the fallback for non-mergeable, incomplete, or failed-leader cases.

Hypothesis: many conflicts are simple metadata disagreements, not true recovery ambiguity. A deterministic adoption rule may turn common slow-path cases into one additional round.

Proof obligations:

- Define the canonical merge/adopt function on dependency evidence.
- Prove adoption cannot remove a dependency needed for a conflicting committed command.
- Prove the adopted object is recoverable by the ordinary EPaxos*/EPaxos recovery path.
- State exactly which mismatch cases still require full validation.

Risks:

- Naive dependency union may increase execution latency or create larger SCCs.
- A merge rule can be safe but too conservative to be useful.
- Importing SwiftPaxos repair without dependency-path semantics may be unsound.

TODO:

- Compare agreed objects: EPaxos* `(cmd, dep)` versus SwiftPaxos dependency paths.
- Model two mismatching PreAccept replies and test whether canonical adoption preserves visibility.

### Candidate B3: Recovery-Only Conflict-Class Anchors

Status: `Primary Idea` if anchors are never contacted on the clean fast path.

Motivation: [[EPaxos]] distributes command leadership well, but recovery is subtle; [[SwiftPaxos]] simplifies fast evidence by requiring leader involvement. A middle ground may use conflict-class anchors only to structure recovery evidence, not to serialize normal commands.

Source mechanisms: EPaxos command leaders in [[leaderless-protocols]], SwiftPaxos leader-including fast quorums in [[leader-roles]], the fusion sketch in [[FastPaxos-EPaxos-SwiftPaxos]], and conflict classes in [[conflict-handling]].

Fast-path rule: the fast path cannot require an online anchor round trip or anchor signature. A command leader may include a deterministic conflict-class id or anchor id in existing messages, but fast commit still depends only on the baseline fast quorum and matching metadata.

Slow-path improvement: recovery uses the class anchor as an evidence locator, tie-breaker, or deterministic recovery coordinator when available. If the anchor is unavailable, the protocol falls back to ordinary recovery quorum evidence.

Hypothesis: anchor-indexed recovery could make conflicting-command recovery closer to SwiftPaxos without reintroducing a single fast-path leader for every command.

Proof obligations:

- Define conflict-class membership and show that all potentially interfering commands share at least one class or deterministic multi-class recovery rule.
- Prove that anchor-indexed evidence cannot hide a conflicting committed command from recovery.
- Prove agreement for the per-command dependency object and deterministic execution for committed dependency components.
- Specify fallback when the anchor crashes or a command spans multiple conflict classes.

TODO:

- Start with a static key-to-anchor function and single-key commands.
- Compare two agreed objects: EPaxos-style `(cmd, seq, deps)` versus SwiftPaxos-style dependency paths.

### Candidate B4: First-Class Abort / No-Op Recovery

Status: `Primary Idea`, useful for faster recovery and cleaner models.

Motivation: [[EPaxosStar|EPaxos*]] may recover `Nop` and resubmit, while [[Mencius]] uses owner-authored `SKIP` to fill gaps cheaply. Treating abort/no-op as a first-class recovery outcome may make dependency protocols easier to model.

Source mechanisms: EPaxos* `Nop` recovery, Mencius `SKIP`, [[recovery-rules]], [[recoverability]].

Fast-path rule: conflict-free commands still fast-commit normally. `Abort` and `Noop` are terminal recovery outcomes only after validation or owner-evidence rules show that preserving the original payload is unsafe, unavailable, or unnecessary.

Slow-path improvement: recovery can terminate ambiguous or abandoned command ids quickly instead of leaving half-visible ids that block dependency execution.

Hypothesis: explicit terminal outcomes may simplify recovery proofs and reduce execution blocking after failed command leaders.

Proof obligations:

- State validity carefully: an aborted id does not mean the client operation is chosen.
- Prove abort/no-op outcomes do not break visibility for commands that depended on the original id.
- Prove resubmission cannot duplicate a non-idempotent operation if the original command later becomes visible.

TODO:

- Compare no-op as an ordinary committed command versus no-op as a separate terminal state.
- Add Rocq lemmas for "terminal outcome uniqueness" and "aborted ids cannot later commit payload."

### Candidate B5: Background Execution Repair

Status: `Support Idea`; helps execution latency more than consensus slow-path latency.

Motivation: [[EPaxos-Revisited-2021]] shows that fast commit alone can be misleading if commands wait behind unresolved dependencies.

Source mechanisms: EPaxos commit/execution distinction in [[fast-paths]] and [[commit-rules]], SwiftPaxos acyclic dependency graph, EPaxos* SCC execution over committed dependencies.

Fast-path rule: do not make execution readiness a precondition for fast commit. Fast commit remains the baseline matching-dependency predicate.

Slow/execution improvement: replicas piggyback dependency-closure hints, known-committed dependency summaries, or bounded unresolved-dependency counters. If a command is committed but not executable, a background repair path resolves missing dependencies or commits no-op/abort outcomes for abandoned ids.

Hypothesis: treating execution readiness as background repair rather than fast-path gating can improve tail execution latency without sacrificing the conflict-free fast path.

Proof obligations:

- Keep safety predicate `CanCommit` separate from performance predicate `LikelyExecutableSoon`.
- Prove dependency repair is monotonic or specify how recovery handles non-monotonic readiness observations.
- Preserve base agreement/visibility even when repair hints are stale or absent.

TODO:

- Evaluate first as a policy layered over EPaxos* rather than as a new safety core.
- Add metrics that distinguish commit latency, execution latency, and repair latency.

### Candidate B6: Recovery Evidence Cache

Status: `Primary Idea` if cache entries are only hints until quorum-validated.

Motivation: slow paths often repeat the same expensive recovery evidence collection after nearby conflicts or a failed leader. Evidence that was already gathered for one recovery may help classify later recoveries in the same conflict neighborhood.

Source mechanisms: [[recovery-rules]], EPaxos* validation evidence, Pando Phase 1b reconstruction, [[recoverability]].

Fast-path rule: cache lookup must not be required before fast commit. The clean 1RTT path ignores the cache.

Slow-path improvement: recovery coordinators publish compact recovery certificates or negative evidence summaries. Later recoveries can reuse them to skip redundant validation or directly identify the relevant ambiguous commands.

Hypothesis: recovery can be amortized across bursts of conflicts or repeated recovery attempts.

Proof obligations:

- Define a recovery certificate format with epoch/ballot and conflict-class scope.
- Prove cached evidence is monotonic or expires safely.
- Prove stale cache entries cannot justify choosing a value that ordinary recovery would reject.

Risks:

- Caches can become unsound if they outlive the ballot/epoch assumptions that made them valid.
- Negative evidence is especially dangerous: "not seen" is not the same as impossible.
- More state may complicate Rocq models.

TODO:

- Start with positive certificates only; add negative summaries later if needed.

### Candidate B7: Two-Level Recovery: Local Repair Before Global Validation

Status: `Idea`, promising for localized conflicts.

Motivation: many mismatches may be local to a conflict class or region. Full global validation may be unnecessary when all possible conflicting commands are known to fall inside a smaller recovery scope.

Source mechanisms: conflict classes from [[conflict-handling]], EPaxos* validation, SwiftPaxos repair, [[quorum-intersection]].

Fast-path rule: unchanged. Scope selection is metadata piggybacked on existing fast replies.

Slow-path improvement: first run a class-local repair/validation step using evidence from replicas responsible for that conflict scope. Escalate to full recovery only if the local proof cannot establish safety.

Hypothesis: many slow paths can terminate after local repair, especially for sharded or key-range workloads.

Proof obligations:

- Prove every possible conflict for the command is covered by the selected local scope.
- Derive local quorum intersections with any fast quorum that could have committed a conflicting command.
- Define escalation conditions that are conservative and checkable.

Risks:

- Multi-key commands may force global validation.
- If conflict classification is dynamic or approximate, safety becomes fragile.
- Local repair can become another hidden leader/anchor if not kept off the fast path.

TODO:

- Start with static single-key conflict classes.
- Model multi-key commands as immediate escalation to global recovery.

### Candidate B8: Deterministic Dependency Pruning After Slow Adoption

Status: `Support Idea`; improves slow-path execution latency rather than commit latency.

Motivation: a safe slow-path merge may over-approximate dependencies, creating large SCCs and poor execution latency. [[EPaxos-Revisited-2021]] warns that commit latency and execution latency diverge under dependency chains.

Source mechanisms: EPaxos dependency sets, EPaxos* SCC execution, SwiftPaxos acyclic dependency graph, [[proof-techniques]] evaluation-sensitive invariants.

Fast-path rule: pruning is not part of clean fast commit.

Slow/execution improvement: after a slow-path adoption or recovery, run a deterministic pruning rule that removes dependencies proven unnecessary by conflict evidence. The committed object may include either the pruned dependency set or a proof that pruning preserves visibility.

Hypothesis: slow path can remain conservative for safety while a second deterministic rule recovers execution performance.

Proof obligations:

- Define "unnecessary dependency" using conflict evidence, not timing.
- Prove pruning preserves visibility for every conflicting committed command.
- Prove all replicas compute the same pruned dependency object.

Risks:

- Pruning can be unsound if it removes a dependency that is needed for a hidden committed command.
- If pruning needs another quorum round, it may not improve slow-path latency.
- Deterministic pruning may be too conservative to matter.

TODO:

- Model pruning as a pure function over a recovery certificate.
- Compare commit-safe dependencies with execute-minimal dependencies.

### Candidate B9: Recovery Coordinator Handoff With Proof-Carrying State

Status: `Idea`, targets failed or slow recovery coordinators.

Motivation: EPaxos* recovery has `Waiting` messages and depends on eventual stable recovery leadership. Slow path can stretch when recovery coordinators fail or restart evidence collection.

Source mechanisms: EPaxos* `Recover / Validate / Waiting`, [[recovery-rules]], [[proof-techniques]] recovery-sensitive invariants.

Fast-path rule: no effect on clean fast commit.

Slow-path improvement: a recovery coordinator hands off a proof-carrying partial recovery state: collected replies, validation obligations already discharged, remaining ambiguous commands, and a monotonic ballot/epoch marker. A successor can continue instead of restarting.

Hypothesis: recovery latency under coordinator churn can be reduced without changing safety predicates.

Proof obligations:

- Define which partial evidence is transferable across coordinators/ballots.
- Prove handoff state is monotonic and cannot skip required validation.
- Prove duplicate coordinators converge or one supersedes the other safely.

Risks:

- Partial evidence can be misinterpreted outside its ballot/epoch.
- Adds complexity to the recovery state machine.
- Does not help simple one-shot conflicts if coordinator churn is rare.

TODO:

- Represent recovery as a state machine with monotone obligations.
- Prove restart-from-proof refines restart-from-scratch.

### Candidate B10: Explicit Collision Classes For Fast Paxos-Style Recovery

Status: `Idea`, mostly useful as an abstraction bridge.

Motivation: [[FastPaxos]] collision recovery chooses a safe value after ambiguous fast-round votes. Dependency SMR protocols have richer collision objects, but some conflicts may be classified into simple collision classes with deterministic safe outcomes.

Source mechanisms: Fast Paxos pickable safe value, EPaxos/SwiftPaxos dependency metadata, [[adopt-commit-abstraction]], [[FastPaxos-EPaxos-SwiftPaxos]].

Fast-path rule: unchanged; collision classes are used only when fast evidence is mixed.

Slow-path improvement: if mixed evidence belongs to a known collision class with a deterministic safe merge/adopt result, use that result directly instead of full generic recovery.

Hypothesis: common conflict patterns can have specialized recovery rules that are easier and faster than the fully general recovery path.

Proof obligations:

- Define collision classes syntactically over evidence, not over hidden runtime intent.
- For each class, prove the specialized outcome refines the generic recovery-safe predicate.
- Prove unclassified evidence always falls back to generic recovery.

Risks:

- Too many specialized cases make the protocol harder to verify.
- Misclassification is catastrophic.
- The idea may collapse into Candidate B2 unless classes add real precision.

TODO:

- Start with one class: two commands that conflict only with each other and have complete dependency evidence.
- Add classes only when they shorten recovery compared with generic adoption.

## Ideas Currently Rejected For The Main Goal

These may still be useful as slow-path or recovery mechanisms, but they violate the current target if placed on the fast path.

| Idea shape | Why rejected | Possible salvage |
|---|---|---|
| Online conflict-class anchor signatures | Adds an anchor wait before every fast commit. | Use anchor id only as recovery metadata. |
| Pre-commit execution-readiness validation | Adds a dependency-closure check before fast commit. | Move readiness to background repair. |
| Larger discovery quorum before commit | Waits for more evidence than EPaxos*/EPaxos fast quorum. | Ask discovery questions only during recovery. |
| Owner-approved dependency fences | Serializes commands through class owners. | Prepublish fences or use them only after mismatch/failure. |
| Client-visible speculative execution before quorum evidence | May be faster than 1RTT, but semantics are not durable consensus commit. | Mark as application-level speculation, not a consensus fast path, unless the failure model is weakened explicitly. |

## Prioritized Next Work

1. Start with Track B, especially SlowAck-Style Adoption For EPaxos* and Fast-Path-Neutral Evidence Split, because they preserve the 1RTT fast path and target the slow-path pain directly.
2. In parallel, derive Track A quorum algebra for Latency-Tiered Fast Quorums and Fast-Quorum Hedges; promote them only if they give a real waited-reply or tail-latency improvement under a stated `f`/`e` target.
3. Treat Metadata-Compressed Fast Evidence and Dependency Pruning as performance candidates until the committed object and recovery evidence are fully formalized.
4. For every candidate, first prove fast-path equivalence to the chosen baseline, then add slow/recovery improvements.
5. Record separate metrics for fast commit latency, slow-path latency, recovery latency, and execution latency.

## Cross-Cutting Modeling Plan

For any promoted idea, start with a tiny abstraction before adding optimizations:

- State: command ids, payload ids, dependency sets or paths, evidence records, terminal outcomes, and quorum memberships.
- Messages: proposal, fast evidence, slow evidence, adoption, recovery request/reply, validation, commit, abort/no-op.
- Core invariants: per-id agreement, visibility for conflicting committed commands, recovery preservation of possible committed evidence, and execution determinism over dependency components.
- Liveness assumptions: keep synchrony, failure detectors, stable anchors/leaders, and quorum availability outside the safety invariant.
- First validation target: search small TLA+ configurations for counterexamples, then port stable invariants into Rocq/Coq.
