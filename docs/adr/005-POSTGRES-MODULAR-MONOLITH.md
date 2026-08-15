# ADR-005: PostgreSQL-First Modular Monolith

Status: proposed

## Decision

Begin with TypeScript process roles backed by PostgreSQL and approved object storage. Use transactional outbox and leased jobs. Add a broker or split services only from measured scale, isolation or ownership needs.

## Consequences

The pilot has fewer distributed failure modes and lower operating cost. Interfaces must still preserve future separation of API, scheduler, workers and event export.

