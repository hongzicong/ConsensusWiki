---
type: protocol
name: Mencius
family: paxos / multi-leader smr
papers: [Mencius-2008]
tags: [consensus, paxos, smr, wan, rotating-coordinator]
---

# Mencius

## Short description
Mencius is a Paxos-derived replicated state machine protocol that rotates coordinators by assigning different log instances to different servers.

## Problem solved
It targets high-throughput and low-latency state machine replication across WAN sites without a single permanent Paxos leader.

## System model
`n` WAN sites, each with a server and local clients. Servers communicate over asynchronous FIFO channels and run an unbounded sequence of simple consensus instances.

## Fault model
Crash-recovery servers with stable storage. With `n = 2f + 1`, Mencius uses quorum size `f + 1` and tolerates up to `f` server failures.

## Timing assumptions
Safety is asynchronous. Liveness depends on eventual message delivery between correct servers and an unreliable failure detector that eventually suspects all and only faulty servers.

## Roles
- Client.
- Server / replica.
- Per-instance coordinator.
- Revocation leader for suspected coordinators.

## Message types
`SUGGEST`, `SKIP`, `ACCEPT`, `LEARN`, plus Paxos Phase 1 `PREPARE`/`ACK` messages during revocation.

## Local state
The key state is `Ip`, the next instance coordinated by server `p`, plus per-instance Paxos state, learned/committed outcomes, stable logs, deferred skip promises, and optional dependency state for out-of-order commit.

## Normal path
Server `p` proposes client requests in instances assigned to `p`, such as `cn + p`. Other servers acknowledge with `ACCEPT`; the coordinator learns after quorum evidence and sends `LEARN`.

## Fast path
There is no Fast Paxos-style fast quorum. Mencius's low-latency path is the ordinary coordinator path for the local server's assigned instances. `SKIP` has a special one-way learning rule because only the coordinator may propose a non-`no-op` value.

## Slow path
Concurrent suggestions can delay commit until earlier instances are learned. Deferred `SKIP` propagation is bounded by `alpha` outstanding messages or `tau` time.

## Recovery
A server that suspects coordinator `q` revokes `q`'s unlearned instances by running a higher Paxos-style round. If no prior value may have been chosen, revocation chooses `no-op`; otherwise it preserves the possible prior outcome. Optimization 3 revokes a block of future instances using parameter `beta`.

## Commit condition
In-order commit requires learning an instance outcome and all previous outcomes. Optional out-of-order commit allows commutable requests to execute once their dependencies have committed.

## Quorum requirement
The paper states Mencius has the same quorum size as Paxos: `f + 1` out of `2f + 1`. `SKIP` learning is a special owner-authored `no-op` rule, not a quorum certificate.

## Safety intuition
Each instance is simple consensus implemented by Coordinated Paxos. Since only the coordinator can propose non-`no-op`, revocation and skip learning preserve the same chosen-value invariant as Paxos.

## Liveness intuition
Correct servers keep advancing their owned instance index, idle coordinators skip, suspected coordinators are revoked, and falsely revoked client requests are resubmitted.

## Strengths
- Removes the single Paxos leader bottleneck.
- Balances WAN bandwidth and CPU load across sites.
- Lets idle sites skip cheaply.
- Can exploit application commutativity with out-of-order commit.

## Weaknesses
- Any server failure affects an unbounded set of future instances.
- Commit latency can be limited by slow coordinators unless skips/revocations arrive quickly.
- Byzantine extension is nontrivial because skipping is not quorum-based.
- Long-term recovery is application-specific.

## Differences from related protocols
Unlike [[FastPaxos]], Mencius avoids fast-round collisions by preassigning instance coordinators. Unlike [[EPaxos]] and [[SwiftPaxos]], it does not make dependency metadata the core consensus object; dependencies are only used for optional out-of-order commit.

## Open questions
- TODO: Extract pseudocode details from the technical report [[Mencius-2008]] references if it is added to `raw/`.

## Sources
- [[Mencius-2008]]
