# rinse

In [[PQR]], rinse is the execution-confirmation step **after** a quorum read discovers the maximum accepted slot. The client asks any one follower whether that slot has executed and retries if it remains pending.

Quorum discovery observes earlier completed writes through intersection with their accepting quorums, even if followers have not learned their commits. However, its maximum observation may be only partially accepted. Rinse prevents returning provisional data by requiring an executed, chosen prefix before response.

One follower suffices for confirmation because host execution already incorporates global Paxos choice and resolution of prior log gaps. That single confirmation does not replace initial quorum discovery or the host's recovery quorum. Rinse also does not write an observed value back to a majority like conventional read repair.

## Rinse-free case

If first-round quorum evidence has equal accepted/executed progress, the rinse condition is already met. Per-object frontiers can avoid dirty status caused by unrelated keys. This is not a requirement that all replicas return an identical value or identical slot.

## Modeling pitfalls

- Keep accepted, chosen, learned committed, and executed distinct.
- Max accepted may skip gaps; executed prefix cannot.
- Bind the returned data to final execution, especially after a higher ballot overwrites an accepted value. The paper does not give complete payload/pseudocode details for this refinement.
- Repeated rinse is possible; empirical two-RTT reads are not a latency bound.

Source: [[Linearizable-Quorum-Reads-2019]], §§1, 3-4.2, 6.

## Related pages

[[PQR]], [[linearizability]], [[quorum]], [[recovery]], [[read-freshness]], [[read-strategies]]
