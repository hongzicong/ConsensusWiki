---
type: protocol
name: Hydra
family: network ordering / ordered unreliable groupcast
papers: [Hydra-2023]
tags: [network-ordering, groupcast, multi-sequencer, physical-clock, drop-detection]
---

# Hydra

## Short description

Hydra is an ordered-unreliable groupcast primitive that merges messages from multiple active network sequencers using monotonic clocks while retaining precise drop detection through per-group sequence numbers.

## Problem solved

Single network sequencers serialize traffic, constrain routing, and couple application recovery to network rerouting. Hydra distributes sequencing without making receivers run agreement for every message.

## System model

Senders route each groupcast through one of several active switch or end-host sequencers. A message names one or more receiver groups, and the sequencer multicasts it to all receivers in those groups. A fault-tolerant configuration service versions membership.

## Fault model

Best-effort network with packet loss, reordering, duplication, sequencer/link failure, and non-Byzantine participants. Receiver failures and durable adoption are delegated to the receiver-group application.

## Timing assumptions

Safety requires strictly monotonic, non-retrograde sequencer clocks, not tight synchronization. Progress requires advancing clocks, delivery of groupcasts/flushes from live sequencers, and configuration-service access to a quorum of every receiver group.

## Roles

Sender, sequencer, receiver, receiver group, and fault-tolerant configuration service.

## Message types

Groupcast, flush, `DROP-NOTIFICATION`, sequencer-removal report/message, sequencer-addition flush report, and numbered configuration transition.

## Local state

Sequencer: ID, monotonic clock, and per-group counter. Receiver: configuration, ordered buffer, per-sequencer largest clock `c[i]`, per-sequencer group counter `s[i]`, and recovery-time cross-group counter observations.

## Normal path

A sequencer atomically stamps clock plus incremented per-destination counters and multicasts. Receivers buffer in `(clock, sequencer ID)` order and deliver only below the minimum frontier from all sequencers; gaps produce drop notifications.

## Fast path

One sequencer pass and local receiver processing. No receiver agreement or explicit flush is needed when normal traffic from every sequencer advances the frontier.

## Slow path

An idle sequencer triggers periodic or receiver-solicited flushes. Flush aggregation can reduce receiver work. This advances the same delivery predicate rather than selecting a different value.

## Recovery

On removal, the configuration service collects the largest observed counters from a quorum in every receiver group and broadcasts a final infinite-time flush that causes all required drops. On addition, it chooses a new-sequencer flush later than all delivered old-configuration messages as the transition frontier.

## Commit condition

Hydra has no application commit. A receiver delivers pending message `p` only if it is no greater than `min{(c[i], i)}`. Group-level delivery is defined by an application quorum; uniform delivery requires the entire group.

## Quorum requirement

No sequencing fast/classic quorum. Addition/removal needs one application-defined quorum from every receiver group. The paper leaves group size, fault parameter, and exact application intersections unspecified.

## Safety intuition

Clock/ID order gives every overlapping receiver a common order. A message can cross the minimum frontier only after each sequencer supplies enough sequence evidence to reveal every earlier message or gap. Reconfiguration turns departed-sequencer suffixes into explicit drops before changing configuration.

## Liveness intuition

Flushes advance idle frontiers. Failed sequencers are removed after timeout and a receiver-group agreement round, allowing remaining sequencers' traffic to resume.

## Strengths

- Concurrent sequencers and path diversity.
- One-pass ordering without per-message receiver agreement.
- Explicit partial-delivery detection.
- Switch and end-host implementations.
- Sequencer failure recovery does not wait for network rerouting.

## Weaknesses

- Unreliable delivery; applications still need consensus/durability logic.
- Non-retrograde physical-clock assumption.
- Slowest sequencer frontier controls delivery latency.
- Central configuration-service dependency.
- Application-defined receiver quorums are abstracted.

## Differences from related protocols

Unlike centralized NOPaxos/Eris sequencers, Hydra merges several independent sequencers. Unlike [[FastPaxos]] or [[WPaxos]], sequencers stamp order metadata but never accept or choose application values. [[HydraPaxos]] is the SMR protocol layered above Hydra.

## Open questions

- Formalize the required receiver-group quorum interface.
- Remove or distribute the configuration service.
- Detect monotonic-clock violations.
- Bound frontier delay under skew and sparse traffic.

## Sources

- [[Hydra-2023]] - primary paper, especially §§4-6 and Appendix B-C.
