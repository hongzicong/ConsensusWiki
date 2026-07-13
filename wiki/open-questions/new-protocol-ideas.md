---
type: open-question
status: speculative
tags: [brainstorm, smr, consensus, protocol-design]
---

# New Protocol Ideas

These are defect-driven SMR and consensus candidates inferred from the current wiki. They are not claimed safe. Each item is an `Idea`, `Hypothesis`, or `Inference` until its quorum intersection, commit rule, and recovery rule are made explicit and checked against a model.

## Source Pages Reread

Core sources: [[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[commit-rules]], [[recovery-rules]], [[conflict-handling]], [[leader-roles]], [[proof-techniques]], [[timing-assumptions]], [[fault-models]], [[paxos-family]], [[fast-consensus]], [[leaderless-protocols]], [[FastPaxos-EPaxos-SwiftPaxos]], [[quorum-intersection]], [[adopt-commit-abstraction]], [[rocq-modeling-notes]], and [[unresolved-confusions]].

Main protocol anchors: [[FastPaxos]], [[GPaxos]], [[EPaxos]], [[EPaxosStar]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], and [[CURP]].

## Evidence Matrix

| Defect cluster | Wiki evidence | Causal bottleneck | Design dimension to perturb |
|---|---|---|---|
| Exact fast evidence is brittle | [[FastPaxos]] wants same value; [[EPaxos]] wants matching attributes; [[SwiftPaxos]] wants matching dependency-path evidence; [[Atlas]] relaxes matching through a recoverable union predicate | Because fast commit treats disagreement as unsafe, harmless observation differences trigger fallback under concurrency | Fast-path predicate, metadata, recovery evidence |
| Recovery is the real protocol | [[EPaxosStar]] centers validation; [[Atlas]] reconstructs remembered fast quorums; [[GPaxos]] uses `ProvedSafe`; [[FastPaxos]] needs safe higher-round value selection | Because early evidence may be partial, later leaders need a proof object that distinguishes committed from merely possible outcomes | Recovery rule, proof obligation, certificate shape |
| Commit can outrun execution | [[EPaxos-Revisited-2021]], [[Atlas]], and [[SwiftPaxos]] separate command commit from dependency execution readiness | Because dependency graphs can grow or form SCCs, low commit latency may hide tail execution latency | Dependency handling, pruning, execution certificate |
| Leaderless designs move cost into metadata | [[EPaxos]], [[EPaxosStar]], [[Atlas]], and [[Rabia]] avoid a stable leader but pay in dependency, validation, or randomized agreement machinery | Because no single sequencer observes all conflicts, replicas must encode conflict visibility in messages | Leader role, conflict-class ownership, metadata |
| Strong leaders bottleneck but simplify recovery | [[PigPaxos]] keeps Paxos safety and optimizes communication; [[CURP]] keeps a master and uses witnesses; [[SwiftPaxos]] includes the ballot leader in fast quorums | Because central ordering simplifies safe recovery, removing the leader often recreates a hidden anchor elsewhere | Leader placement, relay layer, witness layer |
| All-witness or all-fast paths are fragile | [[CURP]] primary-backup fast path needs all `f` witnesses; [[EPaxosStar]] fast quorum `n - e` can be large; [[Atlas]] tunes around small `f` | Because fast recovery wants complete or reconstructible evidence, availability budgets drive quorum size | Quorum shape, witness selection, flexible budgets |
| Randomized fallback trades density for simplicity | [[Rabia]] can decide `bottom` slots and retry requests | Because ambiguous proposal alignment is resolved by forfeiting a slot, the protocol avoids separate leader recovery at the cost of sparse logs | Commit rule, validity, retry policy |
| Transport overlays do not change safety by themselves | [[PigPaxos]] relays aggregate unique votes but do not alter quorums | Because communication optimization preserves the old proof, it cannot fix leader placement or fast-path conflict defects alone | Overlay, aggregation, proof refinement |
| Erasure-coded recovery separates identity from fragments | [[Pando]] distinguishes value identity, coded split availability, and Phase 1a/1b/2 quorums | Because cheap discovery is weaker than full recovery, read/write paths need different evidence shapes | Storage evidence, read path, split quorum |
| Dynamic structure lacks an epoch proof | [[CURP]] exposes stale witness-list risk; [[Mencius]] revokes future owner slots; adaptive quorum and movable-anchor ideas would change evidence membership | Because old and new configurations may each produce locally valid evidence, reconfiguration can create two incomparable recovery histories unless certificates cross an explicit epoch boundary | Reconfiguration, epoch fencing, garbage collection |
| Missing families are negative space | [[bft-consensus]] and [[dag-based-consensus]] are intentionally unpopulated | Because no sourced BFT or DAG protocol is ingested, ideas in those directions need future sources before being promoted | Fault model, DAG ordering, source coverage |

## Defect Clusters

### A. Fast Evidence That Is Recoverable But Not Identical

Because protocols often require identical early evidence, they fall back under benign timing skew or concurrent non-conflicting observations.

1. **Recoverable-Union EPaxos** - Idea: replace exact EPaxos fast-path dependency matching with an [[Atlas]]-style predicate saying every dependency in the final union appears in enough fast replies to be reconstructed. Proof hook: derive the recovery quorum intersection and then add [[EPaxosStar]]-style visibility validation.
2. **Threshold-Path SwiftPaxos** - Idea: let [[SwiftPaxos]] fast-commit when each dependency path segment is reported by a threshold of leader-including fast-quorum members, not only when all paths match exactly. Risk: acyclicity may fail unless path thresholds compose.
3. **Witnessed Fast Paxos Collisions** - Idea: add lightweight conflict witnesses to [[FastPaxos]] fast rounds so recovery can distinguish one dominant fast value from a symmetric collision. Proof hook: define a pickable predicate that uses witness counts without weakening triple intersection.
4. **C-Struct Union Fast SMR** - Idea: use [[GPaxos]] c-struct compatibility as the fast evidence for SMR commands, but attach an explicit recovery certificate for the lub learned by clients. Risk: checkpointing and garbage collection may be harder than dependency sets.
5. **Frequency-Capped Dependency Union** - Idea: cap dependency unions to dependencies that appear in at least `k` replies and send rare dependencies through slow validation. Evaluation hook: measure whether this reduces dependency-chain tails from [[EPaxos-Revisited-2021]].
6. **Recoverable Non-Matching FastAck** - Idea: extend [[SwiftPaxos]] `FastAck` so non-matching follower proposals are acceptable when they can be deterministically repaired to the leader proposal from quorum evidence. Proof hook: show no repaired value can hide a previously fast-committed different dependency path.
7. **Two-Level Fast Evidence** - Idea: commit on a small exact core plus a recoverable optional dependency fringe. The core gives agreement; the fringe only affects execution order after validation.
8. **Dependency Bloom Certificate** - Hypothesis: use compact probabilistic summaries only as a prefilter for dependency disagreement, with exact evidence required before commit. Risk: false positives may hurt liveness; false negatives must be impossible or excluded by construction.
9. **Quorum-Remembered Fast Sets** - Idea: require command coordinators to publish the identity of their fast quorum early, as in [[Atlas]], so later recovery knows which intersection to reconstruct from. Cost: larger metadata and possible quorum-selection gaming.
10. **Fast Evidence Normalizer** - Idea: define a deterministic normalization function over dependency replies before checking equality. Proof hook: prove normalization preserves all conflicts that visibility requires.

### B. Recovery-First Protocol Shapes

Because ambiguous recovery is a recurring failure mode, design the recovery predicate first and derive the fast path from it.

11. **Recovery-First EPaxos Variant** - Idea: start from [[EPaxosStar]] validation and allow only fast commits whose evidence would pass validation if replayed later. This makes fast path an optimization of recovery, not a separate proof case.
12. **Safe-Value DSL Paxos** - Idea: specify Fast Paxos, [[GPaxos]], and dependency SMR recovery through a small language of safe-value predicates. Proof hook: reuse [[adopt-commit-abstraction]] while keeping protocol-specific metadata explicit.
13. **Validation Cache SMR** - Idea: replicas cache negative validation facts, such as "no known conflicting command can have committed without edge X", to accelerate future recoveries. Risk: cache invalidation across ballots.
14. **Recovery Witness Quorum** - Idea: add a small class of non-voting recovery witnesses that store fast-path metadata but do not participate in normal commit. Similar caution as [[CURP]] witnesses: recovery must select evidence consistently.
15. **NoOp-First Recovery** - Idea: recover uncertain command identifiers as `noOp` aggressively, as in [[EPaxosStar]], while requiring clients to resubmit payloads with stable request ids. Evaluation hook: compare extra resubmissions against simpler proof obligations.
16. **Recoverable Ballot Anchors** - Idea: add a per-conflict-class ballot anchor only for recovery, not normal fast commit. Risk: if the anchor is too central, it becomes a hidden leader.
17. **Evidence Lattice Recovery** - Idea: represent recovery evidence as a lattice: accepted proposal > validated fast proposal > reconstructible no-op. Proof hook: prove monotonic selection prevents incompatible lower evidence.
18. **Anti-Ambiguity Prepare** - Idea: add a prepare subphase that asks replicas to report not only accepted state but also "commands I would need validated before accepting this recovery". Cost: more recovery messages, likely acceptable if rare.
19. **Self-Describing Fast Certificates** - Idea: each fast certificate includes enough quorum geometry to recompute its own recovery intersections. Useful for flexible quorums and dynamic relay groups.
20. **Dual-Recovery-Certificate SMR** - Idea: a fast commit carries both a value/command certificate and a separate recovery-hint certificate naming the evidence a later coordinator must reconstruct. Proof hook: show neither certificate is sufficient alone and every recovery quorum intersects the hint's evidence set strongly enough to preserve the committed value.

### C. Commit-Execution Gap Reduction

Because dependency protocols can commit quickly but execute late, target execution readiness directly.

21. **Execution-Ready Fast Commit** - Idea: fast commit only when the dependency set is both agreed and already committed or jointly certifiable as an SCC. Evaluation hook: compare median commit loss against tail execution gain.
22. **Bounded-Depth Dependency SMR** - Idea: allow fast commit only if the transitive dependency depth stays below a configured bound; otherwise fall back to a leader/anchor that linearizes. Risk: liveness under hot-key workloads.
23. **Prunable Dependency Certificates** - Idea: attach a proof that dependencies older than a frontier cannot affect execution order, inspired by the pruning concerns in [[EPaxos-Revisited-2021]]. Proof hook: formalize the frontier invariant.
24. **SCC-First Commit Rule** - Idea: commit batches of mutually dependent commands as the consensus object rather than committing commands separately and discovering SCCs later. Cost: larger consensus values under conflict.
25. **Dependency Lease Horizon** - Hypothesis: use timing only for liveness/performance, never safety, to bias dependency collection toward a short horizon in stable periods. Caveat: any lease claim must be outside the safety proof, per [[timing-assumptions]].
26. **Conflict-Class Serializers** - Idea: route commands that touch the same hot conflict class through a temporary serializer while keeping cold commands leaderless. This targets EPaxos tail latency without globally reintroducing a leader.
27. **Execution-Aware Fast Quorum** - Idea: choose fast quorum members based on who already knows the likely dependencies, not merely geography. Risk: quorum adaptivity complicates intersection proofs.
28. **Dependency Debt Accounting** - Idea: expose a per-command "execution debt" metric and force slow path once debt exceeds a threshold. This is an adaptive performance mechanism with unchanged safety.
29. **Frontier-Stamped Dependencies** - Idea: every dependency proposal includes a local committed frontier; recovery prefers proposals whose frontiers make execution closer. Proof hook: frontier cannot be used to drop required conflicts.
30. **Hot-Key Bottom Slots** - Idea: combine [[Rabia]] slot forfeiture with dependency SMR: under unresolved hot-key contention, decide a `bottom` placeholder and retry, instead of committing long dependency chains.

### D. Leader Role and Anchor Placement

Because leaders simplify recovery but bottleneck throughput, use narrower or movable anchors.

31. **Per-Key Swift Anchors** - Idea: replace one [[SwiftPaxos]] ballot leader with per-key-range leaders that must be included only in fast quorums for their conflict class. Risk: multi-key commands need multiple anchors or a deterministic tie-break.
32. **Rotating Dependency Anchors** - Idea: use [[Mencius]]-style rotation for dependency anchors rather than log slots. Each command's conflict class determines the anchor for validation and slow repair.
33. **Leaderless Fast, Leadered Repair** - Idea: keep [[Atlas]] or [[EPaxos]] fast path fully leaderless, but require all fallback repair to use a stable leader-including quorum. Proof hook: show leadered repair preserves possible fast decisions.
34. **Client-Chosen Anchor With Proof** - Idea: clients select a nearby anchor and include its signed or durable proposal evidence. In crash-only models, the "signature" can be a stored message record; Byzantine assumptions are not inferred.
35. **Relay-Elected Temporary Leader** - Idea: piggyback a temporary slow-path leader election on [[PigPaxos]] relay groups when the stable leader is overloaded. Risk: this changes consensus semantics and needs a real Paxos election proof.
36. **Anchorless Read-Only Fast Path** - Idea: borrow [[SwiftPaxos]] distributed read-only execution but allow read-only commands to use any fast-quorum replica when the dependency certificate is leader-independent. Proof hook: linearizability for reads.
37. **Mencius With Dependency Escape Hatch** - Idea: keep owner-assigned slots, but let non-owners attach dependency certificates for commutable out-of-order execution rather than waiting for gaps. Risk: owner `SKIP` and dependency evidence must remain separate.
38. **Failure-Responsive Anchors** - Idea: shrink or move anchors when the current anchor is suspected, while retaining old-anchor evidence in recovery certificates. Liveness hook: eventual stable anchor per conflict class.
39. **Conflict-Class Omega** - Idea: replace per-command recovery leader detectors with per-conflict-class detectors, reducing recovery churn for hot classes. Risk: cold commands should not wait behind hot-class recovery.
40. **Bottleneck-Aware Ballot Layout** - Idea: assign ballot leadership by measured load and conflict graph, not by replica id. Proof hook: ballot ownership is performance metadata; safe voting rules must remain ballot-based.

### E. Quorum Geometry and Flexible Budgets

Because quorum size determines both latency and recoverability, vary fast, slow, and recovery budgets independently.

41. **Adaptive `e`-Fast SMR** - Idea: expose [[EPaxosStar]]'s `e` as an operational knob that changes fast quorum size by epoch. Proof hook: epoch certificates must prevent mixing incompatible `e` values.
42. **Atlas Failure-Budget Scheduler** - Idea: adjust [[Atlas]] `f` per epoch based on expected concurrent site outages. Risk: safety must be parameterized by the strongest recovery obligation across epochs.
43. **Flexible Dependency Paxos** - Idea: combine small slow quorums with large recovery quorums for dependency SMR, deriving formulas before implementation as [[quorum-systems]] recommends. Proof hook: intersection algebra table first.
44. **Region-Weighted Fast Quorums** - Idea: choose fast quorums with regional weights so nearby sites can form low-latency evidence while preserving recovery intersection. Risk: weighted quorum proofs are easy to get subtly wrong.
45. **Dual Fast Quorum Classes** - Idea: offer a small fast quorum for read-only or commutative commands and a larger fast quorum for arbitrary updates. Recovery must know which class was used.
46. **Leader-Including Small Fast Quorum** - Idea: explore [[SwiftPaxos]] C2-style fixed majority fast quorums for a subset of commands whose leader is co-located with hot clients. Risk: fixed quorum hurts availability and load balance.
47. **Fast-Quorum Market** - Hypothesis: let replicas advertise latency and load, and pick quorums satisfying a certified intersection policy. Protocol value: separates optimization from safety predicate.
48. **Slow-Path-First Flexible Design** - Idea: choose the smallest safe slow quorum first, then derive the fast predicate that remains recoverable from `n - f`. This reverses the usual "make fast path pretty" approach.
49. **Read-Discovery vs Recovery Split for SMR** - Idea: adapt [[Pando]]'s distinction between Phase 1a discovery and Phase 1b recovery to SMR reads: cheap read discovery can observe candidates, but only stronger evidence can finalize.
50. **Quorum-Versioned Certificates** - Idea: every commit certificate carries the quorum-system version used to form it. Necessary if adaptive or flexible quorum policies are allowed.

### F. Witness and Unordered Durability Ideas

Because [[CURP]] shows unordered durability can buy 1 RTT but is availability-sensitive, generalize witnesses carefully.

51. **Majority Witness CURP** - Idea: replace all-witness fast durability with a witness quorum plus stronger recovery selection. Proof hook: show any completed unsynced operation appears in the selected recovery witness set, not just one witness.
52. **Dependency Witnesses** - Idea: witnesses store unordered dependency observations rather than full requests. Recovery uses them to validate conflict visibility for [[EPaxosStar]]-style recovery.
53. **Witnessed Atlas Fast Quorum** - Idea: add cheap witnesses that remember the initial fast quorum and dependency union, reducing recovery dependence on failed coordinators. Risk: witnesses become part of the recovery fault model.
54. **Commutativity Witness for Paxos Leaders** - Idea: a strong leader can reply before majority accept only when witnesses certify the unsynced suffix is mutually commutative, as in [[CURP]]. Proof hook: integrate with Paxos leader change.
55. **Witness Garbage-Collection Frontier** - Idea: witnesses keep records until a quorum-certified execution frontier passes them. This targets CURP's reconfiguration and zombie-client TODOs.
56. **Selected-Witness SMR Recovery** - Idea: recover from one selected witness per conflict class instead of merging witness records. This borrows CURP's "do not union witnesses" warning.
57. **Witnessed Bottom Avoidance** - Idea: in [[Rabia]]-like protocols, witnesses store proposal alignment hints to reduce `bottom` slots without changing binary agreement safety.
58. **Witness-Backed Read-Only Reply** - Idea: read-only commands execute speculatively at nearby replicas and use witnesses to prove no unsynced conflicting update is hidden. Risk: read linearizability proof may dominate the design.
59. **Elastic Witness Placement** - Idea: move witnesses to low-latency failure domains while keeping the master/backup quorum fixed. Open question: co-hosted vs separate crash domains from [[CURP]].
60. **Witness Capacity Slowdown** - Idea: treat witness rejection due to capacity or non-commutativity as an explicit signal to switch a conflict class into slow ordered mode.

### G. Randomization and Slot Forfeiture

Because [[Rabia]] avoids leader recovery by allowing weak validity, explore bounded uses of randomness in Paxos-like SMR.

61. **Randomized Dependency Repair** - Idea: when dependency recovery has multiple safe candidates, use a common coin to select one rather than waiting for a leader preference. Proof hook: all candidates must already be safe.
62. **Selective Bottom for Hot Conflicts** - Idea: only commands in high-contention classes may produce `bottom` slots; cold commands use deterministic fast paths. Evaluation hook: log density under skew.
63. **Coin-Driven Anchor Rotation** - Idea: use common coin outcomes to rotate slow-path anchors after repeated conflict or timeout. Safety remains ordinary quorum safety; randomness is only liveness/load balancing.
64. **Weak-MVC Per Conflict Class** - Idea: run [[Rabia]]-style weak multi-valued consensus only for the next command in a hot conflict class, while other classes use dependency fast paths. Risk: stitching class orders into global SMR order.
65. **Bottom-as-Dependency-Barrier** - Idea: a `bottom` slot can serve as an execution barrier that lets replicas prune older uncertain dependencies. Proof hook: show barriers are globally agreed and do not skip required commands.
66. **Probabilistic Relay Selection for Leaderless Quorums** - Idea: borrow [[PigPaxos]] random relay rotation for leaderless fast quorum collection to avoid repeatedly slow replicas. Safety unchanged if collected evidence is explicit.
67. **Randomized Recovery Backoff** - Idea: reduce recovery storms by randomized per-command recovery retries. This is a liveness optimization, not a safety rule.
68. **Coin-Selected Safe NoOp** - Idea: if recovery validation cannot distinguish payloads, decide between safe `noOp` and waiting using a common coin. Risk: weak validity and client semantics need explicit treatment.
69. **Sparse-Log Compactor** - Idea: design log compaction first for protocols with `bottom` or `noOp` recovery, then make commit produce compaction-friendly evidence. Source motivation: [[Rabia]] open question.
70. **Randomized Fast-Quorum Sampling** - Hypothesis: sample candidate fast quorums randomly but commit only after a deterministic quorum predicate is met. Evaluation hook: latency/tail reduction under partial failures.

### H. Transport, Relay, and Aggregation Beyond PigPaxos

Because transport overlays help throughput but not safety, use them to carry richer evidence without pretending they are votes.

71. **Dependency Relay Aggregation** - Idea: relays aggregate dependency proposals with unique replica ids for [[EPaxos]] or [[Atlas]], reducing coordinator load while preserving evidence sets. Risk: aggregation must not hide dissent needed for fallback.
72. **FastAck Relay Trees** - Idea: relay [[SwiftPaxos]] `FastAck` paths through regional trees and preserve exact signer/member ids. Evaluation hook: reduce quadratic message load.
73. **Recovery Evidence CDN** - Idea: cache immutable commit and recovery certificates at relay nodes so recovering coordinators fetch evidence nearby. Safety hook: relays only serve evidence, never vote.
74. **Graylisted Dependency Relays** - Idea: extend [[PigPaxos]] gray lists to avoid relays that often return partial or stale dependency aggregates. Liveness only.
75. **Relay-Visible Conflict Metrics** - Idea: use relay aggregates to estimate conflict classes and switch them between leaderless and anchored modes. No safety dependence on estimates.
76. **Overlapping Relay Groups for Quorums** - Idea: form relay groups so any global quorum can be reconstructed from group partials. Proof hook: leader counts unique voter ids, per [[PigPaxos]] modeling note.
77. **Relay-Assisted Fast Recovery** - Idea: relays remember which replicas were contacted in a fast quorum, helping later recovery collect the right intersection. Risk: relay memory cannot be required for safety unless modeled as durable.
78. **Two-Hop Leader Fast Path** - Idea: accept an added relay hop only for large clusters where leader CPU is the bottleneck; keep small clusters direct. Evaluation candidate rather than new safety mechanism.
79. **Relay-Filtered Duplicate Requests** - Idea: relays suppress duplicate client requests before they reach leaders or command coordinators, reducing dependency noise. Need exactly-once ids as in [[CURP]].
80. **Regional Aggregated Reads** - Idea: aggregate read-only dependency evidence regionally for [[SwiftPaxos]]-style speculative reads. Proof hook: client acceptance must still see a valid quorum certificate.

### I. Erasure-Coded and Storage-Inspired SMR

Because [[Pando]] separates value identity from fragment recovery, apply that separation to logs and certificates.

81. **Erasure-Coded Commit Certificates** - Idea: store large dependency or path certificates as coded fragments while keeping a small value identity in consensus. Recovery reconstructs enough fragments before selecting a value.
82. **Split Dependency Logs** - Idea: separate command payload, dependency metadata, and execution result into independently recoverable coded objects. Risk: agreement on identity is easier than ensuring all pieces are live.
83. **Phase-1a Read for SMR State** - Idea: use a cheap discovery quorum to find a likely latest executed prefix, then require stronger recovery/write-back before serving a linearizable read.
84. **Coded Witness Records** - Idea: witnesses store erasure-coded request records so recovery can tolerate unavailable witnesses without all-witness fast-path fragility. Proof hook: selected reconstruction set must contain every completed unsynced request.
85. **Pando-Style Dependency Recovery** - Idea: recovery for dependency SMR first recovers dependency-certificate identity, then reconstructs enough replicas' dependency fragments to validate it.
86. **Payload-Late Consensus** - Idea: consensus agrees on command id and dependency certificate first; payload is fetched or reconstructed later from coded storage. Risk: validity and client retry semantics.
87. **Coded FastAck Paths** - Idea: compress [[SwiftPaxos]] dependency paths with erasure coding for archival/recovery, while normal fast commit uses explicit matching evidence.
88. **Quorum Reads Over Dependency Frontiers** - Idea: adapt Pando's read/write-back pattern to dependency frontiers: reads discover, repair, then return from a reconstructed frontier.
89. **Storage-Aware SMR Failover** - Idea: use storage split availability to choose recovery coordinators with nearby evidence, improving tail recovery latency without changing safe selection.
90. **Single-Key Storage to Multi-Key SMR Bridge** - Hypothesis: start with [[Pando]]-style per-key chosen values and add a dependency layer only for multi-key commands. Risk: atomicity across keys becomes the central proof.

### J. Reconfiguration, Epochs, and Evidence Retirement

Because the wiki's protocols rely on configuration-specific quorums, owners, witnesses, or relays, changing those structures needs a protocol-level handoff rather than a local configuration update.

91. **Epoch-Fenced Adaptive Quorums** - Idea: allow fast, slow, and recovery quorum policies to change only after a joint certificate intersects every quorum class in both adjacent epochs. Recovery first selects the highest certified epoch, then applies that epoch's safe-value rule. Proof hook: exclude mixed-epoch fast certificates.
92. **Witness-List Handoff Consensus** - Idea: turn [[CURP]] witness-list changes into an explicit two-epoch handoff: old witnesses freeze and summarize unsynced requests, backups durably absorb that summary, and only then may clients use the new `WitnessListVersion`. Risk: zombie clients must be rejected without losing their completed requests.
93. **Conflict-Class Split/Merge SMR** - Idea: dynamically split a hot conflict class into sub-classes or merge overlapping classes, with a barrier command that certifies all pre-change cross-class dependencies. Proof hook: multi-key commands spanning the barrier must have one unambiguous execution position.
94. **Frontier-Certified Membership Change** - Idea: admit or remove replicas only at an execution frontier whose certificate proves every earlier committed command is reconstructible in the new configuration. This makes execution state, not merely accepted ballots, the reconfiguration boundary.
95. **Dependency-Aware Joint Consensus** - Idea: during membership change, require old/new joint evidence only for commands whose dependency closure crosses the reconfiguration frontier; independent post-frontier commands may use the new configuration immediately. Risk: deciding whether a closure crosses the frontier cannot rely on incomplete local metadata.
96. **Relay-Group Reconfiguration SMR** - Idea: reconfigure [[PigPaxos]]-style relay groups through a ballot-certified group map while retaining unique voter identities end to end. Old and new relays may forward concurrently, but the leader counts each replica once under the certified map. Proof hook: refinement to the unchanged base quorum.
97. **Rotating-Owner Gap Containment** - Idea: modify [[Mencius]] so an owner failure revokes only a certified finite window of future slots; ownership beyond that window transfers by epoch instead of leaving an unbounded revocation tail. Risk: the old owner must be fenced from issuing late `SKIP` messages.
98. **Request-ID Epoch Recovery** - Idea: bind stable client request ids to configuration epochs, then let recovery translate an unfinished old-epoch id into either the preserved command or an explicit retry token. This combines [[CURP]] duplicate suppression with [[EPaxosStar]] `Nop`/resubmission semantics without silently executing twice.
99. **Certificate-Compacting Checkpoints** - Idea: checkpoint only after a quorum certifies both an executed state digest and a recovery frontier covering every still-possible fast decision. Replicas may then discard dependency paths, remembered fast quorums, and witness records below the frontier. Proof hook: no later recovery may require retired evidence.
100. **Recovery-First Reconfigurable SMR** - Idea: define membership change as a special recovery ballot that gathers old-configuration evidence, chooses a safe command/dependency frontier, and installs the new configuration with that frontier as its initial state. Normal reconfiguration is therefore an instance of the protocol's safe-value rule, not a separate mechanism.

## Ranked Shortlist

| Rank | Candidate | Why it survives first red-team pass | Next artifact |
|---|---|---|---|
| 1 | Recoverable-Union EPaxos | Targets exact-matching brittleness using an already sourced [[Atlas]] predicate shape, while acknowledging [[EPaxosStar]] validation obligations | Quorum/recovery model for dependency union plus visibility |
| 2 | Per-Key Swift Anchors | Narrows [[SwiftPaxos]] leader inclusion to conflict classes, attacking leader bottlenecks without removing anchors entirely | Conflict-class quorum formulas and multi-key counterexamples |
| 3 | Execution-Ready Fast Commit | Directly targets the commit/execution gap from [[EPaxos-Revisited-2021]] | Simulator comparing commit latency and execution latency |
| 4 | Recovery-First EPaxos Variant | Treats recovery validation as the source of truth, reducing fast/recovery mismatch | Small Rocq/TLA-style validation abstraction |
| 5 | Majority Witness CURP | Attacks all-witness fragility, but has a crisp recovery-selection proof obligation | Witness quorum intersection derivation |
| 6 | Dependency Relay Aggregation | Likely practical: changes transport load while preserving explicit evidence | Refinement proof to a non-relayed dependency protocol |
| 7 | Adaptive `e`-Fast SMR | Uses sourced [[EPaxosStar]] parameters as an operational knob | Epoch-change safety proof |
| 8 | SCC-First Commit Rule | Targets execution tails by changing the agreed object | SCC agreement and recovery model |
| 9 | Weak-MVC Per Conflict Class | Uses [[Rabia]] only where deterministic ordering is suffering | Global order stitching proof |
| 10 | Pando-Style Dependency Recovery | Separates identity from reconstructability for large certificates | Fragment identity/recovery proof |

## Red-Team Notes

- Any candidate that relaxes exact matching must define exactly what later recovery can reconstruct.
- Any candidate with adaptive quorums must put the quorum-system version inside the certificate.
- Any candidate using timing, leases, or measured load must keep those facts out of the safety theorem unless a sourced protocol justifies them.
- Any candidate using witnesses must decide whether witnesses are separate crash domains, co-hosted components, or merely durable evidence stores.
- Any candidate using `bottom` must define weak validity, retry semantics, and log compaction before claiming it simplifies recovery.
- Any relay candidate must preserve unique voter identities and prove relays are transport refinements, not quorum members.
- Any reconfiguration candidate must fence old participants, state which old/new quorum classes intersect, and define when recovery evidence may be retired.

## Next Proof and Evaluation Work

1. Derive a standalone quorum algebra table for [[FastPaxos]], [[EPaxosStar]], [[Atlas]], and the top three candidates.
2. Build a minimal model for Candidate 1 with command ids, dependency sets, fast quorum evidence, recovery quorum evidence, and visibility.
3. Add a workload simulator that reports both commit latency and execution latency for dependency protocols.
4. Create a recovery counterexample harness that searches for two recoveries selecting incompatible dependency sets.

## Focused Latency Optimization Roadmap

This roadmap responds to the four stable-run classes in [[latency]]. It does not claim that the candidates are safe or novel; each remains an `Idea`, `Inference`, or `Hypothesis` until its commit predicate, quorum intersections, and recovery rule are modeled.

### Boundary of the opportunity

- **Sourced fact:** [[SwiftPaxos]] reports `2δ` client-response latency through contention-free runs and `3δ` in a stable general run.
- **Inference:** under a metric where the client sends a command and then waits for quorum-backed response evidence, `2δ` is a natural information-flow floor, not a proved universal lower bound.
- **Hypothesis:** reducing stable general latency from `3δ` to `2δ` requires moving conflict ordering before the divergent observations, adding an ordering/timing premise, or changing what counts as a completed response. Merely shrinking a quorum does not reconcile different receive orders.

### Ranked opportunities

| Rank | Direction | Expected latency effect | Changed dimension | Main proof obligation |
|---|---|---|---|---|
| 1 | Recoverable non-matching fast evidence | Converts some receive-order disagreements from `3δ` fallback to `2δ` without claiming all general runs | Fast predicate and recovery certificate | Recovery reconstructs one conflict-visible, acyclic dependency result from threshold/normalized evidence |
| 2 | Epoch-fenced conflict-class anchors | Bounds hot-item dependency and recovery tails while retaining leaderless cold paths | Leader role per conflict class | Cross-anchor and multi-key commands have one order; old/new anchor certificates cannot both commit incompatibly |
| 3 | Execution-ready or SCC-first commit | Reduces commit-to-execute tail even if message-delay commit count is unchanged | Agreed object and commit rule | All replicas agree on the same executable batch/frontier and recovery preserves it |
| 4 | Versioned adaptive quorum profiles | Keeps the fast path available across changing failures and WAN conditions | Quorum geometry and epoch metadata | Every certificate identifies its quorum version; recovery intersects all still-live certificate classes |
| 5 | Witness-quorum durability | Preserves one-RTT commutative completion despite some witness loss | Durability and recovery selection | Every completed unsynced operation is present in the uniquely selected recovery evidence |
| 6 | Regional placement, relays, and direct replies | Lowers physical `δ`, client `+1`, and tail load without changing logical rounds | Transport and quorum placement | Relays preserve voter identities; client evidence still represents a valid quorum |

### Top candidate: recoverable non-matching evidence

**Idea:** keep the two-hop client path, but replace all-or-nothing equality of dependency/path replies with a certificate split into an exact ordering core and a recoverable fringe. The core contains conflict edges needed for agreement; the fringe contains additional observations that may be deterministically validated or repaired before execution.

Required next artifacts:

1. Define the certificate object, including direct dependencies, path/acyclicity evidence, reporter identities, and quorum version.
2. Derive fast/recovery intersection inequalities before selecting thresholds.
3. Search for a counterexample where two quorums validate different cores or where individually valid path fragments form a cycle after union.
4. Compare four metrics in simulation: client response, local commit, execution-ready time, and fast-path survival after one unavailable replica.

### Practical low-risk optimization

**Idea:** hedge fast and compatible slow evidence collection in parallel, then use the first valid certificate. This may remove timeout amplification and reduce tail latency without changing the commit predicate, but increases messages. It is safe only if both evidence streams vote for compatible state in the same ballot; otherwise it becomes a protocol change rather than a transport optimization.
5. Revisit this page after ingesting one BFT paper and one DAG-based consensus paper; until then, BFT and DAG claims remain intentionally out of scope.
