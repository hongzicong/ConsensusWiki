---
type: open-question
status: speculative
tags: [brainstorm, smr, consensus, protocol-design]
---

# New Protocol Ideas

These 100 defect-driven candidates are inferred from the current wiki. None is claimed safe or novel. Each remains an `Idea`, `Hypothesis`, or `TODO` until its commit predicate, quorum intersections, recovery rule, and liveness assumptions are explicit and checked against a model and the primary literature.

## Source Pages Reread

Core synthesis: [[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[commit-rules]], [[recovery-rules]], [[conflict-handling]], [[leader-roles]], [[proof-techniques]], [[fault-models]], [[timing-assumptions]], [[latency]], [[reconfiguration]], [[paxos-family]], [[fast-consensus]], [[leaderless-protocols]], [[quorum-intersection]], [[adopt-commit-abstraction]], [[rocq-modeling-notes]], and [[unresolved-confusions]].

Protocol anchors: [[FastPaxos]], [[GPaxos]], [[EPaxos]], [[EPaxosStar]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[CURP]], [[Copilot]], [[Avicenna]], [[Bodega]], [[Jetpack]], [[FPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], and [[WPaxos]].

The refreshed set particularly incorporates nine newer wiki anchors absent from the original brainstorm: [[Copilot]], [[Avicenna]], [[Bodega]], [[Jetpack]], [[FPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], and [[WPaxos]].

## Evidence Matrix

| Defect cluster | Wiki evidence | Causal bottleneck | Dimension to change |
|---|---|---|---|
| Early evidence is easier to create than to recover | [[EPaxosStar]] validates possible fast decisions; [[Atlas]] reconstructs dependency unions; [[Copilot]] inspects another log when vote counts are ambiguous; [[Jetpack]] commits a stability marker | Because early completion admits partial or non-identical evidence, recovery needs more structure than the fast commit predicate appears to expose | Commit rule, recovery certificate, evidence order |
| Special roles become veto points | [[CURP]] needs all witnesses; [[Jetpack]] needs all host proposers; [[Bodega]] writes cover all responders; [[SwiftPaxos]] includes a leader | Because an optimization grants a role unique safety relevance, one slow role member can disable the optimized path or delay writes | Quorum shape, role membership, failure budget |
| Fail-slow tolerance duplicates work | [[Copilot]] orders every command twice; [[Avicenna]] shadow-processes a sampled counterfactual path | Because crash-only protocols cannot distinguish useful slowness evidence from noise, proactive redundancy buys latency at CPU, bandwidth, and recovery complexity cost | Redundancy granularity, detection plane, takeover scope |
| Quorum geometry ignores connectivity state | [[FPaxos]] exposes phase-specific availability; [[OmniPaxos]] elects a quorum-connected server; [[PigPaxos]] changes transport without changing votes | Because static quorums and leader rules do not encode current link topology, a live system may choose slow or unreachable evidence sets | Quorum families, election, topology certificate |
| Read locality taxes writes | [[Bodega]] places every responder on same-key write completion; [[Pando]] separates cheap discovery from full recovery | Because a local reader must prove it has seen every relevant write, read authority creates write-side coverage or catch-up obligations | Read authority, write quorum, version reconstruction |
| Fast promises are fragile across views | [[Jetpack]] pauses and recovers a same-view set; [[CURP]] versions witness lists; [[Bodega]] leases rosters; [[OmniPaxos]] stops an old configuration with `SS` | Because evidence is meaningful only under the roles and membership that created it, a view change can invalidate an otherwise sound fast certificate | View change, epoch fencing, stability marker |
| Commit latency hides execution latency | [[EPaxos-Revisited-2021]], [[Atlas]], [[SwiftPaxos]], and [[Copilot]] separate commit from executable dependency closure | Because ordering metadata can remain unresolved after commit, dependency chains and cycles move latency beyond the advertised fast path | Agreed object, dependency frontier, client completion |
| Recovery and reconfiguration retain unbounded evidence | [[GPaxos]] needs compact c-struct histories; [[CURP]] keeps unsynced witness records; [[OmniPaxos]] migrates the complete decided log; fast protocols retain view-specific evidence | Because future recovery may depend on old metadata, garbage collection is unsafe without a certified impossibility frontier | Checkpoint rule, evidence retirement, state transfer |
| Performance telemetry lacks a safe control boundary | [[Avicenna]] separates counterfactual latency from log safety; [[Bodega]] and [[OmniPaxos]] use timing for liveness; [[PigPaxos]] graylists relays | Because adaptive control observes noisy clocks, load, and reachability, using telemetry directly in safety rules risks hidden synchrony assumptions | Control plane, epoch transition, proof boundary |
| Network ordering splits order from agreement | [[Hydra]] supplies ordered delivery or ordered drops but no application commit; [[HydraPaxos]] adds majority durability and `NO-OP` repair whose inherited evidence rule is partly [[unresolved-confusions|Unclear]] | Because a network primitive decides relative position before replicas establish durability or missing-message fate, composition needs an explicit adapter invariant | Delivery abstraction, receiver quorum, drop recovery, clock fencing |
| Object locality creates ownership churn | [[WPaxos]] localizes repeated `Q2` commits but pays WAN `Q1` for [[object-stealing]]; its base algorithm is single-object and its unfinished-slot recovery payload is partly [[unresolved-confusions|Unclear]] | Because locality is converted into per-object leadership, shifting access and multi-object commands turn placement into recovery and cross-log ordering work | Ownership rule, migration policy, multi-object barrier, topology quorum |
| Current coverage has hard negative space | [[bft-consensus]] and [[dag-based-consensus]] remain placeholders; all cataloged protocols are non-Byzantine | Because the wiki lacks sourced BFT and DAG mechanisms, lifting crash-only ideas into those models would silently change equivocation and ordering assumptions | Fault model, authentication, DAG proof obligations |

## 100 Candidates

### A. Recovery-Derived Fast Evidence

Because fast paths repeatedly create evidence that later recovery struggles to classify, derive early completion from an explicit safe-recovery order.

1. **Recovery-Predicate-Derived Fast Path** - Idea: define a recovery selector first, then permit fast completion only for certificates that every admissible recovery quorum maps to the same result. Proof hook: selector determinism plus quorum intersection implies agreement.
2. **Ambiguity-Bounded Fast Certificate** - Idea: generalize [[Copilot]]'s ambiguous count interval into a certificate that names the exact auxiliary object recovery must inspect. Risk: recursive inspection must terminate without circular safety reasoning.
3. **Validated Dependency Core** - Idea: split dependency evidence into a quorum-certified conflict core and a validation-only fringe, combining the recoverability motivation of [[Atlas]] with the visibility discipline of [[EPaxosStar]]. Proof hook: fringe repair cannot remove a required conflict edge.
4. **Prefix-or-NoOp Recovery Lattice** - Idea: order recovery outcomes as committed prefix, validated candidate, and `noOp`, so recovery can move only upward in evidence strength. Risk: `noOp` resubmission must preserve exactly-once client semantics.
5. **Cross-Log Orientation Certificate** - Idea: make [[Copilot]] commit an explicit certificate orienting every relevant cross-log pair instead of recovering orientation indirectly from another entry. Cost: certificate growth and more replica checks.
6. **Self-Describing Fast Certificate** - Idea: include ballot, view, quorum-family version, member identities, and safe-recovery predicate identifier in every fast certificate. Proof hook: recovery rejects certificates whose geometry or selector cannot be reconstructed.
7. **Same-View Evidence Normalizer** - Idea: deterministically normalize non-identical fast replies within one view before equality testing. Proof hook: normalization is monotone over conflict evidence and cannot merge certificates across views.
8. **Recovery-Selectable C-Struct** - Idea: restrict [[GPaxos]] fast c-structs to a compact class with a unique safe extension computable from any recovery quorum. Evaluation hook: quantify lost concurrency against simpler recovery and checkpointing.
9. **Dual-Plane Commit Certificate** - Idea: commit a small choice certificate plus a separate repair certificate naming evidence needed after failure. Proof hook: neither plane alone authorizes execution, and recovery preserves their binding.
10. **Blocker-Carrying Prepare** - Idea: Prepare replies report both accepted evidence and the first local fact that would invalidate each candidate, turning hidden recovery blockers into explicit proof objects. Risk: blocker sets may be large or stale.

### B. Removing Special-Role Vetoes

Because all-witness, all-proposer, leader-including, and responder-covering rules make optimized paths sensitive to one slow role member, make safety depend on recoverable role subsets.

11. **Quorumized Witness Durability** - Idea: replace [[CURP]]'s all-witness completion with a witness quorum and an agreed recovery-selection rule. Proof hook: every completed unsynced operation appears in the uniquely selected reconstruction, not merely somewhere in a union.
12. **Proposer-Subset Fast Shim** - Idea: let [[Jetpack]] fast-commit with a certified active proposer subset while fencing excluded host proposers for that view. Risk: fencing may cost as much as changing the host view.
13. **Responder-Covered Flexible Writes** - Idea: co-design [[Bodega]] responder sets with [[FPaxos]] Phase 2 families so each write chooses a low-latency `Q2` that covers the key's responders. Proof hook: every later `Q1` intersects every eligible responder-covering `Q2`.
14. **Role-Elastic Fast Quorum** - Idea: express special-role inclusion as `k-of-r` role coverage plus ordinary voter thresholds instead of all-role coverage. Proof hook: recovery reconstructs every promise that could affect order despite omitted role members.
15. **Veto-Budget Certificate** - Idea: attach an explicit budget of unavailable witnesses, proposers, responders, or anchors to each path. Evaluation hook: compare availability and certificate size across [[CURP]], [[Jetpack]], [[Bodega]], and [[SwiftPaxos]].
16. **Phase-Asymmetric Dependency SMR** - Idea: use small common-case dependency quorums and larger recovery quorums as in [[FPaxos]], but derive intersections against every fast-certificate class. Risk: current-command progress may survive when leader replacement cannot.
17. **Restricted-Universe Fast Recovery** - Idea: borrow [[Avicenna]]'s restricted normal acceptor universe to make dependency fast evidence recoverable from a tiny known subset. Risk: multiple failures may require a periodic all-replica phase.
18. **Special-Role Failure Escrow** - Idea: special roles pre-deposit durable promises with ordinary replicas so one slow role member can be omitted without losing its safety contribution. Proof hook: escrow updates are ordered before role activation.
19. **Conflict-Class Quorum Profiles** - Idea: assign different fast, slow, and role-coverage rules to cold, hot, read-heavy, and multi-key classes. Proof hook: class changes are epoch-fenced and multi-class commands select one dominating profile.
20. **Topology-Certified Quorum Market** - Hypothesis: replicas advertise latency and load, while a verifier admits only selections belonging to a pre-proved quorum family, including zone/node-structured [[WPaxos]] families. Safety uses family membership; telemetry affects only which valid quorum is tried first.

### C. Cheaper Fail-Slow Tolerance

Because [[Copilot]] and [[Avicenna]] pay substantial redundant work to mask one slow replica, reduce the granularity of shadow work without making latency detection a safety premise.

21. **Sparse Shadow Execution** - Idea: shadow only state digests and predicted dependencies, executing a command fully only when counterfactual latency diverges. Proof hook: shadow state never authorizes a real reply.
22. **Counterfactual Metadata Shadow** - Idea: estimate an alternative leader using queue, quorum, and prefix metadata rather than a second ordered log. Evaluation hook: measure false rotations and missed gray failures against [[Avicenna]].
23. **Striped Copilots** - Idea: partition commands across several pilot pairs so each command has two paths but no replica duplicates the entire workload. Risk: multi-stripe commands need one recoverable cross-stripe order.
24. **Takeover-on-Dependency-Debt** - Idea: trigger [[Copilot]]-style per-entry takeover only when unresolved predecessors exceed a certified execution-debt threshold. Safety remains ballot-based; the threshold controls liveness and cost only.
25. **Sequencer-Frontier Shadow** - Idea: shadow each [[Hydra]] sequencer's frontier progress and predicted flush latency without letting the predictor authorize delivery, failure, or removal. Evaluation hook: reduce slowest-frontier tails while measuring false flushes and false suspicions.
26. **Client-Visible QC Scoring** - Idea: extend [[OmniPaxos]] quorum-connected candidacy with client-path latency scores, but use scores only to rank already eligible ballots. Proof hook: election safety and monotonic ballots ignore score accuracy.
27. **Fail-Slow-Tolerant Witness Set** - Idea: rotate or hedge [[CURP]] witnesses based on response tails while an epoch certificate fixes the active recovery set. Risk: overlapping old/new unsynced operations during witness rotation.
28. **Slow-Role Quarantine Epoch** - Idea: quarantine a persistently slow proposer, responder, witness, relay, sequencer, or object owner through one common role-removal protocol. Proof hook: all evidence produced before quarantine is either absorbed or recoverable.
29. **Dual-Path Reply Without Dual Execution** - Idea: two ordering paths race, but only the first execution certificate authorizes state-machine execution; the loser stores recovery evidence and deduplication state. Risk: the paths must agree on responses for non-deterministic operations.
30. **Fail-Slow Proof Decomposition** - Idea: design an SMR with separate invariants for crash agreement, eventual progress, and bounded slowdown relative to an explicit counterfactual. Evaluation hook: prevent performance claims from silently assuming perfect failure detection.

### D. Connectivity- and Placement-Aware Consensus

Because a live majority can be unusable under partial link partitions and a safe local quorum can be poorly placed for current demand, combine topology-aware election, ownership, and safely versioned quorum geometry.

31. **QC-Aware Flexible Paxos** - Idea: a leader may select only [[FPaxos]] or zone/node-structured [[WPaxos]] `Q2` sets it can directly reach, while election certifies that some valid `Q1` and `Q2` remain reachable. Proof hook: every allowed `Q1` still intersects every allowed `Q2` independent of observed topology.
32. **Directed-Connectivity BLE** - Idea: generalize [[OmniPaxos]] quorum connectivity from bidirectional sessions to separately certified send and receive reachability. Risk: Phase 1 and Phase 2 may require different directed paths and liveness predicates.
33. **Conflict-Class QC Anchors** - Idea: elect a quorum-connected anchor or [[WPaxos]]-style owner per hot conflict class while cold commands remain leaderless. Proof hook: multi-class commands choose one deterministic anchor order and survive anchor changes.
34. **Connectivity-Carrying Ballots** - Idea: ballots name a topology epoch and the quorum families valid under it. Proof hook: later recovery considers every certificate from topology epochs not yet retired.
35. **Partition-Stable Roster Activation** - Idea: require a prospective [[Bodega]] roster to prove both majority lease support and responder-to-write-quorum connectivity before activation. Risk: connectivity evidence must affect liveness policy, not linearizability safety.
36. **Multi-QC Leader Handoff** - Idea: when several quorum-connected servers exist, prepare the top two candidates so takeover avoids a full cold synchronization. Proof hook: only one ballot may accept new commands; the shadow candidate stores no executable authority.
37. **Receiver-Quorum Groupcast Adapter** - Idea: define the application interface that turns [[Hydra]] receiver deliveries and drops into durable group outcomes, with explicit adoption and reconfiguration quorums. Proof hook: every terminal sequencer-removal report intersects all application evidence that could make an old message executable.
38. **Clock-Epoch-Fenced Sequencers** - Idea: bind each [[Hydra]] sequencer's monotonic clock to a durable boot epoch and require an addition frontier before a restarted clock can stamp traffic. Risk: the fence may recreate a centralized configuration bottleneck.
39. **Thrash-Resistant Object Ownership** - Idea: add hysteresis and minimum-benefit windows to [[WPaxos]] stealing, but let only a successful higher-ballot `Q1` transfer authority. Evaluation hook: WAN Phase 1 rate and locality benefit under alternating demand.
40. **Multi-Object Stealing Barrier** - Idea: transfer ownership of a command's object set through one canonical barrier before any constituent owner opens later slots. Proof hook: overlapping multi-object transfers cannot form a cycle or expose a partly transferred transaction.

### E. Read Authority Without Write Amplification

Because local linearizable reads place responders on the write critical path or require expensive catch-up, change the evidence by which a reader proves freshness.

41. **Threshold-Covered Responders** - Idea: replace all-responder write coverage with threshold responder groups whose intersections guarantee that every local read consults at least one covered member. Risk: one-server reads may be lost unless authority is transferable.
42. **Coded Responder Catch-Up** - Idea: store responder freshness certificates as erasure-coded fragments, following [[Pando]]'s identity/reconstruction split. Proof hook: a reader reconstructs both value identity and a committed-prefix proof.
43. **Transactional Roster Barrier** - Idea: order a multi-key roster barrier that installs one responder set and snapshot frontier for all keys in a transaction. Risk: concurrent roster changes and multi-key writes need a single serialization rule.
44. **Dependency-Aware Local Reads** - Idea: a responder serves locally when it holds an execution-ready dependency frontier for the read set, instead of waiting for the global log prefix. Proof hook: every earlier conflicting write is in the frontier.
45. **Responder Split/Merge Protocol** - Idea: split a hot key's responder set or merge cold sets only at a certified write frontier. Proof hook: a write spanning the transition covers either the old or new authority under one epoch, never an unsafe mixture.
46. **Lease-Free Stable Responder Certificate** - Idea: replace time-bounded roster leases with ballot-ordered durable grants revoked through consensus. Tradeoff: removes bounded clock drift but may make failed responder removal slower.
47. **Multi-Version Read Authority** - Idea: responders retain certified historical versions and serve a linearizable read from the newest version whose frontier dominates the client's token. Risk: clients without tokens still need a global freshness rule.
48. **Decoupled Read-Authority Recovery** - Idea: recover chosen writes with ordinary Paxos while separately recovering which replicas may answer reads, as [[Bodega]] already separates value and roster evidence. Proof hook: read authority never advances beyond recovered write state.
49. **Client-Carried Freshness Proof** - Idea: writes return a compact frontier token that subsequent reads present to any responder. Evaluation hook: reduce responder catch-up stalls while bounding token size and handling clients that lose tokens.
50. **Conflict-Class Read Shadow** - Idea: keep a nearby read shadow only for hot classes, fed by certified conflict updates rather than the entire log. Proof hook: absence of an update cannot be inferred from timing.

### F. Promise-Preserving View Changes

Because fast paths rely on view-specific proposer, witness, responder, or anchor promises, make those promises first-class inputs to view change.

51. **Promise-Carrying View Certificate** - Idea: every view-change certificate summarizes outstanding fast promises and the frontier at which they become irrelevant. Proof hook: a new view orders every recovered promise before conflicting work.
52. **Host-Agnostic Stability Marker** - Idea: standardize [[Jetpack]]'s marker as an abstract host command with required ordering and durability semantics. Risk: some hosts cannot guarantee the receipt/proposal order needed before the marker.
53. **Dependency-Host Fast Shim** - Idea: adapt the fast shim to an [[EPaxos]]-like host by recovering a dependency certificate rather than a total-order set. Proof hook: view change preserves conflict visibility and execution order, not merely log position.
54. **Multi-Leader Promise Compression** - Idea: compress all-proposer promises into one conflict-class certificate endorsed by a quorum of proposer delegates. Risk: delegation must prevent an omitted proposer from ordering a conflict first.
55. **Recovery-Lane Fast Shim** - Idea: keep a dedicated ordered recovery lane beside an unchanged host, so a marker need not pause unrelated conflict classes. Proof hook: the lane and host agree on cross-lane conflicts.
56. **View-Coupled Quorum Versioning** - Idea: bind [[FPaxos]] quorum-family changes to the same view certificate that transfers fast promises. Proof hook: no certificate combines member evidence from incompatible family versions.
57. **Staged New-Proposer Activation** - Idea: activate new-view proposers per conflict class after that class's old promises are stabilized. Risk: multi-class commands can cross active and inactive regions.
58. **Conflict-Predicate Epoch Upgrade** - Idea: version the application conflict predicate and install upgrades only after old-version in-flight commands reach a certified frontier. Proof hook: no pair is classified non-conflicting under one side and conflicting under the other without ordering.
59. **Fast-Path Retirement Certificate** - Idea: explicitly retire a fast layer, sequencer configuration, or role set once a quorum certifies that all its completed operations are in the host log or represented as ordered drops. This enables safe plugin removal and metadata garbage collection.
60. **Composable View-Change Contracts** - Idea: specify host-to-plugin obligations as assume-guarantee contracts for proposal receipt, order, commit, and recovery. Evaluation hook: mechanically test whether [[Jetpack]], [[CURP]]-style witnesses, or read rosters compose with a host.

### G. Execution-Ready Agreement

Because command commitment can precede dependency closure and state-machine execution, make executable structure part of the agreed object.

61. **Execution-Ready Fast Commit** - Idea: reply only when the command and every predecessor needed for deterministic execution have certificates or form one jointly certified SCC. Evaluation hook: compare response tails, not only commit medians.
62. **SCC-Atomic Certificate** - Idea: agree on a canonical SCC batch when conflicts create a cycle, rather than committing vertices independently and discovering the SCC later. Proof hook: concurrent recoveries derive the same SCC membership and order.
63. **Bounded Dependency Horizon** - Idea: use measured dependency depth only to trigger a safe leadered fallback after a bound; never drop edges based on time. Evaluation hook: bound hot-key execution tails while preserving asynchronous safety.
64. **Dependency-Debt Anchor** - Idea: install a temporary conflict-class serializer when unresolved dependency debt crosses a threshold, then retire it at a certified frontier. Proof hook: anchor transition orders commands spanning the boundary.
65. **Orientation-First Dual Log** - Idea: before either [[Copilot]] log commits a command, certify its cross-log orientation; payload ordering then proceeds independently. Tradeoff: extra metadata exchange may erase fast-path latency.
66. **Frontier-Stamped Recovery** - Idea: every accepted dependency value carries the highest execution frontier it assumes. Recovery may prefer a higher frontier only after proving it preserves all lower-frontier conflicts.
67. **Cycle-Budget Commit Rule** - Idea: allow fast commit only while the certificate proves adding the command cannot exceed a bounded SCC size; otherwise use ordered fallback. Risk: cycle prediction may require transitive metadata.
68. **Hot-Class Slot Forfeiture** - Idea: use [[Rabia]]-style `bottom` only for a saturated conflict class whose dependency order remains ambiguous, then retry requests after a barrier. Proof hook: weak validity, deduplication, and compaction are explicit.
69. **Deduplicate-Before-Ordering** - Idea: replicas certify a stable request id before duplicate copies enter dual or leaderless ordering paths. Proof hook: deduplication cannot discard the only copy carrying a required real-time dependency.
70. **Execution Certificate Reply** - Idea: clients complete on a certificate for executable state transition, not a protocol-specific internal commit state. Evaluation hook: compare Paxos logs, dependency SMR, and witness-based completion under one semantic metric.

### H. Reconfiguration and Evidence Retirement

Because old fast, lease, witness, dependency, sequencer, ownership, and log evidence may remain safety-relevant, make reconfiguration and garbage collection one certified frontier problem.

71. **Stop-Sign Witness Handoff** - Idea: order a stop-sign before changing [[CURP]] witness lists, absorb all old unsynced records into backups, then activate the new list. Proof hook: zombie clients cannot complete under the retired version.
72. **Snapshot-Certified Omni Start** - Idea: let a new [[OmniPaxos]] member start from a snapshot digest plus a certified suffix instead of the complete decided log. Proof hook: snapshot state equals execution of the retired prefix and no post-`SS` entry exists.
73. **Recovery-as-Reconfiguration** - Idea: treat membership change as a recovery ballot whose safe value is an executed frontier plus new membership. Proof hook: the normal safe-value selector also fences old participants.
74. **Joint-Epoch Dependency Closure** - Idea: require old/new joint evidence only for commands whose dependency closure crosses the boundary; independent post-frontier commands use the new epoch immediately. Risk: closure detection cannot rely on incomplete local views.
75. **Roster Handoff Barrier** - Idea: couple [[Bodega]] roster replacement with a log barrier carrying responder catch-up thresholds. Proof hook: no old responder serves beyond the barrier and every new responder covers it.
76. **Shadow/Real Log Compaction Frontier** - Idea: certify when [[Avicenna]] shadow state and [[Copilot]] duplicate-log entries can be discarded without affecting takeover or deduplication. Risk: in-flight clients may still refer to compacted request ids.
77. **Fast-Evidence GC Certificate** - Idea: replicas retire fast-quorum reports only after a recovery quorum certifies the operation is durably represented in ordinary committed state. Proof hook: later recovery never needs retired view evidence.
78. **Quorum-Family Epoch Ledger** - Idea: keep a compact ordered ledger of [[FPaxos]] quorum-family versions and their retirement frontiers. Recovery checks only versions not proven retired.
79. **Unified Configuration Serializer** - Idea: assign roster, witness, quorum-family, sequencer, ownership-policy, and replica-membership changes one consensus sequence so their evidence epochs cannot cross. Evaluation hook: determine whether serialization becomes a control-plane bottleneck.
80. **Partial-Membership State Reconstruction** - Idea: use [[Pando]]-style coded state so one new QC server can reconstruct a certified snapshot from fragments spread across old members. Proof hook: data availability and consensus identity remain separate.

### I. Lower-Cost Evidence and Transport

Because richer recovery and execution certificates can dominate bandwidth and storage, optimize representation without treating transport helpers as voters.

81. **Erasure-Coded Recovery Certificates** - Idea: agree on a small certificate identity and store large dependency/path evidence as coded fragments. Proof hook: every valid recovery quorum reconstructs enough exact evidence before selection.
82. **Coded Witness Ledger** - Idea: witnesses store coded request records so fast completion tolerates missing witness fragments. Risk: unioning fragments from incompatible witness epochs may fabricate an invalid record set.
83. **Groupcast Quorum Aggregate** - Idea: aggregate [[Hydra]] receiver-group reports with member identities, largest counters, and configuration tags while leaving the application quorum predicate explicit. Proof hook: the aggregate refines exactly the reports used to close old gaps.
84. **Direct Client Certificate Assembly** - Idea: replicas send compact proof shares directly to clients while relays carry payloads and commit notifications. Risk: client completion must remain recoverable if the client disappears immediately.
85. **Hierarchical Recovery CDN** - Idea: cache immutable certificates regionally so recovery fetches evidence nearby; caches never vote. Evaluation hook: recovery latency and cache invalidation at retirement frontiers.
86. **Payload-Late Consensus** - Idea: agree on command id, validity digest, and ordering metadata before reconstructing the payload from coded storage. Proof hook: a chosen id cannot become permanently unexecutable.
87. **Exact-Core Metadata Tiering** - Idea: keep a small exact conflict core in quorum messages and archive optional transitive paths separately. Proof hook: the exact core suffices for agreement and deterministic execution.
88. **Adaptive Certificate Expansion** - Idea: begin with compact evidence and request exact details only when replies disagree or recovery starts. Risk: replicas must retain expandable source facts until a GC certificate.
89. **Proof-Carrying Frontier Aggregate** - Idea: each relay or receiver-frontier aggregate includes a membership bitmap and hash tree enabling end-to-end verification of included replies, sequence counters, and configuration. This changes transport cost, not quorum semantics.
90. **Checkpoint-First SMR** - Idea: choose a compact, recoverable checkpoint object first, then constrain normal metadata so every committed operation can be folded into it monotonically. Evaluation hook: compare steady-state overhead against bounded recovery state.

### J. Controlled Paradigm Switching

Because no single leader, leaderless, randomized, lease, witness, network-ordered, or object-owned regime dominates every workload and failure pattern, switch mechanisms only at explicit safety boundaries.

91. **Weak-Validity Recovery Slots** - Idea: permit an agreed `bottom` outcome only for recovery ambiguity, not ordinary commands, and resubmit through stable request ids. Proof hook: agreement is on `bottom`; validity and liveness explicitly allow retry.
92. **Randomized Safe-Repair Scheduler** - Idea: use a common coin only to choose among already-safe recovery coordinators or candidates. Safety is deterministic; randomness reduces dueling repairs.
93. **Hydra-Ordered Hot-Class Mode** - Idea: hot conflict classes temporarily use [[Hydra]] ordered-unreliable groupcast while cold classes use dependency fast paths. Proof hook: the transition certificate binds Hydra delivery/drop evidence to application commit, and a missing position cannot race a dependency-path decision.
94. **Per-Class Paradigm Switching** - Idea: each conflict class moves among leaderless, anchored, witness-fast, network-ordered, object-owned, and serialized modes at certified frontiers. Proof hook: multi-class commands select one mode transition order.
95. **Latency-Semantic Commit Modes** - Idea: expose separate client contracts for durable, chosen, executable, and globally observed completion, each with an explicit certificate. Risk: applications may misuse weaker modes unless semantics are mechanically enforced.
96. **Evidence-Plane SMR** - Idea: separate command ordering from an append-only evidence plane storing promises, topology epochs, and retirement certificates. Proof hook: ordering consumes only finalized evidence-plane entries.
97. **Safe-Value DSL Protocol** - Idea: generate fast and recovery handlers from a small language describing evidence orders, quorum families, and safe selectors. Evaluation hook: encode [[FastPaxos]], [[GPaxos]], [[FPaxos]], [[Atlas]], and [[Jetpack]] without erasing protocol-specific metadata.
98. **Mechanism-Budgeted SMR** - Idea: select a mode from explicit budgets for failures, slow roles, conflict, metadata, frontier delay, ownership churn, and recovery latency, then record that choice in the epoch certificate. Safety must hold for every advertised mode and transition.
99. **Source-Gated Byzantine Promise Layer** - Hypothesis: replace crash-only proposer promises with authenticated, quorum-checkable promises that tolerate equivocation. `TODO`: ingest BFT sources before defining quorums or claiming safety.
100. **Source-Gated DAG Recovery Plane** - Hypothesis: store recovery evidence in a DAG while retaining a separately proved command-order rule. `TODO`: ingest DAG consensus sources before asserting availability, ordering, or garbage-collection properties.

## Ranked Shortlist and Red Team

| Rank | Candidate | Why it survives | Main red-team failure | Next artifact |
|---:|---|---|---|---|
| 1 | Recovery-Predicate-Derived Fast Path | Attacks the recurring fast/recovery mismatch at its source and applies across several families | A selector may be deterministic yet preserve too little conflict or prefix information | Small-state model with two fast certificates and two recoveries |
| 2 | Promise-Carrying View Certificate | Unifies [[Jetpack]], roster, witness, and quorum-version handoff problems | New work may start before every old promise is either ordered or retired | View-transition state machine and stale-message counterexample search |
| 3 | Execution-Ready Fast Commit | Targets user-visible dependency tails rather than nominal commit latency | Certifying closure may recreate a global serialization bottleneck | Simulator reporting response, commit, closure, and execution time |
| 4 | QC-Aware Flexible Paxos | Combines phase-specific quorum geometry with explicit partial-connectivity liveness | Locally reachable quorums may be safe but leave no reachable recovery `Q1` | Quorum/topology solver over directed failure graphs |
| 5 | Responder-Covered Flexible Writes | Directly attacks [[Bodega]]'s write-amplification defect with a crisp algebraic obligation | A fast `Q2` covering responders can make future Phase 1 unavailable | Enumerate `Q1`, `Q2`, responder sets, and failure placements |
| 6 | Sparse Shadow Execution | Preserves [[Avicenna]]'s safety/control separation while reducing redundant work | Metadata-only shadows may miss execution or client-path slowdowns | Trace-driven comparison with full shadow processing |
| 7 | Quorumized Witness Durability | Removes [[CURP]]'s all-witness veto and exposes a clear recovery theorem | Different recovery witness sets may omit different completed operations | Witness-quorum intersection and reconstruction model |
| 8 | Dependency-Host Fast Shim | Extends fast-layer composition to a currently excluded host family | Preserving instance agreement may still violate global execution order | Dependency visibility model across host view change |
| 9 | Snapshot-Certified Omni Start | Replaces complete-log migration with a standard compact state-transfer object | Snapshot may omit still-possible or post-frontier decisions | `SS`/snapshot/suffix startup invariant |
| 10 | Role-Elastic Fast Quorum | Generalizes several special-role vetoes without tying the idea to one protocol | Omitted roles may retain authority to create a conflicting order | Role-fencing algebra plus exhaustive small configurations |
| 11 | Per-Class Paradigm Switching | Addresses workload heterogeneity instead of optimizing one global common case | Multi-class commands and concurrent mode changes can form cycles | Two-class transition model with multi-key commands |
| 12 | Fast-Evidence GC Certificate | Makes long-lived recovery metadata and safe reconfiguration tractable | A delayed client or replica may reveal evidence after retirement | Message-delay model with arbitrary stale delivery |
| 13 | Receiver-Quorum Groupcast Adapter | Makes the ordering-to-consensus boundary explicit instead of treating network delivery as commit | Application adoption evidence may not intersect Hydra removal evidence strongly enough | Two receiver groups, partial delivery, removal, and late-message model |
| 14 | Thrash-Resistant Object Ownership | Targets a concrete WAN locality defect while leaving safety in ordinary Phase 1 | Hysteresis may only move churn cost into forwarding or stale placement | Alternating-locality trace model with WAN `Q1` and local `Q2` costs |

Candidates 99 and 100 do not survive ranking yet: the wiki lacks the BFT and DAG sources needed to define their threat models and proof obligations precisely.

## Shared Red-Team Questions

- Can two incompatible results each collect locally valid evidence, especially across different views or quorum-family versions?
- Can recovery observe an evidence set for which the selector has no safe, live choice?
- Does a performance trigger secretly become a clock, load, reachability, or failure-detector premise in the safety theorem?
- Does removing a role veto merely move the veto into fencing, escrow, state transfer, or certificate construction?
- Does client completion imply durability, chosen state, execution readiness, and exactly-once response semantics, or only one of them?
- Which metadata remains necessary after checkpoint, reconfiguration, plugin removal, or client retry?
- Is the candidate a real mechanism shift, or only two existing protocols placed side by side without a new invariant?

## Next Proof and Evaluation Work

1. Specify Candidate 1 as an evidence poset, quorum-family predicate, deterministic recovery selector, and commit rule; search for two incompatible selected outcomes.
2. Model Candidate 2 with old/new proposer sets, delayed fast acknowledgements, a stability frontier, and staged activation.
3. Extend the [[latency]] simulator agenda to report client response, internal commit, dependency closure, execution, and one-fail-slow counterfactual latency.
4. Build a quorum/topology enumerator for Candidates 4, 5, 7, and 10, including phase-specific availability and correlated failure placement.
5. Define one common evidence-retirement invariant for Candidates 9 and 12: no future admissible recovery or retry may require state below the retired frontier.
6. Specify Candidate 37's receiver-group adoption interface and search partial-delivery/removal traces for a message that one group executes after another permanently drops it.
7. Evaluate Candidate 39 with alternating regional demand, forwarding, ownership-transfer `Q1`, steady-state `Q2`, and dueling-steal backoff.
8. Ingest NOPaxos before claiming Candidate 37 composes safely with [[HydraPaxos]]'s missing-message recovery.
9. Ingest at least one BFT paper and one DAG-based consensus paper before refining Candidates 99 and 100.

## Related Pages

[[protocol-catalog]], [[latency]], [[quorum-systems]], [[recovery-rules]], [[reconfiguration]], [[slowdown-tolerance]], [[partial-connectivity]], [[roster-lease]], [[view-change-hazard]], [[object-stealing]], [[Hydra]], [[HydraPaxos]], [[WPaxos]], [[recoverability]]
