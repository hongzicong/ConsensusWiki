---
type: comparison-dimension
dimension: proof techniques
protocols: [FastPaxos, EPaxos, EPaxosStar, Mencius, Atlas, SwiftPaxos, Pando, Rabia]
tags: [proof]
---

# Proof Techniques

| Protocol | Main proof object | Key invariant |
|---|---|---|
| [[FastPaxos]] | Round/value votes | Higher rounds cannot choose incompatible lower possible choices |
| [[EPaxos]] | Instance tuple `(cmd, seq, deps)` | One safe tuple per instance; interfering commands dependency-ordered |
| [[EPaxosStar]] | Command payload/dependency graph plus recovery validation evidence | Agreement for `(cmd, dep)`; visibility for conflicting committed commands |
| [[Mencius]] | Simple consensus instance plus owner/`no-op` restriction | Only the coordinator can choose non-`no-op`; revocation preserves possible prior outcomes |
| [[PigPaxos]] | Paxos ballots/quorum votes plus relay transport refinement | Relays do not change which unique replica votes count toward a majority |
| [[Atlas]] | Command identifier, dependency union, remembered fast quorum, and Paxos ballot evidence | One `(cmd, dep)` per identifier; conflicting non-`noOp` commits are dependency-visible; recovery reconstructs possible fast proposals |
| [[SwiftPaxos]] | Accepted dependency paths | Same dependencies for committed command; acyclic committed graph |
| [[Pando]] | Proposal/value/split evidence | Later proposals recover any earlier chosen value |
| [[Rabia]] | Weak-MVC phase state and votes | Decisions within a phase agree; once decided, the next phase is value-locked |

## Evaluation-sensitive invariants
[[EPaxos-Revisited-2021]] adds a useful modeling distinction for EPaxos: commit safety and execution readiness are separate. A model that only reaches committed states can miss dependency-chain delays and the pruning invariant needed to bound execution latency.

## Recovery-sensitive invariants
[[Making-Democracy-Work-2025]] makes recovery the central proof burden. The useful abstraction is not "recover the largest pre-accepted dependency set"; it is "recover only if validation shows no committed or potentially committing conflicting command would make visibility false." This separates agreement for one command identifier from cross-command visibility.

[[Atlas-2020]] makes a different recovery tradeoff: the fast-path predicate itself is chosen so the dependency union can be reconstructed later. The recovery coordinator first preserves any accepted consensus proposal; only when no slow-path proposal exists does it reason about the remembered fast quorum and whether the initial coordinator answered recovery.

[[Rabia-2021]] uses randomized consensus to avoid a separate recovery proof. The key proof obligation is value-locking: if one replica decides a binary value, later phases force non-decided replicas to carry the same value until they decide.



## Optimization-sensitive invariants
[[Mencius-2008]] is a useful example of proving an optimized protocol by derivation. First prove the unoptimized sequence of simple consensus instances; then show `SKIP` piggybacking, bounded deferred propagation, and block revocation preserve the same chosen/learned/committed facts.

[[PigPaxos-2021]] is a communication-refinement case: the consensus proof should still talk about Paxos ballots and majority evidence, while the relay layer proves only that aggregated messages faithfully represent unique follower replies or trigger retries.
