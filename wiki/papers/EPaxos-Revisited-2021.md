---
type: paper
title: EPaxos Revisited
authors: Sarah Tollman, Seo Jin Park, John Ousterhout
year: 2021
venue: NSDI 2021
source: raw/epaxos_revisited.pdf
protocols: [EPaxos]
tags: [paxos, leaderless, dependencies, fast-path, geo-replication, evaluation, clocks]
status: ingested
---

# EPaxos Revisited

## One-sentence summary
EPaxos Revisited re-evaluates EPaxos under broader conflict workloads and shows that EPaxos keeps its median-latency advantage, but can suffer much worse tail and execution latency than the original evaluation suggested.

## Why this paper matters
This paper is the main ingested cautionary source for using [[EPaxos]] as a baseline: it distinguishes commit latency from execution latency, exposes workload-sensitive conflict behavior, and proposes two performance fixes.

## System model
Geo-replicated state machine replication over EPaxos-style replicas. The evaluation uses five Google Cloud regions: Virginia, California, Oregon, Japan, and London.

## Fault model
The paper focuses on normal-operation performance rather than crash recovery. It inherits EPaxos' non-Byzantine assumptions and explicitly treats crash recovery as outside the performance-oriented discussion.

## Timing assumptions
Safety does not depend on clocks. The Timestamp-Ordered Queueing optimization assumes sufficiently synchronized physical clocks for performance, not correctness; poor synchronization or out-of-order messages reduce conflict mitigation rather than breaking safety.

## Main idea
EPaxos' performance is determined less by median commit latency than by conflict behavior, dependency-chain execution delays, batching, workload skew, read/write mix, and offered load.

## EPaxos behavior under reevaluation
The paper reproduces the original style of result in which many operations achieve one-WAN-RTT commit latency. It then shows that this metric is too favorable because many practical operations require execution, not just durability, before returning to clients.

## Commit vs. execution latency
EPaxos commits when an operation and its dependencies are durable, but execution may wait for transitive dependencies to commit and be ordered. Blind writes may be able to return at commit time; reads, operations that return values, and operations that can fail usually need execution latency.

## Conflict handling
Conflicts occur when fast-path quorum replicas observe interfering operations in different orders and therefore return different dependency metadata. The paper argues that conflict rate is not the same as the fraction of accesses to one hot key; it is a product of workload skew, read/write mix, offered load, batching, topology, and message timing.

## Fast path
The fast path remains valuable when PreAccept replies agree. The reevaluation shows that fast-path commit does not guarantee low execution latency, because a command can commit fast while still waiting on dependency chains before it can execute.

## Slow path
A PreAccept conflict forces extra WAN communication. The paper reports that for EPaxos to beat Multi-Paxos at the 99th percentile, fewer than 1% of operations must encounter PreAccept conflicts in the tested topology.

## Dependency-chain bounding
Original EPaxos can build unbounded transitive dependency chains under highly skewed workloads. The paper describes pruning transitive dependencies that must execute after the current instance because they have higher sequence numbers and already depend on it. With this modification, execution latency is bounded at roughly 3 WAN RTTs in the paper's model.

## Timestamp-Ordered Queueing
Timestamp-Ordered Queueing, or TOQ, uses synchronized clocks to assign a `ProcessAt` time to PreAccept messages. Replicas delay processing so quorum members process a given PreAccept at approximately the same logical point in the message order, reducing disagreement in dependency replies.

## Sync groups
The paper evaluates several TOQ scopes:

- Quorum sync delays only members of the originator's quorum and does not increase minimum latency in the evaluated topology.
- Quorum-union sync reduces conflicts more aggressively but can increase minimum latency.
- Whole-cluster sync can eliminate conflicts but adds too much delay to be attractive.

## Evaluation methodology changes
The reevaluation differs from the original EPaxos evaluation by measuring execution latency, using Zipfian key distributions, mixing reads and writes, measuring near 80% of maximum throughput, using Poisson arrivals, measuring throughput in the WAN configuration, and sweeping many workload configurations instead of using three hot-key fractions.

## Key results
- EPaxos often has better mean latency than Multi-Paxos, except where clients are close to the Multi-Paxos leader.
- EPaxos usually has worse 99th percentile execution latency than Multi-Paxos in the evaluated workloads.
- Many workloads preserve the one-RTT median, so median latency can hide the conflict and dependency behavior.
- Tail latency can be more than 4x worse than Multi-Paxos.
- TOQ reduces conflict rates by at least 50% for quorum sync in the reported experiments, and quorum-union sync reduces conflicts by at least 85% while adding latency in some locations.
- The dependency-chain fix significantly reduces tail latency for highly skewed write-heavy workloads.
- Batching increases throughput but can sharply increase conflict rate and latency because a batch conflicts if any operation inside it conflicts.

## Limitations
The paper does not re-prove EPaxos recovery correctness. It also notes that throughput measurements are sensitive to implementation artifacts, including scheduler behavior and implementation-specific execution scanning.

## Open questions
- TODO: Formalize the dependency-pruning rule as an execution-order invariant.
- TODO: Compare TOQ with CAESAR and Clock-RSM in a dedicated clock-based consensus note.
- TODO: Decide whether wiki latency comparisons should default to commit latency, execution latency, or both.

## Related pages
[[EPaxos]], [[EPaxos-2013]], [[dependency]], [[conflict]], [[fast-path]], [[slow-path]], [[leaderless-protocols]], [[conflict-handling]], [[fast-paths]]
