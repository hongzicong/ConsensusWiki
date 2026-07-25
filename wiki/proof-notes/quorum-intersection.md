# Quorum Intersection

## Notes
- [[FastPaxos]] needs ordinary quorum intersection plus the fast-round condition involving any quorum and two fast quorums.
- [[EPaxosStar]] separates overall fault tolerance `f` from fast-path fault tolerance `e`; optimized correctness matches `n >= max{2e + f - 1, 2f + 1}` with slow/recovery quorums of size `n - f` and fast quorums of size `n - e`.
- [[Atlas]] uses fast quorums of size `floor(n/2) + f`, slow quorums of size `f + 1`, and recovery quorums of size `n - f`; recovery relies on reconstructing fast dependency unions from at least `floor(n/2)` non-coordinator fast-quorum replies.
- [[SwiftPaxos]] requires any two fast quorums in a ballot to intersect in more than `N/2` replicas.
- [[Pando]] distinguishes discovery (`Phase 1a`) from recovery (`Phase 1b`) intersections; only Phase 1b must recover `k` splits.
- [[Copilot]] intersects its fast quorum with a recovery majority in at least `floor((f + 1) / 2)` replicas, but this minimum evidence is not always sufficient for count-only recovery.
- [[Avicenna]] restricts a normal phase to `f + 2` non-standbys, so a commit set of `f + 1` intersects any two non-standby rotation logs; Armageddon phases revert to majority/majority intersection over `2f + 1` replicas.
- [[Bodega]] uses ordinary majority intersection for three distinct obligations: one stable roster, same-ballot responder coverage, and transfer of older committed writes through grantor catch-up thresholds.
- [[Hermes]] has no data-path quorum-intersection argument: a write/replay uses every member of the current live set. Majority intersection belongs to the external RM update; leases and epochs prevent old/new memberships from serving concurrently.
- [[Jetpack]] intersects a same-view fast superquorum `f + ceil(f/2) + 1` with a recovery majority `f + 1` in at least `ceil(f/2) + 1` replicas; this is both its recovery threshold and the reason cross-view acknowledgements cannot be combined.
- [[FPaxos]] removes all same-phase intersection obligations: only every valid Phase 1 quorum intersecting every valid Phase 2 quorum is required for ordinary Paxos agreement.
- [[Hydra]] does not define one global consensus quorum. Sequencer addition/removal collects one application-defined quorum from every receiver group; [[HydraPaxos]] separately uses `f + 1` consistent replies among `2f + 1` replicas, including its leader.
- [[WPaxos]] builds cross-phase intersection in two levels: Phase 1 and Phase 2 zone sets overlap, then their node subsets inside the common zone overlap.

## Fast Paxos size derivation
Let `N` be the number of acceptors, let classic quorum size be `N - F`, and let fast quorum size be `N - E`. Fast Paxos requires one arbitrary quorum and two fast quorums to have non-empty triple intersection. For a classic quorum and two fast quorums this gives:

```text
(N - F) + 2(N - E) > 2N
N > F + 2E
```

If `N = 2f + 1` and the classic quorum tolerates `f` failures, then `F = f` and:

```text
2f + 1 > f + 2E
E <= floor(f/2)
fast quorum size = N - E >= N - floor(f/2)
```

So a Fast Paxos fast quorum under `N = 2f + 1` has minimum size `N - floor(f/2)`, equivalently:

```text
floor((3f + 1)/2) + 1
= f + floor((f + 1)/2) + 1
```

The second form is exactly one more than the [[EPaxos]] fast quorum size under the same `N = 2f + 1` total-member counting convention.

## EPaxos* lower-bound shape
[[Making-Democracy-Work-2025]] states that any `f`-resilient `e`-fast SMR protocol requires:

```text
n >= max{2e + f - 1, 2f + 1}
```

The optimized [[EPaxosStar]] protocol matches this bound. Its recovery arguments use intersections between recovery quorums of size `n - f`, fast quorums of size `n - e`, and validation evidence about conflicting commands.

## Atlas fast recovery shape
[[Atlas-2020]] sets:

```text
fast quorum size = floor(n/2) + f
slow quorum size = f + 1
recovery quorum size = n - f
```

If a recovery quorum `Q` of size `n - f` intersects a remembered fast quorum `Q0` of size `floor(n/2) + f`, then:

```text
|Q intersect Q0| >= |Q| + |Q0| - n
                 = floor(n/2)
```

When the initial coordinator does not reply to recovery, this intersection supplies at least `floor(n/2)` non-coordinator fast-quorum reports. The fast-path predicate ensures that the original fast dependency union can be reconstructed from such reports.

## Copilot fast-recovery shape
[[Copilot-2020]] uses `n = 2f + 1`, fast quorum size

```text
q_fast = f + floor((f + 1) / 2)
```

and recovery quorum size `q_recovery = f + 1`. Therefore:

```text
|Q_fast intersect Q_recovery|
  >= q_fast + q_recovery - n
   = floor((f + 1) / 2)
```

This lower bound matches the start of Copilot's ambiguous fast-accepted interval `[floor((f + 1) / 2), f)`. In that interval, vote count alone cannot distinguish a fast-committed value from a value blocked by an incompatible entry in the other pilot's log. Recovery must inspect and, if necessary, recover that first possibly incompatible entry. This differs from [[FastPaxos]] triple-intersection recovery even though both protocols use larger-than-majority fast evidence.

## Avicenna restricted-acceptor shape

