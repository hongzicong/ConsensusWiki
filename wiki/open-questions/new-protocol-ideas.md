---
type: open-questions
tags: [brainstorm, fast-path, smr, consensus, protocol-design]
status: speculative
---

# New Protocol Ideas

This page collects speculative SMR/consensus protocol candidates. These are not paper claims. They are design sketches grounded in [[EPaxos]], [[EPaxosStar]], [[SwiftPaxos]], [[FastPaxos]], [[Mencius]], [[Pando]], [[fast-path]], [[quorum]], [[recovery]], and [[quorum-intersection]].

## Shared Constraints

- Target: 1-RTT fast commit from command propagation to commit evidence, in the same sense as [[EPaxos]]/[[EPaxosStar]] and [[SwiftPaxos]] fast paths.
- Fast quorum budget: no larger than the EPaxos-style `n = 2f + 1`, `e = ceil((f + 1)/2)`, fast quorum `n - e = f + floor((f + 1)/2)` or SwiftPaxos C1/C2 fast-quorum envelope already recorded in [[quorum-systems]].
- Novelty should come from metadata, routing, conflict prediction, recovery evidence, dependency structure, or execution rules, not from increasing latency or simply requiring larger fast quorums.
- Safety obligations remain TODO unless explicitly derived: agreement for each command/instance, visibility/order for conflicting commands, acyclic or SCC-safe execution, and recoverability from slow/recovery quorums.

## Candidate Ideas

1. **Per-key fast anchors**
   - Core mechanism: Assign each key or conflict class a lightweight SwiftPaxos-style anchor included in the relevant fast quorum; unrelated classes use different anchors.
   - Expected advantage: Keeps 1-RTT commit while spreading leader-like coordination across hot-key partitions.
   - Main risk: Cross-key transactions may need multiple anchors and could create dependency cycles.
   - Closest related protocols: [[SwiftPaxos]], [[EPaxos]].

2. **Rotating dependency anchor**
   - Core mechanism: Rotate the leader-in-fast-quorum anchor by command identifier while preserving a deterministic recovery owner for each identifier.
   - Expected advantage: Avoids a fixed WAN leader bottleneck while retaining SwiftPaxos-style anchored evidence.
   - Main risk: Recovery must prove that anchor changes cannot hide conflicting dependencies.
   - Closest related protocols: [[SwiftPaxos]], [[Mencius]].

3. **Conflict-class command leaders**
   - Core mechanism: Command leaders are chosen by conflict-class hash, not by submitting replica; the fast quorum stays EPaxos-sized.
   - Expected advantage: Commands likely to conflict are observed in a consistent local order.
   - Main risk: Misclassified commands can violate the assumptions behind the fast-path metadata.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

4. **Lease-free locality anchors**
   - Core mechanism: Use deterministic region-preferred anchors without timing leases; all safety evidence is quorum based.
   - Expected advantage: Improves common WAN latency for local clients without making safety depend on clocks.
   - Main risk: Region skew may overload one anchor and increase slow-path frequency.
   - Closest related protocols: [[SwiftPaxos]], [[EPaxosStar]].

5. **Client-chosen anchor hints**
   - Core mechanism: Clients include an anchor hint derived from recent conflict observations; replicas verify the hint before using the 1-RTT path.
   - Expected advantage: Lets clients steer hot commands toward the best fast-path coordinator without protocol-level reconfiguration.
   - Main risk: Bad hints may amplify conflicts or create denial-of-service-like hotspots.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

6. **Two-dimensional anchors**
   - Core mechanism: Anchor fast evidence by both key range and operation type, such as read-modify-write versus blind write.
   - Expected advantage: Reduces unnecessary dependencies among operations that share keys but commute.
   - Main risk: The commutativity classifier becomes part of the safety-critical trusted logic.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

7. **Dependency micro-leaders**
   - Core mechanism: A command leader delegates only dependency computation to a micro-leader, while commit evidence still comes from a normal fast quorum.
   - Expected advantage: Separates metadata coordination from full command leadership.
   - Main risk: Need recovery rules for cases where command and dependency leaders disagree or crash.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

8. **Witnessed command leaders**
   - Core mechanism: Each command leader attaches a small witness set of recent conflicting commands; fast-quorum members verify only the witness boundary.
   - Expected advantage: May reduce dependency divergence without increasing quorum size.
   - Main risk: A witness boundary that is too small may omit a conflicting command needed for visibility.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

9. **Leaderless anchored recovery**
   - Core mechanism: Fast path stays EPaxos-style leaderless, but every fast certificate records a deterministic recovery anchor.
   - Expected advantage: Keeps distributed fast submission while simplifying failed-command recovery.
   - Main risk: The recovery anchor may not have seen enough fast evidence to validate safely.
   - Closest related protocols: [[EPaxos]], [[EPaxosStar]], [[SwiftPaxos]].

