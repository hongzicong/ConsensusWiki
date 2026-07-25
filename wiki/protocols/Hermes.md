---
type: protocol
name: Hermes
family: membership-based reliable replication
papers: [Hermes-2020]
tags: [membership-based-replication, invalidation, logical-timestamp, local-read, linearizability]
---

# Hermes

## Short description

Hermes is a decentralized, broadcast-based read-one/write-all replication protocol that makes local reads linearizable by invalidating every live replica before completing a write.

## Problem solved

Hermes targets in-memory datastores that need local load-balanced reads, writes initiated at any replica, concurrent writes to independent keys, low exposed write latency, and recovery from crash-stop and network failures.

## System model

One datacenter-local shard is replicated on a typical 3 to 7 nodes. Every operational replica stores the shard and may serve single-key reads, writes, and RMWs. A [[reliable-membership]] service provides a leased live-node set and `epoch_id`.

## Fault model

Non-Byzantine crash-stop node failures plus message reordering, duplication, loss, link failures, and network partitions. Only the majority/primary partition remains available during a partition; RM changes membership after old leases expire.

## Timing assumptions

The main design assumes partial synchrony and loosely synchronized clocks for membership leases. Logical timestamps, not physical clocks, order key updates. `mlt` only triggers retries. A no-LSC variant delays a read until a later write or majority membership check validates the reader's epoch.

## Roles

- Any operational replica may be the **coordinator** for one update.
- Other live replicas are **followers** for that update.
- Any invalidated replica may become a **replay coordinator**.
- RM maintains leased membership and epochs.
- A joining node is a **shadow replica** until state transfer completes.

## Message types

- `INV(key, epoch_id, TS, RMW_flag, value)`
- `ACK(key, epoch_id, TS)`
- `VAL(key, epoch_id, TS)`
- RM `m-update` containing lease renewal, live-node list, and incremented epoch
- join notification/acknowledgment and optional no-LSC membership check

## Local state

- per key: value, `version`, `cid`, `RMW_flag`, and state;
- pending update: `mlt`, `ack_bit_vector`;
- per node: `lease`, `epoch_id`, `live_nodes`;
- states: stable `Valid`, `Invalid`, `Write`, `Replay`; transient `Trans`.

## Normal path

A local read returns only from `Valid`. For a write, the coordinator increments `[version, cid]`, applies the value, broadcasts `INV`, and waits for acknowledgments from all live followers. It then replies and broadcasts `VAL`. A follower adopts only a higher timestamp, always acknowledges, and validates only an equal timestamp.

## Fast path

The failure-free write sequence is `INV -> ACK -> VAL`. The complete protocol is 1.5 replica RTTs, while client-visible completion at an internal coordinator is one RTT because `VAL` is off the critical path. Reads are local with no messages when the key is `Valid`.

Hermes does not have a smaller fast quorum or a conflict fallback. Its normal data path is the write-all path.

## Slow path

Reads and new writes stall on a non-`Valid` key. Lost messages cause retransmission or replay. A failed member blocks acknowledgments until RM installs a new epoch. Losing concurrent RMWs abort; ordinary writes do not.

## Recovery

An invalidated replica replays the stored update by broadcasting `INV` with the original value and timestamp, collects acknowledgments from the current live membership, and validates. Membership recovery is external: RM expires old leases, installs a new live set and epoch, and fences stale messages. Joining replicas shadow new writes while copying old state.

## Commit condition

An ordinary write completes when every member of the current live membership has applied or acknowledged at least its timestamp. The coordinator's local copy is implicit, so it waits for `|M_e| - 1` remote follower acknowledgments. `VAL` is not part of client commit.

An RMW commits only if it has the highest timestamp among concurrent updates to the key.

## Quorum requirement

- replication degree: typically 3 to 7, not fixed to `2f + 1`;
- normal/fast acknowledgment set: all of epoch membership `M_e`;
- replay acknowledgment set: all of `M_e`;
- classic majority quorum: none in the data path;
- coordinator inclusion: implicit through its local applied copy;
- configuration change: majority-based RM plus lease expiration and `epoch_id` fencing.

Do not model `M_e` as an arbitrary majority. Linearizable local reads rely on every operational replica being invalidated before client completion.

## Safety intuition

A replica returns a value only while `Valid`. Before a write completes, all operational replicas have seen that timestamp or a higher one, so none can return an older value. Lexicographic `[version, cid]` ordering makes every replica select the same concurrent-write order. Exact-match `VAL` handling makes delayed validations harmless, and replaying the original timestamp/value cannot invent a different order.

## Liveness intuition

Stable, fully connected memberships complete in one exposed replica round trip. Missing messages are retried; invalid keys are replayed. A failed member pauses the write-all path until leased RM removes it. Only the primary partition continues during a split.

## Strengths

- Local linearizable reads at every operational replica.
- Decentralized writes with no fixed leader.
- Inter-key concurrency and deterministic same-key write resolution.
- Ordinary writes never abort under conflict.
- Self-contained replay evidence in every `INV`.
- Delayed, duplicated, and reordered messages are filtered by timestamps and epochs.

## Weaknesses

- Write-all sensitivity to a slow or failed current member.
- Dependence on an external reliable-membership/lease service.
- Stalled reads while a key is invalid.
- Aborting RMWs under concurrent updates.
- No reliable multi-key transactions.
- Datacenter-local and non-Byzantine scope.

## Differences from related protocols

Unlike Paxos-family protocols, Hermes does not select a value from intersecting data quorums. Unlike CRAQ, it has no fixed head/tail chain and avoids sequential write propagation. Unlike [[CURP]], it obtains early completion by invalidating all operational replicas, not through an unordered witness tier.

## Open questions

- Specify a formal composition contract between Hermes and RM across epoch changes.
- Quantify replay amplification under aggressive `mlt` settings.
- State the shadow-replica snapshot invariant precisely.
- Extract the full TLA+ state-space bounds from the external artifact; they are not in the PDF.

## Sources

[[Hermes-2020]]
