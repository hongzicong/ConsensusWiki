# ConsensusWiki Index

## Papers
- [[FastPaxos-2006]] - Fast Paxos fast rounds and quorum requirements.
- [[Generalized-Paxos-2005]] - Generalized Paxos over c-structs, enabling fast learning of compatible concurrent commands.
- [[EPaxos-2013]] - Leaderless dependency-based Paxos for geo-replicated SMR.
- [[EPaxos-Revisited-2021]] - Reevaluation of EPaxos conflict behavior, tail latency, and clock-based mitigation.
- [[Making-Democracy-Work-2025]] - EPaxos* correction with validation-based recovery and optimal `f`/`e` quorum bound.
- [[Mencius-2008]] - Multi-leader Paxos-derived SMR for WANs using rotating coordinators and cheap `no-op` skips.
- [[PigPaxos-2021]] - Multi-Paxos communication-overlay protocol using randomized relay aggregation.
- [[Atlas-2020]] - Leaderless planet-scale SMR with fast quorum `floor(n/2) + f` and recoverable dependency unions.
- [[Pando-2020]] - Erasure-coded geo-storage with Phase 1a/1b/2 quorums.
- [[SwiftPaxos-2024]] - Dependency-based SMR with leader-including fast quorums and optimized read-only execution at fast-quorum replicas.
- [[Rabia-2021]] - Randomized leaderless SMR using Weak-MVC and forfeited `⊥` slots.
- [[CURP-2019]] - Primary-backup fast replication using commutative unordered witness durability.
- [[Copilot-2020]] - Dual-pilot SMR that preserves normal latency despite one slow replica.
- [[Avicenna-2026]] - Single-leader geo-SMR with counterfactual fail-slow detection and fast shadow-leader rotation.
- [[Bodega-2026]] - Lease-based Paxos extension for local linearizable reads at arbitrary per-key responders.
- [[Jetpack-2026]] - General fast-path shim with view-safe proposer promises and prioritized recovery sets.
- [[Flexible-Paxos-2016]] - Paxos generalization requiring intersection only between Phase 1 and Phase 2 quorums.
- [[Omni-Paxos-2023]] - Connectivity-aware SMR separating leader election, prefix log replication, and reconfiguration.
- [[Hydra-2023]] - Multi-sequencer network ordering with monotonic clocks, per-group counters, and ordered drop detection.
- [[WPaxos-2020]] - WAN multi-leader Paxos with object stealing and topology-aware flexible quorums.
- [[Hermes-2020]] - Membership-based single-key replication using invalidations, logical timestamps, and replayable writes.

## Protocols
- [[FastPaxos]] - Classic Paxos extended with fast rounds.
- [[GPaxos]] - Generalized Paxos using compatible command structures instead of a single total command sequence.
- [[EPaxos]] - Egalitarian Paxos with command leaders and dependencies.
- [[EPaxosStar]] - Corrected/simplified EPaxos with validation-based recovery.
- [[Mencius]] - Rotating-coordinator SMR that partitions log instances among servers.
- [[PigPaxos]] - Multi-Paxos with randomized relay followers that aggregate acknowledgements.
- [[Atlas]] - Leaderless dependency SMR parameterized by tolerated concurrent site failures `f`.
- [[Pando]] - Paxos-style erasure-coded storage protocol.
- [[SwiftPaxos]] - WAN SMR using leader-including fast quorums, `FastAck`/`SlowAck`, client-visible dependency-path evidence, and distributed speculative reads.
- [[Rabia]] - Leaderless randomized SMR with weak multi-valued consensus.
- [[CURP]] - Primary-backup protocol that completes commutative updates in 1 RTT via witnesses.
- [[Copilot]] - Dual-leader dependency SMR with redundant processing and per-entry fast takeover.
- [[Avicenna]] - Multi-Paxos-style SMR that masks one fail-slow replica using independent shadow processing and restricted-quorum rotation.
- [[Bodega]] - Multi-Paxos-style SMR with roster leases and responder-covering writes for local linearizable reads.
- [[Jetpack]] - Plugin framework adding a conflict-free 1-RTT path to compatible consensus protocols.
- [[FPaxos]] - Paxos with independently shaped Phase 1 and Phase 2 quorum families.
- [[OmniPaxos]] - Sequence Paxos plus quorum-connected leader election and service-layer reconfiguration.
- [[Hydra]] - Ordered-unreliable groupcast using multiple active sequencers without traffic serialization.
- [[HydraPaxos]] - NOPaxos-derived SMR layered on Hydra with one-round-trip majority commit.
- [[WPaxos]] - Per-object WAN Paxos using cross-zone ownership quorums and local commit quorums.
- [[Hermes]] - Read-one/write-all replication with local reads, per-update coordinators, leased membership, and safe write replay.

