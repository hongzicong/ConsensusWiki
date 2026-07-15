---
type: paper
title: "WPaxos: Wide Area Network Flexible Consensus"
authors: [Ailidani Ailijiang, Aleksey Charapko, Murat Demirbas, Tevfik Kosar]
year: 2020
venue: "IEEE Transactions on Parallel and Distributed Systems, 31(1)"
source: raw/wpaxos.pdf
protocols: [WPaxos]
tags: [paxos, wan, flexible-quorum, multi-leader, object-stealing, locality]
status: ingested
---

# WPaxos: Wide Area Network Flexible Consensus

## One-sentence summary

WPaxos is a WAN multi-leader Paxos protocol that assigns each object its own leader and log, uses a large cross-zone Phase 1 quorum to move ownership, and uses a small leader-local Phase 2 quorum for repeated low-latency commits.

## Why this paper matters

Static per-region Paxos partitions perform poorly when access locality moves, while leaderless protocols pay WAN quorum latency on every command. WPaxos makes ownership dynamic: a nearby node can steal one object's leadership through ordinary Paxos Phase 1, then commit subsequent updates through geographically close Phase 2 quorums.

The paper turns [[FPaxos]]'s cross-phase-only intersection into a two-dimensional zone/node quorum construction. The resulting failure tolerance depends on where failures occur, not only how many occur.

## System model

- Asynchronous message-passing nodes deployed across `Z` zones.
- A zone is an availability/geographic isolation unit, such as a site or datacenter.
- The quorum section uses `l` nodes per zone; total acceptors are therefore `N_total = Zl` in this page's disambiguating notation.
- Every node can lead some objects and act as acceptor for other leaders.
- Each command in the basic algorithm accesses exactly one object.
- Every object has an independent increasing slot sequence, ballot, ownership state, and commit log.
- Ballots are unique lexicographically ordered pairs `(counter, node ID)`.

Source: `raw/wpaxos.pdf`, §§3-4.

## Fault model

- Non-Byzantine crash/recovery and arbitrary message loss are within the Paxos model described by the paper.
- Safety is intended to survive asynchronous execution and concurrent leaders.
- `f_z` is the number of zone failures the quorum layout is configured to tolerate.
- `f_n` is the number of node failures a zone can tolerate before losing availability.
- The default configuration tolerates one zone failure and minority node failures per zone.
- Availability is topology dependent; the paper gives worst- and best-placement bounds `F_min` and `F_max`.

## Timing assumptions

Safety is asynchronous and ballot/quorum based. The paper states Paxos-style liveness: progress is possible while at least one valid `q1 ∈ Q1` and one valid `q2 ∈ Q2` remain alive. As with Paxos, practical progress also requires message delivery and resolution of competing leaders; WPaxos uses ballot tie-breaking and randomized backoff for dueling steal attempts.

WAN latency is a performance assumption, not a safety premise. Phase 1 spans many zones but is intended to be rare; repeated Phase 2 quorums are placed in the leader's zone and nearby zones.

## Main idea

WPaxos assigns ownership at object granularity:

1. A node runs Phase 1 over a cross-zone `Q1` to become leader for one object.
2. Cross-phase intersection makes every old Phase 2 quorum expose or block earlier accepted values.
3. The owner runs repeated Phase 2 instances for that object's slots over a small, nearby `Q2`.
4. When access locality changes, another node repeats Phase 1 with a higher per-object ballot and steals ownership.

This partitions the conflict space dynamically while preserving per-object linearizability.

## Protocol roles

- **Client:** sends an object command, normally to a node in its local zone.
- **Object leader/owner:** holds the object's current ballot, assigns slots, recovers unfinished slots, and drives Phase 2/commit.
- **Candidate owner:** starts Phase 1 to create or steal an object.
- **Acceptor:** promises per-object ballots and accepts per-object slot values.
- **Learner/replica:** records Phase 3 commit notifications and executes the per-object log in slot order.

Every node may hold all roles for different objects.

## Message types

- `1a(o, b, sender)`: prepare/ownership request to `Q1`.
- `1b(o, b, sender, highest slot)`: promise/ownership acknowledgement; the prose also requires prior unresolved values to be recovered.
- `2a(o, b, s, v, sender)`: accept proposal to `Q2`.
- `2b(o, b, s, sender)`: accept acknowledgement.
- `3(o, b, s, v, sender)`: commit notification.
- Client request and redirect/forwarding traffic.
- Reconfiguration proposals for joint old/new and final new configurations.

## Local state

At every node:

- `ballots[o]`: highest known ballot for object `o`;
- `slots[o]`: highest slot for `o`;
- `own`: set of objects currently led by the node;
- `log[o][s] = (b, v, c)`: accepted ballot, value, and committed flag;
- received-message set and pending client queue;
- optional access-frequency state for locality-adaptive migration.

