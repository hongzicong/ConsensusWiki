# leader

A [[leader]] or coordinator chooses rounds, ballots, or proposals. Protocols differ in whether the leader is on every fast path.

[[Atlas]] has no distinguished leader. Each command has an initial coordinator, and recovery may be performed by another process.

[[PigPaxos]] keeps a stable Multi-Paxos leader for ordering and decisions, but rotates relay followers underneath it to reduce fan-out/fan-in load. Relays are not leaders and do not choose values.

[[Rabia]] has no leader or command leader in the core protocol. Every replica participates in Weak-MVC for every slot, and failure handling is inside the randomized consensus protocol.

## Related pages
[[FastPaxos]], [[EPaxos]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]]

