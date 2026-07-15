# conflict

A [[conflict]] means concurrent values or commands cannot both be accepted/executed without ordering. Fast Paxos collisions and EPaxos/Atlas/SwiftPaxos command conflicts are related but not identical.

In [[GPaxos]], conflict means incompatible c-struct evidence rather than merely concurrent proposals. Non-interfering commands can appear in compatible command histories and still be learned on the fast path; interfering commands may produce incompatible histories that require recovery.

[[CURP]] treats non-commutativity among unsynced operations as the conflict condition. Witnesses reject non-commuting records, and the master syncs to backups before replying when a new operation depends on an unsynced one.

[[Copilot]] does not use application commutativity. Its conflict is an incompatible pair of cross-log dependencies that would leave two entries unordered relative to one another. Replicas reject the initial dependency and suggest a later prefix; the regular path may create a deterministic cycle that all replicas order using fixed pilot priority.

[[Jetpack]] delegates `Conflict(c1, c2)` to the application and requires it to report whenever command order can change state or response. A shim checks each fast command against all in-flight commands from both paths; any detected conflict falls back to host ordering.

[[WPaxos]] partitions conflict by object. Commands on different objects have independent owners/logs and need no mutual order; same-object commands share one ballot/slot sequence. Concurrent attempts to steal the same object are leadership conflicts resolved by higher unique ballots and randomized backoff.

## Related pages
[[FastPaxos]], [[GPaxos]], [[EPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[CURP]], [[Copilot]], [[Jetpack]], [[WPaxos]], [[object-stealing]], [[command-structure]]
