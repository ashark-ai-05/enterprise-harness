# ADR-001: Thin Orchestration Control Plane

Status: proposed

## Decision

The harness owns planning, capability/policy admission, run state, approvals, execution coordination and evidence emission. EIL owns knowledge ingestion/retrieval; Observability owns durable evidence and outcome attribution; source systems own business truth.

## Consequences

Clear ownership avoids duplicated indexes and analytics, but requires stable cross-product contracts and coordinated versioning.

