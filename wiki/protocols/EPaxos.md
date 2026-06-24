---
type: protocol
name: EPaxos
family: paxos / leaderless
papers: [EPaxos-2013, EPaxos-Revisited-2021]
tags: [consensus]
---

# EPaxos

## Short description
Leaderless SMR where each replica can lead commands, and conflicting commands are ordered by dependencies.

## Problem solved
Agreement/replication with lower wide-area latency than a simple single-leader design.

## System model
See primary paper page [[EPaxos-2013]]; this page records protocol-level facts only.

## Fault model
Non-Byzantine faults.

## Timing assumptions
Safety is asynchronous; liveness depends on the progress assumptions of the source paper.

## Roles
See [[EPaxos-2013]].

## Message types
See [[EPaxos-2013]].

## Local state
See [[EPaxos-2013]].

## Normal path
See [[EPaxos-2013]].

## Fast path
See [[EPaxos-2013]]. [[EPaxos-Revisited-2021]] warns that fast-path commit can still have delayed execution when transitive dependencies have not committed.

## Slow path
See [[EPaxos-2013]].

## Recovery
See [[EPaxos-2013]].

## Commit condition
See [[EPaxos-2013]].

## Quorum requirement
See [[EPaxos-2013]] and [[quorum-systems]].

## Safety intuition
Quorum intersection prevents incompatible committed outcomes; dependency-based protocols additionally prove compatible execution order.

## Liveness intuition
Requires enough nonfaulty replicas/data sites and eventual stable recovery/leadership where applicable.

## Strengths
Lower latency in its target common case.

## Weaknesses
More complex recovery and quorum reasoning than classic Paxos. Performance is highly workload-sensitive: [[EPaxos-Revisited-2021]] reports that median commit latency can hide poor tail execution latency under conflicts, batching, and dependency chains.

## Differences from related protocols
See [[protocol-catalog]] and dimension pages.

## Modeling notes
Start from abstract state, message evidence, quorum assumptions, and commit predicate before optimizing the real protocol.

## Open questions
- TODO: Expand this protocol page with examples after more query-driven use.

## Sources
- [[EPaxos-2013]]
- [[EPaxos-Revisited-2021]]
