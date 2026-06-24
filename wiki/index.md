# ConsensusWiki Index

## Papers
- [[FastPaxos-2006]] - Fast Paxos fast rounds and quorum requirements.
- [[EPaxos-2013]] - Leaderless dependency-based Paxos for geo-replicated SMR.
- [[EPaxos-Revisited-2021]] - Reevaluation of EPaxos conflict behavior, tail latency, and clock-based mitigation.
- [[Making-Democracy-Work-2025]] - EPaxos* correction with validation-based recovery and optimal `f`/`e` quorum bound.
- [[Pando-2020]] - Erasure-coded geo-storage with Phase 1a/1b/2 quorums.
- [[SwiftPaxos-2024]] - Dependency-based SMR with leader-including fast quorums.

## Protocols
- [[FastPaxos]] - Classic Paxos extended with fast rounds.
- [[EPaxos]] - Egalitarian Paxos with command leaders and dependencies.
- [[EPaxosStar|EPaxos*]] - Corrected/simplified EPaxos with validation-based recovery.
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
- [[leader-roles]] - Leader/delegate involvement.
- [[conflict-handling]] - Collisions, dependencies, and write conflicts.
- [[proof-techniques]] - Main proof invariants.
- [[paxos-family]] - Paxos-family overview.
- [[fast-consensus]] - Fast consensus overview.
- [[leaderless-protocols]] - Leaderless protocols overview.
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
- 2026-06-24 - Added EPaxos* `e = f` boundary-case quorum note.
- 2026-06-24 - Clarified EPaxos* `e` as the fast-path failure budget and why fast quorum size is `n - e`.
- 2026-06-24 - Added Pando quorum sizes to the fast quorum comparison table.
- 2026-06-24 - Ingested EPaxos* from `raw/epaxos_plus.pdf`.
- 2026-06-24 - Added Fast Paxos `N = 2f + 1` fast quorum derivation.
- 2026-06-24 - Added a fast-path distinction table for FastPaxos, EPaxos, and SwiftPaxos.
- 2026-06-24 - Clarified fast quorum size counting for FastPaxos, EPaxos, and SwiftPaxos.
- 2026-06-24 - Ingested EPaxos Revisited from `raw/epaxos_revisited.pdf`.
- 2026-06-24 - Ingested Fast Paxos, EPaxos, PANDO, and SwiftPaxos from `raw/`.
