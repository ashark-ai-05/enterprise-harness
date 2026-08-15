# ADR-002: Manifest-First Information Packs

Status: proposed

## Decision

Default packs are signed manifests referencing EIL resources/scopes. They do not copy source bodies or create a second search index. Offline snapshots are a separate exceptional profile.

## Consequences

Runtime authorization and freshness remain correct. Fully offline use needs explicit export, encryption, TTL and acknowledgement that revocation cannot be instantaneous.

