# FastPaxos, EPaxos, SwiftPaxos

## Core distinction
[[FastPaxos]] chooses one value for one consensus instance. [[EPaxos]] and [[SwiftPaxos]] solve SMR by attaching ordering metadata to commands.

## Fast-path distinction
[[FastPaxos]] fast-chooses when a fast quorum accepts the same value. [[EPaxos]] fast-commits when a command leader observes matching `(seq, deps)` from a fast quorum. [[SwiftPaxos]] fast-commits/executes using matching dependency-path evidence from leader-including fast quorums; disagreement can be repaired with `SlowAck` adopting the leader's proposal.

## Modeling warning
Do not import Fast Paxos collision recovery directly into EPaxos/SwiftPaxos. Their recovery must preserve command dependencies, not just a single value.
