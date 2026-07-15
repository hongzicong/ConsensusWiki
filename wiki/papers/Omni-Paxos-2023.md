---
type: paper
title: "Omni-Paxos: Breaking the Barriers of Partial Connectivity"
authors: [Harald Ng, Seif Haridi, Paris Carbone]
year: 2023
venue: "Eighteenth European Conference on Computer Systems (EuroSys '23)"
source: raw/omni_paxos.pdf
protocols: [OmniPaxos]
tags: [paxos, smr, partial-connectivity, leader-election, reconfiguration]
status: ingested
---

# Omni-Paxos: Breaking the Barriers of Partial Connectivity

## One-sentence summary

Omni-Paxos decouples connectivity-aware leader election, prefix-preserving log replication, and cross-configuration log migration so an SMR can make progress whenever at least one quorum-connected server exists.

## Why this paper matters

The paper identifies three partial-connectivity failures that can deadlock or livelock common leader-based RSMs: quorum loss by a still-alive leader, a constrained election where the only quorum-connected candidate has an outdated log, and chained elections caused by gossiping newer terms. Omni-Paxos removes log freshness from candidacy, detects quorum connectivity directly, and lets Sequence Paxos synchronize any elected leader before it replicates.

The system also separates reconfiguration from log replication. Decided log segments can be migrated in parallel by the service layer instead of only by the old leader.

## System model

- A fixed configuration contains servers running one [[OmniPaxos]] Sequence Paxos instance and one Ballot Leader Election (BLE) instance.
- Clients submit commands to a replicated state machine backed by one strictly growing log.
- Servers communicate over bidirectional, session-based FIFO perfect links; the implementation uses TCP and explicitly resynchronizes after a session drop.
- A service layer spans configurations and stores the decided replicated log.
- Reconfiguration stops one fixed configuration and starts another.

Source: §§3-6.

## Fault model

- Fail-recovery, non-Byzantine servers.
- A correct server may fail and recover finitely many times.
- Persistent state survives recovery.
- Messages may be dropped or delayed; a partial partition may systematically drop traffic on only some links.
- The paper does not parameterize failures with an explicit `n = 2f + 1` formula; its replication and election rules use majorities of a fixed configuration.

Source: §3, "Preliminaries."

## Timing assumptions

Safety is based on persistent ballots, prefix-preserving synchronization, and majority intersection. Liveness assumes partial synchrony: there are sufficiently long synchronous periods, and BLE eventually learns a heartbeat duration `T_delay` for which connected replies are not late. Progress additionally requires at least one quorum-connected server and enough live links/servers to form replication majorities.

Source: §§3 and 5.2.

## Main idea

Omni-Paxos separates three responsibilities:

1. **Ballot Leader Election:** elect a server based only on whether it is directly connected to a majority and on its ballot order.
2. **Sequence Paxos:** run a mandatory Prepare/log-synchronization phase after election, allowing even a trailing leader to adopt all chosen log entries.
3. **Service-layer reconfiguration:** decide a stop-sign in the old log, then migrate only decided entries to the next configuration independently of the old leader.

The decoupling removes the need to elect the most up-to-date log and avoids gossiping the current replication term as leader-election state.

## Protocol roles

- **Client:** proposes commands and receives decided results.
- **Sequence Paxos leader:** synchronizes a majority, appends commands, replicates them, and advances the decided prefix.
- **Sequence Paxos follower:** promises ballots, transfers missing suffixes, accepts synchronized logs and new entries, and learns decisions.
- **BLE participant:** exchanges ballot/connectivity heartbeats and emits a `Leader` event.
- **Service layer:** exposes the unified log, decides when a new configuration may start, and migrates decided log entries.

## Message types

Sequence Paxos:

- `Prepare(n, acceptedRnd, logIdx, decidedIdx)`
- `Promise(n, acceptedRnd, logIdx, decidedIdx, suffix)`
- `AcceptSync(n, suffix, syncIdx)`
- `Accept(n, command)`
- `Accepted(n, logIdx)`
- `Decide(n, decidedIdx)`
- `PrepareReq`
- `Reconnected` event

BLE:

- `HBRequest(round)`
- `HBReply(round, ballot, qc)`
- `Leader(server, ballot)` event

Reconfiguration uses a decided stop-sign entry `SS` containing the next configuration.

Source: Figures 3-4 and §6.

## Local state

Persistent Sequence Paxos state on every server:

- `log[]`
- `promisedRnd`
- `acceptedRnd`
- `decidedIdx`

Volatile Sequence Paxos state includes `state`; leader-only state includes `currentRnd`, `promises{}`, `maxProm`, `accepted[]`, and `buffer[]`.

