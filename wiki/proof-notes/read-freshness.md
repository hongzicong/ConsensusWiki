# Read Freshness and Stable Results

Read optimizations need two distinct arguments: **visibility** of every earlier completed interfering write, and **stability/application** of the state actually returned. The following synthesis records obligations from the source papers; it is not a shared theorem covering arbitrary protocol compositions.

## PQL: notification or activation catch-up

[[Paxos-Quorum-Leases-2014]], §3.6, considers an earlier write `W` and later read `R` at an active holder:

1. If activation precedes `W`, a grantor majority intersects `W`'s accepting majority. The intersecting grantor's reply forces synchronous holder notification before commit, so the holder cannot read past the pending write without executing it.
2. If a grantor accepted `W` before granting, its promise's accepted-instance frontier forces catch-up through `W` before the holder serves locally. Holder acceptance itself also prevents skipping `W`.

The holder executes the read atomically under leases for the full read set. Timer containment keeps promises binding; configuration agreement alone is insufficient. These cases underpin the paper's strict-serializability argument.

## PQR: discovery then execution

[[Linearizable-Quorum-Reads-2019]], §§1, 3-4, 6, supplies the mechanism and explanatory reasoning:

1. Let `Q_W` be an earlier completed write's accepting set and `Q_R` a read discovery set. Require `Q_R ∩ Q_W ≠ ∅` for every allowed `Q_W`; a report therefore exposes at least the necessary slot frontier.
2. A maximum accepted frontier can also contain an unchosen write. Returning that data immediately is unsafe: a later read can miss it, or recovery can replace it.
3. Rinse waits for one replica to execute through the selected slot. Ordered host execution certifies a chosen applied prefix; the returned data must correspond to final executed state.
4. Clean first-round accepted/executed evidence can establish the same condition without a second round.

For unconstrained threshold sets, the derived intersection condition is `q_read + q_write > N`. The one-replica rinse count does not change this condition. Host recovery separately needs Phase 1/Phase 2 intersection.

Unclear in the short paper: complete read-result binding across higher-ballot replacement, exact multi-key/per-key snapshot refinement, and epoch/restart fencing. Do not treat this explanatory synthesis as a full formal proof of those cases.

## Bodega: ballot cases and responder coverage

[[Bodega-2026]], §4.3, splits a prior write by ballot relative to the reader's stable roster. Majority leases exclude higher-ballot commits; same-ballot commits cover all responders; lower-ballot writes are covered by majority-intersecting `thresh_p` catch-up. Its optional early accept notifications supply a distinct release rule for held reads.

## Common modeling checks

- Identify membership/epoch and the allowed set families before reducing evidence to counts.
- Keep highest accepted, chosen, locally learned committed, and locally executed state separate.
- Define how a returned read value binds to an executed state, not only a progress counter.
- Account for gaps, delayed commit messages, changed leaders, and writes concurrent with activation.
- Do not infer leader-independent write progress from a leader-bypassing read path.

## Related pages

[[PQL]], [[PQR]], [[Bodega]], [[quorum-lease]], [[rinse]], [[read-strategies]], [[quorum-intersection]], [[linearizability]], [[recoverability]], [[unresolved-confusions]]
