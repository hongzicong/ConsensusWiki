# quorum

A [[quorum]] is a set of participants whose evidence is sufficient for a protocol step. Important dimensions are size, whether it includes a leader/coordinator, and which other quorum families it must intersect.

[[Atlas]] uses three quorum families: fast quorums of size `floor(n/2) + f`, slow quorums of size `f + 1`, and recovery quorums of size `n - f`.

[[PigPaxos]] keeps ordinary Paxos majority quorums. Relay groups aggregate communication but are not quorum systems; the leader must count unique replica votes and suppress duplicates.

[[GPaxos]] uses ballot-dependent quorums. All quorums intersect, and fast ballots require triple intersection: any two fast quorums and any recovery/classic quorum intersect in at least one acceptor.

[[Rabia]] waits for `n - f` messages in Proposal, State, and vote steps under `n ≥ 2f + 1`; it uses `⌊n/2⌋ + 1` to detect a majority proposal/state and `f + 1` non-`?` votes to decide a binary value.

[[CURP]] uses an all-witness durability rule rather than majority voting in its primary-backup fast path: with one master, `f` backups, and `f` witnesses, a 1 RTT update must be recorded by all `f` witnesses. Recovery restores one backup and replays one selected witness.

## Related pages
[[FastPaxos]], [[GPaxos]], [[EPaxos]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]], [[CURP]]

