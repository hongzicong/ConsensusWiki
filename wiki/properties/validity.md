# validity

[[PigPaxos]] keeps ordinary Multi-Paxos validity: relays may aggregate acknowledgements, but they do not invent commands or choose values.

Validity/nontriviality means chosen values or executed commands originate from proposed/client-submitted values.

[[Rabia]] deliberately weakens ordinary validity. Weak-MVC may decide either a client request or `⊥`; `⊥` means the slot was forfeited, and pending requests are retried in later slots.

[[OmniPaxos]] states Sequence Consensus SC1: if a server decides a log, it contains only proposed commands. The appendix argues that `log` and `buffer` receive commands only from clients and the FIFO link abstraction does not invent commands.

[[WPaxos]] calls the property non-triviality: every committed command is part of a sequence of client-proposed commands. Ownership transfer changes the leader and ballot but does not authorize invented application values.

## Related pages
[[PigPaxos]], [[Rabia]], [[OmniPaxos]], [[WPaxos]], [[sequence-consensus]], [[agreement]], [[recovery]], [[quorum]]