Keeping ballots per object avoids out-balloting every object held by a remote leader and reduces leader dueling.

## Normal path

For an object already in `own`:

1. The owner increments the object's slot and sends `2a` to a nearby `Q2`.
2. An acceptor accepts if the proposal ballot is at least its known ballot, stores `(b, v, false)`, and returns `2b`.
3. After one `Q2` acknowledges the same `(o, b, s)`, the leader marks the slot committed and broadcasts `3`.
4. Logs may commit out of order, but execution is in slot order without gaps.

Phase 2 may be pipelined across slots.

## Fast path

The stable-owner common path skips Phase 1 and repeats one leader-driven Phase 2 over a geographically local `Q2`. It is the paper's low-latency path, but it is not a Fast Paxos-style fast ballot: clients do not bypass the owner, and the commit predicate is ordinary Phase 2 acceptance.

## Slow path

A local node that does not own the object runs Phase 1 across `Q1`. This WAN round either establishes ownership and recovers previous accepted work or is rejected by a higher ballot. Contending candidates use zone/node ID tie-breaking and randomized backoff.

Locality-adaptive mode may avoid stealing on the first remote request. The current owner can forward low-volume remote traffic and initiate handover only when another zone becomes the majority requester.

## Recovery path

### Object stealing / leader replacement

The candidate chooses a higher per-object ballot and gathers `1b` promises from `Q1`. Because every `Q1` intersects every old `Q2`, at least one acceptor exposes the earlier ballot/slot state or has already promised the higher ballot and blocks stale Phase 2. The new owner recovers accepted but uncommitted slots before proposing new work.

**Unclear:** Algorithm 2's displayed `1b` payload includes the highest slot but not an explicit accepted-value log, while the prose says the new leader recovers unresolved commands with suggested values. The full recovery payload/value-selection rule should be checked against the TLA+ artifact or implementation before formalization.

### Reconfiguration

For arbitrary membership/quorum change from `C = ⟨Q1, Q2⟩` to `C' = ⟨Q'1, Q'2⟩`, WPaxos adopts Raft-style two-phase joint configuration so old and new configurations cannot decide divergent states concurrently. Common one-zone or one-row changes can collapse to one phase under:

```text
Q'1 ∪ Q1 = Q'1
Q'2 ∪ Q2 = Q'2
```

## Commit rule

A slot `(o, b, s)` commits when the leader receives matching `2b` replies from every node in some `q2 ∈ Q2`:

```text
Q2Satisfied(o, b, s) ≜
  ∃q ∈ Q2 : ∀n ∈ q : ∃m ∈ msgs :
    m.type = "2b" ∧ m.o = o ∧ m.b = b ∧ m.s = s ∧ m.n = n
```

The leader then sets `log[o][s].c = true` and broadcasts message `3`.

## Quorum system

Let:

- `Z` = number of zones;
- `l` = nodes per zone;
- `N_total = Zl` = total nodes (disambiguating notation used by this page);
- `f_z` = configured zone-failure tolerance;
- `f_n` = configured per-zone node-failure tolerance.

Phase 1 selects `f_n + 1` nodes from each of `Z - f_z` zones:

```text
|Q1| = (f_n + 1)(Z - f_z)
```

Phase 2 selects `l - f_n` nodes from each of `f_z + 1` zones:

```text
|Q2| = (l - f_n)(f_z + 1)
```

Required intersection:

```text
∀q1 ∈ Q1, ∀q2 ∈ Q2 : q1 ∩ q2 ≠ ∅
```

Proof: the zone sets overlap because:

```text
(Z - f_z) + (f_z + 1) = Z + 1 > Z
```

Within a common zone, node sets overlap because:

```text
(f_n + 1) + (l - f_n) = l + 1 > l
```

Additional details:

- **Fast quorum:** none distinct from `Q2`.
- **Classic/commit quorum:** one `Q2`.
- **Recovery/ownership quorum:** one `Q1`.
- **Leader inclusion:** implementation selects `Q2` around the leader; the formal quorum definition does not state a separate must-include-leader constraint.
- **Fixed/flexible:** families are topology-structured; a concrete quorum can vary subject to zone/node counts.
- **Replication quorum `RQ2`:** Phase 2 may be sent to a selected subset of `Q2` quorums, from the minimum replication zones up to all `Z` zones.

The paper states topology-dependent failure-placement bounds:

```text
F_min = Min(Cardinality(Q2), Cardinality(Q1)) - 1

F_max = N - Cardinality(Q1) - Cardinality(Q2)
        + (f_z + 1)(f_n + 1)
```

Here `N` is the paper's total-acceptor notation in these formulas.

**Unclear:** §4.2 says Phase 2 resides in the closest `F + 1` zones, while the formal quorum definition uses `f_z + 1` zones. This page retains the formal definition and records the discrepancy.

