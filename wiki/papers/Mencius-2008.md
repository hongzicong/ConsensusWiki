---
type: paper
title: Mencius: Building Efficient Replicated State Machines for WANs
authors: [Yanhua Mao, Flavio P. Junqueira, Keith Marzullo]
year: 2008
venue: 8th USENIX Symposium on Operating Systems Design and Implementation
source: raw/mencius.pdf
protocols: [Mencius]
tags: [paxos, smr, wan, multi-leader, rotating-coordinator]
status: ingested
---

# Mencius: Building Efficient Replicated State Machines for WANs

## One-sentence summary
Mencius is a Paxos-derived multi-leader state machine replication protocol for WANs that partitions consensus instances across servers, uses cheap `no-op` skipping to avoid waiting for idle coordinators, and optionally commits commutable operations out of order.

## Why this paper matters
Mencius shows how to remove the single-leader bottleneck of Paxos without using Fast Paxos-style collision-prone direct proposals. Its main modeling value is the decomposition into simple consensus, per-instance rotating coordinators, skip evidence, revocation, and commit ordering.

## System model
- The system has `n` sites, each with one server and a group of local clients.
- Sites are connected by a wide-area network with higher latency, lower bandwidth, and potentially high latency variance compared with local-area links.
- The paper models WAN links pairwise between servers; bandwidth can be asymmetric and variable.
- Servers implement replicated state machine execution with 1-copy serializability.
- Clients send requests to their local server. The paper assumes it is acceptable for clients not to make progress while their local server is crashed.
- Servers run an unbounded sequence of concurrent consensus instances and commit an instance only after learning and committing all previous instances, unless the optional out-of-order commit mechanism is used for commutable operations.

## Fault model
- Servers fail by crashing and may later recover.
- Servers use stable storage to recover state after failures.
- The protocol is non-Byzantine; the paper explicitly leaves Byzantine Mencius as future work.
- With `n = 2f + 1`, Mencius has the same quorum size as Paxos, `f + 1`, and tolerates up to `f` server failures.

## Timing assumptions
- Safety is asynchronous and does not depend on the failure detector being accurate.
- Liveness uses an unreliable failure detector. The paper requires that eventually all faulty servers and only faulty servers are suspected.
- Channels between correct servers are FIFO and eventually deliver messages. The implementation uses TCP.
- FIFO channels are used for skip-message piggybacking optimizations; recovery after long disconnection does not rely solely on FIFO buffering.

## Main idea
Mencius partitions the sequence of consensus instances among servers. In the simple assignment used in the paper, server `p` coordinates instances `cn + p`, where `c ∈ N0` and `p ∈ {0, ..., n - 1}`. Each instance is an instance of simple consensus: only the designated coordinator may propose an arbitrary command, while all other servers may propose only `no-op`.

This avoids ordinary proposal contention because only one server can introduce a real command for a given instance. It also lets idle coordinators cheaply skip their turns, so faster or busier servers do not wait for slower or idle ones.

## Protocol roles
- Client: submits a request to its local server.
- Server / replica: participates in all simple consensus instances and executes committed commands.
- Coordinator: the server assigned to a particular instance; it is the default leader for that instance.
- Revocation leader: a server that suspects a coordinator and runs a higher Paxos-style round to finish instances coordinated by that suspect.

## Message types
- `SUGGEST`: a coordinator's phase-2 proposal of a client request for its instance.
- `SKIP`: a coordinator's phase-2 proposal of `no-op` for its instance.
- `ACCEPT`: acknowledgement of a `SUGGEST`; with Optimization 1, it also promises that the sender will not suggest client requests to earlier instances it coordinates.
- `LEARN`: notification of the learned outcome.
- `PREPARE` / `ACK`: Paxos Phase 1 messages used during revocation.
- Application API: `PROPOSE(v)` and `ONCOMMIT(v)`; with out-of-order commit enabled, the implementation also uses `ISCOMMUTE(u, v)`.

