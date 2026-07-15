---
type: protocol
name: Bodega
family: Paxos / lease-based read optimization
papers: [Bodega-2026]
tags: [paxos, roster-lease, local-read, linearizability, geo-replication]
---

# Bodega

## Short description

Bodega is a Multi-Paxos-style SMR extension that leases agreement on per-key read responders and strengthens write quorums to cover those responders, enabling local linearizable reads during interfering writes.

## Problem solved

Leader leases localize reads only at one leader, while earlier follower read leases are interrupted by writes. Bodega makes read authority stable metadata and keeps writes compatible with arbitrary responder placement.

## System model

Odd `n` asynchronous message-passing replicas with majority `m = ceil(n / 2)`, an ordered consensus log, one ballot leader, non-transactional key-value reads/writes, and ballot-tagged rosters containing per-key responder sets.

## Fault model

Fail-slow/stop nodes and asynchronous network faults; minority failures are tolerated for availability. Byzantine behavior is excluded. Leases require bounded clock-speed drift.

## Timing assumptions

No synchronized timestamps are required. Guard/renew/revoke leases require bounded drift and maintain grantor expiration no earlier than grantee expiration. Heartbeat timeouts trigger roster replacement; lease expiration is the safe fallback for unresponsive peers.

## Roles

Client; stable leader; per-key responder; ordinary follower; lease grantor/grantee; higher-ballot roster proposer.

## Message types

`Prepare`, `PrepareReply`, `Accept`, `AcceptReply`, `Commit`, `AcceptNote`, `Heartbeat<bal,ros>`, `Guard<thresh>`, `GuardReply`, `Renew`, `RenewReply`, `Revoke`, `RevokeReply`, client request/reply/redirect.

## Local state

Consensus log; current `<bal, ros>`; heartbeat timers; grantor/grantee guard and renewal timers/sets; per-grantor `thresh`; per-slot `accnote`; pending held reads; committed/executed prefix.

## Normal path

The leader commits a write after an ordinary majority plus every current responder for the key replies. A stable, caught-up responder serves a read from its latest local write slot; if that slot is not yet known committed, it holds the read pending commit evidence.

## Fast path

Local read from one stable responder when its newest key write is committed or has `m` early accept notifications. Writes have no Fast Paxos-style path.

## Slow path

Unsafe local reads are held, redirected, or proposed through classic consensus. Client unhold timeout sends the same read to another responder/leader and uses the first reply.

## Recovery

A higher-ballot roster revokes or expires old leases, starts new guarded leases, removes failed special-role nodes, and runs ordinary `Prepare` if the proposer becomes leader. A healthy-majority roster eventually excludes failed responders.

## Commit condition

Write commit requires at least `m` matching `AcceptReply`s including self and replies from all key responders in the current roster. Early read release uses `m` `AcceptNote`s but does not alter write commitment.

## Quorum requirement

- `m = ceil(n / 2)`, `f = floor(n / 2)`.
- Prepare/recovery: `m`.
- Write: flexible majority `m` plus all roster responders for the key; leader included implicitly.
- Stable roster: `m` active grants for one `<bal, ros>`.
- Catch-up: committed through `thresh_p` for every member of some size-`m` grantor subset.
- Early accept: `m` distinct notifications.

## Safety intuition

Majority-held leases make the stable roster unique. Same-ballot writes cover every responder. For older-ballot writes, the responder's majority grantor subset intersects the old write majority, and the intersecting grantor's threshold forces the responder to catch up through that write before serving locally.

## Liveness intuition

After stale leases expire, a new roster can choose leader/responders entirely within any healthy majority, reducing progress to ordinary consensus. Reads that remain uncertain escape local holding through client timeout and redirection.

## Strengths

- Local linearizable reads at arbitrary configured replicas.
- Read privilege survives interfering writes.
- No external roster oracle.
- Background renewal piggybacks on heartbeats.
- Write protocol remains classic consensus plus responder coverage.

## Weaknesses

- Every responder lies on the corresponding write commit condition.
- Lease safety depends on bounded clock drift.
- Partial partitions can destabilize roster activation without extra liveness mechanisms.
- Roster optimization policy is largely open.
- Non-transactional and non-Byzantine scope.

## Differences from related protocols

Unlike leader leases, Bodega supports many per-key read responders. Unlike Quorum Leases, it leases roster agreement rather than revoking read leases on each write. Unlike [[EPaxos]], writes remain globally leader-ordered; unlike [[Pando]], membership and read locality do not depend on an external planner.

## Open questions

- How should flexible quorums and responder coverage be co-designed?
- Can a transactional roster certify a consistent multi-key snapshot?
- What responder-selection policy minimizes total read/write cost under failures?

## Sources

- [[Bodega-2026]] - source paper, especially §§3-4 and Appendix A.