BLE persists the current leader ballot `l`. Its volatile state is heartbeat round `r`, local ballot `b`, quorum-connectivity flag `qc`, heartbeat `delay`, and received `ballots{}`.

Source: Figures 3b and 4.

## Normal path

After BLE elects a leader and Sequence Paxos completes Prepare, the leader enters Accept state. It appends each client command, sends `Accept` to promised followers, and records `Accepted` log indices. When a majority has accepted an index, the leader advances `decidedIdx` and sends `Decide` to promised followers. FIFO delivery allows commands to be pipelined without waiting for earlier entries to be decided.

Source: §4.1.2 and Figure 3.

## Fast path

There is no separate Fast Paxos-style fast path, fast quorum, or conflict-free commit predicate. Stable Accept replication decides an entry in one leader-to-followers round trip, and pipelining hides per-entry serialization, but the leader remains on the path and still waits for a majority.

Source: §§4.1.2, 7.1, and 9.

## Slow path

The Prepare phase is the leader-change and resynchronization path. A newly elected leader gathers a majority of `Promise` messages, chooses the most up-to-date returned log, adopts its suffix, and sends `AcceptSync` before accepting new commands. Client commands received during Prepare are buffered unless the adopted log ends with `SS`.

This is not a workload-conflict fallback: one leader assigns a strict log order.

Source: §4.1.1 and Figure 3b.

## Recovery path

### Leader replacement

BLE elects a quorum-connected server with the highest eligible ballot. Sequence Paxos then gathers a majority of promises. It ranks logs first by `acceptedRnd` and, on equal `acceptedRnd`, by `logIdx`. The leader adopts the highest-ranked suffix and synchronizes promised followers with `AcceptSync` before entering Accept.

### Server or link recovery

A recovering server reloads persistent state, enters recover state, and sends `PrepareReq` to all peers. The current leader responds with `Prepare`, after which ordinary synchronization runs. Re-established link sessions use the same mechanism.

Source: §§4.1.1 and 4.1.3.

## Commit rule

An entry at `logIdx` is **chosen** once a majority has accepted that index. The leader marks the index decided when it observes that majority and broadcasts `Decide`. Because entries are replicated in FIFO order, deciding an index also decides every preceding entry.

During reconfiguration, choosing `SS` prevents any later entry from being decided in that configuration.

Source: §§4.1, 4.1.2, and 6.

## Quorum system

- **Total replicas `N`:** fixed membership within one configuration.
- **Fault parameter `f`:** Unclear as a paper-level symbolic parameter; the paper states majority conditions instead of an `N,f` formula.
- **Prepare quorum:** a majority of promises, counting the leader's own promise.
- **Accept/commit quorum:** a majority of accepted log indices, counting the leader's own acceptance.
- **Recovery quorum:** the Prepare majority.
- **BLE observation quorum:** a server runs `checkLeader()` only after receiving at least a majority of heartbeat records, including itself.
- **Quorum-connected server:** a correct server with a direct link to at least a majority of correct servers, including itself.
- **Fast quorum:** none.
- **Intersection:** ordinary majority intersection preserves at least one server that carries any chosen prefix into a later Prepare.
- **Flexibility:** fixed majority quorums within each configuration; this paper does not introduce [[flexible-quorum]] families.

Source: §§4.1, 5.1-5.2, Figures 3b-4.

## Conflict handling

Client-command conflicts do not select a separate path. The single Sequence Paxos leader appends commands in strict order. Competing leaders are separated by unique increasing ballots and promises; after a higher ballot wins, non-chosen suffixes at followers may be overwritten during `AcceptSync`, but a chosen prefix is preserved.

Source: §§4.1-4.2.

## Safety argument

Sequence Paxos specifies:

- **SC1 Validity:** a decided log contains only proposed commands.
- **SC2 Uniform Agreement:** any two decided logs are prefix-comparable.
- **SC3 Integrity:** each later log decided by one server strictly extends its earlier decided log.

The proof adapts Paxos P2 from equality of values to prefix extension of sequences. Prepare preserves P2c by adopting the highest-numbered accepted sequence among a majority, then synchronizing that sequence before new commands are appended. The appendix proves SC3 by induction over decisions and role cases, then proves SC2 by contradiction using majority intersection and SC3.

Reconfiguration safety depends on deciding `SS`, forbidding later old-configuration decisions, and allowing a new server to start only after fetching the complete decided log.

Source: §§4, 6, and Appendix A.

## Liveness argument

BLE defines three properties:

- **LE1 QC-Completeness:** if a quorum-connected server exists, quorum-connected servers eventually elect one.
- **LE2 QC-Eventual Agreement:** eventually, a majority contains no two quorum-connected servers that elect differently.
- **LE3 Monotonically Increasing Unique Ballots:** each server observes strictly increasing, unique elected `(server, ballot)` pairs.