## Local state
- `Ip`: server `p`'s next simple consensus sequence number, called its index.
- Per-instance Coordinated Paxos state, including promises and accepted values/rounds.
- Learned and committed instance outcomes.
- Stable log state for crash recovery.
- Application-level sequence numbers on messages to maintain FIFO behavior across broken TCP connections.
- Skip state or promises, including deferred `SKIP` propagation bounded by parameters `alpha` and `tau`.
- Optional dependency information for out-of-order commit.

## Normal path
Rule 1: when server `p` receives a client request, it suggests the request in instance `Ip` and advances `Ip` to the next instance coordinated by `p`.

In the common case, the coordinator sends `SUGGEST`, receives `ACCEPT` messages from a quorum, learns the value, and sends `LEARN`. With no contention and with earlier instances already learned or skipped, the proposing server can commit in one round-trip delay.

## Fast path
Mencius does not use a Fast Paxos-style fast quorum. Its low-latency path comes from local per-instance coordinators: any server can be the coordinator of some future instances and can immediately suggest requests for its own turns.

There is a separate one-way optimization for `no-op`: because only the coordinator can propose non-`no-op`, a server can learn `no-op` for an instance as soon as it receives a `SKIP` message from that instance's coordinator.

## Slow path
Slow behavior appears mainly as delayed commit rather than a separate slow consensus mode. Concurrent suggestions in adjacent instances can make a server learn a later instance before an earlier instance. In the paper's example, the later value may wait up to two communication steps until the earlier instance is learned.

The protocol can reduce the delayed-commit bound by broadcasting `ACCEPT` messages and eliminating `LEARN`, but this increases message complexity from `3n - 3` to `n^2 - 1`.

## Recovery path
Rule 3: if server `p` suspects server `q`, let `Cq` be the smallest instance coordinated by `q` and not learned by `p`. Server `p` revokes `q` for all instances in `[Cq, Ip]` that `q` coordinates.

Revocation is Paxos-like. A new leader starts Phase 1 in a higher round `r' > r`. If Phase 1 shows no value may have been chosen, it proposes `no-op`; otherwise, it proposes the possible consensus outcome indicated by Phase 1.

Optimization 3 revokes ranges in advance: for parameter `beta`, if `Cq < Ip + beta`, `p` revokes `q` for instances in `[Cq, Ip + 2 beta]` that `q` coordinates. This amortizes revocation and tries to avoid delayed commits caused by a crashed coordinator.

Temporary broken TCP connections are handled with application-layer sequence numbers and retransmission. Short-term crash recovery replays stable logs and learns recent chosen requests. Long-term recovery is left to application-specific checkpointing or state transfer.

## Commit rule
- A request is chosen when the corresponding simple consensus instance chooses it.
- A server commits a request after it learns the instance outcome and all previous instances have been learned and committed.
- With out-of-order commit, a server may commit a later request before learning an earlier one if the relevant requests commute and all dependencies of the later request have already committed.
- If a server suggests a non-`no-op` value `v` and learns that `no-op` was chosen, Rule 4 says it suggests `v` again.

## Quorum system
- Total servers: `n = 2f + 1`.
- Fault tolerance: up to `f` crashed servers.
- Basic quorum size: `f + 1`.
- Fast quorum size: not applicable in the Fast Paxos sense.
- Classic/recovery quorum size: `f + 1`.
- `SKIP` learning is not quorum-based; it is safe because simple consensus restricts non-coordinators to proposing only `no-op`.
- The paper states Mencius has the same quorum size as Paxos.

## Conflict handling
Mencius avoids consensus-instance contention by assigning each instance to a unique coordinator and restricting non-coordinators to `no-op`. It does not solve conflicting commands with dependency metadata in the normal consensus path. Optional out-of-order commit uses application commutativity to execute learned commands in different but equivalent orders.

