# Rocq Modeling Notes

## Suggested abstractions
- Quorum family predicates.
- Message evidence predicates.
- Commit predicate per protocol.
- Recovery-safe value or metadata predicate.
- Interference/dependency relation for EPaxos and SwiftPaxos.
- Ownership function `owner : Instance -> Server` for [[Mencius]], plus a proposal restriction saying only `owner i` may propose a non-`no-op` value for instance `i`.

## Pitfalls
Do not collapse command identity, instance identity, and value identity into one type unless the protocol actually does so.


For [[Mencius]], do not encode `SKIP` as a majority quorum certificate. It is safe because of the simple-consensus value restriction. Keep chosen, learned, and committed separate, because commit can lag behind learning due to earlier unlearned instances.

