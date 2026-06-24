---
type: protocol
name: Fast Paxos
family: paxos / fast consensus
papers: [[[FastPaxos-2006]]]
tags: [consensus]
---

# Fast Paxos

## Short description
Classic Paxos with fast rounds using `phase2a any`; fast path chooses when a fast quorum accepts the same value.

## Problem solved
Agreement/replication with lower wide-area latency than a simple single-leader design.

## System model
See primary paper page [[FastPaxos-2006]]; this page records protocol-level facts only.

## Fault model
Non-Byzantine faults.

## Timing assumptions
Safety is asynchronous; liveness depends on the progress assumptions of the source paper.

## Roles
See [[FastPaxos-2006]].

## Message types
See [[FastPaxos-2006]].

## Local state
See [[FastPaxos-2006]].

## Normal path
See [[FastPaxos-2006]].

## Fast path
See [[FastPaxos-2006]].

## Slow path
See [[FastPaxos-2006]].

## Recovery
See [[FastPaxos-2006]].

## Commit condition
See [[FastPaxos-2006]].

## Quorum requirement
See [[FastPaxos-2006]] and [[quorum-systems]].

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
- [[FastPaxos-2006]]
