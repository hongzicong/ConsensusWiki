# New Protocol Ideas

These notes are speculative research prompts derived from the current wiki, not claims about existing papers. Sourced facts are linked to wiki pages. Candidate mechanisms are labeled as ideas or hypotheses and need proofs or counterexamples before they should be treated as protocols.

## Evidence Matrix

| Protocol | Observed limitation | Shared assumption | Source page | Confidence |
|---|---|---|---|---|
| [[FastPaxos]] | Fast rounds need collision-free proposals or recovery must choose a safe value. | Fast evidence is a value-level quorum certificate. | [[fast-paths]], [[recovery-rules]], [[quorum-intersection]] | High |
| [[EPaxos]] | Fast commit needs matching attributes, and evaluation notes conflict/tail-latency sensitivity. | Leaderless fast SMR commits dependency metadata observed before recovery evidence is stable. | [[fast-paths]], [[conflict-handling]], [[leaderless-protocols]] | High |
| [[EPaxosStar]] | Corrected recovery uses validation and may commit `Nop`, making recovery central and complex. | Safety is repaired by validating possibly omitted conflicting commands. | [[EPaxosStar]], [[recovery-rules]], [[proof-techniques]] | High |
| [[Mencius]] | A failed or idle owner can leave gaps until skip or revocation. | Slot ownership prevents conflicts by assigning authority in advance. | [[Mencius]], [[leader-roles]], [[commit-rules]] | High |
| [[PigPaxos]] | Relays reduce leader load but keep a single ordering leader and can add retry tail cases. | Transport aggregation is separated from consensus authority. | [[PigPaxos]], [[leader-roles]], [[recovery-rules]] | High |
| [[Atlas]] | Fast path weakens exact matching but requires recoverable dependency unions and a chosen failure budget `f`. | Fast evidence must be reconstructible after up to `f` failures. | [[Atlas]], [[fast-paths]], [[quorum-intersection]] | High |
| [[SwiftPaxos]] | Leader-including fast quorums reduce ambiguity but reintroduce leader centrality. | Fast evidence is shaped around ballot-leader participation. | [[SwiftPaxos]], [[quorum-systems]], [[leader-roles]] | Medium |
| [[Pando]] | Erasure-coded recovery must distinguish value identity from enough splits. | Recovery evidence can be coded and reconstructible rather than full-value. | [[Pando]], [[quorum-systems]], [[recovery-rules]] | Medium |
| [[Rabia]] | Weak validity avoids recovery by allowing `bottom` slots, with log density depending on alignment. | Ambiguity can be resolved by forfeiting a slot and retrying. | [[Rabia]], [[fast-paths]], [[commit-rules]] | High |

## Limitation Clusters

### Cluster 1: Fast Evidence Is Too Brittle

**Shared weakness:** Fast paths often require identical values, identical dependency metadata, or leader-shaped evidence.

**Protocols exhibiting it:** [[FastPaxos]], [[EPaxos]], [[EPaxosStar]], [[SwiftPaxos]], with [[Atlas]] as a partial exception.

**Shared paradigm / hidden assumption:** A fast decision is safe only if the evidence already looks almost like a final decision.

**Assumption type:** Mechanism and proof.

**Causal bottleneck:** Because protocols make fast evidence double as recovery evidence, they reject useful partial agreement under benign disagreement.

**Design opportunity:** Define fast evidence as a reconstructible object, not necessarily a matching object.

### Cluster 2: Recovery Carries Hidden Complexity

**Shared weakness:** Recovery must distinguish chosen, maybe chosen, and merely observed states.

**Protocols exhibiting it:** [[FastPaxos]], [[EPaxos]], [[EPaxosStar]], [[Atlas]], [[SwiftPaxos]], [[Pando]].

**Shared paradigm / hidden assumption:** Recovery is a later cleanup phase rather than a first-class part of the fast commit predicate.

**Assumption type:** Proof.

**Causal bottleneck:** Because the common path records just enough to go fast, recovery must reconstruct intent from incomplete traces.

**Design opportunity:** Commit only evidence whose recovery certificate is already self-describing.

### Cluster 3: Leader Avoidance Moves Cost Elsewhere

**Shared weakness:** Removing a stable leader often moves ordering cost into dependency tracking, validation, randomized retries, or execution delay.

**Protocols exhibiting it:** [[EPaxos]], [[EPaxosStar]], [[Atlas]], [[Rabia]], [[Mencius]].

**Shared paradigm / hidden assumption:** Load balance requires distributing ordering authority at command or slot granularity.

**Assumption type:** Mechanism and workload.

**Causal bottleneck:** Because authority is spread before conflict structure is known, protocols later pay to reconcile conflicting local views.

**Design opportunity:** Allocate authority adaptively by conflict class, evidence quality, or observed locality.

### Cluster 4: Dependency Metadata Becomes the Log

**Shared weakness:** Dependency-based protocols commit quickly but may execute late due to dependency chains or SCCs.

**Protocols exhibiting it:** [[EPaxos]], [[EPaxosStar]], [[Atlas]], [[SwiftPaxos]].

**Shared paradigm / hidden assumption:** It is cheaper to commit partial order metadata than a total order.

**Assumption type:** Modeling and workload.

**Causal bottleneck:** Because the protocol defers total-order choice to execution, contention creates metadata and scheduling debt.

**Design opportunity:** Make dependencies bounded, typed, compressible, or convertible into local total-order leases.

### Cluster 5: Quorum Size Is Treated As Static

**Shared weakness:** Fast, slow, and recovery quorum sizes are fixed for broad failure assumptions.

**Protocols exhibiting it:** [[FastPaxos]], [[EPaxosStar]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]].

**Shared paradigm / hidden assumption:** A deployment picks one failure budget and one quorum algebra.

