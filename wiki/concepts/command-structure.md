# command-structure

A command structure, or c-struct, is the generalized value learned by [[GPaxos]]. It is built from `bottom` by appending commands, with a prefix relation `<=` and compatibility relation used to express when two learned histories can coexist.

In ordinary sequence consensus, two learned values are compatible only when one is a prefix of the other. In generalized consensus, two c-structs are compatible when they have a common upper bound, allowing non-interfering concurrent commands to remain unordered while still supporting [[agreement]].

## GPaxos requirements
- Every c-struct is constructible from a sequence of proposed commands.
- The prefix relation is a partial order.
- Finite non-empty sets have a glb, and compatible finite sets have a lub.
- Learners advance monotonically by taking lubs of chosen compatible c-structs.

## Modeling notes
Do not replace c-struct compatibility with equality. The point of [[GPaxos]] is that learners may safely learn different but compatible extensions.

## Related pages
[[GPaxos]], [[Generalized-Paxos-2005]], [[conflict]], [[dependency]], [[agreement]], [[recoverability]]
