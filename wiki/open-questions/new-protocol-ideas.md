---
type: research-brainstorm
date: 2026-10-13
status: speculative
candidate_count: 30
tags: [smr, defect-driven, protocol-design, recovery, proof-obligations]
---

# 30 Defect-Driven SMR Protocol Candidates

## Scope and epistemic status

All C001–C030 are **Ideas**, not established protocols. Their predicted benefits are **Hypotheses**. No candidate is proved safe, live, latency-optimal, or novel. Ranking means priority for specification and counterexample search, not endorsement for implementation. No models or performance experiments were run for this brainstorm.

This reread uses the current index, protocol catalog, dimension/family comparisons, protocol mechanisms, proof notes, concept/property pages, and paper-page proof/limitation sections. It preserves source-level uncertainties in [[unresolved-confusions]]. Original PDFs were not re-ingested. Evidence links identify the wiki extraction supporting each defect; inferred bottlenecks and proposed mechanisms are separate below. This is not an external literature or novelty survey.

The corpus covers crash-fault consensus and several replication/ordering boundary cases. [[Pando]] supplies storage-reconstruction lessons, [[Hermes]] supplies per-key readability lessons, and [[Hydra]] supplies ordering/drop lessons; none alone specifies general SMR. No BFT or DAG-consensus candidate inherits correctness from these pages: [[bft-consensus]] and [[dag-based-consensus]] are unpopulated.

## Evidence matrix: recurring defects and causal bottlenecks

The observations in the second column are extracted facts with sources. The causal interpretation and shift columns are **Inferences/Ideas**. A cost is conditional, not a universal flaw in every listed protocol.

| Cluster / candidates | Sourced observation | Inferred causal bottleneck | Candidate paradigm shift |
|---|---|---|---|
| D1: execution debt | EPaxos can commit before transitive dependencies are ready; batches broaden conflicts; Copilot also separates commit from executability. [[EPaxos-Revisited-2021]], “Commit vs. execution latency,” “Dependency-chain bounding,” “Key results”; [[Copilot-2020]], “Commit rule” | Because a command certifies its own order without closing its predecessor obligations, returned-value operations pay execution-tail costs under skew and continuous concurrency. | Agree on closed execution cuts, admission frontiers, or compact ordering constraints. |
| D2: recovery ambiguity | EPaxos* validates potential conflicts; Atlas reconstructs a threshold-supported union; Copilot's ambiguous fast evidence needs another log. [[Making-Democracy-Work-2025]], “Recovery path”; [[Atlas-2020]], “Key proof ideas”; [[Copilot-2020]], “Recovery path”; [[recovery-rules]] | Because partial evidence must distinguish a possible earlier completion from an incomplete attempt, count-only recovery pays validation or recursive inspection under coordinator failure. | Make recovery evidence a first-class commit object, with conservative fallback for ambiguity. |
| D3: quorum/evidence adaptation | FPaxos trades smaller Q2 for larger Q1; WPaxos availability depends on placement; KCensus has fixed core requirements and excludes contention/queueing from its optimality scope. [[Flexible-Paxos-2016]], “Limitations”; [[WPaxos-2020]], “Quorum system”; [[KCensus-2026]], “Limitations,” “Open questions” | Because evidence configurations are optimized for a particular normal workload/topology, protocols pay recovery or adaptation costs when failures, contention, or routes change. | Optimize phase availability and evidence lifetime jointly; activate changes at explicit boundaries. |
| D4: authority transitions | Jetpack must recover promises and order them before conflicting new-view work; OmniPaxos closes the old log with SS; KCensus dynamic-table fencing is unspecified. [[view-change-hazard]]; [[Omni-Paxos-2023]], “Recovery path,” “Open questions”; [[KCensus-2026]], “Open questions” | Because fast evidence belongs to old authority, recovering values alone pays safety risk or global pauses when proposers/configurations change. | Transfer value, ordering obligation, and authority as one epoch-scoped object. |
| D5: fail-slow cost | Copilot duplicates work; Avicenna samples shadow work and targets one slowdown; internal heartbeats can miss client-visible gray faults. [[Copilot-2020]], “Limitations”; [[Avicenna-2026]], “Main idea,” “Limitations”; [[slowdown-tolerance]] | Because crash detection does not identify every latency bottleneck, protocols pay either redundant processing or delayed replacement when a responsive replica becomes slow. | Separate performance policy from durable authority, and target the specific blocked stage. |
| D6: read authority | Bodega writes cover every responder; Hermes invalidates all current live members and depends on leased RM; local reads can block behind uncertainty. [[Bodega-2026]], “Commit rule,” “Limitations”; [[Hermes-2020]], “Commit rule,” “Limitations”; [[linearizability]] | Because local readers must observe or stop serving before every completed write, protocols move read latency into write coverage and authority-removal costs under slow readers. | Use finer serving scopes, explicit fencing, or constrained read semantics. |
| D7: ownership/conflict scope | WPaxos may thrash with moving locality and handles one object; Mencius assigns future slots to all owners; application conflict predicates constrain CURP/Jetpack. [[WPaxos-2020]], “Limitations”; [[Mencius-2008]], “Limitations”; [[CURP-2019]], “Limitations”; [[Jetpack-2026]], “Limitations” | Because routing and authority are fixed at object/slot granularity, protocols pay migrations, empty-slot revocations, or conservative conflicts under changing locality and cross-object work. | Make authority finite, conflict-scoped, and explicit about multi-object atomicity. |
| D8: evidence lifetime | GPaxos needs compact histories; Pando distinguishes chosen identity from reconstructable splits; KSMR restart/GC/dedup details are open; OmniPaxos transfers complete decided state. [[GPaxos]], “Weaknesses”; [[Pando-2020]], “Quorum system”; [[KCensus-2026]], “Open questions”; [[Omni-Paxos-2023]], “Limitations” | Because commit evidence, payload availability, and replay identity have different lifetimes, protocols pay memory, reconstruction, or unsafe compaction risks under long histories and restarts. | Certify the retained recovery summary and payload obligation before deleting evidence. |
| D9: transport/frontier bottlenecks | PigPaxos relays can hide votes until retry; OmniPaxos needs a QC center; Hydra waits for every sequencer frontier and abstracts receiver quorums. [[PigPaxos-2021]], “Limitations”; [[partial-connectivity]]; [[Hydra-2023]], “Limitations” | Because efficient communication introduces intermediary or frontier dependencies, protocols pay tail stalls under partial links, sparse traffic, or lost aggregation. | Keep authoritative evidence end-to-end while changing routes and scope of ordering frontiers. |
| D10: liveness/throughput mismatch | Rabia permits null slots, lacks pipelining in the evaluated implementation, and assumes coin-independent delays; EPaxos* liveness is stated for finite submissions; fast shims add load. [[Rabia-2021]], “Limitations”; [[Making-Democracy-Work-2025]], “Limitations”; [[Jetpack-2026]], “Limitations” | Because slot termination or unloaded fast latency does not guarantee useful request completion, protocols pay holes, starvation, and queueing under persistent load. | Define useful-progress obligations and bound interference while preserving agreement. |

