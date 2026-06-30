# Adopt-Commit Abstraction

Fast paths can be seen as commit when all early evidence matches, and adopt/recover when evidence is mixed. This is only an abstraction: each paper has protocol-specific metadata that must be preserved.

[[Atlas]] refines the abstraction: early evidence need not match exactly, but it must be recoverable. Its commit predicate `union_Q dep = union^f_Q dep` is an explicit recoverability test for the dependency union.
