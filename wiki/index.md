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
- [[new-protocol-ideas]] - 100 defect-driven SMR candidates plus a focused latency-optimization roadmap and proof agenda.
- [[unresolved-confusions]] - TODOs and uncertain extracted facts.
