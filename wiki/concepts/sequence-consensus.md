# sequence-consensus

Sequence consensus decides a strictly growing log rather than independent values. Different decided logs may have different lengths, but they must be prefix-comparable and each server's decisions must grow monotonically.

[[OmniPaxos]] states:

```text
SC1. Validity: If a server decides on a log L then L only contains proposed commands.
SC2. Uniform Agreement: For any two servers that decided logs L and L' respectively then one is the prefix of the other.
SC3. Integrity: If a server decides on a log L and later decides on L' then L is a strict prefix of L'.
```

Sequence Paxos adapts Paxos P2 so every higher-numbered chosen sequence has the earlier chosen sequence as a prefix. A Prepare majority reveals the highest-numbered accepted sequence; the new leader adopts it and synchronizes followers before appending more commands.

FIFO delivery permits a suffix implementation: accepting and deciding index `i` also covers the preceding prefix. Formal models should nevertheless state safety over whole sequences, because suffix transfer is an optimization of the evidence flow rather than a weaker property.

Source: [[Omni-Paxos-2023]], §§4-4.2 and Appendix A.

## Related pages

[[OmniPaxos]], [[agreement]], [[validity]], [[quorum]], [[recovery]], [[quorum-intersection]], [[SMR]]
