---
title: "Enterprise Harness — Tech Stack"
tags: [harness, tech-stack, deepseek-harness, cordis, mcp]
status: draft
created: 2026-08-15
---

# Tech Stack

## 1. The load-bearing decision: relationship to `deepseek-harness`

Per Codex's finding (thread, 2026-08-15, after cloning the repo rather than
reading only the README): `deepseek-harness` is ~45 TypeScript packages on
**Cordis**, a plugin framework with typed events and *reversible effects*.
"Everything is a plugin" is literal — model adapter, tool registry, session
log, and the agent loop itself are all plugins, composed at boot from
*profiles* stacking *bundles*. It already ships a durable session-event log
with the invariant **model-visible means logged**, a scoped tool registry
with a guarded execution pipeline, `ctx.skills`, `mcp-client`, subagents,
compaction, sandboxing, credentials, and an OTel telemetry seam. It has *no*
retrieval layer, *no* corpus, *no* ingestion, *no* identity/ACL model beyond
`anonymous-user-id`, and *no* pack concept.

That means the two missing halves (knowledge + observability) are the
*entire* org gap, and `dsh` already solved the runtime problem this document
would otherwise have to solve from scratch. Three real options, not a
foregone conclusion:

| Option | What it means | For | Against |
|---|---|---|---|
| **A — Fork/vendor `dsh`** | Take the Cordis runtime, add an EIL bundle + an observability bundle | Don't rebuild session logs, tool guarding, sandboxing, compaction — all genuinely hard, already correct here | 45-package monorepo, **developer preview, explicit compatibility-breaking-change warnings** (README). Forking a moving target this size is a large, ongoing maintenance bet for infra you don't control the roadmap of |
| **B — Depend on `dsh` as a library, write two bundles** | Add `enterprise-eil-bundle` + `enterprise-observability-bundle` as Cordis plugins against `dsh` as a pinned dependency, no fork | Smallest surface area; upstream owns the runtime bugs; matches Cordis's own "everything is a plugin" model for exactly the two things that are actually missing | Still coupled to an unstable upstream API; "developer preview" means the bundle interface itself may move under you |
| **C — Runtime-neutral: MCP-only, no `dsh` dependency at all** | The harness is *just* the pack registry + identity/policy layer from `docs/ARCHITECTURE.md`, exposed as its own MCP server (mirroring exactly how EIL itself is consumed — `claude mcp add`). Any agent CLI, `dsh`-based or not, mounts it the same way | Zero dependency on an unstable upstream; works with Claude Code, Codex CLI, Amp, Copilot CLI, and `dsh` identically, today; smallest possible thing to build and to get wrong | Doesn't get `dsh`'s session log, tool guarding, sandboxing "for free" — but the harness doesn't need those, the *agent CLI* already provides them (`docs/PRODUCT.md` §4, "not another coding-agent CLI") |

**Recommendation to test in review: Option C.** The product decision in
`docs/PRODUCT.md` §4 — that the harness is infrastructure other CLIs mount,
not a CLI itself — already answers this. If the harness isn't the run loop,
it doesn't need `dsh`'s run loop, and taking a dependency on a 45-package
developer-preview monorepo to get an MCP-mount layer is a large bet for
capabilities the design doesn't use. Options A/B become live again only if
the product answer to Codex's open question 1 ("is this the agent people use
day to day, or the substrate underneath") turns out to be the former — which
`docs/PRODUCT.md` §4 explicitly argues against. This is the single decision
most likely to be wrong; flagging it as the first thing to confirm, not
build around.

## 2. Language and runtime

**TypeScript / Node 22+, pnpm** — not a new choice, an inherited one. Both
EIL and the observability layer are already this stack; MCP's TypeScript SDK
is the reference implementation; EIL's own README shows the exact mounting
pattern (`import { REGISTRY, callTool } from "eil/tools"`) the harness needs
to consume. Introducing a second language would mean reimplementing MCP
client/server plumbing that already exists correctly in this stack, for no
stated benefit.

## 3. Storage

**No new store.** If Stage 1's pack registry (`docs/USAGE_AND_SCALE.md`)
outgrows a Git-tracked manifest, it's a table in the same Postgres/PGlite
instance EIL and observability already standardized on — not a new
approval, not a new backup/ops surface. A pack registry is low-write,
low-cardinality (hundreds to low-thousands of packs, not millions of rows);
nothing about its access pattern argues for a different engine.

## 4. Transport

**MCP over stdio at Stage 0–1 (matches EIL today), HTTP MCP at Stage 2** —
identical progression to EIL's own stated roadmap ("per-user tokens + HTTP
MCP transport — phase 2, the kube rollout gate"). The harness should not get
ahead of EIL here: there's no value in the harness supporting server-side
multi-tenant transport before its main knowledge backend does.

## 5. Deployment posture

**Laptop-first, no Docker, no admin rights required at Stage 0–1** — same
ethos as EIL's PGlite path. This is a product requirement as much as a
technical one: if the harness requires infra provisioning to try, it will be
evaluated by nobody. Stage 2's service deployment (Kubernetes, per EIL's own
"kube rollout gate" language) is deferred until Stage 0–1 prove the pack
abstraction earns its complexity (`docs/USAGE_AND_SCALE.md` Stage 0 exit
criterion).

## 6. Open constraint to confirm before any stack choice is final

Codex raised (thread, 2026-08-15): does the observability work's
**no-arbitrary-installs / corporate-proxy constraint** bind here? If
laptops can't `npm install` arbitrary packages or reach the public registry
directly, that's a hard blocker on Option A/B (`dsh` is a large external
dependency tree) and a soft constraint even on Option C (MCP SDK, pnpm
itself). This needs an answer before `docs/DECISIONS.md` D2 can be marked
resolved rather than assumed.
