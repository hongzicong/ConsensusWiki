# liveness

Liveness is conditional in asynchronous consensus; each paper states progress under availability and eventual stability assumptions.

[[Atlas]] liveness depends on collecting the required quorum for the path being used. The paper explicitly says that if more than `f` sites are unavailable, Atlas may block until enough sites are reachable, while safety is preserved.

[[PigPaxos]] liveness follows Paxos-style majority availability plus relay timeout/retry behavior. Failed relays can delay an operation, but random relay reselection should eventually find healthy relays when enough nodes are live and message delays satisfy the assumed bound.

[[Rabia]] gives probabilistic liveness: each Weak-MVC phase has probability at least `1/2` of leading to termination, and a slot terminates with probability 1 when a majority of replicas is non-faulty.

## Related pages
[[PigPaxos]], [[Atlas]], [[Rabia]], [[agreement]], [[recovery]], [[quorum]]