10. **Quorum-role specialization**
   - Core mechanism: Within an EPaxos-sized fast quorum, some replicas specialize as dependency observers and others as recovery witnesses, but all vote once.
   - Expected advantage: Adds structure to evidence without changing quorum cardinality.
   - Main risk: Role imbalance may weaken intersections unless the proof tracks role membership exactly.
   - Closest related protocols: [[EPaxosStar]], [[Pando]].

11. **Prefix-compressed dependencies**
   - Core mechanism: Fast acknowledgements carry a compact dependency frontier per conflict class instead of explicit dependency sets.
   - Expected advantage: Reduces message size and makes matching fast acknowledgements more likely.
   - Main risk: Frontier compression must be injective enough for recovery to reconstruct omitted commands.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

12. **Version-vector dependency path**
   - Core mechanism: Replace direct dependency sets with per-class version vectors that imply dependency paths.
   - Expected advantage: Gives SwiftPaxos-like path evidence with fixed-size metadata for bounded classes.
   - Main risk: Dynamic key sets and cross-class commands may make vectors large or ambiguous.
   - Closest related protocols: [[SwiftPaxos]], [[EPaxos]].

13. **Bloom-filter dependency precheck**
   - Core mechanism: Replicas include Bloom filters for recent conflicting commands; false positives add dependencies, never remove them.
   - Expected advantage: Preserves safety while cheaply detecting likely conflicts in 1 RTT.
   - Main risk: False positives may create dependency chains and hurt execution latency.
   - Closest related protocols: [[EPaxos]], [[EPaxos-Revisited-2021]].

14. **Merkle dependency certificates**
   - Core mechanism: Fast acknowledgements commit to a Merkle root of observed dependencies plus proofs for conflicts queried during recovery.
   - Expected advantage: Keeps fast messages small while giving recovery auditable evidence.
   - Main risk: Recovery may become expensive or block on missing proofs.
   - Closest related protocols: [[EPaxosStar]], [[Pando]].

15. **Conflict-frontier intervals**
   - Core mechanism: Track dependencies as intervals over per-key logical clocks rather than command identifiers.
   - Expected advantage: Compresses dense hot-key workloads where many commands conflict.
   - Main risk: Interval gaps can hide individual commands unless clocks are quorum-certified.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

16. **Typed dependency edges**
   - Core mechanism: Edge labels distinguish read-after-write, write-after-read, and non-commuting write dependencies.
   - Expected advantage: Execution can ignore edges that are irrelevant for a command's return value or state transition.
   - Main risk: Incorrect edge weakening can break linearizability.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

17. **Return-value dependency split**
   - Core mechanism: Commit one dependency set for state safety and a stricter read-dependency set only when return values require it.
   - Expected advantage: Recovers some early-return benefits without treating all commands identically.
   - Main risk: Must avoid the linearizability pitfalls noted for EPaxos* early-return optimizations.
   - Closest related protocols: [[EPaxosStar]], [[EPaxos]].

18. **Dependency age caps**
   - Core mechanism: Fast acknowledgements include dependencies only above a quorum-certified stable frontier; older committed commands are summarized.
   - Expected advantage: Prevents dependency metadata from growing with long-running systems.
   - Main risk: Stable-frontier certificates may add background complexity and liveness coupling.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

19. **Negative dependency evidence**
   - Core mechanism: Replicas explicitly certify "no known conflicting command after frontier X" in fast acknowledgements.
   - Expected advantage: Helps recovery distinguish missing evidence from genuine absence.
   - Main risk: Negative claims are hard to preserve under asynchronous message reordering.
   - Closest related protocols: [[EPaxosStar]], [[FastPaxos]].

20. **Conflict-oblivious fast value plus repair edge**
   - Core mechanism: Commit a command fast with minimal dependencies, then require later conflicting commands to attach repair edges if they observed it.
   - Expected advantage: Makes uncontended fast path very small.
   - Main risk: If two conflicting commands commit without either observing the other, visibility fails.
   - Closest related protocols: [[EPaxosStar]], [[FastPaxos]].

21. **Validated fast quorum certificates**
   - Core mechanism: Replicas fast-ack only after locally checking the EPaxos* validation predicate against already committed conflicts.
   - Expected advantage: Moves some recovery validation into the fast path without extra round trips.
   - Main risk: Pre-commit validation cannot see concurrently committing invalidators.
   - Closest related protocols: [[EPaxosStar]], [[EPaxos]].

22. **Deferred validation tokens**
   - Core mechanism: Fast acknowledgements include tokens promising to validate the same dependency set during later recovery.
   - Expected advantage: Recovery can identify which replicas are obligated to support the fast certificate.
   - Main risk: Crashed token holders may be exactly the missing evidence.
   - Closest related protocols: [[EPaxosStar]], [[FastPaxos]].

