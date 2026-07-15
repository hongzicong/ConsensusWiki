---
type: paper
title: "Flexible Paxos: Quorum Intersection Revisited"
authors: Heidi Howard; Dahlia Malkhi; Alexander Spiegelman
year: 2016
venue: arXiv:1608.06696v1
source: raw/fpaxos.pdf
protocols: [FPaxos]
tags: [paxos, quorum, flexible-quorum, recovery, agreement, multi-paxos]
status: ingested
---

# Flexible Paxos: Quorum Intersection Revisited

## One-sentence summary

Flexible Paxos weakens Paxos's quorum rule from all-quorum intersection to cross-phase intersection, allowing Phase 1 leader-election quorums and Phase 2 replication quorums to trade availability, latency, throughput, and topology while preserving agreement.

## Why this paper matters

Classic Paxos normally uses majorities in both phases, but its safety proof needs only each later Phase 1 quorum to intersect the Phase 2 quorum that could have chosen an earlier value. FPaxos exposes this latent flexibility: Phase 1 quorums may be mutually disjoint, Phase 2 quorums may be mutually disjoint, and neither phase requires majority size by itself.

The practical insight is asymmetric. Multi-Paxos runs Phase 2 for every command but Phase 1 mainly during leader election. Shrinking Phase 2 can improve steady-state latency, throughput, tolerance of slow acceptors, and load distribution, at the cost of larger Phase 1 quorums and therefore harder leader recovery.

## System model

- Asynchronous message-passing consensus over a set `A` of acceptors.
- Processes may fail and messages may be lost.
- Roles are proposer, acceptor, learner, and, for Multi-Paxos, client and stable leader.
- Acceptors persist their highest promise and last accepted proposal/value.
- Single-decree consensus decides one value; Multi-Paxos applies the same rule independently across log slots after aggregating Phase 1.

Source: `raw/fpaxos.pdf`, §§2-3.

## Fault model

The paper considers non-Byzantine process failures and message loss. Safety does not depend on how many acceptors fail. Progress in a phase requires enough live, mutually communicating acceptors to form one valid quorum for that phase.

Failure tolerance is quorum-system-dependent rather than a single global number. With simple quorums, forming both phases is guaranteed through `|Q2| - 1` failures when Phase 2 is the smaller quorum. A stable leader may continue Phase 2 after Phase 1 has become unavailable, but a leader failure then stalls recovery.

## Timing assumptions

Safety holds in the asynchronous model. As with Paxos, progress requires additional synchrony assumptions and a proposer/leader able to complete the required quorum exchanges.

FPaxos does not reduce the number of Paxos phases or client message delays. Its steady-state benefit comes from waiting for fewer Phase 2 acceptors and optionally sending only to likely quorum members. Retransmission may add latency if those selected acceptors are slow or failed.

## Main idea

Let `Q1` denote a valid Phase 1 quorum and `Q2` a valid Phase 2 quorum. Paxos commonly requires every quorum to intersect every other quorum. FPaxos requires only:

```text
∀Q1 ∈ Quorum1, ∀Q2 ∈ Quorum2: Q1 ∩ Q2 ≠ ∅
```

The intersecting acceptor either prevents an older Phase 2 from completing after a higher promise, or reports the earlier accepted value to the later Phase 1 proposer. Therefore the later proposer preserves the earlier decision even when same-phase quorums are disjoint.

## Protocol roles

- **Client:** supplies values/commands to a proposer or stable Multi-Paxos leader.
- **Proposer:** selects a unique proposal number, gathers a Phase 1 quorum, chooses the highest-numbered accepted value returned or a fresh value, and starts Phase 2.
- **Leader:** proposer that has completed aggregated Phase 1 and can run repeated Phase 2 rounds for slots.
- **Acceptor:** persists promises and accepted `(proposal, value)` state; participates in whichever valid `Q1` or `Q2` the proposer chooses.
- **Learner:** learns a value after a valid Phase 2 quorum accepts it.

## Message types

- `prepare(p)` / TLA+ `1a`.
- `promise(p', v')` / `1b`, carrying the acceptor's last accepted proposal/value if any.
- `propose(p, v)` / `2a`.
- `accept(p)` / `2b`, carrying acceptor, ballot, and value in the specification.
- Client request and learner/commit notification in Multi-Paxos deployments.

## Local state

At each acceptor:

- highest promised proposal number `maxBal`;
- highest accepted proposal number `maxVBal`;
- accepted value `maxVal`;
- durable accepted/promise state.

At proposers/leaders:

- unique current proposal number;
- returned Phase 1 promises;
- selected `Quorum1` and `Quorum2` families or concrete members;
- per-slot values/status in Multi-Paxos.

