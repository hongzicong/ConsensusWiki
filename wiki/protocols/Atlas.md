---
type: protocol
name: Atlas
family: paxos / leaderless SMR
papers: [Atlas-2020]
tags: [consensus, smr, leaderless, fast-path, quorum, recovery]
---

# Atlas

## Short description
Atlas is a leaderless, dependency-based SMR protocol for planet-scale systems that optimizes for a small number `f` of concurrent site failures.

## Problem solved
It targets low client-perceived latency in geo-replicated SMR without a distinguished leader, while allowing the deployment to choose `f` independently of the total number of sites `n`.

## System model
Asynchronous distributed system with `n` static processes, each representing a data center in the planet-scale deployment.

## Fault model
Non-Byzantine crash failures. At most `f` processes may fail, with `1 <= f <= floor((n - 1)/2)`. Exceeding `f` may block liveness but should not violate safety.

## Timing assumptions
Safety is asynchronous. Progress requires enough reachable processes to form fast, slow, or recovery quorums.

## Roles
- Client.
- Replica/process.
- Initial command coordinator.
- Recovery coordinator.
- Per-command consensus acceptor.

## Message types
`MCollect`, `MCollectAck`, `MConsensus`, `MConsensusAck`, `MCommit`, `MRec`, and `MRecAck`.

## Local state
Per command identifier: command payload `cmd`, phase, dependency set `dep`, remembered fast quorum `quorum`, current ballot `bal`, and last accepted ballot `abal`.

## Normal path
The initial coordinator picks a fast quorum of size `floor(n/2) + f` including itself, sends the command and known conflicting past commands, and unions all dependency replies.

## Fast path
The coordinator broadcasts `MCommit` immediately if:

```text
union_Q dep = union^f_Q dep
```

where `union^f_Q dep` includes only dependencies reported by at least `f` members of the fast quorum. For `f = 1`, this condition always holds and the fast quorum is a majority.

## Slow path
If the fast-path condition fails, the coordinator runs a Flexible Paxos-style Phase 2 over a slow quorum of size `f + 1`. The initial coordinator can skip Phase 1 because it owns the initial ballot.

## Recovery
A recovery coordinator collects `n - f` `MRecAck` replies. It preserves the highest accepted slow-path proposal if one exists; otherwise it reconstructs possible fast-path dependencies from a known initial fast quorum; otherwise it proposes `noOp`.

## Commit condition
Fast commit is all fast-quorum replies plus the `union_Q dep = union^f_Q dep` predicate. Slow commit is `f + 1` `MConsensusAck` replies in a ballot followed by `MCommit`.

Committed commands execute only when dependencies are committed/executed or in the same minimal batch.

## Quorum requirement
- Fast quorum: `floor(n/2) + f`, includes the initial coordinator.
- Slow quorum: `f + 1`.
- Recovery quorum: `n - f`.
- NFR read quorum: plain majority under the transitive-conflict read optimization.

## Safety intuition
Atlas preserves one `(cmd, dep)` decision per identifier and ensures that for any two conflicting committed non-`noOp` commands, at least one identifier appears in the other's dependency set. Batch execution then gives a consistent order for non-commuting commands.

## Liveness intuition
With at most `f` failed/unreachable processes and eventual message delivery, a coordinator can collect the quorums needed for fast path, slow path, or recovery. More than `f` unavailable sites may block the protocol.

## Strengths
- Leaderless client proximity.
- Small fast quorums when `f` is small relative to `n`.
- Fast path can tolerate non-matching dependency replies.
- Recovery rule is comparatively direct: accepted slow proposal, recoverable fast proposal, or `noOp`.

## Weaknesses
- Relies on choosing a small concurrent site-failure budget for good latency.
- Dependency/batch execution can delay command execution after commit.
- Non-fault-tolerant reads require transitive conflicts.

## Differences from related protocols
Unlike [[EPaxos]], Atlas does not require fast-quorum replies to match exactly; it requires included dependencies to be reported by at least `f` fast-quorum processes. Unlike [[FastPaxos]], Atlas commits dependency metadata for SMR commands rather than a single value in a standalone consensus instance.

## Open questions
- TODO: Fill in exact theorem references if the extended technical report is added.

## Sources
- [[Atlas-2020]]
