# failure-model

The ingested papers assume non-Byzantine failures unless marked otherwise. Progress typically requires enough nonfaulty replicas/data sites and eventual communication.

[[Atlas]] explicitly optimizes for small concurrent data-center outages by choosing a tolerated failure count `f` independently of the total site count `n`; violating that bound may block progress but does not by itself break safety.

[[Rabia]] assumes fail-stop crashes, at most `f` faulty replicas, and `n ≥ 2f + 1`; Byzantine behavior is outside the model.

## Related pages
[[FastPaxos]], [[EPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]]
