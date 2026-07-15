# flexible-quorum

A [[flexible-quorum]] system assigns different quorum families to different protocol phases and requires only the intersections consumed by the safety proof.

For [[FPaxos]], every valid Phase 1 quorum must intersect every valid Phase 2 quorum:

```text
∀Q1 ∈ Quorum1, ∀Q2 ∈ Quorum2: Q1 ∩ Q2 ≠ ∅
```

Phase 1 quorums need not intersect each other, and Phase 2 quorums need not intersect each other. For equally weighted threshold quorums over `N` acceptors:

```text
|Q1| + |Q2| > N
```

Shrinking `Q2` speeds common replication but enlarges `Q1`, making leader recovery less available. Structured families such as rows/columns can improve load and exploit topology, but availability then depends on where failures occur rather than count alone.

[[WPaxos]] specializes this geometry to `Z` zones with `l` nodes each:

```text
|Q1| = (f_n + 1)(Z - f_z)
|Q2| = (l - f_n)(f_z + 1)
```

Every pair overlaps first in at least one zone and then in at least one node. Stable object owners use a nearby `Q2`; [[object-stealing]] uses the wider `Q1`.

## Modeling pitfalls

- Do not assume any two quorums intersect; record each quorum's phase/family.
- Do not treat a small Phase 2 quorum as a Fast Paxos fast quorum.
- Separate steady-state availability (`Q2`) from leader-recovery availability (`Q1`).
- Do not allow uncoordinated dynamic quorum changes; later Phase 1 must know which lower Phase 2 quorums require intersection.
- Distinguish quorum evidence from communication overlays such as [[PigPaxos]] relays.
- For structured quorums, model membership and correlated failure placement, not size only.

## Related pages

[[FPaxos]], [[Flexible-Paxos-2016]], [[WPaxos]], [[WPaxos-2020]], [[object-stealing]], [[quorum]], [[quorum-intersection]], [[leader]], [[recovery]], [[agreement]], [[liveness]]
