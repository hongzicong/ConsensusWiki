---
type: protocol
name: Pando
family: geo-distributed storage / paxos
papers: [Pando-2020]
tags: [consensus]
---

# Pando

## Short description
Paxos-style protocol for erasure-coded geo-storage using Phase 1a, Phase 1b, and Phase 2 quorums.

## Problem solved
Agreement/replication with lower wide-area latency than a simple single-leader design.

## System model
See primary paper page [[Pando-2020]]; this page records protocol-level facts only.

## Fault model
Non-Byzantine faults.

## Timing assumptions
Safety is asynchronous; liveness depends on the progress assumptions of the source paper.

## Roles
See [[Pando-2020]].

## Message types
See [[Pando-2020]].

## Local state
See [[Pando-2020]].

## Normal path
See [[Pando-2020]].

## Fast path
See [[Pando-2020]].

## Slow path
See [[Pando-2020]].

## Recovery
See [[Pando-2020]].

## Commit condition
See [[Pando-2020]].

## Quorum requirement
See [[Pando-2020]] and [[quorum-systems]].

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
- [[Pando-2020]]
