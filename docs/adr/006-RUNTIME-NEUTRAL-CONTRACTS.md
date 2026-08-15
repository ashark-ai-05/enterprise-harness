# ADR-006: Runtime-Neutral Core Contracts

Status: proposed

## Decision

Do not fork DeepSeek Harness and do not make Cordis/dsh profiles the enterprise control-plane contract. Define language-neutral pack, capability, workflow, policy and event contracts owned by `enterprise-harness`. Expose them through a centrally managed remote MCP/HTTPS service consumed by already-approved agents and automation clients. DeepSeek Harness remains research input only.

## Options considered

1. **Fork dsh.** Fastest path to a complete user-facing agent, but inherits a large fast-moving developer-preview monorepo, upstream merge burden and an in-process plugin trust model.
2. **Enterprise dsh bundle only.** Reuses the UI, session loop, tools and telemetry with less drift, but makes Cordis configuration and dsh lifecycle the organization-wide platform contract.
3. **Runtime-neutral contracts with adapters.** More contract design up front, but supports dsh, existing Copilot/Amp-style clients, CI and future runtimes without duplicating EIL or Observability.

Choose option 3. Corporate installation restrictions reject options 1 and 2 and also reject a local dsh reference adapter.

## Consequences

The organization avoids a runtime replacement, retains existing approved agent investments and adds no corporate endpoint software. The managed service must provide orchestration, identity delegation and durable state itself rather than relying on dsh internals. Conformance tests become essential so approved-client adapters do not diverge.
