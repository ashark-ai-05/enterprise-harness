# Architecture and Technology Choices

## Context

The harness sits between user intent and enterprise capabilities. It must coordinate long-running, resumable workflows while preserving EIL authorization and producing Observability evidence. It must not become a privileged bypass around either platform.

Corporate endpoints cannot install unapproved runtimes. The architecture therefore defaults to **zero new endpoint software**. Existing approved clients are presentation/control surfaces; all authoritative harness code executes in approved enterprise infrastructure. A reviewed stdio transport adapter is optional only if corporate approval explicitly permits the same installation shape as EIL.

## Logical architecture

```mermaid
flowchart TB
  subgraph Experience
    CLI[Approved Copilot / Amp / IDE]
    WEB[Approved browser / portal]
    API[Approved automation client]
  end
  EDGE[Enterprise access gateway<br/>OIDC/OAuth + remote MCP/HTTPS]
  subgraph Control_Plane[Harness control plane]
    GW[Identity + admission]
    PLAN[Planner + typed plan compiler]
    POL[Policy / approval engine]
    CAT[Pack, workflow + capability catalogs]
    RUN[Run state machine + scheduler]
  end
  subgraph Execution_Plane[Isolated execution plane]
    WK[Workers]
    SB[Sandbox providers]
    TG[Tool/MCP gateway]
    MG[Model gateway]
  end
  subgraph Knowledge_Plane[EIL]
    RES[Pack resolver]
    SEARCH[ACL-aware search/fetch]
  end
  subgraph Evidence_Plane[Enterprise Observability]
    REC[Authenticated receipt]
    LIN[Lineage + outcomes]
  end
  CLI --> EDGE
  WEB --> EDGE
  API --> EDGE
  EDGE --> GW
  GW --> PLAN --> POL --> RUN
  CAT --> PLAN
  RUN --> WK
  WK --> SB
  WK --> TG
  WK --> MG
  WK --> RES --> SEARCH
  GW --> REC
  PLAN --> REC
  POL --> REC
  WK --> REC
  REC --> LIN
```

## Endpoint and access boundary

The default endpoint contains no harness runtime, local database, local EIL corpus, plugin engine, long-lived credential or sandbox. Approved-client configuration contains only the managed service URL, public metadata and an enterprise authentication flow.

The enterprise access gateway authenticates the human/workload through enterprise SSO, binds tenant/groups/device posture/purpose/session, terminates only approved remote MCP/HTTPS, and preserves the original principal in a signed audience-bound delegation token for EIL and tools. No client-supplied user header is trusted.

The harness service identity authenticates the workload to EIL, but EIL authorization evaluates the delegated end-user principal and source ACL. If an approved client cannot perform the required enterprise auth and remote interface, that client remains unsupported until its vendor or the approved gateway supplies it. A local stdio bridge may be considered only as a reviewed, pinned artifact with no policy logic or authority; never fetch-and-run it and never use an ad-hoc shim.

## Managed-service topology

Deploy initially as one regional modular service with separately scalable roles: access/API and remote MCP facade; catalog and plan/policy admission; scheduler and durable run state; isolated read/model/tool workers; later sandbox workers; and a transactional Observability exporter. Use private network paths to EIL, model/tool gateways, PostgreSQL, object storage and Observability. Egress is deny-by-default and capability-specific.

## Runtime contract

1. Authenticate user/service and bind tenant, groups, purpose and device/workload context.
2. Resolve workflow, pack and capability versions to immutable digests.
3. Produce a typed plan containing inputs, dependencies, claimed capabilities, budgets and expected evidence.
4. Evaluate policy. Insert human approval steps where required; rejected capabilities never reach the model tool schema.
5. Execute admitted steps in a tenant-scoped worker/sandbox. Every side effect receives an idempotency key.
6. Emit signed step lifecycle events and artifact digests to Observability.
7. Verify outputs using deterministic validators or external systems of record.
8. Checkpoint the run and publish a result/evidence manifest.

For knowledge retrieval, the resolved lock contains identities and digests only. `view(pack, caller) = lock(pack) ∩ readable(caller, at read time)`. No view or authorization result is cached across principals.

## Plugin model

Adopt DeepSeek Harness's composability idea, not unrestricted in-process plugins.

A plugin is a signed package plus a declarative manifest:

- identity, version, publisher and software-bill-of-materials digest;
- capability types and typed input/output schemas;
- network, filesystem, secret, data-class and mutation requirements;
- supported deployment modes and isolation level;
- event schemas, health checks and conformance tests;
- minimum harness/API versions and rollback information.

Plugin classes:

- **connector** — source/system adapter;
- **retriever** — EIL pack/search/fetch adapter;
- **tool** — model-callable or deterministic capability;
- **model** — approved inference route;
- **sandbox** — execution isolation provider;
- **policy** — admission/approval extension;
- **workflow** — reusable typed step graph;
- **presenter** — CLI/UI rendering only.

Trust tiers:

1. pure in-process transforms with no I/O;
2. out-of-process tools with bounded filesystem/network access;
3. enterprise services reached through authenticated gateways;
4. mutation capabilities requiring scoped credentials and approval.

All I/O crosses host-owned gateways. A plugin cannot emit directly to arbitrary networks, mint credentials, weaken policy, or suppress telemetry. Hot unload is not a phase-one requirement; reproducible process restart is safer.

## Workflow representation