23. **Nop-biased recovery**
   - Core mechanism: Fast path is unchanged, but recovery aggressively chooses `Nop` when validation evidence is ambiguous and resubmits payloads.
   - Expected advantage: Simplifies safety proof and may avoid recovery cycles.
   - Main risk: Liveness and client-visible latency suffer under frequent false `Nop` recovery.
   - Closest related protocols: [[EPaxosStar]].

24. **Dependency-preserving Nop**
   - Core mechanism: A recovered `Nop` still carries dependency metadata needed to preserve visibility.
   - Expected advantage: Avoids losing ordering information when payload recovery is unsafe.
   - Main risk: Need prove a metadata-only command cannot introduce execution anomalies.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

25. **Recovery by conflict witnesses**
   - Core mechanism: Recovery quorums collect compact witness summaries for only commands that conflict with the recovering command.
   - Expected advantage: Narrows recovery cost to the relevant conflict neighborhood.
   - Main risk: Witness selection must be complete despite asynchronous dissemination.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

26. **Recoverable dependency roots**
   - Core mechanism: Fast certificates commit to a root hash whose leaves are held by the fast-quorum replicas and reconstructible from a recovery quorum.
   - Expected advantage: Combines small fast messages with Pando-like recoverability reasoning.
   - Main risk: Hash roots do not by themselves prove semantic conflict coverage.
   - Closest related protocols: [[Pando]], [[EPaxosStar]].

27. **Two-ballot fast preaccept**
   - Core mechanism: Keep 1-RTT fast commit, but every preaccept records separate join ballot and accepted ballot fields from the start.
   - Expected advantage: Avoids EPaxos-style ballot ambiguity in recovery.
   - Main risk: Extra state may not help unless recovery rules use it cleanly.
   - Closest related protocols: [[EPaxosStar]], [[EPaxos]].

28. **Fast certificate escrow**
   - Core mechanism: Replicas store fast certificates for peers in a rolling escrow set selected by hash.
   - Expected advantage: Recovery can find evidence even if several original fast-quorum members crash.
   - Main risk: Escrow dissemination must not add latency to the fast commit.
   - Closest related protocols: [[EPaxosStar]], [[Pando]].

29. **Recovery-prioritized fast quorum**
   - Core mechanism: Prefer fast quorums whose members maximize intersection with likely recovery majorities, without increasing size.
   - Expected advantage: Improves practical recoverability after regional failures.
   - Main risk: Quorum preference can create correlated failure exposure.
   - Closest related protocols: [[SwiftPaxos]], [[EPaxosStar]].

30. **Conflict-cycle recovery breaker**
   - Core mechanism: Recovery detects cycles of mutually waiting conflicting commands and chooses a deterministic victim for `Nop` or slow accept.
   - Expected advantage: Makes liveness behavior explicit under dependency recovery cycles.
   - Main risk: Victim choice must preserve already possible fast commits.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

31. **Acyclic fast dependency rule**
   - Core mechanism: Replicas fast-ack only if adding the command's dependencies preserves a locally known acyclic graph.
   - Expected advantage: Reduces SwiftPaxos-style cycle repair after commit.
   - Main risk: Local acyclicity does not imply global acyclicity under partial knowledge.
   - Closest related protocols: [[SwiftPaxos]], [[EPaxos]].

32. **SCC-first execution protocol**
   - Core mechanism: Commit dependencies fast as in EPaxos*, but design metadata for rapid SCC closure and execution.
   - Expected advantage: Targets execution latency, not just commit latency.
   - Main risk: Fast commit may remain 1 RTT while execution is delayed by missing SCC members.
   - Closest related protocols: [[EPaxosStar]], [[EPaxos-Revisited-2021]].

33. **Acyclic path certificates**
   - Core mechanism: Fast acknowledgements include a path certificate proving the proposed dependencies do not close a known cycle.
   - Expected advantage: Gives recovery stronger evidence about executable order.
   - Main risk: Path certificates may grow with conflict graph depth.
   - Closest related protocols: [[SwiftPaxos]].

34. **Deterministic cycle orientation**
   - Core mechanism: Conflicting concurrent commands always orient dependency edges by a deterministic rank unless one command was already committed.
   - Expected advantage: Makes matching dependency metadata more likely.
   - Main risk: Rank orientation can increase dependency chains for hot commands.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

35. **Two-level dependency graph**
   - Core mechanism: Fast path commits class-level edges first and command-level edges only inside hot classes.
   - Expected advantage: Shrinks metadata for mostly independent workloads.
   - Main risk: Coarse class edges can serialize too much work.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

36. **Commutativity lattice protocol**
   - Core mechanism: Model operations in a lattice of commutativity classes and require dependencies only at the lowest non-commuting join.
   - Expected advantage: Captures more concurrency than binary conflict relations.
   - Main risk: Application-specific lattice errors become safety bugs.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

