# SMR

State-machine replication ([[SMR]]) runs deterministic replicas through the same ordered sequence of commands so they maintain equivalent service state despite failures.

In this wiki, SMR protocols differ mainly in how they choose log slots, handle conflicts, recover after failures, and decide when commands are executable.

[[OmniPaxos]] exposes an additional architectural dimension: leader election, [[sequence-consensus|log replication]], and [[reconfiguration]] can be separate components while presenting one replicated log through a service layer.

[[HydraPaxos]] separates network order from replica durability: [[Hydra]] supplies ordered delivery/drop evidence, while a majority and deterministic execution supply SMR semantics. [[WPaxos]] instead maintains one ordered log per object and provides per-object rather than one global linearizable order.

## Related pages
[[Rabia]], [[Mencius]], [[EPaxos]], [[Atlas]], [[SwiftPaxos]], [[OmniPaxos]], [[HydraPaxos]], [[WPaxos]], [[sequence-consensus]], [[reconfiguration]], [[linearizability]]
