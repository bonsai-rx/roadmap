# Decision Criteria

Shared criteria for weighing options in an Architecture Decision Record. A record cites the subset that applies and notes any decision-specific weighting. The columns in a decision trade-off matrix are drawn from this list.

Scores are qualitative unless a record defines a measured rubric.

## Ecosystem Continuity Risk

Risk that a decision breaks existing workflows or published packages, or splits the community into incompatible camps. Lower is better. This is often the dominant criterion, given a large body of workflows and packages that cannot be recompiled or republished.

## Maintenance Burden

Ongoing effort to keep the result working over time: tracking upstream changes, native runtime updates, API drift, and user support. Lower is better. Estimated as a proportion of having a full-time person dedicated to the outcome.

## Performance

Runtime cost against the current baseline for representative workflows: per-call overhead, allocation, and memory footprint. Measured as a ratio to the baseline where a benchmark exists.

## Feature Velocity

How quickly new upstream capability becomes available to workflow authors after it appears upstream. Higher is better.

## Control

The degree to which the ecosystem controls the public surface, semantics, and release timing of the result, rather than depending on an external party. Higher is better, weighed against maintenance burden, since more control usually costs more effort.

## Effort

One-time cost to reach the outcome, against the available budget. Estimated in months assuming a full-time person is dedicated to the outcome.

## Reversibility

How cheaply the decision can be revised later. A reversible decision is a two-way door; a one-way door that is expensive to undo carries more risk and deserves more evidence before it is taken. Higher is better.

## Community Trust

Effect on user and contributor confidence and continuity, including how a transition is communicated and how conservative users are affected. Higher is better.
