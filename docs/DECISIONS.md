---
title: "Enterprise Harness — Decision Log"
tags: [harness, decisions, adr]
status: draft
created: 2026-08-15
---

# Decision Log

ADR-style. Every row is a **proposal pending confirmation** — nothing here
authorizes implementation (see `README.md`). Status values: `proposed`,
`confirmed`, `rejected`, `superseded`.

| # | Question | Proposed answer | Rationale | Status |
|---|---|---|---|---|
| D1 | Does the pack abstraction earn its complexity? | Validate at Stage 0 (individual, laptop-local) before building any registry | If scoped packs don't beat "mount the whole EIL corpus" on precision or setup time, stop — see `docs/CRITICAL_REVIEW.md` §5 | proposed |
| D2 | Fork, bundle, or stay runtime-neutral re: `deepseek-harness`? | **Option C — runtime-neutral MCP server**, no `dsh` dependency | The harness is infrastructure other CLIs mount, not a run loop itself (`docs/PRODUCT.md` §4); avoids a large dependency on a developer-preview monorepo (`docs/TECH_STACK.md` §1) | proposed, blocked on the no-arbitrary-installs/corporate-proxy question Codex raised |
| D3 | Does a tier-2 (side-effecting) tool pack need per-invocation confirmation or only per-session mount approval? | Not yet decided — flagged, not assumed | Side-effecting tools are the highest-risk surface (`docs/ARCHITECTURE.md` §4, `docs/CRITICAL_REVIEW.md` §3); getting this wrong in either direction (too much friction kills adoption, too little enables an incident) | open |
| D4 | Are knowledge packs a filter or a grant? | **Filter only** — a pack can never expose a document the consuming identity couldn't already read via EIL's own ACL | This is the load-bearing security property of the whole design (`docs/ARCHITECTURE.md` §2 step 3); violating it turns the harness into an ACL bypass | proposed |
| D5 | Who enforces identity at the EIL boundary once the harness proxies calls? | The calling session's real identity, propagated end-to-end — never a shared service account | Prevents the confused-deputy failure mode (`docs/CRITICAL_REVIEW.md` §2) | proposed, contingent on agent CLIs actually forwarding caller identity through MCP (open dependency, `docs/CRITICAL_REVIEW.md` §5) |
| D6 | Does the harness need its own telemetry system? | No — emits into `enterprise-ai-observability`'s existing schema as a producer | Avoids a second observability system (`docs/PRODUCT.md` §4); inherits that layer's D13 (team-level aggregation only, never per-individual) | proposed |
| D7 | Web UI at launch? | No — MCP-native, agent/CLI-facing, matching EIL's own posture | Primary consumers are agent CLIs and services, not people clicking around (`docs/PRODUCT.md` §4); a pack-catalog UI is a plausible later addition for team leads, not a Stage-0 requirement | proposed |
| D8 | Storage for the pack registry? | Same Postgres/PGlite instance EIL and observability already use; Git-tracked manifest at Stage 1, no service until Stage 2 | Low-write, low-cardinality workload; no new approval or ops surface (`docs/TECH_STACK.md` §3) | proposed |
| D9 | Does the no-arbitrary-installs/corporate-proxy constraint from the observability work bind here? | Unknown — Codex raised it, unanswered | Directly determines whether D2's Option C is merely recommended or effectively the only viable path (`docs/TECH_STACK.md` §6) | **open — needs a human or Codex answer before D2 is final** |
| D10 | Should this live as a separate repo, or fold into `eil`? | Separate repo now, revisit if tool packs never get real adoption | The trust-tier split (knowledge vs. tool) is a real architectural boundary EIL shouldn't own, but only if tool packs actually get used (`docs/CRITICAL_REVIEW.md` §1, §5) | proposed |

## Explicitly deferred (not decided, not forgotten)

- Exact pack manifest schema (fields, versioning scheme).
- Whether pack definitions support composition (a pack referencing other
  packs) or must be flat.
- Cross-org / multi-tenant isolation if this ever serves more than one org's
  EIL instance — out of scope until Stage 2 is real.
- Billing/cost attribution for harness-mediated sessions — inherits
  whatever `enterprise-ai-observability` decides; not a new question.
