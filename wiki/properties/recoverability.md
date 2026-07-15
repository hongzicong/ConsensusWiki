# recoverability

Recoverability is the ability to reconstruct a safe value/metadata state from quorum evidence after failures or collisions.

[[Atlas]] makes recoverability part of the fast-path predicate: a coordinator fast-commits only when every dependency in `union_Q dep` appears in at least `f` fast-quorum replies, so a later recovery quorum can reconstruct the fast proposal.

[[PigPaxos]] recoverability is ordinary Paxos recoverability. Relay aggregation must preserve unique voter evidence, but relays do not create a new recoverable value-selection rule.

[[GPaxos]] recoverability is encoded by `ProvedSafe(Q, m, beta)`: a higher ballot chooses an extension of any lower-ballot c-struct that remains choosable from quorum evidence.

[[Rabia]] shifts recoverability into protocol continuation: value-locking ensures later phases preserve any binary value that could already have been decided.

[[CURP]] recoverability is based on a synced backup prefix plus one selected witness. Any fast-completed unsynced request is recorded in every witness, so the selected witness contains it; all selected-witness records commute, so replay order is safe.

[[Copilot]] recoverability combines per-entry Paxos ballots with cross-log evidence. A recovery majority may intersect a fast commit in only `floor((f + 1) / 2)` replicas, so intermediate fast-accepted counts require examining the first possibly incompatible entry in the other log before preserving the candidate or choosing `no-op`.

[[Avicenna]] makes normal recovery cheap by restricting acceptance to `f + 2` non-standbys. Any `f + 1` commit quorum intersects two non-standby logs, so the next leader can preserve committed entries after receiving one additional non-standby's `Rotate` alongside its own log. In Armageddon phases, recovery collects `f + 1` logs from the full `2f + 1` replica set. Uncommitted conflicts are resolved by highest phase.

[[Bodega]] leaves value recovery to ordinary Paxos Prepare evidence. Its extra recoverability obligation concerns local-read authority: a new roster waits for old leases to revoke/expire, and a new responder uses majority-grantor `thresh_p` reports to prove its committed prefix covers any older write majority before serving reads.

[[Jetpack]] makes fast evidence recoverable only within one independent fast-path view. A command stored on `f + ceil(f/2) + 1` replicas appears in at least `ceil(f/2) + 1` replies from every `f + 1` recovery majority. Majority acceptance freezes one recovery set; the host commits it as a stability marker before resuming.

[[FPaxos]] makes recoverability the sole cross-phase geometry constraint: every later Phase 1 quorum must intersect every earlier Phase 2 quorum that may have decided. The new proposer retains the highest accepted ballot/value returned. Dynamic Phase 2 selection would therefore require safely recording which lower quorums were actually used.

[[OmniPaxos]] separates candidate eligibility from recovery evidence. BLE may choose an outdated QC server, but Sequence Paxos Prepare adopts the highest `acceptedRnd` log returned by a majority, breaking ties by log length, and `AcceptSync` establishes the leader-prefix invariant before new commands. A recovering server or link invokes the same synchronization with `PrepareReq`.

[[Hydra]] recovers sequencing progress by converting the removed sequencer's unknown suffix into group-consistent ordered drops. [[HydraPaxos]] then recovers each missing payload from replicas or agrees on `NO-OP`.

[[WPaxos]] relies on a wide per-object `Q1` intersecting every prior `Q2` so a new owner can recover accepted unfinished slots. The prose states this obligation, but the displayed `1b` payload lacks explicit accepted values; formal recovery should retain this as an unresolved artifact-level detail.

## Related pages
[[FPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], [[WPaxos]], [[GPaxos]], [[PigPaxos]], [[Atlas]], [[Rabia]], [[CURP]], [[Copilot]], [[Avicenna]], [[Bodega]], [[Jetpack]], [[partial-connectivity]], [[sequence-consensus]], [[reconfiguration]], [[roster-lease]], [[view-change-hazard]], [[flexible-quorum]], [[agreement]], [[recovery]], [[quorum]]
