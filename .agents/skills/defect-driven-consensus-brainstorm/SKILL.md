---
name: defect-driven-consensus-brainstorm
description: Defect-driven brainstorming for ConsensusWiki. Use when asked to reread the wiki, find recurring weaknesses or design tensions in existing SMR/consensus protocols, and derive new protocol candidates, especially prompts like "brainstorm new consensus ideas", "generate 100 SMR protocol candidates", "find common weaknesses and propose ideas", or "update new-protocol-ideas.md".
---

# Defect Driven Consensus Brainstorm

## Overview

Generate protocol ideas by first mapping the negative space of existing work: recurring limitations, structural tradeoffs, and underexplored design dimensions. Do not jump directly to idea generation.

## Workflow

1. Read `wiki/index.md`, `wiki/comparisons/protocol-catalog.md`, relevant dimension/family comparison pages, proof notes, and existing open questions.
2. Extract recurring limitations, tradeoffs, or unsolved tensions across protocols.
3. Group those limitations into design-opportunity clusters.
4. Derive protocol candidates from the clusters.
5. Update `wiki/open-questions/new-protocol-ideas.md` when the user asks for wiki updates or when the brainstorm is intended as persistent research notes.
6. Update `wiki/index.md` if the idea page changes meaningfully.

## Limitation Clusters

For each cluster, record:

* name,
* shared weakness or design tension,
* protocols exhibiting it,
* why the limitation appears structural rather than accidental,
* why existing solutions are insufficient,
* underexplored design opportunity.

Common dimensions to inspect:

* quorum size vs latency,
* fast-path availability vs recovery complexity,
* leader bottlenecks vs coordination overhead,
* dependency tracking vs metadata cost,
* flexible quorums vs proof complexity,
* optimistic execution vs conflict recovery,
* partial synchrony vs responsiveness,
* WAN latency vs fault tolerance,
* BFT safety vs performance,
* reconfiguration vs quorum continuity,
* speculative execution vs rollback cost.

## Candidate Requirements

For each protocol idea, state:

* limitation targeted or negative space occupied,
* closest related protocols,
* core mechanism,
* expected advantages,
* main risks or proof obligations,
* key design dimensions changed,
* why this is not just a combination of prior work.

Reject superficial combinations like "Protocol A + Protocol B". A valid idea must alter at least one concrete technical dimension, such as:

* quorum system,
* fast-path commit rule,
* recovery rule,
* leader role,
* conflict or dependency handling,
* metadata structure,
* failure model,
* timing assumption,
* reconfiguration mechanism,
* geo-replication strategy,
* speculative execution model,
* proof technique.

## Grounding Rules

* Ground limitations and related-protocol claims in cited wiki pages.
* Do not invent paper claims that are not supported by the wiki or raw PDFs.
* Mark speculative parts as `Idea`, `Hypothesis`, `TODO`, or `Unclear`.
* Do not claim safety unless quorum intersections, recovery rule, and commit condition are stated.
* Keep formulas exactly as stated in source pages or papers.

## Output Format

Use this shape for large brainstorms:

```markdown
## Limitation Clusters

### Cluster N: Name

**Shared weakness:** ...
**Protocols exhibiting it:** ...
**Why structural:** ...
**Design opportunity:** ...

## Protocol Candidates

### Idea N: Name

**Limitation targeted:** ...
**Closest related protocols:** ...
**Core mechanism:** ...
**Expected advantages:** ...
**Main risks / proof obligations:** ...
**Key design dimensions changed:** ...
**Why not just a combination:** ...
```