## Conflict handling

Commands on different objects use different logs and leaders and may proceed concurrently. Commands on the same object are totally ordered by that object's `(ballot, slot)` log. Thus the global history is partially ordered while each object is linearizable.

Concurrent steals of one object are resolved by higher ballots; equal counters are ordered by zone/node ID, and repeated contention uses randomized backoff.

## Safety argument

The paper states:

- **Non-triviality:** committed commands are proposed commands.
- **Stability:** a node's committed sequence is a prefix of its later committed sequence.
- **Consistency:** no two leaders commit different values for the same slot of the same object.

Safety follows the Paxos invariant at object granularity. A later `Q1` intersects every earlier deciding `Q2`; the common acceptor either reports the accepted state to the new owner or rejects the stale owner after promising the higher ballot. Separate per-object ballots prevent ownership changes for one object from disturbing another.

The authors model checked the consistency property in a TLA+/PlusCal specification.

## Liveness argument

WPaxos states ordinary Paxos liveness: progress continues while some valid `q1` and `q2` are alive. Object stealing needs both families. If failures prevent every `Q1`, already-owned objects in surviving regions can remain partially available as long as their owners can form `Q2`; new ownership transfers halt.

Randomized backoff mitigates dueling candidates but the paper does not give a separate termination proof for sustained adversarial contention.

## Key proof ideas

- Apply [[flexible-quorum]] intersection separately across zones and within one common zone.
- Keep ballot, ownership, slot, and log state per object.
- Reuse Phase 1 as both leader election and data-movement authority; do not add an external placement master.
- Preserve every accepted unfinished slot before the new owner opens later slots.
- Distinguish per-object agreement from global total order.
- During reconfiguration, forbid old and new quorum systems from deciding independently.

## Important formulas

Quorum sizes:

```text
|Q1| = (f_n + 1)(Z - f_z)
|Q2| = (l - f_n)(f_z + 1)
```

Cross-phase intersection:

```text
∀q1 ∈ Q1, ∀q2 ∈ Q2 : q1 ∩ q2 ≠ ∅
```

Intersection arithmetic:

```text
Z - f_z + f_z + 1 = Z + 1 > Z
l - f_n + f_n + 1 = l + 1 > l
```

Failure-placement bounds:

```text
F_min = Min(Cardinality(Q2), Cardinality(Q1)) - 1
F_max = N - Cardinality(Q1) - Cardinality(Q2) + (f_z + 1)(f_n + 1)
```

Special-case one-phase reconfiguration:

```text
Q'1 ∪ Q1 = Q'1
Q'2 ∪ Q2 = Q'2
```

## Relationship to other protocols

- [[FPaxos]] supplies the cross-phase-only quorum principle; WPaxos adds multiple object leaders and WAN grid quorums.
- EPaxos allows any node to lead each command and uses large fast quorums/dependencies; WPaxos assigns each object to one current owner and localizes repeated Phase 2.
- Static multi-Paxos partitioning fixes object placement; WPaxos moves it through [[object-stealing]].
- M2Paxos is multi-leader but does not use WPaxos's multi-quorum locality structure.
- Spanner-style placement uses an external relocation/configuration mechanism; WPaxos integrates ownership movement into Phase 1.

## Limitations

- Basic commands access exactly one object; multi-object transactions are outside the algorithm.
- Skewed or rapidly shifting locality can cause ownership thrashing and repeated WAN Phase 1.
- Per-object ballots/logs and locality tracking add metadata and cache state.
- Failure tolerance depends on placement; one count does not characterize availability.
- A small/remote `RQ2` can delay a new leader outside the replication set while it catches up.
- Recovery value transfer is under-specified in the displayed PlusCal algorithms.
- The paper model checks bounded TLA+ executions rather than giving a mechanized unbounded proof.
- Byzantine failures are excluded.

## Open questions

- What exact `1b` evidence and value-selection rule recovers every accepted unfinished slot?
- Is `F + 1` in §4.2 intended to mean `f_z + 1` zones?
- Which online migration policy avoids object thrashing while preserving locality benefit?
- How should multi-object operations compose several independently owned logs?
- Can `Q1/Q2/RQ2` placement be optimized dynamically without unsafe quorum-family drift?
- How does one prove the special one-phase reconfiguration cases for arbitrary pipelined slots?

## Related pages

[[WPaxos]], [[object-stealing]], [[FPaxos]], [[flexible-quorum]], [[quorum]], [[leader]], [[recovery]], [[fast-path]], [[conflict]], [[agreement]], [[linearizability]], [[liveness]], [[quorum-intersection]], [[protocol-catalog]], [[quorum-systems]], [[leader-roles]], [[commit-rules]], [[recovery-rules]], [[proof-techniques]], [[paxos-family]]
