---
title: "Enterprise Harness Decision Memo — Prove the Pack Plane Before Building a Harness"
status: proposed
created: 2026-08-16
reviewers: [Codex CLI independent review, Hermes synthesis]
scope: design only; no runtime implementation authorized
---

# Enterprise Harness Decision Memo

## Decision

**Do not authorize a separate Enterprise Harness runtime or product build now.**

Retain this repository as the architecture, contract, governance, and decision record. Treat the programme as three separately gated products:

1. **P1 — information packs:** named, versioned, EIL-backed retrieval scopes;
2. **P2 — governed execution:** durable orchestration, approval-bound tool use, sandbox work, and reconciliation;
3. **P3 — evidence:** metadata-first receipts and outcome attribution in Enterprise Observability.

Only P1 is eligible for the first experiment. It belongs inside EIL's existing read-safe tool boundary, and P3 instruments it. P2 is **deferred** until P1 proves value and there is repeated demand for a shared, side-effecting workflow that existing approved clients and enterprise gateways cannot meet.

This is not a retreat from the long-term harness concept. It prevents building a platform around an unproven abstraction and preserves the architecture needed if demand earns it.

## Confirmed design boundaries

| Domain | Owner | Must not own |
|---|---|---|
| EIL | Source ingestion, normalized content, source ACLs, retrieval, citations, freshness/coverage | Workflow authority, approval truth, tool mutation policy |
| Information pack | Relevance selection: declared source selectors and content-free resolution locks | Content copies, credentials, a permission grant, a second index |
| Future Harness | Run state, deterministic admission, capability grants, budgets, approval checkpoints, idempotency/reconciliation | Source ACL authority, source-of-record outcomes, a duplicate model/MCP gateway |
| Enterprise Observability | Append-only metadata receipts, lineage, coverage, cost/effectiveness analysis | Inline authorization, raw enterprise content by default, declaring work successful |
| Systems of record | Jira, Confluence, Git, CI/CD, deployment, production approvals and outcomes | Harness run state |

**Invariant:** a knowledge pack only narrows an EIL-authorized query. It is never an ACL, a cached entitlement, or evidence that a caller may fetch a document. EIL re-authorizes each fetch for the real caller.

## Product model

A pack is a reusable statement of relevance: “for this purpose, search these governed Jira projects, Confluence spaces, repositories, paths, quality tiers, and freshness limits.” It is not a portable data bundle.

The first pack type is deliberately small:

```text
manifest identity + owner + version + review/expiry
+ declared selectors + retrieval/freshness policy
+ evaluation references + manifest digest
→ EIL resolves an ordered content-free lock of resource/revision identities
→ normal EIL ranked search applies the lock as a predicate
→ EIL applies caller ACL and validity predicates in every retrieval arm
→ EIL fetch re-authorizes every selected document
```

A lock makes evidence identity reproducible only to the degree EIL can resolve the referenced source revision. It does not make historical source content magically available. The first schema must therefore declare mismatch behaviour: `refuse`, `live-fallback` (explicitly non-reproducible), or an approved historical-source capability. Offline/snapshot exports are excluded from the pilot.

## First falsifiable workflow

> **Jira change request → scoped EIL retrieval across selected Jira, Confluence, and one repository → cited change plan and acceptance criteria → immutable metadata-only evidence manifest.**

Use an already-approved agent/IDE as the user-facing model loop. The pilot has no new agent UI, orchestration daemon, generic workflow DAG, tool pack, sandbox, model-gateway replacement, approval engine, source mutation, or production action.

For a sampled eligible request, execute equivalent retrieval forms:

- **A:** whole authorized corpus;
- **B:** the finest equivalent inline selectors;
- **C:** a named/versioned information pack.

Arm C must use the same ranked, ACL-filtered EIL retrieval path as A/B with lock IDs as an additional predicate. It must not return all lock members or bypass ranking. The assigned arm is shown to the user/model; shadow arms record only permitted metadata such as identities, ranks, coverage, latency, and citations.

## Corporate approval reality

“No arbitrary software install” does **not** mean “a local stdio process requires no approval.” A repository checkout, pinned lockfile, Node runtime, transitive dependencies, compiled JavaScript, local data directory, and MCP client configuration are endpoint artifacts.

