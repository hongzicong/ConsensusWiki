# roster-lease

A [[roster-lease]] is a timed promise protecting agreement on ballot-tagged cluster metadata rather than protecting one operation. In [[Bodega]], the roster names one leader and arbitrary per-key sets of read responders; the leader is implicitly a responder for every key.

Every node acts as lease grantor and grantee. A responder regards `<bal, ros>` as stable after holding active leases from a majority:

```text
|{}renewBy| ⩾ m
```

Majority intersection implies that at most one roster can be stable. This alone does not prove that a newly enabled responder has the latest committed data, so each `Guard` carries the grantor's highest-ever-accepted slot `thresh_p`. Before local reads, the responder must have committed through every threshold in some majority subset of its active grantors.

Roster leases use the standard guard/renew pattern under bounded clock drift. The grantor's expiration is never earlier than the grantee's. A higher-ballot roster first revokes old leases or waits for expiration, then starts guarded leases for the new metadata. Renewals and replies can be piggybacked on heartbeats because roster changes are infrequent.

## Safety relevance

- Lease-majority intersection gives roster uniqueness.
- The injective ballot/roster mapping prevents two different rosters at one ballot.
- Threshold catch-up transfers visibility of older-ballot commits to a new responder.
- Responder-covering write quorums transfer visibility of same-ballot commits.
- Expiration permits failed responders to be safely removed without an external oracle.

## Modeling pitfalls

- Do not model a roster lease as a lease forbidding writes; writes continue while the roster is stable.
- Do not omit `thresh_p`; majority leases alone do not prove a joining responder is caught up.
- Separate grantor and grantee expiration times and preserve their ordering invariant.
- Distinguish stable roster evidence from write commit evidence.
- Model per-key responders even if an implementation compresses them into key ranges.

## Related pages

[[Bodega]], [[Bodega-2026]], [[quorum]], [[leader]], [[recovery]], [[linearizability]], [[liveness]]
