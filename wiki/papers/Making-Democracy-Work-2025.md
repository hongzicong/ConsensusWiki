---
type: paper
title: Making Democracy Work: Fixing and Simplifying Egalitarian Paxos (Extended Version)
authors: Fedor Ryabinin, Alexey Gotsman, Pierre Sutra
year: 2025
venue: OPODIS 2025 / arXiv extended version
source: raw/epaxos_plus.pdf
protocols: [EPaxosStar]
tags: [paxos, leaderless, dependencies, fast-path, recovery, lower-bound]
status: ingested
---

# Making Democracy Work: Fixing and Simplifying Egalitarian Paxos (Extended Version)

## One-sentence summary
EPaxos* is a corrected and simplified variant of [[EPaxos]] with a rigorously proved recovery protocol and an optimal `f`-resilient `e`-fast process bound.

## Why this paper matters
The paper identifies ambiguity, safety bugs, and recovery liveness bugs in original [[EPaxos]], then gives [[EPaxosStar]] as a cleaner protocol whose recovery is designed around validation rather than tentative pre-accept side effects.

## System model
The paper considers a set `Pi` of `n >= 3` processes with reliable links. Processes implement state-machine replication for deterministic services with client commands submitted through `submit(c)` and executed through `execute(c)`. Each submitted command is assumed unique. Commands may commute; non-commuting commands conflict.

## Fault model
Non-Byzantine crash failures. At most `f` processes may crash for overall resilience. The separate parameter `e <= f` is the maximum number of failures tolerated while still executing conflict-free commands fast.

## Timing assumptions
The system is partially synchronous: after GST, messages take at most `Delta`, and processes can accurately measure time, while GST is unknown. The fast-path specification uses an `E`-faulty synchronous run where processes in `E` crash at the beginning of the first round, messages sent in a round arrive at the beginning of the next round, and local computation is instantaneous.

## Main idea
EPaxos* agrees on a dependency graph over commands. Each command identifier `id` is committed with a payload `cmd[id]` and dependency set `dep[id]`; committed commands are executed by traversing committed dependency SCCs in topological order and breaking cycles by command identifier.

## Protocol roles
- Client: submits commands.
- Initial coordinator: the process receiving a command; it owns ballot `0` for the command identifier.
- New coordinator: a process nominated by per-command leader detector `Omega[id]` to recover an uncommitted command.
- Replica/process: pre-accepts, accepts, commits, validates, executes, and may trigger recovery.

## Message types
`PreAccept`, `PreAcceptOK`, `Accept`, `AcceptOK`, `Commit`, `Recover`, `RecoverOK`, `Validate`, `ValidateOK`, `Waiting`, and `TryRecover`.

## Local state
For each command identifier, processes track `cmd`, `initCmd`, `dep`, `initDep`, `phase`, `bal`, and `abal`. Phases are `initial`, `preaccepted`, `accepted`, and `committed`. The execution loop also tracks `executed`. `bal` is the current ballot joined by the process, while `abal` records the last ballot where the process accepted a slow-path proposal.

## Normal path
The initial coordinator assigns a fresh `id`, computes initial dependencies from known conflicting commands, broadcasts `PreAccept(id,c,D)`, and waits for `PreAcceptOK` replies from a quorum `Q` with `|Q| >= n - f`.

## Fast path
The coordinator commits directly at ballot `0` when `Q` is fast and unanimous on the initial dependency set:

```text
|Q| >= n - e
forall q in Q. D_q = initDep[id]
```

It then sends `Commit(0,id,cmd[id],D)` to all. In an `E`-faulty synchronous run with `|E| <= e`, a conflict-free command submitted at time `t` by process `p` is executed at `p` by `t + 2 Delta`.

## Slow path
If the fast predicate fails, the coordinator unions the dependency proposals from the `PreAcceptOK` quorum and sends `Accept(0,id,cmd[id],D)` to all. After `AcceptOK` from a quorum of size at least `n - f`, it broadcasts `Commit`.

## Recovery path
A new coordinator sends `Recover(b,id)` and waits for `RecoverOK` from a recovery quorum `Q` with `|Q| >= n - f`. It first recovers committed or accepted slow-path evidence using the maximum reported `abal`, as in Paxos Phase 1. If only pre-accepted fast-path evidence might exist, it checks whether enough recovery-quorum members pre-accepted the same initial dependencies and then runs a validation phase before proposing the recovered dependency set.

The validation phase sends `Validate(b,id,c,D)` to the recovery quorum. Recipients report committed or potentially committing conflicting commands that are not ordered with `id`. If no invalidating command exists, the coordinator accepts `(c,D)`. If an invalidating command exists, it proposes `Nop`. If only potentially invalidating commands exist, it waits until they commit; `Waiting` messages break recovery cycles safely.