## Normal path

After a leader completes Phase 1 with any valid `Q1`, it proposes each slot's value to a valid `Q2`. Acceptors accept when the proposal number is at least their highest promise. The value is decided after all members of that `Q2` accept. The leader may target only enough acceptors to form the quorum and contact additional acceptors on timeout.

The host's Multi-Paxos command ordering, execution, and client-reply rules are otherwise unchanged.

## Fast path

FPaxos has no Fast Paxos-style leader-bypassing fast round. Its optimization is a smaller or differently shaped normal Phase 2 quorum under a stable leader. The phase count and recovery value-selection rule remain ordinary Paxos.

Disjoint Phase 2 quorums may improve load distribution and steady-state throughput, but competing ballots are still resolved by proposal numbers and cross-phase intersection.

## Slow path

When the stable leader fails or a higher ballot is needed, a proposer runs Phase 1 over a valid `Q1`. If `Q1` is deliberately larger than `Q2`, this recovery/election path may need more acceptors than steady-state replication and may be unavailable even while the old leader could still have committed through `Q2`.

Timeouts or missing replies can cause the proposer to contact additional acceptors, but only sets recognized by the quorum system count as valid.

## Recovery path

A new proposer selects a higher proposal number and gathers promises from a valid Phase 1 quorum `Q1`. It must propose the value attached to the highest accepted proposal returned, or a fresh value if no accepted value is reported. Since `Q1` intersects every valid earlier `Q2`, it sees evidence sufficient to preserve any earlier chosen value.

The base paper assumes fixed `Quorum1` and `Quorum2` families. Section 7 observes that a higher proposer need intersect only Phase 2 quorums actually used at lower proposal numbers, but safely announcing/changing those quorums must be integrated with leader election or Vertical-Paxos-style reconfiguration; the paper leaves that mechanism out of scope.

## Commit rule

A value `v` is decided at proposal number `p` after every acceptor in one valid Phase 2 quorum `Q2 ∈ Quorum2` has accepted `(p, v)`.

```text
Agreed(v, p) ≜ ∃Q ∈ Quorum2: ∀a ∈ Q: Sent2b(a, v, p)
```

No Phase 2 majority is intrinsically required. The quorum family must satisfy cross-phase intersection with every valid Phase 1 quorum.

## Quorum system

General rule:

```text
∀Q1 ∈ Quorum1: Q1 ⊆ A
∀Q2 ∈ Quorum2: Q2 ⊆ A
∀Q1 ∈ Quorum1, ∀Q2 ∈ Quorum2: Q1 ∩ Q2 ≠ ∅
```

No `Q1-Q1` or `Q2-Q2` intersection is required.

For simple, equally weighted threshold quorums over `N` acceptors:

```text
|Q1| + |Q2| > N
minimum |Q1| = N - |Q2| + 1
```

Examples:

- Even `n`: keep `|Q1| = n/2 + 1`, reduce `|Q2|` from `n/2 + 1` to `n/2`.
- Ten acceptors: `|Q2| = 3` is safe with `|Q1| = 8`.
- Grid `N = N1 × N2`: one row of size `N1` can be `Q1`, one column of size `N2` can be `Q2`.
- Extreme `|Q1| = N`, `|Q2| = 1`: any one acceptor can replicate, but leader recovery needs all acceptors.
- Extreme `|Q1| = 1`, `|Q2| = N`: recovery needs one acceptor, but every Phase 2 needs all acceptors.

Quorums can be fixed, threshold-based, grid-based, weighted, or otherwise structured, provided every cross-phase pair intersects.

## Conflict handling

FPaxos decides one value per instance; conflicting concurrent proposals use different proposal numbers. Even if their Phase 2 quorums are disjoint, each Phase 2 quorum intersects the competing Phase 1 quorums. An acceptor that has promised the higher ballot prevents the lower proposal from completing, or reports an earlier accepted value so the higher proposal adopts it.

FPaxos does not use command commutativity or dependencies. Multi-Paxos places commands in separate slots and preserves the host's total-order semantics.

## Safety argument

Theorem 1 states agreement: if `v` is decided at proposal `p` and `v'` is decided at `p'`, then `v = v'`.

The proof establishes the stronger Theorem 2: if `v` is decided at `p`, every later `propose(p', v')` with `p' > p` has `v' = v`. Choose the smallest later `p'` allegedly carrying a different value. Let `Qp,2` be the earlier deciding Phase 2 quorum and `Qp',1` the later Phase 1 quorum. Cross-phase intersection gives an acceptor in both.

