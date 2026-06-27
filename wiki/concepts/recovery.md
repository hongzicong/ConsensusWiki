# recovery

[[recovery]] selects a safe value or state after failures, collisions, or ballot changes. It must preserve all values/commands that could already have been committed.

[[EPaxosStar|EPaxos*]] is a useful recovery case study because possible fast-path evidence is not enough by itself: the new coordinator validates whether recovering a candidate dependency set would violate visibility with already committed or potentially committing conflicting commands.

## Related pages
[[FastPaxos]], [[EPaxos]], [[EPaxosStar|EPaxos*]], [[Mencius]], [[SwiftPaxos]], [[Pando]]

