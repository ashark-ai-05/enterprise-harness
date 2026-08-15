# Decision Review After In-House Software Clarification

Status: recommended defaults pending product-owner confirmation. No implementation is authorized.

## What the new information changes

The organization can create software in-house and EIL appears acceptable on the corporate laptop. This makes the EIL-local pilot the lowest-approval, fastest-learning path. It does **not** justify building a complete harness runtime locally.

The critical challenge to the earlier design is decomposition: the repository described one product but actually contained three.

| Product | Correct initial home | Why | Gate |
|---|---|---|---|
| P1: knowledge-pack selection, resolution and search | EIL's existing read-safe MCP boundary | pack search is scoped retrieval; EIL already owns ACL, coverage, freshness and citations | pack-value experiment |
| P2: governed tool execution and workflows | future managed Enterprise Harness service | side effects, approvals, durable runs, credentials and sandboxes are a different trust boundary | demonstrated tool-pack demand |
| P3: measurement and attribution | Enterprise Observability | it already owns receipts, lineage and outcomes | ships with P1 measurement |

`enterprise-harness` remains the design, contracts and governance repository across P1–P3. It does not imply that P1 needs a separately installed binary.

## Recommended defaults

1. **Product posture:** substrate under existing approved agents, not a new agent UI or run loop.
2. **Pilot distribution:** reviewed first-party changes inside EIL's current source/lockfile/stdio model; no dynamic package fetch, daemon, desktop app or listening endpoint.
3. **Pilot scope:** knowledge packs over Confluence, Jira and code only; read/search/fetch/cite; no tool packs, model loop, source mutation, sandbox or approval engine.
4. **Pilot tenancy:** one developer and their local EIL corpus. Ambient identity is acceptable only inside this single-principal boundary.
5. **Pack form:** content-free manifest and resolution lock. A pack is a relevance filter, never an authority grant.
6. **Measurement:** three retrieval arms from first use—whole corpus, equivalent inline filters, and pack—plus run-level outcomes. A → B values scoping; B → C values the pack abstraction.
7. **Enterprise target:** remote managed service only after P1 earns continuation and EIL per-user HTTP identity is proven.

## Decisions challenged or narrowed

- **“Build a harness first” — rejected.** P1 is an EIL feature plus measurement, not a new runtime.
- **“Everything is a plugin” — rejected as a security model.** Knowledge and tool packs remain separate types and promotion paths.
- **“Central service is the safest first step” — rejected for the pilot if EIL-local extension is approved.** It puts unbuilt shared identity and service operations before product validation.
- **“A local success proves enterprise scale” — rejected.** It validates pack usefulness only. It says nothing about multi-user identity, central governance, service reliability or mutations.
- **“Pack precision improvement is enough” — rejected.** Narrowing can increase precision while hiding relevant evidence. Recall loss and outcomes are first-class guardrails.
- **“Pack registry is part of MVP” — rejected.** Start with reviewed manifests in Git. A registry is earned by reuse and curation demand.
- **“Tool packs naturally follow knowledge packs” — rejected.** P2 starts only after teams request side-effecting capability and governance ownership exists.

## Recommended pilot acceptance gates

- zero ACL or cross-scope disclosure failures in adversarial tests;
- 100% of answer claims that depend on enterprise knowledge carry source/version citations;
- no material recall degradation versus whole-corpus retrieval on an independently authored evaluation set;
- measurable reduction in irrelevant retrieved context or session setup time;
- partial/stale coverage produces abstention or an explicit limitation, never a complete negative claim;
- content-free locks reproduce the evidence identity used by a run;
- pack telemetry reaches Observability without prompt/source-body capture by default;
- pilot users choose scoped packs for eligible tasks and report that the boundary is understandable.
- in a multi-developer follow-on, packs are reused by people who did not author them; otherwise treat packs as personal saved filters, not an enterprise platform.

Numeric recall/context thresholds should be set after a baseline sample rather than chosen to flatter the design. Zero ACL failures and citation completeness are non-negotiable.

## Questions awaiting confirmation

1. Substrate under existing agents, or a new day-to-day agent? Recommended: substrate.
2. Are reviewed EIL-repo changes under its current install/launch model allowed on the target laptop? Recommended assumption: yes, verify through the normal internal path.
3. Knowledge-only first pilot? Recommended: yes.
4. Single-developer local pilot before shared service? Recommended: yes.