The local P1 path may proceed only with evidence that the **exact** EIL artifact, launch model, dependencies, configuration, and proposed first-party modification are permitted on the corporate laptop. “EIL seems acceptable” is a working hypothesis, not an approval. Failure at this gate stops the pilot; it does not justify building a remote harness as a shortcut.

A future shared service is separately blocked on:

- verified per-user/delegated identity through remote MCP/HTTPS to EIL;
- adversarial ACL and revocation tests in shared-serving mode;
- at least one approved client that supports enterprise authentication without a new endpoint runtime;
- an approved service platform, private network path, retention/residency controls, and operating owner.

A broad EIL service identity plus harness-side filtering is prohibited.

## Decision gates

### Gate 0 — feasibility and authority

Proceed only with written evidence that:

- the exact EIL pilot artifact and change path are approved;
- named data owners approve the selected sources and evaluation use;
- no dynamic dependency fetch or additional endpoint runtime is required;
- a reviewed metadata-only receipt route into Observability exists;
- an owner governs the manifest schema, versioning, and retirement path.

### Gate 1 — retrieval safety

Require:

- zero seeded ACL/cross-scope disclosures;
- per-document reauthorization on fetch;
- citation coverage for every enterprise-dependent claim;
- explicit limitation/abstention for stale, incomplete, or partial views;
- revocation and deleted-source tests;
- no raw prompts, source bodies, tool arguments, or hidden reasoning in receipts.

Any disclosure failure stops the pilot.

### Gate 2 — pack value

Use 3–5 developers in one team, independently authored change questions, at least two packs used by non-authors, and a pre-registered evaluation plan. Numeric thresholds are set after a baseline sample, not retrospectively.

Continue only if C adds value beyond B:

- no material independently judged required-evidence recall loss;
- less irrelevant context or setup effort without lower accepted-plan quality;
- demonstrable non-author reuse and comprehensible pack semantics;
- cited evidence is used in a meaningful share of pack-loaded runs.

If B and C are materially equivalent, retain improved inline/saved filters in EIL and close the pack-platform initiative.

### Gate 3 — a separate harness service is earned

Authorize P2 only when all are true:

- at least two teams need the same multi-step side-effecting workflow;
- existing approved clients cannot provide adequate durable state/approval handling;
- EIL delegated per-user identity passes adversarial shared-service tests;
- an authoritative approval system and an approved remote sandbox/job platform exist;
- Observability has a durable production sink and operating owner;
- a team accepts service SLO and on-call responsibility.

## Future target architecture (not a build authorization)

```text
Approved client
  → enterprise authentication
  → Future Harness service (admission, run state, budgets, approval bindings)
      → EIL (delegated real caller; read-only knowledge)
      → existing enterprise model/tool/MCP gateways (protocol + credentials)
      → approved remote sandbox/job platform
      → systems of record (authoritative mutation/outcome truth)
      → Enterprise Observability (append-only receipts + outcome links)
```

Every future side-effecting capability must declare idempotency scope, preconditions, exact approval binding, expiry/revocation, timeout ambiguity handling, reconciliation lookup, retry safety, compensation limits, and externally verified completion. An idempotency key alone is insufficient.

## Explicit non-goals until Gate 3

- a new daily agent runtime, UI, or endpoint daemon;
- an executable plugin registry or dynamic package loading;
- a generic workflow language/DAG;
- snapshot or offline packs;
- a second model gateway, MCP gateway, vector store, event store, or approval database;
- autonomous, merge, deploy, or production mutation;
- individual productivity ranking;
- inferring a successful business outcome from harness completion.

## Review record

- **Codex CLI independent review:** concluded that the current documents contain a credible future control-plane design but no present product case for building the service. Its recommendations shaped the gates, artifact-approval correction, experimental pack semantics, and deferred P2 architecture here.
- **Claude Code:** available locally but not authenticated (`Not logged in; run /login`), so it could not provide an independent review in this run. No Claude output is represented as evidence.
- **DeepSeek Harness:** retained only as an architectural reference. Its plugin composition and durable-event ideas are useful; its developer-preview status, local dependency/runtime model, and in-process extensibility do not meet the enterprise security or endpoint constraints.

## Decisions requested before any implementation

1. Confirm whether the exact EIL local artifact/change path can be approved on the corporate laptop.
2. Name the first team, source owners, and Jira change-planning workflow.
3. Approve the Gate 0 evidence package and metadata-only receipt route.
4. Confirm that the outcome is a design/measurement pilot, not a commitment to build P2.
