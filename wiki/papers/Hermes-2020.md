---
type: paper
title: "Hermes: A Fast, Fault-Tolerant and Linearizable Replication Protocol"
authors: Antonios Katsarakis; Vasilis Gavrielatos; M. R. Siavash Katebzadeh; Arpit Joshi; Aleksandar Dragojevic; Boris Grot; Vijay Nagarajan
year: 2020
venue: 25th ACM International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS '20)
source: raw/hermes.pdf
protocols: [Hermes]
tags: [membership-based-replication, invalidation, logical-timestamp, local-read, linearizability, rdma]
status: ingested
---

# Hermes: A Fast, Fault-Tolerant and Linearizable Replication Protocol

## One-sentence summary

Hermes combines per-key invalidations, Lamport-style logical timestamps, and leased reliable membership so any replica can coordinate a write, all replicas can serve local reads, and interrupted writes can be replayed safely.

## Why this paper matters

Hermes is a useful counterpoint to majority-based consensus protocols. Its failure-free data path is read-one/write-all over the current live membership: a write invalidates every operational replica before the coordinator responds, making a `Valid` local copy safe to read without communication. Majority agreement is moved off the normal data path into the [[reliable-membership]] service that changes the live set.

The paper also presents a reusable recovery pattern. Every invalidation carries both the value and its unique per-key timestamp, so any replica left invalid can replay exactly the same write rather than infer a value from a recovery quorum.

Source: `raw/hermes.pdf`, Abstract, §§1, 2.4, 3.

## System model

- An in-memory datastore inside one local-area network, such as a datacenter.
- Data may be sharded; Hermes independently replicates each shard across a typical replication degree of 3 to 7 nodes.
- Clients establish sessions and issue single-key reads, writes, and read-modify-writes (RMWs).
- Every operational replica stores the keys in its shard and may serve reads or coordinate updates.
- A leased reliable-membership (RM) service supplies each node with a stable live-node set and `epoch_id`.
- The evaluated system, HermesKV, uses RDMA, but the protocol does not require RDMA for correctness.

Source: `raw/hermes.pdf`, §§2.1-2.4, 3, 4.

## Fault model

The paper assumes a partially synchronous, non-Byzantine system with crash-stop process failures. Network faults may reorder, duplicate, or lose messages, and link failures may partition the network.

Hermes preserves consistency during a partition by allowing requests only in the primary partition. The RM service changes membership through a majority-based protocol after old membership leases expire; minority-partition replicas stop serving before the new membership becomes usable.

The paper contrasts an `n - 1` node-failure resilience claim with partition handling when the datastore replicas themselves run RM, where it states that resilience becomes "less than `floor(n/2)` failures." This statement is preserved without silently replacing it with a different majority formula.

Source: `raw/hermes.pdf`, §§2.4, 3.4.

## Timing assumptions

- The main model uses loosely synchronized clocks (LSCs) only for RM lease management.
- The per-key write protocol orders operations with logical timestamps, not physical time.
- `mlt`, the message-loss timeout, triggers retransmission or replay but does not choose a value.
- The paper sketches a no-LSC variant: a speculative local read is returned only after a later committed write or a majority-acknowledged membership check proves that the reader was still in the latest epoch.

Source: `raw/hermes.pdf`, §§2.4, 3.4, 8.

## Main idea

For each key, a replica exposes the local value only in state `Valid`. A coordinator first broadcasts `INV` containing the new value and a unique timestamp. A follower that sees a higher timestamp adopts the value and becomes unreadable, but acknowledges every `INV` even if it is stale. Once every live member has acknowledged, the coordinator can reply to the client because no operational replica can return an older value. A later `VAL` makes matching copies readable again.

Concurrent writes are not aborted. Their per-key timestamps form the same total order at every replica, so a higher write supersedes a lower write locally. RMWs are different: only the highest-timestamp concurrent RMW may commit, and ordinary writes are deliberately assigned higher timestamps than racing RMWs.

## Protocol roles

