# Enterprise Harness

Product and architecture blueprint for a governed enterprise agent harness that composes the Enterprise Intelligence Layer (EIL), enterprise tools, models, sandboxes, approvals, and the Enterprise Observability Layer into auditable workflows.

**Status: design only. No runtime implementation is authorized.**

## Product decision

**Decision (2026-08-16): do not build a separate harness runtime yet.** Prove the information-pack plane as a narrow EIL feature and measure it through Enterprise Observability; a governed orchestration service is deferred until shared, side-effecting workflow demand earns it. See [the decision memo](docs/DECISION_MEMO_2026-08-16.md).

The **future target**, if Gate 3 is met, is a thin orchestration control plane—not another knowledge platform and not a monolithic autonomous agent.

- **EIL** owns ingestion, normalized knowledge, source ACLs, indexing, retrieval, citations, and freshness.
- **Enterprise Harness** owns workflow composition, capability discovery, policy checks, execution state, approvals, resumability, and evidence emission.
- **Enterprise Observability** owns durable run evidence, lineage, cost/token/time measurement, verified outcomes, and fleet-level analysis.
- **Systems of record** remain authoritative for Jira, Confluence, Git, CI/CD, incidents, and approvals.

```mermaid
flowchart LR
  U[User / API / IDE / CI] --> H[Enterprise Harness]
  H --> P[Policy + approval engine]
  H --> E[EIL search and fetch]
  H --> T[Tool and MCP gateway]
  H --> M[Model gateway]
  H --> S[Sandbox / job runner]
  E --> K[(Enterprise knowledge)]
  T --> R[Systems of record]
  H --> O[Enterprise Observability]
  O --> C[Cockpit / audit / outcomes]
```

## The pack model

`Pack` is a tagged union, never a flat plugin list:

- A **knowledge pack** is a signed, versioned manifest that selects governed EIL resources for a purpose. It is searched and fetched. It is not a ZIP of copied enterprise data and not an independent vector index.
- A **tool pack** selects MCP/tool capabilities. It is invoked and may cause side effects, so it follows separate trust, approval and credential paths.

Conflating the two is a security defect: EIL can fail closed per document, while a side effect cannot be undone by filtering its result.

A pack contains:

- identity, semantic version, owner, purpose, classification, tenancy and expiry;
- immutable resource references and source/version digests;
- retrieval defaults, evaluation set, compatible tools and workflow recipes;
- authorization requirements, policy labels and dependency locks;
- signatures, provenance and a reproducible build receipt.

The EIL remains the source of truth. A knowledge pack is a filter, never a grant. At runtime the harness resolves it against the real caller identity, rechecks access, freshness and policy, then requests cited snippets or full resources from EIL. Portable offline snapshots are a separate, explicitly exported pack profile with encrypted content, bounded TTL and revocation limitations.

## Product surfaces

1. **Pack registry** — discover, inspect, validate, promote, deprecate and audit packs.
2. **Workflow catalog** — approved reusable recipes such as incident triage, change planning and code remediation.
3. **Run console** — plan, approve, execute, pause, resume, cancel and inspect a run.
4. **Capability catalog** — models, MCP servers, tools, sandboxes and connectors with trust and policy metadata.
5. **Evidence view** — citations, decisions, mutations, approvals, tests and outcome links supplied by Observability.
6. **Administration** — tenant policy, quotas, identities, pack promotion, kill switches and retention.

## Documents

- [Product and organizational model](docs/PRODUCT.md)
- [Architecture and technology choices](docs/ARCHITECTURE.md)
- [Pack specification](docs/PACK_SPEC.md)
- [Pack resolution, partial views and freshness](docs/PACK_RESOLUTION.md)
- [Pack-value measurement](docs/MEASUREMENT.md)
- [Adoption, identity ceiling and product decomposition](docs/ADOPTION_AND_DECOMPOSITION.md)
- [Deployment options pending approval evidence](docs/DEPLOYMENT_OPTIONS.md)
- [Decision review and recommended defaults](docs/DECISION_REVIEW.md)
- [2026-08-16 decision memo and staged product gates](docs/DECISION_MEMO_2026-08-16.md)
- [Security and governance](docs/SECURITY_AND_GOVERNANCE.md)
- [Delivery plan and decision gates](docs/DELIVERY_PLAN.md)
- [DeepSeek Harness comparison](docs/REFERENCE_ASSESSMENT.md)
- [Architecture decision records](docs/adr/README.md)

## Corporate endpoint constraint

The corporate laptop cannot install DeepSeek Harness because it is unapproved software. DeepSeek Harness is therefore architecture research only: not a dependency, fork, bundle, adapter, reference client or fallback. Do not generalize this evidence into an unsupported claim that every new endpoint artifact is prohibited; confirm the applicable approval policy in Phase 0.

The enterprise target is a centrally managed service, but the first pilot architecture is deliberately unresolved. If EIL's exact local source/lockfile/stdio execution and extension model are formally approved on the same laptop, a narrow knowledge-pack resolver can fold into EIL's existing `REGISTRY`/`callTool()` boundary and run the falsification trial without waiting for shared HTTP identity. Otherwise the pilot is remote-only and blocked on EIL per-user identity over HTTP MCP. See [Deployment options](docs/DEPLOYMENT_OPTIONS.md).

## Recommended pilot

Start with one bounded, read-heavy workflow through an already-approved client: **Jira change request → approved Confluence and code packs → cited change plan and acceptance criteria → evidence bundle**. Add sandboxed patch/test only after read-only value and identity propagation are proven.

The pilot should use one model route, one source-control provider, read-only Jira/Confluence access, and a disposable code sandbox. Mutations remain approval-gated. Do not begin implementation until the decisions in [Delivery plan](docs/DELIVERY_PLAN.md) are approved.

The pilot topology is selected by the approval gate above. Architecture A is local, read-only and knowledge-pack-only; Architecture B is remote-service-only and blocked until EIL provides tested per-user identity over HTTP MCP. Neither may centralize behind a privileged EIL service credential.

Given the current clarification that in-house software is allowed and EIL appears acceptable, **Architecture A is the recommended default pending confirmation of the exact EIL change/approval path**. P1 belongs behind EIL's existing read-safe MCP boundary. A separately deployed harness runtime is not part of the pilot.

## Explicit non-goals

- replacing EIL storage, indexing or retrieval;
- copying all enterprise content into pack files;
- allowing plugins to bypass identity, policy, telemetry or approval;
- autonomous production changes or self-approval;
- hidden chain-of-thought capture;
- individual employee productivity scoring;
- a universal workflow language before two real workflows prove common needs;
- microservices, Kafka, a graph database or a second vector store without measured need.

## Sources

- [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) and its [architecture](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)
- [Enterprise Intelligence Layer](https://github.com/ashark-ai-05/enterprise-intelligence-layer)
- Local observability baseline: `../enterprise-ai-effectiveness-observatory/README.md`