- If it sees `prepare(p')` first, it promises at least `p'` and cannot later accept lower `p`, contradicting membership in the deciding `Qp,2`.
- If it accepts `(p, v)` first, its later Phase 1 response reports `v` at proposal `q` with `p ≤ q < p'`. By minimality of `p'`, no higher returned accepted proposal can contain another value, so the later proposer selects `v`.

The appendix TLA+ specification encodes the cross-phase assumption and checks `SafeValue`: once `Agreed(v, b)`, every later `2a` carries `v`.

Source: `raw/fpaxos.pdf`, §5 and Appendix.

## Liveness argument

Like Paxos, FPaxos cannot guarantee progress in a fully asynchronous system. Under the usual eventual synchrony/stable-leader assumptions, progress depends on the current phase:

- steady-state replication requires one available `Q2`;
- leader election/recovery requires one available `Q1` and then one available `Q2`.

For simple quorums with smaller `Q2`, the system guarantees full recovery through `|Q2| - 1` arbitrary failures. With more failures, an existing leader may still replicate if some `Q2` remains, while a new leader cannot be established until a `Q1` becomes available. Structured quorums make failure placement, not only count, relevant.

## Key proof ideas

- Inspect exactly which quorum intersections Paxos's safety proof consumes.
- Separate the Phase 1 recovery quorum family from the Phase 2 decision quorum family.
- Use one cross-phase acceptor either to block a stale proposal or reveal its accepted value.
- Prove the stronger later-proposal invariant before deriving decision agreement.
- Keep same-phase non-intersection legal; do not silently reintroduce majority assumptions.
- Distinguish safety geometry from phase-specific availability and load.
- Treat dynamic quorum selection as reconfiguration metadata, not an uncoordinated local choice.

## Important formulas

Cross-phase intersection:

```text
∀Q1 ∈ Quorum1, ∀Q2 ∈ Quorum2: Q1 ∩ Q2 ≠ ∅
```

Simple threshold quorums:

```text
|Q1| + |Q2| > N
|Q1|min = N - |Q2| + 1
```

Guaranteed arbitrary-failure tolerance for both phases when `Q2` is smaller:

```text
|Q2| - 1 failures
```

Message count when targeting only quorum members:

```text
4N  →  2|Q1| + 2|Q2|
```

Grid construction:

```text
N1 × N2 = N
|Q1| = N1
|Q2| = N2
```

Safety property in the TLA+ specification:

```text
Agreed(v, b) ⇒ NoFutureProposal(v, b)
```

## Relationship to other protocols

FPaxos is strictly more general than ordinary Paxos: choosing intersecting majority families recovers the classic protocol. Unlike [[FastPaxos]], it does not add leaderless fast rounds or require fast-quorum triple intersection. Unlike [[GPaxos]], it still agrees on one value rather than compatible command structures.

[[Atlas]] uses an FPaxos-style small Phase 2 quorum in its dependency protocol's fallback path. [[PigPaxos]] can combine relay aggregation with FPaxos quorum geometry, but its base protocol retains ordinary majority acceptance. The dynamic extension discussed in §7 resembles Vertical Paxos because quorum choice must be safely announced across ballots.

## Limitations

- Smaller Phase 2 quorums can sharply reduce leader-recovery availability.
- Structured quorum availability depends on failure placement and quorum-selection policy.
- Sending only to one selected quorum may require retransmissions and can hurt latency if the chosen acceptors are slow.
- The prototype uses a naive random simple-quorum implementation in a single VM/Mininet environment.
- The proof and TLA+ appendix focus on single-value agreement; Multi-Paxos ordering/execution is inherited rather than re-proved.
- Safe dynamic quorum announcement and reconfiguration are proposed but not specified.
- Byzantine faults are not addressed.

## Open questions

- How should a deployment choose `Quorum1`/`Quorum2` families from topology, correlated failures, and latency distributions?
- How can quorum selection change online with compact, versioned evidence and a complete reconfiguration proof?
- Can [[PigPaxos]] relay selection and FPaxos quorum selection be optimized jointly without conflating overlays and evidence sets?
- How should read quorums intersect flexible write/recovery quorums in protocols such as [[Bodega]]?
- Which flexible geometries best preserve recovery availability under correlated rack or region failures?
- How do flexible quorums interact with fast-path and generalized-consensus recovery obligations?

## Related pages

[[FPaxos]], [[flexible-quorum]], [[quorum]], [[leader]], [[recovery]], [[agreement]], [[liveness]], [[quorum-intersection]], [[protocol-catalog]], [[quorum-systems]], [[commit-rules]], [[recovery-rules]], [[proof-techniques]], [[paxos-family]]
