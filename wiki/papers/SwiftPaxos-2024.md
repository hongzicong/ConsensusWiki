---
type: paper
title: "SwiftPaxos: Fast Geo-Replicated State Machines"
authors: Fedor Ryabinin, Alexey Gotsman, Pierre Sutra
year: 2024
venue: NSDI 2024
source: raw/swiftpaxos.pdf
protocols: [SwiftPaxos]
tags: [paxos, dependencies, geo-replication, fast-path, leader]
status: ingested
---

# SwiftPaxos: Fast Geo-Replicated State Machines

## One-sentence summary
SwiftPaxos is a geo-replicated state-machine replication protocol that uses leader-including fast quorums, dependency tracking, and optimistic execution to return in two or three message delays.

## Why this paper matters
It refines EPaxos-style dependencies by requiring acyclic committed dependencies and by allowing slow-path repair through double voting inside fast quorums.

## System model
State machine replication with a fixed set `R` of `N = 2f + 1` replicas. Clients submit commands tagged with unique identifiers.

## Fault model
Crash/non-Byzantine failures with at most `f` faulty replicas. Liveness uses an eventual leader/failure detector style assumption.

## Timing assumptions
Designed for WANs with heterogeneous latencies. Safety is asynchronous; liveness follows after recovery stabilizes at a ballot with a correct trusted leader.

## Main idea
Commands carry dependencies on conflicting commands. Replicas agree on dependency paths using `FastAck` and `SlowAck`; execution follows the acyclic dependency graph.

## Protocol roles
Each ballot has a fixed leader `leader(b)`. Other replicas are followers. A replica may belong to fast and/or slow quorums for a ballot.

## Message types
`Propagate(c)`, `FastAck(b,id,D,P)`, `SlowAck(b,id)`, `NewLeader(b)`, `NewLeaderAck`, `Sync`, and recovery/commit messages.

## Local state
Replicas track `bal`, `cbal`, `status` (`NORMAL` or `RECOVERING`), command table `cmd`, phase sets such as `Start`, `Accept`, `Commit`, dependency map `dep`, and executed set `Exec`.

## Normal path
Clients broadcast `Propagate(c)`. Fast quorum replicas compute dependencies from locally known conflicting commands and broadcast `FastAck`. The leader's proposal is central for both fast and slow agreement.

## Fast path
A replica commits when it receives matching `FastAck` messages from all members of some fast quorum and the command's dependencies are committed. Under favorable conditions, execution can complete within two message delays from submission.

## Slow path
If a fast quorum replica disagrees with the leader's dependencies, it can send `SlowAck`, adopting/correcting to the leader's proposal. A command can commit with matching `FastAck`/`SlowAck` evidence from a fast quorum, or with `SlowAck`s from a slow quorum.

## Recovery path
A new leader chooses a higher ballot, gathers `NewLeaderAck` from a majority, reconstructs commands that could have been committed in earlier ballots, breaks unsafe dependency cycles, broadcasts `Sync`, and then resumes normal processing.

## Commit rule
Commit requires either matching dependency-path evidence from all followers in a fast quorum or `SlowAck`s from a slow quorum, with `D = dep[id]` and dependencies already committed.

## Quorum system
`N = 2f + 1`. Slow quorums can be any majority. Fast quorums must include the ballot leader and satisfy fast-quorum intersection:
`forall Q1, Q2 in FQ(b). |Q1 intersect Q2| > N/2`.
The paper studies at least two configurations: `(C1)` any set containing more than `3/4` of all replicas, and `(C2)` a unique fixed majority fast quorum including the leader.

## Conflict handling
Conflicting commands must be ordered by dependencies: for any two conflicting commands committed at a replica, one command belongs to the other's dependencies. Unlike EPaxos, committed dependencies are kept acyclic.

## Safety argument
Invariants include: any two replicas commit a command with the same dependencies; conflicting committed commands are dependency-ordered; the committed dependency graph is acyclic.

## Liveness argument
After recovery stops and all correct replicas stabilize in one ballot, commands submitted by correct clients are eventually accepted, committed, and executed.

## Key proof ideas
The proof defines `acc(b,id,c,P)` for accepted dependency paths by a fast or slow quorum. Recovery preserves accepted paths using quorum intersection between recovery majorities and previous fast/slow quorums.

## Important formulas
- `N = 2f + 1`.
- Slow quorum: majority.
- Fast quorum intersection: `|Q1 intersect Q2| > N/2`.
- `(C1)`: fast quorum size `> 3/4 N`.
- `(C2)`: unique fixed majority fast quorum.

## Relationship to other protocols
SwiftPaxos is closely related to [[EPaxos]] through dependencies, to [[FastPaxos|Fast Paxos]] through fast-quorum reasoning, and to classic Paxos through ballots and leader recovery.

## Modeling notes

### Rocq/Coq modeling notes
Model dependency paths, not only direct dependency sets. Prove acyclicity as a first-class invariant.

## Limitations
Message complexity is quadratic. The extracted PDF text did not expose full bibliographic metadata; venue/authors need verification from the PDF front matter.

## Open questions
- TODO: Capture exact pseudocode line numbers for the commit preconditions.

## Related pages
[[SwiftPaxos]], [[dependency]], [[conflict]], [[fast-path]], [[leader]], [[recovery]]