- **Client:** issues a single-key read, write, or RMW to any operational replica.
- **Coordinator:** the replica that initiates an update; computes its timestamp, broadcasts invalidation, collects acknowledgments, replies, and validates.
- **Follower:** receives invalidations, adopts higher timestamp/value pairs, acknowledges, and validates only a matching timestamp.
- **Replay coordinator:** any operational replica that finds a key invalid beyond `mlt` and retransmits the stored update with its original timestamp.
- **Reliable-membership service:** maintains leased live-node sets and epochs and installs `m-update`s.
- **Shadow replica:** a joining node that acknowledges new writes while copying old key chunks, but serves no clients until fully caught up.

## Message types

- `INV(key, epoch_id, TS, RMW_flag, value)`: invalidates a key and propagates the complete replayable update.
- `ACK(key, epoch_id, TS)`: acknowledges the exact invalidation timestamp; a follower sends it regardless of whether the timestamp won locally.
- `VAL(key, epoch_id, TS)`: makes a key `Valid` only when the timestamp equals the receiver's current timestamp.
- `m-update(lease renewal, live_nodes, incremented epoch_id)`: reliably changes membership after lease expiration.
- Join notification/acknowledgment messages establish a shadow replica before state transfer.
- The no-LSC variant adds a majority-acknowledged membership-check carrying `epoch_id`.

Source: `raw/hermes.pdf`, Figure 3, §§3.2, 3.4, 8.

## Local state

Figure 3 gives the following protocol metadata:

- per key: `key`, timestamp fields `version` and `cid`, `RMW_flag`, and protocol `state`;
- pending update: `mlt` and `ack_bit_vector`;
- per node: `lease`, `epoch_id`, and `live_nodes`;
- key data: the locally stored value associated with the timestamp.

The stable protocol states are `Valid`, `Invalid`, `Write`, and `Replay`; `Trans` is transient. `Trans` records that a coordinator's pending `Write` or `Replay` was invalidated by a higher-timestamp update while its own acknowledgments are still pending.

## Normal path

### Read

An operational replica returns its local value if and only if the key is `Valid`. Reads to `Invalid`, `Write`, `Replay`, or `Trans` keys stall.

### Write coordinator

1. The coordinator starts only from `Valid`; otherwise it stalls.
2. `CTS`: increment the key's local version and append the coordinator ID as `cid`.
3. `CINV`: apply the new value, enter `Write`, and broadcast `INV` to the replica group.
4. `CACK`: after acknowledgments from every live member, complete the write and enter `Valid`, or `Invalid` if the coordinator had entered `Trans`.
5. Reply to the client after the acknowledgment condition is met.
6. `CVAL`: broadcast `VAL` out of the client critical path.

### Write follower

1. `FINV`: if the incoming timestamp is higher, adopt its timestamp and value and enter `Invalid`, or `Trans` if currently in `Write`/`Replay`.
2. `FACK`: always acknowledge the incoming timestamp, even if it was not adopted.
3. `FVAL`: enter `Valid` only if the validation timestamp equals the local timestamp; otherwise ignore it.

Source: `raw/hermes.pdf`, §§3.1-3.2.

## Fast path

Hermes has no separate Fast Paxos-style quorum path. Its failure-free update path is always the broadcast sequence:

```text
INV -> ACK -> VAL
```

The full sequence costs 1.5 replica round trips, but the coordinator exposes only one round trip because it can respond after all acknowledgments and send `VAL` afterward. Local reads require no replica communication when the key is `Valid`.

Optimization O3 has followers broadcast `ACK`s to every replica. Each follower can then validate after seeing all acknowledgments, reducing same-key read blocking from up to one round trip to half a round trip and eliminating separate `VAL` messages.

Source: `raw/hermes.pdf`, §§3.1, 3.3.

## Slow path

Hermes does not have a conflict-triggered write slow path: ordinary writes always commit and concurrent same-key writes are ordered by timestamp. Slow or exceptional behavior is instead:

- stall a read or new write while the local key is not `Valid`;
- retransmit an `INV` when the coordinator's `mlt` expires;
- replay an update when a follower remains `Invalid` beyond `mlt`;
- wait for an RM membership update after a failed member stops acknowledging;
- abort a losing concurrent RMW.

