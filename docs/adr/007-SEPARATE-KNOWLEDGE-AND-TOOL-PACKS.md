# ADR-007: Separate Knowledge and Tool Packs

Status: proposed

## Decision

Model `Pack` as a tagged union of `KnowledgePack` and `ToolPack`, not as one flat plugin collection.

- Knowledge packs are EIL-backed search/fetch filters. They inherit EIL's per-resource fail-closed ACL, may only narrow access and can be composed within policy.
- Tool packs expose invokable MCP/tool capabilities. They declare side effects, credentials and egress and require trust-tier-specific publication, mount and invocation controls.

The harness propagates the real caller identity to EIL. It never uses a broad service identity and treats pack scoping as a substitute ACL.

## Consequences

Manifest schemas, registries, user experience and approval flows must preserve the discriminator. This adds visible complexity but prevents read-safe knowledge composition from normalizing side-effecting tool authority.
