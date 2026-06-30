---
type: paper
title: PigPaxos: Devouring the Communication Bottlenecks in Distributed Consensus
authors: [Aleksey Charapko, Ailidani Ailijiang, Murat Demirbas]
year: 2021
venue: SIGMOD 2021
source: raw/pigpaxos.pdf
protocols: [PigPaxos]
tags: [paxos, multipaxos, smr, leader, relay, aggregation, throughput]
status: ingested
---

# PigPaxos: Devouring the Communication Bottlenecks in Distributed Consensus

## One-sentence summary
PigPaxos keeps the Multi-Paxos protocol and quorum logic unchanged, but replaces direct leader-to-follower fan-out/fan-in with randomly rotated relay groups that aggregate follower acknowledgements.

## Why this paper matters
PigPaxos is a useful design point because it attacks the single-leader communication bottleneck without introducing a leaderless fast path, larger fast quorums, dependency metadata, or a new safety proof. Its main contribution is to separate decision-making at the Paxos leader from much of the message dissemination and acknowledgement aggregation work.

## System model
- A strongly consistent replicated state machine using Multi-Paxos-style consensus instances.
- There are `N` nodes: one stable leader and `N - 1` followers.
- Followers are divided into `R` relay groups, where `R in {1..N - 1}`.
- In the main protocol description, relay groups are static and non-overlapping. The paper notes that overlapping groups are possible future/extension configurations.
- The leader starts one Pig communication pattern per relay group by choosing a random relay from that group.
- The paper evaluates clusters from 5 to 25 nodes and discusses larger geo-replicated deployments.

## Fault model
- Crash failure model: nodes may silently crash.
- Byzantine faults are not considered.
- Because the underlying Paxos protocol is unchanged, PigPaxos tolerates up to `f` node failures in a cluster of size `2f + 1`.
- Relay failures are a liveness/performance concern because a failed relay can hide replies from its whole relay group until the leader retries with new random relays.

## Timing assumptions
- Safety is inherited from Paxos and does not depend on timeouts.
- Liveness needs some synchrony: the paper assumes an upper bound on message propagation and retries when communication takes longer.
- Relay timeout `T_r` must be smaller than the leader timeout so relays can forward partial replies before the leader retries.
- Random relay selection is used to avoid a fixed minority of failed relays denying progress forever.

## Main idea
Use the Pig communication primitive for Paxos fan-out/fan-in. A relay processes the leader's Paxos message as a normal follower, retransmits it to the other followers in its relay group, collects their replies, and returns an aggregated response to the leader. The leader still decides whether a Paxos quorum has accepted.

## Protocol roles
- Client: submits operations to the Paxos leader.
- Leader: stable Multi-Paxos leader; orders operations, chooses ballots, sends Phase 1/Phase 2/commit messages, counts unique replica votes, and decides when a majority quorum has replied.
- Relay: a follower randomly selected within a relay group for a Pig round; acts as a normal follower and as an aggregator for that group.
- Follower: ordinary Paxos acceptor/replica that replies to the selected relay instead of directly to the leader.

## Message types
- Paxos Phase 1a / Phase 1b: leadership proposal and promise.
- Paxos Phase 2a / Phase 2b: accept request and accepted acknowledgement.
- Paxos Phase 3 / commit: commit notification, usually piggybacked on later Phase 2 messages in Multi-Paxos.
- Pig broadcast message: includes a `PigMsgID` that identifies the Pig round.
- Pig send/reply message: includes the same `PigMsgID` so the relay can aggregate replies for the correct round.
- Reject message carrying a higher ballot, sent when a relay learns that the old leader is obsolete.

## Local state
- Standard Paxos leader/follower state: ballot numbers, accepted commands, log slots, and commit status.
- Leader-side set of nodes already counted for a slot in a ballot, to avoid duplicate votes after retries or overlapping relay groups.
- Relay-group configuration and random relay choice for each round.
- Relay aggregation state keyed by `PigMsgID`.
- Optional gray list of slow, failed, or suspicious nodes to avoid selecting them as relays for some time.

## Normal path
For each Paxos communication step:
1. The leader chooses a random relay from each relay group.
2. Each selected relay processes the message as a follower and forwards it to the other followers in its group.
3. Followers reply to the relay.
4. The relay aggregates replies and sends one response to the leader, either after all group replies arrive or after its relay timeout.
5. The leader aggregates relay responses and checks whether a majority quorum has accepted/promised.

With a stable Multi-Paxos leader, Phase 1 is not repeated for every instance and Phase 3 is piggybacked onto later Phase 2 traffic.

## Fast path
PigPaxos has no Fast Paxos-style fast path. Its common path is ordinary Multi-Paxos Phase 2 through relays. The latency optimization target is not fewer consensus phases; it is lower leader-side communication and CPU load.

## Slow path
Slow behavior is retry-based. If followers are slow, a relay returns a partial aggregate after timeout. If a relay fails and the leader cannot collect a majority from other relay groups, the leader times out and retries with newly selected random relays.