## Commit rule
Fast path: a command commits at ballot `0` when the coordinator receives a fast quorum of matching `PreAcceptOK` dependency sets equal to `initDep[id]` and sends `Commit`.

Slow/recovery path: a command commits when a coordinator receives `AcceptOK` from a quorum of size at least `n - f` for the chosen `(cmd, dep)` and sends `Commit`.

## Quorum system
- Recovery/slow quorum size: at least `n - f`.
- Fast quorum size: at least `n - e`.
- Baseline EPaxos* correctness condition: `n >= max{2e + f + 1, 2f + 1}`.
- Optimized EPaxos* correctness condition: `n >= max{2e + f - 1, 2f + 1}`.
- Lower bound: any `f`-resilient `e`-fast SMR protocol requires `n >= max{2e + f - 1, 2f + 1}`.
- Original EPaxos target special case: `n = 2f + 1` and `e = ceil((f + 1)/2)`, yielding fast quorum size `n - e = f + floor((f + 1)/2)`.

## Conflict handling
Commands conflict when they do not commute. Pre-accepting a command adds known conflicting command identifiers to its dependency set. Safety requires visibility: if two distinct committed non-`Nop` commands conflict, then at least one command appears in the other's dependency set.

## Safety argument
The paper centers safety on two main invariants:
- Agreement: if a command `id` is committed at two processes, the committed payload and dependency set are identical.
- Visibility: if distinct committed non-`Nop` commands conflict, their committed dependency sets order at least one direction.

The fast path and validation-based recovery preserve these invariants. Validation avoids resurrecting a command with dependencies that would omit a conflicting command already committed without a reciprocal dependency.

## Liveness argument
Liveness is for runs with finitely many submitted commands. Per-command leader detectors eventually nominate a unique correct recovery coordinator. `Waiting` messages and the optimized recovery rules prevent recovery attempts from deadlocking on cycles of potentially invalidating commands. If a submitted command is recovered as `Nop`, the submitting process resubmits its original payload.

## Key proof ideas
- Theorem 4: any `f`-resilient `e`-fast SMR protocol requires `n >= max{2e + f - 1, 2f + 1}`.
- Lemma 9: in the baseline protocol, if a conflicting potentially invalidating command sends `Waiting`, then the recovered command did not take the fast path.
- Lemma 11: an optimized recovery branch can safely abort when the recovery quorum has exactly `|Q| - e` matching pre-accepts and the potentially invalidating command's initial coordinator is outside `Q`.
- Lemma 12: in the optimized protocol, `Waiting(id',k')` only justifies aborting when `k' > n - f - e`.
- Lemma 13: every command is eventually committed at all correct processes.

## Important formulas
```text
fast run: execute at p by t + 2 Delta
recovery/slow quorum: |Q| >= n - f
fast quorum: |Q| >= n - e
lower bound: n >= max{2e + f - 1, 2f + 1}
baseline condition: n >= max{2e + f + 1, 2f + 1}
optimized condition: n >= max{2e + f - 1, 2f + 1}
original EPaxos target: n = 2f + 1, e = ceil((f + 1)/2)
original EPaxos fast quorum: n - e = f + floor((f + 1)/2)
```

## Relationship to other protocols
EPaxos* is a corrected descendant of [[EPaxos]]. It keeps the leaderless dependency-graph idea, removes the sequence-number machinery from the presentation, and replaces original EPaxos recovery's tentative pre-accept phase with validation. It is closer to Paxos than original EPaxos in its use of two ballot variables, `bal` and `abal`.

## Modeling notes

### Rocq/Coq modeling notes
Model command identifiers separately from payloads, and make `Nop` explicit. The main proof split is instance agreement for `(cmd, dep)` and visibility for conflicting committed commands. Recovery should be modeled as evidence selection plus a validation predicate; do not model validation as a state-changing tentative pre-accept.

## Limitations
The protocol deliberately omits the original EPaxos optimization that returns early for commands without return values, because applying it directly can violate linearizability and requires extra mitigations. Liveness is stated for executions with finitely many submitted commands.

## Open questions
- TODO: Compare the paper's EPaxos bug examples against the current [[EPaxos]] page and decide whether to annotate the original page with explicit "known incorrect/ambiguous" recovery caveats.
- TODO: Formalize `f`-resilient `e`-fast SMR as a standalone proof note.

## Related pages
[[EPaxosStar]], [[EPaxos]], [[dependency]], [[conflict]], [[fast-path]], [[recovery]], [[quorum]], [[quorum-intersection]], [[leaderless-protocols]], [[proof-techniques]]
