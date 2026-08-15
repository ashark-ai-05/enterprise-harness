# ADR-008: Content-Free Resolution Locks

Status: proposed

## Decision

Resolve a knowledge pack into ordered `(resource_id, content_digest, source_cursor)` entries plus selector provenance and EIL index generation. Store no source bytes and no reusable authorization decision. Every read is re-authorized by EIL for the real caller.

Reuse EIL's existing coverage and freshness contract. Add only the pack-specific partial-view signal needed to prevent a partial empty result from rendering as a complete negative answer.

## Consequences

Runs have reproducible evidence identity without laundering content or authority. Offline use requires a separately named and governed snapshot export. Revocation latency remains an EIL SLO rather than a harness cache property.

