# Rocq Modeling Notes

## Suggested abstractions
- Quorum family predicates.
- Message evidence predicates.
- Commit predicate per protocol.
- Recovery-safe value or metadata predicate.
- Interference/dependency relation for EPaxos and SwiftPaxos.

## Pitfalls
Do not collapse command identity, instance identity, and value identity into one type unless the protocol actually does so.
