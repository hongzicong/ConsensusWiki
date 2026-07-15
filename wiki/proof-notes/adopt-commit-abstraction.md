# Adopt-Commit Abstraction

Fast paths can be seen as commit when all early evidence matches, and adopt/recover when evidence is mixed. This is only an abstraction: each paper has protocol-specific metadata that must be preserved.

[[Atlas]] refines the abstraction: early evidence need not match exactly, but it must be recoverable. Its commit predicate `union_Q dep = union^f_Q dep` is an explicit recoverability test for the dependency union.

[[Copilot]] exposes a limit of a count-only adopt/commit abstraction. A recovery majority may contain enough fast-accepted replies to be consistent with a prior fast commit but not enough to prove one directly. The safe action depends on the first possibly incompatible entry in the other pilot's log.

[[Jetpack]] adds a second dimension to adopt/commit: view and order. A count of `ceil(f/2) + 1` recovery reports identifies commands that may have fast-committed in one view, but safe adoption also requires accepting one recovery set and committing it as a stability marker before any conflicting new-view work.
