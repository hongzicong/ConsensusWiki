---
type: paper
title: "SwiftPaxos: Fast Geo-Replicated State Machines"
authors: Fedor Ryabinin, Alexey Gotsman, Pierre Sutra
year: 2024
venue: NSDI 2024
source: raw/swiftpaxos.pdf
protocols: [SwiftPaxos]
tags: [paxos, dependencies, geo-replication, fast-path, leader]
status: ingested
---

# SwiftPaxos: Fast Geo-Replicated State Machines

## One-sentence summary
SwiftPaxos is a geo-replicated state-machine replication protocol that uses leader-including fast quorums, dependency tracking, and optimistic execution to return in two or three message delays.

## Why this paper matters
It refines EPaxos-style dependencies by requiring acyclic committed dependencies and by allowing slow-path repair through double voting inside fast quorums.

## System model
State machine replication with a fixed set `R` of `N = 2f + 1` replicas. Clients submit commands tagged with unique identifiers.

## Fault model
Crash/non-Byzantine failures with at most `f` faulty replicas. Liveness uses an eventual leader/failure detector style assumption.

## Timing assumptions
Designed for WANs with heterogeneous latencies. Safety is asynchronous; liveness follows after recovery stabilizes at a ballot with a correct trusted leader.

For the paper's stable-run latency comparison, `δ` is the upper bound on one message delay. Table 1 measures the maximum time from command submission until the client can deliver a response and distinguishes sequential, conflict-free, contention-free, and general runs; see [[latency]].

## Main idea
Commands carry dependencies on conflicting commands. Replicas agree on dependency paths using `FastAck` and `SlowAck`; execution follows the acyclic dependency graph. The implementation can also execute read-only commands optimistically at any fast-quorum replica, rather than only at the leader, while the client still waits for matching dependency-path evidence.

## Protocol roles
Each ballot has a fixed leader `leader(b)`. Other replicas are followers. A replica may belong to fast and/or slow quorums for a ballot.

## Message types
`Propagate(c)`, `FastAck(b,id,D,P)`, `SlowAck(b,id)`, `NewLeader(b)`, `NewLeaderAck`, `Sync`, and recovery/commit messages.

## Local state
Replicas track `bal`, `cbal`, `status` (`NORMAL` or `RECOVERING`), command table `cmd`, phase sets such as `Start`, `Accept`, `Commit`, dependency map `dep`, and executed set `Exec`.

## Normal path
Clients broadcast `Propagate(c)`. Fast quorum replicas compute dependencies from locally known conflicting commands and broadcast `FastAck`. The leader's proposal is central for both fast and slow agreement.

## Fast path
A replica commits when it receives matching `FastAck` messages from all members of some fast quorum and the command's dependencies are committed. Under favorable conditions, execution can complete within two message delays from submission.

Read-only optimization: a read-only command may be speculatively executed at any fast-quorum replica. The contacted replica computes the tentative read result from its local state plus relevant pending commands; the client accepts that result on the fast path only after other fast-quorum members report matching dependency paths. This distributes read load away from the leader without removing the quorum evidence needed for linearizable ordering.

## Slow path
If a fast quorum replica disagrees with the leader's dependencies, it can send `SlowAck`, adopting/correcting to the leader's proposal. A command can commit with matching `FastAck`/`SlowAck` evidence from a fast quorum, or with `SlowAck`s from a slow quorum.

## Recovery path
A new leader chooses a higher ballot, gathers `NewLeaderAck` from a majority, reconstructs commands that could have been committed in earlier ballots, breaks unsafe dependency cycles, broadcasts `Sync`, and then resumes normal processing.

## Commit rule
Commit requires either matching dependency-path evidence from all followers in a fast quorum or `SlowAck`s from a slow quorum, with `D = dep[id]` and dependencies already committed.

## Quorum system
`N = 2f + 1`. Slow quorums can be any majority. Fast quorums must include the ballot leader and satisfy fast-quorum intersection:
`forall Q1, Q2 in FQ(b). |Q1 intersect Q2| > N/2`.
The paper studies at least two configurations: `(C1)` any set containing more than `3/4` of all replicas, and `(C2)` a unique fixed majority fast quorum including the leader.

## Conflict handling
Conflicting commands must be ordered by dependencies: for any two conflicting commands committed at a replica, one command belongs to the other's dependencies. Unlike EPaxos, committed dependencies are kept acyclic.

## Safety argument
Invariants include: any two replicas commit a command with the same dependencies; conflicting committed commands are dependency-ordered; the committed dependency graph is acyclic.

## Liveness argument
After recovery stops and all correct replicas stabilize in one ballot, commands submitted by correct clients are eventually accepted, committed, and executed.