37. **Read-stability fast path**
   - Core mechanism: Reads fast-commit with a dependency frontier proving all writes they observe are committed or dependency-ordered.
   - Expected advantage: Provides low-latency linearizable reads without a separate read lease.
   - Main risk: Read return values require careful relation to command execution, not only commit.
   - Closest related protocols: [[EPaxos]], [[Pando]].

38. **Inverse-dependency acknowledgements**
   - Core mechanism: Replicas can fast-ack by saying "this command is before these known conflicts" instead of only depending on prior conflicts.
   - Expected advantage: Lets concurrent commands agree on order even when observed in opposite directions.
   - Main risk: Recovery must merge before/after constraints without cycles.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

39. **Partial-order fast value**
   - Core mechanism: The agreed value is a set of ordering constraints rather than a dependency set for one command.
   - Expected advantage: May unify several concurrent commands in one 1-RTT certificate.
   - Main risk: Per-command agreement becomes harder to define and recover.
   - Closest related protocols: [[SwiftPaxos]], [[FastPaxos]].

40. **Conflict-window batching**
   - Core mechanism: Fast acknowledgements may contain a small ordered batch of mutually conflicting commands seen in the same window.
   - Expected advantage: Converts collisions into a deterministic mini-order without extra latency.
   - Main risk: Larger batch metadata may delay wide-area transmission and hurt tail latency.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

41. **Thrifty fast quorum with recovery padding**
   - Core mechanism: Use a minimal EPaxos-style fast quorum for commit, but send asynchronous padding evidence to extra replicas after commit.
   - Expected advantage: Keeps 1-RTT latency while improving later recovery.
   - Main risk: Recovery immediately after commit may occur before padding arrives.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

42. **Topology-aware fast quorum selection**
   - Core mechanism: Choose EPaxos-sized fast quorums by WAN latency while satisfying fixed intersection families.
   - Expected advantage: Reduces common-case latency without changing quorum size.
   - Main risk: Dynamic latency adaptation can accidentally violate quorum-family assumptions.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

43. **Fixed-majority fast island**
   - Core mechanism: Use SwiftPaxos C2-like fixed majority fast quorums per shard, with different shards assigned to different majorities.
   - Expected advantage: Achieves small fast quorums and predictable intersections.
   - Main risk: A shard's fast availability depends on its fixed majority.
   - Closest related protocols: [[SwiftPaxos]], [[Mencius]].

44. **Flexible EPaxos-sized quorums**
   - Core mechanism: Allow different fast-quorum families per conflict class while keeping each quorum no larger than EPaxos' fast quorum.
   - Expected advantage: Tailors intersections to actual conflict topology.
   - Main risk: Cross-class commands need intersection across multiple families.
   - Closest related protocols: [[EPaxos]], [[quorum-intersection]].

45. **Fast quorum colorings**
   - Core mechanism: Color replicas and require each fast quorum to contain a fixed color profile rather than a larger cardinality.
   - Expected advantage: Encodes geographical diversity and recovery intersection structurally.
   - Main risk: Color-profile intersections may be insufficient for arbitrary failures.
   - Closest related protocols: [[SwiftPaxos]], [[FastPaxos]].

46. **Hotspot adaptive quorum family**
   - Core mechanism: Hot conflict classes switch to fixed fast quorum families; cold classes use flexible leaderless quorums.
   - Expected advantage: Reduces dependency disagreement where conflicts are frequent.
   - Main risk: Switching rules must not leave ambiguous certificates during transitions.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

47. **Recovery-aware quorum rotation**
   - Core mechanism: Rotate fast quorum membership only at epochs whose boundary is committed by a slow quorum.
   - Expected advantage: Allows latency optimization while keeping recovery proof epoch-local.
   - Main risk: Epoch changes can become a hidden slow-path bottleneck.
   - Closest related protocols: [[SwiftPaxos]], [[Mencius]].

48. **Client-proximal quorum choice**
   - Core mechanism: For each command, pick the nearest valid fast quorum from a precomputed intersection-safe family.
   - Expected advantage: Reduces client-perceived latency while preserving 1-RTT commit.
   - Main risk: Workload concentration may make the nearest family overloaded.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

49. **Quorum certificates with role bits**
   - Core mechanism: Fast certificates record which members served as leader/anchor, dependency observer, and recovery witness.
   - Expected advantage: Lets recovery reason about evidence quality without larger quorums.
   - Main risk: The proof must show role intersections, not just set intersections.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

50. **Availability-tiered fast quorums**
   - Core mechanism: Prefer highly available replicas in fast quorums but require enough low-latency diversity for intersection.
   - Expected advantage: Improves practical fast-path success under partial outages.
   - Main risk: Availability scoring can become stale and concentrate risk.
   - Closest related protocols: [[EPaxosStar]], [[Pando]].

