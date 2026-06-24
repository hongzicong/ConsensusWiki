# AGENTS.md

## Project

This repository is `ConsensusWiki`, a markdown knowledge base for distributed consensus papers.

The user will put original PDF papers under `raw/`. Your job is to read those papers and maintain a structured wiki under `wiki/`.

The wiki should help with:

* understanding consensus papers,
* extracting protocol mechanisms,
* comparing protocols by technical dimensions,
* identifying quorum, fast-path, recovery, and proof rules,
* preparing Rocq/Coq/TLA+ models,
* collecting ideas for new consensus protocols.

## Important rules

* Never modify files under `raw/`.
* Treat PDFs in `raw/` as the source of truth.
* Do not hallucinate paper details.
* If something is unclear, mark it as `TODO` or `Unclear`.
* Preserve formulas exactly as stated in the paper.
* Do not assume two protocols are identical because their message flow looks similar.
* Do not transfer proof ideas between protocols unless the assumptions justify it.
* Prefer concise, structured markdown.
* Use wikilinks such as `[[FastPaxos]]`, `[[quorum]]`, `[[recoverability]]`.

## Directory structure

Use this structure:

```text
raw/
  paper.pdf

wiki/
  index.md

  papers/
    PaperName-Year.md

  protocols/
    ProtocolName.md

  concepts/
    quorum.md
    fast-path.md
    slow-path.md
    recovery.md
    leader.md
    conflict.md
    dependency.md
    failure-model.md

  properties/
    agreement.md
    validity.md
    liveness.md
    recoverability.md
    linearizability.md

  comparisons/
    protocol-catalog.md

    by-dimension/
      quorum-systems.md
      fast-paths.md
      leader-roles.md
      recovery-rules.md
      commit-rules.md
      conflict-handling.md
      proof-techniques.md
      fault-models.md
      timing-assumptions.md

    by-family/
      paxos-family.md
      fast-consensus.md
      leaderless-protocols.md
      bft-consensus.md
      dag-based-consensus.md

    case-studies/
      FastPaxos-EPaxos-SwiftPaxos.md

  proof-notes/
    quorum-intersection.md
    adopt-commit-abstraction.md
    rocq-modeling-notes.md
    tla-modeling-notes.md

  open-questions/
    new-protocol-ideas.md
    unresolved-confusions.md
```

Create missing files or directories when needed.

## Main task: ingest a paper

When the user asks you to ingest a paper from `raw/`, process one paper at a time unless the user explicitly asks for batch processing.

For each paper:

1. Identify metadata:

   * title,
   * authors,
   * year,
   * venue if available,
   * source PDF path,
   * protocol name,
   * main contribution.

2. Extract protocol details:

   * system model,
   * fault model,
   * timing assumptions,
   * roles,
   * message types,
   * local state,
   * normal path,
   * fast path,
   * slow path,
   * recovery path,
   * commit condition,
   * quorum formulas,
   * conflict handling,
   * metadata such as ballots, dependencies, sequence numbers, timestamps.

3. Extract proof details:

   * safety properties,
   * liveness assumptions,
   * main lemmas,
   * key invariants,
   * quorum intersection arguments,
   * recovery safety argument,
   * modeling pitfalls.

4. Update the wiki:

   * create or update `wiki/papers/PaperName-Year.md`,
   * create or update relevant `wiki/protocols/` pages,
   * create or update relevant `wiki/concepts/` pages,
   * create or update relevant `wiki/properties/` pages,
   * update relevant comparison pages,
   * update relevant proof notes,
   * update `wiki/index.md`.

## Paper page template

Use this template for paper pages:

```markdown
---
type: paper
title:
authors:
year:
venue:
source:
protocols:
tags:
status: ingested
---

# Paper Title

## One-sentence summary

## Why this paper matters

## System model

## Fault model

## Timing assumptions

## Main idea

## Protocol roles

## Message types

## Local state

## Normal path

## Fast path

## Slow path

## Recovery path

## Commit rule

## Quorum system

## Conflict handling

## Safety argument

## Liveness argument

## Key proof ideas

## Important formulas

## Relationship to other protocols

## Modeling notes

### Rocq/Coq modeling notes

### TLA+ modeling notes

## Limitations

## Open questions

## Related pages
```

## Protocol page template

Use this template for protocol pages:

```markdown
---
type: protocol
name:
family:
papers:
tags:
---

# ProtocolName

## Short description

## Problem solved

## System model

## Fault model

## Timing assumptions

## Roles

## Message types

## Local state

## Normal path

## Fast path

## Slow path

## Recovery

## Commit condition

## Quorum requirement

## Safety intuition

## Liveness intuition

## Strengths

## Weaknesses

## Differences from related protocols

## Modeling notes

## Open questions

## Sources
```

## Comparison system

Do not create pairwise comparison pages by default.

Pairwise comparisons do not scale. Prefer:

1. a global protocol catalog,
2. dimension-based comparison pages,
3. family-based comparison pages,
4. a few high-value case studies.

Only create direct A-vs-B comparison pages when:

* the user explicitly asks,
* the protocols are commonly confused,
* the distinction is subtle and important,
* the comparison is useful for formal modeling or new protocol design.

