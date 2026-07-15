---
type: protocol
name: Copilot
family: Paxos / dual-leader dependency SMR
papers: [Copilot-2020]
tags: [paxos, dual-leader, dependency, fast-path, recovery, slowdown-tolerance, linearizability]
---

# Copilot

## Short description
Copilot is a dual-leader SMR protocol that sends every command through a pilot and copilot, orders duplicate copies in two logs, merges them with dependencies, and uses fast per-entry takeover to tolerate any one slow replica.

## Problem solved
Crash-tolerant RSMs can still suffer high latency when a leader or collaborating replica is slow but responsive. Copilot seeks normal latency throughout receive, order, execute, and reply despite one slow replica.

## System model
`n = 2f + 1` asynchronous replicas, including distinguished pilot `P` and copilot `P'`. Each owns a sequential log. Cross-log prefix dependencies plus a fixed cycle-priority rule produce one combined total order.

## Fault model
Non-Byzantine crash failures. Safety holds despite failures; progress tolerates up to `f` failures with a communicating majority. The performance target is [[slowdown-tolerance]] for any one slow or failed replica.

## Timing assumptions
Safety is asynchronous. Liveness assumes eventual partial synchrony and eventual message delivery. Ping-pong-wait and takeover timers affect when fallback begins, not which values are safe.

## Roles
- **Client:** sends each uniquely identified command to both pilots and accepts the first reply.
- **Pilot/copilot:** independently propose into separate logs, collect quorum evidence, execute, reply, and take over blocking entries from the other log.
- **Replica:** checks compatibility, stores entry/ballot state, acknowledges proposals, learns commits, and executes the combined deduplicated log.
- **Replacement pilot:** is elected by view change for one log and recovers unresolved entries.

## Message types
Client command/reply; `FastAccept`; `FastAcceptOk`; `FastAcceptReply`; `Accept`; `AcceptOk`; `Commit`; `Prepare`; `PrepareOk`.

## Local state
Two logs; command and cross-log dependency per entry; ballot and progress (`not-accepted`, `fast-accepted`, `accepted`, `committed`); proposer identity; executed prefixes; `(cliid, cid)` deduplication/result cache; cycle priority; batch/timer state.

## Normal path
Both pilots append the same client command to their next log entries. Each initially depends on the latest entry seen in the other log. Replicas accept compatible dependencies or suggest a later cross-log prefix. Both pilots eventually execute the same merged order, with the duplicate command skipped at its second position.

## Fast path
A pilot commits its initial dependency after `f + floor((f + 1) / 2)` `FastAcceptOk` replies including itself, then broadcasts `Commit`. Ping-pong batching alternates pilot batches so their proposed dependencies remain compatible.

## Slow path
After at least `f + 1` fast-phase replies, the pilot sorts all suggested dependencies and chooses the `(f + 1)`-th. It persists that final dependency with `Accept` and commits after `f + 1` `AcceptOk` replies.

## Recovery
When a committed command is blocked by unresolved predecessors in the other log, the fast pilot prepares those entries at a higher ballot. From `f + 1` replies it preserves committed/accepted evidence, chooses `no-op` when too little fast evidence exists, preserves a candidate with at least `f` fast accepts, or recursively checks the first possibly incompatible cross-log entry for the ambiguous interval `[floor((f + 1) / 2), f)`.

View change replaces one pilot independently and uses the same recovery value-selection rule.

## Commit condition
- Fast: `f + floor((f + 1) / 2)` matching `FastAcceptOk` replies including the proposing pilot.
- Regular/takeover: `f + 1` `AcceptOk` replies including the proposer.

Commit does not imply execution readiness: local predecessors and cross-log dependencies must be executed, unless a cycle is broken in favor of the higher-priority pilot log.

## Quorum requirement
- `n = 2f + 1`.
- Fast quorum = `f + floor((f + 1) / 2)`.
- Regular Accept quorum = `f + 1`.
- Takeover Prepare/Accept quorum = `f + 1`.
- Minimum fast/recovery intersection = `floor((f + 1) / 2)`.
- Neither original pilot is required to answer recovery.

## Safety intuition
Every committed cross-log pair is ordered in at least one dependency direction. Each log has one recovered value per entry, deterministic priority resolves cycles, and deduplication collapses the two copies of a command. Compatibility also preserves real-time order across logs.

## Liveness intuition
Available dependencies let execution advance in one log or the other. A fast pilot takes over unresolved entries of a slow pilot; view change replaces failed pilots. Eventual partial synchrony prevents endless competing takeovers.

## Strengths
- Covers receive, ordering, execution, and reply with proactive redundancy.
- Tolerates a slow-but-responsive replica that ordinary failure detection may not replace.
- Uses per-entry takeover to unblock concrete work instead of always changing leaders.
- Keeps no-slowdown performance competitive with Multi-Paxos and EPaxos in the paper's datacenter experiments.

## Weaknesses
- Handles only one slowdown.
- Duplicates ordering, execution-path, and reply work.
- Uses a subtle cross-log recovery rule for ambiguous fast evidence.
- Does not tolerate Byzantine behavior.
- Full recovery proof is delegated to a separate technical report.

## Differences from related protocols
Unlike [[EPaxos]], Copilot has exactly two distinguished ordering replicas, duplicates every command, uses one cross-log prefix dependency per entry, totally orders all commands, and can take over individual entries. Unlike [[Mencius]], it intentionally duplicates work rather than partitioning it. Unlike [[CURP]], its redundant path participates in consensus ordering and execution rather than only temporary durability.

## Open questions
- TODO: Add the technical report's complete recovery pseudocode and proof.
- Can the dual path be generalized to tolerate multiple simultaneous slowdowns efficiently?
- Can cycle priority and null-dependency elimination be expressed as a small executable invariant set?

## Sources
[[Copilot-2020]]
