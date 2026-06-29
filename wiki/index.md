# ConsensusWiki Index

## Papers
- [[FastPaxos-2006]] - Fast Paxos fast rounds and quorum requirements.
- [[EPaxos-2013]] - Leaderless dependency-based Paxos for geo-replicated SMR.
- [[EPaxos-Revisited-2021]] - Reevaluation of EPaxos conflict behavior, tail latency, and clock-based mitigation.
- [[Making-Democracy-Work-2025]] - EPaxos* correction with validation-based recovery and optimal `f`/`e` quorum bound.
- [[Mencius-2008]] - Multi-leader Paxos-derived SMR for WANs using rotating coordinators and cheap `no-op` skips.
- [[Pando-2020]] - Erasure-coded geo-storage with Phase 1a/1b/2 quorums.
- [[SwiftPaxos-2024]] - Dependency-based SMR with leader-including fast quorums.

## Protocols
- [[FastPaxos]] - Classic Paxos extended with fast rounds.
- [[EPaxos]] - Egalitarian Paxos with command leaders and dependencies.
- [[EPaxosStar|EPaxos*]] - Corrected/simplified EPaxos with validation-based recovery.
- [[Mencius]] - Rotating-coordinator SMR that partitions log instances among servers.
- [[Pando]] - Paxos-style erasure-coded storage protocol.
- [[SwiftPaxos]] - WAN SMR using FastAck/SlowAck dependency evidence.

## Concepts
- [[quorum]] - Evidence sets and intersection requirements.
- [[fast-path]] - Low-latency common-case commit/learn path.
- [[slow-path]] - Fallback after disagreement or uncertainty.
- [[recovery]] - Safe value/metadata reconstruction.
- [[leader]] - Coordinator, command leader, ballot leader, or delegate role.
- [[conflict]] - Concurrent proposals or non-commuting commands.
- [[dependency]] - Command-order metadata.
- [[failure-model]] - Non-Byzantine assumptions and availability.

## Properties
- [[agreement]] - No incompatible decisions.
- [[validity]] - Chosen values come from proposals.
- [[liveness]] - Conditional progress assumptions.
- [[recoverability]] - Safe recovery from evidence.
- [[linearizability]] - Operation order semantics.

## Comparisons
- [[protocol-catalog]] - Concise protocol table.
- [[quorum-systems]] - Quorum mechanisms by protocol.
- [[fast-paths]] - Fast-path assumptions and evidence.
- [[recovery-rules]] - Recovery evidence and selection.
- [[commit-rules]] - Commit predicates and safety evidence.
- [[fault-models]] - Failure assumptions across protocols.
- [[timing-assumptions]] - Safety/liveness timing assumptions.
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
- [[tla-modeling-notes]] - TLA+ modeling reminders.

## Open questions
- [[new-protocol-ideas]] - Ideas inspired by the ingested papers.
- [[unresolved-confusions]] - TODOs and uncertain extracted facts.

## Recently updated
- 2026-06-30 - Added more candidate ideas to [[new-protocol-ideas]] across faster fast paths and faster slow/recovery paths.
- 2026-06-30 - Refined [[new-protocol-ideas]] around faster fast paths or 1RTT fast paths with faster slow paths.
- 2026-06-30 - Reorganized [[new-protocol-ideas]] around a fast-path non-regression gate.
- 2026-06-28 - Reread comparison/proof notes and expanded [[new-protocol-ideas]] with grounded design sketches.
- 2026-06-27 - Ingested Mencius from `raw/mencius.pdf`.
- 2026-06-25 - Added EPaxos/SwiftPaxos fusion notes to [[FastPaxos-EPaxos-SwiftPaxos]] and [[new-protocol-ideas]].
- 2026-06-25 - Added fast-path design notes to [[new-protocol-ideas]].
- 2026-06-25 - Linted wiki structure; added missing comparison pages and fixed catalog/link formatting issues.
- 2026-06-24 - Added EPaxos* `e = f` boundary-case quorum note.
- 2026-06-24 - Clarified EPaxos* `e` as the fast-path failure budget and why fast quorum size is `n - e`.
- 2026-06-24 - Added Pando quorum sizes to the fast quorum comparison table.
- 2026-06-24 - Ingested EPaxos* from `raw/epaxos_plus.pdf`.
- 2026-06-24 - Added Fast Paxos `N = 2f + 1` fast quorum derivation.
- 2026-06-24 - Added a fast-path distinction table for FastPaxos, EPaxos, and SwiftPaxos.
- 2026-06-24 - Clarified fast quorum size counting for FastPaxos, EPaxos, and SwiftPaxos.
- 2026-06-24 - Ingested EPaxos Revisited from `raw/epaxos_revisited.pdf`.
- 2026-06-24 - Ingested Fast Paxos, EPaxos, PANDO, and SwiftPaxos from `raw/`.