LE2 deliberately permits different leaders in different majorities. Their majorities overlap in a non-QC server, and LE3 ensures that server eventually promises the maximum ballot in Sequence Paxos. Under partial synchrony and one QC server, the elected server completes Prepare and resumes replication.

The abstract claims recovery in at most four election timeouts under extreme partial partitions. The evaluation reports four heartbeat rounds for quorum loss and three timeouts for the constrained-election scenario; these are liveness/performance claims under the paper's model and experiments, not safety bounds.

Source: §§5.1-5.2 and 7.2.

## Key proof ideas

1. **Prefix-valued Paxos P2:** higher chosen sequences extend lower chosen sequences.
2. **P2c through log adoption:** a new proposal extends the highest-numbered accepted sequence in a Prepare majority.
3. **Suffix optimization:** FIFO sessions and `AcceptSync` preserve whole-sequence reasoning while transmitting only missing suffixes.
4. **SC3 before SC2:** per-server monotonic decisions plus majority intersection imply cross-server prefix agreement.
5. **Election/replication separation:** BLE needs only identify a QC maximum-ballot candidate; Sequence Paxos repairs log freshness.
6. **Cross-configuration stop boundary:** `SS` closes the old log before the next configuration starts.

## Important formulas

The paper states the Sequence Consensus properties as:

```text
SC1. Validity: If a server decides on a log L then L only contains proposed commands.
SC2. Uniform Agreement: For any two servers that decided logs L and L' respectively then one is the prefix of the other.
SC3. Integrity: If a server decides on a log L and later decides on L' then L is a strict prefix of L'.
```

Sequence Paxos adapts Paxos P2 to:

```text
If a proposal with sequence v is chosen, then every higher-numbered proposal that is chosen has v as a prefix.
```

Its P2c statement is:

```text
For any v and n, if a proposal with sequence v and number n is issued, then there is a majority-set S of acceptors such that sequence v is an extension to the sequence of the highest-numbered proposal less than n accepted by the acceptors in S.
```

The election properties are:

```text
LE1. QC-Completeness
LE2. QC-Eventual Agreement
LE3. Monotonically Increasing Unique Ballots
```

The progress threshold emphasized by the paper is:

```text
at least 1 quorum-connected server
```

The paper uses majority quorums but does not state a separate symbolic majority-size formula in terms of `N` and `f`.

## Relationship to other protocols

- Like Multi-Paxos, [[OmniPaxos]] uses ballot promises and majority acceptance, but Sequence Paxos decides one growing log rather than independent slots.
- Unlike Raft and Zab, candidate eligibility does not require the maximum log; Prepare repairs a stale leader after election.
- Unlike Viewstamped Replication and Zab, a candidate need not be elected by a majority of quorum-connected servers.
- BLE avoids replication-term gossip that can cause chained elections.
- Reconfiguration uses a stop-sign derived from Stoppable Paxos but migrates decided data through a cross-configuration service layer.
- The paper's majority quorum system is distinct from [[FPaxos]]; its flexibility is architectural, not flexible-quorum geometry.

## Limitations

- Byzantine behavior is outside the model.
- Liveness depends on partial synchrony, heartbeat convergence, and existence of a QC server.
- The protocol assumes session-based FIFO perfect links; practical session drops require explicit resynchronization.
- The paper does not provide an explicit `N,f` fault-tolerance formula.
- Reconfiguration starts a new server only after it fetches the complete decided log; the evaluation initializes logs with millions of small entries but does not establish behavior for every state-transfer/storage design.
- BLE prioritizes stable QC leadership, not necessarily the best-connected or lowest-latency leader.
- The formal appendix proves Sequence Paxos safety; the paper gives correctness arguments for BLE and reconfiguration rather than one end-to-end mechanized proof of the complete system.

## Open questions

- How should BLE adapt heartbeat delays without oscillating under asymmetric or rapidly changing latency?
- Can QC-aware leader selection incorporate latency/topology priorities without sacrificing stable leadership?
- How should snapshots and compacted state replace complete-log migration while preserving the stop-sign boundary?
- Can [[flexible-quorum]] systems be combined safely with the QC definition and BLE majority-heartbeat rule?
- What is the minimal formal cross-configuration invariant connecting `SS`, migration completion, and new-configuration startup?

## Related pages

[[OmniPaxos]], [[partial-connectivity]], [[sequence-consensus]], [[leader]], [[quorum]], [[recovery]], [[agreement]], [[validity]], [[liveness]], [[recoverability]], [[quorum-intersection]], [[reconfiguration]]