51. **Speculative read-through commit**
   - Core mechanism: A client executes after collecting a fast certificate and dependency-ready proof from the same quorum.
   - Expected advantage: Avoids waiting for a separate commit broadcast on read-heavy workloads.
   - Main risk: Dependency-ready proof may be too strong to obtain in the same 1 RTT.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

52. **Fast certificate forwarding**
   - Core mechanism: Fast-quorum members forward matching certificates directly to dependent commands' leaders.
   - Expected advantage: Shortens execution waiting caused by missing transitive commits.
   - Main risk: Forwarding is off the critical path but may create message storms.
   - Closest related protocols: [[EPaxos-Revisited-2021]], [[SwiftPaxos]].

53. **Execution-frontier acknowledgements**
   - Core mechanism: Fast acknowledgements include each replica's committed/executable frontier for relevant conflict classes.
   - Expected advantage: Command leaders can predict execution readiness before returning.
   - Main risk: Frontier freshness is not guaranteed in an asynchronous system.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

54. **Dependency prefetch protocol**
   - Core mechanism: On fast commit, replicas proactively fetch dependency certificates in parallel with commit dissemination.
   - Expected advantage: Maintains 1-RTT commit while reducing post-commit execution stalls.
   - Main risk: Prefetching may waste bandwidth on commands that later execute through slow SCCs.
   - Closest related protocols: [[EPaxos-Revisited-2021]], [[SwiftPaxos]].

55. **Client-visible commit/execution split**
   - Core mechanism: Return 1-RTT fast commit with explicit "not yet executed" status for commands whose dependencies are unresolved.
   - Expected advantage: Makes latency semantics honest for applications that can tolerate deferred execution.
   - Main risk: Many SMR clients need return values, so this may be a narrow API.
   - Closest related protocols: [[EPaxos]], [[EPaxosStar]].

56. **Fast commutative apply**
   - Core mechanism: Replicas immediately apply commands proven to commute with all unresolved dependencies.
   - Expected advantage: Reduces execution delay without changing consensus evidence.
   - Main risk: Requires a sound application-level commutativity proof.
   - Closest related protocols: [[Mencius]], [[EPaxos]].

57. **SCC hint certificates**
   - Core mechanism: Fast acknowledgements include hints about likely SCC membership derived from local dependency graphs.
   - Expected advantage: Speeds deterministic SCC execution after commit.
   - Main risk: Hints must remain non-binding unless certified.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

58. **Return-value escrow**
   - Core mechanism: For deterministic operations, replicas include tentative return values plus dependency evidence in fast acknowledgements.
   - Expected advantage: Clients can return after 1 RTT when all tentative return values match.
   - Main risk: Matching return values may mask different internal dependency justifications.
   - Closest related protocols: [[EPaxos]], [[Pando]].

59. **Read-only dependency bypass**
   - Core mechanism: Read-only commands bypass dependency insertion if a fast quorum reports the same stable write frontier.
   - Expected advantage: 1-RTT linearizable reads under stable workloads.
   - Main risk: Concurrent writes between frontier observations can invalidate the read.
   - Closest related protocols: [[Pando]], [[EPaxos]].

60. **Fast irreversible prefix**
   - Core mechanism: Replicas maintain a quorum-certified irreversible prefix per conflict class; commands below it need no explicit dependencies.
   - Expected advantage: Cuts dependency size and execution wait.
   - Main risk: Prefix certification may lag under high conflict.
   - Closest related protocols: [[Mencius]], [[EPaxos]].

61. **ML-guided dependency proposal**
   - Core mechanism: A predictor suggests likely dependencies; replicas verify and can only add safety-preserving edges.
   - Expected advantage: Improves fast acknowledgement matching for recurrent workloads.
   - Main risk: Bad predictions may add excessive dependencies and harm tail latency.
   - Closest related protocols: [[EPaxos]], [[EPaxos-Revisited-2021]].

62. **Conflict-rate adaptive metadata**
   - Core mechanism: Switch between explicit dependencies, frontiers, and batches based on observed conflict rate, without changing quorum size.
   - Expected advantage: Matches metadata form to workload regime.
   - Main risk: Mode changes need careful recovery compatibility.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

63. **Hot-key serialization hint**
   - Core mechanism: For keys above a conflict threshold, replicas orient fast dependencies according to a shared hot-key sequence.
   - Expected advantage: Reduces slow-path fallback on hot spots while preserving 1-RTT commit.
   - Main risk: Hot-key sequencer may become a bottleneck or liveness dependency.
   - Closest related protocols: [[Mencius]], [[EPaxos]].

