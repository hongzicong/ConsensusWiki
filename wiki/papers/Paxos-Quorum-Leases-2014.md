---
type: paper
title: "Paxos Quorum Leases: Fast Reads Without Sacrificing Writes"
authors: Iulian Moraru; David G. Andersen; Michael Kaminsky
year: 2014
venue: ACM Symposium on Cloud Computing (SoCC '14)
source: raw/pql.pdf
protocols: [PQL]
tags: [paxos, quorum-lease, local-read, strict-serializability]
status: ingested
---

# Paxos Quorum Leases: Fast Reads Without Sacrificing Writes

## One-sentence summary

PQL enables strongly consistent local reads at per-object subsets of Paxos replicas by combining majority-backed timed promises with synchronous notification of all relevant lease holders before writes commit.

## Why this paper matters

Quorum leases explore the space between leader-only reads and read-anywhere/write-everywhere replication. Their benefit depends on aligning the holders with a write quorum and read locality; the name does not require the holder set itself to be a majority. The evaluation uses five EC2 regions and reports over 80% local reads per site and over 70% minimum-latency writes in its skewed workload, rather than a universal workload guarantee (§§3.2, 5.2).

## System model

- `N` replicas implement deterministic SMR with an ordered sequence of independent Paxos instances.
- Replicated state contains separately addressable objects; operations declare read/write objects. Composite operations and batching are supported.
- The design is presented for Multi-Paxos with a stable leader; lease management is separate from command consensus.
- Communication is asynchronous; messages may be lost or delayed indefinitely.

Source: `raw/pql.pdf`, §§2.1, 3.1, 3.3, 3.9.

## Fault model

Non-Byzantine crashes or indefinite nonresponse. Paxos progress requires at least `⌊N/2⌋ + 1` working replicas. A failed active holder can temporarily block writes to its leased objects even when a Paxos majority survives (§§2.1, 3.7).

## Timing assumptions

Clocks are not synchronized. Similar clock rates and a guard margin must account for drift, so a holder's usable lease expires before the corresponding grantor promise stops binding. Lease safety depends on this timing assumption; base Paxos safety does not. Guard duration larger than maximum RTT and renewal before expiry are successful-activation/performance conditions, not a bound on all network delays (§§3.1, 3.5, 3.8).

## Main idea

A lease `(Q, O)` authorizes holders `Q` to read objects `O`. A grantor promises to notify a holder synchronously before committing an affected update and to attach that obligation to relevant Paxos replies. Because any accepting majority intersects an active holder's grantor majority, a proposer learns the obligation even if it made no promise itself. Lease configurations are agreed through separate Paxos instances; peer-to-peer guarded promises activate them (§§3.2-3.5).

## Protocol roles

Client; stable Multi-Paxos leader/proposer; acceptor/learner replicas; per-object lease holder; lease grantor. Every replica can grant promises to peers. The implementation chooses each majority-sized holder set as the leader plus the `⌊N/2⌋` replicas reading that object most often; the leader has a default lease for other objects (§§2.1, 3.4).

## Message types

- Paxos `Prepare`, prepare acknowledgements, `Accept`, `AcceptOK`/accept replies, and `Committed` notifications.
- Replies carry lease configuration information; the configuration ID is the latest configuration-change Paxos instance number.
- Lease `Guard`, `GuardACK`, `Promise`, `PromiseReply`; renewal promises include elapsed time `t′` and reference acknowledgement sequence `seq_ACK`.
- Configuration proposals, read-frequency information, client requests/replies, forwarding, and failure-detection heartbeats.
- Promises also report the grantor's most recent accepted Paxos instance for catch-up (§3.6).

Source: §§2.1, 3.2-3.6; Figures 1, 4-5.

## Local state

Paxos ballots, accepted commands, committed/executed log; configuration ID and object-to-holder mapping; per-peer `grant_timer`, `guard_timer`, `lease_timer`; last acknowledgement sequence/time; grantor acceptance frontier; read-frequency counters; pending interfering writes/reads. Figure 5 specifies the timers; §§3.4 and 3.6 supply configuration and catch-up state.

## Normal path

The stable leader sends `Accept` and collects majority acceptance plus synchronous notification acknowledgements from every required holder for the modified objects. Receiving `Accept` at a holder can provide this notification without an extra round. Commit notifications disseminate the outcome. A non-holder forwards reads; a holder reads locally only while its lease is active, required catch-up is complete, and interfering accepted writes have been executed or resolved (§§2.2, 3.2, 3.6; §4 footnote 6; §5.2).

## Fast path

An eligible holder executes a read atomically from local state, without a new inter-replica round. For composite reads it must hold leases for every referenced object. The client can contact any such holder; the write leader is bypassed for that read. There is no distinct fast write quorum, dependency matching, or commutativity requirement. An in-flight interfering write delays the read; inactive/missing authority or incomplete catch-up prevents local execution (§§3.6, 5.2).

## Slow path

If no replica can serve the entire read under leases, commit it through ordinary Paxos. Concurrent writes can make a holder wait for commit/execution. Multi-object writes cover the union of all relevant holder sets; batching across those sets has the same cost, so the implementation separates batches by lease quorum (§§3.6, 3.9).

## Recovery path

- A grantor that suspects a failed holder waits a grace period, stops renewing, and requests a configuration excluding that holder.
- Existing promises must expire before their notification obligations disappear. The implementation waits out old promises before activating the new configuration; an alternative may overlap configurations only while honoring both sets of obligations.
- Writes resume when a majority is no longer bound to the failed holder. A new leader still performs ordinary Paxos Prepare/recovery, including lease obligations in replies. Prepare needs this condition only for instances where the grantor already accepted an affected update (§3.2 footnote 2).
- A restarted replica waits `t_wait` before accepting or proposing, ensuring its old promises have expired.
- A newly active reader catches up through the accepted-instance indices in its grantor promises; a lease alone is insufficient evidence of freshness.

Source: §§3.2, 3.5-3.8.

## Commit rule

An update requires ordinary Paxos majority acceptance **and** synchronous notification acknowledgements from all holders required by the still-binding promises learned from the participating replicas. A single-object update in the normal configured case includes every holder in that object's lease quorum. Multi-object updates cover their union. Local read completion is separate from consensus commitment (§§3.2, 3.9).

The §4 optimization also sends accept replies to the replica near the client, permitting it to observe commitment directly from sufficient acceptance plus holder-coverage evidence. Do not count one reply as proof that all holders acknowledged; see the counting ambiguity under Open questions.

## Quorum system

- Paper notation: total replicas `N = |R|`; availability requires `⌊N/2⌋ + 1` correct replicas. Derived maximum crash budget is `N - (⌊N/2⌋ + 1)`.
- Classic acceptance and Prepare/recovery: `⌊N/2⌋ + 1`.
- Distinct fast write quorum: not applicable.
- Holder set `Q`: any subset/size, not necessarily a Paxos quorum. Implementation: `⌊N/2⌋ + 1`, including leader.
- Lease activation at a holder: at least `⌊N/2⌋` unexpired promises from distinct **other** replicas, plus the holder's implicit self-grant.
- Write evidence: flexible Paxos majority plus all binding holder obligations; holder membership is fixed by a configuration until safely changed.
- Required intersection: accepting majority intersects active grantor majority, so some acceptor relays each active holder's obligation. The read itself contacts one eligible holder.

Source: §§2.1, 3.2, 3.4-3.6; Figure 5.

## Conflict handling

Paxos orders writes; no application-level dependency graph is added. Notification of an interfering write suspends serving stale local reads until the write is safe to execute and has been executed. Multi-object read/write operations execute atomically; local multi-object reads require all corresponding leases (§§3.6, 3.9, 5.2).

## Safety argument

§3.6 proves strict serializability. Paxos orders write-only and composite read-write operations; eligible local reads execute atomically. Consider an earlier committed interfering write `W` and later read `R` at a holder:

1. If the lease predates `W`, `W`'s accepting majority intersects the holder's grantor majority. The intersecting acceptor forces the proposer to notify the holder before commit; the holder executes `W` before serving `R`.
2. If activation follows the start of `W`, either a grantor promised before accepting (the first case applies), the holder accepted `W` itself, or an intersecting grantor accepted `W` before promising. In the last case, its reported highest accepted instance forces catch-up through `W` before `R`.

Timer containment keeps the promises binding whenever the holder uses them. Configuration agreement alone does not establish timer containment or catch-up.

## Liveness argument

Paxos needs a responsive majority and eventual effective stable leadership; a fully asynchronous schedule does not guarantee termination. Local-read availability additionally needs timely renewals and caught-up holders. Failed-holder write blocking ends after promises expire and an available majority can proceed without that holder. This is conditional on the clock assumption and stopping renewals, not immediate majority-only availability (§§2.1, 3.7-3.8).

## Key proof ideas

- Separate holder membership, grantor evidence, write acceptance, and local-read eligibility.
- Relay notification obligations through Accept and applicable Prepare replies.
- Use highest-accepted-instance catch-up to cover writes preceding lease activation.
- Separate configuration consensus from peer-to-peer lease activation.
- Preserve grantor/holder time-window containment across lost and delayed messages.

## Important formulas

From §§2.1, 3.2, 3.5, 3.8 and Figure 5:

```text
N = |R|
Paxos majority = ⌊N/2⌋ + 1
active holder: at least ⌊N/2⌋ unexpired promises from different peers
t_wait = t_guard + t_lease
t_grace + t_wait = t_grace + t_guard + t_lease
```

Renewal promise admissibility: received before time `t′ + guard_duration` since sending acknowledgement `seq_ACK`. The grantor can relinquish an unacknowledged renewal after `t_guard + t_lease` from sending it. Acknowledgement can shorten the grantor timer as in Figure 5.

Evaluation: `t_lease = 2 seconds`, renew every `500 milliseconds`, configuration update every `10 seconds`; a `5 second` grace plus `2 second` guard and `2 second` lease gives approximately `9 seconds` worst-case failed-holder write unavailability (§§3.8, 5.2-5.3).

## Relationship to other protocols

[[Bodega]] leases agreement on responder metadata and adds ballot/roster catch-up; PQL directly propagates timed synchronous-notification obligations. Both can place read-serving replicas on write critical paths, but their activation and read-release rules differ. PQL's suggestion of applicability to [[Mencius]] and [[EPaxos]] is not a complete integration algorithm (§6). [[PQR]] later offers a lease-free quorum-read alternative; see [[read-strategies]].

## Limitations

Clock-rate assumptions; all-to-all lease renewal; potentially seconds of failed-holder write blocking; write inflation for multi-object unions; placement/read-locality sensitivity. In-flight interfering writes can stall local reads. The paper gives a case-based consistency argument, not named mechanized lemmas or a full restart-persistence specification.

## Open questions

- Unclear: §4 footnote 6's final wording counts `⌊N/2⌋` total accepting replicas, whereas §2.1 requires `⌊N/2⌋ + 1`; treat self/implicit leader accounting explicitly before implementing that optimization.
- TODO: specify quantitative clock-drift margins and persistence/restart refinement; the paper assumes similar rates without giving a complete drift-parameterized theorem.

## Related pages

[[PQL]], [[quorum-lease]], [[Bodega]], [[quorum]], [[fast-path]], [[recovery]], [[linearizability]], [[read-strategies]], [[quorum-intersection]], [[read-freshness]], [[protocol-catalog]]
