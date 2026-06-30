---
name: defect-driven-consensus-brainstorm
description: Defect-driven and paradigm-shift brainstorming for ConsensusWiki. Use when asked to reread the wiki, find recurring weaknesses, shared assumptions, hidden paradigms, or design tensions in existing SMR/consensus protocols, and derive new protocol candidates, especially prompts like "brainstorm new consensus ideas", "generate 100 SMR protocol candidates", "find common weaknesses and propose ideas", "identify the paradigm bottleneck", or "update new-protocol-ideas.md".
---

# Defect Driven Consensus Brainstorm

## Overview

Generate protocol ideas by first mapping the negative space of existing work: recurring limitations, structural tradeoffs, hidden assumptions, and underexplored design dimensions. Treat strong ideas as responses to a shared bottleneck created by an existing design paradigm, not as arbitrary feature combinations. Do not jump directly to idea generation.

## Workflow

1. Read `wiki/index.md`, `wiki/comparisons/protocol-catalog.md`, relevant dimension/family comparison pages, proof notes, and existing open questions.
2. Build a concise evidence matrix before generating ideas.
3. Extract recurring limitations, tradeoffs, or unsolved tensions across protocols.
4. Group those limitations into design-opportunity clusters.
5. For each cluster, infer the shared paradigm or hidden assumption that makes the limitation recur.
6. State the bottleneck as a mechanism-level causal claim: "Because protocols do X, they pay cost Y under condition Z."
7. Propose a candidate paradigm shift that changes the assumption, responsibility split, evidence structure, metadata meaning, quorum shape, recovery rule, or proof obligation.
8. Derive protocol candidates from the shifted paradigm.
9. Red-team candidates for safety, novelty, and modeling failures.
10. Rank candidates and identify the most researchable ones.
11. For top candidates, propose the next proof, model, or evaluation artifact.
12. Update `wiki/open-questions/new-protocol-ideas.md` when the user asks for wiki updates or when the brainstorm is intended as persistent research notes.
13. Update `wiki/index.md` if the idea page changes meaningfully.

## Research-Question Pattern

Use this pattern when framing a new direction:

```text
Existing protocols A/B/C optimize goal X with different mechanisms.
They nevertheless share paradigm P or assumption Z.
This causes recurring bottleneck B under condition C.
The proposed paradigm P' changes technical dimension D while preserving invariant I.
This should enable advantage N, but creates proof obligation or risk R.
```

This pattern is inspired by problematization, anomaly-driven research, and design-science artifact framing. Use it as a practical heuristic, not as a citation substitute.

## Evidence Matrix

Before making a broad claim, record the evidence that supports it:

```markdown
| Protocol | Observed limitation | Shared assumption | Source page | Confidence |
|---|---|---|---|---|
| [[ProtocolName]] | | | | High/Medium/Low |
```

Use `Confidence = Low` when the claim is inferred from sparse notes, when the wiki has TODOs, or when the raw paper has not been checked. Do not use low-confidence rows as the sole basis for a major paradigm claim.

## Paradigm Diagnosis

Classify each shared assumption by type:

* mechanism assumption: common protocol structure such as quorum shape, leader role, dependency rule, or recovery evidence,
* proof assumption: invariant or intersection argument that constrains the design space,
* workload assumption: expected conflict rate, locality, skew, batching, or command commutativity,
* operational assumption: deployment, latency, failure, clock, reconfiguration, or geo-replication condition,
* modeling assumption: abstraction that hides implementation cost, metadata growth, rollback, or partial failures.

Prefer concrete diagnoses like "fast-path safety evidence is collected before enough recovery information is stable" over vague labels like "leaderless protocols are complex."

## Limitation Clusters

For each cluster, record:

* name,
* shared weakness or design tension,
* protocols exhibiting it,
* shared paradigm or hidden assumption,
* why the limitation appears structural rather than accidental,
* causal bottleneck mechanism,
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
* paradigm or assumption being challenged,
* new paradigm or reframing,
* core mechanism,
* expected advantages,
* main risks or proof obligations,
* key design dimensions changed,
* invariant that should remain true,
* why this is not just a combination of prior work.