## Safety argument
- Coordinated Paxos is a specialization of Paxos that starts from a safe state in which the coordinator has effectively run Phase 1 for an initial round.
- Simple consensus preserves safety because non-coordinators can only propose `no-op`.
- A `SKIP` from the coordinator safely proves `no-op`, since no other server can propose a non-`no-op` value for that instance.
- Revocation preserves safety by running a higher Paxos-style round and choosing either the possible prior outcome or `no-op`.
- Skip piggybacking relies on FIFO ordering but only delays or compresses evidence; it does not change which values can be chosen.
- Out-of-order commit is safe only when the application reports that reordered operations commute and the tracked dependencies are already committed.

## Liveness argument
Rules 1-4 ensure that a client request sent to a correct server eventually commits, assuming the server is not permanently falsely suspected, enough servers remain live for quorums, messages between correct servers are eventually delivered, and the failure detector eventually stops making relevant mistakes.

## Key proof ideas
- Reduce Mencius to protocol `P`: an unbounded sequence of simple consensus instances solved by Coordinated Paxos.
- Prove safety per instance using Paxos-style reasoning, then prove state machine safety from the learned sequence.
- Prove liveness by showing that any instance before a suggested value is eventually either suggested, skipped, or revoked.
- Treat optimizations as semantics-preserving transformations that defer, piggyback, or batch `SKIP`/revocation evidence.

## Important formulas
- Coordinator assignment: instance `cn + p` is assigned to server `p`, where `c ∈ N0` and `p ∈ {0, ..., n - 1}`.
- Paxos/Mencius quorum statement: with `n = 2f + 1`, quorum size is `f + 1`, tolerating up to `f` server failures.
- Optimization 3 revocation range: if `Cq < Ip + beta`, revoke instances coordinated by `q` in `[Cq, Ip + 2 beta]`.
- Amortized wide-area message complexity after optimizations: `3n - 3`, counted as `(n - 1)` each of `SUGGEST`, `ACCEPT`, and `LEARN`.
- Broadcasting `ACCEPT` instead of `LEARN` increases message complexity to `n^2 - 1`.
- Implementation parameters in the evaluation: `alpha = 20` messages, `tau = 50 ms`, and `beta = 100,000` instances.

## Relationship to other protocols
- [[FastPaxos]] removes a single leader with fast rounds but suffers from collisions under concurrent proposals; Mencius avoids such contention by assigning coordinators to instances.
- [[EPaxos]] and [[SwiftPaxos]] use dependency metadata to order conflicting commands. Mencius uses ordinary sequence instances, with optional commutativity-based out-of-order commit as an execution optimization.
- The paper describes Mencius as a rotating coordinator or moving sequencer protocol.

## Modeling notes

### Rocq/Coq modeling notes
- Model instance ownership as a function `owner : Instance -> Server`.
- Model simple consensus by restricting proposals: only `owner i` may propose non-`no-op` for instance `i`.
- Separate chosen, learned, and committed states.
- Keep `SKIP` as evidence produced by the owner, not as a majority certificate.
- Model revocation with an ordinary higher-round Paxos Phase 1 safe-value predicate.
- Treat FIFO as a hypothesis for the piggybacking optimizations only; the core safety of Coordinated Paxos should not depend on timing.

## Limitations
- Byzantine Mencius is explicitly left open; the paper notes that skipping is not built on a quorum abstraction, making a Byzantine extension nontrivial.
- If any server fails, Mencius may see temporarily reduced performance because every server owns an unbounded number of instances.
- Long-term recovery is application-specific.
- The base assumption that clients do not progress while their local server is crashed is relaxed only as future work.

## Open questions
- What is the cleanest minimal abstraction of `SKIP` evidence for a mechanized proof?
- How should out-of-order commit be represented without entangling consensus safety with application commutativity?
- Can the coordinator allocation idea of using only the fastest `f + 1` servers be formalized without losing fairness or liveness?

## Related pages
- [[Mencius]]
- [[paxos-family]]
- [[leader-roles]]
- [[quorum-systems]]
- [[commit-rules]]
- [[recovery-rules]]
- [[timing-assumptions]]
- [[rocq-modeling-notes]]
