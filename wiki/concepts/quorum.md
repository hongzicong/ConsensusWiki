# quorum

A [[quorum]] is a set of participants whose evidence is sufficient for a protocol step. Important dimensions are size, whether it includes a leader/coordinator, and which other quorum families it must intersect.

[[Atlas]] uses three quorum families: fast quorums of size `floor(n/2) + f`, slow quorums of size `f + 1`, and recovery quorums of size `n - f`.

[[PigPaxos]] keeps ordinary Paxos majority quorums. Relay groups aggregate communication but are not quorum systems; the leader must count unique replica votes and suppress duplicates.

[[GPaxos]] uses ballot-dependent quorums. All quorums intersect, and fast ballots require triple intersection: any two fast quorums and any recovery/classic quorum intersect in at least one acceptor.

[[Rabia]] waits for `n - f` messages in Proposal, State, and vote steps under `n ≥ 2f + 1`; it uses `⌊n/2⌋ + 1` to detect a majority proposal/state and `f + 1` non-`?` votes to decide a binary value.

[[CURP]] uses an all-witness durability rule rather than majority voting in its primary-backup fast path: with one master, `f` backups, and `f` witnesses, a 1 RTT update must be recorded by all `f` witnesses. Recovery restores one backup and replays one selected witness.

[[Copilot]] uses `n = 2f + 1`, a fast quorum of `f + floor((f + 1) / 2)` including the proposing pilot, and majority `f + 1` quorums for regular Accept and takeover. The fast/recovery intersection lower bound is `floor((f + 1) / 2)`, which supplies evidence but does not always identify the safe recovery value without inspecting the other log.

[[Bodega]] uses majority size `m = ceil(n / 2)` but strengthens each write quorum to cover every active responder for the written key. A responder also needs `m` active [[roster-lease]] grants and committed-prefix coverage for thresholds from some `m` grantors before serving locally.

[[Jetpack]] uses `n = 2f + 1`, a fast superquorum of at least `f + ceil(f/2) + 1` that must include every current original-path proposer, and recovery majorities of `f + 1`. Their minimum intersection is `ceil(f/2) + 1`, exactly the recovery candidate threshold.

[[FPaxos]] separates Phase 1 and Phase 2 quorum families. Every valid `Q1` must intersect every valid `Q2`, but `Q1-Q1` and `Q2-Q2` intersections are unnecessary. For simple threshold quorums, `|Q1| + |Q2| > N`.

[[OmniPaxos]] uses fixed majorities for BLE observation, Sequence Paxos Prepare, and Accept. Its liveness contribution is topological rather than a new quorum size: one quorum-connected server may have direct links to a majority even when those followers are not mutually connected. The paper does not state a separate symbolic `N,f` formula.

[[Hydra]] has no sequencing commit quorum. Its configuration service needs one application-defined quorum from every receiver group to close a failed sequencer's sequence-number suffix; using the whole receiver group strengthens group-level drop detection to uniform per-receiver notification. [[HydraPaxos]] separately commits after `f + 1` consistent replies among `2f + 1` replicas, including the leader.

[[WPaxos]] composes zone and node choices. `Q1` takes `f_n + 1` nodes from each of `Z - f_z` zones; `Q2` takes `l - f_n` nodes from each of `f_z + 1` zones. Their sizes are `(f_n + 1)(Z - f_z)` and `(l - f_n)(f_z + 1)`, and every cross-phase pair intersects even though same-phase quorums may not.

## Related pages
[[FastPaxos]], [[FPaxos]], [[OmniPaxos]], [[Hydra]], [[HydraPaxos]], [[WPaxos]], [[GPaxos]], [[EPaxos]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[CURP]], [[Copilot]], [[Bodega]], [[Jetpack]], [[partial-connectivity]], [[sequence-consensus]], [[flexible-quorum]]

