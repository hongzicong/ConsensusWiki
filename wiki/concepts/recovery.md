# recovery

[[recovery]] selects a safe value or state after failures, collisions, or ballot changes. It must preserve all values/commands that could already have been committed.

[[EPaxosStar]] is a useful recovery case study because possible fast-path evidence is not enough by itself: the new coordinator validates whether recovering a candidate dependency set would violate visibility with already committed or potentially committing conflicting commands.

[[Atlas]] is another recovery case study: its fast-path predicate is designed so a later recovery quorum can reconstruct any possible fast-path dependency union from the remembered fast quorum.

[[PigPaxos]] uses ordinary Paxos leader-change recovery. Relay failures are transport failures handled by timeout and retry, not a new safe-value selection rule.

[[Rabia]] avoids a separate leader fail-over path. If one replica decides and crashes, the Weak-MVC value-locking argument makes surviving undecided replicas carry the same binary value into later phases.

## Related pages
[[FastPaxos]], [[EPaxos]], [[EPaxosStar]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]]

