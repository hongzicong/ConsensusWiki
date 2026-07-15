# recovery

[[recovery]] selects a safe value or state after failures, collisions, or ballot changes. It must preserve all values/commands that could already have been committed.

[[EPaxosStar]] is a useful recovery case study because possible fast-path evidence is not enough by itself: the new coordinator validates whether recovering a candidate dependency set would violate visibility with already committed or potentially committing conflicting commands.

[[Atlas]] is another recovery case study: its fast-path predicate is designed so a later recovery quorum can reconstruct any possible fast-path dependency union from the remembered fast quorum.

[[PigPaxos]] uses ordinary Paxos leader-change recovery. Relay failures are transport failures handled by timeout and retry, not a new safe-value selection rule.

[[GPaxos]] recovery starts a higher ballot and computes `ProvedSafe(Q, m, beta)` from phase 1b evidence. The selected c-struct must extend lower-ballot values that could still be chosen.

[[Rabia]] avoids a separate leader fail-over path. If one replica decides and crashes, the Weak-MVC value-locking argument makes surviving undecided replicas carry the same binary value into later phases.

[[CURP]] recovers by restoring an ordered backup prefix, then replaying requests from one selected witness. The single-witness rule matters: different witnesses may have accepted different request sets, but each individual witness set is mutually commutative.

[[Copilot]] recovers a blocking entry in the other pilot's log with a higher-ballot Prepare/Accept. A count in `[floor((f + 1) / 2), f)` fast-accepted replies is ambiguous, so the recoverer examines the first possibly incompatible entry in the other log. This is a cross-instance recovery dependency, not ordinary max-ballot Paxos selection.

[[Bodega]] keeps ordinary Paxos value recovery. Its additional recovery problem is metadata authority: a higher-ballot roster must revoke or expire old leases, establish new guarded leases, and make new responders catch up through majority-reported acceptance thresholds before they serve local reads.

[[Jetpack]] adds recovery of cross-path ordering promises. A majority agrees on a candidate set from the last normal view, the new host proposer commits that set or a no-op as a stability marker, and only then may the new view process commands. This addresses both value loss and the [[view-change-hazard]] of stale conflicts moving ahead.

[[FPaxos]] uses ordinary Paxos value selection over a Phase 1 quorum that may be larger or differently shaped than Phase 2. Cross-phase intersection ensures the new proposer sees at least one acceptor from every earlier deciding `Q2` and adopts the highest accepted value.

[[OmniPaxos]] deliberately allows a stale but quorum-connected server to win BLE. Sequence Paxos repairs it by selecting the Promise with highest `acceptedRnd`, breaking equal-round ties by log length, and synchronizing that suffix through `AcceptSync`. Server and link recovery enter the same path through `PrepareReq`.

[[Hydra]] recovers a failed sequencer by closing its sequence-number stream: one quorum from every receiver group reports the largest observed counters, and a final infinite-time flush turns every remaining hole into an ordered drop before configuration transition. [[HydraPaxos]] separately recovers the payload from replicas or agrees that the position is `NO-OP`.

[[WPaxos]] makes recovery and placement the same Phase 1 action. A higher per-object ballot gathers a wide `Q1` that intersects every prior local `Q2`; the new owner must finish accepted uncommitted slots before opening later slots. The paper's displayed `1b` omits the full accepted-value payload, so exact selection remains Unclear.

## Related pages
[[FastPaxos]], [[FPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], [[WPaxos]], [[GPaxos]], [[EPaxos]], [[EPaxosStar]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[CURP]], [[Copilot]], [[Bodega]], [[Jetpack]], [[partial-connectivity]], [[sequence-consensus]], [[reconfiguration]], [[roster-lease]], [[view-change-hazard]], [[flexible-quorum]], [[object-stealing]]

