# Unresolved Confusions

- Fast Paxos quorum-size inequalities need a precise algebra note beyond the high-level requirement.
- EPaxos optimized recovery should be checked against the technical report before formalization.
- Copilot's OSDI paper summarizes the ambiguous fast-accept recovery case but delegates the complete recursive value-picking procedure and proof to an accompanying technical report; ingest that report before formalizing recovery in full.
- HydraPaxos inherits missing-message recovery from NOPaxos, but the Hydra paper does not restate the exact recovery-evidence quorum or late-message fencing predicate; ingest NOPaxos before formalizing that path.
- WPaxos §4.2 says Phase 2 uses the closest `F + 1` zones, while the formal quorum definition in §3.1 uses `f_z + 1`; verify whether `F` is a typographical shorthand before formalization.
- WPaxos Algorithm 2's displayed `1b` carries the highest slot but not explicit accepted values, while the prose requires recovery of unresolved commands with suggested values; inspect the TLA+ artifact/implementation for the complete recovery payload and selection rule.
- Hermes §3.4 states that partition handling reduces resilience from `n - 1` node failures to "less than `floor(n/2)` failures" when datastore replicas run RM. Preserve this wording and reconcile it with the exact RM majority configuration before deriving a conventional `f` bound.
- Hermes reports TLA+ checking for safety and deadlock freedom but the PDF does not give the finite-state bounds, named lemmas, or full RM composition invariant; inspect the external artifact before formalizing the proof in full.
- KCensus §6 installs new latency/requirement tables by cross-shard consensus, but does not give the exact slot/epoch fencing rule for in-flight instances. The proved core treats requirements as constant; specify that boundary before modeling dynamic installation.
- KCensus §6 overlaps Paxos Phase 1 with adopt-commit, freezes early, and continues graph forwarding with frozen-state piggybacking. Check the implementation refinement against Algorithms 2–7 before treating the optimized runtime as covered by the template proof.
- KCensus Algorithm 6 indexes requirements by `v.proposer`; the abstract value type and handling of repeated identical application values need explicit proposer/instance binding before formalization.
- KCensus does not fully spell out restart persistence, log catch-up/compaction, request deduplication, or read-quorum state reports. Inspect the artifact before claiming a complete KSMR specification. Source for these KCensus items: [[KCensus-2026]], §§4–6; [[knowledge-census-recovery]].

- PQL §4 footnote 6 ends with `⌊N/2⌋` total acceptance signals, while §2.1 requires `⌊N/2⌋ + 1`. Clarify self/implicit leader accounting before implementing near-client commit observation; do not reduce the ordinary Paxos majority. PQL also lacks a quantitative drift-margin/persistence refinement. Source: [[Paxos-Quorum-Leases-2014]], §§3.1, 3.7, 4.

- PQR §4 says majority of followers excluding the leader, but Table 1 uses `⌊n/2⌋ + 1` with `n` total nodes. Use a discovery set intersecting every write quorum; do not shrink the denominator to only followers. A strictly leader-excluding read can lose availability before host writes do; the PDF gives no failover policy. Source: [[Linearizable-Quorum-Reads-2019]], §4 and Table 1.
- PQR lacks full pseudocode for binding first-round data to final executed state after a higher-ballot overwrite/no-op, per-key/multi-key snapshot semantics, and restart/reconfiguration/compaction fencing. Do not treat a maximum accepted counter or experimental history checking as a complete read proof. Source: [[Linearizable-Quorum-Reads-2019]], §§3-4, 6; [[read-freshness]].
