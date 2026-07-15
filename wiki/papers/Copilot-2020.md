---
type: paper
title: Tolerating Slowdowns in Replicated State Machines using Copilots
authors: Khiem Ngo; Siddhartha Sen; Wyatt Lloyd
year: 2020
venue: 14th USENIX Symposium on Operating Systems Design and Implementation (OSDI '20)
source: raw/copilot.pdf
protocols: [Copilot]
tags: [paxos, dependency, fast-path, recovery, slowdown-tolerance, linearizability]
status: ingested
---

# Tolerating Slowdowns in Replicated State Machines using Copilots

## One-sentence summary
Copilot redundantly processes every command through a pilot and copilot, merges their separate logs with cross-log dependencies, and uses per-entry Paxos takeovers so any one slow replica does not appreciably raise client latency.

## Why this paper matters
The paper defines [[slowdown-tolerance]] separately from crash tolerance and argues that a protocol must tolerate slowness in all four RSM stages: receive, order, execute, and reply. Copilot is presented as the first `1`-slowdown-tolerant consensus protocol.

The design is also a useful recovery case: its fast quorum is smaller than a Fast Paxos quorum under the same `n = 2f + 1` configuration, so some recovery quorums contain an ambiguous amount of fast evidence. Copilot resolves that ambiguity by examining a potentially incompatible entry in the other pilot's log, not by counting votes alone.

## System model
Copilot is an RSM over `n = 2f + 1` replicas. Two distinguished replicas are the pilot `P` and copilot `P'`. Clients send every command to both; each pilot orders the command independently in its own log, executes the combined log, and replies.

Processes communicate in an asynchronous system: instruction speeds and message delays are unbounded for safety. Commands are deterministic, or pilots convert nondeterministic choices into command inputs before ordering.

## Fault model
The paper assumes crash failures: a failed process stops executing and responding. Copilot tolerates at most `f` failures for progress with `2f + 1` replicas; linearizability is a safety property despite any number of failures. Byzantine behavior is outside the model.

A failed replica is also slow under the paper's definition, but a slow replica need not have failed. Copilot provides `1`-slowdown-tolerance for any one slow or failed replica.

## Timing assumptions
Safety is asynchronous and does not depend on timeouts. Liveness assumes eventual partial synchrony and eventual message delivery, with a timely communicating majority.

Two timers affect performance and progress behavior:
- a ping-pong-wait timeout lets a fast pilot close a batch without waiting indefinitely for the other pilot;
- a takeover timeout starts per-entry recovery when a committed command is blocked by an unresolved predecessor.

The implementation uses randomized exponential backoff to avoid dueling takeover proposers and reports a `10 ms` takeover timeout; these are implementation choices, not safety premises.

## Main idea
Copilot creates two proactive command-processing paths. Each pilot appends the same client command to its own log. An entry stores a dependency on a prefix endpoint in the other pilot's log. Final dependencies, both per-log orders, and a deterministic cycle-priority rule define one combined total log. Duplicate command IDs ensure each command executes only at its first combined-log position.

The core ordering protocol has a `FastAccept` phase and, when necessary, an `Accept` phase. A fast pilot can also take over specific unresolved entries in the slow pilot's log using Paxos `Prepare`/`Accept` ballots.

## Protocol roles
- **Client:** assigns increasing command ID `cid` under unique client ID `cliid`, sends the command to both pilots, accepts the first reply, and ignores the duplicate reply.
- **Pilot `P` and copilot `P'`:** each owns one log, proposes commands and dependencies, collects quorum evidence, executes the combined log, replies to clients, and can take over blocking entries from the other log.
- **Replica:** checks dependency compatibility, stores per-entry command/dependency/ballot progress, replies to ordering and recovery messages, learns commits, and executes the same deduplicated combined log.
- **View-change initiator/new pilot:** replaces a failed pilot for one log and resolves that log's unresolved entries using the same recovery value-picking rule.

## Message types
- Client command carrying `(command, cliid, cid)` to both pilots.
- `FastAccept` carrying the proposed command, initial dependency, log entry, and ballot.
- `FastAcceptOk` when the initial dependency passes the compatibility check.
- `FastAcceptReply` carrying a later suggested dependency when the initial dependency is incompatible.
- `Accept` carrying the selected final dependency; `AcceptOk` acknowledges it.
- `Commit` announces the final command/dependency without an acknowledgement requirement.
- `Prepare` with higher ballot `b'`; `PrepareOk` returns progress (`not-accepted`, `fast-accepted`, `accepted`, or `committed`), the highest relevant ballot, value, and dependency-proposing pilot.
- Client reply from each pilot after first execution of the command.

## Local state
- Two sequential logs, one for `P` and one for `P'`.
- Per entry: command, initial/final cross-log dependency, ballot/prepared ballot, proposer identity, and progress state.
- At replicas: accepted dependencies needed for compatibility checks and commit/execution state.
- Deduplication state and cached output keyed by `(cliid, cid)`.
- Fixed pilot-over-copilot priority for breaking dependency cycles.
- Ping-pong batch state plus ping-pong-wait and takeover timers.

## Normal path
Each client sends its command to both pilots. A pilot assigns the command to its next local log entry and proposes as the initial dependency the most recent entry it has seen in the other log. Replicas accept that dependency if it is compatible with all dependencies they previously accepted; otherwise they return the latest entry from the other log as a suggested dependency.

Both pilots eventually execute the deduplicated combined order and reply. The client uses the first response.

## Fast path
For entry `P.i` with initial dependency `P'.j`, a replica rejects the proposal only if it has already accepted a later `P'.k`, `k > j`, whose dependency is earlier than `P.i`. Otherwise it sends `FastAcceptOk`.

The pilot fast-commits after receiving a fast quorum of

`f + floor((f + 1) / 2)`

`FastAcceptOk` replies, including itself. The initial dependency becomes final, the pilot sends `Commit`, and it proceeds toward execution. For `n = 3, 5, 7, 9`, the paper lists fast quorums `2, 3, 5, 6` respectively.

Ping-pong batching keeps both pilots' initial dependencies compatible when both are fast: reception of the other pilot's `FastAccept` closes the next batch. A timeout closes the batch if the other pilot is slow.

## Slow path
If the pilot receives incompatible suggestions or cannot collect the fast quorum, it waits for at least `f + 1` total `FastAcceptOk`/`FastAcceptReply` responses. It sorts their suggested dependencies in ascending order and selects the `(f + 1)`-th suggestion as the final dependency.

The pilot sends that dependency in `Accept`, commits after `f + 1` `AcceptOk` replies including itself, and broadcasts `Commit`. The selected dependency is high enough to intersect an already committed cross-log command, but avoids unnecessarily later dependencies and cycles.

## Recovery path
Fast takeover is per blocking log entry rather than a whole-log leader replacement. A fast pilot sends a higher-ballot `Prepare` for entries in the slow pilot's log that may precede its blocked command and waits for `f + 1` `PrepareOk` replies. If any reply reports `committed`, it preserves that command/dependency. Otherwise it considers replies at the highest seen ballot:

1. If any reply is `accepted`, preserve its command/dependency.
2. If fewer than `floor((f + 1) / 2)` replies are `fast-accepted`, choose `no-op` with an empty dependency.
3. If at least `f` replies are `fast-accepted`, preserve that command/dependency.
4. If the fast-accepted count is in `[floor((f + 1) / 2), f)`, inspect the first possibly incompatible entry in the other log and recover it if necessary; its outcome distinguishes whether the candidate may have fast-committed.

The recovering pilot then sends `Accept` and commits after `f + 1` `AcceptOk` replies. The paper states that the complete value-picking procedure is in its accompanying technical report; the OSDI paper gives only the above summary.

View change is separate and elects a replacement owner for one pilot log. The new pilot uses the same value-picking procedure for unresolved entries while the other log can continue making progress.

## Commit rule
An entry commits either:
- on the fast path after `f + floor((f + 1) / 2)` matching `FastAcceptOk` replies including the proposing pilot, or
- on the regular/recovery path after `f + 1` `AcceptOk` replies including the proposer/recoverer.

Commit is not execution readiness. For entry `P.i` depending on `P'.j`, execution requires: `P.i` is committed; `P.(i - 1)` has executed; and either `P'.j` has executed, or `P.i` and `P'.j` are in a dependency cycle and `P` has the fixed higher priority.

## Quorum system
- Total replicas: `n = 2f + 1`.
- Fast quorum: `f + floor((f + 1) / 2)`, including the proposing pilot.
- Regular Accept quorum: `f + 1`, including the proposer.
- Takeover Prepare quorum: `f + 1`, including the recoverer.
- Takeover Accept quorum: `f + 1`, including the recoverer.
- Commit dissemination: broadcast, with no reply required.
- Quorums are not required to include both pilots; recovery explicitly does not depend on a reply from either original pilot.

A fast quorum and recovery majority intersect in at least `floor((f + 1) / 2)` replicas. That lower bound explains the ambiguous recovery interval: minimum intersection alone does not reveal whether the entry actually fast-committed.

## Conflict handling
Copilot totally orders all commands; it does not use application-level commutativity. A conflict is an incompatible pair of cross-log dependencies that would permit different replicas to order `P.i` and `P'.j` differently.

Replicas reject incompatible initial dependencies. The regular path chooses a later dependency that captures the required order. Final dependencies may form cycles; all replicas break cycles using the same fixed rule that orders pilot entries before copilot entries. Commands duplicated across the two logs are suppressed by `(cliid, cid)`.

## Safety argument
Linearizability requires one total order consistent with real time. For real-time order, if command `a` completes before `b` begins and `a` committed at `P.i`, then `b`'s entry in the other log cannot have a dependency before `P.i`; the compatibility check would reject it. Thus `a` precedes `b` without a cycle between those entries.

For total order, the key cross-log invariant is: for committed `P.i` and `P'.j`, either `P.i` depends on at least `P'.j`, or `P'.j` depends on at least `P.i`. Therefore one entry reaches the other through the dependency order. Each log entry also commits with the same command/dependency at all replicas. Majority intersection handles ordinary accepted/committed evidence; the special recovery rule handles ambiguous partial fast evidence.

The per-log order, cross-log invariant, fixed cycle priority, and safe recovery value selection give every replica the same combined order.

## Liveness argument
The paper uses double induction over executed prefixes of the two logs. From prefixes through `P.i` and `P'.k`, either `P.(i + 1)` or `P'.(k + 1)` becomes executable, a fast takeover occurs, or a view change occurs. A dependency already in the executed prefix allows immediate progress; otherwise execution follows the other log or deterministically breaks a cycle.

If one pilot is slow, the fast pilot takes over unresolved predecessors. If both pilots fail, view change elects replacements. Eventual partial synchrony supplies the usual Paxos argument that takeover and view-change contention does not continue forever.

## Key proof ideas
- Separate agreement for each log entry from deterministic merging of the two logs.
- Use the compatibility invariant to orient every committed cross-log pair in at least one direction.
- Preserve ordinary accepted evidence by majority intersection.
- Treat the minimum fast/recovery intersection as ambiguous and inspect the other log when counts alone are insufficient.
- Distinguish committed from executable; dependencies and cycle priority determine readiness.
- Use deduplication to make two ordered copies of each command behave as one client operation.

## Important formulas
- Replicas and crash budget: `n = 2f + 1`.
- Fast quorum: `f + floor((f + 1) / 2)`.
- Regular/prepare/recovery majority: `f + 1`.
- Minimum fast/recovery intersection: `floor((f + 1) / 2)`.
- Ambiguous fast-accepted recovery interval: `[floor((f + 1) / 2), f)`.
- Slowdown definition uses a threshold `t` above normal replica response time; an `s`-slowdown-tolerant RSM has response-time behavior `T` close to counterfactual `T'` after replacing the `s` slowest replicas with clones of the next-fastest replica.

## Relationship to other protocols
Copilot applies Paxos ballots independently to each pilot log and uses EPaxos-inspired dependencies to combine logs. Unlike [[EPaxos]], it orders every command twice, has only one prefix dependency per entry, totally orders all commands, and has fast takeovers. Ping-pong batching is inspired by [[Mencius]].

Unlike ordinary leader election, fast takeover is triggered by a specific command blocked at execution and completes only the entries needed to unblock it. Unlike [[CURP]], Copilot's redundancy covers ordering and execution through two replicated logs rather than unordered witness durability.

## Limitations
- The design targets one slow replica; `s > 1` is future work.
- Byzantine replicas can manipulate mechanisms such as takeover and ping-pong batching; Copilot does not tolerate them.
- The complete recovery value-picking proof is outside the source PDF in an accompanying technical report.
- The evaluation is datacenter-focused; geo-replicated evaluation is left as future work.
- Proactive duplication costs messages and processing. The paper reports maximum throughput about `8%` below non-thrifty Multi-Paxos and `13%` below the thrifty baseline when no replica is slow.

## Open questions
- TODO: Ingest the accompanying technical report before formalizing every recursive recovery subcase.
- How should `s`-slowdown-tolerance for `s > 1` trade redundant processing against quorum availability?
- Can the proactive two-pilot idea be made Byzantine-safe without letting a malicious pilot force repeated takeovers?
- How should a formal model state the qualitative requirement that `T` and `T'` be "close"?

## Related pages
[[Copilot]], [[slowdown-tolerance]], [[quorum]], [[fast-path]], [[slow-path]], [[dependency]], [[recovery]], [[linearizability]], [[recoverability]], [[protocol-catalog]], [[quorum-systems]], [[fast-paths]], [[commit-rules]], [[recovery-rules]], [[proof-techniques]]