## Baseline contracts used by candidates

These labels abbreviate concrete sourced rules, not transferable proof packages. A candidate invoking a baseline retains its full algorithm, not just its counts. **Any changed commit object, metadata, quorum family, or recovery rule creates a new proof obligation.** Where that obligation is unsolved, the candidate cannot claim safe early completion. A fallback cannot undo an already returned unsafe result.

| Contract | Quorum / commit / recovery rule retained unless explicitly changed | Source |
|---|---|---|
| P: conservative log SMR | Proposed baseline `n = 2f + 1`, majority `f + 1` Phase 1 and Phase 2, one value per slot/ballot, persistent promises/accepted values; higher Phase 1 preserves highest accepted value; replies require deterministic execution of the chosen prefix and replicated deduplication state. This is a design baseline, not a new paper formula. | [[quorum-intersection]], “Flexible Paxos cross-phase shape”; [[commit-rules]]; [[sequence-consensus]] |
| F: fixed flexible families | `∀Q1 ∈ Quorum1, ∀Q2 ∈ Quorum2: Q1 ∩ Q2 ≠ ∅`; simple thresholds `|Q1| + |Q2| > N`; commit all members of a valid Q2; Phase 1 over valid Q1 adopts highest accepted ballot/value. No same-phase intersection assumed. | [[Flexible-Paxos-2016]], §§3–5; [[FPaxos]] |
| E: corrected dependency SMR | `n >= max{2e + f - 1, 2f + 1}`, fast `n - e`, slow/recovery `n - f`; fast matching `initDep`, slow accepts one `(cmd, dep)`; recover maximal accepted evidence or validate potential fast evidence, preserving conflicting-command visibility and deterministic SCC execution. | [[Making-Democracy-Work-2025]]; [[EPaxosStar]] |
| A: recoverable dependency union | Fast `floor(n/2) + f` includes initial coordinator; fast commit `union_Q dep = union^f_Q dep`; slow `f + 1`, recovery `n - f`; retain highest slow accepted proposal before fast reconstruction; retain remembered fast quorum. | [[Atlas-2020]]; [[Atlas]] |
| S: leader-anchored dependency paths | `N = 2f + 1`; slow/recovery majority; every fast quorum includes the ballot leader and `|Q1 intersect Q2| > N/2`; C1 size `> 3/4 N`, or C2 one fixed majority. Preserve matching FastAck paths, leader-proposal SlowAck repair, and NewLeaderAck/Sync path recovery. Stable contention-free response `2δ`, stable general response `3δ` are original SwiftPaxos bounds, not bounds for a modified protocol. | [[SwiftPaxos-2024]], Table 1, Proposition 11; [[SwiftPaxos]] |
| K: frozen knowledge census | `n ≥ 2f+1`, `|Q_p| > f`, pairwise `|W_p ∪ W_q| > f`; commit exact proposer requirement coverage plus own acceptance; freeze `n-f` distinct voters and apply both Algorithm 6 exclusion tests, then adopt into fallback. Acceptance is immutable; requirements fixed per instance. | [[knowledge-requirement]]; [[knowledge-census-recovery]]; [[KCensus-2026]], Algorithms 1–7 |
| J: view-safe shim | `n = 2f + 1`; same-view fast `f + ceil(f/2) + 1` including all host proposers; recovery majority `f + 1`, candidate threshold `ceil(f/2) + 1`; accepted recovery set and host stability marker precede new work; host satisfies PR 1/PR 2. | [[Jetpack-2026]]; [[view-change-hazard]] |
| B: covered local readers | Odd `n`, `m = ceil(n / 2)`; write majority plus every key responder; stable roster has `m` leases and responder catch-up through thresholds from some `m` grantors; Prepare/lease transition preserves prior writes and reader fencing. | [[Bodega-2026]]; [[roster-lease]] |
| H: fenced invalidation | Every member of live set `M_e` acknowledges write/replay; local reads only in `Valid`; replay exact original timestamp/value; RM leases and epochs fence removed members before new-set completion. No substitute majority data quorum. | [[Hermes-2020]]; [[reliable-membership]] |
| U: primary-backup unordered durability | One master plus `f` backups and `f` witnesses; fast completion requires master result plus all `f` witness records; sync fallback to `f` backups; recovery freezes/selects one witness and replays its commuting records with exactly-once filtering. | [[CURP-2019]]; [[CURP]] |
| R: randomized slot agreement | `n ≥ 2f + 1`; each exchange waits for `n-f`; proposal majority `⌊n/2⌋ + 1`, decision `f + 1` votes; retain Weak-MVC value-locking and explicit coin/adversary assumption; execute non-null slots in order. | [[Rabia-2021]]; [[Rabia]]; [[common-coin]] |

For all candidates: non-Byzantine authenticated identities are assumed; votes count distinct replicas, not routes or aggregates. Durable restart and arbitrary network loss require explicit persistence/retransmission contracts where absent in the source. Deterministic designs claim progress only under their stated eventual stability and quorum availability; randomized designs state the coin/adversary assumptions separately. Timers never prove that an unobserved value did not commit.

## Candidate register

The register contains exactly 30 ideas, numbered C001–C030 in research-priority order. C001 is Bodega + SwiftPaxos, placed first at the user's request; its position does not imply a proved safety or performance advantage. The remaining entries retain the strongest mechanism-level directions and several distinct alternatives, rather than low-impact transport or scheduling refinements.

Each entry keeps its source-linked defect, mechanism, intended commit/recovery contract, and first counterexample/evaluation test. D1–D10 refer to the evidence clusters above, not candidate-number ranges. All latency benefits and stronger commit/recovery rules remain hypotheses until proved.

### C001 — Bodega + SwiftPaxos: local reads with 2δ/3δ writes

- **Defect / shift:** D6/D1; SwiftPaxos's distributed speculative reads still require command propagation and quorum path evidence, while Bodega localizes eligible reads but retains ordinary leader-based writes. Combine leader-anchored dependency-path writes with leased per-key serving authority. Sources: [[SwiftPaxos-2024]], “Fast path,” “Slow path”; [[Bodega-2026]], “Fast path,” “Safety argument.”
- **Mechanism:** Retain S write ordering; bind a B-style responder roster to the write ballot. Each responder durably records an affecting write and blocks stale local reads before issuing a coverage ACK. Both fast and slow client completion wait for ordinary S evidence plus coverage from every current responder for the affected key. Eligible local reads use caught-up executed state without a per-read quorum exchange.
- **Contract:** Proposed combined predicate `CompleteWrite(c) = SwiftResponseCertificate(c,b) AND AllResponderCoverage(c,b,roster)`. Coverage does not itself decide the write or permit a tentative local result. Reader activation requires majority-backed unique leases and recovery of a closed dependency/execution cut covering prior possible completions; Bodega's scalar slot threshold and AcceptNote rule cannot simply replace that cut. Recovery preserves S accepted paths, fences old serving authority, then activates the new roster.
- **Attack/test:** Let a Swift write complete without coverage and then read its old value at an authorized responder; also test a covered but unresolved dependency, delayed coverage from an old roster, and leader replacement. Goal: `2δ` contention-free / `3δ` stable-general writes plus no inter-replica messages for eligible reads. These are conditional hypotheses requiring stable responsive responders and piggybackable coverage; reader failure, distant coverage, or lease transitions can add delay. Proof and novelty remain TODO.

