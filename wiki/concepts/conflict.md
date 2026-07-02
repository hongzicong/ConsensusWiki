# conflict

A [[conflict]] means concurrent values or commands cannot both be accepted/executed without ordering. Fast Paxos collisions and EPaxos/Atlas/SwiftPaxos command conflicts are related but not identical.

In [[GPaxos]], conflict means incompatible c-struct evidence rather than merely concurrent proposals. Non-interfering commands can appear in compatible command histories and still be learned on the fast path; interfering commands may produce incompatible histories that require recovery.

[[CURP]] treats non-commutativity among unsynced operations as the conflict condition. Witnesses reject non-commuting records, and the master syncs to backups before replying when a new operation depends on an unsynced one.

## Related pages
[[FastPaxos]], [[GPaxos]], [[EPaxos]], [[Atlas]], [[SwiftPaxos]], [[Pando]], [[CURP]], [[command-structure]]
