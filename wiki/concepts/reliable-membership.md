# reliable-membership

[[reliable-membership]] supplies a stable, fenced set of live replicas to a membership-based replication protocol. It allows a fast read-one/write-all data path to avoid majority voting on each operation, while moving membership agreement and partition handling into a separate control plane.

In [[Hermes]], each node stores a lease, `epoch_id`, and `live_nodes`. Messages carry the sender's epoch and receivers drop messages from a different epoch. RM changes the membership through a majority-based protocol only after old leases expire, so removed or minority-partition replicas stop serving before operations complete in the new membership.

This separation creates two proof obligations:

1. within one epoch, every member authorized to serve reads must acknowledge an invalidating write before it commits;
2. across epochs, no old member may remain authorized while a new membership completes operations.

RM is not a data-path quorum. Modeling Hermes as majority-write replication would lose the write-all invariant needed for local reads.

## Related pages

[[Hermes]], [[Hermes-2020]], [[invalidation]], [[quorum]], [[recovery]], [[reconfiguration]], [[failure-model]]
