# partial-connectivity

Partial connectivity is a link-level partition in which two servers cannot communicate even though both remain reachable through another server. It differs from a clean node partition because servers may hold inconsistent views of who is reachable and which candidate can actually form a replication quorum.

[[OmniPaxos]] identifies three liveness hazards:

- **Quorum loss:** the old leader remains alive but is no longer connected to a majority, so a well-connected follower may never trigger election.
- **Constrained election:** the only quorum-connected candidate is rejected because its log is not maximal or because it cannot be elected by a majority of quorum-connected voters.
- **Chained election:** disconnected candidates repeatedly propagate higher terms through an intermediary, causing livelock.

A **quorum-connected (QC) server** is correct and has a direct link to at least a majority of correct servers, including itself. OmniPaxos makes QC status the only candidate-eligibility requirement; Sequence Paxos repairs a stale elected log afterward.

## Modeling notes

- Model connectivity as a directed or bidirectional link relation, not only a set of failed processes.
- Distinguish an alive leader from a leader that can collect a quorum.
- Do not assume all members of a majority can communicate with one another; one QC center is sufficient for OmniPaxos replication.
- Election liveness depends on heartbeat timing and stable-enough connectivity, while safety remains a ballot/quorum property.
- Half-duplex links require an extended model; the paper's BLE elects leaders with full-duplex heartbeat connectivity.

Source: [[Omni-Paxos-2023]], §§2-5 and §8.

## Related pages

[[OmniPaxos]], [[leader]], [[quorum]], [[failure-model]], [[liveness]], [[recovery]]
