---
id: ADR-NNNN
title: "{concise decision title}"
status: Assessment          # Assessment | Review | Approved | Rejected | Suspended | Superseded
area: [Ecosystem]           # Language | Standard Library | Packaging | Infrastructure | Ecosystem
authors: []
related: []                 # [ADR-000N], with the relation explained in prose
discussions:                # links to issues, discussions, or meeting notes that inform the decision
---

<!--
To author a record, copy this directory to decisions/NNNN-short-slug/, fill in the front matter and
the sections below, and add a row to the register in the repository README. The Trade-off Matrix and
Assessments sections are optional; omit them for a decision made by reasoning rather than
measurement, and carry the reasoning in Decision. The full process is defined in ADR-0001.
-->

# ADR-NNNN: {Title}

## Summary

<!-- The decision to be made and, once decided, the outcome. One paragraph, written to stand alone
for a reader arriving without prior context. -->

## Context

<!-- What forces this decision now: background, current state, and why the status quo is
insufficient. State what is out of scope, pointing to a related record where relevant. -->

## Criteria

<!-- The criteria that decide this, drawn from criteria.md. List only the ones that materially
apply, and note any decision-specific weighting. These become the columns of the decision matrix. -->

## Options

<!-- One subsection per candidate option, described neutrally with its pros and cons. Keep option
labels stable so the matrix can refer to them. -->

### Option A: {name}

### Option B: {name}

## Trade-off Matrix

<!-- Options as rows, criteria as columns. Each cell is a rating that cites a finding, or TBD with
a reference to the assessment that will resolve it, e.g. TBD -> ADR-NNNN-A0n. A value of must-pass
means an option is eliminated if it fails the cell. No cell is filled by guesswork. -->

| Criterion | Option A | Option B |
|-----------|----------|----------|
|           |          |          |

## Assessments

<!-- The open questions that resolve the matrix, each with an exit criterion and a file under
assessments/. A disqualifying assessment is a must-pass check whose failure eliminates an option,
worked first because it prunes the option space cheapest. A scoring assessment is a comparative
check that fills a cell to weigh the survivors. -->

| ID | Question | Method | Kind | Status | Exit Criterion |
|----|----------|--------|------|--------|----------------|
|    |          |        |      |        |                |

## Decision

<!-- Filled when the status reaches Approved or Rejected. The chosen option and the reasoning,
referring to the decisive cells of the matrix, stating what evidence decided it. -->

## Consequences

<!-- What the decision commits us to, positive and negative, including the backward compatibility
and migration path for existing workflows and packages, and any follow-up records this generates. -->

## References

<!-- Prior art, external canonical sources, and links to source via full commit-hash URLs. -->

## Design Meetings

<!-- Links to the discussions and meetings where this record was worked. -->
