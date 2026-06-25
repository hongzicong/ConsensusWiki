# FastPaxos, EPaxos, SwiftPaxos

## Core distinction
[[FastPaxos]] chooses one value for one consensus instance. [[EPaxos]] and [[SwiftPaxos]] solve SMR by attaching ordering metadata to commands.

## Fast-path distinction
[[FastPaxos]] fast-chooses when a fast quorum accepts the same value. [[EPaxos]] fast-commits when a command leader observes matching `(seq, deps)` from a fast quorum. [[SwiftPaxos]] fast-commits/executes using matching dependency-path evidence from leader-including fast quorums; disagreement can be repaired with `SlowAck` adopting the leader's proposal.

## Modeling warning
Do not import Fast Paxos collision recovery directly into EPaxos/SwiftPaxos. Their recovery must preserve command dependencies, not just a single value.

## EPaxos and SwiftPaxos fusion sketch

A plausible fusion keeps EPaxos's command-leader distribution but adds a SwiftPaxos-style ballot anchor for dependency evidence. The command leader would still create instances and collect fast evidence, but each fast quorum would either include a current ballot leader/delegate or include evidence derived from that leader's dependency proposal.

Possible variants:

- Per-ballot anchor: one SwiftPaxos-style leader is included in every fast quorum. This simplifies recovery but risks recreating a wide-area leader bottleneck.
- Per-shard or per-key-range anchors: command leaders remain distributed, while conflicting commands route their dependency evidence through the same anchor for that conflict class. This may preserve EPaxos-style load distribution if conflict classes are balanced.
- Rotating command anchors: the command leader also acts as the SwiftPaxos-like anchor for its own instance, but recovery requires dependency-path certificates rather than only EPaxos `(seq, deps)` tuples. This is closest to EPaxos operationally, but the recovery proof is likely hardest.
- Two-tier evidence: fast path commits on EPaxos-style matching dependencies, while fallback/repair uses SwiftPaxos-style `SlowAck` adoption and acyclic dependency-path evidence.

The main modeling choice is whether the agreed object is EPaxos's `(cmd, seq, deps)` or SwiftPaxos's dependency-path certificate. Mixing both naively risks proving instance agreement while missing execution-order safety.
