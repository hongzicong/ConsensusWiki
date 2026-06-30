---
type: protocol
name: PigPaxos
family: paxos / multipaxos communication overlay
papers: [PigPaxos-2021]
tags: [paxos, multipaxos, smr, relay, aggregation, leader]
---

# PigPaxos

## Short description
PigPaxos is Multi-Paxos with randomized relay groups that disseminate leader messages and aggregate follower acknowledgements, reducing leader communication load while preserving Paxos quorum logic.

## Problem solved
It targets the throughput bottleneck caused by a stable Paxos leader sending to and receiving from every follower for each replicated operation.

## System model
`N` replicas run Multi-Paxos. One stable leader orders operations; the `N - 1` followers are partitioned into `R` relay groups. Each Paxos message exchange uses randomly selected relays from those groups.

## Fault model
Crash failures only. In the base Paxos configuration, `N = 2f + 1` tolerates up to `f` crashed nodes.

## Timing assumptions
Safety is asynchronous/Paxos-style. Liveness needs eventual bounded communication for timeout/retry behavior. Relay timeout `T_r` must be lower than the leader timeout.

## Roles
- Client.
- Multi-Paxos leader.
- Relay follower.
- Ordinary follower.

## Message types
Paxos Phase 1a/1b, Phase 2a/2b, and commit messages, carried through Pig broadcast/reply messages keyed by `PigMsgID`; relays may also return higher-ballot rejects.

## Local state
Standard Multi-Paxos state plus relay-group membership, per-round relay selection, relay aggregation buffers keyed by `PigMsgID`, leader-side unique-vote tracking, and optional gray lists for problematic relay candidates.

## Normal path
The leader sends each Paxos message to one random relay in each group. Relays process the message, forward it within the group, aggregate replies, and send one response back. The leader counts unique replica votes and commits when a majority has accepted.

## Fast path
No Fast Paxos-style fast path. PigPaxos optimizes the normal Multi-Paxos path by reducing leader message handling.

## Slow path
If followers are slow, a relay can time out and return a partial aggregate. If relays fail or too few votes arrive, the leader retries with fresh random relays.

## Recovery
Ordinary Paxos leader change and recovery with higher ballots. Relay retries are message-delivery retries, not value-selection recovery.

## Commit condition
Same as Multi-Paxos: a command is chosen when a majority of unique replicas accepts it for a slot and ballot. Relay aggregates are not themselves quorum votes.

## Quorum requirement
Majority quorum `floor(N/2) + 1`; in the common fault-tolerant configuration, `N = 2f + 1`. Partial relay thresholds must still allow the leader to collect at least a global majority, e.g. `sum_{i=1}^R g_i >= floor(N/2) + 1`.

## Safety intuition
PigPaxos preserves Paxos safety because it does not change ballots, accepted values, quorum size, or safe leader takeover rules. Relays only transport and aggregate messages.

## Liveness intuition
Random relay rotation and retries prevent a fixed small set of failed relays from permanently blocking progress, assuming enough nodes are live and message delays eventually satisfy the timeout assumptions.

## Strengths
- Reduces leader CPU/network pressure.
- Keeps the Paxos proof structure.
- Can scale throughput in larger clusters without leaderless conflict handling.
- Region-based relay groups can reduce WAN traffic.

## Weaknesses
- Adds a relay hop, which can increase low-load latency.
- Relay failures create tail-latency and retry cases.
- Still relies on a single leader for ordering and decisions.
- Does not provide a new fast consensus path.

## Differences from related protocols
Unlike [[Mencius]], PigPaxos rotates relays rather than leaders. Unlike [[EPaxos]], [[Atlas]], and [[SwiftPaxos]], it keeps a single leader and does not order conflicts with dependencies. Unlike [[FastPaxos]], it does not use fast quorums or proposer-to-acceptor direct voting.

## Open questions
- TODO: Model dynamic and overlapping relay groups if they are used beyond the paper's base protocol.
- TODO: Check whether combining PigPaxos with Flexible Paxos changes any comparison-page quorum statements after an FPaxos paper is ingested.

## Sources
- [[PigPaxos-2021]]
