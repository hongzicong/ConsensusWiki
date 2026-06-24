---
type: protocol
name: EPaxos*
family: paxos / leaderless SMR
papers: [[[Making-Democracy-Work-2025]]]
tags: [consensus, leaderless, dependencies, recovery]
---

# EPaxos*

## Short description
EPaxos* is a corrected and simplified EPaxos-style SMR protocol that commits dependency sets for commands and uses validation-based recovery.

## Problem solved
Leaderless state-machine replication with fast execution of conflict-free commands while tolerating crash failures and providing a rigorous recovery proof.

## System model
Partially synchronous reliable-message system with `n >= 3` processes. Commands are unique and may commute or conflict.

## Fault model
Crash failures. The protocol is parameterized by `f` total failures for resilience and `e <= f` failures for the fast-path guarantee.

## Timing assumptions
Safety does not rely on synchrony. The `e`-fast guarantee is stated for synchronous runs in which up to `e` processes crash at the beginning and messages advance one round per `Delta`.

## Roles
Clients submit commands. Any process can be the initial coordinator for a command. Per-command leader detector `Omega[id]` eventually nominates a recovery coordinator. All processes store command state, participate in quorums, validate recovery, and execute committed commands.

## Message types
`PreAccept`, `PreAcceptOK`, `Accept`, `AcceptOK`, `Commit`, `Recover`, `RecoverOK`, `Validate`, `ValidateOK`, `Waiting`, and `TryRecover`.

## Local state
Per command identifier: `cmd`, `initCmd`, `dep`, `initDep`, `phase`, `bal`, and `abal`. `phase` ranges over `initial`, `preaccepted`, `accepted`, and `committed`. `executed` records commands already applied to the local state machine.

## Normal path
The initial coordinator broadcasts a `PreAccept` containing the command and initial dependencies. Replicas add conflicting commands they know about and return `PreAcceptOK` with their proposed dependency set. The coordinator waits for at least `n - f` replies.

## Fast path
If the `PreAcceptOK` quorum has size at least `n - e` and every returned dependency set equals `initDep[id]`, the coordinator broadcasts `Commit` at ballot `0`.

## Slow path
If fast commit is unavailable, the coordinator unions the returned dependencies, sends `Accept`, waits for `AcceptOK` from at least `n - f` processes, and then commits.

## Recovery
Recovery starts with Paxos-like `Recover` / `RecoverOK` evidence. Committed or accepted evidence with maximal `abal` is recovered directly. Possible fast-path decisions are recovered only after validation checks whether omitted conflicting commands invalidate the proposed dependency set. Recovery may commit `Nop` when the original payload cannot be safely recovered.

## Commit condition
Fast: matching initial dependency set from a fast quorum `|Q| >= n - e`.

Slow/recovery: `AcceptOK` from a quorum `|Q| >= n - f` for the selected `(cmd, dep)`.

## Quorum requirement
Optimized EPaxos* is proved when:

```text
n >= max{2e + f - 1, 2f + 1}
```

The simpler baseline protocol requires:

```text
n >= max{2e + f + 1, 2f + 1}
```

Here `f` is the overall crash-failure resilience target, while `e <= f` is the fast-path failure budget. A fast quorum has size `n - e` so that, in an `e`-faulty synchronous fast run, the initial coordinator can still collect a complete fast quorum from all surviving processes. Slow and recovery quorums have size `n - f` because they target the full `f`-failure case.

The extreme choice `e = f` is allowed by the parameter relation, but it makes the optimized process bound:

```text
n >= max{3f - 1, 2f + 1}
```

For `f >= 2`, this means `n >= 3f - 1`. With the minimal `n = 3f - 1`, fast and slow/recovery quorums both have size `n - f = 2f - 1`. Thus `e = f` minimizes `n - e` for a fixed `n`, but usually requires more processes than the original EPaxos-style `n = 2f + 1` configuration.

## Safety intuition
Agreement ensures a command identifier commits only one payload/dependency set. Visibility ensures every pair of conflicting committed non-`Nop` commands has at least one dependency edge between them. Execution over committed dependency SCCs then preserves consistent order for conflicting commands.

## Liveness intuition
Eventually stable per-command recovery coordinators keep retrying until commands commit. Validation waits for potentially invalidating commands, and `Waiting` messages let blocked recoveries abort safely rather than deadlock.

## Strengths
- Cleaner recovery than original EPaxos.
- Rigorous proof for non-thrifty and thrifty variants.
- Matches the lower bound `n >= max{2e + f - 1, 2f + 1}` in the optimized protocol.
- Uses two ballot variables, avoiding the original EPaxos single-ballot bug.

## Weaknesses
- Recovery remains complex and has several case distinctions.
- Liveness assumes finitely many submitted commands.
- Commands recovered as `Nop` require resubmission.
- The paper omits the original EPaxos early-return optimization for no-return-value commands.

## Differences from related protocols
Compared with [[EPaxos]], EPaxos* removes sequence numbers from the core dependency-graph presentation, uses `bal` and `abal`, and replaces tentative pre-accept recovery with validation. Compared with [[FastPaxos]], the fast evidence is not just a value vote but dependency metadata that must preserve visibility among conflicting commands.

## Modeling notes
For formal models, separate:
- commit evidence for one command identifier,
- dependency visibility among conflicting committed commands,
- execution over committed dependency SCCs,
- recovery validation and `Waiting` cycle-breaking.

## Open questions
- TODO: Build a small Rocq/TLA+ abstraction of the validation predicate and prove it preserves visibility.
- TODO: Decide how to represent the thrifty variant separately, if needed.

## Sources
- [[Making-Democracy-Work-2025]]
