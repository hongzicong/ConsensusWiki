---
type: paper
title: "Hydra: Serialization-Free Network Ordering for Strongly Consistent Distributed Applications"
authors: [Inho Choi, Ellis Michael, Yunfan Li, Dan R. K. Ports, Jialin Li]
year: 2023
venue: "20th USENIX Symposium on Networked Systems Design and Implementation (NSDI '23)"
source: raw/hydra.pdf
protocols: [Hydra, HydraPaxos]
tags: [network-ordering, groupcast, multi-sequencer, smr, recovery, physical-clock]
status: ingested
---

# Hydra: Serialization-Free Network Ordering for Strongly Consistent Distributed Applications

## One-sentence summary

Hydra replaces a single in-network sequencer with concurrent sequencers whose monotonic physical clocks, per-group sequence numbers, and flush messages let receivers derive a consistent order and detect drops without serializing all traffic at one device.

## Why this paper matters

Earlier network-ordering systems such as NOPaxos and Eris obtain low application coordination cost by routing every request through one sequencer. Hydra retains their ordered-unreliable-delivery abstraction while removing that sequencer's throughput, routing, and failover bottlenecks.

The paper also shows that Hydra is not itself a complete consensus protocol. [[HydraPaxos]] and HydraTxn combine Hydra's ordering and drop detection with application-level quorum protocols to tolerate replica failures and resolve missing messages.

## System model

- Senders issue groupcast messages to one or more receiver groups.
- A dynamic set of sequencers can run on programmable switches or end hosts.
- Each message is routed through exactly one sequencer, then multicast to every receiver in its destination groups.
- A centralized, fault-tolerant configuration service manages sequencer and receiver membership and numbered configurations.
- The network may reorder, duplicate, delay, or drop packets.
- Hydra provides partial ordering, best-effort delivery, and drop detection; it does not provide reliable delivery or application commit by itself.

Source: `raw/hydra.pdf`, §§3-5.

## Fault model

- Sequencers and links may fail; receivers detect a stalled sequencer by timeout and request its removal.
- Messages may be lost between a sequencer and any subset of receivers.
- Receiver failures are handled by the application-level protocol running within each receiver group, not by the groupcast primitive alone.
- The configuration service is assumed fault tolerant and available; the paper does not give its replica count or consensus implementation.
- Byzantine behavior is outside the model.

For [[HydraPaxos]], `n = 2f + 1` replicas tolerate `f` crash failures.

## Timing assumptions

Safety requires each sequencer's physical clock to be strictly monotonic and never move backward. Clocks need only be loosely synchronized: skew can delay delivery by holding back `c_min`, but does not violate ordering.

Liveness requires receivers eventually to receive groupcasts or flush messages from every non-failed sequencer, plus an available configuration service able to communicate with a quorum of every receiver group. Failure detection uses timeouts.

Source: `raw/hydra.pdf`, §§4.3-4.5.

## Main idea

Each sequencer stamps a groupcast with:

- its sequencer ID `i`;
- its current physical clock value `c`;
- a per-destination-group sequence number in a multi-stamp.

Receivers buffer messages in `(clock, sequencer ID)` order. For every sequencer they track the greatest received clock `c[i]` and group sequence number `s[i]`. They deliver only messages no greater than the minimum frontier `c_min` across sequencers. Sequence-number gaps become `DROP-NOTIFICATION`s; flush messages advance idle sequencers' frontiers.

## Protocol roles

- **Sender:** selects destination groups and sends to a reachable sequencer.
- **Sequencer:** atomically reads a monotonic clock, increments per-group counters, stamps the packet, and multicasts it.
- **Receiver:** buffers, orders, delivers, and emits drop notifications.
- **Receiver group:** application-defined replica group that decides whether a message or drop is durably adopted.
- **Configuration service:** versions membership and coordinates sequencer addition/removal.
- **HydraPaxos leader/replicas/client:** run NOPaxos-derived SMR above one Hydra receiver group.

## Message types

- Groupcast: destination-group bitmap, sequencer ID, per-group multi-stamp, clock value, application payload.
- Flush: sequencer ID, current clock, and latest sequence number for every group, without incrementing counters.
- `DROP-NOTIFICATION`: identifies a missing `(sequencer, sequence number)` before later ordered delivery.
- Sequencer-removal reports and removal messages.
- Sequencer-addition flush reports and configuration-transition messages.
- HydraPaxos operation, replica reply, missing-message recovery, and consensus `NO-OP` traffic.

