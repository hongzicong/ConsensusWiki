# agreement

Agreement means no two correct learners/replicas decide incompatible values or command metadata.

[[GPaxos]] generalizes agreement from identical learned values to compatible learned c-structs. Learners may learn different extensions, but the learned c-structs must have a common upper bound.

[[PigPaxos]] inherits Paxos agreement: relay aggregation changes message transport, but a value is still chosen only through a majority of unique replica votes in a ballot.

[[Atlas]] agreement is per command identifier: all `MCommit` messages for an identifier carry the same command and dependency set. Cross-command ordering is handled by the dependency invariant for conflicting non-`noOp` commands.

[[Rabia]] agreement is per log slot over weak outputs: replicas cannot decide different requests or disagree between a request and `⊥` for the same Weak-MVC instance.

[[Bodega]] inherits Paxos agreement for log entries and adds agreement on read-serving metadata: majority-held [[roster-lease]] grants ensure at most one ballot-tagged roster is stable. This does not itself commit writes; it makes the leader/responder assignment unique while ordinary consensus chooses log values.

[[Jetpack]] preserves host agreement by never changing the host's commit predicate. A fast commit is an early client-visible promise: all host proposers must place the command consistently, while majority acceptance of one per-view recovery set ensures later coordinators resubmit the same set before new-view work.

[[FPaxos]] proves ordinary single-value agreement without same-phase quorum intersection. If `v` is decided at proposal `p`, every later proposal carries `v` because its Phase 1 quorum intersects the deciding Phase 2 quorum and learns or preserves that value.

[[OmniPaxos]] uses prefix agreement rather than equal-length log agreement. Sequence Consensus SC2 requires any two decided logs to be prefix-comparable. The proof first establishes SC3, that one server's decisions strictly extend, then uses intersection of the two deciding majorities to rule out incomparable logs.

[[Hydra]] guarantees ordered delivery/drop agreement at receiver-group granularity, not consensus on a value: for each groupcast, either every destination group delivers the message or an ordered drop notification, or none does. [[HydraPaxos]] composes that guarantee with majority durability and agreement on `NO-OP` for unrecoverable gaps.

[[WPaxos]] agreement is per object and slot: no two owners commit different values for the same `(object, slot)`. A later ownership `Q1` intersects every earlier deciding `Q2`, so the new ballot preserves or blocks the earlier value.

## Related pages
[[FPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], [[WPaxos]], [[GPaxos]], [[PigPaxos]], [[Atlas]], [[Rabia]], [[Bodega]], [[Jetpack]], [[partial-connectivity]], [[sequence-consensus]], [[roster-lease]], [[view-change-hazard]], [[flexible-quorum]], [[recovery]], [[quorum]], [[command-structure]]