#### C001 composition obligations and latency scope

This is the first-priority candidate in the 30-idea register. Its composition obligations and viability assessment remain explicit; the latency targets are conditional research goals.

The design target is **SwiftPaxos write responses at `2δ` in stable contention-free runs and `3δ` in stable general runs, with Bodega-style eligible local reads**. Here `3δ` describes SwiftPaxos's general/slow behavior, not a second fast path. A local read still pays client-to-responder communication; “local” means no inter-replica messages on the eligible read path. Interfering reads may wait or fall back. These source-derived targets are not a theorem for the composition.

1. **Same-ballot write coverage:** every final client write response, including SwiftPaxos's early result-plus-path response, must be gated by each affected key responder's durable receipt and stale-read blocking. Merely broadcasting Propagate to all replicas does not establish that every authorized reader received it before completion. The responder can acknowledge coverage without knowing the final dependency path, but must then keep the key unreadable until sufficient decision/execution evidence resolves the pending write.
2. **Execution-aware local eligibility:** a covered reader cannot return the pending write's tentative value or its old value merely because it sent an ACK. It needs stable roster authority and a state consistent with all earlier completed affecting writes, including required dependency execution. Uncertain same-key work conservatively blocks or redirects the read. Releasing such holds from Swift path evidence needs its own proof; Bodega's `m` AcceptNotes are not directly equivalent.
3. **Cross-ballot catch-up:** Bodega transfers old writes through log-slot thresholds. SwiftPaxos recovers command dependency paths. Define a closed command/evidence cut containing every potentially completed earlier write and all execution prerequisites; verify recovery-majority intersection, cut completeness, and deterministic execution before enabling a new reader. One maximum local slot or timestamp is not a sufficient substitute without a new invariant.
4. **Authority fencing:** bind each write ballot injectively to its responder roster; prohibit new-ballot write completion while incompatible old read-serving authority remains active. Retain bounded-drift lease expiration/revocation and majority-backed roster uniqueness. NewLeaderAck/Sync recovers write paths; it does not by itself revoke old read rights. Old-ballot delayed coverage ACKs cannot satisfy a new-roster completion predicate.
5. **Latency accounting:** coverage may fit in Propagate's return exchange if responders record/block promptly and send coverage ACKs directly to the client or inside already required matching FastAck/SlowAck evidence. Routing coverage through a new sequential leader exchange can lose `2δ`. For C2, placing all key responders inside the fixed fast quorum can simplify fast evidence gathering, but the chosen slow quorum must also cover responders or collect their separate coverage. Placing them inside one fast quorum alone does not protect slow-path writes. Increasing responder count or distance raises actual quorum collection latency even without adding a nominal message round.
6. **Failure/liveness cost:** an unresponsive authorized reader can block both write completion paths until its authority is safely removed. Therefore the original SwiftPaxos `3δ` bound cannot be claimed solely from a healthy slow majority. State the additional stable healthy-roster premise and count lease/reconfiguration delay separately. A healthy-majority roster is the intended eventual escape mechanism, whose composition liveness must be proved.

First artifact: a five-replica model with one leader, one C2 fixed majority, a reader outside the initially selected slow majority, two conflicting writes, one local read, two roster epochs, and delayed messages. Check the client response event itself, not only replica Commit. Kill the unqualified latency claim if safe coverage necessarily adds a sequential exchange or waits for a failed reader's lease. Retain a conditional stable-roster claim only after the above safety obligations and latency schedules are established.

#### C001 viability assessment

**Research judgment / Inference:** plausible as a restricted single-key read/write protocol extension and worth a small formal specification. Confidence is higher in the possibility of a conservative safe design than in retaining useful local-read latency during interfering writes, or in a novel publishable contribution. No model has been checked and no prototype has been measured. The desired combination is not inherently contradicted by the source mechanisms, but their proofs do not compose automatically.

- **Why it is plausible:** both source protocols have a ballot leader. SwiftPaxos already disseminates commands and dependency evidence; a responder's receipt/read-blocking ACK could run in parallel with that dissemination. Bodega's leases protect roster agreement rather than prohibit writes, so the serving-authority premise need not serialize every write. These are observations from [[SwiftPaxos-2024]] and [[Bodega-2026]]; preservation of the combined latency is a hypothesis.
- **Main performance tradeoff:** every authorized responder becomes an additional completion dependency for writes to its key. In an ideal independent parallel schedule, hypothesize `L_combined = max(L_Swift, L_coverage)`; this is not a queueing-aware bound or a source-paper formula. A distant/slow responder may dominate actual WAN latency while the nominal message-delay count stays unchanged. Placing readers inside C2 simplifies coverage but limits which locations can serve locally unless the fixed fast quorum grows or changes.
- **Most important weakness of the present sketch:** blocking local reads on every unresolved affecting write proves neither Bodega-like short holds nor robust performance under continuous writes. It may provide local reads chiefly in quiet periods. A useful next mechanism is a responder-verifiable, irrevocable per-key order/result certificate that releases a read once its selected visible state is safe, without waiting for unrelated or provably later writes. Establishing that certificate requires conflict-order and real-time reasoning; “latest locally received write,” majority receipt alone, and tentative execution are insufficient.
- **Proof burden:** old-ballot catch-up must recover sufficient dependency paths, values, and execution obligations; Bodega's scalar log threshold cannot be transplanted unproved. Local reads omitted from SwiftPaxos command agreement also need their own placement in a linearizable history consistent with all write results. Initially restrict commands to single-key GET/PUT with deterministic effects; arbitrary result-bearing SMR commands and multi-key reads stay outside this first specification.
- **Novelty gate:** simply appending roster leases and responder ACKs is a composition/engineering result. A stronger contribution would establish a compact dependency-aware read-eligibility certificate, safe early release during interference, or a proved joint quorum/responder placement tradeoff. External novelty remains unverified. The Bodega paper's claims of non-intrusive application concern classic consensus, not a demonstrated SwiftPaxos composition.

Recommended first version: one stable ballot, C2 fixed fast quorum, and one non-leader responder per key inside C2. Keep the original Swift certificates and extra responder coverage on **both** completion paths. First prove completed-write/following-read freshness; then add two conflicting writes and interrupted read release; only then introduce leader/roster changes. The expanded counterexample model should also place a responder outside C2. Assess the benefit using read p99 under overlapping writes, write p99, and hold duration at equal offered load, rather than quiet-read latency or nominal `2δ/3δ` alone.

