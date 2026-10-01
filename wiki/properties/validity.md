# validity

[[PigPaxos]] keeps ordinary Multi-Paxos validity: relays may aggregate acknowledgements, but they do not invent commands or choose values.

Validity/nontriviality means chosen values or executed commands originate from proposed/client-submitted values.

[[Rabia]] deliberately weakens ordinary validity. Weak-MVC may decide either a client request or `⊥`; `⊥` means the slot was forfeited, and pending requests are retried in later slots.

[[OmniPaxos]] states Sequence Consensus SC1: if a server decides a log, it contains only proposed commands. The appendix argues that `log` and `buffer` receive commands only from clients and the FIFO link abstraction does not invent commands.

[[WPaxos]] calls the property non-triviality: every committed command is part of a sequence of client-proposed commands. Ownership transfer changes the leader and ballot but does not authorize invented application values.

## KCensus: two meanings of validity

[[KCensus]] has ordinary proposal validity: an adopted or committed value originated in a `Propose` call (Theorem A.21). A `⊥` result from the candidate search means no candidate survived; the proposer then adopts its own real input, not `⊥`. Separately, a *valid requirement* means `|R.keys()| > f`; that resilience check is not itself the consensus validity property ([[KCensus-2026]], Algorithms 1–2, Appendix A).

## Related pages
[[PigPaxos]], [[Rabia]], [[OmniPaxos]], [[WPaxos]], [[sequence-consensus]], [[agreement]], [[recovery]], [[quorum]]
