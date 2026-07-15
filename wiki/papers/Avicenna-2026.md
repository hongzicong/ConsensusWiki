---
type: paper
title: "Avicenna: Masking Slowdowns in Replicated State Machines with Counterfactual Evaluation"
authors: Christopher Hodsdon; Zijian Qin; Khiem Ngo; Siddhartha Sen; Ethan Katz-Bassett; Wyatt Lloyd
year: 2026
venue: 21st European Conference on Computer Systems (EuroSys '26)
source: raw/avicenna.pdf
protocols: [Avicenna]
tags: [paxos, leader-based, fail-slow, counterfactual-evaluation, recovery, linearizability]
status: ingested
---

# Avicenna: Masking Slowdowns in Replicated State Machines with Counterfactual Evaluation

## One-sentence summary

Avicenna retains a single Multi-Paxos-style ordering leader, continuously estimates how a designated shadow leader would perform, and safely rotates to that shadow leader using restricted commit quorums that make one nearby non-standby log sufficient takeover evidence.

## Why this paper matters

Avicenna targets a performance failure that crash-tolerant consensus does not mask: a replica can remain correct and responsive while making the system persistently slow. It introduces an achievable geo-distributed definition of fail-slow tolerance, reconstructs a client-visible counterfactual rather than relying on a static timeout, and trades throughput for the same normal-case latency as Multi-Paxos plus tolerance of one fail-slow replica.

Its main proof-design lesson is that restricting which replicas may accept normal-phase commands can make leader replacement faster: a normal-phase commit quorum of `f + 1` among only `f + 2` non-standbys intersects any two-log rotation evidence set.

## System model

- Replicated state machine with `n = 2f + 1` replicas.
- Asynchronous network: messages may be arbitrarily delayed, reordered, or dropped.
- Processes may run at arbitrary speeds; replicas communicate only by messages.
- No clock synchronization is assumed.
- Commands are placed in one real log and executed in log-index order after commitment.

Source: `raw/avicenna.pdf`, §§2, 4.1.

## Fault model

- Non-Byzantine faults only.
- Up to `f` crash failures are tolerated for liveness with `2f + 1` replicas.
- A fail-slow replica remains functionally correct but has degraded latency.
- Avicenna targets one fail-slow replica: `s = 1`, with `s <= f`.
- Gray fail-slow faults are in scope: client-visible performance may degrade even when internal heartbeat monitoring appears healthy.

The paper defines:

`ε-s-Fail-Slow Fault Tolerance`: latency with `s` fail-slow replicas exceeds the latency of the physically realizable counterfactual system with those replicas removed by at most `ε`.

Source: `raw/avicenna.pdf`, §§2-3.

## Timing assumptions

Safety is asynchronous and does not depend on timeout accuracy or synchronized clocks. The liveness proof assumes:

1. No more than `f` replicas are faulty.
2. A majority can communicate within a timeout, and messages eventually arrive before receiver timeouts.
3. Clients keep retransmitting timed-out commands until completion.

Latency and progress timers trigger rotation, but log evidence and phase rules determine safety. Objective-function windows and early-report timeouts affect detection quality and speed rather than agreement.

Source: `raw/avicenna.pdf`, §2 and Appendix A.3.

## Main idea

Each phase has one real leader that determines the executable order and one shadow leader, chosen as the next phase's real leader, that independently shadow-orders sampled commands. Shadow-log commands are never executed. Instead, Avicenna combines client-measured shadow commit latency with the shadow leader's execution of the same command in the real log to estimate the end-to-end latency clients would see under the shadow leader.

Replicas rotate when the real latency is sufficiently worse than the reconstructed shadow latency. The shadow leader already holds its own real log; because normal-phase acceptance is restricted to `f + 2` non-standbys, one additional non-standby `Rotate` log is enough to intersect every `f + 1` commit quorum.

## Protocol roles

- **Client:** sends commands to all non-standbys, assigns `ClientID` and `CommandID`, selects `IsShadow`, measures real/shadow latency, reports paired samples, retransmits after timeout, and accepts execution replies.
- **Real leader:** appends commands to the real log, broadcasts `Accept`, commits after `f + 1` acknowledgements including itself, and orders execution.
- **Shadow leader:** is the next deterministic real leader, coordinates an independent shadow log, measures shadow execution latency in its real log, and takes over after rotation.
- **Non-standby replica:** accepts real and shadow proposals and participates in commit and rotation evidence.
- **Standby replica:** learns commits and executes real-log commands but does not participate in normal-phase commit quorums.

## Message types

- Client `Propose` and replica `ProposeReply`.
- Real path: `Accept`, `AcceptOK`, `Commit`, and optional `RealCommitted`.
- Shadow path: `ShadowAccept`, `ShadowAcceptOK`, `ShadowCommit`, and client-directed `ShadowCommitted`.
- Rotation: `Rotate`, `LongAccept`, and `LongShadowAccept`.
- Client latency feedback: paired real/shadow end-to-end samples, paired commit-latency samples, and optional at-least latency early reports.

Source: `raw/avicenna.pdf`, §§4.1-4.4 and Appendix A.1.

## Local state

- Monotonically increasing `Phase`, starting at `0`.
- Real log indexed by monotonically increasing `Inst`; each entry stores command, phase, and status such as `Received`, `Accepted`, or `Committed`.
- Shadow log indexed by `ShadowInst`; entries progress through shadow acceptance and shadow commitment but are never executed.
- Per-entry acknowledgement counts and command identifiers.
- `sentRotate[phase]`, received rotation logs, role/configuration for each phase, and a progress timer.
- Paired real/shadow latency stores plus buffers for unpaired samples.
- `mininst`/`OverCommitted` truncation metadata for omitting universally accepted prefixes from rotation messages.

## Normal path

The client sends a command to all non-standbys. The real leader assigns the next real-log `Inst`, broadcasts `Accept` to all non-standbys, and counts its own acceptance. A recipient stores the command at that index and returns `AcceptOK`. At `f + 1` acknowledgements including itself, the leader marks the entry committed and broadcasts `Commit` to all replicas. Replicas execute committed entries only after all preceding entries have executed, then send `ProposeReply`; the client completes on the first reply.

Standbys receive commits and execute but are excluded from the acceptance quorum. The `f - 1` farthest replicas are selected as standbys, leaving `f + 2` non-standbys.

Source: `raw/avicenna.pdf`, §4.1.

## Fast path

Avicenna does not have a Fast Paxos-style collision-dependent fast path. Its normal path is the ordinary two-network-message-delay Multi-Paxos path: leader `Accept`, follower `AcceptOK`, then asynchronous `Commit` dissemination and execution replies.

Shadow processing runs independently and does not coordinate proposal order with the real leader, so it does not add a message delay to the real path. A sampled command is shadow-committed after `f + 1` `ShadowAcceptOK`s including the shadow leader.

## Slow path

There is no within-phase consensus slow path for conflicts because one real leader assigns every real-log index. Fallback consists of:

- retransmitting a timed-out command with `IsShadow=True` if it was initially unsampled;
- latency-triggered rotation when the real leader underperforms the shadow counterfactual; or
- progress-triggered rotation when commits stop and a replica's progress timer expires.

## Recovery path

Any replica may initiate rotation by broadcasting `Rotate(Phase, realLog, statuses)`. A matching receiver rebroadcasts its own `Rotate` once, then stops accepting new `Accept`/`ShadowAccept` messages in that phase while still learning, committing, and executing existing work.

For each log index, merge rules are:

1. Preserve a locally committed entry.
2. If any received entry is `Committed`, learn that command as committed and broadcast `Commit`.
3. If only one collected entry is `Accepted`, copy it.
4. If different accepted commands appear, select the one with the highest phase.

In a normal phase, a non-standby merges two non-standby logs, normally its own plus one received `Rotate`. A standby waits for two non-standby `Rotate` messages. In an Armageddon phase, a replica collects `f + 1` logs including itself.

After merging, replicas increment the phase. The previous shadow leader becomes real leader, first broadcasts `LongAccept` covering all uncommitted real-log entries (and may batch newly received commands), commits them with normal `AcceptOK` evidence, and only then proposes fresh commands. The new shadow leader similarly sends `LongShadowAccept`.

Source: `raw/avicenna.pdf`, §§4.3-4.4 and Algorithms 3, 6, 8, 9.

## Commit rule

- Real entry: the real leader commits after `f + 1` `AcceptOK`s from non-standbys, including itself.
- Shadow entry: the shadow leader shadow-commits after `f + 1` `ShadowAcceptOK`s from non-standbys, including itself.
- A real commit is disseminated to all replicas with `Commit`.
- Commit is not execution completion: an entry executes only after it and every preceding real-log entry are committed and earlier entries have executed.
- `OverCommitted` is stronger metadata, recorded after `f + 2` real acknowledgements, meaning every normal-phase non-standby accepted the entry; it is used only to truncate rotation logs.

## Quorum system

- Total replicas: `n = 2f + 1`.
- Normal-phase non-standbys: `f + 2`.
- Normal-phase standbys: `f - 1`.
- Real commit quorum: `f + 1` non-standbys, including the real leader.
- Shadow commit quorum: `f + 1` non-standbys, including the shadow leader.
- Normal rotation evidence: `2` non-standby logs; for the shadow leader this is its own log plus one `Rotate` from a non-standby.
- Armageddon non-standbys: all `2f + 1` replicas.
- Armageddon rotation evidence: `f + 1` logs, including the recovering replica's own log.
- `OverCommitted` threshold: `f + 2` accepts.

Normal-phase intersection is explicit:

```text
|A| = f + 1
|B| = 2
|N_p| = f + 2
|A| + |B| = f + 3 > f + 2 = |N_p|
```

Armageddon intersection is:

```text
|A| = f + 1
|B| = f + 1
|N_p| = 2f + 1
|A| + |B| = 2f + 2 > 2f + 1 = |N_p|
```

These inequalities preserve at least one acceptor of every committed command across rotation.

## Conflict handling

A single real leader assigns one command per real-log index in each phase, so same-phase proposal collisions do not occur. If different commands appear at the same index, Lemma 1 shows their stored phases differ. Rotation therefore preserves a committed command if visible; otherwise it retains the highest-phase accepted command. Shadow-log order may differ from real-log order, but shadow entries are never executed and cannot affect safety.

## Safety argument

Only real processing affects correctness. The proof establishes linearizability through:

- **Lemma 1:** different accepted/committed commands at the same index must have different phases.
- **Lemma 2:** once a command is committed at an index, every replica in every later phase has that command at that index; quorum/merge intersection is the key step.
- **Lemma 3:** a committed prefix remains the same prefix of every later real leader's log.
- **Theorem 4:** in-order execution of the preserved real log gives one total order.
- **Theorem 5:** a command completed before another begins occupies an earlier real-log index, so execution respects real-time order.

The proof deliberately excludes the shadow log: real and shadow processes are independent, and shadow entries are never executed.

Source: `raw/avicenna.pdf`, Appendix A.2.

## Liveness argument

If the real leader is faulty or fewer than `f + 1` non-faulty non-standbys remain, no new `Commit` resets progress timers, so non-faulty replicas eventually rotate. At most `f - 1` standbys imply at least two non-faulty non-standbys, allowing `f + 1` non-faulty replicas to collect the two required normal-phase logs.

Armageddon phases ensure rotation eventually reaches a configuration with a live majority and a live real leader. The paper defines phase `p` as Armageddon when `(p + 1) mod (N + 1) = 0`; all replicas are then non-standbys. Client retransmission and filling empty log gaps with `no-op` complete the argument that every submitted command eventually commits, executes, and receives a reply.

Source: `raw/avicenna.pdf`, §§4.1, 4.4 and Appendix A.3.

## Key proof ideas

- Restrict normal-phase acceptors so two non-standby logs intersect every commit quorum.
- Carry phase numbers with accepted values and select the highest-phase uncommitted value during merging.
- Preserve any explicitly committed value before considering accepted evidence.
- Prove a committed-prefix invariant across phases, then derive total and real-time order.
- Separate detection quality from safety: counterfactual latency only decides when to rotate, not what value survives rotation.
- Insert periodic all-replica Armageddon configurations to recover full `f`-crash liveness lost by the optimized normal configuration.

## Important formulas

Fail-slow tolerance definition:

`ε-s-Fail-Slow Fault Tolerance`: an RSM is `ε-s`-fail-slow fault-tolerant if its latency in the presence of `s` fail-slow replicas increases over that of the corresponding hypothetical system by at most `ε`.

Reconstructed shadow end-to-end latency:

```text
ℓ̂shadow_e2e(c) = ℓshadow_commit(c) + ℓshadow_exec(c)
```

where:

```text
ℓshadow_commit(c) = send(c) -> receive ShadowCommitted(c)
ℓshadow_exec(c) = receive Committed(c) -> finish executing c
```

Objective function over samples inserted within the last `T` seconds:

```text
F_T = A(R_T) - (1 + τ)A(G_T) - β
```

with `τ ∈ (0, 1)` and `β ≥ 0`; rotate when `F_T > 0`.

Short-path commit objective:

```text
F_T_commit = A(R_T_commit) - (1 + τ)A(G_T_commit) - β
```

Rotate when:

```text
max(F_T, F_T_commit) > 0
```

Armageddon schedule, exactly as stated:

```text
(p + 1) mod (N + 1) = 0
```

Source: `raw/avicenna.pdf`, §§3, 4.2, 4.4.

## Relationship to other protocols

Avicenna's real path follows Multi-Paxos-style single-leader ordering. Unlike [[Copilot]], its shadow leader does not coordinate the executable order or execute the shadow log, avoiding Copilot's dual-ordering slow path in exchange for reactive rotation and lower throughput. The rotation pattern is inspired by [[Mencius]] and MR99, but its two-log recovery threshold comes from restricting normal acceptors rather than from owner-authored skips or an ordinary majority view change.

Unlike [[EPaxos]], Avicenna does not distribute command ownership across replicas or use dependencies. It therefore retains Multi-Paxos latency characteristics while aiming to prevent any one slow replica from controlling client latency.

## Limitations

- The protocol tolerates one fail-slow replica; multiple simultaneous slow replicas remain future work.
- Byzantine behavior is excluded.
- Shadow processing consumes messages and CPU. In the evaluated lightweight, seven-shard single-datacenter case, Avicenna reached about one third of Multi-Paxos throughput; sampling reduces overhead.
- Detection is only as robust as the chosen objective function and sample window: max is reactive but jitter-sensitive, while tail means are steadier but slower.
- Normal phases may stall after two non-standby crashes; progress may require repeated rotations until an Armageddon phase.
- The low takeover latency argument depends partly on choosing standbys geographically far from the current leader; this is a performance heuristic, not a safety premise.

## Open questions

- Can multiple shadow leaders tolerate `s > 1` fail-slow replicas without adding normal-path latency?
- Can counterfactual evaluation safely drive non-standby reconfiguration, quorum leases, or coordinator placement?
- How should a formal performance model choose `epsilon`, `T`, `tau`, `beta`, and aggregator `A` under adversarial jitter?
- Can the shadow workload be reduced below sampling without losing timely detection of execution-specific gray failures?
- Can Armageddon frequency be adapted without weakening the liveness argument?

## Related pages

[[Avicenna]], [[slowdown-tolerance]], [[counterfactual-evaluation]], [[quorum]], [[leader]], [[recovery]], [[linearizability]], [[liveness]], [[quorum-intersection]], [[protocol-catalog]], [[quorum-systems]], [[commit-rules]], [[recovery-rules]], [[latency]], [[proof-techniques]]
