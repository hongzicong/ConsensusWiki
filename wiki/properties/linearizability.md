# linearizability

Linearizability gives each operation a single point between invocation and response; PANDO targets linearizable key-value operations, while EPaxos/SwiftPaxos target SMR order.

[[PigPaxos]] targets the same linearizable SMR behavior as Multi-Paxos: one stable leader orders operations into log slots, and relay aggregation does not change the order/commit semantics.

[[Atlas]] targets linearizable SMR by ensuring validity, integrity, and acyclic ordering of real-time plus conflicting execution order. Commands that commute need not execute in the exact same order at every process.

[[Rabia]] targets log-based SMR linearizability by making replicas execute the same sequence of non-`⊥` decided requests in slot order. Duplicate client requests are skipped using unique IDs.

## Related pages
[[PigPaxos]], [[Atlas]], [[Rabia]], [[agreement]], [[recovery]], [[quorum]]
