# Product and Organizational Model

## The job to be done

Enterprise teams need agents to perform multi-step work across trusted knowledge and tools without granting a model ambient authority. The product must make useful context easy to find, actions easy to compose, and every consequential step attributable and reversible where possible.

The harness converts an intent into a governed run:

```text
intent → plan → resolve packs → authorize → retrieve evidence → act in sandbox
       → verify → approve mutation → publish artifact → observe outcome
```

## Product principles

1. **Evidence before action.** Plans and mutations cite the source versions that justified them.
2. **Authority is not intelligence.** A capable model receives only the capabilities approved for this run.
3. **Plans are inspectable data.** The model may propose a plan; deterministic policy admits each step.
4. **Read and write are different trust tiers.** Search is not permission to mutate.
5. **External proof closes work.** Tests, CI, merge, deployment, Jira or incident systems determine outcomes.
6. **Model-neutral, tool-neutral, store-neutral.** Stable contracts isolate providers.
7. **Progressive autonomy.** Repeated evidence can reduce friction for a workflow class, never silently broaden authority.
8. **Tenant isolation and purpose limitation.** Access is evaluated for a user, purpose, pack and step.

## Primary users

| User | Need | Surface |
|---|---|---|
| Developer | Find the right architecture and safely change code | IDE/CLI run console |
| Service owner | Diagnose incidents across tickets, runbooks and code | workflow + evidence view |
| Analyst | Produce cited analysis from approved corpora | pack search + export |
| Platform engineer | Publish reusable capabilities and workflows | catalogs + policy simulator |
| Knowledge owner | Curate and promote reliable information packs | pack registry |
| Security/audit | Prove who accessed or changed what and why | audit + observability |
| Executive | Understand accepted outcomes, speed, quality and risk | aggregate cockpit |

## Initial workflows

### 1. Change implementation

Read Jira and Confluence, search code, draft acceptance criteria, obtain approval, patch in a sandbox, run tests, and open a reviewable change. This is the recommended pilot because evidence and outcome systems are clear.

### 2. Incident investigation

Search runbooks, past incidents, service ownership and code; gather telemetry through tools; propose hypotheses and mitigations. Production actions require separate approval and existing operational controls.

### 3. Enterprise research

Search a bounded portfolio of packs, compare sources, generate a cited brief, and retain its resource/version manifest for later reproduction.

## Organizational operating model

- **Platform Engineering** owns the harness runtime, contracts, reliability and paved road.
- **EIL team** owns source ingestion, retrieval quality, citations, ACL correctness and freshness.
- **Observability team** owns event conformance, retention, lineage and outcome measurement.
- **Domain teams** own workflow recipes, evaluation cases and pack contents for their domain.
- **Source owners** remain responsible for source ACLs and deletion.
- **Security/Privacy/Risk** own policy classes, sensitive payload rules and approval tiers.
- **Model/tool providers** publish capability manifests and pass conformance testing.

Packs and workflows move through `draft → validated → approved → published → deprecated → revoked`. Promotion requires an owner, evaluation results, data classification, compatibility range, expiry/review date and rollback path.

## Scale model

Scale by separating control, execution and knowledge planes:

- stateless regional API/control replicas admit and schedule work;
- workers pull tenant-scoped jobs and run in isolated sandboxes;
- EIL independently scales retrieval and ingestion;
- Observability independently scales receipt and analytics processing;
- quotas apply by tenant, team, workflow, capability and model budget;
- backpressure pauses admission instead of dropping evidence;
- asynchronous runs checkpoint after every step and resume from durable state.

Do not promise arbitrary agent swarms. Most enterprise work benefits more from deterministic parallel steps, bounded fan-out and explicit joins. Nested agents should be a capability with hard depth, concurrency, token and time budgets.

## Product risks and countermeasures

| Risk | Consequence | Countermeasure |
|---|---|---|
| “Pack” becomes another knowledge store | stale copies and conflicting ACLs | manifest-first packs; EIL resolves content |
| Plugin flexibility bypasses controls | invisible exfiltration or mutation | all capabilities pass one policy/execution gateway |
| Natural-language plans become authority | prompt injection drives actions | typed plans + deterministic admission |
| Central platform becomes bottleneck | teams route around it | stable SDK/MCP surfaces, self-service catalogs, SLOs |
| One giant workflow DSL | slow delivery and lock-in | begin with typed step graph and a small step vocabulary |
| Telemetry captures sensitive content | legal/privacy harm | metadata-first events, content references, reveal audit |
| ROI becomes employee scoring | trust and governance failure | outcome/team cohorts; prohibit individual rankings |
| Pack publication implies permanent access | revoked users retain data | runtime authorization and short-lived retrieval grants |

## Falsification conditions

Stop or change direction when evidence invalidates the product premise:

- If finer inline filters do not improve retrieval/context over whole-corpus EIL, stop before packs. If named packs do not add reuse and setup value over equivalent inline filters, keep saved filters as a small EIL feature and do not build a registry.
- If real usage produces only knowledge packs and teams do not adopt governed tool packs, move saved/scoped pack views into EIL and retire the separate harness rather than preserving a thin coordination layer.
- If real caller identity cannot propagate through target clients and MCP to EIL, block shared-service rollout; do not compensate with a privileged service credential and harness-side ACLs.
- If corporate proxy or approved-install constraints exclude the dependency model, change the reference adapter rather than weakening the runtime-neutral contracts.

## Success measures

- time from intent to first cited, useful artifact;
- verified outcome rate and first-pass acceptance;
- retrieval citation correctness, freshness and ACL escape rate;
- plan rejection and policy violation rates by reason;
- active versus wait time, tool retries and tokens per accepted outcome;
- deterministic attribution and telemetry coverage;
- pack reuse, staleness and rollback rate;
- platform availability and queue latency;
- user-reported trust and usefulness without surveillance concerns.