## Local state

At sequencer `i`:

- unique sequencer ID;
- strictly monotonic physical clock;
- sequence counter `seq[g]` for each receiver group.

At a receiver:

- current configuration number;
- ordered buffer `buf`;
- largest received sequence number `s[i]` for its group from each sequencer;
- largest received clock value `c[i]` from each sequencer;
- for recovery, the largest sequence number seen from each sequencer for every receiver group.

HydraPaxos replicas additionally maintain the NOPaxos-derived operation log, deterministic application state, and replica/leader state.

## Normal path

1. A sender sends the groupcast to one randomly selected reachable sequencer.
2. In one atomic block, the sequencer reads its clock and increments every destination group's sequence counter.
3. The sequencer stamps and multicasts the message.
4. Each receiver updates `c[i]` and `s[i]`, emits notifications for sequence gaps, buffers the message, and delivers buffered messages that are at or below `c_min` in `(clock, sequencer ID)` order.
5. If an idle sequencer prevents progress, a flush advances its clock/sequence frontier without entering the application stream.

## Fast path

Hydra's common path uses one sequencer pass and no receiver-to-receiver coordination. If normal traffic from all sequencers advances `c_min`, receivers deliver without explicit flushes.

[[HydraPaxos]] completes an operation in one client round trip in the normal case: replicas log the Hydra-delivered operation, the leader executes it, and the client waits for consistent replies from a majority including the leader.

This is an application fast path, not a Fast Paxos-style fast quorum.

## Slow path

Flush solicitation is a progress mechanism when an idle sequencer holds back `c_min`; it does not change the safety predicate. Receivers can solicit immediately for latency or batch solicitations for throughput, and a ToR switch can aggregate flushes from multiple sequencers.

In HydraPaxos, a delivered `DROP-NOTIFICATION` triggers missing-message recovery and, if recovery fails, leader-driven agreement to commit the missing position as `NO-OP`.

## Recovery path

### Sequencer removal

1. A receiver suspects sequencer `j` and notifies the configuration service.
2. The service proposes configuration `n + 1` without `j`.
3. Every receiver stops processing higher sequence numbers from `j` and reports its largest observed per-group counters.
4. After a quorum from each receiver group reports, the service derives the highest sequence number each group has or should have received.
5. A removal message acts as `j`'s final flush with an infinite timestamp; receivers emit required `DROP-NOTIFICATION`s, drain pending messages, and enter the new configuration.

### Sequencer addition

Receivers pause application delivery until the new sequencer emits a flush later than their latest delivered message. The service collects a quorum of flush reports from each group, chooses the highest timestamp `t_k`, broadcasts it as the new configuration's start frontier, and initializes the new sequencer's receiver state from that flush.

## Commit rule

Hydra has no consensus commit condition. Its delivery rule is:

```text
deliver pending p only if
p ⪯ min { (c[m], m) : m ∈ 1..M }
```

Within a receiver group, the paper treats a message or `DROP-NOTIFICATION` as group-delivered when every receiver in some application-defined quorum delivers it. If uniform delivery at every receiver is required, the quorum is the whole receiver group.

HydraPaxos client commit requires consistent replies from a majority of `2f + 1` replicas, including the leader.

## Quorum system

Hydra primitive:

- **Total receivers:** application-defined per group.
- **Fast/classic quorum:** none at the sequencing layer.
- **Removal/addition quorum:** one application-defined quorum from every receiver group.
- **Leader inclusion:** not applicable to Hydra groupcast.
- **Uniform-drop option:** quorum size equals the whole receiver group.
- **Intersection obligation:** the receiver-group application must ensure its adoption/recovery quorums make divergent individual deliveries safe.

HydraPaxos:

- `n = 2f + 1` replicas;
- normal commit quorum `f + 1`, including the leader;
- recovery first queries other replicas, then uses leader-driven consensus if the message cannot be found;
- **Unclear:** the paper does not restate an exact recovery-evidence quorum for the inherited NOPaxos procedure.

## Conflict handling

Hydra does not inspect application conflicts. Any groupcasts whose destination-group sets overlap are comparable in its partial order; disjoint groupcasts need not be ordered relative to one another.

