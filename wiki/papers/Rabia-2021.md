---
type: paper
title: Rabia: Simplifying State-Machine Replication Through Randomization
authors: Haochen Pan, Jesse Tuglu, Neo Zhou, Tianshu Wang, Yicheng Shen, Xiong Zheng, Joseph Tassarotti, Lewis Tseng, Roberto Palmieri
year: 2021
venue: SOSP 2021
source: raw/rabia.pdf
protocols: [Rabia]
tags: [smr, randomized-consensus, weak-validity, common-coin, leaderless]
status: ingested
---

# Rabia: Simplifying State-Machine Replication Through Randomization

## One-sentence summary
Rabia is a leaderless SMR framework that orders log slots with randomized weak multi-valued consensus, using forfeited `⊥` slots to avoid complex fail-over and recovery protocols.

## Why this paper matters
Rabia is useful as a design point opposite leader/fail-over-heavy Paxos-family systems: every replica participates symmetrically in a randomized consensus instance, so failures are handled inside the core protocol rather than by a separate leader recovery path.

## System model
Asynchronous distributed system of `n` replicas/servers implementing log-based [[SMR]]. Clients send each request to an assigned proxy replica; the proxy stores the request in a local min priority queue and forwards it to all other replicas. Replicas repeatedly agree on the value for each log slot.

## Fault model
Fail-stop crash model. At most `f` replicas may crash. The paper states Rabia ensures safety and liveness as long as `n ≥ 2f + 1`. Byzantine faults are not considered.

## Timing assumptions
Safety is for the asynchronous model with crash failures. Liveness is probabilistic: each Weak-MVC instance terminates with probability 1 as long as a majority of replicas are non-faulty. Performance depends on a stable datacenter network that often gives replicas the same oldest pending request.

## Main idea
Convert binary randomized consensus into a weak multi-valued consensus protocol usable for SMR. Each replica proposes the oldest request in its local priority queue. If replicas see a majority proposal, they run binary consensus on state `1` and return that request; otherwise they run binary consensus on state `0` and forfeit the slot by returning `⊥`.

## Protocol roles
- Client: sends a request to an assigned proxy replica and waits for a response.
- Proxy replica: receives a client request, enqueues it, forwards it to all replicas, and eventually responds after execution.
- Replica/server: participates in every Weak-MVC instance, stores the log, executes decided non-`⊥` slots, and can help lagging replicas catch up.

There is no leader, command leader, or fail-over leader in the core protocol.

## Message types
Weak-MVC uses:
- `Proposal`: carries the proposed client request in the exchange stage.
- `State`: carries phase number `p` and binary `state ∈ {0, 1}`.
- `vote`: carries phase number `p` and `vote ∈ {0, 1, ?}`.

The paper also describes request forwarding and catch-up request messages for slow replicas.

## Local state
Each replica maintains:
- `PQ_i`: local min priority queue of pending client requests, keyed by request timestamp.
- `seq`: current log slot index.
- `log[seq]`: decided request or `⊥`.
- In Weak-MVC: `state`, `vote`, and phase number `p`.
- A dictionary to check whether a request at the head of `PQ_i` is already in the log.

## Normal path
For each slot:
1. Select `proposal_i`, the first element in `PQ_i` not already in the log.
2. Run `Weak-MVC(proposal_i, seq)`.
3. Store the output in `log[seq]`.
4. If the output is `⊥` or differs from `proposal_i`, push `proposal_i` back into `PQ_i`.
5. Increment `seq`.

## Fast path
Weak-MVC's fast path terminates in three message delays:
1. Exchange `Proposal` messages and wait for at least `n - f`.
2. If some request appears at least `⌊n/2⌋ + 1` times, set `state ← 1`; otherwise set `state ← 0`.
3. Exchange `State` messages and wait for at least `n - f`.
4. If a value `v` appears at least `⌊n/2⌋ + 1` times, set `vote ← v`; otherwise `vote ← ?`.
5. Exchange `vote` messages and wait for at least `n - f`.
6. If a non-`?` value `v` appears at least `f + 1` times, return `FindReturnValue(v)`.

The fast path is guaranteed when all replicas have the same proposal, and also when no majority proposes the same request. The second case returns `⊥`.

## Slow path
If the first binary-consensus phase does not terminate, replicas update `state` for the next phase. If a non-`?` vote appears at least once, they adopt that value; otherwise they set `state ← CommonCoinFlip(p)`. The paper proves each phase has probability at least `1/2` of leading to termination.