## Key proof ideas
The proof defines `acc(b,id,c,P)` for accepted dependency paths by a fast or slow quorum. Recovery preserves accepted paths using quorum intersection between recovery majorities and previous fast/slow quorums.

## Important formulas
- `N = 2f + 1`.
- Slow quorum: majority.
- Fast quorum intersection: `|Q1 intersect Q2| > N/2`.
- `(C1)`: fast quorum size `> 3/4 N`.
- `(C2)`: unique fixed majority fast quorum.
- Stable contention-free client-response latency: `2δ`.
- Stable general client-response latency: `3δ`.

## Relationship to other protocols
SwiftPaxos is closely related to [[EPaxos]] through dependencies, to [[FastPaxos|Fast Paxos]] through fast-quorum reasoning, and to classic Paxos through ballots and leader recovery.

Evaluation baseline **FastPaxos+** (§5, proceedings p. 353) means Fast Paxos with **uncoordinated collision recovery**; the `+` denotes that optimization. Clients broadcast to replicas, which independently propose slot orderings. A matching quorum commits; after disagreement, replicas locally derive the next-ballot proposal from a fixed collision-recovery quorum's votes, bypassing the coordinator. The paper describes the fallback as a new N²Paxos ballot; the implementation performs the all-to-all next-ballot voting inside `fastpaxos`, not by invoking the separate `n2paxos` package. The evaluation defaults to C2 for both SwiftPaxos and FastPaxos+. §5.1 distinguishes order collisions from command conflicts: even commuting commands can arrive in different orders and force FastPaxos+ off its fast path.

Artifact mapping checked 2026-09-29, before the local repair: the authors' `imdea-software/swiftpaxos` repository names this package and configuration `fastpaxos`, without `+`. The original ConsensusArena import retained it under that name; comparison with imported commit `4783302` showed only module import-path changes in `fastpaxos.go` and `defs.go`. Both upstream and that original local import explicitly limited collision handling to one fixed fast quorum and stated that failure recovery was absent. Thus the imported artifact contained the `+` collision optimization but not the general failure-recovery mechanism in Lamport's Fast Paxos protocol. This is an implementation observation, not a claim that Fast Paxos cannot recover. Sources: authors' repository `https://github.com/imdea-software/swiftpaxos`, `fastpaxos/fastpaxos.go`; ConsensusArena's original imported revision.

Local repair on 2026-09-29: ConsensusArena's working-tree `fastpaxos/core.go` adds global classic epochs, majority Phase 1 with Figure 2 selection, classic majority acceptance, coordinator replacement, suffix/payload repair and client completion from surviving replicas. It preserves the fixed-quorum collision path before recovery and remains classic after switching. This local extension supports crash-stop service continuation, not durable crash-restart, and must not be attributed to the upstream SwiftPaxos artifact. Design, implementation scope and validation are recorded in the parent repository's `report/fastpaxos-recovery/`; historical baseline measurements still describe the earlier implementation.

## Deployment quorum selection (artifact audit, 2026-10-01)

The paper (§5) places each leader-based protocol's leader to minimize mean client latency and defaults to C2. It does not specify a complete joint quorum/leader optimization algorithm. The authors' artifact links to the separate Flint estimator. In Flint commit `b6b30806141f88442d91f33be8ddbc6a88f9aa04`, `swiftpaxos.go:SetAverageBestFixedQuorumAndLeader` enumerates fixed majority quorums and leaders within each quorum. `algorithm.go` defaults `MinWorstLatency=true`: minimize mean **slow-path** latency across clients, breaking ties by mean contention-free latency. This is not minimization of the worst client or P99. Disabling the option prioritizes the contention-free estimate. `Accept(client,true)` takes the minimum of fast-quorum completion and slow-quorum completion; a three-hop path can beat a two-hop path on heterogeneous links. The UI can export the selected quorum configuration.

The upstream runtime reads a quorum file and selects the ballot-associated quorum; the estimator is separate from the normal request path. Upstream commit `35c69365f1c7737a08e237bfbaf828ee68897080` supplies `quorum.conf` with `ap-south-1`, `ap-northeast-1`, and leader `us-west-1`, matching ConsensusArena's five-replica configuration.

Diagnostic calculation, not a benchmark: enumerating all 30 majority/leader pairs with Flint's formulas over the local five-replica RTT matrix and ten equally weighted clients reproduces that default choice. Its contention-free/slow-path means are 168.45/200.20 ms. Prioritizing contention-free mean selects Japan/Paris/California with Japan as leader, giving 165.25/217.15 ms. These omit queueing, processing, jitter and conflicts. The pure fixed-fast-quorum RTT maximum alone is 197.40 ms for the original configuration and must not be mistaken for the earliest eligible client completion time. Local 9/13-replica topology generation preserves the original quorum and appends members; it does not rerun Flint optimization.

