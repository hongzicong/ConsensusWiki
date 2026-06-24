# dependency

A [[dependency]] records that one command must be considered before another. EPaxos allows dependency cycles and orders SCCs; SwiftPaxos tries to keep committed dependencies acyclic.

[[EPaxos-Revisited-2021]] emphasizes that dependency metadata affects not only safety but execution latency. A command may be committed while still waiting for transitive dependencies, and unbounded dependency chains can cause high tail latency unless dependencies that must execute later are pruned.

[[EPaxosStar|EPaxos*]] uses dependency sets without EPaxos sequence numbers in its core presentation. Its key visibility invariant says that if two distinct committed non-`Nop` commands conflict, at least one command identifier must be in the other's dependency set.

## Related pages
[[FastPaxos]], [[EPaxos]], [[EPaxosStar|EPaxos*]], [[SwiftPaxos]], [[Pando]]