Primary-source cross-check: [SwiftPaxos, NSDI 2024](https://www.usenix.org/system/files/nsdi24-ryabinin.pdf), protocol/latency arguments; [Bodega, OSDI 2026](https://www.usenix.org/system/files/osdi26-hu-guanzhou.pdf), roster coverage and optimistic read holding. These support the starting mechanisms, not the proposed composition or its novelty.

### C002 — Closed-cut SMR

- **Defect / shift:** D1; replace a single command's dependency commit with a finite execution cut that contains every unresolved predecessor. [[dependency]], [[EPaxos-Revisited-2021]].
- **Mechanism:** Any coordinator assembles commands, dependency edges, payloads, and executed-prefix references; epoch fencing prevents later admitted work from becoming an older predecessor of that cut.
- **Contract:** New E variant: quorum accepts one cut and deterministic internal order before reply; recovery must preserve every possibly committed cut, including referenced executed prefixes. New fast predicate is TODO; initially use P to decide cuts.
- **Attack/test:** A concurrent command escapes the cut and reverses a conflict. Model three commands across two cuts with coordinator failure; measure p99 response and cut-building overhead.

### C003 — Continuously loaded recovery windows

- **Defect / shift:** D1/D10; finite-submission liveness does not establish progress with continuous arrivals. [[Making-Democracy-Work-2025]], “Limitations.”
- **Mechanism:** P decides a finite admission frontier for a conflict domain, closes that window to new proposals, and services recovery before opening its successor. Later commands enter the successor window.
- **Contract:** Retain E validation within each window; recovery also checks predecessor-window conflicts. Replies require the normal E executable closure; the frontier alone is not a commit certificate.
- **Attack/test:** An old-window command is admitted after closure or validation waits on unlimited successor traffic. Specify monotone window membership and fair advancement; model an infinite input stream with bounded admitted work.

### C004 — Domain-resilient knowledge requirements

- **Defect / shift:** D3; process-count resilience may poorly describe correlated rack/zone outages. [[WPaxos]], [[KCensus]], [[fault-models]].
- **Mechanism:** Replace the scalar fault set in synthesis with an explicit admissible family `Fset` of failed-domain unions.
- **Contract:** Proposed K extension: for every `F ∈ Fset`, require `Q_p \ F != ∅` and `(W_p ∪ W_q) \ F != ∅`; recovery collects an allowed surviving census. These are proposed generalizations, not paper formulas; termination and all evidence lemmas need reproof.
- **Attack/test:** A legal domain outage erases all distinguishing witnesses despite many live nodes. Enumerate domain cuts and asymmetric survivors; evaluate cost against conservative arbitrary-fault provisioning.

### C005 — Requirement-version slot fencing

- **Defect / shift:** D3/D4; dynamic KSMR requirement installation has no extracted transition-level fencing rule. [[KCensus-2026]], “Open questions.”
- **Mechanism:** P decides `(requirement_hash, first_slot)` for each shard. Every slot permanently binds one table; old slots retain old tables until recovery evidence is retired.
- **Contract:** Within each slot use K unchanged: exact requirement coverage, `n-f` census, both exclusion tests. A proposal carrying a mismatched table is rejected; install decisions and slot binding survive leader change/restart.
- **Attack/test:** Lagging replicas combine old and new evidence in one slot. Model two tables, delayed installation, and one failed proposer; prove single table per slot and measure transition backlog. No live-slot table replacement.

### C006 — Durable proposer-promise records

- **Defect / shift:** D4; early host promises may require a separate recovery set after view change. [[Jetpack]], [[view-change-hazard]].
- **Mechanism:** Before acknowledging a fast command, a proposer persists its receipt position, conflict promise, view, and command identity in a recoverable promise log.
- **Contract:** Retain J superquorum, all-proposer inclusion, and marker activation initially. To shorten recovery, prove the durable records reconstruct both the completed command set and precedence over stale/new conflicts under every majority Prepare.
- **Attack/test:** Durable receipt exists but host buffer order changes after restart. Model persistence and PR 1/PR 2 separately; measure whether fsync costs exceed saved recovery exchanges. Persistent bytes alone do not prove order.

### C007 — Recovery-sufficient compaction certificates

- **Defect / shift:** D8; deleting old history may erase possible decisions or conflict evidence. [[GPaxos]], [[KCensus-2026]], “Open questions.”
- **Mechanism:** Decide a compaction certificate naming the retained state, acceptance/evidence summaries, ordering frontier, and deduplication cut before dropping raw records.
- **Contract:** P decides the certificate only after required durable replicas cover the state cut. K/E-specific summaries must simulate every future recovery test and conflict lookup; P snapshot safety alone does not justify deleting fast evidence.
- **Attack/test:** A late recovery needs an omitted witness fact or unexecuted dependency. Model GC against frozen/accepted records and late messages; prove observational equivalence of compacted recovery. Evaluate retained bytes versus certificate cost.

### C008 — Recovery-sufficient decision envelopes

- **Defect / shift:** D2; commit evidence can leave later recovery unsure which metadata to preserve. [[recoverability]], [[recovery-rules]].
- **Mechanism:** Couple each decision with an envelope containing the value, configuration, quorum provenance, and protocol-specific recovery obligation; optimize the cheapest survivable envelope rather than only early votes.
- **Contract:** Start with P-durable envelopes. Early completion requires a proved survivor predicate for every admissible recovery set; recovery carries any possible envelope decision into a higher ballot, never selecting by mere presence count.
- **Attack/test:** A client completes while all envelope copies are among the failed replicas. Enumerate fault cuts, prove recoverability of every completion, and measure added bytes/delays. Generic envelope safety remains TODO.

### C009 — Quorum-family-aware leader eligibility

- **Defect / shift:** D9/D3; majority connectivity may not predict progress under flexible families. [[OmniPaxos]], [[FPaxos]], [[partial-connectivity]].
- **Mechanism:** Define candidate eligibility by reachable valid recovery and replication sets for the installed family; synchronize freshness after election.
- **Contract:** F Phase 1/2 and highest-accepted selection unchanged; election needs a new stable-convergence argument. Candidate connectivity reports are hints until actual Prepare succeeds; all historical families remain recoverable.
- **Attack/test:** Different eligible centers repeatedly preempt via one intermediary. Model chained election with asymmetric families and stale logs; prove eventual leadership under explicit stable topology assumptions. OmniPaxos's QC theorem does not transfer automatically.

### C010 — Read-serving cohorts independent of voters

- **Defect / shift:** D6; all replicas may be readers even though only some need local read locality. [[Hermes]], [[Bodega]].
- **Mechanism:** Separate replicas that store/vote from replicas currently authorized to serve reads, with explicit per-key cohort membership.
- **Contract:** New protocol: P chooses writes plus acknowledgment/invalidation from every authorized reader before completion. P decides cohort changes with old-reader fencing and new-reader catch-up. H write-all correctness does not transfer to arbitrary majority writes.
- **Attack/test:** A storage-only replica silently starts serving after restart. Model role persistence and epoch activation; compare slow-voter tolerance against cohort write coverage. Initial scope uses leased or fully acknowledged fencing.

### C011 — Separate decision and payload obligations

- **Defect / shift:** D8; knowing a chosen value ID does not imply available command data. [[Pando]], [[recoverability]].
- **Mechanism:** Decide a command descriptor plus a payload-availability certificate, with separately placed small metadata and coded payload holders.
- **Contract:** Initial P chooses the descriptor; completion also requires enough acknowledged distinct fragments to reconstruct after every allowed failure. Proposed threshold `k+f` distinct one-fragment holders leaves `k` after `f` crashes, only under that explicit placement model. Recovery fetches the exact certified payload before execution.
- **Attack/test:** Descriptor is durable but fragments belong to different values or correlated domains. Model matching IDs/fragment indices and fault cuts; evaluate metadata locality versus fetch latency. No direct Pando-to-SMR proof transfer.

### C012 — Joint recovery of conflicting commands

- **Defect / shift:** D2; independent validation can form mutually waiting recoveries. [[EPaxosStar]], “Liveness intuition.”
- **Mechanism:** Recover a finite mutually blocking command set and its ordering obligations as one recovery transaction rather than resolving each independently.
- **Contract:** New E variant requires `n-f` evidence per included identifier and one P decision over a jointly safe assignment. Preserve all previously possible payload/dependency decisions; external potentially committing conflicts still need validation.
- **Attack/test:** Separate recovery transactions overlap or omit a third conflicting command. Model two overlapping sets with accepted/fast-only evidence; compare wait rounds, state-space size, and payload transfer. Joint safety is TODO.

### C013 — Resource-bounded decision-evidence synthesis

- **Defect / shift:** D10/D3/D8; link-latency optimal evidence may be too expensive to process or retain at scale. [[KCensus-2026]], “Limitations”; [[SwiftPaxos]], [[Rabia]].
- **Mechanism:** Search legal evidence strategies with explicit constraints on messages, witness facts, decode cost, recovery bytes, and admitted unresolved work; choose a Pareto frontier rather than one latency optimum.
- **Contract:** K/F safety predicates remain hard constraints; source-specific recovery must remain reconstructible under the memory bound, and C005/C019 fences installations. Approximate optimizer outputs require exact legality validation.
- **Attack/test:** Solver hides processing costs inside an idealized relay or drops facts needed by recovery. Check small-instance strategies against a complete oracle, then evaluate held-out loaded traces. Resource-optimality is a research question, not claimed from KCensus's theorem.

### C014 — Certified conflict lanes

- **Defect / shift:** D1; repeated hot conflicts expand dependency chains. [[conflict-handling]], [[SwiftPaxos]].
- **Mechanism:** Install a finite-lived serializer for a detected conflict class, converting internal edges into lane-position references; unrelated classes remain distributed.
- **Contract:** P decides lane activation and the entry/exit frontier; E handles external conflicts. A lane command replies only when its position and cross-lane predecessors are executable; recovery retains lane epochs and prior decisions.
- **Attack/test:** A two-key command straddles two lanes and produces inconsistent cross-lane order. Search overlapping class changes; evaluate hot-tail savings against serializer queueing and boundary costs.

### C015 — Closed-SCC decision units

- **Defect / shift:** D1; cycles are discovered after individual decisions, delaying execution. [[EPaxosStar]], [[Copilot]].
- **Mechanism:** Agree on a closed strongly connected component with a canonical internal order and an external predecessor frontier as one unit.
- **Contract:** P decides the complete component initially; recovering an accepted component preserves its members/order. A new dependency fast path must prove that concurrent decisions cannot add an external edge that invalidates component closure.
- **Attack/test:** Overlapping components include the same command with different orders. Search split-SCC executions with two coordinators; price component discovery and duplicate suppression against saved waits.

### C016 — Conflict-exclusion certificates

- **Defect / shift:** D2; absence of a visible conflicting command is not evidence of absence. [[EPaxosStar]], [[knowledge-census-recovery]].
- **Mechanism:** Validators freeze a specified conflict domain/admission frontier and attest that all admitted commands before it are represented in the candidate's order obligations.
- **Contract:** New E recovery variant; require `n-f` fenced validation replies plus ordinary Accept before commit. Certificate includes the frontier and cannot exclude future commands without forcing them after the recovered one.
- **Attack/test:** An unseen pre-accepted conflict commits after certificates are issued. Model two-command delayed messages and false suspicion; quantify freezing cost. Negative certificates without acceptance fencing are rejected.

### C017 — Joint normal/recovery optimization

- **Defect / shift:** D3; normal-path latency optimality excludes recovery duration and contention. [[KCensus-2026]], “Limitations”; [[FPaxos]].
- **Mechanism:** Select fixed K requirements using an objective that prices fast completion, census collection, fallback, and evidence storage under an explicit fault/workload distribution.
- **Contract:** K compatibility and commit/census rules remain hard constraints; install through C005. Costs alter which legal configuration is chosen, never what evidence means.
- **Attack/test:** The optimizer favors a tiny normal set that often freezes. Enumerate failure placements and conflict traces; compare held-out p99 useful-response latency. No claim of global optimality under unknown workloads.

### C018 — Prevalidated alternative requirements

- **Defect / shift:** D3; one fixed fast route can be brittle after slowdown. [[KCensus]], “Quorum requirement.”
- **Mechanism:** Install a bounded menu of per-proposer alternatives, verifying compatibility for every alternative pair, including alternatives of the same proposer.
- **Contract:** K commits under one named alternative; bind alternative identity to value evidence and preserve recovery tests for every alternative that could enable completion. Never switch a partially accepted value to an unrecorded requirement.
- **Attack/test:** Recovery excludes a value under one alternative although it committed under another. Model concurrent attempts and census selection; prove alternative-aware exclusion. Source already permits multiple alternatives, so contribution needs a recovery/performance extension.

### C019 — Ballot-scoped flexible-family declarations

- **Defect / shift:** D3; local Q2 changes can outrun later Q1 knowledge. [[Flexible-Paxos-2016]], “Open questions”; [[flexible-quorum]].
- **Mechanism:** A ballot declares a finite allowed Q2 family and a recovery rule that intersects every family still capable of completing an earlier ballot.
- **Contract:** F commit is membership in the declared family. Any later Prepare must use a Q1 certified to intersect all unretired prior Q2 families; declaration updates require a P/fenced control log, not a local latency estimate.
- **Attack/test:** A Q2 declaration survives only at nodes absent from new recovery. Model lost declarations and late accepts; prove recoverable family history. Initial conservative majority declarations may erase the hoped-for gain.

### C020 — Finite-slot acceptor cohorts

- **Defect / shift:** D3/D5; restricted acceptors speed rotation but can reduce immediate availability. [[Avicenna]], [[FPaxos]].
- **Mechanism:** Grant a cohort authority over a finite slot interval, with successor evidence deposited before opening the next interval. Full-majority cohorts remain an escape route.
- **Contract:** New protocol must make every takeover set intersect every deciding set for unclosed intervals; preserve phase-tagged values and fence late accepts. Initially P decides cohort transitions and commands; smaller takeover rules stay TODO.
- **Attack/test:** Two cohorts each finish a boundary slot after a delayed handoff. Model simultaneous rotation and two crashes; evaluate takeover latency against cohort-transition work.

### C021 — Cut-through state migration

- **Defect / shift:** D4/D8; complete decided-log transfer delays new configuration startup. [[OmniPaxos]], [[reconfiguration]].
- **Mechanism:** Old configuration decides SS and a state cut; independent migration workers transfer a certified snapshot plus post-cut decided suffix from any reachable holders.
- **Contract:** P/Sequence Paxos decides the stop and snapshot descriptor; new members start only after validating complete state through SS and inherited deduplication state. No old post-SS decision is allowed. This substitutes a specified state representation for complete raw-log transfer.
- **Attack/test:** Snapshot reflects an unchosen suffix or omits a last decided entry. Model chunks racing SS and source failure; compare startup delay and bytes. Snapshot refinement must preserve all future-observable state.

### C022 — Multi-ledger authority rendezvous

- **Defect / shift:** D4/D7; independently moving ownership, read rosters, and evidence tables can expose mismatched epochs. [[reconfiguration]], [[roster-lease]], [[object-stealing]].
- **Mechanism:** A P-decided activation record names one compatible tuple of owner epoch, reader roster, evidence profile, and state cut.
- **Contract:** Operations use exactly one installed tuple; recovery fetches the tuple before applying its F/B/P selection rule. Old authority is fenced and unfinished operations remain bound to their source tuple.
- **Attack/test:** New owner writes through a profile that omits still-authorized old readers. Model one transition interrupted at each layer; quantify activation wait and tuple-history growth. This requires composition proofs rather than borrowing each layer's theorem independently.

### C023 — Several shadow candidates, one real log

- **Defect / shift:** D5; one shadow candidate may itself be slow. [[Avicenna]], “Open questions.”
- **Mechanism:** Maintain a bounded set of independent counterfactual candidates, choose a successor from paired end-to-end evidence, and activate exactly one through a decided/fenced phase transition.
- **Contract:** Conservative P recovery initially replaces Avicenna's fixed-successor two-log shortcut. A cheaper rotation needs new intersection proofs for every allowed successor/acceptor cohort; shadow logs never execute.
- **Attack/test:** Different replicas activate different winning candidates. Model leader choice under inconsistent samples and two slow replicas; compare coverage against duplicate traffic. Number of shadows alone does not establish multi-slowdown tolerance.

### C024 — Snapshot-scoped multi-key read rosters

- **Defect / shift:** D6/D7; independent local-read authority does not certify an atomic multi-key snapshot. [[Bodega-2026]], “Limitations”; [[Hermes-2020]], “Limitations.”
- **Mechanism:** Grant a responder authority for a snapshot frontier and key set, backed by version retention and complete cross-key commit evidence.
- **Contract:** Initial P orders multi-key updates and the read's frontier; responder returns versions from that frontier only after catch-up/authority checks. Extending to independent object logs requires a new consistent-cut and real-time proof.
- **Attack/test:** Read combines versions on opposite sides of one transaction. Model two keys with delayed update visibility; measure version storage and roster coverage. Explicit transaction extension, not inherited per-key linearizability.

### C025 — Finite authority ranges

- **Defect / shift:** D7; Mencius owners control infinitely many future slots that later require revocation. [[Mencius]].
- **Mechanism:** P grants an owner a finite slot range; unallocated future ranges carry no owner authority. Allocate successor ranges according to current load/connectivity.
- **Contract:** P decides range ownership; each slot retains a unique epoch/owner. Owner may skip only within its decided range and only under the simple-consensus restriction. Recovery of unresolved allocated slots uses ordinary quorum evidence.
- **Attack/test:** Old and new range grants overlap after leader failure. Model delayed grants/skips at a boundary; compare revoked-slot work and allocation overhead. No skip certificate is synthesized from a timeout.

### C026 — Conflict-class ownership

- **Defect / shift:** D7; object boundaries may be too coarse for independent operations or too narrow for cross-object conflicts. [[WPaxos]], [[conflict]], [[GPaxos]].
- **Mechanism:** Own a declared conflict class rather than one key; independent classes proceed without a global log, while overlapping classes use a common barrier/order rule.
- **Contract:** P initially decides class definitions/owners and serializes overlapping commands. A flexible fast design must prove complete conflict detection, class-epoch agreement, and recoverability of every cross-class order obligation.
- **Attack/test:** Non-transitive conflicts produce overlapping classes without one shared owner. Model three commands where A conflicts B and B conflicts C; price definition metadata and hotspot imbalance. No assumption that conflicts partition naturally.

### C027 — Joint conflict-mode transitions

- **Defect / shift:** D7/D1; switching hot items between distributed and serialized modes can duplicate authority. [[fast-paths]], [[object-stealing]].
- **Mechanism:** P decides a domain mode together with the complete unresolved predecessor frontier; only successor-window commands use the new mode.
- **Contract:** Old E decisions/recovery remain valid and must close before conflicting P-lane commands execute. Reverse transitions retain the serialized prefix as an immutable dependency frontier. No in-place conversion of pre-accepted tuples.
- **Attack/test:** A delayed leaderless command commits after lane activation and is omitted by lane reads. Model both directions and multi-key overlap; measure mode-switch amortization against contention tails.

### C028 — Dependency-summary snapshots

- **Defect / shift:** D8; an application snapshot can lose metadata needed to order future conflicts. [[dependency]], [[EPaxosStar]], [[Atlas]].
- **Mechanism:** Snapshot application state together with per-conflict-domain executed frontiers and unresolved dependency identifiers/status.
- **Contract:** E/A recovery treats executed identifiers as satisfied only through a certified snapshot frontier; all unexecuted possible decisions retain full source evidence. New proposals derive conflicts from the summary without false negatives.
- **Attack/test:** A recovered command references an identifier whose status was summarized incorrectly. Model snapshot cuts through an SCC and late Nop recovery; prove execution equivalence and measure conflict-summary size.

### C029 — Conflict-scoped ordering frontiers

- **Defect / shift:** D9; one slow sequencer frontier can delay traffic unrelated to its activity. [[Hydra]], [[conflict-handling]].
- **Mechanism:** Track the frontier only for sequencers authorized to inject traffic into a particular ordering/conflict domain; disjoint domains advance independently.
- **Contract:** New Hydra-like primitive needs an activation/removal fence for every domain and consistent order for overlapping destinations. SMR still requires P/majority durability; source Hydra frontier proof cannot simply ignore a sequencer.
- **Attack/test:** An excluded sequencer injects an old lower-stamped overlapping message. Model membership change plus multi-group commands; evaluate sparse traffic stalls. Hold until the domain-ordering interface and receiver quorums are specified.

### C030 — Age-certified request service

- **Defect / shift:** D10; slot termination alone does not guarantee any particular request is eventually chosen. [[Rabia]], [[liveness]].
- **Mechanism:** P decides a finite admission sequence/age band, then requires each serving window to retire or explicitly carry forward its oldest live obligations.
- **Contract:** Commands still decide with P/R as specified per slot; recovery preserves admitted IDs and retirement evidence. The progress theorem assumes bounded service work, fair resubmission, and a timely quorum, not merely a clock-based priority.
- **Attack/test:** Fresh requests repeatedly consume every non-null slot while an old admitted request survives indefinitely. Model unbounded input with bounded windows; prove eventual service or construct starvation trace. This intentionally adds admission coordination to leaderless designs.

## Research priority and red-team disposition

The order is a research agenda, not a correctness or novelty ranking. C001 is first by user preference. No counterexample model or performance experiment has yet been run. A conservative P baseline can protect agreement while sacrificing a proposed fast benefit; proving the cheaper rule remains separate work.

| Priority | Candidate | Why investigate | Main proof or performance gate |
|---:|---|---|---|
| 1 | C001 Bodega + SwiftPaxos | Combines low-latency dependency-path writes with local-read authority | Both client write-completion paths cover every reader; dependency-aware read release and cross-ballot catch-up; responsive-roster latency premise |
| 2 | C002 Closed-cut SMR | Changes the decision unit to executable work | A cut cannot omit an earlier unresolved conflict or acquire one after closure |
| 3 | C003 Continuously loaded recovery windows | Addresses finite-submission liveness scope | Every admitted/eligible request progresses without requiring global input quiescence |
| 4 | C004 Domain-resilient knowledge requirements | Models correlated failures using distinguishing witnesses | Every admissible surviving census retains presence and rival-exclusion evidence |
| 5 | C005 Requirement-version slot fencing | Closes a concrete KSMR installation gap | One immutable requirement version per instance across shards, restarts, and late messages |
| 6 | C006 Durable proposer-promise records | Makes early ordering promises recoverable | Persisted receipt/order records really enforce host ordering; stable-path persistence cost |
| 7 | C007 Recovery-sufficient compaction certificates | Treats GC as recovery correctness | Retained state preserves every future census/validation and retry outcome |
| 8 | C008 Recovery-sufficient decision envelopes | Makes protocol-specific recovery obligations explicit | Survival predicates cannot collapse different evidence mechanisms into vote counts |
| 9 | C009 Quorum-family-aware leader eligibility | Relates election topology to phase availability | Stable election convergence and reachable historical Q1/current Q2 |
| 10 | C010 Read-serving cohorts independent of voters | Isolates which replicas impose write-coverage costs | Removed readers are fenced and new readers are caught up before serving |
| 11 | C011 Separate decision and payload obligations | Adds executable-data survival to descriptor agreement | Every completed exact payload remains reconstructable under permitted failures |
| 12 | C012 Joint recovery of conflicting commands | Attacks mutually waiting validation | Overlapping recovery sets preserve every possible old command/dependency decision |
| 13 | C013 Resource-bounded decision-evidence synthesis | Includes CPU, memory, and recovery cost in synthesis | Exact strategy legality and realistic loaded-resource accounting |

C014–C030 are reserve directions with the gates stated in their entries. C029's proposed scoped sequencer frontier is specifically on hold until domain membership, overlapping-message order, and old-stream fencing are proved. C024 is a transaction/snapshot extension and cannot inherit per-key linearizability as atomic multi-key correctness. C017 and C030 emphasize optimization/admission policy; they need a distinct theorem to constitute new consensus contributions.

No candidate permits timeout-based evidence deletion, mixed-view fast certificates, unconstrained quorum changes, local reads at unfenced removed replicas, or rewriting a possibly committed dependency tuple.

### First artifacts and explicit kill criteria

These are proposed next artifacts, not files, proofs, or benchmarks already produced.

1. **C001 — local-reader composition model.** Begin with five replicas, stable C2, single-key GET/PUT, and one non-leader responder inside C2. Check final client write response followed by local read, then add two conflicting writes, a responder outside the slow quorum/C2, and two interrupted roster epochs. Prove every completed write is visible or blocks a stale read at every authorized responder. **Kill** unconditional 2δ/3δ or always-fast-local-read claims if safe coverage, dependency closure, or lease removal adds an unavoidable wait; measure read/write p99 and hold duration under overlapping writes.

2. **C002 — closed-cut state model.** Use E parameters `n=5, f=2, e=1` for the baseline comparison; model three commands, two overlapping cuts, one coordinator failure, and delayed external conflicts. State `Completed(c) => CutChosen(c) AND PredClosureExecuted(c)`. Prove a decided cut cannot acquire an earlier unresolved external conflict after its admission fence. **Kill the fast version** if every sound closure certificate requires the same waits it was intended to remove; keep P cut batching only if its tail benefit is measured.
3. **C003 — infinite-input service abstraction.** Model bounded windows and arbitrary arrivals, with per-command stable recovery coordinators, eventually timely `n-f` quorums, and fair service. Prove every admitted command eventually commits/executes and every eligible queued request is eventually admitted under the declared load assumptions. **Kill** if window closure depends on global input quiescence or successor-window validation can recreate unbounded waits.
4. **C004 — failure-family witness lemma.** For small enumerated process/domain placements, define admissible failed sets and allowed frozen censuses. Prove every census intersects the value-presence and rival-distinguishing sets; retain both K exclusion tests and actual truthful evidence. Then prove termination for the declared availability family. **Kill** any synthesis rule that passes counts while one allowed domain cut removes all distinguishing witnesses.
5. **C005 — two-table installation model.** Generate two individually compatible K requirement tables for the allowed proposer set; model two shards, installation lag, in-flight slots, restart, and delayed old spread messages. State `SlotInstance(s,i) => ExactlyOneRequirementVersion(s,i)` and preserve old-version recovery. **Kill** the proposed boundary if one slot can satisfy a commit from mixed versions or activation requires an unspecified cross-shard atomic event.
6. **C006 — durable promise/host refinement.** Model J at `n=5, f=2`, two conflicting commands, persisted admissions, host batching, and failure between receipt, proposal, and marker. Check both recovered-command presence and precedence over stale conflicts; retain the marker until its removal is justified. **Kill** any shortened recovery if persisted records prove only durability while host order still reverses a fast result. Measure stable-path delay before judging the optimization.
7. **C007 — compaction indistinguishability test.** Start with a small K instance, then repeat for E; pair an uncompacted execution with its compacted abstraction. Preserve answers to every subsequent census/validation query, request retry, and conflict lookup. **Kill** summaries for which indistinguishable retained states require different recovery outcomes. Do not infer agreement preservation from equal application snapshots.
8. **C008 — protocol-specific envelope algebra.** Instantiate separately for K and E; enumerate each legal completion and fault cut, then derive the minimum surviving data/provenance sufficient for that protocol's recovery. **Kill** a universal envelope rule that reduces both to vote counts or needs an acknowledged client to survive. Compare total response cost against the original protocols before generalizing.
9. **C009 — link-level election model.** Use five nodes with a fixed non-majority F family, persistent accepted state, bidirectional partial links, stale candidates, and an intermediary that propagates higher ballots. Check F agreement independently from eventual leader convergence in stable eligible topologies. **Kill** any eligibility definition that can name a leader unable to complete both historical-family Prepare and a current Q2.
10. **C010 — serving-cohort/read-history model.** Use three voters, two authorized readers, two membership epochs, and a completed write followed by a read at a removed reader. Prove `WriteCompleted(w) => EveryAuthorizedReaderCoversOrIsFenced(w)` and cross-epoch serving exclusivity. **Kill** any transition that omits a still-authorized reader merely because voters decided its removal. Evaluate write-tail savings with realistic lease/revocation costs.
11. **C011 — decision/data availability model.** Keep majority-decided descriptors, parameterized reconstruction `k`, distinct fragments, and an explicit admissible fault family. Model losing holders after completion and replacing placement epochs. **Kill** if a completed descriptor lacks its exact executable payload after a permitted failure, even though agreement still holds. Compare large and small payload regimes; no automatic benefit for tiny commands.
12. **C012 — joint-recovery visibility model.** Use three conflicting commands, two overlapping recovery transactions, fast-only evidence, highest-ballot slow evidence, and a coordinator failure. Check one payload/dependency decision per identifier, conflict visibility, and deterministic execution before liveness. **Kill** if a joint transaction changes any possible old decision or relies on every conflict already being discovered.
13. **C013 — exhaustive small synthesis oracle plus loaded replay plan.** Enumerate small valid K strategies, compute exact message/fact budgets, and include full census/fallback evidence. Only after legality validation, simulate processing queues and compare held-out conflict/fault traces at equal offered load. **Kill** any claimed gain that exists only when decode, retention, recovery, or duplicate work is excluded.

## Focused latency roadmap

Use [[latency]]'s sequential, conflict-free, contention-free, and general-run distinctions without changing source formulas or client-placement conventions. K's stable single-proposer optimality does not establish loaded execution-response optimality.

| Goal | Candidate starting points | Obligations to count or prove |
|---|---|---|
| SwiftPaxos writes plus Bodega-style local reads | C001 | Both completion paths cover readers; read eligibility under interference; leases, dependency catch-up, response event, responder failure/distance |
| Reduce time from commit to execution/result | C002, C003, C014, C015, C027 | Closed predecessors/components, admission queue, external conflicts, transition recovery, deterministic execution |
| Recover ambiguous fast evidence | C008, C012, C016 | Exact possible-old-decision preservation, conflict visibility, validation and certificate cost |
| Adapt evidence to topology/failures | C004, C005, C009, C017, C018, C019 | Failure placement, immutable versions, alternative-aware recovery, historical-family intersection, actual phase connectivity |
| Shorten authority/fail-slow interruption | C006, C020, C021, C022, C023 | Durable promises, all takeover evidence, state transfer, consistent role epochs, shadow work |
| Retain executable state cheaply | C007, C011, C028 | Recovery-equivalent summaries, matching reconstructable payloads, unexecuted obligations, request identity |
| Bound useful-progress resource cost | C013, C025, C026, C030 | Finite authority, complete conflict scopes, fair service, retained evidence, queueing and useful requests/s |

**Hypothesis, not a theorem:** universal stable-general 2δ final responses require sufficient ordering/result evidence despite conflicting arrival orders. C014/C026 constrain conflict scope; C029 changes the ordering frontier and remains on hold. None establishes an unchanged-assumption general 2δ protocol. A lower-bound claim needs explicit client placement, result semantics, faults, pre-established authority, and processing assumptions.

## Novelty and proof discipline

- Search each shortlisted mechanism against external literature before using “new protocol” in a paper. Grounding here establishes a corpus-relative defect, not priority of invention. Admission control, certificates, snapshots, finite authority ranges, erasure-coded logs, transactions, read barriers, and replicated deduplication may have substantial prior art.
- Do not transfer Fast Paxos triple-intersection, Atlas threshold-union, KCensus distinguishing-witness, or Jetpack proposer-promise arguments solely because quorum sizes or message flows match. See [[quorum-intersection]], [[knowledge-census-recovery]], [[adopt-commit-abstraction]].
- For every shortlisted specification, write the decided object, exact completion predicate, allowed quorum memberships, persistent state, recovery selection rule, and epoch fence before drawing a fast-path diagram.
- Check per-instance agreement, cross-command conflict visibility/order, payload availability, result/dedup preservation, and real-time order separately. Passing agreement alone is insufficient for result-bearing SMR.
- Name liveness assumptions: available phase quorums, fair admission/recovery, eventual authority stability, state-transfer reachability, bounded work per window, or a coin/adversary contract. Distinguish admitted-request progress from slot progress and overload queueing.
- Keep [[unresolved-confusions]] visible: missing Copilot technical-report recovery, HydraPaxos/NOPaxos gap rules, WPaxos recovery payload, and KSMR installation/restart/GC/read details remain source gaps. Speculative fixes do not resolve what the original authors implemented or proved.

## Recommended research sequence

Start with C001's restricted local-read composition: prove stable-ballot write coverage and read eligibility before adding recovery or pursuing early read release. Track read p99 under overlapping writes, write p99, and hold duration, not only nominal message-delay counts.

Then specify C005/C006's evidence-configuration and durable-promise boundaries; explore C002/C003's execution/liveness contracts and C004's failure-family lemma. Use C007/C011/C028 to retain recovery evidence and executable data explicitly. Optimize only after those contracts are fixed, using C017/C013.

The shared research question remains: what smallest recoverable object closes the obligations required for a real client result? Votes, dependencies, acceptance knowledge, reader authority, payload fragments, and proposer promises serve different obligations.

## Related pages

[[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[recovery-rules]], [[commit-rules]], [[proof-techniques]], [[fault-models]], [[timing-assumptions]], [[latency]], [[leader-roles]], [[conflict-handling]], [[reconfiguration]], [[paxos-family]], [[fast-consensus]], [[leaderless-protocols]], [[quorum-intersection]], [[knowledge-census-recovery]], [[adopt-commit-abstraction]], [[recoverability]], [[linearizability]], [[liveness]], [[unresolved-confusions]]
