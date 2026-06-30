---
type: paper
title: There Is More Consensus in Egalitarian Parliaments
authors: Iulian Moraru, David G. Andersen, Michael Kaminsky
year: 2013
venue: SOSP 2013
source: raw/epaxos.pdf
protocols: [EPaxos]
tags: [paxos, leaderless, dependencies, fast-path, geo-replication]
status: ingested
---

# There Is More Consensus in Egalitarian Parliaments

## One-sentence summary
EPaxos is a leaderless Paxos variant that commits non-interfering commands in one round trip while spreading load across all replicas.

## Why this paper matters
It combines fast-path quorum reasoning with dependency-based command ordering, making it a key source for [[leaderless-protocols]], [[dependency]], [[conflict]], and [[recovery]].

## System model
State machine replication over `N = 2F + 1` replicas. Clients can submit commands to any replica; each replica can act as the command leader for its own instances.

## Fault model
Non-Byzantine crash/omission behavior. The paper assumes at most `F` faulty replicas and requires a majority of replicas to be available for progress.

## Timing assumptions
Safety is asynchronous. Liveness is probabilistic/conditional: commands eventually commit while a majority is nonfaulty; recovery relies on timeouts/failure suspicion.

## Main idea
EPaxos separates instance choice from execution order. Each command is committed with attributes `(seq, deps)`; replicas execute by following the dependency graph and sequence numbers.

## Protocol roles
Clients submit commands; any replica may be a command leader; other replicas pre-accept, accept, commit, and recover instances.

## Message types
`Request`, `PreAccept`, `PreAcceptOK`, `Accept`, `AcceptOK`, `Commit`, `Prepare`, `PrepareOK`, and optimized recovery messages such as `TryPreAccept`.

## Local state
Each replica stores a log indexed by replica/instance, with command, `seq`, `deps`, and status such as pre-accepted, accepted, or committed.

## Normal path
A command leader assigns a fresh local instance, computes initial `seq` and `deps` from interfering commands, sends `PreAccept` to a fast quorum, and collects replies.

## Fast path
The leader commits after Phase 1 if it receives matching `(seq, deps)` evidence from a fast quorum. In the optimized presentation with `N = 2F + 1`, the fast quorum has total size `F + floor((F + 1)/2)`, including the command leader.

## Slow path
If replies disagree or the leader lacks enough fast-path replies, it unions dependencies, raises `seq` to the maximum reply sequence, and runs the Paxos-Accept phase with a majority.

## Recovery path
Explicit Prepare is run for a possibly failed replica's instance. A recovering replica gathers at least `floor(N/2) + 1` replies and chooses among committed, accepted, or sufficiently witnessed pre-accepted tuples; otherwise it may finalize an empty/no-op-style outcome. Optimized recovery uses `TryPreAccept` and must preserve dependency safety.

## Commit rule
Fast path: identical attributes from the fast quorum. Slow path: majority `AcceptOK`s for the chosen `(cmd, seq, deps)`. Commit notifications are then sent asynchronously.

## Quorum system
`N = 2F + 1`. Majority quorums are `floor(N/2) + 1 = F + 1`. The paper highlights that EPaxos can use smaller fast-path quorums than Generalized Paxos; for `N = 3`, fast-path commit can use the nearest peer plus the leader.

## Conflict handling
Commands interfere when their serial order affects results. Interfering commands acquire dependencies so that replicas independently compute a compatible execution order.

## Safety argument
The commit protocol ensures at most one tuple is committed per instance and that committed tuples are safe. Execution consistency follows because interfering committed commands appear in at least one another's dependency graph.

## Liveness argument
Commands eventually commit under majority availability and client retries. Execution can wait for dependencies; long dependency chains affect execution latency, not necessarily commit latency.

## Key proof ideas
- Proposition: if a replica commits `gamma` at instance `Q.i`, any replica committing at `Q.i` commits the same command and attributes.
- Lemma: if interfering commands commit, the dependency relation orders them.
- Lemma: if `delta` is proposed after `gamma` commits and they interfere, `delta` depends on `gamma` or is in the same dependency component with larger sequence order.

## Important formulas
- `N = 2F + 1`.
- Majority: `floor(N/2) + 1`.
- Fast-path optimized quorum: `F + floor((F + 1)/2)` replicas total, including the command leader. If counting only non-leader replies, subtract one.
- Command attributes: `(gamma, seq_gamma, deps_gamma)`.

## Relationship to other protocols
EPaxos draws on Paxos, Fast Paxos, Generalized Paxos, and Mencius. [[SwiftPaxos]] reuses dependencies but constrains them to remain acyclic; [[Pando]] uses EPaxos as a geo-replication baseline.

## Modeling notes

### Rocq/Coq modeling notes
Separate instance safety from execution-order safety. Model interference as an explicit relation and prove that committed interfering commands are related by dependency reachability or sequence ordering.

## Limitations
Execution may lag commit under dependency chains or high conflict. Exact optimized recovery is subtle; use the technical report for full proofs.

## Open questions
- TODO: Extract the full optimized recovery proof obligations from the EPaxos technical report cited by the paper.

## Related pages
[[EPaxos]], [[dependency]], [[conflict]], [[leaderless-protocols]], [[recovery]], [[linearizability]]