[[Avicenna-2026]] uses `n = 2f + 1`, but a normal phase permits only `f + 2` non-standbys to accept. If `A` is a committing set and `B` is the set of non-standby logs merged during rotation, then:

```text
|A| = f + 1
|B| = 2
|N_p| = f + 2
|A| + |B| = f + 3 > f + 2 = |N_p|
```

Thus `A intersect B` is non-empty. The next real leader is already a non-standby and contributes its own log, so receiving one additional non-standby `Rotate` supplies the two-log evidence. This is not a one-replica recovery quorum.

In an Armageddon phase, every replica is a non-standby and rotation collects `f + 1` logs:

```text
(f + 1) + (f + 1) = 2f + 2 > 2f + 1
```

The optimization therefore trades normal-phase availability after multiple non-standby crashes for faster takeover, while periodic Armageddon phases restore full-majority recovery.

## Bodega roster/read shape

[[Bodega-2026]] uses odd `n` and `m = ceil(n / 2)`. Any two size-`m` sets intersect. The protocol applies this fact in two evidence domains:

1. Two different rosters cannot each hold `m` active lease grants, because an intersecting grantor cannot actively grant two rosters at the same ballot state.
2. Let `Q_W` be the majority that accepted an older-ballot committed write at slot `x`, and let `E` be any size-`m` subset of a new responder's active lease grantors. Then `Q_W intersect E` contains some grantor `p`. Its `Guard` reports `thresh_p >= x`, so the responder's catch-up condition forces it to commit through `x` before local reads.

For a write committed in the current roster ballot, majority intersection is not enough to ensure every responder saw it. Bodega adds a separate responder-coverage condition: the write cannot commit until every current responder for that key replies.

## Jetpack fast-recovery shape

[[Jetpack-2026]] uses `n = 2f + 1`:

```text
q_fast = f + ceil(f/2) + 1
q_recovery = f + 1
```

Therefore:

```text
|Q_fast intersect Q_recovery|
  >= q_fast + q_recovery - n
   = ceil(f/2) + 1
```

When no accepted recovery set exists, the coordinator includes a command that appears in at least `ceil(f/2) + 1` of the `f + 1` replies. This preserves every command that could have fast-committed in that view.

The view qualifier is essential. If a client were allowed to assemble its superquorum from acknowledgements spanning views, no single view-specific recovery majority would be guaranteed to see the threshold. Jetpack's independent fast-path view and same-view commit rule preserve the intersection argument.

## Flexible Paxos cross-phase shape

[[Flexible-Paxos-2016]] defines quorum families `Quorum1` and `Quorum2` over acceptor set `A`:

```text
∀Q1 ∈ Quorum1: Q1 ⊆ A
∀Q2 ∈ Quorum2: Q2 ⊆ A
∀Q1 ∈ Quorum1, ∀Q2 ∈ Quorum2: Q1 ∩ Q2 ≠ ∅
```

There is no `Q1-Q1` or `Q2-Q2` obligation. For simple threshold quorums this becomes:

```text
|Q1| + |Q2| > N
```

The proof uses a single acceptor in the earlier deciding `Q2` and later recovery `Q1`. If it sees the later prepare first, it blocks the older proposal. If it accepts the older value first, it reports that value in Phase 1 and the later proposer adopts it.

This rule does not replace the stronger intersections required by [[FastPaxos]], [[GPaxos]], or dependency fast paths. Those protocols have additional possible-choice or compatibility obligations beyond ordinary two-phase Paxos.

## OmniPaxos majority-prefix shape

[[Omni-Paxos-2023]] uses ordinary majority intersection but changes the proof object from one value to a log sequence. A new Sequence Paxos leader collects a Prepare majority and adopts the returned log with highest `acceptedRnd`, breaking equal rounds by `logIdx`. Any earlier chosen prefix was accepted by a deciding majority, so at least one report in the new majority carries evidence that prevents that prefix from being removed.

The paper states the adapted P2 condition as:

```text
If a proposal with sequence v is chosen, then every higher-numbered proposal that is chosen has v as a prefix.
```

After the leader adopts that sequence, `AcceptSync` makes each promised follower's log a prefix of the leader's log. FIFO Accept delivery then preserves prefix growth while allowing the implementation to transfer suffixes instead of entire logs.

The liveness topology is a different issue. A single [[partial-connectivity|quorum-connected]] server needs direct links to a majority, but the majority members need not be mutually connected. That condition helps collect a quorum; it does not replace the majority-intersection safety proof.

## TODO
Derive all quorum-size inequalities in a standalone algebra table.

Hydra's receiver-group quorum obligation remains application-parametric. The paper's appendix treats a group as delivering when every receiver in some quorum delivers the message or ordered drop; it does not derive quorum size or intersection from a receiver fault parameter. Do not import HydraPaxos's `f + 1` majority into the generic Hydra groupcast model.

## WPaxos grid shape

With `Z` zones and `l` nodes per zone, WPaxos configures:

```text
|Q1| = (f_n + 1)(Z - f_z)
|Q2| = (l - f_n)(f_z + 1)
```

The selected zone counts overlap because:

```text
(Z - f_z) + (f_z + 1) = Z + 1 > Z
```

Inside any common zone, the selected node counts overlap because:

```text
(f_n + 1) + (l - f_n) = l + 1 > l
```

Therefore every `q1 ∈ Q1` intersects every `q2 ∈ Q2`. This is an [[FPaxos]] cross-phase argument with topology constraints; same-phase quorums need not intersect. Availability is placement-dependent, so a model must retain zone membership rather than representing only the two cardinalities.
