---
type: protocol
name: Avicenna
family: Paxos / leader-based SMR
papers: [Avicenna-2026]
tags: [paxos, leader-based, fail-slow, counterfactual-evaluation, recovery, linearizability]
---

# Avicenna

## Short description

Avicenna is a single-leader SMR protocol that counterfactually evaluates a next-phase shadow leader and uses restricted normal-phase acceptors to rotate leadership after one nearby non-standby log exchange.

## Problem solved

Ordinary crash-tolerant leader protocols can remain safe and live while a responsive but slow leader causes unbounded client-latency degradation. Avicenna aims for the same normal-case latency as Multi-Paxos while masking any one fail-slow replica, including client-visible gray failures missed by internal heartbeats.

## System model

`n = 2f + 1` asynchronous, message-passing replicas. Messages may be delayed, reordered, or dropped, processes may run at arbitrary speeds, and clocks need not be synchronized. One real log determines the total execution order; an independent shadow log is never executed.

## Fault model

Non-Byzantine crash and fail-slow faults. Safety is asynchronous. Liveness tolerates up to `f` crashes under eventual communication assumptions. The performance goal is `ε-1` fail-slow tolerance relative to removing the one slow replica.

## Timing assumptions

Safety does not depend on timers. Liveness assumes at most `f` faulty replicas, eventual timely communication among a majority, and persistent client retransmission. Latency windows, progress timers, and early-report timers control detection and rotation timing.

## Roles

- **Client:** proposes to all non-standbys, measures/report latencies, retransmits, and completes on the first execution reply.
- **Real leader:** assigns real-log indices and commits commands.
- **Shadow leader:** is the deterministic next real leader, shadow-orders sampled commands, estimates counterfactual performance, and takes over after rotation.
- **Non-standby:** accepts real/shadow proposals and contributes rotation evidence.
- **Standby:** learns commits and executes but does not accept normal-phase proposals.

## Message types

`Propose`, `ProposeReply`, `Accept`, `AcceptOK`, `Commit`, `RealCommitted`, `ShadowAccept`, `ShadowAcceptOK`, `ShadowCommit`, `ShadowCommitted`, latency feedback, `Rotate`, `LongAccept`, `LongShadowAccept`.

## Local state

Phase and role; indexed real/shadow logs; command IDs; entry phase/status; acknowledgement counts; paired latency stores and buffers; progress timer; per-phase rotation flag and received logs; `OverCommitted` prefix metadata.

## Normal path

The real leader broadcasts an indexed command to `f + 2` non-standbys, counts itself, commits after `f + 1` `AcceptOK`s, broadcasts `Commit` to all replicas, and replies after ordered execution. The farthest `f - 1` replicas are standbys.

## Fast path

No Fast Paxos-style alternative quorum exists. The normal path has the standard two-message-delay Multi-Paxos commit shape. Independent sampled shadow processing adds work but no ordering coordination to this path.

## Slow path

There is no conflict-driven Accept fallback. Lack of progress or a real-over-shadow latency objective triggers phase rotation. A timed-out unsampled client command is resent with `IsShadow=True`.

## Recovery

Replicas broadcast phase-tagged logs in `Rotate`, stop accepting new proposals in that phase, and merge by preferring committed evidence and otherwise the highest-phase accepted command. Normal rotation merges two non-standby logs; Armageddon rotation merges `f + 1` logs. The previous shadow leader becomes real leader and reproposes the uncommitted suffix with `LongAccept` before new work.

## Commit condition

- Real commit: `f + 1` `AcceptOK`s from normal-phase non-standbys, including the real leader.
- Shadow commit: `f + 1` `ShadowAcceptOK`s, including the shadow leader; never executable.
- Execution: committed real entry plus committed/executed prefix.
- `OverCommitted`: `f + 2` accepts, used to truncate future rotation logs.

## Quorum requirement

- Total: `n = 2f + 1`.
- Normal acceptor set: `f + 2`; standbys: `f - 1`.
- Real/shadow commit: `f + 1`, leader included.
- Normal recovery: `2` non-standby logs.
- Armageddon acceptor set: `2f + 1`; recovery: `f + 1` logs including self.

For normal phases, `(f + 1) + 2 > f + 2`; for Armageddon phases, `(f + 1) + (f + 1) > 2f + 1`.

## Safety intuition

One real leader assigns one command per index in each phase. Every commit quorum intersects the logs merged by a later-phase replica, so committed entries survive. If only accepted values differ, phase numbers select the newest. The resulting committed-prefix invariant gives one total real-log order consistent with real time. Shadow state never affects execution.

## Liveness intuition

Missing commits expire progress timers and force deterministic phase rotation. Periodic Armageddon phases make all replicas eligible non-standbys, so repeated rotation eventually reaches a phase with a live majority and live real leader. Clients retransmit, and the real leader fills empty prefix entries with `no-op`.

## Strengths

- Same normal-case message-delay latency as Multi-Paxos.
- Masks one slow follower without action and one slow leader through client-informed rotation.
- Detects gray failures using end-to-end measurements.
- Fast normal-phase takeover from one additional nearby non-standby log.
- Separates performance detection from safety evidence.

## Weaknesses

- Additional shadow-processing throughput cost; lightweight multi-shard throughput can be substantially below Multi-Paxos.
- Only one fail-slow replica is covered.
- Objective tuning trades reaction time against false rotations under jitter.
- Full `f`-crash progress may require repeated rotations to an Armageddon phase.
- No Byzantine tolerance.

## Differences from related protocols

Unlike [[Copilot]], Avicenna has only one executable ordering path; the shadow path is independent and never executed. Unlike [[EPaxos]], it keeps a global leader and a total log rather than per-command leaders and dependency order. Unlike [[Mencius]], rotating leadership is per phase and counterfactual-performance-driven rather than a fixed owner per log slot.

## Open questions

- Can several shadow leaders cover multiple slow replicas without normal-path coordination?
- Can rotation evidence be generalized to flexible acceptor sets while retaining one-message takeover?
- Can shadow execution cost be estimated without fully executing the real-log command at the shadow leader?

## Sources

- [[Avicenna-2026]] - source paper, especially §§3-4 and Appendix A.