**Assumption type:** Mechanism and operational.

**Causal bottleneck:** Because quorum shapes are static, protocols overpay during healthy periods and underadapt during regional stress.

**Design opportunity:** Use mode-switching quorums with explicit continuity certificates between modes.

### Cluster 6: Communication Overlays Do Not Change Proof Obligations

**Shared weakness:** Relay aggregation reduces load but leaves ordering and quorum proof unchanged.

**Protocols exhibiting it:** [[PigPaxos]], with contrast to leaderless protocols.

**Shared paradigm / hidden assumption:** Transport helpers should not participate in consensus evidence.

**Assumption type:** Mechanism.

**Causal bottleneck:** Because overlays are proof-transparent, they cannot help resolve ambiguity or recovery except by moving bytes.

**Design opportunity:** Let overlays produce auditable evidence summaries without granting them extra votes.

### Cluster 7: Ambiguity Is Either Recovered Or Forfeited

**Shared weakness:** Collisions trigger recovery, validation, slow paths, or `bottom` slots.

**Protocols exhibiting it:** [[FastPaxos]], [[EPaxosStar]], [[Atlas]], [[Rabia]].

**Shared paradigm / hidden assumption:** Ambiguity must be resolved at the same granularity as the original slot or command id.

**Assumption type:** Mechanism and proof.

**Causal bottleneck:** Because ambiguity is local to one instance, protocols cannot cheaply preserve useful partial work across nearby commands.

**Design opportunity:** Convert ambiguous slots into reusable ordering credits, dependency hints, or batch certificates.

### Cluster 8: Timing Is Kept Out Of Safety But Still Shapes Performance

**Shared weakness:** Safety is asynchronous, while latency claims depend on synchrony, stable leaders, aligned queues, or available quorums.

**Protocols exhibiting it:** All ingested protocols.

**Shared paradigm / hidden assumption:** Timing should not appear in safety evidence.

**Assumption type:** Operational and proof.

**Causal bottleneck:** Because timing is excluded from safety, protocols cannot safely exploit stable timing except as a performance heuristic.

**Design opportunity:** Add explicitly discardable timing hints that improve fast-path admission but never become safety evidence.

### Cluster 9: Reconfiguration Is Mostly Outside The Core Evidence Model

**Shared weakness:** The current wiki has limited ingested detail on reconfiguration; protocols are mostly described over static membership.

**Protocols exhibiting it:** [[Atlas]], [[Mencius]], [[PigPaxos]], [[Rabia]], and others as currently summarized.

**Shared paradigm / hidden assumption:** Membership changes are a layer around the protocol rather than part of each commit certificate.

**Assumption type:** Operational and modeling.

**Causal bottleneck:** Because certificates do not encode membership continuity, reconfiguration is hard to compose with fast or leaderless paths.

**Design opportunity:** Make membership epochs and quorum-continuity evidence explicit in every decision object.

### Cluster 10: Proof Burdens Are Protocol-Specific

**Shared weakness:** Each protocol uses a different proof object: values, tuples, dependency visibility, split reconstruction, value-locking, or relay refinement.

**Protocols exhibiting it:** All ingested protocols.

**Shared paradigm / hidden assumption:** Each protocol needs a bespoke invariant.

**Assumption type:** Proof and modeling.

**Causal bottleneck:** Because proof objects are not normalized, design reuse is harder and bugs hide in recovery translations.

**Design opportunity:** Build protocols around reusable evidence algebras: choose, reconstruct, validate, forfeit, refine.

## Protocol Candidates

Legend: `Top candidate` means worth modeling first; `Promising` means researchable but underspecified; `Weak` means likely dominated or high risk; `Reject` means kept only as a negative design note.

