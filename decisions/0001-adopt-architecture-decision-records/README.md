---
id: ADR-0001
title: Adopt Architecture Decision Records for the Bonsai ecosystem
status: Approved
area: [Ecosystem]
authors: [glopesdev]
related: []
discussions:
---

# ADR-0001: Adopt Architecture Decision Records for the Bonsai ecosystem

## Summary

The Bonsai v3 roadmap brings a series of significant, cross-cutting decisions that span more than one repository and, in some cases, the whole community. We will adopt Architecture Decision Records (ADR), kept in this repository, as the durable record of why each such decision was made. This document defines the record format, the status lifecycle, and how records relate to the forward-looking proposals that already live in individual repositories.

## Context

Decisions on the roadmap are high-stakes and often irreversible in practice. They reach across the language, the standard library, packaging, infrastructure, and the community, and a single decision can motivate work in several repositories at once. The existing proposal process in `bonsai-rx/bonsai`, with its specification-proposal issue template, is scoped to language features and is forward-looking by design. It captures what to build specifically in the language, compiler, and editor, but not the reasoning behind a cross-cutting ecosystem direction, and it has no natural place for a decision whose outcome is to remove or replace something rather than add to it.

What we are missing is a community record of the reasoning behind community strategic decisions: the options considered, the criteria applied, the evidence gathered, and why one path was chosen over the others. Without it, this kind of cross-cutting reasoning stays scattered across issues and discussions with a narrower scope, and which may themselves be renumbered, moved, or deleted over time.

## Criteria

- Familiarity. A new contributor should recognize the format without a glossary.
- Breadth and neutrality. The format must fit a decision to build, replace, or remove, across software, community, and process.
- Low ceremony. Fields and status values should be self-explanatory and cheap to keep current.
- Self-containment. Each record should stand alone for a reader arriving without prior context.
- Separation from proposals. The decision layer should be distinct from the forward-looking proposal layer.

## Options

### Option A: Architecture Decision Records

Adopt the established ADR format, introduced by Michael Nygard, kept in this repository. Records carry a decision, the options, and the reasoning. The term is widely recognized, directionally neutral, and fits decisions across the whole ecosystem.

### Option B: An Enhancement-Proposal Series

Adopt a numbered enhancement-proposal series in the style of Python PEPs, Rust RFCs, or Kubernetes KEPs. This is well understood, but enhancement presupposes an additive, monotonic change. A decision whose outcome is to drop or replace something does not fit the frame cleanly, and the series would overlap with the existing language-proposal process rather than sit above it.

### Option C: Ad-hoc Issues and Discussions

Keep recording decisions in issues and discussion threads, as today. This has the lowest setup cost, but the reasoning stays scattered and tied to tracker state that renumbers and moves, and the pressure to keep each issue self-contained to the scope of its repository can often hide the cross-cutting concern itself, which is the main problem we want to solve.

## Decision

Adopt Option A. Architecture Decision Records are kept in this repository, one per directory under `decisions/NNNN-short-slug/`, following the template in `TEMPLATE/` and the shared criteria in `criteria.md`.

Naming conventions and glossary:

- Identifier. `ADR-NNNN`, held in the `id` field and mirrored by the directory number. The generic field name keeps the series easy to rename later without rewriting records.
- Expansion. Architecture Decision Record, using Architecture as a noun naming the subject matter. Architecture is read broadly, spanning software, community, and process.
- Status. Assessment, Review, Approved, Rejected, Suspended, or Superseded. A record starts in Assessment the moment it is opened. It moves to Review once the evidence is gathered, then to Approved or Rejected. Suspended means the decision is on hold for the moment but may return. Superseded means the decision was made but later replaced.
- Area. One or more of Language, Standard Library, Packaging, Infrastructure, and Ecosystem, where Ecosystem covers cross-cutting community and process matters not tied to a single code area.
- Relations. A `related` list which names connected records. The nature of the relation, including supersession, is stated in prose rather than in separate machine-readable fields.
- Proposal. A proposal is a forward-looking request to build something and lives in the repository it affects.
- Record. A record captures why a direction was chosen and may motivate or supersede several proposals across repositories.

A record may sit in Assessment or Review for months while its assessments are worked. This differs from the classic form, where a record is written at the moment of decision. The status field carries how far the decision has progressed.

The template for a decision record also includes a trade-off matrix, options weighed against criteria, and an assessment backlog that resolves what the decision does not yet know. A disqualifying assessment is a must-pass check whose failure removes an option. A scoring assessment is a comparative measurement that fills a cell. A survey assessment gathers facts the decision depends on, such as an inventory of existing usage, and neither removes nor scores an option. Both the matrix and the assessments are optional, and are omitted for a decision made by reasoning rather than measurement, as in this record. A record may also carry assessments without a matrix, where the open questions are about scope or feasibility rather than a choice between options.

## Consequences

Cross-cutting decisions and their reasoning are centralized in this repository, versioned alongside the rest of the roadmap and separate from the per-repository proposal process. Revising a decision is done by superseding a record rather than editing it, so the earlier reasoning stays in history.

The process adds a repository and a convention to maintain, and it depends on contributors opening a record while a decision is still open rather than after it has been settled. The template and criteria are separate files, so editing them does not require superseding this record.

## References

- Michael Nygard, Documenting Architecture Decisions, the original ADR formulation, at https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
- MADR, the Markdown Architecture Decision Records project, at https://adr.github.io and https://github.com/adr/madr
- Python PEP 1 at https://peps.python.org/pep-0001/, the Rust RFCs at https://github.com/rust-lang/rfcs, and Kubernetes KEPs at https://github.com/kubernetes/enhancements, as precedents for numbered decision and proposal records
- The specification-proposal template in `bonsai-rx/bonsai`, introduced in [bonsai-rx/bonsai#2586](https://github.com/bonsai-rx/bonsai/pull/2586), which this format complements at the ecosystem level