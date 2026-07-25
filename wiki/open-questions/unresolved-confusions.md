# Unresolved Confusions

- Fast Paxos quorum-size inequalities need a precise algebra note beyond the high-level requirement.
- EPaxos optimized recovery should be checked against the technical report before formalization.
- Copilot's OSDI paper summarizes the ambiguous fast-accept recovery case but delegates the complete recursive value-picking procedure and proof to an accompanying technical report; ingest that report before formalizing recovery in full.
- HydraPaxos inherits missing-message recovery from NOPaxos, but the Hydra paper does not restate the exact recovery-evidence quorum or late-message fencing predicate; ingest NOPaxos before formalizing that path.
- WPaxos §4.2 says Phase 2 uses the closest `F + 1` zones, while the formal quorum definition in §3.1 uses `f_z + 1`; verify whether `F` is a typographical shorthand before formalization.
- WPaxos Algorithm 2's displayed `1b` carries the highest slot but not explicit accepted values, while the prose requires recovery of unresolved commands with suggested values; inspect the TLA+ artifact/implementation for the complete recovery payload and selection rule.
- Hermes §3.4 states that partition handling reduces resilience from `n - 1` node failures to "less than `floor(n/2)` failures" when datastore replicas run RM. Preserve this wording and reconcile it with the exact RM majority configuration before deriving a conventional `f` bound.
- Hermes reports TLA+ checking for safety and deadlock freedom but the PDF does not give the finite-state bounds, named lemmas, or full RM composition invariant; inspect the external artifact before formalizing the proof in full.