Sources: [paper §§3.1, 5](https://www.usenix.org/system/files/nsdi24-ryabinin.pdf); [upstream quorum](https://github.com/imdea-software/swiftpaxos/blob/35c69365f1c7737a08e237bfbaf828ee68897080/quorum.conf); [Flint SwiftPaxos model](https://github.com/vonaka/flint/blob/b6b30806141f88442d91f33be8ddbc6a88f9aa04/swiftpaxos.go); [Flint objective default](https://github.com/vonaka/flint/blob/b6b30806141f88442d91f33be8ddbc6a88f9aa04/algorithm.go). Local audit inputs: `ConsensusArena/latency.conf`, `slurm/workload.conf`, and `slurm/prepare-topology.sh` in the parent project.

Subsequent local deployment change (2026-10-01): at the user's request, the root fixed quorum file was removed. `slurm/select-quorum.py` now selects the common C2 quorum/initial leader before each size run, with slow-first or fast-first objectives and archived inputs/predictions. The slow certificate explicitly includes the leader and its result. Both latency and fault harnesses use the generated configuration; 9/13 no longer append members to the old quorum. Default selected leaders are California/Virginia/Ohio for 5/9/13 replicas. This is a change for future runs, not a reinterpretation of historical baselines or independent per-protocol leader optimization. See `ConsensusArena/slurm/TOPOLOGY.md` for validation and scope.

Follow-up correction to that deployment change: the user requires independent protocol configuration. The selector now dispatches by protocol, so only SwiftPaxos uses this C2 joint quorum/leader objective. Paxos, N2Paxos and CURP select leaders using their current Arena reply paths; Fast Paxos selects a fixed voter set; EPaxos selects client ingress; Bodega selects leader/responders for each workload's actual read/write mix. KCensus retains native topology synthesis without an external C2 set or global leader. Per-profile/repetition generated settings supersede the size-level template. These are configuration-space/network-model optimizations; runtime queueing, repeated collision/dependency waits and lease read holds remain outside the predictions. No baseline measurement is replaced by a prediction.

Paper-alignment audit of the local selector (2026-10-01): independent configuration does not establish evaluation equivalence. In particular, this paper's EPaxos baseline routes to the closest replica and uses thrifty mode (§5); Arena's new selector searches ingress by estimated full protocol delay. A deeper constructor audit confirms that Arena actually forces EPaxos thrifty mode, despite `thrifty: false` in the workload; the previous claim that thrifty was not enforced was incorrect. The paper and Flint use the closest reply site for N²Paxos, whereas Arena's estimator takes its earliest executable reply path. FastPaxos+'s C2/fixed collision-recovery quorum is paper-backed, but the new set-search objective is a local policy. Bodega's read/write-mix placement policy and Arena KCensus's incomplete graph/delegation optimization must not be attributed to this paper or described as reproducing those protocols' full published optimizers. See the provenance table in `ConsensusArena/slurm/TOPOLOGY.md`; configuration-generation tests do not prove paper equivalence or P99 optimality.

Formula clarification: the local ordinary Paxos estimate `client-leader RTT + f-th fastest follower RTT` is equivalent to Flint's ordinary Paxos majority-path estimate when self-link delay is zero and rounding is ignored. Local code does not imply a distinct mathematical model here. For SwiftPaxos, client completion already requires a leader result in the runtime; the selector explicitly models it rather than adding a protocol step. An offline check of every leader/client pair in the bundled five-replica matrix finds zero change from this constraint and zero difference between the two unrounded ordinary Paxos formulas. Other topologies may expose the SwiftPaxos estimator difference.

Deeper deployment audit: generated EPaxos ingress differs from nearest-replica routing for 0, 2 and 6 of the ten clients at 5, 9 and 13 replicas, respectively. Arena's N²Paxos client actually reads all reply streams, so its earliest-reply estimate follows the current implementation but remains distinct from this paper's routing policy. The unrounded five-replica means are 223.75 ms (Arena model) and 231.95 ms (Flint closest-reply model); selected leaders agree at all three sizes. These are network-model diagnostics, not benchmarks. Full evidence: `report/baseline/configuration-audit.md` and `.json` in the parent repository.

## Limitations
Message complexity is quadratic. The extracted PDF text did not expose full bibliographic metadata; venue/authors need verification from the PDF front matter.

## Open questions
- TODO: Capture exact pseudocode line numbers for the commit preconditions.

## Related pages
[[SwiftPaxos]], [[dependency]], [[conflict]], [[fast-path]], [[leader]], [[recovery]], [[latency]]
