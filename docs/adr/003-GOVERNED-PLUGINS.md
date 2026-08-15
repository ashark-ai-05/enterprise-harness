# ADR-003: Governed Plugin Architecture

Status: proposed

## Decision

Use typed, signed capability plugins inspired by DeepSeek Harness, but keep identity, policy, credential brokering, isolation and audit in a non-replaceable kernel. I/O plugins run out of process according to trust tier.

## Consequences

This sacrifices unrestricted local extensibility for enterprise safety, conformance and revocation. Pure transforms may remain in process.

