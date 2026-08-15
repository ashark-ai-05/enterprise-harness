# Deployment Options

The product contracts are stable across both options: knowledge/tool pack separation, filter-not-grant, real caller identity, typed workflows, policy admission and Observability evidence.

## Resolution (2026-08-15, product owner)

> "I can create software inhouse. EIL seems to be okay."

This changes the shape of the constraint, not just answers the pending question — worth stating precisely because the group (including this repo's own ADR-010, now superseded) had generalized from one data point further than the evidence supported.

**What we had assumed:** the corp-laptop rejection of `deepseek-harness` meant *any new endpoint artifact* needs the same slow, uncertain approval — so the safe default was to avoid shipping any new artifact at all (Architecture B, everything server-side behind an already-approved client).

**What actually failed:** `dsh` specifically — an external package fetched and executed at runtime (`npx @deepseek-ai/dsh web`), unreviewed by anyone in this org, starting a local listening service. That is a materially different risk profile from **internally built, reviewed software**, which the product owner confirms has a real approval path. The binding distinction was never "new vs. existing," it was "unreviewed third party vs. reviewed in-house" — and this repo conflated the two under deadline pressure from a single incident.

**Decision: adopt Architecture A as the default**, on the product owner's direct confirmation that current EIL execution is workable on the target laptop and that in-house software has an approval path. Do not wait for a separate, more formal signal before starting Phase 0 — the falsification trial (scoped packs vs. whole-corpus EIL) needs no new software at all (Opus, PR #2) and should start now regardless of how the rest of this resolves.

Architecture B is not discarded — it remains the correct target the moment either (a) tool packs need real central governance, or (b) more than one developer needs a shared, centrally revocable index, which was always going to require EIL's per-user HTTP identity work regardless of the endpoint-approval question. B is a scale answer now, not a risk-avoidance default.

## Two things still worth a direct answer, with recommended defaults if none arrives

1. **Is "EIL seems to be okay" a documented sign-off, or the product owner's own working assessment?** Recommended default: proceed under Architecture A now — do not block Phase 0 on paperwork — but get a lightweight written confirmation (an email or an internal ticket, not a review board) in parallel, specifically covering the pack-resolver addition described below, not just EIL as it already runs. The reason to bother: "it happened to work" was also true of `dsh` general software in this org right up until someone tried to install it — a five-minute written confirmation is cheap insurance against redoing this analysis later.
2. **Does "software inhouse" cover centrally-hosted services too, or only endpoint/client code?** Recommended default: assume it covers both, using whatever review track already applies to EIL's and Observability's own infrastructure — there's no evidence of a narrower scope, and Architecture B's gateway/workers will need this answered eventually regardless of A's outcome.

Both are non-blocking by design: proceed on the recommended defaults, revise if either answer comes back differently.

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

## Decision rule (superseded by the 2026-08-15 resolution above)

~~Choose A only from explicit approval evidence, never from the observation that EIL has run somewhere. Otherwise choose B.~~ This was calibrated to the wrong distinction (new-vs-existing rather than unreviewed-third-party-vs-in-house) — see the resolution above. Current rule: **build A now**; move to B when tool packs need central governance or more than one developer needs a shared index, not as a hedge against endpoint-approval risk that turned out not to bind this way. Do not build both pilots at once. If A proves pack value, its contracts and measurements feed B later without making the local process the enterprise control plane.