64. **Cold-key leaderless scatter**
   - Core mechanism: Cold commands use EPaxos-style any-replica leadership with minimal dependency metadata.
   - Expected advantage: Preserves egalitarian load distribution for low-conflict workloads.
   - Main risk: Need smooth integration with hot-key modes.
   - Closest related protocols: [[EPaxos]], [[EPaxosStar]].

65. **Dynamic commutativity profiles**
   - Core mechanism: Applications publish versioned commutativity profiles; fast acknowledgements include the profile version used.
   - Expected advantage: Enables richer conflict avoidance without protocol rewrites.
   - Main risk: Profile upgrades become safety-critical reconfiguration events.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

66. **Conflict sketch gossip**
   - Core mechanism: Replicas continuously gossip compact conflict sketches so fast-path dependency proposals are more likely to match.
   - Expected advantage: Improves 1-RTT success without adding messages to the critical path.
   - Main risk: Stale sketches may add dependencies or mislead predictors.
   - Closest related protocols: [[EPaxos-Revisited-2021]], [[SwiftPaxos]].

67. **Topology-sensitive batching**
   - Core mechanism: Batch only commands whose client-to-quorum latency profiles are similar and whose conflicts can be ordered deterministically.
   - Expected advantage: Avoids batching-induced tail penalties noted for EPaxos-like systems.
   - Main risk: Batch formation logic can add delay before the 1-RTT path begins.
   - Closest related protocols: [[EPaxos-Revisited-2021]], [[Mencius]].

68. **Fast-path admission control**
   - Core mechanism: Replicas reject or slow-path commands predicted to cause dependency disagreement, preserving fast path for clean commands.
   - Expected advantage: Raises fast-path success ratio for admitted commands.
   - Main risk: Can starve hot commands unless fallback is fair.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

69. **Conflict-class backpressure**
   - Core mechanism: Apply backpressure only to conflict classes whose fast certificates are frequently invalidated.
   - Expected advantage: Keeps independent classes at 1-RTT latency.
   - Main risk: Requires accurate attribution of slow-path causes.
   - Closest related protocols: [[EPaxos-Revisited-2021]], [[Mencius]].

70. **Predictive quorum pinning**
   - Core mechanism: Pin fast-quorum choice for workload phases predicted to be stable.
   - Expected advantage: Reduces dependency disagreement and cache misses in quorum members.
   - Main risk: Pinning can hurt availability and fairness during phase changes.
   - Closest related protocols: [[SwiftPaxos]], [[EPaxos]].

71. **Shard-local fast SMR with global dependency bridge**
   - Core mechanism: Each shard uses 1-RTT fast commit; cross-shard commands commit a bridge dependency record in each involved shard.
   - Expected advantage: Keeps single-shard operations fast while making cross-shard ordering explicit.
   - Main risk: Bridge records may require multi-shard atomic recovery.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

72. **Hierarchical conflict classes**
   - Core mechanism: Commands first agree at a coarse class, then refine dependencies within sub-classes for execution.
   - Expected advantage: Supports wide key ranges without fully serializing the system.
   - Main risk: Coarse-class agreement may add unnecessary dependencies.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

73. **Cross-shard dependency receipts**
   - Core mechanism: A shard can fast-ack a cross-shard command using receipts from other shards' fast certificates.
   - Expected advantage: Avoids a second consensus round for common cross-shard cases.
   - Main risk: Receipt freshness and atomic visibility need proof.
   - Closest related protocols: [[EPaxos]], [[Pando]].

74. **Commutative escrow commands**
   - Core mechanism: Escrow-style operations reserve bounded resources with fast quorum evidence and commute within safe limits.
   - Expected advantage: High-throughput 1-RTT commits for counters, quotas, and inventories.
   - Main risk: Escrow invariant violations if reservations are recovered incorrectly.
   - Closest related protocols: [[Mencius]], [[EPaxos]].

75. **Hybrid log-DAG protocol**
   - Core mechanism: Use per-shard logs for hot sequential classes and EPaxos-style DAG dependencies between shards/classes.
   - Expected advantage: Combines simple execution in hot streams with concurrent independent streams.
   - Main risk: Log-DAG boundary may be hard to recover safely.
   - Closest related protocols: [[Mencius]], [[EPaxos]].

76. **Region-local command families**
   - Core mechanism: Commands whose keys are region-affine use a local fast quorum family; global commands use normal EPaxos/SwiftPaxos-sized quorums.
   - Expected advantage: Improves WAN locality without changing safety assumptions.
   - Main risk: Region-affinity mistakes can cause unexpected slow paths.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

77. **Dependency namespaces**
   - Core mechanism: Dependencies are scoped by namespace, with explicit namespace-join commands for operations spanning scopes.
   - Expected advantage: Reduces false conflicts across independent application domains.
   - Main risk: Namespace joins become complex and may reintroduce global ordering.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

