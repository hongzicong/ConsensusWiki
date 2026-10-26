---
type: paper
title: Linearizable Quorum Reads in Paxos
authors: Aleksey Charapko; Ailidani Ailijiang; Murat Demirbas
year: 2019
venue: 11th USENIX Workshop on Hot Topics in Storage and File Systems (HotStorage '19)
source: raw/pqr.pdf
protocols: [PQR]
tags: [paxos, quorum-read, rinse, linearizability, read-optimization]
status: ingested
---

# Linearizable Quorum Reads in Paxos

## One-sentence summary

PQR moves reads from the Paxos leader to a client-driven follower quorum, then rinses a dirty observation by waiting for the selected slot to execute at one replica.

## Why this paper matters

A majority read of only committed values can miss a completed write whose commit notification has not reached followers. Reading accepted values instead sees that write but may expose an unchosen value. PQR combines quorum discovery with execution confirmation to address both hazards without leases or changes to the write path (§§1, 4).

Bibliography: the local PDF omits a venue/date banner; year and venue are confirmed by the [USENIX proceedings entry](https://www.usenix.org/conference/hotstorage19/presentation/charapko). Protocol details below come from `raw/pqr.pdf`.

## System model

- Paxos/Multi-Paxos deterministic replicated state machine with globally ordered log slots.
- A stable write leader, follower replicas, and clients that coordinate read requests directly.
- Followers track highest accepted and executed slots, including optional per-object/key frontiers.
- Highest accepted slot may lie beyond gaps; execution must wait until all preceding commands are committed and applied in order.
- The evaluation implements a key-value store in the Go Paxi framework.

Source: §§3-4, 6; Figure 2.

## Fault model

Inherited non-Byzantine Paxos crash/nonresponse model; no Byzantine read certificates are specified. Base write/recovery progress needs its configured quorums. The paper does not give a separate failure theorem for follower-only reads or a complete crash/restart protocol. With majority reads over total membership, excluding the leader can reduce read availability even if a write majority survives; see Quorum system.

## Timing assumptions

No leases, synchronized clocks, bounded clock drift, or network-delay bound are required by the read safety mechanism. Read progress needs responsive quorum members and eventual execution of the selected prefix, which relies on host Paxos progress. Timeouts and stable leadership for host liveness are inherited rather than newly specified (§§3-4).

## Main idea

Read the largest accepted slot from a quorum intersecting write Phase 2 quorums. Before returning its result, confirm execution at any one follower. That follower's execution reflects a globally chosen and locally applied prefix, so the client need not write the observed value back to a majority. Already clean accepted/executed evidence can satisfy the rinse condition within the first round (§§4-4.2).

## Protocol roles

- **Client:** sends read queries, collects a quorum, selects the highest accepted slot, and performs/retries rinse.
- **Follower:** reports accepted/executed progress and data, and confirms execution for rinse.
- **Leader:** continues unchanged Multi-Paxos writes and commit dissemination; absent from the intended read path.

Source: §4, Figure 2; §5.

## Message types

The paper describes operations rather than named message structs:

- Read query and follower reply containing latest accepted slot; rinse-free optimization also needs executed progress and the value/result (§§4-4.1).
- Single-node execution-status query and confirmation; retry if selected slot is pending (§4).
- Unchanged host Phase 1 propose/promise, Phase 2 accept/accepted, and Phase 3 commit messages (§3, Figure 1).

Unclear: exact reply layout, ballot/value identity fields, and read-result payload during overwrite/recovery are not specified by pseudocode in this paper.

## Local state

Host ballots, accepted log values/slots, learned commits, and executed prefix. Additional read-facing state exposes highest accepted/executed slot globally or per object. A client keeps quorum responses, selected maximum slot, and pending rinse status. Frontier names here describe the paper's prose; it does not define algorithm variable names (§§3-4.1).

## Normal path

1. Client queries followers without routing through the leader.
2. Collect replies from a read quorum and select the highest accepted slot, ignoring gaps only for this maximum computation.
3. If rinse is required, query one follower until the selected slot has executed.
4. Return the result justified by that executed state; gaps must be resolved before execution can certify a prefix.

Writes continue through unchanged Multi-Paxos (§§3-4).

## Fast path

Rinse-free read: quorum responses show latest accepted equals latest executed, so the selected result is already execution-confirmed. No identical-value quorum is required: replicas can have different progress frontiers; the client still selects the highest accepted slot. §4.1 extends the check to per-object accepted/executed frontiers, avoiding rinse for unrelated writes.

If any queried replica reports a dirty object (`accepted != executed`), §4.1 calls for rinse. One client-to-quorum RTT is the favorable case; the write leader is bypassed. No new fast write quorum, pre-accept phase, or commutativity requirement exists.

## Slow path

A dirty quorum observation triggers at least one additional client-to-replica rinse RTT. If the slot has not executed, retry; the algorithm has no universal two-RTT bound. Continuous writes need not be absent, but the chosen prefix must become executable. The evaluation observed no reads above two RTTs in its tested workload; that is an empirical observation, not a termination bound (§§4, 6).

## Recovery path

PQR introduces no safe-value-selection or write-repair protocol. A host Paxos leader recovers pending slots using ordinary Phase 1/Phase 2; a rinsing client waits for execution through its selected slot, including gap resolution. Any single follower can supply execution confirmation, but the paper does not specify failover/timeouts for a crashed rinse target or the complete binding of previously read data to a value chosen after recovery (§§3-4).

The invariant required by this read mechanism is that a returned result comes from a chosen, applied prefix covering the discovered frontier; simply observing an accepted value does not establish that invariant.

## Commit rule

Writes retain host Paxos commitment: Phase 2 acceptance by its write quorum, followed by commit dissemination and ordered execution. PQR does not commit a new read command or modify that write quorum.

Read response requires quorum discovery **and** execution confirmation for the selected maximum accepted slot, either already present in clean first-round evidence or obtained from one follower during rinse. Returning provisional accepted bytes before that condition is unsafe (§§3-4.1, 6).

## Quorum system

- Paper uses `n`/`N` for total deployment size; standard read count in Table 1 is `⌊n/2⌋ + 1`.
- Standard host Phase 1/recovery and Phase 2/write: ordinary majorities. Derived base crash budget is `n - (⌊n/2⌋ + 1)`.
- First read phase: a quorum intersecting every possible Phase 2 write quorum; baseline is an ordinary majority selected from followers, excluding leader (§4, Figure 2).
- Rinse: **one** executed-state confirmation; this replaces neither discovery nor host recovery quorum.
- Fast write quorum: not applicable. Rinse-free reads use the same discovery quorum.
- Flexible extension preserves host `Q1`/`Q2` intersection and chooses a read quorum intersecting `Q2` (§4.2).
- Examples as stated: `N = 5`, `Q2 = 4`, read quorum `2`; `N = 7`, `Q2 = 5`, read quorum `3`.

Derived threshold condition for unconstrained sets: `q_read + q_write > N`; host Paxos separately needs `q1 + q2 > N`. Same-phase/read-read intersection is not the discovery obligation.

**Wording/availability caveat:** §4 says a majority of followers, whereas Table 1 counts a majority of total `n`. Do not use a majority of only `n-1` followers if that can miss a valid write quorum. For `n=5`, a leader-excluding read quorum of `3` followers is unavailable after two follower crashes, even though the leader and two followers still form a write majority. Leader inclusion as a failover policy is not specified in the PDF.

## Conflict handling

No dependency graph or special ordering of conflicts: writes remain host-ordered. A concurrent update makes accepted/executed frontiers differ and may trigger rinse. Per-object frontiers reduce false interference from other keys. Unchosen speculative data cannot be returned merely because it has the highest slot (§§4.1, 6).

## Safety argument

The paper gives explanatory reasoning and experimental history checking, not a full lemma-based proof. The following is a synthesis of §§1, 3-4 and 6 under the stated host-execution assumptions:

1. A write completed before read invocation has a Phase 2 quorum. Its intersection with read discovery gives a report at least as recent as that write's slot, even if commit notifications lag.
2. The maximum observed slot may be newer and only partially accepted. Rinse prevents exposing this provisional observation before execution establishes a chosen applied prefix.
3. Execution at one replica implies earlier slots are resolved and applied; one confirmation is sufficient after quorum discovery.
4. Thus the result can be tied to a durable prefix covering earlier completed writes. A complete formalization must additionally bind returned data to the final executed value and prove a read linearization point across recovery.

§6 demonstrates violations with quorum reads when rinse is disabled. Quorum intersection supplies visibility; it alone does not stabilize an unchosen value.

## Liveness argument

A responsive discovery quorum plus a responsive replica that eventually executes the selected slot allows completion. Missing commits, log gaps, a crashed rinse target, or host leader recovery can delay reads. Larger flexible write quorums improve read quorum size at the cost of write availability/latency. There is no independent PQR termination theorem or bound (§§4-4.2).

## Key proof ideas

- Separate accepted/anchored, committed/learned, and executed state.
- Use read/write intersection to observe a lower bound on necessary progress.
- Use one executed prefix as global commitment evidence, instead of client read-repair to a majority.
- Reuse clean first-round progress as rinse evidence.
- Treat per-object progress optimization and host flexible quorums as extra modeling obligations, not interchangeable with ordinary global-prefix counters.

## Important formulas

Table 1, preserving `n` as node count and `r` as read probability, best-case communication counts (rinse omitted):

| Component | Paxos | Paxos Read From Leader | Paxos Quorum Read |
|---|---|---|---|
| Leader | `2n` | `2n(1 − r) + 2r` | `2n(1 − r)` |
| Follower | `2` | `2(1 − r)` | `2` |
| Client | `2` | `2` | `2(1 − r) + 2r(⌊n/2⌋ + 1)` |

§4.2 examples:

```text
N = 5, Q2 = 4, PQR quorum = 2
N = 7, Q2 = 5, PQR quorum = 3
```

The general read/write cardinality inequality above is a derived explanation, not a displayed formula in the PDF.

## Relationship to other protocols

[[PQL]] moves lease obligations onto writes to support one-holder reads. PQR keeps writes unchanged and pays a quorum on reads, avoiding lease timing and failed-holder expiration waits. [[Bodega]] also supports local reads but uses roster leases and responder-covering write quorums. [[FPaxos]] provides the cross-phase intersection idea used by the optional larger-write/smaller-read tradeoff. PQR is a read extension, not leaderless write consensus (§§2, 4.2).

## Limitations

Client coordination and quorum fanout; dirty-read and gap latency; follower-only read availability can be weaker than host write availability; larger flexible write quorums reduce write availability. The paper lacks complete pseudocode, failure/reconfiguration fencing, and a full proof. §6 checks over a hundred 30-second histories without observed violations; this does not establish correctness for all executions. Related-work claims about other databases are historical paper claims, not independently adopted wiki facts.

## Open questions

- TODO: specify exact read-value selection and final execution confirmation across higher-ballot overwrite or a no-op filling the selected slot.
- TODO: formalize per-key frontiers and read snapshot semantics for multi-key queries, RMWs, and state-machine reads that are not simple register retrievals.
- Unclear: intended fallback when a total-membership majority cannot be formed using followers alone.
- TODO: specify replica restart persistence, log compaction, membership epochs, and retry/failover handling; the PDF does not supply these algorithms.

## Related pages

[[PQR]], [[PQL]], [[rinse]], [[quorum]], [[fast-path]], [[recovery]], [[linearizability]], [[flexible-quorum]], [[read-strategies]], [[read-freshness]], [[quorum-intersection]], [[protocol-catalog]]
