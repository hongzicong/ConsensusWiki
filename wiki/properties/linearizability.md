# linearizability

Linearizability gives each operation a single point between invocation and response; PANDO targets linearizable key-value operations, while EPaxos/SwiftPaxos target SMR order.

[[PigPaxos]] targets the same linearizable SMR behavior as Multi-Paxos: one stable leader orders operations into log slots, and relay aggregation does not change the order/commit semantics.

[[Atlas]] targets linearizable SMR by ensuring validity, integrity, and acyclic ordering of real-time plus conflicting execution order. Commands that commute need not execute in the exact same order at every process.

[[Rabia]] targets log-based SMR linearizability by making replicas execute the same sequence of non-`⊥` decided requests in slot order. Duplicate client requests are skipped using unique IDs.

[[CURP]] preserves primary-backup linearizability while replying before backup sync. Its proof relies on witness durability for completed unsynced operations, master-side sync before non-commuting dependent observations, and exactly-once duplicate filtering during replay.

## Related pages
[[PigPaxos]], [[Atlas]], [[Rabia]], [[CURP]], [[agreement]], [[recovery]], [[quorum]]
