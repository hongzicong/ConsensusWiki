# dependency

A [[dependency]] records that one command must be considered before another. EPaxos allows dependency cycles and orders SCCs; SwiftPaxos tries to keep committed dependencies acyclic.

[[EPaxos-Revisited-2021]] emphasizes that dependency metadata affects not only safety but execution latency. A command may be committed while still waiting for transitive dependencies, and unbounded dependency chains can cause high tail latency unless dependencies that must execute later are pruned.

[[EPaxosStar]] uses dependency sets without EPaxos sequence numbers in its core presentation. Its key visibility invariant says that if two distinct committed non-`Nop` commands conflict, at least one command identifier must be in the other's dependency set.

[[Atlas]] also commits dependency sets without sequence numbers. Its fast-path condition does not require identical dependency replies; it requires the final dependency union to be recoverable because each included dependency appears in at least `f` fast-quorum replies.

## Related pages
[[FastPaxos]], [[EPaxos]], [[EPaxosStar]], [[Atlas]], [[SwiftPaxos]], [[Pando]]
