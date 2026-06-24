---
type: comparison-dimension
dimension: proof techniques
protocols: [FastPaxos, EPaxos, EPaxosStar, SwiftPaxos, Pando]
tags: [proof]
---

# Proof Techniques

| Protocol | Main proof object | Key invariant |
|---|---|---|
| [[FastPaxos]] | Round/value votes | Higher rounds cannot choose incompatible lower possible choices |
| [[EPaxos]] | Instance tuple `(cmd, seq, deps)` | One safe tuple per instance; interfering commands dependency-ordered |
| [[EPaxosStar|EPaxos*]] | Command payload/dependency graph plus recovery validation evidence | Agreement for `(cmd, dep)`; visibility for conflicting committed commands |
| [[SwiftPaxos]] | Accepted dependency paths | Same dependencies for committed command; acyclic committed graph |
| [[Pando]] | Proposal/value/split evidence | Later proposals recover any earlier chosen value |

## Evaluation-sensitive invariants
[[EPaxos-Revisited-2021]] adds a useful modeling distinction for EPaxos: commit safety and execution readiness are separate. A model that only reaches committed states can miss dependency-chain delays and the pruning invariant needed to bound execution latency.

## Recovery-sensitive invariants
[[Making-Democracy-Work-2025]] makes recovery the central proof burden. The useful abstraction is not "recover the largest pre-accepted dependency set"; it is "recover only if validation shows no committed or potentially committing conflicting command would make visibility false." This separates agreement for one command identifier from cross-command visibility.
