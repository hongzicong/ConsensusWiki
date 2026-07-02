---
type: protocol
name: CURP
family: primary-backup fast replication
papers: [CURP-2019]
tags: [primary-backup, fast-path, commutativity, witness, recovery, linearizability]
---

# CURP

## Short description
CURP, the Consistent Unordered Replication Protocol, adds temporary unordered witnesses to primary-backup replication so commutative updates can complete in 1 RTT.

## Problem solved
Traditional primary-backup and strong-leader consensus systems usually wait for ordered durable replication before replying to an update, costing 2 RTTs. CURP reduces common-case update latency by making the request durable at witnesses in parallel with speculative master execution.

## System model
CURP is presented primarily for primary-backup storage systems with one master, `f` backups, and `f` witnesses. Witnesses are temporary non-volatile stores for recent client requests and can be separate from or co-hosted with backups.

## Fault model
Fail-stop, non-Byzantine failures. Primary-backup CURP is immediately recoverable despite up to `f` failures. It remains strongly consistent even if more components fail, but availability can block until recovery evidence is reachable.

## Timing assumptions
Safety is asynchronous. Messages may be delayed or dropped. The 1 RTT path is a performance path when the master and all witnesses respond successfully.

## Roles
- **Client:** sends update RPCs to the master and `record` RPCs to witnesses.
- **Master:** serializes, executes, checks unsynced commutativity, replies speculatively, syncs to backups, and garbage-collects witness records.
- **Backup:** stores ordered durable state or logs.
- **Witness:** durably stores unordered request records, rejects non-commuting records, and serves recovery data.
- **Coordinator/configuration manager:** manages master/witness assignment and `WitnessListVersion`.

## Message types
- `record(request, WitnessListVersion)` from client to witness.
- `sync(request/id)` from client to master.
- `gc(keyHashes, rpcIds)` from master to witness.
- `getRecoveryData()` from new master/coordinator to witness.
- `start(masterId)` from coordinator to witness.

## Local state
- `unsynced` operations at the master.
- witness record set per master.
- durable backup state/log.
- unique RPC IDs and stored results for exactly-once semantics.
- `WitnessListVersion` to prevent stale witness lists from completing updates.

## Normal path
Clients send an update to the master and concurrently record it at witnesses. The master executes the update and may return before backup sync if the update is commutative with all unsynced operations. The master later syncs ordered data to backups and sends batched garbage collection to witnesses.

## Fast path
The client completes in 1 RTT after receiving the master result and acceptances from all `f` witnesses. Both witnesses and the master enforce commutativity over records/operations that have not yet been synced to backups.

## Slow path
The slow path is backup sync. It is triggered by witness rejection/failure, master-detected non-commutativity, explicit client `sync`, or inability to use the current witness set. The master then waits until the operation is durable in backups before the client completes.

## Recovery
The new master restores state from one backup, chooses one available witness, freezes that witness, retrieves its saved requests, and replays them in any order. It then syncs to backups and resets/reassigns witnesses before accepting new requests.

Recovery must not merge records from multiple witnesses in the primary-backup protocol because different witnesses may have accepted different commuting subsets in different orders. Selecting one witness preserves the invariant that all replayed requests mutually commute.

## Commit condition
An operation can be considered client-completed if either:
- it is recorded in all `f` witnesses and the master returned the result, or
- it is replicated/synced to the `f` backups.

## Quorum requirement
Primary-backup CURP does not use a majority quorum for its main fast path:
- `n = f + 1` primary-backup replicas,
- `f` backups,
- `f` witnesses,
- fast witness set = all `f` witnesses,
- recovery = one backup plus one selected witness.

The appendix consensus extension uses `2f + 1` replicas/witnesses and requires a fast witness superquorum of `f + ceil(f/2) + 1`.

## Safety intuition
Every fast-completed unsynced operation is stored at every witness, so it is present in whichever witness recovery selects. Since a witness only stores mutually commutative requests, replay order cannot alter visible results. If an operation creates a non-commuting dependency observed by another operation, the master must sync before replying, so recovery from backup preserves the observation.

## Liveness intuition
The fast path depends on all witnesses accepting. Conflicts, failures, or witness capacity pressure reduce performance but not safety because clients fall back to the backup sync path.

## Strengths
- 1 RTT common-case update completion without special network hardware.
- Applies to existing primary-backup systems with limited intrusion.
- Keeps ordered backup mechanism intact.
- Can improve write latency and throughput when most operations commute.

## Weaknesses
- Fast path requires all `f` witnesses in the primary-backup design.
- Needs request-parameter commutativity checks.
- Requires exactly-once/RIFL-style duplicate suppression.
- Hot keys and non-commutative operations trigger fallback.
- Reconfiguration and stale witness lists add protocol surface.

## Differences from related protocols
Unlike [[FastPaxos]] and Generalized Paxos, CURP does not use fast acceptor quorums to choose an ordered consensus value. Unlike [[EPaxos]], it keeps a strong master and uses witnesses only for unordered durability of unsynced operations.

## Open questions
- TODO: Formalize the reconfiguration proof around `WitnessListVersion` and zombie clients.
- TODO: Decide whether to model witnesses as separate crash domains or co-hosted components for new protocol sketches.

## Sources
[[CURP-2019]]
