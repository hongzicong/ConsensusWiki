# randomized-consensus

[[randomized-consensus]] uses random choices to obtain termination in asynchronous systems where deterministic consensus cannot guarantee progress.

[[Rabia]] uses randomized binary consensus as the core of Weak-MVC. Its liveness claim is probabilistic: each phase has probability at least `1/2` of leading to termination, and each slot terminates with probability 1 when a majority of replicas is non-faulty.

## Related pages
[[Rabia]], [[Rabia-2021]], [[common-coin]], [[liveness]], [[validity]]