Reject superficial combinations like "Protocol A + Protocol B". A valid idea must alter at least one concrete technical dimension and explain why that alteration attacks the shared bottleneck. Useful changes include:

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

## Red-Team Pass

For each serious candidate, try to break it before ranking it:

* novelty attack: is this only a renamed mechanism from a related protocol?
* quorum attack: can two incompatible decisions obtain apparently valid evidence?
* recovery attack: can recovery see ambiguous evidence with no safe selection rule?
* liveness attack: does progress depend on a hidden synchrony, leader, clock, or workload assumption?
* metadata attack: does the design move cost into dependency sets, certificates, logs, or garbage collection?
* implementation attack: does it require unrealistic failure detection, atomic broadcast, perfect clocks, or global coordination?
* proof attack: which invariant is missing, weakened, or circular?

Mark candidates as `Reject`, `Weak`, `Promising`, or `Top candidate`.

## Ranking Rubric

Rank surviving ideas with a compact table:

```markdown
| Idea | Novelty | Structural fit | Proof tractability | Evaluation clarity | Risk | Overall |
|---|---:|---:|---:|---:|---:|---:|
| Idea name | 1-5 | 1-5 | 1-5 | 1-5 | 1-5 | 1-5 |
```

Use `Risk = 5` for high risk. A top idea should usually have high novelty, strong structural fit, clear evaluation, and a proof obligation that is difficult but not completely opaque.

## Next-Step Artifact

For each top candidate, state one concrete next step:

* proof sketch to attempt,
* minimal counterexample to search for,
* toy model or TLA+/Rocq model to build,
* experiment workload or simulator change,
* paper section or theorem this could become.

## Contribution Test

Before keeping an idea, check:

* Does it identify at least two or three existing protocols with a shared limitation?
* Does it explain the shared limitation as a consequence of a common assumption or paradigm?
* Does it change a concrete technical dimension rather than merely tuning a parameter?
* Does it preserve or explicitly replace the safety invariant used by the related protocols?
* Does it introduce a new proof obligation, evaluation question, or impossibility risk worth studying?
* Can it be falsified by a small counterexample, workload, quorum-intersection failure, or recovery scenario?
* Does it survive the red-team pass well enough to deserve ranking?

## Grounding Rules

* Ground limitations and related-protocol claims in cited wiki pages.
* Do not invent paper claims that are not supported by the wiki or raw PDFs.
* Mark speculative parts as `Idea`, `Hypothesis`, `TODO`, or `Unclear`.
* Do not claim safety unless quorum intersections, recovery rule, and commit condition are stated.
* Keep formulas exactly as stated in source pages or papers.
* Separate sourced facts from inferred paradigm claims. Label inferred claims as `Inference` when writing persistent notes.

## Output Format

Use this shape for large brainstorms:

```markdown
## Evidence Matrix

| Protocol | Observed limitation | Shared assumption | Source page | Confidence |
|---|---|---|---|---|
| [[ProtocolName]] | | | | |

## Limitation Clusters

### Cluster N: Name

**Shared weakness:** ...
**Protocols exhibiting it:** ...
**Shared paradigm / hidden assumption:** ...
**Assumption type:** Mechanism/Proof/Workload/Operational/Modeling
**Why structural:** ...
**Causal bottleneck:** ...
**Design opportunity:** ...

## Protocol Candidates

### Idea N: Name

**Limitation targeted:** ...
**Closest related protocols:** ...
**Paradigm challenged:** ...
**New paradigm:** ...
**Core mechanism:** ...
**Expected advantages:** ...
**Main risks / proof obligations:** ...
**Key design dimensions changed:** ...
**Invariant to preserve:** ...
**Why not just a combination:** ...
**Red-team result:** Reject/Weak/Promising/Top candidate

## Candidate Ranking

| Idea | Novelty | Structural fit | Proof tractability | Evaluation clarity | Risk | Overall |
|---|---:|---:|---:|---:|---:|---:|
| Idea name | | | | | | |

## Next-Step Artifacts

### Idea N

**Next step:** ...
**Counterexample to search for:** ...
**Proof/model/evaluation artifact:** ...
```
