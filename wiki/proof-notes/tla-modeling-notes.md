# TLA+ Modeling Notes

[[FastPaxos-2006]] and [[Pando-2020]] include TLA+-style specifications in the extracted text. Keep quorum constraints as assumptions and test small configurations for invariant violations.

For [[Mencius-2008]], start with `n = 3`, an ownership function `owner(i) = i mod n`, and in-order commit. Add `SKIP` piggybacking and out-of-order commit only after the base model preserves a single learned sequence.
