# leader

A [[leader]] or coordinator chooses rounds, ballots, or proposals. Protocols differ in whether the leader is on every fast path.

[[Atlas]] has no distinguished leader. Each command has an initial coordinator, and recovery may be performed by another process.

[[PigPaxos]] keeps a stable Multi-Paxos leader for ordering and decisions, but rotates relay followers underneath it to reduce fan-out/fan-in load. Relays are not leaders and do not choose values.

[[Rabia]] has no leader or command leader in the core protocol. Every replica participates in Weak-MVC for every slot, and failure handling is inside the randomized consensus protocol.

[[Copilot]] has two distinguished leaders at once. The pilot and copilot each own one log and redundantly process every client command. Either can temporarily take over specific blocking entries from the other's log without first replacing the other pilot for all future work.

[[Bodega]] generalizes leadership metadata into a roster. One leader still orders writes, while arbitrary per-key responders gain local-read authority. The leader is implicitly a responder for every key; a higher-ballot roster changes both kinds of role together.

[[Jetpack]] treats every proposer authorized in the host's stable view as part of the fast-path promise. The fast acknowledgement set must include all of them; after a view change, the new proposer set remains paused until recovery commits a stability marker.

[[FPaxos]] makes leader availability explicitly asymmetric. A stable leader needs only a Phase 2 quorum for replication, while replacement requires the often larger Phase 1 quorum to recover accepted values before proposing.

[[OmniPaxos]] separates leader eligibility from log freshness. BLE elects the highest-ballot [[partial-connectivity|quorum-connected]] candidate, even if its log trails; Sequence Paxos then gathers a majority and synchronizes the chosen prefix before that leader accepts new commands.

[[Hydra]] has no single sequencing leader: several sequencers stamp concurrently and receivers merge their outputs. [[HydraPaxos]] still has a replica leader that executes operations, participates in every commit reply quorum, and coordinates agreement on unrecoverable dropped positions.

[[WPaxos]] assigns leadership per object. Every node may own different objects concurrently; a candidate changes only one object's owner by running Phase 1 with a higher per-object ballot. This avoids both one global leader and fully opportunistic per-command leadership.

## Related pages
[[FastPaxos]], [[FPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], [[WPaxos]], [[EPaxos]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[Copilot]], [[Bodega]], [[Jetpack]], [[partial-connectivity]], [[roster-lease]], [[view-change-hazard]], [[flexible-quorum]], [[object-stealing]]

