# validity

[[PigPaxos]] keeps ordinary Multi-Paxos validity: relays may aggregate acknowledgements, but they do not invent commands or choose values.

Validity/nontriviality means chosen values or executed commands originate from proposed/client-submitted values.

[[Rabia]] deliberately weakens ordinary validity. Weak-MVC may decide either a client request or `⊥`; `⊥` means the slot was forfeited, and pending requests are retried in later slots.

## Related pages
[[PigPaxos]], [[Rabia]], [[agreement]], [[recovery]], [[quorum]]
