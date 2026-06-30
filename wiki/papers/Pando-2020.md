---
type: paper
title: Near-Optimal Latency Versus Cost Tradeoffs in Geo-Distributed Storage
authors: Muhammed Uluyol, Anthony Huang, Ayush Goel, Mosharaf Chowdhury, Harsha V. Madhyastha
year: 2020
venue: NSDI 2020
source: raw/pando.pdf
protocols: [Pando]
tags: [geo-distribution, erasure-coding, paxos, quorum, storage]
status: ingested
---

# Near-Optimal Latency Versus Cost Tradeoffs in Geo-Distributed Storage

## One-sentence summary
PANDO combines erasure coding, smaller Phase 1/read quorums, and partial delegation to approach optimal latency-cost tradeoffs for strongly consistent geo-distributed storage.

## Why this paper matters
It shows that consensus quorum design changes when values are erasure-coded: reads and Phase 1 can often use smaller nearby quorums, while write safety still depends on Phase 2 intersections.

## System model
Geo-distributed object/key-value storage. Front-end servers serve user requests by accessing data sites. Objects are split into `k` erasure-coded splits; any `k` splits reconstruct the object.

## Fault model
Serve requests as long as fewer than `f` data centers are unavailable. The protocol assumes non-Byzantine acceptors/data sites.

## Timing assumptions
Designed for wide-area latency optimization. Safety is quorum-based; liveness requires available Phase 1b and Phase 2 quorums.

## Main idea
PANDO separates "discover whether the last write completed" from "recover enough data to resolve uncertainty." Common reads use a small Phase 1a quorum; uncertain reads and writes fall back to larger Phase 1b/Phase 2 quorums.

## Protocol roles
Front-ends/proposers initiate reads and writes; optional delegates run Phase 2; data sites/acceptors store proposal metadata, splits, and chosen-value hints.

## Message types
`Prepare`, `Promise`, Phase 2 propose/accept messages, chosen notifications, read responses, and write-back/recovery messages.

## Local state
Acceptors store highest promised proposal number, highest accepted proposal number, accepted value identity/length/split, and a cached `chosen` value when learned.

## Normal path
Writes run Phase 1 against a smaller Phase 1a/read quorum when no prior value is detected, then Phase 2 against a Phase 2 quorum. With delegation, the front-end starts Phase 1 and sends the value to a delegate that starts Phase 2 after receiving enough Phase 1 responses.

## Fast path
Reads return after a Phase 1a quorum if they observe `k` splits of a value known to be chosen. Writes approximate one-round latency by overlapping Phase 1 from the front-end with Phase 2 from a delegate, but still execute two Paxos phases.

## Slow path
If a Phase 1a quorum cannot prove/reconstruct a chosen value, the operation waits for a Phase 1b quorum. Reads may perform write-back when the latest state is uncommitted or uncertain.

## Recovery path
A proposer sorts Promise responses by decreasing proposal number and recovers a value when it finds at least `k` splits for the same value. Otherwise it can propose its own value.

## Commit rule
A value is chosen when a Phase 2 quorum of acceptors all agree on that value identity and enough split metadata exists to reconstruct it.

## Quorum system
The paper states these constraints:
- Any Phase 1a quorum and Phase 2 quorum intersect in at least one site.
- Any Phase 1b quorum and Phase 2 quorum intersect in at least `k` splits.
- A Phase 1a quorum contains at least `k` splits when it is used for fast reads.
- After `f` failures, at least one Phase 1b quorum and one Phase 2 quorum remain available.

Section 3.4 summarizes minimum sizes as Phase 1a `max(k, f + 1)` and Phase 1b/Phase 2 at least `f + k` data sites.

## Conflict handling
Front-ends can conflict when the same write is attempted concurrently. PANDO uses globally unique proposal numbers; front-ends retry through a leader/fallback path after conflict.

## Safety argument
PANDO proves nontriviality, consistency, and stability. The core lemma: if value `v` is chosen with proposal number `p`, any later proposal `p' > p` that succeeds must propose/recover `v`.

## Liveness argument
A value is eventually chosen if at least one Phase 1b quorum and one Phase 2 quorum are available. Front-ends fall back to a leader when direct writes see conflict.

## Key proof ideas
Phase 1a is enough only to discover a chosen value or the absence of relevant chosen evidence; Phase 1b is the recovery quorum that must contain enough splits to reconstruct any previously chosen value.

## Important formulas
- Object split count: any `k` splits reconstruct.
- Phase 1a size: `max(k, f + 1)`.
- Phase 1b and Phase 2 size: at least `f + k`.
- Chosen: exists a Phase 2 quorum that accepted the same value identity.

## Relationship to other protocols
PANDO builds on Paxos and compares against EPaxos, RS-Paxos, Fast Paxos, Mencius, and Multi-Paxos.

## Modeling notes

### Rocq/Coq modeling notes
Model value identity separately from splits. The main obligation is that Phase 1b evidence with `k` splits reconstructs any prior chosen value.

## Limitations
The protocol targets single-key GET and conditional-PUT; multi-key transactions are deferred. The benefits rely on workload and data-site placement.

## Open questions
- TODO: Extract exact pseudocode line numbers for read write-back.

## Related pages
[[Pando]], [[quorum]], [[fast-path]], [[recovery]], [[linearizability]]
