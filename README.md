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

## How to Injest New Papers?

```text
Please ingest the paper of XXX.
```

## How to Brainstorm New Ideas?

```text
Reread the wiki and brainstorm 100 defect-driven SMR protocol candidates.
```