| # | Idea | Targeted limitation | Closest related protocols | Paradigm shift and core mechanism | Invariant to preserve | Main risk / proof obligation | Red-team |
|---:|---|---|---|---|---|---|---|
| 1 | Recoverable-Fast-Union Paxos | Exact fast matching is brittle. | [[EPaxos]], [[Atlas]] | Replace exact dependency equality with a frequency-threshold union certificate parameterized by the recovery quorum. | Conflicting committed commands are dependency-visible. | Prove every recoverer reconstructs the same union or a safe superset. | Top candidate |
| 2 | Self-Describing Fast Certificates | Recovery is under-specified at commit time. | [[FastPaxos]], [[EPaxosStar]] | Fast commits include a compact witness describing how later recovery must interpret partial evidence. | One command id commits one payload/dependency object. | Certificate may be as costly as slow-path evidence. | Promising |
| 3 | Validation-First EPaxos | Validation appears only during recovery. | [[EPaxosStar]], [[EPaxos]] | Add cheap pre-validation summaries during PreAccept so recovery validates fewer omitted conflicts. | Visibility for conflicting committed commands. | Hidden cost in metadata exchange and false negatives. | Promising |
| 4 | Fast Path With Recovery Budget Tags | One quorum algebra fits all conflicts. | [[EPaxosStar]], [[Atlas]] | Tag each command with an intended recovery budget and choose fast predicate accordingly. | Recovery quorums intersect the chosen fast evidence. | Tags could become unsafe if failure budget changes mid-command. | Promising |
| 5 | Dependency Checksum Consensus | Dependency equality is expensive to compare and transmit. | [[EPaxos]], [[SwiftPaxos]] | Use authenticated or deterministic dependency-set digests as fast equality evidence, fetch full deps only on mismatch. | Digest collision model must preserve dependency identity. | Requires trusted collision resistance or exact deterministic hashes in model. | Weak |
| 6 | Two-Tier Fast Evidence | Matching and reconstructible fast evidence are different strengths. | [[EPaxos]], [[Atlas]] | Allow tier 1 exact-match commit and tier 2 recoverable-union commit with different recovery rules. | Later recovery respects the tier used by commit. | Mode confusion could let incompatible evidence pass. | Promising |
| 7 | Witnessed Collision Fast Paxos | Fast Paxos treats collisions as recovery triggers. | [[FastPaxos]] | Acceptors attach collision-witness sets so a later coordinator can choose without full classic recovery. | Higher rounds choose a value safe for any possible lower choice. | May just repackage Phase 1 evidence. | Weak |
| 8 | Leader-Including Optional Fast Quorums | Leader inclusion helps recovery but hurts locality. | [[SwiftPaxos]], [[Atlas]] | Fast quorums include either the ballot leader or a recovery delegate elected for the command class. | Delegate participation must substitute for leader intersection. | Delegate election may add a hidden leader. | Promising |
| 9 | Fast-Quorum Evidence Escrow | Recovery loses original fast-quorum shape. | [[Atlas]], [[FastPaxos]] | Fast quorum members escrow signed summaries to a small durable witness set that recovery queries first. | Escrow cannot create votes, only preserve evidence. | Witness failures and durability assumptions complicate model. | Weak |
| 10 | Negative-Evidence Fast Commit | Protocols record what was seen, not what was absent. | [[EPaxosStar]], [[Rabia]] | Fast certificate includes bounded negative evidence for known non-conflicts or no-majority observations. | Absence claims cannot hide concurrent conflicting commits. | Hard to prove absence in asynchronous systems. | Reject |
| 11 | Recovery-Native Paxos | Recovery is a separate cleanup phase. | [[FastPaxos]], [[EPaxosStar]], [[Pando]] | Define each commit certificate as a recovery program with inputs, quorum type, and safe-selection rule. | Executing the recovery program preserves any chosen value. | Very abstract; may not yield a faster protocol. | Top candidate |
| 12 | Nop-With-Reason SMR | `Nop` or `bottom` loses useful information. | [[EPaxosStar]], [[Rabia]], [[Mencius]] | Replace null decisions with typed nulls carrying retry, dependency, or ownership evidence. | Null result cannot decide a conflicting command. | Typed nulls may leak into validity semantics. | Promising |
| 13 | Recoverable Partial Order Slots | Ambiguity is per command id or slot. | [[EPaxosStar]], [[Rabia]] | Ambiguous recovery commits a partial-order constraint rather than a command or null. | Later commands cannot violate committed constraints. | Constraint accumulation may block progress. | Promising |
| 14 | Evidence-Carrying Slow Path | Slow path discards why fast path failed. | [[EPaxos]], [[Atlas]] | Slow Accept includes the failed fast evidence so future recovery can avoid revalidation. | Slow quorum acceptance dominates earlier fast ambiguity. | Metadata growth on conflict-heavy workloads. | Promising |
| 15 | Phase-1b Dependency Reconstruction | Pando-style reconstruction for SMR metadata. | [[Pando]], [[Atlas]] | Store dependency metadata in coded fragments; recovery needs enough fragments to reconstruct a safe dep set. | Reconstructed dep object equals the committed identity. | Coded metadata may be larger and complex to update. | Promising |
| 16 | Recovery Quorum Hints | Recoverers search too broadly. | [[Atlas]], [[EPaxosStar]] | Commit certificates name preferred recovery quorums plus fallback algebra. | Hints cannot be required for safety. | Liveness may degrade if hints are stale. | Weak |
| 17 | Monotone Recovery Lattice | Recovery case splits are brittle. | [[EPaxosStar]], [[Atlas]], [[Pando]] | Model recovery values as a join-semilattice: accepted value beats reconstructible fast value beats null. | Join result is unique and safe across quorum evidence. | Need show lattice order matches actual safety, not wishful ranking. | Top candidate |
| 18 | Waiting-Certificate Recovery | Recovery cycles need explicit breaking. | [[EPaxosStar]] | Generalize `Waiting` into a certificate that transfers blocked recovery obligations to another command. | Waiting cannot mask an already committed conflicting command. | Could create long chains and liveness hazards. | Promising |
| 19 | Recover-Then-Validate Batching | Per-command validation repeats work. | [[EPaxosStar]], [[Atlas]] | Batch recovery of conflicting commands, validate the induced dependency subgraph once. | Each command id still has a single decision. | Batch graph may include commands that never commit. | Promising |
| 20 | Minimal-Counterexample Recovery | Recovery proofs are hard to audit. | [[EPaxosStar]], [[FastPaxos]] | Recovery coordinator emits the smallest ambiguity set that justifies fallback or null. | If ambiguity set is empty, recovered value must be unique. | More of a proof/debug artifact than a protocol. | Weak |
| 21 | Conflict-Class Leaders | Leaderless protocols pay under conflicts. | [[EPaxos]], [[Mencius]], [[SwiftPaxos]] | Elect temporary leaders per conflict class or key range only after contention is detected. | Commands in the class have one serialization authority per epoch. | Class detection races can create split authority. | Top candidate |
| 22 | Adaptive Authority Leasing | Static slot or command leadership is coarse. | [[Mencius]], [[EPaxos]] | Authority leases migrate between leaderless and class-leader modes based on conflict evidence. | Lease epochs intersect with commit quorums. | Time leases cannot be safety assumptions unless quorum-backed. | Promising |
| 23 | Hot-Key Mencius | Rotating coordinators cause gaps but balance load. | [[Mencius]], [[EPaxos]] | Allocate hot conflict classes to rotating owners while cold commands remain leaderless. | Owner-only non-null proposal per owned class-slot. | Requires robust hot-key detection. | Promising |
| 24 | Delegated Dependency Coordinator | Command leader sees too little under contention. | [[EPaxos]], [[SwiftPaxos]] | Fast quorum elects a dependency coordinator for a short conflict window. | Coordinator proposes deps, quorum still commits them. | May collapse into SwiftPaxos-like leader inclusion. | Weak |
| 25 | Locality-First Command Leaders | Any replica can lead, but not all are good choices. | [[EPaxos]], [[Atlas]] | Choose command leader by predicted conflict locality rather than client proximity alone. | Leader choice affects latency, not safety. | Mostly scheduling, not a protocol contribution. | Weak |
| 26 | Relay-Backed Multi-Leader Paxos | Relays reduce load but not authority. | [[PigPaxos]], [[Mencius]] | Each slot owner uses relays; relay evidence includes unique voter ids and owner id. | Relay aggregation remains a refinement of owner Paxos. | More engineering than new consensus. | Weak |
| 27 | Conflict-Escalating Rabia | Rabia forfeits ambiguous slots. | [[Rabia]], [[EPaxos]] | Repeated `bottom` for same request triggers a temporary conflict-class leader instead of endless retry. | Weak-MVC agreement remains valid for each slot. | Escalation path may add recovery proof back in. | Promising |
| 28 | Leaderless-To-Leaderful Continuity Certificates | Mode switches are risky. | [[EPaxos]], [[SwiftPaxos]] | Switch from leaderless fast path to leaderful repair using a certificate summarizing prior dependency evidence. | Leaderful mode must preserve all visible conflicts. | Certificate may need full dependency graph. | Promising |
| 29 | Owner-Authored Dependency Skips | Mencius skip idea for dependency gaps. | [[Mencius]], [[EPaxosStar]] | A command owner can issue a signed skip for its missing dependency edge under strict conditions. | Skip cannot remove a necessary conflict edge. | Unsafe unless ownership of conflict observation is defined. | Reject |
| 30 | Rotating Recovery Coordinators | Stable recovery coordinator can bottleneck. | [[EPaxosStar]], [[Mencius]] | Recovery authority rotates deterministically per command id and round, with quorum-backed takeover. | Higher recovery rounds preserve lower-round possible decisions. | Similar to standard ballots unless conflict-aware. | Weak |
| 31 | Bounded Dependency SMR | Dependency metadata grows under contention. | [[EPaxos]], [[Atlas]], [[SwiftPaxos]] | Cap dependency sets by replacing older edges with checkpointed conflict summaries. | Summary implies all omitted ordering constraints. | Summary correctness is hard under concurrent commits. | Top candidate |
| 32 | Dependency Garbage Certificates | Execution waits on old dependency chains. | [[EPaxos-Revisited-2021]], [[Atlas]] | Replicas periodically certify that a dependency frontier can replace explicit ancestors. | Frontier preserves order of all non-commuting committed commands. | Certificate construction may require global coordination. | Promising |
| 33 | SCC-Aware Fast Commit | Commit and execution readiness diverge. | [[EPaxos]], [[Atlas]], [[EPaxosStar]] | Fast predicate estimates whether new deps create large SCCs and falls back early if so. | Fallback cannot change safety, only metadata shape. | Performance heuristic, not safety improvement. | Weak |
| 34 | Typed Conflict Dependencies | All conflicts are treated alike. | [[EPaxos]], [[SwiftPaxos]] | Dependencies carry conflict type such as read-write, write-write, or semantic commutativity class. | Execution order respects non-commuting pairs. | Application semantics must be trusted and stable. | Promising |
| 35 | Commutativity-Credit Consensus | Optional out-of-order commit is underused. | [[Mencius]], [[EPaxos]] | Commands earn credits proving they commute with a certified frontier, reducing required deps. | Credits cannot permit reordered non-commuting commands. | Requires application-provided commutativity oracle. | Promising |
| 36 | Dependency Bloom Fast Path | Dependency sets are large. | [[EPaxos]], [[Atlas]] | Use conservative Bloom filters for fast conflict visibility; false positives add deps, false negatives forbidden. | No missing conflict edge. | Needs exact no-false-negative construction and fetch path. | Weak |
| 37 | Path-Compressed Swift Dependencies | Dependency paths are richer than direct deps. | [[SwiftPaxos]] | Commit path certificates that compress repeated dependency prefixes across commands. | Acyclic committed dependency graph. | Compression may hide cycles unless validated. | Promising |
| 38 | Dependency Debt Scheduler | Protocol commits now, execution pays later. | [[EPaxos-Revisited-2021]], [[Atlas]] | Treat unresolved dependency chains as debt and throttle fast commits that increase predicted debt. | Throttling does not affect safety. | Mainly evaluation policy; novelty limited. | Weak |
| 39 | Conflict-Window Consensus | Long-lived dependencies are too broad. | [[EPaxos]], [[Atlas]] | Dependencies only reference commands within quorum-certified conflict windows; older conflicts collapse into barriers. | Barriers preserve total order across windows. | Barrier formation could be expensive. | Promising |
| 40 | Partial-Order To Total-Order Converter | Dependency protocols defer order too long. | [[EPaxos]], [[SwiftPaxos]] | When SCCs exceed a threshold, convert the component into a mini total-order consensus instance. | Component decision must be compatible with existing deps. | Embedded consensus may duplicate work. | Promising |
| 41 | Quorum Mode Switch Paxos | Static quorum sizes overpay or block. | [[FastPaxos]], [[EPaxosStar]], [[Atlas]] | Maintain several quorum modes and require an explicit intersection certificate when changing modes. | Decisions across modes intersect through a continuity certificate. | Algebra may be complex; liveness during mode changes. | Top candidate |
| 42 | Healthy-Period Fast Quorums | Worst-case fast quorums are large. | [[EPaxosStar]], [[FastPaxos]] | Use smaller fast quorums only inside a quorum-certified low-failure epoch. | Epoch certificate intersects all recovery quorums. | Must not rely on failure detectors for safety. | Promising |
| 43 | Regional Outage Atlas | Atlas blocks above configured `f`. | [[Atlas]] | Dynamically reduce geographic spread and switch to a lower-latency partition-safe mode with explicit availability loss. | Safety across unavailable regions through epoch fencing. | May sacrifice liveness for minority regions. | Promising |
| 44 | Flexible Dependency Quorums | Different commands need different quorum shapes. | [[Atlas]], [[EPaxosStar]] | Choose fast quorum size from conflict/failure class, then record the chosen algebra in the certificate. | Recovery derives intersections from recorded class. | Per-command algebra increases proof surface. | Promising |
| 45 | Read-Optimized SMR Quorums | Pando optimizes storage reads, SMR less so. | [[Pando]], [[Atlas]] | Add read-only command certificates that use smaller discovery quorums plus write-back when state is ambiguous. | Reads return linearizable state. | Needs integration with command log order. | Promising |
| 46 | Split Recovery Quorums | Recovery does multiple jobs with one quorum. | [[EPaxosStar]], [[Pando]] | Use separate quorums for value identity, dependency reconstruction, and liveness handoff. | Combined evidence implies one safe value/dependency object. | Intersection matrix may be hard to state. | Promising |
| 47 | Quorum Algebra Synthesizer Protocol | Quorum formulas are manually derived. | [[quorum-intersection]] | Treat quorum choice as a generated proof artifact consumed by the protocol implementation. | Generated quorums satisfy stated intersections. | Tooling contribution, not protocol unless integrated. | Weak |
| 48 | Failure-Budget Negotiated Commands | `e` and `f` are global in EPaxos*. | [[EPaxosStar]] | Clients or replicas request fast-path failure budget per command class. | Budget certificate binds quorum sizes for that command. | Users may select unsafe or unavailable budgets. | Promising |
| 49 | Weighted Fast Recovery | Static sites are not equal in WANs. | [[Atlas]], [[Pando]] | Use weighted quorums where recovery weight corresponds to site reliability or coded split availability. | Weighted intersections reconstruct evidence. | Needs exact weighted quorum theorem. | Promising |
| 50 | Quorum Continuity Log | Reconfiguration and mode changes need history. | [[Mencius]], [[Atlas]] | Every quorum-mode decision appends a continuity record to a small meta-log. | Data decisions cite a continuity record that intersects prior modes. | Meta-log becomes a leader bottleneck unless sharded. | Promising |
| 51 | Evidence-Summarizing Relays | Relays are proof-transparent. | [[PigPaxos]], [[EPaxos]] | Relays aggregate not only votes but minimal safe summaries such as dependency frequencies. | Summary expands to unique replica evidence. | Relay equivocation must be detectable. | Promising |
| 52 | Randomized Relay Recovery | Relay failures cause retries. | [[PigPaxos]] | Recovery coordinators choose random evidence relays to collect Phase 1 faster under partial failures. | Relays do not alter quorum requirements. | Mostly transport-level improvement. | Weak |
| 53 | Relay-Coded Acceptances | Leader load and evidence size remain high. | [[PigPaxos]], [[Pando]] | Relay groups erasure-code acceptance evidence so the leader reconstructs a majority proof from fragments. | Reconstructed proof identifies unique voters. | Coded vote identity is subtle. | Weak |
| 54 | Overlay-Aware Fast Quorums | Fast quorum placement ignores communication topology. | [[Atlas]], [[PigPaxos]] | Pick fast quorums through relay groups to minimize WAN load while preserving algebra. | Selected quorum still satisfies protocol intersections. | Optimization, not new safety mechanism. | Weak |
| 55 | Relay-As-Witness, Not Voter | Need durable summaries without extra votes. | [[PigPaxos]], [[FastPaxos]] | Relays witness message dissemination and can prove equivocation or omission, but never count toward quorum size. | Voter set remains unchanged. | Witness assumptions may require signatures. | Promising |
| 56 | Hierarchical Dependency Aggregation | Leaderless all-to-all is costly. | [[Rabia]], [[EPaxos]], [[PigPaxos]] | Local relays aggregate dependency observations before global fast decision. | Aggregates represent unique replica observations exactly or conservatively. | Aggregation delay may erase fast-path benefit. | Promising |
| 57 | Relay-Selected Command Leaders | Choosing leaders by client site can be poor. | [[PigPaxos]], [[EPaxos]] | Relays nominate the replica with best observed reachability as command leader for a window. | Nomination affects performance only unless quorum-certified. | Failure-detector leakage into safety. | Weak |
| 58 | Proof-Carrying Aggregates | Aggregates are opaque in many models. | [[PigPaxos]] | Every aggregate includes a proof of unique voters and omitted voter status. | Paxos majority remains over unique acceptors. | Certificate overhead may dominate. | Promising |
| 59 | Relay-Buffered Fast Retries | Failed fast path loses partial work. | [[EPaxos]], [[PigPaxos]] | Relays buffer partial PreAccept/FastAck evidence for immediate retry or recovery. | Buffered evidence cannot outlive its ballot/epoch unsafely. | Stale buffers could confuse recovery. | Promising |
| 60 | Topology-Adaptive Mencius Relays | Rotating owners face WAN asymmetry. | [[Mencius]], [[PigPaxos]] | Owners use topology-aware relay trees while preserving owner-only non-null values. | Relay layer refines the same simple consensus instance. | Engineering dominated unless proof of tail reduction is clear. | Weak |
| 61 | Ambiguity Credit Slots | `bottom` and `Nop` waste log space. | [[Rabia]], [[EPaxosStar]] | Null slots produce credits that prioritize or constrain retried commands. | Credits cannot force an unsafe later decision. | Credit semantics may become hidden state. | Promising |
| 62 | Batch Forfeit Consensus | Ambiguous individual slots could be handled together. | [[Rabia]] | Decide `bottom` for a batch only when no majority-supported ordering exists, then reshuffle batch requests. | Agreement on each slot or batch position. | Larger forfeits hurt latency under mild contention. | Weak |
| 63 | Constraint-Carrying Bottom | Null slots can preserve conflict information. | [[Rabia]], [[EPaxos]] | A `bottom` decision carries a set of observed conflicting candidates as future scheduling constraints. | Constraints cannot imply a decided command. | Validity and log compaction become complex. | Promising |
| 64 | Probabilistic Dependency Recovery | Randomization avoids failover in Rabia. | [[Rabia]], [[EPaxosStar]] | Use common coin to break recovery cycles among conflicting dependency recoveries. | Random choice must select only validated safe candidates. | Randomness cannot repair unsafe candidate sets. | Promising |
| 65 | Weak-Validity Fast Paxos | Collision recovery may be expensive. | [[FastPaxos]], [[Rabia]] | Fast Paxos instances may decide a typed null on collision and retry values in later slots. | Agreement allows null but preserves proposed-value validity for concrete slots. | Changes consensus validity; may be unacceptable. | Weak |
| 66 | Retry-Stable Request Identity | Retried requests appear across protocols. | [[Rabia]], [[EPaxosStar]] | Make request identity and retry lineage first-class so null/Nop recovery cannot duplicate effects. | At-most-once execution for client request ids. | Mostly SMR integration rather than consensus core. | Promising |
| 67 | Ambiguity-Aware Client Routing | Proposal alignment matters. | [[Rabia]], [[EPaxos-Revisited-2021]] | Route clients to reduce simultaneous conflicting proposal heads, with protocol-visible but non-safety hints. | Routing hints never decide values. | Scheduling heuristic, not robust protocol idea. | Weak |
| 68 | Null Density Controller | Rabia log density varies. | [[Rabia]] | Dynamically switch between randomized weak slots and leaderful slots when `bottom` density rises. | Mode switch preserves slot agreement and request uniqueness. | Adds leader machinery Rabia avoids. | Promising |
| 69 | Recoverable Forfeit Certificates | Nop decisions need explanation. | [[EPaxosStar]], [[Rabia]] | Null/Nop decisions include the evidence that made concrete recovery unsafe. | Later audit agrees null was safe. | Certificate may be large. | Promising |
| 70 | Partial Progress Slots | Slot output need not be command or null. | [[Rabia]], [[EPaxos]] | Slot can decide a small ordering fact, such as "A before B", when command choice is ambiguous. | Ordering facts are acyclic and respected by later slots. | Validity semantics become nonstandard. | Promising |
| 71 | Timing-Hinted Fast Admission | Timing affects performance but not safety. | [[EPaxosStar]], [[Rabia]] | Use delay-bound hints to decide whether to attempt fast path, while safety evidence remains quorum-only. | Timers never justify commit. | Performance-only unless strongly evaluated. | Weak |
| 72 | Discardable Clock Dependencies | Clocks can mitigate conflict timing but are risky. | [[EPaxos-Revisited-2021]], [[EPaxosStar]] | Attach clock order as a discardable dependency hint; quorum evidence must validate or ignore it. | Unsafe clock order cannot remove real conflict deps. | Clock hints may be mistaken for safety evidence. | Promising |
| 73 | Stabilization-Epoch Fast Path | Fast path works after network stabilizes. | [[FastPaxos]], [[Rabia]], [[SwiftPaxos]] | Quorum-certify a stabilization epoch that enables cheaper fast admission but not smaller safety intersections. | Epoch affects only performance predicates. | May be too weak to matter. | Weak |
| 74 | Latency-Class Quorums | WAN links have diverse delays. | [[Atlas]], [[Pando]] | Commands choose quorum sets by latency class and record class in certificate. | Class-specific quorums satisfy intersection formulas. | Could starve high-latency sites. | Promising |
| 75 | Timeout-Free Slow Trigger | Timeout heuristics cause unnecessary recovery. | [[PigPaxos]], [[EPaxos]] | Trigger fallback based on explicit negative or mismatch evidence instead of elapsed time when possible. | Absence of timeout evidence cannot block forever. | Need liveness escape hatch. | Promising |
| 76 | Synchronous-Window EPaxos | EPaxos* has explicit fast-run timing theorem. | [[EPaxosStar]] | Detect windows where the `e`-fast assumptions likely hold and batch conflict-free fast attempts there. | Detection is performance-only; safety remains asynchronous. | Mostly batching policy. | Weak |
| 77 | Timing-Annotated Proof Obligations | Models often mix latency and safety. | [[timing-assumptions]], [[proof-techniques]] | Protocol messages carry annotations classifying evidence as safety, liveness, or performance. | Only safety evidence enters commit predicates. | Tooling/meta-protocol, not direct SMR design. | Weak |
| 78 | Delay-Adaptive Recovery Priority | Recovery order affects tail latency. | [[EPaxosStar]], [[Atlas]] | Prioritize recovery of commands whose dependencies block many ready commands, using measured delays as hints. | Priority does not change selected safe value. | Scheduling only, but useful for evaluation. | Weak |
| 79 | Stable-Leader Escape Hatch | Leaderless tails can be bad under contention. | [[EPaxos-Revisited-2021]], [[SwiftPaxos]] | Temporarily install a leader after quorum-certified conflict/tail evidence crosses threshold. | Leader epoch intersects unresolved leaderless evidence. | Complex continuity proof. | Promising |
| 80 | Queue-Alignment Rabia | Rabia fast path likes aligned PQ heads. | [[Rabia]] | Replicas exchange cheap queue-head hints before consensus instances to improve proposal alignment. | Hints cannot suppress valid client requests indefinitely. | Mostly performance tuning. | Weak |
| 81 | Epoch-Carrying Fast Certificates | Static membership is assumed. | [[Atlas]], [[EPaxosStar]], [[SwiftPaxos]] | Every fast certificate includes membership epoch and quorum-continuity proof reference. | Certificates from adjacent epochs intersect safely. | Needs reconfiguration theory not yet ingested. | Promising |
| 82 | Reconfigurable Dependency Frontiers | Dependency graphs cross membership changes. | [[EPaxos]], [[Atlas]] | Reconfiguration commits a dependency frontier that new epoch must preserve before executing later commands. | Frontier orders all prior conflicting committed commands. | Frontier may be large. | Promising |
| 83 | Membership-Aware Recovery Quorums | Recovery quorum `n - f` changes with epoch. | [[EPaxosStar]], [[Atlas]] | Recoverer collects evidence from old and new epoch quorums according to a recorded transition algebra. | Possible old decisions remain recoverable. | Intersection conditions may be onerous. | Promising |
| 84 | Relay-Assisted Reconfiguration | Changing relay groups is separate from membership. | [[PigPaxos]] | Relays help disseminate and audit membership changes while votes remain replica-based. | Relay proof is transport refinement only. | Does not solve core quorum continuity. | Weak |
| 85 | Randomized Reconfiguration Slots | Rabia claims simpler reconfiguration direction. | [[Rabia]] | Use Weak-MVC slots to decide membership changes, allowing `bottom` for ambiguous changes. | Membership change slots are totally ordered with commands. | Weak validity for config changes may reduce availability. | Promising |
| 86 | Coded State Transfer Certificates | Reconfiguration needs state transfer. | [[Pando]], [[Rabia]] | New members reconstruct state from erasure-coded certified checkpoints tied to log certificates. | Checkpoint identity matches decided log prefix. | Storage protocol and SMR proof must compose. | Promising |
| 87 | Failure-Domain Epochs | Failure budgets are deployment-specific. | [[Atlas]], [[Pando]] | Epochs encode failure domains and quorum constraints, not only member sets. | Quorum intersections respect domain weights. | Operational complexity. | Promising |
| 88 | Rolling Fast-Quorum Replacement | Fast quorums are static within a command. | [[FastPaxos]], [[SwiftPaxos]] | Replace fast-quorum members gradually with overlap certificates instead of stopping fast path. | Replacement preserves fast-fast and fast-recovery intersections. | High proof complexity. | Weak |
| 89 | Reconfiguration As Dependency | Membership changes conflict with all commands. | [[EPaxos]], [[Mencius]] | Treat config changes as commands with universal conflict dependencies and special execution barriers. | No command executes across config boundary without barrier. | Conservative, may be known and slow. | Weak |
| 90 | Configuration-Leased Conflict Classes | Shard authority and membership together. | [[Mencius]], [[EPaxos]] | Each conflict class has a config epoch and leader/leaderless mode, changed by quorum certificate. | Class epochs compose into one serializable SMR history. | Cross-class transactions are hard. | Promising |
| 91 | Evidence Algebra SMR | Proof objects are bespoke. | [[proof-techniques]] | Build SMR protocol around reusable evidence operations: choose, validate, reconstruct, forfeit, refine. | Algebra laws imply agreement and recoverability. | Too abstract unless instantiated. | Top candidate |
| 92 | Adopt-Commit Dependency Layer | Fast evidence resembles adopt-commit. | [[adopt-commit-abstraction]], [[EPaxosStar]] | Express dependency fast path as adopt-commit: commit if unambiguous, adopt if recoverable. | Adopted value must be safe for later commit. | Need exact abstraction page expansion. | Promising |
| 93 | Refinement-First PigPaxos | Transport and consensus proof separation is useful. | [[PigPaxos]], [[proof-techniques]] | Design overlays by first proving refinement to an abstract vote-collector automaton. | Abstract majority proof unchanged. | Methodology more than protocol. | Weak |
| 94 | Unified Null Semantics | `Nop`, `SKIP`, and `bottom` differ. | [[Mencius]], [[EPaxosStar]], [[Rabia]] | Define typed empty outcomes with explicit validity and retry semantics. | Empty outcomes cannot hide conflicting concrete decisions. | May blur protocol-specific assumptions. | Promising |
| 95 | Dependency Visibility Logic | Visibility proofs recur. | [[EPaxosStar]], [[Atlas]], [[SwiftPaxos]] | Make cross-command visibility a first-class logic independent of per-instance agreement. | Agreement plus visibility implies deterministic execution. | Logic must cover SCCs, paths, and unions. | Top candidate |
| 96 | Recovery Counterexample Generator | Bugs hide in recovery. | [[EPaxosStar]], [[rocq-modeling-notes]] | Protocol design includes a small model that searches for ambiguous recovery evidence before proof. | No accepted candidate lacks a recovery search pass. | Tooling, not a protocol mechanism. | Promising |
| 97 | Certificate Normal Form | Certificates differ across protocols. | [[FastPaxos]], [[Pando]], [[Rabia]] | Normalize evidence into fields: electorate, epoch, object identity, payload, reconstruction rule, fallback. | Normal form preserves original protocol safety semantics. | Could oversimplify distinct mechanisms. | Promising |
| 98 | Proof-Carrying Quorum Changes | Quorum formulas are easy to miscopy. | [[quorum-intersection]] | Quorum changes carry machine-checkable intersection lemmas with the certificate. | Runtime accepts only certificates whose lemmas match policy. | Runtime proof checking may be heavy. | Weak |
| 99 | Visibility-Preserving Compression | Formal models struggle with metadata growth. | [[EPaxos-Revisited-2021]], [[Atlas]] | Compress dependency histories only via transformations proven to preserve visibility. | Compression is a simulation relation on executions. | Finding useful transformations may be hard. | Promising |
| 100 | Protocol Kernel With Pluggable Evidence | Ideas combine dimensions unsafely. | All ingested protocols | Define a small SMR kernel parameterized by evidence type, quorum algebra, recovery rule, and execution relation. | Kernel theorem: valid plugin implies agreement, validity, recoverability, and deterministic execution. | Very ambitious; plugin obligations may be as hard as full proofs. | Top candidate |

