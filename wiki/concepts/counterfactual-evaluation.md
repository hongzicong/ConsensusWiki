# counterfactual-evaluation

[[counterfactual-evaluation]] compares observed system performance with an estimated performance under an alternative configuration that is evaluated in parallel but does not control correctness.

In [[Avicenna]], the real leader controls the executable log while the next leader independently shadow-commits sampled commands. Because the shadow-log order can differ, Avicenna never executes it. Instead it reconstructs shadow end-to-end latency as:

```text
ℓ̂shadow_e2e(c) = ℓshadow_commit(c) + ℓshadow_exec(c)
```

The client measures `ℓshadow_commit(c)` from proposal send to `ShadowCommitted`. The shadow leader measures `ℓshadow_exec(c)` from learning the real-log commit to finishing execution of that same command in the real log. Both are durations, so synchronized clocks are unnecessary.

Replicas compare paired real and shadow samples over a common window:

```text
F_T = A(R_T) - (1 + τ)A(G_T) - β
```

and rotate when `F_T > 0`. A separate commit-latency signal detects ordering-path slowdowns earlier; end-to-end feedback remains necessary for post-commit execution slowdowns.

The method is a performance trigger, not safety evidence. A noisy aggregator may cause an unnecessary rotation, but log-merging rules and quorum intersection still determine which commands survive.

## Modeling pitfalls

- Do not execute or compare independently ordered shadow logs directly; queueing differences would confound the result.
- Pair real and shadow measurements for the same command and insert/remove them together.
- Use the same time window for both populations under nonstationary workloads.
- Separate client-visible gray-failure detection from internal heartbeat health.
- Treat `A`, `T`, `τ`, and `β` as policy parameters, not consensus assumptions.

## Related pages

[[Avicenna]], [[Avicenna-2026]], [[slowdown-tolerance]], [[latency]], [[leader]], [[recovery]]