## Protocol catalog

Maintain `wiki/comparisons/protocol-catalog.md`.

Use this table:

```markdown
# Protocol Catalog

| Protocol | Family | Fault model | Timing model | Leader role | Fast path? | Quorum style | Conflict handling | Recovery style | Main metadata | Primary paper |
|---|---|---|---|---|---|---|---|---|---|---|
| [[ProtocolName]] | | | | | | | | | | |
```

Keep this file concise.

## Dimension-based comparisons

Dimension pages are the default comparison mechanism.

Examples:

* `wiki/comparisons/by-dimension/quorum-systems.md`
* `wiki/comparisons/by-dimension/fast-paths.md`
* `wiki/comparisons/by-dimension/recovery-rules.md`
* `wiki/comparisons/by-dimension/leader-roles.md`
* `wiki/comparisons/by-dimension/proof-techniques.md`

Use this template:

```markdown
---
type: comparison-dimension
dimension:
protocols:
tags:
---

# Dimension Name

## What this dimension means

## Why it matters

## Comparison table

| Protocol | Mechanism | Assumption | Safety relevance | Liveness relevance | Modeling note | Source |
|---|---|---|---|---|---|---|
| [[ProtocolName]] | | | | | | |

## Main patterns

## Important exceptions

## Common pitfalls

## Relevance to new protocol design

## Open questions

## Related pages
```

When ingesting a new paper, update only directly relevant dimension pages.

## Family-based comparisons

Family pages summarize related protocols.

Examples:

* Paxos family,
* fast consensus,
* leaderless protocols,
* BFT consensus,
* DAG-based consensus.

Use this template:

```markdown
---
type: comparison-family
family:
protocols:
tags:
---

# Family Name

## Family overview

## Shared assumptions

## Shared mechanism

## Main variants

## Protocol table

| Protocol | Main idea | What it changes | Strength | Weakness | Source |
|---|---|---|---|---|---|
| [[ProtocolName]] | | | | | |

## Evolution of ideas

## Reusable proof ideas

## Modeling implications

## Open questions

## Related pages
```

## Consensus-specific extraction rules

For every protocol, pay special attention to the following.

### Quorum system

Always record:

* total replicas `n`,
* fault tolerance `f`,
* fast quorum size,
* classic quorum size,
* recovery quorum size,
* whether the quorum must include a leader,
* whether quorums are fixed or flexible,
* required intersection properties.

Do not silently rewrite quorum formulas.

### Fast path

Always identify:

* who proposes,
* who accepts or pre-accepts,
* who sends acknowledgements,
* who is allowed to commit,
* what metadata must match,
* whether a leader is bypassed,
* whether commands must be non-conflicting,
* what triggers fallback.

### Recovery

Always identify:

* who can recover,
* what evidence is collected,
* from which quorum,
* how a safe value is selected,
* what invariant recovery preserves.

### Formal modeling

When useful, extract:

* abstract state,
* message types,
* local transition function,
* commit condition,
* quorum assumptions,
* desired invariants,
* likely proof obligations,
* known pitfalls.

Distinguish between:

* the real paper protocol,
* a simplified abstraction,
* a Rocq/Coq model,
* a TLA+ model.

## Query workflow

When the user asks a question:

1. Read `wiki/index.md`.
2. Find relevant pages.
3. Read those pages.
4. Inspect raw PDFs only if the wiki is insufficient.
5. Answer clearly and precisely.
6. Update the wiki when the answer contains reusable synthesis or when the user asks you to update it.

Prefer this answer structure:

```markdown
## Short answer

## Assumptions

## Mechanism

## Quorum / safety reasoning

## Modeling implication

## Caveats
```

## Lint workflow

When asked to check the wiki, look for:

* orphan pages,
* duplicate concepts,
* inconsistent quorum formulas,
* missing sources,
* stale claims,
* protocol pages missing commit rules,
* paper pages missing system models,
* unresolved TODOs,
* pairwise comparisons that should be replaced by dimension pages.

Fix simple issues directly. Record ambiguous issues in `wiki/open-questions/unresolved-confusions.md`.

## Index maintenance

Always update `wiki/index.md`.

Use this structure:

```markdown
# ConsensusWiki Index

## Papers

## Protocols

## Concepts

## Properties

## Comparisons

## Proof notes

## Open questions

## Recently updated
```

Each entry should include a wikilink and a one-line summary.

## Validation

After editing the wiki:

* Check that all created markdown files are under `wiki/`.
* Check that no files under `raw/` were modified.
* Check that `wiki/index.md` was updated.
* Check that important claims have sources or are marked as uncertain.
* Check that quorum formulas and commit rules are explicit.

## Default behavior

When asked to ingest a paper, perform the ingest workflow.

When asked to compare protocols, use protocol catalog, dimension pages, and family pages before creating pairwise comparisons.

When asked about a new protocol idea, relate it to existing mechanisms, proof notes, and open questions.

When asked to prepare material for Rocq/Coq or TLA+, focus on state, messages, transition rules, quorum assumptions, invariants, and proof obligations.
