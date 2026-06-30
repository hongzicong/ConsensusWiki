# SMR

State-machine replication ([[SMR]]) runs deterministic replicas through the same ordered sequence of commands so they maintain equivalent service state despite failures.

In this wiki, SMR protocols differ mainly in how they choose log slots, handle conflicts, recover after failures, and decide when commands are executable.

## Related pages
[[Rabia]], [[Mencius]], [[EPaxos]], [[Atlas]], [[SwiftPaxos]], [[linearizability]]
