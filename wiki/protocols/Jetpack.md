---
type: protocol
name: Jetpack
family: Fast consensus / plugin framework
papers: [Jetpack-2026]
tags: [fast-path, paxos, plugin, view-change, recovery, commutativity, linearizability]
---

# Jetpack

## Short description

Jetpack is a shim-layer framework that runs a 1-RTT Fast-Paxos-style path beside an existing consensus protocol and uses proposer promises plus prioritized recovery sets to make both paths converge safely.

## Problem solved

Existing mature systems cannot easily replace their consensus protocol just to obtain a fast path. Jetpack adds one while preserving the host's original path, features, and no-fast-traffic performance.

## System model

`n = 2f + 1` asynchronous replicas, each with a Jetpack shim and original-protocol replica. The host exposes monotonic views, membership, proposer set, proposal entry point, log ordering, and view changes. Applications provide a command-conflict predicate.

## Fault model

Up to `f` crash failures; slow/partitioned replicas are handled by host view change. Byzantine faults and interactive transactions are excluded.

## Timing assumptions

Asynchronous safety. Liveness inherits the host's eventual view/majority progress. Fast success is 1 client RTT; failed fast attempts complete through the original path.

## Roles

Client library; Jetpack shim replica; original-path proposer/replica; view-change recovery coordinator.

## Message types

View-tagged fast command/ack/reject; host proposal/commit/reply; GC notification; `BeginRecovery`, `Prepare`, `PrepareOK`, `Accept`, `AcceptOK`, `FinishRecovery`.

## Local state

Independent fast view; normal/recovery mode; per-view fast log; accepted recovery set; all-in-flight command pool; original view/membership/proposer set; client attempt-rate metrics.

## Normal path

Fast commands are broadcast to Jetpack replicas and forwarded to all original proposers. Shims acknowledge only matching-view, conflict-free commands. Original proposals proceed concurrently; the client accepts the first successful path.

## Fast path

Fast commit requires at least `f + ⌈f/2⌉ + 1` same-view acknowledgements including all original proposers. Each proposer promises no concurrent conflicting command will be proposed ahead of the fast command.

## Slow path

Conflict, view mismatch, or insufficient acknowledgements rejects fast commitment; the already-running host protocol commits the command normally.

## Recovery

Pause a majority of the last normal view, agree by majority on a recovery set containing candidates seen in at least `⌈f/2⌉ + 1` replies, resubmit the set/no-op through the new original proposer, and resume only after that stability marker commits.

## Commit condition

- Fast: same-view superquorum `f + ⌈f/2⌉ + 1`, all proposers included.
- Original: unchanged host rule.
- Recovery set: `f + 1` accepts.
- New-view activation: original commit of recovery set or no-op.

## Quorum requirement

- `n = 2f + 1`.
- Fast superquorum `f + ⌈f/2⌉ + 1` plus all proposers.
- Original/recovery majority `f + 1`.
- Minimum fast/recovery intersection and candidate threshold `⌈f/2⌉ + 1`.

## Safety intuition

Same-view superquorum evidence makes every fast commit recoverable. Host prerequisites preserve proposer receipt/proposal/log order. The stability marker forces recovered fast commands before stale or new conflicting work, preserving speculative results and real-time order.

## Liveness intuition

The host path remains independently live. Recovery retries accepted sets and resumes after the host commits one marker in the new view. Fast failure never prevents original-path completion.

## Strengths

- Adds 1-RTT fast commitment without replacing the host protocol.
- Preserves the unmodified original path and its features.
- Makes view-change hazards and compatibility prerequisites explicit.
- Applies to single- and multi-leader hosts satisfying `PR 1`/`PR 2`.
- Adaptive path selection protects saturated throughput.

## Weaknesses

- Application must provide sound conflict detection.
- Extra fast traffic, CPU, memory, and recovery downtime.
- Contention and proposer count reduce fast-path benefit.
- Shared-log multi-leader cost is `Ω(M)` per fast command.
- Not compatible with [[EPaxos]] as presented; `PR 2` violations require waiting for host proposal; no BFT/transaction support.

## Differences from related protocols

Unlike [[FastPaxos]], Jetpack is a reusable layer paired with an independent host path. Unlike [[CURP]], its main abstraction is a view-safe proposer promise rather than unordered witness durability. Unlike [[SwiftPaxos]], it does not redesign dependency consensus; it leaves slow ordering to the host.

## Open questions

- Can durable promises reduce recovery phases?
- Can dependency-ordered hosts be made compatible?
- Which goal must relax to beat the multi-leader `Ω(M)` bound?

## Sources

- [[Jetpack-2026]] - source paper, especially §§3-5 and Appendix B/G.
