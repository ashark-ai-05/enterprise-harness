# ADR-006: Runtime-Neutral Core Contracts

Status: proposed

## Decision

Do not fork DeepSeek Harness and do not make Cordis/dsh profiles the enterprise control-plane contract. Define language-neutral pack, capability, workflow, policy and event contracts owned by `enterprise-harness`. Validate them first through a dsh bundle/adapter and one headless runner. Existing agents may consume the same harness through MCP or APIs.

## Options considered

1. **Fork dsh.** Fastest path to a complete user-facing agent, but inherits a large fast-moving developer-preview monorepo, upstream merge burden and an in-process plugin trust model.
2. **Enterprise dsh bundle only.** Reuses the UI, session loop, tools and telemetry with less drift, but makes Cordis configuration and dsh lifecycle the organization-wide platform contract.
3. **Runtime-neutral contracts with adapters.** More contract design up front, but supports dsh, existing Copilot/Amp-style clients, CI and future runtimes without duplicating EIL or Observability.

Choose option 3. Use option 2 as the first reference integration if phase 0 proves the dependency and corporate installation model acceptable.

## Consequences

The organization avoids a premature runtime replacement and retains existing agent investments. The first pilot must build or adapt a small amount of orchestration plumbing rather than relying entirely on dsh internals. Conformance tests become essential so adapters do not diverge.

