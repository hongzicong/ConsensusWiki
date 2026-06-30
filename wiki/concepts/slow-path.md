# slow-path

A [[slow-path]] resolves disagreement, collisions, missing metadata, or insufficient fast-path evidence, usually by falling back to majority/Paxos-style evidence.

In [[PigPaxos]], slow behavior is timeout/retry behavior in the relay layer. If partial relay aggregates or failed relays do not yield a majority, the leader retries with fresh random relays; safe-value recovery remains ordinary Paxos.

In [[Atlas]], the slow path runs a Flexible Paxos-style consensus step over a slow quorum of size `f + 1` when the fast dependency union is not recoverable by the `union_Q dep = union^f_Q dep` predicate.

In [[Rabia]], the slow path is additional randomized binary-consensus phases after the initial Proposal/State/vote fast sequence does not terminate.

## Related pages
[[FastPaxos]], [[EPaxos]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]]