## Candidate Ranking

| Idea | Novelty | Structural fit | Proof tractability | Evaluation clarity | Risk | Overall |
|---|---:|---:|---:|---:|---:|---:|
| Recoverable-Fast-Union Paxos | 4 | 5 | 3 | 5 | 3 | 5 |
| Recovery-Native Paxos | 5 | 5 | 3 | 3 | 4 | 4 |
| Monotone Recovery Lattice | 5 | 5 | 4 | 3 | 3 | 5 |
| Conflict-Class Leaders | 4 | 5 | 3 | 5 | 3 | 5 |
| Bounded Dependency SMR | 4 | 5 | 3 | 5 | 4 | 4 |
| Quorum Mode Switch Paxos | 5 | 4 | 3 | 4 | 4 | 4 |
| Evidence Algebra SMR | 5 | 5 | 3 | 3 | 4 | 5 |
| Dependency Visibility Logic | 4 | 5 | 4 | 4 | 3 | 5 |
| Protocol Kernel With Pluggable Evidence | 5 | 5 | 2 | 3 | 5 | 4 |
| Nop-With-Reason SMR | 4 | 4 | 4 | 4 | 3 | 4 |
| Read-Optimized SMR Quorums | 4 | 4 | 3 | 5 | 4 | 4 |
| Conflict-Escalating Rabia | 4 | 4 | 3 | 4 | 4 | 4 |
| Epoch-Carrying Fast Certificates | 4 | 4 | 3 | 3 | 4 | 4 |
| Unified Null Semantics | 3 | 5 | 4 | 3 | 3 | 4 |