## Recovery path
PigPaxos uses ordinary Paxos leader change and ballot recovery. A new leader can take over with a higher ballot. Old-leader retries fail when followers or relays report the newer ballot. Relay retry behavior does not choose values; it only changes how messages are delivered.

## Commit rule
A command is anchored/chosen when the leader receives a majority of Phase 2b accepted acknowledgements for that slot and ballot, just as in Multi-Paxos. The leader must count unique follower votes, not relay messages, and must not count duplicate votes twice.

## Quorum system
- Total nodes: `N = 2f + 1` in the standard crash-tolerant configuration.
- Fault tolerance: up to `f` node crashes.
- Classic quorum size: majority, `floor(N/2) + 1`.
- Fast quorum size: not applicable; PigPaxos does not add fast quorums.
- Recovery quorum size: ordinary Paxos Phase 1 majority.
- Relay groups do not replace Paxos quorums. They are only communication/aggregation overlays.
- Partial response collection uses group thresholds `g_i = n_i - PRC`, where `n_i` is group `i`'s size. The paper states the thresholds must satisfy `sum_{i=1}^R g_i >= floor(N/2) + 1`, where `R` is the number of relay groups.

## Conflict handling
PigPaxos is single-leader Multi-Paxos. It does not use dependency metadata or command conflict detection. Concurrent client operations are ordered by the leader into log slots.

## Safety argument
The paper's safety argument is that Paxos safety proofs are oblivious to the communication implementation between leader and followers. Correctness depends on quorum size and the information exchanged in quorums, not on whether votes were sent directly or through relay aggregation. PigPaxos keeps the Paxos decision logic unchanged.

Additional safety notes:
- Retries do not affect safety because they only resend Paxos messages and do not change leader decision-making.
- Duplicate votes are safe because the leader tracks which nodes have voted for the slot/ballot and does not count a node twice.
- Overlapping relay groups remain safe if the leader catches duplicate votes; the paper discusses this as an extension.
- Ballot numbers preserve ordinary Paxos leadership safety during leader changes.

## Liveness argument
Progress requires enough live nodes for a majority and communication that eventually satisfies the assumed timeout bounds. Failed followers can be masked by relay partial responses if the leader still obtains a majority. Failed relays can delay an operation, but random relay retries eventually choose healthy relays when enough nodes are live.

## Key proof ideas
- Treat Pig as a communication refinement of Multi-Paxos, not as a new consensus protocol.
- Preserve the same quorum evidence and ballot rules as Paxos.
- Keep relay aggregation outside the value-selection proof.
- Model retries as message-delivery attempts, not as new decisions.
- Model duplicate vote suppression explicitly at the leader.

## Important formulas
- Fault-tolerant cluster size: `N = 2f + 1`.
- Majority quorum: `floor(N/2) + 1`.
- Relay group count: `R in {1..N - 1}`.
- Partial response threshold per group: `g_i = n_i - PRC`.
- Partial response requirement: `sum_{i=1}^R g_i >= floor(N/2) + 1`.
- Leader message load: `M_l = 2R + 2`.
- Average follower message load:
  `M_f = 2 * (R / (N - 1)) * ((N - R - 1) / R) + 2`
  `= 2 * (N - R - 1) / (N - 1) + 2`.
- For `R = 1` and `N -> infinity`, average follower load approaches `4`, matching the minimum leader load `M_l = 4`.

## Relationship to other protocols
- Unlike [[FastPaxos]], PigPaxos does not let clients/proposers bypass the leader and does not require fast-quorum recovery.
- Unlike [[EPaxos]], [[Atlas]], and [[SwiftPaxos]], PigPaxos does not use dependency metadata or leaderless command coordination.
- Unlike [[Mencius]], PigPaxos does not rotate the command leader; it rotates relay nodes underneath a stable leader.
- The paper notes Pig can be combined with Flexible Paxos-style quorum ideas, but that is not PigPaxos's base protocol.

## Limitations
- PigPaxos still has a single decision-making leader.
- Relay hops can increase low-load latency compared with ordinary Multi-Paxos.
- Relay failures introduce new timeout cases and can hurt tail latency, especially in a single-relay-group configuration.
- The protocol does not increase the total number of messages, but aggregated replies may be larger than individual replies.
- Scaling to hundreds of nodes may require new failure-detector and relay-tree mechanisms.

## Open questions
- TODO: Add a separate page for Paxos Quorum Reads if `raw/` includes the referenced PQR paper.
- TODO: Clarify how to model dynamic relay-group reconfiguration if used in a real implementation.
- TODO: Compare Pig-style aggregation with Compartmentalized Paxos if that paper is ingested later.

## Related pages
- [[PigPaxos]]
- [[paxos-family]]
- [[leader-roles]]
- [[quorum-systems]]
- [[commit-rules]]
- [[recovery-rules]]
- [[timing-assumptions]]
- [[proof-techniques]]
