# ADR-004: Typed Resumable Workflow Graph

Status: deferred pending Gate 3 in `docs/DECISION_MEMO_2026-08-16.md`

## Decision

The pilot uses one fixed, read-only change-planning workflow and a correlation/evidence contract; it does not introduce a generic workflow language.

If Gate 3 earns governed execution, represent admitted work as a versioned typed DAG with explicit capabilities, dependencies, budgets, approvals, evidence, retry and compensation semantics. Models may propose graphs but cannot admit them.

## Consequences

Runs become inspectable and resumable. Some open-ended agent behavior is constrained; bounded dynamic expansion supports justified cases.

