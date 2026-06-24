# ConsensusWiki

ConsensusWiki is a Markdown knowledge base for distributed consensus papers. Original PDF papers live in `raw/`, while structured notes, protocol summaries, comparisons, and proof/modeling notes live under `wiki/`.

The goal is to make consensus papers easier to understand, compare, and formalize.

## Goals

- Understand distributed consensus papers.
- Extract protocol mechanisms, message flows, quorum rules, commit rules, and recovery rules.
- Compare protocols across technical dimensions such as quorum systems, fast paths, leader roles, conflict handling, recovery, and proof techniques.
- Prepare material for Rocq/Coq, TLA+, or other formal models.
- Collect open questions and ideas for new consensus protocols.

## Repository Layout

```text
raw/
  paper.pdf

wiki/
  index.md

  papers/
  protocols/
  concepts/
  properties/
  comparisons/
  proof-notes/
  open-questions/
```

Key directories:

- `raw/` contains original PDFs and is treated as the source of truth. Do not modify files in this directory.
- `wiki/index.md` is the main entry point for the knowledge base.
- `wiki/papers/` contains notes for individual papers.
- `wiki/protocols/` contains protocol-level summaries.
- `wiki/concepts/` contains reusable consensus concepts such as `quorum`, `fast-path`, and `recovery`.
- `wiki/properties/` contains safety, liveness, recoverability, and related properties.
- `wiki/comparisons/` contains the protocol catalog, dimension-based comparisons, family-based comparisons, and selected case studies.
- `wiki/proof-notes/` contains proof and formal-modeling notes.
- `wiki/open-questions/` contains unresolved issues, unclear points, and new protocol ideas.

## Current Contents

Start from [`wiki/index.md`](wiki/index.md).

The wiki currently includes notes on:

- [[FastPaxos]]
- [[EPaxos]]
- [[EPaxosStar|EPaxos*]]
- [[Pando]]
- [[SwiftPaxos]]

Useful entry points:

- Protocol catalog: [`wiki/comparisons/protocol-catalog.md`](wiki/comparisons/protocol-catalog.md)
- Quorum systems: [`wiki/comparisons/by-dimension/quorum-systems.md`](wiki/comparisons/by-dimension/quorum-systems.md)
- Fast paths: [`wiki/comparisons/by-dimension/fast-paths.md`](wiki/comparisons/by-dimension/fast-paths.md)
- Recovery rules: [`wiki/comparisons/by-dimension/recovery-rules.md`](wiki/comparisons/by-dimension/recovery-rules.md)
- Rocq/Coq notes: [`wiki/proof-notes/rocq-modeling-notes.md`](wiki/proof-notes/rocq-modeling-notes.md)
- TLA+ notes: [`wiki/proof-notes/tla-modeling-notes.md`](wiki/proof-notes/tla-modeling-notes.md)

## Ingesting a Paper

To add a new paper:

1. Place the original PDF under `raw/`.
2. Extract metadata, protocol mechanics, proof ideas, and modeling notes from the PDF.
3. Create or update the paper page under `wiki/papers/`.
4. Create or update the protocol page under `wiki/protocols/`.
5. Update relevant concept, property, comparison, and proof-note pages.
6. Update `wiki/index.md`.

When ingesting a paper, pay special attention to:

- Total replicas `n` and fault tolerance `f`.
- Fast quorum, classic quorum, and recovery quorum sizes.
- Normal path, fast path, slow path, and recovery path.
- Commit conditions and fallback triggers.
- Conflict handling and metadata such as ballots, dependencies, sequence numbers, and timestamps.
- Safety invariants, quorum-intersection arguments, recovery safety, and modeling pitfalls.

## Writing Conventions

- Use concise, structured Markdown.
- Use wikilinks such as `[[FastPaxos]]`, `[[quorum]]`, and `[[recoverability]]`.
- Preserve formulas exactly as stated in the paper.
- Mark unclear claims as `TODO` or `Unclear`.
- Do not assume two protocols are identical because their message flows look similar.
- Do not transfer proof ideas between protocols unless the assumptions justify it.
- Prefer dimension-based comparison pages over many pairwise comparison pages.

## Maintenance Checklist

After editing the wiki, check that:

- All created Markdown files are under `wiki/`.
- No files under `raw/` were modified.
- `wiki/index.md` was updated when relevant.
- Important claims have sources or are marked as uncertain.
- Quorum formulas and commit rules are explicit.
- Ambiguous issues are recorded in `wiki/open-questions/unresolved-confusions.md`.

## Project Instructions

Detailed repository instructions are in [`AGENTS.md`](AGENTS.md).

