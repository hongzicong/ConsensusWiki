# agreement

Agreement means no two correct learners/replicas decide incompatible values or command metadata.

[[PigPaxos]] inherits Paxos agreement: relay aggregation changes message transport, but a value is still chosen only through a majority of unique replica votes in a ballot.

[[Atlas]] agreement is per command identifier: all `MCommit` messages for an identifier carry the same command and dependency set. Cross-command ordering is handled by the dependency invariant for conflicting non-`noOp` commands.

[[Rabia]] agreement is per log slot over weak outputs: replicas cannot decide different requests or disagree between a request and `⊥` for the same Weak-MVC instance.

## Related pages
[[PigPaxos]], [[Atlas]], [[Rabia]], [[recovery]], [[quorum]]