## Concepts
- [[quorum]] - Evidence sets and intersection requirements.
- [[fast-path]] - Low-latency common-case commit/learn path.
- [[slow-path]] - Fallback after disagreement or uncertainty.
- [[recovery]] - Safe value/metadata reconstruction.
- [[leader]] - Coordinator, command leader, ballot leader, or delegate role.
- [[conflict]] - Concurrent proposals or non-commuting commands.
- [[command-structure]] - GPaxos c-struct values, compatibility, and lub/glb reasoning.
- [[dependency]] - Command-order metadata.
- [[failure-model]] - Non-Byzantine assumptions and availability.
- [[randomized-consensus]] - Consensus that uses random choices to obtain probabilistic termination.
- [[common-coin]] - Shared random bit abstraction used by Rabia's Weak-MVC.
- [[SMR]] - State-machine replication by ordered command execution.
- [[witness]] - CURP temporary unordered durability component.
- [[slowdown-tolerance]] - Performance resilience when replicas remain responsive but slow.
- [[counterfactual-evaluation]] - Parallel estimation of alternative-system performance without letting the shadow path control correctness.
- [[roster-lease]] - Majority-backed agreement on leader and per-key responder metadata with catch-up thresholds.
- [[view-change-hazard]] - Loss or reordering of fast-path promises when a new proposer set takes over.
- [[flexible-quorum]] - Phase-specific quorum families constrained only by safety-relevant intersections.
- [[partial-connectivity]] - Link-level partitions that leave inconsistent reachability views among live servers.
- [[sequence-consensus]] - Consensus over a strictly growing, prefix-comparable command log.
- [[object-stealing]] - Moving per-object leadership through Paxos Phase 1 as access locality changes.
- [[invalidation]] - Temporarily disabling local reads while an update becomes visible to every authorized replica.
- [[reliable-membership]] - Leased, epoch-fenced agreement on the live replica set used by membership-based protocols.

## Properties
- [[agreement]] - No incompatible decisions.
- [[validity]] - Chosen values come from proposals.
- [[liveness]] - Conditional progress assumptions.
- [[recoverability]] - Safe recovery from evidence, including Atlas recoverable dependency unions.
- [[linearizability]] - Operation order semantics for SMR and storage protocols.

## Comparisons
- [[protocol-catalog]] - Concise protocol table.
- [[quorum-systems]] - Quorum mechanisms by protocol.
- [[fast-paths]] - Fast-path assumptions and evidence.
- [[recovery-rules]] - Recovery evidence and selection.
- [[commit-rules]] - Commit predicates and safety evidence.
- [[fault-models]] - Failure assumptions across protocols.
- [[timing-assumptions]] - Safety/liveness timing assumptions.
- [[latency]] - Stable client-response latency across sequential, conflict-free, contention-free, and general runs.
- [[leader-roles]] - Leader/delegate involvement.
- [[conflict-handling]] - Collisions, dependencies, and write conflicts.
- [[proof-techniques]] - Main proof invariants.
- [[reconfiguration]] - Configuration boundaries, decided-state migration, and startup conditions.
- [[paxos-family]] - Paxos-family overview.
- [[fast-consensus]] - Fast consensus overview.
- [[leaderless-protocols]] - Leaderless protocols overview.
- [[bft-consensus]] - Placeholder for future BFT consensus papers.
- [[dag-based-consensus]] - Placeholder for future DAG-based consensus papers.
- [[FastPaxos-EPaxos-SwiftPaxos]] - Focused modeling comparison.

## Proof notes
- [[quorum-intersection]] - Intersection obligations.
- [[adopt-commit-abstraction]] - Fast-evidence abstraction.
- [[rocq-modeling-notes]] - Rocq/Coq modeling reminders.

## Open questions
- [[new-protocol-ideas]] - 100 defect-driven SMR candidates with evidence clusters, a ranked shortlist, red-team attacks, and next proof/model artifacts.
- [[unresolved-confusions]] - TODOs and uncertain extracted facts.