## Recovery path

### Interrupted update

Any operational replica that observes an `Invalid` key beyond `mlt` becomes replay coordinator, enters `Replay`, and repeats `CINV` through `CVAL` using the locally stored value and the original timestamp, including the original coordinator's `cid`. Reusing both fields preserves the established per-key write order.

### Membership transition

After leases expire, RM installs a new live-node set and increments `epoch_id`. A write may begin at a node with the new epoch, but it cannot commit until every live member has received the `m-update`, become operational, and acknowledged. Nodes in an older epoch drop the new `INV`; the coordinator treats this as message loss and retransmits.

### Adding a replica

After a reliable membership update and acknowledgment from existing replicas, the new node operates as a shadow follower. It receives all new writes while copying chunks from existing replicas. Only after the entire datastore is current does it become operational and serve clients.

Source: `raw/hermes.pdf`, §§3.2, 3.4.

## Commit rule

For an ordinary write in epoch `e`, the coordinator may complete and reply after receiving `ACK`s for its `TS` from every replica in the current live membership. The coordinator's own applied copy is implicit; operationally Figure 2 shows one remote `ACK` from each follower.

`VAL` is not part of the coordinator's client commit condition. It only releases matching follower copies for local reads.

An RMW commits if and only if it has the highest timestamp among concurrent updates to that key. A higher incoming `INV` aborts a pending RMW.

## Quorum system

Let `M_e` be the RM-supplied live membership for epoch `e`.

- total replicas: deployment-specific; the paper targets 3 to 7 replicas per shard;
- fast/data-path acknowledgment set: all members of `M_e` (the coordinator plus remote `ACK`s from `|M_e| - 1` followers);
- classic quorum: not applicable to the Hermes data path;
- recovery/replay acknowledgment set: all members of `M_e`;
- leader inclusion: the per-write coordinator is implicitly included through its local application;
- quorum flexibility: no flexible subset inside an epoch; the write-all set changes only through RM;
- intersection obligation: no data-path quorum-intersection proof substitutes for write-all. Cross-epoch safety comes from leased RM, epoch tags, and delaying reconfiguration until old leases expire.

This is a [[reliable-membership|membership-based]] read-one/write-all rule, not a majority commit rule.

## Conflict handling

The timestamp for a write is `[v, cid]`, where `v` is the incremented per-key version and `cid` is the coordinator ID. The paper defines the order exactly:

```text
[v_A, cid_A] > [v_B, cid_B]
iff
v_A > v_B
or
(v_A = v_B and cid_A > cid_B)
```

Every replica therefore chooses the same winner among concurrent same-version writes. A lower-timestamp write may complete after a higher-timestamp concurrent write yet be linearized before it, which is legal because the operations overlap.

For RMWs:

- increment timestamp version by 1 for an RMW;
- increment timestamp version by 2 for a write;
- acknowledge an RMW `INV` only if its timestamp is at least the local timestamp, otherwise return the local `INV`;
- abort a pending RMW upon a higher `INV`;
- after RM reconfiguration, discard gathered RMW acknowledgments and replay the RMW.

Thus writes win races with concurrent RMWs, and among concurrent RMWs only the highest `cid` can commit.

Source: `raw/hermes.pdf`, §§3.1, 3.5-3.6.

## Safety argument

The central invariant stated by the paper is:

```text
a read may complete if and only if the key is in a Valid state
```

The supporting reasoning is:

1. Before a coordinator replies, every operational replica has acknowledged the update and therefore stores either that timestamp/value or a higher one.
2. A higher timestamp places the key in a non-readable state until its own winning update is validated.
3. A stale `VAL` cannot make an older value readable because validation requires exact timestamp equality.
4. Timestamps totally order writes to one key at every replica.
5. A replay carries the original timestamp and value, so it completes an existing write rather than inventing or reordering one.
6. Membership leases and epoch tags stop removed/minority replicas from serving requests across an RM change.