## Most Researchable Next Steps

### Recoverable-Fast-Union Paxos

**Next step:** Write the exact quorum inequality that makes a dependency union reconstructible from every recovery quorum, starting from the [[Atlas]] proof shape.

**Counterexample to search for:** Two conflicting commands whose unions each satisfy local frequency thresholds but neither dependency edge is reconstructible after `f` failures.

**Artifact:** Small TLA+/Rocq model with command ids, dependency reports, fast quorums, and recovery quorums.

### Monotone Recovery Lattice

**Next step:** Define a lattice with elements for accepted slow proposal, reconstructible fast proposal, validated fast proposal, and null.

**Counterexample to search for:** A recovery quorum where two incomparable fast proposals both appear recoverable.

**Artifact:** Paper note proving that the join operation is deterministic and safe under the stated quorum intersections.

### Conflict-Class Leaders

**Next step:** Formalize conflict-class epochs and the certificate that moves a key/class from leaderless mode to class-leader mode.

**Counterexample to search for:** Two replicas classify the same pair of conflicting commands into different classes and commit without a dependency edge.

**Artifact:** Simulator extension using EPaxos-like workloads to compare tail execution latency against class-leader escalation.

### Bounded Dependency SMR

**Next step:** Define a frontier certificate that can replace explicit dependencies without losing visibility.

