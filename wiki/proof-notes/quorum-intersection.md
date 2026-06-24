# Quorum Intersection

## Notes
- [[FastPaxos]] needs ordinary quorum intersection plus the fast-round condition involving any quorum and two fast quorums.
- [[EPaxosStar|EPaxos*]] separates overall fault tolerance `f` from fast-path fault tolerance `e`; optimized correctness matches `n >= max{2e + f - 1, 2f + 1}` with slow/recovery quorums of size `n - f` and fast quorums of size `n - e`.
- [[SwiftPaxos]] requires any two fast quorums in a ballot to intersect in more than `N/2` replicas.
- [[Pando]] distinguishes discovery (`Phase 1a`) from recovery (`Phase 1b`) intersections; only Phase 1b must recover `k` splits.

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

The optimized [[EPaxosStar|EPaxos*]] protocol matches this bound. Its recovery arguments use intersections between recovery quorums of size `n - f`, fast quorums of size `n - e`, and validation evidence about conflicting commands.

## TODO
Derive all quorum-size inequalities in a standalone algebra table.