## Recovery path
Rabia does not have a separate fail-over/recovery protocol. If one replica decides and crashes, the vote evidence that allowed the decision forces surviving replicas into the same value in later phases. Slow replicas can catch up by asking other replicas for already agreed proposals.

For lost messages, the paper notes log compaction must be more conservative: replicas need to know a quorum has agreed on a slot before compacting it.

## Commit rule
A slot is decided when a replica returns from Weak-MVC:
- If binary decision `v = 1`, `FindReturnValue(1)` returns the request `m` that appears at least `⌊n/2⌋ + 1` times in the Proposal messages received in Phase 0.
- If binary decision `v = 0`, `FindReturnValue(0)` returns `⊥`.

This is a weak multi-valued consensus decision, not ordinary validity: a slot may contain a real client request or `⊥`.

## Quorum system
Always record:
- Total replicas: `n`.
- Fault tolerance: `f`.
- Fault bound: `n ≥ 2f + 1`.
- Proposal wait quorum: at least `n - f` Proposal messages.
- State wait quorum: at least `n - f` phase-`p` State messages.
- Vote wait quorum: at least `n - f` phase-`p` vote messages.
- Majority threshold for proposals/states: `⌊n/2⌋ + 1`.
- Decision threshold for non-`?` votes: `f + 1`.
- Leader inclusion: none; no leader is required.

Intersection is majority-style: with `n ≥ 2f + 1`, quorums of size `n - f` intersect in at least one non-faulty participant, and two majorities cannot support different concrete proposals in the exchange stage.

## Conflict handling
Rabia does not use dependencies or sequence numbers to resolve command conflicts. If proposal sets do not reveal a majority request, the protocol can forfeit the slot as `⊥`, return the original proposal to the priority queue, and try again in a later slot. Duplicate client requests are handled by unique client/request IDs during execution.

## Safety argument
Weak-MVC safety gives weak validity and agreement. Agreement relies on the randomized binary consensus stage: in a phase, quorum thresholds ensure there is at most one non-`?` vote value, and if a value is decided then later phases become value-locked on that value.

## Liveness argument
The paper proves probabilistic termination: each phase has probability at least `1/2` of leading to termination, and therefore Weak-MVC terminates with probability 1. The expected number of rounds is `2 · 2 + 1 = 5`: one exchange round plus two binary-consensus phases in expectation.

## Key proof ideas
- Reduce multi-valued SMR agreement to binary agreement on whether a majority proposal exists.
- Weaken validity to allow `⊥`.
- Use `f + 1` non-`?` votes as enough evidence that at least one correct replica carries the value forward.
- Prove value-locking across phases.
- Split machine-checked safety proof across Ivy and Coq.

## Important formulas
- Fault bound: `n ≥ 2f + 1`.
- Proposal/State/vote wait threshold: `n - f`.
- Majority proposal/state threshold: `⌊n/2⌋ + 1`.
- Decision vote threshold: `f + 1`.
- Per-phase termination probability: at least `1/2`.
- Probability lower bound in the paper: `(1 − 1/2)^{t−1} 1/2`.
- Average round complexity: `5`.

## Relationship to other protocols
Rabia contrasts with [[FastPaxos]], [[Mencius]], [[EPaxos]], [[Atlas]], and [[SwiftPaxos]] by avoiding leaders, command leaders, dependency recovery, and fail-over. It is closer in spirit to Ben-Or-style randomized consensus, but adapted to SMR through Weak-MVC and forfeited slots.

## Limitations
- Performance is best in stable datacenter networks; multi-zone or larger-`n` deployments reduce throughput.
- Communication is `O(n^2)` per slot on the fast path.
- The implementation evaluated in the paper does not include pipelining.
- Weak validity means the log may contain `⊥` slots.
- The common-coin implementation assumes faults and message delays are independent of the coin outcome.

## Open questions
- TODO: Extract the exact technical-report statements for the full Ivy/Coq safety proof if `raw/` later includes the report.
- TODO: Clarify how conservative log compaction should be specified when messages can be lost.

## Related pages
[[Rabia]], [[randomized-consensus]], [[common-coin]], [[fast-path]], [[slow-path]], [[leader]], [[agreement]], [[validity]], [[liveness]], [[linearizability]]
