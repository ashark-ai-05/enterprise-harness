# ADR-004: Typed Resumable Workflow Graph

Status: proposed

## Decision

Represent admitted work as a versioned typed DAG with explicit capabilities, dependencies, budgets, approvals, evidence, retry and compensation semantics. Models may propose graphs but cannot admit them.

## Consequences

Runs become inspectable and resumable. Some open-ended agent behavior is constrained; bounded dynamic expansion supports justified cases.

