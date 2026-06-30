---
type: comparison-dimension
dimension: recovery rules
protocols: [FastPaxos, EPaxos, EPaxosStar, Mencius, PigPaxos, Atlas, SwiftPaxos, Pando, Rabia]
tags: [recovery]
---

# Recovery Rules

## Comparison table
| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[FastPaxos]] | Higher round chooses pickable safe value | Phase 1 evidence from quorum | Preserves possible lower chosen value | Stable coordinator helps | Encode pickable predicate | [[FastPaxos-2006]] |
| [[EPaxos]] | Explicit Prepare / TryPreAccept | Majority evidence | Preserves safe tuple per instance | Timeouts recover failed leaders | Distinguish pre-accepted vs accepted | [[EPaxos-2013]] |
| [[EPaxosStar]] | Recover evidence plus validation of possible fast-path dependencies | Recovery quorum `n - f`; possible fast evidence from at least `Q_size - e` members | Preserves agreement and dependency visibility; invalidating commands force `Nop` | `Waiting` messages break recovery cycles; per-command leader detector eventually stabilizes | Treat validation as evidence gathering, not tentative pre-accept | [[Making-Democracy-Work-2025]] |
| [[Mencius]] | Revocation runs a higher Paxos-style round for suspected coordinator slots; block revocation uses `beta` | Phase 1 evidence from quorum; if no value may have been chosen, propose `no-op` | Preserves any possible prior outcome for a coordinator's slot | Revokes crashed coordinator slots so later instances can commit | Model revocation separately from `SKIP`; false revocation triggers resubmission | [[Mencius-2008]] |
| [[PigPaxos]] | Ordinary Paxos leader takeover with higher ballots; relay failures handled by timeout and retry | Majority Phase 1 evidence for leader change; random relay retries for message delivery | Relay retries do not select safe values, so Paxos recovery invariant is unchanged | Failed relays can delay operations until a retry chooses healthy relays | Separate consensus recovery from transport retry | [[PigPaxos-2021]] |
| [[Atlas]] | MRec recovery over `n - f` replies | Preserve highest accepted consensus proposal; otherwise recover from known fast quorum or choose `noOp` | Preserves either a slow-path decision or a possible fast-path dependency union | Any process may recover a slow/failed initial coordinator if recovery quorum replies | Track whether the initial coordinator replied; that changes the dependency union used | [[Atlas-2020]] |
| [[SwiftPaxos]] | NewLeaderAck plus Sync | Recovery majority | Carries prior accepted dependency paths | Eventually stable ballot | Sync can add/remove deps | [[SwiftPaxos-2024]] |
| [[Pando]] | Phase 1b reconstructs prior value | `k` splits in intersection | Later proposer recovers chosen value | Available 1b and 2 quorums | Value identity and split count | [[Pando-2020]] |
| [[Rabia]] | No separate fail-over; Weak-MVC value-locking advances undecided replicas | Decision needs `f + 1` non-`?` votes; later phases preserve the value | A crashed decider does not strand a hidden decision | Non-faulty majority continues randomized phases | Model recovery as protocol continuation, not leader takeover | [[Rabia-2021]] |


