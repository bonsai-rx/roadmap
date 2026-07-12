# Bonsai ecosystem roadmap

Architecture Decision Records for the Bonsai ecosystem: the durable record of why significant, cross-cutting decisions were made across the language, the standard library, packaging, infrastructure, and the wider community.

An ADR here records a decision and the reasoning behind it, and is distinct from a proposal. A proposal is a forward-looking specification to build something and lives in the repository it affects, for example `bonsai-rx/bonsai`. An ADR instead captures why a strategic ecosystem decision was taken, whether that means building, replacing, or removing, and may motivate or supersede several proposals across repositories.

Architecture is read broadly, spanning software, community, and process. A record can sit un-decided for months while its assessments are worked, and status tracks how far it has progressed. The process itself is defined in [ADR-0001](decisions/0001-adopt-architecture-decision-records/); the record template lives in [TEMPLATE](TEMPLATE/), and the shared decision criteria in [criteria.md](criteria.md).

## Working with Records

Records are living documents, so a record is created early, while still in `Assessment`. This keeps `main` showing which decisions are open and how far each has progressed.

A pull request is scoped to one coherent increment rather than a whole decision: creating a record with its framing and open assessments, recording a completed assessment or a status change, or creating a sub-record that a parent decision spun off. Each merges to `main` on its own, and later work branches again from the updated `main` rather than stacking pull requests on one another.

## Records

| ID | Title | Area | Status |
|----|-------|------|--------|
| [ADR-0001](decisions/0001-adopt-architecture-decision-records/) | Adopt Architecture Decision Records for the Bonsai ecosystem | Ecosystem | Approved |