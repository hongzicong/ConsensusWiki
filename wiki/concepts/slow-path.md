# slow-path

A [[slow-path]] resolves disagreement, collisions, missing metadata, or insufficient fast-path evidence, usually by falling back to majority/Paxos-style evidence.

In [[PigPaxos]], slow behavior is timeout/retry behavior in the relay layer. If partial relay aggregates or failed relays do not yield a majority, the leader retries with fresh random relays; safe-value recovery remains ordinary Paxos.

In [[Atlas]], the slow path runs an [[FPaxos]]-style consensus step over a slow quorum of size `f + 1` when the fast dependency union is not recoverable by the `union_Q dep = union^f_Q dep` predicate.

In [[Rabia]], the slow path is additional randomized binary-consensus phases after the initial Proposal/State/vote fast sequence does not terminate.

In [[Copilot]], the regular path begins after incompatibility or insufficient fast replies. The pilot collects at least `f + 1` suggestions, selects the `(f + 1)`-th sorted dependency, and commits it through majority `Accept` evidence. Fast takeover is a separate higher-ballot path for unresolved entries that block execution.

In [[Bodega]], an uncertain local read is optimistically held until its newest same-key write commits or gains majority early-accept evidence. Client timeout redirects the duplicate idempotent read; a node without stable/caught-up roster evidence falls back to classic consensus or redirects to the leader.

In [[Jetpack]], conflict, view mismatch, or insufficient same-view/proposer acknowledgements only defeats fast commitment. The command has already entered the independent host protocol and completes under that protocol's ordinary slow-path ordering and recovery rules.

In [[FPaxos]], Phase 1 is the infrequent recovery/election path rather than a conflict slow path. Deployments often choose larger `Q1` to shrink common `Q2`, so leader replacement can require more acceptors than steady-state replication.

In [[OmniPaxos]], Prepare/`AcceptSync` is the infrequent leader-change or resynchronization path. It is mandatory after BLE elects a leader, because eligibility ignores log freshness. It is not triggered by command conflicts, which the single leader serializes.

## Related pages
[[FastPaxos]], [[FPaxos]], [[OmniPaxos]], [[EPaxos]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[Copilot]], [[Bodega]], [[Jetpack]], [[sequence-consensus]]
