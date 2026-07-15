---
type: comparison-dimension
dimension: proof techniques
protocols: [FastPaxos, FPaxos, OmniPaxos, GPaxos, EPaxos, EPaxosStar, Mencius, Atlas, SwiftPaxos, Pando, Rabia, CURP, Copilot, Avicenna, Bodega, Jetpack, Hydra, HydraPaxos, WPaxos]
tags: [proof]
---

# Proof Techniques

| Protocol | Main proof object | Key invariant |
|---|---|---|
| [[FastPaxos]] | Round/value votes | Higher rounds cannot choose incompatible lower possible choices |
| [[FPaxos]] | Phase-indexed quorum families, persistent promises, and accepted ballot/value | If `v` is decided at `p`, every later proposal carries `v`; one cross-phase acceptor suffices for the induction |
| [[OmniPaxos]] | Whole-log sequences, accepted rounds, majority promises, and QC ballots | Paxos P2/P2c become prefix extension; SC3 induction plus quorum intersection proves SC2; BLE proves LE1-LE3 separately |
| [[GPaxos]] | Ballot array over c-struct votes | If every vote is safe at its ballot, all chosen c-structs are compatible |
| [[EPaxos]] | Instance tuple `(cmd, seq, deps)` | One safe tuple per instance; interfering commands dependency-ordered |
| [[EPaxosStar]] | Command payload/dependency graph plus recovery validation evidence | Agreement for `(cmd, dep)`; visibility for conflicting committed commands |
| [[Mencius]] | Simple consensus instance plus owner/`no-op` restriction | Only the coordinator can choose non-`no-op`; revocation preserves possible prior outcomes |
| [[PigPaxos]] | Paxos ballots/quorum votes plus relay transport refinement | Relays do not change which unique replica votes count toward a majority |
| [[Atlas]] | Command identifier, dependency union, remembered fast quorum, and Paxos ballot evidence | One `(cmd, dep)` per identifier; conflicting non-`noOp` commits are dependency-visible; recovery reconstructs possible fast proposals |
| [[SwiftPaxos]] | Accepted dependency paths | Same dependencies for committed command; acyclic committed graph |
| [[Pando]] | Proposal/value/split evidence | Later proposals recover any earlier chosen value |
| [[Rabia]] | Weak-MVC phase state and votes | Decisions within a phase agree; once decided, the next phase is value-locked |
| [[CURP]] | Backup history plus one selected witness's unordered request set | Any fast-completed unsynced request is in every witness; selected-witness replay is order-independent because records commute |
| [[Copilot]] | Two per-pilot logs, cross-log prefix dependencies, per-entry ballot evidence, and deduplication IDs | Every committed cross-log pair is dependency-oriented in at least one direction; each entry has one recovered value; fixed priority orders cycles |
| [[Avicenna]] | Phase-tagged real-log entries, restricted acceptor sets, and merged rotation logs | A committed command remains at the same index in every later phase; every later real leader preserves the committed prefix |
| [[Bodega]] | Ballot-tagged rosters, directional lease states, responder-covered writes, and per-grantor acceptance thresholds | At most one roster is stable; every local read responder contains every earlier acknowledged write |
| [[Jetpack]] | View-tagged fast logs, host proposer promises, accepted recovery sets, and stability markers | A fast-replied command remains durable and before every later/conflicting command across host view changes |
| [[Hydra]] | Minimum clock frontier plus per-sequencer gap evidence; terminal-drop interpretation of removal | Before any later ordered delivery, each group either delivers the message or an ordered `DROP-NOTIFICATION`, or no group does |
| [[HydraPaxos]] | Composition of Hydra ordered-drop abstraction with NOPaxos majority/deterministic-execution reasoning | Every committed operation has one durable ordered position; every unrecoverable gap becomes one agreed `NO-OP` |
| [[WPaxos]] | Two-level zone/node intersection plus per-object Paxos ballots/logs; TLA+ model checking | No two leaders commit different values for one slot of one object; committed prefixes remain stable across stealing |

## Evaluation-sensitive invariants
[[EPaxos-Revisited-2021]] adds a useful modeling distinction for EPaxos: commit safety and execution readiness are separate. A model that only reaches committed states can miss dependency-chain delays and the pruning invariant needed to bound execution latency.

## Recovery-sensitive invariants
[[Making-Democracy-Work-2025]] makes recovery the central proof burden. The useful abstraction is not "recover the largest pre-accepted dependency set"; it is "recover only if validation shows no committed or potentially committing conflicting command would make visibility false." This separates agreement for one command identifier from cross-command visibility.

[[Atlas-2020]] makes a different recovery tradeoff: the fast-path predicate itself is chosen so the dependency union can be reconstructed later. The recovery coordinator first preserves any accepted consensus proposal; only when no slow-path proposal exists does it reason about the remembered fast quorum and whether the initial coordinator answered recovery.

[[Rabia-2021]] uses randomized consensus to avoid a separate recovery proof. The key proof obligation is value-locking: if one replica decides a binary value, later phases force non-decided replicas to carry the same value until they decide.

[[CURP-2019]] proves recovery by splitting history into a synced backup prefix and an unsynced suffix. The backup restores the synced prefix; one witness recovers completed unsynced operations; commutativity and duplicate suppression make replay safe.

[[Copilot-2020]] separates three proof layers: per-entry agreement inside each pilot log, orientation of every committed pair across logs, and deterministic execution of the combined order. Its ambiguous fast-evidence interval is a reminder that quorum intersection can guarantee evidence exists without making a count-only recovery rule sufficient.

[[Avicenna-2026]] first proves different commands at one index have different phases, then uses commit/merge intersection to preserve a committed command in every later phase. A committed-prefix lemma reduces total-order and real-time-order theorems to in-order execution of the real log. Shadow processing is excluded from the safety proof because shadow entries never execute.

[[Bodega-2026]] proves only the behavior newly introduced beyond classic consensus. A three-case argument classifies each prior write by ballot relative to the reader's stable roster: higher is excluded by lease-majority uniqueness, equal is covered by the responder write quorum, and lower is transferred through majority intersection plus `thresh_p` catch-up. Its TLA+ model separately checks lease-expiration ordering and stable-roster uniqueness.

[[Jetpack-2026]] separates assumptions about the host (`A1-A4`, `PR 1`, `PR 2`) from layer obligations (same-view evidence and a stability marker). The proof handles durability, execution consistency, and linearizability by isolating the nontrivial case: a command replied on the fast path but not yet committed by the host when a view changes.

[[Flexible-Paxos-2016]] proves a stronger later-proposal invariant by choosing the smallest higher ballot allegedly carrying a different value. The common `Q2/Q1` acceptor either makes the older decision impossible or returns the decided value; no same-phase intersection appears in the proof.



## Optimization-sensitive invariants
[[Mencius-2008]] is a useful example of proving an optimized protocol by derivation. First prove the unoptimized sequence of simple consensus instances; then show `SKIP` piggybacking, bounded deferred propagation, and block revocation preserve the same chosen/learned/committed facts.

[[PigPaxos-2021]] is a communication-refinement case: the consensus proof should still talk about Paxos ballots and majority evidence, while the relay layer proves only that aggregated messages faithfully represent unique follower replies or trigger retries.
