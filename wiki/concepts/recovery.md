# recovery

[[recovery]] selects a safe value or state after failures, collisions, or ballot changes. It must preserve all values/commands that could already have been committed.

[[EPaxosStar]] is a useful recovery case study because possible fast-path evidence is not enough by itself: the new coordinator validates whether recovering a candidate dependency set would violate visibility with already committed or potentially committing conflicting commands.

[[Atlas]] is another recovery case study: its fast-path predicate is designed so a later recovery quorum can reconstruct any possible fast-path dependency union from the remembered fast quorum.

[[PigPaxos]] uses ordinary Paxos leader-change recovery. Relay failures are transport failures handled by timeout and retry, not a new safe-value selection rule.

[[GPaxos]] recovery starts a higher ballot and computes `ProvedSafe(Q, m, beta)` from phase 1b evidence. The selected c-struct must extend lower-ballot values that could still be chosen.

[[Rabia]] avoids a separate leader fail-over path. If one replica decides and crashes, the Weak-MVC value-locking argument makes surviving undecided replicas carry the same binary value into later phases.

[[CURP]] recovers by restoring an ordered backup prefix, then replaying requests from one selected witness. The single-witness rule matters: different witnesses may have accepted different request sets, but each individual witness set is mutually commutative.

## Related pages
[[FastPaxos]], [[GPaxos]], [[EPaxos]], [[EPaxosStar]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[CURP]]

