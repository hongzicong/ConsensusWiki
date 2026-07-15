# view-change-hazard

The [[view-change-hazard]] occurs when a fast path relies on promises made by the original protocol's proposer set, but a later view installs proposers that neither made nor know those promises. A client may already have observed a fast commit, so ordinary leader recovery can become unsafe in two independent ways.

[[Jetpack-2026]] states the structural recovery requirements:

- **R1 - Recoverability:** recover every fast-committed command from the previous normal view.
- **R2 - Ordering:** place each recovered command before every conflicting uncommitted command in the new view's log.

Recovering only the command value satisfies R1 but can still violate R2. Carrying a stale uncommitted entry ahead of the recovered fast command can change its speculative result or reverse real-time order.

Jetpack answers with:

- **Principle 1:** the fast path owns an independent view, and a fast commit combines only same-view evidence from both paths.
- **Principle 2:** new proposers recover the last normal view's fast commands and commit the recovery set/no-op stability marker before activating other work.

Host compatibility also matters:

- **PR 1:** conflicting commands proposed in order by one proposer retain that log/execution order.
- **PR 2:** proposer receipt order determines proposal order.

## Modeling pitfalls

- Do not merge fast acknowledgements from different views.
- Do not treat original-path promises as durable unless the protocol explicitly persists them.
- Do not recover stale uncommitted entries ahead of fast-committed work.
- Do not resume a new view before the recovery marker commits.
- Do not infer execution order from proposal order unless the host satisfies `PR 1` and `PR 2`.
- Distinguish the last attempted view from the last normal view whose predecessor recovery completed.

## Related pages

[[Jetpack]], [[Jetpack-2026]], [[fast-path]], [[recovery]], [[recoverability]], [[linearizability]], [[quorum-intersection]]
