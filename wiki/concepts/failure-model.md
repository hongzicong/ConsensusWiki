# failure-model

The ingested papers assume non-Byzantine failures unless marked otherwise. Progress typically requires enough nonfaulty replicas/data sites and eventual communication.

[[Atlas]] explicitly optimizes for small concurrent data-center outages by choosing a tolerated failure count `f` independently of the total site count `n`; violating that bound may block progress but does not by itself break safety.

[[Rabia]] assumes fail-stop crashes, at most `f` faulty replicas, and `n ≥ 2f + 1`; Byzantine behavior is outside the model.

[[Copilot]] distinguishes failure from slowdown. A failed process is slow because it does not respond, but a slow process may continue sending messages and evade failure detection. Copilot assumes crash failures for safety/liveness and separately targets [[slowdown-tolerance]] for one slow replica.

[[OmniPaxos]] uses fail-recovery servers and models individual bidirectional link partitions. A live leader may still be unable to collect a majority, so process failure and [[partial-connectivity|quorum connectivity]] must be represented separately. Liveness assumes partial synchrony and at least one QC server.

[[Hydra]] separates sequencer/link failure and packet loss from receiver-replica failure. The primitive detects partially delivered messages but delegates receiver durability to an application quorum. [[HydraPaxos]] uses `n = 2f + 1` replicas and tolerates `f` crashes.

[[Hermes]] assumes crash-stop processes and networks that may reorder, duplicate, or lose messages or partition. Timestamps and replay handle message faults within an epoch; leased [[reliable-membership]] makes minority replicas stop serving before the primary partition installs a new live set.

## Related pages
[[FastPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], [[EPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[Hermes]], [[Copilot]], [[reliable-membership]], [[partial-connectivity]], [[slowdown-tolerance]]
