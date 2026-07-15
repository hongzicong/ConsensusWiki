---
type: protocol
name: WPaxos
family: Paxos / WAN multi-leader
papers: [WPaxos-2020]
tags: [paxos, wan, flexible-quorum, multi-leader, object-stealing, locality]
---

# WPaxos

## Short description

WPaxos is a WAN multi-leader Paxos protocol with one leader/log per object, large cross-zone Phase 1 quorums for ownership transfer, and small local Phase 2 quorums for repeated commits.

## Problem solved

It adapts object placement to shifting WAN access locality without a separate placement master and avoids paying a wide-area quorum on every request after ownership stabilizes.

## System model

Asynchronous message-passing nodes are arranged in `Z` zones with `l` nodes per zone. Each basic command accesses one object; each object has independent ballots, slots, owner, and log.

## Fault model

Non-Byzantine crash/recovery and message loss. Quorums are configured for `f_z` zone failures and `f_n` node failures per zone. Availability depends on failure placement.

## Timing assumptions

Asynchronous Paxos safety. Progress requires at least one live `Q1` and `Q2`, message delivery, and eventual resolution of leader contention. WAN/local latency separation motivates quorum placement but is not a safety assumption.

## Roles

Client, per-object owner/leader, candidate owner, acceptor, and learner. Every node can hold different roles for different objects.

## Message types

`1a`, `1b`, `2a`, `2b`, `3`, client request/forward, and reconfiguration proposals.

## Local state

Per-object `ballots[o]`, `slots[o]`, ownership set `own`, and `log[o][s] = (ballot, value, committed)`; pending requests and optional locality counters.

## Normal path

The current owner assigns the next slot and sends `2a` to a nearby `Q2`. Matching `2b` replies from one `Q2` commit; message `3` disseminates the decision. Slots may pipeline but execute without gaps.

## Fast path

Stable ownership skips Phase 1 and uses one local leader-to-`Q2` Phase 2. This is an optimized common path, not a Fast Paxos leader-bypassing fast round.

## Slow path

A non-owner runs WAN Phase 1 over `Q1` to steal the object. Higher ballots reject stale candidates; ID tie-breaking and randomized backoff mitigate dueling steals.

## Recovery

The new owner gathers `1b` from `Q1`, which intersects every earlier `Q2`, then recovers all accepted uncommitted slots before new proposals. The displayed algorithm does not make the full accepted-value payload explicit, so that subrule remains Unclear.

## Commit condition

One object slot commits after all members of any valid `q2 ∈ Q2` return matching `2b(o, ballot, slot)`; the leader then broadcasts `3`.

## Quorum requirement

```text
|Q1| = (f_n + 1)(Z - f_z)
|Q2| = (l - f_n)(f_z + 1)
∀q1 ∈ Q1, ∀q2 ∈ Q2 : q1 ∩ q2 ≠ ∅
```

`Q1` is the recovery/ownership quorum; `Q2` is the commit quorum; there is no separate fast quorum. Concrete quorums are topology-flexible within the zone/node constraints. The implementation places `Q2` near the leader, but the formal family does not require leader inclusion explicitly.

## Safety intuition

Every later ownership quorum intersects every earlier deciding quorum in a zone and then in a node. That acceptor either exposes the accepted state or blocks the stale owner. Per-object ballots isolate ownership changes, and per-object slots give linearizability.

## Liveness intuition

An owner progresses with a nearby `Q2`. Ownership changes require the larger `Q1` as well. If all `Q1`s are unavailable, already-owned objects may remain partially available where a `Q2` survives.

## Strengths

- Low-latency repeated commits near current access locality.
- Concurrent leaders for different objects.
- Ownership movement uses Paxos Phase 1 rather than an external service.
- Explicit topology/fault-placement quorum design.
- Per-object linearizability and pipelining.

## Weaknesses

- WAN Phase 1 on first access or migration.
- Potential locality thrashing and leader dueling.
- Placement-dependent availability and per-object metadata.
- Basic algorithm covers single-object commands.
- Accepted-value recovery payload is not explicit in displayed pseudocode.
- Byzantine faults are excluded.

## Differences from related protocols

Unlike [[FPaxos]], WPaxos combines flexible quorums with many simultaneous per-object leaders. Unlike EPaxos, it avoids WAN fast quorums by maintaining ownership rather than opportunistically leading each command. Unlike static shard-local Paxos, [[object-stealing]] changes placement through Phase 1.

## Open questions

- Resolve the `F + 1` versus `f_z + 1` Phase 2 zone notation.
- Specify and prove complete unfinished-slot recovery.
- Design migration policies resistant to oscillating locality.
- Extend the model to multi-object operations.

## Sources

- [[WPaxos-2020]] - primary paper, especially §§3-5.
