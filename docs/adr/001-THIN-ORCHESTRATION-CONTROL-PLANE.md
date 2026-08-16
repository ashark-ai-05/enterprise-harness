# ADR-001: Thin Orchestration Control Plane

Status: deferred pending Gate 3 in `docs/DECISION_MEMO_2026-08-16.md`

## Decision

Do not build a separate harness control plane yet. First prove information packs as a narrow EIL feature and measure them through Observability.

If Gate 3 earns a shared service, the harness owns planning, capability/policy admission, run state, approval bindings, execution coordination and evidence emission. EIL owns knowledge ingestion/retrieval; Observability owns durable evidence and outcome attribution; source systems own business truth.

## Consequences

Clear ownership avoids duplicated indexes and analytics, but requires stable cross-product contracts and coordinated versioning.

