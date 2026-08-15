---
title: "Enterprise Harness — Architecture"
tags: [harness, architecture, mcp, eil, observability]
status: draft
created: 2026-08-15
---

# Architecture

This describes components and data flow, not source layout — there is no
code yet (see `README.md`). Corrected against Codex's finding (thread,
2026-08-15) that `deepseek-harness` is Cordis-based with typed events and
*reversible effects*, composed from profiles stacking bundles, and already
ships a durable session-event log, scoped tool registry with a guarded
execution pipeline, `ctx.skills`, `mcp-client`, subagents, compaction,
sandboxing, and an OTel telemetry seam — but no retrieval, no corpus, no
ACL beyond `anonymous-user-id`, and no pack concept. That reframes this
document: the harness is not a runtime built from scratch, it is the
**two missing bundles** (a knowledge-pack provider, a governed pack
registry) plus the **identity and observability wiring** that a
Cordis-shaped runtime doesn't have an opinion on today. See
`docs/TECH_STACK.md` for whether that means forking, bundling, or staying
runtime-neutral.

## 1. The four planes

```
┌─────────────────────────────────────────────────────────────┐
│  Agent CLI (Claude Code / Codex / Amp / Copilot CLI)          │
│  — the actual run loop the user talks to; unmodified          │
└───────────────────────────┬────────────────────────────────┘
                             │ MCP (stdio or HTTP)
┌───────────────────────────▼────────────────────────────────┐
│  ENTERPRISE HARNESS  (this repo)                              │
│  • Pack registry client   • Session/mount manager             │
│  • Identity propagation   • Trust-tier policy (knowledge/tool) │
└──────────┬───────────────────────────────────┬───────────────┘
           │ MCP (EIL's REGISTRY/callTool)      │ MCP (arbitrary tool servers)
┌──────────▼──────────────┐         ┌───────────▼───────────────┐
│  KNOWLEDGE PLANE (EIL)    │         │  TOOL PLANE (mounted MCP)   │
│  search_docs, get_doc,    │         │  deploy, page, ticket-write,│
│  search_code, expand, …   │         │  arbitrary org/third-party  │
└───────────────────────────┘         └─────────────────────────────┘
           │                                       │
           └───────────────┬───────────────────────┘
                            │ events (mount, search, call, session lifecycle)
                ┌───────────▼────────────┐
                │  OBSERVABILITY PLANE     │
                │  (enterprise-ai-obs)     │
                └─────────────────────────┘
```

The harness is deliberately thin at the top (it does not replace the agent
CLI's run loop) and thin at the bottom (it does not replace EIL's retrieval
or reimplement MCP). Its entire job is the middle two boxes: deciding what's
mounted, propagating identity so the two backing planes enforce ACL/policy
correctly, and making every mount/search/call an observability event by
construction.

## 2. The info-pack lifecycle

```
ingest (EIL, personal creds) → corpus → PACK DEFINITION → publish → mount → search/call → observe
```

1. **Ingest.** Unchanged. `eil ingest confluence/jira/repo/obsidian`, on the
   ingesting user's own credentials, into the org's shared EIL corpus.
2. **Pack definition.** A named, versioned scope over that corpus (a saved
   query/filter — space, project, repo path, doc-type) *or* a mount
   reference to an MCP tool server, tagged `knowledge` or `tool`
   (`docs/PRODUCT.md` §2). Stored as a manifest, Git-tracked at Stage 1,
   registry-served at Stage 2 (`docs/USAGE_AND_SCALE.md`).
3. **Publish.** A pack becomes discoverable (`harness pack list`) once its
   owner (team lead, Stage 1) or an approval workflow (tool packs, Stage 2)
   signs off. Publishing a knowledge pack **never** grants access beyond
   what EIL's own document ACL already allows the consuming identity — a
   pack is a *filter*, not a *grant*. This is the load-bearing security
   property of the whole design; see `docs/CRITICAL_REVIEW.md` §2 for what
   happens if it's violated.
4. **Mount.** `harness pack use <name>` resolves the pack definition into
   concrete MCP tool bindings for the session — either narrowing EIL's
   `REGISTRY` to the pack's scope (knowledge packs, via a query parameter
   EIL's `callTool()` choke point already threads through) or adding the
   tool server's tools to the session's tool list (tool packs).
5. **Search / call.** The agent CLI calls tools exactly as it does today —
   the harness does not intercept or rewrite tool calls after mount, it only
   decides what's mounted. This keeps the harness out of the hot path and
   out of the business of understanding tool semantics.
6. **Observe.** Every event in steps 3–5 (publish, mount, search, call,
   session start/end) is emitted in `enterprise-ai-observability`'s existing
   schema. No new telemetry system, no content capture by default (inherits
   observability's D3 decision).

## 3. Identity propagation — the part that must not be shortcut

At Stage 0 (`docs/USAGE_AND_SCALE.md`), identity is trivial: one person, one
laptop, one EIL instance already running as them. The architectural risk
appears the moment the harness proxies calls on behalf of more than one
identity (Stage 2, server-side registry) — it would be easy, and wrong, for
the harness to hold one service credential to EIL and fan out pack-scoped
views from behind it. That collapses EIL's per-document ACL into
"anyone who can reach the harness can read anything any pack references,"
which is a strictly worse security posture than not having a harness at
all.

**Requirement, not a nice-to-have:** every EIL call the harness makes must
carry the calling session's real identity through to EIL's ACL evaluation,
the same way EIL already expects a "viewer" argument at its `callTool()`
choke point. The harness is a policy/composition layer in front of EIL's
ACL, never a delegation boundary that flattens it. This is also exactly the
gap `deepseek-harness` has today (`anonymous-user-id` per Codex's read of
its architecture) — it is not a gap the harness can inherit or defer.

## 4. Trust tiers, not a flat plugin list

Directly from `docs/PRODUCT.md` §2: knowledge packs and tool packs are not
interchangeable, so the architecture must not expose them through one
undifferentiated "plugins" surface no matter how tempting that is given
`deepseek-harness`'s "everything is a plugin" framing.

| Tier | Examples | Who can publish | Mount is... |
|---|---|---|---|
| 0 — read-only knowledge | EIL-backed packs | Team lead (Stage 1), self-serve within EIL's own ACL | Automatic, no approval — EIL's ACL is the only gate that matters |
| 1 — read-only external tool | A read-only ticket lookup, a docs-fetch MCP server | Platform team review | Logged, not gated |
| 2 — side-effecting tool | Deploy, write-to-Jira, page-oncall | Named approver per tool, org security sign-off | Gated per-session, expiring |

This table is a proposal, not a spec — see `docs/DECISIONS.md` D3 for the
open question of whether tier 2 needs per-invocation confirmation or only
per-session mount approval.

## 5. What the harness explicitly does not own

- **Ranking/retrieval logic** — EIL's, untouched.
- **The agent run loop, model calls, or tool-call guardrails within a single
  tool** — the agent CLI's (or, if adopted, `deepseek-harness`'s Cordis
  runtime) job, not the harness's.
- **Telemetry storage, cost attribution, or eval** —
  `enterprise-ai-observability`'s, untouched; the harness is a producer.
- **Credential storage for MCP tool servers** — reuse EIL's existing
  OS-keychain pattern rather than inventing a second secrets store.
