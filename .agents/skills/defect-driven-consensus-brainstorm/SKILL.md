---
name: defect-driven-consensus-brainstorm
description: Defect-driven brainstorming for ConsensusWiki. Use when asked to reread the wiki, identify recurring weaknesses, hidden assumptions, design bottlenecks, paradigm limits, or propose new SMR/consensus protocol ideas, including prompts like "brainstorm new consensus ideas", "generate protocol candidates", "find common weaknesses and propose ideas", or "update new-protocol-ideas.md".
---

# Defect-Driven Consensus Brainstorm

## Purpose

Generate consensus protocol ideas by finding recurring limitations, hidden assumptions, and structural tradeoffs in the existing wiki. Treat new ideas as responses to mechanism-level bottlenecks, not arbitrary combinations of prior protocols.

## Core Workflow

1. Read `wiki/index.md`, then read the protocol catalog and directly relevant comparison, proof-note, and open-question pages.
2. Build an evidence matrix for recurring limitations, design tensions, or negative space across protocols.
3. Group limitations into clusters and infer the shared paradigm or hidden assumption behind each cluster.
4. State each bottleneck causally: "Because protocols do X, they pay cost Y under condition Z."
5. Propose candidate paradigm shifts that change a concrete technical dimension: quorum shape, commit rule, recovery rule, leader role, dependency handling, metadata, timing assumption, failure model, reconfiguration mechanism, or proof obligation.
6. Derive protocol candidates from the shifted paradigm.
7. Red-team serious candidates for safety, novelty, liveness, recovery ambiguity, metadata cost, and modeling risks.
8. Rank the surviving candidates.
9. For top candidates, propose the next proof, model, counterexample search, or evaluation artifact.

## Grounding Rules

- Ground protocol claims in wiki pages or raw papers.
- Mark unsupported or speculative claims as `Idea`, `Hypothesis`, `Inference`, `TODO`, or `Unclear`.
- Separate sourced facts from inferred paradigm claims.
- Do not claim safety unless quorum intersection, recovery rule, and commit condition are explicit.
- Preserve source formulas exactly.
- Prefer concise synthesis inside each section, but keep the deep brainstorm structure.

## Idea Quality Check

A strong candidate should usually:

- target a limitation seen in multiple protocols,
- explain that limitation as a consequence of a shared assumption,
- change a concrete technical dimension rather than only tuning a parameter,
- preserve or explicitly replace the relevant safety invariant,
- expose a clear proof obligation, counterexample, or evaluation question,
- avoid being merely "Protocol A + Protocol B".

## Optional Red-Team Questions

For serious candidates, ask only the relevant questions:

- Can incompatible decisions collect apparently valid evidence?
- Can recovery observe ambiguous evidence with no safe choice?
- Does liveness rely on hidden synchrony, clocks, leaders, or workload assumptions?
- Does the design move cost into metadata, dependency tracking, certificates, logs, or garbage collection?
- Is the novelty only a renamed mechanism from an existing protocol?
- Which invariant is missing, weakened, or circular?

## Wiki Updates

Update `wiki/open-questions/new-protocol-ideas.md` when the user asks for wiki updates or when the brainstorm is clearly intended as persistent research notes. Update `wiki/index.md` only if indexed pages change meaningfully.