HydraPaxos applies deterministic state-machine execution in Hydra delivery order. A missing ordered position is recovered or permanently filled with `NO-OP` before execution advances.

## Safety argument

For two overlapping groupcasts stamped by the same sequencer:

```text
s1 ≠ s2 ∧ (s1 < s2 ⇔ c1 < c2)
```

For messages from sequencers `i` and `j`, Hydra defines:

```text
m1 ≺ m2 iff c1 < c2 ∨ (c1 = c2 ∧ i < j)
```

Receivers deliver only below the minimum sequencer frontier and in this order. Before advancing past a missing per-sequencer number they deliver its `DROP-NOTIFICATION`. Thus a receiver either has the message or has later same-sequencer evidence proving the gap.

Sequencer removal is equivalent to issuing all remaining drop notifications for that sequencer. The group-by-group agreement round fixes how many explicit notifications are needed. Addition chooses a start timestamp later than every quorum-delivered old-configuration message, preventing the new sequencer from introducing an earlier delivery.

The authors model checked Hydra groupcast and sequencer add/remove protocols in TLA+ against the safety properties.

Source: `raw/hydra.pdf`, §4.5 and Appendix B-C.

## Liveness argument

In a stable configuration, periodic or solicited flushes ensure every receiver's `c_min` eventually advances even when some sequencers are idle. Clock skew can add delay but not prevent progress if clocks advance and messages arrive.

After a sequencer failure, progress requires failure detection and an available configuration service that can obtain a quorum from each receiver group. Hydra pauses delivery while this evidence is collected.

## Key proof ideas

- Use physical time only for the cross-sequencer order; use per-sequencer sequence numbers to prove which earlier messages are missing.
- Make clock read and all destination-counter increments one atomic sequencer action.
- Delay delivery until every sequencer frontier rules out a smaller future message.
- Define drop notification as occurring before every later ordered message, making it a safety property rather than eventual-delivery liveness.
- Treat sequencer removal as a finite representation of infinitely many terminal drops.
- Version every application delivery by configuration.

## Important formulas

Ordering:

```text
m1 ≺ m2 iff c1 < c2 ∨ (c1 = c2 ∧ i < j)
```

Same-sequencer clock/sequence consistency:

```text
s1 ≠ s2 ∧ (s1 < s2 ⇔ c1 < c2)
```

Receiver frontier:

```text
c_min = min { (c[i], i) : i ∈ 1..M }
```

HydraPaxos crash tolerance:

```text
n = 2f + 1
commit replies = f + 1, including the leader
```

## Relationship to other protocols

- Hydra supplies the same ordered-unreliable groupcast guarantees used by NOPaxos and Eris, but supports several active sequencers.
- [[HydraPaxos]] inherits NOPaxos's replica-fault and missing-message logic while delegating sequencing-failure handling to Hydra.
- Unlike [[FastPaxos]] or [[WPaxos]], Hydra's sequencers do not vote on consensus values.
- Unlike atomic multicast, Hydra receivers do not coordinate for every message; applications coordinate only when durability or drop resolution requires it.

## Limitations

- Safety assumes sequencer clocks never move backward.
- Delivery latency depends on the slowest sequencer frontier; skew and idle sequencers require flushes.
- The primitive is unreliable and cannot replace replica consensus by itself.
- Reconfiguration depends on a centralized fault-tolerant configuration service and a quorum from every receiver group.
- Receiver-group quorum rules are delegated to the application.
- The TLA+ model treats receiver groups atomically and abstracts the application-level quorum protocol.
- Byzantine sequencers, receivers, and configuration services are not addressed.

## Open questions

- What is the minimal receiver-group quorum condition needed for each class of application using Hydra?
- Can sequencer reconfiguration avoid a centralized configuration service while retaining the same drop guarantee?
- How should monotonic-clock faults be detected and fenced?
- Can the ordering frontier tolerate a slow but non-failed sequencer without frequent flush traffic?
- What exact inherited NOPaxos recovery quorum and durable-state assumptions does HydraPaxos require?

## Related pages

[[Hydra]], [[HydraPaxos]], [[quorum]], [[fast-path]], [[recovery]], [[leader]], [[failure-model]], [[agreement]], [[liveness]], [[linearizability]], [[quorum-intersection]], [[protocol-catalog]], [[commit-rules]], [[recovery-rules]], [[proof-techniques]]
