# common-coin

A [[common-coin]] gives replicas the same random bit for a phase. It is useful in randomized consensus because replicas can break ties consistently without electing a leader.

[[Rabia]] uses `CommonCoinFlip(p)` when a Weak-MVC phase observes only `?` votes. The implementation described in the paper uses a shared seed; the paper notes this assumes faults and message delays are independent of the coin outcome.

## Related pages
[[Rabia]], [[Rabia-2021]], [[randomized-consensus]], [[liveness]]
