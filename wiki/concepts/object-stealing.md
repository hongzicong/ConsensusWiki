# object-stealing

[[object-stealing]] is WPaxos's use of Paxos Phase 1 to transfer leadership of one object to a node nearer its current clients.

The candidate chooses a higher object-specific ballot and gathers a cross-zone `Q1`. Every `Q1` intersects every prior local `Q2`, so the candidate can recover earlier accepted work and stale owners are rejected. After stealing, the new owner repeats local Phase 2 for later slots until another node steals the object.

This is both placement and consensus recovery. It differs from external shard migration because no separate placement master authorizes the move, and it differs from leaderless consensus because only one owner at a time leads a given object's slots.

WPaxos keeps ballots per object to avoid accidentally taking every object from a remote leader. Equal ballot counters are ordered by zone/node ID; randomized backoff mitigates repeated contention. Locality-adaptive mode can delay a steal until a remote zone supplies most requests, reducing oscillation.

## Safety checklist

- The stealing `Q1` intersects every possible prior `Q2`.
- The candidate uses a strictly higher unique per-object ballot.
- All accepted unfinished slots are recovered before new slots open.
- Old owners stop after higher-ballot rejection.
- Reconfiguration cannot let old and new quorum families decide independently.

## Related pages

[[WPaxos]], [[WPaxos-2020]], [[leader]], [[recovery]], [[flexible-quorum]], [[quorum]], [[conflict]], [[paxos-family]]
