---
type: protocol
name: SwiftPaxos
family: paxos / dependency-based SMR
papers: [SwiftPaxos-2024]
tags: [consensus]
---

# SwiftPaxos

## Short description
Leader/ballot protocol with leader-including fast quorums, FastAck/SlowAck evidence, and acyclic dependencies.

## Problem solved
Agreement/replication with lower wide-area latency than a simple single-leader design.

## System model
See primary paper page [[SwiftPaxos-2024]]; this page records protocol-level facts only.

## Fault model
Non-Byzantine faults.

## Timing assumptions
Safety is asynchronous; liveness depends on the progress assumptions of the source paper.

## Roles
See [[SwiftPaxos-2024]].

## Message types
See [[SwiftPaxos-2024]].

## Local state
See [[SwiftPaxos-2024]].

## Normal path
See [[SwiftPaxos-2024]].

## Fast path
See [[SwiftPaxos-2024]].

## Slow path
See [[SwiftPaxos-2024]].

## Recovery
See [[SwiftPaxos-2024]].

## Commit condition
See [[SwiftPaxos-2024]].

## Quorum requirement
See [[SwiftPaxos-2024]] and [[quorum-systems]].

## Safety intuition
Quorum intersection prevents incompatible committed outcomes; dependency-based protocols additionally prove compatible execution order.

## Liveness intuition
Requires enough nonfaulty replicas/data sites and eventual stable recovery/leadership where applicable.

## Strengths
Lower latency in its target common case.

## Weaknesses
More complex recovery and quorum reasoning than classic Paxos.

## Differences from related protocols
See [[protocol-catalog]] and dimension pages.

## Modeling notes
Start from abstract state, message evidence, quorum assumptions, and commit predicate before optimizing the real protocol.

## Open questions
- TODO: Expand this protocol page with examples after more query-driven use.

## Sources
- [[SwiftPaxos-2024]]