78. **Fast path for monotone data types**
   - Core mechanism: Treat monotone updates as always commuting and agree only on compact causality metadata.
   - Expected advantage: Very small 1-RTT certificates for CRDT-like subsets inside SMR.
   - Main risk: Mixing monotone and non-monotone operations requires strict boundaries.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

79. **Multi-object deterministic rank**
   - Core mechanism: Cross-object commands orient dependencies by a deterministic rank over involved object sets.
   - Expected advantage: Avoids conflicting replicas choosing opposite dependency directions.
   - Main risk: Rank can serialize large object sets and hurt concurrency.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]].

80. **Fast conflict-class migration**
   - Core mechanism: Move a hot class to a new anchor/quorum family using a slow-quorum-certified migration record, while other classes remain fast.
   - Expected advantage: Supports load balancing without global reconfiguration.
   - Main risk: Commands crossing the migration boundary need careful recovery.
   - Closest related protocols: [[SwiftPaxos]], [[Mencius]].

81. **Fast reconfiguration shadow quorum**
   - Core mechanism: During reconfiguration, old and new fast quorums overlap through a shadow certificate no larger than the normal fast-quorum budget per configuration.
   - Expected advantage: Maintains 1-RTT fast commits across membership changes.
   - Main risk: Cross-configuration intersection proof is likely delicate.
   - Closest related protocols: [[FastPaxos]], [[SwiftPaxos]].

82. **Epoch-stamped dependency graph**
   - Core mechanism: Dependencies carry epoch stamps; recovery validates only within relevant epochs plus certified boundary edges.
   - Expected advantage: Limits recovery search during long runs and reconfigurations.
   - Main risk: Boundary edges can be lost or under-specified.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

83. **Fast quorum health epochs**
   - Core mechanism: Replicas publish health epochs; fast quorums are selected from the current health epoch without leases.
   - Expected advantage: Avoids obviously slow or failed replicas in 1-RTT path.
   - Main risk: Health views may diverge and break intended quorum-family constraints if not certified.
   - Closest related protocols: [[SwiftPaxos]], [[EPaxosStar]].

84. **Graceful degradation e-fast mode**
   - Core mechanism: Dynamically lower the fast-path failure budget `e` during failures to keep `n - e` available, while preserving full `f` recovery.
   - Expected advantage: Makes the EPaxos* `e`/`f` split operational.
   - Main risk: Mode changes must be agreed before certificates can mix.
   - Closest related protocols: [[EPaxosStar]].

85. **Fast certificate format negotiation**
   - Core mechanism: Replicas agree per epoch on dependency encoding formats, not on larger quorums or slower commit rules.
   - Expected advantage: Enables experimentation with metadata without changing core quorum proof.
   - Main risk: Mixed-format recovery may misinterpret old evidence.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

86. **Rolling recovery checkpoints**
   - Core mechanism: Asynchronously checkpoint committed dependency graph cuts so recovery for later commands can ignore older graph regions.
   - Expected advantage: Keeps recovery bounded while fast commits remain 1 RTT.
   - Main risk: Checkpoints must be proved to preserve visibility and execution order.
   - Closest related protocols: [[EPaxosStar]], [[Mencius]].

87. **Fast quorum replacement certificates**
   - Core mechanism: If a fast-quorum member becomes unavailable, a slow quorum certifies a replacement role for future commands only.
   - Expected advantage: Avoids global reconfiguration for small failures.
   - Main risk: Future-only replacement must not be confused with past fast evidence.
   - Closest related protocols: [[SwiftPaxos]], [[FastPaxos]].

88. **Region-failure-aware anchors**
   - Core mechanism: Anchor assignment avoids placing all recovery-critical witnesses in one failure domain.
   - Expected advantage: Improves recoverability after data-center outages without larger fast quorums.
   - Main risk: Failure-domain assumptions may not match real correlated failures.
   - Closest related protocols: [[SwiftPaxos]], [[Pando]].

89. **Fast path with archival witnesses**
   - Core mechanism: Non-voting archival witnesses receive fast certificates asynchronously and can help recovery but do not count in fast quorum size.
   - Expected advantage: Improves auditability and recovery evidence without affecting latency.
   - Main risk: Witnesses cannot be required for safety-critical fast commit.
   - Closest related protocols: [[Pando]], [[EPaxosStar]].

90. **Certificate garbage-collection protocol**
   - Core mechanism: Garbage collect fast evidence only after a slow quorum certifies all dependent commands are committed or checkpointed.
   - Expected advantage: Prevents recovery from losing needed evidence in long-running deployments.
   - Main risk: GC certificates may grow or block under stuck dependencies.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

91. **Mechanized minimal core**
   - Core mechanism: Design a protocol around a tiny adopt-commit-like fast-evidence abstraction, then compile to EPaxos-style dependencies.
   - Expected advantage: Makes proof obligations explicit before optimizing implementation.
   - Main risk: The abstraction may be too weak for real dependency recovery.
   - Closest related protocols: [[adopt-commit-abstraction]], [[EPaxosStar]].

