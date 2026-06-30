# quorum

A [[quorum]] is a set of participants whose evidence is sufficient for a protocol step. Important dimensions are size, whether it includes a leader/coordinator, and which other quorum families it must intersect.

[[Atlas]] uses three quorum families: fast quorums of size `floor(n/2) + f`, slow quorums of size `f + 1`, and recovery quorums of size `n - f`.

[[PigPaxos]] keeps ordinary Paxos majority quorums. Relay groups aggregate communication but are not quorum systems; the leader must count unique replica votes and suppress duplicates.

[[Rabia]] waits for `n - f` messages in Proposal, State, and vote steps under `n ≥ 2f + 1`; it uses `⌊n/2⌋ + 1` to detect a majority proposal/state and `f + 1` non-`?` votes to decide a binary value.

## Related pages
[[FastPaxos]], [[EPaxos]], [[Mencius]], [[PigPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[Rabia]]

