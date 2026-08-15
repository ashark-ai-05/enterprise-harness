# Delivery Plan and Decision Gates

No implementation is authorized yet. This document defines the sequence after approval.

## Phase 0 — validate constraints, contracts and product fit

Time-box a design validation using existing EIL and Observability demonstrations.

- Interview 3–5 target teams and select one change workflow.
- Trace identity, ACL, citation, deletion and revocation end to end.
- Specify pack, capability, workflow and event schemas.
- Build paper prototypes for CLI, pack registry and run evidence.
- Validate approved Node, Postgres, object store, sandbox, registry, policy and model gateway choices.
- Measure expected concurrency, run duration, artifact size, retrieval volume and telemetry volume.
- Threat-model prompt injection, plugins and mutations.
- Run a server-side falsification trial through an approved client: compare scoped knowledge packs with direct whole-corpus EIL on retrieval precision, context size and setup time. Do not build a registry unless the pack abstraction earns its complexity.
- Read and test EIL's actual coverage/freshness contracts (`coverage`, source health, evidence bounds) before specifying harness equivalents. Extend those contracts; do not create a second definition of completeness or staleness.
- Validate target clients can propagate the real caller identity through MCP. Record EIL per-user token and HTTP MCP delivery as a hard dependency for any shared index.
- Inventory which already-approved Copilot/Amp/IDE clients support enterprise-authenticated remote MCP or approved HTTPS integration. No endpoint installation is an admissible workaround.
- Confirm approved server runtime, artifact registry, deployment platform, private connectivity, SSO/OAuth delegation and corporate proxy paths.

**Gate:** scoped packs demonstrate measurable value; approve the product boundary, pack semantics, ownership, pilot workflow, risk controls and success metrics.

## Phase 1 — centrally hosted read-only walking skeleton plus measurement

Only after approval and the identity prerequisite: one approved client, one remote MCP/HTTPS endpoint, one user intent, one manifest-only pack, EIL search/fetch, one model route, typed read-only tools, durable server-side run state and Observability events. No endpoint runtime and no source mutations. Ship the counterfactual measurement at the same time: a sampled shadow whole-corpus retrieval arm plus a small run-level outcome holdout. This control cannot be reconstructed after packs become the default.

**Gate:** zero ACL escapes; complete citations; deterministic replay of plan/evidence; policy and telemetry coverage demonstrated; scoped-pack hit rate and recall loss meet agreed thresholds.

## Shared-service prerequisite

The earlier local rung is not viable on restricted corporate laptops. Do not deploy the centrally hosted pilot until EIL's per-user tokens and HTTP MCP transport are implemented and adversarially verified. This is a dependency on EIL's roadmap, not a harness feature that can be approximated with local ACL filters or endpoint shims.

## Phase 2 — sandboxed work

Add disposable code workspace, bounded patch/test steps, resumability, approval and evidence bundle. Outputs stay in the sandbox or a draft artifact.

**Gate:** isolation tests, retry/idempotency behavior, unknown-completion handling, kill switch and recovery exercise pass.

## Phase 3 — one governed enterprise mutation

Add one low-risk reviewable mutation, such as creating a draft pull request or Jira comment. Keep merge/deploy outside the harness.

**Gate:** source owner, Security and Risk approve; preview/approval/reconciliation evidence is complete; pilot users confirm value.

## Phase 4 — catalog and organizational scale

Add promotion workflows, tenant quotas, domain ownership, conformance suite, SLOs, regional deployment and cost/capacity controls. Expand workflows only when reusable patterns are proven.

## Pilot success criteria

- 100% of retrieved claims cite an authorized source/version;
- no cross-user, cross-team or cross-tenant ACL failures;
- at least 95% metadata telemetry coverage and explicit gap reporting;
- all consequential actions linked to identity, plan, approval and evidence;
- measurable reduction in time to a reviewable artifact without lower acceptance quality;
- users can explain why a result was retrieved and why an action was allowed;
- platform operating cost and latency fit agreed budgets;
- no individual productivity ranking or sensitive content captured outside policy.

## Decisions required before implementation

1. Confirm the harness boundary and the three-plane ownership model.
2. Confirm manifest-first packs and whether offline snapshot packs are required.
3. Select the pilot workflow and participating domain owner.
4. Select the enterprise artifact/signing registry for plugins and packs.
5. Confirm the policy engine approach and policy owner.
6. Confirm sandbox provider, model gateway and approved MCP topology.
7. Approve the metadata/content capture manifest and retention.
8. Set autonomy tiers and identify the first permitted mutation, if any.
9. Agree SLOs, quotas, budget limits and pilot success thresholds.
10. Confirm the EIL per-user HTTP identity milestone and name its owner/date before planning any shared-index phase.
11. Confirm at least one already-approved client can use the authenticated remote interface without installing harness software.

## Work explicitly excluded until approval

- runtime source files, package manifests and executable scripts;
- CI/CD pipelines or cloud resources;
- connector credentials or live enterprise integrations;
- plugin installation or registry publication;
- EIL/Observability schema changes;
- autonomous mutation behavior.