92. **Recoverability-first fast design**
   - Core mechanism: Start from the recovery selection predicate and permit only fast metadata that the predicate can reconstruct.
   - Expected advantage: Avoids EPaxos-style recovery ambiguity by construction.
   - Main risk: May rule out useful optimizations too conservatively.
   - Closest related protocols: [[EPaxosStar]], [[FastPaxos]].

93. **Parametric e-fast protocol generator**
   - Core mechanism: Generate protocol variants for different `e` values under the EPaxos* lower-bound shape.
   - Expected advantage: Lets deployments trade process count, fast availability, and quorum size explicitly.
   - Main risk: Generated variants still need individual proof or a strong generic theorem.
   - Closest related protocols: [[EPaxosStar]], [[quorum-intersection]].

94. **Conflict-invariant DSL**
   - Core mechanism: Applications declare conflict and commutativity rules in a restricted DSL that also emits proof obligations.
   - Expected advantage: Reduces ad hoc conflict predicates in dependency-based SMR.
   - Main risk: DSL expressiveness may be insufficient or unsoundly compiled.
   - Closest related protocols: [[EPaxos]], [[Mencius]].

95. **Proof-carrying fast acknowledgements**
   - Core mechanism: Fast acknowledgements include machine-checkable evidence that the replica's dependency proposal follows the conflict rules.
   - Expected advantage: Helps detect implementation bugs and unsafe metadata.
   - Main risk: Proof payloads may be too large for practical fast paths.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

96. **Invariant-guided slow fallback**
   - Core mechanism: Slow path chooses the smallest dependency repair that restores agreement, visibility, and acyclicity invariants.
   - Expected advantage: Avoids over-unioning dependencies after fast-path disagreement.
   - Main risk: Minimal repair search may be computationally expensive.
   - Closest related protocols: [[EPaxos]], [[SwiftPaxos]], [[EPaxosStar]].

97. **Fast certificate model checker**
   - Core mechanism: Each proposed optimization must define a certificate schema checked against small-model quorum/recovery counterexamples.
   - Expected advantage: Filters unsafe ideas before implementation.
   - Main risk: Passing small models does not prove general correctness.
   - Closest related protocols: [[FastPaxos]], [[EPaxosStar]], [[rocq-modeling-notes]].

98. **Dependency normalization theorem**
   - Core mechanism: Normalize different dependency encodings into one canonical partial order before commit.
   - Expected advantage: Allows replicas to agree despite using different local metadata representations.
   - Main risk: Canonicalization may hide distinctions recovery needs.
   - Closest related protocols: [[SwiftPaxos]], [[EPaxos]].

99. **Safety-envelope annotations**
   - Core mechanism: Every fast certificate carries an annotation naming the exact invariant branch it relies on, such as conflict-free, anchored, or validated.
   - Expected advantage: Makes mixed-mode protocols more auditable and easier to model.
   - Main risk: Incorrect annotations could mislead recovery unless independently checked.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]].

100. **Protocol-family workbench**
   - Core mechanism: Treat quorum family, leader role, dependency encoding, and recovery predicate as pluggable dimensions with generated proof obligations.
   - Expected advantage: Systematically explores novel 1-RTT designs without exceeding known fast-quorum budgets.
   - Main risk: The design space may produce many variants whose proofs do not compose.
   - Closest related protocols: [[EPaxosStar]], [[SwiftPaxos]], [[FastPaxos]].

## Immediate Proof Obligations

- For each candidate, define the committed object precisely: command, dependency set, dependency path, constraint set, or certificate root.
- Prove command-level agreement under the selected fast and recovery quorum families.
- Prove visibility/order for every pair of conflicting committed non-`Nop` commands.
- Prove execution safety: either acyclic committed dependencies or deterministic SCC execution.
- Define the recovery selection predicate before optimizing the fast path.
- Show that any adaptive mode, epoch, anchor, or quorum-family transition has explicit boundary evidence.

## Highest-Potential Clusters

- Anchored-but-distributed fast paths: ideas 1-10.
- Recoverability-first metadata: ideas 21-30 and 91-100.
- Execution-latency repair for EPaxos-like protocols: ideas 51-60.
- Workload-adaptive fast paths: ideas 61-70.
- Sharded and multi-object fast SMR: ideas 71-80.

## TODO

- Pick 5-10 candidates and turn each into a proof sketch with exact quorum assumptions.
- Derive which candidates fit EPaxos `n = 2f + 1` directly and which require SwiftPaxos C2-style fixed fast quorum families.
- Add counterexample searches for candidates involving negative evidence, mode changes, or cross-shard receipts.
