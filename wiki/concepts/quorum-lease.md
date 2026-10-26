# quorum-lease

A [[PQL]] quorum lease `(Q, O)` authorizes replicas in holder set `Q` to read objects `O` locally. The holder set may have **any size**. Activation at an individual holder requires `⌊N/2⌋` live promises from other replicas plus its implicit self-grant, forming a majority of grantors.

Each grantor promises to notify that holder synchronously before committing affected writes and to convey this obligation in Accept and applicable Prepare replies. Any write-accepting majority intersects the active grantor majority, so even a proposer that issued no promise must notify the holder.

Local execution also requires catch-up through the highest accepted instances reported in promises and resolution/execution of interfering accepted writes. Configuration consensus, a live lease, and freshness are separate requirements.

## Timing and failure

Guard/acknowledgement causality makes the holder's usable window lie within the grantor's binding window under the clock-rate assumption. Configuration replacement cannot discard live old obligations. A failed holder delays affected writes until enough promises expire; a restarted replica must wait out old promises.

## Modeling pitfalls

- Do not equate lease holders with grantors or Paxos acceptors.
- Do not make holder-set majority size mandatory merely because the implementation uses it.
- Count implicit self once in activation, and explicitly account for leader/self in write evidence.
- Do not equate a decided lease configuration with an activated, caught-up lease.
- [[roster-lease]] in [[Bodega]] has different metadata and activation rules.

Source: [[Paxos-Quorum-Leases-2014]], §§3.2-3.8, Figures 4-5.

## Related pages

[[PQL]], [[Bodega]], [[roster-lease]], [[quorum]], [[linearizability]], [[recovery]], [[read-freshness]], [[read-strategies]]
