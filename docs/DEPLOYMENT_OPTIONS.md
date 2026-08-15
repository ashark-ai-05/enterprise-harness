# Deployment Options Pending Endpoint-Approval Evidence

The product contracts are stable across both options: knowledge/tool pack separation, filter-not-grant, real caller identity, typed workflows, policy admission and Observability evidence. The deployment critical path is not stable. Do not select an option until the product owner confirms whether EIL's current local execution is formally approved on the same corporate laptop that rejected DeepSeek Harness.

## Architecture A — approved local EIL extension

Use only if current EIL source/lockfile/stdio execution is explicitly approved and adding reviewed first-party code within that approval model is permitted.

```text
approved agent CLI
  → EIL stdio MCP process
      → pack resolver added behind existing REGISTRY/callTool boundary
      → local caller identity and existing EIL ACL
      → remote enterprise Observability receipt
```

- Pack resolution/search folds into EIL's existing tool boundary; no new agent runtime, daemon, browser UI or fetch-and-run package.
- The first falsification trial can run before shared HTTP identity exists.
- Knowledge packs only in the pilot. Tool packs, durable multi-client orchestration, central governance and sandbox execution remain server-side/later products.
- Every dependency and artifact remains pinned and reviewed. Matching EIL's technical shape is not itself approval; IT/Security must confirm the approval scope covers the addition.

Critical path: approval confirmation → small EIL pack-contract change proposal → local read-only measurement.

Risks: local state drift, weaker central revocation/configuration, duplicated per-developer catalogs, endpoint update burden, and accidental expansion from a narrow EIL extension into a hidden new runtime.

## Architecture B — managed remote harness

Use if EIL is not formally approved locally, its approval does not cover extensions, or the organization requires centrally managed enforcement.

```text
approved agent/IDE/browser
  → enterprise-authenticated remote MCP/HTTPS gateway
      → Harness catalog/policy/scheduler/workers
      → EIL with delegated real caller identity
      → Enterprise Observability
```

- No new endpoint executable or local corpus.
- Central policy, revocation, operations, durable runs and isolated execution.
- Corporate pilot is hard-blocked on EIL per-user identity over HTTP MCP and at least one approved client's authenticated remote integration.

Critical path: EIL HTTP identity → approved remote-client auth → managed read-only service → measurement.

Risks: larger infrastructure/procurement scope, service operations before product value is proven, network dependency and higher time-to-first-trial.

## Decision rule

Choose A only from explicit approval evidence, never from the observation that EIL has run somewhere. Otherwise choose B. Do not build both pilots. If A proves pack value, its contracts and measurements feed B later without making the local process the enterprise control plane.