The paper reports TLA+ model checking of reads, writes, RMWs, replays, message reordering/duplication, and membership reconfiguration under crash-stop failures. It checks safety and absence of deadlocks. The PDF does not enumerate named lemmas or include the full specification, so finer proof decomposition remains `Unclear`.

Source: `raw/hermes.pdf`, §§1, 3.1-3.4.

## Liveness argument

In a stable failure-free membership, writes finish after every live follower responds and invalidated reads resume after matching validation or all-ACK optimization. Message loss causes retransmission/replay.

A crashed or unreachable member can block the write-all path until RM detects the problem, waits for membership leases to expire, and installs a new epoch. During partitions only the primary partition continues. A newly listed live replica that has not yet received the `m-update` also temporarily blocks writes by dropping higher-epoch messages.

Timeouts affect when recovery starts, not which value is safe. The no-LSC variant preserves progress but may turn a local read into a wait for a later write or a majority membership check.

## Key proof ideas

- Make readability an explicit state predicate rather than a quorum read.
- Invalidate all current members before acknowledging a write to the client.
- Use one total per-key timestamp order to resolve concurrent writes independently at every replica.
- Include the value in the invalidation so recovery evidence is self-contained.
- Replay the exact timestamp/value pair; never allocate a new timestamp for recovery.
- Validate only an exact timestamp match so delayed `VAL`s are harmless.
- Fence configurations with leases and `epoch_id` rather than transferring data-path quorum claims across epochs.
- Treat RMW conflict semantics separately from ordinary non-aborting writes.

## Important formulas

Timestamp ordering:

```text
[v_A, cid_A] > [v_B, cid_B]
iff
v_A > v_B
or
(v_A = v_B and cid_A > cid_B)
```

Within epoch `e`:

```text
remote ACKs required = |M_e| - 1
effective write-all evidence = |M_e|
```

RMW/write version increments:

```text
RMW:   v := v + 1
write: v := v + 2
```

Failure-free write message sequence:

```text
INV -> ACK -> VAL
```

The paper states 1.5 replica round trips for the complete sequence and one exposed coordinator round trip. For an external client, the baseline is 2 RTTs; sending follower acknowledgments to both coordinator and client reduces this to 1.5 RTTs.

## Relationship to other protocols

Hermes resembles two-phase commit in message shape, but it is specialized to single-key replication, does not abort ordinary conflicting writes, and safely replays after coordinator failure. Unlike Paxos-family protocols, it does not choose a value from a majority data quorum. Unlike CRAQ, it avoids a write chain and central head/tail roles by broadcasting from any replica. Unlike [[CURP]], it obtains local reads and write completion through invalidating every live replica rather than storing speculative unordered durability at witnesses.

## Limitations

- Multi-key reliable transactions are outside the protocol; Hermes can only be an underlying replication layer.
- The design targets replication inside a datacenter, not a single Hermes group spanning datacenters.
- Common-case writes require every live member, so one failed/slow member blocks until RM reconfiguration.
- The main lease design assumes LSCs; the no-LSC variant adds read-validation delay.
- Reads and new writes stall while the local key is not `Valid`.
- RMWs can abort under concurrent updates even though ordinary writes do not.
- Byzantine faults are outside the model.
- The PDF reports model checking but does not present the complete TLA+ model or a named-lemma proof.

## Open questions

- What exact finite-state bounds were used for the reported Hermes TLA+ model checking? The PDF does not state them.
- Can the RM/data-path composition be proved with an explicit cross-epoch refinement invariant rather than relying on the external Vertical Paxos-style service?
- How should a production implementation calibrate `mlt` without causing replay storms under tail latency?
- Can the shadow-replica catch-up protocol expose a precise snapshot/cut invariant for keys copied concurrently with writes?

## Related pages

[[Hermes]], [[invalidation]], [[reliable-membership]], [[quorum]], [[fast-path]], [[recovery]], [[conflict]], [[failure-model]], [[linearizability]], [[liveness]], [[recoverability]], [[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[commit-rules]], [[recovery-rules]], [[reconfiguration]]
