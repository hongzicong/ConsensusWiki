# liveness

Liveness is conditional in asynchronous consensus; each paper states progress under availability and eventual stability assumptions.

[[Atlas]] liveness depends on collecting the required quorum for the path being used. The paper explicitly says that if more than `f` sites are unavailable, Atlas may block until enough sites are reachable, while safety is preserved.

[[PigPaxos]] liveness follows Paxos-style majority availability plus relay timeout/retry behavior. Failed relays can delay an operation, but random relay reselection should eventually find healthy relays when enough nodes are live and message delays satisfy the assumed bound.

[[Rabia]] gives probabilistic liveness: each Weak-MVC phase has probability at least `1/2` of leading to termination, and a slot terminates with probability 1 when a majority of replicas is non-faulty.

[[Hermes]] completes in a stable membership when every live member responds. Message loss causes retransmission or replay; a crashed member blocks the write-all path until RM waits out leases and installs a new epoch. Reads and new writes may stall while their local key is not `Valid`, and only the primary partition serves during a network split.

[[Copilot]] assumes eventual partial synchrony and eventual message delivery. A double-induction argument advances one of the two executed log prefixes; unresolved dependencies trigger fast takeover, while failure of both pilots triggers view changes. Its separate [[slowdown-tolerance]] goal asks for near-normal latency with one slow replica, which is stronger than eventual completion alone.

[[Avicenna]] assumes at most `f` faulty replicas, eventual communication within timeouts among a majority, and persistent client retransmission. Missing commits trigger repeated phase rotations. Periodic Armageddon phases make all `2f + 1` replicas non-standbys, so the deterministic leader schedule eventually reaches a phase containing a live majority and live real leader; empty log gaps are filled with `no-op` before execution advances.

[[Bodega]] proves write liveness by restricting a higher-ballot roster to any healthy majority after old leases are revoked or expire; ordinary consensus then progresses with leader and responders inside that group. Local reads escape uncertain optimistic holding through client timeout and redirection. Heartbeat-only roster activation can still risk liveness under partial partitions unless paired with pre-vote or rerouting techniques.

[[Jetpack]] leaves the original path independently live: a failed fast attempt waits for the host result rather than starting a sequential fallback. View changes pause the shim until a majority agrees on the last normal view's recovery set and the new host proposer commits that set/no-op marker. Progress therefore inherits host view-change assumptions plus availability of both recovery and new-view majorities.

[[FPaxos]] has phase-specific liveness. A stable leader needs an available `Q2`; electing a replacement requires an available `Q1` and then `Q2`. With simple quorums and smaller `Q2`, full recovery is guaranteed through `|Q2| - 1` arbitrary failures, while the old leader may continue replication under additional failures if some `Q2` remains.

[[OmniPaxos]] requires partial synchrony and at least one quorum-connected server. BLE eventually identifies a QC maximum-ballot leader after stable heartbeat rounds; Sequence Paxos then gathers a majority and synchronizes that leader even if its log was stale. The paper's experiments report three or four election timeouts/heartbeat rounds in its constrained and quorum-loss scenarios, but those measurements should not be generalized beyond the stated connectivity/timing model.

[[Hydra]] progresses while receivers obtain advancing groupcasts or flushes from every live sequencer and the configuration service can remove failures using a quorum from every receiver group. [[HydraPaxos]] additionally needs a live majority and leader; message loss may add recovery or `NO-OP` agreement but does not change safety.

[[WPaxos]] progresses while at least one valid `Q1` and `Q2` remain alive and leader contention resolves. If every `Q1` is unavailable, object stealing stops, but an existing owner may continue serving its objects where a valid `Q2` remains; this is partial availability, not full ownership mobility.

## Related pages
[[PigPaxos]], [[FPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], [[WPaxos]], [[Atlas]], [[Rabia]], [[Hermes]], [[Copilot]], [[Avicenna]], [[Bodega]], [[Jetpack]], [[reliable-membership]], [[partial-connectivity]], [[sequence-consensus]], [[reconfiguration]], [[roster-lease]], [[view-change-hazard]], [[flexible-quorum]], [[slowdown-tolerance]], [[agreement]], [[recovery]], [[quorum]]
