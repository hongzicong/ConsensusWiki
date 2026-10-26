---
type: comparison-dimension
dimension: linearizable read strategies
protocols: [PQL, PQR, Bodega, Hermes]
tags: [read-optimization, linearizability, lease, quorum-read]
---

# Linearizable Read Strategies

## What this dimension means

How a replica or client establishes that a read includes every earlier completed interfering write, and that its returned value is stable rather than merely speculative.

## Why it matters

Local state, a lease, majority contact, and execution confirmation prove different things. Read optimization moves coordination between writes, reads, and background authority management; it does not remove the need for freshness evidence.

## Comparison table

| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[PQL]] | One-holder read after majority promise activation and accepted-frontier catch-up | Safe lease clocks; atomic read with leases for all objects | Every write quorum learns holder-notification obligations | Failed holder blocks affected writes until obligations expire | Holder set may be any size; grantor evidence is a majority | [[Paxos-Quorum-Leases-2014]], §§3.2-3.9 |
| [[PQR]] | Read quorum discovers max accepted slot, then one follower confirms execution; clean first round skips rinse | Read intersects all write quorums; host execution certifies chosen prefix | Discovery prevents staleness; rinse prevents unstable observations | Read waits for quorum and selected-prefix execution; writes unchanged | Majority counts total membership; excluding leader may weaken read availability | [[Linearizable-Quorum-Reads-2019]], §§4-4.2 |
| [[Bodega]] | Local read at stable roster responder; hold or use early accept evidence during an in-flight write | Majority roster leases, drift bounds, threshold catch-up | Old writes covered by thresholds; current writes cover responders | All responders lie on relevant write path; roster can remove failed ones | Lease protects metadata; read release differs from PQL | [[Bodega-2026]], §§3-4.3 |
| [[Hermes]] | Local read only while key is `Valid`; writes invalidate all live members first | Leased membership epoch; timestamp and validation rules | Every authorized reader is updated or invalid before write completion | Writes wait for all live members; failures require membership change | Later `VAL` enables readability only for the matching timestamp | [[Hermes-2020]], §§3.1-3.4 |

## Main patterns

PQL, Bodega, and Hermes make the write path cover authorized readers, allowing later single-replica reads. PQR instead leaves the write path alone and performs fresh quorum discovery on the read path. Background authority still matters for the lease/membership designs.

## Important exceptions

PQR's single-replica rinse is sufficient only after an intersecting discovery quorum; it is not a standalone local-read certificate. PQL holder sets can be smaller than a majority even though each active holder needs majority grantor evidence.

## Common pitfalls

- Reading only learned commits from a majority can miss completed writes whose commit notifications are delayed.
- Reading highest accepted data can return an unchosen update; intersection alone does not stabilize it.
- A lease does not establish catch-up by itself.
- A write's chosen state, replica execution, and client read response are distinct events.
- Lease-based and lease-free read mechanisms have different availability costs.

## Relevance to new protocol design

Choose where to pay for freshness: reader coverage on writes, per-read discovery, or background authority plus catch-up. Then separately specify result stability and state application. This is synthesis from the cited mechanisms, not a new proved composition.

## Open questions

- Can a system switch safely between quorum reads and local leases during a configuration transition?
- What placement policy balances read fanout, responder-covering write latency, and failed-reader availability?
- How should per-key read frontiers compose into multi-key snapshots?

## Related pages

[[protocol-catalog]], [[quorum-systems]], [[commit-rules]], [[fast-paths]], [[recovery-rules]], [[timing-assumptions]], [[linearizability]], [[quorum-lease]], [[rinse]], [[roster-lease]], [[invalidation]], [[read-freshness]]