**Counterexample to search for:** A compressed frontier that hides a non-commuting command needed for deterministic execution.

**Artifact:** Graph model over committed dependency DAGs/SCCs with compression simulation lemma.

### Quorum Mode Switch Paxos

**Next step:** Specify two modes, such as normal fast mode and regional-outage mode, plus a continuity certificate between their quorums.

**Counterexample to search for:** A value chosen in old mode and an incompatible value chosen in new mode with disjoint effective evidence.

**Artifact:** Algebra table in [[quorum-intersection]] style for mode-specific fast, slow, and recovery quorums.

### Evidence Algebra SMR

**Next step:** Define abstract evidence operations `choose`, `reconstruct`, `validate`, `forfeit`, and `refine`.

**Counterexample to search for:** An algebra instance that satisfies local operation laws but violates cross-command visibility.

**Artifact:** Reusable proof skeleton connecting per-instance agreement to dependency visibility and deterministic execution.

### Dependency Visibility Logic

**Next step:** Separate per-command agreement from cross-command visibility and prove the latter for EPaxos*, Atlas, and SwiftPaxos-like mechanisms.

**Counterexample to search for:** Two commands each individually agreed but with no recoverable dependency direction.

**Artifact:** A comparison proof note that normalizes visibility obligations across dependency protocols.

### Protocol Kernel With Pluggable Evidence

**Next step:** Draft the kernel theorem and list plugin obligations for quorum algebra, recovery rule, and execution relation.

**Counterexample to search for:** A plugin that proves per-slot agreement but fails SMR linearizability due to dependency execution ambiguity.

**Artifact:** Minimal abstract SMR kernel page or Rocq module.

## Rejected Or Weak Directions To Keep As Warnings

- Negative evidence is attractive but unsafe unless absence claims are quorum-backed.
- Clock and timing hints should be discardable; they must not become safety evidence.
- Relay aggregation is useful, but granting relays implicit voting power breaks the clean [[PigPaxos]] refinement story.
- Null outcomes need validity semantics; treating `Nop`, `SKIP`, and `bottom` as identical would erase important protocol-specific assumptions.
- Per-command quorum customization is promising only if the certificate records the exact quorum algebra used.

## Related Pages

[[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[recovery-rules]], [[commit-rules]], [[conflict-handling]], [[leader-roles]], [[proof-techniques]], [[quorum-intersection]], [[adopt-commit-abstraction]], [[unresolved-confusions]]