Use a versioned typed DAG, not arbitrary model-generated code. Core step types:

`resolve_pack`, `search`, `fetch`, `model`, `tool`, `sandbox_job`, `approval`, `validate`, `publish`, `wait`, `join`.

Each step declares:

- stable ID, type, version and dependencies;
- principal/purpose and requested capabilities;
- typed inputs and outputs with classification;
- retry, timeout, idempotency and compensation behavior;
- token, cost, concurrency and wall-time budgets;
- approval and evidence requirements;
- telemetry content policy.

The planner may propose this graph. Schema validation, capability resolution and policy admission are deterministic. Dynamic expansion is allowed only through a bounded `expand` result with fan-out and depth limits.

## Control-plane persistence

Start with PostgreSQL as authority for tenants, identities, catalogs, policy bindings, workflow/run state, approvals, leases and idempotency. Store immutable manifests, large artifacts and encrypted payloads in approved object storage. Use PostgreSQL job leasing/outbox patterns for the pilot; add a broker only after queue latency, throughput or blast-radius measurements justify it.

Do not store EIL document bodies or duplicate Observability event facts in harness tables. Keep references and delivery receipts.

## Technology recommendation

| Concern | Initial choice | Reason |
|---|---|---|
| Runtime/API | TypeScript on the enterprise-approved server Node LTS | aligns with EIL/Observability and MCP; no endpoint Node requirement |
| Contracts | JSON Schema + generated TypeScript types | language-neutral, versionable, policy-friendly |
| API | Remote MCP/HTTPS for approved clients; REST/JSON and SSE for managed portals/automation | zero-install reach with resumable server-side runs |
| Tool protocol | MCP behind a server-side harness gateway | interoperability without endpoint plugins or making MCP the policy boundary |
| Telemetry | OpenTelemetry + versioned harness semantic contract | vendor-neutral transport; domain-specific facts remain explicit |
| State | PostgreSQL | transactions, leases, outbox, JSON, mature operations |
| Artifacts | approved S3-compatible object store | immutable manifests and large evidence objects |
| Policy | embedded policy adapter; evaluate OPA/Rego in phase 0 | avoid committing before enterprise policy ownership and latency are known |
| Sandbox | provider interface over approved container/job platform | isolation and portability; corporate endpoint execution is prohibited |
| UI | thin web console consuming the same API | no alternate business logic |
| Packaging | OCI artifacts plus signed manifests/SBOM | provenance, scanning and promotion |

Python and other language tools run server-side out of process behind contracts. Rust is appropriate only for measured hot paths or isolation helpers, not as an up-front architectural commitment.

## EIL integration

Required EIL interfaces:

- `packs.resolve(pack_digest, principal, purpose)`;
- `search(query, scope_token, filters, limit)`;
- `fetch(resource_ref, scope_token)`;
- `explain(result_set)` and citation metadata;
- freshness, revocation and source-version checks;
- ingestion/build status for pack publishers.

Search remains deterministic and LLM-free. The harness chooses when to retrieve and how to use results; it does not re-rank around EIL ACL enforcement. Returned context records the pack, query, result/resource versions and authorization decision.

The harness must propagate the calling user/workload identity end-to-end. It must never proxy all users through an EIL service identity whose broader access is then narrowed only by harness filters. That design would turn the harness into a confused deputy and replace EIL's fail-closed boundary with application logic that can fail open.

## Observability integration

Emit metadata-first canonical events for run, plan, step, model call, tool call, retrieval, approval, artifact, policy decision, error and outcome link. Every event carries tenant, run/trace/span IDs, producer/schema versions, observed/received time, idempotency key, capture policy, resource/capability digests and attribution evidence.

Payload bodies remain outside telemetry by default. Capture digests, classifications and references. The harness cannot declare its own business outcome successful; Observability joins external evidence such as tests, CI, review, merge and deployment.

## Deployment and scaling

Begin as a modular monolith with separately runnable API, scheduler and worker roles. Scale workers by queue/capability/tenant class. Use leases and fencing for execution; transactional outbox for commands/events; dead-letter state plus operator replay; per-tenant quotas; regional data residency; graceful draining; and kill switches by plugin, workflow, tenant and model.

Only split services where scaling, ownership, isolation or regulatory boundaries demand it. The first likely separations are sandbox execution and high-volume event export, not catalogs.

Corporate laptops are never a worker pool. Long-running jobs survive client disconnects, and reconnecting clients resume from durable run state after reauthorization. There is no offline content mode: a corpus snapshot on the laptop would reintroduce revocation, retention and exfiltration problems the central design avoids.

## Failure semantics

- At-least-once delivery, idempotent step effects.
- A step is never marked successful without its required evidence.
- Unknown completion after timeout enters `reconcile_required`; do not blindly retry a mutation.
- Revoked pack/capability versions stop new steps; in-flight policy decides cancel versus drain.
- EIL unavailable: fail closed for required knowledge; do not use stale worker-cached context unless the pack explicitly permits a bounded server-side snapshot.
- Observability unavailable: durable server-side outbox buffering within limits; stop consequential mutations if audit delivery exceeds policy threshold.

## Injection separation invariant

Tool grants are a pure function of the signed manifest, caller, policy and approval state. Retrieved content, model output and tool output cannot participate in computing authority. A step containing broadly writable sources such as Jira comments or Confluence pages must not simultaneously hold a high-risk mutation capability; split reading and acting across an approval boundary.
