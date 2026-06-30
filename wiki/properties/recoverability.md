# recoverability

Recoverability is the ability to reconstruct a safe value/metadata state from quorum evidence after failures or collisions.

[[Atlas]] makes recoverability part of the fast-path predicate: a coordinator fast-commits only when every dependency in `union_Q dep` appears in at least `f` fast-quorum replies, so a later recovery quorum can reconstruct the fast proposal.

[[PigPaxos]] recoverability is ordinary Paxos recoverability. Relay aggregation must preserve unique voter evidence, but relays do not create a new recoverable value-selection rule.

[[Rabia]] shifts recoverability into protocol continuation: value-locking ensures later phases preserve any binary value that could already have been decided.

## Related pages
[[PigPaxos]], [[Atlas]], [[Rabia]], [[agreement]], [[recovery]], [[quorum]]
