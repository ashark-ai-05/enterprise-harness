# ADR-003: Governed Plugin Architecture

Status: deferred pending Gate 3 in `docs/DECISION_MEMO_2026-08-16.md`

## Decision

The information-pack pilot uses static, Git-reviewed manifest configuration only: no executable plugins, dynamic loading, registry, or endpoint installation.

If Gate 3 earns governed execution, use typed, signed capability plugins inspired by DeepSeek Harness, but keep identity, policy, credential brokering, isolation and audit in a non-replaceable kernel. I/O plugins run out of process according to trust tier.

## Consequences

This sacrifices unrestricted local extensibility for enterprise safety, conformance and revocation. Pure transforms may remain in process.

