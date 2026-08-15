# ADR-006: Runtime-Neutral Core Contracts

Status: confirmed by evidence (amended 2026-08-15)

## Decision

Do not fork DeepSeek Harness and do not make Cordis/dsh profiles the enterprise control-plane contract. Define language-neutral pack, capability, workflow, policy and event contracts owned by `enterprise-harness`. Existing agents consume the same harness through MCP or APIs.

## Options considered

1. **Fork dsh.** Fastest path to a complete user-facing agent, but inherits a large fast-moving developer-preview monorepo, upstream merge burden and an in-process plugin trust model.
2. **Enterprise dsh bundle only.** Reuses the UI, session loop, tools and telemetry with less drift, but makes Cordis configuration and dsh lifecycle the organization-wide platform contract.
3. **Runtime-neutral contracts with adapters.** More contract design up front, but supports dsh, existing Copilot/Amp-style clients, CI and future runtimes without duplicating EIL or Observability.

Choose option 3.

## Amendment (2026-08-15): option 2 is foreclosed, not deferred

This ADR originally left option 2 open — "use option 2 as the first reference integration if phase 0 proves the dependency and corporate installation model acceptable" — as something Phase 0 would test. It doesn't need testing: the product owner reported directly that `dsh` (`npx @deepseek-ai/dsh web`) could not be installed on their corp laptop under existing unapproved-software policy. That is the exact installation model option 2 requires — running dsh's own packaged app/session loop, not just consuming its contracts. **Option 2 is closed by direct evidence, not by risk analysis.** No phase-0 task should re-open it; do not spend review time re-validating an assumption that already failed once, in production, on the actual target machine.

This also sharpens why option 3 is not merely "safer" but the only option that was ever going to be installable at all: contracts + adapters mean the harness's own artifact can be distributed the same way EIL already is — cloned source run under an interpreter already on the machine, no packaged app, no `npx` fetch-and-run. See ADR-010 for what that constrains going forward, and the open question it raises about whether that installation shape is itself confirmed for the target laptop or only assumed from the agents' build environment.

## Consequences

The organization avoids a premature runtime replacement and retains existing agent investments. The first pilot must build or adapt a small amount of orchestration plumbing rather than relying on dsh internals — this is no longer a hedge against a hypothetical, it is the only path that was going to clear procurement. Conformance tests become essential so adapters do not diverge.

