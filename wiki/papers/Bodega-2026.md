---
type: paper
title: "Bodega: Localized Linearizable Reads at Anywhere Anytime via Roster Leases"
authors: Guanzhou Hu; Andrea C. Arpaci-Dusseau; Remzi H. Arpaci-Dusseau
year: 2026
venue: 20th USENIX Symposium on Operating Systems Design and Implementation (OSDI '26)
source: raw/bodega.pdf
protocols: [Bodega]
tags: [paxos, roster-lease, local-read, linearizability, geo-replication, read-optimization]
status: ingested
---

# Bodega: Localized Linearizable Reads at Anywhere Anytime via Roster Leases

## One-sentence summary

Bodega extends classic leader-based consensus with majority-backed per-key responder rosters, requiring writes to cover every active responder so those responders can serve linearizable reads locally even during interfering writes.

## Why this paper matters

Existing leader leases localize reads only at the leader, while prior quorum leases invalidate read privileges around interfering writes. Bodega instead leases agreement on long-lived metadata - the roster of leader and per-key responders - off the critical path. Writes retain ordinary consensus structure but add responder coverage; reads use local log state, brief optimistic holding, and optional early accept evidence.

The design exposes a reusable proof pattern: a read-serving replica needs both uniqueness of the current metadata configuration and a catch-up certificate showing that its local log covers any value that could have committed under an older configuration.

## System model

- A message-passing replicated state machine with an odd number `n` of servers and ordered log slots.
- Majority size `m = ceil(n / 2)` and minority failure budget `f = floor(n / 2)`.
- The network is asynchronous; nodes may be fail-slow or fail-stop.
- The paper uses non-transactional key-value commands: `Put` represents writes and `Get` represents read-only requests.
- Bodega is a non-intrusive extension to Multi-Paxos-style consensus; the paper states that the mechanism also applies to Raft-style protocols.

Source: `raw/bodega.pdf`, §§2.1, 2.4, 3.

## Fault model

- Minority node/network failures are tolerated for availability; safety must hold in all circumstances considered by the model.
- Fail-slow and fail-stop nodes are in scope.
- Byzantine behavior is not part of the described fault model.
- Failure detection uses per-peer heartbeat timers. A suspected failed special-role node is removed from responder sets and, if necessary, replaced as leader in a higher-ballot roster.
- Membership changes use ordinary consensus reconfiguration after first stabilizing an empty responder roster.

Source: `raw/bodega.pdf`, §§2.1, 3.3.2-3.3.3, 5.3.

## Timing assumptions

Consensus safety uses the usual asynchronous model, but roster leases additionally assume bounded clock-speed drift. They do not require synchronized clock timestamps or bounded clock skew.

The lease invariant is that the grantor-side expiration is never earlier than the grantee-side expiration. Guard phases establish the first safe overlap; renewals extend it; explicit revocation or expiration ends it. Heartbeat and lease timeouts affect liveness and roster-change speed.

The paper's timeout rule is preserved exactly:

```text
avg. RTT < t_hb_send ≪ t_hb_fail < t_guard = t_lease
```

Source: `raw/bodega.pdf`, §§2.2, 3.3.3.

## Main idea

A roster is ballot-tagged metadata containing one leader ID and, for each key or key range, a set of responder IDs allowed to serve local reads. The leader is implicitly a responder for every key. Nodes exchange leases all-to-all in the background; a node treats its roster as stable only after holding leases from a majority and catching its committed log up to thresholds reported by a majority of those grantors.

Writes still use the leader's normal `Accept` round, but the commit condition is strengthened to require both a majority and every active responder for the written key. Consequently, a stable responder has seen the latest same-ballot committed write. If its newest write is not yet known committed, it optimistically holds dependent reads until commit evidence arrives instead of immediately redirecting them.

## Protocol roles

- **Client:** sends writes to the leader, sends reads to a nearby responder, follows redirects, and duplicates a held read after an unhold timeout using the same request ID.
- **Leader:** owns normal log-slot assignment, runs `Prepare` after step-up, proposes writes, commits after majority plus responder coverage, and is an implicit responder for every key.
- **Responder:** is authorized by the stable roster for selected keys and may answer those reads from local state.
- **Follower:** accepts leader proposals, learns commits, optionally sends early accept notifications, and redirects writes.
- **Lease grantor/grantee:** every node acts in both directions for every peer, maintaining guard, renew, revoke, and expiration state.
- **Roster proposer:** announces a unique higher ballot and candidate roster after a user request, tuning decision, or suspected failure.

## Message types

- Consensus: `Prepare`, `PrepareReply`, `Accept`, `AcceptReply`, `Commit`/commit notification.
- Read optimization: `AcceptNote` to active responders.
- Roster dissemination and failure detection: `Heartbeat<bal, ros>`; lightweight heartbeats may carry only the ballot when the roster is unchanged.
- Lease activation: `Guard<bal, thresh>`, `GuardReply`, `Renew<bal>`, `RenewReply`.
- Lease termination: `Revoke<bal>`, `RevokeReply`.
- Client read/write requests, replies, and redirects.

Source: `raw/bodega.pdf`, §§3.2-3.3 and Figure 7.

## Local state

- Normal SMR log of slots and committed/executed prefix.
- Current ballot/roster pair `<bal, ros>`; ballots combine a monotonically increasing integer with proposer ID for uniqueness.
- Per-peer heartbeat timers `T_heartbeat,p`.
- Grantor-side `T_guardTo,p`, `T_renewTo,p`, `guardTo`, and `renewTo`.
- Grantee-side `T_guardBy,p`, `T_renewBy,p`, `guardBy`, and `renewBy`.
- Per-grantor catch-up threshold `thresh_p`, recording the highest slot that grantor had ever accepted when sending `Guard`.
- Per-slot early-accept marker set `accnote,slot`.
- Pending held reads attached to the latest write slot for their key.

## Normal path

For a write, the stable leader assigns the next log slot, broadcasts `Accept`, and receives `AcceptReply`s. It commits only after receiving at least `m` replies including itself and replies from all responders for the key. It then broadcasts a commit notification and executes the committed prefix.

For a read at server `S`:

1. If `S` is the stable leader, it returns the latest committed value for the key.
2. If `S` is not a roster responder for the key, it redirects to a nearby responder or the leader.
3. If `S` is a stable non-leader responder and its highest write slot for the key is committed, it returns that value locally.
4. If that slot is in flight, `S` holds the read until commitment or sufficient early accept evidence.

## Fast path

The read fast path is a single-server local response. It requires:

- `S` is the leader or a responder for the key in a stable roster;
- `S` passes the majority-lease and threshold catch-up check; and
- for a non-leader responder, the newest write slot for that key is committed, or is accepted in the current ballot with at least `m` distinct `AcceptNote`s counting self.

The write path is ordinary leader-based consensus with a strengthened responder-covering quorum; Bodega does not introduce a Fast Paxos-style write fast quorum.

## Slow path

- A non-responder redirects the read.
- A node without a stable/caught-up roster falls back to classic consensus as though the read were a write; an unstable non-leader redirects to the leader.
- A responder with an uncertain newest write optimistically holds the read. On client `t_unhold` timeout, the client sends the same idempotent read to another responder or the leader and uses the first reply.
- If early accept notifications are disabled or insufficient, the held read waits for the normal commit notification.

## Recovery path

On suspected failure or an explicit/tuning change, a node announces a unique higher-ballot roster. Every node that sees the higher ballot first revokes or waits out its old outgoing leases, moves to the new roster, and initiates guarded leases to all peers. If it is the new leader, it restarts in-progress slots from the ordinary `Prepare` phase.

Fast roster replacement uses one message round for revocation and one for new guards. If peers are unresponsive, safety waits for lease expiration. A liveness-preserving roster can eventually restrict both leader and responders to a healthy majority. Membership changes first stabilize an empty responder roster, then use ordinary reconfiguration.

## Commit rule

For a write to key `k`, the leader commits only after:

```text
at least m matching AcceptReply messages, including self
AND
AcceptReply from every responder for k in the current roster
```

The effective write evidence is therefore the union of a flexible majority and the fixed current responder set. A responder's local-read reply is not itself a consensus commit; it is justified by stable-roster leases plus local committed/inevitably-committing evidence.

## Quorum system

- Total servers: odd `n`.
- Fault budget: `f = floor(n / 2)`.
- Majority: `m = ceil(n / 2)`.
- Prepare/recovery quorum: ordinary majority `m`.
- Write commit: any majority `m` plus all current responders for the written key; the leader is implicitly a responder.
- Stable-roster certificate at a grantee: at least `m` active leases for the same ballot/roster.
- Local-read catch-up evidence: some size-`m` subset of active grantors whose `thresh_p` values are all covered by the grantee's committed prefix.
- Early accepted-write evidence: `m` distinct `AcceptNote`s counting self.

Majority intersection supplies both roster uniqueness and old-write visibility. Quorums are flexible in which majority replies, but responder coverage is fixed by the current per-key roster.

## Conflict handling

One leader serializes writes into log slots; Bodega does not use application-level commutativity or dependency conflict resolution. A read conflicts with an in-flight write to the same key. The responder holds the read until the write is committed or has majority acceptance evidence, avoiding a stale reply without revoking its read privilege.

## Safety argument

Ordinary reads/writes that fall back to classic consensus inherit its safety. For a locally served read `R` at stable responder `S`, consider a write `W` acknowledged before `R` begins and let `<bal, ros>` be `S`'s stable roster:

1. `W` cannot have committed at a ballot greater than `bal`, because a majority currently grants leases for `bal`.
2. If `W` committed at `bal`, the injective ballot-to-roster mapping and responder-covering commit rule imply `S` was in `W`'s write quorum and has `W` in its log.
3. If `W` committed below `bal`, its majority accept quorum intersects any size-`m` subset `E` of `S`'s lease grantors. Some `P in E` accepted `W` at slot `x` before granting, so `thresh_P >= x`; the catch-up check requires `S` to have committed through that threshold.

Thus `R` observes every earlier acknowledged interfering write and returns the latest value. The TLA+ appendix additionally checks lease expiration safety, one stable roster, and linearizability.

Source: `raw/bodega.pdf`, §4.3 and Appendix A.

## Liveness argument

The formal write-liveness argument assumes a majority `G` of healthy servers. After old leases are explicitly revoked or expire, a new roster can restrict both the leader and all responders to `G`; ordinary consensus then makes progress.

Local reads that cannot safely complete are held only optimistically: the client unhold timeout sends the request elsewhere. Partial network partitions can still cause competing roster activations and threaten liveness when driven by heartbeat timeouts; the paper recommends standard pre-vote or transparent rerouting techniques.

Source: `raw/bodega.pdf`, §§3.2.2, 4.3, 7.

## Key proof ideas

- Use majority lease intersection to prove at most one stable roster.
- Couple each unique ballot injectively to one roster.
- Strengthen writes with responder coverage so same-ballot responders observe every committed write.
- Carry highest-ever-accepted slot thresholds in `Guard` messages to make new responders prove catch-up across older ballots.
- Separate lease expiration safety from consensus safety.
- Preserve availability by allowing failed responders to disappear after old leases expire and a healthy-majority roster stabilizes.

## Important formulas

Majority and failures:

```text
m = ⌈n/2⌉
f = ⌊n/2⌋
```

Stable roster and catch-up condition, preserving the paper's structure:

```text
|{}renewBy| ⩾ m
∧ ∃ size-m subset E ⊆ {}renewBy:
    committed all slots up to thresh_p, ∀p ∈ E
```

Early local-read evidence:

```text
|{}accnote,slot| ⩾ m
```

Lease timing rule:

```text
avg. RTT < t_hb_send ≪ t_hb_fail < t_guard = t_lease
```

Default WAN values:

```text
t_hb_send = 120 ms
t_hb_fail ≈ 1200 ms
t_guard = t_lease = 2500 ms
```

The formal lease property is that while the grantee's renewed lease is active, the grantor is renewing/revoking and:

```text
grantor leaseExpire >= grantee leaseExpire
```

## Relationship to other protocols

Bodega retains classic Paxos-style leader ordering rather than using [[EPaxos]] dependencies. Unlike ordinary leader leases, it assigns read responders per key. Unlike Quorum Leases, the lease protects the roster rather than each write, so interfering writes do not revoke local-read authority. Unlike [[Pando]], it is a self-contained replicated-state-machine extension rather than an erasure-coded storage protocol with external membership planning.

## Limitations

- Requires bounded clock drift for leases.
- Targets non-transactional commands; multi-key transactions are outside scope.
- Write commit must wait for every active responder for the key, so distant or slow responders increase write latency.
- The paper focuses on roster mechanisms, not an optimal responder-selection policy.
- Benefits diminish for intra-datacenter, write-heavy, or weak-consistency workloads.
- Heartbeat-driven roster activation can risk liveness during partial partitions without pre-votes or rerouting.
- Byzantine faults are not addressed.
- The TLA+ model-checking scope is finite: three nodes, one failure, three ballots, two writes, two reads, three lease-time ticks, and all responder choices.

## Open questions

- How should responder rosters be optimized jointly for read locality, write coverage latency, failures, and skew?
- Can responder-covering write quorums be combined safely with flexible or asymmetric Paxos quorums?
- How should transactional reads obtain equivalent cross-key roster and catch-up guarantees?
- Can partial-partition liveness be proved with a specific pre-vote or rerouting extension?
- Can roster leases safely carry broader metadata such as membership, reliability hints, or quorum shapes?

## Related pages

[[Bodega]], [[roster-lease]], [[quorum]], [[fast-path]], [[slow-path]], [[leader]], [[recovery]], [[linearizability]], [[liveness]], [[quorum-intersection]], [[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[commit-rules]], [[recovery-rules]], [[latency]], [[proof-techniques]]
